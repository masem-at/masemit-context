# Architecture - masemIT Analytics

> Generated: 2026-01-28 | Version: 0.2.0

## Executive Summary

**masemIT Analytics** is an internal unified analytics dashboard that aggregates web analytics data from all masemIT projects. Built as a lightweight, serverless Next.js application with NeonDB as the data store.

### Key Characteristics

| Aspect | Description |
|--------|-------------|
| **Type** | Internal Analytics Dashboard |
| **Architecture** | Serverless Web Application |
| **Data Sources** | Self-hosted tracker.js (primary) + Vercel Drains (legacy) |
| **Projects Tracked** | 4 (masem.at, ChainSights, tellingCube, hoki.help) |
| **Users** | Internal (masemIT team only) |

## Technology Stack

| Layer | Technology | Version | Rationale |
|-------|------------|---------|-----------|
| **Runtime** | Node.js | 22.x | LTS, Vercel native |
| **Framework** | Next.js | 16.1.4 | App Router, Server Components, API Routes |
| **Language** | TypeScript | 5.x | Type safety |
| **UI** | React | 19.x | Server Components |
| **Styling** | Tailwind CSS | 3.4 | Utility-first, rapid development |
| **Database** | PostgreSQL | 17 | Via NeonDB serverless |
| **DB Client** | @neondatabase/serverless | 0.10 | HTTP-based, Edge-compatible |
| **Hosting** | Vercel | - | Optimized for Next.js |

## Architecture Pattern

### Serverless Web Application

```
┌──────────────────────────────────────────────────────────────────────────┐
│                           VERCEL PLATFORM                                 │
├──────────────────────────────────────────────────────────────────────────┤
│                                                                           │
│  ┌──────────────────┐                       ┌──────────────────────────┐ │
│  │ Client Projects  │                       │        NeonDB            │ │
│  │ (Browser)        │                       │     (PostgreSQL 17)      │ │
│  │                  │                       │     eu-central-1         │ │
│  │  tracker.js      │──────────────────────▶│                          │ │
│  │  (4 projects)    │    /api/events        │     events table         │ │
│  └──────────────────┘       (POST)          │     (partitioned)        │ │
│                                             │                          │ │
│  ┌──────────────────┐                       │                          │ │
│  │  Vercel Drains   │──────────────────────▶│                          │ │
│  │  (legacy)        │    /api/ingest        │                          │ │
│  └──────────────────┘       (POST)          └────────────┬─────────────┘ │
│                                                          │               │
│  ┌──────────────────┐    ┌─────────────────┐            │               │
│  │     Browser      │◀───│   Dashboard     │◀───────────┘               │
│  │   (Analytics)    │    │  (Server Comp)  │   (Direct SQL)             │
│  └──────────────────┘    └─────────────────┘                            │
└──────────────────────────────────────────────────────────────────────────┘
```

### tracker.js Architecture

```
┌─────────────────────────────────────────────────────────────────────────┐
│                        tracker.js (Client-Side)                          │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│  ┌─────────────┐   ┌─────────────┐   ┌─────────────┐   ┌─────────────┐ │
│  │  Page Load  │──▶│  Owner      │──▶│  Generate   │──▶│  Track      │ │
│  │  Detection  │   │  Exclusion  │   │  IDs (UUID) │   │  pageView   │ │
│  └─────────────┘   │  Check      │   │  session/   │   └──────┬──────┘ │
│                    │  (?owner=)  │   │  device     │          │        │
│                    └─────────────┘   └─────────────┘          │        │
│                                                                │        │
│  ┌─────────────────────────────────────────────────────────────┘        │
│  │                                                                       │
│  │  ┌────────────────┐   ┌────────────────┐   ┌────────────────┐       │
│  └─▶│  sendBeacon()  │──▶│  /api/events   │──▶│    NeonDB      │       │
│     │  text/plain    │   │  CORS enabled  │   │    INSERT      │       │
│     └────────────────┘   └────────────────┘   └────────────────┘       │
│                                                                          │
│  SPA Navigation Hooks:                                                   │
│  • pushState / replaceState (History API)                                │
│  • popstate / hashchange events                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

### Design Decisions

| Decision | Choice | Rationale |
|----------|--------|-----------|
| **Rendering** | Server Components | Direct DB access, no API layer needed |
| **State** | URL-based | Simple, shareable, no client state library |
| **Auth** | None (private) | Internal tool, `noindex` meta tag |
| **Caching** | None | Real-time data preferred |
| **Partitioning** | Quarterly | Balance between query speed and management |

## Data Architecture

### Entity Relationship

```
┌─────────────┐
│  projects   │
├─────────────┤
│ id (PK)     │◀──────┐
│ name        │       │
│ domain      │       │
│ color       │       │
└─────────────┘       │
                      │
┌─────────────┐       │      ┌─────────────┐
│   events    │       │      │ daily_stats │
├─────────────┤       │      ├─────────────┤
│ id          │       │      │ date (PK)   │
│ timestamp   │       │      │ project_id  │───┐
│ project_id  │───────┘      │ page_views  │   │
│ type        │              │ visitors    │   │
│ path        │              │ ...         │   │
│ country     │              └─────────────┘   │
│ device_type │                                │
│ utm_*       │                                │
│ ...         │                                │
└─────────────┘◀───────────────────────────────┘
  (partitioned)

┌──────────────────────┐
│  v_project_overview  │ (VIEW)
├──────────────────────┤
│ Aggregates today &   │
│ 7-day stats per      │
│ project from events  │
└──────────────────────┘
```

### Table Specifications

#### `projects`
| Column | Type | Description |
|--------|------|-------------|
| id | TEXT PK | Internal project identifier |
| name | TEXT | Display name |
| domain | TEXT | Primary domain |
| color | TEXT | Brand color (hex) |

#### `events` (Partitioned)
| Column | Type | Description |
|--------|------|-------------|
| id | BIGSERIAL | Auto-increment ID |
| timestamp | TIMESTAMPTZ | Event time (partition key) |
| project_id | TEXT FK | → projects.id |
| type | TEXT | 'pageView' or 'customEvent' |
| path | TEXT | Page path |
| country/region/city | TEXT | Geo data |
| os_name/browser_name | TEXT | Device info |
| utm_* | TEXT | Campaign tracking |

**Partitions:** Quarterly (events_2025_q4, events_2026_q1, etc.)

#### `daily_stats`
Aggregated daily metrics for historical queries (optional, for optimization).

## API Design

### Endpoints

| Method | Path | Purpose | Auth |
|--------|------|---------|------|
| POST | `/api/events` | Receive tracker.js events | CORS (domain whitelist) |
| GET | `/api/events` | Query events | None |
| POST | `/api/ingest` | Receive Vercel Drain events (legacy) | HMAC-SHA1 |
| GET | `/api/ingest` | Health check | None |
| GET | `/api/projects/[id]` | Get project details | None |
| PATCH | `/api/projects/[id]` | Update project settings | None |

### Security

**tracker.js (/api/events):**
```
┌──────────────┐     ┌─────────────────────────────────┐
│  tracker.js  │────▶│ CORS Domain Whitelist           │
└──────────────┘     │ • www.masem.at                  │
                     │ • chainsights.one               │
                     │ • tellingcube.com               │
                     │ • hoki.help                     │
                     │ • localhost (dev)               │
                     └─────────────────────────────────┘
                                    │
                                    ▼
                     ┌─────────────────────────────────┐
                     │ sendBeacon with text/plain      │
                     │ (CORS-safelisted, no preflight) │
                     └─────────────────────────────────┘
```

**Vercel Drains (/api/ingest - legacy):**
```
┌──────────────┐     ┌─────────────────────────────────┐
│ Vercel Drain │────▶│ x-vercel-signature header       │
└──────────────┘     │ HMAC-SHA1(body, DRAIN_SECRET)   │
                     └─────────────────────────────────┘
                                    │
                                    ▼
                     ┌─────────────────────────────────┐
                     │ timingSafeEqual() comparison    │
                     │ Reject with 401 if invalid      │
                     └─────────────────────────────────┘
```

## Component Overview

### Pages

| Route | Component | Data Source |
|-------|-----------|-------------|
| `/` | `page.tsx` | Redirect to /dashboard |
| `/dashboard` | `page.tsx` | `v_project_overview`, `daily_stats`, `events` |
| `/dashboard/[project]` | `page.tsx` | Project-filtered queries |
| `/dashboard/settings` | `page.tsx` | `projects` table |

### Components

| Component | Location | Purpose |
|-----------|----------|---------|
| `FilterBar` | `app/components/` | Date range selection (1/7/30/90 days + custom) |
| `WorldMap` | `app/components/` | Geographic visualization with react-simple-maps |
| `ProjectSettingsForm` | `app/components/` | Project settings editor |
| `RefreshButton` | `app/components/` | Dashboard data refresh |
| `Favicon` | `app/components/` | Favicon component |

### UI Patterns

- **Dark theme** - `bg-[#0a0a0f]`, white text
- **Cards** - Rounded (`rounded-2xl`), glassmorphism (`bg-white/5`)
- **Project branding** - Gradient backgrounds per project
- **Data tables** - Simple bordered tables
- **Charts** - Native div-based bar charts (no library)
- **World map** - react-simple-maps with d3-scale color interpolation
- **Vienna timezone** - Dashboard queries use Europe/Vienna calendar days

## Deployment Architecture

```
┌─────────────────────────────────────────────────┐
│                    VERCEL                        │
├─────────────────────────────────────────────────┤
│  Git Push ──▶ Build ──▶ Deploy                  │
│                                                  │
│  Environment Variables:                          │
│  - DATABASE_URL (NeonDB)                        │
│  - DRAIN_SECRET                                 │
│  - PROJECT_* (4x Vercel Project IDs)            │
└─────────────────────────────────────────────────┘
                        │
                        ▼
┌─────────────────────────────────────────────────┐
│                   NEONDB                         │
├─────────────────────────────────────────────────┤
│  Region: eu-central-1 (Frankfurt)               │
│  Plan: Launch (free tier)                       │
│  Database: neondb                               │
└─────────────────────────────────────────────────┘
```

## Security Considerations

| Aspect | Implementation |
|--------|----------------|
| **API Auth** | HMAC-SHA1 signature verification |
| **Headers** | X-Frame-Options: DENY, X-Content-Type-Options: nosniff |
| **CORS** | Restricted to vercel.com for /api/ingest |
| **Privacy** | robots: noindex,nofollow |
| **Data** | All stored in EU (GDPR compliant) |
| **IP Privacy** | Optional Vercel setting to hide IPs |

## Performance Considerations

| Aspect | Strategy |
|--------|----------|
| **DB Queries** | Direct Server Component queries, no round-trips |
| **Partitioning** | Quarterly partitions for query efficiency |
| **Indexes** | On project_id, timestamp, path, referrer, country |
| **Caching** | `revalidate = 0` (real-time data) |

## Future Considerations

### Completed Features (v0.2.0)
- ~~Country map visualization~~ - Implemented with react-simple-maps
- ~~Self-hosted tracker.js~~ - Replaced Vercel Analytics dependency

### Potential Improvements
- Add proper charting library (recharts, chart.js)
- Implement data caching strategy
- Add authentication for production use
- CI/CD pipeline with tests
- Error monitoring (Sentry)
- Phase out Vercel Drains completely (legacy)
