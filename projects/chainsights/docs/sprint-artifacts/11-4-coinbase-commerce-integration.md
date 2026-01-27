# Story 11.4: Coinbase Commerce Integration

Status: done

## Story

**As a** developer
**I want** a Coinbase Commerce integration library
**So that** I can create charges and handle webhooks for crypto payments

## Acceptance Criteria

1. - [x] `createCharge()` function creates Coinbase Commerce charge
2. - [x] Returns `chargeId` and `hostedUrl` for redirect
3. - [x] Webhook signature verification implemented
4. - [x] Webhook handler processes `charge:confirmed` events
5. - [x] Donation record created on confirmed charge
6. - [x] Environment variables documented

## Tasks / Subtasks

- [x] Task 1: Install and configure Coinbase Commerce SDK (AC: #6)
  - [x] 1.1 Install `coinbase-commerce-node` package
  - [x] 1.2 Add environment variables to `.env.example`
  - [x] 1.3 Create `src/lib/coinbase/index.ts` for SDK initialization
- [x] Task 2: Implement charge creation (AC: #1, #2)
  - [x] 2.1 Create `src/lib/coinbase/commerce.ts`
  - [x] 2.2 Implement `createCharge()` function with proper metadata
  - [x] 2.3 Return chargeId and hostedUrl
  - [x] 2.4 Write unit tests for createCharge
- [x] Task 3: Implement Coinbase webhook handler (AC: #3, #4, #5)
  - [x] 3.1 Create `src/app/api/webhooks/coinbase/route.ts`
  - [x] 3.2 Implement webhook signature verification
  - [x] 3.3 Handle `charge:confirmed` event
  - [x] 3.4 Call `createDonation()` on confirmed charge (from Story 11-2)
  - [x] 3.5 Add idempotency check (donations table has unique index)
  - [x] 3.6 Write unit tests for webhook handler
- [x] Task 4: Run tests and verify (AC: all)
  - [x] 4.1 Run all unit tests
  - [x] 4.2 Verify TypeScript compilation

## Dev Notes

### Critical Constraint
> **ALL CODE CHANGES GO TO `develop` BRANCH ONLY!**

### Previous Story Context (Story 11-3)

Story 11-3 implemented Stripe webhook donation tracking. The pattern established:

```typescript
// Pattern from Stripe webhook - use same approach for Coinbase
import { createDonation } from '@/lib/donations'

// On successful payment:
await createDonation({
  orderId: chargeId,
  paymentProvider: 'coinbase',  // Different from 'stripe'
  paymentId: chargeId,
  orderAmount: chargeAmount.toFixed(2),
  donationAmount: (chargeAmount * 0.03).toFixed(2),
  currency: 'EUR',
  project: 'chainsights',
  customerEmail: customerEmail || undefined,
  daoName: metadata?.dao_name || undefined,
})
```

**Key Learnings from Story 11-3:**
- Fail silently on donation tracking errors (don't break payment flow)
- Idempotency handled by unique index on `(payment_provider, payment_id)`
- Calculate 3% server-side, never trust client amounts
- Return 200 to webhook provider even on non-critical errors

### Coinbase Commerce SDK

**Package:** `coinbase-commerce-node` (version ^1.0.4)

**Installation:**
```bash
npm install coinbase-commerce-node
```

**SDK Initialization:**
```typescript
// src/lib/coinbase/index.ts
import coinbase from 'coinbase-commerce-node'

const Client = coinbase.Client
Client.init(process.env.COINBASE_COMMERCE_API_KEY!)

export const Charge = coinbase.resources.Charge
export const Webhook = coinbase.Webhook
```

### File Changes Required

**1. `src/lib/coinbase/index.ts`** - SDK initialization

```typescript
import coinbase from 'coinbase-commerce-node'

const Client = coinbase.Client

// Initialize only if API key is configured
if (process.env.COINBASE_COMMERCE_API_KEY) {
  Client.init(process.env.COINBASE_COMMERCE_API_KEY)
}

export const Charge = coinbase.resources.Charge
export const Webhook = coinbase.Webhook
export const isConfigured = () => !!process.env.COINBASE_COMMERCE_API_KEY
```

**2. `src/lib/coinbase/commerce.ts`** - Charge creation

```typescript
import { Charge, isConfigured } from './index'

export interface CreateChargeParams {
  name: string           // "ChainSights Governance Report"
  description: string    // "Report for [DAO Name]"
  amount: number         // 99.00
  currency: string       // "EUR"
  orderId: string        // Internal order reference
  customerEmail: string
  metadata: {
    order_id: string
    dao_name?: string
    project: string      // "chainsights"
    donation_amount: string // "2.97"
  }
  redirectUrl: string    // Success URL
  cancelUrl: string      // Cancel URL
}

export interface CreateChargeResult {
  chargeId: string
  hostedUrl: string
}

export async function createCharge(params: CreateChargeParams): Promise<CreateChargeResult> {
  if (!isConfigured()) {
    throw new Error('Coinbase Commerce not configured')
  }

  const chargeData = {
    name: params.name,
    description: params.description,
    pricing_type: 'fixed_price',
    local_price: {
      amount: params.amount.toFixed(2),
      currency: params.currency,
    },
    metadata: params.metadata,
    redirect_url: params.redirectUrl,
    cancel_url: params.cancelUrl,
  }

  const charge = await Charge.create(chargeData)

  return {
    chargeId: charge.id,
    hostedUrl: charge.hosted_url,
  }
}
```

**3. `src/app/api/webhooks/coinbase/route.ts`** - Webhook handler

```typescript
import { NextResponse } from 'next/server'
import { headers } from 'next/headers'
import { Webhook } from '@/lib/coinbase'
import { createDonation } from '@/lib/donations'

export const runtime = 'nodejs'

export async function POST(request: Request) {
  const body = await request.text()
  const headersList = await headers()
  const signature = headersList.get('x-cc-webhook-signature')

  const webhookSecret = process.env.COINBASE_COMMERCE_WEBHOOK_SECRET
  if (!webhookSecret) {
    console.error('Coinbase webhook secret not configured')
    return NextResponse.json(
      { error: 'Webhook not configured' },
      { status: 503 }
    )
  }

  if (!signature) {
    return NextResponse.json(
      { error: 'Missing webhook signature' },
      { status: 400 }
    )
  }

  // Verify webhook signature
  let event
  try {
    event = Webhook.verifyEventBody(body, signature, webhookSecret)
  } catch (error) {
    console.error('Coinbase webhook verification failed:', error)
    return NextResponse.json(
      { error: 'Invalid signature' },
      { status: 400 }
    )
  }

  // Handle event
  try {
    switch (event.type) {
      case 'charge:confirmed': {
        await handleChargeConfirmed(event.data)
        break
      }
      case 'charge:failed':
      case 'charge:expired':
        console.log(`Charge ${event.type}:`, event.data.id)
        break
      default:
        console.log(`Unhandled Coinbase event: ${event.type}`)
    }
  } catch (error) {
    console.error('Error handling Coinbase webhook:', error)
    // Still return 200 to prevent retries for non-critical errors
  }

  return NextResponse.json({ received: true })
}

async function handleChargeConfirmed(charge: {
  id: string
  code: string
  pricing: { local: { amount: string; currency: string } }
  metadata?: {
    order_id?: string
    project?: string
    donation_amount?: string
    dao_name?: string
  }
  timeline?: Array<{ status: string; context?: string }>
}) {
  console.log('Coinbase charge confirmed:', charge.id)

  const metadata = charge.metadata

  // Only process chainsights payments
  if (metadata?.project !== 'chainsights') {
    console.log('Not a chainsights payment, skipping donation tracking')
    return
  }

  const donationAmount = metadata?.donation_amount
  if (!donationAmount) {
    console.log('No donation_amount in metadata, skipping')
    return
  }

  // Extract customer email from timeline if available
  let customerEmail: string | undefined
  if (charge.timeline) {
    const confirmedEvent = charge.timeline.find(t => t.status === 'CONFIRMED')
    // Coinbase may include email in context
  }

  try {
    await createDonation({
      orderId: metadata.order_id || charge.id,
      paymentProvider: 'coinbase',
      paymentId: charge.id,
      orderAmount: charge.pricing.local.amount,
      donationAmount: donationAmount,
      currency: charge.pricing.local.currency,
      project: 'chainsights',
      customerEmail: customerEmail,
      daoName: metadata.dao_name || undefined,
    })
    console.log('Donation recorded (Coinbase):', charge.id, 'amount:', donationAmount)
  } catch (error) {
    // Log but don't fail - donation tracking is non-critical
    console.error('Failed to record Coinbase donation:', error)
  }
}
```

### Environment Variables

Add to `.env.example`:

```env
# Coinbase Commerce (for crypto payments)
COINBASE_COMMERCE_API_KEY=
COINBASE_COMMERCE_WEBHOOK_SECRET=
```

**Staging vs Production:**

| Variable | Staging (`develop`) | Production (`main`) |
|----------|---------------------|---------------------|
| `COINBASE_COMMERCE_API_KEY` | Sandbox/Test API key | Production API key |
| `COINBASE_COMMERCE_WEBHOOK_SECRET` | Test webhook secret | Production webhook secret |

### Coinbase Webhook Events

| Event | Description | Action |
|-------|-------------|--------|
| `charge:created` | Charge created, awaiting payment | Log only |
| `charge:pending` | Payment detected, confirming | Log only |
| `charge:confirmed` | Payment confirmed ✅ | Create donation record |
| `charge:failed` | Payment failed | Log only |
| `charge:expired` | Charge expired | Log only |

### Testing Strategy

**Unit Test Scenarios:**

1. `createCharge` tests:
   - Valid params → returns chargeId and hostedUrl
   - Missing API key → throws error
   - API error → propagates error

2. Webhook handler tests:
   - Valid signature, `charge:confirmed` → donation created
   - Invalid signature → 400 response
   - Missing signature → 400 response
   - Non-chainsights project → donation NOT created
   - Missing donation_amount → donation NOT created
   - createDonation error → still returns 200

### Important Design Decisions

1. **Same pattern as Stripe**: Follow the exact same error handling and donation tracking pattern from Story 11-3
2. **Fail silently**: Donation tracking errors should NEVER break the webhook response
3. **Idempotency**: Database unique index handles duplicate webhooks
4. **SDK wrapper**: Create thin wrapper around SDK for testability

### Project Structure

```
src/
├── lib/
│   └── coinbase/
│       ├── index.ts           # SDK initialization (new)
│       └── commerce.ts        # createCharge function (new)
└── app/
    └── api/
        └── webhooks/
            └── coinbase/
                └── route.ts   # Webhook handler (new)
```

### References

- [Source: docs/architecture/crypto-payments-donations-architecture.md#Coinbase Commerce Integration]
- [Source: docs/architecture/crypto-payments-donations-architecture.md#Webhook Handlers]
- [Source: docs/sprint-artifacts/epic-crypto-payments-donations.md#Story 4]
- [Source: src/lib/donations/queries.ts - createDonation function]
- [Source: src/app/api/webhooks/stripe/route.ts - Stripe webhook pattern to follow]

### Dependencies

- **Story 11-1**: `donations` table (DONE ✅)
- **Story 11-2**: `createDonation()` function (DONE ✅)
- **Story 11-3**: Stripe webhook pattern established (DONE ✅)

### Coinbase Commerce Account Setup

**Prerequisites (Mario needs to provide):**
1. Coinbase Commerce account created at commerce.coinbase.com
2. API key generated from Coinbase Commerce dashboard
3. Webhook endpoint registered: `https://chainsights.one/api/webhooks/coinbase` (production) or `https://develop.chainsights.one/api/webhooks/coinbase` (staging)
4. Webhook secret from Coinbase Commerce settings

## Dev Agent Record

### Context Reference

Story created with comprehensive architecture and previous story analysis.

### Agent Model Used

Claude Opus 4.5

### Completion Notes List

1. **SDK Installation**: Installed `coinbase-commerce-node@^1.0.4` (deprecated but functional, as specified in architecture doc)
2. **Type Declarations**: Created custom `.d.ts` file since no `@types/coinbase-commerce-node` exists
3. **SDK Wrapper**: Created thin wrapper in `src/lib/coinbase/index.ts` for testability
4. **createCharge Function**: Implemented with proper metadata support (donation_amount, dao_name, project)
5. **Webhook Handler**: Implemented with signature verification using `Webhook.verifyEventBody()`
6. **Same Pattern as Stripe**: Followed identical error handling and donation tracking pattern from Story 11-3
7. **Tests**: 15 total tests passing (5 for createCharge, 10 for webhook handler)
8. **Idempotency**: Handled by database unique index on `(payment_provider, payment_id)`

### File List

- `src/lib/coinbase/index.ts` (new) - SDK initialization wrapper
- `src/lib/coinbase/commerce.ts` (new) - createCharge function
- `src/lib/coinbase/coinbase-commerce-node.d.ts` (new) - Type declarations
- `src/app/api/webhooks/coinbase/route.ts` (new) - Webhook handler
- `.env.example` (modified) - Added COINBASE_COMMERCE_API_KEY and COINBASE_COMMERCE_WEBHOOK_SECRET
- `tests/unit/lib/coinbase-commerce.test.ts` (new) - 5 tests for createCharge
- `tests/unit/api/coinbase-webhook.test.ts` (new) - 10 tests for webhook handler

## Senior Developer Review (AI)

**Reviewed:** 2026-01-21
**Reviewer:** Claude Opus 4.5 (Adversarial Code Review)
**Outcome:** APPROVED after fixes

### Issues Found and Fixed:

1. **[HIGH] Unused `customerEmail` parameter** - Made optional in `CreateChargeParams` interface since it wasn't being passed to Coinbase API
2. **[MEDIUM] Confusing comment about timeline extraction** - Simplified comment to accurately reflect implementation
3. **[MEDIUM] Missing barrel exports** - Added `createCharge`, `CreateChargeParams`, `CreateChargeResult` exports to `index.ts`
4. **[MEDIUM] Inconsistent error tracking** - Added LogSnag `logError()` call to match Stripe webhook pattern
5. **[MEDIUM] Minimal type declarations** - Expanded `.d.ts` with index signatures and additional Coinbase API fields

### Verification:
- All 15 unit tests passing
- TypeScript compilation clean
- Implementation matches all 6 Acceptance Criteria
- Error handling now consistent with Stripe webhook

## Change Log

- 2026-01-20: Story created by Bob (Scrum Master) with comprehensive context from architecture and previous stories
- 2026-01-20: Implementation completed by Dev Agent (Claude Opus 4.5) - all tests passing, TypeScript clean
- 2026-01-21: Code review completed - 5 issues found and fixed, story approved
