# ChainSights Event Tracking Implementation Guide

**For:** ChainSights BMAD Team
**Dependency:** analytics.masem.at tracker.js (ready)
**Date:** 2026-01-28

---

## Setup

### 1. Include Tracker Script

Add to `app/layout.tsx` (or root layout):

```html
<script src="https://analytics.masem.at/tracker.js" data-project="chainsights"></script>
```

### 2. Owner Exclusion (Auto)

The tracker automatically excludes owner traffic when:
- User visits any `/admin/*` route → sets `localStorage.setItem('masemit_owner', 'true')`
- User visits with `?owner=sempre` parameter

**No action needed** – this is handled by the tracker.

---

## Events to Implement

### Landing Page (`/`)

| Event | Trigger | Code Location | Properties |
|-------|---------|---------------|------------|
| `cta_view` | "Get Report" CTA enters viewport | Hero section component | `button_name: 'hero_get_report'` |
| `cta_click` | "Get Report" CTA clicked | Hero section component | `button_name: 'hero_get_report'` |

```typescript
// Example: Hero CTA
<Button 
  onClick={() => {
    window.trackEvent?.('cta_click', { button_name: 'hero_get_report' })
    router.push('/rankings')
  }}
>
  Get Report
</Button>
```

---

### Rankings Page (`/rankings`)

| Event | Trigger | Code Location | Properties |
|-------|---------|---------------|------------|
| `rankings_view` | Page loaded | Page component useEffect | - |
| `rankings_filter` | Filter applied | Filter component | `filter_type`, `filter_value` |
| `row_expand` | DAO row expanded | LeaderboardRow component | `dao_name` |
| `cta_click` | "Get Report" on row clicked | LeaderboardRow component | `button_name: 'row_get_report'`, `dao_name` |

```typescript
// Example: Row expand
const handleExpand = (daoName: string) => {
  setExpanded(!expanded)
  window.trackEvent?.('row_expand', { dao_name: daoName })
}

// Example: Filter
const handleFilter = (type: string, value: string) => {
  window.trackEvent?.('rankings_filter', { filter_type: type, filter_value: value })
  applyFilter(type, value)
}
```

---

### Report Tier Selection (NEW - from Monetization Requirement)

| Event | Trigger | Code Location | Properties |
|-------|---------|---------------|------------|
| `tier_view` | Tier selection modal/page shown | TierSelection component | `dao_name` |
| `tier_select` | User selects a tier | TierSelection component | `tier`, `dao_name` |
| `quick_check_start` | Email entered for Quick Check | QuickCheckForm component | `dao_name` |
| `quick_check_complete` | Quick Check results shown | QuickCheckResult component | `dao_name`, `gvs_score` |

```typescript
// Example: Tier selection
const handleTierSelect = (tier: 'quick_check' | 'deep_dive' | 'governance_audit') => {
  window.trackEvent?.('tier_select', { tier, dao_name: selectedDao })
  proceedToCheckout(tier)
}
```

---

### Checkout Flow

| Event | Trigger | Code Location | Properties |
|-------|---------|---------------|------------|
| `checkout_start` | Stripe Checkout initiated | Checkout handler | `product`, `price_eur`, `dao_name` |

```typescript
// Example: Before redirecting to Stripe
const handleCheckout = async (product: string, price: number) => {
  window.trackEvent?.('checkout_start', { 
    product, 
    price_eur: price,
    dao_name: selectedDao 
  })
  
  const session = await createStripeSession(...)
  window.location.href = session.url
}
```

**Note:** `checkout_complete` is tracked server-side via Stripe webhook, not client-side.

---

### Quiz (`/quiz` or wherever it lives)

| Event | Trigger | Code Location | Properties |
|-------|---------|---------------|------------|
| `quiz_start` | User starts quiz | Quiz component | - |
| `quiz_complete` | User finishes quiz | Quiz component | `score` |

```typescript
// Example: Quiz
const startQuiz = () => {
  window.trackEvent?.('quiz_start', {})
  setQuizStarted(true)
}

const completeQuiz = (score: number) => {
  window.trackEvent?.('quiz_complete', { score })
  showResults(score)
}
```

---

### Newsletter Signup

| Event | Trigger | Code Location | Properties |
|-------|---------|---------------|------------|
| `newsletter_view` | Newsletter form enters viewport | Newsletter component | - |
| `newsletter_submit` | Form submitted | Newsletter component | - |

```typescript
// Example: Newsletter with Intersection Observer for view tracking
const newsletterRef = useRef(null)

useEffect(() => {
  const observer = new IntersectionObserver(
    ([entry]) => {
      if (entry.isIntersecting) {
        window.trackEvent?.('newsletter_view', {})
        observer.disconnect()
      }
    },
    { threshold: 0.5 }
  )
  
  if (newsletterRef.current) observer.observe(newsletterRef.current)
  return () => observer.disconnect()
}, [])

const handleSubmit = () => {
  window.trackEvent?.('newsletter_submit', {})
  submitNewsletter(email)
}
```

---

### DAO Matrix (NEW - when implemented)

| Event | Trigger | Code Location | Properties |
|-------|---------|---------------|------------|
| `matrix_view` | Matrix page loaded | Matrix page | `is_subscriber` |
| `matrix_filter` | Filter applied | Matrix filters | `filter_type`, `filter_value` |
| `matrix_sort` | Column sorted | Matrix table header | `column`, `direction` |
| `matrix_compare` | Comparison started | Compare button | `dao_count`, `dao_names` |
| `matrix_export` | Export clicked | Export button | `format` |
| `subscription_start` | Subscribe CTA clicked | Paywall component | `product`, `price` |

---

### Guide/Methodology (`/guide`, `/guide/methodology`)

| Event | Trigger | Code Location | Properties |
|-------|---------|---------------|------------|
| `guide_view` | Guide page loaded | Guide page | `section` (if applicable) |

```typescript
// Simple page view tracking
useEffect(() => {
  window.trackEvent?.('guide_view', { section: 'methodology' })
}, [])
```

---

## TypeScript Declaration

Add to `types/global.d.ts` or similar:

```typescript
declare global {
  interface Window {
    trackEvent?: (
      eventName: string, 
      properties?: Record<string, string | number | boolean>
    ) => void
  }
}

export {}
```

---

## Testing Checklist

Before going live, verify:

- [ ] Script loads without errors (check console)
- [ ] Events appear in analytics.masem.at within 5 seconds
- [ ] Owner exclusion works (visit `/admin`, then check no events fire)
- [ ] All CTAs have click tracking
- [ ] Tier selection flow fully tracked
- [ ] No PII in event properties (no emails, no wallet addresses)

---

## Event Summary Table

| Page | Events |
|------|--------|
| `/` (Landing) | `cta_view`, `cta_click` |
| `/rankings` | `rankings_view`, `rankings_filter`, `row_expand`, `cta_click` |
| Tier Selection | `tier_view`, `tier_select` |
| Quick Check | `quick_check_start`, `quick_check_complete` |
| Checkout | `checkout_start` |
| `/quiz` | `quiz_start`, `quiz_complete` |
| Newsletter | `newsletter_view`, `newsletter_submit` |
| `/matrix` | `matrix_view`, `matrix_filter`, `matrix_sort`, `matrix_compare`, `matrix_export`, `subscription_start` |
| `/guide` | `guide_view` |

---

## Questions?

Reach out to Mario or check analytics.masem.at for event verification.
