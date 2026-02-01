# Conversion Funnel - ChainSights

**Last Updated:** 2025-12-23
**Owner:** Mario (Product Manager)
**Review Frequency:** Weekly (Fridays)

---

## Overview

This document defines the 5-stage conversion funnel for ChainSights DAO Governance Reports. The funnel tracks user journey from initial awareness to report purchase, enabling data-driven optimization of conversion rates.

**Conversion Goal:** Report orders (€49 Deep Dive or €149 Governance Audit)

---

## 5-Stage Conversion Funnel

### Stage 1: Awareness
**User Action:** Views `/rankings` leaderboard page
**Metric:** Page views on `/rankings`
**Data Source:** Vercel Analytics (automatic)
**Tracked Since:** Epic 0 (Project Setup)

**What it Means:**
User discovers ChainSights and views the public rankings leaderboard. This is the top of funnel - all potential customers start here.

**Where to Find:**
- Vercel Analytics Dashboard → Top Pages → `/rankings`
- Automatic tracking, no custom events needed

---

### Stage 2: Interest
**User Action:** Expands DAO row to see component breakdown
**Metric:** `row_expand` event count
**Data Source:** Custom event tracking
**Tracked Since:** Story 5.3

**Event Properties:**
```typescript
{
  dao: string,        // e.g., "arbitrum"
  rank: number        // e.g., 3
}
```

**What it Means:**
User is interested enough to explore details. They want to understand how the score is calculated and see the DAO's component breakdown (HPR, DEI, PDI, GPI).

**Where to Find:**
- Vercel Analytics Dashboard → Custom Events → `row_expand`
- Filter by DAO to see which DAOs generate most interest

---

### Stage 3: Consideration
**User Action:** Visits `/rankings/methodology` page
**Metric:** `methodology_view` event count
**Data Source:** Custom event tracking
**Tracked Since:** Story 5.3

**Event Properties:**
```typescript
{
  // No properties - simple page view event
}
```

**What it Means:**
User is seriously considering a report. They're checking the methodology to understand what they'll get and validate the scoring approach.

**Where to Find:**
- Vercel Analytics Dashboard → Custom Events → `methodology_view`
- Also visible in Top Pages as `/rankings/methodology`

---

### Stage 4: Action
**User Action:** Clicks "Order Report" CTA button
**Metric:** `cta_click` event count
**Data Source:** Custom event tracking
**Tracked Since:** Story 5.3

**Event Properties:**
```typescript
{
  dao: string,         // e.g., "optimism"
  score: number,       // e.g., 68
  location: string     // e.g., "leaderboard", "hero", "pricing", "modal"
}
```

**What it Means:**
User has decided to order a report and clicked the CTA. This is the last step before actual payment.

**Where to Find:**
- Vercel Analytics Dashboard → Custom Events → `cta_click`
- Filter by location to see which CTAs convert best

---

### Stage 5: Conversion
**User Action:** Completes Stripe payment for report
**Metric:** Successful Stripe checkout sessions
**Data Source:** Stripe Dashboard (manual tracking)
**Tracked Since:** Epic 0

**What it Means:**
User became a paying customer. This is the ultimate conversion - revenue generated.

**Where to Find:**
- Stripe Dashboard → Payments → Successful charges
- **Note:** Currently tracked manually. Future enhancement: Add Stripe webhook to automatically track `order_completed` event in Vercel Analytics.

---

## Calculating Conversion Rates

### Formulas

**Interest Rate:**
`(row_expand events / rankings page views) × 100`

**Example:** 50 row expansions / 500 rankings views = 10% interest rate

---

**Consideration Rate:**
`(methodology_view events / row_expand events) × 100`

**Example:** 20 methodology views / 50 row expansions = 40% consideration rate

---

**Action Rate:**
`(cta_click events / methodology_view events) × 100`

**Example:** 10 CTA clicks / 20 methodology views = 50% action rate

---

**Conversion Rate:**
`(Stripe orders / cta_click events) × 100`

**Example:** 3 orders / 10 CTA clicks = 30% conversion rate

---

**Overall Conversion:**
`(Stripe orders / rankings page views) × 100`

**Example:** 3 orders / 500 rankings views = 0.6% overall conversion rate

---

## Benchmark Targets

Based on industry standards for B2B SaaS and report sales:

| Stage | Target Rate | Status |
|-------|-------------|--------|
| Interest Rate | >15% | 🎯 Track |
| Consideration Rate | >40% | 🎯 Track |
| Action Rate | >30% | 🎯 Track |
| Conversion Rate | >25% | 🎯 Track |
| Overall Conversion | >1% | 🎯 Track |

**Note:** These are initial targets. Adjust based on actual data after 30 days of tracking.

---

## How to Calculate (Manual Process)

### Step 1: Open Vercel Analytics Dashboard
1. Login to Vercel: https://vercel.com
2. Navigate to `chainsights` project
3. Click "Analytics" tab

### Step 2: Set Date Range
1. Click date picker in top-right
2. Select desired range (e.g., "Last 30 days")
3. All metrics will update to match range

### Step 3: Get Rankings Page Views
1. Go to "Top Pages" section
2. Find `/rankings` in the list
3. Note the page view count (e.g., 500)

### Step 4: Get Custom Event Counts
1. Go to "Custom Events" section
2. Find each event:
   - `row_expand` (e.g., 50 events)
   - `methodology_view` (e.g., 20 events)
   - `cta_click` (e.g., 10 events)
3. Note the counts

### Step 5: Get Stripe Orders
1. Open Stripe Dashboard
2. Go to Payments → Successful
3. Filter by date range (match Vercel Analytics range)
4. Count successful orders (e.g., 3 orders)

### Step 6: Calculate Rates
Use the formulas above to calculate each conversion rate.

**Example Calculation:**
```
Rankings views: 500
Row expansions: 50
Methodology views: 20
CTA clicks: 10
Stripe orders: 3

Interest Rate = (50 / 500) × 100 = 10%
Consideration Rate = (20 / 50) × 100 = 40%
Action Rate = (10 / 20) × 100 = 50%
Conversion Rate = (3 / 10) × 100 = 30%
Overall Conversion = (3 / 500) × 100 = 0.6%
```

---

## Funnel Drop-Off Analysis

### Identify Bottlenecks

**If Interest Rate is Low (<10%):**
- Issue: Not enough users exploring DAO details
- Fix: Make rows more visually appealing, add preview tooltips, improve scores

**If Consideration Rate is Low (<30%):**
- Issue: Users expand rows but don't check methodology
- Fix: Add "How is this calculated?" link in expanded rows, inline methodology snippets

**If Action Rate is Low (<20%):**
- Issue: Users understand methodology but don't click CTA
- Fix: Improve CTA copy, add urgency ("Limited slots"), test pricing display

**If Conversion Rate is Low (<20%):**
- Issue: Users click CTA but don't complete payment
- Fix: Simplify checkout, add trust signals, test pricing, offer discounts

---

## Future Enhancements

### Automated Funnel Tracking (Post-MVP)

**Priority 1: Stripe Webhook Integration**
- Add webhook endpoint to automatically track `order_completed` event
- Eliminates manual Stripe dashboard checking
- Real-time conversion tracking

**Priority 2: Vercel Audience (Pro Tier)**
- Upgrade to Vercel Pro plan for Audience feature
- Automated funnel visualization
- Cohort analysis and retention tracking

**Priority 3: UTM Parameter Tracking**
- Add UTM parameters to marketing campaigns
- Track which sources drive best conversions
- Optimize marketing spend based on ROI

---

## Related Documents

- [Weekly Review Template](./weekly-review-template.md) - Use conversion rates in weekly reviews
- [Dashboard Guide](./dashboard-guide.md) - How to access and navigate Vercel Analytics
- [Analytics README](./README.md) - Complete analytics documentation

---

## Questions?

Contact: Mario (Product Manager)
Email: [Internal - see project-context.md]
