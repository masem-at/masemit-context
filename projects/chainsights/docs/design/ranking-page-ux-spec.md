# Ranking Page UX Spec — Option A (Leaderboard)

**Created:** 2024-12-19
**Author:** Sally (UX Designer)
**Status:** Draft for Review

---

## Overview

A public-facing leaderboard page showing DAO governance health scores, sorted by score descending. Each row expandable to show teaser insight + CTA for full report.

---

## Page Structure

```
┌─────────────────────────────────────────────────────────────────┐
│ HEADER                                                          │
│ ChainSights Logo                            [Request Report]    │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  DAO Governance Health Rankings                                 │
│  How decentralized is your DAO, really?                         │
│                                                                 │
│  Updated: Dec 19, 2024 • Week 51                                │
│                                                                 │
├─────────────────────────────────────────────────────────────────┤
│ LEADERBOARD                                                     │
│                                                                 │
│  #   DAO          Score        Top 5 Control                    │
│  ─────────────────────────────────────────────────────────      │
│  1   Lido         ████████░░ 3/10    52%        [▼ Expand]      │
│  2   ENS          ████████░░ 3/10    31%        [▼ Expand]      │
│  3   Arbitrum     ███████░░░ 2/10    34%        [▼ Expand]      │
│  4   Optimism     ███████░░░ 2/10    45%        [▼ Expand]      │
│  5   Uniswap      ███████░░░ 2/10    54%        [▼ Expand]      │
│  6   Safe         ███████░░░ 2/10    51%        [▼ Expand]      │
│  7   Gitcoin      ███████░░░ 2/10    73%        [▼ Expand]      │
│  8   Balancer     ███████░░░ 2/10    97%        [▼ Expand]      │
│  9   ApeCoin      ███████░░░ 2/10    40%        [▼ Expand]      │
│                                                                 │
├─────────────────────────────────────────────────────────────────┤
│ METHODOLOGY LINK                                                │
│ How we calculate scores → [Learn More]                          │
├─────────────────────────────────────────────────────────────────┤
│ FOOTER                                                          │
│ Data from Snapshot • Updated weekly • © ChainSights 2024        │
└─────────────────────────────────────────────────────────────────┘
```

---

## Expanded Row State

When user clicks [▼ Expand]:

```
│  1   Lido         ████████░░ 3/10    52%        [▲ Collapse]    │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │  "Top 5 wallets control 52% of voting power"            │    │
│  │                                                         │    │
│  │  Gini: 0.847 • Nakamoto: 3 • Voters: 775               │    │
│  │                                                         │    │
│  │  [Get Full Report — €49]    [Share on 𝕏]               │    │
│  └─────────────────────────────────────────────────────────┘    │
```

---

## Component Specs

### Score Bar

| Score | Fill | Color |
|-------|------|-------|
| 1-3 | 10-30% | Red (#EF4444) |
| 4-6 | 40-60% | Yellow (#F59E0B) |
| 7-10 | 70-100% | Green (#10B981) |

```
Score 3/10:  ███░░░░░░░  (30% filled, red)
Score 6/10:  ██████░░░░  (60% filled, yellow)
Score 9/10:  █████████░  (90% filled, green)
```

### Top 5 Control Column

Visual indicator when concentration is concerning:

| Top 5 % | Display |
|---------|---------|
| < 50% | Normal text |
| 50-75% | Bold + yellow |
| > 75% | Bold + red + ⚠️ icon |

Example: `97% ⚠️` (red, bold)

---

## Interactions

| Action | Behavior |
|--------|----------|
| Click row | Toggle expand/collapse |
| Click "Get Full Report" | → Checkout page for that DAO |
| Click "Share on 𝕏" | Open Twitter intent with pre-filled text |
| Click "Learn More" | → Methodology page |
| Hover row | Subtle highlight (#F9FAFB) |

### Share Intent Text

```
{DAO} scores {score}/10 on governance decentralization.

Top 5 wallets control {top5}% of voting power.

See the full rankings: chainsights.one/rankings

via @ChainSights
```

---

## Responsive Behavior

### Desktop (> 768px)
- Full table layout as shown above
- Expanded details inline

### Mobile (< 768px)
- Stack into cards
- Score bar full width
- Tap to expand

```
┌─────────────────────────┐
│ #1 Lido                 │
│ ████████░░ 3/10         │
│ Top 5: 52%              │
│ [Tap for details]       │
└─────────────────────────┘
```

---

## Data Requirements

From `data/rankings.json`:

```typescript
interface RankingRow {
  rank: number
  daoName: string
  spaceId: string
  score: number          // 1-10
  teaser: string         // "Top 5 wallets control X%"
  top5Percentage: number // For display
  gini?: number          // For expanded view
  nakamoto?: number      // For expanded view
  uniqueVoters?: number  // For expanded view
  reportAvailable: boolean
}
```

**Note:** Current `rankings.json` has `teaser` but not `gini`/`nakamoto`/`uniqueVoters`. May need to extend output schema.

---

## Page Route

```
/rankings
```

---

## SEO

```html
<title>DAO Governance Rankings — ChainSights</title>
<meta name="description" content="Weekly rankings of DAO governance health. See how decentralized your favorite DAOs really are." />
```

### Open Graph (for social sharing)

```html
<meta property="og:title" content="DAO Governance Rankings" />
<meta property="og:description" content="Lido: 3/10. Balancer: 2/10 (Top 5 = 97%). See the full rankings." />
<meta property="og:image" content="/og-rankings.png" />
```

---

## Future Enhancements (v2.1+)

- [ ] Historical trend sparkline per DAO
- [ ] Filter by category (L2, DeFi, NFT)
- [ ] Search for specific DAO
- [ ] "Request analysis" for unlisted DAOs
- [ ] Embeddable badge for DAOs to display on their site

---

## Acceptance Criteria

- [ ] Page loads rankings from `data/rankings.json`
- [ ] Rows sorted by score descending
- [ ] Score bar renders with correct color
- [ ] Expand/collapse works on click
- [ ] "Get Full Report" links to checkout
- [ ] "Share on 𝕏" opens Twitter intent
- [ ] Mobile responsive
- [ ] Page is publicly accessible (no auth)

---

*Spec by Sally — ready for Winston's technical review*
