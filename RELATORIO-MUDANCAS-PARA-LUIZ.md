# Relatório de mudanças — Alar

**Para:** Luiz  
**Data:** 05/08/2026 (atualizado no mesmo dia)  
**Branch:** `feat/integracao-api` (backend e frontend)

**Base de comparação**
- Backend: `1ae3e8b` — *feat: adiciona IA no chat, inbox e notificacoes por usuario*
- Frontend: `f6b2178` — *feat: adiciona inbox, contatos, push e corrige tema escuro*

**Onde está o código (org projetoALAR)**

| Repo | URL |
|------|-----|
| Backend | https://github.com/projetoALAR/backend (também: workspace-juridico-backend) |
| Frontend | https://github.com/projetoALAR/frontend |
| Orquestração | https://github.com/projetoALAR/alar (Compose, README, roadmap, submodules) |

Tudo abaixo **já foi commitado e enviado** para a org.

**Docs úteis no repo `alar`**
- Este relatório: [`RELATORIO-MUDANCAS-PARA-LUIZ.md`](https://github.com/projetoALAR/alar/blob/main/RELATORIO-MUDANCAS-PARA-LUIZ.md)
- Próximas etapas: [`ROADMAP.md`](https://github.com/projetoALAR/alar/blob/main/ROADMAP.md)

---

## Commits depois da sua base

### Backend

1. `46b9431` — separa chat geral e chat do caso (privacidade/contexto)
2. `c5a3491` — RBAC, validação, docs privados, PDF no chat, health, CI/Docker

### Frontend

1. `e4642d0` — UI do chat separado + ajustes de layout
2. `2999915` — calendário + descrição nos casos
3. `62ae1d4` — RBAC na UI, painel admin, scroll no chat, CI/README
4. `f928d83` — **corrige layout mobile** (sidebar sobreposta, chat, casos, calendário)

### Repo `alar` (main)

1. Orquestração inicial (Compose, README, submodules, CI da raiz)
2. Este relatório de mudanças
3. **`ROADMAP.md`** + link no README — próximas fases do produto

---

# 1. Backend — o que mudou

## 1.1 Chat geral vs chat do caso (`46b9431`)

- Contexto agregado no chat geral (sem dados sensíveis do caso)
- Contexto completo no chat do painel do caso (`processoId`)
- Prompts distintos (workspace vs caso)

## 1.2 Segurança / RBAC (`c5a3491`)

- Papéis: `ADMIN` | `ADVOGADO` | `ASSISTENTE`
- Migration: `20260805120000_add_usuario_role`
- `@Roles()` + `RolesGuard`
- Cadastro público desligado por padrão (`AUTH_ALLOW_PUBLIC_REGISTER=false`)
- Admin cria usuários: `POST /auth/usuarios`
- Senha admin sem default fraco; mínimo 8 caracteres
- JWT passa a expor `role`

## 1.3 Validação de API

- `ValidationPipe` global (whitelist)
- DTOs em clientes, processos, compromissos, equipe, documentos, auth
- Número do processo: CNJ ou código interno

## 1.4 Hardening HTTP / ops

- Helmet, CORS via `CORS_ORIGINS`, throttling (login + chat 20/min)
- `GET /health` → 503 se o banco estiver down
- Pacote renomeado para `alar-backend`

## 1.5 Documentos / Storage

- Bucket privado + **URLs assinadas** (~1h)
- Allowlist MIME (PDF, imagens, txt/csv, Word)
- `processoId` validado como UUID no upload
- Backend deve usar chave **service_role** no Supabase (`SUPABASE_KEY`)

## 1.6 Chat — isolamento e PDF

- Migration: `20260805140000_conversacao_usuario` (`Conversacao.usuarioId`)
- Cada usuário só vê as próprias conversas
- Extração de texto de PDF (`pdf-parse` v2 / `PDFParse`)
- Prompt reforçado para inventariar anexos

## 1.7 DevOps

- CI (`.github/workflows/ci.yml`), Dockerfile, `.dockerignore`, `.env.example`
- Testes atualizados (RolesGuard, DTOs, health)

### Dependências novas

`@nestjs/throttler`, `helmet`, `class-validator`, `class-transformer`, `pdf-parse`

---

# 2. Frontend — o que mudou

## 2.1 Chat / casos (commits anteriores)

- UI alinhada ao chat geral vs caso
- Calendário melhorado + descrição nos casos

## 2.2 RBAC na UI (`62ae1d4`)

- `lib/roles.ts` — helpers de permissão
- `lib/auth-api.ts` — `role`, `createUser`, `listUsers`
- Login: cadastro só com `NEXT_PUBLIC_ALLOW_REGISTER=true`
- Botões criar/editar/excluir conforme papel
- Painel admin em Configurações (`admin-users-panel.tsx`)
- Header mostra o papel do usuário

## 2.3 UX do chat

- Scroll automático para a mensagem mais recente (chat geral e do caso)
- Layout com altura/overflow corrigidos

## 2.4 Branding / CI

- Nome `alar-frontend`; metadata Alar (sem branding v0)
- README + CI do frontend

## 2.5 Responsividade mobile (`f928d83`) — **atualização recente**

Problema: no celular a **sidebar fixa** cobria o conteúdo (sobreposição).

Correções:
- Sidebar **escondida no mobile**; menu via hamburger (Sheet)
- Variante `Sidebar mobile` (sem `fixed`) dentro do drawer
- Menu mobile também em páginas sem Header completo: **Casos**, **Calendário**, **Chat**
- Chat: histórico em sheet no mobile (“Conversas”); lado a lado só no desktop
- Cards de casos: botões usáveis no touch; menos overflow
- Calendário e mains com `min-w-0` / `overflow-x-hidden`
- Header: nome/e-mail truncados para não estourar a barra

Arquivos principais: `sidebar.tsx`, `mobile-nav.tsx`, `app/chat/page.tsx`, `app/tasks/page.tsx`, `app/calendar/page.tsx`, `tasks-content.tsx`, etc.

---

# 3. Repo `alar` (orquestração)

- `README.md` unificado (+ link para o roadmap)
- `docker-compose.yml` (Postgres + API)
- Workflows da raiz
- Submodules apontando para backend e frontend
- `RELATORIO-MUDANCAS-PARA-LUIZ.md` (este arquivo)
- **`ROADMAP.md`** — próximas etapas (ver seção 6)

Clone completo:

```bash
git clone --recurse-submodules https://github.com/projetoALAR/alar.git
```

> Se o submodule do frontend ainda apontar para um commit antigo, após o pull rode  
> `git submodule update --remote` (ou atualize o ponteiro no repo `alar`) para pegar o `f928d83`.

---

# 4. O que você precisa fazer ao puxar

1. Checkout `feat/integracao-api` no back e no front (ou clone via `alar` com submodules)
2. Rodar migrations:

   ```bash
   npx prisma migrate deploy
   ```

3. Conferir variáveis novas no `.env`:
   - `CORS_ORIGINS`
   - `AUTH_ALLOW_PUBLIC_REGISTER` / `NEXT_PUBLIC_ALLOW_REGISTER`
   - `JWT_SECRET` forte
   - `AUTH_ADMIN_PASSWORD` (≥ 8)
   - Bucket `documentos` **privado** + `SUPABASE_KEY` = service_role

4. No front: `git pull` até incluir **`f928d83`** e testar no DevTools (modo celular)

### OpenAI

Sem troca de provedor — continua `OPENAI_API_KEY` / `OPENAI_MODEL` (ex.: `gpt-4o-mini`).  
Só melhoramos contexto, PDF e prompts.

### Frase útil no Cursor (Agent)

> Puxa as mudanças do Izack (branch `feat/integracao-api` na org projetoALAR), instala as deps e roda o migrate. Não faça commit nem push.

---

# 5. Resumo em uma página

| Área | Mudança |
|------|---------|
| Chat | Geral ≠ caso; privacidade; PDF; scroll; histórico mobile em sheet |
| Auth | RBAC 3 papéis; admin cria users; register off |
| API | DTOs + ValidationPipe |
| Segurança | Helmet, CORS, throttle, health 503 |
| Docs | Signed URLs + MIME allowlist |
| Mobile | Sidebar no hamburger; Casos/Calendário/Chat usáveis no celular |
| Ops | CI, Docker, `.env.example`, repo `alar` |
| Planejamento | `ROADMAP.md` com fases 0–6 |

---

# 6. Roadmap (próximas etapas)

Documento: https://github.com/projetoALAR/alar/blob/main/ROADMAP.md

Foco sugerido das próximas **2–4 semanas**:
1. Staging + deploy + Sentry  
2. AuditLog + export/delete LGPD + disclaimer da IA  
3. Responsável no caso + timeline  
4. Citações no chat do caso / Swagger  

Fases no arquivo: operação → segurança/LGPD → produto/UX → IA → engenharia → escala/negócio.

---

Qualquer dúvida, chama no chat / PR em `feat/integracao-api`.
