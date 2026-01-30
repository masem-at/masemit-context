# Story 11.8: Confirmation Page — Donation Message

Status: done

## Story

**As a** customer
**I want** to see my donation impact after payment
**So that** I feel good about my purchase

## Acceptance Criteria

1. - [x] Donation impact box shows on confirmation page
2. - [x] Displays actual donation amount (€X.XX)
3. - [x] Shows emotional message "You just helped a family"
4. - [x] Works for both Stripe and Coinbase payments
5. - [x] Links to hoki.help
6. - [x] Accessible: screen reader support, color contrast

## Tasks / Subtasks

- [x] Task 1: Create DonationImpactBox component (AC: #1, #2, #3, #5, #6)
  - [x] 1.1 Create `src/components/DonationImpactBox.tsx`
  - [x] 1.2 Accept `orderTotal` prop (number)
  - [x] 1.3 Calculate donation amount: `orderTotal * 0.03`
  - [x] 1.4 Display headline "You just helped a family"
  - [x] 1.5 Display "€[amount] of your purchase supports children's hospice care via hoki.help"
  - [x] 1.6 Format amount to 2 decimal places (€2.97, not €2.9700)
  - [x] 1.7 Link hoki.help with proper target/rel attributes
  - [x] 1.8 Add aria-label for screen readers
- [x] Task 2: Integrate with success page for Stripe (AC: #1, #4)
  - [x] 2.1 Import DonationImpactBox in `src/app/success/page.tsx`
  - [x] 2.2 Extract order total from Stripe session via API (see Dev Notes)
  - [x] 2.3 Pass orderTotal to DonationImpactBox
  - [x] 2.4 Position between success message and intake form
- [x] Task 3: Handle Coinbase success flow (AC: #4)
  - [x] 3.1 Parse `provider` query param to detect Coinbase
  - [x] 3.2 Parse `charge_id` query param for Coinbase
  - [x] 3.3 Create API call to get order total from charge (if needed)
  - [x] 3.4 Or pass order total via query param from redirect
- [x] Task 4: Write unit tests (AC: all)
  - [x] 4.1 Test component renders with correct donation amount
  - [x] 4.2 Test 3% calculation is correct (€99 → €2.97, €199 → €5.97)
  - [x] 4.3 Test hoki.help link has correct href and target
  - [x] 4.4 Test aria-label is present
  - [x] 4.5 Test component handles edge cases (0, undefined)

## Dev Notes

### CRITICAL CONSTRAINT

> **ALL CODE CHANGES GO TO `develop` BRANCH ONLY!**

### UX Spec Reference

From `docs/ux/checkout-crypto-donations-ux-spec.md`:

**Donation Impact Box Wireframe:**
```
┌─────────────────────────────────────────────────────────┐
│                                                         │
│  💙 You just helped a family.                           │
│                                                         │
│  €2.97 of your purchase supports children's             │
│  hospice care via hoki.help                             │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

**Copy:**
- Headline: "💙 You just helped a family."
- Body: "€[amount] of your purchase supports children's hospice care via hoki.help"

### Component Props Interface

```typescript
interface DonationImpactBoxProps {
  orderTotal: number  // e.g., 99 or 199
  className?: string  // optional styling override
}
```

### Donation Calculation

```typescript
const DONATION_PERCENTAGE = 0.03  // 3%

function calculateDonation(orderTotal: number): number {
  return orderTotal * DONATION_PERCENTAGE
}

// €99.00 * 0.03 = €2.97
// €199.00 * 0.03 = €5.97
```

### Getting Order Total

**For Stripe:**
The current success page receives `session_id` query param. We have two options:

**Option A (Simpler):** Pass amount via query param from checkout redirect
- Modify Stripe checkout success URL to include amount: `/success?session_id=xxx&amount=99`
- Parse amount from query param in success page

**Option B (More Secure):** Fetch from Stripe API
- Call Stripe API to get session details using session_id
- Extract `amount_total` from session
- Requires server component or API route

**Recommendation:** Option A is simpler for MVP. The amount is not sensitive data and already displayed to customer during checkout.

**For Coinbase:**
Based on UX spec, Coinbase redirects to `/checkout/success?provider=coinbase&charge_id=xxx`
- The current success page is at `/success` - need to handle both paths
- Or use unified path: `/success?provider=stripe&session_id=xxx` vs `/success?provider=coinbase&charge_id=xxx&amount=99`

**Simplest Implementation:**
1. Always pass `amount` as query param from checkout redirect
2. Both Stripe and Coinbase checkout flows add `amount=XX` to success URL
3. Success page reads `amount` param and displays DonationImpactBox

### Current Success Page Structure

```tsx
// src/app/success/page.tsx
function SuccessContent({ searchParams }) {
  const sessionId = searchParams.session_id || 'unknown'

  return (
    <main>
      {/* Success header with CheckCircle icon */}
      <h1>Payment Successful!</h1>
      <p>Thank you for your purchase...</p>

      {/* Intake form */}
      <IntakeForm sessionId={sessionId} />

      {/* Guide promo section */}
      {/* Footer */}
    </main>
  )
}
```

**Placement:** DonationImpactBox should go BETWEEN the success header and the intake form.

### Integration Points

**Files to modify:**
- `src/app/success/page.tsx` - Add DonationImpactBox component
- `src/app/api/checkout/route.ts` - Add amount to success URL (if not already)

**Files to create:**
- `src/components/DonationImpactBox.tsx` - New component
- `tests/unit/components/donation-impact-box.test.tsx` - Unit tests

### Styling Guidelines

Follow existing patterns from DonationBadge and CryptoCheckoutModal:
- Background: `bg-aqua/5` or `bg-navy-dark/50` for subtle highlight
- Border: `border border-aqua/20` for accent
- Text: `text-gray-400` for secondary, `text-white` for primary
- Emoji: Blue heart `💙` (use `text-blue-400` for emoji wrapper if needed)
- Link: `text-aqua hover:underline` for hoki.help link
- Padding: `p-6` for box content

### Existing Pattern Reference

From `DonationBadge.tsx`:
```tsx
<div className="text-sm text-gray-400 text-center p-3 bg-navy-dark/50 rounded-lg border border-aqua/10">
  <span className="text-blue-400">💙</span>{' '}
  3% of your purchase helps families facing childhood illness via{' '}
  <a href="https://hoki.help" target="_blank" rel="noopener noreferrer" className="text-aqua hover:underline">
    hoki.help
  </a>
</div>
```

The DonationImpactBox should be similar but:
- Larger (more padding, bigger text)
- Show actual amount instead of percentage
- More emotional headline

### Previous Story Learnings (Story 11-7)

1. **Use `&` not `&amp;`:** In JSX strings, use `&` for ampersand
2. **Named constants:** Extract magic values like DONATION_PERCENTAGE to constants
3. **Aria-labels:** Add accessibility labels to interactive elements
4. **Analytics tracking:** Consider adding `trackEvent('donation_shown')` (optional for this story)
5. **Network vs API errors:** Distinguish error types when applicable
6. **Test for aria-labels:** Include test coverage for accessibility attributes

### Edge Cases to Handle

1. **Missing amount param:** Show nothing or show with €0.00 (graceful degradation)
2. **Invalid amount:** If amount is NaN, don't render the box
3. **Negative amount:** Should not happen, but handle gracefully

### Testing Patterns

```typescript
// tests/unit/components/donation-impact-box.test.tsx
import { render, screen } from '@testing-library/react'
import { describe, it, expect } from 'vitest'
import { DonationImpactBox } from '@/components/DonationImpactBox'

describe('DonationImpactBox', () => {
  it('displays correct donation amount for €99 order', () => {
    render(<DonationImpactBox orderTotal={99} />)
    expect(screen.getByText(/€2\.97/)).toBeInTheDocument()
  })

  it('displays correct donation amount for €199 order', () => {
    render(<DonationImpactBox orderTotal={199} />)
    expect(screen.getByText(/€5\.97/)).toBeInTheDocument()
  })

  it('displays emotional headline', () => {
    render(<DonationImpactBox orderTotal={99} />)
    expect(screen.getByText(/You just helped a family/i)).toBeInTheDocument()
  })

  it('links to hoki.help with correct attributes', () => {
    render(<DonationImpactBox orderTotal={99} />)
    const link = screen.getByRole('link', { name: /hoki\.help/i })
    expect(link).toHaveAttribute('href', 'https://hoki.help')
    expect(link).toHaveAttribute('target', '_blank')
    expect(link).toHaveAttribute('rel', 'noopener noreferrer')
  })

  it('has accessible role or aria-label', () => {
    render(<DonationImpactBox orderTotal={99} />)
    expect(screen.getByRole('region') || screen.getByLabelText(/donation/i)).toBeInTheDocument()
  })

  it('does not render for zero amount', () => {
    const { container } = render(<DonationImpactBox orderTotal={0} />)
    expect(container.firstChild).toBeNull()
  })
})
```

### Dependencies

- **Story 11-6:** `DonationBadge` component - reference for styling pattern (DONE)
- **Story 11-7:** `CryptoCheckoutModal` - reference for styling pattern (DONE)
- **shadcn/ui:** No new components needed
- **Existing:** `src/app/success/page.tsx` - target for integration

### References

- [UX Spec: Confirmation Page](../ux/checkout-crypto-donations-ux-spec.md#3-confirmation-page-unified)
- [Architecture: DonationImpactBox](../architecture/crypto-payments-donations-architecture.md)
- [Epic: Story 8](./epic-crypto-payments-donations.md#story-8-confirmation-page--donation-message)
- [DonationBadge Pattern](../../src/components/DonationBadge.tsx)

## Dev Agent Record

### Context Reference

Story created by create-story workflow with comprehensive context from:
- Epic file (docs/sprint-artifacts/epic-crypto-payments-donations.md)
- Architecture doc (docs/architecture/crypto-payments-donations-architecture.md)
- UX spec (docs/ux/checkout-crypto-donations-ux-spec.md)
- Previous story (11-7) completion notes and learnings
- Existing success page analysis (src/app/success/page.tsx)
- DonationBadge component reference (src/components/DonationBadge.tsx)

### Agent Model Used

Claude Opus 4.5

### Debug Log References

### Completion Notes List

1. **DonationImpactBox Component**: Created new component at `src/components/DonationImpactBox.tsx` with 3% donation calculation, emotional headline, hoki.help link, and accessibility support (aria-label, role="region")
2. **Success Page Integration**: Updated `src/app/success/page.tsx` to display DonationImpactBox between success header and intake form
3. **Stripe Checkout Update**: Modified `src/app/api/checkout/route.ts` to include `amount` in success URL for donation display
4. **Coinbase Checkout Update**: Modified `src/app/api/crypto-checkout/route.ts` to include `amount` in redirect URL for donation display
5. **Unit Tests**: Created 17 unit tests covering donation calculation, headline display, link attributes, accessibility, and edge cases (zero, negative, NaN)
6. **Analytics Fix (pre-existing)**: Fixed ESLint error in CryptoCheckoutModal.tsx and added `crypto_checkout_initiated` event type to analytics.ts
7. **Build Verification**: All tests pass (17/17), build succeeds

### File List

- `src/components/DonationImpactBox.tsx` (new) - Donation impact display component
- `src/app/success/page.tsx` (modified) - Added DonationImpactBox and amount parsing
- `src/app/api/checkout/route.ts` (modified) - Added amount to success URL, use shared constant
- `src/app/api/crypto-checkout/route.ts` (modified) - Added amount to redirect URL, use shared constant
- `src/lib/constants.ts` (new) - Shared DONATION_PERCENTAGE constant and calculateDonationAmount function
- `tests/unit/components/donation-impact-box.test.tsx` (new) - 17 unit tests
- `tests/unit/app/success/page.test.tsx` (new) - 8 integration tests for success page
- `src/components/CryptoCheckoutModal.tsx` (modified) - Fixed ESLint apostrophe error
- `src/lib/analytics.ts` (modified) - Added crypto_checkout_initiated event type

## Senior Developer Review

**Reviewed by:** Code Review Workflow
**Date:** 2026-01-21
**Verdict:** APPROVED with fixes applied

### Issues Found & Fixed:
1. **HIGH - AC checkboxes not updated:** All 6 ACs were marked `[ ]` despite implementation being complete. Fixed by marking all `[x]`.
2. **MEDIUM - Duplicate DONATION_PERCENTAGE:** Magic number 0.03 was defined in 3 files. Fixed by creating shared `src/lib/constants.ts` with `calculateDonationAmount()` function using integer math for precision.
3. **MEDIUM - Unused interface params:** Removed dead `provider` and `charge_id` from SuccessSearchParams, added comment for future reference.
4. **MEDIUM - Missing integration tests:** Added 8 integration tests in `tests/unit/app/success/page.test.tsx` verifying DonationImpactBox renders correctly based on amount param.

### Tests After Review:
- 17 unit tests (DonationImpactBox)
- 8 integration tests (Success Page)
- Total: 25 tests passing
- Build: ✅ Verified

## Change Log

- 2026-01-21: Story created by Bob (Scrum Master) via create-story workflow
- 2026-01-21: Implementation completed by Dev Agent (Claude Opus 4.5) - all 17 tests passing, build verified
- 2026-01-21: Code review completed - 1 HIGH, 4 MEDIUM issues found, all fixed automatically
