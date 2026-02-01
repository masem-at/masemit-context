# Story 2.9: Tier-Based Access Control

**Status:** done
**Epic:** 2 - Multi-Year Scenarios (3-5 Years)
**Points:** 2
**Created:** 2026-01-21

---

## Story

As a **product owner protecting premium features**,
I want **multi-year options restricted based on user tier**,
so that **users must upgrade to access 3-year and 5-year scenarios**.

---

## Acceptance Criteria

### AC1: Free Users Cannot Generate Multi-Year
**Given** a free user (curated examples only)
**When** they access the generation dialog
**Then** they see upgrade messaging (no generation ability)

### AC2: Single Tier Gets 1 Year Only
**Given** a Single tier user (€9 purchase)
**When** they access the duration selector
**Then** only "1 Year" is available
**And** 3 and 5 year options show lock icons

### AC3: Lifetime Tier Gets 1-3 Years
**Given** a Lifetime tier user (€99 purchase)
**When** they access the duration selector
**Then** "1 Year" and "3 Years" are available
**And** "5 Years" shows lock icon with upgrade CTA

### AC4: Lifetime+ and Pro Get Full Access
**Given** a Lifetime+, Pro, or Team tier user
**When** they access the duration selector
**Then** all duration options (1, 3, 5 years) are available

### AC5: Backend Enforcement
**Given** a user attempts to bypass frontend restrictions
**When** they send an API request with durationYears beyond their tier
**Then** the API rejects with 403 Forbidden
**And** an error message explains the tier restriction

---

## Tasks / Subtasks

- [x] **Task 1: Frontend Enforcement (Pre-existing)** (AC: 1-4)
  - [x] GenerationDialog already uses `canAccessYears()` helper
  - [x] Lock icons shown on restricted options
  - [x] Upgrade CTA links to pricing page

- [x] **Task 2: Backend Enforcement** (AC: 5)
  - [x] Add tier check in `/api/generate` route
  - [x] For authenticated users: validate durationYears <= maxYears
  - [x] For anonymous users: only allow durationYears = 1
  - [x] Return 403 with clear error message

---

## Dev Notes

### Current Implementation

**Frontend (GenerationDialog):**
- Already restricts options based on `entitlements.maxYears`
- Uses `canAccessYears()` from `lib/auth/entitlements.ts`
- Lock icons and upgrade CTAs are present

**Backend Gap:**
- `/api/generate` currently accepts any `durationYears` value
- No validation against user's tier/maxYears
- Anonymous users could technically request 5 years via URL manipulation

### Backend Fix Required

Add to `app/api/generate/route.ts`:
```typescript
// For authenticated users, check tier restrictions
if (authenticatedUser && durationYears > 1) {
  const user = await prisma.user.findUnique({
    where: { id: authenticatedUser.id },
    select: { maxYears: true }
  })
  if (user && durationYears > user.maxYears) {
    throw new ApiError(
      `Your tier allows up to ${user.maxYears}-year scenarios. Upgrade for ${durationYears}-year access.`,
      ErrorCodes.FORBIDDEN,
      403
    )
  }
}

// Anonymous users can only generate 1-year scenarios
if (!authenticatedUser && durationYears > 1) {
  throw new ApiError(
    'Multi-year scenarios require a paid account. Sign in or purchase a plan.',
    ErrorCodes.FORBIDDEN,
    403
  )
}
```

### Files to Modify

| File | Change |
|------|--------|
| `app/api/generate/route.ts` | Add tier-based durationYears validation |

---

## References

- [Source: docs/epics/epic-02-multi-year-scenarios.md#Story 2.9]
- [Source: lib/auth/entitlements.ts] - Tier definitions and helpers
- [Source: components/dashboard/GenerationDialog.tsx] - Frontend enforcement

---

## Dev Agent Record

### Context Reference

Story context created via BMM workflow 2026-01-21

### Agent Model Used

Claude Opus 4.5 (claude-opus-4-5-20251101)

### Completion Notes List

1. **Frontend Enforcement (Pre-existing)**: GenerationDialog already enforces tier restrictions via `canAccessYears()` helper. Lock icons and upgrade CTAs were implemented in Story 6.8.

2. **Backend Enforcement (New)**:
   - Added `FORBIDDEN` error code to `lib/api/error-handler.ts`
   - Added tier check in `/api/generate` route after durationYears validation
   - For authenticated users: fetches user's `maxYears` from database and validates
   - For anonymous users: rejects any request with durationYears > 1
   - Returns 403 with clear upgrade messaging

3. **Tier Matrix**:
   - Free: 0 generations (can't generate anyway)
   - Single (€9): 1 year only
   - Lifetime (€99): 1-3 years
   - Lifetime+ (€299): 1-5 years
   - Pro (€19/mo): 1-5 years
   - Team (€49/mo): 1-5 years

### File List

| File | Change |
|------|--------|
| `lib/api/error-handler.ts` | Added FORBIDDEN error code |
| `app/api/generate/route.ts` | Added tier-based durationYears validation |
| `docs/sprint-artifacts/2-9-tier-based-access-control.md` | Created and marked done |
