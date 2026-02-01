# Story 2.8: Progressive Generation UI

**Status:** done
**Epic:** 2 - Multi-Year Scenarios (3-5 Years)
**Points:** 3
**Created:** 2026-01-21

---

## Story

As a **user generating a multi-year scenario**,
I want **to see year-by-year progress during generation**,
so that **I understand how long it will take and can see progress even for 5-7 minute generations**.

---

## Acceptance Criteria

### AC1: Progress Bar Shows Year-by-Year Progress
**Given** a 3 or 5 year scenario is generating
**When** the loading page is displayed
**Then** the progress bar shows segmented year markers
**And** the current year being generated is highlighted

### AC2: "Generating Year X of Y..." Status Message
**Given** a multi-year generation is in progress
**When** viewing the loading page
**Then** a status message shows "Generating Year 2 of 5..."
**And** the year number updates as progress increases

### AC3: Estimated Time Remaining
**Given** a multi-year scenario is generating
**When** viewing the loading page
**Then** estimated time remaining is displayed
**And** the estimate accounts for the year multiplier

### AC4: Cancel Option Available
**Given** a multi-year generation is in progress
**When** the user wants to cancel
**Then** a cancel button is available
**And** clicking cancel returns to home page

### AC5: Multi-Year Visual Indicator
**Given** a multi-year scenario (3 or 5 years)
**When** viewing the progress
**Then** a visual indicator shows the year range (e.g., "2026-2030")
**And** year labels appear on the segmented progress bar

---

## Tasks / Subtasks

- [x] **Task 1: Add Year Progress Display** (AC: 1, 2, 5)
  - [x] Calculate current year based on progress percentage
  - [x] Display "Generating Year X of Y..." message
  - [x] Show year range indicator (2026-2030)

- [x] **Task 2: Create Segmented Progress Bar** (AC: 1, 5)
  - [x] Divide progress bar into year segments
  - [x] Add year markers/labels below bar
  - [x] Highlight current year segment

- [x] **Task 3: Update Progress Messaging** (AC: 2, 3)
  - [x] Show year-specific status messages
  - [x] Update estimated time for multi-year scenarios
  - [x] Educational tips reference multi-year data

- [x] **Task 4: Verify Cancel Works** (AC: 4)
  - [x] Confirm cancel button navigates to home (pre-existing)
  - [x] Multi-year scenarios use same cancel logic

---

## Dev Notes

### Implementation Approach

The backend generates all months in a single flow, not year by year. To show year-by-year progress:

1. **Map Progress to Year**: Divide 0-100% progress evenly across years
   - 5 years: 0-20% = Year 1, 20-40% = Year 2, etc.
   - 3 years: 0-33% = Year 1, 33-66% = Year 2, etc.

2. **Display Current Year**: Based on the calculated year from progress percentage

3. **Segmented Progress Bar**: Visual segments with year labels

### Existing Implementation

Current loading page (`app/generate/loading/page.tsx`) already has:
- Progress bar with percentage
- Elapsed/estimated time display
- Cancel button
- Multi-year time estimation (yearMultiplier)

### Files to Modify

| File | Change |
|------|--------|
| `app/generate/loading/page.tsx` | Add year-specific progress UI |

### Dependencies

- Story 2.2 (done): Duration selector passes `durationYears` to loading page
- Story 2.3 (done): Generation handles multi-year scenarios

---

## References

- [Source: docs/epics/epic-02-multi-year-scenarios.md#Story 2.8]
- [Source: app/generate/loading/page.tsx] - Current loading implementation

---

## Dev Agent Record

### Context Reference

Story context created via BMM workflow 2026-01-21

### Agent Model Used

Claude Opus 4.5 (claude-opus-4-5-20251101)

### Completion Notes List

1. **Year Progress Calculation**: Added logic to calculate current year from progress percentage
   - Formula: `Math.ceil((displayProgress / 100) * validDurationYears)`
   - Evenly distributes 0-100% across the number of years

2. **Year Range Indicator**: Added badge showing "2026 – 2030 (5-Year Plan)" style indicator

3. **"Generating Year X of Y..." Message**: Displays below the main title for multi-year scenarios

4. **Segmented Progress Bar**: Added vertical dividers at year boundaries
   - Year markers below the progress bar
   - Current year label highlighted in blue

5. **Multi-Year Educational Tips**: Added 5 new tips specific to multi-year scenarios
   - YoY comparison charts
   - CAGR calculation
   - Strategic inflection points
   - Annual growth patterns
   - Annual summaries in export

6. **Accessibility**: Updated screen reader live region to include year information

### File List

| File | Change |
|------|--------|
| `app/generate/loading/page.tsx` | Added year-by-year progress UI, segmented bar, multi-year tips |
| `docs/sprint-artifacts/2-8-progressive-generation-ui.md` | Created and marked done |
