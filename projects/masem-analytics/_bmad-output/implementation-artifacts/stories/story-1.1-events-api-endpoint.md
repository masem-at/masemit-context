# Story 1.1: Events API Endpoint

**Epic:** 001 - Self-Hosted Analytics Tracking
**Status:** Ready for Development
**Priority:** P0 (Blocker)
**Estimate:** S (Small)
**Assignee:** Amelia (Dev Agent)

---

## User Story

**As a** tracking library (tracker.js)
**I want to** send events to a dedicated API endpoint
**So that** page views and custom events are stored in the analytics database

---

## Description

Create a POST endpoint at `/api/events` in the analytics project that receives tracking data from the client-side `tracker.js` library. This endpoint will handle both page views and custom events, storing them in the existing `events` table.

---

## Acceptance Criteria

- [ ] **AC1:** `POST /api/events` endpoint exists and accepts JSON payload
- [ ] **AC2:** Validates required fields: `type`, `path`, `project_id`, `timestamp`
- [ ] **AC3:** Inserts data into existing `events` table (no schema changes)
- [ ] **AC4:** CORS headers configured for:
  - `chainsights.one`
  - `tellingcube.com`
  - `hoki.help`
  - `masem.at`
  - `localhost:*` (for development)
- [ ] **AC5:** Returns `200 OK` even on DB errors (logs error internally, never fails client)
- [ ] **AC6:** Extracts `country` from request headers (`x-vercel-ip-country` or `cf-ipcountry`)
- [ ] **AC7:** Converts string UUIDs (`session_id`, `device_id`) to BIGINT hash for DB storage

---

## Technical Specification

### Endpoint

```
POST https://analytics.masem.at/api/events
Content-Type: application/json
```

### Request Payload

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
  "timestamp": "2026-01-27T21:00:00.000Z",
  "project_id": "chainsights"
}
```

For custom events:
```json
{
  "type": "customEvent",
  "event_name": "cta_click",
  "event_data": { "button": "get_report", "dao": "gitcoin" },
  "path": "/rankings",
  "session_id": "...",
  "device_id": "...",
  ...
}
```

### Response

**Success:**
```json
{ "ok": true }
```

**Validation Error (still 200):**
```json
{ "ok": false, "error": "Missing required field: project_id" }
```

### Field Mapping to Existing Schema

| Incoming Field | DB Column | Transform |
|----------------|-----------|-----------|
| `type` | `type` | Direct |
| `event_name` | `event_name` | Direct |
| `event_data` | `event_data` | Direct (JSONB) |
| `path` | `path` | Direct |
| `referrer` | `referrer` | Direct |
| `session_id` | `session_id` | UUID string → BIGINT hash |
| `device_id` | `device_id` | UUID string → BIGINT hash |
| `device_type` | `device_type` | Direct |
| `browser_name` | `browser_name` | Direct |
| `browser_version` | `browser_version` | Direct |
| `os_name` | `os_name` | Direct |
| `timestamp` | `timestamp` | ISO string → TIMESTAMPTZ |
| `project_id` | `project_id` | Direct |
| (from headers) | `country` | `x-vercel-ip-country` header |

### UUID to BIGINT Hash

```typescript
function uuidToInt64(uuid: string): bigint {
  // Take first 16 hex chars (64 bits) of UUID
  const hex = uuid.replace(/-/g, '').slice(0, 16);
  return BigInt('0x' + hex);
}
```

### CORS Configuration

```typescript
const ALLOWED_ORIGINS = [
  'https://chainsights.one',
  'https://www.chainsights.one',
  'https://tellingcube.com',
  'https://www.tellingcube.com',
  'https://hoki.help',
  'https://www.hoki.help',
  'https://masem.at',
  'https://www.masem.at',
];

// Also allow localhost for development
if (process.env.NODE_ENV === 'development') {
  // Allow any localhost origin
}
```

---

## Tasks

- [ ] **Task 1:** Create `app/api/events/route.ts` with POST handler
- [ ] **Task 2:** Implement request validation
- [ ] **Task 3:** Implement CORS headers (preflight OPTIONS + response headers)
- [ ] **Task 4:** Implement UUID → BIGINT conversion
- [ ] **Task 5:** Implement country extraction from headers
- [ ] **Task 6:** Implement DB insert with error handling
- [ ] **Task 7:** Test with curl/Postman

---

## Test Scenarios

1. **Valid pageView** → 200, data in DB
2. **Valid customEvent** → 200, data in DB with event_name/event_data
3. **Missing project_id** → 200, `{ ok: false, error: "..." }`
4. **Invalid JSON** → 200, `{ ok: false, error: "..." }`
5. **CORS preflight from chainsights.one** → 200 with correct headers
6. **CORS preflight from evil.com** → No CORS headers (browser blocks)
7. **DB connection error** → 200, error logged server-side

---

## Files to Create/Modify

| File | Action |
|------|--------|
| `app/api/events/route.ts` | Create |

---

## Dependencies

- Existing `events` table in Neon DB
- No external packages needed (using existing neon client)

---

## Notes

- Rate limiting deferred (can add later if abuse detected)
- No authentication required (public endpoint, like Vercel's)
- `origin` column in DB can store the request origin for debugging

---

## Definition of Done

- [ ] Endpoint responds to POST requests
- [ ] CORS works from all allowed origins
- [ ] Events appear in database
- [ ] Errors are logged but don't fail the request
- [ ] Code reviewed
- [ ] Committed and pushed
