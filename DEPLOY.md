# Deploy HTTPS — Alar

Objetivo: API e frontend públicos com HTTPS, para demo externa.

**Stack escolhida (gratuita para começar):**
- API → [Railway](https://railway.app) (Docker já existe)
- Front → [Vercel](https://vercel.com) (Next.js nativo)

Banco e Storage continuam no **mesmo Supabase** (decisão da Fase 0). Staging separado só depois da URL pública.

Não commitar secrets. Tudo no painel do host.

---

## Ordem (obrigatória)

1. Subir a **API** no Railway → anotar a URL `https://….up.railway.app`
2. Subir o **front** no Vercel apontando para essa URL
3. Voltar no Railway e colocar `CORS_ORIGINS` = URL do Vercel
4. Testar: `/health`, login, um caso, Sentry

---

## 1. Contas

Crie (GitHub já serve para login):

1. https://railway.app — **Login with GitHub**
2. https://vercel.com — **Continue with GitHub**

Autorize a org **projetoALAR** nos dois.

---

## 2. API no Railway

Repo: [projetoALAR/workspace-juridico-backend](https://github.com/projetoALAR/workspace-juridico-backend)

**Branch:** `feat/integracao-api` (é a que tem a API atual; `master` está atrás).

1. New Project → **Deploy from GitHub repo** → `workspace-juridico-backend`
2. Settings → **Branch** = `feat/integracao-api` (não use `master`)
3. Root Directory vazio (o repo já é a API)
4. O `railway.toml` usa o `Dockerfile` e health em `/health`
5. **Variables** (Settings → Variables):

| Variável | Valor |
|----------|--------|
| `NODE_ENV` | `production` |
| `DATABASE_URL` | pooler Supabase (`:6543` + `?pgbouncer=true`) — igual ao `.env` local |
| `DIRECT_URL` | session pooler / direto (`:5432`) — **obrigatório** para `prisma migrate` |
| `JWT_SECRET` | string longa aleatória (**nova**, não a de local) |
| `JWT_EXPIRES_IN` | `7d` |
| `CORS_ORIGINS` | `https://localhost` temporário; depois a URL do Vercel |
| `AUTH_ALLOW_PUBLIC_REGISTER` | `false` |
| `AUTH_ADMIN_EMAIL` | `admin@alar.com.br` |
| `AUTH_ADMIN_PASSWORD` | senha forte (só cria admin se a tabela `Usuario` estiver vazia) |
| `AUTH_ADMIN_NOME` | `Administrador` |
| `SUPABASE_URL` | URL do projeto |
| `SUPABASE_KEY` | `service_role` |
| `OPENAI_API_KEY` | se o chat for usado na demo |
| `OPENAI_MODEL` | `gpt-4o-mini` |
| `SENTRY_DSN` | o DSN já usado no local |
| `SENTRY_ENVIRONMENT` | `production` |
| `SENTRY_ENABLE_TEST_ENDPOINT` | `false` |

`PORT` o Railway injeta sozinho — não fixe 3001.

6. Deploy. Confira: `https://SEU-SERVICO.up.railway.app/health` → `"status":"ok"` e `"sentry":true`

Gerar `JWT_SECRET`:

```bash
node -e "console.log(require('crypto').randomBytes(48).toString('hex'))"
```

---

## 3. Frontend no Vercel

Repo: [projetoALAR/workspace-juridico-frontend](https://github.com/projetoALAR/workspace-juridico-frontend)

**Branch:** `feat/integracao-api`

1. Add New → Project → importe `workspace-juridico-frontend`
2. Framework: Next.js (automático)
3. **Environment Variables:**

| Variável | Valor |
|----------|--------|
| `API_URL` | `https://SEU-SERVICO.up.railway.app` (sem barra no final) |
| `NEXT_PUBLIC_API_URL` | a mesma URL da API |
| `NEXT_PUBLIC_ALLOW_REGISTER` | `false` |
| `NEXT_PUBLIC_SENTRY_DSN` | o DSN |
| `NEXT_PUBLIC_SENTRY_ENVIRONMENT` | `production` |
| `NEXT_PUBLIC_SENTRY_ENABLE_TEST` | `false` |

4. Deploy. Anote a URL `https://….vercel.app`

O browser fala só com o Next (`/api/backend/v1` e `/api/auth/*`). O cookie `alar_token` fica httpOnly + `secure` em produção.

---

## 4. CORS

No Railway, atualize:

```
CORS_ORIGINS=https://SEU-APP.vercel.app
```

Se tiver domínio custom, coloque os dois separados por vírgula, sem espaço extra:

```
CORS_ORIGINS=https://app.seudominio.com,https://SEU-APP.vercel.app
```

Redeploy da API (ou Restart).

---

## 5. Checklist de fumaça

- [ ] `GET …/health` → `ok`, `database: up`, `sentry: true`
- [ ] Abrir o front em HTTPS → tela de login
- [ ] Login admin
- [ ] Abrir um caso / lista
- [ ] (Opcional) erro real no Sentry com `SENTRY_ENVIRONMENT=production`

Login local (`localhost`) **não** usa o mesmo `JWT_SECRET` de produção — são ambientes diferentes.

---

## 6. Depois (não bloqueia este passo)

- Domínio próprio (Vercel + Railway)
- Segundo projeto Supabase (`alar-staging`) quando for mostrar a cliente
- Smoke de backup real (RUNBOOK §6)

---

## Problemas comuns

| Sintoma | Causa típica |
|---------|----------------|
| Health `database: down` | `DATABASE_URL` errada ou IP não liberado no Supabase |
| Migrate falha no boot | falta `DIRECT_URL` |
| Login 401 / cookie some | front em HTTP, ou `NODE_ENV` não é `production` no Vercel |
| CORS no browser | `CORS_ORIGINS` sem a URL exata do Vercel (https, sem `/`) |
| Chat IA falha | `OPENAI_API_KEY` ausente no Railway |
| Página carrega, API 500 no proxy | `API_URL` no Vercel apontando para URL errada |

Supabase: em **Settings → Database → Network** (ou “Allow all IPs” no pooler) o Railway precisa alcançar o banco. Se estiver restrito por IP, libere `0.0.0.0/0` no pooler ou use IPv6 — o pooler `aws-*.pooler.supabase.com` costuma funcionar sem allowlist extra.
