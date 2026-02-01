# Fix IBCS Chart Styling - Critical Bug

**Priority:** CRITICAL - Blocking brother validation
**Type:** Bug Fix
**Assignee:** James (Dev)
**Created:** 2025-12-10
**Reference:** `docs/standards/ibcs-cheatsheet.md`

---

## Problem Statement

We claim "IBCS-inspired" visualizations, but our charts violate core IBCS visual standards:

1. **Missing 4th scenario**: We only have AC, PY, FC - IBCS requires 4: AC, PY, **PL**, FC
2. **Wrong bar styling**: All bars are solid fills - IBCS requires different styles per scenario
3. **Terminology confusion**: We may be using "FC" where we mean "PL"

**Impact:** We cannot claim "IBCS-inspired" if the charts don't look like IBCS.

---

## IBCS Visual Standard (Quick Reference)

| Scenario | Abbr | Meaning | Correct Visual Style |
|----------|------|---------|---------------------|
| Actual | **AC** | What really happened (booked figures) | **Solid fill**, dark gray |
| Previous Year | **PY** | Last year's actuals | **Solid fill**, light gray |
| Plan | **PL** | Budget/target set BEFORE period starts | **Outlined only** (no fill) |
| Forecast | **FC** | Updated expectation DURING period | **Hatched/striped** pattern |

### Key Distinction: PL vs FC

- **PL (Plan)** = Budget decided on January 1st, 2025: "We plan to achieve €100k revenue this year"
- **FC (Forecast)** = Updated expectation in June 2025: "Based on H1 results, we now expect €95k"

---

## Current State Analysis

### What we have now:
```
AC (2024) = Solid dark gray ✅ CORRECT
PY (2023) = Solid light gray ✅ CORRECT
FC (2025) = Solid light blue ❌ WRONG (should be hatched)
```

### What we're missing:
- **PL (Plan)** as a separate data point with outlined bars

---

## Tasks

### Task 1: Add PL (Plan) as 4th Data Value
**File:** `lib/utils/comparison-data.ts`
**Effort:** Medium

Currently the comparison data calculates:
- PY (Previous Year) from AC
- FC (Forecast) from AC

**Required changes:**
1. Add PL calculation separate from FC
2. PL = Budget/target (could be AC * plan_target_rate)
3. FC = Updated forecast (current implementation)

**Suggested approach:**
```typescript
// Example structure
const INDUSTRY_RATES = {
  bakery: {
    historical: 0.06,      // PY calculation
    planTarget: 0.08,      // PL - ambitious budget target
    forecastAdjust: 0.05   // FC - realistic updated expectation
  },
  // ...
}
```

### Task 2: Update Sales Chart Bar Styles
**File:** `components/results/SalesView.tsx`
**Effort:** Small-Medium

Update the Revenue Trend chart to use correct IBCS styling:

```tsx
// Current (WRONG):
<Bar dataKey="AC (2024)" fill="#374151" />
<Bar dataKey="PY (2023)" fill="#9CA3AF" />
<Bar dataKey="FC (2025)" fill="#BFDBFE" />

// Required (CORRECT):
<Bar dataKey="AC (2024)" fill="#374151" />           // Solid dark
<Bar dataKey="PY (2023)" fill="#9CA3AF" />           // Solid light
<Bar dataKey="PL (2025)" fill="none" stroke="#374151" strokeWidth={2} />  // Outlined only
<Bar dataKey="FC (2025)" fill="url(#hatchPattern)" />  // Hatched pattern
```

**Note:** Recharts may need a custom SVG pattern for hatching. Research options:
- SVG `<pattern>` definition with `<defs>`
- Custom bar shape component
- Alternative: Use different opacity + dashed stroke as simplified approach

### Task 3: Update Finance Chart Bar Styles
**File:** `components/results/FinanceView.tsx`
**Effort:** Small-Medium

Same changes as Task 2 for the Net Cash Flow chart.

### Task 4: Create Reusable IBCS Pattern Definitions
**File:** `components/charts/IBCSPatterns.tsx` (NEW)
**Effort:** Small

Create reusable SVG pattern definitions for IBCS-compliant charts:

```tsx
export const IBCSPatternDefs = () => (
  <defs>
    {/* Hatched pattern for FC (Forecast) */}
    <pattern id="hatchPattern" patternUnits="userSpaceOnUse" width="8" height="8">
      <path d="M-2,2 l4,-4 M0,8 l8,-8 M6,10 l4,-4"
            stroke="#374151" strokeWidth="1.5" />
    </pattern>
  </defs>
)

export const IBCS_COLORS = {
  AC: '#374151',           // Solid dark gray
  PY: '#9CA3AF',           // Solid light gray
  PL_STROKE: '#374151',    // Outline color for Plan
  FC_PATTERN: 'url(#hatchPattern)',  // Hatched pattern for Forecast
}
```

### Task 5: Update Chart Legends
**Files:** `SalesView.tsx`, `FinanceView.tsx`
**Effort:** Small

Ensure legends correctly show:
- AC (2024) with solid dark square
- PY (2023) with solid light square
- PL (2025) with outlined square
- FC (2025) with hatched square

### Task 6: Review and Fix Terminology
**Files:** All chart components, data utilities
**Effort:** Small

Verify all labels use correct terminology:
- "PL" for Plan/Budget (not "FC" if it means planned target)
- "FC" for Forecast (updated expectation)

---

## Acceptance Criteria

- [ ] Charts display 4 scenario types: AC, PY, PL, FC
- [ ] AC bars = solid dark gray fill
- [ ] PY bars = solid light gray fill
- [ ] PL bars = outlined only (no fill, just border)
- [ ] FC bars = hatched/striped pattern
- [ ] Legend reflects correct visual styles
- [ ] Terminology is consistent (PL=Plan, FC=Forecast)

---

## Technical Notes

### Recharts Hatched Pattern Implementation

Option A - SVG Pattern (Recommended):
```tsx
<BarChart>
  <defs>
    <pattern id="hatch" patternUnits="userSpaceOnUse" width="4" height="4">
      <path d="M-1,1 l2,-2 M0,4 l4,-4 M3,5 l2,-2"
            stroke="#374151" strokeWidth="1" />
    </pattern>
  </defs>
  <Bar dataKey="FC" fill="url(#hatch)" />
</BarChart>
```

Option B - Custom Shape (Fallback):
```tsx
const HatchedBar = (props) => {
  const { x, y, width, height } = props;
  return (
    <g>
      <rect x={x} y={y} width={width} height={height} fill="url(#hatch)" />
    </g>
  );
};

<Bar dataKey="FC" shape={<HatchedBar />} />
```

### Outlined Bar Implementation

```tsx
<Bar
  dataKey="PL"
  fill="transparent"
  stroke="#374151"
  strokeWidth={2}
/>
```

---

## Dependencies

- `docs/standards/ibcs-cheatsheet.md` - Visual reference
- `docs/standards/ibcs-standards.md` - Full IBCS documentation
- `lib/utils/comparison-data.ts` - Current PY/FC calculation logic

---

## Timeline

**Target:** Complete before next brother test round

This is a bug fix, not a feature. No full BMAD workflow required - direct implementation by James.
