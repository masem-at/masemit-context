# Live Data Ticker - Feature 4 Usage Guide

**Status:** ✅ Complete (Behind Feature Flag)
**Implemented:** December 24, 2025

---

## What It Does

Displays a small, rotating ticker in the top-right corner showing recent DAO score changes:
- Rotates every 5 seconds with smooth fade transitions
- Color-coded: **Aqua** for positive changes, **Red** for negative
- Fixed position (doesn't interfere with content)
- Pre-generated data (no API calls)

**Purpose:** Creates perception of "active tool" with fresh data

---

## How to Enable/Disable

### ✅ Enable Ticker

Add to `.env.local`:
```bash
NEXT_PUBLIC_ENABLE_TICKER=true
```

Then rebuild and deploy:
```bash
npm run build
```

### ❌ Disable Ticker

Remove from `.env.local` or set to `false`:
```bash
NEXT_PUBLIC_ENABLE_TICKER=false
```

Or simply remove the line entirely (defaults to disabled).

---

## A/B Test Plan

**Timeline:** 48 hours after enabling

**Metrics to Track:**
1. **Bounce Rate** (homepage)
   - Before ticker: baseline
   - After ticker: should stay same or decrease

2. **Time on Page** (homepage + /rankings)
   - Before: baseline
   - After: should increase (engagement signal)

3. **Scroll Depth**
   - Are users scrolling to see rankings?
   - Or bouncing after seeing ticker?

4. **Click-through to /rankings**
   - Does ticker create curiosity → drive clicks?

**Tools:**
- Vercel Analytics (built-in)
- Google Analytics (if configured)
- Hotjar/heatmaps (optional)

---

## Decision Criteria

### ✅ KEEP TICKER IF:
- Bounce rate stays same OR decreases
- Time on page increases (even slightly)
- Neutral or positive user feedback
- Click-through to /rankings increases

### ❌ REMOVE TICKER IF:
- Bounce rate increases >5%
- Time on page decreases
- Negative user feedback (looks spammy)
- Heatmap shows it's distracting

---

## Technical Details

**Component:** `src/components/LiveTicker.tsx`
**Used in:** `src/app/page.tsx` (homepage)
**Data:** Pre-generated array (no live fetching)
**Performance:** Minimal impact (~0.4kB added to homepage)

**Data Source:**
```typescript
const TICKER_DATA = [
  { dao: 'Lido', change: '+0.2', direction: 'up' },
  { dao: 'ENS', change: '+0.3', direction: 'up' },
  { dao: 'Arbitrum', change: '-0.1', direction: 'down' },
  // ... 9 total entries
]
```

---

## Customization

### Change Rotation Speed
Edit `LiveTicker.tsx`:
```typescript
const interval = setInterval(() => {
  // ...
}, 5000) // Change from 5000ms (5s) to desired duration
```

### Change Position
Edit `LiveTicker.tsx`:
```typescript
<div className="fixed top-20 right-4 z-40">
  // Change top-20 (top position) or right-4 (right position)
</div>
```

### Update Data
Edit the `TICKER_DATA` array in `LiveTicker.tsx` with real score changes.

---

## Testing Checklist

- [ ] Enable flag: `NEXT_PUBLIC_ENABLE_TICKER=true`
- [ ] Rebuild: `npm run build`
- [ ] Visit homepage: ticker appears top-right
- [ ] Wait 5 seconds: data rotates with fade effect
- [ ] Check positive changes: aqua color
- [ ] Check negative changes: red color
- [ ] Disable flag: ticker disappears
- [ ] Mobile responsive: ticker doesn't overlap content

---

## Deployment Steps

1. **Deploy with ticker DISABLED** (default)
2. **Establish baseline metrics** (48 hours)
   - Track bounce rate, time on page
3. **Enable ticker** via Vercel env vars
   - Add `NEXT_PUBLIC_ENABLE_TICKER=true`
   - Redeploy
4. **Monitor for 48 hours**
   - Compare metrics to baseline
5. **Make decision**
   - Keep: do nothing (already enabled)
   - Remove: disable flag and redeploy

---

## Notes

- Ticker is **client-side only** (doesn't affect SEO)
- Data is **static** (no server load)
- Feature flag allows **instant disable** without code deploy
- If metrics are neutral, err on side of **keeping** (adds dynamism)

---

## Status: Ready for A/B Test

Feature is complete and ready to deploy. Start with ticker **disabled**, establish baseline, then enable for 48-hour test.
