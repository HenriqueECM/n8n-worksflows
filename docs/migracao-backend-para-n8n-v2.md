# Guia detalhado: mudanças no backend para integrar com o fluxo n8n v2

Este documento descreve, passo a passo, como adaptar o backend (FastAPI), banco (PostgreSQL) e integração com front (Next.js) para funcionar com o novo fluxo `json/fluxo proposta com IA v2.json`.

> **Contexto importante:**
> O contrato padrão atual de envio para o n8n é:
>
> ```python
> def build_n8n_payload(aba: str, proposal_id: int, dados: dict) -> dict:
>     return {
>         "aba": aba,
>         "proposal_id": proposal_id,
>         "dados": dados,
>     }
> ```
>
> O fluxo v2 atual **aceita esse formato** e adiciona metadados internamente quando necessário (`request_id`, `schema_version`).

---

## 1) O que mudou no n8n e por que isso impacta o backend

No fluxo v2, a etapa de validação/sanitização ficou mais rígida e agora:

1. Valida `aba` com valores permitidos.
2. Exige estrutura mínima por aba:
   - `dados_gerais`: `dados.campo` válido + `dados.conteudo`
   - `requisitos`: `dados.contexto.dados_gerais`
   - `cronograma`: `dados.contexto.requisitos`
3. Gera/propaga rastreabilidade (`request_id`, `proposal_id`, `schema_version`) para logs e chamadas HTTP internas.
4. O webhook do fluxo novo está em `compass-v2`.

**Conclusão prática:**
- Seu payload base continua válido.
- O backend precisa principalmente garantir consistência dos campos por aba e apontar para o endpoint novo.

---

## 2) Alterações mínimas obrigatórias no FastAPI

## Passo 2.1 — Ajustar URL de destino do n8n

Antes:
- `/webhook/compass`

Depois:
- `/webhook/compass-v2`

> Se você usa ambiente por variável, atualizar por `N8N_WEBHOOK_PATH=compass-v2`.

## Passo 2.2 — Garantir contrato por aba antes de enviar

Mesmo com validação no n8n, valide no backend para falhar cedo com erro claro (422), por exemplo:

- `dados_gerais`
  - `dados.campo` obrigatório
  - `dados.conteudo` obrigatório
- `requisitos`
  - `dados.contexto.dados_gerais` obrigatório
  - `dados.estado_atual.requisitos` pode ser vazio, mas precisa existir
- `cronograma`
  - `dados.contexto.requisitos` obrigatório

## Passo 2.3 — Manter builder padrão (com pequeno hardening)

Você pode manter seu builder atual, mas recomenda-se validar tipos:

```python
def build_n8n_payload(aba: str, proposal_id: int, dados: dict) -> dict:
    if not isinstance(aba, str):
        raise ValueError("aba deve ser string")
    if not isinstance(proposal_id, int):
        raise ValueError("proposal_id deve ser int")
    if not isinstance(dados, dict):
        raise ValueError("dados deve ser dict")

    return {
        "aba": aba,
        "proposal_id": proposal_id,
        "dados": dados,
    }
```

## Passo 2.4 — (Recomendado) Enviar `request_id` opcional

O n8n já cria fallback, mas enviar do backend melhora observabilidade ponta a ponta.

Exemplo:

```python
import uuid

def build_n8n_payload(aba: str, proposal_id: int, dados: dict) -> dict:
    return {
        "aba": aba,
        "proposal_id": proposal_id,
        "dados": dados,
        "request_id": str(uuid.uuid4()),
        "schema_version": "2.0.0",
    }
```

> Isso **não quebra** o fluxo atual e melhora correlação com logs de API/DB/front.

---

## 3) Exemplos corretos de payload (compatíveis com v2)

## 3.1 Aba `dados_gerais`

```python
payload = build_n8n_payload(
    aba="dados_gerais",
    proposal_id=proposal_id,
    dados={
        "campo": proposal_field.name,
        "conteudo": proposal_text.text,
    },
)
```

## 3.2 Aba `requisitos`

```python
payload = build_n8n_payload(
    aba="requisitos",
    proposal_id=proposal_id,
    dados={
        "contexto": {
            "dados_gerais": dados_gerais,
        },
        "estado_atual": {
            "requisitos": requisitos,
        },
    },
)
```

## 3.3 Aba `cronograma` (referência)

```python
payload = build_n8n_payload(
    aba="cronograma",
    proposal_id=proposal_id,
    dados={
        "contexto": {
            "requisitos": requisitos_gerados_ou_existentes,
        },
    },
)
```

---

## 4) Implementação sugerida no FastAPI (passo a passo)

## Passo 4.1 — Criar modelos Pydantic por aba

Objetivo: validar estrutura do payload antes de chamar n8n.

```python
from pydantic import BaseModel, Field
from typing import Any, Dict, List, Literal

class DadosGeraisDados(BaseModel):
    campo: str
    conteudo: str

class RequisitosDados(BaseModel):
    contexto: Dict[str, Any]
    estado_atual: Dict[str, Any]

class CronogramaDados(BaseModel):
    contexto: Dict[str, Any]

class N8NPayloadBase(BaseModel):
    aba: Literal["dados_gerais", "requisitos", "cronograma"]
    proposal_id: int
    dados: Dict[str, Any]
    request_id: str | None = None
    schema_version: str | None = None
```

## Passo 4.2 — Validar por aba antes de enviar

```python
def validate_payload_for_aba(payload: dict) -> None:
    aba = payload.get("aba")
    dados = payload.get("dados") or {}

    if aba == "dados_gerais":
        if not dados.get("campo") or not dados.get("conteudo"):
            raise ValueError("dados_gerais exige dados.campo e dados.conteudo")

    elif aba == "requisitos":
        if not (dados.get("contexto") or {}).get("dados_gerais"):
            raise ValueError("requisitos exige dados.contexto.dados_gerais")
        if "requisitos" not in ((dados.get("estado_atual") or {})):
            raise ValueError("requisitos exige dados.estado_atual.requisitos")

    elif aba == "cronograma":
        if not (dados.get("contexto") or {}).get("requisitos"):
            raise ValueError("cronograma exige dados.contexto.requisitos")
```

## Passo 4.3 — Chamar n8n com timeout e tratamento de erro

```python
import httpx

async def send_to_n8n(payload: dict, n8n_url: str, auth_header: str) -> dict:
    timeout = httpx.Timeout(50.0)  # maior que timeout interno do fluxo (45s)

    async with httpx.AsyncClient(timeout=timeout) as client:
        resp = await client.post(
            n8n_url,
            headers={"Authorization": auth_header},
            json=payload,
        )
        resp.raise_for_status()
        return resp.json()
```

---

## 5) Ajustes recomendados no PostgreSQL

## Passo 5.1 — Persistir rastreabilidade de execução

Criar tabela de log técnico (exemplo):

- `request_id` (nullable no início, idealmente not null depois)
- `proposal_id`
- `aba`
- `status` (`received`, `processed`, `error`)
- `latency_ms`
- `error_message`
- `created_at`

## Passo 5.2 — Índices

- Índice por `proposal_id`
- Índice por `created_at`
- Índice único por `request_id` (quando adotar idempotência)

---

## 6) Ajustes recomendados no Next.js

1. Propagar `request_id` para backend (header ou body).
2. Exibir esse ID em mensagens de erro para suporte.
3. Em operação assíncrona futura, usar polling por `request_id` para status.

---

## 7) Plano de migração sem downtime

1. Subir fluxo v2 no n8n (sem desligar v1).
2. Backend com feature flag:
   - `% tráfego -> compass-v1`
   - `% tráfego -> compass-v2`
3. Monitorar taxa de erro e latência por aba.
4. Quando estável, migrar 100% para v2.
5. Desativar v1 após janela de estabilização.

---

## 8) Checklist de aceite (prático)

- [ ] `dados_gerais` com `campo` válido retorna sucesso.
- [ ] `dados_gerais` sem `campo` retorna erro de validação claro no backend.
- [ ] `requisitos` com `dados.contexto.dados_gerais` retorna sucesso.
- [ ] `cronograma` sem `dados.contexto.requisitos` falha corretamente.
- [ ] `proposal_id` presente em todos os envios.
- [ ] Endpoint aponta para `compass-v2`.
- [ ] Logs possuem `request_id`/`proposal_id` (quando informado).

---

## 9) Resposta direta à sua pergunta

**Com o contrato atual que você mostrou, não é obrigatório reestruturar todo backend.**

Você precisa:
1. apontar para o webhook `compass-v2`;
2. garantir validação por aba no FastAPI antes de enviar;
3. (recomendado) incluir `request_id` e `schema_version` no payload para rastreabilidade e evolução de contrato;
4. registrar `request_id/proposal_id` no banco para suporte operacional.

Assim, o payload atual continua compatível com o novo fluxo e você ganha segurança/observabilidade com mudanças incrementais.
