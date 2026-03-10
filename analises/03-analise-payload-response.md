# 03 - Análise de Payload e Response

## Payload de entrada
## Estrutura inferida
- `body.aba`: `requisitos | cronograma | dados_gerais`.
- `body.dados.campo`: obrigatório quando `aba = dados_gerais`.
- `body.dados.contexto.dados_gerais`: obrigatório quando `aba = requisitos`.
- `body.dados.contexto.requisitos`: obrigatório quando `aba = cronograma`.

## Pontos positivos
- Validação de domínio (`abasPermitidas` e lista de campos permitidos).
- Sanitização básica contra padrões de prompt injection.

## Lacunas
1. **Sem schema formal (JSON Schema/OpenAPI)** para payload.
2. **Sem campo de versionamento** (`contract_version`).
3. **Sem política explícita para nulos e campos opcionais**.

## Response atual
- `dados_gerais`: `{"resultado": {"conteudo": ...}}`.
- `requisitos`: `{"resultado": {"requisitos": [...]}}`.
- erro: `{"erro": true, "mensagem": ...}` com HTTP 500.

## Problema real
A estrutura de sucesso varia por aba sem envelope comum de metadados.

## Proposta de contrato unificado
```json
{
  "contract_version": "1.0.0",
  "proposal_id": "string",
  "aba": "dados_gerais|requisitos|cronograma",
  "status": "success|error",
  "resultado": {
    "conteudo": {},
    "requisitos": []
  },
  "erro": {
    "codigo": "string",
    "mensagem": "string"
  },
  "meta": {
    "request_id": "string",
    "model": "string",
    "latency_ms": 0,
    "timestamp": "ISO-8601"
  }
}
```

## Regras recomendadas para nulos
- Campos ausentes: **não enviar** quando opcional.
- Campos obrigatórios sem valor: retornar erro 422 com detalhe técnico.
- Arrays sem itens: retornar `[]` explicitamente.

## Ponto em aberto
Não foi disponibilizada especificação oficial do contrato esperado pelo back-end consumidor.
