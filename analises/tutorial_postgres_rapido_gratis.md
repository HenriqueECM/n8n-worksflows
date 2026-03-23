# Tutorial rápido (grátis) — PostgreSQL para prompts (sem enrolação)

## Objetivo
Subir um PostgreSQL grátis/local em minutos e criar a tabela `ai_prompts` para a versão DB+cache.

---

## Opção 1 (recomendada) — Docker local + DBeaver

### 1) Subir PostgreSQL com Docker
```bash
docker run --name prompts-pg \
  -e POSTGRES_USER=admin \
  -e POSTGRES_PASSWORD=admin123 \
  -e POSTGRES_DB=promptsdb \
  -p 5432:5432 \
  -d postgres:16
```

### 2) Conectar no DBeaver
- Host: `localhost`
- Port: `5432`
- Database: `promptsdb`
- User: `admin`
- Password: `admin123`

> Sim, pode usar só DBeaver com banco local do projeto. Para começar rápido, essa é a forma mais prática.

### 3) Criar tabela de prompts
```sql
CREATE TABLE IF NOT EXISTS ai_prompts (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  campo TEXT NOT NULL,
  versao INT NOT NULL,
  prompt_template TEXT NOT NULL,
  ativo BOOLEAN NOT NULL DEFAULT FALSE,
  created_at TIMESTAMP NOT NULL DEFAULT NOW(),
  updated_at TIMESTAMP NOT NULL DEFAULT NOW(),
  UNIQUE (campo, versao)
);

CREATE INDEX IF NOT EXISTS idx_ai_prompts_campo_ativo
  ON ai_prompts (campo, ativo);
```

> Se der erro no `gen_random_uuid()`, execute antes:
```sql
CREATE EXTENSION IF NOT EXISTS pgcrypto;
```

### 4) Inserir exemplo
```sql
INSERT INTO ai_prompts (campo, versao, prompt_template, ativo)
VALUES
('project_description', 1, '=## Role ... {{ $json.body.dados.conteudo }}', true);
```

### 5) Query que o n8n deve usar
```sql
SELECT campo, versao, prompt_template
FROM ai_prompts
WHERE campo = $1
  AND ativo = true
ORDER BY versao DESC
LIMIT 1;
```

---

## Opção 2 — Grátis em nuvem (sem docker)
- Neon (PostgreSQL serverless) ou Supabase (Postgres + painel).
- Crie projeto free-tier, copie string de conexão e use no n8n.

---

## Integração rápida no n8n
1. Criar credential PostgreSQL.
2. Node `Postgres` com query acima.
3. Passar `campo` como parâmetro (`$json.prompt_key` ou `$json.body.dados.campo`).
4. Se não retornar linha, cair em erro `PROMPT_NOT_FOUND`.

---

## Pronto para produção (mínimo)
- Separar banco por ambiente (`dev`, `hml`, `prod`).
- Não editar prompt direto em produção sem versão nova.
- Logar `prompt_version` na resposta/meta.
