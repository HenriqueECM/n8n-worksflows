# Alterações aplicadas no novo fluxo n8n (v2)

## Arquivo base utilizado
- Base original: `json/fluxo proposta com IA.json`
- Novo fluxo gerado: `json/fluxo proposta com IA v2.json`

## Principais mudanças no fluxo

### 1) Contrato e versionamento
- Incluído `schema_version` no payload de entrada (default `2.0.0` quando não informado).
- Inclusão/propagação de `request_id` e `proposal_id` para rastreabilidade ponta a ponta.

### 2) Validação e sanitização
- Nó **Validação e Sanitização** reforçado para:
  - validar `aba` contra lista permitida;
  - validar pré-condições por aba (`dados_gerais`, `requisitos`, `cronograma`);
  - sanitizar conteúdo textual e objetos/arrays recursivamente;
  - limitar conteúdo de texto para reduzir risco de abuso/prompt injection.

### 3) Observabilidade
- Adicionado nó **Observabilidade e Contexto** entre validação e roteamento.
- Esse nó centraliza metadados de execução (`request_id`, `proposal_id`, `aba`, `schema_version`, timestamp inicial) e gera log estruturado.
- Todos os HTTP Request receberam headers de rastreabilidade:
  - `x-request-id`
  - `x-proposal-id`

### 4) Robustez de integração com IA
- Ajustado timeout de todos os nós de IA para `45s` (reduz fila presa em timeout alto).
- Parse dos resultados em **Parse Dados Gerais** e **Parse Requisitos** agora:
  - retorna `meta.finished_at`;
  - mantém erro explícito quando resposta não estiver em JSON válido.

### 5) Tratamento global de erros
- Atualizado **Error Handler Global** para responder com:
  - `erro`, `request_id`, `timestamp`, `mensagem`, `detalhe_interno`.
- Inclusão de log estruturado com `request_id` para facilitar correlação no monitoramento.

### 6) Endpoint v2
- Webhook atualizado para path `compass-v2` no novo fluxo.
- Mantida autenticação por header auth.

---

## Sugestões fora do n8n (backend FastAPI + PostgreSQL + Next.js)

## FastAPI (API intermediária)
1. **Idempotência por `request_id`**
   - Criar middleware para exigir `request_id` único por submissão.
   - Em reenvio com mesmo ID, retornar resposta anterior (sem reprocessar no n8n).

2. **Contrato versionado**
   - Validar payload com Pydantic por `schema_version`.
   - Permitir convivência entre `v1` e `v2` para migração gradual.

3. **Timeout e retry controlado**
   - Configurar retry somente para erros transitórios (5xx/timeout).
   - Não repetir chamadas em erros de validação (4xx).

4. **Fila assíncrona (opcional para escala)**
   - Para cargas maiores, enviar para fila (ex.: Redis/Rabbit/Kafka) e processar assíncrono.

## PostgreSQL
1. **Tabelas de auditoria de execução**
   - `workflow_execution_log` com `request_id`, `proposal_id`, `aba`, status, latência, erro.
2. **Índices essenciais**
   - Índice por `request_id` (único), `proposal_id`, `created_at`.
3. **Retenção e LGPD**
   - Política de retenção de logs e mascaramento de dados sensíveis.

## Next.js
1. **UX de processamento assíncrono**
   - Exibir status por `request_id` (processando, concluído, erro).
2. **Tratamento de erro amigável**
   - Mostrar mensagem de negócio + código de suporte (correlacionado ao `request_id`).
3. **Telemetria de front**
   - Enviar `request_id` desde o cliente para rastrear toda a jornada.

---

## Próximos passos recomendados
1. Homologar o fluxo v2 em ambiente de teste com payloads reais.
2. Medir latência por aba e taxa de erro por tipo de campo.
3. Definir SLO (tempo de resposta e disponibilidade).
4. Evoluir para processamento assíncrono caso o volume aumente.
