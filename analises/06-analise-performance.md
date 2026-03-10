# 06 - Análise de Performance e Escalabilidade

## Leitura de desempenho da arquitetura atual
- Processamento majoritariamente síncrono.
- Para `dados_gerais`, 1 chamada IA por campo da aba.
- Timeout de 90s por chamada externa.

## Gargalos prováveis
1. **Latência da IA** como principal componente de tempo total.
2. **Throughput limitado** por execução síncrona do webhook.
3. **Escalabilidade linear de custo** com número de requisições e tamanho de prompt.

## Hipóteses de impacto
- Sob pico, aumento de filas no n8n e crescimento de timeout percebido pelo cliente.
- Custo de tokens sensível a repetição de contexto em prompts longos.

## Recomendações
- Introduzir fila assíncrona para cenários de alto volume.
- Cache de contexto estável por `proposal_id` para reduzir payload repetido.
- Instrumentar métricas de latência p50/p95/p99 por nó HTTP.
- Definir limites de tamanho de entrada por aba/campo.
- Avaliar consolidação de prompts quando possível para reduzir round-trips.

## KPIs mínimos
- Sucesso fim-a-fim (%).
- Latência fim-a-fim (p50/p95/p99).
- Erro por categoria (4xx/5xx/timeout).
- Custo médio de tokens por proposta.

## Ponto em aberto
Sem dados históricos de carga real, a análise permanece preditiva.
