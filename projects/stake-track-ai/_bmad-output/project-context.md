---
project_name: 'StakeTrack AI'
user_name: 'Sempre'
date: '2026-02-18'
sections_completed: ['technology_stack', 'language_rules', 'framework_rules', 'testing_rules', 'code_quality', 'workflow_rules', 'critical_rules', 'scope_guard']
existing_patterns_found: 0
status: complete
rule_count: 47
optimized_for_llm: true
---

# Project Context for AI Agents — StakeTrack AI

_Critical rules and patterns that AI agents must follow when implementing code. Focus on unobvious details that agents might otherwise miss._

---

## Scope Guard Rules (CRITICAL — READ FIRST)

**MVP Features F1-F5 sind eine HARTE GRENZE. Keine Erweiterung vor Launch.**

| Feature | Beschreibung | Status |
|---------|-------------|--------|
| **F1** | Multi-Chain Staking Dashboard (ETH, SOL, ATOM) | MVP |
| **F2** | Real-Yield-Calculator (Nominal APY − Inflation − Fees − Slashing) | MVP |
| **F3** | Validator Risk Score (VRS) Engine (5-Faktoren, 0-100) | MVP |
| **F4** | Portfolio-Aggregation (Wallet-Connect, Multi-Chain) | MVP |
| **F5** | Alert-System (VRS-Änderungen, Yield-Drops, Slashing) | MVP |

**Bei jedem Feature-Vorschlag außerhalb F1-F5:**
> ⚠️ **SCOPE GUARD:** Sempre, das klingt nach Feature Creep! Ist das in F1-F5? Wenn nein → Post-Launch-Roadmap. Du bist selbsterklärter Feature-Creep-Weltmeister — halte den MVP schlank!

**Explizit OUT OF SCOPE (Post-Launch):**
- Tax-Export / Steuerreporting
- 4. Chain (Polkadot, Avalanche, etc.)
- KI-Vorhersagen / Predictions
- Social/Community-Features
- Mobile App
- Fiat On-Ramp
- Direct Staking/Delegation (Read-Only Analytics!)

---

## Technology Stack & Versions

### Core Stack

| Technologie | Version | Paket |
|-------------|---------|-------|
| **Next.js** | 15+ | `next` (App Router ONLY, kein Pages Router) |
| **React** | 19+ | `react`, `react-dom` |
| **TypeScript** | 5+ | `typescript` (strict mode) |
| **Drizzle ORM** | Latest | `drizzle-orm`, `drizzle-orm/neon-serverless` |
| **NeonDB** | PostgreSQL 16 + TimescaleDB | `@neondatabase/serverless` |
| **Upstash Redis** | - | `@upstash/redis` |
| **Upstash QStash** | - | `@upstash/qstash` |
| **Upstash Rate Limit** | - | `@upstash/ratelimit` |

### Auth Stack

| Technologie | Paket |
|-------------|-------|
| **NextAuth.js** | `next-auth` |
| **SIWE** | `siwe` |
| **RainbowKit** | `@rainbow-me/rainbowkit` |
| **RainbowKit SIWE** | `@rainbow-me/rainbowkit-siwe-next-auth` |
| **wagmi** | `wagmi` |
| **viem** | `viem` |
| **Keplr** (Cosmos) | `@keplr-wallet/types` |
| **Solana Wallet** | `@solana/wallet-adapter-react` |

### UI Stack

| Technologie | Paket |
|-------------|-------|
| **Tremor** | `@tremor/react` (Charts, Dashboard-Komponenten) |
| **Tailwind CSS** | `tailwindcss` |

### Testing Stack

| Technologie | Paket |
|-------------|-------|
| **Vitest** | `vitest` |
| **React Testing Library** | `@testing-library/react` |
| **Playwright** | `@playwright/test` (E2E) |
| **MSW** | `msw` (Mock Service Worker für API Tests) |

### Deployment

| Service | Plan |
|---------|------|
| **Vercel** | Pro (bereits im masemIT-Stack) |
| **NeonDB** | Launch (bereits im masemIT-Stack) |
| **Upstash** | Free Tier |
| **Resend** | Pro (bereits im masemIT-Stack) |

---

## Critical Implementation Rules

### Drizzle ORM (NICHT Prisma)

- Drizzle wurde bewusst statt Prisma gewählt: 90% kleinere Bundles, <500ms Cold Starts
- Schema-Definition in `src/infrastructure/db/schema.ts`
- Migrations via `drizzle-kit` (`drizzle-kit generate`, `drizzle-kit migrate`)
- Connection IMMER über `@neondatabase/serverless` Driver (Serverless-kompatibel)
- `connection_limit=1` als Default für Serverless Functions
- NIEMALS `drizzle-kit push` in Production — nur `migrate`

### NeonDB + TimescaleDB

- TimescaleDB Extension ist in Neon verfügbar — muss aktiviert werden: `CREATE EXTENSION IF NOT EXISTS timescaledb`
- 3 Hypertables: `validator_metrics`, `vrs_scores`, `yield_snapshots`
- Chunk-Größe: 1 Woche (Metriken), 1 Monat (VRS, Yield)
- Kompression nach 30 Tagen aktivieren für historische Daten
- NeonDB Launch Plan: 0.5 GiB Storage — reicht für 12+ Monate MVP
- Preview Branches: Neon-Vercel-Integration erstellt automatisch DB-Branches pro PR

### Chain Adapter Architecture (Aggregator Pattern)

- JEDE Chain hat einen eigenen Adapter in `src/domain/chain-adapters/{chain}/`
- Adapter implementieren ein gemeinsames Interface (`ChainAdapter`)
- Raw-Daten werden im Adapter auf ein Unified Model gemappt
- NIEMALS chain-spezifische Logik außerhalb des Adapters
- Neue Chain = neuer Adapter, KEINE Änderung am Frontend oder API Layer

```typescript
// Gemeinsames Interface für alle Chain Adapter
interface ChainAdapter {
  fetchValidators(): Promise<UnifiedValidator[]>
  fetchInflationRate(): Promise<number>
  fetchStakingPositions(address: string): Promise<UnifiedPosition[]>
}
```

### VRS Scoring Engine

- 5 Faktoren mit festen Gewichtungen:
  - Uptime/Availability: 30%
  - Slashing-Historie: 25%
  - Commission-Stabilität: 20%
  - Performance-Score: 15%
  - Dezentralisierung: 10%
- Scores: 0-100, mit Kategorien: Grün (80+), Gelb (50-79), Rot (<50)
- Gewichtungen als KONFIGURIERBARE KONSTANTEN, nicht hardcoded in Berechnungslogik
- Zeitgewichteter Durchschnitt: letzte 30 Tage stärker als 90 Tage
- Confidence Level pro Score basierend auf Datenverfügbarkeit

### Real-Yield Calculator

- Formel: `Real Yield = Nominal APY − Token Inflation − Validator Fee − Slashing Risk`
- Alle Parameter als konfigurierbare Werte (NICHT hardcoded) — Solana SIMD-123 ändert Revenue-Sharing
- Chain-spezifische Inflation Rates werden täglich aktualisiert
- Ergebnisse als tägliche Snapshots in `yield_snapshots` Hypertable

### Solana-spezifische Warnung

- `getStakeActivation` ist DEPRECATED und wird in solana-core v2.0 entfernt
- Alternative: Stake-Account-Daten direkt über `getAccountInfo` parsen
- SIMD-123 (genehmigt): Revenue-Sharing mit Delegatoren ändert Yield-Berechnung

---

## Framework-Specific Rules (Next.js App Router)

### Projekt-Struktur (DDD-inspired)

```
src/
├── app/                          # Next.js App Router (UI + API)
│   ├── (dashboard)/              # Route Group: Dashboard
│   ├── api/                      # API Route Handlers
│   │   ├── validators/
│   │   ├── yield/
│   │   ├── vrs/
│   │   └── cron/                 # QStash Webhook Endpoints
│   └── auth/                     # Auth Pages
├── domain/                       # Business Logic (Framework-agnostisch!)
│   ├── chain-adapters/           # ETH, SOL, ATOM Adapters
│   ├── scoring/                  # VRS Engine
│   ├── yield/                    # Real-Yield Calculator
│   └── models/                   # Shared Domain Models
├── infrastructure/               # External Service Adapters
│   ├── db/                       # Drizzle Schema + Repositories
│   ├── cache/                    # Upstash Redis
│   ├── queue/                    # QStash
│   ├── email/                    # Resend
│   └── mms/                      # masemIT MicroServices Client
└── lib/                          # Shared Utilities
    ├── auth/                     # Auth Helpers
    └── utils/
```

### Regeln

- `domain/` Layer ist KOMPLETT framework-agnostisch — kein Next.js, kein React Import
- `domain/` darf NUR auf `domain/models/` zugreifen, NICHT auf `infrastructure/`
- `infrastructure/` implementiert Interfaces aus `domain/`
- API Routes in `app/api/` sind dünn — sie delegieren an `domain/` Layer
- React Server Components (RSC) als Default — Client Components nur wenn nötig (`'use client'`)
- `revalidate` Zeiten: 300s (Dashboard), 3600s (VRS), 86400s (Yield Snapshots)

### Cron/Background Jobs

- QStash Cron Webhooks an `/api/cron/*` Endpoints
- QStash Signature Verification auf ALLEN Cron-Endpoints (HMAC)
- Vercel Pro Timeout: 60 Sekunden — Batch-Verarbeitung bei großen Datenmengen
- QStash Fan-out: Alle 3 Chain-Fetcher PARALLEL auslösen, nicht sequentiell

### Caching (Multi-Layer)

```
Layer 1: RSC (automatisch, revalidate-basiert)
Layer 2: Upstash Redis (Cache-Aside, TTL: 5min Validators, 1h VRS, 24h Yield)
Layer 3: NeonDB (Pre-computed Aggregates, Materialized Views)
```

- Cache-Key-Pattern: `staketrack:{entity}:{chain}:{id}` (z.B. `staketrack:validators:eth:all`)
- Cache IMMER invalidieren wenn neue Daten via Cron geschrieben werden

---

## Authentication Rules

- Dual-Auth: SIWE (Wallet-Login) + Classic (Email via MMS)
- RainbowKit `<ConnectButton>` für Wallet-UI
- `@rainbow-me/rainbowkit-siwe-next-auth` für SIWE + NextAuth Integration
- JWT-Session: `sub` = wallet_address
- Unified User Profile: Ein User kann MEHRERE Wallets (verschiedene Chains) verknüpfen
- MMS (api.masem.at): User/Party anlegen bei erstem Login
- NIEMALS Wallet Private Keys speichern — StakeTrack ist READ-ONLY Analytics

---

## Testing Rules

### Prioritäten

| Prio | Test-Bereich | Tool |
|------|-------------|------|
| **P0** | VRS-Score-Berechnung | Vitest Unit |
| **P0** | Real-Yield-Calculator | Vitest Unit |
| **P1** | Chain Adapter Mapping | Vitest Unit |
| **P1** | API Route Handlers | Vitest + MSW |
| **P2** | Dashboard Load | Playwright E2E |
| **P2** | Wallet Connect Flow | Playwright E2E |

### Regeln

- `domain/` Layer hat > 80% Test Coverage (VRS, Yield, Adapters)
- Tests für `domain/` NIEMALS externe Services aufrufen — alles gemocked
- MSW für API-Mocking in Integration Tests
- E2E Tests nur für Critical User Flows (max 10)
- Test-Datei neben Source-Datei: `vrs-calculator.ts` → `vrs-calculator.test.ts`

---

## Code Quality & Style Rules

- Biome ODER ESLint + Prettier (Entscheidung bei Sprint 0)
- TypeScript strict mode: `strict: true`, `noUncheckedIndexedAccess: true`
- Keine `any` Types — wenn unvermeidbar: `unknown` + Type Guard
- Imports: Absolute Paths via `@/` Alias (`@/domain/scoring/vrs-calculator`)
- Kein `default export` außer in `page.tsx`, `layout.tsx`, `route.ts` (Next.js Konvention)
- Kebab-case für Dateinamen: `vrs-calculator.ts`, `eth-adapter.ts`
- PascalCase für Komponenten: `ValidatorCard.tsx`, `YieldChart.tsx`

---

## Development Workflow Rules

- Feature Branches: `feature/{kurze-beschreibung}`
- Commit Messages: Conventional Commits (`feat:`, `fix:`, `chore:`, `docs:`)
- Vercel Preview: Jeder PR bekommt automatisch Preview + NeonDB Branch
- Production: Nur via merge to `main` — Vercel Auto-Deploy
- Environment Variables: ALLE Chain API Keys + Secrets in Vercel Env, NIEMALS im Code
- NeonDB Migrations: `drizzle-kit generate` lokal → commit → `drizzle-kit migrate` in Build

---

## masemIT Integration Rules

- **MMS** (api.masem.at): REST API für User/Party/Consents — API-Key Auth, Server-to-Server
- **masemIT Analytics**: Client-Side Tracker Script einbinden (wie bei ChainSights, PayWatcher)
- **Resend**: Alert-E-Mails über Resend API — Templates für VRS-Alerts, Yield-Alerts, Onboarding
- **Cloudflare Turnstile**: Bot-Schutz bei Signup/Login
- **Upstash Rate Limiting**: `@upstash/ratelimit` Middleware auf allen Public API Endpoints
- Pattern-Referenzen: ChainSights GVS → VRS-Algorithmus, PayWatcher API → StakeTrack API Design

### MMS Cross-Project Service Rule (CRITICAL)

> ⚠️ **PFLICHT-RÜCKSPRACHE:** Wenn bei der Implementierung ein Feature oder Service benötigt wird, der **projektübergreifend** sinnvoll wäre (z.B. User-Management, Notifications, Consent-Handling, API-Key-Management, Billing, etc.), dann:
> 1. **STOPP** — Nicht lokal in StakeTrack AI implementieren
> 2. **Sempre fragen:** "Soll das in MMS als zentraler Service abgebildet werden?"
> 3. **Abwarten** bis Sempre entscheidet: lokal implementieren ODER in MMS organisieren
>
> MMS ist der zentrale Service-Layer für alle masemIT-Produkte. Jede Duplizierung von Cross-Cutting-Concerns über Projekte hinweg ist technische Schuld.

---

## Security Rules

- StakeTrack AI ist READ-ONLY Analytics — keine Transaktionen, keine Private Keys
- SIWE-Signatur serverseitig verifizieren (nicht nur Client-Side)
- QStash Webhook Endpoints: HMAC Signature Verification pflicht
- Rate Limiting auf allen Public API Endpoints
- Chain API Keys: NUR in Environment Variables, NIEMALS im Client-Bundle
- NeonDB: SSL Required (`sslmode=require`)
- Keine User-Daten in Logs (DSGVO) — Wallet-Adressen sind pseudo-anonym aber trotzdem vorsichtig

---

## Freemium Feature-Gating

- Middleware-basierter Tier-Check auf API Routes
- Tier-Info im JWT-Token oder per DB-Lookup (cached)
- Feature-Limits:

| Feature | Free | Pro ($19) | Business ($99) |
|---------|------|-----------|----------------|
| Chains | 1 | 3 | 3 |
| Validators tracked | 10 | 100 | Unlimited |
| VRS | Basic (3 Faktoren) | Full (5 Faktoren) | Full + Historie |
| Alerts | 0 | 5 | 25 |
| API Access | Nein | Nein | Read-only |

---

## Reference Documents

- Product Brief: `_bmad-output/planning-artifacts/product-brief-new-idea-2026-02-18.md`
- Market Research: `_bmad-output/planning-artifacts/research/market-staking-saas-research-2026-02-18.md`
- Technical Research: `_bmad-output/planning-artifacts/research/technical-staketrack-ai-mvp-research-2026-02-18.md`
- masemIT Info: `docs/_masemIT/company-info.md`

---

## Usage Guidelines

**Für AI Agents:**

- Dieses File VOR jeder Code-Implementierung lesen
- ALLE Regeln exakt befolgen — besonders Scope Guard und Chain Adapter Pattern
- Im Zweifel die restriktivere Option wählen
- Bei neuen Patterns: dieses File aktualisieren (mit User-Bestätigung)

**Für Sempre:**

- File schlank halten — nur Agent-relevante Regeln
- Aktualisieren wenn sich Tech-Stack ändert
- Nach Launch: Out-of-Scope-Liste reviewen und Roadmap-Items freigeben

Last Updated: 2026-02-19
