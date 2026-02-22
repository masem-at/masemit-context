# Story 2.5: Staking Dashboard UI & Portfolio-Aggregation

Status: done

<!-- Note: Validation is optional. Run validate-create-story for quality check before dev-story. -->
<!-- Generated: 2026-02-22 | Epic: 2 | Story: 5 | Key: 2-5-staking-dashboard-ui-portfolio-aggregation -->

## Story

As a authentifizierter User,
I want alle meine Staking-Positionen ueber alle Chains auf einem Dashboard sehen,
So that ich sofort einen Ueberblick ueber mein gesamtes Staking-Portfolio habe.

## Acceptance Criteria

1. **Given** ein authentifizierter User oeffnet `/dashboard` **When** die Seite geladen wird (RSC, <500ms TTFB, NFR1) **Then** sieht er eine KPI-Row mit 3 Cards: Gesamt gestaked (Betrag), Anzahl Validators, Anzahl Chains **And** die KPI-Zahlen nutzen `kpi-large`/`kpi-medium` Font-Size mit `tabular-nums`

2. **Given** ein User hat Positionen auf mehreren Chains **When** das Dashboard geladen wird **Then** zeigt der Chain-Filter horizontale Tabs: "Alle / ETH / SOL / ATOM" (Tremor TabGroup, FR9) **And** "Alle" ist der Default-Tab **And** der aktive Tab bleibt zwischen Sessions erhalten (localStorage)

3. **Given** ein User klickt auf einen Chain-Tab **When** der Filter gewechselt wird **Then** aktualisiert sich die Validator-Liste sofort (Optimistic UI) **And** die Daten laden im Hintergrund nach (TanStack Query)

4. **Given** die Validator-Liste wird angezeigt **When** der User sie betrachtet **Then** sieht er pro Validator: Name, Chain-Badge, delegierter Betrag, Status (FR10) **And** auf Desktop: Full-width Tremor Table mit allen Spalten **And** auf Mobile: Kompakte Cards mit Name, Chain-Badge und Betrag (kein Horizontal-Scroll)

5. **Given** ein User sieht die Portfolio-Aggregation (FR8, FR11) **When** die Daten geladen sind **Then** zeigt das Dashboard den Gesamt-Portfolio-Wert und die Gesamt-Performance **And** die Aggregation beruecksichtigt alle verknuepften Wallets und Chains *(PARTIAL: Gesamt-Portfolio-Wert implementiert. Gesamt-Performance bewusst deferred → Story 3.2 Real-Yield Integration)*

6. **Given** der User nutzt Pull-to-Refresh auf Mobile **When** er nach unten zieht **Then** werden die Dashboard-Daten manuell aktualisiert

## Tasks / Subtasks

- [x] Task 1: Portfolio Aggregator Domain-Logik (AC: #5)
  - [x] 1.1: `src/domain/portfolio/portfolio-aggregator.ts` — Pure Domain-Funktion `aggregatePortfolio(positions: StakingPosition[], validators: UnifiedValidator[], linkedWallets: LinkedWalletInfo[]): PortfolioSummary` — berechnet Gesamt-gestaked, Anzahl aktive Validators, Anzahl Chains, Positionen gruppiert nach Chain
  - [x] 1.2: Typen definieren in `src/domain/models/portfolio.ts` — `PortfolioSummary`, `ChainSummary`, `PositionWithValidator` (Position + zugehoeriger Validator joined)
  - [x] 1.3: Chain-Name-Mapping implementieren: `linked_wallets` nutzt `'eth'`/`'sol'`/`'atom'`, DB-Tabellen `validators`/`staking_positions` nutzen `'ethereum'`/`'solana'`/`'cosmos'` (FIXME [M1]). Mapping-Funktion `linkedWalletChainToDbChain()` und `dbChainToLinkedWalletChain()` in `src/domain/models/chain.ts` hinzufuegen
  - [x] 1.4: `src/domain/portfolio/portfolio-aggregator.test.ts` — Unit-Tests: leeres Portfolio, eine Chain, Multi-Chain, Positionen ohne passenden Validator, doppelte Validators filtern

- [x] Task 2: API-Endpoint GET /api/portfolio (AC: #1, #5)
  - [x] 2.1: `src/app/api/portfolio/route.ts` — GET Handler:
    - Auth-Check via `getServerSession(authOptions)` → 401 bei fehlendem User
    - Repositories aufrufen: `findLinkedWalletsByUserId(userId)`, `findPositionsByUserId(userId)`, `findValidatorsByChain(chain)` fuer jede verknuepfte Chain
    - Portfolio Aggregator aufrufen mit den geladenen Daten
    - Cache-Aside: `cacheGet(stakingPositionsKey(userId))` → bei Hit return, bei Miss → DB-Query → `cacheSet(..., CACHE_TTL_STAKING_POSITIONS)`
    - Response-Format: `{ data: PortfolioSummary, meta: { count, freshness } }`
  - [x] 2.2: `src/app/api/portfolio/route.test.ts` — Tests: Auth-Check, Cache-Hit, Cache-Miss, leeres Portfolio, Multi-Chain-Aggregation

- [x] Task 3: API-Endpoint GET /api/validators (AC: #4)
  - [x] 3.1: `src/app/api/validators/route.ts` — GET Handler:
    - Auth-Check via `getServerSession(authOptions)` → 401 bei fehlendem User
    - Optional Query-Param: `?chainId=ethereum` (filtert nach Chain)
    - Wenn `chainId` angegeben: `findValidatorsByChain(chainId)`, sonst: Validators fuer alle verknuepften Chains laden
    - Nur Validators zurueckgeben, die der User auch als Staking-Positionen hat (Join ueber validator_id/address)
    - Cache-Aside: `cacheGet(validatorsKey(chain))` → DB-Fallback → `cacheSet`
    - Response: `{ data: ValidatorWithPosition[], meta: { count, chain } }`
  - [x] 3.2: `src/app/api/validators/route.test.ts` — Tests: Auth, Filter nach Chain, alle Chains, Cache-Verhalten, leere Ergebnisse

- [x] Task 4: TanStack Query Hooks (AC: #3, #6)
  - [x] 4.1: `src/lib/hooks/use-portfolio.ts` — `usePortfolio()` Hook:
    - `queryKey: ["portfolio"]`
    - `queryFn: () => fetch("/api/portfolio").then(r => r.json())`
    - `staleTime: 5 * 60 * 1000` (5 Minuten, passend zum Cache-TTL)
    - `refetchOnWindowFocus: true`
  - [x] 4.2: `src/lib/hooks/use-validators.ts` — `useValidators(chainId?: string)` Hook:
    - `queryKey: ["validators", chainId ?? "all"]`
    - `queryFn` mit optionalem `?chainId=` Query-Param
    - `staleTime: 5 * 60 * 1000`
    - `keepPreviousData: true` (fuer Optimistic UI bei Chain-Filter-Wechsel)

- [x] Task 5: Format-Utilities erweitern (AC: #1, #4)
  - [x] 5.1: `src/lib/utils/format.ts` — Neue Funktionen:
    - `formatStakedAmount(amount: string, chain: Chain): string` — Raw delegatedAmount (string, 18/9/6 Decimals je nach Chain) in lesbares Format: z.B. "32.5 ETH", "1,250.00 SOL", "500.00 ATOM"
    - `formatNumber(value: number): string` — Tausender-Separator: "1,234"
    - `formatChainBadgeLabel(chain: Chain): string` — "ETH" / "SOL" / "ATOM"
  - [x] 5.2: `src/lib/utils/format.test.ts` — Tests fuer alle neuen Funktionen, Edge-Cases (0, sehr grosse Zahlen, undefined)

- [x] Task 6: PortfolioSummary Komponente — KPI-Row (AC: #1)
  - [x] 6.1: `src/components/dashboard/PortfolioSummary.tsx` — Client Component (`'use client'`):
    - 3 Tremor `Card` Komponenten in einer responsive Grid-Row (`grid grid-cols-1 sm:grid-cols-3 gap-4`)
    - Card 1: "Gesamt gestaked" — Tremor `Metric` mit Gesamtbetrag, `tabular-nums` Font-Feature, `text-text-primary`
    - Card 2: "Validators" — Tremor `Metric` mit Anzahl aktiver Validators
    - Card 3: "Chains" — Tremor `Metric` mit Anzahl verknuepfter Chains
    - Loading State: Tremor Skeleton (3 Cards mit Placeholder)
    - Props: `data: PortfolioSummary | undefined`, `isLoading: boolean`
  - [x] 6.2: Dark-Mode Styling: Cards nutzen `bg-surface-raised` Background, `border-subtle` Border

- [x] Task 7: ValidatorTable Komponente — Desktop (AC: #4)
  - [x] 7.1: `src/components/dashboard/ValidatorTable.tsx` — Client Component (`'use client'`):
    - Tremor `Table`, `TableHead`, `TableHeaderCell`, `TableBody`, `TableRow`, `TableCell`
    - Spalten: Validator-Name, Chain (Tremor `Badge` mit Chain-Farbe), Delegierter Betrag (formatStakedAmount), Status (Badge)
    - Nur auf Desktop sichtbar (`hidden lg:block`)
    - Chain-Badge Farben: ETH → Indigo, SOL → Emerald, ATOM → Purple
    - Validator-Name: Fallback auf `truncateAddress(address)` wenn kein Name vorhanden
    - Status-Badge: "Active" → green, "Inactive" → gray, "Slashed" → red
    - Loading State: Skeleton-Rows (5 Placeholder-Rows)
    - Props: `validators: PositionWithValidator[]`, `isLoading: boolean`

- [x] Task 8: ValidatorCard Komponente — Mobile (AC: #4)
  - [x] 8.1: `src/components/dashboard/ValidatorCard.tsx` — Client Component (`'use client'`):
    - Nur auf Mobile sichtbar (`block lg:hidden`)
    - Tremor `Card` pro Validator: Name (oder gekuerzte Adresse), Chain-Badge (klein), Delegierter Betrag
    - Kompaktes Layout ohne Horizontal-Scroll
    - Vertikale Card-Liste mit `space-y-3`
    - Loading State: 3 Skeleton-Cards

- [x] Task 9: Dashboard Page zusammenbauen (AC: #1, #2, #3, #4, #5, #6)
  - [x] 9.1: `src/app/(dashboard)/page.tsx` — Umbauen:
    - RSC bleibt fuer initiales Layout, Client Components fuer interaktive Teile
    - Composition: `<PortfolioSummary>` + `<ChainFilter>` + `<ValidatorTable>` + `<ValidatorCard>`
    - Wrapper-Komponente `DashboardContent` als Client Component (`'use client'`) die `usePortfolio()` und `useValidators(selectedChain)` aufruft
    - ChainFilter onChange → `setSelectedChain` State → `useValidators` refetcht mit neuem chainId
    - Pull-to-Refresh: TanStack Query `refetch()` via Custom Pull-Down-Gesture oder simplen Refresh-Button auf Mobile
  - [x] 9.2: ChainFilter Integration: `selectedChain` State in DashboardContent, `localStorage` Persistenz fuer aktiven Tab (`staketrack:chain-filter`)

- [x] Task 10: Tests und Regressions-Check (AC: alle)
  - [x] 10.1: Alle neuen Unit-Tests ausfuehren (portfolio-aggregator, format, API-routes)
  - [x] 10.2: Alle bestehenden ~322 Tests muessen weiterhin bestehen (354 Tests total, alle bestanden)
  - [x] 10.3: Biome-Check auf alle neuen Dateien (alle bestanden)
  - [x] 10.4: TypeScript-Check — keine neuen TS-Fehler eingefuehrt (pre-existierende Fehler in anderen Dateien unberuehrt)

## Dev Notes

### Kritische Architektur-Entscheidungen

- **RSC + Client Component Split:** Die Dashboard-Page selbst ist eine RSC (fuer schnellen TTFB). Die interaktiven Teile (KPI-Cards, Validator-Table, ChainFilter) sind Client Components mit `'use client'` die TanStack Query Hooks verwenden. Das RSC-Layout rendert die Client-Component-Shell, die dann clientseitig Daten laedt.

- **Cache-Aside Pattern (aus Story 2-4):** API-Routes pruefen zuerst Redis (`cacheGet`), bei Miss → DB-Query → `cacheSet`. Bestehende Cache-Keys und TTL-Konstanten aus `src/infrastructure/cache/` verwenden. KEIN neues Caching-Pattern einfuehren.

- **Portfolio Aggregator ist DOMAIN-Logik:** Liegt in `src/domain/portfolio/`, darf NUR `domain/models/` importieren. Kein React, kein Next.js, kein Infrastructure-Import. Die Aggregation ist eine reine Funktion die Daten transformiert.

- **Chain-Name-Mismatch (FIXME [M1]):** `linked_wallets.chain` nutzt Kurzformen (`'eth'`, `'sol'`, `'atom'`), waehrend `validators.chain`, `staking_positions.chain` und `chain_metrics.chain` die Langformen verwenden (`'ethereum'`, `'solana'`, `'cosmos'`). Mapping-Funktionen in `chain.ts` hinzufuegen. NICHT die DB-Tabellen aendern — das ist ein bekanntes Issue das separat gefixt wird.

- **Nur User-relevante Validators anzeigen:** Der `/api/validators` Endpoint gibt NUR Validators zurueck, bei denen der User aktive Staking-Positionen hat. NICHT alle Validators der Chain. Join ueber `staking_positions.validator_id` → `validators.id`.

- **TanStack Query ist bereits konfiguriert:** `QueryClientProvider` existiert in `src/providers/web3-provider.tsx`. Neue Hooks funktionieren out-of-the-box. Verwende `queryKey` Arrays fuer automatisches Refetching bei Parameter-Aenderungen.

- **Tremor v3.18.7 Komponenten:** Table, Card, Metric, Badge, TabGroup/Tab sind alle verfuegbar. Dark-Mode funktioniert ueber Tailwind CSS Klassen. KEINE Tremor v4/v5 APIs verwenden — das Projekt nutzt v3.

- **Domain Layer darf KEINEN Cache importieren:** Cache-Logik gehoert in API Route Handlers, NICHT in den Portfolio Aggregator oder andere Domain-Funktionen.

### Bestehende Infrastruktur (MUSS wiederverwendet werden)

| Komponente | Datei | Status |
|---|---|---|
| Dashboard Layout | `src/app/(dashboard)/layout.tsx` | Vollstaendig (Header, Sidebar, BottomNav) |
| Dashboard Page | `src/app/(dashboard)/page.tsx` | Placeholder — MUSS ersetzt werden |
| ChainFilter | `src/components/dashboard/ChainFilter.tsx` | Existiert — nutzt `useLinkedWallets()` Hook |
| use-linked-wallets Hook | `src/lib/hooks/use-linked-wallets.ts` | Existiert — `queryKey: ["linked-wallets"]` |
| DB Schema | `src/infrastructure/db/schema.ts` | Alle Tabellen definiert (validators, staking_positions, chain_metrics, linked_wallets, users) |
| Staking-Position-Repo | `src/infrastructure/db/repositories/staking-position-repository.ts` | `findPositionsByUserId(userId)`, `findPositionsByChain(chain)` |
| Validator-Repo | `src/infrastructure/db/repositories/validator-repository.ts` | `findValidatorsByChain(chain)`, `findValidatorByAddress(chain, address)` |
| Chain-Metrics-Repo | `src/infrastructure/db/repositories/chain-metrics-repository.ts` | `findLatestByChain(chain)` |
| Linked-Wallet-Repo | `src/infrastructure/db/repositories/linked-wallet-repository.ts` | `findLinkedWalletsByUserId(userId)` |
| Cache-Utilities | `src/infrastructure/cache/` | `cacheGet`, `cacheSet`, `stakingPositionsKey`, `validatorsKey`, `CACHE_TTL_*` |
| Rate-Limiting | `src/lib/api/rate-limit.ts` | `authRateLimit`, `withRateLimit()` |
| Auth | `src/lib/auth/` | `getServerSession(authOptions)`, Session-Typen |
| Format | `src/lib/utils/format.ts` | `truncateAddress()` — ERWEITERN, nicht neu erstellen |
| Domain Models | `src/domain/models/` | `Chain`, `UnifiedValidator`, `StakingPosition`, `ChainMetrics`, `ApiResponse`, `ApiError` |
| Chain Adapter Types | `src/domain/chain-adapters/types.ts` | `ChainAdapter` Interface |
| Tremor | `@tremor/react` v3.18.7 | Installiert |
| TanStack Query | via `web3-provider.tsx` | `QueryClientProvider` konfiguriert |

### Chain-spezifische Details fuer Format-Utilities

| Chain | Native Token | Decimals | delegatedAmount Format |
|---|---|---|---|
| Ethereum | ETH | 18 | "32000000000000000000" → "32.00 ETH" |
| Solana | SOL | 9 | "1250000000000" → "1,250.00 SOL" |
| Cosmos | ATOM | 6 | "500000000" → "500.00 ATOM" |

Die `CHAIN_CONFIGS` in `src/domain/models/chain.ts` enthalten bereits `nativeToken` und `decimals` pro Chain.

### Learnings aus Story 2-4 (MUSS beachtet werden)

- **`--legacy-peer-deps` noetig:** Fuer ALLE npm installs wegen React 19 Peer-Dep-Konflikten
- **`serverEnv` statt `process.env`:** IMMER `serverEnv` aus `@/lib/env` verwenden — NIEMALS `process.env` direkt
- **Biome 2.2.0:** Alle neuen Dateien muessen Biome-Check bestehen
- **Zod 4.3.6:** v4 Syntax verwenden (nicht v3!)
- **Named Exports:** Kein `default export` ausser Next.js-Konventionen (`page.tsx`, `layout.tsx`, `route.ts`)
- **Test-Pattern:** Tests nutzen fetch-Mocking / Dependency-Injection; alle Tests co-located neben Source-Dateien
- **Domain-Layer importiert NUR aus domain/models/:** KEIN React, KEIN Next.js, KEIN infra
- **API Response Format:** `{ data, meta? }` (Success), `{ error, code, details? }` (Error)
- **Logging:** `[component] message` Format, KEINE Wallet-Adressen in Logs (nur gekuerzt)
- **Known Issue FIXME [M1]:** `linked_wallets` nutzt `'atom'`, `validators`/`staking_positions`/`chain_metrics` nutzen `'cosmos'` — Mapping-Funktionen erstellen, NICHT DB aendern
- **~322 bestehende Tests** muessen weiterhin bestehen
- **Cache ist Infrastructure, NICHT Domain:** Cache-Logik in API Route Handlers, nicht in Domain-Funktionen

### Git-Intelligence (letzte Commits)

```
355287d feat: add Cosmos chain adapter, Redis cache layer, and API rate limiting
315ae8b feat: add ETH wallet linking for email-authenticated users
030cc10 feat: add chain adapters (ETH/SOL), DB schema, fix /dashboard redirect
dca1785 fix: use RESEND_FROM_EMAIL env var instead of hardcoded staketrack.ai domain
e9e9a9c feat: implement Story 1.6 — Account-Loeschung (DSGVO)
```

Alle Chain Adapters (ETH, SOL, ATOM), DB-Schema, Cache-Layer und Rate-Limiting sind implementiert und getestet. Das Dashboard-UI ist das naechste logische Stueck.

### UX-Design-Spezifikation

**Dashboard Layout (Desktop):**
```
+------------------------------------------------+
| [Sidebar 64px] | [KPI-Row: 3 Cards]            |
|                | Gestaked | Validators | Chains  |
|                |-------------------------------|  |
|                | [Chain-Filter: Alle/ETH/SOL/ATOM]|
|                | [Validator-Table: full-width]  |
+------------------------------------------------+
```

**Dashboard Layout (Mobile):**
```
+--------------------+
| [Header]           |
| [KPI: Gestaked]    |
| [KPI: Validators]  |
| [KPI: Chains]      |
| [Chain-Tabs]       |
| [Validator-Cards]  |
| [Bottom Nav]       |
+--------------------+
```

**Design Tokens (Dark Mode):**
- `bg-base`: #0A0A0F (Page Background)
- `bg-surface`: #12121A (Card Background — Standard)
- `bg-surface-raised`: #1A1A25 (Card Background — KPI-Cards, hervorgehoben)
- `border-subtle`: #2A2A38 (Card Borders)
- `text-primary`: #F0F0F5 (Haupttext, KPI-Zahlen)
- `text-secondary`: #9898A8 (Labels, Subtexte)
- `brand-primary`: #06B6D4 (Teal — aktiver Tab, Highlights)
- Chain-Badge-Farben: ETH #6366F1 (Indigo), SOL #14F195 (Emerald), ATOM #A855F7 (Purple)

### Abgrenzung: Was diese Story NICHT macht

- **Kein Real Yield:** Real-Yield KPI und Spalte kommen in Story 3.2
- **Keine Freshness-Indicators:** Freshness-Timestamps kommen in Story 2.6
- **Keine Empty States:** Empty-State-Komponenten kommen in Story 2.6
- **Keine Graceful Degradation:** Stale-Data-Banner kommen in Story 2.6
- **Kein VRS-Score:** VRS-Badge kommt in Story 4.2
- **Kein Pull-to-Refresh Gesture:** Ein einfacher Refresh-Button genuegt fuer MVP. Native Pull-to-Refresh ist komplex und Post-MVP
- **Keine Sortierung:** Validator-Tabelle ist initial unsortiert — Sortierung kommt ggf. in einer spateren Story
- **Keine Pagination:** Bei <100 Validators pro User nicht noetig — Pagination wenn noetig Post-MVP
- **Keine neuen DB-Tabellen oder Migrationen:** Alle Tabellen existieren bereits
- **Keine neuen npm-Pakete:** Alle benoetigten Pakete (Tremor, TanStack Query, etc.) sind bereits installiert

### Project Structure Notes

- Neue Dateien in `src/domain/portfolio/` (portfolio-aggregator.ts + Test)
- Neue Dateien in `src/domain/models/` (portfolio.ts — neue Typen)
- Neue Dateien in `src/app/api/portfolio/` und `src/app/api/validators/` (Route Handlers + Tests)
- Neue Dateien in `src/lib/hooks/` (use-portfolio.ts, use-validators.ts)
- Neue Dateien in `src/components/dashboard/` (PortfolioSummary, ValidatorTable, ValidatorCard)
- Erweiterte Datei: `src/lib/utils/format.ts` (neue Format-Funktionen)
- Erweiterte Datei: `src/domain/models/chain.ts` (Chain-Mapping-Funktionen)
- Modifizierte Datei: `src/app/(dashboard)/page.tsx` (Placeholder → echtes Dashboard)
- Tests co-located neben Source-Dateien
- KEINE neuen DB-Tabellen oder Migrationen noetig
- KEINE neuen npm-Pakete noetig

### References

- [Source: _bmad-output/planning-artifacts/epics.md#Story 2.5 — Staking Dashboard UI & Portfolio-Aggregation]
- [Source: _bmad-output/planning-artifacts/epics.md#Epic 2 — FR8, FR9, FR10, FR11]
- [Source: _bmad-output/planning-artifacts/architecture.md#Frontend Architecture — RSC Default, TanStack Query]
- [Source: _bmad-output/planning-artifacts/architecture.md#Structure Patterns — Dashboard Layout]
- [Source: _bmad-output/planning-artifacts/architecture.md#API Naming — Plural, kebab-case]
- [Source: _bmad-output/planning-artifacts/architecture.md#Process Patterns — Loading States]
- [Source: _bmad-output/planning-artifacts/architecture.md#Data Flow — User Request (Dashboard)]
- [Source: _bmad-output/planning-artifacts/ux-design-specification.md#Dashboard — KPI-Row, Validator-Table, Chain-Filter]
- [Source: _bmad-output/planning-artifacts/ux-design-specification.md#Design Tokens — Dark Mode Farben]
- [Source: _bmad-output/project-context.md#Caching (Multi-Layer)]
- [Source: _bmad-output/project-context.md#Framework-Specific Rules — Domain Layer]
- [Source: _bmad-output/implementation-artifacts/2-4-upstash-redis-cache-layer-api-security.md — Learnings, Cache-Aside Pattern]
- [Source: Web Research — @tremor/react v3.18.7: Table, Card, Metric, Badge, TabGroup Komponenten verfuegbar]

## Dev Agent Record

### Agent Model Used

Claude Opus 4.6 (claude-opus-4-6)

### Debug Log References

Keine Debug-Probleme.

### Completion Notes List

- Portfolio Aggregator als reine Domain-Funktion implementiert (kein Infrastructure-Import)
- Chain-Name-Mapping (FIXME [M1]) mit `linkedWalletChainToDbChain()` und `dbChainToLinkedWalletChain()` geloest
- API-Endpoints `/api/portfolio` und `/api/validators` mit Auth, Cache-Aside und Rate-Limiting
- TanStack Query Hooks mit `staleTime: 5min`, `refetchOnWindowFocus`, `placeholderData` fuer Optimistic UI
- Format-Utilities: `formatStakedAmount()` konvertiert Raw-Amounts (18/9/6 Decimals) korrekt
- Dashboard mit RSC-Page + Client Components (DashboardContent, PortfolioSummary, ValidatorTable, ValidatorCards)
- ChainFilter mit localStorage-Persistenz (`staketrack:chain-filter`)
- Refresh-Button als MVP-Alternative zu Pull-to-Refresh Gesture
- Responsive Layout: Desktop (Tremor Table) / Mobile (Card-Liste, kein horizontal scroll)
- 354 Tests bestanden (32 neue Tests), keine Regressionen
- Biome-Check bestanden, keine neuen TS-Fehler

### Implementation Plan

1. Domain-Layer: Portfolio-Typen, Chain-Mapping, Aggregator-Funktion
2. API-Layer: Portfolio + Validators Endpoints mit Cache-Aside Pattern
3. Client-Layer: TanStack Query Hooks, UI-Komponenten, Dashboard-Assembly

### File List

**Neue Dateien:**
- src/domain/models/portfolio.ts
- src/domain/portfolio/portfolio-aggregator.ts
- src/domain/portfolio/portfolio-aggregator.test.ts
- src/app/api/portfolio/route.ts
- src/app/api/portfolio/route.test.ts
- src/app/api/validators/route.ts
- src/app/api/validators/route.test.ts
- src/lib/hooks/use-portfolio.ts
- src/lib/hooks/use-validators.ts
- src/components/dashboard/PortfolioSummary.tsx
- src/components/dashboard/ValidatorTable.tsx
- src/components/dashboard/ValidatorCard.tsx
- src/components/dashboard/DashboardContent.tsx

**Modifizierte Dateien:**
- src/domain/models/chain.ts (Chain-Mapping-Funktionen hinzugefuegt)
- src/domain/models/chain.test.ts (Tests fuer Chain-Mapping)
- src/lib/utils/format.ts (formatStakedAmount, formatNumber, formatChainBadgeLabel)
- src/lib/utils/format.test.ts (Tests fuer neue Format-Funktionen)
- src/components/dashboard/ChainFilter.tsx (localStorage-Persistenz)
- src/app/(dashboard)/page.tsx (Placeholder → DashboardContent)

## Change Log

- 2026-02-22: Story 2.5 implementiert — Staking Dashboard UI & Portfolio-Aggregation (10 Tasks, 354 Tests bestanden)
- 2026-02-22: Code Review — 8 Issues gefixed (4 HIGH, 4 MEDIUM): ChainFilter→Tremor TabGroup (H1), ValidatorCard Status-Badge Mobile (H2), Portfolio-Route validatorAddress-Bug (H3), Validators-Route doppelter DB-Call (H4), Domain-Layer Formatting (M1), AC5 PARTIAL dokumentiert (M2), DashboardContent localStorage-Init (M3), ValidatorCard Export-Name (M4). 1 LOW offen (L1: RSC Prefetch-Potenzial).
