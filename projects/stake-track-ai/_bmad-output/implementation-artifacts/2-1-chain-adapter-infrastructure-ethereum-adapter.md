# Story 2.1: Chain Adapter Infrastructure & Ethereum Adapter

Status: done

<!-- Note: Validation is optional. Run validate-create-story for quality check before dev-story. -->
<!-- Generated: 2026-02-22 | Epic: 2 | Story: 1 | Key: 2-1-chain-adapter-infrastructure-ethereum-adapter -->

## Story

As a System-Betreiber,
I want dass Ethereum-Validator- und Staking-Daten automatisch stündlich abgerufen und normalisiert gespeichert werden,
So that das Dashboard aktuelle ETH-Staking-Positionen anzeigen kann.

## Acceptance Criteria

1. **Given** die Chain Adapter Infrastructure wird aufgebaut **When** die Implementierung abgeschlossen ist **Then** existiert ein `ChainAdapter` Interface in `src/domain/chain-adapters/types.ts` mit Methoden: `fetchValidators()`, `fetchStakingPositions(walletAddress)`, `fetchInflationRate()` **And** existieren Unified Domain Models in `src/domain/models/`: `UnifiedValidator`, `StakingPosition`, `Chain` enum **And** alle Models haben korrespondierende Zod Schemas für Runtime-Validierung

2. **Given** der Ethereum Adapter (`src/domain/chain-adapters/ethereum/`) **When** `fetchValidators()` aufgerufen wird **Then** werden Validator-Daten von der Beacon Chain API abgerufen **And** die Raw API Response wird per Zod Schema validiert **And** die Daten werden via `eth-mapper.ts` auf das `UnifiedValidator` Model gemappt **And** die Validators werden in der `validators`-Tabelle gespeichert (upsert)

3. **Given** der Ethereum Adapter **When** `fetchStakingPositions(walletAddress)` aufgerufen wird **Then** werden die Staking-Positionen für die gegebene ETH-Adresse abgerufen **And** die Positionen werden in der `staking_positions`-Tabelle gespeichert

4. **Given** der Ethereum Adapter **When** `fetchInflationRate()` aufgerufen wird **Then** wird die aktuelle ETH-Inflationsrate abgerufen und in der `chain_metrics`-Tabelle gespeichert (FR38)

5. **Given** ein QStash Cron-Job für `/api/cron/fetch-validators` **When** der Cron stündlich feuert (FR37) **Then** wird der ETH-Adapter aufgerufen **And** die HMAC-Signatur des QStash-Webhooks wird verifiziert (FR41) **And** bei Erfolg wird der Cache invalidiert (FR40) **And** bei Fehler loggt das System `[eth-adapter] Fetch failed: {error}` und QStash retried automatisch

6. **Given** die Beacon Chain API ist nicht erreichbar **When** ein Timeout oder Fehler auftritt **Then** werden die anderen Chain Adapters NICHT beeinträchtigt (NFR28, Isolation) **And** der Fehler wird geloggt ohne Wallet-Adressen

## Tasks / Subtasks

- [x] Task 1: Unified Domain Models & Zod Schemas erstellen (AC: #1)
  - [x] 1.1: `src/domain/models/chain.ts` — `Chain` enum (`ethereum`, `solana`, `cosmos`), `ChainConfig` Type
  - [x] 1.2: `src/domain/models/validator.ts` — `UnifiedValidator` Interface + `unifiedValidatorSchema` Zod Schema
  - [x] 1.3: `src/domain/models/position.ts` — `StakingPosition` Interface + `stakingPositionSchema` Zod Schema
  - [x] 1.4: `src/domain/models/common.ts` — `ApiResponse<T>`, `ApiError`, `PaginationMeta` Types
  - [x] 1.5: `src/domain/models/chain-metrics.ts` — `ChainMetrics` Interface + Zod Schema

- [x] Task 2: ChainAdapter Interface definieren (AC: #1)
  - [x] 2.1: `src/domain/chain-adapters/types.ts` — `ChainAdapter` Interface mit `fetchValidators()`, `fetchStakingPositions(walletAddress: string)`, `fetchInflationRate()`
  - [x] 2.2: Adapter-Factory-Funktion `getChainAdapter(chain: Chain): ChainAdapter`

- [x] Task 3: DB Schema erweitern — 3 neue Tabellen (AC: #2, #3, #4)
  - [x] 3.1: `validators`-Tabelle in `src/infrastructure/db/schema.ts` hinzufügen
  - [x] 3.2: `staking_positions`-Tabelle hinzufügen (FK → users, FK → validators)
  - [x] 3.3: `chain_metrics`-Tabelle hinzufügen
  - [x] 3.4: Drizzle Migration generieren: `npx drizzle-kit generate`
  - [x] 3.5: Migration committen (NICHT `drizzle-kit push` in Production!)

- [x] Task 4: Repositories für neue Tabellen erstellen (AC: #2, #3, #4)
  - [x] 4.1: `src/infrastructure/db/repositories/validator-repository.ts` — `upsertValidators()`, `findValidatorsByChain()`, `findValidatorByAddress()`
  - [x] 4.2: `src/infrastructure/db/repositories/staking-position-repository.ts` — `upsertPositions()`, `findPositionsByUserId()`, `findPositionsByChain()`
  - [x] 4.3: `src/infrastructure/db/repositories/chain-metrics-repository.ts` — `upsertChainMetrics()`, `findLatestByChain()`

- [x] Task 5: Environment Variables für Chain APIs ergänzen (AC: #2)
  - [x] 5.1: `src/lib/env.ts` erweitern — `BEACON_API_URL` (required), `BEACONCHAIN_API_KEY` (optional), `ETHEREUM_RPC_URL` (optional, für Execution-Layer-Queries)
  - [x] 5.2: `.env.example` aktualisieren

- [x] Task 6: Ethereum Adapter implementieren (AC: #2, #3, #4, #6)
  - [x] 6.1: `src/domain/chain-adapters/ethereum/eth-schemas.ts` — Zod Schemas für Beacon API Responses
  - [x] 6.2: `src/domain/chain-adapters/ethereum/eth-mapper.ts` — Mapping: Beacon API Response → `UnifiedValidator`, beaconcha.in Response → `StakingPosition`
  - [x] 6.3: `src/domain/chain-adapters/ethereum/eth-adapter.ts` — `ChainAdapter` Implementierung mit `fetchValidators()`, `fetchStakingPositions()`, `fetchInflationRate()`

- [x] Task 7: QStash Cron-Endpoint & HMAC-Verifizierung (AC: #5)
  - [x] 7.1: `src/infrastructure/queue/qstash-verify.ts` — HMAC Signature Verification Middleware
  - [x] 7.2: `src/app/api/cron/fetch-validators/route.ts` — POST Endpoint mit HMAC-Verifizierung, ETH-Adapter-Aufruf

- [x] Task 9: ADR-001 umsetzen — User-Driven Validator Discovery (AC: #2, #5)
  - [x] 9.1: `ChainAdapter` Interface ändern: `fetchValidators()` → `refreshValidators(knownAddresses: string[])` in `src/domain/chain-adapters/types.ts`
  - [x] 9.2: `EthereumAdapter` umbauen: `refreshValidators()` nutzt beaconcha.in Batch-API (`/api/v1/validator/{idx1},{idx2},...`) statt Raw Beacon API Bulk-Fetch. Beacon API Endpoint `/eth/v1/beacon/states/finalized/validators` komplett entfernen.
  - [x] 9.3: Cron-Handler (`route.ts`) anpassen: Vor Adapter-Aufruf bekannte Validators aus DB laden (`findValidatorsByChain("ethereum")`), deren Indices an `refreshValidators()` übergeben. Leere DB = kein Fetch (Validators werden über Wallet-Connect entdeckt).
  - [x] 9.4: Tests aktualisieren: eth-adapter.test.ts, route.test.ts für neues Pattern. Neuer Test: leere DB → kein API-Call.
  - [x] 9.5: Referenz: ADR-001 in `_bmad-output/planning-artifacts/architecture.md`

- [x] Task 8: Tests (AC: alle)
  - [x] 8.1: `src/domain/chain-adapters/ethereum/eth-mapper.test.ts` — Mapping-Tests (Raw → Unified)
  - [x] 8.2: `src/domain/chain-adapters/ethereum/eth-adapter.test.ts` — Adapter-Tests mit fetch-Mocking
  - [x] 8.3: Repository-Tests für validator-repository, staking-position-repository, chain-metrics-repository
  - [x] 8.4: `src/app/api/cron/fetch-validators/route.test.ts` — Cron-Endpoint-Tests (HMAC, Success, Error)
  - [x] 8.5: Regressions-Check: Alle bestehenden 153 Tests bestehen weiterhin (191 total nach Task 9)

## Dev Notes

### Kritische Architektur-Entscheidungen

- **Domain Layer wird ERSTMALS befüllt:** Bisher existieren in `src/domain/` nur `.gitkeep`-Stubs. Diese Story erstellt die ersten echten Domain-Models und die Chain-Adapter-Infrastruktur. Die DDD-Layer-Boundaries (domain darf NUR domain/models importieren, KEIN React, KEIN Next.js, KEIN infra) müssen von Anfang an strikt eingehalten werden.

- **ChainAdapter Interface ist FRAMEWORK-AGNOSTISCH:** Das Interface in `src/domain/chain-adapters/types.ts` darf KEINE Abhängigkeiten auf Infrastructure-Layer haben. Die HTTP-Fetch-Logik lebt im Adapter selbst (der globale `fetch()` ist framework-agnostisch und in Node.js 18+ nativ verfügbar). Die DB-Persistierung erfolgt NICHT im Adapter — der Adapter gibt nur Domain-Objekte zurück. Die Persistierung geschieht im Cron-Handler (Application Layer).

- **Adapter gibt Domain-Objekte zurück, Persistierung im Cron-Handler:** Der `eth-adapter.ts` nutzt `fetch()` für API-Calls und gibt `UnifiedValidator[]` / `StakingPosition[]` / `number` (Inflation Rate) zurück. Der Cron-Handler in `src/app/api/cron/fetch-validators/route.ts` ruft den Adapter auf und persistiert die Ergebnisse via Repository-Funktionen. So bleibt der Domain-Layer frei von Infrastructure-Imports.

- **Beacon API hat KEINEN Withdrawal-Address-Filter:** Man kann NICHT direkt "alle Validators für Wallet 0x..." abfragen. Für Wallet-to-Validator-Lookups wird beaconcha.in `/validator/eth1/{eth1address}` verwendet. Die `fetchValidators()`-Methode holt alle aktiven Validators (Bulk), während `fetchStakingPositions(walletAddress)` beaconcha.in für den Wallet-Lookup nutzt.

- **Pectra Hard Fork (Mai 2025) — maxEB = 2048 ETH:** `effective_balance` kann jetzt 32e9 bis 2048e9 Gwei betragen. NIEMALS 32 ETH als Konstante hardcoden. Withdrawal Credentials `0x02` (Compounding) sind neu neben `0x00` (BLS) und `0x01` (Execution Address). Der Mapper muss alle 3 Typen korrekt parsen.

- **Web3.js wurde März 2025 archiviert:** NICHT verwenden. Für Beacon API: direkter `fetch()`. Für Execution Layer: `viem` (bereits als wagmi-Dependency im Projekt).

- **beaconcha.in Free-Tier: ~1K Requests/Monat:** Für Production-Betrieb unbrauchbar. Entweder beaconcha.in Paid Plan oder alternatives API (Etherscan Beacon Chain API, eigene Lighthouse Node). Im MVP: beaconcha.in mit API-Key, aggressive Caching (5 Min TTL Validators), tägliches Bulk-Fetch statt Einzel-Requests.

- **ETH-Inflationsrate ist KEINE direkte API-Abfrage:** Es gibt keinen Beacon-API-Endpoint für die Inflationsrate. Die Methode ist: beaconcha.in `/ethstore/latest` liefert den ETH.STORE daily reference staking yield, oder die Inflationsrate wird aus `BASE_REWARD_FACTOR`, `BASE_REWARDS_PER_EPOCH` und der Total Active Balance abgeleitet. Für MVP: beaconcha.in `/ethstore/latest` als Quelle für den effektiven Staking-APY verwenden.

- **SIMD-123 (Solana) — Nicht in dieser Story, aber vorplanen:** SIMD-123 führt eine zweite Commission (`block_revenue_commission`) ein. Das `UnifiedValidator`-Model muss so designed sein, dass es chain-spezifische Zusatzfelder über ein `raw_data` JSONB-Feld aufnehmen kann. Die `commission`-Spalte bleibt die primäre/Inflation Commission.

- **Cosmos LCD API — Stabile Endpoints:** `/cosmos/staking/v1beta1/validators` und `/cosmos/staking/v1beta1/delegations/{address}` sind seit Cosmos SDK v0.46 unverändert. Pagination via `pagination.key` (Cursor), NICHT `pagination.offset` (bekannter Bug). Nicht in dieser Story, aber das Interface muss für Cosmos passen.

### Bestehende Infrastruktur (aus Epic 1)

| Komponente | Datei | Status |
|---|---|---|
| DB Schema | `src/infrastructure/db/schema.ts` | Existiert (users + linked_wallets + verification_tokens) — MUSS um 3 Tabellen erweitert werden |
| DB Client | `src/infrastructure/db/index.ts` | Existiert — Drizzle `neon-http` Modus, Singleton `db` |
| User Repository | `src/infrastructure/db/repositories/user-repository.ts` | Existiert — 8 Funktionen |
| Linked-Wallet Repository | `src/infrastructure/db/repositories/linked-wallet-repository.ts` | Existiert — 5 Funktionen |
| Env Validation | `src/lib/env.ts` | Existiert — Server/Client Zod Schemas, MUSS um Chain-API-Vars erweitert werden |
| `.env.example` | `.env.example` | Existiert — MUSS aktualisiert werden |
| QStash (Stub) | `src/infrastructure/queue/` | Nur `.gitkeep` — QStash-Verify muss erstellt werden |
| Cache (Stub) | `src/infrastructure/cache/` | Nur `.gitkeep` — Redis kommt in Story 2-4, hier noch KEIN Caching |
| Domain Models (Stub) | `src/domain/models/` | Nur `.gitkeep` — alle Models werden neu erstellt |
| Chain Adapters (Stub) | `src/domain/chain-adapters/` | Nur `.gitkeep` — alle Adapter-Dateien werden neu erstellt |

### Learnings aus Epic 1 (MUSS beachtet werden)

- **`--legacy-peer-deps` nötig:** Für ALLE npm installs wegen React 19 Peer-Dep-Konflikten
- **`serverEnv` statt `process.env`:** IMMER `serverEnv` aus `@/lib/env` verwenden
- **Biome 2.2.0:** Alle neuen Dateien müssen Biome-Check bestehen
- **Next.js 15.5.12 (LTS), Tailwind v4, React 19.1.0**
- **Drizzle 0.45.1, drizzle-kit 0.31.9, @neondatabase/serverless 1.0.2**
- **Zod 4.3.6:** v4 Syntax verwenden (nicht v3!)
- **Drizzle `neon-http` Modus:** HTTP-basierter Treiber. Für Batch-Inserts: Drizzle `db.insert(...).values([...]).onConflictDoUpdate(...)` verwenden
- **Named Exports:** Kein `default export` außer Next.js-Konventionen (`page.tsx`, `layout.tsx`, `route.ts`)
- **Test-Pattern: Dynamische Imports in `it()` Blöcken:** Module innerhalb von Tests importieren (`const { POST } = await import("./route")`) für korrekte Mock-Isolation
- **In-Memory Rate-Limiting:** Upstash Rate Limit nicht installiert. Gleichen In-Memory-Fallback verwenden
- **Provider-Hierarchy:** Web3Provider > AuthProvider > SolanaProvider > ThemeProvider — NICHT ändern
- **126 bestehende Tests** müssen weiterhin bestehen (21 Test-Dateien)
- **Cascade Delete beachten:** Neue Tabellen mit FK auf `users` MÜSSEN `ON DELETE CASCADE` haben (für DSGVO Account-Löschung aus Story 1-6)

### DB Schema Design

**`validators`-Tabelle:**
```sql
CREATE TABLE validators (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  chain VARCHAR(10) NOT NULL CHECK (chain IN ('ethereum', 'solana', 'cosmos')),
  address VARCHAR(255) NOT NULL,              -- Validator pubkey/address (chain-specific format)
  name VARCHAR(255),                          -- Validator name (moniker)
  commission NUMERIC(10, 6) NOT NULL,         -- Commission rate (0.0 to 1.0)
  uptime NUMERIC(5, 2),                       -- Uptime percentage (0-100)
  status VARCHAR(50) NOT NULL DEFAULT 'active', -- active, inactive, slashed, jailed
  is_active BOOLEAN NOT NULL DEFAULT true,
  raw_data JSONB,                             -- Chain-specific extra data
  fetched_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  UNIQUE (chain, address)
);
```

**`staking_positions`-Tabelle:**
```sql
CREATE TABLE staking_positions (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  chain VARCHAR(10) NOT NULL CHECK (chain IN ('ethereum', 'solana', 'cosmos')),
  validator_id UUID REFERENCES validators(id) ON DELETE SET NULL,
  wallet_address VARCHAR(255) NOT NULL,
  delegated_amount NUMERIC(30, 18) NOT NULL,  -- In native token units (ETH, SOL, ATOM)
  status VARCHAR(50) NOT NULL DEFAULT 'active', -- active, activating, deactivating, inactive
  fetched_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);
```

**`chain_metrics`-Tabelle:**
```sql
CREATE TABLE chain_metrics (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  chain VARCHAR(10) NOT NULL CHECK (chain IN ('ethereum', 'solana', 'cosmos')),
  inflation_rate NUMERIC(10, 6),              -- Annual inflation rate
  total_staked NUMERIC(30, 18),               -- Total staked in native token
  avg_apy NUMERIC(10, 6),                     -- Average APY for chain
  fetched_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);
```

**WICHTIG:** `staking_positions.user_id` hat `ON DELETE CASCADE` → Account-Löschung (Story 1-6) löscht automatisch alle Positionen.
**WICHTIG:** `staking_positions.validator_id` hat `ON DELETE SET NULL` → Validator-Cleanup bricht keine Positionen.

### ChainAdapter Interface Design

```typescript
// src/domain/chain-adapters/types.ts
import type { Chain } from '@/domain/models/chain';
import type { UnifiedValidator } from '@/domain/models/validator';
import type { StakingPosition } from '@/domain/models/position';

export interface ChainAdapter {
  readonly chain: Chain;
  fetchValidators(): Promise<UnifiedValidator[]>;
  fetchStakingPositions(walletAddress: string): Promise<StakingPosition[]>;
  fetchInflationRate(): Promise<number>;
}
```

**ACHTUNG:** Das Interface importiert NUR aus `domain/models/` — KEINE infra-Imports!

### Ethereum Adapter — API Strategy

| Daten | API-Quelle | Endpoint | Rate Limit |
|---|---|---|---|
| Validator-Liste (Bulk) | Beacon API | `GET /eth/v1/beacon/states/finalized/validators?status[]=active_ongoing` | Kein offizielles Limit (eigene Node oder Public) |
| Validator für Wallet | beaconcha.in | `GET /api/v1/validator/eth1/{eth1address}` | ~1K Requests/Monat (Free), API-Key empfohlen |
| Staking-Positionen | beaconcha.in | `GET /api/v1/validator/{index1},{index2},...` (Batch bis 100) | Wie oben |
| Attestation-Performance | beaconcha.in | `GET /api/v1/validator/{index}/attestationeffectiveness` | Wie oben |
| Slashing-Historie | beaconcha.in | `GET /api/v1/validator/{index}/slashings` | Wie oben |
| ETH Staking Yield | beaconcha.in | `GET /api/v1/ethstore/latest` | Wie oben |
| Netzwerk-Stats | beaconcha.in | `GET /api/v1/epoch/latest` | Wie oben |

**Best Practices:**
- IMMER `finalized` State verwenden (nicht `head`) — cacheable und konsistent
- Batch Validator-Requests: Komma-separierte Indices (bis zu 100 pro Request)
- Epoch-Daten aggressiv cachen: Ändern sich nur alle ~6.4 Minuten
- beaconcha.in Auth: `?apikey=YOUR_KEY` oder `Authorization: Bearer YOUR_KEY`
- Timeouts definieren: 10s für Beacon API, 15s für beaconcha.in

### QStash HMAC Verification

```typescript
// src/infrastructure/queue/qstash-verify.ts
import { Receiver } from '@upstash/qstash';
import { serverEnv } from '@/lib/env';

export async function verifyQStashSignature(request: Request): Promise<boolean> {
  const receiver = new Receiver({
    currentSigningKey: serverEnv.QSTASH_CURRENT_SIGNING_KEY,
    nextSigningKey: serverEnv.QSTASH_NEXT_SIGNING_KEY,
  });
  const body = await request.text();
  const signature = request.headers.get('upstash-signature');
  if (!signature) return false;
  try {
    await receiver.verify({ body, signature });
    return true;
  } catch {
    return false;
  }
}
```

### Cron-Endpoint Flow

```
QStash Cron (stündlich)
  → POST /api/cron/fetch-validators
  → HMAC-Signatur verifizieren (verifyQStashSignature)
  → Bei ungültiger Signatur: 401 + Log "[cron] Invalid QStash signature"
  → ETH Adapter aufrufen:
    1. fetchValidators() → UnifiedValidator[]
    2. fetchInflationRate() → number
  → Validators via validator-repository upserten
  → Chain-Metrics via chain-metrics-repository upserten
  → Response: 200 { data: { chain: "ethereum", validatorsUpdated: N, metricsUpdated: true } }
  → Bei Fehler: 500 + Log "[eth-adapter] Fetch failed: {error}" → QStash retried automatisch
```

**HINWEIS:** `fetchStakingPositions()` wird NICHT im stündlichen Cron aufgerufen — Staking-Positionen sind user-spezifisch und werden on-demand geladen (beim Dashboard-Load oder nach Wallet-Connect). Der stündliche Cron holt nur die globalen Validator-Daten und Chain-Metriken.

### Naming-Konventionen (STRIKT beachten)

| Element | Konvention | Beispiel |
|---|---|---|
| DB-Tabellen | snake_case, Plural | `validators`, `staking_positions`, `chain_metrics` |
| DB-Spalten | snake_case | `wallet_address`, `fetched_at`, `inflation_rate` |
| Foreign Keys | `{tabelle_singular}_id` | `validator_id`, `user_id` |
| Dateien | kebab-case | `eth-adapter.ts`, `eth-mapper.ts`, `eth-schemas.ts` |
| Funktionen | camelCase | `fetchValidators()`, `upsertValidators()` |
| Types/Interfaces | PascalCase | `UnifiedValidator`, `StakingPosition`, `ChainAdapter` |
| Zod Schemas | camelCase + `Schema` | `unifiedValidatorSchema`, `beaconValidatorResponseSchema` |
| Konstanten | SCREAMING_SNAKE_CASE | `CACHE_TTL_VALIDATORS`, `BEACON_API_BASE_URL` |
| Exports | Named Exports | `export function fetchValidators()` — KEIN default export |

### Sicherheitsaspekte

- **HMAC auf Cron-Endpoint (NFR12):** QStash-Signatur MUSS verifiziert werden. Ohne gültige Signatur → 401.
- **Keine PII in Logs (NFR15):** Wallet-Adressen nur gekürzt loggen (`0x1234...5678`). Validator-Pubkeys sind öffentlich und dürfen vollständig geloggt werden.
- **Chain API Keys in Env Vars (NFR11):** `BEACONCHAIN_API_KEY` NUR serverseitig, NIEMALS im Client-Bundle.
- **Timeout-Handling (NFR31):** Definierte Timeouts für alle externen API-Calls. Beacon API: 10s, beaconcha.in: 15s.
- **Adapter-Isolation (NFR28):** ETH-Adapter-Fehler dürfen KEINE anderen Chain-Adapter beeinträchtigen. Jeder Adapter läuft in eigenem try/catch.
- **ON DELETE CASCADE auf user_id (DSGVO):** `staking_positions.user_id` MUSS Cascade Delete haben für Account-Löschung.

### Abgrenzung: Was diese Story NICHT macht

- **Kein Redis-Cache:** Upstash Redis Cache-Layer kommt in Story 2-4. Hier noch kein Caching.
- **Kein Dashboard-UI:** Dashboard-UI kommt in Story 2-5. Hier nur Backend-Infrastruktur.
- **Kein SOL/ATOM-Adapter:** Solana kommt in Story 2-2, Cosmos in Story 2-3.
- **Kein Rate-Limiting auf Public API:** Kommt in Story 2-4 mit Upstash Rate Limit.
- **Kein Fan-out:** QStash Fan-out (3 Chains parallel) kommt erst wenn alle 3 Adapter stehen (Story 2-3).
- **Keine Freshness-Anzeige:** Freshness-Indicators kommen in Story 2-6.
- **Keine TimescaleDB Hypertables:** `validators`, `staking_positions`, `chain_metrics` sind KEINE Hypertables. Hypertables kommen erst für `vrs_scores` (Story 4-1) und `yield_snapshots` (Story 3-1).
- **Kein beaconcha.in Paid Plan Setup:** API-Key-Beschaffung ist Prerequisite, nicht Teil der Implementierung.

### Project Structure Notes

- Alle neuen Domain-Dateien in `src/domain/models/` und `src/domain/chain-adapters/ethereum/`
- Neue Repositories in `src/infrastructure/db/repositories/`
- QStash-Verify in `src/infrastructure/queue/`
- Cron-Endpoint in `src/app/api/cron/fetch-validators/`
- Schema-Erweiterung in bestehender `src/infrastructure/db/schema.ts`
- Tests co-located neben Source-Dateien

### References

- [Source: _bmad-output/planning-artifacts/epics.md#Story 2.1 — Chain Adapter Infrastructure & Ethereum Adapter]
- [Source: _bmad-output/planning-artifacts/epics.md#Epic 2 — FR8, FR37, FR38, FR40, FR41]
- [Source: _bmad-output/planning-artifacts/architecture.md#Chain Adapter Architecture (Aggregator Pattern)]
- [Source: _bmad-output/planning-artifacts/architecture.md#Data Architecture — Drizzle ORM + NeonDB]
- [Source: _bmad-output/planning-artifacts/architecture.md#Implementation Patterns & Consistency Rules]
- [Source: _bmad-output/planning-artifacts/architecture.md#Structure Patterns — Domain Layer Organisation]
- [Source: _bmad-output/planning-artifacts/architecture.md#Architectural Boundaries — Layer-Regeln]
- [Source: _bmad-output/planning-artifacts/architecture.md#External Integration Points — Beacon API, QStash]
- [Source: _bmad-output/planning-artifacts/architecture.md#Data Flow — QStash Cron (stündlich)]
- [Source: _bmad-output/planning-artifacts/prd.md#FR37 — Stündliche Daten-Aktualisierung]
- [Source: _bmad-output/planning-artifacts/prd.md#FR38 — Tägliche Inflationsraten-Updates]
- [Source: _bmad-output/planning-artifacts/prd.md#FR41 — HMAC auf Cron-Endpoints]
- [Source: _bmad-output/planning-artifacts/prd.md#NFR28 — Chain Adapter Isolation]
- [Source: _bmad-output/planning-artifacts/prd.md#NFR31 — Definierte Timeouts]
- [Source: _bmad-output/project-context.md#Chain Adapter Architecture (Aggregator Pattern)]
- [Source: _bmad-output/project-context.md#Drizzle ORM (NICHT Prisma)]
- [Source: _bmad-output/project-context.md#Security Rules — HMAC, Keine PII in Logs]
- [Source: _bmad-output/implementation-artifacts/1-6-account-loeschung-dsgvo.md — Learnings, Cascade Delete, Test-Patterns]
- [Source: Web Research — Ethereum Beacon API Spec (github.com/ethereum/beacon-APIs)]
- [Source: Web Research — beaconcha.in API v1 Endpoints und Rate Limits]
- [Source: Web Research — Pectra Hard Fork Mai 2025: maxEB 2048 ETH, 0x02 Credentials]
- [Source: Web Research — Web3.js archiviert März 2025]
- [Source: Web Research — SIMD-123 Block Revenue Distribution (Solana, approved März 2025)]

## Dev Agent Record

### Agent Model Used

Claude Opus 4.6 (claude-opus-4-6)

### Debug Log References

- Biome-Fehler behoben: Import-Sortierung, unused imports, precision loss (Number.MAX_SAFE_INTEGER statt uint64)
- env.test.ts aktualisiert: BEACON_API_URL als required hinzugefügt
- validator-repository.test.ts: Mock-Struktur angepasst für Drizzle-Query ohne .limit()

### Completion Notes List

- Task 1: 5 Domain Models + Zod Schemas erstellt (chain, validator, position, common, chain-metrics) — 27 Tests
- Task 2: ChainAdapter Interface + Factory-Funktion
- Task 3: DB Schema um 3 Tabellen erweitert (validators, staking_positions, chain_metrics) + Migration generiert
- Task 4: 3 Repositories erstellt (validator, staking-position, chain-metrics)
- Task 5: BEACON_API_URL (required), BEACONCHAIN_API_KEY, ETHEREUM_RPC_URL zu env.ts + .env.example
- Task 6: Ethereum Adapter mit 3 API-Methoden (fetchValidators, fetchStakingPositions, fetchInflationRate)
- Task 7: QStash HMAC-Verify + POST /api/cron/fetch-validators Endpoint
- Task 8: 31 neue Tests (8 Mapper, 8 Adapter, 12 Repository, 3 Cron), alle 184 Tests bestanden
- @upstash/qstash als neue Dependency installiert (--legacy-peer-deps)
- Task 9: ADR-001 umgesetzt — fetchValidators() → refreshValidators(knownAddresses). Beacon API Bulk-Fetch entfernt, beaconcha.in Batch-API. Cron lädt bekannte Validators aus DB. Toter Code (BeaconValidator Schema/Mapper) entfernt. Neuer mapBeaconchainDetailToValidator Mapper. Alle 191 Tests bestanden.

### Change Log

- 2026-02-22: Story 2-1 implementiert — Chain Adapter Infrastructure & Ethereum Adapter (alle 8 Tasks abgeschlossen)
- 2026-02-22: Code Review — 12 Issues gefunden (4H, 5M, 3L), 9 auto-gefixt, 3 als TODO/FIXME dokumentiert
- 2026-02-22: Task 9 — ADR-001 User-Driven Validator Discovery umgesetzt (alle 191 Tests bestanden)
- 2026-02-22: Code Review #2 — 9 Issues gefixt (4H, 5M), 3 LOW als dokumentiert belassen. 193 Tests bestanden.
  - H1: ApiResponse/ApiError an Architecture-Standard angepasst (kein `success`-Feld, flaches Error-Format)
  - H2: replacePositionsForWallet in db.transaction() gewrapped (Race Condition behoben)
  - H3: insertChainMetrics → upsertChainMetrics (Delete-then-Insert in Transaction, verhindert unbegrenztes Tabellenwachstum)
  - H4: EthereumAdapter nutzt jetzt beaconchainBaseUrl-Parameter statt Hardcoding; BEACON_API_URL → optional
  - M3: Test für replacePositionsForWallet hinzugefügt (2 neue Tests)
  - M4: fetchStakingPositions nutzt jetzt Batch-API (BEACONCHAIN_BATCH_SIZE=100) für >100 Validators
  - M5: console.warn → console.error für ungültige QStash-Signatur (Security-Event)

### File List

**Neue Dateien:**
- src/domain/models/chain.ts
- src/domain/models/chain.test.ts
- src/domain/models/validator.ts
- src/domain/models/validator.test.ts
- src/domain/models/position.ts
- src/domain/models/position.test.ts
- src/domain/models/common.ts
- src/domain/models/chain-metrics.ts
- src/domain/models/chain-metrics.test.ts
- src/domain/chain-adapters/types.ts
- src/domain/chain-adapters/factory.ts
- src/domain/chain-adapters/ethereum/eth-schemas.ts
- src/domain/chain-adapters/ethereum/eth-mapper.ts
- src/domain/chain-adapters/ethereum/eth-mapper.test.ts
- src/domain/chain-adapters/ethereum/eth-adapter.ts
- src/domain/chain-adapters/ethereum/eth-adapter.test.ts
- src/infrastructure/db/repositories/validator-repository.ts
- src/infrastructure/db/repositories/validator-repository.test.ts
- src/infrastructure/db/repositories/staking-position-repository.ts
- src/infrastructure/db/repositories/staking-position-repository.test.ts
- src/infrastructure/db/repositories/chain-metrics-repository.ts
- src/infrastructure/db/repositories/chain-metrics-repository.test.ts
- src/infrastructure/queue/qstash-verify.ts
- src/app/api/cron/fetch-validators/route.ts
- src/app/api/cron/fetch-validators/route.test.ts
- drizzle/0004_round_blonde_phantom.sql

**Geänderte Dateien:**
- src/infrastructure/db/schema.ts (3 neue Tabellen + neue Imports)
- src/lib/env.ts (BEACON_API_URL, BEACONCHAIN_API_KEY, ETHEREUM_RPC_URL)
- src/lib/env.test.ts (BEACON_API_URL in Tests ergänzt)
- .env.example (Ethereum Chain API Variablen)
- package.json (+ @upstash/qstash)
- package-lock.json (Dependencies)
