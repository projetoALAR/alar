# Demo Alar — agência (roteiro pronto)

Roteiro de **10–12 minutos** para apresentar o Alar como MVP de escritório jurídico.  
Foco: **utilidades do caso** + atalhos que impressionam (busca, IA com fontes, prazos na agenda, capa PDF).

**Atualizado:** 21/08/2026 · branch `feat/integracao-api`

---

## 1. Preparar no dia anterior (ou 30 min antes)

### Ambiente

```bash
# Terminal 1 — API
cd workspace-juridico-backend
npx prisma migrate deploy
npm run start:dev

# Terminal 2 — Front
cd workspace-juridico-frontend
npm run dev
```

- Front: http://localhost:3000  
- API health: http://localhost:3001/health → `"database":"up"`

### Dados de demonstração (obrigatório)

```bash
cd workspace-juridico-backend
npm run seed:demo
```

Isso recria 2 casos ricos + clientes + PDFs + checklist + andamentos + agenda + modelos + chat + inbox.

| Precisa | Status |
|---------|--------|
| `OPENAI_API_KEY` **ou** `CHAT_ALLOW_MOCK=true` | Sem isso chat / gerar doc falham |
| Bucket Supabase `documentos` privado | Seed sobe PDFs |
| Admin **sem** 2FA (ou celular com authenticator) | Login fluido |
| SMTP Ethereal (opcional) | Só se for mostrar e-mail | `npm run smtp:ethereal` |

### Contas (senha padrão do seed)

| Papel | E-mail | Senha |
|-------|--------|-------|
| Admin (do `.env`) | `AUTH_ADMIN_EMAIL` | `AUTH_ADMIN_PASSWORD` |
| Advogada demo | `ana.ribeiro@alar.com.br` | `AlarAdminChangeMe1` |
| Assistente demo | `pedro.alves@alar.com.br` | `AlarAdminChangeMe1` |

**Na apresentação:** use o **admin** (visão completa). Mencione Ana/Pedro se perguntarem sobre RBAC.

### Anotar no papel (Ctrl+K)

Depois do seed (valores atuais desta base):

| Buscar | Abre |
|--------|------|
| `Camila` | Cliente / caso trabalhista |
| `Horizonte` | Cliente PJ / cobrança |
| `1004521` ou CNJ `1004521-38.2025.5.02.0001` | Caso trabalhista |
| `1018834` ou CNJ `1018834-72.2026.8.26.0100` | Caso cobrança |

> Se rodar `seed:demo` de novo, os CNJs podem mudar — confira o log do terminal.

---

## 2. O que o seed deixa pronto

### Clientes

1. **Camila Rodrigues Nunes** (PF) — caso trabalhista  
2. **Horizonte Alimentos Ltda** (PJ) — caso de cobrança  
3. **Lúcia Ferreira Campos** (PF) — sem caso (mostra “cliente novo”)

### Caso âncora (usar a maior parte do tempo)

**Reclamação trabalhista — horas extras e verbas rescisórias** (Camila)  
CNJ: `1004521-38.2025.5.02.0001`

Já vem com:

- PDFs (petição, contestação, procuração, etc.)
- Checklist (holerites, planilha, comparecimento VT)
- Andamentos / audiência
- Compromisso: audiência 42ª VT
- Chat com contexto
- Responsável na equipe

### Caso satélite (30–45 s)

**Cobrança de duplicatas — fornecimento de mercadorias** (Horizonte)  
CNJ: `1018834-72.2026.8.26.0100`

- Prazo de réplica, checklist, andamentos, modelo de réplica

### Modelos

Petição / Procuração / Notificação / Réplica (e afins) em **Modelos** — usados em **Gerar documento com IA**.

---

## 3. Roteiro falado (10–12 min)

> Tom: “sistema do dia a dia do escritório”, não pitch de startup genérico.  
> Evite: Sentry de teste, importações longas, configurar 2FA do zero.

### 0:00 — Abertura (30 s)

1. Abrir http://localhost:3000 → login admin  
2. Se o tour aparecer: **“Pular tour”** (ou concluir rápido)  
3. Painel: “Aqui o escritório vê o volume — casos, ativos, concluídos.”

**Frase:** *“Vocês vão ver o fluxo completo de um processo: do cliente ao PDF da capa, com IA que cita o que está no anexo.”*

---

### 0:30 — Busca global (1 min) · wow #1

1. **Ctrl+K** (ou Cmd+K)  
2. Digitar `Camila`  
3. Abrir o caso trabalhista  

**Frase:** *“Mesmo atalho do Notion/Linear — CPF, CNJ ou nome.”*

---

### 1:30 — Caso · Info (45 s)

Na URL `/casos/{id}`:

- Título, CNJ, prioridade, prazo  
- Responsável / co-responsável  
- Cliente com documento e contato  
- Tags  

**Frase:** *“Tudo o que o advogado precisa antes de abrir o PDF.”*

---

### 2:15 — Documentos + IA (2 min) · wow #2

Aba **Docs**:

1. Mostrar lista (PDFs do seed) — abrir um no browser  
2. **Gerar documento com IA** → escolher modelo (ex.: réplica / petição)  
3. Destacar: **revisão humana obrigatória** + placeholders preenchidos  
4. (Opcional) baixar / salvar o rascunho  

**Frase:** *“A IA não inventa o processo — usa o modelo do escritório e os dados do caso.”*

---

### 4:15 — Chat do caso (1,5–2 min) · wow #3

Aba **Chat**:

Prompt ensaiado (copiar/colar):

```text
Resuma os anexos deste caso em 5 bullets e cite as fontes pelos nomes dos arquivos.
```

Mostrar:

- Resposta com **fontes** (clicáveis quando houver)  
- Feedback útil / não útil (motivo opcional)  
- Disclaimer: não substitui advogado  

**Frase:** *“Pergunta sobre o que está no processo — não sobre a internet genérica.”*

---

### 6:00 — Andamentos (1 min)

Aba **Andam.**:

1. Mostrar movimentações já no caso  
2. Citar **Consultar CNJ** (DataJud — uso não comercial / demo)  
3. **Registrar andamento interno** (só mencionar se sobrar tempo)

**Frase:** *“Histórico do tribunal + anotações internas no mesmo lugar.”*

---

### 7:00 — Prazos → Agenda (1–1,5 min)

Aba **Prazos**:

1. Prazo do caso / intimação (se houver)  
2. Compromisso **Audiência una — 42ª VT**  
3. Ir em **Agenda** (`/agenda`) e achar o mesmo evento  

**Frase:** *“O prazo do processo já aparece na agenda da equipe.”*

---

### 8:30 — Checklist + Timeline (45 s)

- Aba **Check**: itens pendentes da Camila  
- Aba **Time**: feed unificado (docs, prazos, andamentos, comentários)

**Frase:** *“Um feed só — sem caçar e-mail.”*

---

### 9:15 — Capa PDF (20 s)

No header do caso: **Baixar capa do processo**.

Abrir o PDF gerado.

---

### 9:45 — Fechos opcionais (até 2 min — escolha 1 ou 2)

| Se perguntarem… | Mostre |
|-----------------|--------|
| Modelos | `/modelos` — biblioteca + filtros |
| Relatório / gestão | `/relatórios` → Exportar PDF/CSV |
| Equipe / papéis | `/equipe` ou login Ana vs Pedro |
| Segurança | `/configuracoes` — 2FA (não ativar na hora) |
| Migração | botão **Importar** em Clientes/Casos (só preview) |
| Chat geral | `/chat` — conversa sem caso |

---

### 11:00 — Encerramento (30 s)

**Frase de fechamento:**

> *“Do CNJ ao PDF da capa: cliente, documentos, prazos na agenda, andamentos e IA que cita o anexo — com revisão humana. Isso é o que um escritório usa na segunda-feira.”*

Perguntas. Anotar follow-ups (deploy HTTPS depende do Owner / Luiz).

---

## 4. Mapa rápido das abas do caso

| Aba | O que dizer em 10 s |
|-----|---------------------|
| **Info** | Ficha do processo + cliente |
| **Check** | To-dos com prazo |
| **Docs** | Anexos + gerar com IA |
| **Time** | Histórico unificado |
| **Prazos** | Intimação + compromissos |
| **Andam.** | CNJ / DataJud + internos |
| **Chat** | IA do caso com fontes |

---

## 5. Plano B (se algo falhar)

| Problema | Ação |
|----------|------|
| Chat / gerar doc erro | Confirmar `CHAT_ALLOW_MOCK=true` ou chave OpenAI; reiniciar API |
| Sem PDFs no Docs | Rodar de novo `npm run seed:demo` |
| Login 2FA | Usar conta sem TOTP ou código do authenticator |
| Ctrl+K vazio | Seed ok? Digitar só `Camila` / `Horizonte` |
| Front sem API | `http://localhost:3001/health` |
| Travou | Mudar para Agenda + Relatórios + Modelos (sem IA) |

---

## 6. Checklist no dia (imprimir)

- [ ] API + front no ar  
- [ ] `seed:demo` ok (Camila + Horizonte na lista)  
- [ ] Chat responde no caso (teste 1 prompt)  
- [ ] Docs lista ≥ 2 arquivos  
- [ ] Agenda mostra audiência da Camila  
- [ ] Capa PDF baixa  
- [ ] Papel com nomes para Ctrl+K  
- [ ] Prompt do chat colado no Bloco de notas  
- [ ] Zoom 110–125% no browser; notificações silenciadas  
- [ ] Não abrir `.env` na tela compartilhada  

---

## 7. O que **não** prometer na demo

- Multi-tenant / billing (Fase 6)  
- DataJud como produto **comercial** (hoje é uso não comercial)  
- App mobile nativo (há PWA básica)  
- Deploy público HTTPS (ainda bloqueado na org — ver `DEPLOY.md`)

---

## Referências

- Operação: [`RUNBOOK.md`](./RUNBOOK.md)  
- Roadmap: [`ROADMAP.md`](./ROADMAP.md)  
- Seed: `workspace-juridico-backend` → `npm run seed:demo`  
- Ajuda in-app: http://localhost:3000/ajuda  
