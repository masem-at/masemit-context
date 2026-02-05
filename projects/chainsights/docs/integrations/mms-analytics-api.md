# MMS Analytics API Integration

**Status:** Ready for Integration
**Owner:** masemIT
**Used by:** Epic 20 - Engagement Report (Traffic Attribution)

---

## Overview

The MMS Analytics API provides access to analytics data from analytics.masem.at for ChainSights traffic attribution reporting.

## Configuration

### Environment Variables

```env
MMS_API_URL=https://api.masem.at
MMS_API_KEY=mms_chainsights_xxxxx
```

- **Local:** `.env.local` (already configured)
- **Production:** Vercel Environment Variables (already configured)

---

## Authentication

API keys follow the format `mms_{project}_{secret}` and must be included in request headers:

```typescript
headers: {
  'x-api-key': process.env.MMS_API_KEY
}
```

Required scope: `analytics:read`

---

## Endpoints

### GET /v1/analytics/referrers

Retrieves domain-level referrer aggregations for ChainSights.

**URL:** `{MMS_API_URL}/v1/analytics/referrers`

#### Parameters

| Name | Type | Required | Default | Notes |
|------|------|----------|---------|-------|
| `project` | string | Yes | — | Use `chainsights` |
| `from` | string | No | 30 days prior | ISO format (YYYY-MM-DD) |
| `to` | string | No | current date | ISO format (YYYY-MM-DD) |
| `limit` | number | No | 20 | Range: 1–100 |

#### Example Request

```typescript
const response = await fetch(
  `${process.env.MMS_API_URL}/v1/analytics/referrers?project=chainsights&from=2026-01-27&to=2026-02-02&limit=20`,
  {
    headers: {
      'x-api-key': process.env.MMS_API_KEY!,
    },
  }
)
```

#### Response Structure (200 OK)

```typescript
interface ReferrersResponse {
  project: string
  period: {
    from: string  // ISO date
    to: string    // ISO date
  }
  referrers: Array<{
    domain: string      // e.g., "discord.com", "x.com", "(direct)"
    visits: number      // Session count
    pageviews: number   // Total page views
    visitors: number    // Unique visitors
  }>
  totals: {
    visits: number
    pageviews: number
    visitors: number
  }
}
```

**Notes:**
- Referrers are sorted by `visits` descending
- Direct traffic appears as `"(direct)"`
- Full URLs are simplified to domain-only

#### Caching

- Historical date ranges: 5 minute cache
- Current-date queries: 2 minute cache

---

## Error Handling

| Status | Code | Scenario |
|--------|------|----------|
| 400 | `VALIDATION_ERROR` | Invalid parameters |
| 401 | `UNAUTHORIZED` | Missing or invalid API key |
| 403 | `FORBIDDEN` | Insufficient permissions |
| 500 | `INTERNAL_ERROR` | Server-side issues |

**Error Response:**

```typescript
interface ErrorResponse {
  error: {
    code: string
    message: string
    details?: Record<string, string>
  }
}
```

---

## Usage in ChainSights

### Epic 20: Engagement Report - Traffic Attribution

The referrers endpoint provides data for the "Traffic Attribution" section:

```
┌─────────────────────────────────────────────┐
│  Traffic from Community Channels            │
├─────────────────────────────────────────────┤
│  Discord     45 visits   98 pageviews  2.2x │
│  X/Twitter   23 visits   31 pageviews  1.3x │
│  Forum       12 visits   34 pageviews  2.8x │
│  (direct)   156 visits  312 pageviews  2.0x │
└─────────────────────────────────────────────┘
```

### Recommended Implementation

Create a client module at `src/lib/mms/analytics.ts`:

```typescript
export async function getReferrers(from: string, to: string, limit = 20) {
  const url = new URL(`${process.env.MMS_API_URL}/v1/analytics/referrers`)
  url.searchParams.set('project', 'chainsights')
  url.searchParams.set('from', from)
  url.searchParams.set('to', to)
  url.searchParams.set('limit', limit.toString())

  const response = await fetch(url.toString(), {
    headers: { 'x-api-key': process.env.MMS_API_KEY! },
    next: { revalidate: 300 }, // 5 min cache
  })

  if (!response.ok) {
    throw new Error(`MMS API error: ${response.status}`)
  }

  return response.json()
}
```

---

## Future Endpoints (Planned)

- Pageview trends over time
- Top pages by views
- Geographic analysis
- Custom event tracking
- UTM campaign metrics

---

## Links

- **API Docs:** https://api.masem.at/docs/analytics
- **Epic 20:** `docs/epics-phase-3.md`
- **Engagement Hub Ticket:** `docs/_masemIT/requirements/ticket-engagement-hub.md`
