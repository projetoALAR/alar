# Alar — Workspace Jurídico

Sistema full-stack para gestão de escritório jurídico: clientes, casos/processos, compromissos, documentos, equipe, chat com IA e inbox.

**Próximas etapas:** veja o [`ROADMAP.md`](./ROADMAP.md).  
**Operação diária:** [`RUNBOOK.md`](./RUNBOOK.md) (subir, admin, health, Sentry, backup).  
**Deploy HTTPS:** [`DEPLOY.md`](./DEPLOY.md) (Railway + Vercel).  
**Sentry:** [`SENTRY.md`](./SENTRY.md).

## Repositórios (org [projetoALAR](https://github.com/projetoALAR))

| Repo | Conteúdo |
|------|----------|
| [alar](https://github.com/projetoALAR/alar) | Este repo — README, Docker Compose e CI da raiz |
| [backend](https://github.com/projetoALAR/backend) | NestJS + Prisma + PostgreSQL |
| [frontend](https://github.com/projetoALAR/frontend) | Next.js (App Router) + Tailwind |

### Clone completo (Compose + CI)

```bash
git clone --recurse-submodules https://github.com/projetoALAR/alar.git
cd alar
# Se já clonou sem submodules:
git submodule update --init --recursive
```

Estrutura local esperada:

```
alar/
  workspace-juridico-backend/   # submodule → projetoALAR/backend
  workspace-juridico-frontend/  # submodule → projetoALAR/frontend
  docker-compose.yml
  README.md
```

## Pré-requisitos

- Node.js 20+
- Conta Supabase (PostgreSQL + Storage bucket `documentos`)
- (Opcional) chave OpenAI-compatible para o chat

## Setup rápido

### 1. Backend

```bash
cd workspace-juridico-backend
cp .env.example .env
# Preencha DATABASE_URL, DIRECT_URL, JWT_SECRET, SUPABASE_*, AUTH_ADMIN_PASSWORD, CORS_ORIGINS
npm install
npx prisma migrate deploy
npx prisma generate
npm run start:dev
```

API padrão: `http://localhost:3001` (`PORT` no `.env`).

### 2. Frontend

```bash
cd workspace-juridico-frontend
cp .env.example .env.local
# NEXT_PUBLIC_API_URL=http://localhost:3001
# NEXT_PUBLIC_ALLOW_REGISTER=false  (igual AUTH_ALLOW_PUBLIC_REGISTER)
npm install   # ou pnpm / yarn
npm run dev
```

App: `http://localhost:3000`

### 3. Docker Compose (API + Postgres + frontend)

Na raiz deste projeto:

```bash
docker compose up --build
```

- Postgres: `localhost:5432`
- API: `localhost:3001`
- Frontend: `localhost:3000` (serviço `web`)

Admin bootstrap do Compose: `admin@alar.com.br` / `AlarAdminChangeMe1` (troque em produção).

Ajuste `SUPABASE_*` e `OPENAI_API_KEY` no compose / `.env` da raiz para Storage e IA real. Sem OpenAI, o Compose sobe com `CHAT_ALLOW_MOCK=true`.

Para desenvolvimento com hot-reload, use `npm run start:dev` (API) + `npm run dev` (front) em vez do Compose.

## Papéis (RBAC)

| Papel | Capacidade |
|-------|------------|
| `ADMIN` | Tudo + gestão de equipe e criação de usuários |
| `ADVOGADO` | CRUD clientes/casos/docs |
| `ASSISTENTE` | Leitura + upload de docs / compromissos |

Com `AUTH_ALLOW_PUBLIC_REGISTER=false`, crie usuários em **Configurações** (admin) ou via `POST /v1/auth/usuarios`.

## Health

`GET /health` — status da API e do banco (sem versão). Rotas de negócio: `/v1/...` (login, clientes, casos, etc.).

## Documentos

Uploads vão para o bucket Supabase `documentos`. A API devolve **URLs assinadas** (1h). Deixe o bucket **privado** no painel do Supabase.

## Scripts úteis

| Onde | Comando | O quê |
|------|---------|--------|
| backend | `npm run start:dev` | API em watch |
| backend | `npm test` | testes unitários |
| backend | `npm run test:e2e` | E2E (health + fluxo crítico) |
| backend | `npm run lint` | ESLint |
| frontend | `npm run dev` | Next.js |
| frontend | `npm run build` | build de produção |

## Segurança (checklist)

- [ ] `JWT_SECRET` forte
- [ ] `AUTH_ADMIN_PASSWORD` forte (≥ 8)
- [ ] `AUTH_ALLOW_PUBLIC_REGISTER=false` em produção
- [ ] `CORS_ORIGINS` com a URL real do frontend
- [ ] Bucket `documentos` privado
- [ ] Não commitar `.env` / `.env.local`
