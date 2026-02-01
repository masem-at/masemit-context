# Story 3.4: HR View Page

**Status:** done
**Epic:** 3 - HR Data Domain
**Points:** 3
**Created:** 2026-01-21

---

## Story

As a **user viewing scenario results**,
I want **a dedicated HR view tab alongside Finance and Sales**,
so that **I can explore workforce analytics data for the generated scenario**.

---

## Acceptance Criteria

1. **AC1: Tab Navigation Updated**
   - [x] HR tab added to ViewNavigation (Finance | Sales | HR)
   - [x] Tab shows Users icon
   - [x] Works on both desktop (tabs) and mobile (accordion)

2. **AC2: HR View Component Created**
   - [x] `/components/results/HRView.tsx` component
   - [x] Fetches data from `/api/views/hr` endpoint
   - [x] Shows loading and error states

3. **AC3: HR Summary Cards**
   - [x] Headcount card
   - [x] Average Tenure card
   - [x] Turnover Rate card
   - [x] Average Salary card
   - [x] Responsive grid layout

4. **AC4: Synthetic Data Notice**
   - [x] "Synthetic Data" badge visible
   - [x] Footer disclaimer about data being artificial

---

## Tasks / Subtasks

### Task 1: Update View Navigation (AC: #1)
- [x] 1.1 Add 'hr' to ViewType in ViewNavigation.tsx
- [x] 1.2 Add HR tab to ViewTabs.tsx
- [x] 1.3 Add HR accordion to ViewAccordion.tsx

### Task 2: Create HR API Endpoint (AC: #2)
- [x] 2.1 Create `/api/views/hr/route.ts`
- [x] 2.2 Return HR summary data

### Task 3: Create HR View Component (AC: #2, #3, #4)
- [x] 3.1 Create `components/results/HRView.tsx`
- [x] 3.2 Add summary cards with KPIs
- [x] 3.3 Add synthetic data badge

### Task 4: Integrate with Results Page (AC: #1)
- [x] 4.1 Import HRView in results page
- [x] 4.2 Add HR exports to dropdown menu

---

## Dev Notes

### View Navigation Pattern

The existing pattern uses:
- `ViewNavigation` - wrapper managing state
- `ViewTabs` - desktop tabs
- `ViewAccordion` - mobile accordion
- View components (`SalesView`, `FinanceView`) - actual content

### API Response Structure

```typescript
interface HRViewResponse {
  summary: {
    headcount: number
    avgTenure: number
    turnoverRate: number
    avgSalary: number
    totalPayroll: number
    genderDistribution: {
      male: number
      female: number
      diverse: number
    }
  }
  headcountByDepartment: Array<{
    department: string
    count: number
    percentage: number
  }>
}
```

---

## Dev Agent Record

### Agent Model Used

Claude Opus 4.5 (claude-opus-4-5-20251101)

### Completion Notes List

- [x] Story drafted
- [x] View navigation updated
- [x] HR API endpoint created
- [x] HRView component created
- [x] Results page updated

### File List

**Files modified:**
- `components/results/ViewNavigation.tsx` - Added 'hr' to ViewType
- `components/results/ViewTabs.tsx` - Added HR tab button
- `components/results/ViewAccordion.tsx` - Added HR accordion option
- `app/results/[scenarioId]/page.tsx` - Imported HRView, added render case, HR exports
- `types/api.ts` - Added HRViewResponse and related types

**Files created:**
- `app/api/views/hr/route.ts` - HR view API endpoint
- `components/results/HRView.tsx` - HR View component with KPIs

### Change Log

- 2026-01-21: Story 3.4 drafted
- 2026-01-21: Implemented all tasks, story complete
