# Ajustes de estrutura — v4/v5 para importação no n8n

## Problema identificado
Os arquivos `v4` e `v5` anteriores estavam em formato de **blueprint JSON** (documentação técnica), não no formato de export oficial de workflow do n8n.

Por isso, o n8n não importava.

## O que foi corrigido

### 1) Estrutura de export n8n aplicada
Os arquivos foram recriados no padrão esperado pelo n8n, com chaves como:
- `name`
- `nodes`
- `connections`
- `settings`
- `versionId`
- `meta`
- `id`
- `tags`

## Arquivos atualizados
- `json/Fluxo dados Gerais _ produção.v4.db-cache.json`
- `json/Fluxo dados Gerais _ produção.v5.rag.json`

### 2) v4 (DB+Cache) ajustado
- Mantida base funcional da v3 (importável).
- Substituído nó `Preparar Prompt Dinâmico` por `Resolver Prompt (DB+Cache)`.
- Esse nó:
  - tenta ler `prompt_template` do payload/entrada;
  - usa cache global (`workflow static data`);
  - renderiza placeholders (`conteudo` e `dados_gerais`);
  - retorna metadados (`prompt_version`, `cache_hit`, `prompt_source`).

### 3) v5 (RAG) ajustado
- Base da v4 + novo nó `RAG Context Builder`.
- O nó monta `rag_context` a partir de:
  - `body.dados.rag_context`, ou
  - `body.dados.rag_items` (lista de fontes/textos).
- O nó `Resolver Prompt (DB+Cache)` injeta `rag_context` no prompt final (com placeholder `{{RAG_CONTEXT}}` ou append automático).

## Observação importante
Para ambiente real com PostgreSQL/MongoDB:
- Use os tutoriais em `analises/` para infra e modelagem.
- Esses fluxos v4/v5 agora importam no n8n, e servem como base para plugar os nós de banco do seu ambiente.
