# Resumo — v6 RAG + DB + Cache (com Supabase, Embedding e Mongo Atlas)

Arquivo entregue:
- `json/Fluxo dados Gerais _ produção.v6.rag-db-cache-completo.json`

## O que foi implementado

1. **Supabase para prompts (PostgreSQL)**
- Nó: `Supabase - Buscar Prompt Ativo` (HTTP Request)
- Busca prompt por `campo` (`prompt_key`), ordenando por versão desc e limite 1.

2. **Cache de prompt**
- Nó: `Cache Lookup Prompt` (Code)
- Nó: `Cache Store Prompt` (Code)
- Estratégia: `workflow static data` com TTL de 15 minutos por chave `prompt:{campo}:active`.

3. **Embedding real**
- Nó: `Embedding - OpenAI` (HTTP Request)
- Modelo: `text-embedding-3-small`

4. **Busca vetorial no MongoDB Atlas (Data API)**
- Nó: `Mongo Atlas Vector Search` (HTTP Request)
- Pipeline com `$vectorSearch` filtrando por `campo`.

5. **Montagem de prompt final com contexto RAG**
- Nó: `Montar Prompt Final` (Code)
- Injeta contexto semântico (`rag_context`) no prompt antes da chamada da IA.

6. **Chamada IA única + parser robusto + resposta padrão**
- Nó: `IA - Prompt Dinâmico` (HTTP Request)
- Nó: `Code in JavaScript` (parser)
- Nó: `Respond to Webhook`

## Códigos removidos/simplificados
- Removidos blocos de code desnecessários de mapa gigante de prompts hardcoded por campo.
- Mantidos apenas os códigos essenciais: validação, cache, resolução de fonte, anexar embedding, montagem de prompt final, parser e erro de roteamento.

## Fluxo lógico final
`Webhook -> Validar entrada -> Switch1 -> Cache Lookup -> IF cache_hit`
- `true`: `Resolver Prompt Fonte -> Embedding -> Vector Search -> Montar Prompt -> IA -> Parser -> Response`
- `false`: `Supabase Prompt -> Cache Store -> Resolver Prompt Fonte -> Embedding -> Vector Search -> Montar Prompt -> IA -> Parser -> Response`
- fallback do `Switch1` vai para `Erro de validação/roteamento`

## Variáveis de ambiente esperadas
- `SUPABASE_URL`
- `MONGODB_DATA_API_URL`
- `MONGODB_DATASOURCE`
- `MONGODB_DATABASE`
- `MONGODB_COLLECTION`

## Credenciais esperadas no n8n
- `credencial supabase` (HTTP Header Auth)
- `credencial openai` (HTTP Header Auth)
- `credencial mongo data api` (HTTP Header Auth)
- `credencial genai` (já usada no fluxo)
