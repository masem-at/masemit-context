# Story 11.10: Impact Page

Status: done

## Story

**As a** visitor
**I want** to see ChainSights' total donation impact
**So that** I can see the company's values in action

## Acceptance Criteria

1. - [x] Page at `/impact` route
2. - [x] Shows "Coming Soon" if total < €500
3. - [x] Shows donation total prominently if >= €500
4. - [x] Includes description of hoki.help
5. - [x] Link to hoki.help website
6. - [x] Responsive design
7. - [x] SEO meta tags

## Tasks / Subtasks

- [x] Task 1: Create page component (AC: #1, #6)
  - [x] 1.1 Create `src/app/impact/page.tsx`
  - [x] 1.2 Use existing page layout pattern (Header, bg-hero-gradient)
  - [x] 1.3 Center content with max-w-3xl container
  - [x] 1.4 Ensure responsive design for mobile/desktop
- [x] Task 2: Fetch donation total (AC: #2, #3)
  - [x] 2.1 Fetch from `/api/donations/total` endpoint
  - [x] 2.2 Handle loading state
  - [x] 2.3 Handle error state gracefully
- [x] Task 3: Implement threshold logic (AC: #2, #3)
  - [x] 3.1 Define IMPACT_THRESHOLD = 500
  - [x] 3.2 Show "Coming Soon" view when total < threshold
  - [x] 3.3 Show "Impact" view with total when >= threshold
- [x] Task 4: Create Coming Soon view (AC: #2, #4, #5)
  - [x] 4.1 Display blue heart emoji
  - [x] 4.2 Display "We're just getting started."
  - [x] 4.3 Display "Check back soon to see our growing impact..."
  - [x] 4.4 Add hoki.help link
- [x] Task 5: Create Impact view (AC: #3, #4, #5)
  - [x] 5.1 Display blue heart emoji
  - [x] 5.2 Display large donation total (€X,XXX format)
  - [x] 5.3 Display "contributed to families facing childhood illness"
  - [x] 5.4 Display description paragraph about hoki.help
  - [x] 5.5 Display "This total updates automatically with each purchase."
  - [x] 5.6 Add hoki.help link
- [x] Task 6: Add SEO meta tags (AC: #7)
  - [x] 6.1 Set page title: "Impact | ChainSights"
  - [x] 6.2 Set meta description
  - [x] 6.3 Set canonical URL
- [x] Task 7: Write unit tests
  - [x] 7.1 Test Coming Soon view renders when total < 500
  - [x] 7.2 Test Impact view renders when total >= 500
  - [x] 7.3 Test donation amount formatting
  - [x] 7.4 Test hoki.help link attributes
  - [x] 7.5 Test loading state
  - [x] 7.6 Test error handling

## Dev Notes

### CRITICAL CONSTRAINT

> **ALL CODE CHANGES GO TO `develop` BRANCH ONLY!**

### UX Spec Reference

From `docs/ux/checkout-crypto-donations-ux-spec.md`:

**Visibility Rule:**
- Show page: When cumulative donations >= €500
- Before €500: Show "coming soon" message

**Live Version Wireframe (>= €500):**
```
┌─────────────────────────────────────────────────────────────┐
│                                                             │
│  ChainSights Impact                                         │
│                                                             │
│  ─────────────────────────────────────────────────────────  │
│                                                             │
│                         💙                                  │
│                                                             │
│                      €1,247                                 │
│                                                             │
│              contributed to families                        │
│            facing childhood illness                         │
│                                                             │
│  ─────────────────────────────────────────────────────────  │
│                                                             │
│  Every ChainSights report directs 3% of revenue to         │
│  hoki.help — a children's hospice supporting families       │
│  in Lower Austria through their most difficult moments.     │
│                                                             │
│  This total updates automatically with each purchase.       │
│                                                             │
│  ─────────────────────────────────────────────────────────  │
│                                                             │
│              [Learn more about hoki.help ↗]                 │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

**Coming Soon Wireframe (< €500):**
```
┌─────────────────────────────────────────────────────────────┐
│                                                             │
│  ChainSights Impact                                         │
│                                                             │
│  ─────────────────────────────────────────────────────────  │
│                                                             │
│                         💙                                  │
│                                                             │
│              We're just getting started.                    │
│                                                             │
│         Check back soon to see our growing impact           │
│           on families facing childhood illness.             │
│                                                             │
│  ─────────────────────────────────────────────────────────  │
│                                                             │
│              [Learn more about hoki.help ↗]                 │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

**Copy:**
- Page Title: "ChainSights Impact"
- Main Number: "€[total]" (large, prominent)
- Subheadline: "contributed to families facing childhood illness"
- Description: "Every ChainSights report directs 3% of revenue to hoki.help — a children's hospice supporting families in Lower Austria through their most difficult moments."
- Update Note: "This total updates automatically with each purchase."
- CTA: "Learn more about hoki.help ↗" (links to https://hoki.help)
- Coming Soon: "We're just getting started." + "Check back soon to see our growing impact on families facing childhood illness."

### API Dependency (Story 11-9)

The donations total API is already implemented:

```typescript
// GET /api/donations/total?project=chainsights
// Response:
{
  total: 1247.53,
  count: 127,
  currency: "EUR",
  lastUpdated: "2026-01-21T12:00:00Z"
}
```

### Implementation Pattern

Follow existing page patterns (see `src/app/privacy/page.tsx`):

```typescript
// src/app/impact/page.tsx
import { Metadata } from 'next'
import { Header } from "@/components/header"

// SEO meta tags
export const metadata: Metadata = {
  title: 'Impact | ChainSights',
  description: 'See how ChainSights customers are helping families facing childhood illness through our partnership with hoki.help.',
  alternates: {
    canonical: 'https://chainsights.one/impact',
  },
}

// Threshold constant
const IMPACT_THRESHOLD = 500

// Client component for dynamic data fetching
function ImpactContent() {
  // Fetch donation total
  // Render Coming Soon or Impact view based on threshold
}

export default function ImpactPage() {
  return (
    <>
      <Header />
      <main className="min-h-screen bg-hero-gradient pt-24 pb-16 px-6">
        <div className="max-w-3xl mx-auto text-center">
          <h1 className="font-montserrat text-4xl font-bold text-white mb-8">
            ChainSights Impact
          </h1>
          <ImpactContent />
        </div>
      </main>
    </>
  )
}
```

### Number Formatting

Format donation total with thousands separator:
```typescript
// €1,247 format
const formattedTotal = new Intl.NumberFormat('de-DE', {
  style: 'currency',
  currency: 'EUR',
  minimumFractionDigits: 0,
  maximumFractionDigits: 0,
}).format(total)
```

### Styling Guidelines

Follow existing patterns:
- Background: `bg-hero-gradient`
- Container: `max-w-3xl mx-auto`
- Heading: `font-montserrat text-4xl font-bold text-white`
- Subheading: `text-xl text-gray-300`
- Body text: `text-gray-400`
- Blue heart: `<span className="text-blue-400">💙</span>` or `text-6xl`
- Link: `text-aqua hover:underline`
- Large number: `text-6xl font-bold text-white` or larger

### Responsive Considerations

From UX spec:
- Mobile: Number scales down but remains prominent
- Desktop: Page centered with max-width container

### Previous Story Learnings (Story 11-9)

1. **Decimal formatting:** Use `Number(value.toFixed(2))` for precise decimals
2. **Cache-Control:** API already has 5-minute caching, page can leverage this
3. **Error handling:** Handle API errors gracefully, show fallback state
4. **Test edge cases:** Test zero, threshold boundary, and error scenarios

### Files to Create

- `src/app/impact/page.tsx` - Impact page component
- `tests/unit/app/impact/page.test.tsx` - Unit tests

### Files to Modify

None - this is a new isolated page.

### Dependencies

- **Story 11-9:** Donations total API endpoint - DONE (provides `/api/donations/total`)

### References

- [UX Spec: Impact Page](../ux/checkout-crypto-donations-ux-spec.md#4-impact-page-impact)
- [Architecture Doc](../architecture/crypto-payments-donations-architecture.md)
- [Epic: Story 10](./epic-crypto-payments-donations.md#story-10-impact-page)
- [Privacy Page Pattern](../../src/app/privacy/page.tsx)

## Dev Agent Record

### Context Reference

Story created by create-story workflow with context from:
- Epic file (docs/sprint-artifacts/epic-crypto-payments-donations.md)
- UX spec (docs/ux/checkout-crypto-donations-ux-spec.md)
- Previous story (11-9) completion notes
- Existing page patterns (src/app/privacy/page.tsx)
- Donations total API (src/app/api/donations/total/route.ts)

### Agent Model Used

Claude Opus 4.5

### Debug Log References

### Completion Notes List

1. **Component Separation**: Extracted `ImpactContent` to `src/components/ImpactContent.tsx` because Next.js page files cannot export non-default named exports (caused build error)
2. **SEO Metadata**: Added via `src/app/impact/layout.tsx` since page component is client-side
3. **Threshold Logic**: IMPACT_THRESHOLD = 500, shows Coming Soon view below, Impact view at or above
4. **Number Formatting**: Using `Intl.NumberFormat('de-DE')` for €1.247 format with thousands separator
5. **Error Handling**: Fallback to Coming Soon view on fetch errors for graceful degradation
6. **Loading State**: Blue heart with "Loading..." text while fetching
7. **Unit Tests**: 14 tests covering Coming Soon view, Impact view, links, loading, and error states

### File List

- `src/app/impact/page.tsx` (new) - Impact page component
- `src/app/impact/layout.tsx` (new) - SEO metadata
- `src/components/ImpactContent.tsx` (new) - Impact content client component
- `tests/unit/app/impact/page.test.tsx` (new) - 14 unit tests

## Senior Developer Review

**Review Date:** 2026-01-21
**Reviewer:** Claude Opus 4.5 (Adversarial Code Review)

### Issues Found & Fixed

| Severity | Issue | Resolution |
|----------|-------|------------|
| MEDIUM | Unused `error` state in ImpactContent.tsx - declared but never read | Removed unused state variable |

### Issues Noted (Low Priority)

| Severity | Issue | Notes |
|----------|-------|-------|
| LOW | No explicit return type on `fetchDonationTotal` | TypeScript infers correctly; optional enhancement |
| LOW | Could use SWR or React Query for data fetching | Current implementation is simple and appropriate for this use case |

### Verification

- All 14 unit tests passing
- Build verified successful
- No type errors

**Recommendation:** APPROVED - Story meets all acceptance criteria.

## Change Log

- 2026-01-21: Story created by Bob (Scrum Master) via create-story workflow
- 2026-01-21: Implementation completed by Dev Agent (Claude Opus 4.5) - all 14 tests passing, build verified
- 2026-01-21: Code review completed - 1 MEDIUM issue fixed (unused state), story approved and marked done
