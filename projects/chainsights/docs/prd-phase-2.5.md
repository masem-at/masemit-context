---
type: 'prd-lite'
phase: '2.5'
status: 'approved'
author: 'Mario'
date: '2026-02-04'
inputTickets:
  - 'docs/_masemIT/requirements/ticket-dgi-benchmark-in-reports.md'
  - 'docs/_masemIT/requirements/ticket-pricing-page.md'
  - 'docs/_masemIT/requirements/ticket-account-dashboard.md'
priority_order: [1, 2, 3]
---

# PRD Phase 2.5: Report Enhancement + Pricing + Account

> **⚠️ PRICING OUTDATED (as of 2026-02-05)**
> This document contains stale prices. Current pricing:
> - Free Check: €0 | Deep Dive: **€49** | Governance Audit: **€149**
> - DAO Matrix: **FREE** (paywall removed 2026-02-05)
> - ❌ €19/mo Matrix and €99/yr Matrix subscriptions are deprecated
> See `docs/project_context.md` for authoritative pricing.

**Author:** Mario
**Date:** 2026-02-04
**Status:** Approved for Implementation

---

## Executive Summary

Phase 2.5 delivers three incremental enhancements that improve the post-purchase experience and conversion funnel for ChainSights. These features build on the successful DGI launch and address gaps identified during initial market validation.

**Core Theme:** Make reports more valuable, pricing more transparent, and subscription management self-service.

---

## Strategic Context

### Where We Are (Post-Phase 2)

- ✅ DGI (Decentralized Governance Index) launched and calculating daily
- ✅ Governance Audit (€149) with historical trends and peer comparison
- ✅ Deep Dive (€49) with AI analysis
- ✅ Free Check with Open Universe (13,000+ DAOs)
- ✅ DAO Matrix subscription (€19/month)
- ✅ Magic Link Authentication deployed

### What's Missing

1. **Reports lack benchmark context** — Customers get a score but can't easily see "how do I compare?"
2. **No dedicated pricing page** — Transparency and SEO gap
3. **No self-service account management** — Subscribers can't manage billing without contacting support

---

## Feature Overview

| # | Feature | Priority | Effort | Business Impact |
|---|---------|----------|--------|-----------------|
| 1 | DGI Benchmark in Reports | **High** | Small-Medium | Differentiator vs. free tools, increases report value |
| 2 | Pricing Page | Medium | Small | Trust, SEO, conversion support |
| 3 | Account Dashboard | Medium | Medium | Subscriber retention, reduced support |

---

## Feature 1: DGI Benchmark Integration in Reports

**Ticket:** `docs/_masemIT/requirements/ticket-dgi-benchmark-in-reports.md`

### Problem Statement

A report showing "Your GVS is 7.2" without context leaves the customer asking "Is that good?" The data exists to answer this question — we calculate daily DGI ecosystem averages for all dimensions — but it's not surfaced in reports.

### Solution

Add dimension-by-dimension benchmark comparison to paid reports:

- **Deep Dive (€49):** Headline benchmark — "GVS 7.2 vs. DGI Composite 5.4 (Top 25%)"
- **Governance Audit (€149):** Full "How You Compare" section with strengths/weaknesses per dimension

### Key Deliverable

```
┌──────────────────────────────────────────────────┐
│  HOW YOU COMPARE                                 │
├──────────────────────────────────────────────────┤
│  Overall Score     7.2   vs  DGI Composite  5.4  │
│                    ████████████░░░░  +33%        │
├──────────────────────────────────────────────────┤
│  STRENGTHS (above benchmark)                     │
│  • Sybil Resistance   10.0  vs  5.8  🏆 Top 5%   │
│  • Participation       8.2  vs  6.1  ✅ +34%     │
├──────────────────────────────────────────────────┤
│  IMPROVEMENT AREAS (below benchmark)             │
│  • Power Distribution  4.0  vs  4.8  ⚠️ -17%    │
│  • Process Quality     5.5  vs  5.9  📊 -7%     │
└──────────────────────────────────────────────────┘
```

### Data Availability

All required data already exists in `dgiSnapshots` table:
- `composite` — Ecosystem GVS average
- `defi`, `infrastructure`, `public_goods`, `social` — Category averages
- `hpr`, `dei`, `pdi`, `gpi` — Dimension averages

### Success Metrics

- Reports include benchmark context for 100% of paid tiers
- Customer feedback indicates increased perceived value
- Internal: "How You Compare" section renders correctly in PDF

---

## Feature 2: Pricing Page

**Ticket:** `docs/_masemIT/requirements/ticket-pricing-page.md`

### Problem Statement

ChainSights has no dedicated pricing page. Prices are only visible when a user initiates a checkout flow. This hurts:
- **Transparency** — Users can't compare options without clicking through
- **SEO** — "chainsights pricing" searches land nowhere
- **Sales enablement** — No link to share in DMs, forums, Discord

### Solution

Create `/pricing` route with:
- One-time report tiers (Free Check, Deep Dive €49, Governance Audit €149)
- Subscription product (DAO Matrix €19/month or €99/year)
- FAQ section
- Navigation link

### Design Constraints

- Reuse existing `TierCards` component from `/check` flow
- Add `[NEW]` badge to features enhanced by Feature 1 (DGI Benchmark)
- Responsive: 3-column grid on desktop, stacked on mobile

### Success Metrics

- `/pricing` accessible and indexed by search engines
- Navigation includes Pricing link
- Analytics: `pricing_view` events tracked

---

## Feature 3: Account Dashboard

**Ticket:** `docs/_masemIT/requirements/ticket-account-dashboard.md`

### Problem Statement

Matrix subscribers have no way to:
- View/download invoices
- Update payment method
- Cancel subscription
- See their report history

Currently requires contacting support — unacceptable UX for a self-service SaaS.

### Solution

Create `/account` route (protected by Magic Link Auth) with:
- Subscription status display
- "Manage Subscription" button → Stripe Customer Portal
- Report history with download links
- Upsell CTAs for non-subscribers

### Dependencies

- ✅ Magic Link Auth (already deployed)
- ✅ Stripe Customer Portal (Stripe feature, needs API integration)
- ✅ `subscriptions` table with `stripeCustomerId`
- ✅ `reports` table with `finalPdfUrl`

### Success Metrics

- Subscribers can access billing portal without support intervention
- Report download links work for historical purchases
- Zero support tickets for "how do I cancel?"

---

## Implementation Priority & Dependencies

```
Week 1 (Parallel):
┌──────────────────┐     ┌──────────────────┐
│  Feature 1       │     │  Feature 2       │
│  DGI Benchmark   │     │  Pricing Page    │
│  (Backend+PDF)   │     │  (Frontend)      │
└────────┬─────────┘     └────────┬─────────┘
         │                        │
         └────────┬───────────────┘
                  │
                  ▼
         ┌──────────────────┐
         │  Feature 2b      │
         │  Update TierCards│
         │  with [NEW] badge│
         └──────────────────┘

Week 2:
┌──────────────────┐
│  Feature 3       │
│  Account Dashboard│
│  (Full-stack)    │
└──────────────────┘
```

**Dependency Notes:**
- Feature 1 and Feature 2 can run in parallel
- Feature 2 should update TierCards with new Feature 1 capabilities after Feature 1 merges
- Feature 3 is independent but lower priority

---

## Out of Scope (Phase 3+)

| Feature | Reason |
|---------|--------|
| API Access product | Requires separate pricing model validation |
| Starter Tier (automated monthly) | Deferred pending demand validation |
| Team/Organization accounts | Enterprise feature, not MVP |
| Wallet-Connect as login method | Nice-to-have, not blocking |

---

## Acceptance Criteria (Phase Complete)

- [ ] Governance Audit reports include full "How You Compare" benchmark section
- [ ] Deep Dive reports include headline benchmark
- [ ] `/pricing` page live with all current products
- [ ] TierCards updated with benchmark features + [NEW] badges
- [ ] `/account` page live with subscription management
- [ ] Stripe Customer Portal integrated
- [ ] Report history accessible with download links
- [ ] All features tracked in analytics
- [ ] Mobile responsive across all new pages

---

## Reference Documents

| Document | Purpose |
|----------|---------|
| `docs/_masemIT/requirements/ticket-dgi-benchmark-in-reports.md` | Detailed implementation spec for Feature 1 |
| `docs/_masemIT/requirements/ticket-pricing-page.md` | Detailed implementation spec for Feature 2 |
| `docs/_masemIT/requirements/ticket-account-dashboard.md` | Detailed implementation spec for Feature 3 |
| `docs/project_context.md` | Single source of truth for pricing, naming, feature flags |
| `docs/architecture.md` | Technical architecture reference |

---

## Approval

**Product:** Mario ✅
**Date:** 2026-02-04

Ready for Epic & Story creation.
