# Data Models - masemIT Analytics

> Generated: 2026-01-28

## Overview

| Object | Type | Records | Description |
|--------|------|---------|-------------|
| `projects` | Table | 4 | Project metadata |
| `events` | Partitioned Table | Variable | Analytics events |
| `daily_stats` | Table | Variable | Aggregated daily stats |
| `v_project_overview` | View | 4 | Dashboard overview |

---

## Tables

### `projects`

Stores metadata for tracked projects.

```sql
CREATE TABLE projects (
    id TEXT PRIMARY KEY,
    name TEXT NOT NULL,
    domain TEXT,
    color TEXT DEFAULT '#4A89F3',
    vercel_project_id TEXT,          -- Legacy: for Vercel Drain mapping
    created_at TIMESTAMPTZ DEFAULT NOW()
);
```

**Columns:**

| Column | Type | Nullable | Description |
|--------|------|----------|-------------|
| `id` | TEXT | No | Internal project identifier (PK) |
| `name` | TEXT | No | Display name |
| `domain` | TEXT | Yes | Primary domain |
| `color` | TEXT | Yes | Brand color (hex), default #4A89F3 |
| `vercel_project_id` | TEXT | Yes | Legacy: Vercel project ID for Drain mapping |
| `created_at` | TIMESTAMPTZ | Yes | Creation timestamp |

**Seed Data:**

| id | name | domain | color |
|----|------|--------|-------|
| masemit-site | masem.at | www.masem.at | #0F2C55 |
| chainsights | ChainSights | chainsights.one | #00E5C0 |
| telling-cube | tellingCube | tellingcube.com | #B84EFF |
| hoki-help | hoki.help | hoki.help | #FF6B6B |

---

### `events`

Main analytics events table, partitioned by timestamp.

```sql
CREATE TABLE events (
    id BIGSERIAL,

    -- Event Identification
    type TEXT,                        -- 'pageView' | 'customEvent' | NULL
    event_name TEXT,
    event_data JSONB,

    -- Timestamps
    timestamp TIMESTAMPTZ NOT NULL,
    received_at TIMESTAMPTZ DEFAULT NOW(),

    -- Project & Session
    project_id TEXT NOT NULL REFERENCES projects(id),
    vercel_project_id TEXT,
    session_id BIGINT,
    device_id BIGINT,

    -- URL Data
    origin TEXT,
    path TEXT NOT NULL,
    referrer TEXT,
    query_params TEXT,
    route TEXT,

    -- Geo
    country TEXT,
    region TEXT,
    city TEXT,

    -- Device & Browser
    os_name TEXT,
    os_version TEXT,
    browser_name TEXT,
    browser_version TEXT,
    device_type TEXT,
    device_brand TEXT,
    device_model TEXT,

    -- Vercel Metadata
    environment TEXT,
    deployment_url TEXT,
    deployment_id TEXT,

    -- UTM Parameters
    utm_source TEXT,
    utm_medium TEXT,
    utm_campaign TEXT,
    utm_content TEXT,
    utm_term TEXT,

    PRIMARY KEY (id, timestamp)
) PARTITION BY RANGE (timestamp);
```

**Partitions:**

| Partition | Range |
|-----------|-------|
| `events_2025_q4` | 2025-10-01 to 2026-01-01 |
| `events_2026_q1` | 2026-01-01 to 2026-04-01 |
| `events_2026_q2` | 2026-04-01 to 2026-07-01 |

**Indexes:**

| Index | Columns | Purpose |
|-------|---------|---------|
| `idx_events_project_timestamp` | project_id, timestamp DESC | Main query index |
| `idx_events_path` | project_id, path | Top pages queries |
| `idx_events_referrer` | project_id, referrer (WHERE NOT NULL) | Referrer analysis |
| `idx_events_country` | project_id, country | Geo queries |
| `idx_events_utm_source` | project_id, utm_source (WHERE NOT NULL) | Campaign tracking |

**Note on `type` column:**
> Vercel sends `type = NULL` for regular pageviews. Only customEvents have explicit type.

---

### `daily_stats`

Optional aggregated daily statistics for faster historical queries.

```sql
CREATE TABLE daily_stats (
    date DATE NOT NULL,
    project_id TEXT NOT NULL REFERENCES projects(id),

    -- Metrics
    page_views INT DEFAULT 0,
    unique_visitors INT DEFAULT 0,
    unique_sessions INT DEFAULT 0,
    custom_events INT DEFAULT 0,

    -- Top Data (JSONB Arrays)
    top_referrers JSONB,
    top_pages JSONB,
    top_countries JSONB,

    updated_at TIMESTAMPTZ DEFAULT NOW(),

    PRIMARY KEY (date, project_id)
);
```

---

## Views

### `v_project_overview`

Dashboard overview aggregating today and 7-day stats per project.

```sql
CREATE OR REPLACE VIEW v_project_overview AS
SELECT
    p.id,
    p.name,
    p.domain,
    p.color,
    COALESCE(today.page_views, 0)::integer as today_views,
    COALESCE(today.unique_visitors, 0)::integer as today_visitors,
    COALESCE(week.page_views, 0)::bigint as week_views,
    COALESCE(week.unique_visitors, 0)::bigint as week_visitors
FROM projects p
LEFT JOIN (
    SELECT
        project_id,
        COUNT(*) as page_views,
        COUNT(DISTINCT device_id) as unique_visitors
    FROM events
    WHERE DATE(timestamp) = CURRENT_DATE
      AND (type IS NULL OR type = 'pageView')
    GROUP BY project_id
) today ON today.project_id = p.id
LEFT JOIN (
    SELECT
        project_id,
        COUNT(*) as page_views,
        COUNT(DISTINCT device_id) as unique_visitors
    FROM events
    WHERE timestamp >= NOW() - INTERVAL '7 days'
      AND (type IS NULL OR type = 'pageView')
    GROUP BY project_id
) week ON week.project_id = p.id;
```

**Output Columns:**

| Column | Type | Description |
|--------|------|-------------|
| id | TEXT | Project ID |
| name | TEXT | Project name |
| domain | TEXT | Project domain |
| color | TEXT | Brand color |
| today_views | INT | Page views today |
| today_visitors | INT | Unique visitors today |
| week_views | BIGINT | Page views (7 days) |
| week_visitors | BIGINT | Unique visitors (7 days) |

---

## Functions

### `extract_utm_params(query TEXT)`

Extracts UTM parameters from query string.

```sql
CREATE OR REPLACE FUNCTION extract_utm_params(query TEXT)
RETURNS TABLE (
    utm_source TEXT,
    utm_medium TEXT,
    utm_campaign TEXT,
    utm_content TEXT,
    utm_term TEXT
) AS $$
BEGIN
    RETURN QUERY SELECT
        (regexp_match(query, 'utm_source=([^&]+)'))[1],
        (regexp_match(query, 'utm_medium=([^&]+)'))[1],
        (regexp_match(query, 'utm_campaign=([^&]+)'))[1],
        (regexp_match(query, 'utm_content=([^&]+)'))[1],
        (regexp_match(query, 'utm_term=([^&]+)'))[1];
END;
$$ LANGUAGE plpgsql IMMUTABLE;
```

---

## Entity Relationship Diagram

```
┌─────────────────┐
│    projects     │
├─────────────────┤
│ id (PK)         │◀─────────────────┐
│ name            │                  │
│ domain          │                  │
│ color           │                  │
│ created_at      │                  │
└─────────────────┘                  │
                                     │
┌─────────────────┐                  │
│     events      │                  │
├─────────────────┤                  │
│ id (PK)         │                  │
│ timestamp (PK)  │                  │
│ project_id (FK) │──────────────────┤
│ type            │                  │
│ path            │                  │
│ country         │                  │
│ device_type     │                  │
│ utm_*           │                  │
│ ...             │                  │
└─────────────────┘                  │
                                     │
┌─────────────────┐                  │
│   daily_stats   │                  │
├─────────────────┤                  │
│ date (PK)       │                  │
│ project_id (FK) │──────────────────┘
│ page_views      │
│ unique_visitors │
│ ...             │
└─────────────────┘
```

---

## Schema Management

**Initialize/Update Schema:**

```bash
npm run db:push
# or
psql $DATABASE_URL -f lib/schema.sql
```

**File Location:** `lib/schema.sql`
