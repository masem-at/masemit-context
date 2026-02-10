# DEV TICKET: Account-Bereich / Mini-Dashboard

> **⚠️ PRICING OUTDATED** — DAO Matrix is FREE since 2026-02-05. €19/mo and €99/yr subscriptions are deprecated. See `docs/project_context.md`.

**Priority:** Medium
**Effort:** Medium
**Dependencies:** Magic Link Auth (✅ already deployed)

---

## Context

Magic Link Auth ist bereits deployed. Sobald ein Kunde ein Matrix-Abo kauft, braucht er Zugang zu Rechnungen und Abo-Verwaltung. Aktuell gibt es keinen Account-Bereich — Kunden haben nach dem Kauf keine Möglichkeit ihr Abo zu verwalten.

### Ist-Zustand (per Winston's Analyse)

- Kein Customer-Auth-Flow (nur Admin-Login mit Cookie-Session)
- Stripe Customer IDs werden gespeichert (`stripeCustomerId` in orders + subscriptions)
- Wallet-Connect existiert nur für Crypto-Payments, nicht für Auth
- Magic Link Auth ist deployed (✅ seit 2026-02-01)
- Kein User-Table, kein Session-Management für Kunden

### Gewählte Strategie (2 Phasen)

**Phase 1 (dieses Ticket):** Stripe Customer Portal per Magic Link — minimaler Aufwand, löst das akute Abo-Verwaltungs-Problem.

**Phase 2 (Zukunft, separates Ticket):** Richtiger Account-Bereich mit Report-History, Dashboard, Preferences.

---

## Phase 1: Account MVP

### User Flow

```
┌─────────────────────────────────────────────────────────────┐
│                                                             │
│  1. User navigiert zu /account                              │
│     └── oder klickt "Manage Subscription" Link              │
│                                                             │
│  2. Magic Link Auth (bereits deployed)                      │
│     └── Email eingeben → Magic Link erhalten → eingeloggt   │
│                                                             │
│  3. Account Overview Page                                   │
│     ┌─────────────────────────────────────────────┐         │
│     │                                             │         │
│     │  👋 Welcome back, mario@masem.at            │         │
│     │                                             │         │
│     │  ┌─────────────────────────────────────┐    │         │
│     │  │  YOUR SUBSCRIPTION                  │    │         │
│     │  │                                     │    │         │
│     │  │  DAO Matrix Access                  │    │         │
│     │  │  Plan: Monthly (€19/month)          │    │         │
│     │  │  Status: Active ✅                   │    │         │
│     │  │  Next billing: March 4, 2026        │    │         │
│     │  │                                     │    │         │
│     │  │  [Manage Subscription]              │    │         │
│     │  │  Opens Stripe Customer Portal       │    │         │
│     │  └─────────────────────────────────────┘    │         │
│     │                                             │         │
│     │  ┌─────────────────────────────────────┐    │         │
│     │  │  YOUR REPORTS                       │    │         │
│     │  │                                     │    │         │
│     │  │  • Lido - Deep Dive (Jan 28, 2026)  │    │         │
│     │  │    [Download PDF]                   │    │         │
│     │  │                                     │    │         │
│     │  │  • ENS - Gov Audit (Feb 1, 2026)    │    │         │
│     │  │    [Download PDF]                   │    │         │
│     │  │                                     │    │         │
│     │  └─────────────────────────────────────┘    │         │
│     │                                             │         │
│     │  ┌─────────────────────────────────────┐    │         │
│     │  │  QUICK ACTIONS                      │    │         │
│     │  │                                     │    │         │
│     │  │  [Order New Report]                 │    │         │
│     │  │  [View DAO Matrix]                  │    │         │
│     │  │  [Manage Billing] → Stripe Portal   │    │         │
│     │  └─────────────────────────────────────┘    │         │
│     └─────────────────────────────────────────────┘         │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### Alternative: No Subscription

```
┌─────────────────────────────────────────────┐
│                                             │
│  👋 Welcome back, user@example.com          │
│                                             │
│  ┌─────────────────────────────────────┐    │
│  │  NO ACTIVE SUBSCRIPTION             │    │
│  │                                     │    │
│  │  Unlock the full DAO Matrix with    │    │
│  │  44 DAOs tracked daily.             │    │
│  │                                     │    │
│  │  [Subscribe — €19/month]            │    │
│  │  [Subscribe — €99/year (save 57%)]  │    │
│  └─────────────────────────────────────┘    │
│                                             │
│  ┌─────────────────────────────────────┐    │
│  │  YOUR REPORTS                       │    │
│  │                                     │    │
│  │  No reports yet.                    │    │
│  │  [Order Your First Report]          │    │
│  └─────────────────────────────────────┘    │
│                                             │
└─────────────────────────────────────────────┘
```

---

## Technical Requirements

### 1. Stripe Customer Portal Integration

Create API route to generate Stripe Billing Portal session:

```typescript
// /api/billing-portal/route.ts
// 1. Get authenticated user's email from Magic Link session
// 2. Look up stripeCustomerId from subscriptions or orders table
// 3. Create Stripe Billing Portal session
// 4. Return portal URL for redirect

const session = await stripe.billingPortal.sessions.create({
  customer: stripeCustomerId,
  return_url: 'https://chainsights.one/account',
});
```

**Stripe Customer Portal features (configured in Stripe Dashboard):**
- View and download invoices
- Update payment method
- Cancel subscription
- View subscription details

### 2. Report History Query

```sql
-- Get all reports for a user by email
SELECT r.dao_name, r.tier, r.created_at, r.final_pdf_url
FROM reports r
JOIN orders o ON r.order_id = o.id
WHERE o.email = :userEmail
ORDER BY r.created_at DESC;
```

### 3. Subscription Status Query

```sql
-- Get active subscription for user by email
SELECT s.status, s.current_period_end, s.stripe_customer_id,
       s.plan_type, s.price_amount
FROM subscriptions s
WHERE s.email = :userEmail
  AND s.status = 'active'
ORDER BY s.created_at DESC
LIMIT 1;
```

### 4. Account Page Protection

- `/account` requires Magic Link authentication
- Redirect to login if no session
- After login, redirect back to `/account`

### 5. Portal Link in Confirmation Emails

**Quick win:** Add "Manage your subscription" link to:
- Subscription confirmation email
- Report delivery email
- Payment receipt email

Link format: `https://chainsights.one/account` (user logs in, then manages)

---

## Navigation Integration

Add "Account" to navigation (only visible when logged in):

```
Logo | [Rankings] [DGI] [Matrix] [Pricing] | [Account] [Logout]

// Not logged in:
Logo | [Rankings] [DGI] [Matrix] [Pricing] | [Login]
```

---

## Edge Cases

| Scenario | Behavior |
|----------|----------|
| User has subscription but no reports | Show subscription section, empty reports with CTA |
| User has reports but no subscription | Show reports, subscription upsell |
| User has neither | Show both upsells |
| Subscription cancelled but still active (period not ended) | Show "Active until [date], will not renew" |
| Payment failed | Show warning, link to update payment method via Stripe Portal |
| Multiple subscriptions (shouldn't happen) | Show most recent active one |
| Email not found in Stripe | Show "No billing history found" with support contact |

---

## Acceptance Criteria

- [ ] `/account` route exists, protected by Magic Link auth
- [ ] Subscription status displayed (plan, price, next billing, status)
- [ ] "Manage Subscription" button opens Stripe Customer Portal
- [ ] Report history listed with download links
- [ ] Correct state shown for: active sub, no sub, cancelled sub
- [ ] Navigation shows Account/Logout when logged in, Login when not
- [ ] Confirmation emails include "Manage Subscription" link
- [ ] Responsive: works on desktop, tablet, mobile
- [ ] Analytics tracking: `account_view`, `billing_portal_click` events

---

## Files to Create/Touch

1. `src/app/account/page.tsx` — New account page
2. `src/app/api/billing-portal/route.ts` — Stripe Portal session API
3. Navigation component — Add conditional Account/Login links
4. Email templates — Add "Manage Subscription" link
5. Middleware — Protect `/account` with auth check

---

## Out of Scope (Phase 2)

- User preferences / settings
- Notification preferences
- Wallet-Connect as additional login method
- Custom dashboard with charts
- Team/organization accounts
- Starter-Tier with automated monthly reports
