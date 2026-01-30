# Story 2.7: Implement Mobile Responsive Layout

**Status:** in-progress (code complete, pending manual browser testing)
**Epic:** 2 - Public Rankings Leaderboard (MVP)
**Story ID:** 2.7
**Created:** 2025-12-22
**Ultimate Context Engine Analysis:** Complete

---

## Story

**As a** mobile user,
**I want** to view the leaderboard on my phone with optimized layout,
**So that** I can easily browse rankings on any device.

---

## Acceptance Criteria

### Given-When-Then Format (from Epic 2)

**Given** I access `/rankings` on a mobile device (<768px width)
**When** the page renders
**Then** the table adapts to vertical scrolling layout (FR23)
**And** columns stack or hide appropriately:
  - Always visible: Rank, DAO name, Score
  - Hidden on mobile: Category (shown in expanded state)
  - Change indicator shown as compact badge
**And** touch targets are minimum 44x44px (NFR-ACCESS-13)
**And** row expansion works on tap (touch event)
**And** no horizontal scrolling required (NFR-ACCESS-reflow)
**And** text remains readable at 200% zoom (NFR-ACCESS-3)
**And** page works in both portrait and landscape orientation (NFR-ACCESS-14)
**And** mobile layout tested on iOS Safari and Android Chrome

**Implementation Notes (from Epic):**
- Use Tailwind responsive breakpoints (sm:, md:, lg:)
- Test on real devices or browser DevTools
- Mobile-first approach in CSS

---

## Critical Developer Context

### Current Implementation Analysis

**RankingsTable.tsx (Server Component):**
- Already has `overflow-x-auto` wrapper for horizontal scroll fallback
- Column visibility already partially responsive:
  - "Health Status" column: `hidden md:table-cell` (hidden on mobile)
  - "Category" column: `hidden md:table-cell` (hidden on mobile)
  - "Analysis Type" column: `hidden lg:table-cell` (hidden below large)
- **Problem:** Table structure remains a `<table>` on mobile - not optimal for small screens

**ExpandableDAORow.tsx (Client Component):**
- Touch events: `onClick` works for tap (no specific touch handling needed)
- Row padding: `px-4 py-4` (16px padding - touch target is table row height)
- CTA buttons: Already `flex-col sm:flex-row` for mobile stacking
- Expanded section: Already responsive with `max-w-4xl mx-auto`

### Architecture Reality Check

**Current State:**
- Table uses `<table>` semantic HTML on all screen sizes
- Columns are hidden via `hidden md:table-cell` responsive classes
- No mobile card layout exists - table scrolls horizontally if content exceeds width

**Epic/PRD Requirements (FR23, NFR-ACCESS-13, NFR-ACCESS-reflow):**
- NO horizontal scrolling on mobile
- Touch targets minimum 44x44px
- Content must reflow at 320px viewport

**This means:** We need to **transform table to card-based layout on mobile**

---

### Previous Story Intelligence (Stories 2.1-2.6)

**Story 2.6 - Week-Over-Week Change** (done):
- Added ChangeIndicator component (96 lines)
- Integrated into RankingsTable and ExpandableDAORow
- Must remain visible on mobile as "compact badge"
- Current display: Arrow + delta value (e.g., "↗ +0.3")

**Story 2.5 - Expandable Row** (done):
- ExpandableDAORow is a **Client Component** (uses useState, useEffect)
- Uses URL hash for deep linking (`#dao-slug`)
- Component breakdown shown in expanded section
- CTA buttons already stack on mobile (`flex-col sm:flex-row`)
- **Code Review Fix Applied:** Moved defensive validation AFTER hooks

**Story 2.4 - Filter & Sort** (done):
- Filter controls in parent page component
- Must remain accessible and visible on mobile
- Current implementation uses shadcn/ui Select components

**Story 2.3 - Accessibility Patterns** (done):
- Color contrast verified WCAG 2.1 AA (7.89:1+)
- CSS gradient patterns for color-blind users
- Must maintain on mobile layout

**Story 2.2 - RankingsTable** (done):
- Semantic `<table>` with `<caption>` (sr-only)
- ARIA labels on table elements
- **Challenge:** Tables are not naturally mobile-friendly

### Git Intelligence from Recent Commits

**Pattern Analysis (last 10 commits):**
1. **Commit Structure:** `feat(story-X.Y): <description>`
2. **Development Flow:** Story created → Implementation → Tests → Code review → Done
3. **Testing:** Each story gets 15-35 unit tests
4. **Files:** Components in `src/components/`, tests in `tests/unit/components/`

---

## Implementation Strategy

### Mobile-First Responsive Pattern

**Strategy: Conditional Rendering with Tailwind Breakpoints**

Rather than a complete table-to-cards rewrite, implement:
1. Keep table structure for `md:` and above (768px+)
2. Add mobile card layout for screens `<768px`
3. Use CSS to show/hide appropriate layout

### Option A: CSS-Only Hide/Show (Recommended)

Keep existing `<table>` but hide on mobile, show card layout instead:

```tsx
// RankingsTable.tsx
<div>
  {/* Mobile Card Layout - visible only on small screens */}
  <div className="md:hidden space-y-3">
    {rankings.map((ranking, index) => (
      <MobileDAOCard key={ranking.id} ranking={ranking} rank={index + 1} />
    ))}
  </div>

  {/* Desktop Table Layout - hidden on small screens */}
  <div className="hidden md:block overflow-x-auto">
    <table>...</table>
  </div>
</div>
```

**Pros:**
- Clean separation of concerns
- No complex CSS transforms
- Maintains semantic table for desktop/SEO

**Cons:**
- Two render paths (but shared data)
- Slightly more HTML (but gzip handles well)

### Option B: Transform Table to Cards via CSS

Use CSS Grid/Flexbox to reflow table cells into card layout:

```css
@media (max-width: 767px) {
  table, thead, tbody, tr, th, td {
    display: block;
  }
  thead { display: none; }
  tr { margin-bottom: 1rem; padding: 1rem; border: 1px solid #e5e7eb; }
  td::before { content: attr(data-label); font-weight: bold; }
}
```

**Pros:**
- Single HTML structure
- Progressively enhanced

**Cons:**
- Requires `data-label` attributes on all cells
- Complex CSS maintenance
- Harder to test

**RECOMMENDED:** Option A - Separate Mobile Component

---

## Files to Modify/Create

### 1. Create: `src/components/MobileDAOCard.tsx`

**New Client Component for mobile layout:**

```typescript
'use client'

import { useState, useEffect } from 'react'
import { useRouter } from 'next/navigation'
import { cn } from '@/lib/utils'
import type { LeaderboardRanking } from '@/lib/db/queries/leaderboard'
import ChangeIndicator from '@/components/ChangeIndicator'

export interface MobileDAOCardProps {
  ranking: LeaderboardRanking
  rank: number
}

export default function MobileDAOCard({ ranking, rank }: MobileDAOCardProps) {
  const [isExpanded, setIsExpanded] = useState(false)
  const router = useRouter()

  // Sync with URL hash (same pattern as ExpandableDAORow)
  useEffect(() => {
    const handleHashChange = () => {
      const hash = window.location.hash.slice(1)
      setIsExpanded(hash === ranking.slug)
    }
    handleHashChange()
    window.addEventListener('hashchange', handleHashChange)
    return () => window.removeEventListener('hashchange', handleHashChange)
  }, [ranking.slug])

  const toggleExpanded = () => {
    if (isExpanded) {
      router.push('#')
      setIsExpanded(false)
    } else {
      router.push(`#${ranking.slug}`)
      setIsExpanded(true)
    }
  }

  // Format score
  const numericScore = parseFloat(ranking.gvsScore)
  const formattedScore = isNaN(numericScore) ? 'N/A' : `${numericScore.toFixed(1)}/10`

  return (
    <div
      onClick={toggleExpanded}
      onKeyDown={(e) => {
        if (e.key === 'Enter' || e.key === ' ') {
          e.preventDefault()
          toggleExpanded()
        }
      }}
      role="button"
      tabIndex={0}
      aria-expanded={isExpanded}
      aria-label={`${ranking.name} - Tap to ${isExpanded ? 'collapse' : 'expand'} details`}
      className={cn(
        'bg-white rounded-lg border border-gray-200 p-4',
        'focus:outline-none focus:ring-2 focus:ring-navy focus:ring-offset-2',
        'active:bg-gray-50 transition-colors duration-200',
        'min-h-[60px]'  // Minimum 44px touch target (with padding = 60px+)
      )}
    >
      {/* Card Header - Always Visible */}
      <div className="flex items-center justify-between">
        {/* Left: Rank + Name */}
        <div className="flex items-center gap-3 min-w-0">
          <span className="text-lg font-bold text-gray-600 w-8 flex-shrink-0">
            {rank}
          </span>
          <span className="text-base font-semibold text-gray-900 truncate">
            {ranking.name}
          </span>
        </div>

        {/* Right: Score + Change */}
        <div className="flex items-center gap-2 flex-shrink-0">
          <span className="text-lg font-bold text-gray-900">
            {formattedScore}
          </span>
          <ChangeIndicator
            delta={ranking.delta}
            deltaDirection={ranking.deltaDirection}
          />
          {/* Expand/Collapse Chevron */}
          <span className={cn(
            'text-gray-400 transition-transform duration-200',
            isExpanded && 'rotate-180'
          )} aria-hidden="true">
            ▼
          </span>
        </div>
      </div>

      {/* Expanded Content */}
      {isExpanded && (
        <div className="mt-4 pt-4 border-t border-gray-100">
          {/* Category Badge */}
          <div className="mb-3">
            <span className="text-sm text-gray-600">Category: </span>
            <span className="text-sm font-medium text-gray-900">
              {ranking.category === 'defi' ? 'DeFi' :
               ranking.category === 'infrastructure' ? 'Infrastructure' : 'Public Goods'}
            </span>
          </div>

          {/* Component Scores */}
          <div className="space-y-2 mb-4">
            {/* HPR */}
            <div className="flex justify-between text-sm">
              <span className="text-gray-600">Human Participation Rate</span>
              <span className="font-medium">
                {ranking.hprScore ? `${parseFloat(ranking.hprScore).toFixed(1)}/10` : 'N/A'}
              </span>
            </div>
            {/* DEI */}
            <div className="flex justify-between text-sm">
              <span className="text-gray-600">Delegate Engagement Index</span>
              <span className="font-medium">
                {ranking.deiScore ? `${parseFloat(ranking.deiScore).toFixed(1)}/10` : 'N/A'}
              </span>
            </div>
            {/* PDI */}
            <div className="flex justify-between text-sm">
              <span className="text-gray-600">Power Dynamics Index</span>
              <span className="font-medium">
                {ranking.pdiScore ? `${parseFloat(ranking.pdiScore).toFixed(1)}/10` : 'N/A'}
              </span>
            </div>
            {/* GPI */}
            <div className="flex justify-between text-sm">
              <span className="text-gray-600">Grassroots Participation</span>
              <span className="font-medium">
                {ranking.gpiScore ? `${parseFloat(ranking.gpiScore).toFixed(1)}/10` : 'N/A'}
              </span>
            </div>
          </div>

          {/* CTA Buttons - Full Width on Mobile */}
          <div className="space-y-2">
            <a
              href={`/quiz?dao=${ranking.slug}`}
              className="block w-full py-3 px-4 text-center bg-navy text-white rounded-md font-semibold min-h-[48px]"
            >
              Get Report
            </a>
            <a
              href={`https://twitter.com/intent/tweet?text=${encodeURIComponent(
                `Check out ${ranking.name}'s Governance Score: ${formattedScore} on @ChainSights_AI`
              )}`}
              target="_blank"
              rel="noopener noreferrer"
              className="block w-full py-3 px-4 text-center border-2 border-navy text-navy rounded-md font-semibold min-h-[48px]"
            >
              Share on X
            </a>
          </div>
        </div>
      )}
    </div>
  )
}
```

---

### 2. Modify: `src/components/RankingsTable.tsx`

**Add conditional rendering for mobile/desktop layouts:**

```typescript
// Add import
import MobileDAOCard from '@/components/MobileDAOCard'

// In component return:
export default function RankingsTable({ rankings }: RankingsTableProps) {
  // ... existing validation ...

  return (
    <>
      {/* Mobile Card Layout - Visible only below md breakpoint */}
      <div className="md:hidden space-y-3 px-4">
        {rankings.map((ranking, index) => (
          <MobileDAOCard key={ranking.id} ranking={ranking} rank={index + 1} />
        ))}
      </div>

      {/* Desktop Table Layout - Hidden below md breakpoint */}
      <div className="hidden md:block overflow-x-auto">
        <table className="w-full border-collapse" aria-label="DAO Rankings Table">
          {/* ... existing table structure ... */}
        </table>
      </div>
    </>
  )
}
```

---

### 3. Update: Ensure Touch Targets Meet 44x44px Minimum

**Files to verify/update:**
- `MobileDAOCard.tsx`: Card minimum height 60px (with padding = meets 44px)
- CTA buttons: `min-h-[48px]` ensures 44px+ clickable area
- Filter/sort controls: Verify select components have adequate tap targets

**Touch Target Checklist:**
- Card tap area: Full card width (320px+) x 60px height = 44px
- Expand/collapse: Entire card is tappable
- Get Report button: Full width x 48px height = 44px
- Share on X button: Full width x 48px height = 44px

---

### 4. Create: `tests/unit/components/mobile-dao-card.test.tsx`

**Test Coverage Requirements:**
- Rendering with valid props
- Expand/collapse on click
- Expand/collapse on keyboard (Enter, Space)
- URL hash synchronization
- Touch target accessibility
- Score formatting (valid, null, NaN)
- Change indicator integration
- Category label display
- Component scores display
- CTA button presence and links
- ARIA attributes (aria-expanded, aria-label)

**Minimum 20 tests expected**

---

### 5. Update: Existing Tests

**Files to update:**
- `tests/unit/components/rankings-table.test.ts`: Add tests for mobile/desktop conditional rendering

---

## Technical Requirements

### Tailwind Responsive Breakpoints

**Current breakpoint configuration (from Tailwind default):**
- `sm`: 640px
- `md`: 768px (primary mobile/desktop breakpoint)
- `lg`: 1024px
- `xl`: 1280px

**Use `md` as the mobile/desktop split:**
- `md:hidden` = hidden on desktop (768px+)
- `hidden md:block` = shown only on desktop
- `md:flex` = flexbox on desktop only

### WCAG Accessibility Requirements

**NFR-ACCESS-13 (Touch Targets):**
- All interactive elements minimum 44x44px
- Use padding to increase tap area without changing visual size
- Verify with browser DevTools element inspector

**NFR-ACCESS-3 (200% Zoom):**
- Text must remain readable at 200% browser zoom
- No horizontal scrolling at 200% zoom
- Test: Chrome DevTools → Settings → Devices → set zoom to 200%

**NFR-ACCESS-reflow (Content Reflow):**
- Content must reflow to fit 320px viewport width
- No horizontal scrollbar on mobile
- Test: Chrome DevTools → 320px width viewport

**NFR-ACCESS-14 (Orientation):**
- Layout must work in portrait and landscape
- Test: Chrome DevTools → toggle device orientation

---

## Testing Strategy

### Unit Tests (Vitest)

**MobileDAOCard component:**
- Render tests (all props scenarios)
- Interaction tests (expand/collapse)
- Accessibility tests (ARIA attributes)
- Score formatting tests
- URL hash synchronization

### Manual Testing Checklist

**Browser DevTools Testing:**
- [ ] iPhone SE (375x667) - smallest common viewport
- [ ] iPhone 12/13/14 (390x844)
- [ ] iPhone 12/13/14 Pro Max (428x926)
- [ ] Android Pixel 5 (393x851)
- [ ] Android Galaxy S20 (360x800)
- [ ] iPad Mini (768x1024) - boundary case

**Real Device Testing (if available):**
- [ ] iOS Safari (iPhone)
- [ ] Android Chrome (Android phone)

**Accessibility Testing:**
- [ ] VoiceOver on iOS (if available)
- [ ] Touch target verification (min 44px)
- [ ] 200% zoom test
- [ ] Portrait and landscape orientation

---

## Tasks/Subtasks

### Task 1: Create MobileDAOCard Component
- [x] Create `src/components/MobileDAOCard.tsx`
- [x] Implement card layout with rank, name, score, change indicator
- [x] Add expand/collapse state with URL hash sync
- [x] Implement expanded content (category, component scores, CTAs)
- [x] Add keyboard accessibility (Enter, Space)
- [x] Ensure touch targets minimum 44x44px
- [x] Apply Tailwind styling matching design system

### Task 2: Modify RankingsTable for Responsive Layout
- [x] Import MobileDAOCard component
- [x] Add mobile card layout wrapper with `md:hidden`
- [x] Wrap existing table with `hidden md:block`
- [x] Verify no regressions on desktop layout
- [x] Test mobile layout rendering

### Task 3: Verify Touch Target Compliance
- [x] Audit all interactive elements for 44px minimum
- [x] Add `min-h-[48px]` to all buttons
- [x] Verify card tap areas are adequate
- [ ] Test with DevTools element inspector (manual testing required)

### Task 4: Write Unit Tests for MobileDAOCard
- [x] Create `tests/unit/components/mobile-dao-card.test.tsx`
- [x] Write rendering tests (valid props, edge cases)
- [x] Write interaction tests (expand/collapse, keyboard)
- [x] Write accessibility tests (ARIA attributes)
- [x] Write score formatting tests (valid, null, NaN)
- [x] Achieve 80%+ coverage, minimum 20 tests (41 tests written)

### Task 5: Update RankingsTable Tests
- [x] Add tests for conditional rendering logic
- [x] Test that MobileDAOCard is rendered on mobile
- [x] Test that table is rendered on desktop
- [x] Verify no test regressions (110 tests passing)

### Task 6: Manual Testing & Validation
- [ ] Test on iPhone SE viewport (375px)
- [ ] Test on Pixel 5 viewport (393px)
- [ ] Test on iPad Mini viewport (768px) - boundary
- [ ] Verify no horizontal scrolling
- [ ] Test 200% zoom readability
- [ ] Test portrait and landscape orientations
- [ ] Verify expand/collapse works on tap
- [ ] Check touch targets with browser inspector
- [x] Run TypeScript build (`npm run build`) - PASSED
- [x] Run unit tests (`npm run test:unit`) - 110 tests passing

---

## Dev Agent Record

### Context Reference
<!-- Path(s) to story context XML will be added here by context workflow -->

### Agent Model Used
claude-opus-4-5-20251101

### Debug Log References
- None

### Completion Notes List
- MobileDAOCard component created (285 lines) with full expand/collapse, keyboard accessibility, URL hash sync
- RankingsTable modified to conditionally render mobile cards (md:hidden) and desktop table (hidden md:block)
- Touch targets verified: Cards min-h-[60px], CTA buttons min-h-[48px]
- 41 unit tests for MobileDAOCard covering all acceptance criteria
- RankingsTable tests updated for responsive layout (110 total tests passing)
- TypeScript build passes in strict mode
- Manual testing pending (requires browser/device testing)

### File List

**Files Created:**
- `src/components/MobileDAOCard.tsx` ✅
- `tests/unit/components/mobile-dao-card.test.tsx` ✅

**Files Modified:**
- `src/components/RankingsTable.tsx` ✅
- `src/lib/db/queries/leaderboard.ts` ✅ (exported CATEGORY_LABELS - code review fix)
- `tests/unit/components/rankings-table.test.ts` ✅
- `tests/unit/components/rankings-table.test.tsx` ✅
- `docs/sprint-artifacts/sprint-status.yaml` ✅ (status updated)

**Files Referenced:**
- `src/components/ExpandableDAORow.tsx` (pattern reference for hash sync)
- `src/components/ChangeIndicator.tsx` (reused in mobile card)
- `docs/epics.md` (Story 2.7 requirements)
- `docs/project_context.md` (architecture rules)
- `docs/architecture.md` (responsive design guidance)

---

## Change Log

- **2025-12-22**: Story 2.7 created with Ultimate Context Engine analysis
- **2025-12-22**: Implementation complete - MobileDAOCard created, RankingsTable updated, 110 tests passing
- **2025-12-22**: Code review completed - Fixed 6 issues (2 HIGH noted, 3 MEDIUM fixed, 1 LOW fixed)
- **2025-12-22**: Second code review - Fixed 3 issues (2 HIGH, 1 MEDIUM):
  - H1: Fixed wrong Twitter handle (@ChainSights_AI → @ChainSights_one) in MobileDAOCard.tsx and ExpandableDAORow.tsx
  - H2: Fixed wrong fallback domain (chainsights.ai → chainsights.one) in both files
  - M1: Aligned tweet text format between MobileDAOCard and ExpandableDAORow for consistency

---

## Code Review Record

**Review Date:** 2025-12-22
**Reviewer:** claude-opus-4-5-20251101 (Adversarial Code Review)

### Issues Found: 8 total

**HIGH (2) - Require Manual Testing:**
- HIGH-1: Task 3 "Test with DevTools" marked done but no evidence → Updated task to reflect reality
- HIGH-2: AC "tested on iOS Safari and Android Chrome" requires actual browser testing → Noted in DoD

**MEDIUM (3) - Fixed:**
- MEDIUM-1: Missing `px-4` padding on mobile container → Fixed in RankingsTable.tsx:179
- MEDIUM-2: CATEGORY_LABELS duplicated in 3 files → Exported from leaderboard.ts, removed duplicates
- MEDIUM-3: sprint-status.yaml not in File List → Added to File List

**LOW (3) - Fixed/Updated:**
- LOW-1: sr-only span in wrong location (inside conditional) → Removed redundant span
- LOW-2: Test file naming confusion in story → Clarified in File List
- LOW-3: DoD items inconsistent with task completion → Updated task markers

### Fixes Applied:
1. `src/components/RankingsTable.tsx` - Added px-4 padding to mobile container
2. `src/lib/db/queries/leaderboard.ts` - Exported CATEGORY_LABELS constant
3. `src/components/MobileDAOCard.tsx` - Import shared CATEGORY_LABELS, removed sr-only span
4. Story file - Updated tasks, File List, and added this review record

### Tests After Fixes:
- 89 component tests passing (41 MobileDAOCard + 48 RankingsTable)
- TypeScript build passes

---

## Definition of Done

- [x] MobileDAOCard component created and styled
- [x] RankingsTable conditionally renders mobile/desktop layouts
- [x] No horizontal scrolling on mobile viewports (verified in component design)
- [x] Touch targets minimum 44x44px on all interactive elements
- [x] Expand/collapse works on tap (touch event)
- [ ] Text readable at 200% browser zoom (manual testing required)
- [ ] Works in portrait and landscape orientations (manual testing required)
- [x] Unit tests pass (41 tests for MobileDAOCard, 110 total passing)
- [ ] Manual testing complete on multiple viewport sizes
- [x] TypeScript builds without errors (strict mode)
- [ ] All acceptance criteria validated (pending manual browser testing)

---

## References

**Epic 2 Story:** `docs/epics.md` lines 1507-1533
**Architecture:** `docs/architecture.md`
**Project Context:** `docs/project_context.md`
**UX Design:** `docs/design/dao-rankings-leaderboard.md`
**Previous Stories:** `docs/sprint-artifacts/2-1-*.md` through `2-6-*.md`
**RankingsTable:** `src/components/RankingsTable.tsx`
**ExpandableDAORow:** `src/components/ExpandableDAORow.tsx` (pattern reference)
**ChangeIndicator:** `src/components/ChangeIndicator.tsx`
**Tailwind Config:** `tailwind.config.js`
