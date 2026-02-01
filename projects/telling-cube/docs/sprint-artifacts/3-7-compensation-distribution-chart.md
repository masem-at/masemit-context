# Story 3.7: Compensation Distribution Chart

**Status:** done
**Epic:** 3 - HR Data Domain
**Points:** 3
**Created:** 2026-01-22

---

## Story

As a **user viewing the HR tab**,
I want **IBCS-styled visualizations for salary distribution and payroll trends**,
so that **I can analyze compensation patterns across departments and over time**.

---

## Acceptance Criteria

1. **AC1: IBCS Compensation Chart Component**
   - [ ] New `IBCSCompensationChart.tsx` in `/components/charts/`
   - [ ] Follows IBCS patterns (monochrome, minimal gridlines)
   - [ ] Uses Recharts library (existing dependency)

2. **AC2: Salary by Department Horizontal Bars**
   - [ ] Horizontal bar chart showing avg salary per department
   - [ ] Sorted by salary (highest first)
   - [ ] IBCS monochrome styling (gray-700)
   - [ ] Values displayed at bar end with EUR formatting

3. **AC3: Cost per FTE KPI Card**
   - [ ] New KPI card showing total payroll / headcount
   - [ ] Positioned in KPI grid alongside existing cards
   - [ ] Uses same styling as other KPI cards
   - [ ] Shows monthly cost per employee

4. **AC4: Currency Formatting Correct**
   - [ ] All monetary values use `formatScaledValue` with correct scale
   - [ ] Scale unit displayed (K, M) appropriately
   - [ ] EUR currency symbol where needed

5. **AC5: Integration with HRView**
   - [ ] Replace current salary table with visual chart
   - [ ] Add Cost per FTE to KPI card grid
   - [ ] Maintain responsive design

---

## Tasks / Subtasks

### Task 1: Create IBCSCompensationChart Component (AC: #1, #2, #4)
- [ ] 1.1 Create `components/charts/IBCSCompensationChart.tsx`
- [ ] 1.2 Define props interface accepting `SalaryByDepartment[]`
- [ ] 1.3 Implement Recharts horizontal BarChart
- [ ] 1.4 Add IBCS monochrome styling
- [ ] 1.5 Sort data by avgSalary descending
- [ ] 1.6 Format values with EUR and scale unit

### Task 2: Add Cost per FTE KPI Card (AC: #3)
- [ ] 2.1 Calculate `costPerFTE = totalPayroll / headcount`
- [ ] 2.2 Add 5th KPI card to grid (adjust from 4-col to 5-col or keep 4 with overflow)
- [ ] 2.3 Use Banknote icon (already imported)
- [ ] 2.4 Display with proper formatting

### Task 3: Integrate with HRView (AC: #5)
- [ ] 3.1 Import IBCSCompensationChart in HRView.tsx
- [ ] 3.2 Replace table section with chart (or keep both)
- [ ] 3.3 Add Cost per FTE card to KPI grid
- [ ] 3.4 Test responsive layout

---

## Dev Notes

### Current Implementation Analysis

HRView.tsx (lines 184-241) already displays `salaryByDepartment` as a **table**:
- Shows department, headcount, avg salary, min, max
- Uses `formatScaledValue` for proper formatting
- Data is already fetched and available

**Decision needed:** Replace table with horizontal bar chart, or add chart alongside table?
**Recommendation:** Replace table with IBCS horizontal bar chart for consistency with other charts. The min/max can be shown as range indicators or tooltips.

### Data Structure Available

From `types/api.ts` (SalaryByDepartment):
```typescript
export interface SalaryByDepartment {
  department: string
  avgSalary: number
  minSalary: number
  maxSalary: number
  headcount: number
  totalPayroll: number
}
```

### IBCS Horizontal Bar Pattern

Reference: `components/charts/IBCSHeadcountChart.tsx` (Story 3.5):
- Uses table-based horizontal bars (not Recharts)
- `bg-gray-700` for bars
- Width calculated as percentage: `(value / maxValue) * 100`
- Clean, minimal design

**Alternative:** Use Recharts BarChart with `layout="vertical"` for horizontal bars:
```typescript
<BarChart layout="vertical" data={sortedData}>
  <XAxis type="number" />
  <YAxis type="category" dataKey="department" />
  <Bar dataKey="avgSalary" fill="#374151" />
</BarChart>
```

### Cost per FTE Calculation

Already have:
- `summary.totalPayroll` - total monthly payroll
- `summary.headcount` - active employee count

Calculate:
```typescript
const costPerFTE = summary.headcount > 0
  ? summary.totalPayroll / summary.headcount
  : 0
```

### Existing Chart Patterns

Reference files:
- `components/charts/IBCSHeadcountChart.tsx` - Table-based horizontal bars
- `components/charts/IBCSTurnoverChart.tsx` - Recharts ComposedChart
- `components/charts/IBCSVarianceChart.tsx` - Recharts BarChart

### KPI Card Grid Layout

Current grid (line 106):
```tsx
<div className="grid grid-cols-2 md:grid-cols-4 gap-4">
```

Options for 5th card:
1. Keep 4-col grid, 5th card wraps to new row on mobile
2. Change to `md:grid-cols-5` (may be too cramped)
3. Use `md:grid-cols-4 lg:grid-cols-5`

**Recommendation:** Keep current grid, add 5th card - it will naturally wrap. The grid handles this well.

### Previous Story Learnings (3.6)

From Story 3.6 code review:
- Use IBCS monochrome colors consistently (`#374151` for all bars)
- Add `fontFamily: 'ui-monospace, monospace'` to axis ticks
- Include `minTickGap` for responsive handling
- Empty state check for zero data
- Update sprint-status.yaml in file list

### Files to Modify

- `components/charts/IBCSCompensationChart.tsx` - New chart component (CREATE)
- `components/results/HRView.tsx` - Add chart and KPI card
- `docs/sprint-artifacts/sprint-status.yaml` - Story status tracking

### Testing Considerations

1. Test with various department counts (2-10 departments)
2. Test salary ranges (thousands to hundreds of thousands)
3. Test empty salaryByDepartment array
4. Test mobile viewport
5. Verify Cost per FTE calculation accuracy

---

## Dev Agent Record

### Agent Model Used

Claude Opus 4.5

### Completion Notes List

- [x] Story created by SM (Bob) via create-story workflow
- [x] IBCSCompensationChart component created
- [x] Cost per FTE KPI card added
- [x] HRView updated with chart integration
- [x] TypeScript compilation passes

### File List

**Files created:**
- `components/charts/IBCSCompensationChart.tsx` - IBCS horizontal bar chart for compensation

**Files modified:**
- `components/results/HRView.tsx` - Added chart import, Cost per FTE KPI card, replaced table with chart
- `docs/sprint-artifacts/sprint-status.yaml` - Story status tracking

### Change Log

- 2026-01-22: Story 3.7 created via create-story workflow (ultimate context engine)
- 2026-01-22: Story 3.7 implemented - IBCSCompensationChart, Cost per FTE KPI (Claude Opus 4.5)
