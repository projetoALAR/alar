# Roadmap — Alar

Próximas etapas para evoluir o Alar de **MVP operacional** para **produto profissional**.  
Itens já entregues (RBAC, signed URLs, validação, health, CI, Docker, chat geral/caso, PDF, mobile) **não** entram neste roadmap.

**Como usar:** marque `[x]` quando concluir; mantenha a ordem das fases salvo urgência de negócio.

---

## Visão rápida

| Fase | Foco | Horizonte sugerido |
|------|------|--------------------|
| **0** | Fundação de operação | 1 semana |
| **1** | Confiança (staging, deploy, erros) | 1–2 semanas |
| **2** | Segurança & LGPD | 2–3 semanas |
| **3** | Produto / UX de escritório | 2–4 semanas |
| **4** | IA com qualidade | 2 semanas |
| **5** | Engenharia (API, testes, stack) | contínuo |
| **6** | Escala / negócio | depois do product-market fit |

---

## Fase 0 — Alinhamento (agora)

- [ ] Revisar este roadmap com o time (Izack + Luiz)
- [ ] Definir ambiente oficial de dados (só Supabase prod? ou staging separado?)
- [ ] Lista do que é “bloqueador para mostrar a um cliente”
- [ ] Dono de cada fase (quem puxa)

---

## Fase 1 — Confiança e operação

Objetivo: o sistema sobe, falha de forma visível e dá para recuperar.

- [ ] Separar ambientes **dev / staging / prod** (`.env`, banco e bucket distintos)
- [ ] Deploy da **API** (ex.: Railway, Fly, Render) com HTTPS
- [ ] Deploy do **frontend** (ex.: Vercel) apontando para a API de staging/prod
- [ ] Domínio + CORS (`CORS_ORIGINS`) alinhados ao front real
- [ ] **Sentry** (ou similar) no backend e no frontend
- [ ] Logs estruturados na API (request id, user id, rota)
- [ ] Documentar **backup/restore** do Postgres (Supabase ou dump)
- [ ] Runbook curto: subir local, criar admin, o que fazer se `/health` cair
- [ ] Secrets só em painel/CI; checklist de rotação (`JWT_SECRET`, `SUPABASE_KEY`)

**Critério de pronto:** staging acessível por URL, health ok, erro de teste aparece no Sentry.

---

## Fase 2 — Segurança e compliance

Objetivo: escritório confia em quem viu/alterou o quê; base LGPD.

- [ ] Tabela **`AuditLog`**: criar/editar/excluir em cliente, processo, documento, usuário
- [ ] UI simples de auditoria (admin) — filtros por entidade/data/usuário
- [ ] **Exportar dados** de um cliente (JSON/ZIP) sob pedido
- [ ] **Excluir / anonimizar** cliente e vínculos (fluxo LGPD)
- [ ] Disclaimer fixo: IA não substitui advogado (chat + login)
- [ ] Política de senha reforçada + aviso de senha fraca
- [ ] Bloqueio / cooldown após N logins falhos (além do throttle)
- [ ] **2FA** para `ADMIN` (TOTP ou e-mail)
- [ ] Revisar RBAC fino: assistente só vê casos **atribuídos** (quando existir responsável)

**Critério de pronto:** toda ação sensível gera log; admin consegue exportar/apagar cliente.

---

## Fase 3 — Produto e UX profissional

Objetivo: fluxo diário de advogado, não só CRUD.

- [ ] Campo **responsável** (e opcional co-responsável) no processo
- [ ] **Timeline** do caso: uploads, mudanças de status, comentários internos, prazos
- [ ] **Busca global** (nome, CPF, número CNJ, título)
- [ ] Jobs/notificações de **prazo** (e-mail SMTP + inbox)
- [ ] Empty states e onboarding (primeiro login / tour curto)
- [ ] Branding: logo, favicon, e-mails com identidade Alar
- [ ] Revisão de acessibilidade e tablet
- [ ] Atribuição de tarefas / checklist por caso (opcional nesta fase)

**Critério de pronto:** abrir um caso mostra histórico + responsável; busca encontra em &lt; 2 s.

---

## Fase 4 — IA com qualidade de escritório

Objetivo: útil e controlada, sem “alucinar” como verdade.

- [ ] Respostas do chat do caso **citando anexo** (arquivo + trecho quando possível)
- [ ] Limite de **tokens/custo por usuário/dia**
- [ ] Exportar conversa; feedback “útil / não útil”
- [ ] Modo rascunho (petição/contrato) com **revisão humana** obrigatória na UI
- [ ] Prompt + disclaimer: não inventar jurisprudência / números de processo
- [ ] Métricas básicas de uso da IA (admin)

**Critério de pronto:** chat do caso sempre indica fontes usadas ou diz que não achou no anexo.

---

## Fase 5 — Engenharia

Objetivo: mudanças seguras e onboarding de dev rápido.

- [ ] **Swagger/OpenAPI** na API Nest
- [ ] Cliente tipado no front (gerado ou package compartilhado)
- [ ] Versionamento `/v1` nas rotas públicas
- [ ] E2E dos fluxos críticos: login → criar caso → upload → chat
- [ ] Coverage mínimo no CI (ex.: back &gt; 60% nos módulos core)
- [ ] Preview de PR (Vercel / ambiente efêmero)
- [ ] Serviço **web** no `docker-compose` (stack completa com um comando)
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

## Próximas 2–4 semanas (foco sugerido)

Ordem recomendada se o time for enxuto:

1. **Fase 1** — staging + deploy + Sentry  
2. **Fase 2** — AuditLog + export/delete LGPD + disclaimer IA  
3. **Fase 3** — responsável no caso + timeline (+ busca se der tempo)  
4. Em paralelo leve: **Fase 4** (citações no chat) e **Swagger** (Fase 5)

---

## Fora de escopo por enquanto

- Reescrever o stack (manter Nest + Next + Prisma + Supabase)
- Trocar provedor de IA sem necessidade
- Multi-tenant antes de ter 2º cliente real

---

## Referências

- Relatório das mudanças recentes: [`RELATORIO-MUDANCAS-PARA-LUIZ.md`](./RELATORIO-MUDANCAS-PARA-LUIZ.md)
- Setup atual: [`README.md`](./README.md)
- Org: [github.com/projetoALAR](https://github.com/projetoALAR)
