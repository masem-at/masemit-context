# Story 1.5: ChainSights Integration

**Epic:** 001 - Self-Hosted Analytics Tracking
**Status:** Ready for Development
**Priority:** P1
**Estimate:** M (Medium)
**Assignee:** Amelia (Dev Agent)
**Repo:** `C:\Users\mario\Sources\dev\chainsights`

---

## User Story

**As a** ChainSights project owner
**I want to** use our self-hosted tracker.js instead of Vercel Analytics
**So that** all analytics data flows to our own dashboard and we save on Vercel costs

---

## Description

Integrate `tracker.js` into ChainSights and remove the `@vercel/analytics` dependency. This includes:
1. Adding the tracker.js script tag to the root layout
2. Updating the analytics wrapper to use `window.masemit.track()`
3. Adding TypeScript declarations for the global masemit object
4. Removing the @vercel/analytics package

---

## Acceptance Criteria

- [ ] **AC1:** Script tag added to root layout (`<script src="https://analytics.masem.at/tracker.js" data-project="chainsights" defer />`)
- [ ] **AC2:** `src/lib/analytics.ts` updated to use `window.masemit.track()`
- [ ] **AC3:** Type definitions for `window.masemit` added
- [ ] **AC4:** `@vercel/analytics` package removed from dependencies
- [ ] **AC5:** All existing `trackEvent()` calls still work
- [ ] **AC6:** Page views tracked automatically (no manual code needed)
- [ ] **AC7:** Build passes with no TypeScript errors

---

## Technical Specification

### Changes Required

**1. `src/app/layout.tsx`:**
- Remove: `import { Analytics } from '@vercel/analytics/next'`
- Remove: `<Analytics />` component
- Add: `<script src="https://analytics.masem.at/tracker.js" data-project="chainsights" defer />`

**2. `src/lib/analytics.ts`:**
- Remove: `import { track } from '@vercel/analytics'`
- Change: `track(event, props)` → `window.masemit?.track(event, props)`

**3. `src/types/global.d.ts` (new):**
```typescript
interface MasemitTracker {
  track: (eventName: string, properties?: Record<string, unknown>) => void;
  isActive: () => boolean;
}

declare global {
  interface Window {
    masemit?: MasemitTracker;
  }
}

export {};
```

**4. `package.json`:**
- Remove: `"@vercel/analytics": "^1.6.1"`

---

## Tasks

- [ ] **Task 1:** Create type definitions file
- [ ] **Task 2:** Update analytics.ts to use window.masemit
- [ ] **Task 3:** Update layout.tsx (remove Analytics, add script tag)
- [ ] **Task 4:** Remove @vercel/analytics from package.json
- [ ] **Task 5:** Run npm install to update lockfile
- [ ] **Task 6:** Build and verify no errors

---

## Files to Modify

| File | Action | Repo |
|------|--------|------|
| `src/types/global.d.ts` | Create | chainsights |
| `src/lib/analytics.ts` | Modify | chainsights |
| `src/app/layout.tsx` | Modify | chainsights |
| `package.json` | Modify | chainsights |

---

## Dependencies

- Story 1.1 (Events API Endpoint) ✅ Complete
- Story 1.2 (Tracker Library) ✅ Complete
- tracker.js deployed at analytics.masem.at

---

## Definition of Done

- [ ] All trackEvent() calls work
- [ ] Page views tracked automatically
- [ ] @vercel/analytics removed
- [ ] Build passes
- [ ] Code reviewed
- [ ] Committed and pushed to chainsights repo
