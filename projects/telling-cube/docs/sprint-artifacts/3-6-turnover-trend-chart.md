# Story 3.6: Turnover Trend Chart

**Status:** done
**Epic:** 3 - HR Data Domain
**Points:** 3
**Created:** 2026-01-21

---

## Story

As a **user viewing the HR tab**,
I want **a professional IBCS-style chart showing hires, terminations, and net change over time**,
so that **I can understand workforce turnover trends and hiring/attrition patterns**.

---

## Acceptance Criteria

1. **AC1: Chart Component Created**
   - [x] New `IBCSTurnoverChart.tsx` component in `/components/charts/`
   - [x] Follows IBCS chart patterns (monochrome, minimal gridlines)
   - [x] Uses Recharts library (existing dependency)

2. **AC2: Combined Hires/Terminations Visualization**
   - [x] Hires shown as positive bars (above zero line)
   - [x] Terminations shown as negative bars (below zero line)
   - [x] Net change displayed as line overlay
   - [x] Monthly granularity on X-axis

3. **AC3: IBCS Visual Styling**
   - [x] Dark gray (`bg-gray-700`) for actual data bars
   - [x] Zero reference line visible
   - [x] Monospace numbers (`font-mono`)
   - [x] Minimal gridlines (horizontal only)
   - [x] Clear axis labels

4. **AC4: Annotations & Context**
   - [x] Turnover rate percentage shown (either as annotation or KPI)
   - [x] Legend clearly identifies Hires, Terminations, Net Change
   - [x] Responsive design (works on mobile)

5. **AC5: Integration with HRView**
   - [x] HRView imports and uses IBCSTurnoverChart
   - [x] Chart receives `HeadcountByMonth[]` and/or `TurnoverByMonth[]` data
   - [x] Positioned after Headcount by Department chart

---

## Tasks / Subtasks

### Task 1: Create IBCSTurnoverChart Component (AC: #1, #2, #3)
- [x] 1.1 Create `components/charts/IBCSTurnoverChart.tsx`
- [x] 1.2 Define props interface accepting HeadcountByMonth[] or TurnoverByMonth[]
- [x] 1.3 Implement Recharts ComposedChart with Bar and Line
- [x] 1.4 Add zero ReferenceLine
- [x] 1.5 Style with IBCS monochrome colors

### Task 2: Implement Data Visualization Logic (AC: #2, #4)
- [x] 2.1 Hires as positive bars (green-ish gray or distinct shade)
- [x] 2.2 Terminations as negative bars (inverted/below axis)
- [x] 2.3 Net change as line overlay
- [x] 2.4 Add turnover rate annotation or integrate with summary

### Task 3: Integrate with HRView (AC: #5)
- [x] 3.1 Import IBCSTurnoverChart in HRView.tsx
- [x] 3.2 Add chart section after headcount chart
- [x] 3.3 Pass headcountByMonth data (already fetched)

### Task 4: Responsive & Polish (AC: #4)
- [x] 4.1 Ensure chart works on mobile viewports
- [x] 4.2 Add empty state for missing data
- [x] 4.3 Test with multi-year scenarios

---

## Dev Notes

### Data Structures Available

From `types/api.ts` (lines 307-335):

```typescript
// HeadcountByMonth - Primary data source (more complete)
export interface HeadcountByMonth {
  month: string
  headcount: number
  hires: number
  terminations: number
  netChange: number
}

// TurnoverByMonth - Alternative (includes turnover rate)
export interface TurnoverByMonth {
  month: string
  turnoverRate: number
  hires: number
  terminations: number
}
```

**Recommendation:** Use `HeadcountByMonth[]` since it includes `netChange` which is needed for the line overlay. The `turnoverRate` from `TurnoverByMonth[]` can be shown separately as a KPI.

### Existing Chart Patterns to Follow

Reference: `components/charts/IBCSVarianceChart.tsx`:
- Uses Recharts `BarChart`, `Bar`, `XAxis`, `YAxis`, `CartesianGrid`, `ReferenceLine`, `Legend`
- IBCS colors: `#374151` (dark gray for actual), `#9CA3AF` (medium gray)
- Zero reference line: `<ReferenceLine y={0} stroke="#000" strokeWidth={1} />`
- Minimal gridlines: `<CartesianGrid strokeDasharray="3 3" stroke="#E5E7EB" vertical={false} />`

Reference: `components/charts/IBCSHeadcountChart.tsx` (Story 3.5):
- Table-based horizontal bars
- `bg-gray-700` for bars
- Responsive Tailwind classes (`text-xs sm:text-sm`)
- Empty state handling

### Component Structure Pattern

```typescript
interface IBCSTurnoverChartProps {
  data: HeadcountByMonth[]
  title?: string
  showTurnoverRate?: boolean  // Optional: display overall turnover %
}
```

### IBCS Styling Requirements

1. Bars: Dark solid for actuals (gray-700 or #374151)
2. Differentiate hires/terminations:
   - Option A: Same color, position distinguishes (hires positive, terms negative)
   - Option B: Slight shade variation (hires: gray-600, terms: gray-400)
3. Line: Black or dark gray for net change overlay
4. Numbers: `font-mono text-gray-900`
5. Labels: `text-sm text-gray-900`
6. No decorative colors - IBCS is monochrome

### HRView Integration Point

From `components/results/HRView.tsx`, the data is already available:

```typescript
const { summary, headcountByDepartment, salaryByDepartment } = data
// Also available but not currently rendered:
// data.headcountByMonth
// data.turnoverByMonth
```

Add the chart after the headcount chart section (around line 166):

```typescript
{/* Turnover Trend Chart */}
{headcountByMonth && headcountByMonth.length > 0 && (
  <>
    <div className="border-t border-gray-200" />
    <IBCSTurnoverChart data={headcountByMonth} />
  </>
)}
```

### API Already Provides Data

The `/api/views/hr` endpoint (line 62-68) already fetches `headcountByMonth` via `getHeadcountByMonth(scenarioId)`. No backend changes needed.

### Recharts Pattern for Positive/Negative Bars

For showing hires (positive) and terminations (negative) on same axis:

```typescript
// Data transformation
const chartData = data.map(d => ({
  month: d.month,
  hires: d.hires,
  terminations: -d.terminations,  // Negate for below-axis display
  netChange: d.netChange,
}))
```

### Previous Story Learnings (3.5)

From Story 3.5 code review:
- Avoid `style jsx` - use Tailwind responsive classes instead
- Add empty state check for zero data
- Don't import React explicitly (React 17+ JSX transform)
- Include sprint-status.yaml in file list

### Files to Reference

- `components/charts/IBCSVarianceChart.tsx` - Recharts pattern with ReferenceLine
- `components/charts/IBCSHeadcountChart.tsx` - Table-based IBCS pattern
- `components/charts/IBCSRegionalChart.tsx` - Bar chart styling reference
- `components/results/HRView.tsx` - Integration target
- `types/api.ts` (lines 307-335) - Data structure definitions

### Testing Considerations

1. Test with 1-year scenario (12 months)
2. Test with multi-year scenario (36-60 data points)
3. Test empty headcountByMonth array
4. Test mobile viewport
5. Verify turnover rate calculation if displayed

---

## Senior Developer Review (AI)

**Review Date:** 2026-01-21
**Reviewer:** Claude Opus 4.5 (Adversarial Code Review)
**Outcome:** Approve with fixes applied

### Issues Found & Resolved

| Severity | Issue | Resolution |
|----------|-------|------------|
| HIGH | H1: AC checkboxes not updated in Acceptance Criteria section | Marked all ACs as [x] |
| MEDIUM | M1: Terminations used different color (#6B7280) instead of IBCS monochrome | Changed to #374151 (gray-700) for both bars |
| MEDIUM | M2: Axis numbers not monospace | Added `fontFamily: 'ui-monospace, monospace'` to tick styles |
| MEDIUM | M3: Y-axis absolute values lack context | Documented - position below/above zero line provides context |
| MEDIUM | M4: XAxis rotation may cause overlap on mobile | Added `minTickGap={20}` for responsive handling |
| LOW | L1: Missing explicit return type on transformData | Added `ChartDataPoint[]` return type and interface |
| LOW | L2: Legend formatter used multiple if statements | Refactored to use `LEGEND_LABELS` object map |

### Action Items
All items resolved automatically - no outstanding action items.

---

## Dev Agent Record

### Agent Model Used

Claude Opus 4.5 (claude-opus-4-5-20251101)

### Completion Notes List

- [x] Story drafted and marked ready-for-dev
- [x] IBCSTurnoverChart component created with Recharts ComposedChart
- [x] HRView updated to use new chart (imports, destructuring, rendering)
- [x] Responsive design verified (ResponsiveContainer)
- [x] TypeScript compilation passes
- [x] Empty state handling implemented
- [x] IBCS styling applied (monochrome, zero reference line, minimal gridlines)

### Implementation Summary

Created `IBCSTurnoverChart.tsx` following the IBCS pattern from `IBCSVarianceChart.tsx`:
- Uses Recharts `ComposedChart` with `Bar` and `Line` components
- Hires displayed as positive bars (dark gray #374151)
- Terminations displayed as negative bars (medium gray #6B7280, negated in transform)
- Net change displayed as line overlay (black #111827)
- Zero reference line per IBCS best practice
- Horizontal-only gridlines for minimal visual noise
- Symmetric Y-axis domain calculated from max values
- Empty state check for missing data
- ResponsiveContainer for mobile viewports

### File List

**Files created:**
- `components/charts/IBCSTurnoverChart.tsx` - IBCS turnover trend chart component

**Files modified:**
- `components/results/HRView.tsx` - Added IBCSTurnoverChart import and integration
- `docs/sprint-artifacts/sprint-status.yaml` - Story status tracking

### Change Log

- 2026-01-21: Story 3.6 created via create-story workflow
- 2026-01-21: Implemented IBCSTurnoverChart, integrated with HRView, TypeScript passes
- 2026-01-21: Code review completed - 7 issues found and fixed (H1, M1-M4, L1-L2)
