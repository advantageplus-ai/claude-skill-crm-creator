# Claude Code Skill — CRM Creator

Una skill per [Claude Code](https://claude.ai/code) che costruisce automaticamente un **CRM completo e funzionante** in pochi minuti.

Stack: **Next.js 16 + Prisma 7 + PostgreSQL + Tailwind CSS v4**, deploy su **GitHub + Railway**.

---

## Cosa crea

| Sezione | Contenuto |
|---|---|
| **Dashboard** | KPI cards, grafico revenue, pipeline bar chart, attività recenti |
| **Aziende** | Lista card con hover actions, dettaglio, modifica, eliminazione |
| **Contatti** | Tabella con avatar colorati, dettaglio, modifica, eliminazione |
| **Pipeline** | Kanban drag-and-drop per fase opportunità |
| **Analytics** | Grafici Recharts: revenue mensile, pipeline per fase, top aziende |
| **Calendario** | Vista mensile con eventi colorati |
| **Attività** | Task list con priorità, stato e scadenze |
| **Impostazioni** | Configurazione azienda |

Queste sono le sezioni **incluse di default** — ma il CRM è completamente personalizzabile.

### Sezioni aggiuntive su richiesta

Durante le domande iniziali puoi chiedere qualsiasi sezione extra:

| Esempio | Cosa viene creato |
|---|---|
| Ordini | Tabella ordini, stati (Nuovo/In lavorazione/Spedito/Consegnato), collegamento clienti |
| Prodotti & Catalogo | Schede prodotto, categorie, prezzi, disponibilità |
| Preventivi & Fatture | Generazione PDF, stati pagamento, totali automatici |
| Campagne Marketing | Stato campagne, budget, ROI, canali |
| Ticket di supporto | Priorità, assegnatario, SLA |
| Qualsiasi altra cosa | Descrivi cosa ti serve e viene costruita da zero |

La skill adatta l'intero schema database, le pagine e le relazioni in base a quello che chiedi.

### Dati demo vs dati reali

| Modalità | Come funziona |
|---|---|
| **Dati demo** (default) | 10 aziende, 20 contatti, 15 opportunità inseriti automaticamente al primo deploy — utile per presentazioni e test |
| **Dati reali** | Dì "non voglio dati demo" e il CRM parte vuoto, pronto per inserire i tuoi dati reali |
| **Importazione** | Puoi chiedere di aggiungere un'interfaccia di import da CSV/Excel per caricare dati esistenti |

---

## Installazione

### Metodo 1 — Comando curl (consigliato)

```bash
mkdir -p ~/.claude/skills/crm-creator && curl -o ~/.claude/skills/crm-creator/SKILL.md https://raw.githubusercontent.com/advantageplus-ai/claude-skill-crm-creator/main/SKILL.md
```

### Metodo 2 — Chiedi a Claude Code di installarla

Copia e incolla questo messaggio in una conversazione Claude Code:

> Installa questa skill nel mio Claude Code: https://raw.githubusercontent.com/advantageplus-ai/claude-skill-crm-creator/main/SKILL.md — salvala in `~/.claude/skills/crm-creator/SKILL.md`

### Metodo 3 — Manuale

1. Scarica `SKILL.md` da questa repo
2. Copia il file in `~/.claude/skills/crm-creator/SKILL.md`
3. Riavvia Claude Code

---

## Come usarla

Dopo l'installazione, in qualsiasi conversazione Claude Code digita:

```
/crm-creator
```

La skill farà alcune domande (nome azienda, settore, lingua) e poi costruirà tutto il progetto.

---

## Requisiti

- [Claude Code](https://claude.ai/code) installato
- Node.js 20+
- Account [Railway](https://railway.app) (gratuito) per il deploy
- Account [GitHub](https://github.com) per il repo

---

## Stack tecnico

```
Next.js 16.2.6    App Router, React Server Components, Server Actions
Prisma 7.8.0      Driver adapter obbligatorio per PostgreSQL
PostgreSQL         Su Railway (gratis fino a 1GB)
Tailwind CSS v4   Utility classes custom (card, btn, badge, input)
Recharts          Grafici dashboard e analytics
lucide-react      Icone
```

### Note importanti su Prisma 7

Prisma 7 ha breaking changes rispetto alle versioni precedenti — la skill le gestisce automaticamente:

- `generator provider = "prisma-client"` (non `prisma-client-js`)
- Nessun `url` in `datasource db {}` — va in `prisma.config.ts`
- Costruttore: `new PrismaPg(process.env.DATABASE_URL!)`
- Client generato in `app/generated/prisma/`

---

## Struttura progetto generata

```
my-crm/
├── app/
│   ├── (crm)/
│   │   ├── layout.tsx           # Sidebar + layout condiviso
│   │   ├── dashboard/page.tsx
│   │   ├── aziende/
│   │   │   ├── page.tsx
│   │   │   └── [id]/page.tsx
│   │   ├── contatti/
│   │   │   ├── page.tsx
│   │   │   └── [id]/page.tsx
│   │   ├── pipeline/page.tsx
│   │   ├── analytics/page.tsx
│   │   ├── calendario/page.tsx
│   │   ├── attivita/page.tsx
│   │   └── impostazioni/page.tsx
│   └── generated/prisma/        # Client Prisma generato
├── components/                  # Modal, actions, sidebar, header
├── lib/
│   ├── prisma.ts
│   ├── actions.ts               # Server Actions (CRUD)
│   └── utils.ts
├── prisma/
│   ├── schema.prisma
│   ├── prisma.config.ts
│   └── seed.ts
└── railway.json
```

---

## Deploy su Railway

La skill genera automaticamente `railway.json` con la configurazione corretta:

```json
{
  "deploy": {
    "preDeployCommand": "npx prisma db push && npx tsx prisma/seed.ts",
    "startCommand": "npm run start"
  }
}
```

Il `preDeployCommand` esegue le migrazioni e il seed **una sola volta** prima di avviare il container.

---

## Varianti supportate

La skill adatta il CRM al tipo di business:

| Tipo | Personalizzazioni |
|---|---|
| **Agenzia marketing** | Pipeline campagne, clienti, brief |
| **SaaS / Tech** | MRR, churn, trial → paid |
| **E-commerce** | Ordini, prodotti, customer value |
| **Immobiliare** | Immobili, trattative, agenti |

---

Creata da [Advantage Plus](www.advantageplus.ai) — agenzia AI.
