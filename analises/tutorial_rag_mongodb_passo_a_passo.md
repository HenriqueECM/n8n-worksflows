# Tutorial rápido — RAG com MongoDB (passo a passo)

## Objetivo
Guardar propostas + metadados + embeddings para busca semântica futura e melhorar geração da IA.

---

## Arquitetura mínima
1. Recebe proposta.
2. Quebra em chunks.
3. Gera embedding de cada chunk.
4. Salva no MongoDB (`proposal_chunks`).
5. Na geração futura: embedding da pergunta → busca vetorial top-k → injeta no prompt (`rag_context`).

---

## 1) Criar banco/coleções no Mongo
Coleções:
- `proposals`
- `proposal_chunks`
- `ai_runs`

### Exemplo documento `proposal_chunks`
```json
{
  "proposal_id": "P123",
  "chunk_id": "P123-01",
  "campo": "scope_project",
  "chunk_text": "texto do trecho...",
  "embedding": [0.012, -0.004, 0.998],
  "metadata": {
    "area": "Logística",
    "status": "aprovada"
  },
  "created_at": "2026-03-20T10:00:00Z"
}
```

---

## 2) Pipeline de ingestão (quando proposta for enviada)
1. Salvar proposta original em `proposals`.
2. Chunking do texto (500–1000 tokens por chunk).
3. Gerar embedding por chunk.
4. Salvar chunks + embedding em `proposal_chunks`.
5. Registrar telemetria em `ai_runs`.

---

## 3) Pipeline de recuperação (na geração com IA)
1. Gerar embedding da nova entrada.
2. Executar busca vetorial top-k (`k=5` recomendado).
3. Filtrar por `campo` e metadados relevantes.
4. Montar `rag_context` com os melhores trechos.
5. Injetar `rag_context` no prompt antes da chamada da IA.

---

## 4) Índice vetorial
Se usar MongoDB Atlas, configure Vector Search index para campo `embedding`.

Exemplo de configuração lógica:
- campo vetorial: `embedding`
- dimensões: depende do modelo escolhido (ex.: 1536)
- similaridade: cosine

---

## 5) Boas práticas (importante)
- Limitar contexto injetado (`max_context_chars`).
- Nunca mandar dados sensíveis sem anonimização.
- Guardar `sources` no `meta.debug` para auditoria.
- Se não encontrar contexto relevante, seguir sem RAG (fallback).

---

## 6) Sequência de rollout segura
1. Ativar ingestão primeiro.
2. Depois ativar recuperação para 10% das execuções.
3. Medir qualidade/latência.
4. Escalar progressivamente para 100%.
