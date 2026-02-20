# Story 1.1: Project Scaffold & Vercel Deployment

Status: done

<!-- Note: Validation is optional. Run validate-create-story for quality check before dev-story. -->

## Story

As a Developer (Sempre),
I want ein initialisiertes Next.js 16 Projekt mit Tailwind v4, TypeScript, App Router und Vercel-Deployment,
so that ich eine lauffähige Basis habe auf der alle weiteren Features aufbauen.

## Acceptance Criteria

1. **Given** kein Projekt existiert **When** `npx create-next-app@latest paywatcher --typescript --tailwind --eslint --app --turbopack` ausgeführt wird **Then** ein Next.js 16 Projekt mit App Router, Tailwind v4, TypeScript strict und ESLint ist erstellt **And** die Projektstruktur folgt dem Architecture Doc (src/app, src/components, src/lib, src/types, src/hooks, src/styles)

2. **Given** das Projekt ist initialisiert **When** shadcn/ui via `npx shadcn@latest init` eingerichtet wird **Then** die components.json ist konfiguriert und erste Basis-Komponenten (Button, Badge, Card) sind installiert

3. **Given** das Projekt ist auf GitHub gepusht **When** Vercel-Deployment konfiguriert wird **Then** die App ist unter einer Preview-URL erreichbar und automatisches CI/CD (Git Push → Deploy) funktioniert

4. **Given** ein `.env.example` existiert **When** ein Entwickler die Environment Variables prüft **Then** sind folgende Keys dokumentiert: MMS_API_URL, MMS_API_KEY, NEXTAUTH_SECRET, NEXTAUTH_URL, RESEND_API_KEY, ANALYTICS_URL, ANALYTICS_KEY

## Tasks / Subtasks

- [x] Task 1: Next.js 16 Projekt initialisieren (AC: #1)
  - [x] 1.1 `npx create-next-app@latest paywatcher --typescript --tailwind --eslint --app --turbopack` ausfuehren
  - [x] 1.2 Verifizieren: Next.js 16.x, React 19.x, Tailwind v4, TypeScript strict, App Router aktiv
  - [x] 1.3 Default-Boilerplate aufraeumen (Default-Seiten, Styles, Placeholder-Content entfernen)
  - [x] 1.4 `npm run dev` und `npm run build` erfolgreich testen

- [x] Task 2: Projektstruktur gemaess Architecture Doc anlegen (AC: #1)
  - [x] 2.1 Verzeichnisse erstellen: `src/components/ui/`, `src/components/landing/`, `src/components/dashboard/`, `src/components/shared/`
  - [x] 2.2 Verzeichnisse erstellen: `src/hooks/`, `src/lib/`, `src/lib/api/`, `src/types/`, `src/styles/`
  - [x] 2.3 Route Groups anlegen: `src/app/(landing)/`, `src/app/(dashboard)/`, `src/app/(admin)/`
  - [x] 2.4 API-Route Verzeichnis: `src/app/api/`
  - [x] 2.5 Content-Verzeichnis: `content/docs/`
  - [x] 2.6 Placeholder-Dateien in Route Groups (.gitkeep oder minimale page.tsx) damit Struktur commitbar ist

- [x] Task 3: shadcn/ui einrichten (AC: #2)
  - [x] 3.1 `npx shadcn@latest init` ausfuehren (New York Style, Default Color, CSS Variables aktiviert)
  - [x] 3.2 Basis-Komponenten installieren: `npx shadcn@latest add button badge card`
  - [x] 3.3 components.json pruefen: Pfade korrekt (`@/components/ui`), TypeScript aktiviert
  - [x] 3.4 `cn()` Utility in `src/lib/utils.ts` verifizieren (clsx + tailwind-merge)

- [x] Task 4: Environment Variables konfigurieren (AC: #4)
  - [x] 4.1 `.env.example` erstellen mit allen Keys und Kommentaren
  - [x] 4.2 `.env.local` in `.gitignore` verifizieren (sollte bereits vorhanden sein)
  - [x] 4.3 Keys: `MMS_API_URL`, `MMS_API_KEY`, `NEXTAUTH_SECRET`, `NEXTAUTH_URL`, `RESEND_API_KEY`, `ANALYTICS_URL`, `ANALYTICS_KEY`

- [x] Task 5: GitHub Repository & Vercel Deployment (AC: #3)
  - [x] 5.1 Git Repository initialisieren (falls nicht bereits), Initial Commit erstellen
  - [x] 5.2 GitHub Repository erstellen und pushen
  - [ ] 5.3 Vercel-Projekt verbinden (Import from GitHub) — manuell durch User
  - [ ] 5.4 Vercel Environment Variables konfigurieren (Placeholder-Werte fuer Dev) — manuell durch User
  - [ ] 5.5 Automatisches Deployment verifizieren: Push → Build → Preview-URL erreichbar — manuell durch User
  - [ ] 5.6 Production-Domain (paywatcher.dev) konfigurieren (falls bereits gewuenscht) — manuell durch User

## Dev Notes

### Kontext & Zweck

Dies ist die **allererste Story** des PayWatcher-Projekts. Es existiert noch KEIN Code. Du erstellst das Greenfield-Fundament auf dem alle 5 Epics (Landing Page, Auth, Payments Dashboard, Webhooks/Settings, Docs) aufbauen werden.

**Was PayWatcher ist:** Eine Developer-First Verification API fuer Stablecoin-Zahlungen (USDC auf Base Network). Non-custodial, pure Verification Layer. Das Frontend (dieses Projekt) ist ein reiner API-Consumer — es gibt KEINE eigene Datenbank. Alle Daten kommen vom MMS Backend (api.masem.at) ueber Server-Side Route Handlers.

**Was diese Story NICHT beinhaltet:**
- Kein Theme System / Design Tokens (Story 1.2)
- Keine Landing Page Inhalte (Stories 1.3-1.6)
- Kein Auth-System (Epic 2)
- Keine Dashboard-Pages (Epic 3-4)
- Keine Docs-Inhalte (Epic 5)

**Was diese Story liefert:**
- Lauffaehiges Next.js 16 Projekt mit korrekter Verzeichnisstruktur
- shadcn/ui initialisiert mit Basis-Komponenten
- Vercel CI/CD funktioniert (Git Push → Deploy)
- .env.example mit allen benoetigten Keys dokumentiert

### Technische Anforderungen

**Exakte Versionen (Stand Feb 2026):**
- **Next.js:** 16.x (aktuell 16.1.6) — NICHT Next.js 15!
- **React:** 19.x (19.2) mit stabilem React Compiler
- **Tailwind CSS:** v4 — CSS-first Configuration via `@theme` Directive, KEIN `tailwind.config.js`
- **TypeScript:** strict mode (Next.js 16 Default)
- **Turbopack:** Stabil und Default fuer `dev` und `build`
- **Node.js:** >= 20.9 (18 wird NICHT unterstuetzt)
- **Package Manager:** npm

**Initialisierung:**
```bash
npx create-next-app@latest paywatcher --typescript --tailwind --eslint --app --turbopack
```

**KRITISCHE Next.js 16 Aenderungen gegenueber Next.js 15:**
- `middleware.ts` wurde umbenannt zu `proxy.ts` (Edge Runtime nicht mehr unterstuetzt, nur Node.js)
- Caching ist jetzt opt-in — kein automatisches Caching mehr, explizite `"use cache"` Directive noetig
- Async Request APIs: `params`, `searchParams`, `cookies()`, `headers()` und `draftMode()` erfordern jetzt `await`
- React Compiler ist stabil — KEIN manuelles `useMemo`/`useCallback` noetig
- Parallel Routes erfordern explizite `default.js` Dateien in allen Slots

### Architektur-Konformitaet

**Routing-Architektur (Route Groups):**
Das Projekt verwendet Route Groups zur Bereichstrennung. Alle drei muessen in dieser Story angelegt werden:
- `src/app/(landing)/` — Public, SSG, SEO-optimiert (Landing Page + Docs + Login/Signup)
- `src/app/(dashboard)/` — Auth-protected, CSR, Desktop-First (User Dashboard)
- `src/app/(admin)/` — Admin-protected, Phase 1b (Platzhalter)
- `src/app/api/` — Route Handlers als Server-Side Proxy zu MMS Backend

**Rendering-Strategie:**
- Server Components als Default — `'use client'` nur wo explizit noetig
- Landing Page: SSG (Static Site Generation)
- Dashboard: CSR mit TanStack Query (kommt in spaeteren Stories)
- API Routes: Serverless Functions auf Vercel

**Architektur-Boundaries die ab Story 1 gelten:**
- `components/landing/` importiert NIEMALS aus `components/dashboard/` und umgekehrt
- Shared Components nur in `components/shared/`
- shadcn/ui generierte Files nur in `components/ui/` — keine Custom-Logik dort
- Types zentral in `types/` — nie inline in Components (ausser Props)
- `lib/` fuer Utilities — keine React Components in `lib/`
- Tests co-located: `*.test.ts` neben Source-File

**Anti-Patterns (NIEMALS):**
- PascalCase Dateinamen → immer `kebab-case.tsx`
- `enum` → immer Union Types
- `interface IPayment {}` → kein `I`-Prefix
- `any` Type → immer explizite Types
- Inline Styles → immer Tailwind Utilities

### Library & Framework Anforderungen

**In dieser Story zu installieren:**

| Package | Installation | Zweck |
|---------|-------------|-------|
| Next.js 16 | via create-next-app | Framework |
| Tailwind v4 | via create-next-app | Styling |
| TypeScript | via create-next-app | Typisierung |
| ESLint | via create-next-app | Linting |
| shadcn/ui | `npx shadcn@latest init` | UI Components |
| Button | `npx shadcn@latest add button` | Basis-Komponente |
| Badge | `npx shadcn@latest add badge` | Basis-Komponente |
| Card | `npx shadcn@latest add card` | Basis-Komponente |

**NICHT in dieser Story installieren (spaetere Stories):**
- `@tanstack/react-query` → Story 2.1 (Dashboard Shell)
- `next-auth@beta` / `better-auth` → Story 2.2 (Auth)
- `shiki` → Story 1.3 (Code-Beispiele)
- `lucide-react` → wird mit shadcn/ui automatisch installiert
- `recharts` → Story 3.2 (Overview Stats)
- `next-themes` → Story 1.2 (Theme System)
- `sonner` → Story 1.2 (Toast-System)
- `react-hook-form` + `zod` → Story 2.2 (Forms)

**shadcn/ui Konfiguration:**
- Style: New York
- Base Color: Default (wird in Story 1.2 angepasst)
- CSS Variables: Ja (aktivieren!)
- Path Alias: `@/` → `src/`
- Components Path: `@/components/ui`

**WARNUNG:** shadcn/ui hat seit Feb 2026 ein neues unified `radix-ui` Package. Falls `npx shadcn@latest init` das neue Format anbietet, nutzen. Falls es `npx shadcn create` anbietet (Visual Builder), die CLI-Variante bevorzugen fuer reproduzierbare Ergebnisse.

### Dateistruktur-Anforderungen

**Ziel-Verzeichnisstruktur nach Abschluss dieser Story:**

```
paywatcher/
├── .env.example                    # Environment Variables Template
├── .gitignore
├── next.config.ts                  # Next.js 16 Konfiguration
├── package.json
├── tsconfig.json
├── components.json                 # shadcn/ui Konfiguration
├── README.md
├── public/
│   └── favicon.ico
│
├── content/                        # MDX Docs Content (leer, Platzhalter)
│   └── docs/
│       └── .gitkeep
│
└── src/
    ├── app/
    │   ├── layout.tsx              # Root Layout (Fonts, HTML-Grundstruktur)
    │   ├── page.tsx                # Temporaere Root Page (Redirect oder Placeholder)
    │   ├── globals.css             # Tailwind v4 Basis-Styles
    │   │
    │   ├── (landing)/              # PUBLIC — SSG
    │   │   └── page.tsx            # Placeholder: "PayWatcher - Coming Soon"
    │   │
    │   ├── (dashboard)/            # AUTH-PROTECTED (spaeter)
    │   │   └── page.tsx            # Placeholder: "Dashboard"
    │   │
    │   ├── (admin)/                # ADMIN-PROTECTED (Phase 1b)
    │   │   └── page.tsx            # Placeholder: "Admin"
    │   │
    │   └── api/                    # ROUTE HANDLERS
    │       └── health/
    │           └── route.ts        # GET /api/health → { status: "ok" }
    │
    ├── components/
    │   ├── ui/                     # shadcn/ui generierte Components
    │   │   ├── button.tsx          # via shadcn add
    │   │   ├── badge.tsx           # via shadcn add
    │   │   └── card.tsx            # via shadcn add
    │   ├── landing/                # Landing Page Components (leer)
    │   │   └── .gitkeep
    │   ├── dashboard/              # Dashboard Components (leer)
    │   │   └── .gitkeep
    │   └── shared/                 # Shared Components (leer)
    │       └── .gitkeep
    │
    ├── hooks/                      # Custom Hooks (leer)
    │   └── .gitkeep
    │
    ├── lib/                        # Utilities & Config
    │   ├── utils.ts                # cn() Utility (von shadcn/ui generiert)
    │   └── api/                    # MMS API Client (leer)
    │       └── .gitkeep
    │
    ├── types/                      # Zentrale TypeScript Types (leer)
    │   └── .gitkeep
    │
    └── styles/                     # Globale Styles (optional, globals.css ist in app/)
        └── .gitkeep
```

**Wichtige Hinweise:**
- `globals.css` wird von create-next-app in `src/app/` platziert — dort belassen (Tailwind v4 Standard)
- Route Group Pages sind minimale Placeholder — nur damit die Struktur steht
- `/api/health` Route als Smoke Test fuer Vercel Deployment
- `.gitkeep` Dateien in leeren Verzeichnissen damit die Struktur versioniert wird
- Kein `src/middleware.ts` in dieser Story — in Next.js 16 heisst es `proxy.ts` und kommt erst in Story 2.1

### Testing-Anforderungen

**Fuer diese Story:**

Diese Story ist primaer ein Scaffold — keine komplexe Businesslogik zu testen. Aber:

1. **Build-Validierung:** `npm run build` muss ohne Fehler durchlaufen
2. **Dev-Server:** `npm run dev` muss starten und localhost:3000 erreichbar sein
3. **Type-Check:** `npx tsc --noEmit` muss ohne Fehler durchlaufen
4. **Lint:** `npm run lint` muss ohne Fehler durchlaufen (ESLint)
5. **Health Endpoint:** `GET /api/health` muss `{ "status": "ok" }` zurueckgeben
6. **Vercel Deploy:** Push → Build → Preview-URL muss erreichbar sein und die Placeholder-Seite anzeigen

**Testing-Framework (Vorbereitung fuer spaetere Stories):**
- Vitest + React Testing Library wird in dieser Story NICHT installiert
- Testing-Setup kommt in einer spaeteren Story wenn erste testbare Logik existiert
- Co-located Tests: `*.test.ts` neben Source-Files (Konvention bereits festgelegt)

### Aktuelle Technologie-Informationen (Web Research, Feb 2026)

**Next.js 16 Besonderheiten:**
- Turbopack ist jetzt Default-Bundler (dev + build) — kein Webpack mehr noetig
- File System Caching ist stabil und standardmaessig aktiviert (seit 16.1)
- `next lint` wurde entfernt — ESLint direkt verwenden (`npx eslint .`)
- AMP Support wurde entfernt
- Runtime Configs wurden entfernt — `.env` Dateien verwenden

**Tailwind v4 Besonderheiten:**
- CSS-first Configuration: `@theme { }` Directive in CSS statt `tailwind.config.js`
- Automatische Content Detection — keine `content`-Pfade konfigurieren
- 70% kleiner als v3 (6-12 KB gzipped)
- `@import "tailwindcss"` statt `@tailwind base/components/utilities`

**shadcn/ui (Feb 2026):**
- Neues unified `radix-ui` Package (ersetzt individuelle `@radix-ui/react-*` Packages)
- `npx shadcn@latest init` funktioniert weiterhin fuer CLI-basiertes Setup
- Lucide Icons werden automatisch als Dependency installiert
- Unterstuetzt Next.js 16 vollstaendig

**Auth-Landschaft (Vorausschau fuer Epic 2):**
- Auth.js v5 ist immer noch Beta — funktional aber nicht stabil released
- Better Auth wird fuer neue Projekte empfohlen (Auth.js Maintenance uebernommen)
- Entscheidung Auth.js vs. Better Auth wird in Story 2.2 getroffen — NICHT in dieser Story

### Project Structure Notes

- Projekt ist Greenfield — kein bestehender Code, keine Konflikte
- Die Verzeichnisstruktur folgt exakt dem Architecture Doc [Source: _bmad-output/planning-artifacts/architecture.md#Vollständige Projektverzeichnis-Struktur]
- Route Groups `(landing)`, `(dashboard)`, `(admin)` sind architektonisch festgelegt [Source: _bmad-output/planning-artifacts/architecture.md#App Structure & Routing]
- Naming Convention: `kebab-case.tsx` fuer alle Dateien [Source: _bmad-output/planning-artifacts/architecture.md#Naming Patterns]

### References

- [Source: _bmad-output/planning-artifacts/architecture.md#Starter Template Evaluation] — Initialisierungscommand und Rationale
- [Source: _bmad-output/planning-artifacts/architecture.md#Vollständige Projektverzeichnis-Struktur] — Ziel-Dateistruktur
- [Source: _bmad-output/planning-artifacts/architecture.md#Implementation Patterns & Consistency Rules] — Naming, Anti-Patterns, Enforcement Rules
- [Source: _bmad-output/planning-artifacts/architecture.md#Core Architectural Decisions] — Route Groups, API Proxy, Data Transformation
- [Source: _bmad-output/planning-artifacts/epics.md#Story 1.1] — User Story und Acceptance Criteria
- [Source: _bmad-output/planning-artifacts/prd.md#Web App Specific Requirements] — Tech Stack Vorgaben
- [Source: _bmad-output/planning-artifacts/ux-design-specification.md] — Design System Foundation (relevant ab Story 1.2)

## Dev Agent Record

### Agent Model Used

Claude Opus 4.6 (claude-opus-4-6)

### Debug Log References

- Route Group Pfadkonflikt: `(landing)`, `(dashboard)`, `(admin)` hatten alle `page.tsx` auf `/`. Geloest durch Verschiebung der Dashboard/Admin-Pages in Unterverzeichnisse: `(dashboard)/dashboard/page.tsx` → `/dashboard`, `(admin)/admin/page.tsx` → `/admin`.
- `create-next-app` verweigert nicht-leere Verzeichnisse. Workaround: Temporaeres Verzeichnis erstellt, Dateien kopiert, temp-Verzeichnis geloescht.
- React Compiler Prompt konnte nicht automatisch beantwortet werden (interaktiver CLI-Prompt). Default "No" wurde akzeptiert — React Compiler ist NICHT aktiv. Kann spaeter in `next.config.ts` via `reactCompiler: true` aktiviert werden.

### Completion Notes List

- ✅ Next.js 16.1.6 mit React 19.2.3, Tailwind v4, TypeScript strict, App Router, Turbopack
- ✅ Projektstruktur gemaess Architecture Doc: Route Groups, Components, Hooks, Lib, Types, Styles
- ✅ shadcn/ui initialisiert (New York, Neutral, CSS Variables) mit Button, Badge, Card
- ✅ `.env.example` mit allen 7 Keys dokumentiert, `.gitignore` korrekt konfiguriert
- ✅ Code auf GitHub gepusht (masem-at/paywatcher)
- ✅ Build, TypeScript Type-Check und ESLint fehlerfrei
- ⏳ Vercel-Deployment wird manuell durch User konfiguriert (Tasks 5.3-5.6)

### File List

- .env.example (neu)
- .gitignore (neu)
- README.md (neu)
- components.json (neu)
- eslint.config.mjs (neu)
- next.config.ts (neu)
- package.json (neu)
- package-lock.json (neu)
- postcss.config.mjs (neu)
- tsconfig.json (neu)
- public/favicon.ico (neu)
- content/docs/.gitkeep (neu)
- src/app/layout.tsx (neu)
- src/app/globals.css (neu)
- src/app/(landing)/page.tsx (neu)
- src/app/(dashboard)/dashboard/page.tsx (neu)
- src/app/(admin)/admin/page.tsx (neu)
- src/app/api/health/route.ts (neu)
- src/components/ui/button.tsx (neu)
- src/components/ui/badge.tsx (neu)
- src/components/ui/card.tsx (neu)
- src/components/landing/.gitkeep (neu)
- src/components/dashboard/.gitkeep (neu)
- src/components/shared/.gitkeep (neu)
- src/hooks/.gitkeep (neu)
- src/lib/utils.ts (neu)
- src/lib/api/.gitkeep (neu)
- src/types/.gitkeep (neu)
- src/styles/.gitkeep (neu)

### Change Log

- 2026-02-15: Story 1.1 implementiert — Next.js 16 Scaffold mit Tailwind v4, shadcn/ui, Projektstruktur und GitHub-Push. Vercel-Deployment manuell ausstehend.
- 2026-02-16: Code Review (adversarial) — 7 Issues gefunden (3 HIGH, 4 MEDIUM). Alle automatisch gefixt: Duplicate CSS entfernt, Font-Loading auf next/font/local umgestellt (geist Package), React Compiler Status klargestellt, README projektspezifisch geschrieben, .env.example korrigiert, type-check Script hinzugefuegt.
