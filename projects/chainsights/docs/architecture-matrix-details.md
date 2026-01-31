# Architecture: Matrix Details Page

**Author:** Winston (Architect)
**Date:** 2026-01-31
**Status:** Architecture Complete — Ready for Epics & Stories
**Input:** docs/analysis/product-brief-matrix-details-page.md

---

## Architecture Decisions (5 Open Questions Resolved)

### AD-1: Chart Library → Recharts (Already Installed)

**Decision:** Use Recharts 2.15.4 (already in `package.json`).

**Rationale:**
- Zero new dependencies — Recharts is already installed and bundled
- React-native component model fits our Client Component pattern
- `LineChart` + `ResponsiveContainer` covers all 5 chart types
- Tooltip, reference lines (for category average), and color coding are built-in
- Tree-shakeable — only import what we use

**Components needed:**
- `LineChart`, `Line`, `XAxis`, `YAxis`, `Tooltip`, `ReferenceLine`, `ResponsiveContainer`

### AD-2: Category Averages → Calculated On-the-Fly in Server Component

**Decision:** Calculate category averages at request time in the Server Component, not pre-computed.

**Rationale:**
- With ~47 DAOs and ISR caching (1 hour), the computation is trivial
- The leaderboard query already fetches all DAOs — we just need `AVG(gvsScore) WHERE category = X`
- Pre-computing would add complexity to the daily snapshot job for negligible performance gain
- Category averages change daily anyway, so staleness is acceptable within ISR window

**Implementation:** New utility function `getCategoryAverages()` in `src/lib/db/queries/leaderboard.ts` that returns `Map<category, { avgGvs, avgHpr, avgDei, avgPdi, avgGpi }>`.

### AD-3: Static Generation → No (`generateStaticParams` Not Used)

**Decision:** Use dynamic Server Components with ISR (`revalidate = 3600`), same as Matrix page. No `generateStaticParams`.

**Rationale:**
- DAOs are added/removed dynamically (customer reports auto-populate `daos` table)
- `generateStaticParams` would require rebuilding on every DAO addition
- ISR gives us the same performance benefit (cached for 1 hour, revalidated on demand)
- Consistent with existing Matrix page pattern
- Slug validation happens at request time — unknown slugs return 404

### AD-4: Chart Data → Inline Server Component Data (No Separate API)

**Decision:** Fetch all chart data in the Server Component and pass as props. No dedicated chart data API endpoint.

**Rationale:**
- Server Components can fetch directly from DB — no API round-trip needed
- `getHistoricalSnapshots(daoId, 8)` already returns exactly what charts need
- Data is small (~56 points per metric × 5 metrics = ~280 data points)
- ISR caching applies to the entire page, including chart data
- A separate API would add complexity, CORS concerns, and a second cache layer

**Data flow:**
```
Server Component → getHistoricalSnapshots() → serialize → Client Component props → Recharts
```

### AD-5: Mobile Layout → Stacked Vertically (No Carousel)

**Decision:** Charts stack vertically on mobile. No swipeable carousel.

**Rationale:**
- Simpler implementation — CSS-only responsive layout
- Better accessibility — all visible charts are always in the DOM
- Blur overlay for locked charts works naturally in stacked layout
- Carousel would require a swipe library (new dependency) and complex touch handling
- Users can scroll — the pattern is familiar and predictable

---

## System Architecture

### Page Structure

```
/matrix/[slug]/page.tsx          (Server Component — data fetching + access control)
  └─ MatrixDetailClient.tsx       (Client Component — charts + interactivity)
       ├─ DaoHeader.tsx            (Client Component — DAO summary card)
       ├─ MetricChart.tsx          (Client Component — single Recharts chart, reusable)
       ├─ LockedChartOverlay.tsx   (Client Component — blur + lock + CTA)
       └─ GovernanceIndex.tsx      (Client Component — benchmark section)
```

### Routing

- **Path:** `/matrix/[slug]` (dynamic segment)
- **Slug source:** `daos.slug` column (e.g., `aave.eth`, `arbitrumfoundation.eth`)
- **404 handling:** If slug not found in `daos` table → Next.js `notFound()`
- **Back link:** `← Back to DAO Matrix` links to `/matrix?email={email}` (preserves access)

### Server Component (page.tsx)

```typescript
// src/app/matrix/[slug]/page.tsx

interface PageProps {
  params: Promise<{ slug: string }>
  searchParams: Promise<{ email?: string }>
}

export default async function MatrixDetailPage({ params, searchParams }: PageProps) {
  const { slug } = await params
  const { email } = await searchParams

  // 1. Lookup DAO by slug
  const dao = await getDaoBySlug(slug)       // New query function
  if (!dao) notFound()

  // 2. Access control (reuse existing)
  const access = await getMatrixAccess(email)

  // 3. Fetch chart data (8 weeks of daily snapshots)
  const snapshots = await getDetailSnapshots(dao.id, 8)  // New: returns all 5 metrics per day

  // 4. Fetch category averages
  const categoryAvg = await getCategoryAverage(dao.category)

  // 5. Fetch ranking position
  const rankInfo = await getDaoRank(dao.id)

  // 6. Determine which charts are visible
  const visibleCharts = getVisibleChartCount(access)  // anon=1, free=2, subscriber/admin=5

  // 7. Pass everything to client
  return <MatrixDetailClient
    dao={dao}
    snapshots={snapshots}
    categoryAvg={categoryAvg}
    rankInfo={rankInfo}
    access={access}
    visibleCharts={visibleCharts}
    email={email}
  />
}
```

### Tiered Chart Access

New function (not row-based like Matrix, chart-count-based):

```typescript
// In src/lib/db/subscriptions.ts (add to existing file)

export const CHART_LIMITS: Record<Exclude<MatrixAccess, 'admin'>, number> = {
  anonymous: 1,    // GVS only
  free: 2,         // GVS + HPR
  subscriber: 5,   // All charts
}

export function getVisibleChartCount(access: MatrixAccess): number {
  return CHART_LIMITS[access === 'admin' ? 'subscriber' : access]
}
```

**Chart display order** (fixed, not configurable):
1. GVS (always visible)
2. HPR (visible at free+)
3. DEI (visible at subscriber+)
4. PDI (visible at subscriber+)
5. GPI (visible at subscriber+)

### Data Queries (New Functions)

#### `getDaoBySlug(slug: string): Promise<DAO | null>`

Location: `src/lib/db/queries/leaderboard.ts`

```typescript
export async function getDaoBySlug(slug: string): Promise<DAO | null> {
  const db = getDb()
  const [dao] = await db
    .select()
    .from(daos)
    .where(and(eq(daos.slug, slug), eq(daos.isOptedOut, false)))
    .limit(1)
  return dao || null
}
```

#### `getDetailSnapshots(daoId: string, weeks: number): Promise<DetailSnapshot[]>`

Location: `src/lib/gvs/storage.ts`

Returns all 5 metrics per day (not just GVS like `getHistoricalSnapshots`):

```typescript
export interface DetailSnapshot {
  date: string          // ISO date string
  gvs: number
  hpr: number | null
  dei: number | null
  pdi: number | null
  gpi: number | null
  confidence: 'high' | 'medium' | 'low'
}

export async function getDetailSnapshots(
  daoId: string,
  weeks: number = 8
): Promise<DetailSnapshot[]> {
  const db = getDb()
  const startDate = new Date()
  startDate.setDate(startDate.getDate() - weeks * 7)

  const snapshots = await db
    .select({
      snapshotDate: gvsSnapshots.snapshotDate,
      gvsScore: gvsSnapshots.gvsScore,
      hprScore: gvsSnapshots.hprScore,
      deiScore: gvsSnapshots.deiScore,
      pdiScore: gvsSnapshots.pdiScore,
      gpiScore: gvsSnapshots.gpiScore,
      confidenceLevel: gvsSnapshots.confidenceLevel,
    })
    .from(gvsSnapshots)
    .where(
      and(
        eq(gvsSnapshots.daoId, daoId),
        gte(gvsSnapshots.snapshotDate, startDate)
      )
    )
    .orderBy(asc(gvsSnapshots.snapshotDate))

  return snapshots.map(s => ({
    date: s.snapshotDate.toISOString().split('T')[0],
    gvs: parseFloat(s.gvsScore),
    hpr: s.hprScore ? parseFloat(s.hprScore) : null,
    dei: s.deiScore ? parseFloat(s.deiScore) : null,
    pdi: s.pdiScore ? parseFloat(s.pdiScore) : null,
    gpi: s.gpiScore ? parseFloat(s.gpiScore) : null,
    confidence: s.confidenceLevel,
  }))
}
```

Uses existing compound index `gvs_snapshots_dao_date_idx` on `(dao_id, snapshot_date DESC)`.

#### `getCategoryAverage(category: string): Promise<CategoryAverage>`

Location: `src/lib/db/queries/leaderboard.ts`

```typescript
export interface CategoryAverage {
  category: string
  avgGvs: number
  avgHpr: number | null
  avgDei: number | null
  avgPdi: number | null
  avgGpi: number | null
  daoCount: number
}
```

Fetches latest snapshots for all DAOs in the same category, calculates averages. Reuses `getLeaderboardRankings({ category })` and aggregates in JS (simpler than a raw SQL AVG, and the data is already cached via ISR).

#### `getDaoRank(daoId: string): Promise<RankInfo>`

Location: `src/lib/db/queries/leaderboard.ts`

```typescript
export interface RankInfo {
  rank: number           // Position in full leaderboard
  totalDaos: number      // Total DAOs in leaderboard
  categoryRank: number   // Position within category
  categoryTotal: number  // Total DAOs in category
  strongestMetric: string  // 'HPR' | 'DEI' | 'PDI' | 'GPI'
  weakestMetric: string
}
```

Calculates rank by calling `getLeaderboardRankings()` (ISR cached) and finding position. Strongest/weakest metric is determined by comparing the DAO's component scores to category averages.

---

## Component Architecture

### MetricChart (Reusable)

Single chart component used 5 times with different data:

```typescript
interface MetricChartProps {
  title: string              // "Governance Vitality Score"
  abbreviation: string       // "GVS"
  data: { date: string; value: number | null }[]
  categoryAvg: number | null // Dotted reference line
  currentScore: number | null
  weight: string             // "35%" — shown as subtitle
  colorScheme: 'aqua' | 'green' | 'violet' | 'orange' | 'pink'
  isLocked: boolean          // Show blur overlay
}
```

**Chart features:**
- Recharts `LineChart` with `ResponsiveContainer` (100% width, 280px height)
- Single `Line` with smooth curve (`type="monotone"`)
- `ReferenceLine` for category average (dashed, gray)
- `Tooltip` showing date + score on hover
- Current score displayed as large number with color coding (green ≥7, yellow ≥4, red <4)
- Trend arrow (up/down/stable) comparing first vs last data point

### LockedChartOverlay

Renders over locked charts:

```typescript
interface LockedChartOverlayProps {
  type: 'email' | 'subscribe'  // Determines CTA text
  email?: string                // For redirect after email input
  slug: string                  // For redirect URL construction
}
```

- `backdrop-blur-md` + semi-transparent overlay
- Lock icon (lucide-react `Lock`)
- CTA text: "Enter email to unlock" (anon→free) or "Subscribe for full access" (free→subscriber)
- Email form does `window.location.href = /matrix/${slug}?email=...` (same pattern as Matrix)
- Subscribe button links to Matrix subscription checkout

### DaoHeader

Shows DAO summary above charts:

- DAO name + Snapshot space ID
- Category badge (colored pill: DeFi=blue, Infrastructure=purple, Public Goods=green)
- Current GVS score (large, color-coded) + grade letter (A-F)
- Week-over-week delta with arrow
- Confidence level indicator
- Last updated timestamp
- Community links (Discord, Forum, Twitter) — from `daos` table, only shown if not null

**Grade mapping:**
| Score Range | Grade | Color |
|---|---|---|
| 9.0-10.0 | A+ | Bright green |
| 8.0-8.9 | A | Green |
| 7.0-7.9 | B | Light green |
| 5.0-6.9 | C | Yellow |
| 3.0-4.9 | D | Orange |
| 0-2.9 | F | Red |

### GovernanceIndex (Benchmark Section)

Visible to all tiers (not paywalled) — serves as a hook:

- Category rank: "#3 of 12 DeFi DAOs"
- Bar chart comparing this DAO's GVS vs category average
- Strongest metric highlight (e.g., "Excels at: Delegate Engagement")
- Weakest metric highlight (e.g., "Needs work: Power Dynamics")
- CTA: "Get a Deep Dive Report for [DAO Name]" → checkout with prefilled DAO data

---

## SEO & Metadata

```typescript
export async function generateMetadata({ params }: PageProps): Promise<Metadata> {
  const { slug } = await params
  const dao = await getDaoBySlug(slug)
  if (!dao) return {}

  return {
    title: `${dao.name} Governance Score | DAO Matrix | ChainSights`,
    description: `${dao.name} governance health analysis: GVS score, participation rate, delegate engagement, power dynamics. Updated daily.`,
    alternates: {
      canonical: `https://chainsights.one/matrix/${slug}`,
    },
  }
}
```

---

## Navigation Changes

### Matrix Table → Detail Page Links

In `matrix-client.tsx`, DAO names become clickable links:

```tsx
<Link href={`/matrix/${dao.slug}${email ? `?email=${encodeURIComponent(email)}` : ''}`}>
  {dao.name}
</Link>
```

Email param is preserved to maintain access tier when navigating.

### Rankings → Detail Page Links

Same pattern in rankings page — DAO names link to `/matrix/[slug]`.

---

## Caching Strategy

| Layer | TTL | Scope |
|---|---|---|
| ISR (page) | 1 hour | Entire detail page per slug |
| DB Index | N/A | `gvs_snapshots_dao_date_idx` handles query performance |
| Browser | N/A | Standard Next.js cache headers |

No additional caching layer needed. The compound index on `(dao_id, snapshot_date DESC)` makes the historical query fast, and ISR prevents repeated DB hits.

---

## Database Changes

**None required.** All data already exists in `daos` and `gvsSnapshots` tables. No schema migration needed.

---

## File Structure (New Files)

```
src/app/matrix/[slug]/
  ├── page.tsx                    # Server Component (data fetching + access)
  └── matrix-detail-client.tsx    # Client Component (charts + interactivity)

src/components/matrix-detail/
  ├── DaoHeader.tsx               # DAO summary card
  ├── MetricChart.tsx             # Reusable chart component (Recharts)
  ├── LockedChartOverlay.tsx      # Blur + lock + CTA overlay
  └── GovernanceIndex.tsx         # Benchmark section
```

## Files to Modify

```
src/lib/db/subscriptions.ts       # Add CHART_LIMITS + getVisibleChartCount()
src/lib/db/queries/leaderboard.ts # Add getDaoBySlug(), getCategoryAverage(), getDaoRank()
src/lib/gvs/storage.ts            # Add getDetailSnapshots() + DetailSnapshot interface
src/app/matrix/matrix-client.tsx   # DAO names become links to /matrix/[slug]
```

---

## Risk Assessment

| Risk | Likelihood | Mitigation |
|---|---|---|
| Recharts bundle size bloat | Low | Tree-shake imports, dynamic import the client component |
| Slow historical query | Low | Existing compound index covers the query pattern |
| ISR cache miss on first visit | Medium | Acceptable — first visitor triggers build, subsequent visitors get cached |
| Blur bypass via DevTools | Low | Data is server-side gated — blur is UX only, actual data not sent for locked charts |

**Security note:** Locked chart data is NOT sent to the client. The Server Component only passes data for visible charts. Blur overlay is purely visual — there is no data behind it to inspect.

---

## Implementation Order (Suggested for Epics)

1. **Data layer** — `getDaoBySlug()`, `getDetailSnapshots()`, `getCategoryAverage()`, `getDaoRank()`, `getVisibleChartCount()`
2. **Server Component** — `/matrix/[slug]/page.tsx` with access control + data fetching
3. **MetricChart** — Reusable Recharts component with all features
4. **LockedChartOverlay** — Blur + CTA component
5. **DaoHeader** — DAO summary card
6. **GovernanceIndex** — Benchmark section
7. **Navigation** — Wire links from Matrix table + Rankings
8. **SEO** — `generateMetadata()` + canonical URLs
