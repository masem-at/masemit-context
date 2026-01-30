# Story 1.2: Tracker Library (tracker.js)

**Epic:** 001 - Self-Hosted Analytics Tracking
**Status:** Ready for Development
**Priority:** P0
**Estimate:** M (Medium)
**Assignee:** Amelia (Dev Agent)
**Includes:** Story 1.3 (Owner Traffic Exclusion)

---

## User Story

**As a** website owner
**I want to** include a single script tag on my site
**So that** page views and custom events are automatically tracked without any additional setup

---

## Description

Create a lightweight client-side tracking library (`tracker.js`) that:
1. Auto-tracks page views on load and SPA navigation
2. Exposes `window.masemit.track()` for custom events
3. Sends data to `POST /api/events` endpoint (Story 1.1)
4. Excludes owner traffic (Story 1.3)

---

## Acceptance Criteria

### Core Tracking (Story 1.2)
- [ ] **AC1:** Hosted at `https://analytics.masem.at/tracker.js` (via `public/tracker.js`)
- [ ] **AC2:** Auto-generates `session_id` (UUID v4, stored in sessionStorage)
- [ ] **AC3:** Auto-generates `device_id` (UUID v4, stored in localStorage)
- [ ] **AC4:** Auto-tracks page view on script load
- [ ] **AC5:** Tracks SPA navigation (History API: popstate + pushState/replaceState)
- [ ] **AC6:** Exposes `window.masemit.track(eventName, properties)` globally
- [ ] **AC7:** Uses `navigator.sendBeacon()` for non-blocking requests
- [ ] **AC8:** Falls back to `fetch()` if sendBeacon unavailable
- [ ] **AC9:** Graceful failure — never throws, never blocks host app
- [ ] **AC10:** < 5KB gzipped

### Owner Exclusion (Story 1.3)
- [ ] **AC11:** If URL contains `?owner=sempre`, set `localStorage.masemit_owner = 'true'`
- [ ] **AC12:** If path starts with `/admin`, set `localStorage.masemit_owner = 'true'`
- [ ] **AC13:** If `localStorage.masemit_owner === 'true'`, skip ALL tracking
- [ ] **AC14:** Exclusion persists across sessions (localStorage)

---

## Technical Specification

### Integration

```html
<!-- Add to any masemIT project -->
<script src="https://analytics.masem.at/tracker.js" data-project="chainsights" defer></script>
```

### Global API

```javascript
// Track custom event
window.masemit.track('cta_click', { button: 'get_report', dao: 'gitcoin' });

// Check if tracking is active (for debugging)
window.masemit.isActive(); // returns boolean
```

### Event Payload (sent to /api/events)

```json
{
  "type": "pageView",
  "event_name": null,
  "event_data": null,
  "path": "/rankings",
  "referrer": "https://google.com",
  "session_id": "550e8400-e29b-41d4-a716-446655440000",
  "device_id": "6ba7b810-9dad-11d1-80b4-00c04fd430c8",
  "device_type": "desktop",
  "browser_name": "Chrome",
  "browser_version": "120.0",
  "os_name": "Windows",
  "os_version": "10",
  "timestamp": "2026-01-27T21:00:00.000Z",
  "project_id": "chainsights",
  "origin": "https://chainsights.one"
}
```

### Architecture

```javascript
(function() {
  'use strict';

  // 1. Owner exclusion check (FIRST - before any tracking)
  // 2. Generate/retrieve session_id and device_id
  // 3. Detect device info (browser, OS, device type)
  // 4. Get project_id from data-project attribute or hostname
  // 5. Define send() function (sendBeacon with fetch fallback)
  // 6. Define trackPageView() function
  // 7. Define track() function for custom events
  // 8. Hook into History API for SPA navigation
  // 9. Track initial page view
  // 10. Expose window.masemit
})();
```

### UUID v4 Generation

```javascript
function uuid() {
  return 'xxxxxxxx-xxxx-4xxx-yxxx-xxxxxxxxxxxx'.replace(/[xy]/g, function(c) {
    var r = Math.random() * 16 | 0;
    var v = c === 'x' ? r : (r & 0x3 | 0x8);
    return v.toString(16);
  });
}
```

### Device Detection

```javascript
function getDeviceType() {
  var ua = navigator.userAgent;
  if (/Mobi|Android/i.test(ua)) return 'mobile';
  if (/Tablet|iPad/i.test(ua)) return 'tablet';
  return 'desktop';
}

function getBrowserInfo() {
  var ua = navigator.userAgent;
  // Parse Chrome, Firefox, Safari, Edge, etc.
  // Return { name: 'Chrome', version: '120.0' }
}

function getOSInfo() {
  var ua = navigator.userAgent;
  // Parse Windows, macOS, Linux, iOS, Android
  // Return { name: 'Windows', version: '10' }
}
```

### SPA Navigation Tracking

```javascript
// Intercept pushState and replaceState
var origPushState = history.pushState;
history.pushState = function() {
  origPushState.apply(this, arguments);
  trackPageView();
};

var origReplaceState = history.replaceState;
history.replaceState = function() {
  origReplaceState.apply(this, arguments);
  trackPageView();
};

// Listen for popstate (back/forward)
window.addEventListener('popstate', trackPageView);
```

### Owner Exclusion Logic

```javascript
function checkOwnerExclusion() {
  // Check URL param
  if (location.search.includes('owner=sempre')) {
    localStorage.setItem('masemit_owner', 'true');
  }
  // Check admin path
  if (location.pathname.startsWith('/admin')) {
    localStorage.setItem('masemit_owner', 'true');
  }
  // Return exclusion status
  return localStorage.getItem('masemit_owner') === 'true';
}
```

---

## Tasks

- [ ] **Task 1:** Create `public/tracker.js` with IIFE structure
- [ ] **Task 2:** Implement UUID generation
- [ ] **Task 3:** Implement session_id (sessionStorage) and device_id (localStorage)
- [ ] **Task 4:** Implement device/browser/OS detection
- [ ] **Task 5:** Implement sendBeacon with fetch fallback
- [ ] **Task 6:** Implement page view tracking
- [ ] **Task 7:** Implement custom event tracking (`window.masemit.track`)
- [ ] **Task 8:** Implement SPA navigation hooks
- [ ] **Task 9:** Implement owner exclusion (Story 1.3)
- [ ] **Task 10:** Test in browser console
- [ ] **Task 11:** Verify gzipped size < 5KB

---

## Test Scenarios

1. **Script loads** → Page view sent to /api/events
2. **SPA navigation** → New page view on route change
3. **Custom event** → `masemit.track('test', {foo: 'bar'})` sends event
4. **Owner param** → `?owner=sempre` sets localStorage, no tracking
5. **Admin path** → `/admin` sets localStorage, no tracking
6. **Owner returns** → Existing localStorage flag skips tracking
7. **sendBeacon blocked** → Falls back to fetch
8. **Script error** → Never throws, host app unaffected

---

## Files to Create/Modify

| File | Action |
|------|--------|
| `public/tracker.js` | Create |

---

## Dependencies

- Story 1.1 (Events API Endpoint) ✅ Complete

---

## Notes

- Use vanilla JavaScript (no dependencies)
- ES5 compatible for maximum browser support
- Minification can be done later if needed
- No source maps needed for production

---

## Definition of Done

- [ ] Script loads without errors
- [ ] Page views tracked automatically
- [ ] Custom events work via `window.masemit.track()`
- [ ] SPA navigation tracked
- [ ] Owner exclusion working
- [ ] Size < 5KB gzipped
- [ ] Code reviewed
- [ ] Committed and pushed
