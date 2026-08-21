# Runbook — Alar

Operação do dia a dia: subir, login admin, health e recuperação rápida.  
Sem secrets neste arquivo — use `.env` / `.env.local` / painel do provedor.

**Status operacional (atualizado 10/08/2026)**

| # | Seção | Status |
|---|--------|--------|
| 1 | Subir local (API + front) | OK |
| 2 | Criar / usar o admin | OK |
| 3 | Health check | OK |
| 4 | Logs e request id | OK (código) |
| 5 | Sentry | OK (DSN + Issue ALAR-1; flags de teste desligadas) |
| 6 | Backup / restore | Script `backup:smoke` pronto; instalar `pg_dump` / usar `DIRECT_URL` para rodar |
| 7 | Rotação de secrets | Checklist pronto; só usar se precisar rotacionar |
| 8 | Desligar servidores | OK (procedimento) |
| — | Docker Compose (opcional) | Não usado no fluxo diário |
| — | Deploy HTTPS (Fase 1 roadmap) | Em andamento — ver [`DEPLOY.md`](./DEPLOY.md) |
| 9 | SMTP local (Ethereal) | OK (script + teste no admin) |

**Demo para agência:** roteiro completo em [`DEMO.md`](./DEMO.md) (`npm run seed:demo` + script de 10–12 min).

---

## 1. Subir local — OK

### Pré-requisitos — OK
- [x] Node.js 20+
- [x] `.env` no backend
- [x] `.env.local` no frontend
- [x] Supabase com Postgres + bucket **privado** `documentos`

### Backend (API :3001) — OK

```bash
cd workspace-juridico-backend
npm install
npx prisma migrate deploy
npx prisma generate
npm run start:dev
```

Teste: `GET http://localhost:3001/health` → `status: "ok"`, `database: "up"`.  
Validado em smoke test (DB `up`).

### Frontend (app :3000) — OK

```bash
cd workspace-juridico-frontend
npm install   # ou: npx pnpm@9 install
npm run dev   # ou: npx pnpm@9 run dev
```

Abra: http://localhost:3000  
`NEXT_PUBLIC_API_URL` deve apontar para a API (ex.: `http://localhost:3001`).

Sessão atual: cookie httpOnly via BFF (`/api/auth/login`).

### Docker (API + Postgres local, opcional) — não usado no dia a dia

Na raiz do monorepo / repo `alar`:

```bash
docker compose up --build
```

- Postgres: `localhost:5432`
- API: `localhost:3001`  
Frontend continua com `npm run dev`.

---

## 2. Criar / usar o admin — OK

O admin é criado **só se a tabela `Usuario` estiver vazia** e existirem no `.env` do backend:

- `AUTH_ADMIN_EMAIL`
- `AUTH_ADMIN_PASSWORD` (≥ 8 caracteres)
- `AUTH_ADMIN_NOME` (opcional)

Sem senha válida → admin **não** é criado (log de aviso na API).

- [x] Login admin validado no ambiente local
- [x] Cadastro público desligado (`AUTH_ALLOW_PUBLIC_REGISTER=false`)

Com cadastro público desligado:
1. Faça login com o admin (valores no `.env` do backend)
2. Em **Configurações** (admin), crie os demais usuários  
   ou `POST /v1/auth/usuarios` com JWT de admin

---

## 3. Health check — OK

```http
GET /health
```

| Resposta | Significado |
|----------|-------------|
| `200` + `database: "up"` | API e banco ok |
| `503` + `database: "down"` | API no ar, **banco inacessível** |

Validado: `status: "ok"`, `database: "up"`.

### Se `/health` cair ou der 503

1. Conferir `DATABASE_URL` / `DIRECT_URL` no `.env`
2. Supabase: projeto pausado? senha do DB correta? IP/rede?
3. Reiniciar a API (`npm run start:dev`)
4. Ver logs no terminal (linha JSON com `requestId`)
5. Se Sentry estiver ativo (`SENTRY_DSN`), abrir o projeto no Sentry

Porta ocupada (`EADDRINUSE :::3001` / `3000`): encerrar o processo antigo ou mudar `PORT`.

---

## 4. Logs e request id — OK

Código ativo na API. Cada request HTTP gera um log JSON, por exemplo:

```json
{"requestId":"...","method":"GET","path":"/health","statusCode":200,"durationMs":12,"userId":null}
```

- Header de resposta: `x-request-id`
- Cliente pode enviar `x-request-id`; senão a API gera um UUID
- Com usuário autenticado, `userId` aparece no log

Use o `requestId` para correlacionar erro no Sentry / suporte.

---

## 5. Sentry (erros) — OK

- [x] Integração no backend (`SENTRY_DSN`) + `dotenv` antes do init
- [x] Integração no frontend (`NEXT_PUBLIC_SENTRY_DSN`)
- [x] Guia: [`SENTRY.md`](./SENTRY.md)
- [x] DSN no `.env` (API) e `.env.local` (front) — **não commitado**
- [x] `/health` → `"sentry": true`; bootstrap log `Sentry habilitado`
- [x] Teste API: `GET /debug/sentry` → 500 proposital
- [x] Página `/sentry-test` disponível com flag de teste
- [x] Confirmar Issue no painel [sentry.io](https://sentry.io) (ALAR-1) e desligar flags de teste

Sem DSN → Sentry fica desligado (app funciona normalmente).  
Opcional em local; **recomendado** em staging/prod.

| Onde | Variável |
|------|----------|
| Backend | `SENTRY_DSN` |
| Frontend | `NEXT_PUBLIC_SENTRY_DSN` |

Passo a passo completo: **[`SENTRY.md`](./SENTRY.md)**.

---

## 6. Backup / restore (Postgres) — documentado

- [x] Procedimento documentado abaixo
- [x] Smoke de backup (`npm run backup:smoke`) — dump + `pg_restore -l`
- [ ] Restore de teste em ambiente seguro (não produção / não Supabase compartilhado)

**Smoke local (recomendado)**

```bash
cd workspace-juridico-backend
# Prefira DIRECT_URL (conexão direta) no .env — não o pooler
npm run backup:smoke
```

O script grava em `backups/alar-smoke-*.dump` (gitignored) e valida o TOC do arquivo.
Requer `pg_dump` / `pg_restore` no PATH (cliente PostgreSQL).

**Supabase (painel)**  
- Settings → Database → backups automáticos (plano)  
- Ou Table Editor / SQL para export pontual  

**Dump manual (exemplo)**

```bash
# Backup
pg_dump "$DIRECT_URL" -Fc -f alar-backup.dump

# Restore (cuidado: sobrescreve — só em ambiente descartável)
pg_restore --clean --if-exists -d "$DIRECT_URL" alar-backup.dump
```

Prefira `DIRECT_URL` (conexão direta), não o pooler `pgbouncer`, para dump/restore.

---

## 7. Secrets — checklist de rotação

Use quando houver vazamento ou saída de alguém do time (não é tarefa diária):

- [ ] `JWT_SECRET` (invalida sessões atuais — usuários precisam logar de novo)
- [ ] `SUPABASE_KEY` (service_role) no painel Supabase + `.env`
- [ ] `AUTH_ADMIN_PASSWORD` / senhas de usuários
- [ ] `OPENAI_API_KEY`
- [ ] `SENTRY_DSN` (se regenerado no Sentry)
- [ ] SMTP (`SMTP_PASS`)
- [ ] `DATAJUD_API_KEY` (se em uso)

Nunca commitar `.env` / `.env.local`. Em produção: só painel do host (Vercel, Railway, etc.) ou secrets do CI.

---

## 8. Desligar servidores locais — OK

Encerrar os terminais `npm run start:dev` / `npm run dev`, ou liberar as portas 3000/3001.

---

## 9. SMTP local (Ethereal, grátis) — OK

Sem cartão. Útil para testar convite, reset e lembretes.

```bash
cd workspace-juridico-backend
npm run smtp:ethereal
```

Cole a saída no `.env`, reinicie a API, entre como **admin** → **Configurações** → **Enviar e-mail de teste**.  
Com Ethereal, a UI mostra o link de preview. Caixa: [ethereal.email/login](https://ethereal.email/login) (usuário/senha do script).

Sem SMTP o app continua ok: inbox + link de reset em desenvolvimento.

---

## Próximos passos sugeridos

1. Deploy HTTPS — guia [`DEPLOY.md`](./DEPLOY.md) (Railway API + Vercel front)
2. Testar um backup real (§6)
3. Em produção, trocar Ethereal por SMTP real
4. (Opcional) Resolver/ignorar issue ALAR-1 no Sentry — era só o teste

---

## Referências

- Setup geral: [`README.md`](./README.md)
- Roadmap: [`ROADMAP.md`](./ROADMAP.md)
- Deploy HTTPS: [`DEPLOY.md`](./DEPLOY.md)
- Sentry: [`SENTRY.md`](./SENTRY.md)
- Relatório de mudanças: [`RELATORIO-MUDANCAS-PARA-LUIZ.md`](./RELATORIO-MUDANCAS-PARA-LUIZ.md)
- Org: [github.com/projetoALAR](https://github.com/projetoALAR)
