# Epic: DGI (DAO Governance Index)

**Status:** Ready for Implementation
**Priority:** High
**Created:** 2026-02-01
**Requirement:** `docs/_masemIT/requirements/dao-governance-index-requirement-v1.1.md`
**Architecture:** `docs/architecture-dgi.md`

---

## Problem Statement

The GVS tells users how healthy one DAO is, but provides no ecosystem context. Users cannot answer "Is a GVS of 6.7 good or bad?" without a benchmark. The DGI provides that benchmark by aggregating GVS scores across all qualifying DAOs into composite, category, and component indices — turning isolated numbers into meaningful context.

---

## Architecture Decisions

- **AD-1:** Extend existing `schema.ts` — add `dgiSnapshots`, `daoIndexMembership` tables, `dgiIndexTypeEnum`, extend `daoCategoryEnum` with `'social'`
- **AD-2:** DGI calculation as post-hook in `executeDailyRecalculation()` (non-blocking)
- **AD-3:** Admin CRUD at `/admin/dgi` for manual universe curation by Mario
- **AD-4:** Server-side gating of DGI data by access tier (anonymous/free/subscriber/admin)
- **AD-5:** Recharts `<ReferenceLine>` on existing `MetricChart.tsx` — replace `categoryAvg` with `benchmark`
- **AD-6:** Percentile computed at query time, not stored
- **AD-7:** `GET /api/dgi` — public, ISR-cached (1h)
- **AD-8:** Methodology page at `/methodology/dgi` — **BLOCKER for Epic 2 go-live**

---

## Epic 1: DGI Foundation

**Goal:** Calculate and store the DGI index daily. Admin can manage the DAO universe.

---

### Story 1A: Database Schema & Enum Extension

**Files:**
- `src/lib/db/schema.ts` (MODIFY)

**Acceptance Criteria:**

- AC1: `daoCategoryEnum` extended with `'social'` as 4th value (after `public_goods`)
- AC2: `dgiIndexTypeEnum` pgEnum created with values `['composite', 'defi', 'infrastructure', 'public_goods', 'social', 'hpr', 'dei', 'pdi', 'gpi']`
- AC3: `dgiSnapshots` table created with columns: `id` (uuid PK), `snapshotDate` (timestamp, not null), `indexType` (dgiIndexTypeEnum, not null), `value` (numeric(5,2), not null), `daoCount` (integer, not null), `createdAt` (timestamp)
- AC4: `dgiSnapshots` has unique compound index on `(snapshotDate, indexType)` and index on `indexType`
- AC5: `daoIndexMembership` table created with columns: `id` (uuid PK), `daoId` (uuid FK → daos.id, onDelete cascade, not null), `category` (daoCategoryEnum, not null), `includedSince` (timestamp, default now, not null), `excludedAt` (timestamp, nullable), `exclusionReason` (text, nullable), `gracePeriodUntil` (timestamp, nullable — manual note field), `createdAt` (timestamp)
- AC6: `daoIndexMembership` has index on `daoId` and compound index on `(daoId, excludedAt)` for active member queries
- AC7: Type exports added: `DGISnapshot`, `DGISnapshotInsert`, `DaoIndexMembership`, `DaoIndexMembershipInsert`
- AC8: `npm run db:generate` produces clean migration
- AC9: `'social'` added to `CATEGORY_STYLES` in `DaoHeader.tsx` (label: "Social", color: teal/cyan scheme)
- AC10: `'social'` added to `CATEGORY_LABELS` in `leaderboard.ts` and `LeaderboardOptions.category` union type
- AC11: TypeScript compiles with no new errors

---

### Story 1B: DGI Calculation Engine

**Files:**
- `src/lib/dgi/calculate.ts` (NEW)
- `src/lib/jobs/daily-recalculation.ts` (MODIFY)

**Acceptance Criteria:**

- AC1: `calculateAndSaveDGI(): Promise<{ indicesWritten: number; daoCount: number }>` exported from `src/lib/dgi/calculate.ts`
- AC2: Function queries `daoIndexMembership` WHERE `excludedAt IS NULL`, joined with `daos`
- AC3: Fetches latest GVS snapshots for all active members using existing `getLatestSnapshots()`
- AC4: Filters qualifying DAOs: confidence ≥ medium AND snapshotDate within last 14 days
- AC5: Computes DGI Composite = equal-weighted average of all qualifying DAOs' GVS scores
- AC6: Computes 4 category indices (defi, infrastructure, public_goods, social) — only for categories with ≥1 qualifying DAO
- AC7: Computes 4 component indices (hpr, dei, pdi, gpi) — averages of individual component scores, skipping nulls
- AC8: Batch-inserts up to 9 rows into `dgiSnapshots` (1 composite + up to 4 category + 4 component)
- AC9: If no qualifying DAOs exist (empty universe), logs warning and returns `{ indicesWritten: 0, daoCount: 0 }` — does NOT insert empty rows
- AC10: `executeDailyRecalculation()` in `daily-recalculation.ts` calls `calculateAndSaveDGI()` after the GVS processing loop
- AC11: DGI call is wrapped in try/catch — failure logs error but does NOT fail the GVS job (non-blocking per AD-2)
- AC12: DGI result (indicesWritten, daoCount) logged to console: `[DGI] Calculated X indices for Y DAOs`
- AC13: TypeScript compiles with no new errors

---

### Story 1C: Admin UI — DGI Universe Management

**Files:**
- `src/app/admin/dgi/page.tsx` (NEW)
- `src/components/admin/dgi-management-client.tsx` (NEW)
- `src/app/api/admin/dgi/membership/route.ts` (NEW)
- `src/components/admin/admin-dashboard-client.tsx` (MODIFY)

**Acceptance Criteria:**

- AC1: New admin page at `/admin/dgi`, protected by `isAdminAuthenticated()` (same as other admin pages)
- AC2: Server component fetches `daoIndexMembership` LEFT JOIN `daos` + list of all tracked DAOs not yet in membership
- AC3: Client component renders table with columns: DAO Name, Snapshot Space, Category (dropdown: DeFi/Infra/Public Goods/Social), Included (green/red badge), Included Since (date), Notes (grace_period_until date or "—")
- AC4: "Add DAO" button opens dropdown of tracked DAOs (`daos` table, `isOptedOut = false`) not already in `daoIndexMembership` (active)
- AC5: Adding a DAO creates row in `daoIndexMembership` with `includedSince = NOW()`, category from dropdown selection
- AC6: "Remove" action on a row sets `excludedAt = NOW()` — does NOT delete the row (preserves history)
- AC7: Category dropdown on each row allows changing the category via PATCH to the API route
- AC8: `POST /api/admin/dgi/membership` handles add (body: `{ daoId, category }`)
- AC9: `PATCH /api/admin/dgi/membership` handles category update (body: `{ membershipId, category }`) and exclusion (body: `{ membershipId, exclude: true, reason? }`)
- AC10: API routes validate with Zod, require admin auth
- AC11: Navigation link "DGI Universe" added to admin sidebar in `admin-dashboard-client.tsx`
- AC12: Table shows total count: "X DAOs in DGI universe (Y qualifying)" — qualifying = confidence ≥ medium + fresh data
- AC13: Styled consistently with existing admin pages (navy background, card layout, shadcn/ui components)

---

### Story 1D: DGI Read Queries

**Files:**
- `src/lib/dgi/queries.ts` (NEW)

**Acceptance Criteria:**

- AC1: `getCurrentDGI(): Promise<DGICurrentValues | null>` — returns latest DGI snapshot values (composite + 4 categories + 4 components + daoCount + date), or null if no DGI data exists
- AC2: Query fetches from `dgiSnapshots` WHERE `snapshotDate = (SELECT MAX(snapshotDate))` — single query, returns up to 9 rows grouped into one object
- AC3: `getDGIHistory(days: number): Promise<DGIHistoryPoint[]>` — returns daily DGI composite values for last N days, ordered chronologically
- AC4: `getDGIPercentile(daoId: string): Promise<{ overall: number; category: number } | null>` — computes percentile by counting active index members with lower GVS score
- AC5: Percentile formula: `percentile = ROUND((count_with_lower_score / total_count) * 100)` — returns "Top X%" (e.g., Top 28% means 72% of DAOs score lower)
- AC6: Category percentile uses same formula but filtered by DAO's category from `daoIndexMembership`
- AC7: If DAO is not in index membership, returns null
- AC8: All functions handle empty/missing data gracefully (return null, not throw)
- AC9: TypeScript types exported: `DGICurrentValues`, `DGIHistoryPoint`
- AC10: TypeScript compiles with no new errors

---

### Story 1E: DGI Public API Endpoint

**Files:**
- `src/app/api/dgi/route.ts` (NEW)

**Acceptance Criteria:**

- AC1: `GET /api/dgi` returns current DGI values as JSON
- AC2: Response shape: `{ current: { composite, categories: { defi, infrastructure, publicGoods, social }, components: { hpr, dei, pdi, gpi }, daoCount, date } }`
- AC3: Optional query param `?history=true` includes `history: DGIHistoryPoint[]` (last 30 days)
- AC4: If no DGI data exists, returns `{ current: null }` with 200 status
- AC5: ISR cached with `export const revalidate = 3600` (1 hour)
- AC6: No auth required (public endpoint)
- AC7: Standard JSON response envelope, no sensitive data exposed
- AC8: TypeScript compiles with no new errors

---

## Epic 2: Matrix Details Integration

**Goal:** Show DGI benchmark context on existing Matrix Details Page.

**BLOCKER:** Epic 2 stories must NOT be deployed to production until:
1. ≥2 daily DGI values exist in `dgiSnapshots` (needed for trend/delta)
2. Methodology page at `/methodology/dgi` is published (AD-8)

---

### Story 2A: DGI Methodology Page

**Files:**
- `src/app/methodology/dgi/page.tsx` (NEW)

**Acceptance Criteria:**

- AC1: Static page at `/methodology/dgi` explaining DGI calculation
- AC2: Content covers: what DGI is, how it's calculated (equal-weighted average), index family (composite + category + component), inclusion criteria, update cadence (daily), what DGI is NOT
- AC3: References existing GVS methodology page for component score explanations
- AC4: Styled consistently with existing `/methodology` page (navy background, white text, clear headings)
- AC5: SEO metadata: title "DGI Methodology | DAO Governance Index | ChainSights", description explaining the benchmark
- AC6: No dynamic data — pure static content
- AC7: Link added to footer or methodology section where existing GVS methodology link exists
- AC8: **This story is a BLOCKER for Stories 2B-2E** — must be deployed before benchmark lines go live

---

### Story 2B: Benchmark Line on Charts

**Files:**
- `src/components/matrix-detail/MetricChart.tsx` (MODIFY)
- `src/app/matrix/[slug]/page.tsx` (MODIFY)

**Acceptance Criteria:**

- AC1: `MetricChart` prop `categoryAvg: number | null` replaced with `benchmark: { composite: number | null; category: number | null; label: string } | null`
- AC2: If `benchmark.composite` is not null, render a dashed `<ReferenceLine>` at that value, labeled "DGI: X.XX" (gray, position right)
- AC3: If `benchmark.category` is not null (free+ tier), render a second dashed `<ReferenceLine>` at that value, labeled with `benchmark.label` (e.g., "DGI DeFi: X.XX"), slightly different dash pattern or lighter color to distinguish from composite
- AC4: Server component `/matrix/[slug]/page.tsx` calls `getCurrentDGI()` to fetch benchmark data
- AC5: Server component filters benchmark data by access tier before passing to client:
  - `anonymous`: only `composite` value, `category` = null
  - `free`: `composite` + `category` for this DAO's category
  - `subscriber`/`admin`: `composite` + `category` + per-component benchmarks on respective charts
- AC6: For subscriber/admin: each component chart (HPR, DEI, PDI, GPI) receives its own component benchmark value from DGI (e.g., DGI-HPR on HPR chart)
- AC7: If no DGI data exists (`getCurrentDGI()` returns null), charts render without benchmark lines (graceful degradation)
- AC8: Existing ISR caching (`revalidate = 3600`) continues to work
- AC9: `matrix-detail-client.tsx` updated to pass new `benchmark` prop shape to `MetricChart`
- AC10: TypeScript compiles with no new errors

---

### Story 2C: Percentile Display in DAO Header

**Files:**
- `src/components/matrix-detail/DaoHeader.tsx` (MODIFY)
- `src/app/matrix/[slug]/page.tsx` (MODIFY)

**Acceptance Criteria:**

- AC1: `DaoHeader` receives new optional prop `percentile: { overall: number; category: number; categoryLabel: string } | null`
- AC2: If percentile is provided, display below score card: "Top X% of all tracked DAOs" and "Top X% of [Category] DAOs"
- AC3: Percentile text styled as small text below the delta indicator, with chart-bar icon (existing lucide-react)
- AC4: Server component calls `getDGIPercentile(daoId)` and passes result to `DaoHeader`
- AC5: Percentile only shown for `free` tier and above (per AD-4) — anonymous sees no percentile
- AC6: If DAO is not in DGI index, percentile prop is null and nothing is displayed (no "N/A" or empty space)
- AC7: TypeScript compiles with no new errors

---

### Story 2D: Rank Movement Indicator

**Files:**
- `src/lib/dgi/queries.ts` (MODIFY)
- `src/components/matrix-detail/DaoHeader.tsx` (MODIFY)
- `src/app/matrix/[slug]/page.tsx` (MODIFY)

**Acceptance Criteria:**

- AC1: New function `getDGIRankMovement(daoId: string): Promise<{ positions: number; direction: 'up' | 'down' | 'stable' } | null>` in `queries.ts`
- AC2: Function compares current rank position against position from 30 days ago (or earliest available if <30 days of data)
- AC3: Rank position = count of active index members with higher GVS score + 1
- AC4: `DaoHeader` receives new optional prop `rankMovement: { positions: number; direction: string } | null`
- AC5: If provided, display: "↗️ +3 positions" (green) or "↘️ -2 positions" (red) or "→ unchanged" (gray)
- AC6: Shown below percentile for `free` tier and above
- AC7: If DAO is not in DGI index or insufficient historical data, prop is null and nothing is displayed
- AC8: TypeScript compiles with no new errors

---

### Story 2E: Tiered Access Gating for DGI Data

**Files:**
- `src/app/matrix/[slug]/page.tsx` (MODIFY — consolidation story)

**Acceptance Criteria:**

- AC1: All DGI data fetching and filtering consolidated in the server component:
  - `anonymous`: benchmark composite only, no percentile, no rank movement
  - `free`: benchmark composite + category, percentile (overall + category), rank movement
  - `subscriber`/`admin`: all of the above + per-component benchmarks + DGI history trend
- AC2: No DGI-related data leaks to client beyond what the tier allows
- AC3: DGI data is fetched in parallel with existing queries (using `Promise.all`)
- AC4: If DGI data is unavailable, page renders exactly as before (no benchmark, no percentile) — zero regression
- AC5: TypeScript compiles with no new errors

**Note:** This story is primarily a verification/consolidation step. Most of the gating logic is implemented across Stories 2B-2D. This story ensures all pieces work together correctly and no data leaks exist.

---

## Epic 3: DGI Public Page

**Goal:** Standalone `/governance-index` dashboard for thought leadership and SEO.

**Priority:** Medium — implement after Epic 1 + 2 are stable.

---

### Story 3A: DGI Dashboard Page

**Files:**
- `src/app/governance-index/page.tsx` (NEW)
- `src/components/dgi/DGIDashboard.tsx` (NEW)

**Acceptance Criteria:**

- AC1: Public page at `/governance-index` showing DGI Composite trend chart (Recharts LineChart)
- AC2: Category indices comparison displayed (4 lines or bar chart showing DeFi vs Infra vs PG vs Social)
- AC3: "Top Movers" section: 3 DAOs with biggest GVS change in last 30 days (up and down)
- AC4: Current DGI values displayed as hero numbers: "DGI Composite: X.XX" with daoCount
- AC5: ISR cached with `revalidate = 3600`
- AC6: Responsive layout: single column mobile, two columns desktop
- AC7: SEO metadata: title "DAO Governance Index (DGI) | ChainSights", canonical URL
- AC8: Link to `/methodology/dgi` from the page
- AC9: Data fetched via `getCurrentDGI()` and `getDGIHistory(30)` from `src/lib/dgi/queries.ts`
- AC10: If no DGI data, show empty state: "DGI data is being calculated. Check back soon."

---

### Story 3B: Chart Granularity Toggle

**Files:**
- `src/components/dgi/DGIDashboard.tsx` (MODIFY)

**Acceptance Criteria:**

- AC1: Toggle control: Daily (default) / Weekly / Monthly
- AC2: Weekly: aggregates daily values into weekly averages (Mon-Sun)
- AC3: Monthly: aggregates into monthly averages
- AC4: Toggle state persisted via cookie (`dgi_chart_granularity`) for anonymous users, or user preference if authenticated
- AC5: Weekly and Monthly options only enabled when sufficient data exists (≥2 weeks / ≥2 months respectively)
- AC6: Default to daily; no toggle shown if <7 days of data exist

**Note:** This story can be deferred until 3-6 months of DGI data exist. Implement the daily-only view first (Story 3A) and add the toggle later.

---

## Epic 4: Report & Content Integration

**Goal:** DGI context in paid governance reports.

**Priority:** Medium — implement after Epic 1 is stable.

---

### Story 4A: Report Template Update

**Files:**
- Report generation logic (TBD — depends on current report template structure)

**Acceptance Criteria:**

- AC1: Every paid report (Deep Dive €49 / Governance Audit €149) includes a DGI benchmark section
- AC2: Section contains: "X DAO's GVS of Y.YY places it in the Zth percentile of the DAO Governance Index (DGI Composite: W.WW)."
- AC3: Category-specific context: "Within [Category] DAOs (DGI [Category]: V.VV), X DAO ranks in the Wth percentile."
- AC4: If DAO is not in DGI index, section is omitted (not shown as "N/A")
- AC5: DGI data fetched at report generation time using `getCurrentDGI()` and `getDGIPercentile()`

---

## Implementation Order

```
Epic 1: Foundation
  Story 1A (Schema) → Story 1B (Calculation) → Story 1C (Admin UI) → Story 1D (Queries) → Story 1E (API)

  [Mario curates 50 DAOs via Admin UI]
  [Daily cron runs ≥2 times → DGI data exists]

Epic 2: Matrix Integration (BLOCKED until methodology page + ≥2 DGI values)
  Story 2A (Methodology Page) → Story 2B (Benchmark Lines) → Story 2C (Percentile) → Story 2D (Rank Movement) → Story 2E (Tiered Access)

Epic 3: Public Page (after Epic 2 stable)
  Story 3A (Dashboard) → Story 3B (Granularity Toggle — deferred)

Epic 4: Reports (after Epic 1 stable, parallel with Epic 2)
  Story 4A (Report Template)
```

**Dependencies:**
- Story 1B depends on 1A (schema must exist)
- Story 1C depends on 1A (membership table must exist)
- Story 1D depends on 1A (tables must exist for queries)
- Story 1E depends on 1D (API uses query functions)
- Story 2A depends on nothing (static content, can start anytime)
- Stories 2B-2E depend on 1D (need query functions) + 2A (methodology blocker)
- Story 2E depends on 2B-2D (consolidation)
- Story 3A depends on 1D + 1E (needs DGI data + API)
- Story 4A depends on 1D (needs percentile queries)

---

*ChainSights — Wallets lie. We don't.*
