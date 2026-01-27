---
title: "Epic: Crypto Payments + Donation Tracking"
author: "Bob (Scrum Master)"
date: "2026-01-19"
status: "Ready for Development"
epic_id: "CRYPTO-001"
related_docs:
  - "../analysis/product-brief-crypto-payments-donations-2026-01-19.md"
  - "../ux/checkout-crypto-donations-ux-spec.md"
  - "../architecture/crypto-payments-donations-architecture.md"
---

# Epic: Crypto Payments + Donation Tracking

**Epic ID:** CRYPTO-001
**Created:** 2026-01-19
**Status:** Ready for Development
**Owner:** Bob (Scrum Master)

---

## 🚨 CRITICAL CONSTRAINT

> **ALL CODE CHANGES GO TO `develop` BRANCH ONLY!**
>
> No exceptions. Staging validation required before production.

---

## Epic Summary

Enable crypto payments via Coinbase Commerce and implement 3% donation tracking across all payment methods (Stripe + Coinbase), with visible donation messaging at checkout and a public `/impact` page.

---

## Acceptance Criteria (Epic Level)

- [ ] Customers can pay with crypto (USDC, ETH, BTC) via Coinbase Commerce
- [ ] Customers can still pay with cards via Stripe (existing)
- [ ] 3% of each payment is tracked as donation
- [ ] Donation badge visible on checkout page
- [ ] Donation confirmation shown after successful payment
- [ ] `/impact` page shows cumulative donations (when ≥€500)
- [ ] Webhook monitoring alerts if Coinbase stops responding
- [ ] All code on `develop` branch, validated on staging

---

## Stories

---

### Story 1: Database Schema for Donations

**Story ID:** CRYPTO-001-S01
**Points:** 2
**Priority:** P0 (Blocker)

**As a** system
**I want** a database table to store donation records
**So that** we can track and aggregate donations from all payment sources

#### Acceptance Criteria

- [ ] Migration creates `donations` table with schema per architecture doc
- [ ] Drizzle schema updated in `src/lib/db/schema.ts`
- [ ] Indexes created for `created_at`, `project`, `status`
- [ ] Migration runs successfully on staging

#### Tasks

- [ ] Create migration file `drizzle/XXXX_add_donations.sql`
- [ ] Add donations table to Drizzle schema
- [ ] Run migration on local
- [ ] Test migration on staging
- [ ] Verify table structure

#### Technical Notes

```typescript
// Schema from architecture doc
donations: {
  id, orderId, paymentProvider, paymentId,
  orderAmount, donationAmount, currency,
  project, customerEmail, daoName,
  status, transferredAt, createdAt, updatedAt
}
```

---

### Story 2: Donation Queries Library

**Story ID:** CRYPTO-001-S02
**Points:** 2
**Priority:** P0 (Blocker)

**As a** developer
**I want** reusable functions for donation data operations
**So that** I can easily create and query donation records

#### Acceptance Criteria

- [ ] `createDonation()` function creates donation record
- [ ] `getDonationTotal()` returns sum for a project
- [ ] `getLastWebhookTimestamp()` returns last donation timestamp by provider
- [ ] All functions have TypeScript types
- [ ] Unit tests pass

#### Tasks

- [ ] Create `src/lib/donations/queries.ts`
- [ ] Implement `createDonation()`
- [ ] Implement `getDonationTotal()`
- [ ] Implement `getLastWebhookTimestamp()`
- [ ] Write unit tests
- [ ] Export from index

---

### Story 3: Stripe Webhook — Add Donation Tracking

**Story ID:** CRYPTO-001-S03
**Points:** 3
**Priority:** P0 (Blocker)

**As a** system
**I want** Stripe webhooks to record donations automatically
**So that** every card payment contributes to the donation total

#### Acceptance Criteria

- [ ] Stripe checkout includes `donation_amount` in metadata
- [ ] Webhook handler extracts donation amount on `payment_intent.succeeded`
- [ ] Donation record created in database
- [ ] Existing checkout flow continues to work
- [ ] Unit tests pass

#### Tasks

- [ ] Update Stripe checkout session creation to include metadata
- [ ] Update Stripe webhook handler to extract donation data
- [ ] Call `createDonation()` on successful payment
- [ ] Add error handling (don't break payment if donation fails)
- [ ] Write integration tests
- [ ] Test on staging with real Stripe test payment

---

### Story 4: Coinbase Commerce Integration

**Story ID:** CRYPTO-001-S04
**Points:** 5
**Priority:** P1 (High)

**As a** developer
**I want** a Coinbase Commerce integration library
**So that** I can create charges and handle webhooks

#### Acceptance Criteria

- [ ] `createCharge()` function creates Coinbase charge
- [ ] Returns `chargeId` and `hostedUrl` for redirect
- [ ] Webhook signature verification implemented
- [ ] Webhook handler processes `charge:confirmed` events
- [ ] Donation record created on confirmed charge
- [ ] Environment variables documented

#### Tasks

- [ ] Install `coinbase-commerce-node` package
- [ ] Create `src/lib/coinbase/commerce.ts`
- [ ] Implement `createCharge()` with proper metadata
- [ ] Create `src/app/api/webhooks/coinbase/route.ts`
- [ ] Implement webhook signature verification
- [ ] Handle `charge:confirmed` event
- [ ] Call `createDonation()` on confirmed charge
- [ ] Add idempotency check (don't duplicate donations)
- [ ] Write unit tests
- [ ] Add env vars to `.env.example`

#### Environment Variables

```env
COINBASE_COMMERCE_API_KEY=xxx
COINBASE_COMMERCE_WEBHOOK_SECRET=xxx
```

---

### Story 5: Crypto Checkout API Endpoint

**Story ID:** CRYPTO-001-S05
**Points:** 3
**Priority:** P1 (High)

**As a** frontend
**I want** an API endpoint to initiate crypto checkout
**So that** I can redirect users to Coinbase

#### Acceptance Criteria

- [ ] `POST /api/crypto-checkout` creates charge and returns redirect URL
- [ ] Request validates required fields
- [ ] Response includes `chargeId` and `checkoutUrl`
- [ ] Error handling for Coinbase API failures
- [ ] Rate limiting considered

#### Tasks

- [ ] Create `src/app/api/crypto-checkout/route.ts`
- [ ] Implement request validation
- [ ] Call `createCharge()` from Coinbase library
- [ ] Return checkout URL
- [ ] Add error handling
- [ ] Write integration tests

#### API Contract

```typescript
// Request
POST /api/crypto-checkout
{
  orderId: string;
  orderTotal: number;
  customerEmail: string;
  daoName: string;
}

// Response
{
  chargeId: string;
  checkoutUrl: string;
}
```

---

### Story 6: Payment Method Selection Component

**Story ID:** CRYPTO-001-S06
**Points:** 3
**Priority:** P1 (High)

**As a** customer
**I want** to choose between card and crypto payment
**So that** I can pay with my preferred method

#### Acceptance Criteria

- [ ] Two equal-sized buttons: Card and Crypto
- [ ] Card button triggers existing Stripe checkout
- [ ] Crypto button opens redirect confirmation modal
- [ ] Donation badge displayed below buttons
- [ ] Responsive: stacks on mobile
- [ ] Accessible: focus states, keyboard navigation

#### Tasks

- [ ] Create `src/components/PaymentMethodSelector.tsx`
- [ ] Create `src/components/DonationBadge.tsx`
- [ ] Style buttons per UX spec (equal size, icons)
- [ ] Implement card selection handler
- [ ] Implement crypto selection handler (opens modal)
- [ ] Add responsive styles
- [ ] Add accessibility attributes
- [ ] Write component tests

#### Copy (from UX Spec)

- Headline: "How would you like to pay?"
- Card: "💳 Card" / "Visa, Mastercard, Klarna, EPS"
- Crypto: "⟠ Crypto" / "USDC, ETH, BTC & more"
- Badge: "💙 3% of your purchase helps families facing childhood illness via hoki.help"

---

### Story 7: Crypto Checkout Modal

**Story ID:** CRYPTO-001-S07
**Points:** 3
**Priority:** P1 (High)

**As a** customer
**I want** a confirmation before redirecting to Coinbase
**So that** I understand I'm leaving the site temporarily

#### Acceptance Criteria

- [ ] Modal opens when crypto selected
- [ ] Shows order total
- [ ] Explains redirect to Coinbase
- [ ] "Continue to Coinbase" button initiates checkout
- [ ] Close button and back link work
- [ ] Donation badge visible in modal
- [ ] Loading state while creating charge
- [ ] Error state if charge creation fails

#### Tasks

- [ ] Create `src/components/CryptoCheckoutModal.tsx`
- [ ] Implement modal open/close logic
- [ ] Display order information
- [ ] Call `/api/crypto-checkout` on continue
- [ ] Show loading spinner during API call
- [ ] Redirect to Coinbase on success
- [ ] Show error message on failure
- [ ] Add escape key to close
- [ ] Write component tests

---

### Story 8: Confirmation Page — Donation Message

**Story ID:** CRYPTO-001-S08
**Points:** 2
**Priority:** P1 (High)

**As a** customer
**I want** to see my donation impact after payment
**So that** I feel good about my purchase

#### Acceptance Criteria

- [ ] Donation impact box shows on confirmation page
- [ ] Displays actual donation amount (€X.XX)
- [ ] Shows emotional message "You just helped a family"
- [ ] Works for both Stripe and Coinbase payments
- [ ] Links to hoki.help

#### Tasks

- [ ] Create `src/components/DonationImpactBox.tsx`
- [ ] Calculate donation amount (orderTotal × 0.03)
- [ ] Add to existing confirmation page
- [ ] Handle both payment providers
- [ ] Style per UX spec
- [ ] Write component tests

#### Copy (from UX Spec)

- "💙 You just helped a family."
- "€[amount] of your purchase supports children's hospice care via hoki.help"

---

### Story 9: Donations Total API Endpoint

**Story ID:** CRYPTO-001-S09
**Points:** 2
**Priority:** P2 (Medium)

**As a** frontend
**I want** an API to get cumulative donation total
**So that** I can display it on the impact page

#### Acceptance Criteria

- [ ] `GET /api/donations/total` returns total and count
- [ ] Filters by project (default: chainsights)
- [ ] Returns currency
- [ ] Caches result for performance (optional)

#### Tasks

- [ ] Create `src/app/api/donations/total/route.ts`
- [ ] Call `getDonationTotal()` from queries library
- [ ] Return formatted response
- [ ] Add caching header (optional)
- [ ] Write tests

#### API Contract

```typescript
// Request
GET /api/donations/total?project=chainsights

// Response
{
  total: 1247.53,
  count: 127,
  currency: "EUR",
  lastUpdated: "2026-01-19T12:00:00Z"
}
```

---

### Story 10: Impact Page

**Story ID:** CRYPTO-001-S10
**Points:** 3
**Priority:** P2 (Medium)

**As a** visitor
**I want** to see ChainSights' total donation impact
**So that** I can see the company's values in action

#### Acceptance Criteria

- [ ] Page at `/impact` route
- [ ] Shows "Coming Soon" if total < €500
- [ ] Shows donation total prominently if ≥ €500
- [ ] Includes description of hoki.help
- [ ] Link to hoki.help website
- [ ] Responsive design
- [ ] SEO meta tags

#### Tasks

- [ ] Create `src/app/impact/page.tsx`
- [ ] Fetch donation total from API
- [ ] Implement threshold logic (€500)
- [ ] Create "Coming Soon" view
- [ ] Create "Impact" view with total
- [ ] Add link to hoki.help
- [ ] Add meta tags
- [ ] Style per UX spec
- [ ] Write tests

---

### Story 11: Webhook Health Monitoring

**Story ID:** CRYPTO-001-S11
**Points:** 2
**Priority:** P2 (Medium)

**As a** operator
**I want** alerts if Coinbase webhooks stop working
**So that** I can investigate payment issues quickly

#### Acceptance Criteria

- [ ] System tracks last webhook timestamp per provider
- [ ] Alert triggered if no webhook in 48h AND payments expected
- [ ] Alert sent via existing notification channel (LogSnag/Slack)

#### Tasks

- [ ] Implement `getLastWebhookTimestamp()` query
- [ ] Create monitoring check (cron or manual script)
- [ ] Integrate with notification system
- [ ] Document monitoring process
- [ ] Test alert flow

---

### Story 12: Integration Testing & Staging Validation

**Story ID:** CRYPTO-001-S12
**Points:** 3
**Priority:** P0 (Blocker before merge)

**As a** team
**I want** full integration testing on staging
**So that** we can confidently merge to production

#### Acceptance Criteria

- [ ] Stripe test payment → donation recorded
- [ ] Coinbase test payment → donation recorded
- [ ] `/api/donations/total` returns correct sum
- [ ] `/impact` page displays correctly
- [ ] Confirmation page shows donation for both providers
- [ ] No console errors
- [ ] Mobile responsive

#### Tasks

- [ ] Deploy all stories to staging
- [ ] Run smoke test checklist
- [ ] Test Stripe payment flow end-to-end
- [ ] Test Coinbase payment flow end-to-end
- [ ] Verify donation records in database
- [ ] Test impact page threshold
- [ ] Cross-browser testing
- [ ] Mobile testing
- [ ] Document any issues
- [ ] Get sign-off from Mario

---

## Story Dependency Graph

```
S01 (Database) ─────┬──→ S02 (Queries) ──┬──→ S03 (Stripe Webhook)
                    │                    │
                    │                    └──→ S04 (Coinbase Integration)
                    │                              │
                    │                              ▼
                    │                         S05 (Crypto API)
                    │                              │
                    └──────────────────────────────┼──→ S06 (Payment Selector)
                                                   │         │
                                                   │         ▼
                                                   │    S07 (Crypto Modal)
                                                   │
                                                   ├──→ S08 (Confirmation)
                                                   │
                                                   └──→ S09 (Total API) ──→ S10 (Impact Page)

S11 (Monitoring) - Independent, can run parallel

S12 (Testing) - Final, depends on all others
```

---

## Sprint Recommendation

### Sprint 1 (Foundation)

| Story | Points | Priority |
|-------|--------|----------|
| S01 - Database Schema | 2 | P0 |
| S02 - Donation Queries | 2 | P0 |
| S03 - Stripe Webhook | 3 | P0 |
| S04 - Coinbase Integration | 5 | P1 |
| **Total** | **12** | |

### Sprint 2 (Frontend + Integration)

| Story | Points | Priority |
|-------|--------|----------|
| S05 - Crypto API | 3 | P1 |
| S06 - Payment Selector | 3 | P1 |
| S07 - Crypto Modal | 3 | P1 |
| S08 - Confirmation | 2 | P1 |
| **Total** | **11** | |

### Sprint 3 (Polish + Release)

| Story | Points | Priority |
|-------|--------|----------|
| S09 - Total API | 2 | P2 |
| S10 - Impact Page | 3 | P2 |
| S11 - Monitoring | 2 | P2 |
| S12 - Testing & Validation | 3 | P0 |
| **Total** | **10** | |

---

## Definition of Done (All Stories)

- [ ] Code on `develop` branch
- [ ] Unit tests pass
- [ ] No TypeScript errors
- [ ] Code reviewed (self-review acceptable for solo)
- [ ] Works on staging
- [ ] Acceptance criteria met

---

## Risks & Mitigations

| Risk | Impact | Mitigation |
|------|--------|------------|
| Coinbase API changes | High | Pin package version, monitor changelog |
| Webhook delivery failures | Medium | Implement retry logic, monitoring |
| €500 threshold never reached | Low | Threshold is configurable |

---

## Open Questions

1. **Coinbase account setup:** Does Mario have a Coinbase Commerce account ready?
2. **Test mode:** Do we have Coinbase test API keys?
3. **Notification channel:** LogSnag or Slack for webhook alerts?

---

**Epic Status: ✅ READY FOR DEVELOPMENT**

*Created by Bob (Scrum Master)*
*2026-01-19*
