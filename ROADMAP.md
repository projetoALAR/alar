# Roadmap — Alar

Próximas etapas para evoluir o Alar de **MVP operacional** para **produto profissional**.  
Itens já entregues (RBAC, signed URLs, validação, health, CI, Docker, chat geral/caso, PDF, mobile) **não** entram neste roadmap.

**Como usar:** marque `[x]` quando concluir; mantenha a ordem das fases salvo urgência de negócio.

**Última atualização:** 13/08/2026

---

## Visão rápida

| Fase | Foco | Status |
|------|------|--------|
| **0** | Fundação de operação | ✅ concluída |
| **1** | Confiança (runbook, Sentry, deploy) | 🟡 parcial — falta deploy HTTPS |
| **2** | Segurança & LGPD | ✅ concluída |
| **3** | Produto / UX de escritório | ✅ core + branding + a11y/tablet |
| **4** | IA com qualidade | ✅ core (citações, quota, feedback, export, rascunho+revisão) |
| **5** | Engenharia (API, testes, stack) | 🟡 Swagger + E2E + compose web + coverage CI |
| **6** | Escala / negócio | depois do product-market fit |

---

## Fase 0 — Alinhamento ✅

**Status:** decisões fechadas (Izack + Luiz; trabalho em conjunto em todas as fases).

### Decisões

**Ambiente de dados**
- Por enquanto: **um único projeto Supabase** (dev/demo compartilhado).
- Criar **staging separado** (`alar-staging`) só quando front/API tiverem **URL pública** para mostrar a terceiros.
- Enquanto isso: senhas fortes, bucket privado, sem compartilhar URL de produção com cliente final.

**Bloqueadores para mostrar a um cliente** (obrigatório antes da demo externa)

| Obrigatório | Motivo | Status |
|-------------|--------|--------|
| HTTPS (front + API) | confiança básica | ⏳ aguardando Owner/apps na org |
| Login + RBAC ok | não vazar dados | ✅ |
| Disclaimer da IA | risco jurídico | ✅ |
| Mobile usável | demo no celular | ✅ |
| Health + como criar usuário documentado | não travar na demo | ✅ |
| Bucket privado + `service_role` | documentos sensíveis | ✅ |

**Não bloqueia demo:** 2FA, multi-tenant, Stripe, Sentry (desejável, mas demo possível sem). *(Busca global já entregue.)*

**Time**
- Izack e Luiz fazem **tudo juntos** — sem dono exclusivo por fase.
- Cursor ajuda a manter o roadmap e a implementar.

### Checklist

- [x] Revisar este roadmap com o time
- [x] Definir ambiente oficial de dados
- [x] Lista do que é bloqueador para mostrar a um cliente
- [x] Forma de trabalho: tudo em conjunto (sem divisão rígida de donos)

---

## Fase 1 — Confiança e operação ← **bloqueador restante: deploy**

Objetivo: o sistema sobe, falha de forma visível e dá para recuperar.

**Progresso (12/08/2026):** runbook + Sentry + logs ✅ · **próximo: deploy HTTPS** (depende do Luiz liberar Owner ou instalar Railway/Vercel na org)

Ordem sugerida nesta fase:
1. ~~Runbook curto~~ ✅  
2. ~~Sentry (back + front)~~ ✅  
3. ~~Logs estruturados (request id)~~ ✅  
4. **Deploy com HTTPS + CORS** ← agora  
5. Staging Supabase separado **quando** a URL for pública  

- [x] Runbook curto: subir local, criar admin, o que fazer se `/health` cair
- [x] **Sentry** (ou similar) no backend e no frontend *(código + DSN local + Issue ALAR-1 + flags de teste off)*
- [x] Logs estruturados na API (request id, user id, rota)
- [x] Documentar **backup/restore** do Postgres (Supabase ou dump) *(no RUNBOOK)*
- [x] Secrets só em painel/CI; checklist de rotação (`JWT_SECRET`, `SUPABASE_KEY`) *(no RUNBOOK)*
- [x] Colar DSN do Sentry *(local; não commitado)* + Issue validada; flags de teste desligadas
- [ ] Deploy da **API** no Railway com HTTPS — guia [`DEPLOY.md`](./DEPLOY.md)
- [ ] Deploy do **frontend** no Vercel apontando para a API
- [ ] Domínio + CORS (`CORS_ORIGINS`) alinhados ao front real
- [ ] Separar ambientes **dev / staging / prod** (segundo Supabase) quando houver URL pública
- [ ] (Opcional, antes do deploy) Smoke de **backup real** do Postgres — ver RUNBOOK §6

**Critério de pronto:** app acessível por HTTPS (ou staging), health ok, erro de teste aparece no Sentry.  
**Parcial:** operação local + observabilidade ok; **falta deploy HTTPS** (bloqueador de demo externa).

---

## Fase 2 — Segurança e compliance ✅

Objetivo: escritório confia em quem viu/alterou o quê; base LGPD.

- [x] Tabela **`AuditLog`**: criar/editar/excluir em cliente, processo, documento, usuário
- [x] UI simples de auditoria (admin) — filtros por entidade/data/usuário
- [x] **Exportar dados** de um cliente (JSON/ZIP) sob pedido
- [x] **Excluir / anonimizar** cliente e vínculos (fluxo LGPD)
- [x] Disclaimer fixo: IA não substitui advogado (chat + login)
- [x] Política de senha reforçada + aviso de senha fraca
- [x] Bloqueio / cooldown após N logins falhos (além do throttle)
- [x] **2FA** para `ADMIN` (TOTP)
- [x] Revisar RBAC fino: assistente só vê casos **atribuídos** (quando existir responsável)

**Critério de pronto:** toda ação sensível gera log; admin consegue exportar/apagar cliente. ✅

---

## Fase 3 — Produto e UX profissional 🟡

Objetivo: fluxo diário de advogado, não só CRUD.

- [x] Campo **responsável** (e opcional co-responsável) no processo
- [x] **Timeline** do caso: uploads, mudanças de status, comentários internos, prazos
- [x] **Busca global** (nome, CPF, número CNJ, título) — `GET /busca` + Ctrl+K
- [x] Jobs/notificações de **prazo** (e-mail SMTP + inbox) — cron diário 8h
- [x] Empty states e onboarding (primeiro login / tour curto)
- [x] Branding: logo, favicon, e-mails com identidade Alar
- [x] Revisão de acessibilidade e tablet
- [ ] Atribuição de tarefas / checklist por caso (opcional nesta fase)

**Critério de pronto:** abrir um caso mostra histórico + responsável; busca encontra em &lt; 2 s. ✅ (core)

---

## Fase 4 — IA com qualidade de escritório 🟡

Objetivo: útil e controlada, sem “alucinar” como verdade.

- [x] Respostas do chat do caso **citando anexo** (arquivo + trecho quando possível)
- [x] Limite de **tokens/custo por usuário/dia**
- [x] Feedback “útil / não útil” nas respostas da IA
- [x] Exportar conversa
- [x] Modo rascunho (petição/contrato) com **revisão humana** obrigatória na UI
- [x] Prompt + disclaimer: não inventar jurisprudência / números de processo
- [x] Métricas básicas de uso da IA (admin) — `GET /chat/metricas`

**Critério de pronto:** chat do caso sempre indica fontes usadas ou diz que não achou no anexo. ✅ (citações entregues)

---

## Fase 5 — Engenharia

Objetivo: mudanças seguras e onboarding de dev rápido.

- [x] **Swagger/OpenAPI** na API Nest
- [ ] Cliente tipado no front (gerado ou package compartilhado)
- [ ] Versionamento `/v1` nas rotas públicas
- [x] E2E dos fluxos críticos: login → criar caso → upload → chat
- [x] Coverage mínimo no CI (auth, processos, chat, documentos, clientes, casos-acesso ≥ 60% linhas/statements)
- [ ] Preview de PR (Vercel / ambiente efêmero)
- [x] Serviço **web** no `docker-compose` (stack completa com um comando)
- [ ] Atualizar submodule do repo `alar` quando front/back avançarem

**Critério de pronto:** PR nova quebra e2e → CI vermelho; compose sobe API+DB(+web).

---

## Fase 6 — Escala e negócio (depois)

Só quando houver demanda real de mais de um escritório / monetização.

- [ ] **Multi-tenant** (isolamento por escritório)
- [ ] Convites por e-mail (magic link / token)
- [ ] Planos e billing (Stripe)
- [ ] Integração Google Calendar / e-mail
- [ ] Consulta processual (DataJud/CNJ ou provedor)
- [ ] App mobile (PWA primeiro)

---

## Entregas recentes (12/08/2026) — branch `feat/integracao-api`

| Item | Backend | Frontend |
|------|---------|----------|
| Timeline + comentários internos | `2af15c8` | `20c1671` |
| Busca global | `a836d03` | `d1ce933` |
| Lembretes de prazo (cron 8h) | `9599f2f` | — |
| Onboarding + empty states | — | `5397e11` |
| Citações de anexos no chat | `2303b23` | `f58b5a9` |

Migrations aplicadas no Supabase compartilhado:
- `20260812160000_add_processo_comentario`
- `20260812170000_add_mensagem_fontes`
*(anteriores: responsável, TOTP, etc.)*

---

## Próximas 2–4 semanas (foco sugerido)

1. **Bloqueador — Fase 1:** deploy HTTPS da API + frontend + CORS (`CORS_ORIGINS`) assim que o Luiz liberar a org  
2. **Fase 5:** cliente tipado no front (a partir do Swagger)  
3. **Fase 5:** versionamento `/v1` nas rotas públicas  

---

## Fora de escopo por enquanto

- Reescrever o stack (manter Nest + Next + Prisma + Supabase)
- Trocar provedor de IA sem necessidade
- Multi-tenant antes de ter 2º cliente real

---

## Referências

- Runbook (operação): [`RUNBOOK.md`](./RUNBOOK.md)
- Deploy HTTPS: [`DEPLOY.md`](./DEPLOY.md)
- Sentry: [`SENTRY.md`](./SENTRY.md)
- Relatório das mudanças recentes: [`RELATORIO-MUDANCAS-PARA-LUIZ.md`](./RELATORIO-MUDANCAS-PARA-LUIZ.md)
- Setup atual: [`README.md`](./README.md)
- Org: [github.com/projetoALAR](https://github.com/projetoALAR)
