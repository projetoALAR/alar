# Runbook — Alar

Operação do dia a dia: subir, login admin, health e recuperação rápida.  
Sem secrets neste arquivo — use `.env` / `.env.local` / painel do provedor.

---

## 1. Subir local

### Pré-requisitos
- Node.js 20+
- `.env` no backend (copie de `.env.example`)
- `.env.local` no frontend (copie de `.env.example`)
- Supabase com Postgres + bucket **privado** `documentos`

### Backend (API :3001)

```bash
cd workspace-juridico-backend
npm install
npx prisma migrate deploy
npx prisma generate
npm run start:dev
```

Teste: `GET http://localhost:3001/health` → `status: "ok"`, `database: "up"`.

### Frontend (app :3000)

```bash
cd workspace-juridico-frontend
npm install
npm run dev
```

Abra: http://localhost:3000  
`NEXT_PUBLIC_API_URL` deve apontar para a API (ex.: `http://localhost:3001`).

### Docker (API + Postgres local, opcional)

Na raiz do monorepo / repo `alar`:

```bash
docker compose up --build
```

- Postgres: `localhost:5432`
- API: `localhost:3001`  
Frontend continua com `npm run dev`.

---

## 2. Criar / usar o admin

O admin é criado **só se a tabela `Usuario` estiver vazia** e existirem no `.env` do backend:

- `AUTH_ADMIN_EMAIL`
- `AUTH_ADMIN_PASSWORD` (≥ 8 caracteres)
- `AUTH_ADMIN_NOME` (opcional)

Sem senha válida → admin **não** é criado (log de aviso na API).

Com cadastro público desligado (`AUTH_ALLOW_PUBLIC_REGISTER=false`):
1. Faça login com o admin bootstrap
2. Em **Configurações** (admin), crie os demais usuários  
   ou `POST /auth/usuarios` com JWT de admin

---

## 3. Health check

```http
GET /health
```

| Resposta | Significado |
|----------|-------------|
| `200` + `database: "up"` | API e banco ok |
| `503` + `database: "down"` | API no ar, **banco inacessível** |

### Se `/health` cair ou der 503

1. Conferir `DATABASE_URL` / `DIRECT_URL` no `.env`
2. Supabase: projeto pausado? senha do DB correta? IP/rede?
3. Reiniciar a API (`npm run start:dev`)
4. Ver logs no terminal (linha JSON com `requestId`)
5. Se Sentry estiver ativo (`SENTRY_DSN`), abrir o projeto no Sentry

Porta ocupada (`EADDRINUSE :::3001` / `3000`): encerrar o processo antigo ou mudar `PORT`.

---

## 4. Logs e request id

Cada request HTTP da API gera um log JSON, por exemplo:

```json
{"requestId":"...","method":"GET","path":"/health","statusCode":200,"durationMs":12,"userId":null}
```

- Header de resposta: `x-request-id`
- Cliente pode enviar `x-request-id`; senão a API gera um UUID
- Com usuário autenticado, `userId` aparece no log

Use o `requestId` para correlacionar erro no Sentry / suporte.

---

## 5. Sentry (erros)

Opcional em local; **recomendado** em staging/prod.

| Onde | Variável |
|------|----------|
| Backend | `SENTRY_DSN` |
| Frontend | `NEXT_PUBLIC_SENTRY_DSN` |

Sem DSN → Sentry fica desligado (app funciona normalmente).

Crie um projeto em [sentry.io](https://sentry.io), copie o DSN e reinicie os servidores.

Para validar: force um erro e confira no painel Sentry (leve alguns segundos).

---

## 6. Backup / restore (Postgres)

**Supabase (painel)**  
- Settings → Database → backups automáticos (plano)  
- Ou Table Editor / SQL para export pontual  

**Dump manual (exemplo)**

```bash
# Backup
pg_dump "$DIRECT_URL" -Fc -f alar-backup.dump

# Restore (cuidado: sobrescreve)
pg_restore --clean --if-exists -d "$DIRECT_URL" alar-backup.dump
```

Prefira `DIRECT_URL` (conexão direta), não o pooler `pgbouncer`, para dump/restore.

---

## 7. Secrets — checklist de rotação

Troque se houver vazamento ou saída de alguém do time:

- [ ] `JWT_SECRET` (invalida sessões atuais — usuários precisam logar de novo)
- [ ] `SUPABASE_KEY` (service_role) no painel Supabase + `.env`
- [ ] `AUTH_ADMIN_PASSWORD` / senhas de usuários
- [ ] `OPENAI_API_KEY`
- [ ] `SENTRY_DSN` (se regenerado no Sentry)
- [ ] SMTP (`SMTP_PASS`)

Nunca commitar `.env` / `.env.local`. Em produção: só painel do host (Vercel, Railway, etc.) ou secrets do CI.

---

## 8. Desligar servidores locais

Encerrar os terminais `npm run start:dev` / `npm run dev`, ou liberar as portas 3000/3001.

---

## Referências

- Setup geral: [`README.md`](./README.md)
- Roadmap: [`ROADMAP.md`](./ROADMAP.md)
- Relatório de mudanças: [`RELATORIO-MUDANCAS-PARA-LUIZ.md`](./RELATORIO-MUDANCAS-PARA-LUIZ.md)
- Org: [github.com/projetoALAR](https://github.com/projetoALAR)
