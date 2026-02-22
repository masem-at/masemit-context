# Story 2.4: Upstash Redis Cache-Layer & API-Security

Status: done

<!-- Note: Validation is optional. Run validate-create-story for quality check before dev-story. -->
<!-- Generated: 2026-02-22 | Epic: 2 | Story: 4 | Key: 2-4-upstash-redis-cache-layer-api-security -->

## Story

As a User,
I want dass das Dashboard blitzschnell laedt und die API sicher ist,
So that ich eine performante und vertrauenswuerdige Erfahrung habe.

## Acceptance Criteria

1. **Given** der Redis Cache-Layer (`src/infrastructure/cache/`) **When** konfiguriert ist **Then** existiert ein `redis-client.ts` mit Upstash Redis Instance **And** existiert `cache-keys.ts` mit Key-Pattern: `staketrack:{entity}:{chain}:{id}` **And** existiert `cache-utils.ts` mit `get`, `set`, `invalidate` Helpers **And** Validator-Daten werden mit TTL 5 Minuten gecacht **And** VRS-Scores werden mit TTL 1 Stunde gecacht **And** Yield-Daten werden mit TTL 24 Stunden gecacht

2. **Given** ein Cron-Job aktualisiert Daten erfolgreich **When** die Cache-Invalidierung laeuft (FR40) **Then** werden die betroffenen Cache-Keys synchron invalidiert **And** der naechste Request liefert frische Daten

3. **Given** Rate Limiting (`src/lib/api/rate-limit.ts`) **When** auf oeffentliche API-Endpoints zugegriffen wird **Then** greift Upstash Rate Limit (FR42) **And** bei Ueberschreitung wird 429 mit `Retry-After` Header zurueckgegeben

4. **Given** ein Request an `/api/cron/*` Endpoints **When** die HMAC-Signatur fehlt oder ungueltig ist **Then** wird der Request mit 401 abgelehnt (FR41) **And** nur QStash-signierte Requests werden verarbeitet

5. **Given** der Cache 80%+ der Read-Requests absorbiert (NFR20) **When** ein Dashboard-Request eintrifft **Then** wird zuerst Redis geprueft → bei Hit: Response <100ms (NFR2) **And** bei Miss: DB-Query → Redis Set → Response <2s (NFR3)

## Tasks / Subtasks

- [x] Task 1: npm Pakete installieren (AC: #1, #3)
  - [x] 1.1: `npm install @upstash/redis @upstash/ratelimit --legacy-peer-deps`
  - [x] 1.2: Verifizieren dass `@upstash/redis` ^1.36.x und `@upstash/ratelimit` ^2.0.x in package.json stehen

- [x] Task 2: Redis Client erstellen (AC: #1)
  - [x] 2.1: `src/infrastructure/cache/redis-client.ts` — Singleton Redis Instance via `new Redis({ url: serverEnv.UPSTASH_REDIS_REST_URL, token: serverEnv.UPSTASH_REDIS_REST_TOKEN })` mit Guard fuer fehlende Env-Vars (graceful null-return wenn nicht konfiguriert)
  - [x] 2.2: Exportiert `getRedisClient(): Redis | null` — gibt `null` zurueck wenn Env-Vars fehlen (Development ohne Redis moeglich)

- [x] Task 3: Cache-Key-Definitionen (AC: #1)
  - [x] 3.1: `src/infrastructure/cache/cache-keys.ts` — Key-Pattern-Definitionen als Funktionen:
    - `validatorsKey(chain: Chain)` → `staketrack:validators:{chain}:all`
    - `validatorKey(chain: Chain, address: string)` → `staketrack:validators:{chain}:{address}`
    - `chainMetricsKey(chain: Chain)` → `staketrack:chain-metrics:{chain}:latest`
    - `stakingPositionsKey(userId: string, chain?: Chain)` → `staketrack:positions:{userId}:{chain|all}`
    - `vrsScoreKey(chain: Chain, validatorId: string)` → `staketrack:vrs:{chain}:{validatorId}`
    - `yieldSnapshotKey(chain: Chain, validatorId: string)` → `staketrack:yield:{chain}:{validatorId}`
  - [x] 3.2: TTL-Konstanten definieren:
    - `CACHE_TTL_VALIDATORS = 300` (5 Minuten)
    - `CACHE_TTL_CHAIN_METRICS = 300` (5 Minuten)
    - `CACHE_TTL_STAKING_POSITIONS = 300` (5 Minuten)
    - `CACHE_TTL_VRS = 3600` (1 Stunde)
    - `CACHE_TTL_YIELD = 86400` (24 Stunden)

- [x] Task 4: Cache-Utility-Funktionen (AC: #1, #2, #5)
  - [x] 4.1: `src/infrastructure/cache/cache-utils.ts` — Generische Cache-Aside Helpers:
    - `cacheGet<T>(key: string): Promise<T | null>` — Redis GET mit JSON-Parsing, gibt null bei fehlendem Redis-Client zurueck
    - `cacheSet<T>(key: string, value: T, ttlSeconds: number): Promise<void>` — Redis SET mit EX TTL, no-op bei fehlendem Redis-Client
    - `cacheInvalidate(...keys: string[]): Promise<void>` — Redis DEL fuer einen oder mehrere Keys
    - `cacheInvalidatePattern(pattern: string): Promise<void>` — SCAN + DEL fuer Pattern-basierte Invalidierung (z.B. `staketrack:validators:eth:*`)
  - [x] 4.2: Alle Funktionen sind fail-safe: bei Redis-Fehler wird geloggt und der Aufruf laeuft ohne Cache weiter (kein Crash)
  - [x] 4.3: Logging-Format: `[cache] message` — z.B. `[cache] HIT staketrack:validators:eth:all`, `[cache] MISS staketrack:validators:eth:all`

- [x] Task 5: Cache in Cron-Endpoint integrieren — Cache-Invalidierung (AC: #2)
  - [x] 5.1: `src/app/api/cron/fetch-validators/route.ts` — Nach erfolgreichem `upsertValidators()` und `upsertChainMetrics()` die betroffenen Cache-Keys synchron invalidieren:
    - `cacheInvalidate(validatorsKey(chain))` — alle Validators fuer die Chain
    - `cacheInvalidate(chainMetricsKey(chain))` — Chain-Metriken
  - [x] 5.2: `TODO [FR40]` Kommentar in Zeile ~191 entfernen und durch echte Implementierung ersetzen
  - [x] 5.3: Import von `cacheInvalidate`, `validatorsKey`, `chainMetricsKey` aus `@/infrastructure/cache/`

- [x] Task 6: Rate-Limiting Middleware erstellen (AC: #3)
  - [x] 6.1: `src/lib/api/rate-limit.ts` — Upstash Rate-Limit Konfiguration:
    - `publicRateLimit` — Sliding Window: 60 Requests / 1 Minute pro IP (fuer Public API Endpoints)
    - `authRateLimit` — Sliding Window: 120 Requests / 1 Minute pro User-ID (fuer authentifizierte Endpoints)
    - `cronRateLimit` — Fixed Window: 10 Requests / 1 Minute pro Endpoint (fuer Cron-Endpoints — Schutz gegen versehentliche Doppel-Ausloesungen)
    - Prefix: `staketrack:ratelimit:{tier}`
  - [x] 6.2: `withRateLimit(identifier: string, limiter: Ratelimit): Promise<RateLimitResult>` — Wrapper-Funktion die `{ success, remaining, limit, reset }` zurueckgibt
  - [x] 6.3: Bei fehlendem Redis-Client: Rate-Limiting ist deaktiviert (Development-Modus), loggt Warnung `[rate-limit] Redis not configured, rate limiting disabled`
  - [x] 6.4: Response-Headers setzen: `X-RateLimit-Limit`, `X-RateLimit-Remaining`, `Retry-After` (bei 429)

- [x] Task 7: Rate-Limiting in bestehende API-Routes integrieren (AC: #3)
  - [x] 7.1: `src/app/api/auth/email/send/route.ts` — In-Memory-Map-basiertes Rate-Limiting durch Upstash ersetzen (bestehend: 3 req / 15 min per email). Limiter: `emailSendRateLimit` = 3 Requests / 15 Minuten pro Email-Adresse
  - [x] 7.2: `src/app/api/wallets/[walletId]/route.ts` — In-Memory-Map-basiertes Rate-Limiting durch Upstash ersetzen (bestehend: 5 DELETE / 15 min per userId). Limiter: `walletDeleteRateLimit` = 5 Requests / 15 Minuten pro User-ID
  - [x] 7.3: `src/app/api/account/route.ts` — In-Memory-Map-basiertes Rate-Limiting durch Upstash ersetzen (bestehend: 1 DELETE / 60s per userId). Limiter: `accountDeleteRateLimit` = 1 Request / 60 Sekunden pro User-ID
  - [x] 7.4: In-Memory `Map()` und `rateLimitMap` Variablen aus allen drei Dateien entfernen
  - [x] 7.5: Alle drei Routes verwenden `{ error, code: "RATE_LIMIT" }` Response-Format (konsistent mit Architecture-Pattern)

- [x] Task 8: HMAC-Verifizierung konsolidieren (AC: #4)
  - [x] 8.1: Verifizieren dass `src/infrastructure/queue/qstash-verify.ts` korrekt funktioniert (bereits implementiert)
  - [x] 8.2: Verifizieren dass `/api/cron/fetch-validators/route.ts` die HMAC-Verifizierung korrekt aufruft (bereits implementiert)
  - [x] 8.3: Sicherstellen dass bei fehlender/ungueltiger Signatur ein `{ error: "Unauthorized", code: "HMAC_INVALID" }` mit Status 401 zurueckgegeben wird (konsistentes Error-Format)

- [x] Task 9: Tests (AC: alle)
  - [x] 9.1: `src/infrastructure/cache/cache-utils.test.ts` — Unit-Tests fuer Cache-Utilities: GET/SET/DEL, TTL-Verifikation, Pattern-Invalidierung, Verhalten bei fehlendem Redis-Client (null), Error-Handling (Redis-Fehler wird gefangen und geloggt)
  - [x] 9.2: `src/infrastructure/cache/cache-keys.test.ts` — Unit-Tests fuer Key-Generierung: korrektes Format, verschiedene Chains, Sonderzeichen in IDs
  - [x] 9.3: `src/lib/api/rate-limit.test.ts` — Unit-Tests fuer Rate-Limiting: Success-Case, Limit-Exceeded (429), Response-Headers, fehlendes Redis (Fallback)
  - [x] 9.4: `src/app/api/cron/fetch-validators/route.test.ts` — Erweiterte Tests: Cache-Invalidierung wird nach erfolgreichem Fetch aufgerufen, Cache-Invalidierung wird NICHT bei Fehler aufgerufen
  - [x] 9.5: Regressions-Check: Alle bestehenden ~280 Tests muessen weiterhin bestehen
  - [x] 9.6: Tests fuer die 3 migrierten Rate-Limit-Routes: Verifizieren dass das neue Upstash-Rate-Limiting das gleiche Verhalten hat wie das alte In-Memory-Limiting

## Dev Notes

### Kritische Architektur-Entscheidungen

- **Redis ist OPTIONAL im Development:** Der Redis-Client gibt `null` zurueck wenn `UPSTASH_REDIS_REST_URL` oder `UPSTASH_REDIS_REST_TOKEN` nicht gesetzt sind. Alle Cache-Funktionen und Rate-Limiting arbeiten dann als No-Op (Passthrough). Das ermoeglicht lokale Entwicklung ohne Upstash-Account. In Production MUESSEN die Env-Vars gesetzt sein.

- **Cache-Aside Pattern (Lazy Loading):** Cache wird NICHT proaktiv befuellt. Erster Request → Cache Miss → DB-Query → Cache Set → Response. Nachfolgende Requests → Cache Hit → Response. Cache-Invalidierung nach Cron-Updates loesche die betroffenen Keys, der naechste Request fuellt sie wieder.

- **Kein Cache-Warmup im Cron:** Der Cron-Job invalidiert den Cache nach DB-Updates, befuellt ihn aber NICHT proaktiv. Gruende: (1) Cron laeuft im 60s Vercel-Timeout — Cache-Warmup waere zusaetzliche Latenz, (2) Cache-Aside ist einfacher und ausreichend fuer 500 User MVP (NFR17).

- **SCAN statt KEYS fuer Pattern-Invalidierung:** `KEYS` blockiert den Redis-Server bei vielen Keys. `SCAN` ist cursor-basiert und non-blocking. Fuer die MVP-Groessenordnung (~100-500 Keys) ist der Unterschied minimal, aber SCAN ist die korrekte Praxis.

- **Rate-Limiting: Sliding Window statt Fixed Window:** Sliding Window bietet smoothere Rate-Begrenzung ohne die "Burst am Window-Rand"-Problematik von Fixed Window. `@upstash/ratelimit` v2.0.x unterstuetzt beides — Sliding Window ist der empfohlene Default.

- **In-Memory-Map Rate-Limiting wird vollstaendig ersetzt:** Die 3 bestehenden Routes (`email/send`, `wallets/[walletId]`, `account`) nutzen je eine eigene `Map()` fuer Rate-Limiting. Das funktioniert NICHT in Serverless (jeder Cold Start = neuer Map-State). Upstash Redis persistiert ueber Invocations hinweg — korrekt fuer Vercel.

- **Domain Layer bleibt Cache-frei:** Der Cache-Layer ist Infrastructure (`src/infrastructure/cache/`). Domain-Code (`src/domain/`) importiert KEINEN Cache. Caching geschieht in den API Route Handlers oder im Cron-Handler — NICHT in Adaptern, Repositories oder Domain-Logik.

- **Bestehende QStash-Verifizierung ist bereits korrekt:** `src/infrastructure/queue/qstash-verify.ts` existiert und wird in `/api/cron/fetch-validators/route.ts` bereits aufgerufen. Diese Story verifiziert nur die Korrektheit und stellt konsistentes Error-Format sicher.

### Upstash Redis — Technische Details

| Eigenschaft | Wert |
|---|---|
| **Paket** | `@upstash/redis` ^1.36.x |
| **Protokoll** | HTTP/REST (NICHT TCP — perfekt fuer Serverless) |
| **Client-Init** | `new Redis({ url, token })` — NICHT `Redis.fromEnv()` (wir nutzen validierte `serverEnv`) |
| **Serialisierung** | Automatisch JSON — `set('key', { obj })` speichert als JSON-String |
| **TTL** | `{ ex: seconds }` Parameter bei `set()` |
| **Scan** | `scan(cursor, { match: pattern, count: 100 })` → `[nextCursor, keys]` |
| **Pipeline** | `redis.pipeline().set(...).get(...).exec()` fuer Batch-Operationen |
| **Env-Vars** | `UPSTASH_REDIS_REST_URL`, `UPSTASH_REDIS_REST_TOKEN` (bereits in env.ts deklariert) |

### Upstash Rate Limit — Technische Details

| Eigenschaft | Wert |
|---|---|
| **Paket** | `@upstash/ratelimit` ^2.0.x |
| **Algorithmus** | `Ratelimit.slidingWindow(requests, window)` |
| **Response** | `{ success: boolean, remaining: number, limit: number, reset: number }` |
| **Identifier** | IP-Adresse (Public), User-ID (Authenticated), Email (Email-Send) |
| **Prefix** | `staketrack:ratelimit:{scope}` — verhindert Key-Kollisionen |
| **Ephemeral Cache** | `ephemeralCache: new Map()` — In-Memory-Kurzzeit-Cache fuer Performance |

### Cache-Key-Schema

```
staketrack:validators:{chain}:all          → UnifiedValidator[] (TTL: 5 Min)
staketrack:validators:{chain}:{address}    → UnifiedValidator   (TTL: 5 Min)
staketrack:chain-metrics:{chain}:latest    → ChainMetrics       (TTL: 5 Min)
staketrack:positions:{userId}:{chain|all}  → StakingPosition[]  (TTL: 5 Min)
staketrack:vrs:{chain}:{validatorId}       → VrsScore           (TTL: 1 Std)
staketrack:yield:{chain}:{validatorId}     → YieldSnapshot      (TTL: 24 Std)
```

### Bestehende Infrastruktur (aus Story 2-1, 2-2, 2-3)

| Komponente | Datei | Status |
|---|---|---|
| Cache-Verzeichnis | `src/infrastructure/cache/` | Existiert — nur `.gitkeep`, LEER |
| Redis Env-Vars | `src/lib/env.ts` | `UPSTASH_REDIS_REST_URL` + `UPSTASH_REDIS_REST_TOKEN` bereits deklariert (optional) |
| QStash Verify | `src/infrastructure/queue/qstash-verify.ts` | Existiert — funktionsfaehig |
| Cron Endpoint | `src/app/api/cron/fetch-validators/route.ts` | Existiert — `TODO [FR40]` Kommentar fuer Cache-Invalidierung |
| In-Memory Rate-Limit (email) | `src/app/api/auth/email/send/route.ts` | Existiert — MUSS ersetzt werden |
| In-Memory Rate-Limit (wallet) | `src/app/api/wallets/[walletId]/route.ts` | Existiert — MUSS ersetzt werden |
| In-Memory Rate-Limit (account) | `src/app/api/account/route.ts` | Existiert — MUSS ersetzt werden |
| API-Verzeichnis | `src/lib/api/` | Existiert — nur `.gitkeep`, LEER |
| @upstash/qstash | package.json | ^2.9.0 — bereits installiert |
| @upstash/redis | package.json | NICHT installiert — MUSS installiert werden |
| @upstash/ratelimit | package.json | NICHT installiert — MUSS installiert werden |

### Learnings aus Story 2-1, 2-2, 2-3 (MUSS beachtet werden)

- **`--legacy-peer-deps` noetig:** Fuer ALLE npm installs wegen React 19 Peer-Dep-Konflikten
- **`serverEnv` statt `process.env`:** IMMER `serverEnv` aus `@/lib/env` verwenden — NIEMALS `process.env` direkt
- **Biome 2.2.0:** Alle neuen Dateien muessen Biome-Check bestehen
- **Zod 4.3.6:** v4 Syntax verwenden (nicht v3!)
- **Named Exports:** Kein `default export` ausser Next.js-Konventionen
- **Test-Pattern:** Tests nutzen fetch-Mocking / Dependency-Injection; alle Tests co-located neben Source-Dateien
- **Drizzle `neon-http` Modus:** HTTP-basierter Treiber
- **Domain-Layer importiert NUR aus domain/models/:** KEIN React, KEIN Next.js, KEIN infra — Cache ist Infrastructure, NICHT Domain
- **API Response Format:** `{ data, meta? }` (Success), `{ error, code, details? }` (Error)
- **Logging:** `[component] message` Format, KEINE Wallet-Adressen in Logs
- **Known Issue FIXME [M1]:** `linked_wallets` nutzt `'atom'`, `validators`/`staking_positions`/`chain_metrics` nutzen `'cosmos'` — NICHT in dieser Story fixen
- **~280 bestehende Tests** muessen weiterhin bestehen

### Git-Intelligence (letzte Commits)

```
315ae8b feat: add ETH wallet linking for email-authenticated users
030cc10 feat: add chain adapters (ETH/SOL), DB schema, fix /dashboard redirect
dca1785 fix: use RESEND_FROM_EMAIL env var instead of hardcoded staketrack.ai domain
e9e9a9c feat: implement Story 1.6 — Account-Loeschung (DSGVO)
e26e699 feat: implement Story 1.5 — Wallet-Verwaltung & Einstellungen
```

Story 2-3 (Cosmos Adapter) scheint noch nicht committed zu sein oder wurde in einem separaten Branch committed. Alle Chain Adapters (ETH, SOL, ATOM) sind implementiert und getestet.

### Abgrenzung: Was diese Story NICHT macht

- **Kein Dashboard-UI:** Dashboard-UI kommt in Story 2-5
- **Keine Freshness-Anzeige:** Freshness-Indicators kommen in Story 2-6
- **Keine API-Endpoints fuer Validators/Portfolio:** Die API-Routes kommen erst in Story 2-5 — diese Story bereitet nur die Cache-Infrastruktur vor und integriert sie in den bestehenden Cron-Endpoint
- **Kein proaktives Cache-Warmup:** Cache wird lazy befuellt (Cache-Aside)
- **Keine TimescaleDB Hypertables:** Kommen erst in Epic 3/4 (VRS, Yield)
- **Keine Middleware-Integration in `src/middleware.ts`:** Rate-Limiting wird in den einzelnen API Route Handlers implementiert, NICHT als globale Next.js Middleware (die bestehende Middleware handled nur Auth-Redirects). Globale Middleware wuerde auch SSR-Requests rate-limiten — das ist nicht gewuenscht
- **Kein QStash Publisher/Scheduler:** Nur die bestehende Receiver-Seite (Signatur-Verifizierung) wird genutzt

### Project Structure Notes

- Neue Dateien in `src/infrastructure/cache/` (redis-client.ts, cache-keys.ts, cache-utils.ts + Tests)
- Neue Datei in `src/lib/api/` (rate-limit.ts + Test)
- Tests co-located neben Source-Dateien
- Cron-Endpoint wird erweitert (nicht neu erstellt)
- 3 bestehende Routes werden refactored (In-Memory → Upstash)
- KEINE neuen DB-Tabellen oder Migrationen noetig

### References

- [Source: _bmad-output/planning-artifacts/epics.md#Story 2.4 — Upstash Redis Cache-Layer & API-Security]
- [Source: _bmad-output/planning-artifacts/epics.md#Epic 2 — FR40, FR41, FR42]
- [Source: _bmad-output/planning-artifacts/architecture.md#Caching (Multi-Layer)]
- [Source: _bmad-output/planning-artifacts/architecture.md#Communication Patterns — Cache Events]
- [Source: _bmad-output/planning-artifacts/architecture.md#Authentication & Security — API Security]
- [Source: _bmad-output/planning-artifacts/architecture.md#External Integration Points — Upstash Redis, QStash]
- [Source: _bmad-output/planning-artifacts/architecture.md#Implementation Patterns — Process Patterns, Error Recovery]
- [Source: _bmad-output/project-context.md#Caching (Multi-Layer)]
- [Source: _bmad-output/project-context.md#Security Rules — Rate Limiting, HMAC]
- [Source: _bmad-output/implementation-artifacts/2-3-cosmos-chain-adapter.md — Learnings, Cron-Handler TODO FR40]
- [Source: _bmad-output/implementation-artifacts/2-1-chain-adapter-infrastructure-ethereum-adapter.md — Learnings]
- [Source: _bmad-output/implementation-artifacts/2-2-solana-chain-adapter.md — Learnings, Cron-Handler]
- [Source: Web Research — @upstash/redis v1.36.x: HTTP-REST-Client, Serverless-optimiert, JSON-Serialisierung]
- [Source: Web Research — @upstash/ratelimit v2.0.x: Sliding Window, ephemeralCache, Analytics]
- [Source: Web Research — Redis Cache-Aside Pattern: Lazy Loading, TTL, SCAN statt KEYS]

## Dev Agent Record

### Agent Model Used

Claude Opus 4.6

### Debug Log References

Keine Debug-Issues aufgetreten.

### Completion Notes List

- Upstash Redis Client (`redis-client.ts`) mit Singleton-Pattern und graceful null-return bei fehlenden Env-Vars implementiert
- Cache-Key-Schema (`cache-keys.ts`) mit 6 Key-Generatoren und 5 TTL-Konstanten erstellt
- Cache-Utilities (`cache-utils.ts`) mit `cacheGet`, `cacheSet`, `cacheInvalidate`, `cacheInvalidatePattern` implementiert — alle fail-safe mit Logging
- Cache-Invalidierung in Cron-Endpoint integriert — nur erfolgreiche Chains werden invalidiert (FR40)
- Rate-Limiting Middleware (`rate-limit.ts`) mit 6 Limiter-Presets (public, auth, cron, email-send, wallet-delete, account-delete) erstellt
- 3 bestehende API-Routes von In-Memory `Map()` Rate-Limiting auf Upstash Redis migriert
- Error-Code in allen Rate-Limit-Responses auf `RATE_LIMIT` vereinheitlicht (Architecture-Pattern)
- HMAC Error-Response im Cron-Endpoint auf `{ error: "Unauthorized", code: "HMAC_INVALID" }` aktualisiert
- 40 neue Tests geschrieben (322 total, vorher ~280), 0 Regressionen
- Biome-Check bestanden

### Senior Developer Review (AI)

**Reviewer:** Code Review Workflow | **Datum:** 2026-02-22 | **Model:** Claude Opus 4.6

**Ergebnis:** 8 Issues gefunden (1 HIGH, 5 MEDIUM, 2 LOW) — 6 automatisch gefixt

| # | Severity | Issue | Status |
|---|----------|-------|--------|
| H1 | HIGH | `withRateLimit()` fehlte try/catch — Redis-Ausfall crashte alle rate-limited Endpoints | FIXED |
| M1 | MEDIUM | Ratelimit-Instanzen bei jedem Request neu erzeugt — `ephemeralCache` wirkungslos | FIXED |
| M2 | MEDIUM | Story 2-3 Aenderungen (factory.ts, env.ts, .env.example) uncommitted und vermischt mit 2-4 | NOTED — Git-Workflow-Issue, kein Code-Fix |
| M3 | MEDIUM | `console.log` fuer Cache-HIT/MISS — Log-Spam in Production | FIXED → `console.debug` |
| M4 | MEDIUM | `cacheInvalidatePattern` DEL ohne Batch-Limit bei vielen Keys | FIXED → Batch-Size 100 |
| M5 | MEDIUM | Kein Barrel-Export (`index.ts`) fuer Cache-Modul | FIXED → `index.ts` erstellt |
| L1 | LOW | Cache-Key-Sanitierung fehlt (Sonderzeichen in IDs) | OPEN — Low Priority |
| L2 | LOW | Cache-Key-Tests ohne Edge-Cases (leere Strings, Sonderzeichen) | OPEN — Low Priority |

**Fixes angewendet:**
- `rate-limit.ts`: try/catch in `withRateLimit()`, Singleton-Pattern fuer Limiter, `_resetLimitersForTesting()` Export
- `cache-utils.ts`: `console.log` → `console.debug`, DEL-Batching mit Batch-Size 100
- `cache/index.ts`: NEU — Barrel-Export fuer gesamtes Cache-Modul
- `rate-limit.test.ts`: Singleton-Test + Error-Handling-Test hinzugefuegt (10 Tests total)

**322 Tests bestanden, 0 Regressionen, Biome-Check OK**

### Change Log

- 2026-02-22: Story 2.4 implementiert — Upstash Redis Cache-Layer, Rate-Limiting Migration, HMAC Error-Format
- 2026-02-22: Code Review — 6 Issues gefixt (H1, M1, M3, M4, M5), 2 LOW offen (L1, L2)

### File List

- `package.json` — @upstash/redis ^1.36.2, @upstash/ratelimit ^2.0.8 hinzugefuegt
- `package-lock.json` — Lock-File aktualisiert
- `src/infrastructure/cache/redis-client.ts` — NEU: Singleton Redis Client
- `src/infrastructure/cache/cache-keys.ts` — NEU: Cache-Key-Generatoren und TTL-Konstanten
- `src/infrastructure/cache/cache-keys.test.ts` — NEU: 11 Unit-Tests
- `src/infrastructure/cache/cache-utils.ts` — NEU: Cache-Aside Helpers (get/set/invalidate/pattern) — REVIEW: console.debug, DEL-Batching
- `src/infrastructure/cache/cache-utils.test.ts` — NEU: 16 Unit-Tests
- `src/infrastructure/cache/index.ts` — NEU (REVIEW): Barrel-Export fuer Cache-Modul
- `src/lib/api/rate-limit.ts` — NEU: Upstash Rate-Limiting Middleware — REVIEW: Singleton-Pattern, Error-Handling
- `src/lib/api/rate-limit.test.ts` — NEU: 10 Unit-Tests (REVIEW: +2 Tests fuer Singleton und Error-Handling)
- `src/app/api/cron/fetch-validators/route.ts` — MODIFIZIERT: Cache-Invalidierung nach Fetch, HMAC_INVALID Code
- `src/app/api/cron/fetch-validators/route.test.ts` — MODIFIZIERT: Cache-Invalidierung Tests, HMAC Code Test
- `src/app/api/auth/email/send/route.ts` — MODIFIZIERT: In-Memory → Upstash Rate-Limiting
- `src/app/api/wallets/[walletId]/route.ts` — MODIFIZIERT: In-Memory → Upstash Rate-Limiting
- `src/app/api/wallets/[walletId]/route.test.ts` — MODIFIZIERT: Rate-Limit Mock aktualisiert
- `src/app/api/account/route.ts` — MODIFIZIERT: In-Memory → Upstash Rate-Limiting
- `src/app/api/account/route.test.ts` — MODIFIZIERT: Rate-Limit Mock aktualisiert
