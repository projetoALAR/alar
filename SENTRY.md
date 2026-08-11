# Ativar Sentry no Alar (5 minutos)

O código já está integrado. Sem DSN, o app funciona normalmente e o Sentry fica desligado.

## 1. Criar projetos no Sentry

1. Conta em https://sentry.io (plano free basta)
2. Crie **dois** projetos (ou um e reutilize o DSN se preferir no começo):
   - Plataforma **Node.js / NestJS** → backend
   - Plataforma **Next.js** → frontend
3. Em cada projeto: **Settings → Client Keys (DSN)** → copie o DSN

## 2. Colar no `.env` local

### Backend (`workspace-juridico-backend/.env`)

```env
SENTRY_DSN=https://...@....ingest.sentry.io/...
SENTRY_ENVIRONMENT=development
SENTRY_ENABLE_TEST_ENDPOINT=true
```

### Frontend (`workspace-juridico-frontend/.env.local`)

```env
NEXT_PUBLIC_SENTRY_DSN=https://...@....ingest.sentry.io/...
NEXT_PUBLIC_SENTRY_ENVIRONMENT=development
NEXT_PUBLIC_SENTRY_ENABLE_TEST=true
```

Pode usar o **mesmo DSN** nos dois no início; depois separe projetos se quiser.

## 3. Reiniciar servidores

```bash
# backend
npm run start:dev

# frontend
npx pnpm@9 run dev
```

Confirme no health: `GET http://localhost:3001/health` → `"sentry": true`

## 4. Disparar erro de teste

- API: abra `http://localhost:3001/debug/sentry` (deve retornar 500)
- Front: abra `http://localhost:3000/sentry-test` → botão **Disparar erro de teste**

Em até ~1 minuto os eventos aparecem em **Issues** no Sentry.

## 5. Desligar os testes

Depois de validar:

```env
SENTRY_ENABLE_TEST_ENDPOINT=false
NEXT_PUBLIC_SENTRY_ENABLE_TEST=false
```

Em **produção**, nunca deixe esses flags `true`. O endpoint da API também bloqueia se `NODE_ENV=production`.

## Produção / staging

No painel do host (Railway, Vercel, etc.) defina as mesmas variáveis, com:

```env
SENTRY_ENVIRONMENT=production
NEXT_PUBLIC_SENTRY_ENVIRONMENT=production
```

(sem flags de teste)
