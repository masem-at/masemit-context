# Story 2.8: Add Keyboard Navigation and Screen Reader Support

**Status:** in-review
**Epic:** 2 - Public Rankings Leaderboard (MVP)
**Story ID:** 2.8
**Created:** 2025-12-22
**Ultimate Context Engine Analysis:** Complete

---

## Story

**As a** keyboard-only or screen reader user,
**I want** to navigate the leaderboard without a mouse,
**So that** I can access all functionality regardless of my input method.

---

## Acceptance Criteria

### Given-When-Then Format (from Epic 2)

**Given** I navigate using only keyboard
**When** I interact with the leaderboard
**Then** I can Tab through all interactive elements in logical order (NFR-ACCESS-5)
**And** visible focus indicator (2px aqua outline) shows on focused elements (NFR-ACCESS-2)
**And** pressing Enter on a DAO row expands/collapses it (FR25)
**And** pressing Enter on filter dropdown opens options
**And** arrow keys navigate within dropdown menus
**And** pressing Escape closes expanded rows or dropdowns
**And** "Skip to main content" link available at page top (NFR-ACCESS-6)
**And** no keyboard traps exist (NFR-ACCESS-8)
**And** screen reader announces:
  - Table structure with proper headers
  - Current sort order
  - Expanded/collapsed state changes
  - Dynamic content updates (filter changes)
**And** ARIA live regions announce filter/sort changes (NFR-ACCESS-11)
**And** manual testing with NVDA (Windows) and VoiceOver (macOS) passes

**Implementation Notes (from Epic):**
- Use semantic HTML for automatic screen reader support
- Add explicit ARIA labels where needed
- Test focus order matches visual order
- Document keyboard shortcuts in accessibility statement

---

## Critical Developer Context

### Current Implementation Analysis

**Existing Keyboard Support (GOOD):**
- `ExpandableDAORow.tsx`: Has `tabIndex={0}`, `role="button"`, handles Enter/Space (lines 196-203)
- `MobileDAOCard.tsx`: Has `tabIndex={0}`, `role="button"`, handles Enter/Space (lines 168-178)
- `LeaderboardFilters.tsx`: Select and buttons are natively keyboard accessible
- Focus rings: All interactive elements use `focus:ring-2 focus:ring-navy focus:ring-offset-2`

**GAPS Identified (Must Fix):**

1. **Skip Link Missing (NFR-ACCESS-6):**
   - No "Skip to main content" link exists at page top
   - Required for keyboard users to bypass navigation

2. **Escape Key Not Implemented:**
   - `ExpandableDAORow.tsx` and `MobileDAOCard.tsx` don't handle Escape to collapse
   - Epic requires: "pressing Escape closes expanded rows"

3. **Focus Ring Color Mismatch (NFR-ACCESS-2):**
   - Epic specifies "2px aqua outline" but current implementation uses `ring-navy`
   - Must update to `ring-aqua` or `ring-[#00D4FF]` per design system

4. **Screen Reader Announcements (NFR-ACCESS-11):**
   - `LeaderboardFilters.tsx` has `aria-live="polite"` on status div (line 220)
   - BUT: Sort button clicks don't announce sort state clearly
   - Need to verify screen reader announces all state changes

5. **Table Navigation (NFR-ACCESS-9):**
   - `RankingsTable.tsx` has `<table>`, `<thead>`, `<tbody>` semantic structure
   - BUT: `<th>` elements are missing `scope="col"` on some headers
   - Need to add `aria-sort` attribute to sortable columns

6. **Arrow Key Navigation:**
   - Epic requires: "arrow keys navigate within dropdown menus"
   - Native `<select>` handles this automatically
   - BUT: Sort button group (`role="group"`) needs Left/Right arrow key support

7. **Focus Management on Expand:**
   - When row expands, focus stays on trigger (correct)
   - Consider moving focus to expanded content for better UX

8. **Accessibility Statement Page:**
   - NFR-ACCESS-18 requires `/accessibility` page
   - Must document keyboard shortcuts and WCAG compliance

---

### Previous Story Intelligence (Stories 2.1-2.7)

**Story 2.7 - Mobile Responsive Layout** (in-progress):
- Created `MobileDAOCard.tsx` with keyboard support (Enter, Space)
- Already has `tabIndex={0}`, `role="button"`, `aria-expanded`
- NEEDS: Escape key handler, focus ring color update

**Story 2.5 - Expandable Row** (done):
- `ExpandableDAORow.tsx` has full ARIA implementation
- `aria-expanded`, `aria-label`, `aria-live="polite"`
- NEEDS: Escape key handler, focus ring color update

**Story 2.4 - Filter & Sort** (done):
- `LeaderboardFilters.tsx` uses native `<select>` (keyboard accessible)
- Sort buttons have `aria-pressed`, `aria-label`
- Screen reader status div exists but placement could improve

**Story 2.3 - Accessibility Patterns** (done):
- WCAG 2.1 AA color contrast verified (7.89:1+)
- CSS gradient patterns for color-blind users
- Focus indicators exist on all components

**Story 2.2 - RankingsTable** (done):
- Semantic `<table>` with `<caption>` (sr-only)
- Headers have `scope="col"` (line 195-238)
- Missing: `aria-sort` on sortable columns

### Git Intelligence from Recent Commits

**Pattern Analysis (last 10 commits):**
1. **Commit Structure:** `feat(story-X.Y): <description>`
2. **Testing:** Each story gets comprehensive unit tests
3. **Files:** Components in `src/components/`, tests in `tests/unit/components/`
4. **Code Review:** Adversarial review catches accessibility issues

---

## Implementation Strategy

### Phase 1: Skip Link Implementation

Create a reusable skip link component that appears on focus:

```tsx
// src/components/SkipLink.tsx
export default function SkipLink() {
  return (
    <a
      href="#main-content"
      className="sr-only focus:not-sr-only focus:absolute focus:z-50 focus:top-4 focus:left-4 focus:px-4 focus:py-2 focus:bg-aqua focus:text-navy focus:rounded-md focus:font-semibold focus:outline-none focus:ring-2 focus:ring-navy"
    >
      Skip to main content
    </a>
  )
}
```

### Phase 2: Escape Key Handler

Add Escape key support to existing components:

```tsx
// In ExpandableDAORow.tsx and MobileDAOCard.tsx handleKeyDown:
const handleKeyDown = (e: React.KeyboardEvent) => {
  if (e.key === 'Enter' || e.key === ' ') {
    e.preventDefault()
    toggleExpanded()
  } else if (e.key === 'Escape' && isExpanded) {
    e.preventDefault()
    setIsExpanded(false)
    router.push('#')
  }
}
```

### Phase 3: Focus Ring Standardization

Update all focus rings to use aqua color per NFR-ACCESS-2:

```tsx
// Replace: focus:ring-navy
// With: focus:ring-aqua

// In tailwind.config.js, ensure aqua is defined:
// colors: { aqua: '#00D4FF' }
```

### Phase 4: ARIA Sort Attributes

Add `aria-sort` to sortable table headers:

```tsx
// RankingsTable.tsx - Dynamic aria-sort based on current sort state
<th
  scope="col"
  aria-sort={currentSortBy === 'score' ? (sortOrder === 'asc' ? 'ascending' : 'descending') : 'none'}
>
  GVS Score
</th>
```

NOTE: RankingsTable is a Server Component. Sort state comes from parent page via searchParams. Need to pass sortBy/sortOrder as props.

### Phase 5: Arrow Key Navigation for Sort Buttons

Add Left/Right arrow key navigation to sort button group:

```tsx
// LeaderboardFilters.tsx
const handleSortKeyDown = (e: React.KeyboardEvent, currentIndex: number) => {
  if (e.key === 'ArrowRight' || e.key === 'ArrowDown') {
    e.preventDefault()
    const nextIndex = (currentIndex + 1) % SORT_OPTIONS.length
    // Focus next button
  } else if (e.key === 'ArrowLeft' || e.key === 'ArrowUp') {
    e.preventDefault()
    const prevIndex = (currentIndex - 1 + SORT_OPTIONS.length) % SORT_OPTIONS.length
    // Focus previous button
  }
}
```

### Phase 6: Accessibility Statement Page

Create `/accessibility` page with keyboard shortcuts documentation:

```tsx
// src/app/accessibility/page.tsx
export default function AccessibilityPage() {
  return (
    <main id="main-content">
      <h1>Accessibility Statement</h1>
      <section>
        <h2>Keyboard Navigation</h2>
        <ul>
          <li>Tab: Move between interactive elements</li>
          <li>Enter/Space: Activate buttons and expand rows</li>
          <li>Escape: Close expanded rows</li>
          <li>Arrow keys: Navigate sort options</li>
        </ul>
      </section>
      {/* WCAG compliance info, contact details */}
    </main>
  )
}
```

---

## Files to Create/Modify

### 1. Create: `src/components/SkipLink.tsx`

New component for "Skip to main content" functionality.

**Requirements:**
- Visually hidden until focused (`sr-only focus:not-sr-only`)
- High contrast when visible (aqua background, navy text)
- Links to `#main-content` anchor
- 2px focus ring with navy color

---

### 2. Modify: `src/app/layout.tsx`

Add SkipLink component at top of body.

**Changes:**
- Import and render `<SkipLink />` as first child of body

---

### 3. Modify: `src/app/rankings/page.tsx`

Add `id="main-content"` to main element for skip link target.

**Changes:**
- Add `id="main-content"` to `<main>` element
- Pass `sortBy` and `sortOrder` to RankingsTable for aria-sort

---

### 4. Modify: `src/components/ExpandableDAORow.tsx`

Add Escape key handler and update focus ring color.

**Changes:**
- Add `e.key === 'Escape'` case to `handleKeyDown`
- Change `focus:ring-navy` to `focus:ring-aqua`
- Update aria-label for better screen reader experience

---

### 5. Modify: `src/components/MobileDAOCard.tsx`

Add Escape key handler and update focus ring color.

**Changes:**
- Add `e.key === 'Escape'` case to `handleKeyDown`
- Change `focus:ring-navy` to `focus:ring-aqua`

---

### 6. Modify: `src/components/RankingsTable.tsx`

Add `aria-sort` attributes to sortable columns.

**Changes:**
- Accept `sortBy` and `sortOrder` as optional props
- Add `aria-sort` to relevant `<th>` elements
- Ensure all `<th>` have `scope="col"`

---

### 7. Modify: `src/components/LeaderboardFilters.tsx`

Add arrow key navigation to sort button group.

**Changes:**
- Add refs array for sort buttons
- Implement `handleSortKeyDown` for arrow key navigation
- Update focus ring color to aqua

---

### 8. Create: `src/app/accessibility/page.tsx`

New accessibility statement page per NFR-ACCESS-18.

**Requirements:**
- Document WCAG 2.1 Level AA compliance goal
- List keyboard shortcuts
- Provide contact information: hello@chainsights.one
- Include last audit date

---

### 9. Create: `tests/unit/components/skip-link.test.tsx`

Unit tests for SkipLink component.

**Test Coverage:**
- Renders skip link (hidden by default)
- Link has correct href
- Link is focusable
- Link visible on focus (if testable with vitest)

---

### 10. Update: Existing Component Tests

Add Escape key tests to:
- `tests/unit/components/expandable-dao-row.test.tsx`
- `tests/unit/components/mobile-dao-card.test.tsx`

---

## Technical Requirements

### Keyboard Navigation Requirements (NFR-ACCESS-5 through 8)

| Requirement | Status | Notes |
|-------------|--------|-------|
| NFR-ACCESS-5: Full keyboard navigation | Partial | Need arrow keys for sort group |
| NFR-ACCESS-6: Skip links | Missing | Must implement |
| NFR-ACCESS-7: Focus management | OK | Focus stays on trigger |
| NFR-ACCESS-8: No keyboard traps | OK | Verified in existing code |

### Screen Reader Requirements (NFR-ACCESS-9 through 12)

| Requirement | Status | Notes |
|-------------|--------|-------|
| NFR-ACCESS-9: Semantic HTML | OK | Proper table structure |
| NFR-ACCESS-10: ARIA labels | OK | All interactive elements labeled |
| NFR-ACCESS-11: ARIA live regions | Partial | Need aria-sort updates |
| NFR-ACCESS-12: Alt text | N/A | No images on leaderboard |

### Focus Indicator Requirements (NFR-ACCESS-2)

**Current:** `focus:ring-2 focus:ring-navy focus:ring-offset-2`
**Required:** `focus:ring-2 focus:ring-aqua focus:ring-offset-2` (2px aqua outline)

Tailwind `ring-2` = 2px, `ring-aqua` = #00D4FF per design system.

### Touch Target Requirements (NFR-ACCESS-13)

Already implemented in Story 2.7:
- MobileDAOCard: min-h-[60px] (exceeds 44px)
- CTA buttons: min-h-[48px] (exceeds 44px)

---

## Testing Strategy

### Unit Tests (Vitest)

**SkipLink component:**
- Renders with correct href
- Has sr-only class
- Has focus:not-sr-only class

**ExpandableDAORow updates:**
- Escape key collapses expanded row
- Focus ring color is aqua

**MobileDAOCard updates:**
- Escape key collapses expanded card
- Focus ring color is aqua

**LeaderboardFilters updates:**
- Arrow keys navigate sort buttons
- Focus moves correctly between buttons

### Manual Testing Checklist

**Keyboard Testing (Required):**
- [ ] Tab through all interactive elements in logical order
- [ ] Skip link appears on first Tab press
- [ ] Skip link jumps to main content
- [ ] Enter/Space expands/collapses DAO rows
- [ ] Escape closes expanded rows
- [ ] Arrow keys navigate sort buttons
- [ ] No keyboard traps (can always Tab away)

**Screen Reader Testing (Required per NFR-ACCESS-17):**
- [ ] NVDA on Windows:
  - [ ] Table headers announced
  - [ ] Sort order announced
  - [ ] Expanded/collapsed state announced
  - [ ] Filter changes announced via live region
- [ ] VoiceOver on macOS (if available):
  - [ ] Same checks as NVDA

**Focus Indicator Testing:**
- [ ] Focus ring is 2px aqua on all interactive elements
- [ ] Focus visible in both light and dark modes

---

## Tasks/Subtasks

### Task 1: Create Skip Link Component
- [x] Create `src/components/SkipLink.tsx`
- [x] Implement sr-only to visible on focus pattern
- [x] Style with aqua background per design system
- [x] Write unit tests (10 tests passing)

### Task 2: Integrate Skip Link into Layout
- [x] Import SkipLink in `src/app/layout.tsx`
- [x] Add as first child of body
- [x] Add `id="main-content"` to main element in `rankings/page.tsx`
- [x] Test skip link functionality

### Task 3: Add Escape Key Handler
- [x] Update `ExpandableDAORow.tsx` handleKeyDown
- [x] Update `MobileDAOCard.tsx` handleKeyDown
- [x] Write unit tests for Escape key behavior (4 new tests each)

### Task 4: Standardize Focus Ring Color
- [x] Update `ExpandableDAORow.tsx` focus classes to ring-aqua
- [x] Update `MobileDAOCard.tsx` focus classes to ring-aqua
- [x] Update `LeaderboardFilters.tsx` focus classes to ring-aqua
- [x] Verified aqua color defined in Tailwind config (#00E5C0)

### Task 5: Add aria-sort to Table Headers
- [x] Modify `RankingsTable.tsx` to accept sortBy/sortOrder props
- [x] Add `aria-sort` attribute to sortable columns (Rank, GVS Score, Change, Category)
- [x] Update `rankings/page.tsx` to pass sort state

### Task 6: Add Arrow Key Navigation to Sort Buttons
- [x] Add refs array for sort buttons in `LeaderboardFilters.tsx`
- [x] Implement handleSortKeyDown function (Arrow, Home, End keys)
- [x] Add onKeyDown to each sort button

### Task 7: Create Accessibility Statement Page
- [x] Create `src/app/accessibility/page.tsx`
- [x] Document keyboard shortcuts (Tab, Enter, Space, Escape, Arrow, Home, End)
- [x] Add WCAG compliance statement (targeting Level AA)
- [x] Add contact information (hello@chainsights.one, @ChainSights_one)
- [x] Include last audit date (December 2025)

### Task 8: Run Tests and Validate
- [x] All Story 2.8 related unit tests passing
- [x] TypeScript build succeeds (strict mode)
- [ ] Manual keyboard testing (pending)
- [ ] NVDA screen reader testing (pending)

---

## Dev Agent Record

### Context Reference
<!-- Path(s) to story context XML will be added here by context workflow -->

### Agent Model Used
claude-opus-4-5-20251101

### Debug Log References
- None

### Completion Notes List
- 2025-12-22: All code implementation complete
- Created SkipLink component with sr-only to visible-on-focus pattern
- Added Escape key support to ExpandableDAORow and MobileDAOCard
- Standardized all focus rings to aqua color (#00E5C0)
- Added aria-sort attributes to RankingsTable sortable columns
- Implemented arrow key navigation in sort button group
- Created comprehensive /accessibility page with keyboard shortcut documentation
- All Story 2.8 related unit tests passing
- TypeScript build succeeds in strict mode
- NOTE: Manual keyboard/screen reader testing still required before marking done

### Code Review Fixes Applied (2025-12-22)
- **H1 Fixed:** Removed confusing aria-sort inversion logic in RankingsTable.tsx - now correctly reflects visual order
- **M3 Fixed:** Changed role="group" to role="toolbar" in LeaderboardFilters.tsx for better ARIA semantics
- **M2 Fixed:** Added 8 new unit tests for arrow key navigation in LeaderboardFilters
- **M1 Fixed:** Updated test count claim to be accurate ("All Story 2.8 related unit tests passing")
- **Test Fix:** Updated loading state test to check for specific text instead of role="status"

### Re-Review Fixes Applied (2025-12-22)
- **H1 Fixed:** ExpandableDAORow.tsx colSpan changed from 6 to 7 to match all table columns on large screens
- **M2 Fixed:** Added /accessibility link to footer.tsx for discoverability per WCAG guidelines

### File List

**Files to Create:**
- `src/components/SkipLink.tsx`
- `src/app/accessibility/page.tsx`
- `tests/unit/components/skip-link.test.tsx`

**Files to Modify:**
- `src/app/layout.tsx`
- `src/app/rankings/page.tsx`
- `src/components/ExpandableDAORow.tsx`
- `src/components/MobileDAOCard.tsx`
- `src/components/RankingsTable.tsx`
- `src/components/LeaderboardFilters.tsx`
- `tests/unit/components/expandable-dao-row.test.tsx`
- `tests/unit/components/mobile-dao-card.test.tsx`

**Files Referenced:**
- `docs/epics.md` (Story 2.8 requirements, lines 1536-1567)
- `docs/project_context.md` (architecture rules)
- `docs/sprint-artifacts/2-7-*.md` (previous story for patterns)
- `tailwind.config.js` (aqua color definition)

---

## Change Log

- **2025-12-22**: Story 2.8 created with Ultimate Context Engine analysis

---

## Definition of Done

- [ ] Skip link component created and integrated
- [ ] Escape key closes expanded rows/cards
- [ ] Focus ring color is 2px aqua on all interactive elements
- [ ] aria-sort attributes on sortable table columns
- [ ] Arrow key navigation in sort button group
- [ ] Accessibility statement page at `/accessibility`
- [ ] Unit tests pass (skip link, escape key, arrow keys)
- [ ] Manual keyboard testing complete
- [ ] Manual screen reader testing complete (NVDA minimum)
- [ ] TypeScript builds without errors (strict mode)
- [ ] All acceptance criteria validated

---

## References

**Epic 2 Story:** `docs/epics.md` lines 1536-1567
**Architecture:** `docs/architecture.md`
**Project Context:** `docs/project_context.md`
**Previous Stories:**
- `docs/sprint-artifacts/2-5-build-expandable-row-with-component-breakdown.md`
- `docs/sprint-artifacts/2-7-implement-mobile-responsive-layout.md`
**Existing Components:**
- `src/components/ExpandableDAORow.tsx`
- `src/components/MobileDAOCard.tsx`
- `src/components/RankingsTable.tsx`
- `src/components/LeaderboardFilters.tsx`
**Tailwind Config:** `tailwind.config.js`
