# Phase 2 Design: Governance Audit + DAO Matrix

**Status:** Ready for Implementation
**Created:** 2026-01-28
**Source Requirements:** `docs/_masemIT/chainsights-monetization-expansion.md`, `docs/_masemIT/bmad-team-response.md`

---

## Overview

Phase 2 expands ChainSights with two features:
1. **Governance Audit (€149)** - Premium report tier with historical trends and peer comparison
2. **DAO Matrix** - Interactive data table with subscription paywall

**Implementation Order:** Governance Audit first, then Matrix.

---

## Part 1: Governance Audit (€149)

### What It Adds (on top of Deep Dive)

| Feature | Description |
|---------|-------------|
| **Historical Trends (12 weeks)** | GVS + component trend charts, week-over-week analysis |
| **Peer Comparison Matrix** | Side-by-side with 3-5 similar DAOs in same category |
| **Strategic Recommendations** | Priority-ranked, effort indicators, examples from other DAOs |
| **Executive Summary Page** | 1-page overview for board/team presentations |
| **Professional PDF** | Branded, charts/visualizations, suitable for sharing |

### Architecture

#### Current State
- `tier` stored on `orders` table (`deep_dive`, `governance_audit`)
- `gvsSnapshots` has historical data with component scores (HPR, DEI, PDI, GPI)
- Analyze job treats all tiers the same - no differentiation
- PDF template is basic - no charts

#### File Changes Required

```
src/lib/gvs/
  └── historical.ts (NEW)
      - getHistoricalTrends(snapshotSpace: string, weeks: number): Promise<TrendData[]>
      - getPeerComparison(snapshotSpace: string, category: string, limit: number): Promise<PeerData[]>

src/lib/ai/
  └── analysis.ts
      - Add governance_audit prompt (3-5x tokens, includes trends + peers context)
      - Extend ReportDraft interface with historicalTrends, peerComparison, executiveSummary

src/app/api/jobs/analyze/route.ts
  └── Join with orders to get tier
  └── If tier === 'governance_audit':
      - Fetch historical trends (12 weeks)
      - Fetch peer comparison (same category, 5 DAOs)
      - Use extended AI prompt
      - Generate enhanced PDF

src/lib/pdf/
  └── report-template.tsx
      - Add executive summary page (page 1)
      - Add trend charts section
      - Add peer comparison table
  └── charts.tsx (NEW)
      - TrendChart component (GVS over time)
      - ComponentChart component (HPR/DEI/PDI/GPI breakdown)
      - PeerRadarChart component (radar comparison)
```

#### Data Flow

```
Order (tier=governance_audit)
    → Intake form submitted
    → fetch-snapshot job (unchanged)
    → analyze job:
        1. Get tier from order (JOIN orders ON reports.order_id)
        2. If tier === 'governance_audit':
           a. getHistoricalTrends(snapshotSpace, 12) → 12 weeks of GVS data
           b. getPeerComparison(snapshotSpace, category, 5) → 5 similar DAOs
        3. Call AI with extended prompt (trends + peers in context)
        4. Generate enhanced PDF with charts
    → Email delivery (unchanged)
```

#### Database Queries

**Historical Trends:**
```sql
SELECT snapshot_date, gvs_score, hpr_score, dei_score, pdi_score, gpi_score
FROM gvs_snapshots
WHERE dao_id = (SELECT id FROM daos WHERE snapshot_space = $1)
  AND snapshot_date >= NOW() - INTERVAL '12 weeks'
ORDER BY snapshot_date ASC;
```

**Peer Comparison:**
```sql
SELECT d.name, d.slug, d.snapshot_space, g.gvs_score, g.hpr_score, g.dei_score, g.pdi_score, g.gpi_score
FROM daos d
JOIN gvs_snapshots g ON g.dao_id = d.id
WHERE d.category = $1
  AND d.snapshot_space != $2
  AND g.snapshot_date = (SELECT MAX(snapshot_date) FROM gvs_snapshots WHERE dao_id = d.id)
ORDER BY ABS(g.gvs_score - $3) ASC
LIMIT 5;
```

#### AI Prompt Extension

For `governance_audit`, prepend to existing prompt:

```
## Historical Context (12 weeks)
{historicalTrends as formatted table}

## Peer Comparison (same category)
{peerData as formatted table}

Additional analysis required for Governance Audit tier:
1. Analyze trends - is governance improving or declining? What's driving the change?
2. Compare to peers - where does this DAO excel vs lag behind similar DAOs?
3. Provide strategic recommendations with:
   - Priority (High/Medium/Low)
   - Estimated effort (Quick Win / Medium / Long-term)
   - Examples from other DAOs that have done this successfully
4. Write an executive summary (3-4 sentences) suitable for board presentations
```

#### PDF Enhancements

**New sections for governance_audit:**

1. **Executive Summary (Page 1)**
   - 1-page overview
   - Key metrics at a glance (GVS, trend direction, peer rank)
   - Top 3 priorities

2. **Historical Trends (New section)**
   - Line chart: GVS over 12 weeks
   - Component breakdown chart (HPR, DEI, PDI, GPI trends)
   - Week-over-week change callouts

3. **Peer Comparison (New section)**
   - Table: DAO vs 5 peers with all metrics
   - Radar chart visualization
   - Strengths/weaknesses callout

**Chart Library:** Use `recharts` to generate SVG, then embed in `@react-pdf/renderer`.

---

## Part 2: DAO Matrix (€19/month or €99/year)

### Features

| Feature | Description |
|---------|-------------|
| **Data Table** | All DAOs, sortable by GVS/HPR/DEI/PDI/GPI/7d Change/Trend |
| **Filtering** | By category, score range, trend direction |
| **Search** | Real-time fuzzy search by DAO name |
| **Comparison Mode** | Select 2-5 DAOs → radar chart visualization |
| **Historical View** | Toggle to see past 12 weeks |
| **Export** | CSV/JSON (subscribers only) |

### Paywall Tiers

| User Type | Access |
|-----------|--------|
| Anonymous | 5 DAOs visible, rest blurred |
| Free (email signup) | 10 DAOs, no export, no historical |
| Subscriber (€19/mo) | Full access, all features |

### Architecture

#### New Database Table

```sql
CREATE TABLE subscriptions (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  email VARCHAR(255) NOT NULL,
  stripe_customer_id VARCHAR(255) NOT NULL,
  stripe_subscription_id VARCHAR(255) NOT NULL,
  product VARCHAR(50) NOT NULL, -- 'matrix', 'api' (future)
  status VARCHAR(50) NOT NULL, -- 'active', 'canceled', 'past_due'
  current_period_start TIMESTAMP WITH TIME ZONE,
  current_period_end TIMESTAMP WITH TIME ZONE,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

CREATE INDEX idx_subscriptions_email ON subscriptions(email);
CREATE INDEX idx_subscriptions_stripe_sub ON subscriptions(stripe_subscription_id);
```

#### File Structure

```
src/app/matrix/
  └── page.tsx - Matrix page (Server Component)
  └── matrix-table.tsx - Interactive table (Client Component)
  └── comparison-modal.tsx - Comparison view
  └── paywall-overlay.tsx - Subscription CTA

src/lib/db/
  └── schema.ts - Add subscriptions table
  └── subscriptions.ts (NEW) - Subscription queries

src/app/api/matrix/
  └── route.ts - Matrix data API (respects subscription status)
  └── export/route.ts - CSV/JSON export (subscribers only)

src/app/api/webhooks/stripe/route.ts
  └── Handle subscription events (customer.subscription.*)

src/lib/stripe.ts
  └── Add MATRIX_MONTHLY and MATRIX_YEARLY price IDs
```

#### Stripe Products

| Product | Price ID Env Var | Type |
|---------|------------------|------|
| DAO Matrix Monthly | `STRIPE_PRICE_MATRIX_MONTHLY` | Recurring (€19/mo) |
| DAO Matrix Yearly | `STRIPE_PRICE_MATRIX_YEARLY` | Recurring (€99/yr) |

#### Subscription Flow

```
User clicks Subscribe → Stripe Checkout (subscription mode)
    → On success: redirect to /matrix?session_id=xxx
    → Webhook: customer.subscription.created
        → Create subscription record in DB
    → Matrix page checks subscription status
    → Full access granted
```

---

## Implementation Order

### Governance Audit (Do First)

1. `src/lib/gvs/historical.ts` - Historical trends + peer comparison functions
2. `src/lib/ai/analysis.ts` - Extended prompt for governance_audit
3. `src/app/api/jobs/analyze/route.ts` - Tier-aware analysis
4. `src/lib/pdf/charts.tsx` - Chart components
5. `src/lib/pdf/report-template.tsx` - Enhanced template with charts

### DAO Matrix (Do Second)

1. Database: Add subscriptions table
2. `src/lib/stripe.ts` - Add Matrix price IDs
3. `src/app/api/webhooks/stripe/route.ts` - Subscription webhook handlers
4. `src/app/matrix/` - Matrix page + components
5. `src/app/api/matrix/` - Matrix API endpoints

---

## Mario's Decisions (from bmad-team-response.md)

| Topic | Decision |
|-------|----------|
| 12-week history | Launch with "Limited History" disclaimer if < 12 weeks available |
| Peer selection | Simple algorithm: same category, closest GVS score |
| Mobile Matrix | Card-based view, not horizontal scroll |
| Subscription billing | Rolling 30 days (Stripe default) |
| Quick Check rate limit leakage | Accept it, no fingerprinting |

---

## Open Questions (Resolved)

All questions from the original requirements have been answered in `bmad-team-response.md`.

---

## Success Criteria

### Governance Audit
- [ ] Historical trends display for DAOs with sufficient data
- [ ] Peer comparison shows 3-5 similar DAOs
- [ ] PDF includes executive summary, charts, and enhanced recommendations
- [ ] At least 1 Governance Audit sale in first month

### DAO Matrix
- [ ] Matrix page loads < 2s with 100 DAOs
- [ ] Paywall correctly limits anonymous/free users
- [ ] Subscription flow works end-to-end
- [ ] 2+ Matrix subscribers in first month

---

*This document is the single source of truth for Phase 2 implementation. All agents must read this before working on Phase 2 features.*
