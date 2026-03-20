# Fluxo v3 — Otimização de HTTPs e centralização de prompts

## Arquivo entregue
- `json/Fluxo dados Gerais _ produção.v3.json`

## O que foi alterado na v3

### 1) Redução de 11 nós HTTP para 1 nó HTTP
- Antes: 11 nós `HTTP Request` (um por campo).
- Agora: 1 nó único `IA - Prompt Dinâmico`.
- Benefício: manutenção muito mais simples (URL, auth, timeout e ajustes de integração em um único lugar).

### 2) Centralização dos prompts em um nó de preparação
- Adicionado nó `Preparar Prompt Dinâmico` (Code) que:
  - identifica o `campo` solicitado;
  - seleciona o template de prompt correspondente em um mapa interno;
  - injeta `conteudo` e `dados_gerais` no template;
  - retorna `prompt` pronto para o nó HTTP único.

### 3) Switch principal mantido para governança de rota
- O `Switch` continua validando o campo.
- Cada saída válida do `Switch` converge para `Preparar Prompt Dinâmico`.
- O **Fallback do segundo Switch** continua indo para `Erro de validação/roteamento`.

### 4) Cadeia final preservada
- `IA - Prompt Dinâmico` → `Code in JavaScript` (parser robusto) → `Respond to Webhook`.
- Mantém a estratégia de parse resiliente e resposta com metadados.

---

## Sugestões práticas para fase 2 (próximos passos)

### A) Tirar prompts do JSON e versionar externamente
**Problema atual:** prompt map está dentro do node Code (arquivo grande e difícil de revisar).

**Sugestão:**
1. Criar fonte única de prompts (ex.: arquivo JSON versionado no repo, banco, ou endpoint interno).
2. No fluxo, carregar por `campo` em tempo de execução.
3. Adotar versionamento de prompt (`prompt_version`) no retorno para rastreabilidade.

### B) Cache de prompt por campo
- Se prompts ficarem externos, usar cache em memória/Redis para reduzir latência e chamadas repetidas.

### C) Observabilidade mínima por execução
- Registrar: `campo`, tempo total, tempo IA, status, código de erro, tamanho de entrada.
- Isso ajuda no cenário de alto volume citado nas observações.

### D) Padronização de contrato de erro
- Consolidar códigos (`VALIDATION_ERROR`, `UNSUPPORTED_ABA`, `UNSUPPORTED_CAMPO`, `PARSE_ERROR`, `PROMPT_NOT_FOUND`) e documentar para frontend/backend.

### E) Estratégia de rollout seguro
1. Rodar v2 e v3 em paralelo (canary).
2. Comparar taxa de erro, tempo de resposta e qualidade de saída por campo.
3. Migrar 100% somente após estabilidade.

---

## Riscos e pontos de atenção da v3
- Como os prompts agora estão centralizados em um Code node, esse nó ficou grande; qualquer ajuste deve ser feito com cuidado.
- Idealmente, mover os prompts para armazenamento externo para reduzir risco de manutenção.
- Testar em n8n se todos os placeholders de contexto foram substituídos conforme esperado para cada campo.
