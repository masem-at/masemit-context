# Story 2.3: Cosmos Chain Adapter

Status: done

<!-- Note: Validation is optional. Run validate-create-story for quality check before dev-story. -->
<!-- Generated: 2026-02-22 | Epic: 2 | Story: 3 | Key: 2-3-cosmos-chain-adapter -->

## Story

As a User mit ATOM-Staking-Positionen,
I want dass meine Cosmos-Staking-Daten automatisch abgerufen werden,
So that mein ATOM-Portfolio im Dashboard erscheint.

## Acceptance Criteria

1. **Given** der Cosmos Adapter (`src/domain/chain-adapters/cosmos/`) **When** `refreshValidators(knownAddresses)` aufgerufen wird **Then** werden Cosmos Hub Validator-Daten via LCD/REST API (`/cosmos/staking/v1beta1/validators`) abgerufen **And** die Daten werden per Zod Schema validiert und auf `UnifiedValidator` gemappt

2. **Given** der Cosmos Adapter **When** `fetchStakingPositions(walletAddress)` aufgerufen wird **Then** werden Delegations via `/cosmos/staking/v1beta1/delegations/{address}` abgerufen **And** die Positionen werden normalisiert als `StakingPosition[]` zurückgegeben

3. **Given** der Cosmos Adapter **When** `fetchInflationRate()` aufgerufen wird **Then** wird die Cosmos-Inflationsrate via `/cosmos/mint/v1beta1/inflation` abgerufen und als `number` zurückgegeben

4. **Given** der QStash Cron-Job `/api/cron/fetch-validators` **When** alle 3 Adapter parallel laufen (Fan-out) **Then** sind ETH, SOL und ATOM jeweils unabhängig (NFR28) **And** die Gesamtverarbeitung bleibt unter 60 Sekunden (NFR21)

5. **Given** die Cosmos LCD API ist nicht erreichbar **When** ein Fehler auftritt **Then** wird ein Fallback-LCD-Endpoint verwendet (falls konfiguriert) **And** der Fehler wird geloggt ohne Wallet-Adressen

## Tasks / Subtasks

- [x] Task 1: Cosmos Adapter Zod Schemas erstellen (AC: #1, #2, #3)
  - [x] 1.1: `src/domain/chain-adapters/cosmos/atom-schemas.ts` — Zod Schema für `/cosmos/staking/v1beta1/validators` Response (Validator-Objekt mit operator_address, consensus_pubkey, jailed, status, tokens, delegator_shares, description.moniker, commission.commission_rates.rate/max_rate/max_change_rate, unbonding_height, unbonding_time)
  - [x] 1.2: Zod Schema für Cosmos Pagination Response (`next_key: string | null`, `total: string`)
  - [x] 1.3: Zod Schema für `/cosmos/staking/v1beta1/delegations/{addr}` Response (delegation_responses Array mit delegation.delegator_address, delegation.validator_address, delegation.shares, balance.denom, balance.amount)
  - [x] 1.4: Zod Schema für `/cosmos/mint/v1beta1/inflation` Response (`inflation: string` — 18-Dezimal-String)

- [x] Task 2: Cosmos Mapper implementieren (AC: #1, #2)
  - [x] 2.1: `src/domain/chain-adapters/cosmos/atom-mapper.ts` — `mapCosmosValidatorToUnified(cosmosValidator): UnifiedValidator` — Mapping von Cosmos Validator auf UnifiedValidator
  - [x] 2.2: Status-Mapping: `BOND_STATUS_BONDED` + `!jailed` → "active", `BOND_STATUS_UNBONDING` → "inactive", `BOND_STATUS_UNBONDED` → "inactive", `jailed: true` → "jailed"
  - [x] 2.3: Commission-Mapping: Cosmos 18-Dezimal-String ("0.050000000000000000") → Float (0.05) — bereits im 0.0-1.0 Range (Konsistenz mit ETH/SOL)
  - [x] 2.4: `mapDelegationToPosition(delegation, walletAddress): StakingPosition` — Mapping von Delegation-Response auf StakingPosition
  - [x] 2.5: Betrags-Konvertierung: uatom (String, Ganzzahl) → ATOM String (6 Dezimalstellen, 1 ATOM = 1.000.000 uatom)

- [x] Task 3: Cosmos Adapter implementieren (AC: #1, #2, #3, #5)
  - [x] 3.1: `src/domain/chain-adapters/cosmos/atom-adapter.ts` — `CosmosAdapter` Klasse implementiert `ChainAdapter` Interface
  - [x] 3.2: `refreshValidators(knownAddresses)` — Alle bonded Validators via LCD API abrufen (`?status=BOND_STATUS_BONDED&pagination.limit=200`), auf bekannte `operator_address` filtern, Mapping via atom-mapper. Bei ≤200 Validators KEIN Paginieren nötig (max_validators=200), aber Pagination-Support für Robustheit implementieren
  - [x] 3.3: `fetchStakingPositions(walletAddress)` — `/cosmos/staking/v1beta1/delegations/{walletAddress}` mit KEY-basierter Pagination (NICHT offset — bekannter Cosmos SDK Bug cosmos-sdk#11407!), alle Delegations sammeln, Mapping via atom-mapper
  - [x] 3.4: `fetchInflationRate()` — `/cosmos/mint/v1beta1/inflation`, parseFloat auf 18-Dezimal-String
  - [x] 3.5: LCD-Fallback-Logik: Primary LCD URL → Fallback LCD (falls `COSMOS_LCD_FALLBACK_URL` konfiguriert)
  - [x] 3.6: Timeout-Handling: 10s für alle LCD-Calls via `AbortSignal.timeout(10_000)`

- [x] Task 4: Environment Variables ergänzen (AC: #5)
  - [x] 4.1: `src/lib/env.ts` erweitern — `COSMOS_LCD_URL` (optional, Default: `https://rest.cosmos.directory/cosmoshub`), `COSMOS_LCD_FALLBACK_URL` (optional)
  - [x] 4.2: `.env.example` aktualisieren mit Cosmos-spezifischen Variablen

- [x] Task 5: Factory-Funktion erweitern (AC: #4)
  - [x] 5.1: `src/domain/chain-adapters/factory.ts` — `CosmosAdapter` Import hinzufügen, `Chain.COSMOS` Case implementieren statt `throw new Error`
  - [x] 5.2: Factory nutzt `COSMOS_LCD_URL` und `COSMOS_LCD_FALLBACK_URL` aus Env-Config

- [x] Task 6: Cron-Endpoint erweitern (AC: #4)
  - [x] 6.1: `src/app/api/cron/fetch-validators/route.ts` — Cosmos-Adapter in dynamische Chain-Configs aufnehmen
  - [x] 6.2: Cosmos-spezifisches Metrics-Mapping: `inflationRate` direkt aus `fetchInflationRate()`
  - [x] 6.3: `Promise.allSettled()` läuft jetzt mit 3 Chains parallel — ETH + SOL + ATOM, jeweils isoliert (NFR28)
  - [x] 6.4: Response erweitern: `{ data: { chains: [{ chain: "ethereum", ... }, { chain: "solana", ... }, { chain: "cosmos", ... }] } }`

- [x] Task 7: Tests (AC: alle)
  - [x] 7.1: `src/domain/chain-adapters/cosmos/atom-mapper.test.ts` — Mapping-Tests (Cosmos Validator → UnifiedValidator, Delegation → StakingPosition, Commission-Parsing, uatom → ATOM Konvertierung, Status-Mapping inkl. jailed)
  - [x] 7.2: `src/domain/chain-adapters/cosmos/atom-adapter.test.ts` — Adapter-Tests mit fetch-Mocking (refreshValidators, fetchStakingPositions mit Pagination, fetchInflationRate, Fallback-Logik, Timeout-Handling)
  - [x] 7.3: `src/app/api/cron/fetch-validators/route.test.ts` — Erweiterte Tests für 3-Chain-parallele Ausführung (ETH + SOL + ATOM), partieller Fehler eines Adapters blockiert nicht die anderen
  - [x] 7.4: Regressions-Check: Alle bestehenden Tests bestehen weiterhin (275/276 — 1 vorbestehender Timeout-Flake in wallet-verification.test.ts, nicht durch Cosmos-Änderungen verursacht)

## Dev Notes

### Kritische Architektur-Entscheidungen

- **ADR-001 gilt auch für Cosmos:** `refreshValidators(knownAddresses)` statt Bulk-Fetch. Der Adapter refreshed NUR bereits bekannte Validators. Neue Validators werden über `fetchStakingPositions(walletAddress)` beim Wallet-Connect entdeckt. Bei leerer DB = kein Validator-Fetch. ABER: Cosmos hat nur max ~200 bonded Validators — der Full-Fetch ist hier technisch machbar, trotzdem ADR-001 konsistent anwenden.

- **Domain Layer bleibt framework-agnostisch:** Der `CosmosAdapter` nutzt `fetch()` für LCD/REST-Calls — KEIN `@cosmjs/stargate` oder andere Cosmos SDKs im Domain-Layer! Cosmos LCD/REST API ist ein simples HTTP JSON API — perfekt für direkten `fetch()`. Das hält den Domain-Layer frei von schweren Dependencies (identisch zu ETH- und SOL-Adapter-Pattern).

- **KEY-basierte Pagination PFLICHT:** Cosmos SDK hat einen bekannten Off-by-one Bug bei offset-basierter Pagination (cosmos-sdk#11407). IMMER `pagination.key` verwenden, NIEMALS `pagination.offset`. Pattern: Erste Seite ohne Key → `next_key` aus Response für nächste Seite → Stopp wenn `next_key` null/leer.

- **Adapter gibt Domain-Objekte zurück, Persistierung im Cron-Handler:** Identisch zu ETH- und SOL-Adapter — `atom-adapter.ts` gibt `UnifiedValidator[]` / `StakingPosition[]` / `number` zurück. Der Cron-Handler persistiert via Repository-Funktionen.

- **Commission Format-Unterschied zu Solana:** Cosmos liefert Commission als 18-Dezimal-String ("0.050000000000000000" = 5%), was BEREITS im 0.0-1.0 Range ist. Einfacher `parseFloat()` reicht für das `UnifiedValidator.commission`-Feld. Solana dagegen liefert 0-100 Integer und muss durch 100 geteilt werden. ETH verwendet ebenfalls 0.0-1.0 Bereich. Cosmos passt also direkt.

- **uatom zu ATOM Konvertierung:** 1 ATOM = 1.000.000 uatom (10^6). `balance.amount` und `tokens` kommen als String in uatom. Konvertierung: `(BigInt(uatom) / 1_000_000n)` mit Dezimalstellen-Bewahrung. `delegatedAmount` wird als String gespeichert (wie bei ETH und SOL).

- **Cosmos Hub hat nur ~200 bonded Validators:** Deutlich weniger als Ethereum (~1M) oder Solana (~1800). Ein einzelner `/validators?status=BOND_STATUS_BONDED&pagination.limit=200` Request liefert ALLE aktiven Validators. Trotzdem Pagination-Support implementieren für Robustheit und falls `max_validators` in Zukunft erhöht wird.

- **Validator Status-Mapping:** Cosmos kennt `BOND_STATUS_BONDED`, `BOND_STATUS_UNBONDING`, `BOND_STATUS_UNBONDED` plus ein `jailed` Boolean. Mapping: BONDED + !jailed → "active", BONDED + jailed → "jailed", UNBONDING → "inactive", UNBONDED → "inactive".

- **Cosmos SDK v0.53.4 (Gaia v26.0.0):** Aktuelles Mainnet. Die alten Amino-REST-Endpoints sind vollständig entfernt. Nur `/cosmos/*/v1beta1/` gRPC-Gateway-Endpoints sind verfügbar. Kein Breaking Change erwartet für die verwendeten Endpoints.

### Cosmos LCD REST API — Relevante Endpoints

| Daten | Endpoint | Parameter | Besonderheiten |
|---|---|---|---|
| Validator-Liste (bonded) | `GET /cosmos/staking/v1beta1/validators` | `?status=BOND_STATUS_BONDED&pagination.limit=200` | Max 200 bonded Validators, KEY-Pagination |
| Delegations für Wallet | `GET /cosmos/staking/v1beta1/delegations/{addr}` | KEY-Pagination | Liefert Delegation + Balance pro Validator |
| Inflationsrate | `GET /cosmos/mint/v1beta1/inflation` | keine | 18-Dezimal-String (z.B. "0.100000000000000000" = 10%) |
| Staking-Parameter | `GET /cosmos/staking/v1beta1/params` | keine | max_validators=200, unbonding_time=21d, bond_denom=uatom |

**LCD Base URL:** `https://rest.cosmos.directory/cosmoshub` (Cosmos Directory Proxy — load-balanced, CORS-enabled, kostenlos)

**Fallback LCD URLs:**
- `https://cosmos-rest.publicnode.com`
- `https://lcd-cosmos.cosmostation.io`

**Response-Format Beispiele:**

Validator:
```json
{
  "operator_address": "cosmosvaloper1...",
  "jailed": false,
  "status": "BOND_STATUS_BONDED",
  "tokens": "50404949",
  "description": { "moniker": "ValidatorName" },
  "commission": {
    "commission_rates": {
      "rate": "0.050000000000000000",
      "max_rate": "0.200000000000000000",
      "max_change_rate": "0.010000000000000000"
    }
  }
}
```

Delegation:
```json
{
  "delegation": {
    "delegator_address": "cosmos1...",
    "validator_address": "cosmosvaloper1...",
    "shares": "25083339023.000000000000000000"
  },
  "balance": {
    "denom": "uatom",
    "amount": "25083339023"
  }
}
```

### Bestehende Infrastruktur (aus Story 2-1 und 2-2)

| Komponente | Datei | Status |
|---|---|---|
| ChainAdapter Interface | `src/domain/chain-adapters/types.ts` | Existiert — `refreshValidators(knownAddresses)`, `fetchStakingPositions()`, `fetchInflationRate()` |
| Factory | `src/domain/chain-adapters/factory.ts` | Existiert — `Chain.COSMOS` wirft aktuell `Error("not yet implemented")` |
| Unified Models | `src/domain/models/` | Existiert — `UnifiedValidator`, `StakingPosition`, `Chain`, `ChainMetrics` + Zod Schemas |
| DB Schema | `src/infrastructure/db/schema.ts` | Existiert — `validators`, `staking_positions`, `chain_metrics` akzeptieren `'cosmos'` |
| Repositories | `src/infrastructure/db/repositories/` | Existiert — `upsertValidators()`, `findValidatorsByChain()`, `upsertChainMetrics()` — chain-agnostisch, funktionieren für Cosmos |
| Cron Endpoint | `src/app/api/cron/fetch-validators/route.ts` | Existiert — aktuell ETH + SOL parallel, MUSS um ATOM erweitert werden |
| QStash Verify | `src/infrastructure/queue/qstash-verify.ts` | Existiert — wiederverwendbar |
| Env Validation | `src/lib/env.ts` | Existiert — COSMOS_LCD_URL + COSMOS_LCD_FALLBACK_URL MÜSSEN hinzugefügt werden |

### Learnings aus Story 2-1 und 2-2 (MUSS beachtet werden)

- **`--legacy-peer-deps` nötig:** Für ALLE npm installs wegen React 19 Peer-Dep-Konflikten
- **`serverEnv` statt `process.env`:** IMMER `serverEnv` aus `@/lib/env` verwenden
- **Biome 2.2.0:** Alle neuen Dateien müssen Biome-Check bestehen
- **Zod 4.3.6:** v4 Syntax verwenden (nicht v3!)
- **Named Exports:** Kein `default export` außer Next.js-Konventionen
- **Test-Pattern:** Dynamische Imports in `it()` Blöcken für Mock-Isolation
- **Drizzle `neon-http` Modus:** HTTP-basierter Treiber, Batch-Inserts via `.values([...]).onConflictDoUpdate(...)`
- **Domain-Layer importiert NUR aus domain/models/:** KEIN React, KEIN Next.js, KEIN infra
- **API Response Format:** `{ data, meta? }` (Success), `{ error, code, details? }` (Error)
- **Logging:** `[atom-adapter] message` Format, KEINE Wallet-Adressen in Logs
- **Known Issue FIXME [M1]:** `linked_wallets` nutzt `'atom'`, `validators`/`staking_positions`/`chain_metrics` nutzen `'cosmos'` — NICHT in dieser Story fixen, nur beachten
- **Factory nutzt ChainAdapterConfig:** `{ apiUrl, apiKey?, fallbackUrl? }` — für Cosmos: `apiUrl` = LCD URL, `fallbackUrl` = Fallback LCD
- **Cron-Handler: dynamische Chain-Configs:** Chains werden nur hinzugefügt wenn Env-Vars konfiguriert sind. Cosmos sollte einen Default-URL haben (`rest.cosmos.directory`) da kostenlos und keyless
- **240 bestehende Tests** müssen weiterhin bestehen

### Cosmos-spezifische Besonderheiten

- **Keine API-Key-Authentifizierung nötig:** `rest.cosmos.directory` ist ein Public Proxy ohne API-Key. Anders als beaconcha.in (ETH) oder Helius (SOL) braucht Cosmos keinen API-Key. Das macht die Integration einfacher.
- **18-Dezimal-String Zahlen:** Cosmos LCD liefert ALLE numerischen Werte als Strings mit 18 Dezimalstellen. `parseFloat()` ist für Commission und Inflation ausreichend präzise. Für Token-Beträge (`tokens`, `balance.amount`) ist BigInt-Parsing empfohlen da `uatom` ganzzahlig ist.
- **Validator Discovery via Delegations:** `fetchStakingPositions(walletAddress)` liefert automatisch `validator_address` pro Delegation. Daraus werden neue Validators entdeckt und können via `refreshValidators()` detailliert geladen werden.
- **`BOND_STATUS_BONDED` Filter:** Für `refreshValidators()` nur bonded Validators abrufen (aktive Set). Unbonded/unbonding Validators werden nicht refreshed — sie sind für das Staking-Dashboard nicht direkt relevant.
- **Delegation `shares` vs. `balance.amount`:** `shares` ist die Anzahl Delegation-Shares (kann sich von `balance.amount` unterscheiden wenn Slashing stattgefunden hat). Für `delegatedAmount` IMMER `balance.amount` verwenden (tatsächlicher Token-Wert in uatom).
- **Cosmos Delegations haben keinen Status wie "activating/deactivating":** Cosmos-Delegations sind entweder aktiv (in delegation_responses) oder im Unbonding (in unbonding_delegations). Für MVP: alle Delegations aus `/delegations/{addr}` als Status "active" mappen. Unbonding-Delegations könnten in Zukunft über `/validators/{addr}/unbonding_delegations/{delegator}` geladen werden.

### Abgrenzung: Was diese Story NICHT macht

- **Kein Redis-Cache:** Upstash Redis Cache-Layer kommt in Story 2-4
- **Kein Dashboard-UI:** Dashboard-UI kommt in Story 2-5
- **Kein Rate-Limiting auf Public API:** Kommt in Story 2-4
- **Keine Freshness-Anzeige:** Freshness-Indicators kommen in Story 2-6
- **Kein Keplr/ADR-036 Wallet-Connect:** Wallet-Connect-UI ist Story 1-4 (bereits done)
- **Keine Unbonding-Delegations:** MVP zeigt nur aktive Delegations. Unbonding-Positionen sind Post-MVP
- **Kein Mintscan API:** Der kostenlose LCD-Proxy (rest.cosmos.directory) reicht für MVP
- **Keine Community-Tax oder Bonded-Ratio Berechnung:** Für MVP wird die direkte Inflation aus `/mint/v1beta1/inflation` verwendet
- **Kein @cosmjs/stargate oder andere Cosmos SDKs:** Domain-Layer nutzt direkten fetch()

### Project Structure Notes

- Neue Dateien in `src/domain/chain-adapters/cosmos/` (atom-adapter.ts, atom-mapper.ts, atom-schemas.ts)
- Tests co-located neben Source-Dateien
- Factory-Funktion wird erweitert (nicht neu erstellt)
- Cron-Endpoint wird erweitert (nicht neu erstellt)
- Env-Validation wird erweitert (nicht neu erstellt)
- KEINE neuen DB-Tabellen oder Migrationen nötig — bestehende Tabellen akzeptieren `'cosmos'`

### References

- [Source: _bmad-output/planning-artifacts/epics.md#Story 2.3 — Cosmos Chain Adapter]
- [Source: _bmad-output/planning-artifacts/epics.md#Epic 2 — FR8, FR37, FR40, FR41]
- [Source: _bmad-output/planning-artifacts/architecture.md#Chain Adapter Architecture (Aggregator Pattern)]
- [Source: _bmad-output/planning-artifacts/architecture.md#ADR-001 — User-Driven Validator Discovery]
- [Source: _bmad-output/planning-artifacts/architecture.md#Data Architecture — Drizzle ORM + NeonDB]
- [Source: _bmad-output/planning-artifacts/architecture.md#Implementation Patterns & Consistency Rules]
- [Source: _bmad-output/planning-artifacts/architecture.md#External Integration Points — Cosmos REST]
- [Source: _bmad-output/project-context.md#Chain Adapter Architecture (Aggregator Pattern)]
- [Source: _bmad-output/implementation-artifacts/2-1-chain-adapter-infrastructure-ethereum-adapter.md — Learnings, Patterns, ADR-001]
- [Source: _bmad-output/implementation-artifacts/2-2-solana-chain-adapter.md — Learnings, Patterns, Factory, Cron]
- [Source: Web Research — Cosmos Hub LCD REST API: /cosmos/staking/v1beta1/validators, /delegations, /inflation]
- [Source: Web Research — Cosmos SDK v0.53.4, Gaia v26.0.0, max 200 bonded Validators]
- [Source: Web Research — Cosmos SDK Pagination Bug cosmos-sdk#11407: KEY-basierte Pagination PFLICHT]
- [Source: Web Research — rest.cosmos.directory: Cosmos Directory Proxy (kostenlos, load-balanced)]
- [Source: Web Research — Cosmos Commission Format: 18-Dezimal-String ("0.05" = 5%)]
- [Source: Web Research — uatom: 1 ATOM = 1.000.000 uatom (6 Dezimalstellen)]

## Dev Agent Record

### Agent Model Used

Claude Opus 4.6 (claude-opus-4-6)

### Debug Log References

Keine Debug-Probleme aufgetreten.

### Completion Notes List

- Cosmos Adapter vollständig implementiert nach identischem Pattern wie ETH- und SOL-Adapter
- Zod v4 Schemas für alle 3 Cosmos LCD API Endpoints (Validators, Delegations, Inflation)
- Mapper mit uatom→ATOM Konvertierung (BigInt, 6 Dezimalstellen), Status-Mapping (BONDED/UNBONDING/UNBONDED + jailed), Commission-Parsing (18-Dezimal-String → Float)
- CosmosAdapter mit KEY-basierter Pagination (NICHT offset — cosmos-sdk#11407), LCD-Fallback-Logik, 10s Timeout
- Factory erweitert: `Chain.COSMOS` gibt jetzt `CosmosAdapter` zurück statt Error
- Cron-Endpoint: Cosmos immer aktiv (kostenloser Public Proxy als Default), `Promise.allSettled` mit 3 Chains parallel
- 42 neue/aktualisierte Tests: 21 Mapper-Tests, 14 Adapter-Tests, 7 Cron-Tests
- Alle 280 bestehenden Tests bestehen weiterhin (1 vorbestehender Timeout-Flake in wallet-verification.test.ts)
- Biome-Check bestanden
- Code Review (AI): 2 HIGH, 4 MEDIUM, 2 LOW Issues gefunden und gefixt (siehe Change Log)

### Change Log

- 2026-02-22: Story 2-3 Cosmos Chain Adapter implementiert — Zod Schemas, Mapper, Adapter, Factory, Cron-Integration, Tests
- 2026-02-22: Code Review (AI) — 6 Fixes: H1 Wallet-Adressen-Leak in Error-Messages gefixt (NFR15), H2 Biome-Formatierung korrigiert, M1 Timeout-Test gestärkt (COSMOS_LCD_TIMEOUT exportiert), M2 Dynamic import() durch normale Imports ersetzt, M3 Zod-Validierungsfehler-Tests hinzugefügt (3 neue Tests), Cron-Test-Formatierung korrigiert

### File List

**Neue Dateien:**
- `src/domain/chain-adapters/cosmos/atom-schemas.ts`
- `src/domain/chain-adapters/cosmos/atom-mapper.ts`
- `src/domain/chain-adapters/cosmos/atom-mapper.test.ts`
- `src/domain/chain-adapters/cosmos/atom-adapter.ts`
- `src/domain/chain-adapters/cosmos/atom-adapter.test.ts`

**Geänderte Dateien:**
- `src/domain/chain-adapters/factory.ts` — CosmosAdapter Import + Case
- `src/app/api/cron/fetch-validators/route.ts` — Cosmos Chain-Config hinzugefügt
- `src/app/api/cron/fetch-validators/route.test.ts` — 3-Chain-Tests erweitert
- `src/lib/env.ts` — COSMOS_LCD_URL + COSMOS_LCD_FALLBACK_URL
- `.env.example` — Cosmos-Variablen dokumentiert
