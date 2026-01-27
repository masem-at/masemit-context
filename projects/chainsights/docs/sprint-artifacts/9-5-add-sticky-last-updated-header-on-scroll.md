# Story 9.5: Add Sticky "Last Updated" Header on Scroll

## Story Information
- **Epic:** 9 - UX Enhancements (MVP+1 Week)
- **Story ID:** 9.5
- **Status:** Ready for Development
- **Priority:** Medium
- **Complexity:** Medium
- **Estimated Effort:** 4-6 hours

## User Story
As a user scrolling through the leaderboard,
I want to still see when data was updated,
So that I maintain context about freshness while browsing.

## Business Value
Users who scroll through the leaderboard lose visual context about data freshness. The static "Last Updated" timestamp is only visible at the top of the page, forcing users to scroll back up to verify recency. This creates friction and reduces trust in the rankings.

A sticky header that appears on scroll solves this by:
- **Maintaining Trust:** Users always see when data was last updated
- **Reducing Friction:** No need to scroll back to top to check freshness
- **Increasing Transparency:** Expandable details show next update schedule and methodology
- **Mobile Optimization:** Collapsed format works on small screens

**Success Metric:** Sticky header visible when scrolled >200px from top (100% of qualified scrolls)

## Acceptance Criteria

### AC1: Sticky Header Appears on Scroll
**Given** I am viewing the leaderboard page
**When** I scroll down past 200px from the top
**Then** a sticky header appears at the top of the viewport with the following content:
  - Default text: "DAO Rankings • Updated 2 hours ago"
  - Chevron icon (down-facing) indicating expandable content
  - Backdrop blur effect: `bg-navy/95 backdrop-blur-sm`
  - White text on navy background for contrast

### AC2: Sticky Header Disappears When Scrolled to Top
**Given** the sticky header is visible (scroll position >200px)
**When** I scroll back to the top of the page (scroll position <200px)
**Then** the sticky header smoothly disappears (CSS transition)

### AC3: Expandable Dropdown Shows Full Details
**Given** the sticky header is visible
**When** I click the chevron icon or anywhere on the sticky header
**Then** the header expands to show:
  - **Last calculation:** [Full ISO timestamp, e.g., "December 22, 2024 at 12:00 AM UTC"]
  - **Next update:** "Sunday midnight UTC"
  - **Link:** "See methodology →" (links to `/rankings/methodology`)
**And** the chevron icon rotates 180° to indicate expanded state
**And** clicking again collapses the dropdown back to default view

### AC4: Mobile-Friendly Collapsed Format
**Given** I am viewing the leaderboard on a mobile device (screen width <640px)
**When** the sticky header appears
**Then** the header shows a condensed format:
  - Icon only (Clock or Calendar icon from Lucide React)
  - Relative timestamp only (e.g., "2h ago")
  - Expandable dropdown still functions on tap

### AC5: Z-Index Management (No Content Overlap)
**Given** the sticky header is visible
**When** I scroll through the leaderboard
**Then** the sticky header remains fixed at the top of the viewport
**And** the sticky header does NOT cover leaderboard content (proper z-index stacking)
**And** the sticky header appears ABOVE leaderboard rows but BELOW any modal dialogs

### AC6: Accessibility Compliance
**Given** the sticky header is present
**Then**:
  - The header uses semantic HTML (`<header>` tag with `role="banner"`)
  - The expandable button has `aria-expanded` attribute (true/false)
  - The chevron icon has `aria-hidden="true"`
  - Focus styles are visible on keyboard navigation (`focus:ring-2 focus:ring-aqua`)
  - Screen readers announce "DAO Rankings last updated 2 hours ago, expandable button"

### AC7: Performance (Scroll Event Optimization)
**Given** the user is scrolling through the leaderboard
**When** rapid scroll events occur
**Then**:
  - Scroll listener is debounced or throttled to prevent excessive re-renders
  - No visual jank or stuttering during scroll
  - Component renders in <16ms per frame (60fps target)

## Technical Requirements

### Data Requirements
**Source:** Server-side data passed to `StickyUpdateBanner` component

1. **Last Update Timestamp:**
   - Type: `Date | string (ISO 8601)`
   - Source: `gvsSnapshots` table, `calculatedAt` field (most recent snapshot)
   - Query: `SELECT MAX(calculatedAt) FROM gvsSnapshots`
   - Format for display: Use `formatDistanceToNow()` from `date-fns` for relative time

2. **Next Update Schedule:**
   - Hardcoded: "Sunday midnight UTC" (matches FR12 - weekly recalculation job)
   - Consider: Calculate dynamically based on current date if needed

3. **Methodology Link:**
   - Hardcoded: `/rankings/methodology`

### Component Architecture

**Component:** `src/components/StickyUpdateBanner.tsx`

**Type:** Client Component (`'use client'` directive required)
- Reason: Uses React hooks (`useState`, `useEffect`) and browser APIs (`window.scrollY`)

**Props Interface:**
```typescript
export interface StickyUpdateBannerProps {
  /** ISO 8601 timestamp of last GVS calculation */
  lastUpdated: string | Date
  /** Optional: Custom z-index value (default: 40) */
  zIndex?: number
  /** Optional: Scroll threshold in pixels (default: 200) */
  scrollThreshold?: number
  /** Optional: Additional CSS classes for container */
  className?: string
}
```

**Component Structure:**
```typescript
'use client'

import { useState, useEffect } from 'react'
import { formatDistanceToNow } from 'date-fns'
import { ChevronDown, Clock } from 'lucide-react'
import { cn } from '@/lib/utils'
import Link from 'next/link'

export default function StickyUpdateBanner({
  lastUpdated,
  zIndex = 40,
  scrollThreshold = 200,
  className
}: StickyUpdateBannerProps) {
  const [isVisible, setIsVisible] = useState(false)
  const [isExpanded, setIsExpanded] = useState(false)

  useEffect(() => {
    // Scroll listener with throttle
    let timeoutId: NodeJS.Timeout | null = null

    const handleScroll = () => {
      if (timeoutId) return

      timeoutId = setTimeout(() => {
        setIsVisible(window.scrollY > scrollThreshold)
        timeoutId = null
      }, 100) // 100ms throttle
    }

    window.addEventListener('scroll', handleScroll, { passive: true })
    return () => {
      window.removeEventListener('scroll', handleScroll)
      if (timeoutId) clearTimeout(timeoutId)
    }
  }, [scrollThreshold])

  if (!isVisible) return null

  const relativeTime = formatDistanceToNow(new Date(lastUpdated), { addSuffix: true })
  const fullTimestamp = new Date(lastUpdated).toLocaleString('en-US', {
    dateStyle: 'long',
    timeStyle: 'short',
    timeZone: 'UTC'
  })

  return (
    <header
      className={cn(
        'fixed top-0 left-0 right-0',
        'bg-navy/95 backdrop-blur-sm text-white',
        'transition-all duration-300 ease-in-out',
        className
      )}
      style={{ zIndex }}
      role="banner"
    >
      <div className="container mx-auto px-4 py-3">
        <button
          onClick={() => setIsExpanded(!isExpanded)}
          className={cn(
            'flex items-center justify-between w-full',
            'focus:outline-none focus:ring-2 focus:ring-aqua focus:ring-offset-2'
          )}
          aria-expanded={isExpanded}
          aria-label={`DAO Rankings last updated ${relativeTime}, expandable button`}
        >
          {/* Default view (collapsed) */}
          <div className="flex items-center gap-2">
            <Clock className="w-4 h-4 sm:hidden" aria-hidden="true" />
            <span className="text-sm font-medium">
              <span className="hidden sm:inline">DAO Rankings • Updated </span>
              <span className="sm:hidden">{relativeTime}</span>
              <span className="hidden sm:inline">{relativeTime}</span>
            </span>
          </div>

          <ChevronDown
            className={cn(
              'w-4 h-4 transition-transform duration-200',
              isExpanded && 'rotate-180'
            )}
            aria-hidden="true"
          />
        </button>

        {/* Expanded view (dropdown) */}
        {isExpanded && (
          <div className="mt-3 pt-3 border-t border-white/20 space-y-2 text-sm">
            <p><strong>Last calculation:</strong> {fullTimestamp}</p>
            <p><strong>Next update:</strong> Sunday midnight UTC</p>
            <Link
              href="/rankings/methodology"
              className="inline-flex items-center text-aqua hover:underline focus:outline-none focus:ring-2 focus:ring-aqua"
            >
              See methodology →
            </Link>
          </div>
        )}
      </div>
    </header>
  )
}
```

### Integration Points

**File:** `src/app/rankings/page.tsx`

**Changes Required:**
1. Fetch last update timestamp from database
2. Import and render `StickyUpdateBanner` component
3. Pass `lastUpdated` prop to component

**Example Integration:**
```typescript
// In rankings page.tsx (Server Component)
import StickyUpdateBanner from '@/components/StickyUpdateBanner'
import { db } from '@/lib/db'
import { gvsSnapshots } from '@/lib/db/schema'
import { sql } from 'drizzle-orm'

export default async function RankingsPage({ searchParams }: { searchParams: { category?: string } }) {
  // ... existing code for fetching rankings ...

  // Fetch last update timestamp
  const lastUpdateResult = await db
    .select({ calculatedAt: gvsSnapshots.calculatedAt })
    .from(gvsSnapshots)
    .orderBy(sql`${gvsSnapshots.calculatedAt} DESC`)
    .limit(1)

  const lastUpdated = lastUpdateResult[0]?.calculatedAt || new Date()

  return (
    <>
      {/* Sticky header component */}
      <StickyUpdateBanner lastUpdated={lastUpdated} />

      {/* Existing leaderboard content */}
      <main className="container mx-auto px-4 py-8">
        {/* ... existing JSX ... */}
      </main>
    </>
  )
}
```

### Styling Requirements

**Tailwind Classes:**
- Background: `bg-navy/95 backdrop-blur-sm` (95% opacity + blur)
- Text: `text-white` (contrast 4.5:1 minimum on navy)
- Transition: `transition-all duration-300 ease-in-out` (smooth appearance)
- Focus: `focus:ring-2 focus:ring-aqua focus:ring-offset-2`
- Responsive: `hidden sm:inline` for desktop text, `sm:hidden` for mobile-only content

**Z-Index Values:**
- Sticky header: `z-40` (above leaderboard content)
- Leaderboard rows: `z-10` (default)
- Modal dialogs: `z-50` (above sticky header)

### Performance Optimizations

**Scroll Event Throttling:**
- Implement 100ms throttle on scroll listener to prevent excessive re-renders
- Use `{ passive: true }` flag for scroll event listener (improves scroll performance)

**Debounced Alternative (if needed):**
```typescript
import { debounce } from 'lodash-es'

useEffect(() => {
  const handleScroll = debounce(() => {
    setIsVisible(window.scrollY > scrollThreshold)
  }, 100)

  window.addEventListener('scroll', handleScroll, { passive: true })
  return () => {
    handleScroll.cancel()
    window.removeEventListener('scroll', handleScroll)
  }
}, [scrollThreshold])
```

### Accessibility Requirements (WCAG 2.1 AA)

1. **Semantic HTML:**
   - Use `<header role="banner">` for sticky header
   - Use `<button>` for expandable control (not div with onClick)

2. **ARIA Attributes:**
   - `aria-expanded={isExpanded}` on button
   - `aria-hidden="true"` on decorative icons
   - `aria-label` describing button purpose

3. **Keyboard Navigation:**
   - Tab key focuses button
   - Enter/Space toggles expanded state
   - Focus ring visible (`focus:ring-2 focus:ring-aqua`)

4. **Screen Reader Support:**
   - Button announces: "DAO Rankings last updated 2 hours ago, expandable button"
   - Expanded content is read aloud when expanded
   - Link to methodology is keyboard accessible

5. **Color Contrast:**
   - White text on navy background: Verify 4.5:1 minimum contrast
   - Aqua link color: Verify 3:1 contrast on navy background

### Testing Requirements

**Unit Tests:** `tests/unit/components/sticky-update-banner.test.tsx`

Minimum 20 tests covering:
1. **Rendering Logic (5 tests)**
   - Should NOT render when scroll position is <200px
   - Should render when scroll position is ≥200px
   - Should display relative timestamp (e.g., "2 hours ago")
   - Should display chevron icon
   - Should display clock icon on mobile screens

2. **Scroll Behavior (4 tests)**
   - Should appear when scrolling down past threshold
   - Should disappear when scrolling back to top
   - Should throttle scroll events (performance check)
   - Should cleanup event listener on unmount

3. **Expandable Dropdown (4 tests)**
   - Should expand dropdown when button clicked
   - Should collapse dropdown when button clicked again
   - Should rotate chevron icon 180° when expanded
   - Should display full timestamp, next update, methodology link when expanded

4. **Responsive Behavior (3 tests)**
   - Should show full text on desktop (screen width ≥640px)
   - Should show condensed format on mobile (screen width <640px)
   - Should show clock icon only on mobile

5. **Accessibility (4 tests)**
   - Should use semantic `<header>` tag with `role="banner"`
   - Should have `aria-expanded` attribute on button
   - Should have `aria-hidden="true"` on decorative icons
   - Should have focus ring styles on button

**Integration Tests:** `tests/unit/components/sticky-update-banner-integration.test.tsx`

Minimum 8 tests covering:
1. **Scroll Threshold Edge Cases (3 tests)**
   - Should NOT appear at scroll position 199px (boundary test)
   - Should appear at scroll position 200px (boundary test)
   - Should appear at scroll position 201px (above threshold)

2. **Timestamp Formatting (2 tests)**
   - Should format recent timestamp as "2 hours ago"
   - Should format full timestamp as "December 22, 2024 at 12:00 AM UTC"

3. **Link Integration (1 test)**
   - Should link to `/rankings/methodology` page

4. **Z-Index Management (2 tests)**
   - Should use default z-index 40 when not specified
   - Should accept custom z-index via prop

**Total Test Count Target:** 28 tests (20 unit + 8 integration)

### Manual Testing Checklist

**Desktop Testing:**
- [ ] Test in Chrome (latest version)
- [ ] Test in Firefox (latest version)
- [ ] Test in Safari (latest version)
- [ ] Verify sticky header appears at 200px scroll threshold
- [ ] Verify smooth transition when appearing/disappearing
- [ ] Verify expandable dropdown opens/closes on click
- [ ] Verify chevron icon rotation animation
- [ ] Verify methodology link navigates to correct page
- [ ] Verify keyboard navigation (Tab to focus, Enter to toggle)
- [ ] Verify focus ring is visible

**Mobile Testing:**
- [ ] Test on iOS Safari (iPhone)
- [ ] Test on Android Chrome (Android device)
- [ ] Verify condensed format displays on small screens
- [ ] Verify clock icon appears on mobile
- [ ] Verify tappable area is large enough (44x44px minimum)
- [ ] Verify expandable dropdown works on mobile tap
- [ ] Verify no horizontal scrolling occurs

**Performance Testing:**
- [ ] Monitor scroll event throttling (no excessive re-renders)
- [ ] Verify no visual jank during scroll
- [ ] Check Chrome DevTools Performance tab (should be <16ms per frame)

**Accessibility Testing:**
- [ ] Run axe-core DevTools extension (0 violations expected)
- [ ] Test with NVDA screen reader (Windows)
- [ ] Test with VoiceOver screen reader (macOS/iOS)
- [ ] Verify all interactive elements are keyboard accessible
- [ ] Verify color contrast meets WCAG 2.1 AA standards

## Implementation Plan

### Phase 1: Component Creation (TDD - RED Phase)
**Task 1.1:** Write failing unit tests for `StickyUpdateBanner` component (20 tests)
- [ ] Write rendering logic tests (5 tests)
- [ ] Write scroll behavior tests (4 tests)
- [ ] Write expandable dropdown tests (4 tests)
- [ ] Write responsive behavior tests (3 tests)
- [ ] Write accessibility tests (4 tests)
- [ ] Run tests to confirm RED phase (all failing)

**Task 1.2:** Write failing integration tests (8 tests)
- [ ] Write scroll threshold edge case tests (3 tests)
- [ ] Write timestamp formatting tests (2 tests)
- [ ] Write link integration test (1 test)
- [ ] Write z-index management tests (2 tests)
- [ ] Run tests to confirm RED phase (all failing)

### Phase 2: Component Implementation (TDD - GREEN Phase)
**Task 2.1:** Create `StickyUpdateBanner` component
- [ ] Create file: `src/components/StickyUpdateBanner.tsx`
- [ ] Add `'use client'` directive
- [ ] Define `StickyUpdateBannerProps` interface
- [ ] Implement scroll listener with throttle
- [ ] Implement visibility toggle logic (200px threshold)
- [ ] Implement expandable dropdown logic
- [ ] Add timestamp formatting with `date-fns`
- [ ] Add responsive mobile/desktop views
- [ ] Add accessibility attributes (aria-expanded, aria-label, role)
- [ ] Run tests to confirm GREEN phase (all passing)

**Task 2.2:** Add JSDoc documentation (TDD - REFACTOR Phase)
- [ ] Add component-level JSDoc comment
- [ ] Add props interface documentation
- [ ] Add usage example in JSDoc
- [ ] Add inline comments for complex logic (scroll throttle, cleanup)

### Phase 3: Database Integration
**Task 3.1:** Verify data source exists
- [ ] Query database: `SELECT MAX(calculatedAt) FROM gvsSnapshots`
- [ ] Verify `calculatedAt` field exists and is populated
- [ ] Test query in Drizzle Studio or SQL client
- [ ] Document fallback behavior if no snapshots exist (use current date)

**Task 3.2:** Update `rankings/page.tsx` to fetch last update timestamp
- [ ] Import necessary database modules
- [ ] Add query to fetch most recent `calculatedAt` timestamp
- [ ] Handle edge case: No snapshots exist (fallback to current date)
- [ ] Pass `lastUpdated` prop to `StickyUpdateBanner` component

### Phase 4: Integration & Testing
**Task 4.1:** Integrate component into rankings page
- [ ] Import `StickyUpdateBanner` component
- [ ] Render component at top of page (before main content)
- [ ] Verify component receives correct `lastUpdated` prop
- [ ] Test scroll behavior in development server

**Task 4.2:** Run all tests
- [ ] Run unit tests: `npm run test:unit`
- [ ] Run integration tests (if separate): `npm run test:integration`
- [ ] Verify 28/28 tests passing
- [ ] Check test coverage report (aim for 100% coverage)

**Task 4.3:** Manual browser testing (use checklist above)
- [ ] Test desktop browsers (Chrome, Firefox, Safari)
- [ ] Test mobile browsers (iOS Safari, Android Chrome)
- [ ] Test keyboard navigation
- [ ] Test screen reader compatibility

**Task 4.4:** Build verification
- [ ] Run production build: `npm run build`
- [ ] Verify no TypeScript errors
- [ ] Verify no ESLint errors
- [ ] Check bundle size impact (aim for <3KB increase)

### Phase 5: Documentation & Handoff
**Task 5.1:** Update story documentation
- [ ] Mark all implementation tasks as [x] completed
- [ ] Add implementation notes section
- [ ] Document any deviations from original plan
- [ ] Document any challenges encountered and solutions
- [ ] Add list of files created/modified

**Task 5.2:** Update sprint status
- [ ] Update `docs/sprint-artifacts/sprint-status.yaml`
- [ ] Change story status from `ready-for-dev` → `in-progress` → `review`

**Task 5.3:** Create git commit
- [ ] Stage all files related to Story 9.5
- [ ] Create atomic commit with story-specific changes only
- [ ] Use commit message format from Stories 9.3-9.4 pattern
- [ ] Push to feature branch: `feature/story-9.5`

## Dependencies

### Internal Dependencies
- **Epic 1 (GVS Calculation Engine):** Required - `gvsSnapshots.calculatedAt` field must exist
- **Story 2.1 (Leaderboard Server Component):** Required - Integration point in `rankings/page.tsx`

### External Dependencies
- **date-fns:** Already installed (`^2.30.0` in package.json) - for `formatDistanceToNow()`
- **lucide-react:** Already installed (`^0.344.0` in package.json) - for `ChevronDown`, `Clock` icons
- **Next.js Link:** Built-in (`next/link`) - for methodology link

### Technical Constraints
- Must use Client Component (`'use client'`) due to scroll listener and React hooks
- Must throttle scroll events to prevent performance issues
- Must cleanup event listeners on unmount to prevent memory leaks
- Must use `{ passive: true }` flag for scroll listener (browser optimization)

## Risks & Mitigation

### Risk 1: Scroll Event Performance Issues
**Risk:** Rapid scroll events can cause excessive re-renders and UI jank
**Impact:** High (poor user experience)
**Mitigation:**
- Implement 100ms throttle on scroll listener
- Use `{ passive: true }` flag for browser optimization
- Test with Chrome DevTools Performance profiler
- Fallback: Use debounce instead of throttle if needed

### Risk 2: Z-Index Conflicts with Existing UI
**Risk:** Sticky header may overlap with modal dialogs or other fixed elements
**Impact:** Medium (visual bugs)
**Mitigation:**
- Use standardized z-index scale (header: 40, modals: 50)
- Document z-index values in component props
- Test all pages with sticky header + modal combinations

### Risk 3: Mobile Viewport Height Issues
**Risk:** Sticky header may take up too much screen space on small mobile devices
**Impact:** Medium (reduced content visibility)
**Mitigation:**
- Use condensed format on mobile (<640px screens)
- Minimize header height (py-3 instead of py-4)
- Test on real devices with various screen sizes

### Risk 4: Date Formatting Inconsistencies
**Risk:** `date-fns` may format dates differently across browsers or timezones
**Impact:** Low (cosmetic issue)
**Mitigation:**
- Always parse dates as UTC timestamps
- Test date formatting in multiple browsers
- Add timezone suffix to full timestamp display ("UTC")

## Definition of Done

- [ ] All 28 tests passing (20 unit + 8 integration)
- [ ] Component integrated into `rankings/page.tsx`
- [ ] Sticky header appears at 200px scroll threshold
- [ ] Expandable dropdown functions correctly
- [ ] Mobile-responsive (condensed format on small screens)
- [ ] Accessibility: WCAG 2.1 AA compliant (axe-core 0 violations)
- [ ] Keyboard navigation works (Tab, Enter, Space)
- [ ] Screen reader tested (NVDA or VoiceOver)
- [ ] Manual browser testing complete (desktop + mobile)
- [ ] Production build succeeds (`npm run build`)
- [ ] ESLint passes (`npm run lint`)
- [ ] Story documentation updated with implementation notes
- [ ] Sprint status updated to `review`
- [ ] Atomic git commit created for Story 9.5 only

## Related Documents
- Epic 9 requirements: `docs/epics.md` (lines 2642-2668)
- UX validation report: `docs/design/dao-rankings-leaderboard-ux-validation.md`
- Architecture decisions: `docs/architecture.md`
- Database schema: `src/lib/db/schema.ts` (gvsSnapshots table)
- Previous story patterns: Stories 9.3, 9.4 in `docs/sprint-artifacts/`

## Notes for Developer

### Key Learnings from Stories 9.3-9.4

**Proven Component Pattern:**
- Props interface + Configuration object + Helper functions
- Early null checks to prevent rendering bugs
- Semantic HTML + ARIA attributes for accessibility
- Tailwind utilities with `cn()` for conditional classes

**Testing Strategy:**
- TDD approach (RED-GREEN-REFACTOR) reduces bugs and ensures coverage
- Aim for 25-30 tests per story (unit + integration)
- Test boundary conditions (e.g., 199px vs 200px scroll position)
- Mock external dependencies (date-fns, window APIs)

**Git Workflow:**
- Create dedicated feature branch: `feature/story-9.5`
- Ensure clean working directory (no contamination from other stories)
- Atomic commits with clear commit messages
- Follow commit message format from Stories 9.3-9.4

**Common Pitfalls to Avoid:**
- ❌ Forgetting to cleanup event listeners (memory leak)
- ❌ Not throttling/debouncing scroll events (performance issues)
- ❌ Using `div` with `onClick` instead of semantic `<button>` (accessibility fail)
- ❌ Forgetting `'use client'` directive (hydration errors)
- ❌ Not testing on real mobile devices (responsive bugs)

### Scroll Listener Best Practices

**DO:**
- ✅ Use `{ passive: true }` flag for scroll listeners
- ✅ Throttle or debounce scroll events (100ms recommended)
- ✅ Cleanup listeners in `useEffect` return function
- ✅ Store scroll position in state, not directly in DOM

**DON'T:**
- ❌ Attach scroll listeners without throttle/debounce
- ❌ Use `setState()` on every scroll event (causes excessive re-renders)
- ❌ Forget to remove listeners on unmount (memory leak)
- ❌ Use synchronous scroll listeners (blocks main thread)

### Date Formatting Reference

**Using date-fns:**
```typescript
import { formatDistanceToNow } from 'date-fns'

// Relative time (e.g., "2 hours ago")
const relativeTime = formatDistanceToNow(new Date(lastUpdated), { addSuffix: true })

// Full timestamp (e.g., "December 22, 2024 at 12:00 AM")
const fullTimestamp = new Date(lastUpdated).toLocaleString('en-US', {
  dateStyle: 'long',
  timeStyle: 'short',
  timeZone: 'UTC'
})
```

### Z-Index Scale Reference

| Element | Z-Index | Usage |
|---------|---------|-------|
| Leaderboard rows | 10 | Default content |
| Sticky header | 40 | Fixed position header |
| Modal dialogs | 50 | Overlays |
| Toasts/notifications | 60 | Temporary alerts |

## Change Log

### 2024-12-25 - Story Created
- Initial story creation by SM (Sonnet 4.5)
- Comprehensive analysis of Epic 9 context and Stories 9.3-9.4 learnings
- Detailed technical requirements and implementation plan
- TDD approach with 28 tests planned
- Integration points identified and documented
- Risk assessment and mitigation strategies defined
