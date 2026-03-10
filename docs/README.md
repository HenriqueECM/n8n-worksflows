# Projeto: Workflow n8n de Análise de Propostas (Fase 1)

Este repositório contém a documentação e a arquitetura da **Fase 1: Análise Técnica e Funcional** do workflow de automação para análise e validação de formulários de proposta.

## 📝 Contexto do Projeto
O objetivo é automatizar a validação de propostas enviadas a comitês/comissões. A arquitetura segue o fluxo:
* **Front-end:** Envio dos dados.
* **Back-end Intermediário:** Padronização em contrato JSON.
* **n8n:** Recebimento do payload e orquestração.
* **IA (via HTTP Request):** Processamento e resposta padronizada.
* **Retorno:** Entrega do resultado ao back-end no formato esperado.

### Escopo Atual da Análise:
1. **Aba "Dados Gerais":** 12 campos com regras específicas de conteúdo, padrão e consistência.
2. **Aba "Requisitos":** Processamento e estruturação de requisitos da proposta.

---

## 🎯 Objetivo desta Etapa
Realizar uma análise técnica e funcional completa para identificar o que está bem estruturado, lacunas, riscos arquiteturais e melhorias necessárias antes da execução, preparando o fluxo para evolução futura.

### Premissas Importantes
* Uso de payload e response padrão.
* Chamada de IA própria via **HTTP Request** (gestão direta de tokens/credenciais).
* Foco em: Confiabilidade, Escalabilidade, Segurança e Observabilidade.

---

## 🔍 Blocos de Análise Obrigatória
A análise deve cobrir detalhadamente os seguintes pilares:

1.  **Arquitetura do fluxo:** Separação de responsabilidades e riscos de integração.
2.  **Contrato de payload e response:** Consistência, versionamento e tratamento de nulos.
3.  **Aba “Dados Gerais”:** Estrutura de validação dos 12 campos e previsibilidade.
4.  **Aba “Requisitos”:** Classificação, enriquecimento e redução de ambiguidade.
5.  **Uso da IA via HTTP Request:** Timeout, retry, idempotência e estabilidade da saída.
6.  **Vetorização e Base de Conhecimento:** Estratégia para massa histórica e busca semântica.
7.  **Segurança:** Autenticação, proteção de endpoints, criptografia e logs mascarados.
8.  **Performance e Escalabilidade:** Gargalos, paralelismo e processamento síncrono vs assíncrono.
9.  **Observabilidade e Operação:** Rastreabilidade por `proposal_id`, alertas e métricas.
10. **Governança e Evolução:** Documentação de nós, separação de ambientes e versionamento.

---

## 📂 Entregáveis (Pasta `/analises`)
Os resultados desta fase estão organizados em documentos padronizados:

* **`00-visao-geral-analise.md`**: Consolidado da análise.
* **`01-resumo-executivo.md`**: Resumo executivo para tomada de decisão.
* **`02-analise-arquitetura.md`**: Detalhamento técnico da estrutura do workflow.
* **`03-analise-payload-response.md`**: Definição e crítica aos contratos de dados.
* **`04-analise-ia-http-request.md`**: Gestão da comunicação com o modelo próprio.
* **`05-analise-seguranca.md`**: Mapeamento de riscos e protocolos de proteção.
* **`06-analise-performance.md`**: Estudo de carga, latência e custos (tokens).
* **`07-analise-vetorizacao.md`**: Planejamento para inteligência evolutiva.
* **`08-plano-de-melhoria.md`**: Roadmap priorizado (Alta, Média, Baixa).

---

## 📋 Regras de Análise
* **Fidelidade:** Não inventar detalhes inexistentes.
* **Sinalização:** Itens sem informação são marcados como "ponto em aberto".
* **Diferenciação:** Distinção clara entre Problema Real, Hipótese e Melhoria Opcional.
* **Mindset:** Engenharia de Integração + Arquitetura de Software.

---
**Status:** Aguardando conclusão da documentação analítica.