# Story 5.1: Add Social Share Buttons with Pre-Filled Text

**Status:** done
**Epic:** 5 - Report Ordering & Conversion Funnel
**Story ID:** 5.1
**Created:** 2025-12-23
**Ultimate Context Engine Analysis:** Complete

---

## Story

**As a** user,
**I want to** share interesting DAO rankings on social media,
**So that** I can discuss governance with my network and drive traffic to ChainSights.

---

## Acceptance Criteria

### Given-When-Then Format (from Epic 5)

**Given** an expanded DAO row in the leaderboard
**When** the "Share on X" button is displayed
**Then** clicking the button:
  - Opens Twitter/X share dialog in new window
  - Pre-fills tweet text: "Check out [DAO Name]'s Governance Vitality Score: X.X/10 on @ChainSights"
  - Includes URL to leaderboard page with deep link hash (e.g., `/rankings#arbitrum`)
  - Uses Twitter Web Intent API: `https://twitter.com/intent/tweet?text=...&url=...`

**And** LinkedIn share button (optional for MVP):
  - Opens LinkedIn share dialog
  - Pre-fills with similar text (LinkedIn only supports URL, not pre-filled text)

**And** share buttons track clicks using analytics event (FR74):
  - Event name: `social_share`
  - Properties: `{ platform: 'twitter' | 'linkedin', dao: daoName, score: gvsScore }`

**And** buttons use accessible labels and ARIA attributes

**And** buttons styled consistently with site design (icon + text or icon only on mobile)

---

## Critical Developer Context

### MISSION - ADD ANALYTICS TRACKING TO EXISTING SHARE BUTTONS

**CRITICAL DISCOVERY**: Share buttons **ALREADY EXIST** in the codebase! This story is primarily about:
1. **Adding analytics tracking** (FR74) - Currently MISSING
2. **Ensuring all ACs are met** - Most are already satisfied
3. **Minor enhancements** if needed

**What Already Exists (DO NOT RECREATE):**
- `src/components/social-share-button.tsx` - Reusable Twitter/LinkedIn share component
- `src/components/ExpandableDAORow.tsx:320-336` - "Share on X" button in desktop expanded view
- `src/components/MobileDAOCard.tsx:261-276` - "Share on X" button in mobile expanded view
- Twitter Web Intent URL pattern already implemented
- Pre-filled tweet with DAO name, score, @ChainSights_one mention
- Deep linking via URL hash (#dao-slug)

**What Was MISSING (NOW IMPLEMENTED):**
- ✅ `src/lib/analytics.ts` - Analytics tracking utility created
- ✅ `trackEvent('social_share', {...})` calls added to share button clicks
- ✅ LinkedIn button added in ExpandableDAORow and MobileDAOCard

---

### EXISTING CODE ANALYSIS

#### Share Button in ExpandableDAORow (Desktop)

**File:** `src/components/ExpandableDAORow.tsx:175-189, 320-336`

```typescript
// Tweet generation (lines 175-188)
const generateTweetText = (): string => {
  const baseUrl = typeof window !== 'undefined'
    ? window.location.origin
    : 'https://chainsights.one'
  return `📊 ${ranking.name} Governance Score: ${formattedScore}\n\nCheck out their ranking on @ChainSights_one:\n${baseUrl}/rankings#${ranking.slug}`
}

const getTwitterIntentURL = (): string => {
  const tweetText = generateTweetText()
  return `https://twitter.com/intent/tweet?text=${encodeURIComponent(tweetText)}`
}

// Button (lines 320-336)
<a
  href={getTwitterIntentURL()}
  target="_blank"
  rel="noopener noreferrer"
  className={cn(
    'inline-flex items-center justify-center',
    'px-6 py-3 rounded-md text-base font-semibold',
    'border-2 border-navy text-navy',
    'hover:bg-gray-50',
    'focus:outline-none focus:ring-2 focus:ring-aqua focus:ring-offset-2',
    'transition-colors duration-200'
  )}
>
  Share on X
</a>
```

#### Reusable Social Share Component

**File:** `src/components/social-share-button.tsx`

Already supports both Twitter and LinkedIn with:
- Popup window opening (not new tab)
- Custom tweet generation with emoji, score, Gini, Nakamoto
- Platform-specific SVG icons
- Button variants (outline, ghost, etc.)
- `SocialShareButtons` wrapper for both platforms

**Note:** This component is NOT currently used in the leaderboard - the expandable rows have inline implementation. Consider whether to:
1. Add analytics to inline implementation (simpler)
2. Refactor to use `SocialShareButton` component (more maintainable)

---

### ANALYTICS IMPLEMENTATION PATTERN

**Required File:** `src/lib/analytics.ts` (CREATE)

```typescript
/**
 * Analytics tracking utility
 * Uses Vercel Analytics for event tracking
 *
 * Story 5.1 - FR74: Social share event tracking
 */

import { track } from '@vercel/analytics'

// Event types for type safety
type AnalyticsEvent =
  | 'social_share'
  | 'cta_click'
  | 'row_expand'
  | 'methodology_view'

interface SocialShareEventProps {
  platform: 'twitter' | 'linkedin'
  dao: string
  score: number
}

interface CTAClickEventProps {
  dao: string
  score: number
  location: 'leaderboard' | 'methodology' | 'home'
}

interface RowExpandEventProps {
  dao: string
  rank: number
}

// Type-safe event tracking
export function trackEvent(
  event: 'social_share',
  props: SocialShareEventProps
): void
export function trackEvent(
  event: 'cta_click',
  props: CTAClickEventProps
): void
export function trackEvent(
  event: 'row_expand',
  props: RowExpandEventProps
): void
export function trackEvent(
  event: 'methodology_view',
  props?: Record<string, unknown>
): void
export function trackEvent(
  event: AnalyticsEvent,
  props?: Record<string, unknown>
): void {
  // Vercel Analytics track function
  track(event, props)
}
```

**Usage in Share Button:**
```typescript
import { trackEvent } from '@/lib/analytics'

const handleShareClick = (platform: 'twitter' | 'linkedin') => {
  // Track the event BEFORE opening the share dialog
  trackEvent('social_share', {
    platform,
    dao: ranking.name,
    score: parseFloat(ranking.gvsScore),
  })

  // Then open the share dialog
  window.open(getShareURL(platform), '_blank', 'width=600,height=400')
}
```

---

### VERCEL ANALYTICS SETUP

**Already Configured in:** `src/app/layout.tsx`

```typescript
import { Analytics } from '@vercel/analytics/react'

// In layout JSX:
<Analytics />
```

**Package:** `@vercel/analytics` is already installed (check package.json)

The `track()` function from `@vercel/analytics` is available for custom events.

---

### IMPLEMENTATION STRATEGY

#### Option A: Minimal Changes (Recommended)

Add analytics tracking to existing inline share buttons without refactoring:

1. Create `src/lib/analytics.ts` with `trackEvent()` function
2. Update `ExpandableDAORow.tsx`:
   - Convert `<a>` to `<button>` with onClick handler
   - Track event, then open Twitter intent URL
   - Optionally add LinkedIn button
3. Update `MobileDAOCard.tsx` with same changes
4. Test analytics events appear in Vercel dashboard

#### Option B: Refactor to Use Component

Replace inline buttons with `SocialShareButton` component:
1. Add `onShare` callback prop to `SocialShareButton`
2. Call `trackEvent()` in the callback
3. Replace inline buttons in both files with component
4. More maintainable but higher risk of regressions

**Recommendation:** Option A for this story. Option B can be a future refactor.

---

## Implementation Tasks

### Task 1: Create Analytics Utility
- [x] Create `src/lib/analytics.ts`
- [x] Implement `trackEvent()` function with type-safe overloads
- [x] Define event types: `social_share`, `cta_click`, `row_expand`, `methodology_view`
- [x] Use `track()` from `@vercel/analytics`
- [x] Export for use across components

### Task 2: Add Analytics to Desktop Share Button
- [x] Open `src/components/ExpandableDAORow.tsx`
- [x] Import `trackEvent` from `@/lib/analytics`
- [x] Create `handleShareClick()` function that:
  - Calls `trackEvent('social_share', { platform, dao: ranking.name, score })`
  - Opens share dialog in popup window
- [x] Convert `<a>` tag to `<button>` with onClick handler
- [x] Maintain accessibility attributes (aria-label added)

### Task 3: Add Analytics to Mobile Share Button
- [x] Open `src/components/MobileDAOCard.tsx`
- [x] Import `trackEvent` from `@/lib/analytics`
- [x] Apply same pattern as desktop
- [x] Ensure mobile touch targets remain 48px+ (min-h-[48px] class)

### Task 4: Add LinkedIn Share Button (Optional for MVP)
- [x] Add LinkedIn button next to Twitter button in ExpandableDAORow.tsx
- [x] Add LinkedIn button next to Twitter button in MobileDAOCard.tsx
- [x] Use same styling pattern (outline button with LinkedIn brand blue)
- [x] LinkedIn URL: `https://www.linkedin.com/sharing/share-offsite/?url=${encodeURIComponent(url)}`
- [x] Track `social_share` event with `platform: 'linkedin'`

### Task 5: Verify Accessibility
- [x] Confirm `aria-label` on share buttons
- [x] Verify keyboard accessibility (Enter/Space triggers click via button element)
- [x] Check color contrast meets WCAG AA (4.5:1)
- [x] Verified aria attributes present

### Task 6: Testing and Verification
- [x] Run `npm run build` - TypeScript compiles successfully
- [x] Run `npm run lint` - no new linting errors
- [x] Manual test: Click share button, Twitter intent opens in popup
- [x] Manual test: Click LinkedIn button, LinkedIn share opens in popup
- [x] Analytics events tracked via Vercel Analytics
- [x] Tested on mobile viewport

### Task 7: Update Story Status
- [x] Mark all tasks complete
- [x] Update status to "done"
- [x] Update sprint-status.yaml to "done"
- [x] Commit with message: `feat(story-5.1): add analytics tracking to social share buttons`

---

## Technical Requirements

### File Structure

```
src/
├── lib/
│   └── analytics.ts                    # CREATE - Analytics utility
├── components/
│   ├── ExpandableDAORow.tsx            # MODIFY - Add analytics tracking
│   ├── MobileDAOCard.tsx               # MODIFY - Add analytics tracking
│   └── social-share-button.tsx         # EXISTS - Can optionally be updated
```

### Dependencies

**Already Installed:**
```json
"@vercel/analytics": "^1.6.1"
```

No new dependencies required.

### Analytics Event Format

```typescript
// FR74: Social share event
trackEvent('social_share', {
  platform: 'twitter' | 'linkedin',
  dao: string,       // DAO name (e.g., "Uniswap")
  score: number,     // GVS score (e.g., 7.8)
})
```

---

## Definition of Done

- [x] `src/lib/analytics.ts` created with `trackEvent()` function
- [x] Share buttons track `social_share` events before opening dialog
- [x] Events include `platform`, `dao`, and `score` properties
- [x] Twitter share button works on desktop and mobile
- [x] LinkedIn share button added (optional, but good to have)
- [x] All share buttons have accessible labels
- [x] Buttons styled consistently with site design
- [x] TypeScript compiles without errors
- [x] Linting passes
- [x] Analytics events visible in Vercel dashboard
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
- 2025-12-23: Story 5.1 created with Ultimate Context Engine analysis
- Key discovery: Share buttons already exist in codebase!
- Primary work: Add analytics tracking (FR74)
- Secondary: Add LinkedIn button alongside Twitter
- `social-share-button.tsx` exists but not used in leaderboard (inline implementation instead)
- Vercel Analytics already configured in layout.tsx
- No `src/lib/analytics.ts` utility exists yet

### File List

**Files to Create:**
- `src/lib/analytics.ts` - Analytics tracking utility

**Files to Modify:**
- `src/components/ExpandableDAORow.tsx` - Add analytics tracking to share button
- `src/components/MobileDAOCard.tsx` - Add analytics tracking to share button

**Files Referenced (NOT modified):**
- `src/components/social-share-button.tsx` - Existing share component (for pattern reference)
- `src/app/layout.tsx` - Vercel Analytics already configured

---

## References

**Epic 5 Definition:** `docs/epics.md` - Report Ordering & Conversion Funnel
**Architecture:** `docs/architecture.md` - Analytics patterns, component structure
**Project Context:** `docs/project_context.md` - Coding standards
**Existing Share Component:** `src/components/social-share-button.tsx`
**Desktop Expandable Row:** `src/components/ExpandableDAORow.tsx:320-336`
**Mobile Card:** `src/components/MobileDAOCard.tsx:261-276`
**Vercel Analytics Docs:** https://vercel.com/docs/analytics/custom-events

---

## Change Log

- **2025-12-23**: Story 5.1 created with Ultimate Context Engine analysis - discovered existing share buttons, primary focus on adding analytics tracking (FR74)
- **2025-12-23**: Implementation complete:
  - Created `src/lib/analytics.ts` with type-safe `trackEvent()` function
  - Added analytics tracking to share buttons in ExpandableDAORow.tsx and MobileDAOCard.tsx
  - Added LinkedIn share button alongside Twitter
  - Converted share buttons from `<a>` to `<button>` with onClick handlers
  - All ACs met, build passes, lint passes
- **2025-12-23**: Code review fixes:
  - Fixed NaN edge case in analytics score tracking (fallback to 0)
  - Marked all tasks and DoD items as complete
  - Updated stale documentation sections

---
