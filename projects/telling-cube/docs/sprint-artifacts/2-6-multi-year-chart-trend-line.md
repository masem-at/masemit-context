# Story 2.6: Multi-Year Chart - Trend Line

**Status:** done
**Epic:** 2 - Multi-Year Scenarios (3-5 Years)
**Points:** 5
**Created:** 2026-01-21

---

## Story

As a **data analyst viewing multi-year scenarios**,
I want **a line chart showing 3-5 year trends for key financial metrics**,
so that **I can visualize long-term business trajectories and compare actual vs planned performance**.

---

## Acceptance Criteria

### AC1: Line Chart with Year Granularity
**Given** a 3-year or 5-year scenario
**When** the trend chart is displayed
**Then** X-axis shows years (2026, 2027, 2028, etc.)
**And** data points are aggregated by year

### AC2: Key Metrics Display
**Given** the trend chart is rendered
**When** viewing the chart
**Then** the following metrics are displayed as lines:
  - Revenue (AC)
  - Profit (AC)
  - Costs (AC)

### AC3: AC vs PL Comparison Lines
**Given** a multi-year scenario with Plan data
**When** the trend chart is displayed
**Then** Revenue PL and Profit PL are shown as dashed lines
**And** AC lines are solid, PL lines are dashed (IBCS standard)

### AC4: CAGR Annotation on Chart
**Given** a multi-year trend chart
**When** viewing Revenue or Profit lines
**Then** CAGR percentage is displayed as an annotation
**And** CAGR is calculated from first to last year

### AC5: Year Selector/Zoom Capability
**Given** a 5-year scenario trend chart
**When** the user wants to focus on specific years
**Then** a year range selector or zoom control is available
**And** the chart updates to show only selected years

### AC6: IBCS Styling
**Given** the trend chart component
**When** rendered
**Then** follows IBCS standards:
  - Grayscale color palette
  - Clean axes with subtle grid lines
  - IBCS scenario notation (AC, PL) in legend
  - Dashed lines for PL (Plan)

---

## Tasks / Subtasks

- [x] **Task 1: Create MultiYearTrendChart Component** (AC: 1, 2, 6)
  - [x] Create `components/charts/MultiYearTrendChart.tsx`
  - [x] Accept data with yearly aggregations (revenue, profit, costs)
  - [x] Extend existing LineChart base or create specialized version
  - [x] Apply IBCS grayscale styling

- [x] **Task 2: Add AC vs PL Comparison Lines** (AC: 3)
  - [x] Add PL (Plan) lines with dashed styling
  - [x] AC lines remain solid per IBCS standard
  - [x] Use IBCS colors from `lib/constants/ibcs-colors.ts`

- [x] **Task 3: Add CAGR Annotation** (AC: 4)
  - [x] Import calculateCAGR from `lib/utils/time-dimensions.ts`
  - [x] Calculate CAGR for Revenue and Profit
  - [x] Display as annotation in header section
  - [x] Format as percentage with 1 decimal (e.g., "CAGR: 18.5%")

- [x] **Task 4: Add Year Selector/Zoom** (AC: 5)
  - [x] Add year range selector buttons (All, 3Y, 2Y, 1Y)
  - [x] Allow filtering visible years
  - [x] Default to showing all years

- [x] **Task 5: Integrate with Finance View** (AC: 1-6)
  - [x] Add to FinanceView component for multi-year scenarios
  - [x] Conditionally render only when durationYears > 1
  - [x] Pass yearly data and CAGR from API

- [x] **Task 6: Create API Endpoint for Yearly Aggregation** (AC: 1, 2)
  - [x] Create `/api/views/finance/yearly?scenarioId=xxx`
  - [x] Return yearly aggregated data with AC and PL
  - [x] Use calculateCAGR from time-dimensions helpers

---

## Dev Notes

### Existing LineChart Component

The existing `components/charts/LineChart.tsx` uses Recharts and IBCS styling:
- Supports multiple line series with scenario types
- Uses `IBCS_COLORS` and `getScenarioColor` helpers
- Dashed lines for FORECAST scenario type

### Data Structure for Trend Chart

```typescript
interface YearlyTrendData {
  year: number          // 2026, 2027, etc.
  revenueAC: number     // Actual revenue
  revenuePL: number     // Planned revenue
  profitAC: number      // Actual profit
  profitPL: number      // Planned profit
  costsAC: number       // Actual costs (COGS + OpEx)
  costsPL: number       // Planned costs
}
```

### CAGR Annotation Approach

Use Recharts' `ReferenceLine` or custom label to display CAGR:
```tsx
<ReferenceLine
  label={{
    value: `CAGR: ${cagr}%`,
    position: 'insideRight'
  }}
/>
```

Or add custom annotation layer after the chart.

### Year Selector Options

1. **Button Group**: [All | 3Y | 2Y | 1Y] - simple filter
2. **Slider**: Range slider for custom year selection
3. **Brush**: Recharts Brush component for zooming

Recommend Button Group for simplicity.

### Files to Create/Modify

| File | Change |
|------|--------|
| `components/charts/MultiYearTrendChart.tsx` | NEW: Main trend chart component |
| `app/api/views/finance/yearly/route.ts` | NEW: Yearly aggregation API |
| `components/results/FinanceView.tsx` | Add MultiYearTrendChart for multi-year |

### Dependencies

- Story 2.4 (done): `calculateCAGR()` helper
- Story 2.5 (done): Annual summaries concept (similar aggregation)
- Recharts library (already installed)

---

## References

- [Source: docs/epics/epic-02-multi-year-scenarios.md#Story 2.6]
- [Source: components/charts/LineChart.tsx] - Base chart component
- [Source: lib/constants/ibcs-colors.ts] - IBCS color constants
- [Source: lib/utils/time-dimensions.ts] - calculateCAGR helper

---

## Dev Agent Record

### Context Reference

Story context created via BMM create-story workflow 2026-01-21

### Agent Model Used

Claude Opus 4.5 (claude-opus-4-5-20251101)

### Debug Log References

N/A - No errors encountered

### Completion Notes List

1. **Created Yearly Finance API** (`app/api/views/finance/yearly/route.ts`)
   - New endpoint: GET `/api/views/finance/yearly?scenarioId=xxx`
   - Returns: scenarioId, durationYears, startYear, endYear, yearlyData[], cagr
   - Aggregates events by year (revenue, costs, profit)
   - Generates PL (Plan) comparison values using existing generateComparisonData utility
   - Calculates CAGR using time-dimensions helper

2. **Created MultiYearTrendChart Component** (`components/charts/MultiYearTrendChart.tsx`)
   - IBCS-compliant line chart using Recharts
   - Shows Revenue, Profit, Costs with AC (solid) and PL (dashed) lines
   - Year range filter: All, 3Y, 2Y, 1Y buttons
   - Metric toggles to show/hide Revenue, Profit, Costs
   - CAGR display in header section
   - Custom tooltip with Euro formatting
   - IBCS grayscale color palette

3. **Integrated with FinanceView** (`components/results/FinanceView.tsx`)
   - Added YearlyFinanceData type definition
   - Parallel fetch for finance data and yearly data
   - Conditionally renders MultiYearTrendChart when durationYears > 1
   - Renders between P&L chart and Cost Breakdown table

### File List

| File | Change |
|------|--------|
| `app/api/views/finance/yearly/route.ts` | NEW: Yearly aggregation API endpoint |
| `components/charts/MultiYearTrendChart.tsx` | NEW: Multi-year trend line chart component |
| `components/results/FinanceView.tsx` | Added multi-year chart integration |
| `docs/sprint-artifacts/2-6-multi-year-chart-trend-line.md` | Created and marked done |
