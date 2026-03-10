# 07 - Análise de Vetorização e Base de Conhecimento

## Objetivo da vetorização na evolução
Aumentar consistência e qualidade analítica reaproveitando histórico de propostas, decisões e requisitos aprovados.

## Estratégia sugerida
1. **Fonte primária**: dados gerais, requisitos, pareceres e decisões finais.
2. **Chunking orientado ao domínio**: separar por seção/campo sem misturar contextos.
3. **Metadados obrigatórios**: `proposal_id`, área, data, status, versão do contrato.
4. **Busca híbrida**: semântica + filtros estruturados.

## Casos de uso iniciais
- Recuperar propostas similares para apoiar geração de requisitos.
- Sugerir riscos/mitigações com base em histórico validado.
- Melhorar explicabilidade com referências recuperadas.

## Riscos
- Contaminação por dados desatualizados sem controle de versão.
- Viés por histórico de aprovações.
- Exposição de dados sensíveis em base vetorial sem governança.

## Requisitos mínimos de governança
- Política de retenção/expurgo.
- Versionamento dos embeddings.
- Auditoria de consultas e trilha de acesso.
- Camada de autorização por perfil/unidade.

## Ponto em aberto
A Fase 1 não traz definição de provedor vetorial nem critérios de qualidade para recuperação.
