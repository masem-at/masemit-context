# Social Proof Counter — "Insights Uncovered"

**Author:** Sally (UX Designer)
**Status:** Ready for Implementation
**Priority:** Medium — adds credibility once thresholds met

---

## Overview

A dynamic social proof element displaying aggregate insights discovered across all ChainSights reports, positioned to build credibility without relying on sales volume.

---

## Metrics to Display

Start with **2 metrics**:

| Metric | Source | Display Format |
|--------|--------|----------------|
| Governance risks identified | Count of `concerns` from AI analysis | "47 governance risks identified" |
| Voting power analyzed | Sum of `totalVotingPower` across reports | "€3.1M voting power analyzed" |

**Future addition (when ≥25):** "X DAOs audited"

---

## Placement

```
┌─────────────────────────────────────────────────┐
│  HERO SECTION                                   │
│  "Wallets lie. We don't."                       │
│  [Get Your Report CTA]                          │
│                                                 │
│  ─────────────────────────────────────────────  │
│                                                 │
│  ┌─────────────────────────────────────────┐   │
│  │ 🔍 47 governance risks identified       │   │
│  │ 📊 €3.1M voting power analyzed          │   │
│  └─────────────────────────────────────────┘   │
│                                                 │
└─────────────────────────────────────────────────┘
```

**Position:** Below hero CTA, above the "How it works" section.

---

## Visual Design

- **Container:** Subtle card with `border-border` and `bg-card/50` background
- **Layout:** Horizontal on desktop (side by side), stacked on mobile
- **Icons:** Small accent icons in `text-aqua`
- **Numbers:** Bold, `text-white`, `text-2xl` or `text-3xl`
- **Labels:** `text-gray-400`, `text-sm`
- **Animation:** Optional — subtle count-up on scroll into view

---

## API Endpoint

**`GET /api/stats`** (public, cached 1hr)

```typescript
interface StatsResponse {
  risksIdentified: number      // sum of concerns.length across sent reports
  votingPowerAnalyzed: number  // sum of totalVotingPower
  daosAnalyzed: number         // count of reports with status='sent'
  lastUpdated: string          // ISO timestamp
}
```

### Aggregation Logic

- Only count from reports with `status = 'sent'` (delivered)
- Cache aggressively (1 hour minimum)
- Format large numbers: "3.1M" not "3,107,552"

---

## Threshold Rules

| Metric | Minimum to Display |
|--------|-------------------|
| Risks identified | ≥10 |
| Voting power | ≥1M |
| DAOs analyzed | ≥25 (don't show until then) |

**If below all thresholds:** Don't render the component. No empty states.

---

## Copy (Option A — Factual)

```
🔍 47 governance risks identified
📊 €3.1M voting power analyzed
```

---

## Component Structure

```tsx
// src/components/social-proof-counter.tsx

interface SocialProofCounterProps {
  risksIdentified: number
  votingPowerAnalyzed: number
}

// Only render if thresholds met
// Fetch from /api/stats on page load or use SSR
```

---

## Implementation Notes

1. Create `/api/stats` endpoint with caching
2. Create `<SocialProofCounter />` component
3. Add to landing page below hero
4. Component self-hides if thresholds not met
