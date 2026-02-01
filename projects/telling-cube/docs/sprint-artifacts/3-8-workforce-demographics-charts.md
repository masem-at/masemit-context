# Story 3.8: Workforce Demographics Charts

**Status:** done
**Epic:** 3 - HR Data Domain
**Points:** 3
**Created:** 2026-01-22

---

## Story

As a **user viewing the HR tab**,
I want **IBCS-styled visualizations for age distribution and tenure distribution**,
so that **I can analyze workforce demographics and understand employee composition**.

---

## Acceptance Criteria

1. **AC1: Age Bracket Histogram**
   - [ ] Horizontal bar chart showing employee count by age bracket
   - [ ] Age brackets: 20-29, 30-39, 40-49, 50-59, 60+
   - [ ] IBCS monochrome styling (gray-700)
   - [ ] Sorted by age bracket (ascending)
   - [ ] Shows count and percentage for each bracket

2. **AC2: Tenure Distribution Chart**
   - [ ] Horizontal bar chart showing employee count by tenure range
   - [ ] Tenure ranges: <1 year, 1-2 years, 3-5 years, 6-10 years, 10+ years
   - [ ] IBCS monochrome styling (gray-700)
   - [ ] Sorted by tenure range (ascending)
   - [ ] Shows count and percentage for each range

3. **AC3: Integration with HRView**
   - [ ] New charts displayed in "Workforce Demographics" section
   - [ ] Replaces or enhances existing gender distribution section
   - [ ] Maintains responsive design
   - [ ] Privacy disclaimer remains visible

4. **AC4: Data Derivation**
   - [ ] Age calculated from `birthYear` vs current year
   - [ ] Tenure calculated from `hireDate` vs current date
   - [ ] Only active employees (no terminationDate) included

---

## Tasks / Subtasks

### Task 1: Create IBCSDemographicsChart Component (AC: #1, #2)
- [x] 1.1 Create `components/charts/IBCSDemographicsChart.tsx`
- [x] 1.2 Define props interface for generic bracket-based chart
- [x] 1.3 Implement table-based horizontal bars (matching IBCSHeadcountChart pattern)
- [x] 1.4 Add IBCS monochrome styling (gray-700 bars)
- [x] 1.5 Show count and percentage columns
- [x] 1.6 Add empty state handling

### Task 2: Add Demographics Data to HRView API (AC: #4)
- [x] 2.1 Add `getAgeBracketDistribution()` function to `hr-queries.ts`
- [x] 2.2 Add `getTenureDistribution()` function to `hr-queries.ts`
- [x] 2.3 Add new types: `AgeBracket`, `TenureRange` to `types/api.ts`
- [x] 2.4 Update `HRViewResponse` to include demographics arrays
- [x] 2.5 Update `/api/views/hr` to include new data

### Task 3: Integrate Charts with HRView (AC: #3)
- [x] 3.1 Import IBCSDemographicsChart in HRView.tsx
- [x] 3.2 Add Age Distribution chart to demographics section
- [x] 3.3 Add Tenure Distribution chart to demographics section
- [x] 3.4 Reorganize gender + age + tenure under "Workforce Demographics" header
- [x] 3.5 Test responsive layout

---

## Dev Notes

### Current Implementation Analysis

**HRView.tsx** already has a "Workforce Demographics" section (lines 208-278) showing:
- Gender distribution as stacked horizontal bar
- Legend with Male/Female/Diverse counts
- This should be **enhanced**, not replaced

**Decision:** Add age and tenure charts below existing gender distribution, keeping the unified "Workforce Demographics" section.

### Data Sources

From `lib/types/scenario-world.ts` (Employee interface):
```typescript
birthYear: number      // Year only, NOT full DOB (privacy)
hireDate: string       // Format: "YYYY-MM"
gender: EmployeeGender // M/F/D
terminationDate?: string // Only set if employee left
```

From `lib/queries/hr-queries.ts`:
- `getHRSummary()` already filters active employees
- Pattern to follow for new query functions

### Age Bracket Calculation

```typescript
const currentYear = scenario.timePeriodEnd.getFullYear()
const age = currentYear - employee.birthYear
const bracket =
  age < 20 ? '<20' :
  age < 30 ? '20-29' :
  age < 40 ? '30-39' :
  age < 50 ? '40-49' :
  age < 60 ? '50-59' : '60+'
```

### Tenure Calculation

```typescript
const currentDate = scenario.timePeriodEnd
const [hireYear, hireMonth] = employee.hireDate.split('-').map(Number)
const tenureMonths = (currentYear - hireYear) * 12 + (currentMonth - hireMonth)
const tenureYears = tenureMonths / 12

const range =
  tenureYears < 1 ? '<1 year' :
  tenureYears < 3 ? '1-2 years' :
  tenureYears < 6 ? '3-5 years' :
  tenureYears < 11 ? '6-10 years' : '10+ years'
```

### IBCS Horizontal Bar Pattern

Reference: `components/charts/IBCSHeadcountChart.tsx`:
```typescript
function getBarWidth(value: number, maxValue: number): number {
  if (maxValue === 0) return 0
  return Math.min((value / maxValue) * 100, 100)
}

// Bar styling
<div className="h-6 bg-gray-700 rounded-sm transition-all"
     style={{ width: `${barWidth}%`, minWidth: barWidth > 0 ? '4px' : '0' }} />
```

### Type Definitions to Add

```typescript
// types/api.ts
export interface AgeBracket {
  bracket: string      // "20-29", "30-39", etc.
  count: number
  percentage: number
}

export interface TenureRange {
  range: string        // "<1 year", "1-2 years", etc.
  count: number
  percentage: number
}

// Update HRViewResponse
export interface HRViewResponse {
  summary: HRSummary
  headcountByDepartment: HeadcountByDepartment[]
  headcountByMonth: HeadcountByMonth[]
  salaryByDepartment: SalaryByDepartment[]
  turnoverByMonth: TurnoverByMonth[]
  ageBrackets: AgeBracket[]        // NEW
  tenureRanges: TenureRange[]      // NEW
}
```

### Previous Story Learnings (3.7)

From Story 3.7 code review:
- Use IBCS monochrome colors consistently (`#374151` / gray-700)
- Add empty state check for zero data
- Keep table-based horizontal bar pattern for consistency with other HR charts
- Update sprint-status.yaml in file list

### Files to Modify

**Files to create:**
- `components/charts/IBCSDemographicsChart.tsx` - Reusable bracket/range chart

**Files to modify:**
- `lib/queries/hr-queries.ts` - Add age/tenure query functions
- `types/api.ts` - Add AgeBracket, TenureRange types, update HRViewResponse
- `app/api/views/hr/route.ts` - Include new demographics data
- `components/results/HRView.tsx` - Add demographics charts
- `docs/sprint-artifacts/sprint-status.yaml` - Story status tracking

### Testing Considerations

1. Test with various age distributions (young startup vs mature enterprise)
2. Test with different tenure patterns
3. Test empty employee list (should show empty state)
4. Test single-employee scenario
5. Verify calculations match expected brackets/ranges
6. Test mobile viewport responsiveness

### Component Design Decision

**Option A:** Single reusable `IBCSDemographicsChart` with props for title, data, etc.
**Option B:** Separate `IBCSAgeChart` and `IBCSTenureChart` components

**Recommendation:** Option A - Create a generic bracket chart component that accepts:
```typescript
interface IBCSDemographicsChartProps {
  data: Array<{ label: string; count: number; percentage: number }>
  title: string
  subtitle?: string
}
```

This follows DRY principle and can be reused for future demographic visualizations.

---

## Dev Agent Record

### Agent Model Used

Claude Opus 4.5

### Completion Notes List

- [x] Story created via create-story workflow
- [x] IBCSDemographicsChart component created
- [x] hr-queries.ts updated with age/tenure functions
- [x] types/api.ts updated with new interfaces
- [x] API endpoint returns new data
- [x] HRView updated with demographics charts
- [x] TypeScript compilation passes

### File List

**Files created:**
- `components/charts/IBCSDemographicsChart.tsx` - Reusable demographics chart

**Files modified:**
- `lib/queries/hr-queries.ts` - Added getAgeBracketDistribution, getTenureDistribution, AgeBracket, TenureRange types
- `types/api.ts` - Added AgeBracket, TenureRange interfaces, updated HRViewResponse
- `app/api/views/hr/route.ts` - Returns ageBrackets and tenureRanges data
- `components/results/HRView.tsx` - Integrated demographics charts with section header
- `docs/sprint-artifacts/sprint-status.yaml` - Story status tracking

### Change Log

- 2026-01-22: Story 3.8 created via create-story workflow (ultimate context engine)
- 2026-01-22: Story 3.8 implemented - Age/tenure distribution charts (Claude Opus 4.5)
