# 05 - Análise de Segurança

## Controles identificados
- Webhook com autenticação de cabeçalho.
- Sanitização básica contra padrões comuns de prompt injection.
- HTTPS no endpoint de IA.

## Riscos de segurança
1. **Sanitização insuficiente para cenários avançados** (obfuscação, payload multi-idioma, encoding).
2. **Possível exposição de detalhes internos**: handler de erro carrega `detalhe_interno` no processamento.
3. **Sem evidência de mascaramento de dados sensíveis em log**.
4. **Sem política visível de rotação de credenciais/API keys**.

## Melhorias prioritárias
- Introduzir política de **redaction** de PII/sigilosos em logs.
- Limitar retorno de erro ao cliente (sem dados internos sensíveis).
- Implementar assinatura/HMAC e/ou mTLS entre back-end e n8n quando aplicável.
- Rotação automática de segredos e segregação por ambiente.
- Definir allow-list de origem e rate limit no endpoint.

## Recomendação de baseline
- OWASP ASVS para APIs.
- Auditoria trimestral de prompts, credenciais e trilha de acesso.
- Testes de segurança focados em prompt injection e abuso de payload.

## Ponto em aberto
Não há matriz de classificação de dados do payload para definir política de retenção e mascaramento.
