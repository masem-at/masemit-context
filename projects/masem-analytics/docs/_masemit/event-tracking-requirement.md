# Product Requirement: Self-Hosted Analytics Tracking

**Project:** masemIT Analytics (analytics.masem.at)
**Related Projects:** ChainSights, tellingCube, hoki.help, masem.at
**Priority:** High
**Author:** Mario Semper
**Date:** 2026-01-27
**Revised:** 2026-01-27 (Party Mode Session with Mary, Winston, Sally, John)

---

## Problem Statement

We have traffic but zero conversions. Current analytics only tracks page views via Vercel Drain, making it impossible to identify where users drop off in the conversion funnel. We cannot answer critical questions like:

- Do users see the CTAs?
- Do they click CTAs but abandon checkout?
- Where exactly in the funnel do we lose them?

**Additional Discovery:** ChainSights already has custom event tracking via `@vercel/analytics` (`cta_click`, `row_expand`, etc.) but these events **do not flow through Vercel Drain** — they're trapped in Vercel's dashboard only.

**Cost Factor:** Vercel Analytics Pro costs $10/project/month for Speed Insights + usage fees. With 4 projects, this becomes $40+/month for features we can build ourselves.

---

## Objective

Build a **fully self-hosted analytics tracking system** that replaces Vercel Analytics entirely:

1. Track both **page views** and **custom events** via a single `tracker.js` library
2. Remove dependency on Vercel Analytics (cost savings)
3. Disable Vercel Drain (no longer needed)
4. Enable funnel analysis to identify conversion blockers

---

## Scope

### In Scope

1. **`tracker.js`** — Universal tracking library hosted on analytics.masem.at
2. **`POST /api/events`** — Backend endpoint to receive all tracking data
3. **Page view tracking** — Replace Vercel's automatic page view tracking
4. **Custom event tracking** — Capture user interactions
5. **Owner exclusion** — Filter out Mario's traffic
6. **Dashboard updates** — Display events alongside page views
7. **ChainSights migration** — Remove `@vercel/analytics`, integrate `tracker.js`
8. **Vercel cleanup** — Remove Analytics & Drain from all projects

### Out of Scope

- Third-party analytics tools (Mixpanel, Amplitude, etc.)
- User identification / authentication tracking
- A/B testing infrastructure (future enhancement)
- Real-time streaming (page refresh is sufficient)
- Web Vitals / Speed Insights (use Lighthouse on-demand if needed)
- Funnel visualization (Phase 2 — collect data first)

---

## Architecture Decision

### Chosen Approach: Script Tag Distribution

```html
<!-- Add to all masemIT projects -->
<script src="https://analytics.masem.at/tracker.js" defer></script>
```

**Why this approach:**
- Zero setup in client projects (no npm install, no imports)
- Instant updates across all projects when `tracker.js` changes
- Works with any framework (Next.js, static HTML, etc.)
- Single source of truth

**Data Flow:**
```
Browser → tracker.js → POST analytics.masem.at/api/events → Neon DB (events table)
```

### Database: Use Existing Schema

The `events` table already supports custom events:

```sql
-- Existing columns we'll use:
type TEXT,              -- 'pageView' or 'customEvent'
event_name TEXT,        -- e.g., 'cta_click'
event_data JSONB,       -- custom properties
session_id BIGINT,
device_id BIGINT,
path TEXT,
country TEXT,           -- from GeoIP (server-side)
-- etc.
```

**No schema changes required.**

---

## Functional Requirements

### FR1: Tracker Library (`tracker.js`)

Lightweight client-side library (~3KB gzipped) that:

```javascript
// Auto-tracks page views on load and navigation
// Exposes global function for custom events:
window.masemit.track('cta_click', { button: 'get_report', dao: 'gitcoin' });
```

**Requirements:**
- Auto-generate anonymous `session_id` (UUID v4, stored in sessionStorage)
- Auto-generate `device_id` (UUID v4, stored in localStorage for returning visitors)
- Auto-detect: `page_path`, `referrer`, `device_type`, `browser_name`
- Use `navigator.sendBeacon()` for reliable non-blocking delivery
- Graceful failure (never break the host app)
- Support SPA navigation (track route changes)

### FR2: Owner Traffic Exclusion

Exclude Mario's traffic from all tracking:

**Auto-detection:**
```javascript
// If user visits /admin on any project, mark as owner
if (location.pathname.startsWith('/admin')) {
  localStorage.setItem('masemit_owner', 'true');
}
```

**Manual trigger:**
```
https://chainsights.one/?owner=sempre
```

**Exclusion check:**
```javascript
if (localStorage.getItem('masemit_owner') === 'true') return; // Skip all tracking
```

### FR3: Events API Endpoint

`POST https://analytics.masem.at/api/events`

```json
{
  "type": "customEvent",
  "event_name": "cta_click",
  "event_data": { "button": "get_report", "dao": "gitcoin" },
  "path": "/rankings",
  "referrer": "https://google.com",
  "session_id": "abc-123-def",
  "device_id": "xyz-456-ghi",
  "device_type": "desktop",
  "browser_name": "Chrome",
  "timestamp": "2026-01-27T21:00:00Z",
  "project_id": "chainsights"
}
```

**Requirements:**
- CORS: Allow `chainsights.one`, `tellingcube.com`, `hoki.help`, `masem.at`
- Validate required fields
- Add `country` from request IP (server-side GeoIP)
- Rate limiting: 100 requests/minute per session_id
- Return 200 even on DB errors (log internally, never fail client)

### FR4: Dashboard Updates

Add to analytics.masem.at:

1. **Event counts card** on main dashboard
   - Total custom events (period)
   - Breakdown by event name

2. **Events in "All Events" table**
   - Show both page views AND custom events
   - Indicate type with icon or badge

3. **Per-project event breakdown**
   - On project detail page, show top events

---

## Events Already Implemented (ChainSights)

These events exist in ChainSights code and will flow to our DB once `tracker.js` is integrated:

| Event Name | Current Status |
|------------|----------------|
| `cta_click` | ✅ Tracked (43 events in Vercel) |
| `row_expand` | ✅ Tracked (15 events) |
| `onboarding_banner_shown` | ✅ Tracked |
| `onboarding_banner_dismissed` | ✅ Tracked |
| `social_share` | ✅ Implemented |
| `crypto_checkout_initiated` | ✅ Implemented |
| `methodology_view` | ✅ Implemented |

**Migration:** Replace `track()` from `@vercel/analytics` with `window.masemit.track()`.

---

## Events to Add (ChainSights)

| Event Name | Trigger | Properties |
|------------|---------|------------|
| `cta_view` | CTA enters viewport (3s+ on page) | `button_name` |
| `quiz_start` | User starts governance quiz | — |
| `quiz_complete` | User completes quiz | `score` |
| `checkout_complete` | Stripe payment successful | `product`, `price_eur` |

---

## Events to Implement (tellingCube - Secondary)

| Event Name | Trigger | Properties |
|------------|---------|------------|
| `example_view` | User views example dataset | `example_name` |
| `generate_start` | User starts data generation | `template` |
| `generate_complete` | Generation finished | `template`, `row_count` |
| `download_click` | User downloads data | `format` (csv/json) |

---

## Non-Functional Requirements

### NFR1: Performance
- `tracker.js` < 5KB gzipped
- Use `sendBeacon()` — non-blocking, survives page unload
- Async loading (`defer` attribute)
- No impact on Lighthouse scores

### NFR2: Privacy / GDPR
- No PII stored (no email, name, wallet, IP in DB)
- Session storage only (no persistent cookies)
- No cross-site tracking
- First-party analytics (same legal basis as before)
- Update Privacy Policy to mention "analytics for improving user experience"

### NFR3: Reliability
- Client: Catch all errors, fail silently
- Server: Return 200 even on DB errors
- No single point of failure

### NFR4: Data Retention
- Raw events: 90 days
- Daily aggregates: Indefinite

---

## Migration Plan

### Phase 1: Build Infrastructure (analytics.masem.at)
1. Create `POST /api/events` endpoint
2. Create `public/tracker.js` library
3. Test with manual calls

### Phase 2: Integrate ChainSights
1. Add `<script src="https://analytics.masem.at/tracker.js">`
2. Replace `@vercel/analytics` track() calls with `window.masemit.track()`
3. Remove `@vercel/analytics` package
4. Verify events flowing to DB

### Phase 3: Dashboard Updates
1. Show custom events in dashboard
2. Add event counts cards

### Phase 4: Cleanup & Rollout
1. Disable Vercel Drain
2. Remove Vercel Analytics from all projects
3. Integrate remaining projects (tellingCube, hoki.help, masem.at)

---

## Success Criteria

1. **Events flowing** from ChainSights within 1 week
2. **Dashboard shows** event counts within 2 weeks
3. **Vercel Analytics removed** from all projects within 3 weeks
4. **Cost savings:** $0/month for analytics (vs ~$50/month Vercel)
5. **Actionable insight:** Identify #1 drop-off point within 30 days

---

## Open Questions (Resolved)

| Question | Decision |
|----------|----------|
| Extend Vercel Drain or Direct API? | **Direct API** — Drain doesn't forward custom events |
| NPM package or Script tag? | **Script tag** — Zero setup, instant updates |
| Keep Vercel Analytics as backup? | **No** — Full replacement to save costs |
| Track scroll depth? | **No** — Keep it simple for now |
| Real-time display? | **No** — Page refresh is sufficient |

---

## Acceptance Criteria

- [ ] `tracker.js` hosted at `analytics.masem.at/tracker.js`
- [ ] `window.masemit.track()` function available globally
- [ ] Page views auto-tracked on load and SPA navigation
- [ ] Custom events stored in `events` table with `type = 'customEvent'`
- [ ] Owner exclusion working (`/admin` auto-detect + `?owner=` param)
- [ ] CORS configured for all masemIT domains
- [ ] Events visible in dashboard within 5 seconds
- [ ] ChainSights migrated, `@vercel/analytics` removed
- [ ] Vercel Drain disabled
- [ ] Zero performance regression (Lighthouse maintained)
- [ ] Privacy Policy updated
