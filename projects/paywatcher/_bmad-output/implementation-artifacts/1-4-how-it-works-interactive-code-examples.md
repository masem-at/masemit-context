# Story 1.4: How It Works & Interactive Code Examples

Status: done

<!-- Note: Validation is optional. Run validate-create-story for quality check before dev-story. -->

## Story

As a Besucher,
I want den 3-Step "How It Works" Flow und interaktive Code-Beispiele sehen und kopieren koennen,
so that ich verstehe wie einfach die PayWatcher-Integration ist.

## Acceptance Criteria

1. **Given** ich scrolle zur "How It Works" Section (FR5) **When** die Section sichtbar ist **Then** sehe ich eine visuelle 3-Step-Darstellung: (1) Create Payment Intent → (2) Customer Pays → (3) Webhook Fires **And** die Animation respektiert prefers-reduced-motion

2. **Given** ich scrolle zur Code Examples Section (FR2) **When** die Section sichtbar ist **Then** sehe ich Code-Beispiele mit Tabs fuer mindestens curl und JavaScript **And** jedes Code-Beispiel hat Syntax Highlighting (Shiki, server-side rendered) und einen Copy-Button **And** der Copy-Button zeigt Inline-Checkmark-Feedback (kein Toast)

3. **Given** ich klicke auf den Copy-Button eines Code-Beispiels **When** der Code in die Zwischenablage kopiert wird **Then** wechselt das Clipboard-Icon fuer 1.5s zu einem Checkmark und kehrt dann zurueck

4. **Given** die Seite auf Mobile geladen wird **When** ein Code-Block breiter als der Viewport ist **Then** wird horizontal gescrollt (kein Umbruch), mit dezentem Scroll-Indicator

## Tasks / Subtasks

- [x] Task 1: shadcn/ui Tabs installieren und CodeTabs Client Component erstellen (AC: #2)
  - [x] 1.1 `npx shadcn@latest add tabs` ausfuehren
  - [x] 1.2 `src/components/landing/code-tabs.tsx` erstellen — Client Component (`'use client'`), shadcn/ui Tabs mit Tab-Wechsel fuer verschiedene Sprachen (curl, JavaScript). Empfaengt vorgerendertes Shiki-HTML als children/props vom Server Component.
  - [x] 1.3 Verifizieren: Tab-Wechsel funktioniert sofort (kein Laden, kein Flackern), Shiki-HTML korrekt gerendert in allen Tabs

- [x] Task 2: How It Works Section erstellen (AC: #1)
  - [x] 2.1 `src/components/landing/how-it-works.tsx` erstellen — Server Component
  - [x] 2.2 3-Step Flow visuell darstellen: (1) "Create Payment Intent" mit curl-Beispiel, (2) "Customer Sends USDC" mit Deposit Address, (3) "Webhook Fires" mit JSON Payload
  - [x] 2.3 Steps mit Verbindungslinien/Pfeilen verbinden (CSS-only, keine JS-Animation)
  - [x] 2.4 Section-ID `id="how-it-works"` setzen fuer Anker-Navigation vom Header
  - [x] 2.5 `prefers-reduced-motion` respektieren: bei aktivierter Einstellung keine Animationen, alle Steps sofort sichtbar
  - [x] 2.6 Responsive: auf Mobile einspaltig gestackt, auf Desktop dreispaltig oder horizontal

- [x] Task 3: Code Examples Section mit Tabs erstellen (AC: #2, #3)
  - [x] 3.1 `src/components/landing/code-examples.tsx` erstellen — Async Server Component
  - [x] 3.2 Code-Beispiele definieren fuer: curl (Create Payment Intent + Get Payment Status), JavaScript/Node.js (fetch-basiert)
  - [x] 3.3 Alle Code-Varianten server-seitig mit Shiki rendern (codeToHtml), vorgerendertes HTML an CodeTabs Client Component uebergeben
  - [x] 3.4 Jeder Code-Block mit CopyButton (bestehende Shared Component)
  - [x] 3.5 Sprache "javascript" zu Shiki hinzufuegen (bisher nur bash + json)
  - [x] 3.6 Layout: max-w-5xl zentriert, Section-Spacing gemaess UX-Spec (space-16)

- [x] Task 4: Landing Page aktualisieren und integrieren (AC: #1, #2, #4)
  - [x] 4.1 `src/app/(landing)/page.tsx` aktualisieren — HowItWorks + CodeExamples Sections nach ProblemSolution einbinden
  - [x] 4.2 Sections-Reihenfolge: Hero → ProblemSolution → HowItWorks → CodeExamples (Pricing/Trust kommt in Story 1.5)
  - [x] 4.3 Sicherstellen: Anker-Navigation `#how-it-works` vom Header funktioniert
  - [x] 4.4 Sicherstellen: Kein `'use client'` auf page.tsx — SSG muss funktionieren

- [x] Task 5: Build-Validierung und Qualitaetspruefung (Alle ACs)
  - [x] 5.1 `npm run build` muss fehlerfrei durchlaufen
  - [x] 5.2 `npm run type-check` muss fehlerfrei durchlaufen
  - [x] 5.3 `npm run lint` muss fehlerfrei durchlaufen
  - [x] 5.4 Verifizieren: Landing Page zeigt als `○ (Static)` im Build Output (SSG bestaetigt)
  - [x] 5.5 Visuelle Pruefung: Dark Mode korrekt, Light Mode korrekt (Shiki Dual-Theme), Tabs funktionieren
  - [x] 5.6 Responsive: Code-Bloecke horizontal scrollbar auf Mobile, How It Works responsiv
  - [x] 5.7 Copy-Button: Klick kopiert Code in Clipboard, Checkmark-Feedback erscheint (1.5s)
  - [x] 5.8 prefers-reduced-motion: Animation deaktiviert wenn Einstellung aktiv

## Dev Notes

### Kontext & Zweck

Dies ist die **vierte Story** des PayWatcher-Projekts. Story 1.1 lieferte das Projekt-Scaffold, Story 1.2 das Design System, Story 1.3 den oberen Teil der Landing Page (Header, Hero, Problem/Solution, Footer). Diese Story ergaenzt die **mittleren Sektionen** der Landing Page: "How It Works" (3-Step Flow) und interaktive Code-Beispiele mit Sprach-Tabs.

**Was PayWatcher ist:** Developer-First Verification API fuer Stablecoin-Zahlungen (USDC auf Base Network). Non-custodial, reiner Verification Layer. Die Landing Page muss in 30 Sekunden die Value Proposition kommunizieren. Code-Beispiele sind das Hauptargument, nicht Grafiken oder Animationen.

**Design-Philosophie (UX-Spec):** "Stripe-Style, nicht Vercel-Style." Nuechtern-technisch. Der Wow-Moment ist im Code-Beispiel, nicht in Animationen. Subtile Animation im "How It Works" ist die EINZIGE Ausnahme zur nuechtern-technischen Linie — informativ, nicht dekorativ. `prefers-reduced-motion` MUSS respektiert werden.

**Was diese Story liefert:**
- How It Works Section mit 3-Step-Visualisierung
- Code Examples Section mit Sprach-Tabs (curl, JavaScript)
- Tab-Wechsel ohne Neuladen (Shiki server-side pre-rendered, client-side Tab-Switch)
- Anker-Navigation `#how-it-works` funktionierend
- SSG Pre-Rendering bestaetigt

**Was diese Story NICHT beinhaltet:**
- Pricing Section (Story 1.5)
- Comparison/Trust Signals (Story 1.5)
- SEO Meta Tags, OG Images, Sitemap, Analytics (Story 1.6)
- Signup-Formular/-Flow (Epic 2) — CTA linkt nur auf `/signup`

### Technische Anforderungen

**Zu installierende Packages:**

| Package | Version | Zweck |
|---------|---------|-------|
| shadcn/ui Tabs | (via `npx shadcn@latest add tabs`) | Tab-Wechsel fuer Code-Beispiel-Sprachen — basiert auf Radix UI Tabs |

**Shiki ist bereits installiert (v3.22.0)** — kein erneutes Installieren noetig. Neue Sprache `javascript` wird automatisch lazy-geladen.

**NICHT in dieser Story installieren:**
- `@tanstack/react-query` → Story 2.1
- `next-auth` / `better-auth` → Story 2.2
- `recharts` → Story 3.2
- `react-hook-form` + `zod` → Story 2.2

### Shiki Tab-Pattern (KRITISCH)

**Architektur-Entscheidung:** Alle Code-Varianten werden **server-seitig** mit Shiki gerendert, das vorgerenderte HTML wird an eine **Client Component** (CodeTabs) uebergeben, die nur die Sichtbarkeit per Tabs umschaltet. Kein Client-Side Shiki Rendering.

**Pattern:**

```typescript
// src/components/landing/code-examples.tsx — Async SERVER Component
import { codeToHtml } from 'shiki'
import { CodeTabs } from './code-tabs'
import { CopyButton } from '@/components/shared/copy-button'

const curlExample = `curl -X POST https://api.paywatcher.dev/v1/payments \\
  -H "x-api-key: YOUR_API_KEY" \\
  -d '{"amount": "49.99", "currency": "USDC"}'`

const jsExample = `const response = await fetch('https://api.paywatcher.dev/v1/payments', {
  method: 'POST',
  headers: {
    'x-api-key': 'YOUR_API_KEY',
    'Content-Type': 'application/json',
  },
  body: JSON.stringify({ amount: '49.99', currency: 'USDC' }),
})

const { data } = await response.json()
console.log(data.id, data.deposit_address)`

export async function CodeExamples() {
  // Server-Side Rendering — alle Varianten werden hier gerendert
  const curlHtml = await codeToHtml(curlExample, {
    lang: 'bash',
    themes: { light: 'github-light', dark: 'github-dark' },
    defaultColor: false,
  })

  const jsHtml = await codeToHtml(jsExample, {
    lang: 'javascript',
    themes: { light: 'github-light', dark: 'github-dark' },
    defaultColor: false,
  })

  return (
    <section id="code-examples" className="px-4 py-16">
      <div className="mx-auto max-w-5xl">
        <h2>Integrate in minutes</h2>
        <CodeTabs
          tabs={[
            { value: 'curl', label: 'cURL', html: curlHtml, code: curlExample },
            { value: 'javascript', label: 'JavaScript', html: jsHtml, code: jsExample },
          ]}
        />
      </div>
    </section>
  )
}
```

```typescript
// src/components/landing/code-tabs.tsx — CLIENT Component
'use client'
import { Tabs, TabsList, TabsTrigger, TabsContent } from '@/components/ui/tabs'
import { CopyButton } from '@/components/shared/copy-button'

interface CodeTab {
  value: string
  label: string
  html: string
  code: string
}

export function CodeTabs({ tabs }: { tabs: CodeTab[] }) {
  return (
    <Tabs defaultValue={tabs[0].value}>
      <TabsList>
        {tabs.map((tab) => (
          <TabsTrigger key={tab.value} value={tab.value}>
            {tab.label}
          </TabsTrigger>
        ))}
      </TabsList>
      {tabs.map((tab) => (
        <TabsContent key={tab.value} value={tab.value}>
          <div className="group relative rounded-lg border border-border bg-card overflow-hidden">
            <div className="relative">
              <CopyButton code={tab.code} />
              <div
                className="overflow-x-auto p-4 text-code font-mono [&>pre]:!bg-transparent"
                dangerouslySetInnerHTML={{ __html: tab.html }}
              />
            </div>
          </div>
        </TabsContent>
      ))}
    </Tabs>
  )
}
```

**WICHTIG:**
- `codeToHtml()` ist async — Server Component muss `async function` sein
- Shiki lazy-loaded Sprachen automatisch (JavaScript muss nicht manuell registriert werden)
- Dual-Theme CSS fuer Shiki existiert bereits in `globals.css` (aus Story 1.3)
- `defaultColor: false` erzeugt CSS Variables (`--shiki-light`, `--shiki-dark`)
- CopyButton bekommt den Raw-Code (String), nicht das HTML

### How It Works Section Content

**3-Step Flow (FR5):**

| Step | Titel | Beschreibung | Visuell |
|------|-------|-------------|---------|
| 1 | **Create Payment Intent** | "Tell us what you expect" | Code-Snippet: `POST /v1/payments` |
| 2 | **Customer Sends USDC** | "They pay to the deposit address" | Deposit Address + Amount |
| 3 | **Webhook Fires** | "We notify you when it's confirmed" | JSON Payload: `{ "event": "payment.confirmed" }` |

**Design-Regeln:**
- Steps sind visuell mit Verbindungselementen (Pfeile/Linien) verbunden
- Auf Desktop: dreispaltig (horizontal)
- Auf Mobile: einspaltig gestackt (vertikal)
- Subtile Einblend-Animation beim Scrollen (IntersectionObserver + CSS Transitions)
- `prefers-reduced-motion`: alle Steps sofort sichtbar, keine Animation
- Nummerierung: Ziffern 1, 2, 3 als visuelle Anker (nicht Icons)
- Section-ID: `id="how-it-works"` fuer Anker-Navigation

**ANTI-Pattern:**
- KEINE komplexe Infographic-Animation — nur Einblenden der Steps
- KEINE Auto-Play-Animation — Animation startet erst wenn Section im Viewport
- KEINE Lottie oder SVG-Animation — reine CSS Transitions

### Code Examples Section Content

**Beispiel-Code fuer Tabs:**

**Tab 1: cURL**
```bash
# Create a payment intent
curl -X POST https://api.paywatcher.dev/v1/payments \
  -H "x-api-key: YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"amount": "49.99", "currency": "USDC"}'

# Check payment status
curl https://api.paywatcher.dev/v1/payments/pay_2x7k... \
  -H "x-api-key: YOUR_API_KEY"
```

**Tab 2: JavaScript**
```javascript
// Create a payment intent
const response = await fetch('https://api.paywatcher.dev/v1/payments', {
  method: 'POST',
  headers: {
    'x-api-key': 'YOUR_API_KEY',
    'Content-Type': 'application/json',
  },
  body: JSON.stringify({ amount: '49.99', currency: 'USDC' }),
})
const { data } = await response.json()
console.log(data.id, data.deposit_address)

// Check payment status
const status = await fetch(
  `https://api.paywatcher.dev/v1/payments/${data.id}`,
  { headers: { 'x-api-key': 'YOUR_API_KEY' } }
)
const payment = await status.json()
console.log(payment.data.status) // "confirmed"
```

**Tab-Verhalten:**
- Default-Tab: curl (erste Tab aktiv)
- Wechsel sofort ohne Laden oder Flackern (alle Varianten vorgerendert)
- Tab-Auswahl-State bleibt lokal (kein URL-Parameter noetig fuer Landing Page)

### Responsive Strategy

| Element | Desktop (>=1024px) | Mobile (<1024px) |
|---------|-------------------|------------------|
| How It Works Steps | Dreispaltig, horizontal mit Verbindungslinien | Einspaltig, vertikal gestackt |
| Code Examples Tabs | Tab-Leiste horizontal, Code-Block max-w-5xl | Tab-Leiste horizontal, Code-Block volle Breite |
| Code-Bloecke | Ausreichend Platz, kein Scroll noetig | overflow-x-auto, horizontaler Scroll |

**Tailwind Breakpoint-Strategie:** Mobile-First. Base-Styles = Mobile. `lg:` Prefix fuer Desktop-Overrides.

### Architektur-Konformitaet

**Dateinamen:** `kebab-case.tsx` — IMMER. Keine PascalCase-Dateinamen.

**Komponenten-Pfade:**
- `src/app/(landing)/page.tsx` — AENDERN: HowItWorks + CodeExamples einbinden
- `src/components/landing/how-it-works.tsx` — NEU: 3-Step Flow Section
- `src/components/landing/code-examples.tsx` — NEU: Async Server Component mit Shiki
- `src/components/landing/code-tabs.tsx` — NEU: Client Component fuer Tab-Wechsel
- `src/components/ui/tabs.tsx` — NEU: shadcn/ui Tabs Component (via CLI)

**Server vs Client Components:**
- Server Components (DEFAULT): page.tsx, how-it-works.tsx, code-examples.tsx
- Client Components (NUR wo noetig): code-tabs.tsx (Tab-State via Radix UI Tabs)

**ANTI-Patterns:**
- KEIN `'use client'` auf code-examples.tsx — Shiki muss server-seitig rendern
- KEIN Client-Side Shiki Import — Shiki gehoert nur in Server Components
- KEINE dynamischen Daten / fetch Calls — alles statisch, SSG muss funktionieren
- KEIN Full-Page-Spinner oder Loading States — SSG liefert sofort
- KEINE hardcoded Farben — immer Design Token Klassen
- KEIN `tailwind.config.js` — Tailwind v4 CSS-first via @theme

### Library & Framework Anforderungen

**Shiki v3.22.0 (bereits installiert):**
- ESM-only Package
- `codeToHtml()` ist async — Server Component muss `async function` sein
- Dual-Theme via `themes: { light, dark }` + `defaultColor: false` → CSS Variables
- Sprachen fuer Story 1.4: `bash`, `json`, `javascript` (Shiki lazy-loaded automatisch)
- NICHT auf Edge Runtime verwenden — Serverless/Node.js Runtime
- Performance: `codeToHtml()` shorthand verwaltet intern einen Highlighter-Singleton — kein manuelles Caching noetig

**shadcn/ui Tabs (zu installieren):**
- Basiert auf `@radix-ui/react-tabs` (bereits als Dependency via `radix-ui` Paket)
- Client Component — braucht `'use client'`
- API: `Tabs`, `TabsList`, `TabsTrigger`, `TabsContent`
- `defaultValue` fuer Initial-Tab, kein kontrollierter State noetig
- Kann server-gerendertes HTML als children empfangen (Children-Prop-Pattern)

**shadcn/ui Komponenten die bereits installiert sind:**
- Button, Badge, Card, Sheet, DropdownMenu, Sonner

### Dateistruktur-Anforderungen

**Neue/geaenderte Dateien nach Abschluss:**

```
src/
├── app/
│   └── (landing)/
│       └── page.tsx                     # AENDERN: HowItWorks + CodeExamples einbinden
│
├── components/
│   ├── landing/
│   │   ├── how-it-works.tsx            # NEU: 3-Step Flow Server Component
│   │   ├── code-examples.tsx           # NEU: Async Server Component (Shiki)
│   │   └── code-tabs.tsx               # NEU: Client Component (Tab-Wechsel)
│   └── ui/
│       └── tabs.tsx                     # NEU: shadcn/ui Tabs (via CLI)
```

### Testing-Anforderungen

1. **Build-Validierung:** `npm run build` muss ohne Fehler durchlaufen
2. **Type-Check:** `npm run type-check` muss ohne Fehler durchlaufen
3. **Lint:** `npm run lint` muss ohne Fehler durchlaufen
4. **SSG-Validierung:** Landing Page muss als `○ (Static)` im Build Output erscheinen
5. **Visuelle Pruefung Dark Mode:** How It Works korrekt, Code-Tabs mit Syntax Highlighting, Tab-Wechsel funktioniert
6. **Visuelle Pruefung Light Mode:** Alle Farben invertiert korrekt (Shiki Dual-Theme)
7. **Tab-Wechsel:** Sofort, kein Flackern, kein Laden — alle Varianten sind vorgerendert
8. **Copy-Button:** Klick kopiert Raw-Code in Clipboard, Checkmark-Feedback erscheint (1.5s)
9. **Responsive:** Code-Bloecke horizontal scrollbar auf Mobile, How It Works gestackt auf Mobile
10. **prefers-reduced-motion:** Animation deaktiviert, alle Steps sofort sichtbar
11. **Anker-Navigation:** Klick auf "How It Works" im Header scrollt zur Section

### Vorherige Story Intelligence

**Story 1.3 Learnings (KRITISCH — direkte Vorgaenger-Story):**
- **CodeBlock + CopyButton sind bereits implementiert** — `src/components/shared/code-block.tsx` (async Server Component, 3 Varianten: full/compact/inline) und `src/components/shared/copy-button.tsx` (Client Component mit try/catch)
- CopyButton ist **immer sichtbar** (kein opacity-0/group-hover) — dieses Fix kam aus dem Code Review
- CopyButton hat **try/catch** fuer Clipboard API (Fehlerbehandlung fuer unsichere Kontexte)
- `navItems` in separater Datei `src/components/landing/nav-items.ts` extrahiert — "How It Works" Link (`#how-it-works`) ist bereits darin
- Shiki Dual-Theme CSS existiert in `globals.css` (`html.dark / html:not(.dark)`)
- Landing Layout (`(landing)/layout.tsx`) mit Header + Footer existiert
- Section-IDs (`id="hero"`, `id="problem-solution"`) sind bereits gesetzt — konsistent `id="how-it-works"` und `id="code-examples"` verwenden
- SSG bestaetigt funktionierend — keine `'use client'` auf page.tsx
- Mobile-First mit `lg:` Breakpoint fuer Desktop
- Sheet-Komponente fuer Mobile Menu installiert

**Story 1.2 Learnings:**
- `next-themes` mit `attribute="class"` → `.dark` Klasse auf `<html>`
- Type Scale verfuegbar: `text-display`, `text-h1`, `text-h2`, `text-h3`, `text-body`, `text-body-sm`, `text-caption`
- Status-Farben definiert: success, warning, error, neutral
- ThemeToggle existiert in `src/components/shared/theme-toggle.tsx`

**Story 1.1 Learnings:**
- Route Group `(landing)` existiert
- `geist` Package laedt Geist Sans + Geist Mono
- Pfad-Alias `@/` → `src/`

**Code Review 1.3 Fixes:**
- CopyButton opacity war vorher opacity-0 (unsichtbar auf Touch) → jetzt immer sichtbar
- Footer braucht "by masemIT" Branding
- navItems DRY in separater Datei
- try/catch fuer Clipboard API hinzugefuegt
- package-lock.json in File List dokumentieren

### Git Intelligence

**Letzte Commits:**
```
36a83f8 Story 1.3: Landing page layout, hero section, and problem/solution
5a45117 Story 1.2: Theme system, design tokens, and payment status colors
6da6119 Update Story 1.1 status to review with completion notes
48d5ba9 Story 1.1: Project scaffold with Next.js 16, Tailwind v4, shadcn/ui
```

**Relevante Dateien aus Story 1.3 (direkt wiederverwendbar):**
- `src/components/shared/code-block.tsx` — CodeBlock Server Component (wiederverwendbar in How It Works)
- `src/components/shared/copy-button.tsx` — CopyButton Client Component (wiederverwendbar)
- `src/components/landing/nav-items.ts` — navItems mit `#how-it-works` Link
- `src/app/globals.css` — Shiki Dual-Theme CSS

**Patterns aus vorherigen Stories:**
- Async Server Components fuer Shiki — bewaehrt
- Section-IDs fuer Anker-Navigation — konsistent fortfuehren
- Mobile-First mit `lg:` Breakpoint
- `max-w-5xl mx-auto` fuer zentriertes Layout
- `space-y-16` / `py-16` fuer Section-Spacing

### Aktuelle Technologie-Informationen (Web Research, Feb 2026)

**Shiki v3.22.0:**
- `codeToHtml()` Shorthand verwaltet intern einen Highlighter-Singleton
- Sprachen werden automatisch lazy-geladed — JavaScript muss nicht manuell registriert werden
- Dual-Theme (`defaultColor: false`) erzeugt `--shiki-light` / `--shiki-dark` CSS Variables
- Keine bekannten Issues mit Next.js 16
- Performance: Mehrere Code-Bloecke auf einer Seite sind kein Problem dank internem Caching

**shadcn/ui Tabs (Radix UI Tabs v1.1.13):**
- Client Component — `'use client'` Directive erforderlich
- API: `<Tabs defaultValue="...">`, `<TabsList>`, `<TabsTrigger value="...">`, `<TabsContent value="...">`
- Kann server-gerendertes HTML als children empfangen (Children-Prop-Pattern)
- Keyboard-Navigation built-in (Pfeil-Tasten zwischen Tabs)
- ARIA-Attribute automatisch gesetzt

**CSS Animations + prefers-reduced-motion:**
- IntersectionObserver ist der empfohlene Ansatz fuer scroll-basierte Animationen
- `@media (prefers-reduced-motion: reduce)` ist PFLICHT (WCAG 2.1 Technik C39)
- Pattern: `opacity: 0; transform: translateY(20px);` → `.visible` → `opacity: 1; transform: none;`
- `content-visibility: hidden` fuer bessere Performance bei versteckten Tab-Inhalten

### Project Structure Notes

- Landing Page ist eine **Single Page** mit Sektionen — How It Works und Code Examples sind die naechsten beiden nach Problem/Solution
- CodeBlock und CopyButton sind **Shared Components** — werden in code-examples.tsx nicht direkt verwendet, da CodeTabs sein eigenes Rendering macht (vorgerendertes HTML + CopyButton)
- Die **code-examples.tsx** ist eine async Server Component die `codeToHtml()` aufruft — sie rendert NICHT ueber die bestehende CodeBlock Component, sondern erstellt eigenes vorgerendertes HTML fuer den Tab-Wechsel
- Alternativ KANN die bestehende CodeBlock Component innerhalb der How It Works Section wiederverwendet werden (fuer die kleinen Code-Snippets in den Steps)
- **Component Boundaries beachten:** Landing Components importieren NIE aus `components/dashboard/`

### References

- [Source: _bmad-output/planning-artifacts/ux-design-specification.md#Design Opportunities] — "Subtile Animation im How It Works, einzige Ausnahme zur nuechtern-technischen Linie"
- [Source: _bmad-output/planning-artifacts/ux-design-specification.md#Flow 3: Landing Page Conversion] — Conversion-Flow: Problem/Solution → How It Works → Code Examples → Pricing
- [Source: _bmad-output/planning-artifacts/ux-design-specification.md#Accessibility Considerations] — "prefers-reduced-motion respektieren"
- [Source: _bmad-output/planning-artifacts/ux-design-specification.md#Component Implementation Strategy] — "HowItWorksAnimation (Phase 1, einziges animiertes Element)"
- [Source: _bmad-output/planning-artifacts/ux-design-specification.md#Dual Mental Model Messaging] — Code Examples: Payment Intent + Webhook vs. Transfer Verification
- [Source: _bmad-output/planning-artifacts/architecture.md#Component Patterns] — Button-Hierarchie, Copy-Feedback, Shiki Server-Side
- [Source: _bmad-output/planning-artifacts/architecture.md#Structure Patterns] — kebab-case Files, Component Boundaries
- [Source: _bmad-output/planning-artifacts/architecture.md#Naming Patterns] — PascalCase Components, kebab-case Dateien
- [Source: _bmad-output/planning-artifacts/architecture.md#Frontend Architecture] — Shiki: "Server-Side Rendering moeglich, bessere Performance als Prism"
- [Source: _bmad-output/planning-artifacts/epics.md#Story 1.4] — User Story und Acceptance Criteria
- [Source: _bmad-output/implementation-artifacts/1-3-landing-page-layout-hero-section-problem-solution.md] — Vorherige Story mit CodeBlock/CopyButton Patterns

## Dev Agent Record

### Agent Model Used

Claude Opus 4.6

### Debug Log References

Keine Debug-Probleme aufgetreten. Build, Type-Check und Lint liefen beim ersten Versuch fehlerfrei durch.

### Completion Notes List

- **Task 1:** shadcn/ui Tabs installiert (`npx shadcn@latest add tabs`), CodeTabs Client Component erstellt mit dangerouslySetInnerHTML fuer vorgerendertes Shiki-HTML und CopyButton Integration
- **Task 2:** How It Works Section als async Server Component mit 3-Step Flow, CSS Grid Layout (1 Spalte Mobile / 3 Spalten Desktop), SVG-Pfeil-Connectors, IntersectionObserver-basierter Scroll-Animation via AnimateOnScroll Client Component, prefers-reduced-motion Unterstuetzung
- **Task 3:** Code Examples Section als async Server Component, server-seitiges Shiki Rendering (bash + javascript), vorgerendertes HTML an CodeTabs Client Component uebergeben, beide Tabs (cURL + JavaScript) mit vollstaendigen Beispielen
- **Task 4:** Landing Page aktualisiert mit korrekter Sections-Reihenfolge (Hero → ProblemSolution → HowItWorks → CodeExamples), SSG bestaetigt (○ Static)
- **Task 5:** Build, Type-Check und Lint alle fehlerfrei. Landing Page als Static im Build Output bestaetigt.

### Change Log

- 2026-02-16: Story 1.4 implementiert — How It Works 3-Step Section, Code Examples mit Sprach-Tabs (cURL/JavaScript), Shiki server-side rendering, Scroll-Animation mit prefers-reduced-motion Support
- 2026-02-16: Code Review Fixes — CopyButton Timeout 2000ms→1500ms (AC#3), Mobile Scroll-Indicator hinzugefuegt (AC#4), FOUC-Fix noscript Fallback (M1), Grid-Arrows Zentrierung items-center (M2)

### File List

- src/components/ui/tabs.tsx (NEU — shadcn/ui Tabs via CLI)
- src/components/landing/code-tabs.tsx (NEU — Client Component fuer Tab-Wechsel)
- src/components/landing/how-it-works.tsx (NEU — 3-Step Flow Server Component)
- src/components/landing/code-examples.tsx (NEU — Async Server Component mit Shiki)
- src/components/landing/animate-on-scroll.tsx (NEU — Client Component fuer Scroll-Animation)
- src/app/(landing)/page.tsx (GEAENDERT — HowItWorks + CodeExamples eingebunden)
- src/app/globals.css (GEAENDERT — Scroll-Animation CSS + prefers-reduced-motion + Scroll-Indicator)
- src/app/layout.tsx (GEAENDERT — noscript Fallback fuer AnimateOnScroll FOUC)
- src/components/shared/copy-button.tsx (GEAENDERT — Timeout 2000ms→1500ms)
- src/components/shared/code-block.tsx (GEAENDERT — code-scroll Klasse fuer Scroll-Indicator)
- package-lock.json (GEAENDERT — shadcn/ui Tabs Dependencies)
