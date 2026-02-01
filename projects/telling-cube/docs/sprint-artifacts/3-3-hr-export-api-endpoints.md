# Story 3.3: HR Export API Endpoints

**Status:** done
**Epic:** 3 - HR Data Domain
**Points:** 3
**Created:** 2026-01-21

---

## Story

As a **BI developer**,
I want **API endpoints to export HR data in JSON and CSV formats**,
so that **I can import workforce data into my analytics tools**.

---

## Acceptance Criteria

1. **AC1: JSON Export Endpoint**
   - [x] `GET /api/export/{scenarioId}/hr.json` returns complete HR data
   - [x] Includes summary, headcountByDepartment, headcountByMonth
   - [x] Proper CORS headers for cross-origin requests

2. **AC2: CSV Export Endpoint**
   - [x] `GET /api/export/{scenarioId}/hr.csv` returns tabular export
   - [x] Columns: employee_id, department, role, level, salary, hire_date
   - [x] Downloadable with proper Content-Disposition header

3. **AC3: Export Data Structure**
   - [x] summary: headcount, avgTenure, turnoverRate, avgSalary, totalPayroll
   - [x] headcountByDepartment: array of {department, count}
   - [x] headcountByMonth: array of {month, headcount, hires, terminations}
   - [x] employees: anonymized list with synthetic names

---

## Tasks / Subtasks

### Task 1: Create HR Queries (AC: #3)
- [x] 1.1 Create `lib/queries/hr-queries.ts`
- [x] 1.2 Implement `getHRSummary()` function
- [x] 1.3 Implement `getHeadcountByDepartment()` function
- [x] 1.4 Implement `getHeadcountByMonth()` function

### Task 2: Create JSON Export Endpoint (AC: #1)
- [x] 2.1 Create `app/api/export/[scenarioId]/hr.json/route.ts`
- [x] 2.2 Return complete HR data structure
- [x] 2.3 Add CORS headers

### Task 3: Create CSV Export Endpoint (AC: #2)
- [x] 3.1 Create `app/api/export/[scenarioId]/hr.csv/route.ts`
- [x] 3.2 Format as downloadable CSV
- [x] 3.3 Add Content-Disposition header

---

## Dev Notes

### HR Data Structure

```typescript
interface HRExportResponse {
  scenarioId: string
  summary: {
    headcount: number
    avgTenure: number  // months
    turnoverRate: number  // percentage
    avgSalary: number
    totalPayroll: number
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
  }>
  employees: Array<{
    id: string
    name: string  // synthetic
    department: string
    role: string
    level: string
    salary: number
    hireDate: string
    gender: string
  }>
}
```

### Query Patterns

Use the Event table to aggregate HR metrics:
- Count EMPLOYEE_HIRED events
- Count EMPLOYEE_TERMINATED events
- Use ScenarioWorld for employee details

---

## Dev Agent Record

### Agent Model Used

Claude Opus 4.5 (claude-opus-4-5-20251101)

### Completion Notes List

- [x] Story drafted
- [x] HR queries created with 6 query functions
- [x] JSON endpoint created with full HR data export
- [x] CSV endpoint created with downloadable employee roster

### File List

**Files created:**
- `lib/queries/hr-queries.ts` - HR query functions (getHRSummary, getHeadcountByDepartment, etc.)
- `app/api/export/[scenarioId]/hr.json/route.ts` - JSON export endpoint
- `app/api/export/[scenarioId]/hr.csv/route.ts` - CSV export endpoint

### Change Log

- 2026-01-21: Story 3.3 drafted
- 2026-01-21: Story 3.3 implemented - HR Export API Endpoints
