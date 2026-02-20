---
stepsCompleted: [1, 2, 3, 4, 5, 6, 7, 8]
lastStep: 8
status: 'complete'
completedAt: '2026-02-19'
inputDocuments:
  - _bmad-output/planning-artifacts/product-brief-new-idea-2026-02-18.md
  - _bmad-output/planning-artifacts/prd.md
  - _bmad-output/planning-artifacts/ux-design-specification.md
  - _bmad-output/planning-artifacts/research/market-staking-saas-research-2026-02-18.md
  - _bmad-output/planning-artifacts/research/technical-staketrack-ai-mvp-research-2026-02-18.md
  - _bmad-output/project-context.md
  - docs/_masemIT/company-info.md
  - docs/_masemIT/gemini.md
  - docs/_masemIT/chatgpt.md
workflowType: 'architecture'
project_name: 'StakeTrack AI'
user_name: 'Sempre'
date: '2026-02-19'
---

# Architecture Decision Document

_This document builds collaboratively through step-by-step discovery. Sections are appended as we work through each architectural decision together._

## Project Context Analysis

### Requirements Overview

**Functional Requirements (42 FRs in 6 Kategorien):**

| Kategorie | FRs | Architektonische Implikation |
|---|---|---|
| **Wallet & Identity** | FR1-FR7 | Dual-Auth (SIWE + Email), Multi-Chain-Wallet-Linking, Unified User Profile, DSGVO-Löschung |
| **Dashboard & Portfolio** | FR8-FR14 | Multi-Chain-Aggregation, Graceful Degradation, Freshness-Timestamps, Empty States |
| **Real-Yield-Kalkulation** | FR15-FR20 | Chain-spezifische Inflationsraten, tägliche Snapshots, Public Calculator (unauthentifiziert) |
| **Validator Risk Score** | FR21-FR26 | 5-Faktoren-Scoring, zeitgewichtete Berechnung, immutable Historie, Tier-basierte Granularität |
| **Alert-System** | FR27-FR32 | Event-Driven Architektur, Tier-limitierte Alerts, kontextbezogene E-Mails |
| **Daten-Pipeline & System** | FR33-FR42 | Stündliche Cron-Jobs, Cache-Invalidierung, HMAC-Verifizierung, Rate Limiting, Freemium-Gating |

**Non-Functional Requirements (34 NFRs):**

| Kategorie | NFRs | Architektur-treibende Anforderungen |
|---|---|---|
| **Performance** | NFR1-NFR7 | TTFB < 500ms, Cached < 100ms, Uncached < 2s, Bundle < 200KB, Cold Start < 1s, Time-to-First-Insight < 60s |
| **Security** | NFR8-NFR16 | SIWE serverseitig, SSL Required, keine Private Keys, HMAC auf Cron-Endpoints, Rate Limiting, keine PII in Logs |
| **Scalability** | NFR17-NFR21 | 500-5.000 User ohne Architekturänderung, < 0.5 GiB Storage/12 Monate, 80%+ Cache-Hit-Rate, 60s QStash Parallel |
| **Reliability** | NFR22-NFR27 | 99.5% Uptime, < 1h Data Freshness, Graceful Degradation, deterministischer VRS, immutable Historie |
| **Integration** | NFR28-NFR31 | Chain-Adapter isoliert, VRS > 95% Korrelation, synchrone Cache-Invalidierung, definierte Timeouts |
| **Compliance** | NFR32-NFR34 | DSGVO, Disclaimer, Wallet-Adressen als pseudo-anonyme Daten |

**Scale & Complexity:**

- Primäre Domäne: Full-Stack Serverless Web mit Blockchain Data Ingestion
- Komplexitätslevel: **Hoch**
- Geschätzte architektonische Hauptkomponenten: ~12 (3 Chain Adapters, VRS Engine, Yield Calculator, Auth Layer, Caching Layer, Cron Pipeline, Alert Engine, API Layer, Dashboard UI, Calculator UI)

### Technical Constraints & Dependencies

| Constraint | Quelle | Architektonische Auswirkung |
|---|---|---|
| Vercel Pro 60s Timeout | Infrastruktur | Batch-Verarbeitung via QStash Fan-out, keine Long-Running-Prozesse |
| NeonDB `connection_limit=1` | Serverless | Connection Pooling, kurze DB-Transaktionen |
| NeonDB 0.5 GiB Storage (Launch) | Plan-Limit | TimescaleDB Kompression nach 30 Tagen, effizientes Schema |
| beaconcha.in 1K Req/Monat (Free) | API-Limit | Tägliches Bulk-Fetch, aggressive Caching |
| Solana `getStakeActivation` deprecated | API-Deprecation | Von Anfang an `getAccountInfo` + manuelles Parsing |
| SIMD-123 Revenue-Sharing | Solana Governance | Real-Yield-Formel muss adaptierbar sein |
| Bestehender masemIT-Stack | Brownfield | MMS für User-Management, masemIT Analytics, Resend, Upstash — Cross-Project Service Rule beachten |
| MVP F1-F5 Scope Guard | Projekt-Kontext | Strikte Feature-Begrenzung, kein Scope Creep |

### Cross-Cutting Concerns Identified

| Concern | Betroffene Komponenten | Lösungsansatz |
|---|---|---|
| **Authentication & Identity** | Alle authentifizierten Routes, Dashboard, Alerts, API | Dual-Auth (SIWE + MMS Email), Unified User Profile, JWT-Sessions |
| **Caching (3 Layer)** | Dashboard, API Responses, VRS, Yield | RSC revalidate + Upstash Redis + NeonDB Materialized Views |
| **Freemium-Gating** | Dashboard, Alerts, VRS-Detail, Wallet-Linking | Middleware-basierter Tier-Check, Feature-Flags |
| **Graceful Degradation** | Dashboard, Calculator, alle Chain-Daten | Letzte bekannte Daten + Freshness-Timestamp bei API-Ausfall |
| **Rate Limiting** | Alle Public Endpoints | Upstash Rate Limit Middleware |
| **DSGVO-Compliance** | User-Daten, Wallet-Adressen, Analytics, Logging | Datenminimierung, Consent-Management via MMS, Löschfunktion, keine PII in Logs |
| **Error Handling & Monitoring** | Chain Adapters, Cron Pipeline, Auth | Definierte Timeouts, QStash Retry, Vercel Logs (MVP-ausreichend) |
| **Data Freshness Transparency** | Dashboard, VRS, Yield, Alerts | Timestamps pro Datenpunkt, Stale-Data-Banner |

## Starter Template Evaluation

### Primary Technology Domain

Full-Stack Serverless Web Application (Next.js 15 App Router) — basierend auf Project Context und PRD-Analyse. Blockchain Data Ingestion als sekundäre Domäne.

### Starter Options Considered

| Option | Bewertung | Ergebnis |
|---|---|---|
| **`create-next-app`** (Offiziell) | Minimal, sauber, Tailwind + TypeScript + Biome-Option, volle Kontrolle über DDD-Struktur | **Gewählt** |
| **`create-t3-app`** (T3 Stack) | Drizzle + NextAuth + Tailwind out of the box, aber tRPC ist Overhead (nicht im Stack), T3-Struktur weicht von DDD ab | Verworfen |
| **Supastarter / Paid Boilerplates** | Multi-Tenancy, Payment, Admin — alles explizit OUT OF SCOPE für MVP, fremde Code-Patterns | Verworfen |

### Selected Starter: `create-next-app`

**Rationale:**
1. Kein Ballast — T3's tRPC ist Overhead, Project Context definiert REST API Routes
2. Volle Kontrolle über die DDD-inspirierte Projektstruktur (`src/domain/`, `src/infrastructure/`, `src/app/`)
3. Native Biome-Option löst die "Biome ODER ESLint"-Entscheidung direkt beim Setup
4. Sauberster Ausgangspunkt für ein Brownfield-Projekt mit klar definiertem Stack
5. Tremor und RainbowKit müssen in jedem Fall manuell ergänzt werden

**Initialization Command:**

```bash
npx create-next-app@latest staketrack-ai --typescript --tailwind --app --src-dir --use-npm
```

Bei interaktivem Prompt: Biome als Linter wählen, React Compiler aktivieren.

**Architectural Decisions Provided by Starter:**

| Entscheidung | Wert |
|---|---|
| **Language & Runtime** | TypeScript 5+ (strict mode), React 19+ |
| **Styling Solution** | Tailwind CSS (konfiguriert mit PostCSS) |
| **Build Tooling** | Turbopack (Dev), Webpack (Prod), Next.js Compiler |
| **Code Organization** | `src/` Verzeichnis, App Router (`src/app/`) |
| **Linting** | Biome (bevorzugt) oder ESLint |
| **Development Experience** | Hot Reloading via Turbopack, TypeScript-Fehler im Terminal |

**Sprint 0 — Manuell zu ergänzen:**

| Paket-Gruppe | Pakete |
|---|---|
| **ORM + DB** | `drizzle-orm`, `drizzle-kit`, `@neondatabase/serverless` |
| **Auth** | `next-auth`, `siwe`, `@rainbow-me/rainbowkit`, `@rainbow-me/rainbowkit-siwe-next-auth`, `wagmi`, `viem`, `@keplr-wallet/types`, `@solana/wallet-adapter-react` |
| **UI** | `@tremor/react` |
| **Infrastructure** | `@upstash/redis`, `@upstash/qstash`, `@upstash/ratelimit`, `resend` |
| **Testing** | `vitest`, `@testing-library/react`, `@playwright/test`, `msw` |
| **DDD-Struktur** | `src/domain/`, `src/infrastructure/`, `src/lib/` Ordner anlegen |

**Note:** Projekt-Initialisierung mit diesem Kommando ist die erste Implementierungs-Story.

## Core Architectural Decisions

### Decision Priority Analysis

**Critical Decisions (Block Implementation):**
- Data Validation: Zod v4.3 — Runtime-Validierung für Chain-API-Daten ist sicherheitskritisch
- Server State: TanStack Query v5.90 — Client-Interaktionen (Filter, Refresh) brauchen Caching-Layer
- Linter: Biome — muss vor erstem Code-Commit feststehen

**Important Decisions (Shape Architecture):**
- Environment Validation: Zod Env Schema — Fail-fast bei fehlenden Vars
- Admin-Zugang: Hardcoded Wallet-Adresse — MVP-Scope

**Deferred Decisions (Post-MVP):**
- API Versioning Strategy (erst relevant bei Public API in Phase 3)
- Payment Integration (manuelles Onboarding für erste Pro-User)
- Multi-Tenant Isolation (Thomas-Persona, Phase 2)
- Custom Admin-Dashboard (existierende Tools reichen für MVP)

### Data Architecture

| Entscheidung | Wahl | Version | Rationale |
|---|---|---|---|
| **Database** | NeonDB PostgreSQL + TimescaleDB | PG 16 | Bereits im masemIT-Stack, Serverless-kompatibel, Hypertables für Zeitreihen |
| **ORM** | Drizzle ORM | Latest | 90% kleinere Bundles als Prisma, <500ms Cold Starts, SQL-like Syntax |
| **Data Validation** | Zod | v4.3 | TypeScript-first, Runtime-Validierung für externe Chain-API-Daten, Drizzle-Zod-Integration |
| **Environment Validation** | Zod Env Schema (`src/lib/env.ts`) | - | Fail-fast bei fehlenden Vars, kein Extra-Paket (T3 Env nicht nötig), Trennung Server/Client Vars im Schema |
| **Caching** | 3-Layer (RSC + Upstash Redis + NeonDB) | - | RSC revalidate für initiale Loads, Redis für API-Responses, DB für Pre-computed Aggregates |
| **Migration** | drizzle-kit | Latest | `generate` lokal → commit → `migrate` in Build, NIEMALS `push` in Production |

**Zod-Einsatzgebiete:**
- Chain-Adapter-Response-Validierung (externe API-Daten)
- API Request/Response Schemas
- Environment Variable Validation (`src/lib/env.ts`)
- Form-Input-Validierung (Calculator, Alert-Setup)
- Drizzle-Zod-Integration für DB-Schema-basierte Validierung

### Authentication & Security

| Entscheidung | Wahl | Rationale |
|---|---|---|
| **Primary Auth** | SIWE via RainbowKit + NextAuth | Web3-nativer Login, De-facto-Standard |
| **Secondary Auth** | Email via MMS | Prosumer/B2B-Pfad, Classic Login |
| **Session** | JWT (NextAuth) | Serverless-kompatibel, kein Session-Store nötig |
| **Admin-Zugang** | Hardcoded Wallet-Adresse in Env-Var | MVP: Solo-Betrieb, Sempre's ETH-Wallet = Admin-Check in Middleware |
| **API Security** | Upstash Rate Limit + HMAC auf Cron-Endpoints | Rate Limiting auf Public Endpoints, QStash Signature Verification auf Cron-Webhooks |
| **Bot Protection** | Cloudflare Turnstile | Signup/Login-Schutz |

**Admin-Implementierung:**
- `ADMIN_WALLET_ADDRESS=0x...` in Vercel Env Vars
- Middleware prüft: `session.wallet === process.env.ADMIN_WALLET_ADDRESS` → role: admin

### API & Communication Patterns

| Entscheidung | Wahl | Rationale |
|---|---|---|
| **API Style** | REST (Next.js API Routes) | Simpel, kein tRPC-Overhead, ausreichend für MVP |
| **Server State** | TanStack Query v5.90 | RainbowKit bringt es als Dependency mit (kein Extra-Bundle), Caching, Retry, Suspense, Background-Refetching |
| **Error Format** | Standardisiertes JSON `{ error: string, code: string, details?: unknown }` | Konsistent über alle API Routes |
| **Cron Communication** | QStash Webhooks → `/api/cron/*` | Fan-out für parallele Chain-Fetcher, At-least-once Delivery, Retry |

**Data-Flow-Pattern:**
- RSC (Initial Load) → Server-Rendered Dashboard
- TanStack Query (Client) → Chain-Filter-Wechsel, Pull-to-Refresh, Optimistic UI
- QStash Cron → Chain Adapters → NeonDB → Cache Invalidation → Fresh Data

### Frontend Architecture

| Entscheidung | Wahl | Rationale |
|---|---|---|
| **Server State** | TanStack Query v5.90 | Caching, Refetching, Suspense-Support |
| **Client State** | React Context (minimal) | Nur für Theme-Toggle, Sidebar-State — kein globaler State-Store nötig |
| **Components** | RSC Default, Client Components nur wenn nötig | Minimaler Client-JS-Bundle |
| **UI Library** | Tremor + Custom Tailwind | Dashboard-optimiert, Dark-Mode-nativ |
| **Forms** | Native HTML + Zod Validation | Calculator und Alert-Setup sind simple Formulare, kein React Hook Form nötig |
| **Linter/Formatter** | Biome | Rust-basiert (schnell), Linting + Formatting in einem Tool, native create-next-app-Unterstützung |

### Infrastructure & Deployment

| Entscheidung | Wahl | Rationale |
|---|---|---|
| **Hosting** | Vercel Pro | Bereits im masemIT-Stack, Auto-Deploy, Preview Deployments |
| **CI/CD** | Vercel Git Integration | Push to main → Production, PR → Preview + NeonDB Branch |
| **Environment** | Vercel Environment Variables | Server/Client-Trennung, Preview/Production getrennt |
| **Monitoring** | Vercel Logs + QStash Dashboard | MVP-ausreichend, kein Custom-Monitoring nötig |
| **Analytics** | masemIT Analytics (Client-Side) | Bereits im Stack, Feature-Usage-Tracking |
| **Error Tracking** | Vercel Logs | MVP-ausreichend, Sentry evaluieren Post-Launch bei Bedarf |

### Decision Impact Analysis

**Implementation Sequence:**
1. Biome-Config + TypeScript strict → Code-Qualität von Tag 1
2. Drizzle Schema + NeonDB + TimescaleDB → Daten-Fundament
3. Zod Env Schema → Fail-fast Environment Validation
4. NextAuth + SIWE + RainbowKit → Auth-Layer
5. Chain Adapters + Zod Validation → Daten-Ingestion
6. TanStack Query Setup → Client-Side Data Layer
7. Tremor + Tailwind Dark Theme → UI-Foundation
8. QStash Cron Pipeline → Automatische Datenaktualisierung
9. VRS Engine + Yield Calculator → Domain-Logik
10. Alert Engine + Resend → Notifications

**Cross-Component Dependencies:**
- Zod ist Querschnittsbibliothek: Chain Adapters, API Routes, Env Config, Forms
- TanStack Query wird von RainbowKit mitgebracht — keine separate Installation nötig
- Drizzle-Zod-Bridge verbindet DB-Schema mit Validierung
- Admin-Check in Auth-Middleware nutzt gleichen JWT-Flow wie User-Auth

## Implementation Patterns & Consistency Rules

### Pattern Categories Defined

**18 Conflict Points identifiziert** — Bereiche, in denen AI Agents unterschiedliche Entscheidungen treffen könnten. Alle sind nachfolgend mit verbindlichen Patterns aufgelöst.

### Naming Patterns

**Database Naming (Drizzle Schema):**

| Element | Konvention | Beispiel |
|---|---|---|
| Tabellen | snake_case, Plural | `validators`, `yield_snapshots`, `vrs_scores` |
| Spalten | snake_case | `wallet_address`, `created_at`, `inflation_rate` |
| Foreign Keys | `{referenzierte_tabelle_singular}_id` | `validator_id`, `user_id` |
| Indizes | `idx_{tabelle}_{spalten}` | `idx_validators_chain_address` |
| Enums | snake_case | `chain_type`, `alert_severity` |

**API Naming:**

| Element | Konvention | Beispiel |
|---|---|---|
| Endpoints | Plural, kebab-case | `/api/validators`, `/api/yield-snapshots` |
| Route Params | camelCase | `/api/validators/[validatorId]` |
| Query Params | camelCase | `?chainId=eth&sortBy=vrsScore` |
| Cron Endpoints | `/api/cron/{action}` | `/api/cron/fetch-validators`, `/api/cron/calculate-vrs` |

**Code Naming:**

| Element | Konvention | Beispiel |
|---|---|---|
| Dateien | kebab-case | `vrs-calculator.ts`, `eth-adapter.ts` |
| Komponenten-Dateien | PascalCase | `ValidatorCard.tsx`, `YieldChart.tsx` |
| Funktionen | camelCase | `calculateRealYield()`, `fetchValidators()` |
| Variablen | camelCase | `vrScore`, `inflationRate` |
| Konstanten | SCREAMING_SNAKE_CASE | `VRS_WEIGHTS`, `CACHE_TTL_VALIDATORS` |
| Types/Interfaces | PascalCase, kein `I`-Prefix | `Validator`, `YieldSnapshot`, `VrsScore` |
| Zod Schemas | camelCase + `Schema` Suffix | `validatorSchema`, `yieldSnapshotSchema` |
| Enums (TS) | PascalCase + PascalCase Values | `Chain.Ethereum`, `AlertSeverity.Critical` |
| Exports | Named Exports | Kein `default export` außer `page.tsx`, `layout.tsx`, `route.ts` |
| Imports | Absolute Paths | `@/domain/scoring/vrs-calculator` |

### Structure Patterns

**Domain Layer Organisation:**

```
src/domain/chain-adapters/
├── types.ts              # ChainAdapter Interface + Unified Models
├── ethereum/
│   ├── eth-adapter.ts
│   ├── eth-adapter.test.ts
│   └── eth-mapper.ts     # Raw → Unified Mapping
├── solana/
│   └── ...
└── cosmos/
    └── ...
```

**Infrastructure Layer:**

```
src/infrastructure/db/
├── schema.ts             # Alle Drizzle Tabellen-Definitionen
├── schema.test.ts        # Schema-Validierungs-Tests
├── repositories/
│   ├── validator-repository.ts
│   ├── yield-repository.ts
│   └── vrs-repository.ts
└── migrations/           # Generiert von drizzle-kit
```

**Shared Types:** `src/domain/models/` für domänenübergreifende Types (`Chain`, `Tier`, `UnifiedValidator`)

**Config Files:** Root-Level (`drizzle.config.ts`, `biome.json`, `vitest.config.ts`)

**Tests:** Co-located neben Source-Dateien (`vrs-calculator.ts` → `vrs-calculator.test.ts`)

### Format Patterns

**API Response (Success):**

```typescript
// Einzelnes Objekt
{ data: Validator }

// Liste
{ data: Validator[], meta: { count: number, chain: string, freshness: string } }

// Leere Liste
{ data: [], meta: { count: 0 } }
```

**API Response (Error):**

```typescript
{ error: string, code: string, details?: unknown }
```

**JSON Field Naming:** camelCase in API Responses (Drizzle mappt automatisch von snake_case DB-Spalten)

**Date Format:** ISO 8601 Strings in API (`"2026-02-19T14:30:00Z"`), `Date` Objekte intern

**Null Handling:** `null` für fehlende optionale Werte, NIEMALS `undefined` in API Responses

### Communication Patterns

**Cache Events (Redis Key-Invalidierung):**

```typescript
// Pattern: staketrack:{entity}:{chain}:{scope}
await invalidateCache('staketrack:validators:eth:all')
await invalidateCache('staketrack:vrs:eth:*')  // Wildcard für Chain
```

**Logging (Vercel Logs):**

| Level | Verwendung | Beispiel |
|---|---|---|
| `console.error` | Fehler, die Aktion erfordern | Chain API timeout, DB connection failed |
| `console.warn` | Degraded aber funktional | Stale data served, cache miss |
| `console.info` | Wichtige Business Events | Cron completed, VRS recalculated |
| `console.log` | NUR in Development | Debug-Daten |

**Logging-Format:** `[{component}] {message}` — z.B. `[eth-adapter] Fetched 500 validators in 1.2s`

**DSGVO-Regel:** KEINE Wallet-Adressen in Logs (nur gekürzt: `0x1234...abcd`)

### Process Patterns

**Loading States (RSC + Client):**

```typescript
// RSC: Suspense Boundary mit Tremor Skeleton
<Suspense fallback={<ValidatorTableSkeleton />}>
  <ValidatorTable chain={chain} />
</Suspense>

// Client (TanStack Query): isLoading, isError, data Pattern
const { data, isLoading, isError, error } = useValidators(chain)
```

**Error Recovery:**

| Szenario | Verhalten |
|---|---|
| Chain API timeout | Letzte bekannte Daten + Stale-Banner mit Timestamp |
| DB connection fail | 503 + Retry-After Header |
| Auth failure | Redirect zu Login |
| Validation error (Zod) | 400 + Zod-Error-Details im `details` Feld |
| Rate Limit | 429 + Retry-After Header |

**Retry Pattern:** QStash eigenes Retry (3x exponential backoff) — KEIN manuelles Retry in Cron-Handlers

**Validation Timing:**

| Stelle | Methode | Zeitpunkt |
|---|---|---|
| API Input | Zod Schema | Am Anfang des Route Handlers (fail-fast) |
| Chain API Response | Zod Schema | Direkt nach Fetch im Adapter |
| Form Input | Zod Schema | onSubmit (nicht onChange) |
| Environment Vars | Zod Schema | App-Start (`src/lib/env.ts`) |

### Enforcement Guidelines

**Alle AI Agents MÜSSEN:**

1. Dieses Dokument VOR jeder Code-Implementierung lesen
2. snake_case für DB, camelCase für TypeScript/JSON, kebab-case für Dateien
3. API Responses IMMER im `{ data, meta? }` Wrapper (Success) bzw. `{ error, code, details? }` (Error)
4. Keine Wallet-Adressen in Logs (nur gekürzt)
5. Tests co-located, Domain-Layer komplett framework-agnostisch
6. Zod-Validierung an allen System-Grenzen (API Input, Chain API Output, Env Vars)
7. Named Exports (kein `default export` außer Next.js-Konventionen)

**Anti-Patterns (VERBOTEN):**

| Anti-Pattern | Korrekt |
|---|---|
| `interface IValidator` | `interface Validator` |
| `export default function` (in Nicht-Next.js-Dateien) | `export function` |
| `console.log` in Production Code | `console.info` / `console.error` |
| Wallet-Adresse `0xABC123...` in Logs | `0xABC1...3456` (gekürzt) |
| `undefined` in API Response | `null` |
| manuelles Retry in Cron-Handler | QStash Retry nutzen |
| chain-spezifische Logik außerhalb Adapter | Immer im Chain Adapter kapseln |

## Project Structure & Boundaries

### Requirements → Structure Mapping

| FR-Kategorie | Architekturkomponente | Verzeichnis |
|---|---|---|
| **Wallet & Identity** (FR1-FR7) | Auth Layer, User Profile | `src/app/auth/`, `src/lib/auth/`, `src/infrastructure/mms/` |
| **Dashboard & Portfolio** (FR8-FR14) | Dashboard UI, Portfolio Aggregation | `src/app/(dashboard)/`, `src/domain/portfolio/` |
| **Real-Yield-Kalkulation** (FR15-FR20) | Yield Calculator, Public Calculator | `src/domain/yield/`, `src/app/(public)/calculator/` |
| **Validator Risk Score** (FR21-FR26) | VRS Engine | `src/domain/scoring/`, `src/app/api/vrs/` |
| **Alert-System** (FR27-FR32) | Alert Engine, Email Templates | `src/domain/alerts/`, `src/infrastructure/email/` |
| **Daten-Pipeline** (FR33-FR42) | Chain Adapters, Cron Pipeline, Caching | `src/domain/chain-adapters/`, `src/app/api/cron/`, `src/infrastructure/cache/` |

### Complete Project Directory Structure

```
staketrack-ai/
├── .env.example                          # Template für Environment Variables
├── .env.local                            # Lokale Env Vars (gitignored)
├── .gitignore
├── biome.json                            # Biome Linter + Formatter Config
├── drizzle.config.ts                     # Drizzle Kit Config (DB Connection, Migration Output)
├── next.config.ts                        # Next.js Config (Images, Redirects, Headers)
├── package.json
├── package-lock.json
├── postcss.config.js                     # PostCSS für Tailwind
├── tailwind.config.ts                    # Tailwind + Tremor Theme Extension
├── tsconfig.json                         # TypeScript strict, @/ Alias
├── vitest.config.ts                      # Vitest Config (Setup, Coverage)
├── playwright.config.ts                  # Playwright E2E Config
│
├── public/
│   ├── favicon.ico
│   ├── chains/                           # Chain-Logos (eth.svg, sol.svg, atom.svg)
│   └── og/                               # Open Graph Images
│
├── drizzle/                              # Generierte Migrations (drizzle-kit generate)
│   └── migrations/
│       └── 0000_initial.sql
│
├── e2e/                                  # Playwright E2E Tests (max 10 Critical Flows)
│   ├── dashboard-load.spec.ts
│   ├── wallet-connect.spec.ts
│   ├── calculator-public.spec.ts
│   └── fixtures/
│       └── test-wallets.ts
│
├── src/
│   ├── app/                              # Next.js App Router — UI + API Layer
│   │   ├── globals.css                   # Tailwind Directives + Tremor Theme
│   │   ├── layout.tsx                    # Root Layout (Providers, Fonts, Analytics)
│   │   ├── not-found.tsx                 # 404 Page
│   │   │
│   │   ├── (public)/                     # Route Group: Unauthentifizierte Pages
│   │   │   ├── page.tsx                  # Landing Page
│   │   │   └── calculator/
│   │   │       └── page.tsx              # Public Real-Yield Calculator (FR18)
│   │   │
│   │   ├── (dashboard)/                  # Route Group: Authentifiziertes Dashboard
│   │   │   ├── layout.tsx                # Dashboard Layout (Sidebar, Header)
│   │   │   ├── dashboard/
│   │   │   │   └── page.tsx              # Portfolio Overview (FR8-FR14)
│   │   │   ├── validators/
│   │   │   │   ├── page.tsx              # Validator-Liste mit VRS (FR21-FR23)
│   │   │   │   └── [validatorId]/
│   │   │   │       └── page.tsx          # Validator-Detail + VRS-Historie (FR24-FR26)
│   │   │   ├── yield/
│   │   │   │   └── page.tsx              # Yield-Übersicht + Calculator (FR15-FR17)
│   │   │   ├── alerts/
│   │   │   │   └── page.tsx              # Alert-Management (FR27-FR30)
│   │   │   └── settings/
│   │   │       └── page.tsx              # Wallet-Linking, Profil, Tier (FR4-FR7)
│   │   │
│   │   ├── auth/                         # Auth Pages
│   │   │   ├── signin/
│   │   │   │   └── page.tsx              # Login (SIWE + Email)
│   │   │   └── error/
│   │   │       └── page.tsx              # Auth Error Page
│   │   │
│   │   └── api/                          # API Route Handlers (dünn — delegieren an domain/)
│   │       ├── auth/
│   │       │   └── [...nextauth]/
│   │       │       └── route.ts          # NextAuth API Routes
│   │       ├── validators/
│   │       │   ├── route.ts              # GET /api/validators?chainId=eth
│   │       │   └── [validatorId]/
│   │       │       └── route.ts          # GET /api/validators/:id
│   │       ├── vrs/
│   │       │   ├── route.ts              # GET /api/vrs?chainId=eth
│   │       │   └── [validatorId]/
│   │       │       └── route.ts          # GET /api/vrs/:validatorId (Historie)
│   │       ├── yield/
│   │       │   ├── route.ts              # GET /api/yield?chainId=eth
│   │       │   └── calculate/
│   │       │       └── route.ts          # POST /api/yield/calculate (Public Calculator)
│   │       ├── portfolio/
│   │       │   └── route.ts              # GET /api/portfolio (User's Positions)
│   │       ├── alerts/
│   │       │   ├── route.ts              # GET/POST /api/alerts
│   │       │   └── [alertId]/
│   │       │       └── route.ts          # PATCH/DELETE /api/alerts/:id
│   │       └── cron/                     # QStash Webhook Endpoints (HMAC-geschützt)
│   │           ├── fetch-validators/
│   │           │   └── route.ts          # POST — Fan-out: 3 Chain Adapters parallel
│   │           ├── calculate-vrs/
│   │           │   └── route.ts          # POST — VRS für alle Validators berechnen
│   │           ├── calculate-yield/
│   │           │   └── route.ts          # POST — Daily Yield Snapshots
│   │           └── process-alerts/
│   │               └── route.ts          # POST — Alert-Bedingungen prüfen + E-Mails
│   │
│   ├── domain/                           # Business Logic — KOMPLETT framework-agnostisch!
│   │   ├── models/                       # Shared Domain Models + Types
│   │   │   ├── chain.ts                  # Chain enum, ChainConfig
│   │   │   ├── validator.ts              # UnifiedValidator, ValidatorMetrics
│   │   │   ├── position.ts               # UnifiedPosition, StakingPosition
│   │   │   ├── yield.ts                  # YieldSnapshot, RealYieldResult
│   │   │   ├── vrs.ts                    # VrsScore, VrsFactors, VrsCategory
│   │   │   ├── alert.ts                  # Alert, AlertCondition, AlertSeverity
│   │   │   ├── user.ts                   # User, UserTier, LinkedWallet
│   │   │   └── common.ts                 # Pagination, ApiResponse, ApiError
│   │   │
│   │   ├── chain-adapters/               # Chain-spezifische Adapter (Aggregator Pattern)
│   │   │   ├── types.ts                  # ChainAdapter Interface
│   │   │   ├── ethereum/
│   │   │   │   ├── eth-adapter.ts        # ChainAdapter Implementation
│   │   │   │   ├── eth-adapter.test.ts
│   │   │   │   ├── eth-mapper.ts         # Raw Beacon API → UnifiedValidator
│   │   │   │   ├── eth-mapper.test.ts
│   │   │   │   └── eth-schemas.ts        # Zod Schemas für Beacon API Responses
│   │   │   ├── solana/
│   │   │   │   ├── sol-adapter.ts
│   │   │   │   ├── sol-adapter.test.ts
│   │   │   │   ├── sol-mapper.ts         # getAccountInfo Parsing (NICHT getStakeActivation!)
│   │   │   │   ├── sol-mapper.test.ts
│   │   │   │   └── sol-schemas.ts
│   │   │   └── cosmos/
│   │   │       ├── atom-adapter.ts
│   │   │       ├── atom-adapter.test.ts
│   │   │       ├── atom-mapper.ts
│   │   │       ├── atom-mapper.test.ts
│   │   │       └── atom-schemas.ts
│   │   │
│   │   ├── scoring/                      # VRS Engine (P0 Test Coverage)
│   │   │   ├── vrs-calculator.ts         # Haupt-Scoring-Logik (5 Faktoren)
│   │   │   ├── vrs-calculator.test.ts
│   │   │   ├── vrs-weights.ts            # Konfigurierbare Gewichtungen (KONSTANTEN)
│   │   │   ├── vrs-confidence.ts         # Confidence Level basierend auf Datenverfügbarkeit
│   │   │   └── vrs-confidence.test.ts
│   │   │
│   │   ├── yield/                        # Real-Yield Calculator (P0 Test Coverage)
│   │   │   ├── yield-calculator.ts       # Real Yield = Nominal − Inflation − Fee − Slashing
│   │   │   ├── yield-calculator.test.ts
│   │   │   └── yield-config.ts           # Konfigurierbare Parameter (SIMD-123 adaptierbar)
│   │   │
│   │   ├── alerts/                       # Alert Engine
│   │   │   ├── alert-evaluator.ts        # Alert-Bedingungen prüfen
│   │   │   ├── alert-evaluator.test.ts
│   │   │   └── alert-types.ts            # Alert-Condition-Definitions
│   │   │
│   │   └── portfolio/                    # Portfolio Aggregation
│   │       ├── portfolio-aggregator.ts   # Multi-Chain Position Aggregation
│   │       └── portfolio-aggregator.test.ts
│   │
│   ├── infrastructure/                   # External Service Adapters
│   │   ├── db/
│   │   │   ├── index.ts                  # Drizzle Client Instance (NeonDB Serverless)
│   │   │   ├── schema.ts                 # ALLE Tabellen-Definitionen
│   │   │   └── repositories/
│   │   │       ├── validator-repository.ts
│   │   │       ├── vrs-repository.ts
│   │   │       ├── yield-repository.ts
│   │   │       ├── alert-repository.ts
│   │   │       ├── user-repository.ts
│   │   │       └── portfolio-repository.ts
│   │   │
│   │   ├── cache/
│   │   │   ├── redis-client.ts           # Upstash Redis Instance
│   │   │   ├── cache-keys.ts             # Cache-Key-Patterns (staketrack:{entity}:{chain}:{id})
│   │   │   └── cache-utils.ts            # get/set/invalidate Helpers
│   │   │
│   │   ├── queue/
│   │   │   ├── qstash-client.ts          # QStash Instance
│   │   │   └── qstash-verify.ts          # HMAC Signature Verification Middleware
│   │   │
│   │   ├── email/
│   │   │   ├── resend-client.ts          # Resend Instance
│   │   │   └── templates/
│   │   │       ├── vrs-alert.tsx          # React Email Template: VRS Change
│   │   │       ├── yield-alert.tsx        # React Email Template: Yield Drop
│   │   │       └── onboarding.tsx         # React Email Template: Welcome
│   │   │
│   │   └── mms/
│   │       ├── mms-client.ts             # masemIT MicroServices Client (api.masem.at)
│   │       └── mms-types.ts              # MMS API Types (User/Party/Consent)
│   │
│   ├── components/                       # React Komponenten (UI Layer)
│   │   ├── ui/                           # Basis-Komponenten (Tremor Wrappers + Custom)
│   │   │   ├── Skeleton.tsx              # Loading Skeleton
│   │   │   ├── StaleDataBanner.tsx       # Freshness-Warning Banner
│   │   │   ├── EmptyState.tsx            # Empty State Component
│   │   │   └── ErrorBoundary.tsx         # Error Boundary Wrapper
│   │   │
│   │   ├── dashboard/                    # Dashboard-spezifische Komponenten
│   │   │   ├── PortfolioSummary.tsx      # Portfolio-Übersicht Card
│   │   │   ├── ChainSelector.tsx         # Chain-Filter (ETH/SOL/ATOM)
│   │   │   ├── ValidatorTable.tsx        # Validator-Liste mit VRS-Badge
│   │   │   └── YieldChart.tsx            # Yield-Trend-Chart (Tremor)
│   │   │
│   │   ├── validators/                   # Validator-Detail-Komponenten
│   │   │   ├── VrsScoreCard.tsx          # VRS Score + Kategorie (Grün/Gelb/Rot)
│   │   │   ├── VrsHistoryChart.tsx       # VRS-Verlauf-Chart
│   │   │   └── ValidatorMetrics.tsx      # Validator-Metriken-Grid
│   │   │
│   │   ├── calculator/                   # Yield Calculator Komponenten
│   │   │   ├── YieldCalculatorForm.tsx   # Calculator-Formular
│   │   │   └── YieldResultCard.tsx       # Ergebnis-Anzeige
│   │   │
│   │   ├── alerts/                       # Alert-Management Komponenten
│   │   │   ├── AlertList.tsx             # Alert-Übersicht
│   │   │   └── AlertSetupForm.tsx        # Alert erstellen/bearbeiten
│   │   │
│   │   ├── auth/                         # Auth-Komponenten
│   │   │   ├── ConnectWalletButton.tsx   # RainbowKit ConnectButton Wrapper
│   │   │   └── AuthGuard.tsx             # Client-Side Auth Check
│   │   │
│   │   └── layout/                       # Layout-Komponenten
│   │       ├── Sidebar.tsx               # Dashboard Sidebar Navigation
│   │       ├── Header.tsx                # Top Header (User, Theme Toggle)
│   │       └── Footer.tsx                # Footer
│   │
│   ├── lib/                              # Shared Utilities
│   │   ├── env.ts                        # Zod Environment Validation (Server + Client)
│   │   ├── auth/
│   │   │   ├── auth-options.ts           # NextAuth Config (SIWE + Email Provider)
│   │   │   ├── session.ts               # Session Helpers (getServerSession Wrapper)
│   │   │   └── middleware.ts             # Auth + Rate Limit + Tier-Check Middleware
│   │   ├── utils/
│   │   │   ├── format.ts                # Formatting Helpers (Zahlen, Prozent, Adressen)
│   │   │   ├── date.ts                  # Date Helpers (ISO, relative Time)
│   │   │   └── chain.ts                 # Chain-spezifische Utility-Funktionen
│   │   ├── api/
│   │   │   ├── response.ts              # API Response Wrapper ({ data, meta } / { error, code })
│   │   │   └── validation.ts            # API Input Validation Helpers
│   │   └── hooks/                       # React Custom Hooks (Client-Side)
│   │       ├── use-validators.ts        # TanStack Query: Validators
│   │       ├── use-vrs.ts               # TanStack Query: VRS Scores
│   │       ├── use-yield.ts             # TanStack Query: Yield Data
│   │       └── use-portfolio.ts         # TanStack Query: Portfolio
│   │
│   ├── providers/                       # React Context Providers
│   │   ├── theme-provider.tsx           # Dark/Light Mode Toggle
│   │   ├── wallet-provider.tsx          # RainbowKit + wagmi Config
│   │   └── query-provider.tsx           # TanStack Query Client
│   │
│   └── middleware.ts                    # Next.js Middleware (Auth, Rate Limit, Tier-Check)
│
└── msw/                                 # Mock Service Worker (API-Mocking für Tests)
    ├── handlers/
    │   ├── validator-handlers.ts
    │   ├── vrs-handlers.ts
    │   └── yield-handlers.ts
    └── server.ts                        # MSW Server Setup
```

### Architectural Boundaries

**Layer-Regeln (DDD-Enforcement):**

```
┌─────────────────────────────────────────────┐
│  src/app/         UI + API Layer            │
│  (darf importieren: domain, infrastructure, │
│   components, lib, providers)               │
├─────────────────────────────────────────────┤
│  src/components/  React Komponenten         │
│  (darf importieren: domain/models, lib)     │
├─────────────────────────────────────────────┤
│  src/lib/         Shared Utilities          │
│  (darf importieren: domain/models,          │
│   infrastructure)                           │
├─────────────────────────────────────────────┤
│  src/infrastructure/  External Services     │
│  (darf importieren: domain/models)          │
│  (implementiert Interfaces aus domain/)     │
├─────────────────────────────────────────────┤
│  src/domain/      Business Logic            │
│  (DARF NUR domain/models importieren!)      │
│  (KEIN React, KEIN Next.js, KEIN infra!)   │
└─────────────────────────────────────────────┘
```

**API Boundaries:**

| Boundary | Schutz | Endpunkte |
|---|---|---|
| Public (unauthentifiziert) | Rate Limiting | `/api/yield/calculate`, Landing Page |
| Authenticated | JWT + Rate Limiting | `/api/validators`, `/api/vrs`, `/api/portfolio`, `/api/alerts` |
| Tier-Gated | JWT + Tier-Check | VRS-Detail (Pro+), Alert-Erstellung (Pro+), Multi-Chain (Pro+) |
| Admin | JWT + Wallet-Check | Zukünftige Admin-Endpoints |
| Cron (Internal) | HMAC Signature | `/api/cron/*` (nur QStash) |

**Data Flow:**

```
QStash Cron (stündlich)
    ↓ POST /api/cron/fetch-validators
    ↓ Fan-out: 3x parallel
    ├── eth-adapter.fetchValidators() → NeonDB → Redis invalidate
    ├── sol-adapter.fetchValidators() → NeonDB → Redis invalidate
    └── atom-adapter.fetchValidators() → NeonDB → Redis invalidate

QStash Cron (täglich)
    ↓ POST /api/cron/calculate-vrs
    ↓ vrs-calculator.calculate() → NeonDB (vrs_scores Hypertable)
    ↓ POST /api/cron/calculate-yield
    ↓ yield-calculator.calculate() → NeonDB (yield_snapshots Hypertable)
    ↓ POST /api/cron/process-alerts
    ↓ alert-evaluator.evaluate() → Resend (E-Mails)

User Request (Dashboard)
    ↓ RSC Initial Load
    ↓ Redis Cache Check → Hit? Return cached
    ↓ Miss? → NeonDB Query → Redis Set → Return fresh
    ↓ Client: TanStack Query für Filter/Refresh
```

### External Integration Points

| Service | Client | Verzeichnis | Auth |
|---|---|---|---|
| **NeonDB** | `@neondatabase/serverless` | `src/infrastructure/db/` | Connection String (SSL) |
| **Upstash Redis** | `@upstash/redis` | `src/infrastructure/cache/` | REST Token |
| **Upstash QStash** | `@upstash/qstash` | `src/infrastructure/queue/` | Signing Key (HMAC) |
| **Resend** | `resend` | `src/infrastructure/email/` | API Key |
| **MMS** | Custom REST Client | `src/infrastructure/mms/` | API Key (Server-to-Server) |
| **Beacon API** (ETH) | `fetch` | `src/domain/chain-adapters/ethereum/` | API Key (optional) |
| **Solana RPC** | `@solana/web3.js` | `src/domain/chain-adapters/solana/` | RPC URL |
| **Cosmos REST** | `fetch` | `src/domain/chain-adapters/cosmos/` | Public API |

## Architecture Validation Results

### Coherence Validation ✅

**Decision Compatibility:**
- Next.js 15 + React 19 + TypeScript 5 — native Unterstützung, keine Konflikte
- Drizzle ORM + `@neondatabase/serverless` — offiziell unterstützt
- RainbowKit bringt TanStack Query + wagmi + viem als Dependencies mit — kein Versionskonflikt
- Upstash Redis/QStash/Ratelimit — alle für Vercel Serverless optimiert
- Tremor + Tailwind CSS — Tremor basiert auf Tailwind, kompatibel
- Biome — native `create-next-app`-Integration seit Next.js 15

**Pattern Consistency:**
- snake_case (DB) → camelCase (API/TS) Mapping via Drizzle: konsistent
- kebab-case Dateien + PascalCase Komponenten: Standard Next.js Convention
- `{ data, meta }` Response Wrapper + `{ error, code }` Error Format: einheitlich
- Co-located Tests + Domain-Layer-Isolation: kein Konflikt

**Structure Alignment:**
- DDD-Layer-Boundaries klar definiert (domain darf NUR models importieren)
- Chain Adapter Pattern in eigenen Verzeichnissen isoliert
- Cron-Endpoints separat von Auth-geschützten Endpoints

### Requirements Coverage Validation ✅

**FR Coverage (42/42):**

| FR-Gruppe | Coverage | Architektonische Unterstützung |
|---|---|---|
| FR1-FR7 (Wallet & Identity) | ✅ | Auth Layer, MMS Client, User Repository |
| FR8-FR14 (Dashboard) | ✅ | RSC Pages, TanStack Query, Portfolio Aggregator |
| FR15-FR20 (Yield) | ✅ | Yield Calculator (domain), Public Calculator Page, Daily Snapshots |
| FR21-FR26 (VRS) | ✅ | VRS Engine (domain), Hypertable Historie, Tier-Gating |
| FR27-FR32 (Alerts) | ✅ | Alert Evaluator (domain), Resend Templates, Cron Pipeline |
| FR33-FR42 (Pipeline) | ✅ | QStash Cron, Chain Adapters, Cache Layer, Rate Limiting |

**NFR Coverage (34/34):**

| NFR-Gruppe | Coverage | Wie adressiert |
|---|---|---|
| Performance (NFR1-7) | ✅ | RSC + 3-Layer-Cache, Vercel Edge, Bundle < 200KB durch RSC Default |
| Security (NFR8-16) | ✅ | SIWE serverseitig, HMAC auf Cron, Rate Limiting, keine PII in Logs |
| Scalability (NFR17-21) | ✅ | Serverless Auto-Scale, TimescaleDB Kompression, Redis Cache 80%+ |
| Reliability (NFR22-27) | ✅ | Graceful Degradation Pattern, QStash Retry, Freshness Timestamps |
| Integration (NFR28-31) | ✅ | Chain Adapter Isolation, Cache-Invalidierung, definierte Timeouts |
| Compliance (NFR32-34) | ✅ | DSGVO via MMS Consents, Disclaimer, gekürzte Wallet-Adressen in Logs |

### Gap Analysis Results

**Kritische Gaps:** Keine gefunden.

**Wichtige Gaps (nicht blockierend, bei Implementierung zu beachten):**

| Gap | Bereich | Empfehlung |
|---|---|---|
| DB Schema noch nicht definiert | Data | Wird in Story S0 (Sprint 0) erstellt, Tabellen sind als Hypertables vorgesehen |
| E-Mail-Template-Design | Infrastructure | React Email Templates im Verzeichnis vorgesehen, Inhalt folgt in Alert-Stories |
| Solana `@solana/web3.js` vs. direkter RPC | Chain Adapters | Adapter-Interface abstrahiert dies — Entscheidung kann pro Adapter getroffen werden |

**Nice-to-Have (Post-MVP):**
- OpenAPI/Swagger-Dokumentation für API Routes
- Storybook für Tremor-Komponenten
- Sentry Integration für Production Error Tracking

### Architecture Completeness Checklist

**✅ Requirements Analysis**

- [x] 42 FRs in 6 Kategorien analysiert
- [x] 34 NFRs in 6 Kategorien analysiert
- [x] Scale & Complexity bewertet (Hoch)
- [x] 8 Technical Constraints identifiziert
- [x] 8 Cross-Cutting Concerns dokumentiert

**✅ Architectural Decisions**

- [x] 5 offene Entscheidungen getroffen (Zod, TanStack Query, Biome, Env Validation, Admin)
- [x] Tech-Stack vollständig spezifiziert mit Versionen
- [x] Data Architecture definiert (Drizzle + NeonDB + TimescaleDB + 3-Layer Cache)
- [x] Auth Architecture definiert (SIWE + MMS + JWT)
- [x] API Architecture definiert (REST + QStash Cron)

**✅ Implementation Patterns**

- [x] 18 Conflict Points identifiziert und aufgelöst
- [x] Naming Conventions: DB, API, Code — komplett
- [x] Format Patterns: Response Wrapper, Error Format, Dates, Null Handling
- [x] Communication: Cache Events, Logging Levels + Format
- [x] Process: Loading States, Error Recovery, Retry, Validation Timing

**✅ Project Structure**

- [x] ~80 Dateien/Verzeichnisse explizit definiert
- [x] FR → Directory Mapping für alle 6 Kategorien
- [x] Layer-Boundaries mit Import-Regeln
- [x] 8 External Integration Points dokumentiert
- [x] Data Flow Diagramm (Cron + User Request)

### Architecture Readiness Assessment

**Overall Status:** READY FOR IMPLEMENTATION

**Confidence Level:** HOCH

**Key Strengths:**
- Extrem detaillierter Project Context (47 Regeln) als Leitplanke für AI Agents
- Klare DDD-Layer-Boundaries verhindern architektonische Erosion
- Chain Adapter Pattern ermöglicht isolierte Chain-Entwicklung
- 3-Layer-Caching adressiert alle Performance-NFRs
- Scope Guard (F1-F5) verhindert Feature Creep

**Areas for Future Enhancement (Post-MVP):**
- API Versioning (Phase 3, Public API)
- Payment Integration (manuelles Onboarding erst)
- Sentry/Custom Monitoring
- OpenAPI Documentation

### Implementation Handoff

**AI Agent Guidelines:**

- Dieses Dokument + Project Context VOR jeder Code-Implementierung lesen
- Alle architektonischen Entscheidungen exakt wie dokumentiert umsetzen
- Implementation Patterns konsistent über alle Komponenten anwenden
- Projektstruktur und Layer-Boundaries respektieren
- Bei Unklarheiten: dieses Dokument als Referenz nutzen

**Erste Implementierungs-Priorität:**

```bash
npx create-next-app@latest staketrack-ai --typescript --tailwind --app --src-dir --use-npm
```

Danach: DDD-Ordnerstruktur anlegen → Biome konfigurieren → Drizzle + NeonDB Setup → Zod Env Schema
