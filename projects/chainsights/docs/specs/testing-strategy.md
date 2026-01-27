# ChainSights Testing Strategy — Technical Specification

**Author:** Murat (Master Test Architect)
**Status:** Ready for Implementation
**Priority:** High — Zero tests = flying blind

---

## Executive Summary

ChainSights has **zero tests**. This spec defines a pragmatic, risk-based testing strategy that focuses on what matters: revenue paths, data integrity, and the report pipeline.

**Core principle:** Test depth scales with business impact. Payment processing gets comprehensive coverage. Quiz page gets smoke tests.

---

## Risk Assessment

### P0 — Critical (Must Test First)

| Component | Risk | Impact if Broken |
|-----------|------|------------------|
| Stripe Checkout | Revenue | No sales |
| Stripe Webhook | Revenue + Data | Orders lost |
| Report Delivery | Revenue | Customers don't get product |
| Admin Auth | Security | Unauthorized access |
| Download Tokens | Security | Report leakage |

### P1 — High (Core Journey)

| Component | Risk | Impact if Broken |
|-----------|------|------------------|
| Snapshot Fetch | Pipeline | Reports can't be generated |
| AI Analysis | Pipeline | Reports incomplete |
| PDF Generation | Pipeline | No deliverable |
| Email Sending | Delivery | Customer doesn't know report is ready |

### P2 — Medium (Secondary)

| Component | Risk | Impact if Broken |
|-----------|------|------------------|
| Share & Save Claims | Growth | Viral loop broken |
| Admin Dashboard | Operations | Manual workarounds needed |
| Report Decline/Regenerate | Quality | Can't fix bad reports |

### P3 — Low (Nice to Have)

| Component | Risk | Impact if Broken |
|-----------|------|------------------|
| Quiz | Marketing | Lead gen affected |
| Static Pages | SEO | Discoverability |
| Guide Page | Support | More support questions |

---

## Framework Selection

### Recommendation: Vitest + Playwright

| Tool | Purpose | Why |
|------|---------|-----|
| **Vitest** | Unit + Integration | Fast, native ESM, works with Next.js |
| **Playwright** | E2E | Best-in-class for Next.js, network interception |

**Not using:**
- Jest (slower, ESM issues)
- Cypress (weaker for API testing)
- Testing Library alone (need E2E for critical paths)

---

## Test Architecture

```
tests/
├── unit/                    # Pure function tests (Vitest)
│   ├── tokens.test.ts
│   ├── snapshot-parser.test.ts
│   └── classification.test.ts
├── integration/             # API route tests (Vitest + supertest)
│   ├── checkout.test.ts
│   ├── webhook.test.ts
│   ├── share-claim.test.ts
│   └── download.test.ts
├── e2e/                     # Full journey tests (Playwright)
│   ├── checkout-flow.spec.ts
│   ├── admin-flow.spec.ts
│   └── share-save.spec.ts
├── fixtures/                # Test data factories
│   ├── orders.ts
│   ├── reports.ts
│   └── mock-snapshot.ts
└── utils/                   # Test helpers
    ├── db.ts               # Test DB setup/teardown
    ├── stripe-mock.ts      # Stripe test helpers
    └── auth.ts             # Admin auth helpers
```

---

## Test Scenarios by Priority

### P0: Payment Flow

#### CHECKOUT-001: Successful Checkout Creates Stripe Session
```typescript
// tests/integration/checkout.test.ts
test('POST /api/checkout creates Stripe session with correct metadata', async () => {
  const response = await request(app)
    .post('/api/checkout')
    .send({
      email: 'test@example.com',
      daoName: 'Test DAO',
      snapshotSpace: 'test.eth',
      tier: 'standard',
    })

  expect(response.status).toBe(200)
  expect(response.body.sessionId).toBeDefined()
  expect(response.body.url).toContain('checkout.stripe.com')
})
```

#### WEBHOOK-001: Stripe Webhook Creates Order
```typescript
// tests/integration/webhook.test.ts
test('checkout.session.completed webhook creates order in database', async () => {
  const event = createStripeEvent('checkout.session.completed', {
    id: 'cs_test_123',
    customer_details: { email: 'buyer@example.com', name: 'Test Buyer' },
    amount_total: 9900,
    metadata: { product: 'governance_report', tier: 'standard' },
  })

  const response = await request(app)
    .post('/api/webhooks/stripe')
    .set('stripe-signature', signEvent(event))
    .send(event)

  expect(response.status).toBe(200)

  const order = await db.query.orders.findFirst({
    where: eq(orders.stripeSessionId, 'cs_test_123'),
  })

  expect(order).toBeDefined()
  expect(order.customerEmail).toBe('buyer@example.com')
  expect(order.status).toBe('paid')
  expect(order.tier).toBe('standard')
})
```

#### WEBHOOK-002: Invalid Signature Rejected
```typescript
test('webhook with invalid signature returns 400', async () => {
  const event = createStripeEvent('checkout.session.completed', {})

  const response = await request(app)
    .post('/api/webhooks/stripe')
    .set('stripe-signature', 'invalid_signature')
    .send(event)

  expect(response.status).toBe(400)
  expect(response.body.error).toContain('Webhook Error')
})
```

---

### P0: Admin Authentication

#### AUTH-001: Valid Password Grants Access
```typescript
// tests/integration/admin-auth.test.ts
test('POST /api/admin/auth with correct password sets cookie', async () => {
  const response = await request(app)
    .post('/api/admin/auth')
    .send({ password: process.env.ADMIN_PASSWORD })

  expect(response.status).toBe(200)
  expect(response.headers['set-cookie']).toBeDefined()
  expect(response.headers['set-cookie'][0]).toContain('admin_session')
})
```

#### AUTH-002: Invalid Password Denied
```typescript
test('POST /api/admin/auth with wrong password returns 401', async () => {
  const response = await request(app)
    .post('/api/admin/auth')
    .send({ password: 'wrong_password' })

  expect(response.status).toBe(401)
})
```

#### AUTH-003: Admin Routes Require Authentication
```typescript
test('GET /api/admin/reports without auth returns 401', async () => {
  const response = await request(app).get('/api/admin/reports')
  expect(response.status).toBe(401)
})
```

---

### P0: Download Security

#### DOWNLOAD-001: Valid Token Returns PDF
```typescript
// tests/integration/download.test.ts
test('GET /api/download/:token returns PDF for valid token', async () => {
  const report = await createTestReport({ pdfToken: 'valid_token_123' })

  const response = await request(app)
    .get('/api/download/valid_token_123')

  expect(response.status).toBe(200)
  expect(response.headers['content-type']).toBe('application/pdf')
  expect(response.headers['content-disposition']).toContain('attachment')
})
```

#### DOWNLOAD-002: Invalid Token Returns 404
```typescript
test('GET /api/download/:token returns 404 for invalid token', async () => {
  const response = await request(app)
    .get('/api/download/nonexistent_token')

  expect(response.status).toBe(404)
})
```

#### DOWNLOAD-003: Expired Token Returns 410
```typescript
test('GET /api/download/:token returns 410 for expired token', async () => {
  const report = await createTestReport({
    pdfToken: 'expired_token',
    tokenExpiresAt: new Date('2020-01-01'), // Past date
  })

  const response = await request(app)
    .get('/api/download/expired_token')

  expect(response.status).toBe(410) // Gone
})
```

---

### P1: Snapshot Integration

#### SNAPSHOT-001: Valid Space Returns Data
```typescript
// tests/integration/snapshot.test.ts
test('fetchGovernanceData returns valid structure for known space', async () => {
  // Use real API with known stable space
  const data = await fetchGovernanceData('ens.eth')

  expect(data.space).toBeDefined()
  expect(data.space.name).toBe('ENS')
  expect(data.proposals.length).toBeGreaterThan(0)
  expect(data.topVoters.length).toBeGreaterThan(0)
})
```

#### SNAPSHOT-002: Invalid Space Returns Error
```typescript
test('validateSnapshotSpace returns error for nonexistent space', async () => {
  const result = await validateSnapshotSpace('definitely-not-a-real-space.eth')

  expect(result.valid).toBe(false)
  expect(result.error).toContain('not found')
})
```

---

### P1: Voter Classification

#### CLASSIFY-001: Whale Detection
```typescript
// tests/unit/classification.test.ts
test('voter with high direct power and low delegated power is WHALE', () => {
  const voter = {
    directPower: 50000,
    delegatedPower: 1000,
  }

  const classification = classifyVoter(voter)
  expect(classification).toBe('whale')
})
```

#### CLASSIFY-002: Delegate Detection
```typescript
test('voter with high delegated power is DELEGATE', () => {
  const voter = {
    directPower: 1000,
    delegatedPower: 50000,
  }

  const classification = classifyVoter(voter)
  expect(classification).toBe('delegate')
})
```

#### CLASSIFY-003: Unknown Source Fallback
```typescript
test('voter without strategy data is TOP_VOTER', () => {
  const voter = {
    directPower: 0,
    delegatedPower: 0,
    totalVp: 50000, // Has VP but no strategy breakdown
  }

  const classification = classifyVoter(voter)
  expect(classification).toBe('top-voter')
})
```

---

### P1: Token Generation

#### TOKEN-001: Tokens Are Unique
```typescript
// tests/unit/tokens.test.ts
test('generateSecureToken returns unique values', () => {
  const tokens = new Set()
  for (let i = 0; i < 1000; i++) {
    tokens.add(generateSecureToken())
  }
  expect(tokens.size).toBe(1000) // All unique
})
```

#### TOKEN-002: Token Expiry Calculation
```typescript
test('generateTokenExpiry returns date 30 days in future', () => {
  const now = new Date()
  const expiry = generateTokenExpiry(30)

  const diff = expiry.getTime() - now.getTime()
  const days = diff / (1000 * 60 * 60 * 24)

  expect(days).toBeCloseTo(30, 0)
})
```

---

### P2: Share & Save Claims

#### SHARE-001: Valid Claim Accepted
```typescript
// tests/integration/share-claim.test.ts
test('POST /api/share-claim creates claim for valid order', async () => {
  const order = await createTestOrder({ customerEmail: 'sharer@example.com' })
  const report = await createTestReport({ orderId: order.id, deliveredAt: new Date() })

  const response = await request(app)
    .post('/api/share-claim')
    .send({
      email: 'sharer@example.com',
      tweetUrl: 'https://x.com/user/status/123456789',
    })

  expect(response.status).toBe(200)
  expect(response.body.success).toBe(true)
})
```

#### SHARE-002: Duplicate Claim Rejected
```typescript
test('POST /api/share-claim rejects duplicate tweet URL', async () => {
  // First claim
  await createShareClaim({ tweetUrl: 'https://x.com/user/status/123' })

  // Attempt duplicate
  const response = await request(app)
    .post('/api/share-claim')
    .send({
      email: 'another@example.com',
      tweetUrl: 'https://x.com/user/status/123',
    })

  expect(response.status).toBe(400)
  expect(response.body.error).toContain('already been submitted')
})
```

#### SHARE-003: Invalid Tweet URL Rejected
```typescript
test('POST /api/share-claim rejects invalid tweet URL', async () => {
  const response = await request(app)
    .post('/api/share-claim')
    .send({
      email: 'test@example.com',
      tweetUrl: 'https://facebook.com/post/123',
    })

  expect(response.status).toBe(400)
  expect(response.body.error).toContain('Invalid tweet URL')
})
```

---

### E2E: Full Checkout Journey (P0)

```typescript
// tests/e2e/checkout-flow.spec.ts
import { test, expect } from '@playwright/test'

test.describe('Checkout Flow', () => {
  test('customer can purchase standard report @p0 @smoke', async ({ page }) => {
    // Go to homepage
    await page.goto('/')

    // Click CTA
    await page.getByRole('button', { name: /get.*report/i }).click()

    // Fill order form
    await page.getByLabel('Email').fill('e2e-test@example.com')
    await page.getByLabel('DAO Name').fill('Test DAO')
    await page.getByLabel('Snapshot Space').fill('ens.eth')

    // Select tier
    await page.getByRole('radio', { name: /standard/i }).check()

    // Intercept Stripe redirect
    const stripePromise = page.waitForURL(/checkout\.stripe\.com/)

    // Submit
    await page.getByRole('button', { name: /checkout/i }).click()

    // Should redirect to Stripe
    await stripePromise
    expect(page.url()).toContain('checkout.stripe.com')
  })
})
```

---

### E2E: Admin Report Management (P1)

```typescript
// tests/e2e/admin-flow.spec.ts
test.describe('Admin Flow', () => {
  test.beforeEach(async ({ page }) => {
    // Login
    await page.goto('/admin')
    await page.getByLabel('Password').fill(process.env.ADMIN_PASSWORD!)
    await page.getByRole('button', { name: 'Login' }).click()
    await expect(page).toHaveURL('/admin')
  })

  test('admin can view pending reports @p1', async ({ page }) => {
    await expect(page.getByText('Reports')).toBeVisible()
    // Should show reports list or empty state
  })

  test('admin can send approved report @p1', async ({ page }) => {
    // Create test report in review_pending state
    const report = await createTestReport({ status: 'review_pending' })

    await page.goto(`/admin/reports/${report.id}`)

    // Click approve and send
    await page.getByRole('button', { name: /approve.*send/i }).click()

    // Confirm
    await expect(page.getByText(/sent successfully/i)).toBeVisible()
  })
})
```

---

## Implementation Plan

### Phase 1: Foundation (Day 1)

1. Install Vitest + Playwright
2. Create test config files
3. Set up test database (separate from dev)
4. Create fixture factories

```bash
npm install -D vitest @vitest/coverage-v8 playwright @playwright/test
```

### Phase 2: P0 Tests (Day 1-2)

1. Webhook signature validation
2. Order creation from webhook
3. Admin auth tests
4. Download token security

### Phase 3: P1 Tests (Day 2-3)

1. Snapshot API integration
2. Voter classification logic
3. Token generation
4. Pipeline status transitions

### Phase 4: E2E Setup (Day 3)

1. Playwright config
2. Checkout flow test
3. Admin flow test

### Phase 5: CI Integration (Day 4)

1. GitHub Actions workflow
2. Test database provisioning
3. Parallel execution
4. Coverage reporting

---

## Test Data Strategy

### Fixtures (Factories)

```typescript
// tests/fixtures/orders.ts
export function createTestOrder(overrides: Partial<NewOrder> = {}): NewOrder {
  return {
    stripeSessionId: `test_${randomId()}`,
    customerEmail: 'test@example.com',
    customerName: 'Test Customer',
    tier: 'standard',
    status: 'paid',
    amountCents: 9900,
    currency: 'eur',
    ...overrides,
  }
}
```

### Mock External Services

| Service | Strategy |
|---------|----------|
| Stripe | Use test mode + stripe-mock for webhooks |
| Snapshot API | Record real responses, replay in tests |
| Claude AI | Mock with canned responses |
| Resend Email | Mock, verify payload |
| QStash | Mock, verify job enqueue |

---

## Coverage Targets

| Level | P0 Coverage | P1 Coverage | P2 Coverage |
|-------|-------------|-------------|-------------|
| Unit | 90%+ | 80%+ | 60%+ |
| Integration | 100% critical paths | 80% | 50% |
| E2E | All revenue paths | Main happy paths | Smoke only |

---

## CI Pipeline

```yaml
# .github/workflows/test.yml
name: Tests

on: [push, pull_request]

jobs:
  unit-integration:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
      - run: npm ci
      - run: npm run test:unit
      - run: npm run test:integration

  e2e:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
      - run: npm ci
      - run: npx playwright install --with-deps
      - run: npm run test:e2e
```

---

## Quality Gates

### Pre-commit
- Unit tests pass
- TypeScript compiles
- Lint clean

### Pre-merge
- All P0 tests pass
- All P1 tests pass
- Coverage thresholds met

### Pre-deploy
- Full test suite green
- E2E smoke tests pass
- No security vulnerabilities

---

## Files to Create

| File | Purpose |
|------|---------|
| `vitest.config.ts` | Vitest configuration |
| `playwright.config.ts` | Playwright configuration |
| `tests/setup.ts` | Global test setup |
| `tests/fixtures/*.ts` | Data factories |
| `tests/utils/db.ts` | Test DB helpers |
| `tests/utils/stripe-mock.ts` | Stripe mocking |

---

## Success Criteria

After implementation:
1. P0 tests run in <30 seconds
2. Full suite runs in <5 minutes
3. CI fails fast on critical path failures
4. Coverage visible in PR reviews
5. Zero flaky tests (or quarantined)

---

*"Risk-based testing means we protect what matters. Payment flow gets fortress-level coverage. Quiz page gets a smoke test. That's not cutting corners — that's smart allocation."*

— Murat
