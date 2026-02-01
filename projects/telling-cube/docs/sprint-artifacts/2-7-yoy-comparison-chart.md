# Story 2.7: YoY Comparison Chart

**Status:** done
**Epic:** 2 - Multi-Year Scenarios (3-5 Years)
**Points:** 5
**Created:** 2026-01-21

---

## Story

As a **data analyst comparing multi-year performance**,
I want **a grouped bar chart comparing the same metrics across years**,
so that **I can easily spot year-over-year trends and identify which years outperformed others**.

---

## Acceptance Criteria

### AC1: Grouped Bars by Year
**Given** a multi-year scenario with 3 or 5 years of data
**When** the YoY comparison chart is displayed
**Then** bars are grouped by year (max 5 groups)
**And** each group shows Revenue, Profit, and Costs

### AC2: ΔPY Variance Indicators
**Given** the comparison chart
**When** viewing year-over-year data
**Then** ΔPY (variance from previous year) is indicated
**And** positive variance shows green indicator
**And** negative variance shows red indicator

### AC3: ΔPY% Badges
**Given** the comparison chart
**When** variance data is available
**Then** percentage change badges (e.g., "+12.5%") appear above bars
**And** badges are color-coded (green for positive, red for negative)

### AC4: Click-to-Drill into Specific Year
**Given** a grouped bar chart with yearly data
**When** the user clicks on a specific year's bars
**Then** detailed breakdown for that year is shown
**Or** the chart zooms to show monthly data for that year

### AC5: IBCS-Inspired Styling
**Given** the comparison chart component
**When** rendered
**Then** follows IBCS standards:
  - Grayscale bars for data
  - Green/red only for variance indicators
  - Clean axes with subtle grid lines
  - Year labels on X-axis

---

## Tasks / Subtasks

- [x] **Task 1: Create YoYComparisonChart Component** (AC: 1, 5)
  - [x] Create `components/charts/YoYComparisonChart.tsx`
  - [x] Accept yearly data array with metrics
  - [x] Render grouped bar chart using Recharts
  - [x] Apply IBCS grayscale styling for bars

- [x] **Task 2: Add ΔPY Variance Indicators** (AC: 2)
  - [x] Calculate ΔPY for each metric per year
  - [x] Add variance indicator colors in tooltip
  - [x] Color-code: green for positive, red for negative

- [x] **Task 3: Add ΔPY% Badges** (AC: 3)
  - [x] Calculate percentage change from previous year
  - [x] Render badge labels above bars via LabelList
  - [x] Style with IBCS variance colors

- [x] **Task 4: Add Click-to-Drill Functionality** (AC: 4)
  - [x] Add click handler on bar cells
  - [x] Show expanded detail panel for clicked year
  - [x] Selected year highlighting with opacity dimming

- [x] **Task 5: Integrate with Finance View** (AC: 1-5)
  - [x] Add YoYComparisonChart to FinanceView
  - [x] Conditionally render for multi-year scenarios
  - [x] Use same yearly data from Story 2.6 API

---

## Dev Notes

### Reuse Story 2.6 API

The `/api/views/finance/yearly` endpoint from Story 2.6 already provides:
- `yearlyData[]` with revenue, profit, costs per year
- This can be reused for the comparison chart

### Variance Calculation

For each year (except first), calculate:
```typescript
const variancePY = currentYear.revenue - previousYear.revenue
const variancePYPercent = ((currentYear.revenue - previousYear.revenue) / previousYear.revenue) * 100
```

### Recharts Grouped Bar

```tsx
<BarChart data={yearlyData}>
  <Bar dataKey="revenueAC" fill={IBCS_COLORS.ACTUAL} name="Revenue" />
  <Bar dataKey="profitAC" fill={IBCS_COLORS.FORECAST} name="Profit" />
  <Bar dataKey="costsAC" fill={IBCS_COLORS.PREVIOUS_YEAR} name="Costs" />
</BarChart>
```

### Files to Modify

| File | Change |
|------|--------|
| `components/charts/YoYComparisonChart.tsx` | NEW: Grouped bar chart component |
| `components/results/FinanceView.tsx` | Add YoYComparisonChart integration |

### Dependencies

- Story 2.6 (done): `/api/views/finance/yearly` endpoint
- Story 2.5 (done): Annual summaries concept

---

## References

- [Source: docs/epics/epic-02-multi-year-scenarios.md#Story 2.7]
- [Source: components/charts/MultiYearTrendChart.tsx] - Similar component from Story 2.6
- [Source: lib/constants/ibcs-colors.ts] - IBCS color constants

---

## Dev Agent Record

### Context Reference

Story context created via BMM create-story workflow 2026-01-21

### Agent Model Used

Claude Opus 4.5 (claude-opus-4-5-20251101)

### Debug Log References

N/A - No errors encountered

### Completion Notes List

1. **Created YoYComparisonChart Component** (`components/charts/YoYComparisonChart.tsx`)
   - Grouped bar chart using Recharts with IBCS styling
   - Revenue, Profit, Costs bars grouped by year
   - Metric filter buttons (All, Revenue, Profit, Costs)
   - calculateVariances() function for ΔPY and ΔPY% calculation
   - Custom tooltip showing values and variance percentages
   - VarianceBadge component for ΔPY% labels above bars
   - Color-coded variance: green for positive, red for negative
   - Click-to-select year with detail panel showing breakdown
   - Selected year highlighting with opacity dimming on other bars

2. **Integrated with FinanceView** (`components/results/FinanceView.tsx`)
   - Imported YoYComparisonChart component
   - Conditionally renders after MultiYearTrendChart
   - Only shown for multi-year scenarios (durationYears > 1)
   - Reuses yearlyData from Story 2.6 API

### File List

| File | Change |
|------|--------|
| `components/charts/YoYComparisonChart.tsx` | NEW: YoY comparison grouped bar chart |
| `components/results/FinanceView.tsx` | Added YoYComparisonChart integration |
| `docs/sprint-artifacts/2-7-yoy-comparison-chart.md` | Created and marked done |
