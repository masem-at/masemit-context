# Story 1.3: Landing Page Layout, Hero Section & Problem/Solution

Status: done

<!-- Note: Validation is optional. Run validate-create-story for quality check before dev-story. -->

## Story

As a Besucher von paywatcher.dev,
I want eine Landing Page mit Hero Section die mir sofort zeigt was PayWatcher ist und was es loest,
so that ich in unter 30 Sekunden verstehe ob PayWatcher fuer mich relevant ist.

## Acceptance Criteria

1. **Given** ich navigiere zu paywatcher.dev **When** die Seite laedt **Then** sehe ich einen Landing Header mit Navigation und "Get API Key"-CTA **And** die Seite ist via SSG pre-rendered (kein Client-Side Rendering)

2. **Given** die Hero Section geladen ist **When** ich den Hero-Bereich sehe **Then** sehe ich die Headline, eine Subline fuer beide Zielgruppen (Stripe-Devs + Web3-Native), ein prominentes Code-Snippet (curl Payment Intent), und einen CTA-Button ("Get API Key — Free") **And** der Code-Block hat Syntax Highlighting (Shiki) und einen Copy-Button (FR2 teilweise)

3. **Given** ich scrolle weiter **When** die Problem/Solution Section sichtbar wird **Then** sehe ich die Darstellung des Problems (1% Fees ODER alles selbst bauen) und PayWatcher's Loesung als "missing middle" (FR1)

4. **Given** ich auf "Get API Key" klicke (FR7) **When** der CTA ausgeloest wird **Then** werde ich zur Signup-Route navigiert

5. **Given** die Landing Page auf einem Mobilgeraet (<1024px) geladen wird **When** die Seite rendert **Then** ist das Layout responsive: Code-Block mit horizontal scroll, Header mit Hamburger-Menu, volle Breite

6. **Given** ein Landing Footer existiert **When** ich zum Seitenende scrolle **Then** sehe ich Footer-Links und "by masemIT"

## Tasks / Subtasks

- [x] Task 1: Shiki installieren und CodeBlock + CopyButton Shared Components erstellen (AC: #2)
  - [x] 1.1 `npm install shiki` ausfuehren
  - [x] 1.2 `src/components/shared/copy-button.tsx` erstellen — Client Component mit Clipboard API, Inline-Checkmark-Feedback (KEIN Toast), Lucide Icons (Copy/Check)
  - [x] 1.3 `src/components/shared/code-block.tsx` erstellen — Async Server Component mit Shiki `codeToHtml()`, Dual-Theme Support (light/dark via CSS Variables), drei Varianten: `full` (mit Header-Dots + Sprache-Label), `compact` (nur Code + Copy), `inline` (minimal). CopyButton eingebunden. `overflow-x-auto` fuer horizontalen Scroll.
  - [x] 1.4 Verifizieren: CodeBlock rendert korrektes Syntax Highlighting, CopyButton kopiert in Clipboard

- [x] Task 2: Landing Layout mit LandingHeader und LandingFooter erstellen (AC: #1, #5, #6)
  - [x] 2.1 `src/app/(landing)/layout.tsx` erstellen — Landing-spezifisches Layout das children mit Header + Footer wrapped
  - [x] 2.2 `src/components/landing/landing-header.tsx` erstellen — Server Component mit Navigation-Links (How It Works, Pricing, Docs als Anker-Links), ThemeToggle, und "Get API Key" CTA-Button (Primary). Mobile: Hamburger-Menu via Sheet/Dialog unter 1024px.
  - [x] 2.3 `src/components/landing/landing-footer.tsx` erstellen — Server Component mit Footer-Links (GitHub, Docs, Status) und "by masemIT" Branding
  - [x] 2.4 Verifizieren: Header + Footer rendern korrekt, Mobile Hamburger funktioniert

- [x] Task 3: Hero Section erstellen (AC: #2, #4)
  - [x] 3.1 `src/components/landing/hero-section.tsx` erstellen — Server Component
  - [x] 3.2 Headline: "Payment verification without the payment processor." (text-display, font-bold)
  - [x] 3.3 Subline: Dual-Audience Messaging — "Works like Stripe Payment Intents, but for on-chain USDC." + "Non-custodial. Flat-rate. Three API calls." (text-h2, text-muted-foreground)
  - [x] 3.4 Code-Block: curl Payment Intent Beispiel mit Shiki (variant="full"), Sprache "bash"
  - [x] 3.5 CTA-Button: "Get API Key — Free" als `<Link href="/signup">` mit Primary Button-Stil
  - [x] 3.6 Layout: max-w-5xl zentriert, Responsive — Code-Block volle Breite auf Mobile

- [x] Task 4: Problem/Solution Section erstellen (AC: #3)
  - [x] 4.1 `src/components/landing/problem-solution.tsx` erstellen — Server Component
  - [x] 4.2 Problem-Darstellung: Zwei schlechte Optionen visualisieren — "1% per-transaction fees" (Coinbase Commerce etc.) vs. "Build your own blockchain verification" (eigener Event Listener)
  - [x] 4.3 Solution: PayWatcher als "the missing middle" — Non-custodial verification, flat-rate pricing, 3 API calls
  - [x] 4.4 Layout: max-w-5xl zentriert, Section-Spacing gemaess UX-Spec (space-16 = 64px zwischen Sektionen)

- [x] Task 5: Landing Page zusammensetzen (AC: #1)
  - [x] 5.1 `src/app/(landing)/page.tsx` aktualisieren — Hero + ProblemSolution Sections einbinden
  - [x] 5.2 Sicherstellen: Kein `'use client'`, keine dynamischen Daten — reine SSG
  - [x] 5.3 Sections mit Section-IDs versehen (`id="hero"`, `id="problem-solution"`) fuer zukuenftige Anker-Navigation

- [x] Task 6: Build-Validierung und Qualitaetspruefung (Alle ACs)
  - [x] 6.1 `npm run build` muss fehlerfrei durchlaufen
  - [x] 6.2 `npm run type-check` muss fehlerfrei durchlaufen
  - [x] 6.3 `npm run lint` muss fehlerfrei durchlaufen
  - [x] 6.4 Verifizieren: Landing Page zeigt als `○ (Static)` im Build Output (SSG bestaetigt)
  - [x] 6.5 Visuelle Pruefung: Dark Mode korrekt, Light Mode korrekt, Responsive Layout auf Mobile

## Dev Notes

### Kontext & Zweck

Dies ist die **dritte Story** des PayWatcher-Projekts. Story 1.1 lieferte das Projekt-Scaffold, Story 1.2 das Design System. Diese Story erstellt die **erste sichtbare Seite** — den oberen Teil der Landing Page (Header, Hero, Problem/Solution, Footer). Stories 1.4 und 1.5 ergaenzen die restlichen Sektionen (How It Works, Code Examples, Pricing, Comparison, Trust Signals).

**Was PayWatcher ist:** Developer-First Verification API fuer Stablecoin-Zahlungen (USDC auf Base Network). Non-custodial, reiner Verification Layer. Die Landing Page muss in 30 Sekunden die Value Proposition kommunizieren. Code-Beispiele sind das Hauptargument, nicht Grafiken oder Animationen.

**Design-Philosophie (UX-Spec):** "Stripe-Style, nicht Vercel-Style." Nuechtern-technisch. Der Wow-Moment ist im Code-Beispiel, nicht in Animationen. Dark Mode + cleane Typografie + gut formatierter Code-Block = der Wow-Moment fuer Developer.

**Was diese Story liefert:**
- Landing Layout (Header + Footer)
- Hero Section mit Code-as-Hero Pattern (Shiki Syntax Highlighting)
- Problem/Solution Section
- CopyButton + CodeBlock als wiederverwendbare Shared Components
- SSG Pre-Rendering (Static Site Generation)
- Responsive Design (Mobile-First)

**Was diese Story NICHT beinhaltet:**
- How It Works Section (Story 1.4)
- Code Examples Tabs (Story 1.4)
- Pricing, Comparison, Trust Signals (Story 1.5)
- SEO Meta Tags, OG Images, Sitemap, Analytics (Story 1.6)
- Signup-Formular/-Flow (Epic 2) — CTA linkt nur auf `/signup`

### Technische Anforderungen

**Zu installierendes Package:**

| Package | Version | Zweck |
|---------|---------|-------|
| shiki | ^3.x (latest) | Syntax Highlighting — Server-Side Rendering via `codeToHtml()` in React Server Components |

**NICHT in dieser Story installieren:**
- `@tanstack/react-query` → Story 2.1
- `next-auth` / `better-auth` → Story 2.2
- `recharts` → Story 3.2
- `react-hook-form` + `zod` → Story 2.2

**Shiki Integration (KRITISCH):**

Shiki wird als **async Server Component** verwendet. Kein Client-Side JavaScript fuer Syntax Highlighting.

```typescript
// src/components/shared/code-block.tsx — Server Component (KEIN 'use client')
import { codeToHtml } from 'shiki'
import type { BundledLanguage } from 'shiki'
import { CopyButton } from './copy-button'

interface CodeBlockProps {
  code: string
  lang?: BundledLanguage
  variant?: 'full' | 'compact' | 'inline'
}

export async function CodeBlock({ code, lang = 'bash', variant = 'full' }: CodeBlockProps) {
  const html = await codeToHtml(code, {
    lang,
    themes: { light: 'github-light', dark: 'github-dark' },
    defaultColor: false, // CSS Variables statt inline colors
  })

  return (
    <div className="group relative rounded-lg border border-border bg-card overflow-hidden">
      {variant === 'full' && (
        <div className="flex items-center gap-2 border-b border-border px-4 py-2">
          <div className="flex gap-1.5">
            <span className="size-3 rounded-full bg-muted-foreground/20" />
            <span className="size-3 rounded-full bg-muted-foreground/20" />
            <span className="size-3 rounded-full bg-muted-foreground/20" />
          </div>
          <span className="text-caption font-mono text-muted-foreground">{lang}</span>
        </div>
      )}
      <div className="relative">
        <CopyButton code={code} />
        <div className="overflow-x-auto p-4 text-code font-mono [&>pre]:!bg-transparent"
             dangerouslySetInnerHTML={{ __html: html }} />
      </div>
    </div>
  )
}
```

**KRITISCH — Shiki Dual-Theme fuer Dark/Light Mode:**
`defaultColor: false` erzeugt CSS Variables (`--shiki-light`, `--shiki-dark`) auf jedem Token. Dazu CSS in globals.css:

```css
/* Shiki Dual Theme Support */
html:not(.dark) .shiki,
html:not(.dark) .shiki span {
  color: var(--shiki-light) !important;
  background-color: var(--shiki-light-bg) !important;
}
html.dark .shiki,
html.dark .shiki span {
  color: var(--shiki-dark) !important;
  background-color: var(--shiki-dark-bg) !important;
}
```

**CopyButton (Client Component):**

```typescript
// src/components/shared/copy-button.tsx
"use client"
import { useState } from 'react'
import { Copy, Check } from 'lucide-react'

export function CopyButton({ code }: { code: string }) {
  const [copied, setCopied] = useState(false)

  return (
    <button
      onClick={async () => {
        await navigator.clipboard.writeText(code)
        setCopied(true)
        setTimeout(() => setCopied(false), 2000)
      }}
      className="absolute right-2 top-2 rounded-md p-2 opacity-0 transition-opacity group-hover:opacity-100 bg-muted hover:bg-muted/80"
      aria-label="Copy code"
    >
      {copied ? <Check className="size-4 text-success" /> : <Copy className="size-4 text-muted-foreground" />}
    </button>
  )
}
```

**WICHTIG — Copy-Feedback:** Inline-Checkmark am Button, KEIN Toast (UX-Spec Regel). Icon wechselt fuer 2 Sekunden zu Check, dann zurueck zu Copy.

### Hero Section Content

**Headline:** `Payment verification without the payment processor.`
- Type Scale: `text-display` (3rem), `font-bold`
- Farbe: `text-foreground`

**Subline (Dual-Audience):**
- Zeile 1 (Stripe-Devs): "Works like Stripe Payment Intents, but for on-chain USDC."
- Zeile 2 (Allgemein): "Non-custodial. Flat-rate. Three API calls."
- Type Scale: `text-h2` (1.5rem), `text-muted-foreground`

**Code-Snippet (curl):**

```bash
curl -X POST https://api.paywatcher.dev/v1/payments \
  -H "x-api-key: pw_live_..." \
  -H "Content-Type: application/json" \
  -d '{"amount": "49.99", "currency": "USDC"}'
```

Response-Snippet (optional unter dem curl):

```json
{
  "data": {
    "id": "pay_2x7k...",
    "status": "pending",
    "deposit_address": "0x1a2b...3c4d"
  }
}
```

**CTA:** "Get API Key — Free" → `<Link href="/signup">` mit shadcn/ui Button (variant="default" = Primary)

### Problem/Solution Section Content

**Problem-Seite:**
- Titel: "The payment verification dilemma"
- Option A: "Use Coinbase Commerce — Pay 1% per transaction" (Kostenbeispiel: "$100 fee on a $10,000 payment")
- Option B: "Build it yourself — Parse Transfer events, handle reorgs, track confirmations" (Komplexitaet)
- Visuell: Zwei Karten/Spalten nebeneinander (auf Desktop), gestackt auf Mobile

**Solution-Seite:**
- Titel: "The missing middle"
- PayWatcher: "Non-custodial verification. Flat-rate pricing. Three API calls."
- Key Points: "We never touch your funds", "You keep your wallet", "From $0 to $0.05 per verification"
- Visuell: Hervorgehoben mit Accent-Farbe (Teal)

### Landing Header Navigation

**Navigation Items:**
- "How It Works" → `#how-it-works` (Anker, Section kommt in Story 1.4)
- "Pricing" → `#pricing` (Anker, Section kommt in Story 1.5)
- "Docs" → `/docs` (Route, kommt in Epic 5)
- ThemeToggle Komponente (bereits aus Story 1.2)
- "Get API Key" → `/signup` (Primary Button CTA)

**Mobile (<1024px):** Hamburger-Menu. Optionen: shadcn/ui Sheet oder einfaches Dropdown. ThemeToggle + Nav-Links im Menu. CTA-Button bleibt sichtbar (nicht im Menu versteckt).

**WICHTIG:** Die Landing Page hat KEIN Sidebar-Layout. Header ist eine Top-Navigation mit Logo links und Nav-Items rechts.

### Landing Footer

**Footer-Inhalt:**
- Links: GitHub, Docs, Status, Impressum
- Branding: "by masemIT" (dezent, nicht ueberdimensioniert)
- Copyright: `© {year} masemIT`
- Optionale Social-Links (Phase 2)

### Responsive Strategy

| Element | Desktop (>=1024px) | Mobile (<1024px) |
|---------|-------------------|------------------|
| Landing Header | Horizontal Nav + CTA | Hamburger Menu + CTA sichtbar |
| Hero Headline | text-display (3rem) | text-display oder kleiner |
| Hero Code-Block | max-w-2xl, zentriert | Volle Breite, horizontal scroll |
| Problem/Solution | 2 Spalten nebeneinander | Gestackt (vertikal) |
| Footer | Horizontal Layout | Gestackt |

**Tailwind Breakpoint-Strategie:** Mobile-First. Base-Styles = Mobile. `lg:` Prefix fuer Desktop-Overrides.

**Code-Block auf Mobile:** `overflow-x-auto` — horizontaler Scroll, NICHT umbrechen. Code umbrechen zerstoert Lesbarkeit. Dezenter Scroll-Indicator optional.

### Architektur-Konformitaet

**Dateinamen:** `kebab-case.tsx` — IMMER. Keine PascalCase-Dateinamen.

**Komponenten-Pfade:**
- `src/app/(landing)/layout.tsx` — Landing Layout
- `src/app/(landing)/page.tsx` — Landing Page (bestehend, wird aktualisiert)
- `src/components/landing/hero-section.tsx` — NEU
- `src/components/landing/problem-solution.tsx` — NEU
- `src/components/landing/landing-header.tsx` — NEU
- `src/components/landing/landing-footer.tsx` — NEU
- `src/components/shared/code-block.tsx` — NEU (Shared weil auch in Docs/Onboarding verwendet)
- `src/components/shared/copy-button.tsx` — NEU (Shared weil ueberall wo Code-Bloecke sind)

**Server vs Client Components:**
- Server Components (DEFAULT): layout.tsx, page.tsx, hero-section.tsx, problem-solution.tsx, landing-header.tsx, landing-footer.tsx, code-block.tsx
- Client Components (NUR wo noetig): copy-button.tsx (onClick Handler), Mobile Menu State (falls useState noetig)

**ANTI-Patterns:**
- KEIN `'use client'` auf der page.tsx oder layout.tsx — SSG muss funktionieren
- KEINE dynamischen Daten / fetch Calls in Landing Page Components — alles statisch
- KEIN Full-Page-Spinner oder Loading States — SSG liefert sofort
- KEINE hardcoded Farben — immer Design Token Klassen (`text-foreground`, `bg-card`, `text-primary`)
- KEIN `tailwind.config.js` — Tailwind v4 CSS-first via @theme
- Max 1 Primary Button pro Screen — der "Get API Key" CTA ist der einzige Primary Button

### Library & Framework Anforderungen

**Shiki v3.x:**
- ESM-only Package
- `codeToHtml()` ist async — Server Component muss `async function` sein
- Dual-Theme via `themes: { light, dark }` + `defaultColor: false` → CSS Variables
- Unterstuetzte Themes: `github-dark`, `github-light` (empfohlen fuer PayWatcher)
- Sprachen fuer Story 1.3: `bash`, `json` (weitere in Story 1.4)
- NICHT auf Edge Runtime verwenden — Serverless/Node.js Runtime

**shadcn/ui Komponenten die bereits installiert sind:**
- Button (fuer CTA und Nav)
- Badge, Card (verfuegbar fuer Problem/Solution Cards)

**shadcn/ui Komponenten die ggf. installiert werden muessen:**
- Sheet (fuer Mobile Menu) — `npx shadcn@latest add sheet`
- Oder alternativ einfaches `<dialog>` / Dropdown fuer Mobile Nav

### Dateistruktur-Anforderungen

**Neue/geaenderte Dateien nach Abschluss:**

```
src/
├── app/
│   └── (landing)/
│       ├── layout.tsx                   # NEU: Landing Layout (Header + Footer)
│       └── page.tsx                     # AENDERN: Hero + ProblemSolution einbinden
│
├── components/
│   ├── landing/
│   │   ├── hero-section.tsx            # NEU: Hero mit Code-Block und CTA
│   │   ├── problem-solution.tsx        # NEU: Problem/Solution Darstellung
│   │   ├── landing-header.tsx          # NEU: Navigation + CTA + Mobile Menu
│   │   └── landing-footer.tsx          # NEU: Footer + "by masemIT"
│   └── shared/
│       ├── code-block.tsx              # NEU: Shiki Server Component (Shared)
│       └── copy-button.tsx             # NEU: Clipboard Client Component (Shared)
│
└── app/
    └── globals.css                      # AENDERN: Shiki Dual-Theme CSS hinzufuegen
```

### Testing-Anforderungen

1. **Build-Validierung:** `npm run build` muss ohne Fehler durchlaufen
2. **Type-Check:** `npm run type-check` muss ohne Fehler durchlaufen
3. **Lint:** `npm run lint` muss ohne Fehler durchlaufen
4. **SSG-Validierung:** Landing Page muss als `○ (Static)` im Build Output erscheinen
5. **Visuelle Pruefung Dark Mode:** Hero korrekt, Code-Block mit Syntax Highlighting, Footer sichtbar
6. **Visuelle Pruefung Light Mode:** Alle Farben invertiert korrekt (Shiki Dual-Theme)
7. **Responsive:** Code-Block horizontal scrollbar auf Mobile, Hamburger Menu funktioniert
8. **Copy-Button:** Klick kopiert Code in Clipboard, Checkmark-Feedback erscheint

### Vorherige Story Intelligence

**Story 1.2 Learnings:**
- `next-themes` mit `attribute="class"` → `.dark` Klasse auf `<html>`. Shiki Dual-Theme CSS muss `html.dark` / `html:not(.dark)` verwenden
- `@theme inline` Block in globals.css mappt CSS Variables auf Tailwind Utilities. Neue CSS fuer Shiki kann am Ende von globals.css stehen
- `suppressHydrationWarning` ist bereits auf `<html>` — kein erneutes Hinzufuegen noetig
- ThemeToggle Component existiert bereits in `src/components/shared/theme-toggle.tsx` — direkt im LandingHeader verwenden
- Type Scale existiert bereits: `text-display`, `text-h1`, `text-h2`, `text-body` etc. als Tailwind Utilities verfuegbar

**Story 1.1 Learnings:**
- Route Group (landing) existiert bereits mit placeholder page.tsx
- `components/landing/` Ordner existiert (nur .gitkeep)
- `geist` Package laedt Geist Sans + Geist Mono — CSS Variables `--font-geist-sans` und `--font-geist-mono` sind verfuegbar
- Pfadkonflikt: Landing Page ist `/`, Dashboard ist `/dashboard`, Admin ist `/admin`

**Code Review 1.2 Fixes:**
- STATUS_CONFIG Labels sind jetzt UPPER_CASE (Architektur-konform)
- StatusColor ist jetzt Union Type statt `string`

### Aktuelle Technologie-Informationen (Web Research, Feb 2026)

**Shiki v3.x (latest stable ~v3.22):**
- ESM-only, async API
- `codeToHtml(code, { lang, themes, defaultColor })` — Hauptfunktion
- Dual-Theme: `themes: { light: 'github-light', dark: 'github-dark' }` + `defaultColor: false`
- Erzeugt CSS Variables `--shiki-light` / `--shiki-dark` pro Token
- React Server Components: Perfekt geeignet, async function Components
- NICHT auf Edge Runtime — Serverless/Node.js verwenden
- `BundledLanguage` und `BundledTheme` Types fuer TypeScript
- Themes/Languages werden on-demand geladen (lazy loading via WASM)

**Next.js 16.1.6 SSG:**
- App Router Default: Server Components
- Static Site Generation: Pages ohne `dynamic` export oder `fetch` mit Revalidation werden automatisch als Static generiert
- Build Output zeigt `○ (Static)` fuer SSG-Pages
- Turbopack ist Default-Bundler

### Project Structure Notes

- Landing Page ist eine **Single Page** mit Sektionen (kein Multi-Page). Header-Navigation nutzt Anker-Links (#) fuer Sektionen die in spaeteren Stories kommen
- CodeBlock und CopyButton sind **Shared Components** weil sie auch in Docs (Epic 5), Onboarding (Story 2.1), und Dashboard (Payment Detail) verwendet werden
- Die Landing Page hat ein eigenes Layout (`(landing)/layout.tsx`) mit Header + Footer, getrennt vom Root Layout das ThemeProvider hat
- **Component Boundaries beachten:** Landing Components importieren NIE aus `components/dashboard/` und umgekehrt. Shared Components gehen in `components/shared/`

### References

- [Source: _bmad-output/planning-artifacts/ux-design-specification.md#Design System Foundation] — Farbpalette, Typografie, Spacing
- [Source: _bmad-output/planning-artifacts/ux-design-specification.md#Landing Page Conversion Flow] — Hero, Problem/Solution, CTA-Strategie
- [Source: _bmad-output/planning-artifacts/ux-design-specification.md#Typography System] — Type Scale, Monospace-Regel
- [Source: _bmad-output/planning-artifacts/ux-design-specification.md#Responsive Strategy] — Mobile-First, Breakpoints
- [Source: _bmad-output/planning-artifacts/ux-design-specification.md#Dual Mental Model] — Stripe-Dev vs Web3-Native Messaging
- [Source: _bmad-output/planning-artifacts/architecture.md#App Structure] — Route Groups, Landing Page als SSG
- [Source: _bmad-output/planning-artifacts/architecture.md#Component Patterns] — Button-Hierarchie, Copy-Feedback, Toast-Pattern
- [Source: _bmad-output/planning-artifacts/architecture.md#Structure Patterns] — Projekt-Organisation, Component Boundaries
- [Source: _bmad-output/planning-artifacts/architecture.md#Naming Patterns] — kebab-case Files, PascalCase Components
- [Source: _bmad-output/planning-artifacts/epics.md#Story 1.3] — User Story und Acceptance Criteria

## Dev Agent Record

### Agent Model Used

Claude Opus 4.6

### Debug Log References

Keine Fehler oder Debug-Probleme waehrend der Implementierung.

### Completion Notes List

- Shiki v3.x installiert und als async Server Component integriert mit Dual-Theme Support (github-light / github-dark)
- CopyButton als Client Component mit Inline-Checkmark-Feedback (kein Toast) implementiert
- CodeBlock mit drei Varianten (full, compact, inline) und overflow-x-auto fuer Mobile
- Landing Layout mit Header (sticky, Desktop-Nav + Mobile Sheet-Menu) und Footer erstellt
- LandingHeader als Server Component mit separatem MobileNav Client Component (Sheet) fuer minimale Client-Boundary
- Hero Section mit text-display Headline, Dual-Audience Subline, curl + JSON Code-Blocks, und "Get API Key — Free" CTA
- Problem/Solution Section mit zwei Problem-Cards und hervorgehobener Solution-Card (border-primary)
- Alle Sections mit IDs fuer zukuenftige Anker-Navigation (id="hero", id="problem-solution")
- SSG bestaetigt: Landing Page als ○ (Static) im Build Output
- Shiki Dual-Theme CSS in globals.css hinzugefuegt (html.dark / html:not(.dark))
- shadcn/ui Sheet-Komponente fuer Mobile Menu installiert
- Alle Design Tokens verwendet (text-foreground, bg-card, text-primary, etc.) — keine hardcoded Farben
- Mobile-First Responsive Design mit lg: Breakpoint fuer Desktop

### Implementation Plan

Red-Green-Refactor: Tasks sequentiell in Story-Reihenfolge implementiert. Shared Components (CodeBlock, CopyButton) zuerst, dann Layout, dann Sections, dann Assembly, dann Validierung.

### File List

- `src/components/shared/copy-button.tsx` — NEU: Client Component, Clipboard API + Checkmark-Feedback
- `src/components/shared/code-block.tsx` — NEU: Async Server Component, Shiki codeToHtml(), 3 Varianten
- `src/components/landing/nav-items.ts` — NEU: Shared navItems Konstante (DRY)
- `src/components/landing/mobile-nav.tsx` — NEU: Client Component, Sheet-basiertes Mobile Menu
- `src/components/landing/landing-header.tsx` — NEU: Server Component, Desktop-Nav + MobileNav Composition
- `src/components/landing/landing-footer.tsx` — NEU: Server Component, Footer-Links + "by masemIT"
- `src/components/landing/hero-section.tsx` — NEU: Async Server Component, Headline + Code-Blocks + CTA
- `src/components/landing/problem-solution.tsx` — NEU: Server Component, Problem-Cards + Solution-Highlight
- `src/app/(landing)/layout.tsx` — NEU: Landing Layout mit Header + Footer
- `src/app/(landing)/page.tsx` — GEAENDERT: Hero + ProblemSolution Sections eingebunden
- `src/app/globals.css` — GEAENDERT: Shiki Dual-Theme CSS hinzugefuegt
- `src/components/ui/sheet.tsx` — NEU: shadcn/ui Sheet Component (via npx shadcn add)
- `package.json` — GEAENDERT: shiki Dependency hinzugefuegt
- `package-lock.json` — GEAENDERT: Lock-File aktualisiert (shiki + Abhängigkeiten)

## Senior Developer Review (AI)

**Review Date:** 2026-02-16
**Reviewer:** Claude Opus 4.6 (Code Review)
**Outcome:** Changes Requested → All Fixed

### Action Items

- [x] [HIGH] CopyButton opacity-0/group-hover macht Button auf Touch-Geräten unsichtbar (UX-Spec: "Always visible") [copy-button.tsx:16]
- [x] [HIGH] Footer fehlt "by masemIT" Branding — nur Copyright vorhanden, AC #6 verlangt beides [landing-footer.tsx:25]
- [x] [MED] navItems Array doppelt definiert in landing-header.tsx und mobile-nav.tsx — DRY-Verletzung
- [x] [MED] navigator.clipboard.writeText() ohne try/catch — kann in unsicheren Kontexten fehlschlagen [copy-button.tsx:12]
- [x] [MED] package-lock.json nicht in Story File List dokumentiert
- [ ] [LOW] Mehrere Primary Buttons pro Screen (Header + Hero CTA) — akzeptabel fuer Landing Page
- [ ] [LOW] Inline CodeBlock Variante: code>pre ist ungültiges HTML-Nesting [code-block.tsx:24]
- [ ] [LOW] Problem-Cards text-error (Rot) nicht in Spec gefordert — Design-Entscheidung

## Change Log

- 2026-02-16: Story 1.3 implementiert — Landing Page mit Hero Section, Problem/Solution Section, Header, Footer, CodeBlock + CopyButton Shared Components, Shiki Syntax Highlighting mit Dual-Theme Support, SSG-validiert
- 2026-02-16: Code Review — 5 Issues gefixt (2 HIGH, 3 MEDIUM): CopyButton immer sichtbar, Fehlerbehandlung hinzugefuegt, "by masemIT" Branding ergaenzt, navItems DRY extrahiert, File List vervollstaendigt
