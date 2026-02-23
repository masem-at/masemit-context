# Story 3.2: Dashboard Real-Yield Integration

Status: done

<!-- Note: Validation is optional. Run validate-create-story for quality check before dev-story. -->
<!-- Generated: 2026-02-23 | Epic: 3 | Story: 2 | Key: 3-2-dashboard-real-yield-integration -->

## Story

As a authentifizierter User,
I want den Real Yield pro Staking-Position und als Portfolio-Gesamt auf meinem Dashboard sehen,
So that ich sofort verstehe, was mein Staking wirklich einbringt.

## Acceptance Criteria

1. **Given** ein authentifizierter User oeffnet das Dashboard **When** die Daten geladen sind **Then** zeigt die KPI-Row einen zusaetzlichen/aktualisierten KPI-Card: "Portfolio Real Yield" als prominenteste Zahl (FR16) **And** positiver Real Yield wird in `status-positive` (Gruen) mit "+" Vorzeichen angezeigt **And** negativer Real Yield wird in `status-negative` (Rot) mit "-" Vorzeichen angezeigt **And** die Zahl hat 1 Dezimalstelle (z.B. +2.7%, -1.3%)

2. **Given** die Validator-Tabelle im Dashboard **When** die Daten angezeigt werden (FR15) **Then** enthaelt jede Validator-Row eine "Real Yield"-Spalte **And** der Wert ist farbcodiert (Gruen/Rot) mit Vorzeichen **And** auf Mobile: Real Yield ist in der kompakten Card sichtbar (nicht versteckt)

3. **Given** ein User will Real Yields vergleichen (FR17) **When** er das Dashboard mit mehreren Chains/Validators betrachtet **Then** zeigt ein RealYieldComparison-Element (`dashboard`-Variante, kompakt) den Vergleich: Nominal APY vs. Real Yield als zwei Balken **And** der Breakdown (Inflation, Fee, Slashing) ist per Klick/Tap erweiterbar (Progressive Disclosure) **And** auf Mobile: Breakdown oeffnet als Bottom Sheet

4. **Given** ein User wechselt den Chain-Filter **When** "Alle" ausgewaehlt ist **Then** zeigt der Portfolio Real Yield den gewichteten Durchschnitt ueber alle Chains **When** eine einzelne Chain ausgewaehlt ist **Then** zeigt er den Real Yield nur fuer diese Chain

## Tasks / Subtasks

- [x] Task 1: API-Route `/api/yield` erstellen (AC: #1, #2, #4)
  - [x] 1.1: `src/app/api/yield/route.ts` — GET, auth-gated, optional `?chainId=`
    - Auth-Check via `getServerSession` + 401 bei fehlender Session
    - Lade User's linked Wallets und deren Chains
    - Lade `findLatestByChain(chain)` aus yield-repository fuer jede relevante Chain
    - Filtere Yield-Snapshots: Nur Validators bei denen der User Positionen hat (Join mit staking_positions)
    - Berechne gewichteten Portfolio Real Yield: `sum(realYield * delegatedAmount) / sum(delegatedAmount)` ueber alle gefilterten Positionen
    - Cache-aside mit `yieldSnapshotKey(chain, 'portfolio:' + userId)`, TTL 86400s
    - Response: `{ data: { portfolioRealYield, portfolioNominalApy, yieldByValidator: [{ validatorId, chain, nominalApy, inflationRate, validatorFee, slashingRisk, realYield, confidence }] }, meta: { count, chain, freshness } }`
    - Bei `?chainId=ethereum` → nur ETH-Yield-Daten, bei keinem Parameter → alle Chains
    - API Response Format: `{ data, meta? }` (Success), `{ error, code, details? }` (Error)
    - Logging: `[yield-api] Served yield data for user (chains: X)` — KEINE Wallet-Adressen
  - [x] 1.2: `src/app/api/yield/route.test.ts` — Tests:
    - Erfolgreiche Response mit Yield-Daten
    - Auth-Check: 401 ohne Session
    - Chain-Filter funktioniert
    - Leere Yield-Daten (kein Snapshot vorhanden) → leeres Array, kein Error

- [x] Task 2: Icon-Library installieren und Bestandscode migrieren (AC: #1, #2)
  - [x] 2.1: `npm install --legacy-peer-deps lucide-react` installieren
    - Lucide ist die im Epic 3-2 geforderte dedizierte Icon-Library
    - WICHTIG: `--legacy-peer-deps` noetig wegen React 19 Peer-Dep-Konflikten
  - [x] 2.2: Bestehende Emoji-Icons in Navigation ersetzen:
    - `src/components/layout/Sidebar.tsx`: Emoji-Icons (📊, 🧮, 🔔, ⚙️) durch Lucide-Icons ersetzen (`LayoutDashboard`, `Calculator`, `Bell`, `Settings`)
    - `src/components/layout/BottomNav.tsx`: Gleiche Migration wie Sidebar
  - [x] 2.3: Bestehende Inline-SVGs in UI-Komponenten ersetzen:
    - `src/components/ui/EmptyState.tsx`: WalletIcon, SearchIcon, ChartIcon durch Lucide ersetzen (`Wallet`, `Search`, `BarChart3`)
    - `src/components/ui/StaleDataBanner.tsx`: Inline-SVG durch `AlertTriangle` oder `Clock` aus Lucide ersetzen
  - [x] 2.4: Sicherstellen dass alle bestehenden Tests weiterhin bestehen nach Icon-Migration

- [x] Task 3: TanStack Query Hook `use-yield` (AC: #1, #4)
  - [x] 3.1: `src/lib/hooks/use-yield.ts` — Custom Hook:
    - `useYield(chainId?: string)` — TanStack Query Hook
    - `queryKey: ["yield", chainId ?? "all"]`
    - `queryFn: () => fetch(\`/api/yield?chainId=\${chainId}\`).then(r => r.json())`
    - `staleTime: 5 * 60 * 1000` (5 Minuten — Dashboard Refresh-Zyklus)
    - `refetchOnWindowFocus: true`
    - `placeholderData: previousData` (Optimistic UI bei Chain-Filter-Wechsel)
    - Return-Typ: `{ portfolioRealYield, portfolioNominalApy, yieldByValidator: YieldByValidator[] }`
    - Named Export: `export function useYield(...)`

- [x] Task 4: PortfolioSummary um Real-Yield-KPI erweitern (AC: #1, #4)
  - [x] 4.1: `src/components/dashboard/PortfolioSummary.tsx` modifizieren:
    - Grid von `grid-cols-1 sm:grid-cols-3` auf `grid-cols-1 sm:grid-cols-2 lg:grid-cols-4` aendern
    - Neuen KPI-Card "Portfolio Real Yield" als ERSTE Card einfuegen (prominenteste Position, FR16)
    - Wert: `portfolioRealYield` als Prozentzahl mit 1 Dezimalstelle und Vorzeichen
    - Farbcodierung: `text-status-positive` bei >= 0, `text-status-negative` bei < 0
    - Font: `kpi-large` (32px/2rem, font-weight 700) fuer den Wert, `tabular-nums` fuer Monospace-Zahlen
    - Icon: `TrendingUp` (Lucide) bei positivem Yield, `TrendingDown` bei negativem
    - Neuer Prop: `yieldData: { portfolioRealYield?: number, portfolioNominalApy?: number }`
    - Skeleton fuer den Yield-Card wenn Daten laden
  - [x] 4.2: Confidence-Badge anzeigen wenn Yield-Daten `low` Confidence haben (subtil, nicht alarmierend)

- [x] Task 5: ValidatorTable um Real-Yield-Spalte erweitern (AC: #2)
  - [x] 5.1: `src/components/dashboard/ValidatorTable.tsx` modifizieren:
    - Neue Spalte "Real Yield" nach "Delegiert" einfuegen
    - Wert: `realYield` als Prozentzahl mit 1 Dezimalstelle und Vorzeichen
    - Farbcodierung: `text-status-positive` bei >= 0, `text-status-negative` bei < 0
    - Confidence-Indikator: Kleines `(~)` Symbol bei `low` Confidence (Tooltip-Erklaerung)
    - Spalte sortierbar machen (absteigend: hoechster Real Yield zuerst)
    - Neuer Prop: `yieldData: Map<string, YieldByValidator>` (validatorId → Yield-Daten)

- [x] Task 6: ValidatorCard um Real-Yield-Anzeige erweitern (AC: #2)
  - [x] 6.1: `src/components/dashboard/ValidatorCard.tsx` modifizieren:
    - Real Yield prominent in der Card anzeigen (nach Betrag, vor Status)
    - Gleiche Farbcodierung wie Tabelle
    - Auf Mobile MUSS Real Yield sichtbar sein (nicht versteckt, gemaess AC#2)
    - Neuer Prop: `yieldData?: YieldByValidator`

- [x] Task 7: RealYieldComparison Komponente erstellen (AC: #3)
  - [x] 7.1: `src/components/dashboard/RealYieldComparison.tsx` — Neue Komponente:
    - `dashboard`-Variante: Kompakter Vergleich Nominal APY vs. Real Yield
    - Zwei horizontale Balken: Nominal APY (grauer Balken, `bg-bg-surface-raised`) oben, Real Yield (Gruen/Rot je nach Vorzeichen) darunter
    - Balken-Breite proportional zum Wert (max-Wert = 100% Breite)
    - Labels: "Nominal APY: X.X%" und "Real Yield: X.X%"
    - Progressive Disclosure: Klick/Tap expandiert den Breakdown:
      - "- Inflation: X.X%"
      - "- Validator Fee: X.X%"
      - "- Slashing-Risiko: X.X%"
    - Animation: `transition-all duration-300` fuer das Expandieren (respektiert `prefers-reduced-motion`)
    - Props: `nominalApy: number, realYield: number, inflationRate: number, validatorFee: number, slashingRisk: number, variant: 'dashboard' | 'calculator'`
    - ARIA: `aria-expanded` fuer den Breakdown-Toggle, `aria-label` fuer Balken
  - [x] 7.2: Bottom Sheet fuer Mobile-Breakdown:
    - `src/components/ui/BottomSheet.tsx` — Wiederverwendbare Komponente:
      - `position: fixed`, `bottom: 0`, `z-50`
      - Hintergrund-Overlay (`bg-black/50`), Klick schliesst Sheet
      - Drag-Handle oben (visueller Indicator)
      - Content-Slot via `children`
      - Animation: `translate-y-full` → `translate-y-0` (300ms, respektiert `prefers-reduced-motion`)
      - Body-Scroll-Lock wenn offen
      - `lg:hidden` — nur auf Mobile sichtbar (auf Desktop: Inline-Expand)
    - Props: `isOpen: boolean, onClose: () => void, title?: string, children: ReactNode`
  - [x] 7.3: Integration: Auf Mobile oeffnet Klick auf Breakdown den BottomSheet, auf Desktop expandiert inline

- [x] Task 8: DashboardContent orchestrieren (AC: #1, #2, #3, #4)
  - [x] 8.1: `src/components/dashboard/DashboardContent.tsx` modifizieren:
    - `useYield(dbChainId)` Hook integrieren — dbChainId basiert auf selectedChain (gleiche Logik wie useValidators)
    - Yield-Daten an PortfolioSummary weiterreichen: `yieldData={{ portfolioRealYield, portfolioNominalApy }}`
    - Yield-Daten als Map aufbereiten und an ValidatorTable/ValidatorCard weiterreichen
    - Bei Chain-Filter-Wechsel: useYield re-fetcht automatisch (queryKey aendert sich)
    - RealYieldComparison-Element unterhalb der KPI-Row einfuegen (zeigt Portfolio-Gesamtvergleich)
    - Loading-State: Yield-Skeleton parallel zu Portfolio-Skeleton anzeigen

- [x] Task 9: Formatting Utility fuer Yield-Anzeige (AC: #1, #2)
  - [x] 9.1: `src/lib/utils/format.ts` erweitern:
    - `formatYieldPercent(value: number): string` — Formatiert als "+X.X%" oder "-X.X%" mit 1 Dezimalstelle und Vorzeichen
    - `formatApy(value: number): string` — Formatiert als "X.X%" ohne Vorzeichen (fuer Nominal APY)
    - KEIN neuer File — bestehende format.ts erweitern

- [x] Task 10: Tests und Regressions-Check (AC: alle)
  - [x] 10.1: Alle neuen Unit-Tests ausfuehren (yield-api route, use-yield hook, Komponenten)
  - [x] 10.2: Alle bestehenden ~442 Tests muessen weiterhin bestehen (keine Regressionen durch Icon-Migration oder Prop-Aenderungen)
  - [x] 10.3: Biome-Check auf alle neuen/modifizierten Dateien
  - [x] 10.4: TypeScript-Check — keine neuen TS-Fehler
  - [ ] 10.5: Manueller visueller Check: Dashboard mit Yield-Daten, Chain-Filter, Mobile-Ansicht

## Dev Notes

### Kritische Architektur-Entscheidungen

- **Gewichteter Portfolio Real Yield:** Der Portfolio-Gesamt-Real-Yield ist KEIN einfacher Durchschnitt, sondern ein nach `delegatedAmount` gewichteter Durchschnitt: `sum(realYield_i * delegatedAmount_i) / sum(delegatedAmount_i)`. Dadurch haben groessere Positionen mehr Einfluss auf den Portfolio-Wert.

- **Yield-Daten-Fluss:** Yield-Snapshots werden vom Cron-Job (Story 3-1) taeglich berechnet und in `yield_snapshots` gespeichert. Die neue `/api/yield` Route liest die LETZTEN Snapshots pro Validator (`findLatestByChain`) und filtert auf User-relevante Validators (ueber staking_positions Join). Es werden KEINE Live-Berechnungen im API-Request durchgefuehrt — nur gespeicherte Snapshots serviert.

- **Cache-Strategie:** Yield-Daten haben TTL 86400s (24h) im Redis-Cache, da Snapshots nur taeglich aktualisiert werden. Der Dashboard-Hook hat `staleTime: 5min` fuer TanStack Query — der Client refresht alle 5 Minuten, aber Redis liefert cached Daten.

- **Icon-Library Migration:** Die Epics-Datei fordert explizit: "Alle Icons im Dashboard MUESSEN aus einer dedizierten Icon-Library stammen (z.B. Lucide, Phosphor oder Heroicons)". Das Projekt verwendet aktuell Emoji-Icons und Inline-SVGs. Diese Story migriert auf `lucide-react` als projektweite Icon-Library. Die Migration betrifft Sidebar, BottomNav, EmptyState und StaleDataBanner.

- **RealYieldComparison — Dual Varianten:** Die Komponente wird mit `variant`-Prop gebaut (`dashboard` und `calculator`). Story 3-2 implementiert nur die `dashboard`-Variante (kompakt). Die `calculator`-Variante (mit Enthuellungs-Animation) kommt in Story 3.3b.

- **Bottom Sheet — Mobile Only:** Der BottomSheet fuer den Yield-Breakdown wird nur auf Mobile (<lg) angezeigt. Auf Desktop expandiert der Breakdown inline. Die Breakpoint-Logik folgt dem bestehenden Pattern (`lg:hidden` fuer Mobile-Only, `hidden lg:block` fuer Desktop-Only).

- **Validator-Yield Matching:** Die `/api/yield` Route muss Yield-Snapshots den User-Positionen zuordnen. Match ueber `validatorId` (UUID FK in beiden Tabellen). Validators ohne Yield-Snapshot (z.B. neu hinzugefuegt, noch kein Cron-Lauf) zeigen "-" statt einer Zahl.

### Bestehende Infrastruktur (MUSS wiederverwendet werden)

| Komponente | Datei | Verwendung |
|---|---|---|
| Yield Repository | `src/infrastructure/db/repositories/yield-repository.ts` | `findLatestByChain(chain)` — liefert letzte Snapshots pro Validator |
| Yield Domain Types | `src/domain/models/yield.ts` | `RealYieldResult`, `Confidence`, Zod Schemas |
| Yield Calculator | `src/domain/yield/yield-calculator.ts` | NICHT direkt — Berechnung passiert im Cron, nicht in der API |
| Cache Keys | `src/infrastructure/cache/cache-keys.ts` | `yieldSnapshotKey(chain, validatorId)`, `CACHE_TTL_YIELD` |
| Cache Utils | `src/infrastructure/cache/cache-utils.ts` | `cacheGet()`, `cacheSet()`, `cacheInvalidate()` |
| Portfolio Route | `src/app/api/portfolio/route.ts` | Pattern-Vorlage fuer auth-gated API mit Cache-aside |
| Validators Route | `src/app/api/validators/route.ts` | Pattern-Vorlage fuer Chain-Filter-Logik |
| use-portfolio Hook | `src/lib/hooks/use-portfolio.ts` | Pattern-Vorlage fuer TanStack Query Hook |
| use-validators Hook | `src/lib/hooks/use-validators.ts` | Pattern fuer `placeholderData: previousData` bei Filter-Wechsel |
| DashboardContent | `src/components/dashboard/DashboardContent.tsx` | Orchestrator — hier werden neue Hooks und Props integriert |
| PortfolioSummary | `src/components/dashboard/PortfolioSummary.tsx` | MUSS erweitert werden (4. KPI Card) |
| ValidatorTable | `src/components/dashboard/ValidatorTable.tsx` | MUSS erweitert werden (Real-Yield-Spalte) |
| ValidatorCard | `src/components/dashboard/ValidatorCard.tsx` | MUSS erweitert werden (Real-Yield-Anzeige) |
| FreshnessIndicator | `src/components/ui/FreshnessIndicator.tsx` | Wiederverwendbar fuer Yield-Freshness |
| Staking Positions Repo | `src/infrastructure/db/repositories/portfolio-repository.ts` | Fuer User-Validator-Mapping in /api/yield |
| Server Env | `src/lib/env.ts` | `serverEnv` statt `process.env` verwenden |
| Auth Session | `src/lib/auth/session.ts` | `getServerSession` fuer Auth-Check in API |
| Format Utils | `src/lib/utils/format.ts` | MUSS erweitert werden (formatYieldPercent, formatApy) |
| Design Tokens | `src/app/globals.css` | Custom Properties: status-positive, status-negative, bg-surface, etc. |
| Tremor Components | `@tremor/react` | Card, Metric, Table, Tab — bereits in Verwendung |

### Learnings aus Story 3-1 (MUSS beachtet werden)

- **`--legacy-peer-deps` noetig:** Fuer ALLE `npm install` wegen React 19 Peer-Dep-Konflikten
- **`serverEnv` statt `process.env`:** IMMER `serverEnv` aus `@/lib/env` verwenden
- **Biome 2.2.0:** Alle neuen Dateien muessen Biome-Check bestehen
- **Zod 4.3.6 v4 Syntax:** `z.enum()` statt `z.nativeEnum()`, `z.record(keySchema, valueSchema)` mit 2 Args
- **Named Exports:** Kein `default export` ausser Next.js-Konventionen (`page.tsx`, `layout.tsx`, `route.ts`)
- **Domain-Layer importiert NUR aus domain/models/:** KEIN React, KEIN Next.js, KEIN infra
- **API Response Format:** `{ data, meta? }` (Success), `{ error, code, details? }` (Error)
- **Logging:** `[component] message` Format, KEINE Wallet-Adressen in Logs (nur gekuerzt)
- **Known Issue FIXME [M1]:** `linked_wallets` nutzt `'atom'`, `validators`/`staking_positions`/`chain_metrics` nutzen `'cosmos'` — Mapping-Funktionen in `chain.ts`
- **~442 bestehende Tests** muessen weiterhin bestehen (Stand nach 3-1 Code Review)
- **Tremor v3.18.7:** Card, Metric, Table, Tab, Badge — in Dashboard-Komponenten aktiv genutzt
- **DB numeric Felder:** Drizzle gibt `numeric` Spalten als `string` zurueck — `Number()` oder `parseFloat()` beim Lesen verwenden
- **Design Tokens:** Dark-Mode-Tokens in globals.css: `bg-bg-base`, `bg-bg-surface`, `text-status-positive`, `text-status-negative` etc.
- **Grid-Pattern PortfolioSummary:** Aktuell `grid-cols-1 sm:grid-cols-3` — wird auf 4 Spalten erweitert
- **Responsive Pattern:** `lg` Breakpoint (1024px) trennt Mobile/Desktop. `ValidatorTable` = Desktop, `ValidatorCard` = Mobile
- **TanStack Query v5.90:** `placeholderData: previousData` fuer Optimistic UI bei Filter-Wechsel (kein `keepPreviousData` — v5 Breaking Change)
- **Keine Icon-Library installiert:** Aktuell Emoji-Icons + Inline-SVGs. Diese Story installiert `lucide-react`

### Real-Yield Anzeige-Beispiele (fuer Tests/visuelle Validierung)

```
// Portfolio Real Yield KPI Card:
// Positiv: "+2.7%"  (gruen, text-status-positive)
// Negativ: "-1.3%"  (rot, text-status-negative)
// Null:    "0.0%"   (gruen, text-status-positive)

// Validator Table Row:
// | Validator | Chain | Delegiert | Real Yield | Status | Aktualisiert |
// | Lido-42   | ETH   | 32.00 ETH | +3.2%     | Active | Vor 15 Min   |
// | Figment   | SOL   | 500 SOL   | +0.9%     | Active | Vor 22 Min   |
// | Cosmos-1  | ATOM  | 100 ATOM  | -2.8%     | Active | Vor 45 Min   |

// RealYieldComparison (dashboard-Variante):
// Nominal APY  ████████████████     7.0%
// Real Yield   █████                0.9%
//   [v Breakdown anzeigen]
//     - Inflation:      -5.3%
//     - Validator Fee:   -0.6%
//     - Slashing-Risiko: -0.2%
```

### Abgrenzung: Was diese Story NICHT macht

- **Kein Public Calculator UI:** Calculator-Page kommt in Story 3.3b
- **Keine Landing Page:** Kommt in Story 3.3a
- **Keine Calculator-Variante der RealYieldComparison:** `calculator`-Variante mit Enthuellungs-Animation kommt in 3.3b
- **Kein VRS-Score:** Kommt in Epic 4
- **Keine Alert-Integration:** Kommt in Epic 5
- **Kein Tier-Gating:** Kommt in Epic 6
- **Keine Sortierung der Validator-Tabelle nach Real Yield:** Nur die Spalte wird angezeigt — interaktive Sortierung ist Enhancement
- **Kein historischer Yield-Trend-Chart:** Kommt spaeter (VRS-Historie-Pattern aus Epic 4 wiederverwendbar)

### Neue Dateien

```
src/app/api/yield/route.ts                              # API-Route fuer Yield-Daten
src/app/api/yield/route.test.ts                         # API-Tests
src/lib/hooks/use-yield.ts                              # TanStack Query Hook
src/components/dashboard/RealYieldComparison.tsx         # Nominal vs. Real Yield Vergleich
src/components/ui/BottomSheet.tsx                        # Wiederverwendbare Bottom Sheet Komponente
```

### Modifizierte Dateien

```
package.json                                            # + lucide-react Dependency
src/components/dashboard/DashboardContent.tsx            # + useYield Hook, Yield-Daten-Weiterreichung
src/components/dashboard/PortfolioSummary.tsx            # + 4. KPI Card (Portfolio Real Yield)
src/components/dashboard/ValidatorTable.tsx              # + Real Yield Spalte
src/components/dashboard/ValidatorCard.tsx               # + Real Yield Anzeige
src/components/layout/Sidebar.tsx                        # Emoji → Lucide Icons
src/components/layout/BottomNav.tsx                      # Emoji → Lucide Icons
src/components/ui/EmptyState.tsx                         # Inline-SVG → Lucide Icons
src/components/ui/StaleDataBanner.tsx                    # Inline-SVG → Lucide Icons
src/lib/utils/format.ts                                 # + formatYieldPercent, formatApy
```

### Project Structure Notes

- Alle neuen Komponenten in `src/components/dashboard/` (konsistent mit bestehenden Dashboard-Komponenten)
- BottomSheet in `src/components/ui/` (wiederverwendbar, nicht Dashboard-spezifisch)
- API-Route in `src/app/api/yield/` (konsistent mit bestehenden API-Routes)
- Hook in `src/lib/hooks/` (konsistent mit use-portfolio, use-validators)
- KEINE neuen Domain-Layer-Dateien noetig (Yield-Logik existiert bereits aus Story 3-1)
- Tests co-located neben Source-Dateien

### References

- [Source: _bmad-output/planning-artifacts/epics.md#Story 3.2 — Dashboard Real-Yield Integration]
- [Source: _bmad-output/planning-artifacts/epics.md#Epic 3 — FR15, FR16, FR17]
- [Source: _bmad-output/planning-artifacts/architecture.md#Frontend Architecture — TanStack Query v5.90, Tremor, RSC Default]
- [Source: _bmad-output/planning-artifacts/architecture.md#Implementation Patterns — Naming, Structure, Format Patterns]
- [Source: _bmad-output/planning-artifacts/architecture.md#API Boundaries — Authenticated Endpoints]
- [Source: _bmad-output/planning-artifacts/architecture.md#Data Flow — User Request: RSC → Redis → NeonDB]
- [Source: _bmad-output/planning-artifacts/architecture.md#Project Structure — src/components/dashboard/, src/app/api/yield/]
- [Source: _bmad-output/planning-artifacts/ux-design-specification.md#Defining Experience — Real-Yield-Enthuellung]
- [Source: _bmad-output/planning-artifacts/ux-design-specification.md#Design Tokens — status-positive, status-negative, kpi-large]
- [Source: _bmad-output/planning-artifacts/ux-design-specification.md#Responsive Patterns — Desktop Table, Mobile Cards]
- [Source: _bmad-output/planning-artifacts/ux-design-specification.md#Progressive Disclosure — Real Yield → Klick → Formel-Breakdown]
- [Source: _bmad-output/planning-artifacts/ux-design-specification.md#Custom Components — Real-Yield-Schock-Balken, Bottom Sheet]
- [Source: _bmad-output/planning-artifacts/prd.md#FR15 — Real Yield pro Position]
- [Source: _bmad-output/planning-artifacts/prd.md#FR16 — Aggregierter Portfolio Real Yield]
- [Source: _bmad-output/planning-artifacts/prd.md#FR17 — Real Yield visueller Vergleich]
- [Source: _bmad-output/implementation-artifacts/3-1-real-yield-calculator-domain-logik-taegliche-snapshots.md — Vorherige Story Learnings]
- [Source: src/components/dashboard/DashboardContent.tsx — Orchestrator-Pattern, Hooks, Chain-Filter]
- [Source: src/components/dashboard/PortfolioSummary.tsx — KPI-Cards, Grid-Layout, Skeleton-Pattern]
- [Source: src/components/dashboard/ValidatorTable.tsx — Tremor Table, Desktop-Only]
- [Source: src/components/dashboard/ValidatorCard.tsx — Mobile Card, Tremor Card+Badge]
- [Source: src/app/api/portfolio/route.ts — Pattern: Auth-Check, Cache-aside, Response-Format]
- [Source: src/app/api/validators/route.ts — Pattern: Chain-Filter, User-Validator-Mapping]
- [Source: src/infrastructure/db/repositories/yield-repository.ts — findLatestByChain(chain)]
- [Source: src/infrastructure/cache/cache-keys.ts — yieldSnapshotKey(), CACHE_TTL_YIELD]
- [Source: src/lib/hooks/use-portfolio.ts — TanStack Query Pattern]
- [Source: src/lib/hooks/use-validators.ts — placeholderData Pattern bei Filter-Wechsel]

## Dev Agent Record

### Agent Model Used

Claude Opus 4.6

### Debug Log References

- Keine Debugging-Probleme aufgetreten

### Completion Notes List

- Task 1: API-Route `/api/yield` mit Auth-Check, Chain-Filter, Cache-Aside und gewichtetem Portfolio Real Yield implementiert. 5 Tests geschrieben und bestanden.
- Task 2: `lucide-react` installiert. Emoji-Icons in Sidebar/BottomNav und Inline-SVGs in EmptyState/StaleDataBanner durch Lucide-Icons ersetzt. Alle bestehenden Komponenten-Tests bestehen.
- Task 3: `useYield` TanStack Query Hook mit `placeholderData`, `staleTime: 5min`, `refetchOnWindowFocus` implementiert.
- Task 4: PortfolioSummary von 3-Spalten auf 4-Spalten Grid erweitert. Portfolio Real Yield als prominenteste KPI-Card (erste Position) mit Farbcodierung, Vorzeichen, TrendingUp/TrendingDown Icons und Skeleton-State.
- Task 5: ValidatorTable um Real-Yield-Spalte mit Farbcodierung, Vorzeichen und Confidence-Indikator (~) erweitert.
- Task 6: ValidatorCard um Real-Yield-Anzeige erweitert — prominent sichtbar auf Mobile (nicht versteckt, gemaess AC#2).
- Task 7: RealYieldComparison Komponente mit Dashboard-Variante, zwei proportionalen Balken (Nominal APY vs Real Yield), Progressive Disclosure Breakdown (Desktop: inline expand, Mobile: BottomSheet).
- Task 8: DashboardContent als Orchestrator modifiziert — useYield Hook integriert, Yield-Daten als Map an ValidatorTable/ValidatorCard und als Props an PortfolioSummary weitergereicht, RealYieldComparison eingefuegt.
- Task 9: `formatYieldPercent` (+X.X% / -X.X%) und `formatApy` (X.X%) in format.ts hinzugefuegt.
- Task 10: 447/447 Tests bestehen (5 neue Yield-API-Tests). Biome clean. Keine neuen TS-Fehler. Subtask 10.5 (visueller Check) bleibt fuer User.

### Implementation Plan

- Yield-Daten-Fluss: API -> Redis Cache (24h TTL) -> TanStack Query (5min staleTime) -> Dashboard-Komponenten
- Gewichteter Portfolio Real Yield: sum(realYield * delegatedAmount) / sum(delegatedAmount)
- Icon-Migration: Projektweiter Wechsel von Emoji/Inline-SVG zu lucide-react
- RealYieldComparison: Dual-Breakpoint-Strategie (Desktop inline, Mobile BottomSheet)

### File List

**Neue Dateien:**
- src/app/api/yield/route.ts
- src/app/api/yield/route.test.ts
- src/lib/hooks/use-yield.ts
- src/components/dashboard/RealYieldComparison.tsx
- src/components/ui/BottomSheet.tsx

**Modifizierte Dateien:**
- package.json (+ lucide-react)
- package-lock.json (+ lucide-react)
- src/domain/models/yield.ts (+ YieldByValidatorResponse, YieldApiData shared types)
- src/components/dashboard/DashboardContent.tsx (+ useYield, RealYieldComparison, Yield-Props)
- src/components/dashboard/PortfolioSummary.tsx (+ Real Yield KPI Card, 4-Spalten Grid, Confidence Badge)
- src/components/dashboard/ValidatorTable.tsx (+ Real Yield Spalte, shared type import)
- src/components/dashboard/ValidatorCard.tsx (+ Real Yield Anzeige, shared type import)
- src/components/dashboard/RealYieldComparison.tsx (Nominal APY Balkenfarbe korrigiert)
- src/components/layout/Sidebar.tsx (Emoji -> Lucide ChevronLeft/ChevronRight Icons)
- src/components/layout/BottomNav.tsx (Emoji -> Lucide Icons)
- src/components/ui/BottomSheet.tsx (+ globaler Escape-Key-Listener)
- src/components/ui/EmptyState.tsx (Inline-SVG -> Lucide Icons)
- src/components/ui/StaleDataBanner.tsx (Inline-SVG -> Lucide Icons)
- src/lib/hooks/use-yield.ts (shared type import statt lokaler Interfaces)
- src/lib/utils/format.ts (+ formatYieldPercent, formatApy)

## Change Log

- 2026-02-23: Story 3-2 implementiert — Dashboard Real-Yield Integration mit API-Route, TanStack Query Hook, KPI-Card, Validator-Tabelle/Card-Erweiterung, RealYieldComparison-Komponente, BottomSheet, Icon-Library-Migration (lucide-react), Formatting Utilities
- 2026-02-23: Code Review (adversarial) — 10 Findings (3 HIGH, 5 MEDIUM, 2 LOW), alle HIGH/MEDIUM automatisch behoben:
  - [HIGH] DRY-Verletzung behoben: YieldByValidatorResponse/YieldApiData als shared types in domain/models/yield.ts
  - [HIGH] Gewichtete Portfolio-Breakdowns (Inflation, Fee, Slashing) in API-Route korrekt berechnet
  - [HIGH] Fehlende Confidence-Badge (Task 4.2) in PortfolioSummary nachimplementiert
  - [MEDIUM] Nominal APY Balkenfarbe in RealYieldComparison von bg-bg-surface-raised zu bg-text-secondary/30 korrigiert
  - [MEDIUM] console.log → console.info in API-Route
  - [MEDIUM] BottomSheet: globaler Escape-Key-Listener hinzugefuegt
  - [MEDIUM] Zod-Validierung fuer chainId-Parameter in /api/yield hinzugefuegt
  - [MEDIUM] hasLowConfidence in YieldData-Interface von PortfolioSummary ergaenzt
  - [LOW] Sidebar: Emoji-Toggle (◀/▶) durch Lucide ChevronLeft/ChevronRight ersetzt
  - [LOW] Story-Inkonsistenz: Task 5.1 "sortierbar" vs. Abgrenzung "Keine Sortierung" — Abgrenzung gilt
  - Tests: 448/448 bestanden (+1 neuer Test fuer ungueltige chainId-Validierung)
