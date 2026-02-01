# Event Tracking Implementation - All Projects

**Date:** 2026-01-28
**Tracker Script:** `https://analytics.masem.at/tracker.js`

---

## Setup (All Projects)

Add to root layout:

```html
<script src="https://analytics.masem.at/tracker.js" data-project="PROJECT_NAME"></script>
```

Where `PROJECT_NAME` is: `chainsights`, `tellingcube`, `hoki-help`, or `masemit-site`

**Owner Exclusion (automatic):**
- Visits to `/admin/*` → sets exclusion flag
- URL parameter `?owner=sempre` → sets exclusion flag

---

## 1. ChainSights (chainsights.one)

### Landing Page (`/`)

| Event | Trigger | Properties |
|-------|---------|------------|
| `cta_view` | Hero CTA enters viewport | `button_name` |
| `cta_click` | Hero CTA clicked | `button_name` |

### Rankings Page (`/rankings`)

| Event | Trigger | Properties |
|-------|---------|------------|
| `rankings_view` | Page loaded | - |
| `rankings_filter` | Filter applied | `filter_type`, `filter_value` |
| `rankings_sort` | Sort changed | `sort_by`, `direction` |
| `row_expand` | DAO row expanded | `dao_name` |
| `cta_click` | "Get Report" button clicked | `button_name`, `dao_name` |

### Tier Selection (Modal/Page)

| Event | Trigger | Properties |
|-------|---------|------------|
| `tier_view` | Tier selection shown | `dao_name` |
| `tier_select` | User selects a tier | `tier`, `dao_name` |

### Quick Check Flow

| Event | Trigger | Properties |
|-------|---------|------------|
| `quick_check_start` | Email submitted | `dao_name` |
| `quick_check_complete` | Results displayed | `dao_name`, `gvs_score` |

### Checkout

| Event | Trigger | Properties |
|-------|---------|------------|
| `checkout_start` | Stripe checkout initiated | `product`, `price_eur`, `dao_name` |

### Quiz

| Event | Trigger | Properties |
|-------|---------|------------|
| `quiz_start` | Quiz started | - |
| `quiz_complete` | Quiz finished | `score` |

### Newsletter

| Event | Trigger | Properties |
|-------|---------|------------|
| `newsletter_view` | Form enters viewport | - |
| `newsletter_submit` | Form submitted | - |

### Guide/Methodology

| Event | Trigger | Properties |
|-------|---------|------------|
| `guide_view` | Page loaded | `section` |

### DAO Matrix (Future)

| Event | Trigger | Properties |
|-------|---------|------------|
| `matrix_view` | Page loaded | `is_subscriber` |
| `matrix_filter` | Filter applied | `filter_type`, `filter_value` |
| `matrix_sort` | Column sorted | `column`, `direction` |
| `matrix_compare` | Comparison started | `dao_count`, `dao_names` |
| `matrix_export` | Export clicked | `format` |
| `subscription_start` | Subscribe clicked | `product`, `price` |

---

## 2. tellingCube (tellingcube.com)

### Landing Page (`/`)

| Event | Trigger | Properties |
|-------|---------|------------|
| `cta_view` | Main CTA enters viewport | `button_name` |
| `cta_click` | Main CTA clicked | `button_name` |

### Examples (`/examples/*`)

| Event | Trigger | Properties |
|-------|---------|------------|
| `example_view` | Example page loaded | `example_name` |
| `example_download` | Example data downloaded | `example_name`, `format` |

### Dashboard (`/dashboard`)

| Event | Trigger | Properties |
|-------|---------|------------|
| `dashboard_view` | Dashboard loaded | - |

### Data Generation

| Event | Trigger | Properties |
|-------|---------|------------|
| `generate_start` | Generation started | `template` |
| `generate_complete` | Generation finished | `template`, `row_count` |
| `generate_error` | Generation failed | `template`, `error_type` |

### Download/Export

| Event | Trigger | Properties |
|-------|---------|------------|
| `download_click` | Download button clicked | `format`, `row_count` |

### Pricing/Checkout

| Event | Trigger | Properties |
|-------|---------|------------|
| `pricing_view` | Pricing section viewed | - |
| `checkout_start` | Checkout initiated | `product`, `price_eur` |

---

## 3. hoki.help

### Landing Page (`/`)

| Event | Trigger | Properties |
|-------|---------|------------|
| `page_view` | Page loaded | - |
| `donate_cta_view` | Donate button enters viewport | - |
| `donate_cta_click` | Donate button clicked | - |

### Donation Flow

| Event | Trigger | Properties |
|-------|---------|------------|
| `amount_select` | Amount selected | `amount_eur`, `is_custom` |
| `donate_start` | Checkout initiated | `amount_eur`, `payment_method` |
| `donate_complete` | Donation confirmed | `amount_eur` |

### Info Pages

| Event | Trigger | Properties |
|-------|---------|------------|
| `info_view` | Info page viewed | `page_name` |

### Social Sharing

| Event | Trigger | Properties |
|-------|---------|------------|
| `share_click` | Share button clicked | `platform` |

---

## 4. masem.at (Portfolio Site)

### Landing Page (`/`, `/en`, `/de`)

| Event | Trigger | Properties |
|-------|---------|------------|
| `page_view` | Page loaded | `language` |

### Project Links

| Event | Trigger | Properties |
|-------|---------|------------|
| `project_click` | Project link clicked | `project_name` |

### Contact

| Event | Trigger | Properties |
|-------|---------|------------|
| `contact_click` | Contact link clicked | `contact_type` |

### Navigation

| Event | Trigger | Properties |
|-------|---------|------------|
| `nav_click` | Navigation item clicked | `target` |

---

## Summary Table

| Project | Total Events | Priority |
|---------|--------------|----------|
| **ChainSights** | 21 events | High |
| **tellingCube** | 10 events | Medium |
| **hoki.help** | 7 events | Medium |
| **masem.at** | 4 events | Low |

---

## TypeScript Declaration (All Projects)

Add to `types/global.d.ts`:

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

## Usage Example

```typescript
// Simple event
window.trackEvent?.('cta_click', { button_name: 'hero_get_started' })

// Event with multiple properties
window.trackEvent?.('checkout_start', {
  product: 'deep_dive',
  price_eur: 49,
  dao_name: 'gitcoin'
})

// Viewport tracking with Intersection Observer
useEffect(() => {
  const observer = new IntersectionObserver(
    ([entry]) => {
      if (entry.isIntersecting) {
        window.trackEvent?.('cta_view', { button_name: 'donate' })
        observer.disconnect()
      }
    },
    { threshold: 0.5 }
  )
  if (ref.current) observer.observe(ref.current)
  return () => observer.disconnect()
}, [])
```

---

## CSP Note

All projects need `https://analytics.masem.at` in their Content Security Policy `script-src` directive.

```javascript
// next.config.js
script-src 'self' ... https://analytics.masem.at;
```
