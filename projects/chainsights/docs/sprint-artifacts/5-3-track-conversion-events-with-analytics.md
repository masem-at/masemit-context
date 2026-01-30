# Story 5.3: Track Conversion Events with Analytics

**Status:** done
**Epic:** 5 - Report Ordering & Conversion Funnel
**Story ID:** 5.3
**Created:** 2025-12-23
**Ultimate Context Engine Analysis:** Complete

---

## Story

**As a** product manager (Mario),
**I want to** track CTA clicks and engagement events,
**So that** I can measure conversion funnel effectiveness.

---

## Acceptance Criteria

### Given-When-Then Format (from Epic 5)

**AC1: CTA Click Tracking (FR73)**
```
Given Vercel Analytics is configured
When a user clicks any "Get Report" button
Then a `cta_click` event is tracked with:
  - dao: string (DAO name or 'direct-pricing')
  - score: number (GVS score or 0 if N/A)
  - location: 'leaderboard' | 'hero' | 'pricing' | 'quiz' | 'modal'
```

**AC2: Row Expand Tracking**
```
Given a user is viewing the rankings leaderboard
When a user expands a DAO row (desktop or mobile)
Then a `row_expand` event is tracked with:
  - dao: string (DAO name)
  - rank: number (current rank position)
```

**AC3: Methodology View Tracking**
```
Given the methodology page loads
When the page is fully rendered
Then a `methodology_view` event is tracked
```

**AC4: Event Visibility**
```
Given events are tracked
When checking Vercel Analytics dashboard
Then all events are visible within 24 hours
And event properties match expected schema
And no PII is included in events
```

**AC5: Existing Social Share Tracking (FR74)**
```
Given Story 5.1 is complete
When social share buttons are clicked
Then `social_share` events continue to work (no regression)
```

---

## Critical Developer Context

### MISSION - ADD MISSING CTA TRACKING ACROSS ALL CONVERSION POINTS

**CRITICAL DISCOVERY**: Analytics utility (`src/lib/analytics.ts`) EXISTS and works! Story 5.1 proved the pattern. The gap is that **CTA clicks and row expands are NOT tracked** despite event types being defined.

**What Already Exists (DO NOT RECREATE):**
- `src/lib/analytics.ts` - Type-safe `trackEvent()` function with all event types defined
- `src/components/ExpandableDAORow.tsx` - Social share tracking works (FR74 complete)
- `src/components/MobileDAOCard.tsx` - Social share tracking works (FR74 complete)
- `src/app/layout.tsx:194` - Vercel Analytics `<Analytics />` component configured
- `@vercel/analytics` package installed

**What Is MISSING (PRIMARY TASKS):**

| Location | Component | Event | Status |
|----------|-----------|-------|--------|
| Leaderboard "Get Report" | `ExpandableDAORow.tsx:336-347` | `cta_click` | **MISSING** |
| Mobile "Get Report" | `MobileDAOCard.tsx:279-291` | `cta_click` | **MISSING** |
| Hero "Get Report" | `hero.tsx:51` | `cta_click` | **MISSING** |
| Pricing buttons | `pricing.tsx:104,113,170,179` | `cta_click` | **MISSING** |
| Report Modal tier select | `report-selection-modal.tsx:280-285` | `cta_click` | **MISSING** |
| Quiz results CTA | `quiz/page.tsx:265,271` | `cta_click` | **MISSING** |
| Row expand toggle | `ExpandableDAORow.tsx:132-143` | `row_expand` | **MISSING** |
| Mobile card expand | `MobileDAOCard.tsx:112-122` | `row_expand` | **MISSING** |
| Methodology page | `methodology/page.tsx` | `methodology_view` | **MISSING** |

---

### EXISTING ANALYTICS PATTERNS (FROM STORY 5.1)

#### Analytics Utility Pattern
**File:** `src/lib/analytics.ts`

```typescript
import { track } from '@vercel/analytics'

type AnalyticsEvent = 'social_share' | 'cta_click' | 'row_expand' | 'methodology_view'

interface CTAClickEventProps {
  dao: string
  score: number
  location: 'leaderboard' | 'methodology' | 'home'
}

interface RowExpandEventProps {
  dao: string
  rank: number
}

export function trackEvent(event: 'cta_click', props: CTAClickEventProps): void
export function trackEvent(event: 'row_expand', props: RowExpandEventProps): void
export function trackEvent(event: 'methodology_view', props?: Record<string, unknown>): void
// ... implementation calls track(event, props)
```

#### Click Handler Pattern (PROVEN IN STORY 5.1)
```typescript
// Track BEFORE navigation/action
const handleCTAClick = () => {
  trackEvent('cta_click', {
    dao: ranking.name,
    score: isNaN(numericScore) ? 0 : numericScore,
    location: 'leaderboard',
  })
  // Then navigate or perform action
}
```

---

## Implementation Tasks

### Task 1: Update Analytics Utility Types
- [x] Open `src/lib/analytics.ts`
- [x] Extend `CTAClickEventProps.location` type to include all locations:
  ```typescript
  location: 'leaderboard' | 'hero' | 'pricing' | 'quiz' | 'modal' | 'methodology' | 'home'
  ```
- [x] Verify existing types are correct

### Task 2: Add CTA Tracking to Leaderboard (Desktop)
- [x] Open `src/components/ExpandableDAORow.tsx`
- [x] Find "Get Report" link at line ~336
- [x] Convert `<a>` to click handler with tracking:
  ```typescript
  const handleGetReportClick = () => {
    trackEvent('cta_click', {
      dao: ranking.name,
      score: isNaN(numericScore) ? 0 : numericScore,
      location: 'leaderboard',
    })
    window.location.href = `/quiz?dao=${ranking.slug}`
  }
  ```
- [x] Replace `<a href=...>` with `<button onClick={handleGetReportClick}>`
- [x] Maintain styling and accessibility

### Task 3: Add CTA Tracking to Leaderboard (Mobile)
- [x] Open `src/components/MobileDAOCard.tsx`
- [x] Find "Get Report" link at line ~279
- [x] Apply same pattern as Task 2
- [x] Ensure touch target remains 48px+

### Task 4: Add Row Expand Tracking (Desktop)
- [x] Open `src/components/ExpandableDAORow.tsx`
- [x] Find `toggleExpanded()` function at line ~132
- [x] Add tracking when expanding (NOT when collapsing):
  ```typescript
  const toggleExpanded = () => {
    if (!isExpanded) {
      // Track expand event
      trackEvent('row_expand', {
        dao: ranking.name,
        rank: rank,
      })
    }
    // ... existing toggle logic
  }
  ```

### Task 5: Add Row Expand Tracking (Mobile)
- [x] Open `src/components/MobileDAOCard.tsx`
- [x] Find `toggleExpanded()` function
- [x] Apply same pattern as Task 4

### Task 6: Add CTA Tracking to Hero Section
- [x] Open `src/components/hero.tsx`
- [x] Find "Get Report" button at line ~51
- [x] Add tracking before modal opens:
  ```typescript
  const handleHeroGetReport = () => {
    trackEvent('cta_click', {
      dao: 'hero-cta',  // No specific DAO context
      score: 0,
      location: 'hero',
    })
    setModalOpen(true)
  }
  ```

### Task 7: Add CTA Tracking to Pricing Component
- [x] Open `src/components/pricing.tsx`
- [x] Find `handleCheckout()` function at line ~29
- [x] Add tracking before checkout:
  ```typescript
  const handleCheckout = async (tier: TierId) => {
    trackEvent('cta_click', {
      dao: 'pricing-page',
      score: 0,
      location: 'pricing',
    })
    // ... existing checkout logic
  }
  ```
- [x] Similarly update `handleCryptoCheckout()` if exists

### Task 8: Add CTA Tracking to Report Selection Modal
- [x] Open `src/components/report-selection-modal.tsx`
- [x] Find `handleTierSelect()` function
- [x] Add tracking when tier selected:
  ```typescript
  const handleTierSelect = (tierId: string) => {
    trackEvent('cta_click', {
      dao: selectedDaoName || 'modal-selection',
      score: 0,  // Score may not be available in modal context
      location: 'modal',
    })
    // ... existing tier selection logic
  }
  ```

### Task 9: Add Methodology View Tracking
- [x] Open `src/app/rankings/methodology/page.tsx`
- [x] Option A: Add client-side tracking component
- [x] Option B: Create wrapper with useEffect:
  ```typescript
  'use client'
  import { useEffect } from 'react'
  import { trackEvent } from '@/lib/analytics'

  export default function MethodologyTracker() {
    useEffect(() => {
      trackEvent('methodology_view')
    }, [])
    return null
  }
  ```
- [x] Import and render `<MethodologyTracker />` in page

### Task 10: Testing and Verification
- [x] Run `npm run build` - no TypeScript errors
- [x] Run `npm run lint` - no linting errors
- [x] Manual test: Click "Get Report" → verify event in console (dev)
- [x] Manual test: Expand DAO row → verify event in console (dev)
- [x] Manual test: Visit methodology page → verify event
- [x] Verify no regression on social share tracking

### Task 11: Update Story Status
- [x] Mark all tasks complete
- [x] Update status to "review"
- [x] Update sprint-status.yaml
- [ ] Commit with message: `feat(story-5.3): add conversion event tracking for CTAs and engagement`

---

## Technical Requirements

### File Structure

```
src/
├── lib/
│   └── analytics.ts                    # MODIFY - Extend location types
├── components/
│   ├── ExpandableDAORow.tsx            # MODIFY - Add cta_click, row_expand
│   ├── MobileDAOCard.tsx               # MODIFY - Add cta_click, row_expand
│   ├── hero.tsx                        # MODIFY - Add cta_click
│   ├── pricing.tsx                     # MODIFY - Add cta_click
│   └── report-selection-modal.tsx      # MODIFY - Add cta_click
└── app/
    └── rankings/
        └── methodology/
            └── page.tsx                # MODIFY - Add methodology_view
```

### Event Schema Reference

```typescript
// CTA Click Event (FR73)
trackEvent('cta_click', {
  dao: string,       // DAO name or context identifier
  score: number,     // GVS score or 0
  location: 'leaderboard' | 'hero' | 'pricing' | 'quiz' | 'modal',
})

// Row Expand Event
trackEvent('row_expand', {
  dao: string,       // DAO name
  rank: number,      // Current rank position (1-indexed)
})

// Methodology View Event
trackEvent('methodology_view')

// Social Share Event (FR74 - Already implemented)
trackEvent('social_share', {
  platform: 'twitter' | 'linkedin',
  dao: string,
  score: number,
})
```

### Dependencies

**Already Installed:**
```json
"@vercel/analytics": "^1.6.1"
```

No new dependencies required.

---

## Definition of Done

- [x] `cta_click` events tracked on all "Get Report" buttons (6+ locations)
- [x] `row_expand` events tracked when expanding DAO rows (desktop + mobile)
- [x] `methodology_view` event tracked on methodology page load
- [x] All events include correct properties per schema
- [x] No regression on `social_share` tracking (Story 5.1)
- [x] No PII in event payloads
- [x] TypeScript compiles without errors
- [x] Linting passes
- [ ] Events visible in Vercel Analytics dashboard (requires deployment)
- [x] Build succeeds (`npm run build`)

---

## Dev Agent Record

### Context Reference
<!-- Ultimate Context Engine analysis completed -->

### Agent Model Used
Claude Opus 4.5

### Debug Log References
- None

### Completion Notes List
- 2025-12-23: Story 5.3 created with Ultimate Context Engine analysis
- Key discovery: Analytics utility EXISTS but CTA tracking NOT implemented
- Pattern established in Story 5.1: track() before action, handle NaN scores
- 9 locations identified needing `cta_click` tracking
- 2 locations need `row_expand` tracking
- 1 location needs `methodology_view` tracking
- FR74 (social_share) already complete from Story 5.1
- 2025-12-23: Implementation complete. All CTA locations tracked, row_expand on desktop/mobile, methodology_view via client component wrapper. Build passes, lint passes.

### File List

**Files Modified:**
- `src/lib/analytics.ts` - Extended location type union to include all CTA locations
- `src/components/ExpandableDAORow.tsx` - Added cta_click and row_expand tracking
- `src/components/MobileDAOCard.tsx` - Added cta_click and row_expand tracking
- `src/components/hero.tsx` - Added cta_click tracking, removed unused scrollToPricing
- `src/components/pricing.tsx` - Added cta_click tracking to handleCheckout and handleCryptoCheckout
- `src/components/report-selection-modal.tsx` - Added cta_click tracking, removed debug console.logs
- `src/components/methodology-tracker.tsx` - NEW FILE: Client component for tracking methodology page views
- `src/app/rankings/methodology/page.tsx` - Added MethodologyTracker component
- `src/app/quiz/page.tsx` - Added cta_click tracking to results CTAs (code review fix)

**Files Referenced (NOT modified):**
- `src/app/layout.tsx` - Vercel Analytics already configured
- Story 5.1 file - Pattern reference

---

## References

**Epic 5 Definition:** `docs/epics.md` - Report Ordering & Conversion Funnel
**Architecture:** `docs/architecture.md` - Analytics patterns (FR73, FR74, FR87-93)
**Project Context:** `docs/project_context.md` - Coding standards
**Previous Story:** Story 5.1 - Established analytics tracking pattern
**Vercel Analytics Docs:** https://vercel.com/docs/analytics/custom-events

---

## Change Log

- **2025-12-23**: Story 5.3 created with Ultimate Context Engine analysis
- Analysis identified 9+ CTA locations missing tracking despite analytics.ts being ready
- **2025-12-23**: Implementation complete - 8 files modified, all CTA tracking added
- **2025-12-23**: Code review completed - 6 issues fixed:
  - H1: Added missing quiz page CTA tracking
  - H2: Removed unused scrollToPricing dead code
  - M1: Removed 4 debug console.log statements
  - M3: Standardized JSDoc comment style
  - L1: Added return type to MethodologyTracker
  - Build verified passing

---
