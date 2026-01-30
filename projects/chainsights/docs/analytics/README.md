# Analytics Documentation - ChainSights

**Last Updated:** 2025-12-23
**Story:** 8.8 - Configure Analytics and Conversion Tracking

---

## Overview

ChainSights uses **Vercel Analytics** for tracking user behavior, conversion funnel, and site performance. This document is your complete guide to understanding and using our analytics infrastructure.

---

## Quick Links

- **[Dashboard Guide](./dashboard-guide.md)** - How to access and use Vercel Analytics dashboard
- **[Conversion Funnel](./conversion-funnel.md)** - 5-stage funnel definition and calculation formulas
- **[Weekly Review Template](./weekly-review-template.md)** - Template for weekly analytics reviews
- **Dashboard URL:** https://vercel.com → chainsights → Analytics

---

## What's Tracked Automatically

### 1. Page Views (Automatic)
- **All public pages** automatically tracked
- **No code needed** - Vercel Analytics handles this
- **Data retention:** 30 days (free tier)

**Pages tracked:**
- `/` - Homepage
- `/rankings` - Leaderboard
- `/rankings/methodology` - Methodology explanation
- `/privacy` - Privacy policy
- `/terms` - Terms of service
- `/accessibility` - Accessibility statement
- `/rankings/opt-out` - Opt-out form
- `/quiz` - Quiz page

---

### 2. Core Web Vitals (Automatic)
- **LCP (Largest Contentful Paint)** - Target: <2.5s
- **FCP (First Contentful Paint)** - Target: <1.8s
- **CLS (Cumulative Layout Shift)** - Target: <0.1
- **FID (First Input Delay)** - Target: <100ms
- **TTFB (Time to First Byte)** - Target: <600ms

**No code needed** - Automatic real user monitoring

---

### 3. Visitor Metrics (Automatic)
- **Unique visitors** - Number of distinct users
- **Returning visitors** - Users who visited before
- **Device type** - Mobile vs desktop
- **Geographic location** - Country-level
- **Traffic sources** - Direct, referral, social

---

## Custom Events (Manual Tracking)

### Event 1: `cta_click`
**When:** User clicks "Order Report" CTA button
**Properties:**
- `dao` - DAO name (e.g., "arbitrum")
- `score` - GVS score (e.g., 42)
- `location` - Where CTA was clicked ("leaderboard", "hero", "pricing", etc.)

**Code Location:** Multiple components (hero, pricing, modal, etc.)
**Story:** 5.3

---

### Event 2: `social_share`
**When:** User clicks Twitter or LinkedIn share button
**Properties:**
- `platform` - "twitter" or "linkedin"
- `dao` - DAO name
- `score` - GVS score

**Code Location:** Social share buttons
**Story:** 5.1

---

### Event 3: `row_expand`
**When:** User expands DAO row in leaderboard to see details
**Properties:**
- `dao` - DAO name
- `rank` - Position in rankings (e.g., 3)

**Code Location:** `ExpandableDAORow.tsx`, `MobileDAOCard.tsx`
**Story:** 5.3

---

### Event 4: `methodology_view`
**When:** User visits `/rankings/methodology` page
**Properties:** None

**Code Location:** `methodology-tracker.tsx`
**Story:** 5.3

---

## Conversion Funnel

ChainSights tracks a **5-stage conversion funnel** from awareness to report purchase:

1. **Awareness** → User views `/rankings` page (automatic)
2. **Interest** → User expands DAO row (`row_expand` event)
3. **Consideration** → User views methodology (`methodology_view` event)
4. **Action** → User clicks CTA (`cta_click` event)
5. **Conversion** → User completes Stripe payment (manual tracking)

**Full details:** [Conversion Funnel Guide](./conversion-funnel.md)

---

## Opt-Out Rate Tracking

### Function: `calculateOptOutRate()`
**Location:** `src/lib/analytics.ts`
**Story:** 8.8 (FR91)

**Usage:**
```typescript
import { calculateOptOutRate } from '@/lib/analytics'

const metrics = await calculateOptOutRate()
console.log(`Total DAOs: ${metrics.totalDaos}`)
console.log(`Opt-out requests: ${metrics.optOutRequests}`)
console.log(`Opt-out rate: ${metrics.optOutRate.toFixed(2)}%`)
```

**Formula:** `(Total Opt-Out Requests / Total DAOs Ranked) × 100`

---

## How to Use Analytics

### Weekly Review Process
1. **Schedule:** Every Friday
2. **Duration:** 15-30 minutes
3. **Template:** [Weekly Review Template](./weekly-review-template.md)
4. **Archive:** Save completed reviews in `./reviews/YYYY-MM-DD.md`

**Steps:**
1. Open [Vercel Analytics Dashboard](https://vercel.com)
2. Set date range to "Last 7 days"
3. Collect metrics (page views, events, orders)
4. Calculate conversion rates
5. Document findings and action items

---

### Calculating Conversion Rates

**Interest Rate:**
`(row_expand events / rankings page views) × 100`

**Consideration Rate:**
`(methodology_view events / row_expand events) × 100`

**Action Rate:**
`(cta_click events / methodology_view events) × 100`

**Conversion Rate:**
`(Stripe orders / cta_click events) × 100`

**Overall Conversion:**
`(Stripe orders / rankings page views) × 100`

**Full guide:** [Conversion Funnel](./conversion-funnel.md)

---

## Privacy & GDPR Compliance

### What's NOT Tracked (GDPR-Compliant)
- ❌ No PII (emails, names, wallet addresses)
- ❌ No persistent cookies beyond analytics session
- ❌ No third-party tracking pixels
- ❌ IP addresses not stored beyond Vercel's standard 30-day retention

### What IS Tracked
- ✅ Page views (anonymous)
- ✅ Custom events (no PII in properties)
- ✅ Core Web Vitals (performance metrics)
- ✅ Country-level geographic data
- ✅ Device type (mobile/desktop)

**Compliance:** Story 8.6 - GDPR verification complete
**Privacy Policy:** Updated in Story 3.4

---

## Technical Implementation

### Package
- **@vercel/analytics:** ^1.6.1 (installed in Epic 0)

### Integration
```tsx
// src/app/layout.tsx
import { Analytics } from '@vercel/analytics/next'

export default function RootLayout({ children }) {
  return (
    <html>
      <body>
        {children}
        <Analytics /> {/* Automatic tracking */}
      </body>
    </html>
  )
}
```

### Custom Event Helper
```typescript
// src/lib/analytics.ts
import { track } from '@vercel/analytics'

export function trackEvent(
  event: 'cta_click' | 'social_share' | 'row_expand' | 'methodology_view',
  props?: Record<string, unknown>
) {
  track(event, props)
}
```

---

## Troubleshooting

### No Data Showing
- **Wait 10-15 minutes** after deployment
- Check Analytics component is in `src/app/layout.tsx`
- Verify events only track in production (not `npm run dev`)

### Custom Events Missing
- Test in production environment (`vercel --prod`)
- Manually trigger events (click CTA, expand row, etc.)
- Wait 10 minutes for events to appear in dashboard

### Core Web Vitals Missing
- Need 50+ page views for reliable metrics
- Check back in 24-48 hours

---

## Future Enhancements

### Priority 1: Stripe Webhook Integration
- Automatically track `order_completed` event
- Real-time conversion tracking
- Eliminates manual Stripe dashboard checking

### Priority 2: Vercel Audience (Pro Tier)
- Upgrade to Vercel Pro for automated funnel visualization
- Cohort analysis and retention tracking
- Cost: $20/month per team member

### Priority 3: UTM Parameter Tracking
- Add UTM parameters to marketing campaigns
- Track which sources drive best conversions
- Optimize marketing spend based on ROI

---

## NFR Compliance

| NFR | Requirement | Status |
|-----|-------------|--------|
| FR87 | Core Web Vitals tracking | ✅ Automatic |
| FR88 | Page view tracking | ✅ Automatic |
| FR89 | CTA click tracking | ✅ Implemented (Story 5.3) |
| FR90 | Social share tracking | ✅ Implemented (Story 5.1) |
| FR91 | Opt-out rate calculation | ✅ Implemented (Story 8.8) |
| FR92 | Repeat visitor tracking | ✅ Automatic |
| NFR-MON-8 | Analytics integration | ✅ Complete |
| NFR-PRIV-1 | No PII tracking | ✅ Verified (Story 8.6) |

---

## Related Stories

- **Epic 0 (Story 0.1):** Vercel Analytics integration
- **Epic 5 (Story 5.1):** Social share event tracking
- **Epic 5 (Story 5.3):** CTA and conversion tracking
- **Epic 8 (Story 8.6):** GDPR privacy compliance
- **Epic 8 (Story 8.8):** Analytics configuration and documentation

---

## Questions?

**Owner:** Mario (Product Manager)
**Contact:** [Internal - see project-context.md]
**Dashboard:** https://vercel.com → chainsights → Analytics
