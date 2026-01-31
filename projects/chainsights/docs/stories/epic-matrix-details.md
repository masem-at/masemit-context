# Epic: Matrix Details Page

**Status:** Ready for Implementation
**Priority:** High
**Created:** 2026-01-31
**Product Brief:** docs/analysis/product-brief-matrix-details-page.md
**Architecture:** docs/architecture-matrix-details.md

## Problem Statement

The DAO Matrix shows only a flat row of scores per DAO. Users who want to understand WHY a DAO scores high or low have no way to drill down. The data already exists (8 weeks of daily GVS snapshots with all 5 component metrics), but there is no UI to explore it. This is a missed monetization opportunity: the detail page becomes the primary paywall surface for Matrix subscribers and a lead-gen funnel for email collection.

## Solution

A `/matrix/[slug]` detail page per DAO showing 5 governance metric charts (Recharts) with tiered access (anon=1, free=2, subscriber=5). Server-side data gating — locked chart data is never sent to the client. Reuses existing `getMatrixAccess()` for access control.

---

## Story A: Data Layer — Query Functions

**Files to modify:**
- `src/lib/db/queries/leaderboard.ts` — add `getDaoBySlug()`, `getCategoryAverage()`, `getDaoRank()`
- `src/lib/gvs/storage.ts` — add `getDetailSnapshots()` + `DetailSnapshot` interface
- `src/lib/db/subscriptions.ts` — add `CHART_LIMITS` + `getVisibleChartCount()`

### Acceptance Criteria

- [ ] AC1: `getDaoBySlug(slug)` returns DAO record or null, filters out opted-out DAOs
- [ ] AC2: `getDetailSnapshots(daoId, 8)` returns array of `DetailSnapshot` with all 5 metrics per day, ordered chronologically (oldest first), using existing compound index
- [ ] AC3: `DetailSnapshot` interface: `{ date: string, gvs: number, hpr: number|null, dei: number|null, pdi: number|null, gpi: number|null, confidence: 'high'|'medium'|'low' }`
- [ ] AC4: `getCategoryAverage(category)` returns `CategoryAverage` with avgGvs, avgHpr, avgDei, avgPdi, avgGpi, daoCount — calculated from latest snapshots of all DAOs in category
- [ ] AC5: `getDaoRank(daoId)` returns `RankInfo` with rank, totalDaos, categoryRank, categoryTotal, strongestMetric, weakestMetric
- [ ] AC6: `CHART_LIMITS` constant: anonymous=1, free=2, subscriber=5
- [ ] AC7: `getVisibleChartCount(access)` maps MatrixAccess to chart count (admin maps to subscriber)
- [ ] AC8: All functions have explicit TypeScript return types and JSDoc comments
- [ ] AC9: No new database schema or migration required

---

## Story B: Server Component — Page & Access Control

**Files to create:**
- `src/app/matrix/[slug]/page.tsx`

### Acceptance Criteria

- [ ] AC1: Dynamic route at `/matrix/[slug]` using Next.js App Router
- [ ] AC2: Server Component fetches DAO by slug via `getDaoBySlug()` — returns `notFound()` for unknown or opted-out DAOs
- [ ] AC3: Reads `?email=` from searchParams, calls `getMatrixAccess(email)` for tiered access
- [ ] AC4: Fetches chart data via `getDetailSnapshots(dao.id, 8)`
- [ ] AC5: Fetches category average via `getCategoryAverage(dao.category)`
- [ ] AC6: Fetches rank info via `getDaoRank(dao.id)`
- [ ] AC7: Calculates `visibleCharts` via `getVisibleChartCount(access)`
- [ ] AC8: **Security**: Only passes snapshot data for visible charts to client — locked chart data is NOT sent
- [ ] AC9: ISR caching: `export const revalidate = 3600` (1 hour, same as Matrix)
- [ ] AC10: `generateMetadata()` returns DAO-specific title, description, canonical URL

---

## Story C: MetricChart — Reusable Recharts Component

**Files to create:**
- `src/components/matrix-detail/MetricChart.tsx`

### Acceptance Criteria

- [ ] AC1: Client Component (`'use client'`) using Recharts `LineChart` + `ResponsiveContainer`
- [ ] AC2: Props: `title`, `abbreviation`, `data: {date, value}[]`, `categoryAvg`, `currentScore`, `weight`, `colorScheme`, `isLocked`
- [ ] AC3: `ResponsiveContainer` at 100% width, 280px height
- [ ] AC4: Single `Line` with `type="monotone"`, smooth curve, color based on `colorScheme`
- [ ] AC5: `ReferenceLine` for category average (dashed, gray, labeled "Category Avg")
- [ ] AC6: `Tooltip` shows date + score on hover
- [ ] AC7: Current score displayed as large number above chart with color coding (green ≥7, yellow ≥4, red <4)
- [ ] AC8: Trend arrow (up/down/stable) comparing first vs last data point in array
- [ ] AC9: Weight percentage shown as subtitle (e.g., "35% of GVS")
- [ ] AC10: When `isLocked=true`, chart renders a placeholder shape (no real data) with blur overlay on top
- [ ] AC11: XAxis shows dates (formatted as "Jan 15"), YAxis shows 0-10 scale
- [ ] AC12: Mobile responsive — chart fills container width

---

## Story D: LockedChartOverlay — Blur + CTA

**Files to create:**
- `src/components/matrix-detail/LockedChartOverlay.tsx`

### Acceptance Criteria

- [ ] AC1: Client Component with `backdrop-blur-md` + semi-transparent navy overlay
- [ ] AC2: Props: `type: 'email' | 'subscribe'`, `email?: string`, `slug: string`
- [ ] AC3: Lock icon from lucide-react centered in overlay
- [ ] AC4: When `type='email'`: shows "Enter your email to unlock more charts" + email input + submit button
- [ ] AC5: Email form submits via `window.location.href = /matrix/${slug}?email=...` (server redirect, same pattern as Matrix)
- [ ] AC6: When `type='subscribe'`: shows "Subscribe for full access" + button linking to Matrix subscription checkout
- [ ] AC7: Analytics: `trackEvent('cta_click', { button_name: 'detail_unlock_email' | 'detail_unlock_subscribe', location: 'matrix_detail' })`
- [ ] AC8: Positioned absolutely over the MetricChart container
- [ ] AC9: Mobile friendly — CTA text and input scale down properly

---

## Story E: DaoHeader — Summary Card

**Files to create:**
- `src/components/matrix-detail/DaoHeader.tsx`

### Acceptance Criteria

- [ ] AC1: Client Component showing DAO summary above charts
- [ ] AC2: DAO name (large heading) + Snapshot space ID (smaller, muted)
- [ ] AC3: Category badge as colored pill (DeFi=blue, Infrastructure=purple, Public Goods=green)
- [ ] AC4: Current GVS score displayed large with grade letter (A+ through F, per architecture grade mapping)
- [ ] AC5: Week-over-week delta with colored arrow (green up, red down, gray stable)
- [ ] AC6: Confidence level indicator (high=green dot, medium=yellow dot, low=red dot + text)
- [ ] AC7: "Last updated: [date]" timestamp
- [ ] AC8: Community links (Discord, Forum, Twitter) shown as icon links — only rendered if URL exists in DAO data
- [ ] AC9: "← Back to DAO Matrix" link at top, preserves email param in URL
- [ ] AC10: Mobile responsive — stacks vertically on small screens

---

## Story F: GovernanceIndex — Benchmark Section

**Files to create:**
- `src/components/matrix-detail/GovernanceIndex.tsx`

### Acceptance Criteria

- [ ] AC1: Client Component visible to ALL access tiers (not paywalled — serves as a hook)
- [ ] AC2: Shows category rank: "#3 of 12 DeFi DAOs"
- [ ] AC3: Visual comparison bar: this DAO's GVS vs category average GVS (e.g., color-coded horizontal bars)
- [ ] AC4: Strongest metric highlight with label (e.g., "Excels at: Delegate Engagement")
- [ ] AC5: Weakest metric highlight with label (e.g., "Needs work: Power Dynamics")
- [ ] AC6: Cross-sell CTA: "Get a Deep Dive Report for [DAO Name]" button
- [ ] AC7: CTA links to report checkout with prefilled DAO data (daoName, daoSlug, snapshotSpace)
- [ ] AC8: Analytics: `trackEvent('cta_click', { button_name: 'detail_cross_sell_report', location: 'matrix_detail', dao: dao.slug })`

---

## Story G: Client Layout — MatrixDetailClient Assembly

**Files to create:**
- `src/app/matrix/[slug]/matrix-detail-client.tsx`

### Acceptance Criteria

- [ ] AC1: Client Component that assembles DaoHeader + MetricCharts + LockedChartOverlays + GovernanceIndex
- [ ] AC2: Renders 5 chart slots in fixed order: GVS, HPR, DEI, PDI, GPI
- [ ] AC3: Charts 1 to `visibleCharts` render with real data; remaining charts render with `isLocked=true` + overlay
- [ ] AC4: Locked charts for anonymous users show `type='email'` overlay; for free users show `type='subscribe'` overlay
- [ ] AC5: Chart grid: 2 columns on desktop (lg+), 1 column stacked on mobile
- [ ] AC6: GovernanceIndex section placed below all charts
- [ ] AC7: Page background consistent with Matrix page design (navy gradient)
- [ ] AC8: Analytics: `trackEvent('page_view', { page: 'matrix_detail', dao: dao.slug, access: access })` on mount

---

## Story H: Navigation — Wire Links from Matrix & Rankings

**Files to modify:**
- `src/app/matrix/matrix-client.tsx` — DAO names become links to `/matrix/[slug]`
- Rankings page component — DAO names become links to `/matrix/[slug]`

### Acceptance Criteria

- [ ] AC1: In Matrix table, DAO name column becomes a clickable `<Link>` to `/matrix/[slug]`
- [ ] AC2: Link preserves email param: `/matrix/${slug}?email=${email}` if user has email access
- [ ] AC3: Link styling: text color unchanged, underline on hover, cursor pointer
- [ ] AC4: In Rankings page, DAO name/card links to `/matrix/[slug]`
- [ ] AC5: Rankings links also preserve email param if available
- [ ] AC6: Analytics: `trackEvent('cta_click', { button_name: 'matrix_dao_detail_click', dao: slug, location: 'matrix' | 'rankings' })`

---

## Implementation Order

1. **Story A** — Data layer (no dependencies, all other stories depend on this)
2. **Story B** — Server Component (depends on A)
3. **Story C** — MetricChart (no data dependency, can parallel with B)
4. **Story D** — LockedChartOverlay (no data dependency, can parallel with B/C)
5. **Story E** — DaoHeader (no data dependency, can parallel with B/C/D)
6. **Story F** — GovernanceIndex (no data dependency, can parallel)
7. **Story G** — Client layout assembly (depends on B, C, D, E, F)
8. **Story H** — Navigation wiring (depends on B working)

**Parallelizable:** Stories C, D, E, F can be built in parallel after A is complete.

## Out of Scope

- Interactive chart zooming/panning
- Custom date range selection
- PDF export of detail page
- Comparison mode (side-by-side two DAOs)
- Alert subscriptions ("notify me when GVS drops below X")
- Server-side rendering of chart images for social sharing
