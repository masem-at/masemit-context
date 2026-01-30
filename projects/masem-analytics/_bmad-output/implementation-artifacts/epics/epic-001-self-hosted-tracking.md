# Epic 001: Self-Hosted Analytics Tracking

**Status:** Ready for Development
**Priority:** High
**PRD Reference:** `docs/_masemit/event-tracking-requirement.md`
**Created:** 2026-01-27
**Owner:** Sempre

---

## Epic Summary

Replace Vercel Analytics with a fully self-hosted tracking system. Build `tracker.js` library that tracks both page views and custom events, integrate with existing analytics dashboard, and remove all Vercel Analytics dependencies.

---

## Business Value

- **Cost Savings:** Eliminate ~$50/month Vercel Analytics fees
- **Data Ownership:** Full control over analytics data
- **Conversion Insights:** Enable funnel analysis to identify drop-off points
- **Flexibility:** Custom events without third-party limitations

---

## Stories

### Story 1.1: Events API Endpoint

**Priority:** P0 (Blocker for all other stories)
**Estimate:** S (Small)
**Assignee:** Dev Agent

#### Description
Create a POST endpoint at `/api/events` to receive tracking data from client-side `tracker.js`.

#### Acceptance Criteria
- [ ] `POST /api/events` accepts JSON payload
- [ ] Validates required fields: `type`, `path`, `project_id`, `timestamp`
- [ ] Inserts into existing `events` table
- [ ] CORS headers allow: `chainsights.one`, `tellingcube.com`, `hoki.help`, `masem.at`, `localhost:*`
- [ ] Returns 200 even on DB errors (logs error internally)
- [ ] Adds `country` from request headers (Vercel geo headers or CF-IPCountry)
- [ ] Rate limiting: 100 req/min per session_id (optional, can defer)

#### Technical Notes
- Use existing `events` table schema — no migrations needed
- Map incoming fields to existing columns:
  - `type` → `type` ('pageView' or 'customEvent')
  - `event_name` → `event_name`
  - `event_data` → `event_data` (JSONB)
  - `session_id` → `session_id` (convert UUID string to BIGINT hash)
  - `device_id` → `device_id` (convert UUID string to BIGINT hash)
- Country detection: `request.headers.get('x-vercel-ip-country')` or `cf-ipcountry`

#### Files to Create/Modify
- `app/api/events/route.ts` (new)

---

### Story 1.2: Tracker Library (tracker.js)

**Priority:** P0
**Estimate:** M (Medium)
**Assignee:** Dev Agent
**Blocked By:** Story 1.1

#### Description
Create the client-side tracking library that auto-tracks page views and exposes `window.masemit.track()` for custom events.

#### Acceptance Criteria
- [ ] Hosted at `https://analytics.masem.at/tracker.js`
- [ ] Auto-generates `session_id` (UUID v4, sessionStorage)
- [ ] Auto-generates `device_id` (UUID v4, localStorage)
- [ ] Auto-tracks page view on script load
- [ ] Tracks SPA navigation (History API popstate + pushState)
- [ ] Exposes `window.masemit.track(eventName, properties)` globally
- [ ] Uses `navigator.sendBeacon()` for non-blocking requests
- [ ] Falls back to `fetch()` if sendBeacon unavailable
- [ ] Graceful failure — never throws, never blocks
- [ ] < 5KB gzipped

#### Technical Notes
- IIFE pattern to avoid global pollution
- Detect device type from `navigator.userAgent`
- Detect browser name from `navigator.userAgent`
- Project ID from `data-project` attribute on script tag OR auto-detect from hostname

```html
<script src="https://analytics.masem.at/tracker.js" data-project="chainsights" defer></script>
```

#### Files to Create/Modify
- `public/tracker.js` (new)

---

### Story 1.3: Owner Traffic Exclusion

**Priority:** P0
**Estimate:** S (Small)
**Assignee:** Dev Agent
**Part of:** Story 1.2 (can be implemented together)

#### Description
Implement logic to exclude Mario's traffic from analytics.

#### Acceptance Criteria
- [ ] If URL contains `?owner=sempre`, set `localStorage.masemit_owner = 'true'`
- [ ] If path starts with `/admin`, set `localStorage.masemit_owner = 'true'`
- [ ] If `localStorage.masemit_owner === 'true'`, skip ALL tracking
- [ ] Exclusion persists across sessions (localStorage)
- [ ] Works on all domains

#### Technical Notes
- Check at start of tracker initialization
- Check BEFORE sending any beacon

#### Files to Modify
- `public/tracker.js`

---

### Story 1.4: Dashboard - Custom Events Display

**Priority:** P1
**Estimate:** M (Medium)
**Assignee:** Dev Agent
**Blocked By:** Story 1.1

#### Description
Update analytics dashboard to display custom events alongside page views.

#### Acceptance Criteria
- [ ] Main dashboard shows "Custom Events" count card
- [ ] "All Events" table shows both page views and custom events
- [ ] Events have visual indicator (icon/badge) for type
- [ ] Event name and properties displayed for custom events
- [ ] Filter by event type (page view / custom event / all)

#### Technical Notes
- Query: `WHERE type = 'customEvent'` for event counts
- In table: Show `event_name` and truncated `event_data`

#### Files to Modify
- `app/dashboard/page.tsx`
- `app/dashboard/[project]/page.tsx`

---

### Story 1.5: ChainSights Integration

**Priority:** P1
**Estimate:** M (Medium)
**Assignee:** Dev Agent
**Blocked By:** Stories 1.1, 1.2
**Repo:** `C:\Users\mario\Sources\dev\chainsights`

#### Description
Integrate `tracker.js` into ChainSights and remove `@vercel/analytics`.

#### Acceptance Criteria
- [ ] Script tag added to root layout
- [ ] `src/lib/analytics.ts` updated to use `window.masemit.track()`
- [ ] Type definitions for `window.masemit` added
- [ ] `@vercel/analytics` package removed from dependencies
- [ ] All existing `trackEvent()` calls still work
- [ ] Page views tracked automatically (no manual code needed)
- [ ] Events visible in analytics.masem.at dashboard

#### Technical Notes
- Keep existing `trackEvent()` wrapper for type safety
- Change internal implementation from Vercel's `track()` to `window.masemit.track()`

```typescript
// Before
import { track } from '@vercel/analytics'
track(event, props)

// After
window.masemit?.track(event, props)
```

#### Files to Modify (ChainSights repo)
- `src/app/layout.tsx` — add script tag
- `src/lib/analytics.ts` — change track implementation
- `package.json` — remove @vercel/analytics
- `src/types/global.d.ts` (new) — window.masemit types

---

### Story 1.6: Vercel Drain Removal

**Priority:** P2
**Estimate:** XS (Extra Small)
**Assignee:** Sempre (Manual)
**Blocked By:** Story 1.5 verified working

#### Description
Disable Vercel Drain now that tracker.js handles all tracking.

#### Acceptance Criteria
- [ ] Vercel Drain disabled in Vercel Dashboard
- [ ] No duplicate page views in database
- [ ] Existing data preserved

#### Technical Notes
- Manual action in Vercel Dashboard
- Project Settings → Integrations → Log Drains → Delete

---

### Story 1.7: Remaining Projects Integration

**Priority:** P2
**Estimate:** S (Small)
**Assignee:** Dev Agent
**Blocked By:** Story 1.5

#### Description
Add tracker.js to remaining masemIT projects.

#### Sub-tasks
- [ ] tellingCube integration
- [ ] hoki.help integration
- [ ] masem.at integration

#### Acceptance Criteria
- [ ] Script tag added to each project's root layout
- [ ] Page views flowing to analytics dashboard
- [ ] No `@vercel/analytics` in any project

#### Files to Modify
- `tellingcube/src/app/layout.tsx`
- `hoki-help/src/app/layout.tsx`
- `masemit-site/src/app/layout.tsx`

---

### Story 1.8: Project-Level Event Breakdown

**Priority:** P2
**Estimate:** S (Small)
**Assignee:** Dev Agent
**Blocked By:** Story 1.4

#### Description
Add event breakdown section to project detail pages.

#### Acceptance Criteria
- [ ] Project page shows "Top Events" section
- [ ] Lists event names with counts
- [ ] Clickable to filter events table

#### Files to Modify
- `app/dashboard/[project]/page.tsx`

---

## Definition of Done

- [ ] All P0 stories completed
- [ ] ChainSights tracking live and verified
- [ ] At least 10 real events captured in production
- [ ] Vercel Drain disabled
- [ ] No regressions in existing dashboard functionality

---

## Dependencies

| Dependency | Type | Notes |
|------------|------|-------|
| Neon DB | External | Existing — no changes |
| Vercel Hosting | External | For analytics.masem.at |
| ChainSights repo | Internal | For Story 1.5 |

---

## Risks & Mitigations

| Risk | Impact | Mitigation |
|------|--------|------------|
| CORS issues | Events fail silently | Test with all domains before rollout |
| sendBeacon blocked by ad blockers | Some events lost | Accept ~5-10% loss, no mitigation needed |
| Session ID collision | Incorrect funnels | Use UUID v4 — collision probability negligible |

---

## Notes

- Funnel visualization deferred to Epic 002
- A/B testing deferred to future epic
- Privacy Policy update is a manual task (not a dev story)
