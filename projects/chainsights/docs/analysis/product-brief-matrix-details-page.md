# Product Brief: Matrix Details Page

**Author:** Mario Semper (initial notes) + John (PM, expanded)
**Date:** 2026-01-31
**Status:** Product Brief Complete — Ready for Architecture
**Priority:** High

---

## Vision

A detail page per DAO inside the Matrix, showing 5 governance indicator charts with tiered access. Turns the Matrix from a flat comparison table into a deep-dive analytics tool. Creates a natural upgrade funnel: see one chart free → unlock more with email → see everything with subscription.

---

## Problem Statement

The DAO Matrix currently shows only a single row of scores per DAO. Users who want to understand WHY a DAO scores high or low have no way to drill down. The data already exists (GVS snapshots with all 4 component scores + historical trends), but there is no UI to explore it.

This is a missed monetization opportunity: the detail page becomes the primary paywall surface for Matrix subscribers, and a lead-gen funnel for email collection.

---

## Target Users

1. **DAO contributors/delegates** — Want to understand their DAO's governance health in detail
2. **Investors/researchers** — Comparing governance quality across DAOs before staking or delegating
3. **Content consumers** — Arrived via LinkedIn/Twitter posts about specific DAOs, want to see the full picture

---

## Feature Specification

### URL Structure

`/matrix/[slug]` — where slug is the DAO slug from the `daos` table (e.g., `/matrix/aave.eth`, `/matrix/arbitrumfoundation.eth`)

### The 5 Charts

Each chart visualizes one governance metric over time (8 weeks of daily snapshots):

| # | Chart | Metric | What It Shows |
|---|-------|--------|---------------|
| 1 | **Governance Vitality Score** | GVS (0-10) | Overall score trend line with grade badge (A-F) |
| 2 | **Human Participation Rate** | HPR (0-10, 35% weight) | Sybil-adjusted participation, clusters detected |
| 3 | **Delegate Engagement Index** | DEI (0-10, 25% weight) | Active vs inactive delegates, recency-weighted |
| 4 | **Power Dynamics Index** | PDI (0-10, 20% weight) | Gini coefficient trend, Nakamoto coefficient, leadership turnover |
| 5 | **Grassroots Participation** | GPI (0-10, 20% weight) | Small holder participation, delegation breadth, vote diversity |

### Chart Display

- **Line chart** with 8-week daily data points
- **Current score** prominently displayed with color coding (green ≥7, yellow ≥4, red <4)
- **Trend indicator** (improving/declining/stable arrow)
- **Category average** shown as dotted reference line for context
- **Confidence badge** if data completeness is low

### Tiered Access (Server-Side)

Uses existing `getMatrixAccess()` from server-side auth epic.

| Access Level | Charts Visible | Hidden Charts |
|-------------|---------------|---------------|
| Anonymous (no email) | 1 (GVS only) | 4 blurred with lock overlay |
| Free (email param) | 2 (GVS + HPR) | 3 blurred with lock overlay |
| Subscriber | All 5 | — |
| Admin | All 5 | — |

**Blurred charts:** Show the chart container with a blur + lock icon overlay and CTA text ("Subscribe to unlock" or "Enter email to see more"). The chart shape should be vaguely visible through the blur to create desire.

### DAO Header Section

Above the charts, show a DAO summary card:

- DAO name + Snapshot space ID
- Category badge (DeFi / Infrastructure / Public Goods)
- Current GVS score (large) with grade letter
- Week-over-week delta with arrow
- Confidence level indicator
- Last updated timestamp
- Links: Snapshot space, Discord, Forum, Twitter (from `daos` table)

### DAO Governance Index (Benchmark Concept)

Below the charts, show a "DAO Governance Index" section (S&P-style):

- **Category rank:** "#3 of 12 DeFi DAOs"
- **Category average GVS** vs this DAO's GVS
- **Strongest metric** (e.g., "Excels at: Delegate Engagement")
- **Weakest metric** (e.g., "Needs work: Power Dynamics")

This section is visible to all tiers (not paywalled) — it's a hook to make users want the full charts.

---

## Lead Generation Funnel

### Anonymous → Email

- **Trigger:** User clicks blurred chart or "Unlock more charts" CTA
- **Action:** Email input form (same pattern as Matrix paywall)
- **Result:** Redirect to `/matrix/[slug]?email={input}` — server-side re-render with 2 charts

### Email → Subscriber

- **Trigger:** User clicks blurred chart #3-5 or "Subscribe for full access" CTA
- **Action:** Stripe checkout flow (same as Matrix subscription)
- **Result:** After payment, redirect to `/matrix/[slug]?email={customer_email}` with all 5 charts

### Cross-Sell: Reports

- **Persistent CTA:** "Get a Deep Dive Report for [DAO Name]" button (visible on all tiers)
- **Pre-filled:** Links to checkout with DAO name, slug, snapshotSpace pre-populated
- **Only show for DAOs with `badgeType: 'featured_analysis'` or all DAOs** (TBD in architecture)

---

## Data Availability

All data already exists in the database:

- **Current scores:** `gvsSnapshots` table (latest per DAO, all 5 metrics)
- **Historical data:** 8 weeks of daily snapshots per DAO
- **Category averages:** Calculable from existing leaderboard query
- **DAO metadata:** `daos` table (name, slug, category, links)
- **Access control:** `getMatrixAccess()` + `getMatrixLimit()` already built

**No new data collection required.** This is a pure frontend + server component feature.

---

## Navigation

- **From Matrix table:** Each DAO name in the Matrix becomes a link to `/matrix/[slug]`
- **From Rankings:** Each DAO card links to `/matrix/[slug]`
- **Direct URL:** Shareable links (e.g., from social media posts about specific DAOs)
- **Back link:** "← Back to DAO Matrix" at top of detail page

---

## Success Metrics

| Metric | Target |
|--------|--------|
| Detail page visits per week | >50 (from Matrix click-through) |
| Email conversions on detail page | >5% of anonymous visitors |
| Subscription conversions | >2% of email-tier visitors |
| Report cross-sell clicks | >3% of detail page visitors |
| Avg time on detail page | >30 seconds |

---

## Technical Constraints

- Must be a **Server Component** (data fetching, access control)
- Chart rendering: Client Component (interactive, tooltips)
- Reuse existing `getMatrixAccess()` for tiered access
- ISR caching: 1-hour revalidate (same as Matrix)
- Chart library: TBD in Architecture phase (Recharts, Chart.js, or lightweight alternative)

---

## Open Questions for Architecture Phase

1. **Chart library choice** — Recharts (already in ecosystem?) vs lightweight option
2. **Category average calculation** — Pre-computed in snapshot job or calculated on-the-fly?
3. **Static generation** — Can we use `generateStaticParams()` for all known DAOs?
4. **Chart data API** — Dedicated endpoint or inline Server Component data?
5. **Mobile layout** — Charts stacked vertically, or swipeable carousel?

---

## Out of Scope (Future)

- Interactive chart zooming/panning
- Custom date range selection
- PDF export of detail page
- Comparison mode (side-by-side two DAOs)
- Alert subscriptions ("notify me when GVS drops below X")
