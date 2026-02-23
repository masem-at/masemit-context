# Story 2.6: Daten-Freshness, Graceful Degradation & Empty States

Status: done

<!-- Note: Validation is optional. Run validate-create-story for quality check before dev-story. -->
<!-- Generated: 2026-02-23 | Epic: 2 | Story: 6 | Key: 2-6-daten-freshness-graceful-degradation-empty-states -->

## Story

As a User,
I want immer wissen wie aktuell meine Daten sind und auch bei API-Ausfaellen eine nutzbare Erfahrung haben,
So that ich dem Dashboard vertrauen kann.

## Acceptance Criteria

1. **Given** das Dashboard zeigt Staking-Daten **When** die Daten angezeigt werden **Then** zeigt jeder Datenpunkt einen Freshness-Indicator (FR12): Gruener Dot + "Vor X Min" (<30 Min), Gelber Dot + "Vor X Min" (30 Min - 2h), Roter Dot + "Vor X h" (>2h) **And** der Freshness-Indicator hat `aria-label="Daten aktualisiert vor X Minuten"`

2. **Given** eine Chain-API ist temporaer nicht erreichbar **When** das Dashboard geladen wird (FR13) **Then** werden die letzten bekannten Daten fuer diese Chain angezeigt **And** ein Stale-Data-Banner erscheint: "[Chain]-Daten von vor [Zeitraum]. Aktualisierung laeuft." (status-info Farbe, nicht Alarm-Rot) **And** die anderen Chains werden NICHT beeintraechtigt (NFR28) **And** es gibt keinen Crash und keine leere Seite (NFR24)

3. **Given** ein User hat eine Wallet verbunden aber keine Staking-Positionen (FR14) **When** das Dashboard geladen wird **Then** zeigt ein freundlicher Empty State: "Keine Staking-Positionen gefunden." **And** ein Aktions-Hinweis: "Hast du auf einer anderen Chain gestaked? → SOL hinzufuegen · ATOM hinzufuegen" **And** kein Schuldzuweisungs-Text, minimales Line-Icon, monochrom

4. **Given** ein User hat keine Wallets verknuepft **When** das Dashboard geladen wird **Then** zeigt ein Empty State: "Verbinde deine Wallet und sieh dein Staking-Portfolio." **And** ein Primary Button: "Connect Wallet"

5. **Given** der Chain-Filter zeigt keine Ergebnisse nach Filterung **When** der User eine Chain ohne Positionen waehlt **Then** zeigt ein Empty State: "Keine Validators fuer [Chain] gefunden." **And** ein Link: "Alle Chains anzeigen →"

## Tasks / Subtasks

- [x] Task 1: FreshnessIndicator Komponente (AC: #1)
  - [x] 1.1: `src/components/ui/FreshnessIndicator.tsx` — Client Component (`'use client'`):
    - Props: `fetchedAt: string | Date`, `variant: 'inline' | 'banner'` (default: 'inline')
    - Berechnet relative Zeit seit `fetchedAt`
    - Inline-Variante: Farbiger Dot (8px) + relative Zeitangabe ("Vor 5 Min", "Vor 1h 23 Min", "Vor 3h")
    - Farblogik: Gruen (`bg-status-positive`) bei <30 Min, Gelb (`bg-yellow-500`) bei 30 Min-2h, Rot (`bg-status-negative`) bei >2h
    - `aria-label="Daten aktualisiert vor X Minuten"` auf dem Container
    - Kompakt: auf Mobile nur Dot ohne Text (via `size: 'sm' | 'md'` Prop)
    - Nutzt `src/lib/utils/date.ts` fuer `formatRelativeTime(date: Date): string` Utility
  - [x] 1.2: `src/lib/utils/date.ts` — Neue Utility-Funktion:
    - `formatRelativeTime(date: Date): string` — "Vor 5 Min", "Vor 1h 23 Min", "Vor 3h"
    - `getFreshnessLevel(date: Date): 'fresh' | 'stale' | 'old'` — <30 Min = fresh, 30 Min-2h = stale, >2h = old
    - Reine Funktionen, kein React-Import
  - [x] 1.3: `src/lib/utils/date.test.ts` — Tests fuer `formatRelativeTime` und `getFreshnessLevel`:
    - Edge-Cases: gerade eben (0 Min), genau 30 Min, genau 2h, 24h+, undefined/null Input
  - [x] 1.4: `src/components/ui/FreshnessIndicator.test.tsx` — Tests:
    - Rendert gruenen Dot bei fetchedAt < 30 Min
    - Rendert gelben Dot bei fetchedAt 30 Min-2h
    - Rendert roten Dot bei fetchedAt > 2h
    - `aria-label` korrekt gesetzt
    - Inline vs Banner Variante

- [x] Task 2: StaleDataBanner Komponente (AC: #2)
  - [x] 2.1: `src/components/ui/StaleDataBanner.tsx` — Client Component (`'use client'`):
    - Props: `chain: string`, `lastFetchedAt: string | Date`, `className?: string`
    - Zeigt: "[Chain]-Daten von vor [Zeitraum]. Aktualisierung laeuft."
    - Styling: `bg-blue-900/20 border border-blue-700/30 text-blue-300` (info-Stil, NICHT Alarm-Rot)
    - Info-Icon links, Text mittig, optional Dismiss-Button rechts
    - `role="status"` und `aria-live="polite"` fuer Accessibility
    - Nutzt `formatChainBadgeLabel()` aus `format.ts` und `formatRelativeTime()` aus `date.ts`
  - [x] 2.2: `src/components/ui/StaleDataBanner.test.tsx` — Tests:
    - Rendert Chain-Name und Zeitraum korrekt
    - Info-Stil (blau, kein rot)
    - ARIA-Attribute vorhanden
    - Dismiss-Funktion

- [x] Task 3: EmptyState Komponente (AC: #3, #4, #5)
  - [x] 3.1: `src/components/ui/EmptyState.tsx` — Funktionale Komponente:
    - Props: `title: string`, `description?: string`, `action?: { label: string, onClick?: () => void, href?: string }`, `secondaryActions?: { label: string, onClick?: () => void, href?: string }[]`, `icon?: 'wallet' | 'search' | 'chart'`
    - Layout: Zentriert, minimales monochrom Line-Icon (SVG inline), Titel, Beschreibung, Action-Button(s)
    - Kein Schuldzuweisungs-Text, freundlicher Ton
    - Responsive: gleiche Darstellung auf Mobile und Desktop
    - Action-Button: Tremor `Button` (Primary) fuer Hauptaktion
    - Secondary Actions: Text-Links mit `text-brand-primary` Farbe
  - [x] 3.2: `src/components/ui/EmptyState.test.tsx` — Tests:
    - Rendert Titel und Beschreibung
    - Rendert Action-Button wenn vorhanden
    - Rendert Secondary Actions als Links
    - Rendert ohne Action-Button korrekt

- [x] Task 4: API-Responses um Freshness-Daten erweitern (AC: #1, #2)
  - [x] 4.1: `src/app/api/portfolio/route.ts` — Erweitern:
    - `meta.freshness` aendern: statt "cache"/"db" → ISO-Timestamp des aeltesten `fetchedAt` aus den geladenen Positionen/Validators zurueckgeben
    - Neues Feld `meta.chainFreshness: Record<string, string>` — pro Chain den aeltesten `fetchedAt` Timestamp (fuer Stale-Data-Banner pro Chain)
    - Feld `meta.staleCains: string[]` — Chains deren Daten aelter als 2h sind (optional Helper fuer Client)
  - [x] 4.2: `src/app/api/validators/route.ts` — Erweitern:
    - `meta.freshness` aendern: ISO-Timestamp des aeltesten `fetchedAt` aus den zurueckgegebenen Validators
    - Response-Objekte um `fetchedAt: string` Feld erweitern (pro Validator)
  - [x] 4.3: API-Route Tests aktualisieren:
    - `src/app/api/portfolio/route.test.ts` — Freshness-Meta korrekt
    - `src/app/api/validators/route.test.ts` — fetchedAt im Response

- [x] Task 5: TanStack Query Hooks erweitern (AC: #1, #2)
  - [x] 5.1: `src/lib/hooks/use-portfolio.ts` — Response-Typen erweitern:
    - `meta.freshness: string` (ISO-Timestamp)
    - `meta.chainFreshness: Record<string, string>`
    - `meta.staleChains: string[]`
  - [x] 5.2: `src/lib/hooks/use-validators.ts` — Response-Typen erweitern:
    - `ValidatorWithPosition` Interface um `fetchedAt: string` erweitern
    - `meta.freshness: string` (ISO-Timestamp)

- [x] Task 6: Dashboard Integration — Freshness-Indicators (AC: #1)
  - [x] 6.1: `src/components/dashboard/ValidatorTable.tsx` — Erweitern:
    - Neue Spalte "Aktualisiert" mit `<FreshnessIndicator fetchedAt={v.fetchedAt} />` pro Validator-Row
    - Desktop: Volle inline Variante (Dot + Text)
  - [x] 6.2: `src/components/dashboard/ValidatorCard.tsx` — Erweitern:
    - `<FreshnessIndicator fetchedAt={v.fetchedAt} size="sm" />` in jeder Card (nur Dot auf Mobile)
  - [x] 6.3: `src/components/dashboard/PortfolioSummary.tsx` — Erweitern:
    - Freshness-Indicator unter den KPI-Cards als Banner-Variante: aeltester Datenpunkt als Referenz

- [x] Task 7: Dashboard Integration — Stale-Data-Banner (AC: #2)
  - [x] 7.1: `src/components/dashboard/DashboardContent.tsx` — Erweitern:
    - Pruefen ob `portfolioData?.meta?.staleChains` nicht-leer ist
    - Fuer jede stale Chain ein `<StaleDataBanner>` ueber der Validator-Liste rendern
    - Mehrere Banner stapelbar (max 3 — eine pro Chain)
    - Banner werden NICHT angezeigt wenn Daten noch laden (`isLoading`)

- [x] Task 8: Dashboard Integration — Empty States (AC: #3, #4, #5)
  - [x] 8.1: `src/components/dashboard/DashboardContent.tsx` — Empty State Logik:
    - **Keine Wallets:** Wenn `useLinkedWallets()` leer ist → EmptyState "Verbinde deine Wallet..." mit ConnectWallet-Button
    - **Keine Positionen:** Wenn Portfolio geladen aber `positions.length === 0` → EmptyState "Keine Staking-Positionen..." mit Links zu "SOL hinzufuegen · ATOM hinzufuegen"
    - **Gefilterter Empty State:** Wenn `selectedChain` gesetzt und `validatorsData?.data?.length === 0` → EmptyState "Keine Validators fuer [Chain]..." mit Link "Alle Chains anzeigen →"
  - [x] 8.2: Empty State Reihenfolge (Prioritaet):
    1. Keine Wallets → ConnectWallet Empty State (hoechste Prioritaet)
    2. Keine Positionen → No-Positions Empty State
    3. Gefilterter Empty → Filtered Empty State (nur Validator-Bereich)
    - KPI-Row und Chain-Filter werden IMMER angezeigt (auch bei Empty States fuer Positionen)
  - [x] 8.3: `useLinkedWallets()` Hook in DashboardContent einbinden:
    - Bereits existierend in `src/lib/hooks/use-linked-wallets.ts`
    - Import hinzufuegen, Wallet-Count pruefen

- [x] Task 9: Error Boundary fuer Graceful Degradation (AC: #2)
  - [x] 9.1: `src/components/ui/ErrorBoundary.tsx` — Error Boundary Wrapper:
    - Faengt React-Rendering-Fehler ab
    - Zeigt freundliche Fehlermeldung statt weissem Bildschirm
    - "Etwas ist schiefgelaufen. Versuche die Seite neu zu laden."
    - Refresh-Button der `window.location.reload()` aufruft
    - `componentDidCatch` loggt Fehler (ohne PII)
  - [x] 9.2: Error Boundary in Dashboard-Layout einbinden:
    - `src/app/(dashboard)/layout.tsx` — Dashboard-Content in ErrorBoundary wrappen

- [x] Task 10: Tests und Regressions-Check (AC: alle)
  - [x] 10.1: Alle neuen Unit-Tests ausfuehren (date-utils, FreshnessIndicator, StaleDataBanner, EmptyState, ErrorBoundary)
  - [x] 10.2: Alle bestehenden ~354 Tests muessen weiterhin bestehen
  - [x] 10.3: Biome-Check auf alle neuen/modifizierten Dateien
  - [x] 10.4: TypeScript-Check — keine neuen TS-Fehler eingefuehrt

## Dev Notes

### Kritische Architektur-Entscheidungen

- **Freshness basiert auf `fetchedAt` Timestamps:** Alle DB-Tabellen (`validators`, `staking_positions`, `chain_metrics`) haben bereits `fetchedAt` Felder. Diese werden durch die API an den Client weitergegeben und von FreshnessIndicator ausgewertet. KEIN neues DB-Feld noetig.

- **Stale-Chain-Detection auf Server-Seite:** Die API-Route `/api/portfolio` berechnet `meta.staleChains` serverseitig durch Vergleich der `fetchedAt` Timestamps mit der 2h-Grenze. Der Client muss die Freshness-Logik NICHT selbst berechnen fuer die Banner-Entscheidung.

- **Empty State Hierarchie:** Es gibt 3 verschiedene Empty States mit klarer Prioritaet. Der "Keine Wallets"-State ersetzt das GESAMTE Dashboard (kein KPI, kein Filter). Die anderen Empty States ersetzen nur den Validator-Bereich.

- **Error Boundary ist Class Component:** React Error Boundaries muessen Class Components sein (kein Hook-Aequivalent). Dies ist die einzige Stelle im Projekt wo eine Class Component verwendet wird.

- **FreshnessIndicator ist stateless:** Berechnet die Freshness bei jedem Render neu basierend auf `fetchedAt` und `Date.now()`. KEIN Interval/Timer — TanStack Query refetcht ohnehin regelmaessig und re-rendert die Komponente.

- **Graceful Degradation = Letzte bekannte Daten anzeigen:** Die Chain Adapters (Story 2-1 bis 2-3) speichern Daten mit `fetchedAt` Timestamp. Wenn ein Cron-Refresh fehlschlaegt, bleiben die alten Daten in der DB. Das Dashboard zeigt diese alten Daten mit rotem Freshness-Indicator. KEIN spezieller "Degradation Mode" in der API noetig — die Freshness-Information genuegt.

### Bestehende Infrastruktur (MUSS wiederverwendet werden)

| Komponente | Datei | Status |
|---|---|---|
| DashboardContent | `src/components/dashboard/DashboardContent.tsx` | MUSS erweitert werden |
| ValidatorTable | `src/components/dashboard/ValidatorTable.tsx` | MUSS erweitert werden (Freshness-Spalte) |
| ValidatorCard | `src/components/dashboard/ValidatorCard.tsx` | MUSS erweitert werden (Freshness-Dot) |
| PortfolioSummary | `src/components/dashboard/PortfolioSummary.tsx` | MUSS erweitert werden (Freshness-Banner) |
| ChainFilter | `src/components/dashboard/ChainFilter.tsx` | Unberuehrt |
| Portfolio API | `src/app/api/portfolio/route.ts` | MUSS erweitert werden (Freshness-Meta) |
| Validators API | `src/app/api/validators/route.ts` | MUSS erweitert werden (fetchedAt) |
| usePortfolio Hook | `src/lib/hooks/use-portfolio.ts` | MUSS erweitert werden (Response-Typen) |
| useValidators Hook | `src/lib/hooks/use-validators.ts` | MUSS erweitert werden (Response-Typen) |
| useLinkedWallets Hook | `src/lib/hooks/use-linked-wallets.ts` | Wiederverwendung in DashboardContent |
| Dashboard Layout | `src/app/(dashboard)/layout.tsx` | MUSS erweitert werden (ErrorBoundary) |
| format.ts | `src/lib/utils/format.ts` | Unberuehrt (formatChainBadgeLabel wird wiederverwendet) |
| date.ts | `src/lib/utils/date.ts` | MUSS erweitert werden (formatRelativeTime, getFreshnessLevel) |
| Toast.tsx | `src/components/ui/Toast.tsx` | Existiert — kann fuer Dismiss-Feedback genutzt werden |
| Cache | `src/infrastructure/cache/` | Unberuehrt — Cache-Aside Pattern weiter nutzen |
| DB Schema | `src/infrastructure/db/schema.ts` | Unberuehrt — `fetchedAt` Felder bereits vorhanden |

### Learnings aus Story 2-5 (MUSS beachtet werden)

- **`--legacy-peer-deps` noetig:** Fuer ALLE npm installs wegen React 19 Peer-Dep-Konflikten
- **`serverEnv` statt `process.env`:** IMMER `serverEnv` aus `@/lib/env` verwenden
- **Biome 2.2.0:** Alle neuen Dateien muessen Biome-Check bestehen
- **Zod 4.3.6:** v4 Syntax verwenden (nicht v3!)
- **Named Exports:** Kein `default export` ausser Next.js-Konventionen (`page.tsx`, `layout.tsx`, `route.ts`)
- **Domain-Layer importiert NUR aus domain/models/:** KEIN React, KEIN Next.js, KEIN infra
- **API Response Format:** `{ data, meta? }` (Success), `{ error, code, details? }` (Error)
- **Logging:** `[component] message` Format, KEINE Wallet-Adressen in Logs (nur gekuerzt)
- **Known Issue FIXME [M1]:** `linked_wallets` nutzt `'atom'`, `validators`/`staking_positions`/`chain_metrics` nutzen `'cosmos'` — Mapping-Funktionen existieren in `chain.ts`
- **~354 bestehende Tests** muessen weiterhin bestehen
- **Tremor v3.18.7:** NUR v3 Komponenten-APIs verwenden (Table, Card, Metric, Badge, TabGroup, Button)
- **TanStack Query:** QueryClientProvider in `src/providers/web3-provider.tsx` bereits konfiguriert
- **`placeholderData: (prev) => prev`:** Fuer Optimistic UI bei Filter-Wechsel — Pattern beibehalten
- **Chain-Name-Mapping:** `linkedWalletChainToDbChain()` und `dbChainToLinkedWalletChain()` in `chain.ts` nutzen
- **Design Tokens:** `bg-base: #0A0A0F`, `bg-surface: #12121A`, `bg-surface-raised: #1A1A25`, `border-subtle: #2A2A38`, `text-primary: #F0F0F5`, `text-secondary: #9898A8`, `brand-primary: #06B6D4`

### Freshness-Berechnung Detail

```typescript
// Zeitspannen (in Millisekunden)
const FRESH_THRESHOLD = 30 * 60 * 1000;    // 30 Minuten
const STALE_THRESHOLD = 2 * 60 * 60 * 1000; // 2 Stunden

// Freshness Level
type FreshnessLevel = 'fresh' | 'stale' | 'old';

// fresh:  fetchedAt < 30 Min ago → Gruener Dot
// stale:  fetchedAt 30 Min - 2h ago → Gelber Dot
// old:    fetchedAt > 2h ago → Roter Dot
```

### Relative-Time-Formatierung (Deutsch)

```typescript
// Beispiele:
// 0-59 Sekunden → "Gerade eben"
// 1-59 Minuten  → "Vor 5 Min"
// 1-23 Stunden  → "Vor 1h 23 Min" oder "Vor 3h"
// 1+ Tage       → "Vor 2 Tagen"
```

### Empty-State-Varianten

```
Variante 1: Keine Wallets (hoechste Prioritaet)
┌───────────────────────────────┐
│          [Wallet-Icon]        │
│                               │
│  Verbinde deine Wallet und    │
│  sieh dein Staking-Portfolio. │
│                               │
│  [Connect Wallet]  (Primary)  │
└───────────────────────────────┘

Variante 2: Wallet verbunden, keine Positionen
┌───────────────────────────────┐
│          [Chart-Icon]         │
│                               │
│  Keine Staking-Positionen     │
│  gefunden.                    │
│                               │
│  Hast du auf einer anderen    │
│  Chain gestaked?              │
│  SOL hinzufuegen · ATOM       │
│  hinzufuegen                  │
└───────────────────────────────┘

Variante 3: Chain-Filter ohne Ergebnisse
┌───────────────────────────────┐
│         [Search-Icon]         │
│                               │
│  Keine Validators fuer        │
│  [Chain] gefunden.            │
│                               │
│  Alle Chains anzeigen →       │
└───────────────────────────────┘
```

### Abgrenzung: Was diese Story NICHT macht

- **Kein Real Yield:** Real-Yield KPI und Spalte kommen in Story 3.2
- **Kein VRS-Score:** VRS-Badge kommt in Story 4.2
- **Keine neuen DB-Tabellen oder Migrationen:** Alle Tabellen und `fetchedAt` Felder existieren bereits
- **Keine neuen npm-Pakete:** Alle benoetigten Pakete sind bereits installiert
- **Kein Polling/Interval fuer Live-Freshness-Updates:** TanStack Query Refetch genuegt
- **Keine Push-Notifications bei Stale Data:** Nur visuelle Anzeige im Dashboard
- **Kein Retry-Button auf Stale-Data-Banner:** Refresh-Button im Header reicht (existiert bereits)

### Project Structure Notes

- Neue Dateien in `src/components/ui/` (FreshnessIndicator, StaleDataBanner, EmptyState, ErrorBoundary + Tests)
- Neue/Erweiterte Datei: `src/lib/utils/date.ts` (formatRelativeTime, getFreshnessLevel + Tests)
- Modifizierte Dateien in `src/components/dashboard/` (DashboardContent, ValidatorTable, ValidatorCard, PortfolioSummary)
- Modifizierte Dateien in `src/app/api/` (portfolio/route.ts, validators/route.ts)
- Modifizierte Dateien in `src/lib/hooks/` (use-portfolio.ts, use-validators.ts)
- Modifizierte Datei: `src/app/(dashboard)/layout.tsx` (ErrorBoundary)
- Tests co-located neben Source-Dateien
- KEINE neuen DB-Tabellen oder Migrationen noetig
- KEINE neuen npm-Pakete noetig

### References

- [Source: _bmad-output/planning-artifacts/epics.md#Story 2.6 — Daten-Freshness, Graceful Degradation & Empty States]
- [Source: _bmad-output/planning-artifacts/epics.md#Epic 2 — FR12, FR13, FR14]
- [Source: _bmad-output/planning-artifacts/architecture.md#Process Patterns — Error Recovery: Chain API timeout → Letzte bekannte Daten + Stale-Banner]
- [Source: _bmad-output/planning-artifacts/architecture.md#Implementation Patterns — Loading States (Suspense + TanStack Query)]
- [Source: _bmad-output/planning-artifacts/architecture.md#Structure Patterns — src/components/ui/ (Skeleton, StaleDataBanner, EmptyState, ErrorBoundary)]
- [Source: _bmad-output/planning-artifacts/architecture.md#Format Patterns — API Response { data, meta }]
- [Source: _bmad-output/planning-artifacts/architecture.md#Architectural Boundaries — NFR24 (kein Crash, keine leere Seite), NFR28 (Chain-Isolation)]
- [Source: _bmad-output/planning-artifacts/ux-design-specification.md#Dashboard — Freshness-Timestamps, Empty States]
- [Source: _bmad-output/planning-artifacts/ux-design-specification.md#Design Tokens — Status-Farben: status-positive (Gruen), status-negative (Rot)]
- [Source: _bmad-output/project-context.md#Caching (Multi-Layer) — TTL-basiertes Caching, Freshness-Transparency]
- [Source: _bmad-output/project-context.md#Framework-Specific Rules — RSC Default, Client Components nur wenn noetig]
- [Source: _bmad-output/implementation-artifacts/2-5-staking-dashboard-ui-portfolio-aggregation.md — Bestehende Dashboard-Komponenten, Chain-Mapping, Format-Utilities]
- [Source: src/infrastructure/db/schema.ts — validators.fetchedAt, staking_positions.fetchedAt, chain_metrics.fetchedAt]
- [Source: src/app/api/portfolio/route.ts — Bestehende meta.freshness Implementierung]
- [Source: src/app/api/validators/route.ts — Bestehende Response-Struktur]

## Dev Agent Record

### Agent Model Used

Claude Opus 4.6

### Debug Log References

- Vorbestehender flaky Test: `wallet-verification.test.ts` timeout bei vollem Test-Suite-Lauf (isoliert bestanden)
- Vorbestehende TS-Fehler in: wallets/route.test.ts, linked-wallet-repository.test.ts, mms-client.test.ts, magic-link.test.ts, turnstile.test.ts

### Completion Notes List

- Task 1: FreshnessIndicator Komponente + date.ts Utilities implementiert (26 Tests)
- Task 2: StaleDataBanner mit Dismiss-Funktion, Info-Styling, ARIA (5 Tests)
- Task 3: EmptyState mit 3 Varianten (wallet/search/chart Icons), Actions (7 Tests)
- Task 4: Portfolio API um chainFreshness/staleChains erweitert, Validators API um fetchedAt (10 Tests)
- Task 5: usePortfolio/useValidators Hooks um Freshness-Typen erweitert
- Task 6: ValidatorTable/Card/PortfolioSummary um FreshnessIndicator erweitert
- Task 7: DashboardContent zeigt StaleDataBanner fuer stale Chains (max 3)
- Task 8: 3 Empty States implementiert (keine Wallets, keine Positionen, gefilterter Leerstand)
- Task 9: ErrorBoundary (Class Component) in Dashboard Layout eingebunden
- Task 10: 391/392 Tests bestehen, Biome clean, keine neuen TS-Fehler

### Change Log

- 2026-02-23: Story 2-6 implementiert — Freshness-Indicators, Stale-Data-Banner, Empty States, ErrorBoundary
- 2026-02-23: Code Review (Adversarial) — 4 HIGH, 5 MEDIUM Issues gefunden und behoben:
  - H1: aria-label nutzt jetzt formatRelativeTimeVerbose (volle Woerter statt Abkuerzungen)
  - H2: "Connect Wallet" Empty State nutzt jetzt ConnectWalletButton statt Link
  - H3: ErrorBoundary.test.tsx erstellt (5 Tests)
  - H4: date.ts NaN/Invalid-Date-Guards hinzugefuegt + Tests
  - M1: Gefilterter Empty State nutzt formatChainBadgeLabel statt rohem Chain-ID
  - M2: ValidatorWithPosition in shared Type ausgelagert (4 Duplikate entfernt)
  - M3: EmptyState.tsx 'use client' Directive hinzugefuegt
  - M4: StaleDataBanner Chain-Fallback fuer unbekannte Chains
  - M5: Portfolio API Freshness-Berechnung optimiert (Map<string, Date> statt wiederholte new Date())

### File List

**Neue Dateien:**
- `src/lib/utils/date.ts` — formatRelativeTime, getFreshnessLevel, formatRelativeTimeVerbose Utility
- `src/lib/utils/date.test.ts` — 29 Tests fuer date utilities (inkl. verbose + invalid date)
- `src/components/ui/FreshnessIndicator.tsx` — Freshness-Dot + Zeitanzeige
- `src/components/ui/FreshnessIndicator.test.tsx` — 8 Tests
- `src/components/ui/StaleDataBanner.tsx` — Info-Banner fuer stale Chain-Daten
- `src/components/ui/StaleDataBanner.test.tsx` — 5 Tests
- `src/components/ui/EmptyState.tsx` — Generische Empty State Komponente (mit children-Support)
- `src/components/ui/EmptyState.test.tsx` — 7 Tests
- `src/components/ui/ErrorBoundary.tsx` — React Error Boundary (Class Component)
- `src/components/ui/ErrorBoundary.test.tsx` — 5 Tests
- `src/domain/models/validator-with-position.ts` — Shared ValidatorWithPosition Interface

**Modifizierte Dateien:**
- `src/app/api/portfolio/route.ts` — meta.freshness als ISO-Timestamp, chainFreshness, staleChains, optimierte Freshness-Berechnung
- `src/app/api/portfolio/route.test.ts` — Freshness-Meta-Erwartungen aktualisiert
- `src/app/api/validators/route.ts` — fetchedAt pro Validator, freshness Meta, shared Type
- `src/app/api/validators/route.test.ts` — fetchedAt in Mock-Daten hinzugefuegt
- `src/lib/hooks/use-portfolio.ts` — PortfolioResponse Typ erweitert
- `src/lib/hooks/use-validators.ts` — Shared ValidatorWithPosition Type, Meta erweitert
- `src/components/dashboard/ValidatorTable.tsx` — "Aktualisiert" Spalte mit FreshnessIndicator, shared Type
- `src/components/dashboard/ValidatorCard.tsx` — FreshnessIndicator (sm) pro Card, shared Type
- `src/components/dashboard/PortfolioSummary.tsx` — Freshness-Banner unter KPIs
- `src/components/dashboard/DashboardContent.tsx` — StaleDataBanner, Empty States, ConnectWalletButton, formatChainBadgeLabel
- `src/app/(dashboard)/layout.tsx` — ErrorBoundary Wrapper
