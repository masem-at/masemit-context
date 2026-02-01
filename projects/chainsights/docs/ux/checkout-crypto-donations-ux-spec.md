---
title: "UX Specification: Crypto Payments + Donation Tracking"
author: "Sally (UX Designer) + Maya Rivera (Marketing)"
date: "2026-01-19"
status: "Ready for Architecture"
related_doc: "../analysis/product-brief-crypto-payments-donations-2026-01-19.md"
---

# UX Specification: Crypto Payments + Donation Tracking

**Designers:** Sally (UX) + Maya Rivera (Marketing)
**Date:** 2026-01-19
**Status:** Ready for Architecture Review

---

## 🚨 DEVELOPMENT CONSTRAINT

> **ALL CHANGES GO TO `develop` BRANCH ONLY!**

---

## Design Principles

1. **Equal Partners:** Card and Crypto options presented equally — no hierarchy
2. **Authentic, Not Cringe:** Web3-native without trying too hard
3. **Emotional, Not Manipulative:** Donation messaging touches hearts without guilt-tripping
4. **Transparency:** Show actual donation amounts, not vague promises
5. **Simplicity:** Clean, uncluttered checkout experience

---

## 1. Payment Method Selection

### Wireframe

```
┌─────────────────────────────────────────────────────────────┐
│                                                             │
│  How would you like to pay?                                 │
│                                                             │
│  ┌─────────────────────────┐  ┌─────────────────────────┐  │
│  │                         │  │                         │  │
│  │      💳 Card            │  │      ⟠ Crypto           │  │
│  │                         │  │                         │  │
│  │   Visa, Mastercard,     │  │   USDC, ETH, BTC        │  │
│  │   Klarna, EPS           │  │   & more                │  │
│  │                         │  │                         │  │
│  └─────────────────────────┘  └─────────────────────────┘  │
│                                                             │
│  ┌─────────────────────────────────────────────────────┐   │
│  │  💙 3% of your purchase helps families facing       │   │
│  │     childhood illness via hoki.help                 │   │
│  └─────────────────────────────────────────────────────┘   │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### Design Decisions

| Element | Decision | Rationale |
|---------|----------|-----------|
| Button Size | Equal (50/50) | No payment method is "primary" |
| Card Icon | 💳 | Universal, recognizable |
| Crypto Icon | ⟠ | Ethereum-inspired, neutral for all crypto |
| Donation Badge | Below buttons | Natural reading flow after selection |
| Badge Color | Blue (💙) | Warm, trustworthy, matches hoki.help |

### Copy

**Headline:**
> "How would you like to pay?"

**Card Button:**
> "💳 Card"
> "Visa, Mastercard, Klarna, EPS"

**Crypto Button:**
> "⟠ Crypto"
> "USDC, ETH, BTC & more"

**Donation Badge:**
> "💙 3% of your purchase helps families facing childhood illness via hoki.help"

---

## 2. Crypto Payment Modal (Redirect Confirmation)

> **Architecture Update:** Coinbase Commerce does NOT support iframe embedding.
> Modal now serves as redirect confirmation, not embedded checkout.

### Wireframe

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

### Design Decisions

| Element | Decision | Rationale |
|---------|----------|-----------|
| Modal Purpose | Redirect confirmation | Coinbase doesn't support iframe |
| CTA Button | "Continue to Coinbase →" | Clear action, sets expectation |
| Close Button | Top right [✕] | Standard modal pattern |
| Back Option | Bottom link | Easy escape without closing |
| Donation Reminder | Compact version | Reinforce before redirect |

### Behavior

1. User clicks "⟠ Crypto" button
2. Modal opens with smooth animation
3. Modal explains redirect to Coinbase
4. User clicks "Continue to Coinbase"
5. API creates Coinbase charge
6. User redirected to Coinbase hosted checkout
7. User completes payment on Coinbase
8. Coinbase redirects back to our Confirmation page
9. Webhook confirms payment in background

### Technical Notes (from Winston)

- **Coinbase Commerce:** Hosted checkout only (no iframe)
- **Redirect URL:** Returns to `/checkout/success?provider=coinbase&charge_id=xxx`
- **Webhook:** Confirms payment and creates donation record

---

## 3. Confirmation Page (Unified)

### Wireframe

```
┌─────────────────────────────────────────────────────────────┐
│                                                             │
│                     ✓                                       │
│              Payment Successful!                            │
│                                                             │
│  ─────────────────────────────────────────────────────────  │
│                                                             │
│  Thank you for your purchase, [Customer Name]!              │
│                                                             │
│  Your Governance Report for [DAO Name]                      │
│  will be delivered to [email] within 24 hours.              │
│                                                             │
│  ─────────────────────────────────────────────────────────  │
│                                                             │
│  ┌─────────────────────────────────────────────────────┐   │
│  │                                                     │   │
│  │  💙 You just helped a family.                       │   │
│  │                                                     │   │
│  │  €2.97 of your purchase supports children's         │   │
│  │  hospice care via hoki.help                         │   │
│  │                                                     │   │
│  └─────────────────────────────────────────────────────┘   │
│                                                             │
│  ─────────────────────────────────────────────────────────  │
│                                                             │
│       [View Receipt]            [Back to Home]              │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### Design Decisions

| Element | Decision | Rationale |
|---------|----------|-----------|
| Success Icon | ✓ (checkmark) | Universal success indicator |
| Donation Box | Highlighted section | Celebrate the impact |
| Actual Amount | Show €X.XX | Transparency builds trust |
| Headline | "You just helped a family" | Emotional reward, not guilt |

### Copy

**Success Headline:**
> "Payment Successful!"

**Order Confirmation:**
> "Thank you for your purchase, [Customer Name]!"
> "Your Governance Report for [DAO Name] will be delivered to [email] within 24 hours."

**Donation Impact Box:**
> "💙 You just helped a family."
> "€[amount] of your purchase supports children's hospice care via hoki.help"

**Buttons:**
- "View Receipt" → opens receipt/invoice
- "Back to Home" → returns to homepage

### Calculation

```
donation_amount = order_total * 0.03
// €49.00 * 0.03 = €1.47
```

---

## 4. Impact Page (`/impact`)

### Visibility Rule

- **Show page:** When cumulative donations ≥ €500
- **Before €500:** Show "coming soon" message

### Wireframe (Live Version)

```
┌─────────────────────────────────────────────────────────────┐
│                                                             │
│  ChainSights Impact                                         │
│                                                             │
│  ─────────────────────────────────────────────────────────  │
│                                                             │
│                         💙                                  │
│                                                             │
│                      €1,247                                 │
│                                                             │
│              contributed to families                        │
│            facing childhood illness                         │
│                                                             │
│  ─────────────────────────────────────────────────────────  │
│                                                             │
│  Every ChainSights report directs 3% of revenue to         │
│  hoki.help — a children's hospice supporting families       │
│  in Lower Austria through their most difficult moments.     │
│                                                             │
│  This total updates automatically with each purchase.       │
│                                                             │
│  ─────────────────────────────────────────────────────────  │
│                                                             │
│              [Learn more about hoki.help ↗]                 │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### Wireframe (Coming Soon Version — under €500)

```
┌─────────────────────────────────────────────────────────────┐
│                                                             │
│  ChainSights Impact                                         │
│                                                             │
│  ─────────────────────────────────────────────────────────  │
│                                                             │
│                         💙                                  │
│                                                             │
│              We're just getting started.                    │
│                                                             │
│         Check back soon to see our growing impact           │
│           on families facing childhood illness.             │
│                                                             │
│  ─────────────────────────────────────────────────────────  │
│                                                             │
│              [Learn more about hoki.help ↗]                 │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### Copy

**Page Title:**
> "ChainSights Impact"

**Main Number:**
> "€[total]" (large, prominent)

**Subheadline:**
> "contributed to families facing childhood illness"

**Description:**
> "Every ChainSights report directs 3% of revenue to hoki.help — a children's hospice supporting families in Lower Austria through their most difficult moments."

**Update Note:**
> "This total updates automatically with each purchase."

**CTA:**
> "Learn more about hoki.help ↗" (links to https://hoki.help)

**Coming Soon (under €500):**
> "We're just getting started."
> "Check back soon to see our growing impact on families facing childhood illness."

---

## 5. Component Summary

| Component | Location | Status |
|-----------|----------|--------|
| Payment Selection | Checkout page | New |
| Crypto Modal | Overlay on checkout | New |
| Donation Badge (Checkout) | Below payment buttons | New |
| Donation Box (Confirmation) | Confirmation page | New |
| Impact Page | `/impact` route | New |

---

## 6. Responsive Considerations

### Mobile

- Payment buttons stack vertically on small screens
- Modal becomes full-screen on mobile
- Impact page number scales down but remains prominent

### Desktop

- Payment buttons side-by-side
- Modal centered with backdrop
- Impact page centered with max-width container

---

## 7. Accessibility

- All buttons have clear focus states
- Color contrast meets WCAG AA standards
- Donation badge readable by screen readers
- Modal can be closed with Escape key

---

## Ready for Architecture

**Next Step:** Winston reviews this spec and creates Architecture Document

**Questions for Winston:**
1. Can Coinbase Commerce be embedded in iframe, or must we use redirect?
2. Webhook endpoint design for payment confirmation
3. Database schema for donation tracking

---

*Document created by Sally (UX) + Maya Rivera (Marketing)*
*2026-01-19*
