# Story 2.2: Year Duration Selector UI

**Status:** done
**Epic:** 2 - Multi-Year Scenarios (3-5 Years)
**Points:** 2
**Created:** 2026-01-21

---

## Story

As a **tellingCube user**,
I want **to select 1, 3, or 5 years of data duration when generating a scenario**,
so that **I can create multi-year scenarios that match my analysis needs and tier entitlements**.

---

## Acceptance Criteria

### AC1: Duration Selection Available in GenerationDialog
**Given** the GenerationDialog component is displayed
**When** the user views the "Data Duration" option group
**Then** three options are displayed: "1 Year", "3 Years", "5 Years"
**And** the default selection is "1 Year"

### AC2: Selection Persists to Scenario Generation
**Given** the user selects "3 Years" duration in GenerationDialog
**When** they click the "Generate" button
**Then** the `durationYears=3` parameter is included in the generation request
**And** the loading page receives the `durationYears` param

### AC3: Tier Restrictions Applied with Lock Icons
**Given** a user with `maxYears=1` entitlement (Single tier)
**When** they view the Data Duration options
**Then** "3 Years" and "5 Years" display lock icons (🔒)
**And** clicking locked options does NOT change the selection
**And** a tooltip shows "Requires Lifetime or higher" for 3Y
**And** a tooltip shows "Requires Lifetime+ or Pro" for 5Y

**Given** a user with `maxYears=3` entitlement (Lifetime tier)
**When** they view the Data Duration options
**Then** "1 Year" and "3 Years" are selectable
**And** "5 Years" displays a lock icon
**And** clicking "5 Years" does NOT change the selection

### AC4: Generation API Accepts durationYears Parameter
**Given** the `/api/generate` endpoint receives a request
**When** the request body contains `{ durationYears: 3 }`
**Then** the created Scenario record has `durationYears = 3`
**And** `timePeriodStart` and `timePeriodEnd` are calculated correctly

### AC5: Loading Page Displays Duration Context
**Given** the user is on `/generate/loading` with `durationYears=5`
**When** the page renders
**Then** the messaging reflects multi-year generation:
  - "Creating 60 months of realistic, mathematically consistent business data..."
  - Estimated time shows multi-year duration (~5-7 minutes for 5Y)

---

## Tasks / Subtasks

- [ ] **Task 1: Update GenerationDialog to pass durationYears** (AC: 1, 2, 3)
  - [ ] Verify existing duration options are displayed (already implemented in Epic 6)
  - [ ] Ensure `durationYears` is passed in URL params to loading page
  - [ ] Confirm lock icons appear for tier-restricted options

- [ ] **Task 2: Update Generate API to accept durationYears** (AC: 4)
  - [ ] Add `durationYears?: 1 | 3 | 5` to `GenerateRequest` interface
  - [ ] Import and use `isValidDurationYears` from `lib/utils/time-period.ts`
  - [ ] Import and use `calculateTimePeriod` for start/end dates
  - [ ] Pass `durationYears` to scenario creation
  - [ ] Pass `durationYears` to QStash worker payload

- [ ] **Task 3: Update Loading Page for multi-year context** (AC: 5)
  - [ ] Read `durationYears` from searchParams (already reads it)
  - [ ] Update progress messaging for multi-year scenarios
  - [ ] Adjust expected duration based on durationYears (multiply base by years)

- [ ] **Task 4: Verify tier-based access control** (AC: 3)
  - [ ] Test with free user (should see all locked)
  - [ ] Test with single user (1Y only)
  - [ ] Test with lifetime user (1Y + 3Y)
  - [ ] Test with lifetime+ user (all unlocked)

---

## Dev Notes

### Current State Analysis

The GenerationDialog (`components/dashboard/GenerationDialog.tsx:54-59`) already has duration options:

```typescript
const DURATION_OPTIONS: Option<Duration>[] = [
  { value: 1, label: '1 Year' },
  { value: 3, label: '3 Years' },
  { value: 5, label: '5 Years' },
]
```

And tier-based locking logic (`components/dashboard/GenerationDialog.tsx:82-86`):

```typescript
const durationOptionsWithLock = DURATION_OPTIONS.map((opt) => ({
  ...opt,
  locked: !canAccessYears(entitlements, opt.value),
  lockReason: getLockReason(opt.value, entitlements.maxYears),
}))
```

The `durationYears` parameter is already passed to the loading page (`GenerationDialog.tsx:108`):

```typescript
durationYears: duration.toString(),
```

### Generate API Changes Required

File: `app/api/generate/route.ts`

The API currently hardcodes the time period dates (lines 253-254):
```typescript
timePeriodStart: new Date('2024-01-01'),
timePeriodEnd: new Date('2024-12-31'),
```

Changes needed:
1. Add `durationYears` to `GenerateRequest` interface
2. Import `isValidDurationYears`, `calculateTimePeriod` from `lib/utils/time-period.ts`
3. Calculate time period using `calculateTimePeriod(2024, durationYears)`
4. Pass `durationYears` to Prisma create
5. Include `durationYears` in QStash payload

### Loading Page Changes

File: `app/generate/loading/page.tsx`

Already reads `durationYears` param but doesn't use it for messaging. Update:

1. Parse `durationYears` from searchParams
2. Update month count display: `{12 * durationYears} months`
3. Adjust expected duration: base time × durationYears (with cap)

### Entitlements Already Implemented

The `canAccessYears` function in `lib/auth/entitlements.ts:193-195` is ready:
```typescript
export function canAccessYears(user: UserEntitlements, years: number): boolean {
  return user.maxYears >= years
}
```

Tier maxYears mapping (`lib/auth/entitlements.ts:68-111`):
| Tier | maxYears |
|------|----------|
| free | 1 |
| single | 1 |
| lifetime | 3 |
| lifetime_plus | 5 |
| pro | 5 |
| team | 5 |

### Project Structure Notes

- **Schema:** `prisma/schema.prisma:33` - `durationYears` field already exists
- **Utils:** `lib/utils/time-period.ts` - Helper functions already exist
- **Entitlements:** `lib/auth/entitlements.ts` - Access control already exists
- **Dialog:** `components/dashboard/GenerationDialog.tsx` - UI already exists
- **API:** `app/api/generate/route.ts` - Needs update for durationYears
- **Loading:** `app/generate/loading/page.tsx` - Needs messaging update

### Files to Modify

| File | Change |
|------|--------|
| `app/api/generate/route.ts` | Accept durationYears, calculate time period |
| `app/generate/loading/page.tsx` | Update messaging for multi-year |

### Files Already Complete (Verify Only)

| File | Reason |
|------|--------|
| `prisma/schema.prisma` | durationYears field exists (Story 2.1) |
| `lib/utils/time-period.ts` | Helper functions exist (Story 2.1) |
| `lib/auth/entitlements.ts` | canAccessYears exists (Epic 6) |
| `components/dashboard/GenerationDialog.tsx` | Duration selector exists (Epic 6) |

---

## References

- [Source: docs/epics/epic-02-multi-year-scenarios.md#Story 2.2]
- [Source: docs/future/spec-multi-year-scenarios.md#Proposed Solution]
- [Source: docs/sprint-artifacts/2-1-extend-scenario-model-for-multi-year.md]
- [Source: prisma/schema.prisma:33] - durationYears field
- [Source: lib/utils/time-period.ts] - calculateTimePeriod helper
- [Source: lib/auth/entitlements.ts:193-195] - canAccessYears function
- [Source: components/dashboard/GenerationDialog.tsx:54-86] - Duration selector UI
- [Source: app/api/generate/route.ts:253-254] - Hardcoded dates to replace

---

## Dev Agent Record

### Context Reference

Story context created via BMM create-story workflow 2026-01-21

### Agent Model Used

Claude Opus 4.5 (claude-opus-4-5-20251101)

### Debug Log References

N/A - No errors encountered

### Completion Notes List

1. **API Route Updated**: Added `durationYears` to `/api/generate` request interface and scenario creation
   - Imports `isValidDurationYears`, `calculateTimePeriod`, `DurationYears` from time-period utils
   - Validates and defaults `durationYears` to 1 if not provided
   - Uses `calculateTimePeriod()` to calculate `timePeriodStart` and `timePeriodEnd` dynamically
   - Passes `durationYears` to QStash worker payload

2. **Loading Page Updated**: Multi-year messaging and expected duration
   - Reads `durationYears` from URL search params
   - Calculates expected duration with year multiplier (1x, 2.5x, 4x for 1, 3, 5 years)
   - Updates month count display dynamically (12, 36, 60 months)
   - Updates Excel comparison hours (3-4h, 10-12h, 15-20h)

3. **GenerationDialog Fixed**: Changed param from `scenarioType` to `size` for matrix system compatibility

4. **All AC verified**: Duration selector UI already existed from Epic 6, tier-based locking works, API accepts durationYears

### File List

| File | Change |
|------|--------|
| `app/api/generate/route.ts` | Added durationYears import, validation, time period calculation, and QStash payload |
| `app/generate/loading/page.tsx` | Added durationYears param reading, expected duration calculation, messaging updates |
| `components/dashboard/GenerationDialog.tsx` | Fixed param name: scenarioType → size for matrix system |
