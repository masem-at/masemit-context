# Masemit Analytics Tracker - Integration Guide

> **For AI Agents:** Fetch this doc via `GET https://analytics.masem.at/api/docs/tracker`
> **Last Updated:** 2026-02-03
> **Tracker Version:** 1.2.0

---

## Quick Start

Add this script tag to your root layout (`app/layout.tsx` or `src/app/layout.tsx`):

```tsx
import Script from 'next/script';

export default function RootLayout({ children }) {
  return (
    <html>
      <body>
        {children}
        <Script
          src="https://analytics.masem.at/tracker.js"
          strategy="afterInteractive"
          data-project="your-project-id"
        />
      </body>
    </html>
  );
}
```

---

## Critical Settings

### Script Strategy (IMPORTANT!)

```tsx
// ✅ CORRECT - Use afterInteractive
strategy="afterInteractive"

// ❌ WRONG - Will break SPA navigation tracking!
strategy="beforeInteractive"
```

**Why?** According to the documentation, "`beforeInteractive` loads the script before Next.js initializes its router. The tracker hooks `history.pushState`, but Next.js then overwrites it with its own version → SPA navigation is not tracked."

### Required Attributes

| Attribute | Required | Description |
|-----------|----------|-------------|
| `data-project` | Yes | Your project ID (e.g., `chainsights`, `tellingcube`) |

### Optional Attributes

| Attribute | Default | Description |
|-----------|---------|-------------|
| `data-engagement-pages` | none | Comma-separated paths for scroll/time tracking (e.g., `/,/pricing`) |
| `data-scroll-buckets` | `25,50,75,100` | Scroll depth percentages to track |
| `data-time-buckets` | `5,15,30,60` | Seconds on page to track |

---

## Full Example with Engagement Tracking

```tsx
<Script
  src="https://analytics.masem.at/tracker.js"
  strategy="afterInteractive"
  data-project="chainsights"
  data-engagement-pages="/,/pricing,/features"
  data-scroll-buckets="25,50,75,100"
  data-time-buckets="5,15,30,60,120"
/>
```

---

## CSP Headers

If your project uses Content Security Policy headers, add these:

```
script-src: 'self' https://analytics.masem.at
connect-src: 'self' https://analytics.masem.at
```

### Next.js Example (`next.config.js`)

```js
const securityHeaders = [
  {
    key: 'Content-Security-Policy',
    value: `
      script-src 'self' 'unsafe-inline' 'unsafe-eval' https://analytics.masem.at;
      connect-src 'self' https://analytics.masem.at;
    `.replace(/\n/g, '')
  }
];
```

---

## Custom Event Tracking

The tracker exposes a global `masemit` object:

```javascript
// Track custom events
masemit.track('button_click', { button_id: 'cta-hero', position: 'header' });

// Check if tracker is active (false if owner-excluded)
if (masemit.isActive()) {
  // tracking is enabled
}
```

### Common Events

```javascript
// CTA interactions
masemit.track('cta_click', { cta_name: 'Get Started', location: 'hero' });
masemit.track('cta_view', { cta_name: 'Pricing', location: 'nav' });

// Form events
masemit.track('form_start', { form_id: 'contact' });
masemit.track('form_submit', { form_id: 'contact' });

// E-commerce
masemit.track('checkout_start', { plan: 'pro' });
masemit.track('checkout_complete', { plan: 'pro', value: 49 });
```

---

## Owner Exclusion

To exclude yourself from tracking:

1. Visit any page with `?owner=sempre` in the URL
2. Or visit any `/admin/*` path

This sets a localStorage flag and all future visits are excluded.

---

## Troubleshooting Checklist

If events are not appearing in the dashboard:

| Check | How to Verify |
|-------|---------------|
| Script loads | DevTools → Network → filter `tracker.js` → Status 200? |
| Strategy correct | Must be `afterInteractive`, NOT `beforeInteractive` |
| Project ID set | Check `data-project` attribute is present |
| CSP allows | Console → no CSP errors for `analytics.masem.at`? |
| Not owner-excluded | localStorage → no `masemit_owner` key? |
| Events sending | Network → filter `analytics.masem.at/api/events` → POST requests? |
| SPA navigation | Navigate between pages → new POST for each page? |

### Debug Mode

Open browser console and check:

```javascript
// Is tracker loaded?
console.log(window.masemit); // Should show { track: fn, isActive: fn }

// Is tracking active?
console.log(masemit.isActive()); // true if not owner-excluded

// Manual test
masemit.track('test_event', { debug: true });
// Check Network tab for POST to analytics.masem.at
```

---

## Changelog

### 1.2.0 (2026-02-03)
- **BREAKING:** Projects MUST use `strategy="afterInteractive"` (not `beforeInteractive`)
- Fixed: SPA navigation tracking now works correctly with Next.js App Router
- Note: Tracker already had History API hooks, but wrong script timing prevented them from working

### 1.1.0 (2026-01-15)
- Added engagement tracking (scroll depth, time on page)
- Added `data-engagement-pages`, `data-scroll-buckets`, `data-time-buckets` attributes

### 1.0.0 (2025-12-01)
- Initial release
- PageView tracking
- Custom event tracking
- Device/browser detection
- Owner exclusion

---

## Support

- Dashboard: https://analytics.masem.at/dashboard
- Issues: Contact masemIT team
- This doc: https://analytics.masem.at/api/docs/tracker
