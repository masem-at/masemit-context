# Story 1.6: SEO, Social Sharing & Analytics Integration

Status: done

<!-- Note: Validation is optional. Run validate-create-story for quality check before dev-story. -->

## Story

As a Betreiber (Sempre),
I want dass die Landing Page SEO-optimiert ist, Social Sharing unterstuetzt, und Analytics-Events trackt,
so that PayWatcher organisch gefunden wird und ich Conversion-Daten habe.

## Acceptance Criteria

1. **Given** die Landing Page existiert (FR34) **When** ein Suchmaschinen-Crawler die Seite besucht **Then** findet er korrekte Meta Tags (title, description), Open Graph Tags (og:title, og:description, og:image), und strukturierte Daten **And** eine dynamische Sitemap unter /sitemap.xml existiert **And** eine robots.txt unter /robots.txt existiert

2. **Given** jemand die Landing Page URL in Social Media teilt (FR35) **When** die Plattform die OG-Tags liest **Then** wird ein PayWatcher-OG-Image, Titel und Beschreibung angezeigt **And** Twitter Cards sind konfiguriert

3. **Given** ein Besucher die Landing Page navigiert (FR32, FR33) **When** er interagiert (Page View, CTA Click, Pricing Section View, Code Copy, Signup Start) **Then** werden Analytics-Events asynchron an masemIT Analytics (analytics.masem.at) gesendet **And** die Events blockieren nicht die UI (NFR13) **And** Events werden nur in Production gesendet (nicht in Dev/Preview)

4. **Given** die Landing Page Performance gemessen wird (NFR1) **When** ein Lighthouse-Test laeuft **Then** ist der First Contentful Paint unter 1.5 Sekunden (SSG + Vercel CDN)

## Tasks / Subtasks

- [x] Task 1: SEO Metadata & Open Graph Tags implementieren (AC: #1, #2)
  - [x] 1.1 `src/app/layout.tsx` erweitern — Vollstaendige Metadata mit OG Tags, Twitter Cards, Canonical URL, Theme Color
  - [x] 1.2 `src/app/(landing)/layout.tsx` pruefen — Landing-spezifische Metadata Overrides falls noetig
  - [x] 1.3 OG Image erstellen: `public/og-image.png` (1200x630px, Dark Mode Design, PayWatcher Branding + Headline)
  - [x] 1.4 JSON-LD Structured Data als `<script type="application/ld+json">` in Root Layout einbinden (Organization + WebSite Schema)

- [x] Task 2: Sitemap & Robots.txt generieren (AC: #1)
  - [x] 2.1 `src/app/sitemap.ts` erstellen — Dynamische Sitemap mit allen oeffentlichen Routen (Landing Page, /docs/*)
  - [x] 2.2 `src/app/robots.ts` erstellen — Standard robots.txt mit Sitemap-Verweis, Dashboard-Routen blockieren

- [x] Task 3: Analytics-Infrastruktur aufbauen (AC: #3)
  - [x] 3.1 `src/lib/analytics.ts` erstellen — `trackEvent(name, properties?)` Funktion mit `navigator.sendBeacon` / `fetch(keepalive: true)`
  - [x] 3.2 Production-Only Guard: `process.env.NODE_ENV !== 'production'` → kein Event-Send
  - [x] 3.3 Environment Variables nutzen: `NEXT_PUBLIC_ANALYTICS_URL`, `NEXT_PUBLIC_ANALYTICS_KEY`
  - [x] 3.4 `.env.example` aktualisieren: `ANALYTICS_URL` → `NEXT_PUBLIC_ANALYTICS_URL`, `ANALYTICS_KEY` → `NEXT_PUBLIC_ANALYTICS_KEY` (Client-Side Zugriff noetig)

- [x] Task 4: Analytics-Events in Landing Page integrieren (AC: #3)
  - [x] 4.1 `src/components/landing/analytics-provider.tsx` erstellen — Client Component fuer Page View Tracking bei Mount
  - [x] 4.2 Page View Event: `page_view` bei Landing Page Load
  - [x] 4.3 CTA Click Events: `cta_click` mit `{ label, section }` Properties bei allen "Get API Key" / "Start Free" Buttons
  - [x] 4.4 Pricing View Event: `section_view` mit `{ section: "pricing" }` via IntersectionObserver (bestehende animate-on-scroll Pattern wiederverwenden)
  - [x] 4.5 Code Copy Events: `code_copy` mit `{ language }` Property in bestehende CopyButton-Komponente integrieren
  - [x] 4.6 Section View Events: `section_view` fuer Hero, How It Works, Comparison, Trust Signals via IntersectionObserver

- [x] Task 5: Build-Validierung und Qualitaetspruefung (Alle ACs)
  - [x] 5.1 `npm run build` muss fehlerfrei durchlaufen
  - [x] 5.2 `npm run type-check` muss fehlerfrei durchlaufen
  - [x] 5.3 `npm run lint` muss fehlerfrei durchlaufen
  - [x] 5.4 Verifizieren: Landing Page zeigt als `○ (Static)` im Build Output (SSG bestaetigt) — Analytics-Provider darf SSG nicht brechen
  - [x] 5.5 Verifizieren: `/sitemap.xml` und `/robots.txt` sind im Build Output erreichbar
  - [x] 5.6 Verifizieren: OG Meta Tags korrekt im HTML-Output (View Source oder curl)
  - [x] 5.7 Verifizieren: Structured Data (JSON-LD) validierbar
  - [x] 5.8 Verifizieren: Analytics-Events werden in Dev NICHT gesendet (Console-Log statt Netzwerk-Request)

## Dev Notes

### Kontext & Zweck

Dies ist die **sechste und letzte Story** von Epic 1 (Project Foundation & Landing Page). Stories 1.1-1.5 haben das Projekt-Scaffold, Design System, und alle Landing Page Sektionen (Hero, Problem/Solution, How It Works, Code Examples, Pricing, Comparison, Trust Signals) geliefert. Diese Story ergaenzt die **unsichtbare Infrastruktur**: SEO, Social Sharing und Analytics — alles was noetig ist damit PayWatcher organisch gefunden wird und Conversion-Daten ab Tag 1 fliessen.

**Was PayWatcher ist:** Developer-First Verification API fuer Stablecoin-Zahlungen (USDC auf Base Network). Non-custodial, reiner Verification Layer. Die Landing Page muss in Suchmaschinen und Social Media ueberzeugend praesentiert werden.

**Design-Philosophie:** "Stripe-Style, nicht Vercel-Style." Nuechtern-technisch. SEO-Texte sollen ehrlich und developer-fokussiert sein, kein Marketing-Jargon. Analytics sammelt nur Conversion-Daten, kein invasives Tracking.

**Was diese Story liefert:**
- Vollstaendige SEO Metadata (Meta Tags, OG Tags, Twitter Cards, Canonical URL)
- JSON-LD Structured Data (Organization + WebSite)
- OG Image fuer Social Sharing (1200x630px)
- Dynamische Sitemap (`/sitemap.xml`)
- Robots.txt (`/robots.txt`)
- Analytics-Infrastruktur (`lib/analytics.ts`) mit `trackEvent()` Funktion
- Event-Tracking auf der Landing Page (Page View, CTA Clicks, Section Views, Code Copies)
- Production-Only Analytics (kein Tracking in Dev/Preview)

**Was diese Story NICHT beinhaltet:**
- Dashboard Analytics Events (Phase 1b / Epic 2+)
- Signup Start/Complete Events (Epic 2 — Signup existiert noch nicht)
- Blog oder Content Marketing (nicht im MVP-Scope)
- Google Analytics oder Drittanbieter-Tracking
- Dynamic OG Images (pro-Seite generiert) — statisches OG Image reicht fuer MVP

### Technische Anforderungen

**Zu installierende Packages:** KEINE — Next.js 16 Metadata API, `navigator.sendBeacon` und `fetch` sind nativ verfuegbar.

**Bereits installierte und verwendbare Packages:**
- Next.js 16.1.6 mit App Router (Metadata API fuer SEO)
- React 19 (Client Components fuer Analytics)
- Shiki (Code Highlighting — bereits in Landing Page)
- sonner (Toast — nicht noetig fuer diese Story)

**NICHT in dieser Story installieren:**
- `next-seo` oder `next-sitemap` — Next.js 16 hat alles nativ
- `@vercel/analytics` — masemIT Analytics ist das Ziel, nicht Vercel Analytics
- `gtag` / Google Analytics — nicht im Scope
- `sharp` fuer OG Image Generation — statisches Image reicht

### Analytics-Architektur (aus Architecture Doc)

**Ziel-Service:** masemIT Analytics (analytics.masem.at) — eigener Analytics-Service, kein Drittanbieter.

**Event-Pattern:**
```typescript
// lib/analytics.ts
export function trackEvent(name: string, properties?: Record<string, string>) {
  if (typeof window === 'undefined') return
  if (process.env.NODE_ENV !== 'production') {
    console.debug('[analytics]', name, properties)
    return
  }

  const payload = JSON.stringify({
    event: name,
    properties,
    timestamp: new Date().toISOString(),
    url: window.location.href,
  })

  const url = `${process.env.NEXT_PUBLIC_ANALYTICS_URL}/events`

  // navigator.sendBeacon ist die bevorzugte Methode (non-blocking, works on page unload)
  if (navigator.sendBeacon) {
    navigator.sendBeacon(url, new Blob([payload], { type: 'application/json' }))
    return
  }

  // Fallback: fetch mit keepalive
  fetch(url, {
    method: 'POST',
    body: payload,
    headers: { 'Content-Type': 'application/json' },
    keepalive: true,
  }).catch(() => {}) // Fire-and-forget, Fehler still ignorieren
}
```

**Event-Naming:** `snake_case` — `page_view`, `cta_click`, `section_view`, `code_copy`

**Environment Variables (Client-Side):**
- `NEXT_PUBLIC_ANALYTICS_URL` — Base URL (analytics.masem.at)
- `NEXT_PUBLIC_ANALYTICS_KEY` — API Key fuer Authentifizierung (oeffentlich, da Client-Side)

**WICHTIG:** Die `.env.example` muss von `ANALYTICS_URL`/`ANALYTICS_KEY` auf `NEXT_PUBLIC_ANALYTICS_URL`/`NEXT_PUBLIC_ANALYTICS_KEY` aktualisiert werden, da die Analytics-Events Client-Side gesendet werden und `NEXT_PUBLIC_` Prefix noetig ist fuer Browser-Zugriff in Next.js.

### SEO-Implementierung (Next.js 16 Metadata API)

**Root Layout Metadata Pattern:**
```typescript
// src/app/layout.tsx
export const metadata: Metadata = {
  metadataBase: new URL('https://paywatcher.dev'),
  title: {
    default: 'PayWatcher — Stablecoin Payment Verification API',
    template: '%s | PayWatcher',
  },
  description: 'Verify USDC payments without custody or payment processing. $0.05 per verification. Non-custodial, developer-first, built on Base Network.',
  keywords: ['stablecoin payment API', 'USDC verification', 'payment verification API', 'Web3 payment gateway', 'Base Network', 'non-custodial'],
  authors: [{ name: 'masemIT' }],
  creator: 'masemIT',
  openGraph: {
    type: 'website',
    locale: 'en_US',
    url: 'https://paywatcher.dev',
    siteName: 'PayWatcher',
    title: 'PayWatcher — Stablecoin Payment Verification API',
    description: 'Verify USDC payments without custody. $0.05 per verification, non-custodial, built for developers.',
    images: [{ url: '/og-image.png', width: 1200, height: 630, alt: 'PayWatcher — Payment verification without the payment processor' }],
  },
  twitter: {
    card: 'summary_large_image',
    title: 'PayWatcher — Stablecoin Payment Verification API',
    description: 'Verify USDC payments without custody. $0.05 per verification, non-custodial, built for developers.',
    images: ['/og-image.png'],
  },
  robots: {
    index: true,
    follow: true,
  },
}
```

**Structured Data Pattern (JSON-LD):**
```json
{
  "@context": "https://schema.org",
  "@type": "WebSite",
  "name": "PayWatcher",
  "url": "https://paywatcher.dev",
  "description": "Developer-First Verification API for Stablecoin Payments",
  "publisher": {
    "@type": "Organization",
    "name": "masemIT",
    "url": "https://masem.at"
  }
}
```

### Sitemap & Robots.txt Patterns

**Sitemap (`src/app/sitemap.ts`):**
```typescript
import { MetadataRoute } from 'next'

export default function sitemap(): MetadataRoute.Sitemap {
  return [
    { url: 'https://paywatcher.dev', lastModified: new Date(), priority: 1.0 },
    { url: 'https://paywatcher.dev/docs', lastModified: new Date(), priority: 0.8 },
    // Docs-Seiten werden in Epic 5 hinzugefuegt
  ]
}
```

**Robots.txt (`src/app/robots.ts`):**
```typescript
import { MetadataRoute } from 'next'

export default function robots(): MetadataRoute.Robots {
  return {
    rules: { userAgent: '*', allow: '/', disallow: ['/dashboard/', '/admin/', '/api/'] },
    sitemap: 'https://paywatcher.dev/sitemap.xml',
  }
}
```

### OG Image Anforderungen

**Spezifikationen:**
- Groesse: 1200x630px (16:9)
- Format: PNG
- Design: Dark Mode Hintergrund (konsistent mit Product Aesthetic)
- Inhalt:
  - PayWatcher Logo/Wordmark (oben links oder zentriert)
  - Headline: "Payment verification without the payment processor"
  - Sub-Info: "$0.05 per verification" oder "Non-custodial USDC Verification"
  - Accent-Farbe Teal (#00D4AA) als Akzent
  - "Built by masemIT" (klein, unten rechts)
- Datei: `public/og-image.png`

**HINWEIS:** Das OG Image muss manuell erstellt oder via Figma/Design-Tool generiert werden. Der Dev Agent kann ein Placeholder-Image erstellen, aber das finale Design sollte professionell sein. Alternative: Next.js `ImageResponse` API fuer programmatische OG-Image-Generierung (einfacher, aber weniger Design-Kontrolle).

### Analytics-Events Spezifikation

| Event Name | Trigger | Properties | Section |
|-----------|---------|------------|---------|
| `page_view` | Landing Page Load | `{ page: "landing" }` | Global |
| `cta_click` | CTA Button Click | `{ label: "Get API Key", section: "hero" }` | Hero, Pricing, Comparison, Trust |
| `section_view` | Section wird sichtbar (IntersectionObserver, threshold 0.3) | `{ section: "pricing" }` | Alle 8 Sections |
| `code_copy` | Copy-Button in Code Block | `{ language: "curl" }` | Code Examples, Hero |
| `pricing_view` | Pricing Section sichtbar | `{ section: "pricing" }` | Pricing |

**IntersectionObserver Strategy:**
- Bestehende `animate-on-scroll.tsx` nutzt bereits IntersectionObserver
- Fuer Analytics: Separater Observer oder Erweiterung des bestehenden Patterns
- `threshold: 0.3` — Section gilt als "gesehen" wenn 30% sichtbar
- Jede Section nur EINMAL tracken (Set oder Flag um Duplikate zu vermeiden)

### Responsive Strategy

Keine visuellen Aenderungen — diese Story betrifft nur unsichtbare Infrastruktur (Meta Tags, Analytics). Die Landing Page Responsive-Behaviour bleibt unveraendert.

### Architektur-Konformitaet

**Dateinamen:** `kebab-case.ts` / `kebab-case.tsx` — IMMER.

**Komponenten-Pfade:**
- `src/app/layout.tsx` — AENDERN: Metadata erweitern, JSON-LD einbinden
- `src/app/sitemap.ts` — NEU: Dynamische Sitemap
- `src/app/robots.ts` — NEU: Robots.txt
- `src/lib/analytics.ts` — NEU: Analytics trackEvent() Funktion
- `src/components/landing/analytics-provider.tsx` — NEU: Client Component fuer Page View + Section Tracking
- `src/components/shared/copy-button.tsx` — AENDERN: `code_copy` Event einbauen
- `public/og-image.png` — NEU: OG Image (1200x630px)
- `.env.example` — AENDERN: ANALYTICS_URL → NEXT_PUBLIC_ANALYTICS_URL

**Server vs Client Components:**
- Server Components (DEFAULT): layout.tsx, sitemap.ts, robots.ts
- Client Components: `analytics-provider.tsx` (braucht useEffect + IntersectionObserver), Aenderungen in `copy-button.tsx` (bereits Client Component)
- `lib/analytics.ts` ist eine reine Utility-Funktion (kein Component), wird von Client Components importiert

**ANTI-Patterns:**
- KEIN `@vercel/analytics` — masemIT Analytics ist das Ziel
- KEIN Google Analytics / gtag — nicht im Scope
- KEIN synchrones Event-Sending — IMMER sendBeacon oder fetch keepalive
- KEIN Tracking in Dev/Preview — Production-Only Guard
- KEIN invasives Tracking (Mausbewegungen, Scroll-Tiefe in Pixeln, etc.)
- KEIN `next-seo` Package — Next.js 16 Metadata API reicht
- KEINE dynamisch generierten OG Images — statisches Image fuer MVP
- KEINE Analytics die SSG brechen — Client Component fuer Tracking, Server Component fuer Metadata

### Library & Framework Anforderungen

**Next.js 16 Metadata API (nativ):**
- `export const metadata: Metadata` in layout.tsx / page.tsx
- `metadataBase` fuer relative URL-Aufloesung
- `title.template` fuer Page-spezifische Titles: `'%s | PayWatcher'`
- OG Image wird automatisch als `<meta property="og:image">` gerendert
- Twitter Cards automatisch aus `twitter` Config
- Keine zusaetzlichen Packages noetig

**Next.js 16 Sitemap/Robots (nativ):**
- `app/sitemap.ts` exportiert `MetadataRoute.Sitemap`
- `app/robots.ts` exportiert `MetadataRoute.Robots`
- Automatisch unter `/sitemap.xml` und `/robots.txt` erreichbar
- Keine zusaetzlichen Packages noetig

**navigator.sendBeacon (Browser API):**
- Nativ in allen modernen Browsern
- Sendet Daten asynchron, blockiert nicht die Seite
- Funktioniert auch bei Page Unload (ideal fuer Analytics)
- Fallback: `fetch` mit `keepalive: true`

### Dateistruktur-Anforderungen

**Neue/geaenderte Dateien nach Abschluss:**

```
src/
├── app/
│   ├── layout.tsx                              # AENDERN: Metadata erweitern + JSON-LD
│   ├── sitemap.ts                              # NEU: Dynamische Sitemap
│   └── robots.ts                               # NEU: Robots.txt
│
├── components/
│   ├── landing/
│   │   └── analytics-provider.tsx              # NEU: Client Component fuer Page View + Section Tracking
│   └── shared/
│       └── copy-button.tsx                     # AENDERN: code_copy Event hinzufuegen
│
├── lib/
│   └── analytics.ts                            # NEU: trackEvent() Utility
│
public/
├── og-image.png                                # NEU: OG Image (1200x630px)

.env.example                                    # AENDERN: NEXT_PUBLIC_ Prefix fuer Analytics Vars
```

### Testing-Anforderungen

1. **Build-Validierung:** `npm run build` muss ohne Fehler durchlaufen
2. **Type-Check:** `npm run type-check` muss ohne Fehler durchlaufen
3. **Lint:** `npm run lint` muss ohne Fehler durchlaufen
4. **SSG-Validierung:** Landing Page muss als `○ (Static)` im Build Output erscheinen — Analytics Client Component darf SSG NICHT brechen
5. **Sitemap:** `curl localhost:3000/sitemap.xml` liefert valides XML
6. **Robots.txt:** `curl localhost:3000/robots.txt` liefert korrekte Regeln
7. **Meta Tags:** View Source der Landing Page zeigt korrekte `<meta>` Tags (og:title, og:description, og:image, twitter:card)
8. **Structured Data:** JSON-LD Block im HTML validierbar (Google Rich Results Test oder Schema.org Validator)
9. **Analytics Dev-Mode:** In Dev zeigt `console.debug('[analytics]', ...)` statt Netzwerk-Requests
10. **Analytics Production Guard:** Kein `fetch`/`sendBeacon` Call wenn `NODE_ENV !== 'production'`
11. **Performance:** Lighthouse FCP < 1.5s (SSG + kein synchroner Analytics-Code)

### Vorherige Story Intelligence

**Story 1.5 Learnings (KRITISCH — direkte Vorgaenger-Story):**
- **Alle Landing Page Sektionen komplett** — Hero, ProblemSolution, HowItWorks, CodeExamples, Pricing, Comparison, TrustSignals sind fertig
- **Section-IDs konsistent:** `id="hero"`, `id="problem-solution"`, `id="how-it-works"`, `id="code-examples"`, `id="pricing"`, `id="comparison"`, `id="trust-signals"` — NUTZEN fuer Section View Tracking
- **CTA-Buttons in ALLEN Sections** verlinken auf `/signup` — NUTZEN fuer CTA Click Tracking
- **SSG bestaetigt funktionierend** — Landing Page ist `○ (Static)` im Build Output
- **Keine `'use client'` auf page.tsx** — Analytics-Provider muss als separater Client Component eingebunden werden, NICHT auf page.tsx
- **`max-w-5xl mx-auto` bzw. `max-w-6xl`** fuer Layout — nicht relevant fuer diese Story
- **`py-16`** fuer Section-Spacing — nicht relevant fuer diese Story
- **Enterprise-CTA** linkt auf `mailto:enterprise@paywatcher.dev` (nicht /signup)

**Story 1.4 Learnings:**
- **`animate-on-scroll.tsx`** existiert als Client Component mit IntersectionObserver — Pattern kann fuer Analytics Section Tracking wiederverwendet werden
- **CopyButton** in `src/components/shared/copy-button.tsx` — AENDERN fuer `code_copy` Event
- **Code Tabs** in `code-tabs.tsx` — Sprach-Info verfuegbar fuer `code_copy` Event Properties

**Story 1.2 Learnings:**
- **Design Tokens** via OKLch in globals.css — Accent/Primary: Teal (~#00D4AA) fuer OG Image
- **`next-themes`** mit `attribute="class"` → Theme Color Meta Tag beachten

**Story 1.1 Learnings:**
- **Route Group `(landing)`** existiert
- **`geist` Package** fuer Fonts
- **`.env.example`** definiert `ANALYTICS_URL` und `ANALYTICS_KEY` (muessen zu `NEXT_PUBLIC_` werden)

### Git Intelligence

**Letzte Commits:**
```
95b7f01 Story 1.5: Pricing section, comparison callout, trust signals with code review fixes
ef7f25e Code review fixes for Story 1.4: CopyButton timeout, scroll indicator, FOUC, grid alignment
3b8016a Story 1.4: How It Works section, interactive code examples with tabs
36a83f8 Story 1.3: Landing page layout, hero section, and problem/solution
5a45117 Story 1.2: Theme system, design tokens, and payment status colors
```

**Patterns aus vorherigen Stories:**
- Server Components als Default — bewaehrt, beibehalten
- Client Components nur wenn noetig (animate-on-scroll, copy-button, code-tabs, mobile-nav, theme-toggle)
- `'use client'` Directive nur in Component-Dateien, NIE in page.tsx
- kebab-case Dateinamen — strikt einhalten
- `npm run build && npm run type-check && npm run lint` als Validierung

### Aktuelle Technologie-Informationen (Web Research, Feb 2026)

**Next.js 16 Metadata API:**
- `Metadata` Type importieren aus `next`
- `metadataBase` setzt Base-URL fuer relative Pfade (z.B. OG Image `/og-image.png` → `https://paywatcher.dev/og-image.png`)
- `title.template` ermoeglicht Page-spezifische Titles mit konsistentem Suffix
- OG Images koennen als relative Pfade angegeben werden wenn `metadataBase` gesetzt
- Keine Breaking Changes seit Next.js 15 Metadata API

**Next.js 16 Sitemap/Robots:**
- `app/sitemap.ts` mit `MetadataRoute.Sitemap` Return Type
- `app/robots.ts` mit `MetadataRoute.Robots` Return Type
- Automatisch unter `/sitemap.xml` und `/robots.txt` verfuegbar
- Unterstuetzt `changeFrequency` und `priority` Felder

**navigator.sendBeacon:**
- Supported in allen modernen Browsern (Chrome, Firefox, Safari, Edge)
- Max Payload: ~64KB (mehr als ausreichend fuer Analytics Events)
- Sendet als POST Request mit `Content-Type: text/plain` (CORS-safe) oder `application/json` via Blob
- Funktioniert zuverlaessig bei Page Navigation/Unload

### Project Structure Notes

- Story 1.6 ist die **letzte Story in Epic 1** — nach Abschluss ist die gesamte Landing Page inkl. Infrastruktur fertig
- **SEO und Analytics sind unsichtbar** — keine visuellen UI-Aenderungen, nur Meta Tags im `<head>` und asynchrone Event-Sends
- **Analytics-Provider als Client Component** muss in `(landing)/layout.tsx` eingebunden werden, NICHT in page.tsx (um SSG nicht zu brechen)
- **CopyButton-Aenderung** ist minimal — `trackEvent('code_copy', { language })` am Ende der Copy-Funktion
- **OG Image** wird als statische Datei in `/public` abgelegt — kein Build-Step noetig
- **Component Boundaries beachten:** Analytics-Utility (`lib/analytics.ts`) ist shared, Analytics-Provider (`landing/analytics-provider.tsx`) ist Landing-spezifisch

### References

- [Source: _bmad-output/planning-artifacts/epics.md#Story 1.6] — User Story und Acceptance Criteria
- [Source: _bmad-output/planning-artifacts/prd.md#Functional Requirements] — FR32 (Event Tracking), FR33 (Analytics), FR34 (SEO), FR35 (Social Sharing)
- [Source: _bmad-output/planning-artifacts/prd.md#NonFunctional Requirements] — NFR1 (FCP < 1.5s), NFR13 (Async Analytics), NFR16 (Env Vars), NFR17 (No Secrets)
- [Source: _bmad-output/planning-artifacts/architecture.md#Analytics] — masemIT Analytics Pattern, sendBeacon, Event-Schema
- [Source: _bmad-output/planning-artifacts/architecture.md#Route Structure] — app/sitemap.ts, app/robots.ts Platzierung
- [Source: _bmad-output/planning-artifacts/architecture.md#Environment Variables] — ANALYTICS_URL, ANALYTICS_KEY
- [Source: _bmad-output/planning-artifacts/architecture.md#Metadata] — Next.js Metadata API Pattern
- [Source: _bmad-output/planning-artifacts/ux-design-specification.md#Flow 3: Landing Page Conversion] — 8 Sektionen mit Tracking Points
- [Source: _bmad-output/implementation-artifacts/1-5-pricing-comparison-trust-signals.md] — Section-IDs, CTA-Patterns, SSG-Bestaetigung

## Dev Agent Record

### Agent Model Used

Claude Opus 4.6

### Debug Log References

### Completion Notes List

- Task 1: Root layout.tsx erweitert mit vollstaendiger Next.js Metadata (metadataBase, title template, OG tags, Twitter Cards, keywords, authors). JSON-LD Structured Data (WebSite + Organization Schema) via script tag eingebunden. Landing layout benoetigt keine eigenen Overrides. OG Image als Placeholder PNG (1200x630, dunkler Hintergrund mit Teal-Akzent) erstellt — muss durch professionelles Design ersetzt werden.
- Task 2: sitemap.ts und robots.ts mit nativer Next.js MetadataRoute API erstellt. Sitemap enthaelt Landing Page Route. Robots.txt blockiert /dashboard/, /admin/, /api/ und verweist auf Sitemap.
- Task 3: analytics.ts Utility mit trackEvent() erstellt — delegiert an masemIT tracker.js (window.masemit.track). Production-Only: tracker.js wird nur bei VERCEL_ENV=production geladen, console.debug in Dev. .env.example auf NEXT_PUBLIC_ANALYTICS_PROJECT aktualisiert.
- Task 4: AnalyticsProvider als Client Component erstellt mit page_view bei Mount, section_view via IntersectionObserver (threshold 0.3, jede Section nur einmal), cta_click via delegiertem Event Listener auf a[href='/signup']. CopyButton um language Prop und code_copy Event erweitert. CodeTabs und CodeBlock uebergeben jetzt language an CopyButton.
- Task 5: Build fehlerfrei (build, type-check, lint). Landing Page bestaetigt als Static (SSG). /sitemap.xml und /robots.txt im Build Output.

### Change Log

- 2026-02-16: Story 1.6 implementiert — SEO Metadata, OG Tags, Twitter Cards, JSON-LD, Sitemap, Robots.txt, Analytics-Infrastruktur, Event-Tracking auf Landing Page
- 2026-02-16: Code Review Fixes — tracker.js Guard auf VERCEL_ENV umgestellt (kein Tracking auf Preview), CTA-Section-Erkennung fuer Header/Nav verbessert

### File List

**Neue Dateien:**
- src/app/sitemap.ts
- src/app/robots.ts
- src/lib/analytics.ts
- src/components/landing/analytics-provider.tsx
- public/og-image.png (Placeholder — muss durch professionelles Design ersetzt werden)

**Geaenderte Dateien:**
- src/app/layout.tsx (Metadata erweitert + JSON-LD)
- src/app/(landing)/layout.tsx (AnalyticsProvider eingebunden)
- src/components/shared/copy-button.tsx (language Prop + code_copy Event)
- src/components/landing/code-tabs.tsx (language Prop an CopyButton)
- src/components/shared/code-block.tsx (language Prop an CopyButton)
- .env.example (ANALYTICS_URL/KEY → NEXT_PUBLIC_ANALYTICS_PROJECT)
