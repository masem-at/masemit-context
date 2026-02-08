# MMS Analytics Service — Referrer Endpoint

**Project:** MMS — masemIT MicroServices (api.masem.at)
**Author:** Claude (Co-Founder masemIT)
**Owner:** Mario Semper
**Date:** 2026-02-05
**Version:** 1.0
**Status:** Ready for Claude Code
**Priority:** High
**First Consumer:** ChainSights Admin Dashboard

---

## 1. Summary

Add an Analytics Service to MMS that reads from the existing masemIT Analytics database (analytics.masem.at / NeonDB) and exposes referrer data per project via a REST endpoint. This is the second MMS service after Donations, following the same architectural pattern (routes/handlers/queries/types) but with a **separate, read-only database connection** to the Analytics DB.

### Guiding Principle

MMS becomes the central API gateway for all masemIT data. Each service owns its DB connection. Analytics data stays in the Analytics DB — MMS only reads it.

---

## 2. Architecture

### 2.1 Service Location

```
src/services/
├── donations/        ← DB: mms (NeonDB)
├── analytics/        ← DB: analytics (NeonDB, READ-ONLY)
│   ├── routes.ts
│   ├── handlers.ts
│   ├── queries.ts
│   └── types.ts
```

### 2.2 Database Connection

A second Drizzle client pointing to the Analytics NeonDB instance. Completely independent from the MMS donations database.

```typescript
// src/db/analytics.ts
import { neon } from '@neondatabase/serverless'
import { drizzle } from 'drizzle-orm/neon-http'

const sql = neon(process.env.ANALYTICS_DATABASE_URL!)
export const analyticsDb = drizzle(sql)
```

**Environment Variable:**
```
ANALYTICS_DATABASE_URL=postgresql://...@ep-xxx.eu-central-1.aws.neon.tech/analytics?sslmode=require
```

> ⚠️ Use a **read-only database role** for the connection string. MMS must never write to the Analytics DB.

### 2.3 Auth

Same `x-api-key` system as Donations. The analytics endpoints are internal — only masemIT projects call them.

---

## 3. Analytics DB Schema (Relevant Tables)

### 3.1 projects

```sql
CREATE TABLE projects (
    id          text PRIMARY KEY,   -- 'chainsights', 'hoki', 'tellingcube', 'masem'
    name        text NOT NULL,
    domain      text,
    color       text DEFAULT '#4A89F3',
    created_at  timestamptz DEFAULT now(),
    vercel_project_id text,
    linkedin_url text,
    x_url       text
);
```

**Known project IDs:** `chainsights`, `hoki`, `tellingcube`, `masem`

### 3.2 events (Partitioned by Quarter)

```sql
CREATE TABLE events (
    id              bigint NOT NULL,
    type            text,                    -- 'pageView', 'custom', etc.
    event_name      text,
    event_data      jsonb,
    "timestamp"     timestamptz NOT NULL,
    project_id      text NOT NULL,           -- FK → projects.id
    session_id      bigint,
    device_id       bigint,
    referrer        text,                    -- Full referrer URL or NULL
    path            text NOT NULL,
    country         text,
    utm_source      text,
    utm_medium      text,
    utm_campaign    text,
    is_first_visit  boolean DEFAULT true
    -- ... (weitere Spalten: os_name, browser_name, etc.)
) PARTITION BY RANGE ("timestamp");
```

**Partitions:** `events_2025_q4`, `events_2026_q1`, `events_2026_q2`

**Relevant Index:**
```sql
CREATE INDEX idx_events_referrer ON events (project_id, referrer)
    WHERE (referrer IS NOT NULL);
```

### 3.3 daily_stats (Voraggregiert — nicht verwenden)

Hat `top_referrers` als JSONB, aber die Struktur ist unklar und möglicherweise nur Top-N. Wir gehen direkt auf die `events`-Tabelle für maximale Flexibilität.

---

## 4. Endpoint Specification

### GET /v1/analytics/referrers

Returns aggregated referrer data (domain-level) for a given project.

**Auth:** `x-api-key` header (internal)

**Query Parameters:**

| Parameter    | Type   | Required | Default    | Description |
|-------------|--------|----------|------------|-------------|
| `project`   | string | ✅       | —          | Project ID: `chainsights`, `hoki`, `tellingcube`, `masem` |
| `from`      | string | ❌       | 30 days ago | Start date (ISO: `2026-01-01`) |
| `to`        | string | ❌       | today       | End date (ISO: `2026-01-31`) |
| `limit`     | number | ❌       | 20          | Max referrer domains returned (1–100) |

**Example Request:**
```
GET /v1/analytics/referrers?project=chainsights&from=2026-01-01&to=2026-01-31&limit=10
x-api-key: cs_xxxxxx
```

**Success Response (200):**
```json
{
  "project": "chainsights",
  "period": {
    "from": "2026-01-01",
    "to": "2026-01-31"
  },
  "referrers": [
    {
      "domain": "google.com",
      "visits": 142,
      "pageviews": 387,
      "unique_visitors": 98
    },
    {
      "domain": "t.co",
      "visits": 87,
      "pageviews": 156,
      "unique_visitors": 71
    },
    {
      "domain": "(direct)",
      "visits": 234,
      "pageviews": 512,
      "unique_visitors": 189
    }
  ],
  "totals": {
    "visits": 463,
    "pageviews": 1055,
    "unique_visitors": 358
  }
}
```

**Error Responses:**

| Status | Code | When |
|--------|------|------|
| 400 | `VALIDATION_ERROR` | Missing/invalid project, bad date format |
| 401 | `UNAUTHORIZED` | Missing or invalid API key |
| 404 | `NOT_FOUND` | Unknown project ID |
| 500 | `INTERNAL_ERROR` | DB connection failure |

---

## 5. SQL Query

```sql
SELECT
    CASE
        WHEN e.referrer IS NULL OR e.referrer = '' THEN '(direct)'
        ELSE regexp_replace(
            regexp_replace(e.referrer, '^https?://(www\.)?', ''),
            '/.*$', ''
        )
    END AS domain,
    COUNT(DISTINCT e.session_id) AS visits,
    COUNT(*) AS pageviews,
    COUNT(DISTINCT e.device_id) AS unique_visitors
FROM events e
WHERE e.project_id = $1
  AND e."timestamp" >= $2::date
  AND e."timestamp" < ($3::date + interval '1 day')
  AND (e.type IS NULL OR e.type = 'pageView')
GROUP BY domain
ORDER BY visits DESC
LIMIT $4;
```

**Performance Notes:**
- Partitionierung auf `timestamp` sorgt dafür, dass nur relevante Partitionen gescannt werden
- `idx_events_referrer` Index auf `(project_id, referrer)` wird für die WHERE-Clause genutzt
- `idx_events_project_timestamp` Index auf `(project_id, timestamp DESC)` hilft beim Partition Pruning
- Bei großen Zeiträumen (>3 Monate) evtl. Caching mit Upstash Redis (5min TTL)

**Totals:** Separate Query oder im gleichen Call aggregieren (SUM über die Referrer-Ergebnisse reicht nicht, da LIMIT greift). Empfehlung: Zweite Query für Totals ohne LIMIT und ohne Referrer-Gruppierung:

```sql
SELECT
    COUNT(DISTINCT e.session_id) AS visits,
    COUNT(*) AS pageviews,
    COUNT(DISTINCT e.device_id) AS unique_visitors
FROM events e
WHERE e.project_id = $1
  AND e."timestamp" >= $2::date
  AND e."timestamp" < ($3::date + interval '1 day')
  AND (e.type IS NULL OR e.type = 'pageView');
```

---

## 6. Drizzle Schema (Analytics DB — Read-Only)

Nur die für den Referrer-Endpoint benötigten Tabellen definieren. Kein Migrations-Management — die Analytics-DB wird von analytics.masem.at verwaltet.

```typescript
// src/services/analytics/schema.ts
import { pgTable, text, bigint, boolean, timestamp } from 'drizzle-orm/pg-core'

export const projects = pgTable('projects', {
  id: text('id').primaryKey(),
  name: text('name').notNull(),
  domain: text('domain'),
})

export const events = pgTable('events', {
  id: bigint('id', { mode: 'number' }).notNull(),
  type: text('type'),
  timestamp: timestamp('timestamp', { withTimezone: true }).notNull(),
  projectId: text('project_id').notNull(),
  sessionId: bigint('session_id', { mode: 'number' }),
  deviceId: bigint('device_id', { mode: 'number' }),
  referrer: text('referrer'),
  path: text('path').notNull(),
  country: text('country'),
  utmSource: text('utm_source'),
  utmMedium: text('utm_medium'),
  utmCampaign: text('utm_campaign'),
  isFirstVisit: boolean('is_first_visit'),
})
```

> ⚠️ **Kein `drizzle-kit push/migrate` auf diese DB.** Schema ist read-only und wird nur für Type-Safety und Query-Building verwendet.

---

## 7. Zod Validation

```typescript
// src/services/analytics/types.ts
import { z } from 'zod'

const VALID_PROJECTS = ['chainsights', 'hoki', 'tellingcube', 'masem'] as const

export const referrerQuerySchema = z.object({
  project: z.enum(VALID_PROJECTS),
  from: z.string()
    .regex(/^\d{4}-\d{2}-\d{2}$/, 'Date must be YYYY-MM-DD')
    .optional(),
  to: z.string()
    .regex(/^\d{4}-\d{2}-\d{2}$/, 'Date must be YYYY-MM-DD')
    .optional(),
  limit: z.coerce.number().int().min(1).max(100).default(20),
})

export type ReferrerQuery = z.infer<typeof referrerQuerySchema>

export type ReferrerResult = {
  domain: string
  visits: number
  pageviews: number
  unique_visitors: number
}

export type ReferrerResponse = {
  project: string
  period: { from: string; to: string }
  referrers: ReferrerResult[]
  totals: {
    visits: number
    pageviews: number
    unique_visitors: number
  }
}
```

---

## 8. Route Registration

```typescript
// src/index.ts (Express-Version nach Migration)
import { analyticsRoutes } from './services/analytics/routes'

// ... existing routes ...
app.use('/v1/analytics', authMiddleware, analyticsRoutes)
```

---

## 9. Caching Strategy

| Endpoint | TTL | Key Pattern |
|----------|-----|-------------|
| `/v1/analytics/referrers` | 5 min | `analytics:referrers:{project}:{from}:{to}:{limit}` |

Upstash Redis, gleicher Cache wie Donations. Bei Datumsfilter auf "heute" → kürzerer TTL (2 min) weil sich die Daten laufend ändern.

---

## 10. Implementation Checklist

- [ ] `ANALYTICS_DATABASE_URL` als Environment Variable in Vercel setzen (read-only role)
- [ ] `src/db/analytics.ts` — Zweiter Drizzle-Client für Analytics-DB
- [ ] `src/services/analytics/schema.ts` — Read-only Drizzle-Schema (events, projects)
- [ ] `src/services/analytics/types.ts` — Zod Validation + TypeScript Types
- [ ] `src/services/analytics/queries.ts` — Referrer-Aggregation + Totals Query
- [ ] `src/services/analytics/handlers.ts` — Request Handler mit Caching
- [ ] `src/services/analytics/routes.ts` — Express Router mit GET `/referrers`
- [ ] Route in `src/index.ts` registrieren unter `/v1/analytics`
- [ ] `/health` Endpoint erweitern: Analytics-DB Connection Status
- [ ] Caching mit Upstash Redis (5 min TTL)
- [ ] Testen: curl mit verschiedenen Filtern
- [ ] API-Docs aktualisieren (`/docs` Route)

---

## 11. Future Analytics Endpoints (Out of Scope)

Diese Endpoints nutzen dieselbe Analytics-DB-Connection und können später nach gleichem Pattern hinzugefügt werden:

| Endpoint | Description |
|----------|-------------|
| `GET /v1/analytics/pageviews` | Pageviews & Visitors pro Tag/Woche/Monat |
| `GET /v1/analytics/pages` | Top Pages mit Pageviews/Unique Visitors |
| `GET /v1/analytics/countries` | Geo-Verteilung der Besucher |
| `GET /v1/analytics/events` | Custom Events (CTA clicks, scroll depth, etc.) |
| `GET /v1/analytics/utm` | UTM Campaign Performance |
| `GET /v1/analytics/overview` | Dashboard-Summary (entspricht `v_project_overview` View) |

---

## 12. Open Decisions

- **Read-only DB User:** Existiert bereits ein read-only User in der Analytics NeonDB, oder muss einer angelegt werden?
- **Project-ID Validierung:** Hardcoded Liste (`VALID_PROJECTS`) oder dynamisch aus der `projects`-Tabelle? Empfehlung: Dynamisch mit Cache (1h TTL), dann braucht man bei neuem Projekt kein Redeploy.
- **Rate Limiting:** Gleiche Limits wie Donations-Endpoints, oder großzügiger weil read-only?
