# Story 11.5: Crypto Checkout API Endpoint

Status: done

## Story

**As a** frontend
**I want** an API endpoint to initiate crypto checkout
**So that** I can redirect users to Coinbase Commerce for crypto payment

## Acceptance Criteria

1. - [x] `POST /api/crypto-checkout` creates charge and returns redirect URL
2. - [x] Request validates required fields (orderId, orderTotal, customerEmail, daoName)
3. - [x] Response includes `chargeId` and `checkoutUrl`
4. - [x] Error handling for Coinbase API failures
5. - [x] Error handling for missing/invalid configuration
6. - [x] Consistent response structure with existing `/api/checkout` endpoint

## Tasks / Subtasks

- [x] Task 1: Create crypto-checkout route handler (AC: #1, #3)
  - [x] 1.1 Create `src/app/api/crypto-checkout/route.ts`
  - [x] 1.2 Implement POST handler with request body parsing
  - [x] 1.3 Call `createCharge()` from `@/lib/coinbase`
  - [x] 1.4 Return `{ chargeId, checkoutUrl }` response
- [x] Task 2: Implement request validation (AC: #2)
  - [x] 2.1 Validate required fields present
  - [x] 2.2 Validate orderTotal is positive number
  - [x] 2.3 Return 400 with descriptive error for invalid requests
- [x] Task 3: Implement error handling (AC: #4, #5)
  - [x] 3.1 Handle Coinbase not configured (503)
  - [x] 3.2 Handle Coinbase API errors (500 with logged details)
  - [x] 3.3 Add LogSnag error tracking for failures
- [x] Task 4: Write unit tests (AC: all)
  - [x] 4.1 Test successful charge creation
  - [x] 4.2 Test validation errors (missing fields)
  - [x] 4.3 Test Coinbase not configured error
  - [x] 4.4 Test Coinbase API error propagation

## Dev Notes

### 🚨 CRITICAL CONSTRAINT

> **ALL CODE CHANGES GO TO `develop` BRANCH ONLY!**

### API Contract (from Architecture Doc)

```typescript
// Request
POST /api/crypto-checkout
{
  orderId: string;      // Internal order reference
  orderTotal: number;   // Price in EUR (e.g., 99.00)
  customerEmail: string;
  daoName: string;
}

// Response (Success - 200)
{
  chargeId: string;     // Coinbase charge ID
  checkoutUrl: string;  // Coinbase hosted checkout URL for redirect
}

// Response (Error - 400)
{
  error: string;        // Descriptive validation error
}

// Response (Error - 503)
{
  error: string;        // "Coinbase Commerce not configured"
}

// Response (Error - 500)
{
  error: string;        // "Failed to create crypto checkout"
}
```

### Dependencies Already Implemented (Story 11-4)

The `createCharge()` function is already available from `@/lib/coinbase`:

```typescript
import { createCharge, isConfigured } from '@/lib/coinbase'

// CreateChargeParams interface:
{
  name: string           // "ChainSights Governance Report"
  description: string    // "Report for [DAO Name]"
  amount: number         // 99.00
  currency: string       // "EUR"
  orderId: string        // Internal order reference
  customerEmail?: string // Optional
  metadata: {
    order_id: string
    dao_name?: string
    project: string      // "chainsights"
    donation_amount: string // "2.97"
  }
  redirectUrl: string    // Success URL
  cancelUrl: string      // Cancel URL
}

// Returns: { chargeId: string, hostedUrl: string }
```

### Pattern Reference: Existing Stripe Checkout

Follow the pattern from `src/app/api/checkout/route.ts`:

```typescript
// Key patterns to replicate:
// 1. Check if payment provider is configured (return 503 if not)
// 2. Parse request body with try/catch
// 3. Calculate donation amount: (orderTotal * 0.03).toFixed(2)
// 4. Use NEXT_PUBLIC_BASE_URL for redirect URLs
// 5. Return JSON response with consistent structure
// 6. Log errors to console before returning
```

### Donation Amount Calculation

Same as Stripe - 3% of order total:

```typescript
const donationAmount = (orderTotal * 0.03).toFixed(2)
```

### Redirect URLs

```typescript
const baseUrl = process.env.NEXT_PUBLIC_BASE_URL || 'http://localhost:3000'
const redirectUrl = `${baseUrl}/success?provider=coinbase&charge_id={CHARGE_ID}`
const cancelUrl = `${baseUrl}/#pricing`
```

**Note:** Coinbase redirects don't support template variables like Stripe. You'll need to either:
1. Append the chargeId to the redirect URL before creating the charge (circular dependency - won't work)
2. Use a generic success URL and look up the charge via query param later
3. Just use `/success?provider=coinbase` and handle lookup differently

**Recommended:** Use simple success URL: `${baseUrl}/success?provider=coinbase`

### TIERS Reference

Use the existing TIERS from `@/lib/stripe` for pricing info if needed, or accept orderTotal directly from request (preferred for flexibility).

### File Structure

```
src/app/api/crypto-checkout/
└── route.ts              # POST handler
tests/unit/api/
└── crypto-checkout.test.ts  # Unit tests
```

### Test Mocking Pattern (from Story 11-4)

```typescript
// Mock the coinbase module
vi.mock('@/lib/coinbase', () => ({
  createCharge: vi.fn(),
  isConfigured: vi.fn(),
}))

// In tests:
import { createCharge, isConfigured } from '@/lib/coinbase'
vi.mocked(isConfigured).mockReturnValue(true)
vi.mocked(createCharge).mockResolvedValue({
  chargeId: 'charge_test_123',
  hostedUrl: 'https://commerce.coinbase.com/charges/charge_test_123',
})
```

### Project Structure Notes

- Route file goes in `src/app/api/crypto-checkout/route.ts`
- Test file goes in `tests/unit/api/crypto-checkout.test.ts`
- Follow Next.js 14 App Router conventions
- Use `NextResponse.json()` for responses

### References

- [Architecture Doc: API Endpoints Section](../architecture/crypto-payments-donations-architecture.md#api-endpoints)
- [Existing Stripe Checkout](../../src/app/api/checkout/route.ts)
- [Coinbase Commerce Library](../../src/lib/coinbase/commerce.ts)
- [Story 11-4 Completion Notes](./11-4-coinbase-commerce-integration.md#completion-notes-list)

## Dev Agent Record

### Context Reference

Story created with comprehensive architecture analysis and previous story learnings.

### Agent Model Used

Claude Opus 4.5

### Debug Log References

### Completion Notes List

1. **Route Implementation**: Created `POST /api/crypto-checkout` endpoint following existing `/api/checkout` patterns
2. **Validation**: All required fields validated with descriptive error messages (400 responses)
3. **Configuration Check**: Returns 503 if Coinbase not configured (uses `isConfigured()` from lib)
4. **Error Handling**: Coinbase API errors logged to LogSnag and return 500 with generic message
5. **Donation Calculation**: Uses same 3% formula as Stripe checkout: `(orderTotal * 0.03).toFixed(2)`
6. **Tests**: 14 unit tests covering success, validation errors, configuration errors, and API errors

### File List

- `src/app/api/crypto-checkout/route.ts` (new) - POST handler for crypto checkout
- `tests/unit/api/crypto-checkout.test.ts` (new) - 14 unit tests

## Senior Developer Review (AI)

**Reviewed:** 2026-01-21
**Reviewer:** Claude Opus 4.5 (Adversarial Code Review)
**Outcome:** APPROVED after fixes

### Issues Found and Fixed:

1. **[MEDIUM] No email format validation** - Added basic email regex validation before processing
2. **[MEDIUM] Missing rate limiting consideration** - Documented rate limiting strategy in file header
3. **[LOW] Inconsistent error message casing** - Standardized to "Invalid value: description" pattern
4. **[LOW] Test imports pattern** - No change needed (pattern was correct)
5. **[LOW] No test for zero orderTotal** - Added test case
6. **[LOW] Missing test for string orderTotal** - Added test case for type coercion

### Verification:
- All 14 unit tests passing (3 new tests added)
- TypeScript compilation clean
- Implementation matches all 6 Acceptance Criteria
- Email validation prevents invalid formats reaching Coinbase API

## Change Log

- 2026-01-21: Story created by Bob (Scrum Master) with comprehensive context from architecture, epic, and Story 11-4 learnings
- 2026-01-21: Implementation completed by Dev Agent (Claude Opus 4.5) - all 11 tests passing
- 2026-01-21: Code review completed - 6 issues found, 5 fixed (1 was already correct), story approved
