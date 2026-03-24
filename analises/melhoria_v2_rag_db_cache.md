# Melhoria v2 — RAG + DB + Cache

Arquivo gerado:
- `json/Fluxo dados Gerais _ produção _ v2 rag+db+cache.json`

## Contexto
O arquivo com esse nome não existia no repositório no momento da execução. Foi usada como base a versão mais completa disponível (`v6`) e gerada uma versão v2 de melhoria com ajustes de robustez.

## Ajustes aplicados
1. **Criação da versão solicitada no caminho/padrão de nome pedido**.
2. **`Resolver Prompt Fonte` reforçado**:
   - retorna erro padronizado com `conteudo`, `erro` e `debug` quando não encontra prompt.
3. **Novo gate `IF Prompt Encontrado`**:
   - `true` -> segue para embeddings e pipeline RAG.
   - `false` -> responde direto no webhook sem custo de embedding/vector search.
4. Mantida arquitetura principal:
   - Supabase para prompt ativo
   - cache lookup/store
   - embedding de entrada
   - busca vetorial Mongo Atlas
   - montagem de prompt final
   - IA única + parser

## Benefício principal
Redução de custo e latência em erros de prompt (não chama embedding/vector quando prompt não existe).
