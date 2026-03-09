# Análise técnica do `fluxo.json` (n8n)

## Diagnóstico do fluxo atual

Com base no workflow atual, os principais pontos observados foram:

1. **Múltiplos nós HTTP quase idênticos para IA (17 no total, sendo 12+ campos de Dados Gerais)**, com prompts longos e repetição de configuração.
2. **Timeout alto (300s) por requisição** em praticamente todos os nós de IA, impactando experiência em caso de lentidão.
3. **`allowUnauthorizedCerts: true` em nós HTTP**, o que reduz significativamente a segurança do canal HTTPS.
4. **Parsing e limpeza de resposta em nós `Code` separados**, o que dificulta padronização e observabilidade.
5. **Ausência de estratégia explícita de resiliência** (retry/backoff/circuit breaker/controle de concorrência por chave de negócio).

---

## Objetivos de arquitetura recomendados

- **Manter 12 IAs especializadas por campo** (se requisito de negócio), mas reduzir custo operacional.
- **Aumentar segurança** (TLS estrito, autenticação forte, saneamento de entrada/saída).
- **Melhorar performance percebida** (menor p95/p99 de resposta e menos timeout).
- **Melhorar governança** (versionamento de prompts, telemetria e auditoria por execução).

---

## Proposta de nova arquitetura (mantendo 12 prompts)

## 1) Camada de orquestração (n8n)

- **Webhook de entrada único** recebe payload da aba e metadados (id proposta, aba, campo, versão de prompt, idioma, usuário).
- **Validação inicial de schema** (campos obrigatórios, tamanho máximo, caracteres inválidos).
- **Roteamento por mapa de configuração** (não por 12 nós manuais):
  - Um `Code` node monta o objeto `{ campo, promptTemplateId, outputSchema }`.
  - Um único nó de chamada HTTP reutilizável consome esse objeto.
- **Padronização de pós-processamento** em um único parser JSON robusto.

## 2) Catálogo de prompts versionado

- Tirar prompts gigantes de dentro dos nós e mover para:
  - banco/config service, ou
  - arquivo versionado (Git) + carregamento por chave.
- Cada campo aponta para um `promptTemplateId` (ex.: `dados_gerais.descricao_projeto.v3`).
- Permite rollback rápido e A/B testing sem duplicar nó.

## 3) Gateway de IA interno (recomendado)

Em vez do n8n chamar diretamente o endpoint de LLM para cada regra, criar um **serviço intermediário** (`ai-gateway`) com:

- injeção segura de credenciais (vault/secret manager),
- timeout e retry padronizados,
- validação de schema de resposta,
- cache semântico/opcional por hash de entrada,
- rate-limit por usuário/sistema,
- logs com tracing (`request_id`, `proposal_id`, `field_id`).

Assim, o n8n fica simples e o comportamento técnico crítico sai do low-code.

## 4) Modelo de execução para performance

Para **Dados Gerais (12 campos)**:

- Se o front precisa resposta campo a campo: manter chamadas individuais, mas com:
  - conexão HTTP keep-alive,
  - timeout menor (ex.: 20–45s),
  - retries curtos (2 tentativas com backoff exponencial + jitter).
- Se pode responder em lote: usar **fan-out/fan-in controlado**:
  - dispara as 12 tarefas em paralelo com limite de concorrência (ex.: 3–4),
  - agrega resultados no final,
  - retorna parcial quando um campo falhar.

## 5) Segurança

- **Remover `allowUnauthorizedCerts: true`** e validar cadeia TLS corretamente.
- Exigir autenticação forte no webhook:
  - HMAC assinado, OAuth2 client credentials ou mTLS.
- Input hardening:
  - limite de tamanho de texto,
  - blocklist de padrões maliciosos,
  - normalização de unicode.
- Output hardening:
  - JSON schema validation por campo,
  - rejeitar resposta fora do contrato.
- Segredos fora do fluxo (credenciais só no cofre/credential manager).

## 6) Observabilidade e SLO

Medir por campo e por aba:

- latência p50/p95/p99,
- taxa de erro por tipo (timeout, parse, schema, 5xx),
- custo por requisição/token,
- taxa de retrabalho manual.

Definir SLO inicial:

- `p95 < 8s` por campo de Dados Gerais,
- erro técnico `< 1%` por dia,
- timeout `< 0.5%`.

---

## Design prático para manter 12 especializações com menos complexidade

Você **não precisa remover as 12 IAs**. A otimização está em transformar 12 nós em **12 configurações**:

- `field_configs` (JSON):
  - `field_id`
  - `prompt_template_id`
  - `output_schema`
  - `temperature`, `max_tokens`, `timeout_ms`
- Loop no n8n (ou subworkflow) percorre a configuração e executa o mesmo pipeline técnico.

Benefícios:

- mantém especialização de negócio,
- reduz manutenção/erros de cópia,
- facilita testes e versionamento.

---

## Plano de migração sugerido (baixo risco)

1. **Fase 1 (1–2 dias):** hardening imediato
   - remover `allowUnauthorizedCerts`,
   - reduzir timeout e configurar retry básico,
   - adicionar validação de entrada/saída.
2. **Fase 2 (3–5 dias):** refatoração estrutural
   - criar subworkflow genérico de chamada IA,
   - extrair prompts para catálogo versionado,
   - centralizar parser.
3. **Fase 3 (1 semana):** maturidade operacional
   - ai-gateway, cache, rate-limit, tracing,
   - métricas e alertas por SLO.

---

## Riscos atuais prioritários

- risco de segurança por TLS permissivo,
- risco de indisponibilidade por timeout elevado e sem política de retry sofisticada,
- risco de inconsistência por múltiplos pontos de parse/prompt duplicados.

---

## Conclusão

A melhor estratégia é **manter a lógica de 12 campos especializados**, porém migrar de um desenho “12 nós quase iguais” para um desenho **configurável + subworkflow/gateway**. Isso preserva qualidade de geração por campo e melhora significativamente segurança, performance e governança.
