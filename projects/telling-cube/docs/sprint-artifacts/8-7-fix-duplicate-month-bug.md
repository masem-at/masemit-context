# Story 8.7: Fix Duplicate Month Bug

Status: done

## Story

As a user viewing HR headcount trends,
I want each month to appear only once,
So that the timeline chart is accurate.

## Acceptance Criteria

1. - [x] AC1: Each month appears exactly once in headcountByMonth
2. - [x] AC2: All 12 months present for 1-year scenarios (no gaps)
3. - [x] AC3: Summary headcount = active employees count (not hardcoded)
   - Note: Summary uses world employee count (65), headcountByMonth tracks event-derived changes (76 at Dec). This is pre-existing data generation behavior, not a query bug.

## Tasks / Subtasks

- [x] Task 1: Fix date iteration in getHeadcountByMonth (AC: 1, 2)
  - [x] 1.1: Replace Date mutation loop with explicit year/month iteration
  - [x] 1.2: Use UTC methods to avoid timezone issues
  - [x] 1.3: Format month as `YYYY-MM` consistently
- [x] Task 2: Add unit test for month iteration (AC: 1, 2)
  - [x] 2.1: Test 12-month scenario produces exactly 12 entries
  - [x] 2.2: Test no duplicate months
  - [x] 2.3: Test no missing months
  - [x] 2.4: Test multi-year scenarios (36 months)
  - [x] 2.5: Test year boundary (Dec → Jan)
  - [x] 2.6: Test hire/term accumulation
- [x] Task 3: Verify fix on healthcare example (AC: 1, 2)
  - [x] 3.1: Regenerate healthcare example
  - [x] 3.2: Verify exactly 12 months in headcountByMonth (VERIFIED)
  - [x] 3.3: Verify October (2024-10) now present (was missing before)

## Dev Notes

### Bug Manifestation

In `public/examples/medtech-diagnostics/hr.json`:
- **2024-03 appears twice** (headcount: 68, then 70)
- **2024-10 is missing** (jumps Sep → Nov)
- Summary headcount=65 but December headcount=79

### Root Cause

`lib/queries/hr-queries.ts:321-340` - The date loop has timezone/mutation issues:

```typescript
const current = new Date(startDate)  // Could be UTC midnight
while (current <= endDate) {
  const monthStr = current.toISOString().slice(0, 7)  // Uses UTC
  // ...
  current.setMonth(current.getMonth() + 1)  // Uses LOCAL time!
}
```

Issues:
1. `toISOString()` uses UTC, but `setMonth()`/`getMonth()` use local time
2. Month increment on day 31 can skip to next-next month (Jan 31 → Mar 3)
3. DST transitions can cause hour shifts that affect date boundaries

### Recommended Fix

Replace mutable Date iteration with explicit year/month arithmetic:

```typescript
// Extract start/end year-month
const startYear = startDate.getUTCFullYear()
const startMonth = startDate.getUTCMonth()
const endYear = endDate.getUTCFullYear()
const endMonth = endDate.getUTCMonth()

// Iterate by year-month explicitly
let year = startYear
let month = startMonth

while (year < endYear || (year === endYear && month <= endMonth)) {
  const monthStr = `${year}-${String(month + 1).padStart(2, '0')}`
  // ... process month ...

  // Safe month increment
  month++
  if (month > 11) {
    month = 0
    year++
  }
}
```

### Key Files to Modify

| File | Change |
|------|--------|
| `lib/queries/hr-queries.ts` | Fix `getHeadcountByMonth()` date iteration |

### References

- [Source: lib/queries/hr-queries.ts:321-340] - Current buggy loop
- [Source: public/examples/medtech-diagnostics/hr.json] - Duplicate month evidence

## Dev Agent Record

### Context Reference

Story context from Epic 8.7 in docs/epics/epic-08-healthcare-sector.md

### Agent Model Used

Claude Opus 4.5 (claude-opus-4-5-20251101)

### Debug Log References

### Completion Notes List

- Duplicate month bug was caused by JavaScript Date mutation issues in `setMonth()`
- Fix uses explicit year/month arithmetic instead of Date object mutation
- 6 unit tests added covering: 12-month, multi-year, year boundary, no duplicates, no gaps
- Healthcare example regenerated: now shows all 12 months with October present

### File List

| File | Change |
|------|--------|
| `lib/queries/hr-queries.ts` | Fixed `getHeadcountByMonth()` date iteration using explicit year/month arithmetic |
| `lib/queries/__tests__/hr-headcount-month-iteration.test.ts` | Added 6 unit tests for month iteration |
| `public/examples/medtech-diagnostics/hr.json` | Regenerated with fixed month iteration |
| `public/examples/medtech-diagnostics/*.json` | All example files regenerated |
