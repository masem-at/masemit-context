# Auth & Dashboard Architecture Spec

**Created:** 2026-01-19
**Author:** Winston (Architect)
**Status:** Approved in Party Mode
**Related:** `docs/ux/dashboard-ux-spec.md`, `docs/future/big-bang-launch-plan.md`

---

## Overview

Technical architecture for user authentication, entitlements, scheduled generation, and subscription billing to support the Big Bang Launch features.

**Design Principles:**
- Boring technology that works
- NeonDB for everything (no new databases)
- JWT in httpOnly cookies (secure, simple)
- QStash for job queues (Vercel-native)
- Resend for emails (already integrated)
- Stripe for payments (already integrated)

---

## 1. Database Schema

### Users Table (New)

```sql
CREATE TABLE users (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  email TEXT UNIQUE NOT NULL,
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW(),

  -- Entitlements (denormalized for speed)
  tier TEXT NOT NULL DEFAULT 'free', -- 'free', 'single', 'lifetime', 'lifetime_plus', 'pro', 'team'
  billing_model TEXT NOT NULL DEFAULT 'none', -- 'none', 'one_time', 'subscription'
  max_years INTEGER NOT NULL DEFAULT 1, -- 1, 3, or 5
  has_hr_domain BOOLEAN NOT NULL DEFAULT false,
  generations_remaining INTEGER, -- NULL = unlimited, number = limited

  -- Stripe
  stripe_customer_id TEXT,
  stripe_subscription_id TEXT,
  subscription_status TEXT, -- 'active', 'canceled', 'past_due'
  subscription_ends_at TIMESTAMPTZ,

  -- Metadata
  last_login_at TIMESTAMPTZ,
  login_count INTEGER DEFAULT 0
);
```

### Magic Links Table (New)

```sql
CREATE TABLE magic_links (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  email TEXT NOT NULL,
  token TEXT UNIQUE NOT NULL,
  created_at TIMESTAMPTZ DEFAULT NOW(),
  expires_at TIMESTAMPTZ NOT NULL, -- created_at + 1 hour
  used_at TIMESTAMPTZ,

  -- Context
  redirect_to TEXT DEFAULT '/dashboard',
  created_for TEXT -- 'login', 'purchase_complete'
);
```

### User Scenarios Junction (New)

```sql
CREATE TABLE user_scenarios (
  user_id UUID REFERENCES users(id),
  scenario_id UUID REFERENCES scenarios(id),
  created_at TIMESTAMPTZ DEFAULT NOW(),
  PRIMARY KEY (user_id, scenario_id)
);
```

### Scenarios Table Additions

```sql
ALTER TABLE scenarios ADD COLUMN status TEXT DEFAULT 'completed';
-- Values: 'queued', 'generating', 'completed', 'failed'

ALTER TABLE scenarios ADD COLUMN progress INTEGER DEFAULT 100;
-- 0-100 percentage

ALTER TABLE scenarios ADD COLUMN queued_at TIMESTAMPTZ;
ALTER TABLE scenarios ADD COLUMN started_at TIMESTAMPTZ;
ALTER TABLE scenarios ADD COLUMN completed_at TIMESTAMPTZ;
ALTER TABLE scenarios ADD COLUMN error_message TEXT;
```

### Indexes

```sql
CREATE INDEX idx_magic_links_token ON magic_links(token);
CREATE INDEX idx_magic_links_email ON magic_links(email);
CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_users_stripe_customer ON users(stripe_customer_id);
```

---

## 2. Auth Flow (Magic Link)

### Purchase Flow

```
1. User clicks "Buy €99 Lifetime"
2. Stripe Checkout (collects email)
3. Stripe webhook: checkout.session.completed
4. Backend:
   a. Create/update user with tier='lifetime'
   b. Generate magic link token (crypto.randomUUID)
   c. Store in magic_links table (expires: NOW + 1 hour)
   d. Send email via Resend with link: /auth/verify?token={token}
5. User clicks link in email
6. Backend /auth/verify:
   a. Validate token exists and not expired
   b. Mark token as used (used_at = NOW)
   c. Create JWT session cookie (httpOnly, secure, 7 days)
   d. Redirect to /dashboard
```

### Login Flow (Returning User)

```
1. User visits /login, enters email
2. Backend:
   a. Check user exists
   b. Generate magic link token
   c. Send email via Resend
3. User clicks link → same verify flow
```

### JWT Session

- **Storage:** httpOnly cookie (secure, sameSite: lax)
- **Payload:** `{ userId, email, tier, exp }`
- **Expiry:** 7 days
- **Refresh:** On each authenticated request, extend if < 1 day remaining

### Magic Link Email Note

**IMPORTANT:** Email must clearly state "This link expires in 1 hour" per Mario's requirement.

---

## 3. Entitlements Model

### TypeScript Implementation

```typescript
// lib/auth/entitlements.ts

export type Tier = 'free' | 'single' | 'lifetime' | 'lifetime_plus' | 'pro' | 'team';

export interface UserEntitlements {
  tier: Tier;
  billingModel: 'none' | 'one_time' | 'subscription';
  maxYears: 1 | 3 | 5;
  hasHrDomain: boolean;
  generationsRemaining: number | 'unlimited';
  isSubscriptionActive: boolean;
}

export const TIER_ENTITLEMENTS: Record<Tier, Omit<UserEntitlements, 'isSubscriptionActive'>> = {
  free: {
    tier: 'free',
    billingModel: 'none',
    maxYears: 1,
    hasHrDomain: false,
    generationsRemaining: 0, // Can only view examples
  },
  single: {
    tier: 'single',
    billingModel: 'one_time',
    maxYears: 1,
    hasHrDomain: false,
    generationsRemaining: 1,
  },
  lifetime: {
    tier: 'lifetime',
    billingModel: 'one_time',
    maxYears: 3,
    hasHrDomain: false,
    generationsRemaining: 'unlimited',
  },
  lifetime_plus: {
    tier: 'lifetime_plus',
    billingModel: 'one_time',
    maxYears: 5,
    hasHrDomain: true,
    generationsRemaining: 'unlimited',
  },
  pro: {
    tier: 'pro',
    billingModel: 'subscription',
    maxYears: 5,
    hasHrDomain: true,
    generationsRemaining: 'unlimited',
  },
  team: {
    tier: 'team',
    billingModel: 'subscription',
    maxYears: 5,
    hasHrDomain: true,
    generationsRemaining: 'unlimited',
  },
};
```

### Helper Functions

```typescript
export function canGenerate(user: UserEntitlements): boolean {
  if (user.generationsRemaining === 'unlimited') return true;
  if (user.generationsRemaining > 0) return true;
  return false;
}

export function canAccessYears(user: UserEntitlements, years: number): boolean {
  return years <= user.maxYears;
}

export function canAccessHr(user: UserEntitlements): boolean {
  return user.hasHrDomain;
}
```

---

## 4. Scheduled Generation (QStash)

### Flow

```
1. User submits generation request
2. API /api/generate:
   a. Validate entitlements
   b. Create scenario record (status: 'queued')
   c. Link to user (user_scenarios table)
   d. Push job to QStash:
      POST https://qstash.upstash.io/v2/publish
      Headers: Authorization: Bearer {QSTASH_TOKEN}
      Body: { scenarioId, userId, size, sector, years }
      URL: https://tellingcube.com/api/jobs/generate
   e. Return { scenarioId, status: 'queued' }

3. QStash calls /api/jobs/generate (async, retries on failure):
   a. Verify QStash signature
   b. Update scenario status: 'generating'
   c. Run generation (existing logic)
   d. Update scenario status: 'completed'
   e. Send completion email via Resend

4. Frontend polling:
   GET /api/scenarios/{id}/status
   Returns: { status, progress, estimatedTimeRemaining }
```

### Job Payload

```typescript
interface GenerationJob {
  scenarioId: string;
  userId: string;
  size: 'startup' | 'midcap' | 'largecap';
  sector: 'consumer_staples' | 'industrials' | 'financials';
  years: 1 | 3 | 5;
}
```

---

## 5. Stripe Webhook Handling

```typescript
// app/api/webhooks/stripe/route.ts

export async function POST(req: Request) {
  const sig = req.headers.get('stripe-signature');
  const body = await req.text();

  const event = stripe.webhooks.constructEvent(body, sig, STRIPE_WEBHOOK_SECRET);

  switch (event.type) {
    // ONE-TIME PURCHASES
    case 'checkout.session.completed': {
      const session = event.data.object;
      const email = session.customer_details.email;
      const tier = session.metadata.tier; // 'single', 'lifetime', 'lifetime_plus'

      // Create/update user
      await upsertUser(email, {
        tier,
        billingModel: 'one_time',
        stripeCustomerId: session.customer,
      });

      // Send magic link email
      await sendMagicLinkEmail(email, '/dashboard');
      break;
    }

    // SUBSCRIPTIONS
    case 'customer.subscription.created':
    case 'customer.subscription.updated': {
      const subscription = event.data.object;
      const customerId = subscription.customer;
      const tier = subscription.metadata.tier; // 'pro', 'team'

      await updateUserByStripeCustomer(customerId, {
        tier,
        billingModel: 'subscription',
        stripeSubscriptionId: subscription.id,
        subscriptionStatus: subscription.status,
        subscriptionEndsAt: new Date(subscription.current_period_end * 1000),
      });
      break;
    }

    case 'customer.subscription.deleted': {
      const subscription = event.data.object;
      await updateUserByStripeCustomer(subscription.customer, {
        tier: 'free',
        subscriptionStatus: 'canceled',
      });
      break;
    }

    case 'invoice.payment_failed': {
      // Send email, update status to 'past_due'
      break;
    }
  }

  return new Response('OK');
}
```

---

## 6. API Endpoints

### Auth

| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/api/auth/request-link` | Request magic link (email input) |
| GET | `/api/auth/verify` | Verify magic link token, set session |
| POST | `/api/auth/logout` | Clear session cookie |
| GET | `/api/auth/me` | Get current user & entitlements |

### Dashboard

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/api/dashboard` | User stats + recent scenarios |
| GET | `/api/dashboard/scenarios` | Paginated scenario list |

### Generation

| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/api/generate` | Queue new generation |
| GET | `/api/scenarios/:id/status` | Poll generation status |

### Webhooks

| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/api/webhooks/stripe` | Stripe events |
| POST | `/api/jobs/generate` | QStash callback (internal) |

---

## 7. Environment Variables

```env
# Auth
JWT_SECRET=xxx                    # For signing JWTs

# QStash (Upstash)
QSTASH_TOKEN=xxx                  # For publishing jobs
QSTASH_CURRENT_SIGNING_KEY=xxx   # For verifying callbacks
QSTASH_NEXT_SIGNING_KEY=xxx

# Stripe Products (new)
STRIPE_PRICE_SINGLE=price_xxx         # €9
STRIPE_PRICE_LIFETIME=price_xxx       # €99
STRIPE_PRICE_LIFETIME_PLUS=price_xxx  # €299
STRIPE_PRICE_PRO_MONTHLY=price_xxx    # €19/mo
STRIPE_PRICE_TEAM_MONTHLY=price_xxx   # €49/mo
```

---

## 8. Security Considerations

1. **Magic Links:** 1-hour expiry, single-use, secure random tokens
2. **JWT:** httpOnly cookie prevents XSS, secure flag for HTTPS only
3. **QStash:** Signature verification on all callbacks
4. **Stripe:** Webhook signature verification
5. **Rate Limiting:** Add to /api/auth/request-link (5 requests per email per hour)

---

## 9. Migration Path

### For Existing Users (3 Early Supporters)

Per party mode decision: All existing customers get **Lifetime+** tier automatically.

```sql
-- Run after deploying users table
INSERT INTO users (email, tier, billing_model, max_years, has_hr_domain, generations_remaining, stripe_customer_id)
SELECT
  customer_email,
  'lifetime_plus',
  'one_time',
  5,
  true,
  NULL, -- unlimited
  stripe_customer_id
FROM invoices
WHERE status = 'paid';
```

---

*Architecture spec created by Winston (Architect) during Party Mode session 2026-01-19*
