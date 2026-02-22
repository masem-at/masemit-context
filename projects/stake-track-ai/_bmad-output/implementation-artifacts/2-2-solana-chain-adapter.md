# Story 2.2: Solana Chain Adapter

Status: done

<!-- Note: Validation is optional. Run validate-create-story for quality check before dev-story. -->
<!-- Generated: 2026-02-22 | Epic: 2 | Story: 2 | Key: 2-2-solana-chain-adapter -->

## Story

As a User mit SOL-Staking-Positionen,
I want dass meine Solana-Staking-Daten automatisch abgerufen werden,
So that mein SOL-Portfolio im Dashboard erscheint.

## Acceptance Criteria

1. **Given** der Solana Adapter (`src/domain/chain-adapters/solana/`) **When** `refreshValidators(knownAddresses)` aufgerufen wird **Then** werden Solana-Validator-Daten via RPC (`getVoteAccounts`) abgerufen **And** die Daten werden per Zod Schema validiert und auf `UnifiedValidator` gemappt

2. **Given** der Solana Adapter **When** `fetchStakingPositions(walletAddress)` aufgerufen wird **Then** werden Stake Accounts via `getProgramAccounts` + `getAccountInfo` mit manuellem Parsing abgerufen (NICHT `getStakeActivation` — deprecated und in Agave 2.0 entfernt!) **And** die Positionen werden normalisiert als `StakingPosition[]` zurückgegeben

3. **Given** der Solana Adapter **When** `fetchInflationRate()` aufgerufen wird **Then** wird die aktuelle Solana-Inflationsrate via `getInflationRate` RPC-Methode abgerufen und als `number` zurückgegeben

4. **Given** der QStash Cron-Job `/api/cron/fetch-validators` **When** der Cron feuert **Then** wird der SOL-Adapter parallel zum ETH-Adapter aufgerufen (QStash Fan-out, NFR21) **And** beide Adapter laufen unabhängig — ein SOL-Fehler blockiert nicht ETH (NFR28)

5. **Given** die Solana RPC API ist nicht erreichbar **When** ein Fehler auftritt **Then** wird Helius als Fallback-RPC verwendet (falls konfiguriert) **And** der Fehler wird geloggt ohne Wallet-Adressen

## Tasks / Subtasks

- [x] Task 1: Solana Adapter Zod Schemas erstellen (AC: #1, #2, #3)
  - [x] 1.1: `src/domain/chain-adapters/solana/sol-schemas.ts` — Zod Schemas für `getVoteAccounts` Response (VoteAccount-Struktur mit votePubkey, nodePubkey, activatedStake, commission, epochVoteAccount, lastVote, rootSlot, epochCredits)
  - [x] 1.2: Zod Schema für `getInflationRate` Response (total, validator, foundation, epoch)
  - [x] 1.3: Zod Schema für Stake-Account-Daten (discriminant, meta, delegation Felder)
  - [x] 1.4: Zod Schema für `getProgramAccounts` Response

- [x] Task 2: Solana Mapper implementieren (AC: #1, #2)
  - [x] 2.1: `src/domain/chain-adapters/solana/sol-mapper.ts` — `mapVoteAccountToValidator(voteAccount): UnifiedValidator` — Mapping von Solana VoteAccount auf UnifiedValidator
  - [x] 2.2: `mapStakeAccountToPosition(stakeAccount, walletAddress): StakingPosition` — Mapping von decodiertem Stake-Account auf StakingPosition
  - [x] 2.3: Stake-Account-Status-Determination: activationEpoch/deactivationEpoch → PositionStatus (active, activating, deactivating, inactive)
  - [x] 2.4: Commission Mapping: Solana commission (0-100 integer) → UnifiedValidator commission (0.0-1.0 float)
  - [x] 2.5: Delegated Amount: lamports (bigint) → SOL String (9 Dezimalstellen)

- [x] Task 3: Stake-Account Binary Parser (AC: #2)
  - [x] 3.1: `src/domain/chain-adapters/solana/stake-account-parser.ts` — Binary Parser für base64-kodierte Stake-Account-Daten
  - [x] 3.2: Discriminant parsing (4 bytes u32): 0=Uninitialized, 1=Initialized, 2=Delegated, 3=RewardsPool
  - [x] 3.3: Meta-Section parsing: rentExemptReserve (u64), authorized (staker + withdrawer, je 32 bytes), lockup (unixTimestamp u64, epoch u64, custodian 32 bytes)
  - [x] 3.4: Delegation parsing: voterPubkey (32 bytes), stake (u64 lamports), activationEpoch (u64), deactivationEpoch (u64)
  - [x] 3.5: Status-Ableitung aus activationEpoch/deactivationEpoch relativ zur aktuellen Epoch (via `getEpochInfo`)

- [x] Task 4: Solana Adapter implementieren (AC: #1, #2, #3, #5)
  - [x] 4.1: `src/domain/chain-adapters/solana/sol-adapter.ts` — `SolanaAdapter` Klasse implementiert `ChainAdapter` Interface
  - [x] 4.2: `refreshValidators(knownAddresses)` — `getVoteAccounts` RPC-Call, Filter auf bekannte Vote-Pubkeys, Mapping via sol-mapper
  - [x] 4.3: `fetchStakingPositions(walletAddress)` — `getProgramAccounts` mit Stake Program ID + memcmp-Filter auf Staker/Withdrawer Authority, base64-Decoding, Stake-Account-Parsing
  - [x] 4.4: `fetchInflationRate()` — `getInflationRate` RPC-Call, Rückgabe von `result.total`
  - [x] 4.5: RPC-Fallback-Logik: Primary RPC → Helius Fallback (falls `SOLANA_HELIUS_API_KEY` konfiguriert)
  - [x] 4.6: Timeout-Handling: 10s für alle RPC-Calls via `AbortSignal.timeout()`

- [x] Task 5: Environment Variables ergänzen (AC: #5)
  - [x] 5.1: `src/lib/env.ts` erweitern — `SOLANA_RPC_URL` (required), `SOLANA_HELIUS_API_KEY` (optional für Fallback)
  - [x] 5.2: `.env.example` aktualisieren mit Solana-spezifischen Variablen
  - [x] 5.3: Bestehende `NEXT_PUBLIC_SOLANA_RPC_URL` (client-seitig) bleibt unverändert — Adapter nutzt serverseitige `SOLANA_RPC_URL`

- [x] Task 6: Factory-Funktion erweitern (AC: #4)
  - [x] 6.1: `src/domain/chain-adapters/factory.ts` — `SolanaAdapter` Import hinzufügen, `Chain.SOLANA` Case implementieren statt `throw new Error`
  - [x] 6.2: `ChainAdapterConfig` erweitern falls nötig (z.B. fallbackUrl)

- [x] Task 7: Cron-Endpoint erweitern (AC: #4)
  - [x] 7.1: `src/app/api/cron/fetch-validators/route.ts` — Solana-Adapter parallel zum ETH-Adapter aufrufen
  - [x] 7.2: `Promise.allSettled()` für parallele Ausführung — SOL-Fehler blockiert nicht ETH (NFR28)
  - [x] 7.3: Response erweitern: `{ data: { chains: [{ chain, validatorsUpdated, metricsUpdated }] } }`

- [x] Task 8: Tests (AC: alle)
  - [x] 8.1: `src/domain/chain-adapters/solana/sol-mapper.test.ts` — Mapping-Tests (VoteAccount → UnifiedValidator, StakeAccount → StakingPosition)
  - [x] 8.2: `src/domain/chain-adapters/solana/stake-account-parser.test.ts` — Binary-Parsing-Tests (Discriminant, Meta, Delegation, Status-Ableitung)
  - [x] 8.3: `src/domain/chain-adapters/solana/sol-adapter.test.ts` — Adapter-Tests mit fetch-Mocking (refreshValidators, fetchStakingPositions, fetchInflationRate, Fallback-Logik)
  - [x] 8.4: `src/app/api/cron/fetch-validators/route.test.ts` — Erweiterte Tests für parallele Chain-Ausführung
  - [x] 8.5: Regressions-Check: Alle bestehenden Tests bestehen weiterhin (min. 193 Tests aus Story 2-1)

## Dev Notes

### Kritische Architektur-Entscheidungen

- **ADR-001 gilt auch für Solana:** `refreshValidators(knownAddresses)` statt Bulk-Fetch. Der Adapter refreshed NUR bereits bekannte Validators. Neue Validators werden über `fetchStakingPositions(walletAddress)` beim Wallet-Connect entdeckt. Bei leerer DB = kein Validator-Fetch.

- **Domain Layer bleibt framework-agnostisch:** Der `SolanaAdapter` nutzt `fetch()` für JSON-RPC-Calls — KEIN `@solana/web3.js` im Domain-Layer! Die `@solana/web3.js@1.98.4` Dependency existiert bereits im Projekt, wird aber NUR im Frontend (Wallet-Adapter, Provider) verwendet. Der Backend-Adapter nutzt direkten `fetch()` gegen den JSON-RPC-Endpoint. Das hält den Domain-Layer frei von schweren Dependencies.

- **Stake-Account Binary-Parsing MUSS manuell erfolgen:** `getStakeActivation` ist deprecated und in Agave 2.0 entfernt. Stake-Account-Daten kommen als base64-kodierter Buffer von `getAccountInfo`. Der Parser muss die 200-Byte-Struktur manuell dekodieren: 4 Bytes Discriminant + Meta (rentExemptReserve, authorized, lockup) + Stake (delegation mit voterPubkey, stake, activationEpoch, deactivationEpoch).

- **Adapter gibt Domain-Objekte zurück, Persistierung im Cron-Handler:** Identisch zu ETH-Adapter — `sol-adapter.ts` gibt `UnifiedValidator[]` / `StakingPosition[]` / `number` zurück. Der Cron-Handler persistiert via Repository-Funktionen.

- **SIMD-123 ist noch NICHT live (Feb 2026):** SIMD-123 (Block Revenue Commission) ist approved aber wartet auf Vote Account V4 und Agave 4.0/4.1. Aktuell gibt `getVoteAccounts` nur die Inflation-Commission zurück. Das `raw_data` JSONB-Feld im UnifiedValidator kann zukünftig die zweite Commission aufnehmen. Für jetzt: nur `commission` aus VoteAccount verwenden.

- **Solana Commission ist 0-100 (Ganzzahl), NICHT 0.0-1.0:** `getVoteAccounts` liefert `commission: 95` (= 95%). Der Mapper MUSS dies auf `0.95` (0.0-1.0 Float) konvertieren für das `UnifiedValidator` Model (Konsistenz mit ETH-Adapter).

- **`getProgramAccounts` mit memcmp-Filter für Wallet-Lookup:** Stake Accounts werden über den Stake Program (`Stake11111111111111111111111111111111111111`) gefiltert. `memcmp` Filter auf Offset 12 (authorized.staker) mit der Wallet-Adresse liefert alle delegierten Stake Accounts. Alternativ auch auf Offset 44 (authorized.withdrawer) filtern und deduplizieren.

- **Lamports zu SOL Konvertierung:** 1 SOL = 1.000.000.000 lamports (10^9). `delegatedAmount` wird als String gespeichert (wie bei ETH). Konvertierung: `(lamports / 1_000_000_000n).toString()` — aber mit Dezimalstellen-Bewahrung über Division-mit-Rest-Logik.

### Solana RPC API — Relevante Endpoints

| Daten | RPC-Methode | Parameter | Besonderheiten |
|---|---|---|---|
| Validator-Liste | `getVoteAccounts` | `{ commitment: "finalized" }` | Liefert `current` + `delinquent` Arrays |
| Stake Accounts für Wallet | `getProgramAccounts` | Stake Program ID + memcmp Filter | base64-kodierte Account-Daten, max 200 Bytes |
| Einzelne Account-Daten | `getAccountInfo` | `{ encoding: "base64" }` | Für detailliertes Stake-Account-Parsing |
| Inflationsrate | `getInflationRate` | keine | Liefert total, validator, foundation Anteile |
| Aktuelle Epoch | `getEpochInfo` | keine | Nötig für Status-Bestimmung (activating/deactivating) |

**JSON-RPC Request Format:**
```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "method": "getVoteAccounts",
  "params": [{ "commitment": "finalized" }]
}
```

### Stake-Account Binary Layout (200 Bytes)

```
Offset  Größe  Feld
------  -----  ----
0       4      discriminant (u32): 0=Uninitialized, 1=Initialized, 2=Delegated
4       8      meta.rentExemptReserve (u64, lamports)
12      32     meta.authorized.staker (Pubkey)     ← memcmp Filter-Offset für Wallet-Lookup
44      32     meta.authorized.withdrawer (Pubkey)  ← alternativer Filter-Offset
76      8      meta.lockup.unixTimestamp (u64)
84      8      meta.lockup.epoch (u64)
92      32     meta.lockup.custodian (Pubkey)
124     32     stake.delegation.voterPubkey (Pubkey)  ← Validator Vote Address
156     8      stake.delegation.stake (u64, lamports) ← Delegierter Betrag
164     8      stake.delegation.activationEpoch (u64)
172     8      stake.delegation.deactivationEpoch (u64)  ← MAX_U64 = nie deaktiviert
180     8      stake.delegation.unused (u64, legacy warmup_cooldown_rate)
188     8      stake.creditsObserved (u64)
196     4      padding
```

### Status-Ableitung aus Epoch-Daten

```typescript
// Vereinfachte Status-Logik (ohne Warmup/Cooldown-Rate-Berechnung)
function deriveStakeStatus(activationEpoch: bigint, deactivationEpoch: bigint, currentEpoch: bigint): PositionStatus {
  const MAX_U64 = 0xFFFFFFFFFFFFFFFFn;
  if (activationEpoch === deactivationEpoch) return "inactive"; // sofort deaktiviert
  if (currentEpoch < activationEpoch) return "activating";       // noch nicht aktiv
  if (deactivationEpoch !== MAX_U64 && currentEpoch >= deactivationEpoch) return "deactivating";
  return "active";
}
```

**Hinweis:** Für MVP ist diese vereinfachte Logik ausreichend. Die exakte Warmup/Cooldown-Rate-Berechnung (9% pro Epoch basierend auf StakeHistory Sysvar) ist für das Dashboard nicht relevant — der User sieht nur "aktiv", "wird aktiviert" oder "wird deaktiviert".

### Bestehende Infrastruktur (aus Story 2-1)

| Komponente | Datei | Status |
|---|---|---|
| ChainAdapter Interface | `src/domain/chain-adapters/types.ts` | Existiert — `refreshValidators(knownAddresses)`, `fetchStakingPositions()`, `fetchInflationRate()` |
| Factory | `src/domain/chain-adapters/factory.ts` | Existiert — `Chain.SOLANA` wirft aktuell `Error("not yet implemented")` |
| Unified Models | `src/domain/models/` | Existiert — `UnifiedValidator`, `StakingPosition`, `Chain`, `ChainMetrics` + Zod Schemas |
| DB Schema | `src/infrastructure/db/schema.ts` | Existiert — `validators`, `staking_positions`, `chain_metrics` akzeptieren `'solana'` |
| Repositories | `src/infrastructure/db/repositories/` | Existiert — `upsertValidators()`, `findValidatorsByChain()`, `upsertChainMetrics()` — chain-agnostisch, funktionieren für Solana |
| Cron Endpoint | `src/app/api/cron/fetch-validators/route.ts` | Existiert — aktuell nur ETH, MUSS um SOL erweitert werden |
| QStash Verify | `src/infrastructure/queue/qstash-verify.ts` | Existiert — wiederverwendbar |
| Env Validation | `src/lib/env.ts` | Existiert — `NEXT_PUBLIC_SOLANA_RPC_URL` (client) vorhanden, `SOLANA_RPC_URL` (server) MUSS hinzugefügt werden |
| Solana Provider | `src/providers/solana-provider.tsx` | Existiert — Frontend Wallet-Adapter (NICHT für Backend-Adapter relevant) |
| @solana/web3.js | package.json | v1.98.4 installiert — NUR für Frontend, NICHT im Domain-Layer verwenden |

### Learnings aus Story 2-1 (MUSS beachtet werden)

- **`--legacy-peer-deps` nötig:** Für ALLE npm installs wegen React 19 Peer-Dep-Konflikten
- **`serverEnv` statt `process.env`:** IMMER `serverEnv` aus `@/lib/env` verwenden
- **Biome 2.2.0:** Alle neuen Dateien müssen Biome-Check bestehen
- **Zod 4.3.6:** v4 Syntax verwenden (nicht v3!)
- **Named Exports:** Kein `default export` außer Next.js-Konventionen
- **Test-Pattern:** Dynamische Imports in `it()` Blöcken für Mock-Isolation
- **Drizzle `neon-http` Modus:** HTTP-basierter Treiber, Batch-Inserts via `.values([...]).onConflictDoUpdate(...)`
- **ON DELETE CASCADE auf user_id:** Bereits in Schema implementiert für DSGVO
- **193 bestehende Tests** müssen weiterhin bestehen
- **Domain-Layer importiert NUR aus domain/models/:** KEIN React, KEIN Next.js, KEIN infra
- **API Response Format:** `{ data, meta? }` (Success), `{ error, code, details? }` (Error)
- **Logging:** `[sol-adapter] message` Format, KEINE Wallet-Adressen in Logs
- **Known Issue FIXME [M1]:** `linked_wallets` nutzt `'sol'`, `validators`/`staking_positions`/`chain_metrics` nutzen `'solana'` — NICHT in dieser Story fixen, nur beachten

### RPC Rate Limits & Best Practices

- **Solana Public RPC (`api.mainnet-beta.solana.com`):** 100 req/10s pro IP, 40 req/10s pro Methode — NICHT für Production
- **Helius (empfohlen):** Free: 1M Credits/Monat, 10 RPS. Developer ($49): 10M Credits, 50 RPS
- **Immer `commitment: "finalized"` verwenden** — konsistent und cacheable
- **Timeouts:** 10s für alle RPC-Calls via `AbortSignal.timeout(10_000)`
- **`getVoteAccounts` ist ein schwerer Call:** Liefert ALLE aktiven Validators (~1800+). Für `refreshValidators()` den Response filtern auf bekannte Vote-Pubkeys
- **`getProgramAccounts` kann langsam sein:** memcmp-Filter reduzieren die Datenmenge erheblich

### Abgrenzung: Was diese Story NICHT macht

- **Kein Redis-Cache:** Upstash Redis Cache-Layer kommt in Story 2-4
- **Kein Dashboard-UI:** Dashboard-UI kommt in Story 2-5
- **Kein Cosmos-Adapter:** Cosmos kommt in Story 2-3
- **Kein Rate-Limiting auf Public API:** Kommt in Story 2-4
- **Keine Freshness-Anzeige:** Freshness-Indicators kommen in Story 2-6
- **Keine SIMD-123 Block-Revenue-Commission:** Noch nicht live, raw_data Feld steht bereit
- **Kein @solana/kit Migration:** Projekt nutzt @solana/web3.js@1.98.4 im Frontend — Backend-Adapter nutzt direkten fetch()
- **Kein Helius Paid Plan Setup:** API-Key-Beschaffung ist Prerequisite
- **Keine Warmup/Cooldown-Rate-Berechnung:** Vereinfachte Status-Logik reicht für MVP

### Project Structure Notes

- Neue Dateien in `src/domain/chain-adapters/solana/` (sol-adapter.ts, sol-mapper.ts, sol-schemas.ts, stake-account-parser.ts)
- Tests co-located neben Source-Dateien
- Factory-Funktion wird erweitert (nicht neu erstellt)
- Cron-Endpoint wird erweitert (nicht neu erstellt)
- Env-Validation wird erweitert (nicht neu erstellt)
- KEINE neuen DB-Tabellen oder Migrationen nötig — bestehende Tabellen akzeptieren `'solana'`

### References

- [Source: _bmad-output/planning-artifacts/epics.md#Story 2.2 — Solana Chain Adapter]
- [Source: _bmad-output/planning-artifacts/epics.md#Epic 2 — FR8, FR37, FR40, FR41]
- [Source: _bmad-output/planning-artifacts/architecture.md#Chain Adapter Architecture (Aggregator Pattern)]
- [Source: _bmad-output/planning-artifacts/architecture.md#ADR-001 — User-Driven Validator Discovery]
- [Source: _bmad-output/planning-artifacts/architecture.md#Data Architecture — Drizzle ORM + NeonDB]
- [Source: _bmad-output/planning-artifacts/architecture.md#Implementation Patterns & Consistency Rules]
- [Source: _bmad-output/planning-artifacts/architecture.md#External Integration Points — Solana RPC]
- [Source: _bmad-output/project-context.md#Chain Adapter Architecture (Aggregator Pattern)]
- [Source: _bmad-output/project-context.md#Solana-spezifische Warnung — getStakeActivation deprecated]
- [Source: _bmad-output/implementation-artifacts/2-1-chain-adapter-infrastructure-ethereum-adapter.md — Learnings, Patterns, ADR-001]
- [Source: Web Research — Solana RPC Docs: getVoteAccounts, getInflationRate, getProgramAccounts]
- [Source: Web Research — Anza solana-rpc-client-extensions: Stake Account Parsing Reference]
- [Source: Web Research — SIMD-123 Status: Approved, nicht live, wartet auf Vote Account V4]
- [Source: Web Research — @solana/web3.js v1.x maintenance, @solana/kit als Nachfolger]
- [Source: Web Research — Helius RPC: Free 1M Credits, Endpoint mainnet.helius-rpc.com]
- [Source: Web Research — Solana Public RPC Rate Limits: 100 req/10s]

## Dev Agent Record

### Agent Model Used

Claude Opus 4.6

### Debug Log References

- Biome non-null assertion Warnings: Refactored `readU32LE` to use `DataView` statt direkter Buffer-Indizierung, `ALPHABET` zu `charAt()`, Test-Assertions zu `NonNullable<>` cast
- u64 Overflow in Test: `5_000_000_000` > `2^32`, erforderte korrekte lo/hi Split (lo=705032704, hi=1)
- env.test.ts: `SOLANA_RPC_URL` als required env var musste in bestehende Test-Fixtures eingefügt werden

### Completion Notes List

- Solana Chain Adapter komplett implementiert mit direktem `fetch()` gegen JSON-RPC (kein `@solana/web3.js` im Domain-Layer)
- Stake-Account Binary Parser dekodiert 200-Byte base64-Buffer manuell (kein `getStakeActivation` — deprecated in Agave 2.0)
- Status-Ableitung (active/activating/deactivating/inactive) via activationEpoch/deactivationEpoch vs. currentEpoch
- Commission Mapping: Solana 0-100 Integer → 0.0-1.0 Float (Konsistenz mit ETH-Adapter)
- Lamports → SOL Konvertierung mit korrekter Dezimalstellen-Bewahrung
- RPC-Fallback: Primary → Helius (wenn `SOLANA_HELIUS_API_KEY` konfiguriert)
- Cron-Endpoint: ETH + SOL parallel via `Promise.allSettled()` — ein Fehler blockiert nicht den anderen (NFR28)
- Response-Format erweitert: `{ data: { chains: [...] } }` mit 207 Multi-Status bei partiellen Fehlern
- Factory erweitert: `Chain.SOLANA` → `SolanaAdapter` mit neuem `fallbackUrl` Config-Feld
- 240 Tests bestehen (193 bestehende + 47 neue), Biome-Check bestanden

### File List

**Neue Dateien:**
- `src/domain/chain-adapters/solana/sol-schemas.ts`
- `src/domain/chain-adapters/solana/sol-mapper.ts`
- `src/domain/chain-adapters/solana/sol-mapper.test.ts`
- `src/domain/chain-adapters/solana/stake-account-parser.ts`
- `src/domain/chain-adapters/solana/stake-account-parser.test.ts`
- `src/domain/chain-adapters/solana/sol-adapter.ts`
- `src/domain/chain-adapters/solana/sol-adapter.test.ts`

**Modifizierte Dateien:**
- `src/domain/chain-adapters/factory.ts` — SolanaAdapter Import + Chain.SOLANA Case + fallbackUrl Config
- `src/lib/env.ts` — SOLANA_RPC_URL (optional) + SOLANA_HELIUS_API_KEY (optional)
- `src/lib/env.test.ts` — Angepasst an optionale SOLANA_RPC_URL
- `src/app/api/cron/fetch-validators/route.ts` — Refactored: Factory-Pattern, dynamische Chain-Configs, parallele Ausführung via Promise.allSettled
- `src/app/api/cron/fetch-validators/route.test.ts` — Refactored: Factory-Mock statt direkte Adapter-Mocks
- `.env.example` — Solana server-side Env-Vars
- `_bmad-output/implementation-artifacts/sprint-status.yaml` — Status: in-progress → review

## Change Log

- 2026-02-22: Story 2.2 implementiert — Solana Chain Adapter mit Binary Stake-Account-Parser, RPC-Fallback, paralleler Cron-Ausführung (8 Tasks, 45 neue Tests)
- 2026-02-22: Code Review (Adversarial) — 7 Issues gefixed: [H1] SOLANA_RPC_URL optional statt required, [H2] JSON-RPC Error-Response-Validierung, [H3] Cron-Handler auf Factory-Pattern refactored, [M1] fetchStakingPositions 3 RPC-Calls parallel, [M2] Withdrawer-Only Test hinzugefügt, [M3] Story File List korrigiert, [M4] Helius-URL in Chain-Config zentralisiert. 240 Tests bestehen.
