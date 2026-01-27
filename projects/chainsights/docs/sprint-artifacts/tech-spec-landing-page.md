# Tech-Spec: ChainSights Landing Page

**Created:** 2025-12-15
**Status:** Ready for Development

---

## Overview

### Problem Statement

Elena (Governance Lead at a mid-sized DeFi DAO) needs to find ChainSights, understand its value immediately, and purchase a Governance Truth Report for €99. Currently, no landing page exists — she can't discover or buy the product.

### Solution

Build a high-converting landing page that:
1. Communicates the "Wallets lie. We don't." value proposition instantly
2. Shows sample report sections to reduce pre-purchase anxiety
3. Enables frictionless €99 payment via Stripe
4. Collects DAO contract address for report generation

### Scope

**In Scope:**
- Hero section with headline and CTA
- Value proposition section (why identity-first matters)
- Sample report preview (placeholder structure)
- Pricing section with single €99 CTA
- Trust signals (founder, methodology, Austrian company)
- Stripe Checkout integration
- Basic intake form (DAO address collection post-payment)
- Mobile responsive
- Dark mode with brand colors

**Out of Scope (Future):**
- USDC/crypto payment (Phase 2)
- User accounts/authentication
- Report delivery system
- Analytics/tracking beyond basic Vercel Analytics
- Blog/content pages
- Multi-language support

---

## Context for Development

### Tech Stack

| Technology | Version | Purpose |
|------------|---------|---------|
| Next.js | 14+ | App Router, React framework |
| Tailwind CSS | 3.4+ | Utility-first styling |
| shadcn/ui | latest | Component library |
| Stripe | latest | Payment processing |
| Vercel | Pro | Hosting, edge functions |
| next/font | built-in | Montserrat + Inter fonts |

### Brand Colors (Tailwind Config)

```javascript
// tailwind.config.js colors extension
colors: {
  navy: {
    DEFAULT: '#0F2C55',
    dark: '#1A1F2E',
  },
  aqua: {
    DEFAULT: '#00E5C0',
  },
  azure: {
    DEFAULT: '#4A89F3',
  },
  violet: {
    DEFAULT: '#B84EFF',
  },
  neutral: {
    light: '#F5F7FA',
  }
}
```

### Typography

- **Headlines:** Montserrat (font-montserrat), bold/semibold
- **Body:** Inter (font-inter), regular/medium
- **Code/Data:** Roboto Mono (optional, for report metrics)

### Design Direction

- **Aesthetic:** Hybrid — Clean professional foundation + subtle Web3 accents
- **Background:** Dark navy (#0F2C55 to #1A1F2E gradient)
- **Accents:** Aqua (#00E5C0) for CTAs and highlights
- **Text:** White/light gray on dark background
- **Cards:** Subtle glass-morphism or soft borders
- **Vibe:** "Serious analytics firm that speaks Web3 fluently"

### File Structure

```
chainsights/
├── app/
│   ├── layout.tsx          # Root layout with fonts, metadata
│   ├── page.tsx            # Landing page
│   ├── api/
│   │   └── checkout/
│   │       └── route.ts    # Stripe checkout session
│   └── success/
│       └── page.tsx        # Post-payment success + intake form
├── components/
│   ├── ui/                 # shadcn/ui components
│   ├── hero.tsx            # Hero section
│   ├── value-prop.tsx      # Why identity-first matters
│   ├── sample-report.tsx   # Report preview section
│   ├── pricing.tsx         # Pricing CTA section
│   ├── trust-signals.tsx   # Founder, methodology, company
│   ├── footer.tsx          # Simple footer
│   └── intake-form.tsx     # DAO address collection
├── lib/
│   ├── stripe.ts           # Stripe client config
│   └── utils.ts            # shadcn/ui utils
├── public/
│   └── images/             # Logo, founder photo, icons
├── tailwind.config.js
├── next.config.js
└── .env.local              # STRIPE_SECRET_KEY, etc.
```

---

## Implementation Plan

### Tasks

- [ ] **Task 1: Project Setup**
  - Initialize Next.js 14 with App Router
  - Install and configure Tailwind CSS
  - Install and configure shadcn/ui (dark mode)
  - Configure custom colors in tailwind.config.js
  - Set up Montserrat + Inter fonts via next/font
  - Create basic layout.tsx with metadata

- [ ] **Task 2: Hero Section**
  - Large headline: "Wallets lie. We don't."
  - Subheadline: "Identity-first governance analytics for DAOs"
  - Primary CTA button: "Get Your Report — €99" (links to #pricing)
  - Subtle animated gradient or glow effect on headline
  - Mobile responsive

- [ ] **Task 3: Value Proposition Section**
  - "The Problem" — DAOs count wallets, not people
  - "Our Solution" — Identity-aware analysis with confidence levels
  - 3 key benefits with icons:
    1. "True Participation" — Know how many humans, not wallets
    2. "Power Concentration" — See who really controls governance
    3. "Shareable Proof" — Professional report for your community
  - Visual: simple diagram or illustration

- [ ] **Task 4: Sample Report Preview**
  - Mock report sections showing structure:
    - Executive Summary preview
    - "Unique Voters Analysis" with [Sample Data] placeholders
    - "Power Distribution" chart placeholder
    - "Recommendations" section preview
  - Styled to look like actual report output
  - Note: "Based on real on-chain analysis methodology"
  - CTA: "See what your report will contain"

- [ ] **Task 5: Pricing Section**
  - Single tier: "Governance Truth Report — €99"
  - What's included list:
    - True participation score
    - Wallet clustering analysis
    - Decentralization assessment
    - MiCA-readiness indicator
    - Recommendations
    - Shareable summary page
  - "Delivered within 24 hours"
  - Primary CTA button: "Get Your Report Now"
  - Trust badge: "Secure payment via Stripe"

- [ ] **Task 6: Trust Signals Section**
  - Founder photo + brief intro (Mario, Austria)
  - "Built for MiCA-era compliance"
  - Methodology transparency: "Heuristics + AI analysis"
  - Optional: "As seen in..." (placeholder for future)

- [ ] **Task 7: Footer**
  - Logo
  - "ChainSights — Identity-first Web3 analytics"
  - Contact: email
  - Legal links: Privacy, Terms (placeholder pages)
  - "Built in Austria"

- [ ] **Task 8: Stripe Integration**
  - Create Stripe account + get API keys
  - Implement /api/checkout/route.ts:
    - Create Checkout Session for €99 one-time payment
    - Success URL: /success?session_id={CHECKOUT_SESSION_ID}
    - Cancel URL: / (back to landing)
  - Wire up CTA buttons to trigger checkout
  - Test in Stripe test mode

- [ ] **Task 9: Success Page + Intake Form**
  - /success/page.tsx
  - Confirm payment received
  - Intake form fields:
    - DAO Name (text)
    - Governance Contract Address (text, required)
    - Chain (dropdown: Ethereum mainnet for MVP)
    - Email for delivery (text, required)
    - Additional context (textarea, optional)
  - Submit button
  - For MVP: Form submits to email (or simple API that sends notification)
  - Thank you message: "Report delivered within 24 hours"

- [ ] **Task 10: Polish & Deploy**
  - Responsive testing (mobile, tablet, desktop)
  - Lighthouse performance check
  - Add favicon and OG image
  - Set up Vercel project
  - Configure environment variables
  - Deploy to Vercel
  - Connect domain (chainsights.io or chosen domain)

### Acceptance Criteria

- [ ] **AC1:** Landing page loads in <3s on mobile (Lighthouse performance >80)
- [ ] **AC2:** "Wallets lie. We don't." headline is visible above fold on all devices
- [ ] **AC3:** Clicking any CTA button initiates Stripe Checkout for €99
- [ ] **AC4:** After successful payment, user sees intake form
- [ ] **AC5:** Intake form submission sends notification (email or webhook)
- [ ] **AC6:** All text is readable (WCAG AA contrast compliance)
- [ ] **AC7:** Site works on Chrome, Firefox, Safari (latest versions)
- [ ] **AC8:** Dark mode aesthetic matches brand guide (navy, aqua accents)

---

## Additional Context

### Dependencies

| Dependency | Purpose | Notes |
|------------|---------|-------|
| Stripe Account | Payment processing | Need to create account, get keys |
| Vercel Pro | Hosting | Already have subscription |
| Domain | chainsights.one | Need to purchase and configure |
| Email | Notification delivery | Can use personal email for MVP |

### Environment Variables

```bash
# .env.local
STRIPE_SECRET_KEY=sk_test_xxx
STRIPE_PUBLISHABLE_KEY=pk_test_xxx
NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY=pk_test_xxx
STRIPE_WEBHOOK_SECRET=whsec_xxx  # Optional for MVP
NOTIFICATION_EMAIL=mario@example.com
```

### Testing Strategy

**Manual Testing:**
- Test Stripe checkout flow in test mode
- Test on mobile devices (iOS Safari, Android Chrome)
- Test form validation on intake form
- Visual review against brand guidelines

**Automated (Optional for MVP):**
- Basic smoke test that page loads
- Can add Playwright tests later

### Notes

1. **MVP Mindset:** Ship fast, iterate. Placeholder content is fine.
2. **Stripe Test Mode:** Use test cards (4242 4242 4242 4242) until ready for real payments
3. **Form Handling:** For MVP, simple email notification is enough. No database needed yet.
4. **Analytics:** Vercel Analytics (free with Pro) is sufficient to start
5. **Legal Pages:** Placeholder Privacy/Terms pages are fine for MVP, update before real traffic

### Copy/Content Needed

| Section | Content Status |
|---------|---------------|
| Headline | "Wallets lie. We don't." ✅ |
| Subheadline | "Identity-first governance analytics for DAOs" ✅ |
| Value prop bullets | Need to finalize 3 key benefits |
| Sample report | Placeholder structure defined |
| Pricing copy | List from Product Brief ✅ |
| Founder bio | Need 2-3 sentences |
| Legal pages | Placeholder needed |

---

## Ready for Development

This spec contains everything needed to build the ChainSights landing page. A fresh dev agent can implement this without additional context.

**Estimated effort:** 4-6 hours for experienced developer
**Priority:** HIGH — Blocks all revenue
