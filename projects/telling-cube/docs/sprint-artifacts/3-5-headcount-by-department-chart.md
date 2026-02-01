# Story 3.5: Headcount by Department Chart

**Status:** done
**Epic:** 3 - HR Data Domain
**Points:** 2
**Created:** 2026-01-21

---

## Story

As a **user viewing the HR tab**,
I want **a professional IBCS-style horizontal bar chart showing headcount by department**,
so that **I can quickly understand workforce distribution with variance indicators**.

---

## Acceptance Criteria

1. **AC1: Chart Component Created**
   - [x] New `IBCSHeadcountChart.tsx` component in `/components/charts/`
   - [x] Follows IBCS horizontal bar chart pattern
   - [x] Dark gray bars for actual headcount
   - [x] Labels showing department name, count, and percentage

2. **AC2: Chart Replaces Inline Bars**
   - [x] HRView imports and uses IBCSHeadcountChart
   - [x] Remove existing inline bar visualization from HRView
   - [x] Chart accepts `HeadcountByDepartment[]` as prop

3. **AC3: Visual Design Requirements**
   - [x] Sorted by headcount descending (largest department first)
   - [x] Department name left-aligned (truncated if >20 chars)
   - [x] Bar width proportional to percentage of total
   - [x] Value label (count) displayed inside or after bar
   - [x] Percentage shown in separate column
   - [x] Total row at bottom

4. **AC4: Responsive Design**
   - [x] Works on mobile (stacked labels if needed)
   - [x] Minimum bar height for touch targets
   - [x] Proper spacing on all screen sizes

---

## Tasks / Subtasks

### Task 1: Create IBCSHeadcountChart Component (AC: #1, #3)
- [x] 1.1 Create `components/charts/IBCSHeadcountChart.tsx`
- [x] 1.2 Define props interface with HeadcountByDepartment data
- [x] 1.3 Implement horizontal bar rendering with IBCS styling
- [x] 1.4 Add total row calculation and display
- [x] 1.5 Ensure bars sorted by count descending

### Task 2: Integrate with HRView (AC: #2)
- [x] 2.1 Import IBCSHeadcountChart in HRView.tsx
- [x] 2.2 Replace inline bar code (lines 162-203) with chart component
- [x] 2.3 Pass headcountByDepartment data to chart

### Task 3: Responsive Styling (AC: #4)
- [x] 3.1 Add responsive breakpoints for mobile
- [x] 3.2 Ensure touch-friendly bar heights (min 32px)
- [x] 3.3 Test on narrow viewports

---

## Dev Notes

### Existing Pattern Reference

The HRView already has a basic visualization at lines 162-203. The new chart should:
- Be extracted to a dedicated component for reusability
- Follow IBCS conventions like `IBCSRegionalChart` and `IBCSProductChart`
- Use the same styling patterns (gray-700 bars, font-mono numbers)

### Component Pattern to Follow

Reference: `components/charts/IBCSRegionalChart.tsx` for structure:
```typescript
interface IBCSHeadcountChartProps {
  data: HeadcountByDepartment[]
  title?: string
  period?: string
}
```

### Data Structure Available

From `types/api.ts` (line 298-302):
```typescript
export interface HeadcountByDepartment {
  department: string
  count: number
  percentage: number
}
```

### IBCS Styling Requirements

1. Bars: `bg-gray-700` (dark solid for actuals)
2. Numbers: `font-mono text-gray-900`
3. Labels: `text-sm text-gray-900`
4. Percentages: `text-gray-500` (secondary)
5. No colors except data highlighting

### Files to Reference

- `components/charts/IBCSRegionalChart.tsx` - Regional bar chart pattern
- `components/charts/IBCSProductChart.tsx` - Product performance pattern
- `components/results/HRView.tsx` - Current implementation to replace
- `lib/utils/format-units.ts` - Scaling utilities

### Previous Story Learnings (3.4)

From story 3.4 implementation:
- ViewType union includes 'hr'
- HRView fetches from `/api/views/hr`
- Data includes `headcountByDepartment` array
- Existing inline bars work but aren't componentized

---

## Senior Developer Review (AI)

**Review Date:** 2026-01-21
**Reviewer:** Claude Opus 4.5 (Adversarial Code Review)
**Outcome:** Approve with fixes applied

### Issues Found & Resolved

| Severity | Issue | Resolution |
|----------|-------|------------|
| MEDIUM | M1: `style jsx` not supported in Next.js App Router | Replaced with Tailwind responsive classes |
| MEDIUM | M2: Dev Notes referenced unused `unit` prop | Removed from documentation |
| MEDIUM | M3: Missing empty state for zero departments | Added empty state check |
| LOW | L1: File List missing sprint-status.yaml | Added to File List |
| LOW | L2: Unnecessary React import | Removed |

### Action Items
All items resolved automatically - no outstanding action items.

---

## Dev Agent Record

### Agent Model Used

Claude Opus 4.5 (claude-opus-4-5-20251101)

### Completion Notes List

- [x] Story drafted and marked ready-for-dev
- [x] IBCSHeadcountChart component created
- [x] HRView updated to use new chart
- [x] Responsive design verified
- [x] Code review fixes applied (5 issues resolved)

### Implementation Summary

Created `IBCSHeadcountChart.tsx` following the IBCS pattern from `IBCSRegionalChart.tsx`:
- Table-based layout with header row
- Horizontal bars using `bg-gray-700` (solid dark gray per IBCS actuals)
- Data sorted by count descending
- Department names truncated at 20 chars with full name in title attribute
- Total row calculated dynamically
- Responsive Tailwind classes (text-xs/sm:text-sm, px-2/sm:px-4)
- Touch-friendly bar heights (h-6 = 24px visible, h-8 = 32px container)
- Empty state handling for zero departments

### File List

**Files created:**
- `components/charts/IBCSHeadcountChart.tsx` - IBCS headcount bar chart component

**Files modified:**
- `components/results/HRView.tsx` - Replaced inline bars with IBCSHeadcountChart
- `docs/sprint-artifacts/sprint-status.yaml` - Story status tracking

### Change Log

- 2026-01-21: Story 3.5 created via create-story workflow
- 2026-01-21: Implemented IBCSHeadcountChart, integrated with HRView, TypeScript passes
- 2026-01-21: Code review completed - 5 issues found and fixed (M1, M2, M3, L1, L2)
