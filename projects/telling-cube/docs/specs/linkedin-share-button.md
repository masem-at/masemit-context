# Tech Spec: LinkedIn Share Button on Results Page

**Created:** 2025-12-17
**Author:** Maya Rivera (Marketing) + Claude (Tech)
**Status:** Implemented
**Priority:** Low (nice-to-have growth feature)

---

## Overview

Add a LinkedIn share button to the scenario results page, allowing users to share their generation experience with their professional network.

## Goal

Enable organic word-of-mouth marketing by making it easy for users to share their tellingCube experience on LinkedIn - the primary platform for our IBCS/BI target audience.

## User Story

> As a user who just generated a scenario, I want to easily share my experience on LinkedIn so I can show my network a useful tool I discovered.

---

## Design

### What Gets Shared

**Pre-populated post text:**
```
Just generated a realistic {period} {companyName} dataset with tellingCube - Finance and Sales views that actually reconcile from the same events.

Took {duration} to generate {eventCount}+ business events.

tellingcube.com
```

**Example:**
```
Just generated a realistic 12-month TechNova GmbH dataset with tellingCube - Finance and Sales views that actually reconcile from the same events.

Took 47 seconds to generate 2,400+ business events.

tellingcube.com
```

### What NOT to Share

- Actual data/numbers (privacy)
- Scenario ID or direct link to results (data is user's, not public)
- Screenshots of financials

### Placement

Location: Results page action bar, after the download buttons

```
[Download Events CSV]  [More Downloads ▾]  [Share on LinkedIn]
```

- Icon: LinkedIn logo (simple, not branded blue - keep UI clean)
- Style: Ghost/outline button to not compete with primary download CTA
- Position: Right side of action bar (secondary action)

### Interaction

1. User clicks "Share on LinkedIn" button
2. New tab opens with LinkedIn share dialog
3. Pre-populated text (user can edit before posting)
4. User posts (or cancels)

---

## Technical Implementation

### File to Modify

`app/results/[scenarioId]/page.tsx`

### Data Required (already available)

From `ScenarioMetadata`:
- `companyName` - e.g., "TechNova GmbH"
- `period` - e.g., "Jan 2024 - Dec 2024" (extract duration like "12-month")
- `generationDurationSeconds` - e.g., 47

From validation/events API (optional enhancement):
- Event count (for "2,400+ business events")

### LinkedIn Share URL Format

```typescript
const linkedInShareUrl = `https://www.linkedin.com/sharing/share-offsite/?url=${encodeURIComponent(url)}`;
```

Note: LinkedIn's share dialog doesn't support pre-filled text via URL params (unlike Twitter). Options:

**Option A: Simple URL share (Recommended)**
- Just share `https://tellingcube.com`
- User writes their own text
- Pros: Simple, authentic posts
- Cons: Less control over message

**Option B: Share with Open Graph preview**
- Share `https://tellingcube.com/shared?ref=result`
- OG meta tags provide preview card
- Pros: Nice preview card
- Cons: Still no pre-filled text

**Option C: Copy-to-clipboard + prompt**
- Button copies pre-written text to clipboard
- Toast: "Post text copied! Opening LinkedIn..."
- Opens LinkedIn new post page
- User pastes text
- Pros: Full control over suggested text
- Cons: Extra step for user

### Recommended: Option C (Copy + Open)

Best balance of user experience and message control.

```typescript
const handleShareLinkedIn = async () => {
  const shareText = `Just generated a realistic ${periodDuration} ${metadata.companyName} dataset with tellingCube - Finance and Sales views that actually reconcile from the same events.\n\ntellingcube.com`;

  await navigator.clipboard.writeText(shareText);

  toast.success("Post text copied to clipboard!");

  // Small delay so user sees the toast
  setTimeout(() => {
    window.open('https://www.linkedin.com/feed/', '_blank');
  }, 500);
};
```

### Component Addition

```tsx
// In action bar section of results page
<Button
  variant="outline"
  size="sm"
  onClick={handleShareLinkedIn}
  className="gap-2"
>
  <LinkedInIcon className="h-4 w-4" />
  Share
</Button>
```

### LinkedIn Icon

Use Lucide or add simple SVG:

```tsx
const LinkedInIcon = ({ className }: { className?: string }) => (
  <svg className={className} viewBox="0 0 24 24" fill="currentColor">
    <path d="M20.447 20.452h-3.554v-5.569c0-1.328-.027-3.037-1.852-3.037-1.853 0-2.136 1.445-2.136 2.939v5.667H9.351V9h3.414v1.561h.046c.477-.9 1.637-1.85 3.37-1.85 3.601 0 4.267 2.37 4.267 5.455v6.286zM5.337 7.433c-1.144 0-2.063-.926-2.063-2.065 0-1.138.92-2.063 2.063-2.063 1.14 0 2.064.925 2.064 2.063 0 1.139-.925 2.065-2.064 2.065zm1.782 13.019H3.555V9h3.564v11.452zM22.225 0H1.771C.792 0 0 .774 0 1.729v20.542C0 23.227.792 24 1.771 24h20.451C23.2 24 24 23.227 24 22.271V1.729C24 .774 23.2 0 22.222 0h.003z"/>
  </svg>
);
```

---

## Helper Function

```typescript
// utils/share.ts
export function formatPeriodDuration(period: string): string {
  // Input: "Jan 2024 - Dec 2024"
  // Output: "12-month"

  const match = period.match(/(\w+)\s+(\d+)\s*-\s*(\w+)\s+(\d+)/);
  if (!match) return period;

  const months = [
    'Jan', 'Feb', 'Mar', 'Apr', 'May', 'Jun',
    'Jul', 'Aug', 'Sep', 'Oct', 'Nov', 'Dec'
  ];

  const startMonth = months.indexOf(match[1]);
  const startYear = parseInt(match[2]);
  const endMonth = months.indexOf(match[3]);
  const endYear = parseInt(match[4]);

  const totalMonths = (endYear - startYear) * 12 + (endMonth - startMonth) + 1;

  return `${totalMonths}-month`;
}
```

---

## Analytics (Optional)

Track share button clicks:

```typescript
// If using Vercel Analytics or similar
track('share_linkedin_clicked', {
  scenarioType: metadata.scenarioType,
  companySize: metadata.size
});
```

---

## Acceptance Criteria

- [ ] Share button visible on results page (desktop + mobile)
- [ ] Click copies pre-formatted text to clipboard
- [ ] Toast confirms copy success
- [ ] Opens LinkedIn feed in new tab
- [ ] Button styling matches existing UI (outline/ghost style)
- [ ] No sensitive data included in share text

---

## Out of Scope (Future)

- Twitter/X share button
- Direct image/screenshot sharing
- Tracking which shares convert to signups
- "Share your results" email CTA

---

## Estimated Effort

**~1-2 hours** - Simple UI addition, no backend changes

---

## Files to Change

1. `app/results/[scenarioId]/page.tsx` - Add button + handler
2. `utils/share.ts` (new) - Helper functions
3. Optional: `components/icons/LinkedInIcon.tsx`

