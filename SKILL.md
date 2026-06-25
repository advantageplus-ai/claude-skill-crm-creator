---
name: crm-creator
description: Crea un CRM completo e funzionante con Next.js, Prisma e PostgreSQL. Interfaccia in italiano, dati demo realistici, deploy su GitHub + Railway. Usa quando l'utente vuole costruire un gestionale CRM da zero.
---

# CRM Creator — Skill di Creazione CRM Completo

## Quando Usarla

Invoca questa skill quando l'utente dice:
- "voglio creare un CRM"
- "costruisci un gestionale per la mia azienda"
- "crea un sistema CRM con dashboard, aziende, contatti"
- "voglio un CRM come HubSpot/Salesforce ma personalizzato"

---

## Step 1 — Domande Preliminari

Prima di iniziare, raccogli queste informazioni tramite `AskUserQuestion`:

```
1. Nome azienda e settore (es. "Advantage Plus - Agenzia Marketing")
2. Lingua interfaccia (Italiano / Inglese)
3. Tipo di business:
   - Agenzia marketing/consulenza
   - E-commerce / Retail
   - SaaS / Tech
   - Immobiliare
   - Altro (specificare)
4. Design preferito:
   - Light professionale (HubSpot style) — default consigliato
   - Dark mode
   - Custom (chiedere colori)
5. Sezioni extra desiderate oltre alle standard:
   - Standard incluse sempre: Dashboard, Aziende, Contatti, Pipeline, Analytics, Calendario, Attività
   - Opzionali: Preventivi, Fatture, Report PDF, Campagne, Prodotti/Servizi
```

---

## Step 2 — Stack Tecnologico

### Stack Standard (usare sempre questo)

```
Framework:    Next.js 16+ (App Router, React Server Components)
ORM:          Prisma 7+ (con driver adapter obbligatorio)
Database:     SQLite locale → PostgreSQL su Railway
Styling:      Tailwind CSS v4
Grafici:      Recharts
Icone:        lucide-react
Linguaggio:   TypeScript
```

### ⚠️ Attenzione — Prisma 7 Breaking Changes

Prisma 7 richiede **driver adapters obbligatori**. Non usare il client standard.

**Per SQLite (sviluppo locale):**
```bash
npm install @prisma/adapter-better-sqlite3 better-sqlite3 @types/better-sqlite3
```

**Per PostgreSQL (produzione Railway):**
```bash
npm install @prisma/adapter-pg pg @types/pg
```

**`lib/prisma.ts` per PostgreSQL:**
```typescript
import { PrismaClient } from "@/app/generated/prisma/client";
import { PrismaPg } from "@prisma/adapter-pg";

function createPrismaClient(): PrismaClient {
  const adapter = new PrismaPg(process.env.DATABASE_URL!);
  return new PrismaClient({ adapter });
}

const globalForPrisma = globalThis as unknown as { prisma: PrismaClient };
export const prisma = globalForPrisma.prisma || createPrismaClient();
if (process.env.NODE_ENV !== "production") globalForPrisma.prisma = prisma;
export default prisma;
```

**`prisma/schema.prisma`:**
```prisma
generator client {
  provider = "prisma-client"
  output   = "../app/generated/prisma"
}

datasource db {
  provider = "postgresql"
  // NON mettere url qui — va in prisma.config.ts
}
```

> **Nota:** In Prisma 7 la `url` NON va nel `datasource` dello schema, va solo in `prisma.config.ts` (auto-generato). Il campo `provider` del generator è `"prisma-client"` (non `"prisma-client-js"`).

---

## Step 3 — Schema Database Prisma

Schema base per un CRM di agenzia marketing/consulenza. Adatta i campi al tipo di business dell'utente.

```prisma
model Azienda {
  id          Int       @id @default(autoincrement())
  nome        String
  settore     String
  sito        String?
  indirizzo   String?
  citta       String?
  cap         String?
  paese       String    @default("Italia")
  telefono    String?
  email       String?
  piva        String?
  dimensione  String?   // Micro, Piccola, Media, Grande
  fatturato   Float?
  note        String?
  stato       String    @default("Attivo") // Attivo, Prospect, Inattivo
  createdAt   DateTime  @default(now())
  updatedAt   DateTime  @updatedAt

  contatti    Contatto[]
  opportunita Opportunita[]
  attivita    Attivita[]
}

model Contatto {
  id          Int       @id @default(autoincrement())
  nome        String
  cognome     String
  email       String?
  telefono    String?
  cellulare   String?
  ruolo       String?
  reparto     String?
  linkedin    String?
  note        String?
  stato       String    @default("Attivo")
  createdAt   DateTime  @default(now())
  updatedAt   DateTime  @updatedAt

  aziendaId   Int?
  azienda     Azienda?  @relation(fields: [aziendaId], references: [id])
  attivita    Attivita[]
  opportunita OpportunityContact[]
}

model Opportunita {
  id           Int       @id @default(autoincrement())
  titolo       String
  valore       Float
  fase         String    @default("Lead")
  // Fasi: Lead, Qualificato, Proposta, Negoziazione, Chiusa Vinta, Chiusa Persa
  probabilita  Int       @default(10)
  dataChiusura DateTime?
  fonte        String?
  descrizione  String?
  note         String?
  createdAt    DateTime  @default(now())
  updatedAt    DateTime  @updatedAt

  aziendaId    Int?
  azienda      Azienda?  @relation(fields: [aziendaId], references: [id])
  contatti     OpportunityContact[]
  attivita     Attivita[]
}

model OpportunityContact {
  opportunitaId Int
  contattoId    Int
  opportunita   Opportunita @relation(fields: [opportunitaId], references: [id])
  contatto      Contatto    @relation(fields: [contattoId], references: [id])

  @@id([opportunitaId, contattoId])
}

model Attivita {
  id            Int       @id @default(autoincrement())
  tipo          String    // Chiamata, Email, Meeting, Task, Nota
  titolo        String
  descrizione   String?
  stato         String    @default("Da fare") // Da fare, In corso, Completata
  priorita      String    @default("Media")   // Alta, Media, Bassa
  scadenza      DateTime?
  completata    Boolean   @default(false)
  createdAt     DateTime  @default(now())
  updatedAt     DateTime  @updatedAt

  aziendaId     Int?
  azienda       Azienda?     @relation(fields: [aziendaId], references: [id])
  contattoId    Int?
  contatto      Contatto?    @relation(fields: [contattoId], references: [id])
  opportunitaId Int?
  opportunita   Opportunita? @relation(fields: [opportunitaId], references: [id])
}

model Evento {
  id            Int       @id @default(autoincrement())
  titolo        String
  descrizione   String?
  tipo          String    @default("Meeting") // Meeting, Call, Scadenza, Altro
  inizio        DateTime
  fine          DateTime?
  luogo         String?
  colore        String    @default("#3B82F6")
  tuttoIlGiorno Boolean   @default(false)
  createdAt     DateTime  @default(now())
  updatedAt     DateTime  @updatedAt
}
```

---

## Step 4 — Struttura File del Progetto

```
app/
├── (crm)/                    ← Route group con sidebar condivisa
│   ├── layout.tsx            ← Layout con <Sidebar />
│   ├── dashboard/page.tsx    ← KPI + grafici + feed attività
│   ├── aziende/
│   │   ├── page.tsx          ← Griglia card aziende
│   │   └── [id]/page.tsx     ← Dettaglio azienda
│   ├── contatti/
│   │   ├── page.tsx          ← Tabella contatti
│   │   └── [id]/page.tsx     ← Profilo contatto
│   ├── pipeline/page.tsx     ← Kanban board opportunità
│   ├── analytics/page.tsx    ← Grafici avanzati
│   ├── calendario/
│   │   ├── page.tsx          ← Server component (carica eventi)
│   │   └── CalendarioClient.tsx ← Client component (logica calendario)
│   ├── attivita/page.tsx     ← Task list con filtri
│   └── impostazioni/page.tsx ← Impostazioni profilo
├── generated/prisma/         ← Client Prisma auto-generato (in .gitignore)
└── globals.css               ← Variabili CSS + classi utility

components/
├── Sidebar.tsx               ← Navigazione laterale
├── Header.tsx                ← Header pagina con titolo + azioni
├── Modal.tsx                 ← Modal riutilizzabile (ESC + click fuori)
├── NuovaAziendaModal.tsx     ← Form creazione azienda
├── NuovoContattoModal.tsx    ← Form creazione contatto
├── NuovoEventoModal.tsx      ← Form creazione evento
├── NuovaAttivitaModal.tsx    ← Form creazione attività
├── ModificaAziendaModal.tsx  ← Form modifica azienda (pre-compilato)
├── ModificaContattoModal.tsx ← Form modifica contatto (pre-compilato)
├── DeleteButton.tsx          ← Pulsante elimina con conferma inline
├── AziendaCardActions.tsx    ← Azioni hover su card azienda (edit/delete)
└── ContattoRowActions.tsx    ← Azioni hover su riga contatto (edit/delete)

lib/
├── prisma.ts                 ← Singleton Prisma Client
├── actions.ts                ← Server Actions (create/update/delete)
└── utils.ts                  ← Helpers: formatCurrency, formatDate, getInitials, ecc.

prisma/
├── schema.prisma
└── seed.ts                   ← Dati demo (idempotente: skip se già popolato)
```

---

## Step 5 — Dati Demo (Seed)

Il seed deve essere **idempotente** — non duplica dati se eseguito più volte:

```typescript
async function main() {
  const existing = await prisma.azienda.count();
  if (existing > 0) {
    console.log("Database già popolato, skip seed.");
    return;
  }
  // ... inserimento dati
}
```

**Quantità minima consigliata:**
- 8-10 aziende con settori diversi e dati realistici
- 12-15 contatti distribuiti tra le aziende
- 10-12 opportunità in fasi diverse della pipeline
- 8-10 attività con priorità miste
- 8-10 eventi nel calendario (inclusi meeting e scadenze)

**Per agenzie marketing italiane, usa questi settori:**
Tecnologia & Software, Moda & Fashion, Alimentare & Bio, Edilizia & Real Estate, Farmaceutico & Salute, Hospitality & Turismo, Retail & E-commerce, Media & Comunicazione

---

## Step 6 — Pattern Fondamentali

### Server Actions (`lib/actions.ts`)

```typescript
"use server";
import { revalidatePath } from "next/cache";
import prisma from "@/lib/prisma";

export async function createAzienda(formData: FormData) {
  const nome = formData.get("nome") as string;
  if (!nome?.trim()) return { error: "Nome obbligatorio." };

  await prisma.azienda.create({ data: { nome: nome.trim(), /* ... */ } });
  revalidatePath("/aziende");
  return { success: true };
}

export async function updateAzienda(formData: FormData) {
  const id = parseInt(formData.get("id") as string);
  await prisma.azienda.update({ where: { id }, data: { /* campi */ } });
  revalidatePath("/aziende");
  revalidatePath(`/aziende/${id}`);
  return { success: true };
}

export async function deleteAzienda(id: number) {
  // Cascade manuale: pulisci relazioni prima di eliminare
  await prisma.contatto.updateMany({ where: { aziendaId: id }, data: { aziendaId: null } });
  await prisma.attivita.updateMany({ where: { aziendaId: id }, data: { aziendaId: null } });
  await prisma.opportunita.deleteMany({ where: { aziendaId: id } });
  await prisma.azienda.delete({ where: { id } });
  revalidatePath("/aziende");
  return { success: true };
}
```

### Passare Server Actions a Client Components

```tsx
// ✅ CORRETTO — usa .bind() per creare una Server Action serializzabile
<DeleteButton action={deleteAzienda.bind(null, azienda.id)} />

// ❌ SBAGLIATO — le closure non sono serializzabili
<DeleteButton action={() => deleteAzienda(azienda.id)} />
```

### Rendering Dinamico (obbligatorio per Railway)

Aggiungere su **ogni pagina** che fa query al DB:
```typescript
export const dynamic = "force-dynamic";
```
Senza questo, Next.js tenta di pre-renderizzare le pagine a build time quando il DB non è disponibile → build fallisce.

### CSS Utility Classes (`globals.css`)

```css
.card { @apply bg-white rounded-2xl border border-slate-100 shadow-sm; }
.btn { @apply inline-flex items-center gap-2 px-4 py-2 rounded-xl text-sm font-medium transition-colors; }
.btn-primary { @apply btn bg-blue-600 text-white hover:bg-blue-700; }
.btn-secondary { @apply btn bg-white border border-slate-200 text-slate-700 hover:bg-slate-50; }
.input { @apply w-full px-3 py-2 rounded-xl border border-slate-200 text-sm focus:outline-none focus:ring-2 focus:ring-blue-500; }
.badge { @apply inline-flex items-center px-2 py-0.5 rounded-full text-xs font-medium; }
```

---

## Step 7 — Deploy GitHub + Railway

### `railway.json`

```json
{
  "$schema": "https://railway.app/railway.schema.json",
  "build": {
    "builder": "NIXPACKS",
    "buildCommand": "npm run build"
  },
  "deploy": {
    "preDeployCommand": "npx prisma db push && npx tsx prisma/seed.ts",
    "startCommand": "npm run start",
    "healthcheckPath": "/",
    "healthcheckTimeout": 100,
    "restartPolicyType": "ON_FAILURE",
    "restartPolicyMaxRetries": 10
  }
}
```

> **Nota:** `prisma db push` va in `preDeployCommand`, NON in `startCommand`. Se fosse nello startCommand causerebbe restart loop quando il DB è temporaneamente non disponibile.

### `package.json` — scripts

```json
{
  "scripts": {
    "dev": "next dev",
    "build": "next build",
    "start": "next start",
    "db:seed": "tsx prisma/seed.ts",
    "db:push": "prisma db push",
    "postinstall": "prisma generate"
  },
  "engines": {
    "node": ">=20.19.0"
  }
}
```

### `.gitignore` — aggiungi sempre

```
.env*
*.db
*.db-journal
/app/generated/prisma
/.next/
/node_modules
```

### `.env.example`

```
DATABASE_URL="postgresql://user:password@host:5432/dbname"
```

### Procedura Deploy

1. **GitHub:** `gh repo create <nome> --private && git push -u origin main`
2. **Railway:** New Project → Deploy from GitHub repo → seleziona repo
3. **PostgreSQL:** In Railway → + New → Database → Add PostgreSQL
4. **Variabili:** Nel servizio Next.js → Variables → `DATABASE_URL` = `${{Postgres.DATABASE_URL}}`
5. Railway rideploya automaticamente → `prisma db push` crea le tabelle → seed popola i dati

---

## Step 8 — Checklist Completamento

Prima di dichiarare il CRM pronto, verifica:

- [ ] `npx tsc --noEmit` → zero errori TypeScript
- [ ] `npm run build` → build pulita (nessun errore)
- [ ] Tutte le pagine hanno `export const dynamic = "force-dynamic"`
- [ ] Il seed è idempotente (controlla `count > 0` prima di inserire)
- [ ] `.env` è in `.gitignore` (non committare credenziali)
- [ ] `/app/generated/prisma` è in `.gitignore`
- [ ] `railway.json` usa `preDeployCommand` per `prisma db push`
- [ ] `package.json` ha `"postinstall": "prisma generate"` (per Railway build)
- [ ] Tutti i form modali usano `.bind()` per le Server Actions

---

## Varianti per Tipo di Business

### Agenzia Marketing / Consulenza (default)
Sezioni extra utili: Campagne, Report Clienti, Preventivi

### E-commerce / Retail
Modelli aggiuntivi: Prodotto, Ordine, Reso
Sezioni: Catalogo Prodotti, Gestione Ordini, Reso/Rimborsi

### SaaS / Tech
Modelli aggiuntivi: Abbonamento, Ticket Supporto, Feature Request
Sezioni: Abbonamenti, Supporto, Roadmap Feedback

### Immobiliare
Modelli aggiuntivi: Immobile, Visita, Contratto
Sezioni: Portfolio Immobili, Visite Programmate, Contratti

---

## Note Tecniche Importanti

- **lucide-react v1.16+**: l'icona `Linkedin` è stata rimossa → usa `Link2`
- **Recharts Tooltip**: il `formatter` riceve `value: unknown`, fare cast `Number(value)`
- **Next.js 16 params**: i `params` delle pagine dinamiche sono `Promise<{id: string}>` → usare `await params`
- **`revalidatePath`**: dopo ogni mutation, chiamare sia `/lista` che `/lista/[id]` per aggiornare entrambe
- **SQLite → PostgreSQL**: il `datasource provider` in schema.prisma cambia ma la `url` resta sempre in `prisma.config.ts`, mai nel schema
