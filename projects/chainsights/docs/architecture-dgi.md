# Architecture: DGI (DAO Governance Index)

**Status:** Approved
**Date:** 2026-02-01
**Author:** Winston (Architect Agent)
**Requirement:** `docs/_masemIT/requirements/dao-governance-index-requirement-v1.1.md`
**Context:** `docs/project_context.md`

---

## 1. Overview

The DGI adds an ecosystem-wide benchmark layer on top of the existing GVS (Governance Vitality Score) infrastructure. GVS measures individual DAO governance health (0–10 scale). DGI aggregates GVS scores across all qualifying DAOs to produce composite, category, and component benchmark indices.

**Key constraint:** DGI reuses existing infrastructure (Drizzle ORM, Neon PostgreSQL, Vercel Cron, Recharts, ISR). No new external services, no new AI pipeline, no new data sources. The only new data is the aggregation of existing GVS scores.

---

## 2. Architectural Decisions

### AD-1: Schema Strategy — Extend Existing, Don't Duplicate

**Decision:** Add two new tables (`dgiSnapshots`, `daoIndexMembership`) to the existing `src/lib/db/schema.ts`. Do NOT create a separate schema file.

**Rationale:** All other tables (daos, gvsSnapshots, users, etc.) live in one schema file. Splitting would break the established pattern and complicate migrations.

**Note on `daoCategoryEnum`:** The existing enum has 3 values: `defi`, `infrastructure`, `public_goods`. The requirement adds a 4th category: `social`. This requires a Drizzle enum migration (`ALTER TYPE dao_category ADD VALUE 'social'`). The `social` value must be added to `daoCategoryEnum` in schema.ts. This also affects `DaoHeader.tsx` (CATEGORY_STYLES map) and `leaderboard.ts` (CATEGORY_LABELS, LeaderboardOptions type).

### AD-2: DGI Calculation — Post-Hook on Daily GVS Cron

**Decision:** DGI calculation runs as a final step inside the existing `executeDailyRecalculation()` function in `src/lib/jobs/daily-recalculation.ts`, after all individual GVS scores have been computed and saved.

**Rationale:** DGI depends on fresh GVS scores. Running it as a separate cron introduces timing risks (DGI could run before GVS completes). Running it as a post-hook guarantees fresh data.

**Implementation:**
1. After the existing GVS loop completes, call `calculateAndSaveDGI()` (new function)
2. This function queries all qualifying DAOs from `daoIndexMembership` (where `excludedAt IS NULL`)
3. Fetches their latest GVS snapshots
4. Computes DGI Composite, 4 category indices, 4 component indices (9 values total)
5. Inserts 9 rows into `dgiSnapshots` (one per `indexType`)
6. Non-blocking: DGI calculation failure does not fail the GVS job

### AD-3: DGI Universe Management — Admin CRUD at `/admin/dgi`

**Decision:** New admin page at `src/app/admin/dgi/page.tsx` with a client component for managing the DGI universe.

**Rationale:** Mario is the sole curator (D7). The admin dashboard already exists at `/admin` with auth, navigation, and consistent styling. Adding a sub-page follows the established pattern (`/admin/featured`, `/admin/jobs`, etc.).

**UI spec:**
- Table of all DAOs from `daoIndexMembership` joined with `daos`
- Columns: Name, Snapshot Space, Category (dropdown: DeFi/Infra/PG/Social), Included (toggle), Included Since, Notes (grace_period_until as date picker)
- Add button: dropdown of tracked DAOs not yet in index membership
- Remove button: sets `excludedAt = NOW()`, preserves history
- No bulk actions needed (50 DAOs, manual curation)

### AD-4: Benchmark Data Delivery — Server-Side Gating

**Decision:** DGI benchmark data follows the same server-side gating pattern as existing chart data. The server component (`/matrix/[slug]/page.tsx`) fetches DGI data and filters it by access tier before passing to the client.

**Tiered access:**

| Access Level | DGI Data Sent to Client |
|---|---|
| `anonymous` | DGI Composite value only (1 number for ReferenceLine) |
| `free` | + Category benchmark + percentile rank |
| `subscriber` / `admin` | + All 4 component benchmarks + historical DGI trend |

**Rationale:** Matches existing `getVisibleChartCount()` / `filteredSnapshots` pattern. Premium data never reaches the client for unauthorized users (security principle already established).

### AD-5: Benchmark Display — ReferenceLine on Existing Charts

**Decision:** Use Recharts `<ReferenceLine>` component (already imported in `MetricChart.tsx`) to display DGI benchmark lines. No new chart components needed.

**Current state:** `MetricChart.tsx` already has a `categoryAvg` ReferenceLine (line 144-153). The DGI benchmark replaces this with ecosystem-wide DGI values instead of per-category averages from `getCategoryAverage()`.

**Implementation:**
- Rename `categoryAvg` prop → `benchmark` (object: `{ composite: number; category: number | null; label: string }`)
- Anonymous: show only composite line, label "DGI: XX"
- Free+: show composite + category line, label "DGI DeFi: XX"
- Paid: show all component benchmarks on their respective charts

### AD-6: Percentile Calculation — Computed at Query Time

**Decision:** Percentile rank is NOT stored in the database. It's computed at query time when loading a DAO's detail page.

**Rationale:** Percentile changes whenever any DAO's score changes. Storing it would require updating all DAOs' percentiles on every GVS recalculation. Computing it on-demand is cheap (one query: `SELECT COUNT(*) WHERE gvsScore <= :thisScore` over ~50 rows).

**Implementation:**
- New function `getDGIPercentile(daoId: string): Promise<{ overall: number; category: number }>` in `src/lib/dgi/queries.ts`
- Returns: "Top X% of all DAOs" and "Top X% of [Category] DAOs"
- Formula: `percentile = ROUND((count_below / total_count) * 100)`
- Displayed in `DaoHeader.tsx` for `free` and higher access tiers

### AD-7: DGI API Endpoint — Read-Only, ISR-Cached

**Decision:** `GET /api/dgi` returns current DGI values + configurable history. ISR cached with `revalidate = 3600` (1 hour, matching Matrix pages).

**Response shape:**
```typescript
interface DGIResponse {
  current: {
    composite: number
    categories: { defi: number; infrastructure: number; publicGoods: number; social: number }
    components: { hpr: number; dei: number; pdi: number; gpi: number }
    daoCount: number
    date: string
  }
  history?: DGIHistoryPoint[]  // Only if ?history=true
}
```

**Auth:** Public endpoint (no auth required). History depth limited to 30 days for anonymous, unlimited for authenticated.

### AD-8: Methodology Page — Blocker for Epic 2 Go-Live

**Decision:** Static MDX page at `/methodology/dgi` (alongside existing `/methodology` GVS page). Must be published BEFORE DGI benchmark lines become visible on Matrix Details pages.

**Rationale:** Paige's rule: "kann nachher kommen == nie". The methodology page is a hard dependency for Epic 2 — DGI values can be calculated in the background (Epic 1) but must not be user-facing without explanation.

**Implementation:** Markdown content rendered via Next.js MDX or simple static page. No dynamic data needed — references the calculation formulas from the requirement doc.

---

## 3. Database Schema

### 3.1 New Tables

```typescript
// In src/lib/db/schema.ts

// DGI index type enum
export const dgiIndexTypeEnum = pgEnum('dgi_index_type', [
  'composite',
  'defi',
  'infrastructure',
  'public_goods',
  'social',
  'hpr',
  'dei',
  'pdi',
  'gpi',
])

// DGI Snapshots — daily index values
export const dgiSnapshots = pgTable('dgi_snapshots', {
  id: uuid('id').defaultRandom().primaryKey(),
  snapshotDate: timestamp('snapshot_date').notNull(),
  indexType: dgiIndexTypeEnum('index_type').notNull(),
  value: numeric('value', { precision: 5, scale: 2 }).notNull(),
  daoCount: integer('dao_count').notNull(),
  createdAt: timestamp('created_at').defaultNow().notNull(),
}, (table) => ({
  dateTypeIdx: uniqueIndex('dgi_snapshots_date_type_idx').on(table.snapshotDate, table.indexType),
  typeIdx: index('dgi_snapshots_type_idx').on(table.indexType),
}))

// DAO Index Membership — tracks which DAOs are in the DGI universe
export const daoIndexMembership = pgTable('dao_index_membership', {
  id: uuid('id').defaultRandom().primaryKey(),
  daoId: uuid('dao_id').references(() => daos.id, { onDelete: 'cascade' }).notNull(),
  category: daoCategoryEnum('category').notNull(),
  includedSince: timestamp('included_since').defaultNow().notNull(),
  excludedAt: timestamp('excluded_at'),
  exclusionReason: text('exclusion_reason'),
  gracePeriodUntil: timestamp('grace_period_until'), // Manual note field (D7)
  createdAt: timestamp('created_at').defaultNow().notNull(),
}, (table) => ({
  daoIdIdx: index('dao_index_membership_dao_id_idx').on(table.daoId),
  activeIdx: index('dao_index_membership_active_idx').on(table.daoId, table.excludedAt),
}))
```

### 3.2 Enum Extension

```typescript
// MODIFY existing enum — add 'social' category
export const daoCategoryEnum = pgEnum('dao_category', [
  'defi',
  'infrastructure',
  'public_goods',
  'social',  // NEW — DGI requirement D4
])
```

**Migration note:** Adding a value to a pgEnum requires `ALTER TYPE dao_category ADD VALUE 'social'`. Drizzle handles this via `db:generate`. Non-breaking — existing rows remain valid.

---

## 4. Module Architecture

### 4.1 New Files

| File | Purpose |
|---|---|
| `src/lib/dgi/calculate.ts` | `calculateAndSaveDGI()` — core DGI calculation engine |
| `src/lib/dgi/queries.ts` | `getCurrentDGI()`, `getDGIHistory()`, `getDGIPercentile()` — read queries |
| `src/app/api/dgi/route.ts` | `GET /api/dgi` — public API endpoint |
| `src/app/admin/dgi/page.tsx` | Admin DGI Universe Management (server component) |
| `src/components/admin/dgi-management-client.tsx` | Admin DGI client component (table, dropdowns, toggles) |
| `src/app/methodology/dgi/page.tsx` | Public DGI methodology page (static) |

### 4.2 Modified Files

| File | Change |
|---|---|
| `src/lib/db/schema.ts` | Add `dgiSnapshots`, `daoIndexMembership` tables, `dgiIndexTypeEnum`, extend `daoCategoryEnum` with `'social'` |
| `src/lib/jobs/daily-recalculation.ts` | Add `calculateAndSaveDGI()` call after GVS loop |
| `src/app/matrix/[slug]/page.tsx` | Fetch DGI benchmark data, pass to client (tiered) |
| `src/components/matrix-detail/MetricChart.tsx` | Replace `categoryAvg` prop with `benchmark` object, add DGI ReferenceLine |
| `src/components/matrix-detail/DaoHeader.tsx` | Add percentile display, add `'social'` to CATEGORY_STYLES |
| `src/lib/db/queries/leaderboard.ts` | Add `'social'` to CATEGORY_LABELS, LeaderboardOptions type |
| `src/components/admin/admin-dashboard-client.tsx` | Add DGI nav link in admin sidebar |
| `vercel.json` | No change needed — DGI runs inside existing daily cron |

---

## 5. Data Flow

### 5.1 Daily Calculation Flow

```
Vercel Cron (0 0 * * *)
  → GET /api/jobs/daily-recalculation
    → executeDailyRecalculation()
      → For each non-opted-out DAO:
          calculateGVS(spaceId) → saveGVSSnapshot(daoId, result)
      → [NEW] calculateAndSaveDGI()
          → Query daoIndexMembership WHERE excludedAt IS NULL
          → Fetch latest gvsSnapshots for each member DAO
          → Compute 9 index values (composite + 4 category + 4 component)
          → INSERT 9 rows into dgiSnapshots
          → Log results
```

### 5.2 Detail Page Request Flow

```
User visits /matrix/arbitrum
  → Server Component: page.tsx
    → getDaoBySlug('arbitrum')
    → getMatrixAccess(email)               // existing
    → getDetailSnapshots(daoId)            // existing
    → [NEW] getCurrentDGI()                // fetch latest DGI values
    → [NEW] getDGIPercentile(daoId)        // compute percentile (free+ only)
    → Filter DGI data by access tier (AD-4)
    → Pass to MatrixDetailClient
      → MetricChart receives benchmark prop
      → DaoHeader receives percentile prop
```

### 5.3 Admin Management Flow

```
Mario visits /admin/dgi
  → Server: fetch daoIndexMembership JOIN daos
  → Client: render table with edit controls
  → On change:
    POST /api/admin/dgi/membership  →  INSERT/UPDATE daoIndexMembership
```

---

## 6. Calculation Logic

### 6.1 `calculateAndSaveDGI()`

```typescript
// Pseudocode for src/lib/dgi/calculate.ts

async function calculateAndSaveDGI(): Promise<void> {
  const db = getDb()

  // 1. Get all active members
  const members = await db.select()
    .from(daoIndexMembership)
    .where(isNull(daoIndexMembership.excludedAt))
    .innerJoin(daos, eq(daoIndexMembership.daoId, daos.id))

  // 2. Fetch latest GVS snapshots for all members
  const daoIds = members.map(m => m.dao_index_membership.daoId)
  const latestSnapshots = await getLatestSnapshots(daoIds)

  // 3. Filter: only include DAOs with confidence >= medium and fresh data (14 days)
  const qualifying = members.filter(m => {
    const snapshot = latestSnapshots.get(m.dao_index_membership.daoId)
    if (!snapshot) return false
    if (snapshot.confidenceLevel === 'low') return false
    const age = Date.now() - snapshot.snapshotDate.getTime()
    return age < 14 * 24 * 60 * 60 * 1000
  })

  // 4. Compute indices
  const today = new Date()
  const indices: { indexType: string; value: number; daoCount: number }[] = []

  // Composite
  const allScores = qualifying.map(m => parseFloat(latestSnapshots.get(m.dao_index_membership.daoId)!.gvsScore))
  indices.push({ indexType: 'composite', value: avg(allScores), daoCount: allScores.length })

  // Category indices
  for (const cat of ['defi', 'infrastructure', 'public_goods', 'social']) {
    const catMembers = qualifying.filter(m => m.dao_index_membership.category === cat)
    const catScores = catMembers.map(m => parseFloat(latestSnapshots.get(m.dao_index_membership.daoId)!.gvsScore))
    if (catScores.length > 0) {
      indices.push({ indexType: cat, value: avg(catScores), daoCount: catScores.length })
    }
  }

  // Component indices (HPR, DEI, PDI, GPI)
  for (const component of ['hpr', 'dei', 'pdi', 'gpi']) {
    const scores = qualifying
      .map(m => latestSnapshots.get(m.dao_index_membership.daoId)!)
      .map(s => s[`${component}Score`])
      .filter(Boolean)
      .map(s => parseFloat(s as string))
    if (scores.length > 0) {
      indices.push({ indexType: component, value: avg(scores), daoCount: scores.length })
    }
  }

  // 5. Batch insert
  await db.insert(dgiSnapshots).values(
    indices.map(i => ({
      snapshotDate: today,
      indexType: i.indexType,
      value: i.value.toFixed(2),
      daoCount: i.daoCount,
    }))
  )
}

function avg(nums: number[]): number {
  return nums.reduce((a, b) => a + b, 0) / nums.length
}
```

### 6.2 Percentile Calculation

```typescript
// src/lib/dgi/queries.ts

async function getDGIPercentile(daoId: string): Promise<{ overall: number; category: number } | null> {
  // Get this DAO's latest GVS score
  // Count how many active index members have a lower score
  // percentile = (count_below / total) * 100
  // Repeat for category-specific percentile
}
```

---

## 7. Scaling Considerations

| Concern | Current State | DGI Impact | Mitigation |
|---|---|---|---|
| Daily cron duration | ~8s for 10 DAOs | +50 DAOs GVS + DGI calc | Batch processing already exists. 50 DAOs ≈ 40s. DGI calc adds <1s. Well within 5min Vercel Pro limit. |
| `dgi_snapshots` table size | N/A (new) | 9 rows/day = ~3,285 rows/year | Trivial. No cleanup needed for years. |
| Detail page query count | 3 queries | +2 queries (DGI + percentile) | All simple index lookups. Adds <10ms total. ISR caches result for 1 hour. |
| Admin UI | 7 existing pages | +1 page | Same pattern, no new infra. |

---

## 8. Risk Register

| Risk | Likelihood | Impact | Mitigation |
|---|---|---|---|
| `social` enum migration breaks existing DAOs | Low | Medium | `ADD VALUE` is additive, never removes existing values. Test with `db:generate` before deploy. |
| DGI calculation fails, blocks GVS job | Low | High | DGI runs in try/catch after GVS loop — failure is logged but non-blocking (AD-2). |
| Category assignment ambiguity (Arbitrum = Infra or DeFi?) | Medium | Low | Manual curation by Mario (D7). Single primary category per DAO (D4). |
| Methodology page forgotten before Epic 2 | Medium | High | Hard blocker documented (AD-8). Epic 2 stories reference this dependency. |

---

## 9. Epic-to-Architecture Mapping

| Epic | Story | Primary AD | Key Files |
|---|---|---|---|
| 1 | 1.1 Universe Setup | AD-1, AD-3 | `schema.ts`, admin DGI page |
| 1 | 1.2 Batch GVS | — | Existing pipeline, no changes |
| 1 | 1.3 DGI Calculation | AD-2 | `src/lib/dgi/calculate.ts`, `daily-recalculation.ts` |
| 1 | 1.4 Admin UI | AD-3 | `src/app/admin/dgi/`, `src/components/admin/dgi-management-client.tsx` |
| 1 | 1.5 DGI API | AD-7 | `src/app/api/dgi/route.ts` |
| 2 | 2.1 Benchmark Line | AD-5 | `MetricChart.tsx` |
| 2 | 2.2 Percentile | AD-6 | `src/lib/dgi/queries.ts`, `DaoHeader.tsx` |
| 2 | 2.3 Rank Movement | AD-6 | `DaoHeader.tsx` |
| 2 | 2.4 Tiered Access | AD-4 | `/matrix/[slug]/page.tsx` |
| — | Methodology | AD-8 | `/methodology/dgi/page.tsx` |

---

## 10. Dependencies & Sequencing

```
Schema Migration (AD-1)
  ↓
Admin UI (AD-3) ──────────────┐
  ↓                           │
Universe Setup (manual by Mario) │
  ↓                           │
Batch GVS (existing pipeline) │
  ↓                           │
DGI Calculation Engine (AD-2) │
  ↓                           │
DGI API (AD-7) ───────────────┤
  ↓                           │
[≥2 DGI daily values exist]   │
  ↓                           │
Methodology Page (AD-8) ◄─────┘  ← BLOCKER for go-live
  ↓
Matrix Integration (AD-4, AD-5, AD-6)
```

---

*ChainSights — Wallets lie. We don't.*
