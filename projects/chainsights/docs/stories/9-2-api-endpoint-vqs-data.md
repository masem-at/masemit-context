# Story 9.2: API Endpoint for VQS Data

Status: done

## Story

As a frontend developer,
I want an API endpoint that returns per-voter VQS data for a DAO,
so that the Matrix Details page can display delegate quality scores.

## Acceptance Criteria

1. **AC-9.2.1: Route handler** — `GET /api/dao/[spaceId]/vqs` returns VQS data for a DAO
   - Route file: `src/app/api/dao/[spaceId]/vqs/route.ts`
   - Dynamic segment `[spaceId]` maps directly to `voter_quality_signals.space_id`
   - `export const dynamic = 'force-dynamic'` (no ISR — data changes with each GVS run)

2. **AC-9.2.2: Response shape** — JSON matching the epic spec:
   ```json
   {
     "spaceId": "lido-snapshot.eth",
     "calculationDate": "2026-02-12",
     "totalVotersAnalyzed": 245,
     "voters": [
       {
         "address": "0x7eE0...",
         "vqsScore": 4.1,
         "deliberationScore": 2.1,
         "independenceScore": 3.4,
         "focusScore": 6.2,
         "originalityScore": 4.5,
         "patternLabel": "Rapid, Consistent",
         "totalVotesAnalyzed": 47,
         "checkboxScore": 0.72
       }
     ]
   }
   ```

3. **AC-9.2.3: Latest snapshot query** — Query `voter_quality_signals` for the latest `snapshot_date` per spaceId
   - First query: `SELECT MAX(snapshot_date) FROM voter_quality_signals WHERE space_id = $1`
   - Second query: fetch voter rows for that date
   - If no rows found for spaceId → return `{ spaceId, calculationDate: null, totalVotersAnalyzed: 0, voters: [] }`

4. **AC-9.2.4: Minimum vote threshold** — Only return voters with `total_votes_analyzed >= 3` (consistent with Phase 1 MIN_VOTES_FOR_ANALYSIS)

5. **AC-9.2.5: VQS-only filter** — Only return voters where `vqi_score IS NOT NULL` (Phase 2 VQS has been computed). Voters with only Phase 1 data (no VQS) are excluded.

6. **AC-9.2.6: Sort order** — Sorted by `vqi_score ASC` (lowest quality first — most interesting for analysis)

7. **AC-9.2.7: Limit parameter** — Optional `?limit=N` query parameter
   - Default: 50
   - Maximum: 200
   - Clamp invalid/negative values to default
   - Non-numeric values → default

8. **AC-9.2.8: No auth required** — VQS data is public (same as GVS scores on Matrix page). No session check.

9. **AC-9.2.9: Score scale conversion** — DB stores 0-1 range, API returns 0-10 range:
   - `vqsScore = vqi_score * 10`
   - `focusScore = cross_category_signal * 10`
   - `originalityScore = (1 - pattern_correlation_signal) * 10`
   - `deliberationScore = (1 - speed_signal) * 10`
   - `independenceScore = (1 - diversity_signal) * 10`

10. **AC-9.2.10: Pattern label generation** — `patternLabel` is NOT stored in DB; generate at read-time using `generatePatternLabel(deliberationScore, independenceScore)` from `checkbox-detection.ts`

11. **AC-9.2.11: Error handling** — Standard project patterns:
    - `isDatabaseConfigured()` guard → 503
    - Empty spaceId → 400
    - Internal error → 500 with `[VQS API]` log prefix

12. **AC-9.2.12: Cache headers** — `Cache-Control: public, s-maxage=3600, stale-while-revalidate=1800` (same as Attention Map API — data refreshes daily)

## Tasks / Subtasks

- [x] Task 1: Create route handler file (AC: 1, 8, 11, 12)
  - [x] 1.1 Create directory `src/app/api/dao/[spaceId]/vqs/`
  - [x] 1.2 Implement GET handler with `isDatabaseConfigured()` guard
  - [x] 1.3 Extract and validate `spaceId` from dynamic params (`{ params }: { params: Promise<{ slug: string }> }` pattern from attention API — but param name is `spaceId`)
  - [x] 1.4 Parse `?limit=N` from searchParams, clamp to 1-200, default 50
  - [x] 1.5 Add `export const dynamic = 'force-dynamic'`
  - [x] 1.6 Add Cache-Control headers on success response
  - [x] 1.7 Error handling: 503 (no DB), 400 (missing spaceId), 500 (internal)

- [x] Task 2: DB query — latest snapshot date (AC: 3)
  - [x] 2.1 Query `MAX(snapshot_date)` from `voter_quality_signals` WHERE `space_id = spaceId`
  - [x] 2.2 If no rows → return empty response `{ spaceId, calculationDate: null, totalVotersAnalyzed: 0, voters: [] }`

- [x] Task 3: DB query — voter VQS rows (AC: 3, 4, 5, 6, 7)
  - [x] 3.1 SELECT relevant columns from `voter_quality_signals` WHERE `space_id = spaceId` AND `snapshot_date = latestDate` AND `total_votes_analyzed >= 3` AND `vqi_score IS NOT NULL`
  - [x] 3.2 ORDER BY `vqi_score ASC`
  - [x] 3.3 LIMIT from parsed query parameter
  - [x] 3.4 Also count total voters matching the WHERE (without LIMIT) for `totalVotersAnalyzed`

- [x] Task 4: Response mapping (AC: 2, 9, 10)
  - [x] 4.1 Convert each DB row to response format with 0-10 scale scores
  - [x] 4.2 Generate `patternLabel` using imported `generatePatternLabel()` from `@/lib/gvs/checkbox-detection`
  - [x] 4.3 Assemble final response JSON with spaceId, calculationDate, totalVotersAnalyzed, voters array

## Dev Notes

- **First route under `dao/[spaceId]/`** — This will be the first API route using a dynamic `[spaceId]` segment. Existing DAO routes use `dao/snapshot-available?dao=...` (query param) or `matrix/[slug]/attention` (slug-based). The VQS route uses spaceId directly.

- **DB column mapping is non-obvious** — The DB column names from Phase 1 don't match the Phase 2 semantics:
  - `cross_category_signal` → focusScore (stored as 0-1, display as 0-10)
  - `pattern_correlation_signal` → INVERTED originalityScore (stored as `1 - orig/10`, so `orig = (1 - stored) * 10`)
  - `vqi_score` → vqsScore (stored as 0-1, display as 0-10)
  - See `checkbox-storage.ts:57-59` for the storage transforms and reverse them.

- **No dedicated query file needed** — Follow the Attention Map pattern (`matrix/[slug]/attention/route.ts`) with inline Drizzle queries. Single read-only endpoint doesn't justify a separate queries module.

- **DB indexes already exist** — `idx_vqs_space_id` on `(space_id)` and `idx_vqs_space_voter_date` unique on `(space_id, voter, snapshot_date)`. The space_id index covers the MAX(snapshot_date) subquery. If VQS sorting performance becomes an issue, a `(space_id, vqi_score)` index can be added later (noted in epic).

- **Pattern: Attention Map API** (`src/app/api/matrix/[slug]/attention/route.ts`) — Closest reference for:
  - Dynamic segment params extraction: `const { slug } = await params`
  - Cache headers: `s-maxage=3600, stale-while-revalidate=1800`
  - Error logging: `console.error('[VQS API] ...')`

- **Pattern: DGI API** (`src/app/api/dgi/route.ts`) — Reference for:
  - `export const dynamic = 'force-dynamic'`
  - Public endpoint with no auth
  - Clean response structure

### Project Structure Notes

- New directory: `src/app/api/dao/[spaceId]/vqs/` — fits Next.js 14 App Router conventions
- Imports from existing modules only: `@/lib/db`, `@/lib/db/schema`, `@/lib/gvs/checkbox-detection`
- No new shared modules or types needed — response shape is route-local
- Drizzle ORM for all DB queries (project constraint from `project_context.md`)
- TypeScript strict mode — ensure all nullable DB columns are handled

### References

- [Source: docs/stories/epic-checkbox-voting-phase2.md#Story 9-2] — Epic AC spec, response shape, sort order
- [Source: src/app/api/matrix/[slug]/attention/route.ts] — Dynamic segment pattern, Cache-Control, inline queries
- [Source: src/app/api/dgi/route.ts] — Public endpoint pattern, `force-dynamic`
- [Source: src/lib/db/schema.ts#L959-991] — `voterQualitySignals` table definition and indexes
- [Source: src/lib/gvs/checkbox-storage.ts#L57-59] — Phase 2 storage transforms (inverse needed for read)
- [Source: src/lib/gvs/checkbox-detection.ts#L479-491] — `generatePatternLabel()` function to import
- [Source: docs/project_context.md] — Tech stack constraints (Drizzle, strict TS, snake_case DB)

## Dev Agent Record

### Agent Model Used
- Claude Opus 4.6

### Debug Log References
- No debug issues encountered.

### Completion Notes List
- Implemented GET /api/dao/[spaceId]/vqs route handler following Attention Map API and DGI API patterns
- Route uses inline Drizzle queries (no separate query module per story spec)
- Score conversion: DB 0-1 range → API 0-10 range with correct inversion for deliberation, independence, originality
- Pattern label generated at read-time via imported `generatePatternLabel()` from checkbox-detection.ts
- Limit parameter: default 50, max 200, clamped for invalid/negative/non-numeric values
- Cache-Control: `public, s-maxage=3600, stale-while-revalidate=1800`
- 11 unit tests covering: 503/400/500 error paths, empty response, score conversion, cache headers, limit clamping
- ESLint: clean, no warnings

### Change Log
- Story 9-1 learnings: DB stores Phase 2 signals in 0-1 range with inverted semantics for some columns. Always verify transform direction against `checkbox-storage.ts`.
- Story 9-1 code review: `generatePatternLabel()` is a pure function exported from `checkbox-detection.ts` — safe to import and call from API route without side effects.
- 2026-02-12: Implemented VQS API endpoint with all 12 ACs satisfied. Created route handler and 11 unit tests.
- 2026-02-12: **Code Review (AI)** — 3 issues fixed:
  - [H1] checkboxScore precision loss: removed unnecessary `round1()` — raw DB value now passed through (AC-9.2.2 compliance)
  - [M1] Removed unused `desc` import from drizzle-orm
  - [M2] Simplified `generatePatternLabel` test mock — no longer reimplements production logic; uses simple mock + argument verification

### File List

- `src/app/api/dao/[spaceId]/vqs/route.ts` — **NEW** — API route handler (main deliverable)
- `tests/unit/api/vqs-route.test.ts` — **NEW** — 11 unit tests for VQS API route
