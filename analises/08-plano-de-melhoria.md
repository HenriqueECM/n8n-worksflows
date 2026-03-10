# 08 - Plano de Melhoria Priorizado

## Alta prioridade (0-30 dias)
1. **Contrato unificado e versionado** (`contract_version`, `status`, `meta`, `erro`).
2. **Observabilidade mínima**: `proposal_id`, `request_id`, latência por nó e erro categorizado.
3. **Resiliência HTTP IA**: retry exponencial com jitter + idempotência.
4. **Hardening de segurança**: redaction de logs, revisão de mensagens de erro e rotação de credenciais.

## Média prioridade (31-60 dias)
1. **Padronização de prompts e governança** (catálogo e versionamento).
2. **Refatoração dos 12 nós de dados gerais** para reduzir acoplamento e esforço operacional.
3. **Dashboards e alertas SRE** (SLO de sucesso e latência).

## Baixa prioridade (61-90 dias)
1. **Arquitetura assíncrona opcional** para alto volume.
2. **Piloto de vetorização** com base histórica curada.
3. **Teste de carga e chaos testing** para dependência IA.

## Critérios de aceite por etapa
- **Alta**: 0 divergência de contrato nos consumidores; rastreabilidade fim-a-fim disponível.
- **Média**: redução de esforço de manutenção e visibilidade operacional consolidada.
- **Baixa**: ganho mensurável de qualidade e custo na análise com IA.

## Dependências
- Definição oficial do contrato com back-end.
- Acesso a métricas reais de produção/homologação.
- Política corporativa de segurança para dados de proposta.
