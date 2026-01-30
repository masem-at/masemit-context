# API Contracts - masemIT Analytics

> Generated: 2026-01-28

## Overview

| Endpoint | Method | Purpose |
|----------|--------|---------|
| `/api/events` | POST | Receive events from tracker.js (primary) |
| `/api/events` | GET | Query events by project/range |
| `/api/projects/[id]` | GET | Get project details |
| `/api/projects/[id]` | PATCH | Update project settings |
| `/api/ingest` | POST | Receive events from Vercel Drains (legacy) |
| `/api/ingest` | GET | Health check |

---

## POST /api/events

Receives analytics events from self-hosted tracker.js (primary data source).

### CORS

Allowed origins:
- `https://www.masem.at`
- `https://chainsights.one`
- `https://tellingcube.com`
- `https://hoki.help`
- `http://localhost:*` (development)

### Request

**Headers:**
```
Content-Type: text/plain
```

**Body (JSON string):**

```typescript
interface TrackerEvent {
  // Required
  type: 'pageView' | string;      // Event type
  path: string;                    // Page path
  project_id: string;              // Project identifier
  timestamp: string;               // ISO 8601 timestamp

  // Session (UUID → BIGINT conversion)
  session_id?: string;             // UUID v4 (15 hex chars used)
  device_id?: string;              // UUID v4 (15 hex chars used)

  // Device Info
  browser?: string;                // Browser name
  browser_version?: string;
  os?: string;                     // Operating system
  os_version?: string;
  device_type?: 'desktop' | 'mobile' | 'tablet';

  // URL Data
  origin?: string;
  referrer?: string;
  query_params?: string;

  // Custom Event Data
  event_data?: Record<string, unknown>;
}
```

### Response

**Success (200):**
```json
{
  "success": true
}
```

**Bad Request (400):**
```json
{
  "error": "Missing required fields: type, path, project_id, timestamp"
}
```

**CORS Error (403):**
```json
{
  "error": "CORS origin not allowed"
}
```

### UUID to BIGINT Conversion

tracker.js generates UUID v4 IDs. For database storage, 15 hex characters are extracted and converted to BIGINT:

```typescript
// Example: "a1b2c3d4-e5f6-7890-abcd-ef1234567890"
// Extract: "a1b2c3d4e5f6789" (15 chars)
// Convert: parseInt("a1b2c3d4e5f6789", 16)
// Result:  725983580987562889n (fits in signed BIGINT)
```

---

## GET /api/events

Query events for debugging/admin purposes.

### Request

**Query Parameters:**
| Param | Type | Description |
|-------|------|-------------|
| `project` | string | Filter by project ID |
| `range` | string | Date range (e.g., "7d") |

### Response

**Success (200):**
```json
{
  "events": [
    {
      "id": "123",
      "type": "pageView",
      "path": "/",
      "timestamp": "2026-01-28T12:00:00.000Z",
      ...
    }
  ]
}
```

---

## GET /api/projects/[id]

Get details for a specific project.

### Response

**Success (200):**
```json
{
  "id": "chainsights",
  "name": "ChainSights",
  "domain": "chainsights.one",
  "color": "#00E5C0",
  "vercel_project_id": "prj_xxx",
  "created_at": "2026-01-01T00:00:00.000Z"
}
```

**Not Found (404):**
```json
{
  "error": "Project not found"
}
```

---

## PATCH /api/projects/[id]

Update project settings.

### Request

**Body:**
```json
{
  "name": "ChainSights",
  "domain": "chainsights.one",
  "color": "#00E5C0",
  "vercel_project_id": "prj_xxx"
}
```

### Response

**Success (200):**
```json
{
  "success": true,
  "project": { ... }
}
```

---

## POST /api/ingest

Receives analytics events from Vercel Web Analytics Drains.

### Authentication

**HMAC-SHA1 Signature Verification**

| Header | Description |
|--------|-------------|
| `x-vercel-signature` | HMAC-SHA1 hash of request body using `DRAIN_SECRET` |

```typescript
// Verification logic
const expectedSignature = createHmac('sha1', DRAIN_SECRET)
  .update(body)
  .digest('hex');

timingSafeEqual(Buffer.from(signature), Buffer.from(expectedSignature));
```

### Request

**Headers:**
```
Content-Type: application/json | application/x-ndjson
x-vercel-signature: <hmac-sha1-signature>
```

**Body (JSON array or NDJSON):**

```typescript
interface VercelAnalyticsEvent {
  // Required
  schemaVersion: string;
  timestamp: number;          // Unix timestamp (ms)
  projectId: string;          // Vercel project ID (prj_xxx)
  path: string;               // Page path

  // Optional - Event Type
  type?: 'pageView' | 'customEvent';
  eventName?: string;         // For customEvent
  eventData?: string;         // JSON string

  // Session
  ownerId: string;
  dsn: string;
  sessionId?: number;
  deviceId?: number;

  // URL Data
  origin?: string;
  referrer?: string;
  queryParams?: string;
  route?: string;

  // Geo
  country?: string;           // ISO 3166-1 alpha-2
  region?: string;
  city?: string;

  // Device & Browser
  osName?: string;
  osVersion?: string;
  browserName?: string;
  browserVersion?: string;
  deviceType?: string;        // 'desktop' | 'mobile' | 'tablet'
  deviceBrand?: string;
  deviceModel?: string;
  clientType?: string;

  // Vercel Metadata
  environment?: string;
  deploymentUrl?: string;
  deploymentId?: string;

  // SDK Info
  sdkVersion?: string;
  sdkName?: string;
  sdkFullVersion?: string;
  engineName?: string;
  engineVersion?: string;
  flags?: string;
}
```

### Response

**Success (200):**
```json
{
  "success": true,
  "processed": 5
}
```

**Unauthorized (401):**
```json
{
  "error": "Invalid signature"
}
```

**Server Error (500):**
```json
{
  "error": "Failed to process events"
}
```

### Project Mapping

Incoming Vercel `projectId` is mapped to internal project IDs:

| Environment Variable | Internal ID |
|---------------------|-------------|
| `PROJECT_MASEMIT` | `masemit-site` |
| `PROJECT_CHAINSIGHTS` | `chainsights` |
| `PROJECT_TELLINGCUBE` | `telling-cube` |
| `PROJECT_HOKIHELP` | `hoki-help` |

Unknown project IDs are logged and skipped.

### UTM Extraction

UTM parameters are extracted from `queryParams`:

```typescript
// Input: "?utm_source=twitter&utm_medium=social"
// Output: { utm_source: 'twitter', utm_medium: 'social', ... }
```

---

## GET /api/ingest

Health check endpoint for Vercel Drain configuration testing.

### Request

No parameters required.

### Response

**Success (200):**
```json
{
  "status": "ok",
  "service": "masemIT-analytics",
  "timestamp": "2026-01-26T12:00:00.000Z"
}
```

---

## Error Handling

| Status | Condition | Action |
|--------|-----------|--------|
| 200 | Valid signature, events processed | Continue |
| 401 | Invalid or missing signature | Reject request |
| 500 | Database error, parse error | Log and return error |

## Rate Limits

No explicit rate limiting. Vercel Drains batch events and send periodically.

## Data Flow

```
┌─────────────────┐
│  Vercel Drains  │
│  (JSON/NDJSON)  │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ Signature Check │
│ (HMAC-SHA1)     │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ Parse Events    │
│ (JSON/NDJSON)   │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ Map Project ID  │
│ Extract UTM     │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ Insert to       │
│ events table    │
└─────────────────┘
```
