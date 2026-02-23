# Story 3.1: Real-Yield Calculator Domain-Logik & Taegliche Snapshots

Status: done

<!-- Note: Validation is optional. Run validate-create-story for quality check before dev-story. -->
<!-- Generated: 2026-02-23 | Epic: 3 | Story: 1 | Key: 3-1-real-yield-calculator-domain-logik-taegliche-snapshots -->

## Story

As a System-Betreiber,
I want dass Real-Yield-Werte korrekt berechnet und taeglich als Snapshots gespeichert werden,
So that User und der Public Calculator jederzeit auf aktuelle Real-Yield-Daten zugreifen koennen.

## Acceptance Criteria

1. **Given** der Yield Calculator (`src/domain/yield/yield-calculator.ts`) **When** `calculateRealYield()` aufgerufen wird **Then** berechnet er: Real Yield = Nominal APY - Inflation Rate - Validator Fee - Slashing-Risiko **And** jede Komponente der Formel ist einzeln im Ergebnis enthalten (fuer Breakdown-Anzeige) **And** die Berechnung ist deterministisch (gleiche Inputs = gleiches Ergebnis) **And** das Ergebnis enthaelt: `nominalApy`, `inflationRate`, `validatorFee`, `slashingRisk`, `realYield`, `chain`, `validatorId`

2. **Given** eine `yield-config.ts` mit konfigurierbaren Parametern **When** sich Chain-spezifische Faktoren aendern (z.B. SIMD-123 auf Solana) **Then** kann die Formel ueber Konfiguration angepasst werden ohne Code-Aenderung

3. **Given** der QStash Cron-Job `/api/cron/calculate-yield` **When** der Cron taeglich feuert (FR20) **Then** werden Yield-Snapshots fuer alle aktiven Validators ueber alle Chains berechnet **And** die Snapshots werden in der `yield_snapshots`-Tabelle gespeichert (TimescaleDB Hypertable) **And** historische Snapshots sind immutable (NFR27) **And** der Cache wird nach erfolgreicher Berechnung invalidiert

4. **Given** Chain-Daten fehlen (z.B. Inflationsrate nicht verfuegbar) **When** die Berechnung versucht wird **Then** wird der letzte bekannte Wert als Fallback verwendet **And** das Ergebnis enthaelt ein `confidence`-Feld das den Fallback kennzeichnet

## Tasks / Subtasks

- [x] Task 1: Yield Domain Models & Zod Schemas (AC: #1)
  - [x] 1.1: `src/domain/models/yield.ts` — Domain Types + Zod Schemas:
    - `RealYieldResult` Schema: `nominalApy`, `inflationRate`, `validatorFee`, `slashingRisk`, `realYield` (alle `number`), `chain` (chainSchema), `validatorId` (string), `confidence` (enum: `'high' | 'medium' | 'low'`)
    - `YieldInput` Schema: `nominalApy` (number), `inflationRate` (number), `validatorFee` (number, 0-1), `slashingRisk` (number, >= 0), `chain` (chainSchema), `validatorId` (string)
    - `YieldSnapshotRecord` Schema: fuer DB-Insert (chain, validatorId, nominalApy, inflationRate, validatorFee, slashingRisk, realYield, confidence, snapshotDate)
    - Alle Schemas mit Zod v4 Syntax (`z.enum()` statt `z.nativeEnum()`, `z.record(keySchema, valueSchema)`)
    - Named Exports, KEIN default export

- [x] Task 2: Yield-Konfiguration (AC: #2)
  - [x] 2.1: `src/domain/yield/yield-config.ts` — Konfigurierbare Parameter:
    - `YIELD_CONFIG` Objekt mit Chain-spezifischen Slashing-Risk-Defaults: ETH: 0.001 (0.1%), SOL: 0.002 (0.2%), ATOM: 0.005 (0.5%)
    - SIMD-123 Flag: `solanaFeeRevenueSharing: false` (NICHT live, erst Q3 2026 erwartet)
    - Alle Werte als exportierte KONSTANTEN (SCREAMING_SNAKE_CASE)
    - KEIN Import aus infrastructure/ oder React — reine Domain-Logik
  - [x] 2.2: `src/domain/yield/yield-config.test.ts` — Tests:
    - Config-Werte sind innerhalb sinnvoller Bereiche (0-1 fuer Raten)
    - Alle 3 Chains haben Default-Slashing-Risiko

- [x] Task 3: Yield Calculator Domain-Logik (AC: #1, #4)
  - [x] 3.1: `src/domain/yield/yield-calculator.ts` — Kern-Berechnungslogik:
    - `calculateRealYield(input: YieldInput): RealYieldResult` — Reine Funktion
    - Formel: `realYield = nominalApy - inflationRate - (nominalApy * validatorFee) - slashingRisk`
    - WICHTIG: `validatorFee` ist ein Anteil am `nominalApy`, NICHT ein absoluter Abzug — die Fee wird als `nominalApy * commission` berechnet
    - Confidence-Berechnung: `'high'` wenn alle Inputs > 0, `'medium'` wenn slashingRisk = 0 (Default genutzt), `'low'` wenn inflationRate = 0 (Daten fehlen)
    - Ergebnis-Zahlen auf 6 Dezimalstellen runden (Determinismus, NFR26)
    - KEIN Import aus infrastructure/, React oder Next.js — reine Domain-Logik
    - Named Export: `export function calculateRealYield(...)`
  - [x] 3.2: `calculateBatchRealYields(inputs: YieldInput[]): RealYieldResult[]` — Batch-Variante fuer Cron
    - Mapped ueber calculateRealYield (keine separate Logik)
  - [x] 3.3: `buildYieldInput(validator, chainMetrics, config): YieldInput` — Helper der Rohdaten in YieldInput umwandelt:
    - `nominalApy`: Aus `chainMetrics.avgApy` (ETH: wird als APR gespeichert) oder eigene Berechnung
    - `inflationRate`: Aus `chainMetrics.inflationRate` — Fallback 0 wenn null
    - `validatorFee`: Aus `validator.commission` (bereits 0-1 Skala im UnifiedValidator)
    - `slashingRisk`: Aus `YIELD_CONFIG.defaultSlashingRisk[chain]`
    - WICHTIG: `chain_metrics.avgApy` ist bei ETH gesetzt (von beaconcha.in ethstore), bei SOL/ATOM null — fuer SOL/ATOM muss `nominalApy` aus `inflationRate * (1 - avgValidatorCommission)` approximiert werden
  - [x] 3.4: `src/domain/yield/yield-calculator.test.ts` — P0 Tests (>80% Coverage):
    - Positiver Real Yield (nominalApy 5% - inflation 2% - fee 0.5% - slashing 0.1% = ~2.4%)
    - Negativer Real Yield (hohe Inflation)
    - Null/Zero Inputs (fehlende Daten → low confidence)
    - Determinismus (gleiche Inputs = exakt gleiches Ergebnis, 2x ausfuehren)
    - Alle 3 Chains mit realistischen Werten: ETH (~3.5% APR, ~0% Inflation wg. EIP-1559), SOL (~7% APR, ~5% Inflation), ATOM (~15% APR, ~10% Inflation)
    - Edge Case: Validator mit 100% Commission → realYield = -inflation - slashing
    - Edge Case: inflationRate = null → Confidence = 'low', inflationRate = 0 in Berechnung
    - buildYieldInput Tests mit Mock-Validator und Mock-ChainMetrics

- [x] Task 4: DB-Schema fuer yield_snapshots (AC: #3)
  - [x] 4.1: `src/infrastructure/db/schema.ts` — Neue Tabelle `yieldSnapshots`:
    - `id`: uuid, NOT NULL (Teil des PK zusammen mit snapshotDate)
    - `chain`: varchar(10), NOT NULL, check constraint ('ethereum', 'solana', 'cosmos')
    - `validatorId`: uuid, FK → validators.id (onDelete: 'set null'), nullable
    - `nominalApy`: numeric(10,6), NOT NULL
    - `inflationRate`: numeric(10,6), NOT NULL
    - `validatorFee`: numeric(10,6), NOT NULL
    - `slashingRisk`: numeric(10,6), NOT NULL
    - `realYield`: numeric(10,6), NOT NULL
    - `confidence`: varchar(10), NOT NULL, check constraint ('high', 'medium', 'low')
    - `snapshotDate`: timestamp with timezone, NOT NULL (Teil des PK — NeonDB-Pflicht fuer Hypertable)
    - `createdAt`: timestamp with timezone, NOT NULL, defaultNow()
    - PrimaryKey: COMPOSITE aus `id` + `snapshotDate` (NeonDB Hypertable-Constraint: Zeitstempel MUSS im PK sein)
    - Index: `idx_yield_snapshots_chain_validator` auf (chain, validatorId)
    - Index: `idx_yield_snapshots_snapshot_date` auf (snapshotDate DESC)
    - KEINE update-Operationen auf dieser Tabelle (NFR27 — immutable)
  - [x] 4.2: Migration generieren: `npx drizzle-kit generate`
  - [x] 4.3: Custom Hypertable-Migration: `npx drizzle-kit generate --custom --name=create-yield-snapshots-hypertable`
    - SQL-Inhalt:
    ```sql
    CREATE EXTENSION IF NOT EXISTS timescaledb;
    SELECT create_hypertable('yield_snapshots', 'snapshot_date', if_not_exists => TRUE, migrate_data => TRUE);
    ```
    - KEIN `set_chunk_time_interval` — Default 7 Tage ist OK fuer taegliche Snapshots
    - KEINE Kompression konfigurieren — NeonDB unterstuetzt TimescaleDB-Kompression NICHT (nur Apache-2 Features)
    - KEINE Background-Jobs (`add_job`) — NeonDB Scale-to-Zero verhindert deren Ausfuehrung

- [x] Task 5: Yield Repository (AC: #3, #4)
  - [x] 5.1: `src/infrastructure/db/repositories/yield-repository.ts`:
    - `insertYieldSnapshots(snapshots: YieldSnapshotRecord[]): Promise<void>` — Batch-Insert, KEINE upsert-Logik (immutable Eintraege)
    - `findLatestByChainAndValidator(chain: string, validatorId: string): Promise<YieldSnapshot | null>` — Letzter Snapshot fuer Fallback
    - `findLatestByChain(chain: string): Promise<YieldSnapshot[]>` — Alle aktuellen Snapshots einer Chain (fuer Dashboard/API)
    - Named Exports
  - [x] 5.2: `src/infrastructure/db/repositories/yield-repository.test.ts` — Tests:
    - Insert-Batch funktioniert
    - findLatestByChainAndValidator gibt neuesten Eintrag zurueck
    - findLatestByChain gibt alle Snapshots einer Chain zurueck

- [x] Task 6: Cron-Endpoint /api/cron/calculate-yield (AC: #3, #4)
  - [x] 6.1: `src/app/api/cron/calculate-yield/route.ts` — POST Handler:
    - HMAC-Signatur-Verifizierung (verifyQStashSignature) — exakt wie in fetch-validators/route.ts
    - Fuer jede Chain (ethereum, solana, cosmos):
      1. Alle aktiven Validators laden (`findValidatorsByChain`)
      2. Aktuelle Chain-Metrics laden (`findLatestByChain` aus chain-metrics-repository)
      3. `buildYieldInput()` pro Validator aufrufen
      4. `calculateBatchRealYields()` aufrufen
      5. Ergebnisse als `yield_snapshots` batch-inserten
    - Chain-Isolation (NFR28): Jede Chain unabhaengig via Promise.allSettled
    - Cache-Invalidierung nach Erfolg: Keys `staketrack:yield:{chain}:*` fuer erfolgreiche Chains
    - Response: `{ data: { chains: [{ chain, snapshotsCreated, error? }] } }`
    - Bei 0 Validators fuer eine Chain: Skip ohne Error (kein Snapshot noetig)
    - Logging: `[yield-cron] Calculated {n} snapshots for {chain}` — KEINE Wallet-Adressen
  - [x] 6.2: `src/app/api/cron/calculate-yield/route.test.ts` — Tests:
    - Erfolgreiche Berechnung fuer alle 3 Chains
    - HMAC-Verifizierung schlaegt fehl → 401
    - Eine Chain schlaegt fehl → andere Chains laufen trotzdem (207)
    - Keine Validators → leere Snapshots, kein Error

- [x] Task 7: Tests und Regressions-Check (AC: alle)
  - [x] 7.1: Alle neuen Unit-Tests ausfuehren (yield-calculator, yield-config, yield-repository, cron-route)
  - [x] 7.2: Alle bestehenden ~391 Tests muessen weiterhin bestehen
  - [x] 7.3: Biome-Check auf alle neuen/modifizierten Dateien
  - [x] 7.4: TypeScript-Check — keine neuen TS-Fehler
  - [x] 7.5: Migration lokal generieren und verifizieren (`npx drizzle-kit generate`)

## Dev Notes

### Kritische Architektur-Entscheidungen

- **Real Yield Formel:** `realYield = nominalApy - inflationRate - (nominalApy * validatorFee) - slashingRisk`. Die `validatorFee` ist KEIN absoluter Abzug, sondern die Commission des Validators angewendet auf den nominalen APY. Beispiel: 5% APY * 10% Commission = 0.5% Fee-Abzug.

- **Chain-spezifische APY-Quellen:**
  - **ETH:** `chain_metrics.avg_apy` enthaelt die Netzwerk-APR von beaconcha.in ethstore. Inflation ist bei ETH effektiv ~0% (EIP-1559 Burn kompensiert Issuance). `chain_metrics.inflation_rate` ist NULL fuer ETH.
  - **SOL:** `chain_metrics.inflation_rate` enthaelt die Solana-Inflationsrate (~5.3% Stand Feb 2026). `chain_metrics.avg_apy` ist NULL. Nominal APY muss approximiert werden: `inflationRate * (1 - avgCommission)` basierend auf Netzwerk-Durchschnitt.
  - **ATOM:** `chain_metrics.inflation_rate` enthaelt die Cosmos-Inflationsrate (~10-15%). `chain_metrics.avg_apy` ist NULL. Nominal APY = `inflationRate * (bondedRatio_adjustment)` — vereinfacht: `inflationRate` als Naeherung.

- **SIMD-123 (Solana Fee Revenue Sharing):** Noch NICHT live auf Mainnet (erwartet Q3 2026 in Agave 4.1). Flag in yield-config.ts ist `false`. Wenn aktiviert: Delegator-Yield erhaelt zusaetzliche Fee-Komponente. Aktuell: Nur Inflations-basierte Rewards fuer Delegatoren.

- **TimescaleDB Hypertable auf NeonDB — Einschraenkungen:**
  - Zeitstempel-Spalte (`snapshot_date`) MUSS Teil des Primary Key sein (NeonDB-Plattform-Constraint)
  - KEINE Kompression verfuegbar (nur Apache-2 Features auf NeonDB)
  - KEINE Background-Jobs (`add_job`, Retention-Policies) — NeonDB Scale-to-Zero verhindert Ausfuehrung
  - `drop_chunks()` muss manuell aufgerufen werden (z.B. via separatem Cron — NICHT in dieser Story)
  - Hypertable-Erstellung via Custom-Migration mit Raw SQL (`drizzle-kit generate --custom`)

- **Drizzle ORM + Hypertable:** Drizzle hat KEINEN nativen TimescaleDB-Support. Schema wird als normales `pgTable` definiert. Die Hypertable-Konvertierung passiert in einer separaten Custom-Migration mit Raw SQL. Drizzle-Queries funktionieren normal gegen Hypertables.

- **Composite Primary Key:** Die `yield_snapshots` Tabelle verwendet `primaryKey({ columns: [id, snapshotDate] })` statt eines einfachen `id.primaryKey()`. Das ist die NeonDB-Pflicht fuer Hypertables. UUID + Timestamp als PK.

- **Immutability (NFR27):** yield_snapshots hat KEINE update-Operationen. Repository bietet nur `insert` und `find`. KEIN upsert, KEIN delete (ausser spaeterem Retention-Cron).

- **Confidence-Logik:** `'high'` = alle Inputs vorhanden und > 0, `'medium'` = Slashing-Risk ist Default-Wert (nicht chain-spezifisch berechnet), `'low'` = Inflation-Rate fehlt oder ist 0 (Fallback). Bei ETH ist inflationRate = 0 normal (EIP-1559) → trotzdem `'high'` Confidence fuer ETH wenn avgApy vorhanden.

- **Validator-Commission in DB:** Die `validators.commission` Spalte speichert den Wert als `numeric(10,6)`. Im `UnifiedValidator`-Schema ist `commission: z.number().min(0).max(1)` — also BEREITS auf 0-1 Skala. KEINE Umrechnung von Prozent noetig.

### Bestehende Infrastruktur (MUSS wiederverwendet werden)

| Komponente | Datei | Verwendung |
|---|---|---|
| ChainAdapter Interface | `src/domain/chain-adapters/types.ts` | Nicht direkt — aber Validator-Daten kommen von diesen Adaptern |
| Chain Enum + Config | `src/domain/models/chain.ts` | Chain-Typen, CHAIN_CONFIGS fuer Token-Infos |
| UnifiedValidator Schema | `src/domain/models/validator.ts` | Validator.commission fuer Fee-Berechnung |
| DB Schema | `src/infrastructure/db/schema.ts` | MUSS erweitert werden (yieldSnapshots Tabelle) |
| DB Index | `src/infrastructure/db/index.ts` | Drizzle Client Instance, import als `db` |
| Validator Repository | `src/infrastructure/db/repositories/validator-repository.ts` | `findValidatorsByChain()` fuer Cron |
| Chain Metrics Repository | `src/infrastructure/db/repositories/chain-metrics-repository.ts` | `findLatestByChain()` fuer inflationRate/avgApy |
| Cache Keys | `src/infrastructure/cache/cache-keys.ts` | `yieldSnapshotKey()` und `CACHE_TTL_YIELD` bereits definiert |
| Cache Utils | `src/infrastructure/cache/cache-utils.ts` | `cacheInvalidate()` fuer Post-Cron-Invalidierung |
| QStash Verify | `src/infrastructure/queue/qstash-verify.ts` | `verifyQStashSignature()` fuer HMAC |
| Cron Pattern | `src/app/api/cron/fetch-validators/route.ts` | EXAKTE Vorlage fuer den Yield-Cron (selbes Pattern: HMAC → allSettled → Invalidate) |
| Server Env | `src/lib/env.ts` | `serverEnv` statt `process.env` verwenden |
| Yield .gitkeep | `src/domain/yield/.gitkeep` | Verzeichnis existiert bereits, .gitkeep loeschen wenn erste Datei erstellt wird |

### Learnings aus Epic 2 (MUSS beachtet werden)

- **`--legacy-peer-deps` noetig:** Fuer ALLE `npm install` wegen React 19 Peer-Dep-Konflikten
- **`serverEnv` statt `process.env`:** IMMER `serverEnv` aus `@/lib/env` verwenden
- **Biome 2.2.0:** Alle neuen Dateien muessen Biome-Check bestehen
- **Zod 4.3.6 v4 Syntax:** `z.enum()` statt `z.nativeEnum()`, `z.record(keySchema, valueSchema)` mit 2 Args, Error-Messages via `{ error: "..." }` statt `{ message: "..." }`
- **Named Exports:** Kein `default export` ausser Next.js-Konventionen (`page.tsx`, `layout.tsx`, `route.ts`)
- **Domain-Layer importiert NUR aus domain/models/:** KEIN React, KEIN Next.js, KEIN infra
- **API Response Format:** `{ data, meta? }` (Success), `{ error, code, details? }` (Error)
- **Logging:** `[component] message` Format, KEINE Wallet-Adressen in Logs (nur gekuerzt)
- **Known Issue FIXME [M1]:** `linked_wallets` nutzt `'atom'`, `validators`/`staking_positions`/`chain_metrics` nutzen `'cosmos'` — Mapping-Funktionen in `chain.ts`
- **~391 bestehende Tests** muessen weiterhin bestehen
- **Tremor v3.18.7:** Nur relevant fuer UI (nicht in dieser Story)
- **Design Tokens:** Nicht relevant fuer diese Backend-Story
- **DB numeric Felder:** Drizzle gibt `numeric` Spalten als `string` zurueck — `Number()` oder `parseFloat()` beim Lesen verwenden

### Yield-Berechnungs-Beispiele (fuer Tests)

```typescript
// ETH: Netzwerk-APR ~3.5%, Inflation ~0%, Commission 5%, Slashing 0.1%
// validatorFee = 0.035 * 0.05 = 0.00175
// realYield = 0.035 - 0 - 0.00175 - 0.001 = 0.03225 (3.225%)

// SOL: Inflation ~5.3%, Nominal APY ~7%, Commission 8%, Slashing 0.2%
// validatorFee = 0.07 * 0.08 = 0.0056
// realYield = 0.07 - 0.053 - 0.0056 - 0.002 = 0.0094 (0.94%)

// ATOM: Inflation ~12%, Nominal APY ~15%, Commission 10%, Slashing 0.5%
// validatorFee = 0.15 * 0.10 = 0.015
// realYield = 0.15 - 0.12 - 0.015 - 0.005 = 0.01 (1.0%)

// Negativ: SOL mit hoher Inflation
// Inflation 8%, Nominal 6%, Commission 10%, Slashing 0.2%
// validatorFee = 0.06 * 0.10 = 0.006
// realYield = 0.06 - 0.08 - 0.006 - 0.002 = -0.028 (-2.8%)
```

### Abgrenzung: Was diese Story NICHT macht

- **Kein Dashboard-UI:** Real-Yield-Integration ins Dashboard kommt in Story 3.2
- **Kein Public Calculator UI:** Calculator-Page kommt in Story 3.3b
- **Keine Landing Page:** Kommt in Story 3.3a
- **Kein VRS-Score:** Kommt in Epic 4
- **Keine Alert-Integration:** Kommt in Epic 5
- **Kein Tier-Gating:** Kommt in Epic 6
- **Kein `drop_chunks` Retention-Cron:** Separate Story (TimescaleDB-Maintenance)
- **Keine API-Route fuer User-facing Yield-Daten:** `GET /api/yield` kommt in Story 3.2
- **Keine neuen npm-Pakete noetig:** Alle Abhaengigkeiten (Drizzle, Zod, QStash, Upstash) bereits installiert

### Neue Dateien

```
src/domain/models/yield.ts                               # Domain Types + Zod Schemas
src/domain/yield/yield-config.ts                         # Konfigurierbare Parameter
src/domain/yield/yield-config.test.ts                    # Config-Tests
src/domain/yield/yield-calculator.ts                     # Kern-Berechnungslogik
src/domain/yield/yield-calculator.test.ts                # P0 Tests
src/infrastructure/db/repositories/yield-repository.ts   # DB-Zugriff
src/infrastructure/db/repositories/yield-repository.test.ts # Repository-Tests
src/app/api/cron/calculate-yield/route.ts                # QStash Cron-Endpoint
src/app/api/cron/calculate-yield/route.test.ts           # Cron-Tests
drizzle/XXXX_create_yield_snapshots.sql                  # Schema-Migration (generiert)
drizzle/XXXX_create-yield-snapshots-hypertable.sql       # Custom Hypertable-Migration
```

### Modifizierte Dateien

```
src/infrastructure/db/schema.ts                          # + yieldSnapshots Tabelle
```

### Project Structure Notes

- Alle neuen Domain-Dateien in `src/domain/yield/` (Framework-agnostisch, KEIN React/Next.js-Import)
- Domain-Model `yield.ts` in `src/domain/models/` (konsistent mit validator.ts, chain.ts, position.ts)
- Repository in `src/infrastructure/db/repositories/` (konsistent mit bestehenden Repositories)
- Cron-Route in `src/app/api/cron/calculate-yield/` (konsistent mit fetch-validators)
- Tests co-located neben Source-Dateien
- `.gitkeep` in `src/domain/yield/` kann geloescht werden sobald erste echte Datei erstellt wird

### References

- [Source: _bmad-output/planning-artifacts/epics.md#Story 3.1 — Real-Yield Calculator Domain-Logik & Taegliche Snapshots]
- [Source: _bmad-output/planning-artifacts/epics.md#Epic 3 — FR15, FR16, FR17, FR20]
- [Source: _bmad-output/planning-artifacts/architecture.md#Data Architecture — Zod v4.3, Drizzle ORM, TimescaleDB Hypertables]
- [Source: _bmad-output/planning-artifacts/architecture.md#Core Architectural Decisions — Real-Yield Calculator]
- [Source: _bmad-output/planning-artifacts/architecture.md#Implementation Patterns — Naming, Structure, Format Patterns]
- [Source: _bmad-output/planning-artifacts/architecture.md#Project Structure — src/domain/yield/, src/app/api/cron/calculate-yield/]
- [Source: _bmad-output/planning-artifacts/architecture.md#ADR-001 — User-Driven Validator Discovery, ChainAdapter Interface]
- [Source: _bmad-output/planning-artifacts/architecture.md#External Integration Points — NeonDB TimescaleDB, Upstash QStash/Redis]
- [Source: _bmad-output/project-context.md#Real-Yield Calculator — Formel, konfigurierbare Parameter, SIMD-123]
- [Source: _bmad-output/project-context.md#Testing Rules — P0 Coverage, Domain-Layer >80%, MSW fuer API-Mocking]
- [Source: _bmad-output/project-context.md#NeonDB + TimescaleDB — Hypertable-Constraints, keine Kompression auf NeonDB]
- [Source: _bmad-output/project-context.md#Cron/Background Jobs — QStash HMAC, 60s Timeout, Fan-out Pattern]
- [Source: src/infrastructure/db/schema.ts — Bestehende Tabellen: validators, chain_metrics, staking_positions]
- [Source: src/domain/chain-adapters/types.ts — ChainAdapter Interface mit fetchInflationRate()]
- [Source: src/domain/models/validator.ts — UnifiedValidator.commission (0-1 Skala)]
- [Source: src/domain/models/chain.ts — Chain Enum, chainSchema, CHAIN_CONFIGS]
- [Source: src/infrastructure/cache/cache-keys.ts — yieldSnapshotKey(), CACHE_TTL_YIELD bereits definiert]
- [Source: src/infrastructure/db/repositories/chain-metrics-repository.ts — findLatestByChain() fuer Inflationsrate]
- [Source: src/infrastructure/db/repositories/validator-repository.ts — findValidatorsByChain() fuer aktive Validators]
- [Source: src/app/api/cron/fetch-validators/route.ts — Pattern-Vorlage: HMAC, Promise.allSettled, Cache-Invalidierung]
- [Source: _bmad-output/implementation-artifacts/2-6-daten-freshness-graceful-degradation-empty-states.md — Learnings: --legacy-peer-deps, serverEnv, Biome 2.2.0, Zod v4]

## Change Log

- 2026-02-23: Story 3-1 implementiert — Real-Yield Calculator Domain-Logik, DB-Schema, Repository, Cron-Endpoint mit 33 neuen Tests (alle bestanden), keine Regressionen
- 2026-02-23: Code Review (AI) — 8 Issues gefixed: H1 (AC#4 Fallback-Logik im Cron), H2 (findLatestByChain limit(100) → selectDistinctOn), M1 (console.log→console.info), M2 (500 Error code Feld), M4 (SQL-level Dedup), M5 (DESC Index im Schema). 1 neuer Fallback-Test hinzugefuegt. 442/442 Tests bestanden.

## Dev Agent Record

### Agent Model Used

Claude Opus 4.6

### Debug Log References

- Biome-Fix: Unused import `YieldSnapshotRecord` in cron route entfernt
- Biome-Fix: Formatting in yield-calculator.ts (multi-line to single-line expression)
- Test-Fix: Confidence-Erwartungen fuer SOL/ATOM korrigiert (medium statt high bei Default-Slashing-Risk)

### Completion Notes List

- Task 1: `src/domain/models/yield.ts` erstellt — 3 Zod Schemas (YieldInput, RealYieldResult, YieldSnapshotRecord) mit confidenceSchema, alle Zod v4 Syntax, named exports
- Task 2: `src/domain/yield/yield-config.ts` + Tests erstellt — DEFAULT_SLASHING_RISK (ETH: 0.001, SOL: 0.002, ATOM: 0.005), SIMD-123 Flag false, 5 Tests bestanden
- Task 3: `src/domain/yield/yield-calculator.ts` + Tests erstellt — calculateRealYield (reine Funktion), calculateBatchRealYields (Batch), buildYieldInput (Rohdaten-Konverter mit ETH/SOL/ATOM-spezifischer Logik), 17 Tests bestanden
- Task 4: `yield_snapshots` Tabelle in schema.ts hinzugefuegt — Composite PK (id + snapshotDate), 2 Indexe, 2 Check Constraints, FK zu validators. Migration 0005 + Custom Hypertable-Migration 0006 generiert
- Task 5: `yield-repository.ts` + Tests erstellt — insertYieldSnapshots (batch, immutable), findLatestByChainAndValidator, findLatestByChain (mit Dedup), 6 Tests bestanden
- Task 6: `calculate-yield/route.ts` + Tests erstellt — HMAC-Verify, Promise.allSettled pro Chain, Cache-Pattern-Invalidierung, 5 Tests bestanden
- Task 7: Alle 33 neuen Tests gruen, 440 von 441 Gesamt-Tests bestanden (1 vorbestehender Timeout in wallet-verification.test.ts), Biome clean, keine neuen TS-Fehler

### Implementation Plan

- Reine Domain-Logik in `src/domain/yield/` ohne Framework-Imports
- Formel exakt wie spezifiziert: `realYield = nominalApy - inflationRate - (nominalApy * validatorFee) - slashingRisk`
- Chain-spezifische APY-Quellen: ETH nutzt avgApy, SOL/ATOM fallen auf inflationRate zurueck
- Confidence-Logik: high (alle Inputs vorhanden), medium (Default-Slashing-Risk), low (fehlende Inflation)
- ETH-Sonderfall: inflationRate = 0 ist normal (EIP-1559), daher trotzdem high confidence
- Cron folgt exakt dem fetch-validators Pattern: HMAC, Promise.allSettled, Cache-Invalidierung
- DB-Schema mit Composite PK fuer NeonDB Hypertable-Kompatibilitaet

### File List

**Neue Dateien:**
- src/domain/models/yield.ts
- src/domain/yield/yield-config.ts
- src/domain/yield/yield-config.test.ts
- src/domain/yield/yield-calculator.ts
- src/domain/yield/yield-calculator.test.ts
- src/infrastructure/db/repositories/yield-repository.ts
- src/infrastructure/db/repositories/yield-repository.test.ts
- src/app/api/cron/calculate-yield/route.ts
- src/app/api/cron/calculate-yield/route.test.ts
- drizzle/0005_flippant_speed.sql
- drizzle/0006_create-yield-snapshots-hypertable.sql

**Modifizierte Dateien:**
- src/infrastructure/db/schema.ts (+ yieldSnapshots Tabelle)
- drizzle/meta/_journal.json (+ Migration 0005 + 0006 Eintraege)

**Neue Migrations-Metadaten:**
- drizzle/meta/0005_snapshot.json
- drizzle/meta/0006_snapshot.json

**Geloeschte Dateien:**
- src/domain/yield/.gitkeep
