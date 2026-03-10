# 04 - Análise de IA via HTTP Request

## Estado atual
- Endpoint externo único de IA.
- Método `POST`, autenticação por header credential.
- Timeout configurado em **90s**.
- Prompt extenso e fortemente instrucional por nó.

## Pontos fortes
- Timeout explícito evita bloqueio infinito.
- Parsing pós-processamento remove markdown e tenta JSON estrito.

## Lacunas críticas
1. **Retry/backoff não explícito** para falhas transitórias (5xx/timeout).
2. **Idempotência ausente** (sem chave idempotente por `proposal_id + campo + hash_input`).
3. **Sem validação semântica robusta de output** (além de parse e chaves mínimas).
4. **Sem proteção de custo por token** (limites por requisição/tenant).

## Recomendações objetivas
- Aplicar política de retry exponencial com jitter (ex.: 3 tentativas, teto 8s).
- Definir `idempotency_key` no header da chamada externa.
- Adicionar validação de schema de saída antes do `Respond`.
- Registrar metadados de execução: modelo, latência, tamanho de prompt/response.
- Implementar fallback controlado (resposta de degradação) para indisponibilidade de IA.

## Modelo de classificação de erro recomendado
- `IA_TIMEOUT` → 504
- `IA_UNAVAILABLE` → 503
- `IA_INVALID_OUTPUT` → 422
- `INPUT_VALIDATION_ERROR` → 400/422

## Ponto em aberto
Não há SLA/SLO formal do fornecedor do endpoint para calibrar timeout e retries.
