# Architecture: Open-Universe Free Check

**Status:** Draft
**Author:** Winston (Architect)
**Date:** 2026-02-01
**Input:** `docs/_masemIT/bugs/issue-free-check-open-universe.md` (PO-reviewed)

---

## Overview

This document defines the architectural changes required to allow the Free Check feature to work with **any valid Snapshot space**, not just the 21 DAOs currently in the `daos` table.

**Core insight from code analysis:** The existing `calculateGVS(spaceId)` in `src/lib/gvs/aggregate.ts` already takes a raw `snapshotSpaceId` and fetches directly from the Snapshot API. It does NOT read from the `daos` table. The GVS engine is already "standalone" — the restriction is entirely in the **API layer** (`/api/quick-check/route.ts` lines 160-228) which gates on DB lookups.

---

## Architectural Decisions

### AD-1: No GVS Engine Refactoring Needed

**Decision:** The GVS engine (`calculateGVS`) requires **zero changes**. It already operates in standalone mode.

**Evidence:**
- `src/lib/gvs/aggregate.ts:94` — `calculateGVS(spaceId: string)` takes a Snapshot space ID directly
- Each component calculator (HPR, DEI, PDI, GPI) fetches from the Snapshot API independently
- No DB dependency in the calculation path

**What actually needs changing:** The API endpoints and UI components that gate on `daos` table / `reportedDaos` table lookups.

### AD-2: MVP Uses Synchronous Response with ReadableStream

**Decision:** On-the-fly calculations use a synchronous HTTP response with `ReadableStream` for progressive status updates. No polling, no WebSockets, no job queue.

**Rationale:**
- Simplest implementation (boring technology)
- Vercel Serverless Functions support streaming responses
- 3-20s response time is within Vercel's 30s function timeout (Pro plan)
- V2 async pattern is a separate triggered ticket (see requirement doc)

**Pattern:**
```
Client                        Server
  │── POST /api/quick-check ──→│
  │                             │── Validate space (Snapshot API)
  │←── stream: {"step":"validating"} ──│
  │                             │── Check proposal count
  │←── stream: {"step":"counting"} ──│
  │                             │── calculateGVS(spaceId)
  │←── stream: {"step":"calculating"} ──│
  │                             │── Cache in reportedDaos
  │←── stream: {"step":"done","result":{...}} ──│
```

### AD-3: Single Endpoint, Two Response Modes

**Decision:** Extend the existing `/api/quick-check` endpoint rather than creating a new one.

**Rationale:**
- The existing endpoint already handles the "known DAO" path
- Adding an "unknown DAO" path is a natural extension
- Frontend already calls this endpoint — minimal UI changes

**Response modes:**
1. **Known DAO (fast path):** Immediate JSON response (unchanged, < 200ms)
2. **Unknown DAO (stream path):** ReadableStream with progress + final result

The client distinguishes by response content-type: `application/json` vs `text/event-stream`.

### AD-4: Three-Tier Cache Hierarchy

**Decision:** Cache results at three levels. On-the-fly results are cached persistently in `reportedDaos` but never auto-promoted to `daos`.

| Tier | Storage | TTL | Scope |
|------|---------|-----|-------|
| L1 | Client state (React) | Session | Single browser tab |
| L2 | `reportedDaos` table | 24 hours | All users |
| L3 | `daos` + `gvsSnapshots` | Daily recalc | Only admin-promoted DAOs |

**Cache lookup order in `/api/quick-check`:**
1. `daos` table → `gvsSnapshots` (existing fast path)
2. `reportedDaos` table with `cached_gvs` (if `last_free_check_at` < 24h ago)
3. On-the-fly `calculateGVS()` → cache result in `reportedDaos`

### AD-5: Confidence Levels Based on Proposal Count

**Decision:** Two confidence tiers determined by proposal count, checked BEFORE running GVS.

| Proposals | Confidence | Behavior |
|-----------|------------|----------|
| < 5 | — | Reject: "Not enough governance activity" |
| 5–19 | Low | Calculate GVS + show ⚠️ badge |
| ≥ 20 | High | Calculate GVS, normal display |

**Implementation:** Fetch space metadata (`SPACE_QUERY` → `proposalsCount`) before invoking `calculateGVS()`. This is a cheap API call (~100ms) that prevents wasting 10+ seconds on DAOs with insufficient data.

### AD-6: Differentiated Rate Limiting

**Decision:** Rate limits apply ONLY to on-the-fly calculations (unknown DAOs). Cached lookups are unlimited.

| User Type | Limit | Enforcement |
|-----------|-------|-------------|
| Anonymous (no email yet) | 2 / IP / hour | IP-based via request headers |
| Free (post email-gate) | 5 / email / day | Email-based via `quickChecks` table |
| Admin (`@masem.at`) | Unlimited | Email whitelist (existing pattern) |

**Storage:** Rate limit counters in the existing `quickChecks` table (add `is_on_the_fly` boolean column to distinguish cached vs. on-the-fly checks).

### AD-7: Snapshot Space Validation Query Enhancement

**Decision:** Extend `VALIDATE_SPACE_QUERY` to include `proposalsCount` for the threshold check.

**Current query** (returns only `id`, `name`):
```graphql
query ValidateSpace($id: String!) {
  space(id: $id) { id, name }
}
```

**New query** (adds `proposalsCount` for threshold check):
```graphql
query ValidateSpaceWithCount($id: String!) {
  space(id: $id) { id, name, proposalsCount, followersCount }
}
```

This avoids a second API call. Single request validates existence + checks proposal threshold.

### AD-8: DaoAutocomplete Becomes Open Input

**Decision:** Replace the current `DaoAutocomplete` component (which fetches from `reportedDaos` API) with a hybrid component that:
1. Shows known DAOs from DB as suggestions (fast, existing)
2. Accepts **any** text input as a Snapshot space ID (new)
3. Validates the space on blur/submit via `VALIDATE_SPACE_QUERY`

**Rationale:** The current `DaoAutocomplete` only shows DAOs from the `reportedDaos` table, which is the root cause of the closed-universe problem.

---

## Files to Modify

### Backend (API Layer)

| File | Change | Complexity |
|------|--------|------------|
| `src/app/api/quick-check/route.ts` | Add on-the-fly path with ReadableStream, confidence levels, cache-check in `reportedDaos`, rate limiting | High |
| `src/app/api/dao/snapshot-available/route.ts` | Extend to validate unknown spaces via Snapshot API (not just DB lookup) | Medium |
| `src/lib/snapshot/queries.ts` | Add `VALIDATE_SPACE_WITH_COUNT_QUERY` | Low |
| `src/lib/snapshot/client.ts` | Add `validateSpaceWithCount()` function | Low |
| `src/lib/db/schema.ts` | Extend `reportedDaos` table with cache columns | Low |

### Frontend (UI Layer)

| File | Change | Complexity |
|------|--------|------------|
| `src/components/dao-autocomplete.tsx` | Convert from closed dropdown to hybrid open input + suggestions | High |
| `src/components/quick-check/QuickCheckEmailModal.tsx` | Handle streaming response, progressive loading states | High |
| `src/components/quick-check/QuickCheckResultsModal.tsx` | Add confidence badge (Low/High), handle on-the-fly data | Medium |
| `src/components/report-selection-modal.tsx` | Pass through confidence data, handle new availability states | Low |

### No Changes Required

| File | Why |
|------|-----|
| `src/lib/gvs/aggregate.ts` | Already standalone — takes spaceId, fetches from Snapshot API |
| `src/lib/gvs/storage.ts` | Only used for `gvsSnapshots` (tracked DAOs) — not involved in on-the-fly path |
| `src/lib/gvs/hpr.ts`, `dei.ts`, `pdi.ts`, `gpi.ts` | Component calculators are already standalone |
| `src/app/api/jobs/daily-recalculation/route.ts` | No change — on-the-fly DAOs are NOT added to daily job |

---

## Detailed Design

### 1. Extended `/api/quick-check` Route

```
POST /api/quick-check
{
  email: string,
  snapshot_space: string,
  marketing_optin?: boolean
}

Response Path A (known DAO — unchanged):
→ Content-Type: application/json
→ { success: true, data: { ... } }

Response Path B (unknown DAO — new):
→ Content-Type: text/event-stream
→ Stream events:
   data: {"step":"validating","message":"Validating Snapshot space..."}
   data: {"step":"counting","message":"Checking governance activity...","proposalCount":42}
   data: {"step":"calculating","message":"Analyzing voting patterns..."}
   data: {"step":"scoring","message":"Calculating governance score..."}
   data: {"step":"done","result":{"gvs_score":72,"hpr_score":68,...,"confidence":"high"}}
   OR
   data: {"step":"error","error":"not_found","message":"This Snapshot space doesn't exist."}
   data: {"step":"error","error":"insufficient_data","message":"Not enough governance activity.","proposalCount":3}
```

### 2. Algorithm Flow (Server-Side)

```
POST /api/quick-check { email, snapshot_space }
│
├── 1. Rate limit check (AD-6)
│   └── Over limit? → 429 + error message
│
├── 2. Cache L3: Check daos table
│   └── Found? → Return cached gvsSnapshot (FAST PATH, unchanged)
│
├── 3. Cache L2: Check reportedDaos.cached_gvs
│   └── Found AND last_free_check_at < 24h? → Return cached result
│
├── 4. START STREAM RESPONSE
│   │
│   ├── 4a. Validate space (validateSpaceWithCount)
│   │   ├── Space not found → stream error "not_found"
│   │   └── Space found → stream step "validating" + proposalCount
│   │
│   ├── 4b. Check proposal threshold (AD-5)
│   │   ├── < 5 proposals → stream error "insufficient_data"
│   │   └── ≥ 5 proposals → continue
│   │
│   ├── 4c. Calculate GVS (existing engine, unchanged)
│   │   stream step "calculating"
│   │   const result = await calculateGVS(snapshot_space)
│   │
│   ├── 4d. Determine confidence
│   │   proposalCount < 20 → "low" : "high"
│   │
│   ├── 4e. Cache result in reportedDaos (upsert)
│   │   UPDATE reportedDaos SET cached_gvs = {...}, last_free_check_at = NOW(),
│   │   confidence_level = "low"|"high", proposal_count = N, free_check_count++
│   │
│   ├── 4f. Store in quickChecks table (existing, for rate limiting)
│   │
│   ├── 4g. Store/update lead (existing lead capture logic)
│   │
│   └── 4h. Stream final result
│       stream step "done" + { gvs_score, hpr, dei, pdi, gpi, confidence, ... }
```

### 3. `reportedDaos` Schema Extension

```sql
ALTER TABLE "reported_daos"
  ADD COLUMN IF NOT EXISTS last_free_check_at TIMESTAMP,
  ADD COLUMN IF NOT EXISTS free_check_count INTEGER DEFAULT 0,
  ADD COLUMN IF NOT EXISTS confidence_level VARCHAR(10),
  ADD COLUMN IF NOT EXISTS proposal_count INTEGER,
  ADD COLUMN IF NOT EXISTS cached_gvs JSONB;
```

The `cached_gvs` JSONB stores the full GVS result:
```json
{
  "gvs": 6.82,
  "hpr": 7.1,
  "dei": 6.5,
  "pdi": 6.2,
  "gpi": 7.4,
  "confidence": "high",
  "calculated_at": "2026-02-01T12:00:00Z"
}
```

### 4. Frontend: Streaming Client

The `QuickCheckEmailModal` needs to handle two response types:

```typescript
const response = await fetch('/api/quick-check', { method: 'POST', body, headers })

if (response.headers.get('content-type')?.includes('text/event-stream')) {
  // STREAMING PATH (on-the-fly calculation)
  const reader = response.body!.getReader()
  const decoder = new TextDecoder()

  while (true) {
    const { done, value } = await reader.read()
    if (done) break

    const lines = decoder.decode(value).split('\n')
    for (const line of lines) {
      if (!line.startsWith('data: ')) continue
      const event = JSON.parse(line.slice(6))

      if (event.step === 'done') {
        onSuccess(event.result)
        return
      }
      if (event.step === 'error') {
        setError(event.message)
        return
      }
      // Update loading step UI
      setLoadingStep(event.message)
    }
  }
} else {
  // JSON PATH (cached result — existing behavior)
  const data = await response.json()
  // ... existing handling
}
```

### 5. Frontend: Progressive Loading UI

Replace the current single-state `<Loader2 />` spinner with a stepped progress indicator:

```
Step 1: ✅ "Validating Snapshot space..."
Step 2: ✅ "Checking governance activity..." (shows proposal count)
Step 3: ⏳ "Analyzing voting patterns..."     ← currently here
Step 4: ○ "Calculating governance score..."
```

Each step transitions as stream events arrive. Final step reveals the score with a short fade-in animation.

### 6. DaoAutocomplete → Open Input Hybrid

**Current:** Fetches from `/api/admin/reported-daos` → shows as dropdown options.
**New:** Keeps existing dropdown suggestions BUT also allows free-text input.

```
[  Enter Snapshot space ID (e.g., ens.eth)   ▼ ]
  ┌─────────────────────────────────────────────┐
  │ 📋 Known DAOs                               │
  │   Uniswap (uniswap)                        │
  │   Aave (aave.eth)                           │
  │   Gitcoin (gitcoindao.eth)                  │
  │ ─────────────────────────────────────────── │
  │ 💡 Or type any Snapshot space ID            │
  │   e.g., "makerdao-sky.eth"                  │
  └─────────────────────────────────────────────┘
```

On blur/submit with custom input → call `validateSpaceWithCount()` to verify the space exists before proceeding.

---

## Monitoring & Analytics

### Custom Events (masemIT Analytics)

| Event | Properties | Purpose |
|-------|-----------|---------|
| `free_check_on_the_fly` | `space_id`, `duration_ms`, `proposal_count`, `confidence` | V2 async trigger monitoring |
| `free_check_cached` | `space_id`, `cache_source` (`daos`/`reportedDaos`) | Cache hit rate |
| `free_check_rejected` | `space_id`, `reason` (`not_found`/`insufficient_data`/`rate_limited`) | Rejection analysis |
| `free_check_error` | `space_id`, `error_type` | Error monitoring |

### V2 Async Triggers (from requirement doc)

Monitor these metrics to determine when V2 async migration is needed:
- **T1:** `free_check_on_the_fly` events > 100 / week
- **T2:** `free_check_error` with `error_type: timeout` > 0
- **T3:** `free_check_on_the_fly` p95 `duration_ms` > 25000

---

## Migration Path

### Database Migration

Single migration file with 5 new columns on `reportedDaos`:

```
drizzle/XXXX_open_universe.sql
```

Non-breaking — all new columns are nullable with defaults. Zero downtime.

### Rollout Strategy

1. **Deploy backend changes** (extended `/api/quick-check` + schema migration)
2. **Deploy frontend changes** (open input + streaming handler + progressive loading)
3. **Monitor** first 24h: check error rates, cache hit ratios, response times
4. **Enable rate limits** (can be soft-launched with high limits initially)

No feature flag needed — the change is additive. Known DAOs continue to work identically.

---

## Risk Assessment

| Risk | Likelihood | Impact | Mitigation |
|------|-----------|--------|------------|
| Snapshot API timeout for large DAOs | Medium | Medium | Vercel 30s limit is hard. Stream allows partial progress before timeout. V2 async as escape hatch. |
| Abuse: Someone hammers on-the-fly checks | Low | Medium | Rate limiting (AD-6). Anonymous limit: 2/IP/hour. |
| Cache poisoning: Incorrect scores cached | Low | High | 24h TTL. Admin-only promotion to `daos` table. `reportedDaos` is never used for DGI calculations. |
| Snapshot API rate limiting | Low | Medium | Existing circuit breaker + retry logic handles this. |

---

## What This Architecture Does NOT Cover

1. **Young DAO Notification Opt-In** — Separate ticket per PO decision
2. **V2 Async Pattern** — Separate triggered ticket (see requirement doc for triggers)
3. **Snapshot Typeahead/Search** — R9 (Should-Have), not in MVP scope
4. **Admin Promote Flow** — Already exists in admin UI (`/api/admin/daos`), no changes needed
