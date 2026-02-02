# Epic: Open-Universe Free Check

**Status:** Ready for Implementation
**Requirement:** `docs/_masemIT/bugs/issue-free-check-open-universe.md`
**Architecture:** `docs/architecture-open-universe.md`
**Estimated Effort:** 31–33h
**Priority:** HIGH — Conversion-Killer / DGI Blocker

---

## Epic Overview

Enable the Free Check to work with **any valid Snapshot space** (13,000+), not just the 21 DAOs in the `daos` table. Key architectural insight: The GVS engine is already standalone (AD-1) — changes are limited to API layer, caching, and UI.

---

## Story Map

```
Epic: Open-Universe Free Check
│
├── Story 1: Schema + Snapshot Query (Foundation)
│
├── Story 2: API — On-the-Fly Stream Path (Core)
│   └── depends on: Story 1
│
├── Story 3: API — Cache Layer + Rate Limiting
│   └── depends on: Story 2
│
├── Story 4: UI — Open Input Hybrid (DaoAutocomplete)
│   └── depends on: Story 1
│
├── Story 5: UI — Progressive Loading + Streaming Client
│   └── depends on: Story 2, Story 4
│
├── Story 6: UI — Confidence Badge + Error States
│   └── depends on: Story 5
│
├── Story 7: Analytics + Monitoring Events
│   └── depends on: Story 2
│
└── Story 8: snapshot-available Endpoint Extension
    └── depends on: Story 1
```

---

## Story 1: Schema Extension + Snapshot Validation Query

**As a** developer,
**I want** the `reportedDaos` table extended with cache columns and a new Snapshot validation query,
**so that** the foundation exists for on-the-fly checks and caching.

**Effort:** ~2h

### Tasks

1. **Extend `reportedDaos` schema** (`src/lib/db/schema.ts`)
   - Add columns: `lastFreeCheckAt` (timestamp), `freeCheckCount` (integer, default 0), `confidenceLevel` (varchar(10)), `proposalCount` (integer), `cachedGvs` (jsonb)
   - All nullable with defaults — non-breaking migration
   - Run `npm run db:generate` to create migration

2. **Add `VALIDATE_SPACE_WITH_COUNT_QUERY`** (`src/lib/snapshot/queries.ts`)
   ```graphql
   query ValidateSpaceWithCount($id: String!) {
     space(id: $id) { id, name, proposalsCount, followersCount }
   }
   ```

3. **Add `validateSpaceWithCount()` function** (`src/lib/snapshot/client.ts`)
   - Returns `{ valid: boolean, name?: string, proposalsCount?: number, error?: string }`
   - Uses existing `executeQuery()` with retry + circuit breaker

### Acceptance Criteria

- [ ] AC-1.1: `reportedDaos` table has 5 new columns (migration generated and pushable)
- [ ] AC-1.2: `validateSpaceWithCount('ens.eth')` returns `{ valid: true, name: 'ENS', proposalsCount: N }`
- [ ] AC-1.3: `validateSpaceWithCount('nonexistent.eth')` returns `{ valid: false, error: '...' }`
- [ ] AC-1.4: Existing `reportedDaos` queries and data are unaffected (non-breaking)
- [ ] AC-1.5: Type exports updated for new `reportedDaos` columns

### Files

- `src/lib/db/schema.ts` — extend `reportedDaos` table
- `src/lib/snapshot/queries.ts` — add `VALIDATE_SPACE_WITH_COUNT_QUERY`
- `src/lib/snapshot/client.ts` — add `validateSpaceWithCount()`

---

## Story 2: API — On-the-Fly Stream Path

**As a** user entering an unknown DAO,
**I want** the `/api/quick-check` endpoint to calculate my GVS on-the-fly with progressive status updates,
**so that** I get a governance score for any valid Snapshot space.

**Effort:** ~8h
**Depends on:** Story 1

### Tasks

1. **Refactor `/api/quick-check/route.ts`** — add three-path lookup:
   - Path A: `daos` table → `gvsSnapshots` (existing fast path, unchanged)
   - Path B: `reportedDaos` with valid `cachedGvs` (TTL < 24h) → return cached JSON
   - Path C: Unknown DAO → ReadableStream with SSE events

2. **Implement SSE stream response** for Path C:
   - Stream events: `validating` → `counting` → `calculating` → `scoring` → `done`/`error`
   - Content-Type: `text/event-stream`
   - Format: `data: {"step":"...","message":"..."}\n\n`

3. **Proposal count threshold check** (AD-5):
   - Call `validateSpaceWithCount()` before `calculateGVS()`
   - < 5 proposals → stream error `insufficient_data`
   - 5–19 → continue with confidence `low`
   - ≥ 20 → continue with confidence `high`

4. **Cache result in `reportedDaos`** after successful calculation:
   - Upsert by `spaceId`
   - Set `cachedGvs` JSONB, `lastFreeCheckAt`, `confidenceLevel`, `proposalCount`, increment `freeCheckCount`

5. **Preserve existing behavior**: Path A (known DAOs) must remain identical — same response format, same performance.

### Acceptance Criteria

- [ ] AC-2.1: Known DAOs return JSON immediately (< 200ms, existing format unchanged)
- [ ] AC-2.2: Unknown valid space with ≥ 20 proposals returns SSE stream → final `done` event with GVS + 4 components + `confidence: "high"`
- [ ] AC-2.3: Unknown valid space with 5–19 proposals returns SSE stream → final `done` event with `confidence: "low"`
- [ ] AC-2.4: Unknown valid space with < 5 proposals returns SSE stream → `error` event with `insufficient_data`
- [ ] AC-2.5: Non-existent space returns SSE stream → `error` event with `not_found`
- [ ] AC-2.6: Result is cached in `reportedDaos.cachedGvs` with timestamp
- [ ] AC-2.7: Subsequent check of same space within 24h returns cached JSON (Path B, no stream)
- [ ] AC-2.8: Email-gate behavior (lead capture, quickChecks insert) unchanged
- [ ] AC-2.9: Stream events follow the format: `data: {"step":"...","message":"..."}\n\n`

### Files

- `src/app/api/quick-check/route.ts` — major refactor (three-path lookup + stream)

### Error Handling (from Error-Copy-Guide)

| Scenario | Stream Error Event |
|----------|-------------------|
| Space not found | `{"step":"error","error":"not_found","message":"This Snapshot space doesn't exist. Double-check the ENS name and try again."}` |
| < 5 proposals | `{"step":"error","error":"insufficient_data","message":"This DAO is still early — not enough governance activity for a meaningful score yet. We need at least 5 closed proposals.","proposalCount":N}` |
| Snapshot API down | `{"step":"error","error":"api_unavailable","message":"We're having trouble reaching Snapshot's API right now. Please try again in a few minutes."}` |
| GVS calculation failed | `{"step":"error","error":"calculation_failed","message":"Something went wrong analyzing this DAO. Our team has been notified. Try again or contact us."}` |

---

## Story 3: API — Cache Layer + Rate Limiting

**As a** system operator,
**I want** on-the-fly checks cached for 24h and rate-limited per user type,
**so that** we avoid redundant calculations and prevent API abuse.

**Effort:** ~4h
**Depends on:** Story 2

### Tasks

1. **Implement L2 cache check** in quick-check route:
   - Before starting stream: check `reportedDaos` for `cachedGvs` where `lastFreeCheckAt` > 24h ago
   - If valid cache: return standard JSON response (not stream)
   - If cache expired or missing: fall through to stream path

2. **Add `isOnTheFly` column** to `quickChecks` table:
   - Boolean, default false
   - Set to true when on-the-fly calculation is performed
   - Used to distinguish rate limit scopes

3. **Implement differentiated rate limiting** (AD-6):
   - Anonymous (no email submitted yet): 2 on-the-fly checks / IP / hour
   - Free (post email-gate): 5 on-the-fly checks / email / day
   - Admin (`@masem.at`): unlimited (existing whitelist pattern)
   - Cached checks do NOT count against limits

4. **Rate limit error response:**
   - HTTP 429 with message from Error-Copy-Guide
   - Include `retryAfter` field (seconds until next allowed check)

### Acceptance Criteria

- [ ] AC-3.1: Second check of same space within 24h returns cached JSON (no stream, < 200ms)
- [ ] AC-3.2: Cache expired (> 24h) → fresh on-the-fly calculation
- [ ] AC-3.3: Anonymous user gets 429 after 2nd on-the-fly check within 1 hour
- [ ] AC-3.4: Free user gets 429 after 5th on-the-fly check within 1 day
- [ ] AC-3.5: `@masem.at` email has no rate limit on on-the-fly checks
- [ ] AC-3.6: Cached/known DAO lookups are NOT rate-limited (unlimited)
- [ ] AC-3.7: Rate limit response includes `retryAfter` seconds
- [ ] AC-3.8: `quickChecks` table has `isOnTheFly` column

### Files

- `src/app/api/quick-check/route.ts` — cache check + rate limiting logic
- `src/lib/db/schema.ts` — add `isOnTheFly` to `quickChecks`

---

## Story 4: UI — Open Input Hybrid (DaoAutocomplete)

**As a** user,
**I want** to type any Snapshot space ID in the Free Check input,
**so that** I'm not limited to pre-selected DAOs.

**Effort:** ~5h
**Depends on:** Story 1

### Tasks

1. **Convert `DaoAutocomplete` to hybrid component** (`src/components/dao-autocomplete.tsx`):
   - Keep existing dropdown suggestions from known DAOs
   - Allow free-text input (any string accepted)
   - Show helper text: "Or type any Snapshot space ID (e.g., makerdao-sky.eth)"
   - On selection of known DAO: behave as before (immediate)
   - On submit of custom input: validate via Snapshot API

2. **Add space validation on blur/submit**:
   - Call `/api/dao/snapshot-available?dao=<input>` (extended in Story 8)
   - Show inline validation state: loading spinner → ✅ valid / ❌ "Space not found"
   - Only allow proceeding to tier selection when validation passes

3. **Update `report-selection-modal.tsx`**:
   - Handle the case where user enters unknown space (no `preSelectedDao`)
   - Pass validation status through to tier selection step

4. **Update placeholder text and CTA**:
   - Input placeholder: "Enter Snapshot space ID (e.g., ens.eth)" instead of current dropdown-only UX
   - Retain "Select" behavior for known DAOs in the dropdown

### Acceptance Criteria

- [ ] AC-4.1: User can type any text into the DAO input field
- [ ] AC-4.2: Known DAOs appear as dropdown suggestions (existing behavior preserved)
- [ ] AC-4.3: Custom space input is validated via Snapshot API on blur/submit
- [ ] AC-4.4: Valid custom space shows ✅ indicator and allows proceeding
- [ ] AC-4.5: Invalid custom space shows ❌ "This Snapshot space doesn't exist"
- [ ] AC-4.6: Loading spinner shown during validation
- [ ] AC-4.7: Helper text visible: "Or type any Snapshot space ID"
- [ ] AC-4.8: Pre-selected DAO flow (from Matrix detail page) unchanged

### Files

- `src/components/dao-autocomplete.tsx` — major refactor to hybrid
- `src/components/report-selection-modal.tsx` — handle unknown space flow

---

## Story 5: UI — Progressive Loading + Streaming Client

**As a** user waiting for an on-the-fly check,
**I want** to see step-by-step progress instead of a generic spinner,
**so that** I know something is happening and trust the process.

**Effort:** ~6h
**Depends on:** Story 2, Story 4

### Tasks

1. **Add SSE stream handler** to `QuickCheckEmailModal`:
   - Detect response content-type (`text/event-stream` vs `application/json`)
   - For stream: read chunks with `ReadableStream` reader
   - Parse `data: {...}` lines, update UI per event
   - For JSON: existing handler (unchanged)

2. **Implement progressive loading UI**:
   - Replace single `<Loader2>` spinner with stepped progress list:
     ```
     ✅ Validating Snapshot space...
     ✅ Checking governance activity... (42 proposals found)
     ⏳ Analyzing voting patterns...
     ○  Calculating governance score...
     ```
   - Each step transitions as stream events arrive
   - Current step shows spinner icon, completed steps show ✅
   - Pending steps shown as ○ (gray)

3. **Score reveal animation**:
   - On `done` event: brief fade-in/scale animation for the score display
   - Transition from loading state to results modal

4. **Error state handling in stream**:
   - On `error` event: show appropriate message from Error-Copy-Guide
   - Error messages displayed inline (not as toast/alert)
   - Include retry button for retriable errors (API down)
   - Include "Contact us" link for non-retriable errors

5. **Handle stream interruption**:
   - Network disconnect during stream → "Connection lost. Please try again."
   - ReadableStream reader error → graceful fallback message

### Acceptance Criteria

- [ ] AC-5.1: Known DAOs show existing instant behavior (no progressive loading)
- [ ] AC-5.2: Unknown DAOs show 4-step progressive loading with step transitions
- [ ] AC-5.3: Each completed step shows ✅, current step shows spinner, pending shows ○
- [ ] AC-5.4: Proposal count shown in step 2 when available (e.g., "42 proposals found")
- [ ] AC-5.5: Score reveal has fade-in animation
- [ ] AC-5.6: Stream errors show user-friendly message (from Error-Copy-Guide)
- [ ] AC-5.7: Network interruption shows "Connection lost" with retry button
- [ ] AC-5.8: JSON responses (cached) handled identically to current behavior

### Files

- `src/components/quick-check/QuickCheckEmailModal.tsx` — stream handler + progressive loading UI

---

## Story 6: UI — Confidence Badge + Error States

**As a** user viewing an on-the-fly check result,
**I want** to see a confidence indicator for DAOs with limited data,
**so that** I understand the score may be preliminary.

**Effort:** ~3h
**Depends on:** Story 5

### Tasks

1. **Add confidence badge to `QuickCheckResultsModal`**:
   - `confidence: "low"` → Yellow badge: "⚠️ Preliminary score — based on limited governance history. Scores may shift as more proposals are created."
   - `confidence: "high"` → No badge (normal display)
   - Badge positioned below GVS score, above component breakdown

2. **Add `confidence` field to `QuickCheckResult` interface**:
   - Extend type: `confidence?: 'low' | 'high'`
   - Pass through from stream `done` event or cached response

3. **Update error states** for new scenarios:
   - "Not enough governance activity" — shown when < 5 proposals (from stream error)
   - "Space not found" — shown when space doesn't exist
   - Both with clear, non-technical copy from Error-Copy-Guide

4. **Handle `data_source` distinction**:
   - On-the-fly results: show confidence badge
   - Cached results from `reportedDaos`: show confidence from cached data
   - Results from `daos` table: no badge (tracked DAOs = established)

### Acceptance Criteria

- [ ] AC-6.1: Low confidence result shows yellow ⚠️ badge with explanatory text
- [ ] AC-6.2: High confidence result shows no badge (normal display)
- [ ] AC-6.3: Results from tracked DAOs (`daos` table) never show confidence badge
- [ ] AC-6.4: "Not enough activity" message shown for < 5 proposals (wertschaetzend, not technical)
- [ ] AC-6.5: "Space not found" message shown for invalid spaces
- [ ] AC-6.6: `QuickCheckResult` interface includes optional `confidence` field
- [ ] AC-6.7: Badge text matches Error-Copy-Guide copy exactly

### Files

- `src/components/quick-check/QuickCheckResultsModal.tsx` — confidence badge + error states
- `src/components/quick-check/QuickCheckEmailModal.tsx` — pass confidence through

---

## Story 7: Analytics + Monitoring Events

**As a** product owner,
**I want** analytics events for on-the-fly checks,
**so that** I can monitor usage, cache hit rates, and V2 async trigger conditions.

**Effort:** ~2h
**Depends on:** Story 2

### Tasks

1. **Add server-side analytics events** in `/api/quick-check`:
   - `free_check_on_the_fly`: `{ space_id, duration_ms, proposal_count, confidence }`
   - `free_check_cached`: `{ space_id, cache_source }` (`daos` or `reportedDaos`)
   - `free_check_rejected`: `{ space_id, reason }` (`not_found`, `insufficient_data`, `rate_limited`)
   - `free_check_error`: `{ space_id, error_type }`

2. **Add client-side tracking events**:
   - `quick_check_stream_start`: User initiated on-the-fly check
   - `quick_check_stream_complete`: On-the-fly check completed successfully (includes `duration_ms`)
   - `quick_check_stream_error`: On-the-fly check failed

3. **Ensure `duration_ms` is captured** for V2 async trigger monitoring:
   - Measure from stream start to `done` event
   - Send to masemIT Analytics as custom event property

### Acceptance Criteria

- [ ] AC-7.1: `free_check_on_the_fly` event fires for every on-the-fly calculation with `duration_ms`
- [ ] AC-7.2: `free_check_cached` event fires for every cache hit (L2 and L3)
- [ ] AC-7.3: `free_check_rejected` event fires for not-found, insufficient-data, rate-limited
- [ ] AC-7.4: Client-side events tracked via existing `trackEvent()` from `@/lib/analytics`
- [ ] AC-7.5: `duration_ms` measured accurately (server-side start to result ready)

### Files

- `src/app/api/quick-check/route.ts` — server-side events
- `src/components/quick-check/QuickCheckEmailModal.tsx` — client-side events

---

## Story 8: Snapshot-Available Endpoint Extension

**As a** developer,
**I want** the `/api/dao/snapshot-available` endpoint to check Snapshot API for unknown spaces,
**so that** the DaoAutocomplete can validate any input.

**Effort:** ~2h
**Depends on:** Story 1

### Tasks

1. **Extend `/api/dao/snapshot-available/route.ts`**:
   - Current: Only checks `daos` table → `reportedDaos` table → returns `available: false`
   - New: After DB miss, call `validateSpaceWithCount()` → if valid, return:
     ```json
     {
       "available": true,
       "source": "snapshot_api",
       "proposalCount": 42,
       "spaceName": "MakerDAO Sky",
       "dataQuality": "on_the_fly"
     }
     ```
   - If not valid → return `{ available: false, error: "Space not found" }`

2. **Return proposal count** for frontend threshold display:
   - Frontend can show "42 proposals available" before user starts the check
   - Frontend can warn "Only 3 proposals — not enough for analysis" preemptively

### Acceptance Criteria

- [ ] AC-8.1: Known DAOs return existing response format (unchanged)
- [ ] AC-8.2: Unknown valid space returns `source: "snapshot_api"` with `proposalCount` and `spaceName`
- [ ] AC-8.3: Unknown invalid space returns `available: false` with error message
- [ ] AC-8.4: Response includes `proposalCount` for threshold display
- [ ] AC-8.5: Snapshot API errors handled gracefully (return `available: false` with generic error)

### Files

- `src/app/api/dao/snapshot-available/route.ts` — extend with Snapshot API fallback

---

## Implementation Order (Recommended)

```
Week 1: Foundation + Core API
  Story 1 (Schema + Query)           ~2h   ← START HERE
  Story 2 (Stream Path)              ~8h   ← Core feature
  Story 8 (snapshot-available)        ~2h   ← Can parallel with Story 2

Week 2: UI + Polish
  Story 4 (Open Input)               ~5h
  Story 5 (Progressive Loading)      ~6h
  Story 3 (Cache + Rate Limiting)    ~4h   ← Can parallel with Story 5
  Story 6 (Confidence Badge)         ~3h
  Story 7 (Analytics)                ~2h

Total: ~32h
```

**Critical path:** Story 1 → Story 2 → Story 5 → Story 6

**Parallelizable:**
- Story 8 can run in parallel with Story 2 (both depend only on Story 1)
- Story 4 can run in parallel with Story 2 (both depend only on Story 1)
- Story 3 can run in parallel with Story 5 (Story 3 depends on Story 2, Story 5 depends on Story 2 + 4)
- Story 7 can run at any point after Story 2

---

## Out of Scope (Separate Tickets)

| Item | Status | Trigger |
|------|--------|---------|
| Young DAO Notification Opt-In | Nice-to-Have ticket | PO decision |
| V2 Async Pattern | Backlog ticket | >100 on-the-fly/week OR first timeout OR p95 >25s |
| Snapshot Typeahead/Search (R9) | Should-Have | Post-MVP |
| Positioning Update (Hero, Pricing Page) | Marketing ticket | After MVP deploy |
