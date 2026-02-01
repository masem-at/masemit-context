# Story 2.4: Time Dimension Extensions

**Status:** done
**Epic:** 2 - Multi-Year Scenarios (3-5 Years)
**Points:** 3
**Created:** 2026-01-21

---

## Story

As a **data analyst using tellingCube**,
I want **time dimension fields (fiscalYear, fiscalQuarter, yearIndex) on aggregated data**,
so that **I can perform year-over-year comparisons and filter by fiscal periods**.

---

## Acceptance Criteria

### AC1: fiscalYear Field on Aggregations
**Given** monthly data with timePeriod "2024-03"
**When** the time dimensions are calculated
**Then** `fiscalYear` = 2024

### AC2: fiscalQuarter Calculated
**Given** timePeriod "2024-03"
**When** the time dimensions are calculated
**Then** `fiscalQuarter` = "Q1"

**Given** timePeriod "2024-06"
**When** the time dimensions are calculated
**Then** `fiscalQuarter` = "Q2"

### AC3: yearIndex for Position in Range
**Given** a 3-year scenario starting 2024
**And** timePeriod "2025-06"
**When** the time dimensions are calculated
**Then** `yearIndex` = 2 (second year in range)

### AC4: isCurrentYear Boolean Flag
**Given** current year is 2026
**And** timePeriod "2026-03"
**When** the time dimensions are calculated
**Then** `isCurrentYear` = true

**Given** timePeriod "2024-03"
**When** the time dimensions are calculated
**Then** `isCurrentYear` = false

### AC5: CAGR Calculation Helper
**Given** Year 1 revenue = €1,000,000
**And** Year 3 revenue = €1,440,000
**When** CAGR is calculated
**Then** CAGR = 20% (approx: (1.44^(1/2) - 1) * 100)

---

## Tasks / Subtasks

- [x] **Task 1: Create Time Dimension Types** (AC: 1-4)
  - [x] Add `TimeDimensions` interface to types
  - [x] Include fiscalYear, fiscalQuarter, yearIndex, isCurrentYear

- [x] **Task 2: Create Time Dimension Helpers** (AC: 1-5)
  - [x] Create `lib/utils/time-dimensions.ts`
  - [x] Implement `getTimeDimensions(timePeriod, scenarioStart)` function
  - [x] Implement `calculateCAGR(startValue, endValue, years)` function
  - [x] Implement `getFiscalQuarter(month)` helper

- [x] **Task 3: Export Time Dimensions from Module** (AC: 1-5)
  - [x] Direct module import (no index file needed - follows existing pattern)
  - [x] Add JSDoc documentation

---

## Dev Notes

### Time Dimension Helper Functions

```typescript
// lib/utils/time-dimensions.ts

export interface TimeDimensions {
  fiscalYear: number;      // 2024, 2025, etc.
  fiscalQuarter: string;   // "Q1", "Q2", "Q3", "Q4"
  yearIndex: number;       // 1-5 (position in multi-year range)
  isCurrentYear: boolean;  // true if fiscalYear === current year
}

export function getTimeDimensions(
  timePeriod: string,    // "2024-03"
  scenarioStartYear: number,  // 2024
  durationYears: number  // 1, 3, or 5
): TimeDimensions {
  const [yearStr, monthStr] = timePeriod.split('-')
  const year = parseInt(yearStr, 10)
  const month = parseInt(monthStr, 10)

  return {
    fiscalYear: year,
    fiscalQuarter: getFiscalQuarter(month),
    yearIndex: year - scenarioStartYear + 1,
    isCurrentYear: year === new Date().getFullYear(),
  }
}

export function getFiscalQuarter(month: number): string {
  if (month <= 3) return 'Q1'
  if (month <= 6) return 'Q2'
  if (month <= 9) return 'Q3'
  return 'Q4'
}

export function calculateCAGR(
  startValue: number,
  endValue: number,
  years: number
): number {
  if (startValue <= 0 || years <= 0) return 0
  return (Math.pow(endValue / startValue, 1 / years) - 1) * 100
}
```

### Integration Points

These helpers will be used by:
- `lib/queries/finance-queries.ts` - Add time dims to monthly aggregations
- `lib/queries/sales-queries.ts` - Add time dims to revenue by month
- Export API routes - Include time dimensions in responses

### Fiscal Year Assumption

Using calendar year as fiscal year (Jan-Dec). If fiscal year differs from calendar year (e.g., Apr-Mar), this would need configuration per company.

---

## References

- [Source: docs/epics/epic-02-multi-year-scenarios.md#Story 2.4]
- [Source: docs/future/spec-multi-year-scenarios.md#Phase 3: Data Dimensions]
- [Source: lib/queries/finance-queries.ts] - Current aggregation queries
- [Source: types/api.ts] - Current API types

---

## Dev Agent Record

### Context Reference

Story context created via BMM create-story workflow 2026-01-21

### Agent Model Used

Claude Opus 4.5 (claude-opus-4-5-20251101)

### Debug Log References

N/A - No errors encountered

### Completion Notes List

1. **Created Time Dimension Utilities** (`lib/utils/time-dimensions.ts`)
   - `TimeDimensions` interface: fiscalYear, fiscalQuarter, yearIndex, isCurrentYear
   - `ExtendedTimeDimensions` interface: Adds timePeriod, monthOfYear, isFirstYear, isLastYear
   - `getFiscalQuarter(month)`: Returns Q1-Q4 based on month (1-12)
   - `getTimeDimensions(timePeriod, scenarioStartYear, durationYears)`: Calculates all time dimensions
   - `getExtendedTimeDimensions()`: Full context with comparison flags
   - `calculateCAGR(startValue, endValue, years)`: Compound Annual Growth Rate as percentage
   - `getPreviousYearPeriod(timePeriod, yearsBack)`: Get prior year period for YoY comparisons
   - `isWithinScenarioRange()`: Check if period falls within scenario bounds
   - `generateTimePeriods(startYear, durationYears)`: Generate all YYYY-MM periods for scenario

2. **Comprehensive JSDoc Documentation**
   - All functions include @param, @returns, and @example tags
   - Interface fields documented with inline comments
   - References to epic/story documentation

### File List

| File | Change |
|------|--------|
| `lib/utils/time-dimensions.ts` | NEW: Time dimension helper functions and types |
| `docs/sprint-artifacts/2-4-time-dimension-extensions.md` | Updated status to done |
