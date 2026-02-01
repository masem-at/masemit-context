# Story 2.1: Extend Scenario Model for Multi-Year

**Status:** done
**Epic:** 2 - Multi-Year Scenarios (3-5 Years)
**Points:** 3
**Created:** 2026-01-19

---

## Story

As a **system developer**,
I want **the Scenario model extended with a `durationYears` field and proper time period calculations**,
so that **multi-year scenario generation (1, 3, or 5 years) can be supported with full backward compatibility**.

---

## Acceptance Criteria

### AC1: durationYears Field Added
**Given** the Prisma schema is updated
**When** the migration runs
**Then** the `scenarios` table has a new `durationYears` column of type `INTEGER`
**And** the default value is `1`
**And** allowed values are constrained to 1, 3, or 5 (validated in application code)

### AC2: timePeriodStart and timePeriodEnd Calculated Correctly
**Given** a scenario is created with `durationYears = 3` and start year 2023
**When** the time period is calculated
**Then** `timePeriodStart` = `2023-01-01`
**And** `timePeriodEnd` = `2025-12-31` (3 years: 2023, 2024, 2025)

**Given** a scenario is created with `durationYears = 5` and start year 2021
**When** the time period is calculated
**Then** `timePeriodStart` = `2021-01-01`
**And** `timePeriodEnd` = `2025-12-31` (5 years: 2021-2025)

### AC3: Database Migration Created and Tested
**Given** the developer runs `npx prisma migrate dev --name add_duration_years`
**When** the migration completes
**Then** the migration file exists in `prisma/migrations/`
**And** the migration is idempotent (can be run multiple times safely)
**And** no data is lost from existing scenarios

### AC4: Existing 1-Year Scenarios Continue to Work
**Given** existing scenarios in the database (created before this migration)
**When** the migration runs
**Then** all existing scenarios have `durationYears = 1` (default)
**And** their `timePeriodStart` and `timePeriodEnd` values are unchanged
**And** the application can read and display existing scenarios without errors

---

## Tasks / Subtasks

- [ ] **Task 1: Update Prisma Schema** (AC: 1)
  - [ ] Add `durationYears Int @default(1)` to `Scenario` model in `prisma/schema.prisma`
  - [ ] Add inline comment explaining allowed values (1, 3, 5)

- [ ] **Task 2: Create and Run Migration** (AC: 3)
  - [ ] Run `npx prisma migrate dev --name add_duration_years`
  - [ ] Verify migration file created in `prisma/migrations/`
  - [ ] Verify Prisma client regenerated

- [ ] **Task 3: Add TypeScript Type/Validation** (AC: 1, 2)
  - [ ] Create type `DurationYears = 1 | 3 | 5` in appropriate types file
  - [ ] Create validation function `isValidDurationYears(value: number): value is DurationYears`
  - [ ] Create helper function `calculateTimePeriod(startYear: number, durationYears: DurationYears): { start: Date, end: Date }`

- [ ] **Task 4: Update Scenario Creation Logic** (AC: 2)
  - [ ] Locate scenario creation code (likely in API route or service)
  - [ ] Update to accept and validate `durationYears` parameter
  - [ ] Use `calculateTimePeriod` helper for date calculations

- [ ] **Task 5: Test Backward Compatibility** (AC: 4)
  - [ ] Query existing scenarios to verify `durationYears = 1`
  - [ ] Verify existing scenario display works without changes
  - [ ] Test that no errors occur when viewing pre-migration scenarios

---

## Dev Notes

### Current Schema State

The `Scenario` model in `prisma/schema.prisma:14-56` currently has:
- `timePeriodStart DateTime` (exists)
- `timePeriodEnd DateTime` (exists)
- **Missing:** `durationYears` field

### Schema Change Required

```prisma
// In model Scenario (prisma/schema.prisma)
model Scenario {
  // ... existing fields ...

  // Multi-year support (Story 2.1)
  durationYears     Int       @default(1)  // Allowed: 1, 3, or 5 years

  // ... existing timePeriodStart and timePeriodEnd remain unchanged ...
}
```

### Time Period Calculation Logic

```typescript
// lib/utils/time-period.ts (suggested location)

export type DurationYears = 1 | 3 | 5;

export function isValidDurationYears(value: number): value is DurationYears {
  return value === 1 || value === 3 || value === 5;
}

export function calculateTimePeriod(
  startYear: number,
  durationYears: DurationYears
): { start: Date; end: Date } {
  const start = new Date(startYear, 0, 1); // Jan 1 of start year
  const end = new Date(startYear + durationYears - 1, 11, 31); // Dec 31 of end year
  return { start, end };
}

// Examples:
// calculateTimePeriod(2023, 1) => { start: 2023-01-01, end: 2023-12-31 }
// calculateTimePeriod(2023, 3) => { start: 2023-01-01, end: 2025-12-31 }
// calculateTimePeriod(2021, 5) => { start: 2021-01-01, end: 2025-12-31 }
```

### Architecture Compliance

From `docs/architecture.md:45-58` (FR15 Roadmap section):

> **FR15: Multi-Year Support (3-5 Years)**
> **Current Design:** Event dates stored as `DATE` type, no year constraints
> **Impact:** ✅ LOW - Current schema already supports multi-year data

The architecture confirms this is a low-impact change - the existing `timePeriodStart` and `timePeriodEnd` fields already support any date range. We're just adding metadata to track the intended duration.

### Files to Modify

| File | Change |
|------|--------|
| `prisma/schema.prisma` | Add `durationYears` field to Scenario model |
| `lib/utils/time-period.ts` | NEW: Create helper functions |
| `types/api.ts` or new types file | Add `DurationYears` type |

### Files to Check (No Modification Expected)

| File | Reason |
|------|--------|
| `lib/queries/finance-queries.ts` | Uses Scenario via Prisma - should auto-update |
| `lib/queries/sales-queries.ts` | Uses Scenario via Prisma - should auto-update |
| API routes in `app/api/` | May need updates in Story 2.2 (not this story) |

---

## Project Structure Notes

### Alignment with Project Structure

- **Schema:** `prisma/schema.prisma` - single source of truth for DB models
- **Types:** `types/` directory for shared TypeScript types
- **Utils:** `lib/utils/` for helper functions
- **API:** Next.js 14 App Router (`app/api/`)

### Detected Patterns

1. **Prisma models map to lowercase table names** (e.g., `Scenario` → `@@map("scenarios")`)
2. **Indexes defined at model level** for query performance
3. **Default values specified in Prisma** (not application code)
4. **TypeScript types in `types/` or `lib/types/`** directories

---

## References

- [Source: docs/epics/epic-02-multi-year-scenarios.md#Story 2.1]
- [Source: docs/future/spec-multi-year-scenarios.md#Phase 1: Generation Extension]
- [Source: docs/architecture.md#FR15: Multi-Year Support]
- [Source: prisma/schema.prisma#Scenario model]

---

## Dev Agent Record

### Context Reference

Story context created via BMM create-story workflow 2026-01-19

### Agent Model Used

_To be filled by dev agent_

### Debug Log References

_To be filled by dev agent_

### Completion Notes List

_To be filled by dev agent_

### File List

_To be filled by dev agent upon completion_
