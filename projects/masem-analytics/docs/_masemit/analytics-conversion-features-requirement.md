# Product Requirement: Analytics Dashboard – Conversion Features

**Project:** masemIT Analytics (analytics.masem.at)
**Priority:** Medium
**Author:** Mario Semper
**Date:** 2026-01-28

---

## Executive Summary

Extend the masemIT Analytics dashboard with conversion-focused features. Build generic components that work across all projects (Funnel, CTA Performance, Time to Conversion), plus a custom tab system for project-specific analytics.

---

## Goals

1. **Identify conversion bottlenecks** across all masemIT projects
2. **Measure CTA effectiveness** to optimize placement and copy
3. **Enable project-specific insights** without building separate dashboards
4. **Maintain single codebase** with reusable components

---

## Architecture

```
analytics.masem.at/
├── /dashboard                        ← Overview (all projects)
├── /dashboard/[project]              ← Project view
│   ├── ?tab=overview                 ← Current: Page views, visitors, etc.
│   ├── ?tab=funnel                   ← NEW: Funnel visualization
│   ├── ?tab=cta                      ← NEW: CTA performance
│   ├── ?tab=conversions              ← NEW: Time to conversion
│   └── ?tab=custom                   ← NEW: Project-specific widgets
```

---

## Phase 0: Prerequisites

### FR-0: Stripe Webhook → Analytics Integration

**Purpose:** Enable full funnel tracking including payment completion events.

**Problem:** `checkout_complete` events are currently only processed by Stripe webhooks in each project. They do NOT reach masemIT Analytics, breaking the funnel at the final step.

**Solution:** Extend existing Stripe webhook handlers to forward completion events to Analytics.

**Implementation Required In:**

| Project | Webhook Location | Status |
|---------|------------------|--------|
| ChainSights | `/api/webhooks/stripe/route.ts` | To Do |
| tellingCube | `/api/webhooks/stripe/route.ts` | To Do |
| hoki.help | N/A (`donate_complete` already client-side) | ✅ Done |

**Event Format to Send:**

```typescript
// After successful Stripe webhook processing, add:
await fetch('https://analytics.masem.at/api/events', {
  method: 'POST',
  headers: { 'Content-Type': 'text/plain' },
  body: JSON.stringify({
    type: 'checkout_complete',
    project_id: 'chainsights', // or 'telling-cube'
    timestamp: new Date().toISOString(),
    path: '/checkout/success',
    event_data: {
      product: session.metadata?.product || 'unknown',
      price_eur: (session.amount_total || 0) / 100,
      dao_name: session.metadata?.dao_name,        // ChainSights only
      stripe_session_id: session.id,
      customer_email: session.customer_email,      // Optional, for debugging
    }
  })
});
```

**Acceptance Criteria:**
- [ ] ChainSights: `checkout_complete` events appear in Analytics dashboard
- [ ] tellingCube: `checkout_complete` events appear in Analytics dashboard
- [ ] Events include `price_eur` for revenue tracking

**Note:** This is a **prerequisite** for FR-1 (Funnel) to show the complete conversion path.

---

## Phase 1: Generic Features (All Projects)

### FR-1: Funnel Visualization

**Purpose:** Show user journey drop-off at each step.

**UI:** Horizontal funnel chart with percentages and absolute numbers.

```
┌─────────────────────────────────────────────────────────────────────────┐
│ Conversion Funnel (Last 7 days)                                         │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  Landing     →    Page 2      →    CTA Click   →    Checkout   → Paid  │
│  ████████████     ████████         ████              ██           █     │
│  450 (100%)       210 (47%)        45 (10%)          12 (2.7%)    3     │
│                                                                  (0.7%) │
│                   -53%             -79%              -73%        -75%   │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

**Data Requirements:**
- Funnel steps defined per project (config)
- Count unique sessions per step
- Calculate drop-off percentages

**Configuration per Project:**

```typescript
// Example: ChainSights funnel config
const CHAINSIGHTS_FUNNEL = [
  { step: 'landing', event: 'page_view', filter: { path: '/' } },
  { step: 'rankings', event: 'page_view', filter: { path: '/rankings' } },
  { step: 'tier_modal', event: 'tier_view' },
  { step: 'checkout', event: 'checkout_start' },
  { step: 'paid', event: 'checkout_complete' },
]

// Example: hoki.help funnel config
const HOKI_FUNNEL = [
  { step: 'landing', event: 'page_view', filter: { path: '/' } },
  { step: 'donate_click', event: 'donate_cta_click' },
  { step: 'amount_select', event: 'amount_select' },
  { step: 'donated', event: 'donate_complete' },
]
```

---

### FR-2: CTA Performance Table

**Purpose:** Compare effectiveness of different CTAs.

**UI:** Sortable table with key metrics.

```
┌─────────────────────────────────────────────────────────────────────────┐
│ CTA Performance (Last 7 days)                              [Sort: CTR ▼]│
├─────────────────────────────────────────────────────────────────────────┤
│ CTA Name                    │ Views │ Clicks │ CTR    │ Conversions    │
├─────────────────────────────┼───────┼────────┼────────┼────────────────┤
│ Quiz "Let us help"          │ 30    │ 12     │ 40.0%  │ 3 (25%)        │
│ Hero "Get Free Check"       │ 120   │ 15     │ 12.5%  │ 8 (53%)        │
│ Rankings Row "Get Report"   │ 85    │ 8      │ 9.4%   │ 2 (25%)        │
│ Footer "Contact"            │ 200   │ 4      │ 2.0%   │ 1 (25%)        │
└─────────────────────────────────────────────────────────────────────────┘
```

**Data Requirements:**
- Track `cta_view` events (via Intersection Observer)
- Track `cta_click` events
- Join with conversion events to calculate CTA → Conversion rate

**Columns:**
| Column | Description |
|--------|-------------|
| CTA Name | From `button_name` property |
| Views | Count of `cta_view` events |
| Clicks | Count of `cta_click` events |
| CTR | Clicks / Views × 100 |
| Conversions | Users who clicked AND later converted |

---

### FR-3: Time to Conversion

**Purpose:** Understand how long users take to convert.

**UI:** Stats cards + histogram.

```
┌─────────────────────────────────────────────────────────────────────────┐
│ Time to Conversion (Last 30 days)                                       │
├───────────────────┬───────────────────┬─────────────────────────────────┤
│ Avg. Time to      │ Avg. Time to      │ Returning Visitor              │
│ First Action      │ Paid Conversion   │ Conversion Rate                │
│                   │                   │                                 │
│ 2.3 minutes       │ 1.4 days          │ 4.2%                           │
│ (first visit →    │ (first visit →    │ (vs 0.8% new visitors)         │
│ CTA click)        │ purchase)         │                                 │
└───────────────────┴───────────────────┴─────────────────────────────────┘
│                                                                         │
│ Distribution: Time from First Visit to Purchase                         │
│ ██████████████████████████ Same session (45%)                          │
│ ████████████ 1-24 hours (22%)                                          │
│ ██████ 1-3 days (15%)                                                  │
│ ███ 3-7 days (10%)                                                     │
│ ██ 7+ days (8%)                                                        │
└─────────────────────────────────────────────────────────────────────────┘
```

**Data Requirements:**
- Session tracking with timestamps
- First visit timestamp per user/session
- Conversion event timestamp
- Calculate deltas

**Metrics:**
| Metric | Calculation |
|--------|-------------|
| Avg. Time to First Action | Mean of (first CTA click timestamp - session start) |
| Avg. Time to Paid | Mean of (purchase timestamp - first visit timestamp) |
| Returning Visitor Rate | Conversions from returning / Total returning visitors |
| New Visitor Rate | Conversions from new / Total new visitors |

---

### FR-4: Returning vs. New Visitors

**Purpose:** Understand if returning visitors convert better.

**UI:** Comparison cards.

```
┌─────────────────────────────────────────────────────────────────────────┐
│ Visitor Segments (Last 7 days)                                          │
├─────────────────────────────────┬───────────────────────────────────────┤
│ NEW VISITORS                    │ RETURNING VISITORS                    │
│                                 │                                       │
│ 340 visitors                    │ 45 visitors                           │
│ 2 conversions                   │ 3 conversions                         │
│ 0.6% conversion rate            │ 6.7% conversion rate                  │
│                                 │ ████████████ 11x better               │
└─────────────────────────────────┴───────────────────────────────────────┘
```

**Data Requirements:**
- Track visitor return (session_id seen before? or localStorage flag)
- Segment conversions by new vs. returning

---

## Phase 2: Project-Specific Custom Tabs

### FR-5: Custom Tab Framework

**Purpose:** Allow project-specific widgets without code duplication.

**Implementation:**

```typescript
// src/config/project-custom-tabs.ts

export const PROJECT_CUSTOM_WIDGETS = {
  chainsights: [
    {
      id: 'tier_breakdown',
      title: 'Tier Selection Breakdown',
      component: 'TierBreakdownWidget',
    },
    {
      id: 'dao_ranking',
      title: 'DAO Interest Ranking',
      component: 'DaoInterestWidget',
    },
    {
      id: 'quiz_performance',
      title: 'Quiz Performance',
      component: 'QuizPerformanceWidget',
    },
  ],
  'hoki-help': [
    {
      id: 'donation_amounts',
      title: 'Donation Amount Breakdown',
      component: 'DonationAmountsWidget',
    },
  ],
  tellingcube: [
    {
      id: 'template_popularity',
      title: 'Template Popularity',
      component: 'TemplatePopularityWidget',
    },
    {
      id: 'download_formats',
      title: 'Download Formats',
      component: 'DownloadFormatsWidget',
    },
  ],
  'masemit-site': [], // No custom widgets needed
}
```

---

### FR-6: ChainSights Custom Widgets

#### 6a: Tier Selection Breakdown

```
┌─────────────────────────────────────────────────────────────────────────┐
│ Tier Selection Breakdown (Last 7 days)                                  │
├─────────────────────────────────────────────────────────────────────────┤
│ Tier               │ Selected │ Completed │ Conversion │ Revenue       │
├────────────────────┼──────────┼───────────┼────────────┼───────────────┤
│ Free Check         │ 45       │ 38        │ 84%        │ €0 (leads)    │
│ Deep Dive (€49)    │ 12       │ 2         │ 17%        │ €98           │
│ Gov. Audit (€149)  │ 3        │ 1         │ 33%        │ €149          │
├────────────────────┼──────────┼───────────┼────────────┼───────────────┤
│ TOTAL              │ 60       │ 41        │ 68%        │ €247          │
└─────────────────────────────────────────────────────────────────────────┘
```

#### 6b: DAO Interest Ranking

```
┌─────────────────────────────────────────────────────────────────────────┐
│ DAO Interest Ranking (Last 30 days)                                     │
├─────────────────────────────────────────────────────────────────────────┤
│ DAO          │ Quick Checks │ Deep Dives │ Audits │ Revenue │ Trend    │
├──────────────┼──────────────┼────────────┼────────┼─────────┼──────────┤
│ Optimism     │ 18           │ 3          │ 1      │ €296    │ ↗ +45%   │
│ ENS          │ 12           │ 2          │ 0      │ €98     │ → 0%     │
│ Arbitrum     │ 10           │ 1          │ 1      │ €198    │ ↗ +20%   │
│ Uniswap      │ 8            │ 1          │ 0      │ €49     │ ↘ -15%   │
│ Gitcoin      │ 6            │ 0          │ 0      │ €0      │ → 0%     │
└─────────────────────────────────────────────────────────────────────────┘
```

#### 6c: Quiz Performance

```
┌─────────────────────────────────────────────────────────────────────────┐
│ Quiz Performance (Last 30 days)                                         │
├─────────────────────────────────────────────────────────────────────────┤
│ Quiz Outcome          │ Count │ → Free Check │ → Paid │ Conversion     │
├───────────────────────┼───────┼──────────────┼────────┼────────────────┤
│ "Needs improvement"   │ 24    │ 12 (50%)     │ 4      │ 17%            │
│ "Moderate health"     │ 18    │ 6 (33%)      │ 1      │ 6%             │
│ "Strong governance"   │ 8     │ 2 (25%)      │ 0      │ 0%             │
├───────────────────────┼───────┼──────────────┼────────┼────────────────┤
│ INSIGHT: "Needs improvement" outcome converts 3x better than others    │
└─────────────────────────────────────────────────────────────────────────┘
```

---

### FR-7: hoki.help Custom Widget

#### Donation Amount Breakdown

```
┌─────────────────────────────────────────────────────────────────────────┐
│ Donation Breakdown (Last 30 days)                                       │
├─────────────────────────────────────────────────────────────────────────┤
│ Amount        │ Count │ % of Donations │ Total Revenue │ Avg/Donation  │
├───────────────┼───────┼────────────────┼───────────────┼───────────────┤
│ €5            │ 12    │ 40%            │ €60           │               │
│ €10           │ 8     │ 27%            │ €80           │               │
│ €25           │ 6     │ 20%            │ €150          │               │
│ €50+          │ 4     │ 13%            │ €280          │               │
├───────────────┼───────┼────────────────┼───────────────┼───────────────┤
│ TOTAL         │ 30    │ 100%           │ €570          │ €19           │
└─────────────────────────────────────────────────────────────────────────┘
```

---

### FR-8: tellingCube Custom Widgets

#### Template Popularity

```
┌─────────────────────────────────────────────────────────────────────────┐
│ Template Popularity (Last 30 days)                                      │
├─────────────────────────────────────────────────────────────────────────┤
│ Template              │ Views │ Generations │ Downloads │ Conv. Rate   │
├───────────────────────┼───────┼─────────────┼───────────┼──────────────┤
│ Alpine Bakery         │ 85    │ 23          │ 18        │ 21%          │
│ TechParts Mfg         │ 62    │ 15          │ 12        │ 19%          │
│ EuroCredit Union      │ 45    │ 8           │ 5         │ 11%          │
│ MedTech Diagnostics   │ 38    │ 12          │ 10        │ 26%          │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## Database Requirements

### New Tables/Columns

```sql
-- Funnel configuration (or keep in code as config)
-- No new tables needed if config is in TypeScript

-- For Time to Conversion, we need first_seen tracking
ALTER TABLE sessions ADD COLUMN first_seen_at TIMESTAMP WITH TIME ZONE;
ALTER TABLE sessions ADD COLUMN is_returning BOOLEAN DEFAULT FALSE;
```

### Required Event Properties

Ensure these properties are tracked consistently:

| Event | Required Properties |
|-------|---------------------|
| `cta_view` | `button_name`, `page_path` |
| `cta_click` | `button_name`, `page_path` |
| `tier_view` | `dao_name` |
| `tier_select` | `tier`, `dao_name` |
| `quiz_complete` | `score`, `outcome` |
| `checkout_start` | `product`, `price_eur`, `dao_name` |
| `checkout_complete` | `product`, `price_eur`, `dao_name` |

---

## Implementation Priority

```
Phase 1 (Generic - All Projects):
├── FR-1: Funnel Visualization        ← Highest priority
├── FR-2: CTA Performance Table       ← High priority
├── FR-3: Time to Conversion          ← Medium priority
└── FR-4: Returning vs. New           ← Medium priority

Phase 2 (Project-Specific):
├── FR-5: Custom Tab Framework        ← Build framework first
├── FR-6: ChainSights Widgets         ← Immediate need
├── FR-7: hoki.help Widget            ← When donations increase
└── FR-8: tellingCube Widgets         ← When traffic increases
```

---

## Success Criteria

### Phase 1
- [ ] Funnel view shows drop-off for each project
- [ ] CTA table sortable by CTR and conversions
- [ ] Time to Conversion stats display correctly
- [ ] Returning vs. New breakdown visible

### Phase 2
- [ ] Custom tab loads project-specific widgets
- [ ] ChainSights: Tier breakdown shows accurate revenue
- [ ] ChainSights: DAO ranking updates in real-time
- [ ] ChainSights: Quiz performance shows conversion correlation

---

## Notes

- All components should be responsive (mobile-friendly)
- Date range picker should apply to all widgets consistently
- Export functionality (CSV) nice-to-have for custom tabs
- Real-time updates not required – 5-minute refresh is fine
