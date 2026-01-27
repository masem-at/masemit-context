# Story 2.10: Add "Last Updated" Timestamp Display

**Status:** review
**Epic:** 2 - Public Rankings Leaderboard (MVP)
**Story ID:** 2.10
**Created:** 2025-12-22
**Ultimate Context Engine Analysis:** Complete

---

## Story

**As a** user,
**I want** to know when the rankings were last updated,
**So that** I can assess the freshness of the data.

---

## Acceptance Criteria

### Given-When-Then Format (from Epic 2)

**Given** GVS snapshots have snapshot_date timestamps
**When** the leaderboard page renders
**Then** a timestamp display shows near the page header:
  - "Updated: December 19, 2024 at 12:05 AM UTC"
  - OR "Last updated: 2 hours ago" (relative time)
**And** timestamp uses the most recent snapshot_date from the dataset
**And** if data is >7 days old, warning banner shows: "Rankings may be outdated - last updated [date]"
**And** timestamp is visible on both desktop and mobile
**And** timestamp updates after ISR revalidation (1-hour cache)

**Implementation Notes (from Epic):**
- Use date-fns or similar for formatting
- Component: `src/components/LastUpdatedBadge.tsx`
- Warning banner only shows if data staleness detected

---

## Critical Developer Context

### Current Implementation Analysis

**GOOD NEWS: Partial implementation already exists!**

From `src/app/rankings/page.tsx` (lines 86-108):
```tsx
// Get "last updated" timestamp from most recent snapshot
const lastUpdated = rankings.length > 0
  ? rankings[0].snapshotDate
  : new Date()

// ... later in JSX:
<p className="text-sm text-gray-500 mt-4">
  Last updated: {lastUpdated.toLocaleDateString('en-US', {
    year: 'numeric',
    month: 'long',
    day: 'numeric',
    hour: '2-digit',
    minute: '2-digit'
  })}
</p>
```

**Current Output:** "Last updated: December 19, 2024, 12:05 AM"

**Required Output Options:**
1. "Updated: December 19, 2024 at 12:05 AM UTC"
2. OR "Last updated: 2 hours ago"

**GAPS TO FILL:**

| Requirement | Current Status | Action Needed |
|-------------|----------------|---------------|
| Timestamp near header | ✅ Done | None |
| "at X:XX AM UTC" format | ⚠️ Partial | Add "at" and "UTC" suffix |
| Relative time option | ❌ Missing | Add date-fns `formatDistanceToNow` |
| Warning banner >7 days | ❌ Missing | Create new component |
| Mobile visibility | ✅ Done | Verify responsive |
| ISR cache update | ✅ Done | revalidate = 3600 already set |

---

### Previous Story Intelligence (Stories 2.5-2.9)

**Story 2.9 - CTA Pricing** (drafted):
- Created `PRICING_CONFIG` pattern for centralized config
- Could use similar pattern for date formatting options

**Story 2.8 - Accessibility** (review):
- All components must maintain WCAG 2.1 AA compliance
- Screen readers should announce timestamp meaningfully
- Focus indicators: `focus:ring-2 focus:ring-aqua`

**Story 2.7 - Mobile Responsive** (in-progress):
- Card-based layout for mobile
- Timestamp must be readable on small screens

**Story 2.6 - Change Indicators** (done):
- ChangeIndicator component pattern
- Similar badge-style component approach

### Git Intelligence (Recent Commits)

```
863b2a4 fix: correct GVSResult property names in seed-leaderboard script
c2b23ec feat(story-2.8): add keyboard navigation and screen reader support
4de822b feat(story-2.7): implement mobile responsive layout
1b7a1c7 feat(story-2.6): implement week-over-week change indicators
```

**Patterns Established:**
- Components in `src/components/` with PascalCase
- Unit tests in `tests/unit/components/`
- JSDoc comments for public functions
- `cn()` utility for className merging

---

### Architectural Considerations

**date-fns Usage:**
Per project architecture, external libraries should be:
- Imported directly where needed (no barrel exports)
- Tree-shaken for bundle size
- TypeScript-typed

**date-fns Functions Needed:**
```typescript
import { format, formatDistanceToNow, differenceInDays } from 'date-fns'
```

**Component Architecture Options:**

**Option A: Inline Enhancement (Simplest)**
- Modify existing timestamp code in `rankings/page.tsx`
- Add warning banner directly in page
- Pros: Minimal changes, no new files
- Cons: Less reusable

**Option B: Extract to Component (Recommended)**
- Create `LastUpdatedBadge.tsx` per epic notes
- Reusable across pages
- Encapsulate formatting logic
- Pros: Clean, testable, reusable
- Cons: More files

**Recommended: Option B** - Create dedicated component for:
1. Testability
2. Reusability (methodology page also needs "last updated")
3. Separation of concerns

---

## Implementation Strategy

### Phase 1: Install date-fns (if not present)

Check if date-fns is already a dependency:
```bash
npm ls date-fns
```

If not installed:
```bash
npm install date-fns
```

### Phase 2: Create LastUpdatedBadge Component

```tsx
// src/components/LastUpdatedBadge.tsx
'use client'

import { format, formatDistanceToNow, differenceInDays } from 'date-fns'

interface LastUpdatedBadgeProps {
  date: Date
  showRelative?: boolean  // true = "2 hours ago", false = full date
  warnAfterDays?: number  // Default: 7
}

export default function LastUpdatedBadge({
  date,
  showRelative = false,
  warnAfterDays = 7
}: LastUpdatedBadgeProps) {
  const isStale = differenceInDays(new Date(), date) > warnAfterDays

  const formattedDate = showRelative
    ? `Last updated: ${formatDistanceToNow(date, { addSuffix: true })}`
    : `Updated: ${format(date, "MMMM d, yyyy 'at' h:mm a")} UTC`

  return (
    <>
      <p className="text-sm text-gray-500">
        {formattedDate}
      </p>

      {isStale && (
        <div className="mt-2 p-3 bg-amber-50 border border-amber-200 rounded-md">
          <p className="text-sm text-amber-800">
            Rankings may be outdated — last updated {format(date, 'MMMM d, yyyy')}
          </p>
        </div>
      )}
    </>
  )
}
```

### Phase 3: Update Rankings Page

```tsx
// src/app/rankings/page.tsx
import LastUpdatedBadge from '@/components/LastUpdatedBadge'

// Replace existing timestamp code:
<LastUpdatedBadge date={lastUpdated} />
```

### Phase 4: Write Unit Tests

```typescript
// tests/unit/components/last-updated-badge.test.tsx
import { render, screen } from '@testing-library/react'
import { describe, it, expect, vi, beforeEach, afterEach } from 'vitest'
import LastUpdatedBadge from '@/components/LastUpdatedBadge'

describe('LastUpdatedBadge', () => {
  beforeEach(() => {
    vi.useFakeTimers()
    vi.setSystemTime(new Date('2024-12-22T12:00:00Z'))
  })

  afterEach(() => {
    vi.useRealTimers()
  })

  it('renders absolute date format by default', () => {
    const date = new Date('2024-12-22T10:30:00Z')
    render(<LastUpdatedBadge date={date} />)
    expect(screen.getByText(/Updated: December 22, 2024 at 10:30 AM UTC/)).toBeInTheDocument()
  })

  it('renders relative time when showRelative is true', () => {
    const date = new Date('2024-12-22T10:00:00Z')
    render(<LastUpdatedBadge date={date} showRelative />)
    expect(screen.getByText(/Last updated: about 2 hours ago/)).toBeInTheDocument()
  })

  it('shows warning banner when data is stale', () => {
    const staleDate = new Date('2024-12-10T12:00:00Z') // 12 days old
    render(<LastUpdatedBadge date={staleDate} />)
    expect(screen.getByText(/Rankings may be outdated/)).toBeInTheDocument()
  })

  it('does not show warning when data is fresh', () => {
    const freshDate = new Date('2024-12-22T10:00:00Z')
    render(<LastUpdatedBadge date={freshDate} />)
    expect(screen.queryByText(/Rankings may be outdated/)).not.toBeInTheDocument()
  })

  it('respects custom warnAfterDays threshold', () => {
    const date = new Date('2024-12-19T12:00:00Z') // 3 days old
    render(<LastUpdatedBadge date={date} warnAfterDays={2} />)
    expect(screen.getByText(/Rankings may be outdated/)).toBeInTheDocument()
  })
})
```

---

## Files to Create/Modify

### 1. Create: `src/components/LastUpdatedBadge.tsx`

New component for timestamp display with warning.

**Requirements:**
- Accept `date` prop (Date object)
- Accept `showRelative` prop (boolean, default false)
- Accept `warnAfterDays` prop (number, default 7)
- Display formatted date with "UTC" suffix
- Conditionally render warning banner
- WCAG 2.1 AA compliant colors

---

### 2. Modify: `src/app/rankings/page.tsx`

Replace inline timestamp with component.

**Changes:**
- Import `LastUpdatedBadge`
- Replace `<p className="text-sm...">` with `<LastUpdatedBadge date={lastUpdated} />`
- Remove inline date formatting

---

### 3. Create: `tests/unit/components/last-updated-badge.test.tsx`

Comprehensive unit tests.

**Test Coverage:**
- Default absolute date format
- Relative time format
- Warning banner appears when stale
- Warning banner hidden when fresh
- Custom warnAfterDays threshold
- Accessibility (readable text)

---

### 4. Optional: Install date-fns

If not already installed:

```bash
npm install date-fns
```

---

## Technical Requirements

### Date Formatting Requirements (FR21)

| Format | Example | Use Case |
|--------|---------|----------|
| Absolute | "December 22, 2024 at 10:30 AM UTC" | Default display |
| Relative | "2 hours ago" | Alternative display |

### Warning Banner Requirements

| Condition | Display |
|-----------|---------|
| Data < 7 days old | No banner |
| Data ≥ 7 days old | Amber warning banner |

### Accessibility Requirements

- Warning banner must have `role="alert"` for screen readers
- Color contrast must meet WCAG 2.1 AA (amber-800 on amber-50 = 4.5:1+)
- Timestamp must be machine-readable (datetime attribute if using <time>)

---

## Testing Strategy

### Unit Tests (Vitest)

**LastUpdatedBadge:**
- [ ] Renders absolute date format correctly
- [ ] Renders relative time when enabled
- [ ] Shows warning for stale data (>7 days)
- [ ] Hides warning for fresh data
- [ ] Respects custom threshold
- [ ] Has correct ARIA attributes

### Manual Testing Checklist

**Visual:**
- [ ] Timestamp visible on desktop
- [ ] Timestamp visible on mobile
- [ ] Warning banner styling matches design system
- [ ] UTC timezone displayed correctly

**Functional:**
- [ ] Date comes from actual snapshot_date
- [ ] ISR revalidation updates timestamp after 1 hour

**Accessibility:**
- [ ] Screen reader announces timestamp
- [ ] Warning banner announced properly
- [ ] Color contrast passes

---

## Tasks/Subtasks

### Task 1: Check/Install date-fns
- [x] Run `npm ls date-fns` to check if installed
- [x] Install if needed: `npm install date-fns`
- [x] Verify TypeScript types available

### Task 2: Create LastUpdatedBadge Component
- [x] Create `src/components/LastUpdatedBadge.tsx`
- [x] Implement absolute date formatting with UTC suffix
- [x] Implement relative time option
- [x] Add warning banner for stale data
- [x] Add JSDoc comments

### Task 3: Update Rankings Page
- [x] Import LastUpdatedBadge in `rankings/page.tsx`
- [x] Replace inline timestamp with component
- [x] Remove old formatting code
- [x] Verify page renders correctly

### Task 4: Write Unit Tests
- [x] Create `tests/unit/components/last-updated-badge.test.tsx`
- [x] Test absolute date format
- [x] Test relative time format
- [x] Test warning banner logic
- [x] Test accessibility attributes

### Task 5: Validate and Test
- [x] TypeScript build succeeds
- [x] All unit tests pass (18/18)
- [ ] Manual visual inspection (pending)
- [ ] Mobile responsiveness check (pending)
- [ ] Screen reader test (pending)

---

## Dev Agent Record

### Context Reference
<!-- Path(s) to story context XML will be added here by context workflow -->

### Agent Model Used
claude-opus-4-5-20251101

### Debug Log References
- None

### Completion Notes List
- 2025-12-22: Story created with Ultimate Context Engine analysis
- Identified partial implementation exists in rankings/page.tsx
- Main gaps: UTC suffix, relative time, warning banner
- 2025-12-22: Implementation complete
  - Installed date-fns v4.1.0
  - Created LastUpdatedBadge component with absolute/relative formats
  - Added staleness warning banner (>7 days threshold)
  - Updated rankings/page.tsx to use new component
  - All 18 unit tests passing
  - TypeScript build successful

### File List

**Files Created:**
- `src/components/LastUpdatedBadge.tsx`
- `tests/unit/components/last-updated-badge.test.tsx`

**Files Modified:**
- `src/app/rankings/page.tsx`
- `package.json` (added date-fns dependency)
- `package-lock.json`

**Files Referenced:**
- `docs/epics.md` (Story 2.10 requirements, lines 1597-1619)
- `docs/project_context.md` (coding standards)
- `docs/sprint-artifacts/2-9-*.md` (previous story patterns)

---

## Change Log

- **2025-12-22**: Story 2.10 created with Ultimate Context Engine analysis
- **2025-12-22**: Implementation complete - LastUpdatedBadge component created, 18 unit tests passing

---

## Definition of Done

- [x] date-fns installed and available
- [x] LastUpdatedBadge component created
- [x] Absolute date format: "Month D, YYYY at H:MM AM/PM UTC"
- [x] Relative time format available via prop
- [x] Warning banner shows when data >7 days old
- [x] Component used in rankings/page.tsx
- [x] Unit tests pass (18 tests)
- [x] TypeScript builds without errors
- [ ] Visible on desktop and mobile (manual testing pending)
- [x] Accessible (WCAG 2.1 AA compliant - role="alert", <time> element)
- [ ] All acceptance criteria validated (manual testing pending)

---

## References

**Epic 2 Story:** `docs/epics.md` lines 1597-1619
**Architecture:** `docs/architecture.md`
**Project Context:** `docs/project_context.md`
**Previous Stories:**
- `docs/sprint-artifacts/2-6-display-week-over-week-change-indicators.md`
- `docs/sprint-artifacts/2-8-add-keyboard-navigation-and-screen-reader-support.md`
- `docs/sprint-artifacts/2-9-fix-cta-pricing-ambiguity.md`
**Existing Implementation:**
- `src/app/rankings/page.tsx` (lines 86-108)
