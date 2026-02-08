# Epic: Analytics Service MVP — Referrer Endpoint

**Epic ID:** ANALYTICS-01
**Status:** Complete (MVP)
**Priority:** High
**Created:** 2026-02-05
**Updated:** 2026-02-05
**Completed:** 2026-02-05
**First Consumer:** ChainSights Admin Dashboard

---

## Implementation Checklist

### Story 1: Analytics DB Connection + Health Check ✅
- [x] `ANALYTICS_DATABASE_URL` environment variable documented
- [x] `src/db/analytics.ts` — Second Drizzle client for Analytics DB
- [x] Read-only connection to analytics.masem.at NeonDB
- [x] `/health` endpoint extended with Analytics DB status
- [x] Graceful degradation: MMS stays healthy if Analytics DB is down

### Story 2: Drizzle Schema (Read-Only) ✅
- [x] `src/services/analytics/schema.ts` created
- [x] `projects` table schema defined
- [x] `events` table schema defined (partitioned table)
- [x] **No migrations** — schema is read-only reference

### Story 3: Referrer Endpoint Implementation ✅
- [x] `src/services/analytics/types.ts` — Zod validation + TypeScript types
- [x] `src/services/analytics/queries.ts` — Referrer aggregation + totals query
- [x] `src/services/analytics/handlers.ts` — Request handler
- [x] `src/services/analytics/routes.ts` — Express Router with GET `/referrers`
- [x] Route registered in `src/index.ts` under `/v1/analytics`
- [x] Domain extraction from referrer URLs
- [x] `(direct)` handling for null/empty referrers
- [x] Date range filtering with defaults (30 days)
- [x] Limit parameter (1-100, default 20)

### Story 4: Caching Layer ✅
- [x] Upstash Redis integration for analytics endpoints
- [x] Cache key pattern: `analytics:referrers:{project}:{from}:{to}:{limit}`
- [x] TTL: 5 minutes (standard), 2 minutes (if "to" date is today)
- [x] Cache invalidation not needed (read-only data)

### Story 5: API Documentation ✅
- [x] `/docs/analytics` route added
- [x] Referrer endpoint fully documented
- [x] **Validated against real API responses**
- [x] Example requests and responses from actual calls

### Story 6: Dynamic Project Validation (Optional/Future)
- [ ] Query `projects` table instead of hardcoded list
- [ ] Cache valid project IDs (1h TTL)
- [ ] No redeploy needed for new projects

---

## Description

Add an Analytics Service to MMS that reads from the existing masemIT Analytics database (analytics.masem.at / NeonDB) and exposes referrer data per project via a REST endpoint. This is the second MMS service after Donations, following the same architectural pattern (routes/handlers/queries/types) but with a **separate, read-only database connection**.

### Core Principle

MMS becomes the central API gateway for all masemIT data. Each service owns its DB connection. Analytics data stays in the Analytics DB — MMS only reads it.

### Architecture

```
src/services/
├── donations/        ← DB: mms (NeonDB, read-write)
├── analytics/        ← DB: analytics (NeonDB, READ-ONLY)
│   ├── schema.ts     ← Read-only Drizzle schema
│   ├── routes.ts
│   ├── handlers.ts
│   ├── queries.ts
│   └── types.ts
```

### Database Connection

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

> Use a **read-only database role** for the connection string. MMS must never write to the Analytics DB.

---

## Stories

### Story 1: Analytics DB Connection + Health Check

**ID:** ANALYTICS-01-1
**Status:** To Do
**Depends On:** None
**Blocked By:** Read-only DB user creation (Sempre handles)

**As a** developer
**I want** a separate database connection to the Analytics DB
**So that** MMS can read analytics data without affecting the donations DB

**Acceptance Criteria:**
- [ ] New file `src/db/analytics.ts` with Drizzle client
- [ ] Uses `ANALYTICS_DATABASE_URL` environment variable
- [ ] Connection is HTTP-based (Neon serverless driver)
- [ ] `/health` endpoint returns status for both DBs:
  ```json
  {
    "status": "healthy",
    "timestamp": "2026-02-05T10:00:00Z",
    "services": {
      "mms_db": "connected",
      "analytics_db": "connected"
    }
  }
  ```
- [ ] If Analytics DB is down: `status: "degraded"`, `analytics_db: "disconnected"`
- [ ] MMS stays operational even if Analytics DB fails (donations still work)

**Technical Notes:**
- Winston: Health check should use a simple query like `SELECT 1` with short timeout (2s)
- Winston: Use try-catch, don't let Analytics DB failure crash the health endpoint
- Pattern: Same as donations DB but separate client instance

---

### Story 2: Drizzle Schema (Read-Only)

**ID:** ANALYTICS-01-2
**Status:** To Do
**Depends On:** Story 1

**As a** developer
**I want** TypeScript type-safety for Analytics DB queries
**So that** I can use Drizzle ORM for query building

**Acceptance Criteria:**
- [ ] New file `src/services/analytics/schema.ts`
- [ ] `projects` table schema defined:
  ```typescript
  export const projects = pgTable('projects', {
    id: text('id').primaryKey(),
    name: text('name').notNull(),
    domain: text('domain'),
  })
  ```
- [ ] `events` table schema defined (only columns needed for referrer query):
  ```typescript
  export const events = pgTable('events', {
    id: bigint('id', { mode: 'number' }).notNull(),
    type: text('type'),
    timestamp: timestamp('timestamp', { withTimezone: true }).notNull(),
    projectId: text('project_id').notNull(),
    sessionId: bigint('session_id', { mode: 'number' }),
    deviceId: bigint('device_id', { mode: 'number' }),
    referrer: text('referrer'),
    path: text('path').notNull(),
  })
  ```
- [ ] **NO `drizzle-kit push/migrate`** — Schema is read-only reference
- [ ] Schema matches Analytics DB (analytics.masem.at)

**Important:** This is not a migration. The Analytics DB is managed by analytics.masem.at. We only define the schema for type-safety and query building.

---

### Story 3: Referrer Endpoint Implementation

**ID:** ANALYTICS-01-3
**Status:** To Do
**Depends On:** Story 1, Story 2

**As a** ChainSights Admin Dashboard
**I want** aggregated referrer data for my project
**So that** I can see where my visitors come from

**Acceptance Criteria:**
- [ ] `GET /v1/analytics/referrers` endpoint
- [ ] Query parameters validated with Zod:
  | Parameter | Type | Required | Default | Validation |
  |-----------|------|----------|---------|------------|
  | `project` | string | Yes | — | One of: `chainsights`, `hoki`, `tellingcube`, `masem` |
  | `from` | string | No | 30 days ago | ISO date YYYY-MM-DD |
  | `to` | string | No | today | ISO date YYYY-MM-DD |
  | `limit` | number | No | 20 | 1-100 |
- [ ] Response format:
  ```json
  {
    "project": "chainsights",
    "period": { "from": "2026-01-01", "to": "2026-01-31" },
    "referrers": [
      { "domain": "google.com", "visits": 142, "pageviews": 387, "unique_visitors": 98 },
      { "domain": "t.co", "visits": 87, "pageviews": 156, "unique_visitors": 71 },
      { "domain": "(direct)", "visits": 234, "pageviews": 512, "unique_visitors": 189 }
    ],
    "totals": { "visits": 463, "pageviews": 1055, "unique_visitors": 358 }
  }
  ```
- [ ] Domain extraction: `https://www.google.com/search?q=test` → `google.com`
- [ ] `(direct)` for NULL or empty referrers
- [ ] `totals` is a separate query (not sum of limited referrers)
- [ ] Error responses follow MMS pattern:
  | Status | Code | When |
  |--------|------|------|
  | 400 | `VALIDATION_ERROR` | Missing/invalid project, bad date format |
  | 401 | `UNAUTHORIZED` | Missing or invalid API key |
  | 404 | `NOT_FOUND` | Unknown project ID |
  | 500 | `INTERNAL_ERROR` | DB connection failure |

**SQL Query (Referrers):**
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

**SQL Query (Totals):**
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

**Performance Notes:**
- Partitioned table: `events_2025_q4`, `events_2026_q1`, `events_2026_q2`
- Index: `idx_events_referrer ON events (project_id, referrer) WHERE (referrer IS NOT NULL)`
- Index: `idx_events_project_timestamp ON events (project_id, timestamp DESC)`

---

### Story 4: Caching Layer

**ID:** ANALYTICS-01-4
**Status:** To Do
**Depends On:** Story 3

**As a** API consumer
**I want** fast response times
**So that** the dashboard feels responsive

**Acceptance Criteria:**
- [ ] Upstash Redis caching (same instance as donations)
- [ ] Cache key: `analytics:referrers:{project}:{from}:{to}:{limit}`
- [ ] TTL logic:
  - If `to` date equals today → 2 min TTL (data still changing)
  - Otherwise → 5 min TTL
- [ ] Cache hit returns cached response directly
- [ ] Cache miss executes queries and caches result
- [ ] No cache invalidation needed (read-only source)

**Example:**
```typescript
const cacheKey = `analytics:referrers:${project}:${from}:${to}:${limit}`
const cached = await redis.get(cacheKey)
if (cached) return JSON.parse(cached)

const result = await getReferrerData(project, from, to, limit)
const ttl = isToday(to) ? 120 : 300
await redis.set(cacheKey, JSON.stringify(result), { ex: ttl })
return result
```

---

### Story 5: API Documentation

**ID:** ANALYTICS-01-5
**Status:** To Do
**Depends On:** Story 3

**As a** API consumer
**I want** accurate documentation
**So that** I can integrate without guessing

**Acceptance Criteria:**
- [ ] New route: `GET /docs/analytics` returns markdown
- [ ] `/docs` index includes analytics entry
- [ ] Documentation covers:
  - Endpoint: `GET /v1/analytics/referrers`
  - Authentication (x-api-key)
  - Query parameters with examples
  - Success response with real example
  - Error responses
  - Rate limiting info
- [ ] **Rule U1:** All example responses must come from actual API calls
- [ ] Paige validates against deployed API before finalizing

**Validation Process:**
1. Story 3 deployed to production
2. Paige makes real API calls with `curl`
3. Paige uses actual responses in documentation
4. Documentation matches reality

---

### Story 6: Dynamic Project Validation (Optional/Future)

**ID:** ANALYTICS-01-6
**Status:** Backlog (Optional)
**Depends On:** Story 3

**As a** maintainer
**I want** project IDs validated against the database
**So that** new projects work without code changes

**Acceptance Criteria:**
- [ ] Query `projects` table for valid IDs instead of hardcoded list
- [ ] Cache valid project IDs in Redis (1h TTL)
- [ ] Refresh cache on miss
- [ ] Unknown project → 404 NOT_FOUND

**Current Approach (Story 3):**
```typescript
const VALID_PROJECTS = ['chainsights', 'hoki', 'tellingcube', 'masem'] as const
```

**Future Approach (Story 6):**
```typescript
async function getValidProjects(): Promise<string[]> {
  const cached = await redis.get('analytics:valid_projects')
  if (cached) return JSON.parse(cached)

  const projects = await analyticsDb.select({ id: projectsTable.id }).from(projectsTable)
  const ids = projects.map(p => p.id)
  await redis.set('analytics:valid_projects', JSON.stringify(ids), { ex: 3600 })
  return ids
}
```

**Decision:** Start with hardcoded list (Story 3). Implement dynamic validation later if new projects are added frequently.

---

## Technical Decisions

| Topic | Decision | Rationale |
|-------|----------|-----------|
| DB Connection | Separate Drizzle client | Isolation from donations DB, different credentials |
| Read-only | Enforced by DB role | MMS cannot accidentally write to Analytics DB |
| Schema | No migrations | Analytics DB managed by analytics.masem.at |
| Project validation | Hardcoded list (MVP) | 4 known projects, can add dynamic later |
| Caching | Upstash Redis | Same infrastructure as donations |
| TTL | 5min / 2min | Balance freshness vs performance |
| Domain extraction | SQL regex | Efficient, done in DB |
| Totals query | Separate from referrers | LIMIT would skew totals if combined |

---

## Database Access

### Analytics DB Tables (Read-Only)

| Table | Purpose | MMS Access |
|-------|---------|------------|
| `projects` | Project metadata | Read (validation) |
| `events` | Pageview events | Read (aggregation) |
| `daily_stats` | Pre-aggregated data | Not used (incomplete) |

### Indexes Used

```sql
idx_events_referrer ON events (project_id, referrer) WHERE (referrer IS NOT NULL)
idx_events_project_timestamp ON events (project_id, timestamp DESC)
```

---

## Dependencies

```
Story 1 (DB Connection) ──┬──→ Story 2 (Schema)
                          │
                          └──→ Story 3 (Endpoint) ──┬──→ Story 4 (Caching)
                                                    │
                                                    └──→ Story 5 (Docs)

Story 6 (Dynamic Validation) ← Optional, after MVP
```

**External Dependency:**
- Read-only DB user in Analytics NeonDB (Sempre handles in parallel)

---

## Out of Scope

- Writing to Analytics DB
- Real-time websocket updates
- Custom event queries (future endpoint)
- UTM campaign analytics (future endpoint)
- Geographic distribution (future endpoint)
- Pre-aggregated dashboard data (use `daily_stats` view later)
- Admin endpoints for analytics configuration

---

## Future Analytics Endpoints

These endpoints use the same Analytics DB connection and can be added after MVP:

| Endpoint | Description |
|----------|-------------|
| `GET /v1/analytics/pageviews` | Pageviews & visitors per day/week/month |
| `GET /v1/analytics/pages` | Top pages with pageviews/unique visitors |
| `GET /v1/analytics/countries` | Geographic distribution |
| `GET /v1/analytics/events` | Custom events (CTA clicks, scroll depth) |
| `GET /v1/analytics/utm` | UTM campaign performance |
| `GET /v1/analytics/overview` | Dashboard summary |

---

## Open Questions (Resolved)

| Question | Resolution |
|----------|------------|
| Read-only DB user exists? | Sempre creates in parallel |
| Project validation: hardcoded vs dynamic? | Hardcoded for MVP (Story 3), dynamic optional (Story 6) |
| Rate limiting? | Same as donations (internal API) |
