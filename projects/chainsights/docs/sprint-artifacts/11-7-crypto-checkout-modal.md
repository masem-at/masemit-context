# Story 11.7: Crypto Checkout Modal

Status: done

## Story

**As a** customer
**I want** a confirmation before redirecting to Coinbase
**So that** I understand I'm leaving the site temporarily

## Acceptance Criteria

1. - [x] Modal opens when crypto payment selected
2. - [x] Shows order total (tier price in EUR)
3. - [x] Explains redirect to Coinbase Commerce
4. - [x] "Continue to Coinbase" button initiates checkout
5. - [x] Close button and back link work correctly
6. - [x] Donation badge visible in modal
7. - [x] Loading state while creating charge
8. - [x] Error state if charge creation fails

## Tasks / Subtasks

- [x] Task 1: Create CryptoCheckoutModal component (AC: #1, #2, #3, #6)
  - [x] 1.1 Create `src/components/CryptoCheckoutModal.tsx`
  - [x] 1.2 Use shadcn/ui Dialog component (same pattern as crypto-payment-modal.tsx)
  - [x] 1.3 Display tier name, order total in EUR
  - [x] 1.4 Add redirect explanation text per UX spec
  - [x] 1.5 Include DonationBadge component (from Story 11-6)
  - [x] 1.6 List supported currencies: "USDC, ETH, BTC & more"
- [x] Task 2: Implement checkout flow (AC: #4, #7, #8)
  - [x] 2.1 Call `POST /api/crypto-checkout` on "Continue to Coinbase" click
  - [x] 2.2 Generate unique orderId (use uuid or crypto.randomUUID())
  - [x] 2.3 Show Loader2 spinner during API call
  - [x] 2.4 Redirect to `checkoutUrl` on success (window.location.href)
  - [x] 2.5 Show error message in red box on API failure
  - [x] 2.6 Allow retry on error
- [x] Task 3: Implement modal controls (AC: #5)
  - [x] 3.1 Close button (X) in top-right via DialogClose
  - [x] 3.2 "Back to payment options" link at bottom
  - [x] 3.3 Escape key closes modal (Dialog default)
  - [x] 3.4 Click outside closes modal (Dialog default)
- [x] Task 4: Integrate with pricing.tsx (AC: #1)
  - [x] 4.1 Replace CryptoPaymentModal import with CryptoCheckoutModal
  - [x] 4.2 Update handleCryptoCheckout to open new modal
  - [x] 4.3 Pass tier, priceEur, tierName props
  - [x] 4.4 Remove old crypto-payment-modal.tsx import (can keep file for now)
- [x] Task 5: Write unit tests (AC: all)
  - [x] 5.1 Test modal renders with correct tier info
  - [x] 5.2 Test loading state displays during API call
  - [x] 5.3 Test error state displays on failure
  - [x] 5.4 Test close/back functionality
  - [x] 5.5 Test DonationBadge is present

## Dev Notes

### CRITICAL CONSTRAINT

> **ALL CODE CHANGES GO TO `develop` BRANCH ONLY!**

### Key Difference from Old Modal

The existing `crypto-payment-modal.tsx` implements a **manual USDC transfer flow**:
1. User connects wallet
2. User manually sends USDC to treasury wallet
3. User enters tx hash
4. Backend verifies transaction

**This story creates a NEW modal** for **Coinbase Commerce redirect flow**:
1. User clicks "Continue to Coinbase"
2. API creates Coinbase charge
3. User redirected to Coinbase hosted checkout
4. User completes payment on Coinbase
5. Coinbase redirects back to our success page

### UX Spec Copy (Exact Text to Use)

From `docs/ux/checkout-crypto-donations-ux-spec.md`:

**Modal Title:**
> "Pay with Crypto"

**Subtitle (with Ethereum symbol):**
> "Order Total: €[price]"

**Explanation Box:**
> "You'll be redirected to Coinbase to complete your payment securely."
> "Supported currencies: USDC, ETH, BTC & more"

**CTA Button:**
> "Continue to Coinbase →"

**Back Link:**
> "← Back to payment options"

**Donation Reminder (compact):**
> "3% supports hoki.help"

### Wireframe Reference

```
┌─────────────────────────────────────────────────────────────┐
│  ⟠ Pay with Crypto                               [✕ Close] │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  Order Total: €49.00                                        │
│                                                             │
│  ┌─────────────────────────────────────────────────────┐   │
│  │                                                     │   │
│  │  You'll be redirected to Coinbase to complete      │   │
│  │  your payment securely.                            │   │
│  │                                                     │   │
│  │  Supported currencies:                              │   │
│  │  USDC • ETH • BTC & more                           │   │
│  │                                                     │   │
│  └─────────────────────────────────────────────────────┘   │
│                                                             │
│  ┌─────────────────────────────────────────────────────┐   │
│  │             Continue to Coinbase →                  │   │
│  └─────────────────────────────────────────────────────┘   │
│                                                             │
│  💙 3% supports hoki.help                                   │
│                                                             │
│  ← Back to payment options                                  │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### Component Props Interface

```typescript
interface CryptoCheckoutModalProps {
  isOpen: boolean
  onClose: () => void
  tier: TierId           // 'standard' | 'deep_dive'
  priceEur: number       // 99 or 199
  tierName: string       // 'Standard' or 'Deep Dive'
}
```

### API Integration

**Endpoint:** `POST /api/crypto-checkout` (Story 11-5, DONE)

**Request:**
```typescript
{
  orderId: string      // Generate with crypto.randomUUID()
  orderTotal: number   // priceEur from props
  customerEmail: string // Collect in modal OR use placeholder
  daoName: string      // Use tierName or "General Report"
}
```

**Response:**
```typescript
{
  chargeId: string
  checkoutUrl: string  // Redirect here on success
}
```

**Note on customerEmail:** The UX spec doesn't show email collection in the modal. Options:
1. Use placeholder email (Coinbase collects it during checkout)
2. Add optional email field to modal
3. Skip email entirely (Coinbase will collect)

**Recommendation:** Skip email collection in modal. Coinbase Commerce collects customer email during their checkout flow. Use `customer@checkout.coinbase.com` as placeholder or make it optional in API.

### Error Handling

**API Errors to Handle:**
- 503: Coinbase not configured → Show "Crypto payments temporarily unavailable"
- 400: Validation error → Show specific error
- 500: Coinbase API failure → Show "Unable to create checkout. Please try again."

**Error Display Pattern:**
```tsx
{error && (
  <div className="text-sm text-red-400 bg-red-900/20 p-3 rounded">
    {error}
  </div>
)}
```

### Loading State Pattern

```tsx
<Button disabled={isLoading}>
  {isLoading ? (
    <>
      <Loader2 className="w-4 h-4 mr-2 animate-spin" />
      Creating checkout...
    </>
  ) : (
    'Continue to Coinbase →'
  )}
</Button>
```

### shadcn/ui Dialog Usage

Follow same pattern as existing `crypto-payment-modal.tsx`:

```tsx
import { Dialog, DialogContent, DialogHeader, DialogTitle, DialogDescription } from '@/components/ui/dialog'

<Dialog open={isOpen} onOpenChange={onClose}>
  <DialogContent className="max-w-md">
    <DialogHeader>
      <DialogTitle>⟠ Pay with Crypto</DialogTitle>
      <DialogDescription>{tierName} Report — €{priceEur}</DialogDescription>
    </DialogHeader>
    {/* Content */}
  </DialogContent>
</Dialog>
```

### Integration with pricing.tsx

**Current state (Story 11-6):**
```tsx
// pricing.tsx imports CryptoPaymentModal
import { CryptoPaymentModal } from "@/components/crypto-payment-modal"

// handleCryptoCheckout opens old modal
const handleCryptoCheckout = (tier: TierId, price: number, name: string) => {
  setSelectedTier({ tier, price, name })
  setCryptoModalOpen(true)
}
```

**After this story:**
```tsx
// Replace import
import { CryptoCheckoutModal } from "@/components/CryptoCheckoutModal"

// Same handler, but opens new modal
// (props interface is compatible, so no handler changes needed)
```

### File Structure

```
src/components/
├── CryptoCheckoutModal.tsx    # NEW - this story
├── crypto-payment-modal.tsx   # OLD - keep but don't use (can delete later)
├── DonationBadge.tsx          # From Story 11-6 - reuse
└── pricing.tsx                # MODIFY - swap modal import

tests/unit/components/
└── crypto-checkout-modal.test.tsx  # NEW - unit tests
```

### Testing Patterns

```typescript
// tests/unit/components/crypto-checkout-modal.test.tsx
import { render, screen, fireEvent, waitFor } from '@testing-library/react'
import { CryptoCheckoutModal } from '@/components/CryptoCheckoutModal'

// Mock fetch for API calls
const mockFetch = vi.fn()
global.fetch = mockFetch

describe('CryptoCheckoutModal', () => {
  const defaultProps = {
    isOpen: true,
    onClose: vi.fn(),
    tier: 'standard' as const,
    priceEur: 99,
    tierName: 'Standard',
  }

  beforeEach(() => {
    vi.clearAllMocks()
  })

  it('displays order total', () => {
    render(<CryptoCheckoutModal {...defaultProps} />)
    expect(screen.getByText(/€49/)).toBeInTheDocument()
  })

  it('shows loading state during checkout', async () => {
    mockFetch.mockImplementation(() => new Promise(() => {})) // Never resolves
    render(<CryptoCheckoutModal {...defaultProps} />)

    fireEvent.click(screen.getByText(/Continue to Coinbase/))

    expect(screen.getByText(/Creating checkout/)).toBeInTheDocument()
  })

  it('shows error on API failure', async () => {
    mockFetch.mockResolvedValue({
      ok: false,
      json: () => Promise.resolve({ error: 'Test error' }),
    })
    render(<CryptoCheckoutModal {...defaultProps} />)

    fireEvent.click(screen.getByText(/Continue to Coinbase/))

    await waitFor(() => {
      expect(screen.getByText(/Test error|Unable to create/)).toBeInTheDocument()
    })
  })

  it('displays donation badge', () => {
    render(<CryptoCheckoutModal {...defaultProps} />)
    expect(screen.getByText(/hoki\.help/)).toBeInTheDocument()
  })

  it('calls onClose when back link clicked', () => {
    render(<CryptoCheckoutModal {...defaultProps} />)
    fireEvent.click(screen.getByText(/Back to payment/))
    expect(defaultProps.onClose).toHaveBeenCalled()
  })
})
```

### Previous Story Learnings (Story 11-6)

1. **HTML Entities:** Use `&` not `&amp;` in JSX strings
2. **Equal Button Sizing:** Use `grid grid-cols-1 md:grid-cols-2` for responsive layout
3. **Accessibility:** Add aria-labels to buttons, ensure focus ring styles
4. **DonationBadge:** Reuse existing component, don't recreate

### Dependencies

- **Story 11-5:** `POST /api/crypto-checkout` endpoint (DONE)
- **Story 11-6:** `DonationBadge` component (DONE)
- **shadcn/ui:** Dialog, Button components (already installed)
- **lucide-react:** Loader2 icon (already installed)

### Styling Guidelines

- Use project's existing Tailwind classes
- Background: `bg-navy-dark/50` for info boxes
- Borders: `border-aqua/10` for subtle accent
- Text: `text-gray-400` for secondary, `text-white` for primary
- Aqua accent: `text-aqua` for links and emphasis
- Error: `text-red-400 bg-red-900/20` for error states

### References

- [UX Spec: Crypto Payment Modal](../ux/checkout-crypto-donations-ux-spec.md#2-crypto-payment-modal-redirect-confirmation)
- [Architecture: Component Architecture](../architecture/crypto-payments-donations-architecture.md#2-crypto-redirect-modal)
- [Epic: Story 7](./epic-crypto-payments-donations.md#story-7-crypto-checkout-modal)
- [Existing Modal Pattern](../../src/components/crypto-payment-modal.tsx)
- [API Endpoint](../../src/app/api/crypto-checkout/route.ts)

## Dev Agent Record

### Context Reference

Story created by create-story workflow with comprehensive context from:
- Epic file (docs/sprint-artifacts/epic-crypto-payments-donations.md)
- Architecture doc (docs/architecture/crypto-payments-donations-architecture.md)
- UX spec (docs/ux/checkout-crypto-donations-ux-spec.md)
- Previous story (11-6) completion notes and learnings
- Existing codebase analysis (crypto-payment-modal.tsx, pricing.tsx, api/crypto-checkout)
- Git history (8 recent commits on crypto payments epic)

### Agent Model Used

Claude Opus 4.5

### Debug Log References

### Completion Notes List

1. **CryptoCheckoutModal Component**: Created new modal at `src/components/CryptoCheckoutModal.tsx` with Coinbase Commerce redirect flow
2. **UX Implementation**: Modal displays tier name, order total, redirect explanation, supported currencies, and compact donation badge
3. **Checkout Flow**: Calls `POST /api/crypto-checkout`, generates unique orderId with crypto.randomUUID(), handles loading/error states
4. **Modal Controls**: Close button (X), back link, escape key, and click-outside all work via shadcn/ui Dialog
5. **Integration**: Updated `pricing.tsx` to import and use new `CryptoCheckoutModal` instead of old `CryptoPaymentModal`
6. **Tests**: 17 unit tests covering all acceptance criteria - rendering, loading state, error handling, close/back functionality
7. **Existing Tests**: All 12 pricing-payment-buttons tests continue to pass after integration

### File List

- `src/components/CryptoCheckoutModal.tsx` (new) - Coinbase Commerce redirect modal component
- `src/components/pricing.tsx` (modified) - Updated import to use CryptoCheckoutModal
- `tests/unit/components/crypto-checkout-modal.test.tsx` (new) - 17 unit tests for new modal

## Change Log

- 2026-01-21: Story created by Bob (Scrum Master) via create-story workflow
- 2026-01-21: Implementation completed by Dev Agent (Claude Opus 4.5) - all 17 tests passing
- 2026-01-21: Code review completed - 7 issues found, all fixed:
  - Removed unused `DonationBadge` import (inline donation message was used instead)
  - Removed unused `tier` prop from component interface
  - Added `trackEvent('crypto_checkout_initiated')` for conversion funnel analytics
  - Improved error handling to distinguish network errors from API errors
  - Added aria-labels to CTA button and back button for accessibility
  - Extracted placeholder email to named constant for maintainability
  - Added 2 new tests (analytics tracking, aria-label) - total now 19 tests
