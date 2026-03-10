# 02 - Análise de Arquitetura

## Visão do fluxo
Entrada via `Webhook` (`POST /compass`) → validação/sanitização → roteamento por `aba`:
- `requisitos`: bifurca em lista vazia/com requisitos e chama IA.
- `dados_gerais`: roteia por 12 campos e chama IA específica por campo.
Saída final via `Respond to Webhook`, com caminho de erro dedicado.

## Pontos fortes
- **Separação por responsabilidade**: validação, roteamento, execução IA e parsing estão em nós distintos.
- **Fail-fast**: validação de campos obrigatórios interrompe fluxo cedo.
- **Padronização mínima de erro**: handler global evita falhas silenciosas.

## Fragilidades arquiteturais
1. **Acoplamento alto com prompts por nó**: 12 nós HTTP semelhantes para `dados_gerais` elevam custo de manutenção.
2. **Rota `cronograma` validada, mas sem trilha funcional no fluxo**: potencial inconsistência entre domínio aceito e implementação.
3. **Tratamento de exceções parcial**: parsing captura erro, porém não há classificação por tipo (transitório x permanente).
4. **Dependência síncrona da IA**: ausência de desenho assíncrono para picos.

## Riscos de integração
- Variação de estrutura de resposta por aba.
- Dependência de endpoint externo único (single external dependency).
- Falta de versionamento explícito da API interna.

## Melhorias recomendadas
- Consolidar nós HTTP de `dados_gerais` em padrão parametrizado (sub-workflow ou nó reutilizável).
- Implementar estratégia assíncrona opcional para cargas elevadas.
- Definir taxonomia de erros (400/422/429/5xx) com mapeamento consistente.
- Encapsular prompts/versionamento em artefato de configuração.

## Ponto em aberto
Não há evidência de ambientes separados (dev/hml/prod) no artefato do workflow.
