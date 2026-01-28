# Product Requirement: ChainSights Monetization Expansion

**Project:** ChainSights (chainsights.one)
**Priority:** High
**Author:** Mario Semper
**Date:** 2026-01-28
**Version:** 1.0

---

## Executive Summary

Expand ChainSights from a single-tier report model to a comprehensive freemium platform with multiple revenue streams. This includes tiered reports (Free → €149), an interactive DAO Matrix for researchers/analysts, and foundation for future API monetization.

Current state: €0.50 total AI costs for 19 reports (~€0.03/report). Margin is effectively 100%, enabling aggressive pricing flexibility.

---

## Strategic Goals

1. **Increase conversion rate** by offering lower-friction entry points (Free Quick Check with email)
2. **Increase revenue per customer** with premium Governance Audit tier
3. **Create recurring revenue** via DAO Matrix subscription
4. **Build moat** through comprehensive, interactive governance data
5. **Maintain viral potential** by keeping all rankings publicly visible

---

## Pricing Structure

### Report Tiers

| Tier | Price | Target Audience | Margin |
|------|-------|-----------------|--------|
| **Quick Check** | Free (email required) | Curious visitors, lead gen | 100% (lead value) |
| **Deep Dive** | €49 | DAO contributors, small teams | ~99% |
| **Governance Audit** | €149 | DAO core teams, serious governance leads | ~99% |

### Subscription Products

| Product | Price | Target Audience |
|---------|-------|-----------------|
| **DAO Matrix Access** | €19/month or €99/year | Researchers, VCs, analysts |
| **API Access** (Future) | €49/month | Tooling platforms, aggregators |
| **Verified Badge** (Future) | €199/year | DAOs claiming their listing |

---

## Functional Requirements

### Part A: Tiered Report System

#### FR-A1: Quick Check (Free with Email)

**Purpose:** Lead generation entry point. Lower friction than paid reports.

**User Flow:**
1. User selects a DAO (from rankings or search)
2. User clicks "Quick Check" CTA
3. Modal/page asks for email address
4. Email validation + confirmation sent
5. User receives Quick Check results (on-page or via email)

**Quick Check Contents:**
- GVS Score (0-100) with label (Vital/Stable/Concerning/Critical)
- Component Breakdown: HPR, DEI, PDI, GPI (scores only, no analysis)
- Comparison to category average (e.g., "Above average for DeFi DAOs")
- CTA to upgrade: "Want detailed analysis? Get Deep Dive for €49"

**Requirements:**
- Email stored in leads table with `source: 'quick_check'`
- Marketing opt-in checkbox (unchecked by default, GDPR compliant)
- Rate limit: 1 free check per email per 30 days
- No PDF generation (on-screen only)

#### FR-A2: Deep Dive Report (€49)

**Purpose:** Current report tier, repositioned as mid-tier.

**Price Change:** €99 → €49

**Contents (unchanged from current):**
- Full GVS Score with component breakdown
- AI-generated analysis and insights
- Specific recommendations for improvement
- Benchmark against similar DAOs
- PDF download

**Requirements:**
- Update Stripe product/price
- Update all pricing displays on site
- Existing report generation pipeline unchanged

#### FR-A3: Governance Audit Report (€149)

**Purpose:** Premium tier for serious governance teams.

**Contents (everything in Deep Dive PLUS):**

1. **Historical Trend Analysis (12 weeks)**
   - GVS score trend chart
   - Component score trends (HPR, DEI, PDI, GPI over time)
   - Week-over-week change analysis
   - Trend direction indicators (improving/declining/stable)

2. **Peer Comparison Matrix**
   - Side-by-side comparison with 3-5 similar DAOs (same category)
   - Strengths/weaknesses relative to peers
   - "You rank #X in [category] for [metric]"

3. **Custom Strategic Recommendations**
   - More detailed, actionable recommendations (AI-generated with more context)
   - Priority ranking (high/medium/low impact)
   - Estimated effort indicators
   - Example implementations from other DAOs

4. **Executive Summary Page**
   - 1-page overview suitable for board/team presentations
   - Key metrics at a glance
   - Top 3 priorities

5. **Professional PDF Report**
   - Branded, designed PDF (not just text dump)
   - Charts and visualizations
   - Suitable for sharing with stakeholders

**Technical Requirements:**
- Extended AI prompt for deeper analysis (estimate: 3-5x current tokens)
- Historical data query (requires 12 weeks of stored snapshots)
- Peer selection algorithm (same category, similar size/activity)
- Enhanced PDF template with charts

#### FR-A4: Report Selection UI

**Landing Page / Rankings Page Updates:**

Replace single "Get Report" CTA with tiered options:

```
┌─────────────────────────────────────────────────────────┐
│  Get Governance Insights for [DAO Name]                 │
├─────────────────────────────────────────────────────────┤
│  ○ Quick Check          FREE                            │
│    Score + Component breakdown                          │
│    [Requires email]                                     │
│                                                         │
│  ○ Deep Dive            €49                             │
│    Full AI analysis + recommendations                   │
│    [Most Popular]                                       │
│                                                         │
│  ○ Governance Audit     €149                            │
│    Historical trends + peer comparison + strategic plan │
│    [Best Value]                                         │
│                                                         │
│                              [Continue →]               │
└─────────────────────────────────────────────────────────┘
```

**Requirements:**
- Radio button or card selection UI
- Clear feature differentiation
- Visual hierarchy (highlight Deep Dive as "Most Popular")
- Mobile responsive
- Accessible (keyboard navigation, screen reader friendly)

---

### Part B: DAO Matrix (Subscription Product)

#### FR-B1: Matrix Dashboard

**Purpose:** Interactive, sortable/filterable view of all DAO governance data for researchers and analysts.

**URL:** `/matrix` or `/dashboard/matrix`

**Features:**

1. **Data Table View**
   - All tracked DAOs (current: ~20-50, scaling to 100+)
   - Columns: DAO Name, Category, GVS, HPR, DEI, PDI, GPI, 7d Change, Trend
   - Sortable by any column (click header)
   - Resizable columns

2. **Filtering**
   - By category (DeFi, Infrastructure, Public Goods, etc.)
   - By score range (e.g., "GVS > 60")
   - By trend (Improving, Declining, Stable)
   - Multi-select filters combinable

3. **Search**
   - Real-time search by DAO name
   - Fuzzy matching

4. **Comparison Mode**
   - Select 2-5 DAOs via checkboxes
   - Click "Compare" → side-by-side view
   - Radar chart visualization
   - Exportable comparison

5. **Historical View**
   - Toggle: "Show historical data"
   - Dropdown: Select week (last 12 weeks available)
   - See how rankings/scores looked at any past point

6. **Export**
   - CSV export (all data or filtered selection)
   - JSON export (for technical users)
   - Export limited to subscribers

**Data Included:**

| Column | Description | Notes |
|--------|-------------|-------|
| DAO Name | Name + logo if available | Link to public profile |
| Category | DeFi / Infrastructure / Public Goods / etc. | Filterable |
| GVS | Governance Vitality Score (0-100) | Primary metric |
| HPR | Human Participation Rate | Component |
| DEI | Delegate Engagement Index | Component |
| PDI | Power Dynamics Index | Component |
| GPI | Grassroots Participation Index | Component |
| 7d Change | Week-over-week GVS change | +/- with color |
| Trend | 4-week trend direction | ↗️ ↘️ → |
| Last Updated | Timestamp of last calculation | |

#### FR-B2: Matrix Paywall

**Access Control:**

| User Type | Access Level |
|-----------|--------------|
| Anonymous | See matrix exists, blurred/limited preview (5 DAOs) |
| Free (email signup) | 10 DAOs visible, no export, no historical |
| Subscriber (€19/mo) | Full access, all features |

**Paywall UI:**
- Matrix loads with blur overlay after row 5/10
- Clear CTA: "Unlock full DAO Matrix – €19/month"
- Show what they're missing: "[X] more DAOs", "Export", "Historical data"

**Subscription Flow:**
1. User clicks subscribe
2. Redirect to Stripe Checkout (subscription mode)
3. On success → redirect back to Matrix with full access
4. Store subscription status in database
5. Check subscription on each Matrix page load

#### FR-B3: Subscription Management

**Requirements:**
- Stripe Customer Portal integration for self-service:
  - Cancel subscription
  - Update payment method
  - View invoices
- Grace period: 3 days after failed payment before access revoked
- Webhook handling for subscription events (created, updated, canceled, payment_failed)

---

### Part C: Database & Infrastructure

#### FR-C1: New Database Tables

```sql
-- Leads table for Quick Check signups
CREATE TABLE leads (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  email VARCHAR(255) NOT NULL,
  source VARCHAR(50) NOT NULL, -- 'quick_check', 'newsletter', etc.
  dao_checked VARCHAR(255), -- which DAO they checked (if applicable)
  marketing_optin BOOLEAN DEFAULT FALSE,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  UNIQUE(email)
);

-- Subscriptions table
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

-- Quick Check results cache (avoid recalculating)
CREATE TABLE quick_checks (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  email VARCHAR(255) NOT NULL,
  dao_slug VARCHAR(255) NOT NULL,
  gvs_score DECIMAL(5,2),
  hpr_score DECIMAL(5,2),
  dei_score DECIMAL(5,2),
  pdi_score DECIMAL(5,2),
  gpi_score DECIMAL(5,2),
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  UNIQUE(email, dao_slug)
);
```

#### FR-C2: Historical Data Storage

Ensure GVS snapshots are stored weekly for at least 12 weeks:
- Verify `gvs_snapshots` table has sufficient history
- Add cleanup job: Keep 12 weeks detailed, aggregate older data monthly
- Index for efficient historical queries

#### FR-C3: Stripe Products Setup

Create in Stripe Dashboard:

| Product | Price ID Pattern | Type |
|---------|------------------|------|
| Deep Dive Report | `price_deepdive_49` | One-time |
| Governance Audit | `price_audit_149` | One-time |
| DAO Matrix Monthly | `price_matrix_monthly_19` | Recurring |
| DAO Matrix Yearly | `price_matrix_yearly_99` | Recurring |

---

### Part D: Event Tracking Integration

#### FR-D1: New Events to Track

Add these events (using the event tracking system from separate requirement):

| Event | Trigger | Properties |
|-------|---------|------------|
| `tier_view` | Report tier selection shown | `dao_name` |
| `tier_select` | User selects a tier | `tier`, `dao_name` |
| `quick_check_start` | User enters email for quick check | `dao_name` |
| `quick_check_complete` | Quick check results shown | `dao_name`, `gvs_score` |
| `matrix_view` | Matrix page loaded | `is_subscriber` |
| `matrix_filter` | Filter applied | `filter_type`, `filter_value` |
| `matrix_compare` | Comparison started | `dao_count` |
| `matrix_export` | Export clicked | `format`, `row_count` |
| `subscription_start` | Checkout initiated | `product`, `price` |
| `subscription_complete` | Subscription confirmed | `product` |

---

## Non-Functional Requirements

### NFR1: Performance

- Matrix page must load < 2s with 100 DAOs
- Sorting/filtering must be < 100ms (client-side for loaded data)
- Historical queries optimized with proper indexing

### NFR2: Mobile Responsiveness

- Report tier selection: Fully mobile optimized
- Matrix: Responsive table with horizontal scroll on mobile
- Comparison view: Stacked cards on mobile instead of side-by-side

### NFR3: Security

- Subscription status verified server-side (never trust client)
- Rate limiting on Quick Check (prevent email harvesting)
- Email validation (prevent fake signups)

### NFR4: GDPR Compliance

- Marketing opt-in is explicit and unchecked by default
- Clear privacy policy link on email capture
- Ability to delete lead data on request

### NFR5: Subscription Billing

- Handle all Stripe webhook events properly
- Grace period for failed payments (3 days)
- Clear communication on subscription status changes
- Prorated upgrades if user switches plans

---

## Future Considerations (Not in Scope)

### New Metrics to Consider

**Governance Velocity Index (GVI):**
- Measures: Time from Proposal → Vote → Execution
- Shows: DAO agility vs bureaucracy
- Useful for: Comparing operational efficiency

**Voter Retention Rate (VRR):**
- Measures: % of voters participating in 3+ consecutive proposals
- Shows: Community stickiness and engagement consistency
- Useful for: Identifying healthy vs declining communities

*These can be added to Matrix and Reports in a future iteration.*

### API Access (Phase 2)

- REST API for programmatic access to GVS data
- API key authentication
- Rate limits by tier
- Pricing: €49/month
- Documentation with OpenAPI spec

### Verified Badge Program (Phase 2)

- DAOs can "claim" their listing
- Verification via on-chain signature or admin approval
- Badge displayed on rankings
- Ability to add context/response to their score
- Pricing: €199/year

---

## Success Criteria

### Week 2 (Post-Launch)

- [ ] All 3 report tiers available and purchasable
- [ ] At least 5 Quick Check signups (lead gen working)
- [ ] Matrix accessible to subscribers
- [ ] Event tracking capturing tier selection behavior

### Month 1

- [ ] 20+ Quick Check signups (leads)
- [ ] 3+ Deep Dive sales (€147 revenue)
- [ ] 1+ Governance Audit sale (€149 revenue)
- [ ] 2+ Matrix subscribers (€38+ MRR)
- [ ] Conversion funnel visible: Quick Check → Paid upgrade rate

### Month 3

- [ ] 100+ leads from Quick Check
- [ ] 10+ paid reports/month
- [ ] 10+ Matrix subscribers (~€190 MRR)
- [ ] Data on which tier converts best
- [ ] Iteration on pricing/packaging based on data

---

## Implementation Priority

**Phase 1 (Parallel):**
1. Report Tier System (FR-A1 through FR-A4)
2. DAO Matrix MVP (FR-B1 through FR-B3)

**Phase 2:**
3. Governance Audit PDF enhancements
4. Matrix historical view
5. Export functionality

**Phase 3 (Future):**
6. API Access
7. Verified Badge Program
8. New metrics (GVI, VRR)

---

## Open Questions for BMAD Team

1. **PDF Design:** Do we need a designer for Governance Audit PDF, or can we use enhanced markdown → PDF?

2. **Matrix Chart Library:** Preference for radar/comparison charts? (Recharts is already available)

3. **Peer Selection Algorithm:** Simple (same category, closest GVS) or more sophisticated?

4. **Email Delivery:** Use existing Resend setup for Quick Check delivery, or show results inline only?

5. **Subscription Billing Cycle:** Calendar month or rolling 30 days?

---

## Appendix: Wireframe Sketches

### Report Tier Selection (Mobile)

```
┌─────────────────────────┐
│ ← Back                  │
│                         │
│ Get Insights for        │
│ Gitcoin                 │
│                         │
│ ┌─────────────────────┐ │
│ │ ○ Quick Check       │ │
│ │   FREE              │ │
│ │   Score breakdown   │ │
│ │   [Email required]  │ │
│ └─────────────────────┘ │
│                         │
│ ┌─────────────────────┐ │
│ │ ● Deep Dive    ⭐   │ │
│ │   €49               │ │
│ │   Full AI analysis  │ │
│ │   [Most Popular]    │ │
│ └─────────────────────┘ │
│                         │
│ ┌─────────────────────┐ │
│ │ ○ Governance Audit  │ │
│ │   €149              │ │
│ │   Trends + Strategy │ │
│ │   [Best Value]      │ │
│ └─────────────────────┘ │
│                         │
│ ┌─────────────────────┐ │
│ │     Continue →      │ │
│ └─────────────────────┘ │
└─────────────────────────┘
```

### Matrix Paywall (Desktop)

```
┌────────────────────────────────────────────────────────────────────┐
│ DAO Governance Matrix                              [Search] [Filter]│
├────────────────────────────────────────────────────────────────────┤
│ DAO          │ Category │ GVS  │ HPR  │ DEI  │ PDI  │ GPI  │ 7d   │
├──────────────┼──────────┼──────┼──────┼──────┼──────┼──────┼──────┤
│ Gitcoin      │ PubGoods │ 74   │ 78   │ 72   │ 70   │ 76   │ +2.1 │
│ Arbitrum     │ Infra    │ 71   │ 65   │ 80   │ 68   │ 72   │ -0.5 │
│ ENS          │ Infra    │ 69   │ 70   │ 75   │ 62   │ 68   │ +1.2 │
│ Uniswap      │ DeFi     │ 67   │ 55   │ 78   │ 65   │ 70   │ +0.8 │
│ Lido         │ DeFi     │ 52   │ 45   │ 58   │ 50   │ 55   │ -1.3 │
├──────────────┴──────────┴──────┴──────┴──────┴──────┴──────┴──────┤
│ ░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░│
│ ░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░│
│ ░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░│
├──────────────────────────────────────────────────────────────────────┤
│                                                                      │
│   🔒 Unlock Full DAO Matrix                                         │
│                                                                      │
│   You're seeing 5 of 47 DAOs                                        │
│                                                                      │
│   Get full access to:                                                │
│   ✓ All 47 DAOs and growing                                         │
│   ✓ Historical data (12 weeks)                                      │
│   ✓ CSV/JSON export                                                 │
│   ✓ Comparison tools                                                │
│                                                                      │
│   ┌──────────────────┐  ┌──────────────────┐                        │
│   │ €19/month        │  │ €99/year (save 57%)│                       │
│   │ [Subscribe]      │  │ [Subscribe]      │                        │
│   └──────────────────┘  └──────────────────┘                        │
│                                                                      │
└──────────────────────────────────────────────────────────────────────┘
```

---

*Document ready for BMAD team review and implementation planning.*
