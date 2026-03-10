# 00 - Visão Geral da Análise

## Escopo avaliado
Análise da **Fase 1** do workflow n8n `fluxo proposta com IA`, considerando:
- Documentação em `docs/README.md`.
- Topologia visual do fluxo (captura de tela em `assets/`).
- Definições técnicas do workflow em `json/fluxo proposta com IA.json`.

## Diagnóstico consolidado

### O que está bem estruturado
1. **Entrada com validação inicial forte**: existe nó dedicado de validação/sanitização que verifica `aba`, campos permitidos e contexto mínimo por aba.
2. **Roteamento por domínio funcional**: separação explícita entre `requisitos` e `dados_gerais` via `Switch`.
3. **Padronização de saída**: há parse dedicado para resposta de IA e resposta via `Respond to Webhook`.
4. **Tratamento de erro padronizado**: existe `Error Handler Global` com retorno controlado ao cliente.

### Lacunas e riscos críticos
1. **Risco de inconsistência contratual**: retorno de `dados_gerais` entrega apenas `resultado.conteudo`, enquanto `requisitos` retorna estrutura diferente (`resultado.requisitos`).
2. **Escalabilidade limitada**: arquitetura atual processa 1 campo de `dados_gerais` por requisição; para 12 campos, depende de múltiplas chamadas externas.
3. **Dependência de IA sem robustez completa**: timeout configurado (90s), mas sem evidência explícita de retry/backoff, idempotência e circuit-breaker.
4. **Observabilidade parcial**: não há correlação explícita por `proposal_id` propagada em logs e resposta.
5. **Segurança operacional incompleta**: autenticação de entrada existe, porém faltam evidências de mascaramento de logs sensíveis e política de rotação de credenciais.

## Classificação por tipo

### Problemas reais
- Contrato de response heterogêneo entre abas.
- Ausência de padrão de idempotência na chamada externa à IA.
- Falta de rastreabilidade ponta a ponta por identificador de proposta.

### Hipóteses (dependem de validação com operação)
- Possível aumento de latência e custo por campo em `dados_gerais` (12 prompts diferentes).
- Possível risco de concorrência sob alta carga sem fila assíncrona.

### Melhorias opcionais
- Introduzir `contract_version` em payload/response.
- Criar camada de policy para prompts (governança e auditoria).
- Preparar arquitetura para vetorização incremental por proposta.

## Pontos em aberto
- Não há amostra real de payload/response de produção para medir aderência semântica e taxa de erro.
- Não há evidência de SLAs formais do endpoint de IA externo.
- Não há medição histórica de throughput/latência para dimensionamento.

## Recomendação executiva
Prosseguir para Fase 2 somente com um **hardening mínimo obrigatório**:
1. Contrato versionado e uniforme.
2. Correlação por `proposal_id` + métricas por nó.
3. Estratégia de retry/idempotência no HTTP Request.
4. Política de segurança de credenciais e logs mascarados.
