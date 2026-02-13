# DEV TICKET: DGI Benchmark Integration in Reports

**Priority:** High
**Effort:** Small-Medium (data exists, wiring + template needed)
**Feature:** Add dimension-by-dimension DGI benchmarks to paid reports

---

## Context

The DGI (Decentralized Governance Index) already calculates and stores daily ecosystem averages for all component dimensions (HPR, DEI, PDI, GPI) in `dgiSnapshots`. The report generator already attaches a basic `dgiBenchmark` object (composite value, category value, percentiles). What's missing is the **component-level benchmark** — showing the customer exactly where their DAO is strong or weak compared to the ecosystem.

This is the killer differentiator: "Your Sybil Resistance (10.0) is top 5% ecosystem-wide. Your Power Distribution (4.0) is below DeFi average (4.8)."

---

## What Already Exists

### Data Layer (`dgiSnapshots` table)

Daily snapshots with these `indexType` values:

| indexType | What it stores |
|-----------|----------------|
| `composite` | Overall ecosystem GVS average |
| `defi` | DeFi category average |
| `infrastructure` | Infrastructure category average |
| `public_goods` | Public Goods category average |
| `social` | Social category average |
| `hpr` | HPR ecosystem average ✅ |
| `dei` | DEI ecosystem average ✅ |
| `pdi` | PDI ecosystem average ✅ |
| `gpi` | GPI ecosystem average ✅ |

Calculation source: `dgi/calculate.ts` lines 120-130

### Report Generator (`analyze/route.ts` lines 206-253)

Already builds `dgiBenchmark` object with:

```typescript
analysis.dgiBenchmark = {
  gvs: 7.2,
  compositeValue: 5.4,       // ✅ DGI Composite
  overallPercentile: 75,      // ✅ "Top 25%"
  categoryValue: 6.0,         // ✅ Category Average
  categoryPercentile: 60,     // ✅ Rank in category
  categoryLabel: "DeFi"       // ✅ Category name
}
```

### PDF Template (`report-template.tsx` lines 1042-1058)

Renders basic benchmark text:

> This DAO's GVS of {gvs} places it in the {percentile}th percentile of the DAO Governance Index (DGI Composite: {compositeValue}).

---

## What Needs To Be Built

### 1. Extend `getCurrentDGI()` query (src/lib/dgi/queries.ts)

Add component dimension averages to the return:

```sql
SELECT index_type, value
FROM dgi_snapshots
WHERE snapshot_date = (SELECT MAX(snapshot_date) FROM dgi_snapshots)
  AND index_type IN ('composite', 'defi', 'infrastructure', 'public_goods', 'social', 'hpr', 'dei', 'pdi', 'gpi');
```

### 2. Extend `dgiBenchmark` object (analyze/route.ts)

Add `componentBenchmarks` to the existing object:

```typescript
analysis.dgiBenchmark = {
  // ... existing fields ...
  componentBenchmarks: {
    hpr: {
      ecosystemAvg: 5.8,   // from dgiSnapshots where indexType = 'hpr'
      daoScore: 10.0,       // from the DAO's own gvsSnapshot
      percentile: 95,       // PERCENT_RANK() for this dimension
      status: "above"       // above | below | average (±10% threshold)
    },
    dei: {
      ecosystemAvg: 6.2,
      daoScore: 7.5,
      percentile: 68,
      status: "above"
    },
    pdi: {
      ecosystemAvg: 4.8,
      daoScore: 4.0,
      percentile: 35,
      status: "below"
    },
    gpi: {
      ecosystemAvg: 5.9,
      daoScore: 5.5,
      percentile: 42,
      status: "below"
    }
  }
}
```

**Status logic:**
- `above`: daoScore > ecosystemAvg * 1.1 (more than 10% above)
- `below`: daoScore < ecosystemAvg * 0.9 (more than 10% below)
- `average`: within ±10%

### 3. Extend PDF Template (report-template.tsx)

Add a new "How You Compare" section after the existing benchmark text.

**Visual structure:**

```
┌──────────────────────────────────────────────────┐
│  HOW YOU COMPARE                                 │
├──────────────────────────────────────────────────┤
│  Overall Score     7.2   vs  DGI Composite  5.4  │
│                    ████████████░░░░  +33%        │
├──────────────────────────────────────────────────┤
│  STRENGTHS (above benchmark)                     │
│  • Sybil Resistance   10.0  vs  5.8  🏆 Top 5%  │
│  • Participation       8.2  vs  6.1  ✅ +34%    │
├──────────────────────────────────────────────────┤
│  IMPROVEMENT AREAS (below benchmark)             │
│  • Power Distribution  4.0  vs  4.8  ⚠️ -17%    │
│  • Process Quality     5.5  vs  5.9  📊 -7%     │
└──────────────────────────────────────────────────┘
```

**Rules:**
- Sort strengths by percentile (highest first)
- Sort improvement areas by percentile (lowest first)
- Show percentage difference: `((daoScore - ecosystemAvg) / ecosystemAvg * 100).toFixed(0)`
- Color coding: green for strengths, amber/orange for improvement areas
- Background: light teal (#F0FDFA) consistent with existing benchmark section

### 4. Tier Differentiation

| Tier | What they get |
|------|---------------|
| **Deep Dive (€49)** | Headline benchmark only: "GVS 7.2 vs DGI Composite 5.4 (Top 25%)" |
| **Governance Audit (€149)** | Full "How You Compare" section with dimension breakdown |

Implementation: Wrap the dimension breakdown in a conditional `{tier === 'governance_audit' && ...}`

---

## Acceptance Criteria

- [ ] `getCurrentDGI()` returns HPR, DEI, PDI, GPI ecosystem averages
- [ ] `dgiBenchmark.componentBenchmarks` is populated for all 4 dimensions
- [ ] Percentile calculation works per dimension
- [ ] PDF renders "How You Compare" section for Governance Audit tier
- [ ] PDF renders headline benchmark for Deep Dive tier
- [ ] Strengths sorted by percentile desc, improvement areas by percentile asc
- [ ] Percentage difference calculated and displayed correctly
- [ ] Section gracefully hidden if benchmark data unavailable

---

## Files to Touch

1. `src/lib/dgi/queries.ts` — extend getCurrentDGI()
2. `src/app/api/reports/analyze/route.ts` — extend dgiBenchmark object
3. `src/lib/reports/report-template.tsx` — add "How You Compare" section
4. Optional: `src/lib/dgi/types.ts` — add TypeScript types for componentBenchmarks

---

## Test Plan

1. Generate a Governance Audit report for a DAO with known scores (e.g., Lido, Giveth)
2. Verify component benchmarks match current dgiSnapshots values
3. Verify strengths/weaknesses are correctly categorized
4. Verify Deep Dive only shows headline, not full breakdown
5. Verify PDF renders cleanly with 0, 1, 2, 3, or 4 dimensions above/below
