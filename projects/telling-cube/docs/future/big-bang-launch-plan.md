# Big Bang Launch Plan

**Date:** 2026-01-19
**Status:** Approved in Party Mode Session
**Participants:** Mario, Mary (Analyst), John (PM), Winston (Architect), Maya (Marketing), Sally (UX), Bob (SM), Anna (Billing), Daniel (Legal)
**Branch:** develop

---

## Executive Summary

Major feature release combining new pricing model, improved UX, extended data capabilities, and charity integration. Full agile cycle - no shortcuts.

---

## Agreed Features

| # | Feature | Description | Priority |
|---|---------|-------------|----------|
| 1 | **9 Curated Scenarios** | Replace 3x3 matrix with 9 pre-built example tiles | High |
| 2 | **€9 One-Time Tier** | Single generation, low barrier entry | High |
| 3 | **Magic Link Login** | Passwordless auth via email (Resend) | High |
| 4 | **User Dashboard** | Generation history, re-downloads for paid users | High |
| 5 | **Scheduled Generation** | Async via QStash, email when ready | High |
| 6 | **Multi-Year Data (3-5yr)** | Epic 2 - extended time horizons | High |
| 7 | **HR Domain** | Epic 3 - workforce analytics view | Medium |
| 8 | **3% to hoki.help** | Charity donation, quarterly transfer | High |
| 9 | **Live Donation Counter** | "€XX donated to children's hospice" on landing | Medium |
| 10 | **Discount Coupons** | HOKIHEART, BRIAN10, PHTC2026 | Medium |
| 11 | **Brian Julius Promo** | Leverage 55K followers for launch | Launch |

---

## Hybrid Pricing Model

### One-Time (Founding Member)

| Tier | Price | Generations | Data Years | HR Domain | Seats |
|------|-------|-------------|------------|-----------|-------|
| Single | €9 | 1 | 1 year | No | 1 |
| Lifetime | €99 | Unlimited | 1-3 years | No | 1 |
| Lifetime+ | €299 | Unlimited | 1-5 years | Yes | 1 |

### Subscriptions (Pro Plans)

| Tier | Price | Generations | Data Years | HR Domain | Seats |
|------|-------|-------------|------------|-----------|-------|
| Pro | €19/mo | Unlimited | 1-5 years | Yes | 1 |
| Team | €49/mo | Unlimited | 1-5 years | Yes | 5 |

### Early Supporter Policy

All existing customers (3 people) who purchased before this launch receive **full Lifetime+ benefits forever** - no action required on their part.

---

## UX Architecture

### Free User Flow
```
Landing Page
    ↓
9 Curated Tiles (click to explore)
    ↓
Example Detail View (Finance/Sales/HR views)
    ↓
CTA: "Generate Your Own - €9"
```

### Paid User Flow
```
Payment Complete (Stripe)
    ↓
Magic Link Email Sent
    ↓
Click Link → Generation Dialog
    ↓
┌─────────────────────────────────┐
│  Generate Your Scenario         │
│                                 │
│  Company Size:                  │
│  ○ Startup (10-50 employees)    │
│  ○ MidCap (200-500 employees)   │
│  ○ LargeCap (2,000+ employees)  │
│                                 │
│  Industry Sector:               │
│  ○ Consumer Staples             │
│  ○ Industrials                  │
│  ○ Financials                   │
│                                 │
│  [Generate My Scenario]         │
└─────────────────────────────────┘
    ↓
Queued (QStash) → Email when ready
    ↓
Dashboard: History + Downloads
```

### Pricing Page UX
Two-path selection to reduce cognitive load:
- Path 1: "Individual" → Shows €9, €99, €299 one-time options
- Path 2: "Team/Agency" → Shows €19/mo, €49/mo subscription options

---

## 9 Curated Scenarios

### Existing (5)
| # | Size | Sector | Company Name |
|---|------|--------|--------------|
| 1 | Startup | Consumer Staples | Alpine Bakery GmbH |
| 2 | MidCap | Industrials | TechParts Manufacturing AG |
| 3 | LargeCap | Financials | EuroCredit Union |
| 4 | Startup | Industrials | GreenTech Solutions |
| 5 | MidCap | Consumer Staples | FreshFood Distribution |

### To Create (4)
| # | Size | Sector | Company Name |
|---|------|--------|--------------|
| 6 | LargeCap | Consumer Staples | TBD |
| 7 | Startup | Financials | TBD |
| 8 | MidCap | Financials | TBD |
| 9 | LargeCap | Industrials | TBD |

---

## Charity Integration: hoki.help

### What is hoki.help?
HoKi NÖ (Hospizteams für Kinder, Jugendliche und junge Erwachsene) - Austrian hospice organization providing free support to families with seriously ill children and young people.

Website built pro-bono by masemIT. 100% of donations go directly to the organization.

### Implementation
- **Percentage:** 3% of ALL revenue (one-time + subscriptions)
- **Transfer:** Quarterly, handled manually by Mario
- **Tracking:** Simple spreadsheet
- **Display:** Live counter on landing page ("€XX donated to children's hospice care")
- **Badge:** Footer text "3% of every purchase supports hoki.help"

---

## Discount Coupons

| Code | Discount | Purpose |
|------|----------|---------|
| `HOKIHEART` | 10% off + 5% charity (instead of 3%) | Charity-focused buyers |
| `BRIAN10` | 10% off | Brian Julius audience |
| `PHTC2026` | 10% off | Product Hunt launch day |

---

## Sprint Plan (Full Agile Cycle)

| Sprint | Focus | Key Deliverables | Dependencies |
|--------|-------|------------------|--------------|
| **Sprint 2** | Auth & €9 Tier | Magic link login (Resend), basic user model, minimal dashboard, €9 Stripe product | None |
| **Sprint 3** | Multi-Year | 3-5 year generation logic, QStash scheduled jobs, progressive UI for long generations | Sprint 2 |
| **Sprint 4** | HR Domain | HR event types, HR deriver, HR view components, HR export endpoints | Sprint 2 |
| **Sprint 5** | 9 Tiles & Subscriptions | 4 new curated scenarios, subscription Stripe products, simplified landing page UI | Sprint 2 |
| **Sprint 6** | Launch Prep | Charity counter component, coupon code system, pricing page redesign, Brian outreach | All above |

---

## Technical Notes

### Entitlements Model
```typescript
type UserEntitlement = {
  type: 'single' | 'lifetime' | 'lifetime_plus' | 'pro' | 'team'
  model: 'one_time' | 'subscription'
  maxYears: 1 | 3 | 5
  hasHR: boolean
  seats: number
  generationsRemaining: number | 'unlimited'
  expiresAt: Date | null  // null for lifetime
}
```

### Stripe Products Needed
- 3 one-time products: €9, €99, €299
- 2 subscription products: €19/mo, €49/mo
- Webhook handling for both `checkout.session.completed` and `invoice.paid`

---

## Marketing Strategy

### Positioning
- **Keep** "ivory-billed woodpecker" angle (no subscription REQUIRED)
- **Add** "Founding Member Lifetime Access" language
- **Create urgency** with "Limited time pricing"
- Brian's original endorsement stays valid

### Brian Julius Collaboration
- Share new features with Brian
- Provide exclusive coupon code (BRIAN10)
- Request promotion to his 55K followers
- Time with Product Hunt launch if possible

### Key Message
> "9 ready-to-explore business scenarios. Generate your own for €9. Pro plans for teams. 3% goes to children's hospice care."

---

## Definition of Done

This launch is complete when:
- [ ] All 9 curated scenarios live and explorable
- [ ] €9 one-time tier functional end-to-end
- [ ] Magic link login working
- [ ] User dashboard showing history and downloads
- [ ] Scheduled generation via QStash operational
- [ ] Multi-year (3-5 year) data generation working
- [ ] HR domain views available
- [ ] Subscription tiers (Pro/Team) purchasable
- [ ] Charity counter displaying on landing page
- [ ] All coupon codes active in Stripe
- [ ] Early supporters upgraded to Lifetime+
- [ ] Brian Julius contacted and promo scheduled

---

## References

- Epic 01: `docs/epics/epic-01-curated-reports.md` (DONE)
- Epic 02: `docs/epics/epic-02-multi-year-scenarios.md`
- Epic 03: `docs/epics/epic-03-hr-data-domain.md`
- Product Hunt Plan: `docs/future/plan-product-hunt-launch-2026.md`
- Curated Reports Concept: `docs/future/concept-curated-reports-free-tier.md`
- hoki.help: https://hoki.help

---

*Document created from Party Mode session 2026-01-19. Full agile cycle - no shortcuts!*
