# Story 3.11: API Docs - Add HR Endpoint

**Status:** done
**Epic:** 3 - HR Data Domain
**Points:** 1
**Created:** 2026-01-21

---

## Story

As a **developer using the tellingCube API**,
I want **HR endpoint documentation on the API docs page**,
so that **I can programmatically access workforce data for my applications**.

---

## Acceptance Criteria

1. **AC1: HR JSON Endpoint Documented**
   - [x] Endpoint listed: `GET /api/export/{scenarioId}/hr.json`
   - [x] Description explains HR data contents
   - [x] Example cURL request shown
   - [x] Response structure documented with all fields

2. **AC2: HR CSV Endpoint Documented**
   - [x] Endpoint listed: `GET /api/export/{scenarioId}/hr.csv`
   - [x] Download badge shown (matches events.csv style)
   - [x] CSV columns listed

3. **AC3: Response Structure Accurate**
   - [x] Documents `summary` object (headcount, avgTenure, turnoverRate, avgSalary, totalPayroll, genderDistribution)
   - [x] Documents `headcountByDepartment` array
   - [x] Documents `headcountByMonth` array
   - [x] Documents `salaryByDepartment` array
   - [x] Documents `turnoverByMonth` array

4. **AC4: Consistent Style**
   - [x] Matches existing Finance/Sales endpoint documentation style
   - [x] Uses same CodeBlock component
   - [x] Same visual hierarchy and badges

---

## Tasks / Subtasks

### Task 1: Add HR JSON Endpoint Section (AC: #1, #3, #4)
- [x] 1.1 Add HR endpoint card after Sales endpoint
- [x] 1.2 Write description mentioning workforce analytics
- [x] 1.3 Add example cURL request
- [x] 1.4 Document complete response structure

### Task 2: Add HR CSV Endpoint Section (AC: #2, #4)
- [x] 2.1 Add HR CSV endpoint card
- [x] 2.2 Add DOWNLOAD badge (like events.csv)
- [x] 2.3 List CSV columns

### Task 3: Update Usage Examples (AC: #4)
- [x] 3.1 Add HR example to Python code block
- [x] 3.2 Add HR example to JavaScript code block

---

## Dev Notes

### HR API Response Structure

From `types/api.ts` (HRViewResponse):
```typescript
interface HRViewResponse {
  summary: {
    headcount: number
    avgTenure: number        // months
    turnoverRate: number     // percentage
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
  headcountByMonth: Array<{
    month: string
    headcount: number
    hires: number
    terminations: number
    netChange: number
  }>
  salaryByDepartment: Array<{
    department: string
    avgSalary: number
    minSalary: number
    maxSalary: number
    headcount: number
    totalPayroll: number
  }>
  turnoverByMonth: Array<{
    month: string
    turnoverRate: number
    hires: number
    terminations: number
  }>
}
```

### Existing Endpoint Pattern

Follow the pattern from Finance/Sales endpoints in `/app/api-docs/page.tsx`:
- Gradient header with GET badge
- Description paragraph
- Parameters table (if applicable)
- Example Request (CodeBlock)
- Response Structure (CodeBlock with JSON)

### Files to Reference

- `app/api-docs/page.tsx` - API documentation page
- `app/api/export/[scenarioId]/hr.json/route.ts` - HR JSON endpoint
- `app/api/export/[scenarioId]/hr.csv/route.ts` - HR CSV endpoint
- `types/api.ts` - HRViewResponse interface

---

## Dev Agent Record

### Agent Model Used

Claude Opus 4.5 (claude-opus-4-5-20251101)

### Completion Notes List

- [x] Story created by Bob (SM)
- [x] HR JSON endpoint documented with WORKFORCE badge
- [x] HR CSV endpoint documented with DOWNLOAD badge
- [x] Response structure includes all fields: summary, headcountByDepartment, headcountByMonth, salaryByDepartment, turnoverByMonth, employees
- [x] Documented ?anonymize=true parameter for both endpoints
- [x] Python usage example updated with HR data fetch
- [x] JavaScript usage example updated with HR data fetch
- [x] TypeScript passes

### File List

**Files modified:**
- `app/api-docs/page.tsx` - Added HR JSON and CSV endpoint documentation sections
- `docs/sprint-artifacts/sprint-status.yaml` - Story status tracking
- `docs/sprint-artifacts/3-11-api-docs-hr-endpoint.md` - This story file

### Change Log

- 2026-01-21: Story 3.11 created via Party Mode
- 2026-01-22: All tasks implemented - HR endpoints documented, usage examples updated
