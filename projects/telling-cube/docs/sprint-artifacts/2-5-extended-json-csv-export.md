# Story 2.5: Extended JSON/CSV Export

**Status:** done
**Epic:** 2 - Multi-Year Scenarios (3-5 Years)
**Points:** 3
**Created:** 2026-01-21

---

## Story

As a **data analyst using tellingCube exports**,
I want **multi-year metadata and annual summaries in export files**,
so that **I can analyze year-over-year trends and import data into external tools with proper time context**.

---

## Acceptance Criteria

### AC1: Export includes Multi-Year Metadata
**Given** a 3-year scenario with `durationYears = 3`
**When** the finance.json or events.json export is downloaded
**Then** the response includes:
  - `durationYears: 3`
  - `startYear: 2026`
  - `endYear: 2028`

### AC2: Annual Summaries Array
**Given** a multi-year scenario export
**When** the finance.json export is downloaded
**Then** `annualSummaries` array contains:
  - One entry per year with: { year, revenue, cogs, payroll, profit }
  - All amounts in EUR

### AC3: CAGR Object in Export
**Given** a 3-year scenario with revenue growth
**When** the finance.json export is downloaded
**Then** `cagr` object contains:
  - `revenue`: CAGR for revenue (%)
  - `profit`: CAGR for profit (%)
  - Calculated using the `calculateCAGR()` helper from Story 2.4

### AC4: All Months in Monthly Data
**Given** a 5-year scenario
**When** the export is downloaded
**Then** `monthlyData` or events array contains 60 months of data
**And** each month follows the YYYY-MM format

### AC5: CSV Export Has Year Column
**Given** an events.csv export for a multi-year scenario
**When** the file is opened
**Then** a `Year` column exists after `Time Period`
**And** the year is extracted from the time period (e.g., 2026)

---

## Tasks / Subtasks

- [x] **Task 1: Add Scenario Metadata to JSON Exports** (AC: 1)
  - [x] Update `events.json/route.ts` to include durationYears, startYear, endYear
  - [x] Update `finance.json/route.ts` to include durationYears, startYear, endYear
  - [x] Query scenario record for timePeriodStart, timePeriodEnd, durationYears

- [x] **Task 2: Add Annual Summaries to Finance Export** (AC: 2)
  - [x] Create helper function to aggregate monthly data by year
  - [x] Add `annualSummaries` array to FinanceViewResponse
  - [x] Include revenue, cogs, payroll, profit per year

- [x] **Task 3: Add CAGR Calculation to Finance Export** (AC: 3)
  - [x] Import `calculateCAGR` from `lib/utils/time-dimensions.ts`
  - [x] Calculate revenue CAGR from first to last year
  - [x] Calculate profit CAGR from first to last year
  - [x] Add `cagr` object to export response

- [x] **Task 4: Add Year Column to CSV Export** (AC: 5)
  - [x] Add `year` column to CSV_COLUMNS definition
  - [x] Extract year from timePeriod (split on '-', take first part)
  - [x] Ensure column is positioned after Time Period

---

## Dev Notes

### Current Export Structure

**events.json** returns:
```json
{
  "scenarioId": "uuid",
  "eventCount": 3000,
  "events": [...]
}
```

**finance.json** returns:
```json
{
  "plSummary": {...},
  "monthlyCashFlow": [...],
  "costBreakdown": {...},
  "marginAnalysis": {...}
}
```

### Target Export Structure

**events.json** should return:
```json
{
  "scenarioId": "uuid",
  "durationYears": 3,
  "startYear": 2026,
  "endYear": 2028,
  "eventCount": 9000,
  "events": [...]
}
```

**finance.json** should return:
```json
{
  "scenarioId": "uuid",
  "durationYears": 3,
  "startYear": 2026,
  "endYear": 2028,
  "annualSummaries": [
    { "year": 2026, "revenue": 1200000, "cogs": 480000, "payroll": 360000, "profit": 360000 },
    { "year": 2027, "revenue": 1440000, "cogs": 576000, "payroll": 432000, "profit": 432000 },
    { "year": 2028, "revenue": 1670000, "cogs": 668000, "payroll": 501000, "profit": 501000 }
  ],
  "cagr": {
    "revenue": 18.0,
    "profit": 18.0
  },
  "plSummary": {...},
  "monthlyCashFlow": [...],
  "costBreakdown": {...},
  "marginAnalysis": {...}
}
```

### CSV Column Addition

Current CSV columns include `Time Period` (YYYY-MM format).
Add `Year` column extracted from time period:

```typescript
const CSV_COLUMNS = [
  // ... existing columns
  { key: 'timePeriod', header: 'Time Period' },
  { key: 'year', header: 'Year' },  // NEW: 2026, 2027, etc.
  // ... rest of columns
]
```

### Dependencies

- Story 2.4 (done): `calculateCAGR()` helper function available
- Prisma schema: `durationYears`, `timePeriodStart`, `timePeriodEnd` fields exist

### Files to Modify

| File | Change |
|------|--------|
| `app/api/export/[scenarioId]/events.json/route.ts` | Add scenario metadata |
| `app/api/export/[scenarioId]/finance.json/route.ts` | Add metadata + annual summaries + CAGR |
| `app/api/export/[scenarioId]/events.csv/route.ts` | Add Year column |

---

## References

- [Source: docs/epics/epic-02-multi-year-scenarios.md#Story 2.5]
- [Source: lib/utils/time-dimensions.ts] - calculateCAGR helper
- [Source: app/api/export/[scenarioId]/finance.json/route.ts] - Current finance export
- [Source: app/api/export/[scenarioId]/events.csv/route.ts] - Current CSV export

---

## Dev Agent Record

### Context Reference

Story context created via BMM create-story workflow 2026-01-21

### Agent Model Used

Claude Opus 4.5 (claude-opus-4-5-20251101)

### Debug Log References

N/A - No errors encountered

### Completion Notes List

1. **Events JSON Export Updated** (`app/api/export/[scenarioId]/events.json/route.ts`)
   - Added Prisma import for scenario queries
   - Query scenario for durationYears, timePeriodStart, timePeriodEnd
   - Include durationYears, startYear, endYear in response

2. **Finance JSON Export Updated** (`app/api/export/[scenarioId]/finance.json/route.ts`)
   - Added calculateCAGR import from time-dimensions utility
   - Created AnnualSummary interface and calculateAnnualSummaries helper
   - Parallel query for scenario metadata
   - Generate annualSummaries array for multi-year scenarios
   - Calculate revenue and profit CAGR for 2+ year scenarios
   - Extended response with scenarioId, durationYears, startYear, endYear, annualSummaries, cagr

3. **Events CSV Export Updated** (`app/api/export/[scenarioId]/events.csv/route.ts`)
   - Added `year` column to CSV_COLUMNS after Time Period
   - Extract year from timePeriod using split('-')[0]

### File List

| File | Change |
|------|--------|
| `app/api/export/[scenarioId]/events.json/route.ts` | Added multi-year metadata (durationYears, startYear, endYear) |
| `app/api/export/[scenarioId]/finance.json/route.ts` | Added metadata + annualSummaries + CAGR calculation |
| `app/api/export/[scenarioId]/events.csv/route.ts` | Added Year column after Time Period |
| `docs/sprint-artifacts/2-5-extended-json-csv-export.md` | Created and marked done |
