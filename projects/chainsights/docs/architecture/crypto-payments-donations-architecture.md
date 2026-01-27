---
title: "Architecture Document: Crypto Payments + Donation Tracking"
author: "Winston (Architect)"
date: "2026-01-19"
status: "Ready for Stories"
related_docs:
  - "../analysis/product-brief-crypto-payments-donations-2026-01-19.md"
  - "../ux/checkout-crypto-donations-ux-spec.md"
---

# Architecture Document: Crypto Payments + Donation Tracking

**Architect:** Winston
**Date:** 2026-01-19
**Status:** Ready for Story Creation

---

## 🚨 CRITICAL CONSTRAINT

> **ALL CODE CHANGES GO TO `develop` BRANCH ONLY!**
>
> - Staging environment on Vercel linked to `develop`
> - Production merge only after full validation

---

## Executive Summary

This document specifies the technical architecture for adding crypto payments via Coinbase Commerce and implementing 3% donation tracking across both payment rails (Stripe + Coinbase).

**Key Architectural Decisions:**
1. **Coinbase Commerce:** Hosted checkout (redirect) — iframe embedding not supported
2. **Two-Rail Architecture:** Stripe (fiat) + Coinbase (crypto) as parallel payment systems
3. **Unified Donation Tracking:** Database table aggregating donations from both sources
4. **Webhook-Driven:** Both Stripe and Coinbase trigger webhooks on successful payment

---

## System Architecture

### High-Level Flow

```
┌──────────────────────────────────────────────────────────────────┐
│                         CHECKOUT PAGE                            │
│  ┌─────────────┐                      ┌─────────────┐           │
│  │ 💳 Card     │                      │ ⟠ Crypto    │           │
│  └──────┬──────┘                      └──────┬──────┘           │
└─────────┼────────────────────────────────────┼──────────────────┘
          │                                    │
          ▼                                    ▼
┌─────────────────────┐              ┌─────────────────────┐
│   STRIPE CHECKOUT   │              │ COINBASE COMMERCE   │
│   (embedded)        │              │ (redirect)          │
└──────────┬──────────┘              └──────────┬──────────┘
           │                                    │
           ▼                                    ▼
┌─────────────────────┐              ┌─────────────────────┐
│  Stripe Webhook     │              │  Coinbase Webhook   │
│  payment_intent     │              │  charge:confirmed   │
│  .succeeded         │              │                     │
└──────────┬──────────┘              └──────────┬──────────┘
           │                                    │
           └────────────────┬───────────────────┘
                            ▼
              ┌─────────────────────────┐
              │   DONATION TRACKING     │
              │   - Calculate 3%        │
              │   - Store in database   │
              │   - Log for monthly     │
              └─────────────┬───────────┘
                            ▼
              ┌─────────────────────────┐
              │   CONFIRMATION PAGE     │
              │   - Show success        │
              │   - Display donation    │
              └─────────────────────────┘
```

---

## Answering Sally's Questions

### 1. Can Coinbase Commerce be embedded in iframe?

**Answer: No — Redirect Only**

Coinbase Commerce uses a hosted checkout page. For security and PCI compliance, payment processors typically do not support iframe embedding.

**Flow:**
1. User clicks "⟠ Crypto" button
2. Frontend creates a Coinbase Commerce charge via API
3. User is redirected to `commerce.coinbase.com/charges/[charge_id]`
4. User completes payment on Coinbase
5. Coinbase redirects back to our success/cancel URL
6. Webhook confirms payment

**UX Implication for Sally:**
- Instead of modal with embedded checkout, use modal that explains the redirect
- "You'll be redirected to Coinbase to complete your payment"
- Then redirect to Coinbase hosted checkout

### 2. Webhook Endpoint Design

**Two separate endpoints:**

```
POST /api/webhooks/stripe      → handles Stripe payment events
POST /api/webhooks/coinbase    → handles Coinbase Commerce events
```

### 3. Database Schema for Donation Tracking

**New table: `donations`**

See Database Schema section below.

---

## Component Architecture

### 1. Payment Selection Component

**File:** `src/components/PaymentMethodSelector.tsx`

```typescript
interface PaymentMethodSelectorProps {
  orderTotal: number;
  orderId: string;
  onSelectCard: () => void;
  onSelectCrypto: () => void;
}
```

**Behavior:**
- Renders two equal buttons (Card / Crypto)
- Shows donation badge below
- Card → triggers existing Stripe checkout
- Crypto → opens redirect confirmation modal

### 2. Crypto Redirect Modal

**File:** `src/components/CryptoCheckoutModal.tsx`

```typescript
interface CryptoCheckoutModalProps {
  isOpen: boolean;
  onClose: () => void;
  orderTotal: number;
  orderId: string;
}
```

**Behavior:**
- Shows order total
- Explains redirect to Coinbase
- "Continue to Coinbase" button
- Creates charge and redirects

### 3. Coinbase Commerce Integration

**File:** `src/lib/coinbase/commerce.ts`

```typescript
interface CreateChargeParams {
  name: string;           // "ChainSights Governance Report"
  description: string;    // "Report for [DAO Name]"
  amount: number;         // 99.00
  currency: string;       // "EUR"
  orderId: string;        // Internal order reference
  customerEmail: string;
  metadata: {
    order_id: string;
    dao_name: string;
    project: string;      // "chainsights"
    donation_amount: number; // 2.97
  };
  redirectUrl: string;    // Success URL
  cancelUrl: string;      // Cancel URL
}

async function createCharge(params: CreateChargeParams): Promise<{
  chargeId: string;
  hostedUrl: string;
}>;
```

### 4. Webhook Handlers

**File:** `src/app/api/webhooks/coinbase/route.ts`

```typescript
// POST /api/webhooks/coinbase
export async function POST(request: Request) {
  // 1. Verify webhook signature
  // 2. Parse event type
  // 3. Handle charge:confirmed event
  // 4. Create donation record
  // 5. Update order status
  // 6. Log for monitoring
}
```

**Coinbase Webhook Events:**
- `charge:created` — Charge created, awaiting payment
- `charge:pending` — Payment detected, confirming
- `charge:confirmed` — Payment confirmed ✅
- `charge:failed` — Payment failed
- `charge:expired` — Charge expired without payment

**File:** `src/app/api/webhooks/stripe/route.ts` (update existing)

```typescript
// Add donation tracking to existing Stripe webhook
// On payment_intent.succeeded:
// 1. Extract donation_amount from metadata
// 2. Create donation record
// 3. Continue existing order fulfillment
```

---

## Database Schema

### New Table: `donations`

```sql
CREATE TABLE donations (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

  -- Payment reference
  order_id VARCHAR(255) NOT NULL,
  payment_provider VARCHAR(50) NOT NULL,  -- 'stripe' | 'coinbase'
  payment_id VARCHAR(255) NOT NULL,       -- Stripe payment_intent_id or Coinbase charge_id

  -- Amounts
  order_amount DECIMAL(10,2) NOT NULL,    -- Total order (e.g., 99.00)
  donation_amount DECIMAL(10,2) NOT NULL, -- 3% donation (e.g., 2.97)
  currency VARCHAR(3) NOT NULL DEFAULT 'EUR',

  -- Project tracking (for multi-project support)
  project VARCHAR(100) NOT NULL DEFAULT 'chainsights',

  -- Metadata
  customer_email VARCHAR(255),
  dao_name VARCHAR(255),

  -- Status
  status VARCHAR(50) NOT NULL DEFAULT 'confirmed',  -- 'confirmed' | 'transferred'
  transferred_at TIMESTAMP,  -- When included in monthly transfer

  -- Timestamps
  created_at TIMESTAMP NOT NULL DEFAULT NOW(),
  updated_at TIMESTAMP NOT NULL DEFAULT NOW()
);

-- Index for monthly aggregation queries
CREATE INDEX idx_donations_created_at ON donations(created_at);
CREATE INDEX idx_donations_project ON donations(project);
CREATE INDEX idx_donations_status ON donations(status);
```

### Drizzle Schema

**File:** `src/lib/db/schema.ts` (add to existing)

```typescript
export const donations = pgTable('donations', {
  id: uuid('id').primaryKey().defaultRandom(),

  // Payment reference
  orderId: varchar('order_id', { length: 255 }).notNull(),
  paymentProvider: varchar('payment_provider', { length: 50 }).notNull(),
  paymentId: varchar('payment_id', { length: 255 }).notNull(),

  // Amounts
  orderAmount: decimal('order_amount', { precision: 10, scale: 2 }).notNull(),
  donationAmount: decimal('donation_amount', { precision: 10, scale: 2 }).notNull(),
  currency: varchar('currency', { length: 3 }).notNull().default('EUR'),

  // Project tracking
  project: varchar('project', { length: 100 }).notNull().default('chainsights'),

  // Metadata
  customerEmail: varchar('customer_email', { length: 255 }),
  daoName: varchar('dao_name', { length: 255 }),

  // Status
  status: varchar('status', { length: 50 }).notNull().default('confirmed'),
  transferredAt: timestamp('transferred_at'),

  // Timestamps
  createdAt: timestamp('created_at').notNull().defaultNow(),
  updatedAt: timestamp('updated_at').notNull().defaultNow(),
});
```

---

## Stripe Metadata Enhancement

### Current Stripe Integration

Update existing Stripe checkout to include donation metadata:

```typescript
// When creating Stripe checkout session
const session = await stripe.checkout.sessions.create({
  // ... existing params
  metadata: {
    order_id: orderId,
    project: 'chainsights',
    donation_amount: (orderTotal * 0.03).toFixed(2),
    dao_name: daoName,
  },
  payment_intent_data: {
    metadata: {
      order_id: orderId,
      project: 'chainsights',
      donation_amount: (orderTotal * 0.03).toFixed(2),
      dao_name: daoName,
    },
  },
});
```

---

## API Endpoints

### New Endpoints

| Method | Path | Purpose |
|--------|------|---------|
| POST | `/api/crypto-checkout` | Create Coinbase charge, return redirect URL |
| POST | `/api/webhooks/coinbase` | Handle Coinbase webhook events |
| GET | `/api/donations/total` | Get cumulative donation total for /impact page |

### Endpoint: Create Crypto Checkout

**File:** `src/app/api/crypto-checkout/route.ts`

```typescript
// POST /api/crypto-checkout
// Request body:
{
  orderId: string;
  orderTotal: number;
  customerEmail: string;
  daoName: string;
}

// Response:
{
  chargeId: string;
  checkoutUrl: string;  // Coinbase hosted checkout URL
}
```

### Endpoint: Get Donation Total

**File:** `src/app/api/donations/total/route.ts`

```typescript
// GET /api/donations/total?project=chainsights
// Response:
{
  total: number;        // e.g., 1247.53
  count: number;        // e.g., 127 donations
  currency: "EUR";
  lastUpdated: string;  // ISO timestamp
}
```

---

## Impact Page Implementation

### Route

**File:** `src/app/impact/page.tsx`

### Logic

```typescript
export default async function ImpactPage() {
  const { total } = await getDonationTotal('chainsights');

  const VISIBILITY_THRESHOLD = 500;

  if (total < VISIBILITY_THRESHOLD) {
    return <ComingSoonView />;
  }

  return <ImpactView total={total} />;
}
```

### Data Fetching

```typescript
// src/lib/donations/queries.ts

export async function getDonationTotal(project: string): Promise<{
  total: number;
  count: number;
}> {
  const result = await db
    .select({
      total: sql<number>`COALESCE(SUM(donation_amount), 0)`,
      count: sql<number>`COUNT(*)`,
    })
    .from(donations)
    .where(eq(donations.project, project));

  return {
    total: Number(result[0].total),
    count: Number(result[0].count),
  };
}
```

---

## Webhook Monitoring

### Health Check Query

```typescript
// Check if Coinbase webhooks are working
export async function getLastWebhookTimestamp(provider: string): Promise<Date | null> {
  const result = await db
    .select({ lastCreated: sql<Date>`MAX(created_at)` })
    .from(donations)
    .where(eq(donations.paymentProvider, provider));

  return result[0]?.lastCreated || null;
}
```

### Alert Logic

```typescript
// In monitoring/cron job
const lastCoinbaseWebhook = await getLastWebhookTimestamp('coinbase');
const hoursSinceLastWebhook = /* calculate */;

if (hoursSinceLastWebhook > 48) {
  // Send alert (LogSnag, email, Slack, etc.)
  await sendAlert('Coinbase webhook inactive for 48+ hours');
}
```

---

## Environment Variables

### New Variables Required

```env
# Coinbase Commerce
COINBASE_COMMERCE_API_KEY=xxx
COINBASE_COMMERCE_WEBHOOK_SECRET=xxx

# URLs (for redirect after payment)
NEXT_PUBLIC_BASE_URL=https://chainsights.one  # or staging URL for develop
```

### Staging vs Production

| Variable | Staging (`develop`) | Production (`main`) |
|----------|---------------------|---------------------|
| `COINBASE_COMMERCE_API_KEY` | Test API key | Production API key |
| `NEXT_PUBLIC_BASE_URL` | `https://develop.chainsights.one` | `https://chainsights.one` |

---

## File Structure

### New Files

```
src/
├── app/
│   ├── api/
│   │   ├── crypto-checkout/
│   │   │   └── route.ts           # Create Coinbase charge
│   │   ├── webhooks/
│   │   │   └── coinbase/
│   │   │       └── route.ts       # Coinbase webhook handler
│   │   └── donations/
│   │       └── total/
│   │           └── route.ts       # Get donation total
│   └── impact/
│       └── page.tsx               # Impact page
├── components/
│   ├── PaymentMethodSelector.tsx  # Card/Crypto selection
│   ├── CryptoCheckoutModal.tsx    # Redirect confirmation modal
│   ├── DonationBadge.tsx          # Checkout donation badge
│   └── DonationImpactBox.tsx      # Confirmation page donation box
└── lib/
    ├── coinbase/
    │   └── commerce.ts            # Coinbase Commerce SDK wrapper
    └── donations/
        └── queries.ts             # Donation database queries
```

### Modified Files

```
src/
├── app/
│   └── api/
│       └── webhooks/
│           └── stripe/
│               └── route.ts       # Add donation tracking
├── lib/
│   └── db/
│       └── schema.ts              # Add donations table
└── drizzle/
    └── XXXX_add_donations.sql     # Migration
```

---

## Security Considerations

### Webhook Verification

**Coinbase:**
```typescript
import { Webhook } from 'coinbase-commerce-node';

function verifyWebhook(payload: string, signature: string): boolean {
  return Webhook.verifyEventBody(
    payload,
    signature,
    process.env.COINBASE_COMMERCE_WEBHOOK_SECRET
  );
}
```

**Stripe:** (existing, ensure it's implemented)
```typescript
const event = stripe.webhooks.constructEvent(
  payload,
  signature,
  process.env.STRIPE_WEBHOOK_SECRET
);
```

### Data Validation

- Validate all amounts server-side (never trust client)
- Verify order exists before creating donation record
- Idempotency: check if donation already recorded for payment_id

---

## Testing Strategy

### Unit Tests

- Donation calculation (3% of order amount)
- Webhook signature verification
- Database queries

### Integration Tests

- Coinbase charge creation
- Webhook handling flow
- Impact page threshold logic

### E2E Tests (Staging)

- Full checkout flow with Coinbase (test mode)
- Webhook receipt and donation recording
- Impact page display

### Smoke Test Checklist (Post-Deploy)

- [ ] Create test Stripe payment → donation recorded
- [ ] Create test Coinbase payment → donation recorded
- [ ] Check `/api/donations/total` returns correct sum
- [ ] Check `/impact` page displays correctly

---

## Rollout Plan

### Phase 1: Database & Backend

1. Create migration for `donations` table
2. Run migration on staging
3. Implement donation queries
4. Update Stripe webhook to record donations

### Phase 2: Coinbase Integration

1. Set up Coinbase Commerce account
2. Implement charge creation endpoint
3. Implement Coinbase webhook handler
4. Test on staging with test payments

### Phase 3: Frontend

1. Implement PaymentMethodSelector
2. Implement CryptoCheckoutModal
3. Add DonationBadge to checkout
4. Add DonationImpactBox to confirmation
5. Implement Impact page

### Phase 4: Validation

1. Full E2E testing on staging
2. Verify donation tracking accuracy
3. Test webhook monitoring
4. Review with team
5. Merge to main

---

## Dependencies

### NPM Packages

```json
{
  "coinbase-commerce-node": "^1.0.4"
}
```

### External Services

- Coinbase Commerce account
- Coinbase API keys (test + production)

---

## Next Steps

| Step | Owner | Action |
|------|-------|--------|
| 1 | Bob | Create Epics & Stories from this architecture |
| 2 | Amelia | Implement on `develop` branch |
| 3 | Murat | Write tests |
| 4 | Team | Staging validation |
| 5 | Mario | Production release approval |

---

**Architecture Status: ✅ COMPLETE**

*Created by Winston (Architect)*
*2026-01-19*
