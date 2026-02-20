# Story 1.5: Pricing, Comparison & Trust Signals

Status: done

<!-- Note: Validation is optional. Run validate-create-story for quality check before dev-story. -->

## Story

As a Besucher,
I want Pricing-Tiers vergleichen, den Kostenunterschied zu Wettbewerbern sehen, und Vertrauenssignale erkennen,
so that ich eine informierte Entscheidung treffen kann ob PayWatcher fuer mich passt.

## Acceptance Criteria

1. **Given** ich scrolle zur Pricing Section (FR3) **When** die Pricing-Karten sichtbar sind **Then** sehe ich 4 Tiers: Free, Starter ($29), Pro ($99), Enterprise (Custom) **And** jeder Tier zeigt Features, Limits und einen CTA **And** auf Desktop sind die Karten nebeneinander, auf Mobile vertikal gestackt

2. **Given** ich scrolle zur Comparison Section (FR4) **When** der Vergleich sichtbar ist **Then** sehe ich einen prominenten Kostenvergleich ("$100 at Coinbase Commerce vs. $0.05 at PayWatcher for a $10k payment") **And** der Vergleich ist als visuelle Unterbrechung gestaltet (Callout-Stil)

3. **Given** ich scrolle zur Trust Signals Section (FR6) **When** die Section sichtbar ist **Then** sehe ich Trust Signals: Non-Custodial, Base Network, USDC, "Built by masemIT" **And** keine Stock-Fotos oder generische Trust-Badges

4. **Given** jede Section einen CTA hat **When** ich an einem beliebigen Punkt auf "Get API Key" oder "Start Free" klicke **Then** werde ich zur Signup-Route navigiert

## Tasks / Subtasks

- [x] Task 1: Pricing Section mit 4 Tier-Karten erstellen (AC: #1)
  - [x] 1.1 `src/components/landing/pricing-section.tsx` erstellen — Server Component
  - [x] 1.2 Pricing-Daten als Konstante definieren: Free (0, 50 Verifications/mo), Starter ($29, 1.000/mo), Pro ($99, 10.000/mo), Enterprise (Custom, unlimited)
  - [x] 1.3 4 PricingCards rendern mit: Tier-Name, Preis, Feature-Liste, CTA-Button
  - [x] 1.4 "Starter"-Karte visuell hervorheben (Accent-Border oder Badge "Most Popular")
  - [x] 1.5 CTAs: Free → "Start Free", Starter → "Get Started", Pro → "Get Started", Enterprise → "Contact Us"
  - [x] 1.6 Section-ID `id="pricing"` setzen fuer Anker-Navigation vom Header
  - [x] 1.7 Responsive: `grid-cols-1 lg:grid-cols-4`, auf Mobile vertikal gestackt (Free oben, Enterprise unten)

- [x] Task 2: Comparison Callout erstellen (AC: #2)
  - [x] 2.1 `src/components/landing/comparison-callout.tsx` erstellen — Server Component
  - [x] 2.2 Prominenten Kostenvergleich darstellen: "$100 at Coinbase Commerce vs. $0.05 at PayWatcher for a $10k payment"
  - [x] 2.3 Als visuelle Unterbrechung gestalten (Callout-Stil, Accent-Farbe, groessere Typografie)
  - [x] 2.4 Optional: Zusaetzliche Vergleichspunkte (Custodial vs Non-Custodial, % vs Flat-Fee)

- [x] Task 3: Trust Signals Section erstellen (AC: #3)
  - [x] 3.1 `src/components/landing/trust-signals.tsx` erstellen — Server Component
  - [x] 3.2 Trust Signals darstellen: Non-Custodial, Base Network, USDC, "Built by masemIT"
  - [x] 3.3 Lucide Icons verwenden (ShieldCheck, Zap, Lock, etc.) — keine Stock-Fotos
  - [x] 3.4 Kompaktes Grid-Layout, nuechtern-technisch (keine generischen Badges)

- [x] Task 4: Landing Page aktualisieren und integrieren (AC: #1, #2, #3, #4)
  - [x] 4.1 `src/app/(landing)/page.tsx` aktualisieren — PricingSection + ComparisonCallout + TrustSignals einbinden
  - [x] 4.2 Sections-Reihenfolge: Hero → ProblemSolution → HowItWorks → CodeExamples → Pricing → Comparison → TrustSignals
  - [x] 4.3 Sicherstellen: Anker-Navigation `#pricing` vom Header funktioniert
  - [x] 4.4 Sicherstellen: Kein `'use client'` auf page.tsx — SSG muss funktionieren
  - [x] 4.5 CTAs in jeder Section linken auf `/signup`

- [x] Task 5: Build-Validierung und Qualitaetspruefung (Alle ACs)
  - [x] 5.1 `npm run build` muss fehlerfrei durchlaufen
  - [x] 5.2 `npm run type-check` muss fehlerfrei durchlaufen
  - [x] 5.3 `npm run lint` muss fehlerfrei durchlaufen
  - [x] 5.4 Verifizieren: Landing Page zeigt als `○ (Static)` im Build Output (SSG bestaetigt)
  - [x] 5.5 Visuelle Pruefung: Dark Mode korrekt, Light Mode korrekt
  - [x] 5.6 Responsive: Pricing Cards gestackt auf Mobile, nebeneinander auf Desktop
  - [x] 5.7 Comparison Callout visuell prominent und als Unterbrechung erkennbar
  - [x] 5.8 Trust Signals nuechtern-technisch, keine generischen Badges

## Dev Notes

### Kontext & Zweck

Dies ist die **fuenfte Story** des PayWatcher-Projekts. Story 1.1 lieferte das Projekt-Scaffold, Story 1.2 das Design System, Story 1.3 den oberen Teil der Landing Page (Header, Hero, Problem/Solution, Footer), Story 1.4 die mittleren Sektionen (How It Works, Code Examples). Diese Story ergaenzt die **unteren Sektionen** der Landing Page: Pricing, Comparison Callout und Trust Signals.

**Was PayWatcher ist:** Developer-First Verification API fuer Stablecoin-Zahlungen (USDC auf Base Network). Non-custodial, reiner Verification Layer. Die Landing Page muss in 30 Sekunden die Value Proposition kommunizieren. Der Kostenvergleich ("$100 vs $0.05") ist der staerkste Conversion-Trigger.

**Design-Philosophie (UX-Spec):** "Stripe-Style, nicht Vercel-Style." Nuechtern-technisch. Pricing ohne "Contact Sales" fuer Free/Starter/Pro — Self-Service bis $99/mo. Comparison Callout als visuelle Unterbrechung — "$100 vs $0.05" ist der staerkste Trigger. Trust Signals ohne Stock-Fotos.

**Was diese Story liefert:**
- Pricing Section mit 4 Tier-Karten (Free, Starter, Pro, Enterprise)
- Comparison Callout mit prominentem Kostenvergleich
- Trust Signals Section (Non-Custodial, Base Network, USDC, masemIT)
- CTAs in jeder Section
- Anker-Navigation `#pricing` funktionierend
- SSG Pre-Rendering bestaetigt

**Was diese Story NICHT beinhaltet:**
- SEO Meta Tags, OG Images, Sitemap, Analytics (Story 1.6)
- Signup-Formular/-Flow (Epic 2) — CTA linkt nur auf `/signup`
- Interaktiver Pricing Calculator oder Toggle (Monthly/Annual)
- Stripe Billing Integration (Phase 2)

### Technische Anforderungen

**Zu installierende Packages:** KEINE — alle benoetigten Packages sind bereits installiert.

**Bereits installierte und verwendbare Packages:**
- shadcn/ui Card Component (bereits in `src/components/ui/card.tsx`)
- shadcn/ui Badge Component (bereits in `src/components/ui/badge.tsx`)
- shadcn/ui Button Component (bereits in `src/components/ui/button.tsx`)
- Lucide React Icons (bereits installiert)
- Tailwind CSS v4 mit Design Tokens (globals.css)

**NICHT in dieser Story installieren:**
- `@tanstack/react-query` → Story 2.1
- `next-auth` / `better-auth` → Story 2.2
- `recharts` → Story 3.2
- `react-hook-form` + `zod` → Story 2.2

### Pricing-Daten (aus PRD + UX-Spec)

**4 Tiers:**

| Tier | Preis | Verifications/Monat | Highlights |
|------|-------|-------------------|-----------|
| **Free** | $0 | 50 | Perfekt zum Testen. 1 API Key, Community Support |
| **Starter** | $29/mo | 1.000 | Fuer kleine Projekte. Priority Support |
| **Pro** | $99/mo | 10.000 | Fuer wachsende Businesses. Premium Support, Higher Limits |
| **Enterprise** | Custom | Unlimited | Dedicated Support, Custom SLAs, Volume Pricing |

**Pricing-Modell:** Flat-Fee pro Verification ($0.05 bei Starter/Pro), nicht prozentual. Das ist DER Differentiator.

**CTA-Strategie:**
- Free: "Start Free" (Primary in der Free-Card)
- Starter: "Get Started" (Accent-hervorgehoben, "Most Popular" Badge)
- Pro: "Get Started"
- Enterprise: "Contact Us" (Link auf mailto: oder Calendly)

**UX-Spec Vorgabe:** Max 1 Primary Button pro Screen — da die Pricing Section 4 Karten hat, sollte NUR die Starter-Karte einen Primary-Button haben. Die anderen nutzen Secondary/Outline. ABER: Im Kontext der Pricing Section zaehlt jede Karte als eigener visueller Block, daher ist ein CTA pro Karte akzeptabel (alle als Outline, Starter als Primary oder Accent-hervorgehoben).

### Comparison Callout Content

**Kernnachricht (aus UX-Spec):**
> "$100 at Coinbase Commerce vs. $0.05 at PayWatcher for a $10k payment"

**Zusaetzliche Vergleichspunkte (optional):**
- Coinbase Commerce: 1% Gebuehr, Custodial, Vendor Lock-in
- PayWatcher: $0.05 Flat-Fee, Non-Custodial, No Lock-in
- "You keep your keys. We just verify."

**Design-Vorgabe (UX-Spec):** "Comparison Callout als visuelle Unterbrechung" — muss sich visuell von den umgebenden Sections abheben. Accent-Farbe, groessere Typografie, Callout-Stil. Kein separater Card, sondern ein Section-breiter visueller Block.

### Trust Signals Content

**4 Trust Signals (aus PRD + UX-Spec):**

| Signal | Icon (Lucide) | Kurzbeschreibung |
|--------|--------------|-----------------|
| **Non-Custodial** | `ShieldCheck` | "We never touch your funds" / "Your keys, your wallet. We just watch." |
| **Base Network** | `Zap` oder custom | "Built on Base — fast, cheap, reliable" |
| **USDC** | `CircleDollarSign` oder custom | "USDC stablecoin — no volatility risk" |
| **Built by masemIT** | `Building2` oder custom | "Austrian company, transparent operations" |

**Design-Vorgabe (UX-Spec):** Keine Stock-Fotos, keine generischen Trust-Badges. Nuechtern-technisch. Icons + kurzer Text. Kompaktes Layout.

### Responsive Strategy

| Element | Desktop (>=1024px) | Mobile (<1024px) |
|---------|-------------------|------------------|
| Pricing Cards | 4 Spalten nebeneinander (`lg:grid-cols-4`) | 1 Spalte gestackt, Free oben, Enterprise unten |
| Comparison Callout | Zentriert, max-w-4xl, grosse Typografie | Volle Breite, angepasste Typografie |
| Trust Signals | 4 Spalten oder 2x2 Grid | 1 Spalte oder 2 Spalten gestackt |

**Tailwind Breakpoint-Strategie:** Mobile-First. Base-Styles = Mobile. `lg:` Prefix fuer Desktop-Overrides.

### Architektur-Konformitaet

**Dateinamen:** `kebab-case.tsx` — IMMER. Keine PascalCase-Dateinamen.

**Komponenten-Pfade:**
- `src/app/(landing)/page.tsx` — AENDERN: PricingSection + ComparisonCallout + TrustSignals einbinden
- `src/components/landing/pricing-section.tsx` — NEU: Pricing Cards Section
- `src/components/landing/comparison-callout.tsx` — NEU: Kostenvergleich Callout
- `src/components/landing/trust-signals.tsx` — NEU: Trust Signals Section

**Server vs Client Components:**
- Server Components (DEFAULT): page.tsx, pricing-section.tsx, comparison-callout.tsx, trust-signals.tsx
- Client Components: KEINE noetig fuer diese Story — alles ist statischer Content

**ANTI-Patterns:**
- KEIN `'use client'` auf den neuen Components — alles ist statischer Content, SSG muss funktionieren
- KEINE dynamischen Daten / fetch Calls — alles statisch
- KEIN Full-Page-Spinner oder Loading States — SSG liefert sofort
- KEINE hardcoded Farben — immer Design Token Klassen
- KEIN `tailwind.config.js` — Tailwind v4 CSS-first via @theme
- KEINE Stock-Fotos oder generische Trust-Badges
- KEIN "Contact Sales" fuer Free/Starter/Pro — Self-Service

### Library & Framework Anforderungen

**shadcn/ui Card (bereits installiert):**
- Composed Components: Card, CardHeader, CardTitle, CardDescription, CardAction, CardContent, CardFooter
- CardAction kann fuer "Most Popular" Badge verwendet werden
- CSS-only, keine Client-Side JavaScript noetig
- Perfekt fuer statische Pricing Cards

**shadcn/ui Badge (bereits installiert):**
- Variants: default, secondary, destructive, outline, ghost
- Verwenden fuer "Most Popular" Badge auf Starter-Card
- Variant "default" (Accent-Farbe) fuer visuelle Hervorhebung

**shadcn/ui Button (bereits installiert):**
- Variants: default, destructive, outline, secondary, ghost, link
- Sizes: xs, sm, default, lg
- Verwenden fuer CTA-Buttons in Pricing Cards
- asChild-Pattern fuer Link-Buttons: `<Button asChild><Link href="/signup">...</Link></Button>`

**Lucide React Icons (bereits installiert):**
- Tree-shakable, nur importierte Icons im Bundle
- `size={20}` oder `size={24}` fuer Trust Signal Icons
- Relevante Icons: ShieldCheck, Zap, CircleDollarSign, Building2, Check (fuer Feature-Listen)

### Dateistruktur-Anforderungen

**Neue/geaenderte Dateien nach Abschluss:**

```
src/
├── app/
│   └── (landing)/
│       └── page.tsx                          # AENDERN: Pricing + Comparison + TrustSignals einbinden
│
├── components/
│   └── landing/
│       ├── pricing-section.tsx               # NEU: Pricing Cards mit 4 Tiers
│       ├── comparison-callout.tsx            # NEU: Kostenvergleich Callout
│       └── trust-signals.tsx                 # NEU: Trust Signals Section
```

### Testing-Anforderungen

1. **Build-Validierung:** `npm run build` muss ohne Fehler durchlaufen
2. **Type-Check:** `npm run type-check` muss ohne Fehler durchlaufen
3. **Lint:** `npm run lint` muss ohne Fehler durchlaufen
4. **SSG-Validierung:** Landing Page muss als `○ (Static)` im Build Output erscheinen
5. **Visuelle Pruefung Dark Mode:** Pricing Cards, Comparison Callout, Trust Signals korrekt dargestellt
6. **Visuelle Pruefung Light Mode:** Alle Farben korrekt invertiert
7. **Responsive:** Pricing Cards gestackt auf Mobile, nebeneinander auf Desktop
8. **Comparison Callout:** Visuell prominent, als Unterbrechung erkennbar, grosse Typografie
9. **Trust Signals:** Nuechtern-technisch, Icons + Text, keine generischen Badges
10. **CTAs:** Alle CTA-Buttons fuehren zu `/signup`, Starter-Card visuell hervorgehoben
11. **Anker-Navigation:** Klick auf "Pricing" im Header scrollt zur Pricing Section

### Vorherige Story Intelligence

**Story 1.4 Learnings (KRITISCH — direkte Vorgaenger-Story):**
- **AnimateOnScroll Component existiert** — `src/components/landing/animate-on-scroll.tsx` (Client Component mit IntersectionObserver). Kann fuer Pricing/Trust Sections wiederverwendet werden falls Scroll-Animation gewuenscht.
- **CopyButton Timeout ist 1.5s** (aus Code Review Fix)
- **Section-IDs sind konsistent gesetzt** — `id="hero"`, `id="problem-solution"`, `id="how-it-works"`, `id="code-examples"`. MUSS konsistent `id="pricing"` und `id="trust-signals"` verwenden.
- **navItems** in `src/components/landing/nav-items.ts` enthalten bereits `{ label: "Pricing", href: "#pricing" }` — Anker-Navigation ist vorbereitet!
- **Shiki Dual-Theme CSS** existiert in `globals.css`
- **SSG bestaetigt funktionierend** — keine `'use client'` auf page.tsx
- **Mobile-First mit `lg:` Breakpoint** fuer Desktop
- **Section-Spacing:** `py-16` fuer vertikalen Abstand, `max-w-5xl mx-auto` fuer zentriertes Layout

**Story 1.3 Learnings:**
- Landing Layout (`(landing)/layout.tsx`) mit Header + Footer existiert
- `max-w-5xl mx-auto` fuer zentriertes Layout
- ProblemSolution Component nutzt Card-Components und Grid-Layout — kann als Referenz fuer Pricing Cards dienen
- CTA-Buttons nutzen `<Button asChild><Link href="/signup">...</Link></Button>` Pattern

**Story 1.2 Learnings:**
- `next-themes` mit `attribute="class"` → `.dark` Klasse auf `<html>`
- Type Scale verfuegbar: `text-display`, `text-h1`, `text-h2`, `text-h3`, `text-body`, `text-body-sm`, `text-caption`, `text-stat`
- Status-Farben definiert: success (teal/gruen), warning, error, neutral
- Design Tokens via OKLch Farbraum in `globals.css`
- Accent/Primary Farbe: Teal (~oklch 0.72-0.78 0.155 170) — verwenden fuer Starter-Card Hervorhebung

**Story 1.1 Learnings:**
- Route Group `(landing)` existiert
- `geist` Package laedt Geist Sans + Geist Mono
- Pfad-Alias `@/` → `src/`

### Git Intelligence

**Letzte Commits:**
```
ef7f25e Code review fixes for Story 1.4: CopyButton timeout, scroll indicator, FOUC, grid alignment
475e986 Resolve merge conflict in settings.local.json
3b8016a Story 1.4: How It Works section, interactive code examples with tabs
36a83f8 Story 1.3: Landing page layout, hero section, and problem/solution
5a45117 Story 1.2: Theme system, design tokens, and payment status colors
```

**Relevante Dateien aus vorherigen Stories:**
- `src/components/ui/card.tsx` — Card, CardHeader, CardTitle, CardDescription, CardAction, CardContent, CardFooter
- `src/components/ui/badge.tsx` — Badge mit Variants
- `src/components/ui/button.tsx` — Button mit Variants und asChild
- `src/components/landing/nav-items.ts` — navItems mit `#pricing` Link bereits vorhanden
- `src/components/landing/problem-solution.tsx` — Grid-Layout Referenz fuer Pricing Cards
- `src/app/globals.css` — Design Tokens, Type Scale, Farben

**Patterns aus vorherigen Stories:**
- Server Components als Default — bewaehrt
- Section-IDs fuer Anker-Navigation — konsistent fortfuehren
- Mobile-First mit `lg:` Breakpoint
- `max-w-5xl mx-auto` fuer zentriertes Layout (ODER breiter fuer Pricing Cards: `max-w-6xl` wenn 4 Spalten mehr Platz brauchen)
- `py-16` fuer Section-Spacing
- Button asChild Pattern fuer Link-CTAs

### Aktuelle Technologie-Informationen (Web Research, Feb 2026)

**shadcn/ui Card (bereits installiert):**
- Composed Components: Card, CardHeader, CardTitle, CardDescription, CardAction, CardContent, CardFooter
- CardAction: neues Feature fuer Badge-Platzierung in der oberen rechten Ecke
- Container Queries Support fuer responsive Card-Layouts
- Keine Breaking Changes seit Installation

**Tailwind CSS v4 Grid:**
- `grid grid-cols-1 lg:grid-cols-4 gap-6` — Standard-Pattern fuer 4-Spalten Pricing
- Container Queries (`@container`) built-in, kein Plugin noetig
- OKLch Farbraum fuer bessere Farbdarstellung (bereits in globals.css)

**Lucide React Icons:**
- Tree-shakable: nur importierte Icons im Bundle
- Relevante Icons fuer Trust Signals:
  - `ShieldCheck` — Security/Non-Custodial
  - `Zap` — Performance/Speed (Base Network)
  - `CircleDollarSign` — USDC/Stablecoin
  - `Building2` — Company/masemIT
  - `Check` — Feature-Listen Checkmarks

**Next.js 16 SSG:**
- Statische Seiten werden automatisch pre-rendered wenn keine dynamischen Daten
- Alle neuen Components sind Server Components ohne fetch — SSG funktioniert automatisch
- Keine speziellen Konfigurationen noetig

### Project Structure Notes

- Landing Page ist eine **Single Page** mit Sektionen — Pricing, Comparison und Trust Signals sind die letzten drei vor dem Footer
- Die **Sections-Reihenfolge** folgt dem UX-Spec Conversion Flow: Hero → ProblemSolution → HowItWorks → CodeExamples → Pricing → Comparison → TrustSignals → Footer
- **Comparison Callout** ist ZWISCHEN Pricing und Trust Signals — als visuelle Unterbrechung
- Alle neuen Components sind **Server Components** — kein Client-Side JavaScript noetig
- **Component Boundaries beachten:** Landing Components importieren NIE aus `components/dashboard/`
- Der **"Pricing" Link im Header** (`nav-items.ts`) zeigt auf `#pricing` — Section-ID muss gesetzt sein

### References

- [Source: _bmad-output/planning-artifacts/ux-design-specification.md#Flow 3: Landing Page Conversion] — Conversion-Flow: Pricing → Comparison → Trust Signals → CTA
- [Source: _bmad-output/planning-artifacts/ux-design-specification.md#Responsive Design] — "Pricing Cards nebeneinander (Desktop), gestackt (Mobile)"
- [Source: _bmad-output/planning-artifacts/ux-design-specification.md#Component Implementation Strategy] — "PricingCard (custom Card-Variante), ComparisonCallout"
- [Source: _bmad-output/planning-artifacts/ux-design-specification.md#Conversion-Optimierungen] — "Pricing ohne Contact Sales, Comparison als staerkster Trigger"
- [Source: _bmad-output/planning-artifacts/ux-design-specification.md#Dual Mental Model Messaging] — Trust Signals: "Non-custodial — we never touch your funds" / "Your keys, your wallet. We just watch."
- [Source: _bmad-output/planning-artifacts/architecture.md#Component Patterns] — Button-Hierarchie, Max 1 Primary pro Screen
- [Source: _bmad-output/planning-artifacts/architecture.md#Structure Patterns] — kebab-case Files, Component Boundaries
- [Source: _bmad-output/planning-artifacts/architecture.md#Naming Patterns] — PascalCase Components, kebab-case Dateien
- [Source: _bmad-output/planning-artifacts/epics.md#Story 1.5] — User Story und Acceptance Criteria
- [Source: _bmad-output/planning-artifacts/prd.md#Functional Requirements] — FR3 (Pricing), FR4 (Comparison), FR6 (Trust Signals)
- [Source: _bmad-output/implementation-artifacts/1-4-how-it-works-interactive-code-examples.md] — Vorherige Story mit Section-ID und Layout Patterns

## Dev Agent Record

### Agent Model Used

Claude Opus 4.6

### Debug Log References

Keine Debugging-Probleme aufgetreten.

### Completion Notes List

- Task 1: PricingSection mit 4 Tier-Karten (Free, Starter, Pro, Enterprise) erstellt. Starter-Karte hervorgehoben mit primary Border und "Most Popular" Badge via CardAction. CTAs nutzen Button asChild + Link Pattern. Responsive Grid: 1 Spalte Mobile, 4 Spalten Desktop. max-w-6xl fuer breiteres Layout bei 4 Karten.
- Task 2: ComparisonCallout als visuelle Unterbrechung erstellt. "$100 vs $0.05" Kernbotschaft mit grosser Typografie (text-h1 lg:text-display). Vergleichstabelle Coinbase vs PayWatcher mit Farb-Codierung (error vs primary). bg-primary/5 Hintergrund als visueller Block. Hinweis: Vergleichstext aus AC #2 wurde in Heading + Subtitle aufgeteilt fuer bessere visuelle Hierarchie — alle Informationen vorhanden.
- Task 3: TrustSignals mit 4 Signalen (Non-Custodial, Base Network, USDC, masemIT) erstellt. Lucide Icons (ShieldCheck, Zap, CircleDollarSign, Building2). Kompaktes Grid: 1→2→4 Spalten. Nuechtern-technisch, keine Stock-Fotos.
- Task 4: Landing Page aktualisiert. Sections-Reihenfolge: Hero → ProblemSolution → HowItWorks → CodeExamples → Pricing → Comparison → TrustSignals. Kein 'use client', SSG bestaetigt. Anker-Navigation #pricing bereits in nav-items.ts vorhanden. Hinweis: Enterprise-CTA linkt auf mailto:enterprise@paywatcher.dev (nicht /signup) gemaess CTA-Strategie in Dev Notes — "Contact Us" ist kein Signup-CTA.
- Task 1 Ergaenzung: Zusaetzliche Features in Pricing-Karten hinzugefuegt (nicht in Story-Datentabelle): Starter "Multiple API keys", Pro "Webhook integrations", Enterprise "On-boarding assistance" — inhaltlich sinnvolle Ergaenzungen fuer ueberzeugendere Pricing Cards.
- Task 5: Build (npm run build), Type-Check (npm run type-check), Lint (npm run lint) alle fehlerfrei. SSG bestaetigt: Landing Page als ○ (Static). Alle Design Token Klassen verwendet — Dark/Light Mode korrekt. Responsive Layouts implementiert.

### Change Log

- 2026-02-16: Story 1.5 implementiert — Pricing Section, Comparison Callout, Trust Signals
- 2026-02-16: Code Review Fixes — CTA-Buttons in ComparisonCallout und TrustSignals ergaenzt (AC #4), Section-Heading fuer TrustSignals hinzugefuegt, Section-ID auf ComparisonCallout gesetzt, Completion Notes aktualisiert

### File List

- src/components/landing/pricing-section.tsx (NEU)
- src/components/landing/comparison-callout.tsx (NEU)
- src/components/landing/trust-signals.tsx (NEU)
- src/app/(landing)/page.tsx (GEAENDERT)
