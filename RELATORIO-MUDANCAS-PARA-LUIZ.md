# Relatório de mudanças — Alar

**Para:** Luiz  
**Data:** 12/08/2026 (atualização)  
**Branch:** `feat/integracao-api` (backend e frontend)

**Onde está o código (org projetoALAR)**

| Repo | URL |
|------|-----|
| Backend | https://github.com/projetoALAR/backend |
| Frontend | https://github.com/projetoALAR/frontend |
| Orquestração | https://github.com/projetoALAR/alar (Compose, README, roadmap, submodules) |

**Docs úteis no repo `alar`**
- Este relatório: [`RELATORIO-MUDANCAS-PARA-LUIZ.md`](https://github.com/projetoALAR/alar/blob/main/RELATORIO-MUDANCAS-PARA-LUIZ.md)
- Próximas etapas: [`ROADMAP.md`](https://github.com/projetoALAR/alar/blob/main/ROADMAP.md)

---

## Status por fase (resumo)

| Fase | Status |
|------|--------|
| 0 — Alinhamento | ✅ |
| 1 — Confiança / ops | 🟡 falta **deploy HTTPS** (bloqueado até liberar org Railway/Vercel) |
| 2 — Segurança & LGPD | ✅ (AuditLog, export/anonimizar, senha forte, lockout, 2FA admin, RBAC assistente) |
| 3 — Produto / UX | 🟡 core ✅ (responsável, timeline, busca, prazos, onboarding); falta branding/a11y |
| 4 — IA | 🟡 citações de anexos ✅ |
| 5 — Engenharia | pendente (Swagger, E2E…) |
| 6 — Escala | depois |

---

## Entregas 12/08/2026 (já na `feat/integracao-api`)

### Backend

| Commit | Feature |
|--------|---------|
| `2af15c8` | Timeline do caso + comentários internos (`ProcessoComentario`) |
| `a836d03` | Busca global `GET /busca` (nome, CPF, CNJ, título) com RBAC |
| `9599f2f` | Job diário 8h de lembretes de prazo (inbox + SMTP se configurado) |
| `2303b23` | Citações de anexos no chat do caso (`Mensagem.fontes`) |

### Frontend

| Commit | Feature |
|--------|---------|
| `20c1671` | Aba Timeline no painel do caso |
| `d1ce933` | Busca global Ctrl+K no header |
| `5397e11` | Onboarding (tour 1º login) + empty states |
| `f58b5a9` | UI “Fontes consultadas” nas respostas do chat |

### Migrations (já aplicadas no Supabase compartilhado)

- `20260812160000_add_processo_comentario`
- `20260812170000_add_mensagem_fontes`

*(Anteriores nesta sprint: responsável no processo, TOTP admin, etc.)*

---

## Como testar local (rápido)

1. API `:3001` + front `:3000` (`npm run start:dev` / `npm run dev`)
2. **Ctrl+K** → buscar cliente/CNJ
3. Abrir caso → **Timeline** → publicar comentário
4. Caso com PDF/txt → **Chat** → perguntar sobre o doc → ver **Fontes consultadas**
5. **Ajuda** → “Ver tour novamente” (onboarding)
6. Admin com 2FA: Configurações → ativar TOTP

### Ao puxar

```bash
# backend
git checkout feat/integracao-api && git pull
npx prisma migrate deploy
npm run start:dev

# frontend
git checkout feat/integracao-api && git pull
npm run dev
```

---

## O que ainda depende de você (Luiz)

Para **demo externa** (HTTPS):

1. Owner na org `projetoALAR` **ou** instalar apps **Railway** + **Vercel** no GitHub
2. Seguir [`DEPLOY.md`](./DEPLOY.md) — API Railway, front Vercel, `CORS_ORIGINS`

Enquanto isso, o produto avança localmente (Fase 3/4).

---

## Histórico anterior (05/08/2026)

Base antiga e entregas iniciais (chat geral≠caso, RBAC, signed URLs, mobile, CI/Docker) permanecem válidas. Detalhes das seções técnicas daquele dia estão no histórico do Git deste arquivo / commits anteriores na branch.

### Frase útil no Cursor (Agent)

> Puxa as mudanças do Izack (branch `feat/integracao-api` na org projetoALAR), instala as deps e roda o migrate. Não faça commit nem push.

---

Qualquer dúvida, chama no chat / PR em `feat/integracao-api`.
