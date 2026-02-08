# Product Requirement: MMS Donations Service (Phase 1)

**Project:** MMS — masemIT MicroServices  
**Author:** Claude (Co-Founder masemIT)  
**Owner:** Mario Semper  
**Date:** 2026-02-02  
**Version:** 1.0  
**Status:** Draft — Ready for BMAD  
**Priority:** High

---

## 1. Executive Summary

MMS (masemIT MicroServices) is the central service layer for all masemIT products. The **Donations Service** is the first microservice to be built on api.masem.at, establishing the foundational architecture that will later host user management, subscriptions, newsletters, consents, and party data.

This service replaces duplicated donation-tracking logic across three projects (hoki.help, tellingCube, ChainSights) with a single, canonical API. It powers the **Verantwortungsseite** on masem.at with live, aggregated donation data.

> **Guiding Principle:** Start small, validate the architecture, scale later. The Donations Service is deliberately simple — but the patterns established here (routing, auth, DB access, caching, deployment) become the template for every future MMS service.

---

## 2. Tech Stack

| Layer | Technology | Rationale |
|-------|-----------|-----------|
| **Framework** | Hono | Ultralight (~14KB), TypeScript-first, zero-config Vercel deployment. Built on Web Standards. Perfect for a pure API without frontend rendering overhead. Native Vercel support since Aug 2025 with Fluid Compute. |
| **Runtime** | Vercel (Node.js) | Pro plan already available. Fluid Compute reduces cold starts to ~115ms. Auto-scaling included. |
| **Database** | NeonDB (PostgreSQL) | Dedicated MMS database, separate from project DBs. Frankfurt region (eu-central-1) for GDPR. Serverless driver for HTTP connections. |
| **ORM** | Drizzle ORM | Lightweight, TypeScript-first, SQL-like syntax. Schema-as-code with migration support via drizzle-kit. Proven combo with Hono + NeonDB. |
| **Caching** | Upstash Redis | Free tier available. Used for public endpoint caching (15min TTL) and rate limiting. |
| **Email** | Resend | Pro plan already available. Used for admin notifications (e.g. donation received alerts). |
| **Domain** | api.masem.at | Custom domain on Vercel. Clean separation from project domains. |

---

## 3. Architecture Overview

### 3.1 Project Structure

```
mms-api/
├── src/
│   ├── index.ts                 # Hono app entry + default export
│   ├── middleware/
│   │   ├── auth.ts              # API key validation
│   │   ├── cors.ts              # CORS configuration
│   │   └── rate-limit.ts        # Upstash rate limiting
│   ├── services/
│   │   └── donations/
│   │       ├── routes.ts        # Hono route definitions
│   │       ├── handlers.ts      # Request handlers
│   │       ├── queries.ts       # Drizzle DB queries
│   │       └── types.ts         # TypeScript types/Zod schemas
│   ├── db/
│   │   ├── index.ts             # Drizzle + Neon client
│   │   ├── schema.ts            # All Drizzle table definitions
│   │   └── seed.ts              # Initial data (HoKi target)
│   └── lib/
│       ├── cache.ts             # Upstash Redis helper
│       └── errors.ts            # Error response helpers
├── drizzle/                     # Generated migrations
├── drizzle.config.ts
├── package.json
├── tsconfig.json
└── .env.example
```

### 3.2 Service Pattern

Each future MMS service follows the same pattern established by Donations:

- **routes.ts** — Hono route definitions with path prefixes
- **handlers.ts** — Request/response logic, input validation via Zod
- **queries.ts** — Drizzle ORM database operations
- **types.ts** — Shared TypeScript types and Zod schemas

New services (users, subscriptions, etc.) plug into `src/services/` following this exact structure. The main `index.ts` composes all service routes into one Hono app.

---

## 4. Data Model

### 4.1 Database Schema (Drizzle)

All tables use UUID primary keys and timestamp tracking.

#### donation_targets

Organizations or causes that receive donations. Currently only HoKi NÖ, but extensible.

| Column | Type | Nullable | Description |
|--------|------|----------|-------------|
| `id` | uuid | No | PK, default gen_random_uuid() |
| `name` | varchar(255) | No | Display name, e.g. "HoKi NÖ — Kinderhospizteam" |
| `slug` | varchar(100) | No | URL-friendly identifier, unique |
| `description` | text | Yes | What the organization does |
| `external_url` | varchar(500) | Yes | Link to organization website |
| `is_active` | boolean | No | Default true. Soft-disable without deleting |
| `created_at` | timestamptz | No | Default now() |
| `updated_at` | timestamptz | No | Auto-updated on change |

#### donations

Individual donation records from all sources.

| Column | Type | Nullable | Description |
|--------|------|----------|-------------|
| `id` | uuid | No | PK, default gen_random_uuid() |
| `target_id` | uuid | No | FK → donation_targets.id |
| `source_project` | enum | No | hoki_help \| tellingcube \| chainsights \| manual |
| `amount_cents` | integer | No | Amount in cents (e.g. 2500 = €25.00) |
| `currency` | varchar(3) | No | Default 'EUR'. ISO 4217 code |
| `donation_type` | enum | No | direct \| revenue_share |
| `reference` | varchar(255) | Yes | External ref, e.g. Stripe Payment ID |
| `donor_note` | text | Yes | Optional note from donor |
| `donated_at` | timestamptz | No | When the donation actually occurred |
| `created_at` | timestamptz | No | Default now() |
| `updated_at` | timestamptz | No | Auto-updated on change |

#### donation_config

Configurable revenue-share percentages per project and target.

| Column | Type | Nullable | Description |
|--------|------|----------|-------------|
| `id` | uuid | No | PK, default gen_random_uuid() |
| `source_project` | enum | No | Which project this config applies to |
| `target_id` | uuid | No | FK → donation_targets.id |
| `percentage` | decimal(5,2) | No | Revenue share %, e.g. 3.00 = 3% |
| `is_active` | boolean | No | Default true |
| `valid_from` | timestamptz | No | Config effective from |
| `valid_until` | timestamptz | Yes | NULL = indefinite |
| `created_at` | timestamptz | No | Default now() |
| `updated_at` | timestamptz | No | Auto-updated on change |

#### api_keys

API keys for service-to-service authentication. One key per project, stored hashed.

| Column | Type | Nullable | Description |
|--------|------|----------|-------------|
| `id` | uuid | No | PK, default gen_random_uuid() |
| `project` | enum | No | hoki_help \| tellingcube \| chainsights \| masem_at \| admin |
| `key_hash` | varchar(255) | No | SHA-256 hash of the API key |
| `key_prefix` | varchar(10) | No | First 8 chars for identification (e.g. mms_hoki_) |
| `name` | varchar(100) | No | Human-readable name |
| `scopes` | text[] | No | Allowed scopes: donations:read, donations:write, config:read, config:write |
| `is_active` | boolean | No | Default true. Revoke without deleting |
| `last_used_at` | timestamptz | Yes | Track last usage |
| `expires_at` | timestamptz | Yes | NULL = never expires |
| `created_at` | timestamptz | No | Default now() |

### 4.2 Indexes

- `donations.target_id` — FK lookups for aggregation queries
- `donations.source_project` — Filter by source
- `donations.donated_at` — Time-range queries
- `donation_targets.slug` — Unique, URL lookups
- `api_keys.key_hash` — Unique, auth lookups
- `donation_config(source_project, target_id)` — Composite unique for active configs

### 4.3 Enums

- **source_project_enum:** `hoki_help | tellingcube | chainsights | manual`
- **donation_type_enum:** `direct | revenue_share`

Note: Enums are defined as PostgreSQL enums via Drizzle's `pgEnum()`. New values require a migration.

---

## 5. API Specification

Base URL: `https://api.masem.at`. All endpoints versioned under `/v1`.

### 5.1 Public Endpoints (No Auth)

Consumed by masem.at for the Verantwortungsseite. No authentication required, rate-limited and cached.

#### GET /v1/donations/public/summary

Returns aggregated donation totals per active target. Cached for 15 minutes via Upstash.

**Response 200:**

```json
{
  "data": [
    {
      "target": {
        "id": "uuid",
        "name": "HoKi NÖ — Kinderhospizteam",
        "slug": "hoki-noe",
        "description": "Begleitung von Familien mit schwer erkrankten Kindern und Jugendlichen",
        "external_url": "https://www.hospiz-noe.at/projekte/hospizteam-fuer-jugendliche/"
      },
      "total_donated_cents": 11000,
      "currency": "EUR",
      "last_donation_at": "2026-01-28T14:30:00Z",
      "donation_count": 15
    }
  ],
  "updated_at": "2026-02-02T10:00:00Z"
}
```

**Key design decision:** No breakdown by source project. The public endpoint shows only total amounts per target. This is intentional — the Verantwortungsseite should not feel promotional.

### 5.2 Internal Endpoints (Authenticated)

All internal endpoints require `x-api-key` header. Keys are scoped per project.

#### Donations

| Method | Path | Scope Required |
|--------|------|---------------|
| `POST` | `/v1/donations` | `donations:write` |
| `GET` | `/v1/donations` | `donations:read` |
| `GET` | `/v1/donations/:id` | `donations:read` |
| `GET` | `/v1/donations/summary` | `donations:read` |

**POST /v1/donations — Request Body:**

```json
{
  "target_id": "uuid",
  "source_project": "hoki_help",
  "amount_cents": 2500,
  "currency": "EUR",
  "donation_type": "direct",
  "reference": "stripe_pi_xxx",
  "donor_note": "Keep up the great work!",
  "donated_at": "2026-02-01T12:00:00Z"
}
```

**GET /v1/donations — Query Parameters:**

- `target_id` — Filter by donation target
- `source_project` — Filter by source
- `donation_type` — Filter: direct | revenue_share
- `from` / `to` — Date range (ISO 8601)
- `page` / `limit` — Pagination (default: page=1, limit=50, max 100)

**GET /v1/donations/summary** — Unlike the public endpoint, this returns per-source breakdown, useful for admin dashboards and accounting.

#### Donation Targets

| Method | Path | Scope Required |
|--------|------|---------------|
| `GET` | `/v1/donation-targets` | `donations:read` |
| `POST` | `/v1/donation-targets` | `config:write` |
| `PATCH` | `/v1/donation-targets/:id` | `config:write` |

#### Donation Configuration

| Method | Path | Scope Required |
|--------|------|---------------|
| `GET` | `/v1/donation-config` | `config:read` |
| `PUT` | `/v1/donation-config/:project` | `config:write` |

### 5.3 Auth & Security

- **Public endpoints:** No auth. Rate limited (100 req/min via Upstash). Cached response (15min TTL).
- **Internal endpoints:** `x-api-key` header required. Key validated against `api_keys` table (SHA-256 hash comparison).
- **Key format:** `mms_{project}_{random}` — e.g. `mms_hoki_a1b2c3d4e5f6`. Prefix allows quick identification.
- **Scope enforcement:** Each key has scoped permissions. A hoki.help key with `[donations:write]` cannot read config.
- **CORS:** Public endpoints allow `masem.at`. Internal endpoints restrict to known project domains.
- **Input validation:** All request bodies validated via Zod schemas. 400 response on invalid input.

### 5.4 Error Responses

```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "amount_cents must be a positive integer",
    "details": [...]
  }
}
```

Standard error codes: `VALIDATION_ERROR` (400), `UNAUTHORIZED` (401), `FORBIDDEN` (403), `NOT_FOUND` (404), `RATE_LIMITED` (429), `INTERNAL_ERROR` (500).

---

## 6. Caching Strategy

| Endpoint | TTL | Invalidation |
|----------|-----|-------------|
| `/v1/donations/public/summary` | 15 min | Auto-expire. Also invalidated on POST /v1/donations (cache bust). |
| Internal list endpoints | No cache | Always fresh data for project integrations. |

Cache key pattern: `mms:donations:public:summary`. On POST /v1/donations, the handler deletes this cache key so the next public request gets fresh data.

---

## 7. Implementation Phases

### Phase 1: Foundation (Target: 1 evening)

Scaffold the project, establish patterns, deploy to Vercel.

1. Initialize Hono project with Vercel template (`npm create hono@latest`)
2. Configure NeonDB connection with Drizzle ORM (neon-http driver)
3. Define Drizzle schema for all 4 tables + enums
4. Run initial migration (`drizzle-kit generate` + `migrate`)
5. Deploy to Vercel, verify `api.masem.at` responds with health check

### Phase 2: Core API (Target: 1–2 evenings)

Build the donation endpoints and auth middleware.

1. Implement API key auth middleware (`x-api-key` header, SHA-256 lookup)
2. Build `POST /v1/donations` with Zod validation
3. Build `GET /v1/donations` with filtering and pagination
4. Build `GET /v1/donations/:id`
5. Build `GET /v1/donations/summary` (internal, with source breakdown)
6. Build `GET /v1/donations/public/summary` (public, cached)
7. Implement Upstash caching + cache invalidation on POST
8. Build Donation Targets CRUD (GET, POST, PATCH)
9. Build Donation Config endpoints (GET, PUT)

### Phase 3: Seed & Test (Target: 1 evening)

Populate with real data and verify end-to-end.

1. Create seed script: HoKi NÖ as first donation target
2. Generate API keys for all projects (hoki_help, tellingcube, chainsights, masem_at)
3. Manually import existing donations (~15–30 records via seed script or POST calls)
4. Test all endpoints via curl/Insomnia/Bruno
5. Verify public endpoint returns correct aggregated data

### Phase 4: Project Integration (Ongoing)

Migrate each project to use MMS instead of local donations tables.

1. hoki.help: Replace local donations DB calls with MMS API calls
2. ChainSights: Same migration
3. tellingCube: Same migration
4. masem.at: Build Verantwortungsseite consuming `/v1/donations/public/summary`
5. Deprecate and eventually remove old donations tables from project DBs

---

## 8. Future MMS Services (Out of Scope)

| Service | Description | Priority |
|---------|-------------|----------|
| **Users / Parties** | Central user + company storage across all projects | High |
| **Subscriptions** | Plan management, billing state, feature flags | Medium |
| **Newsletters** | Cross-project newsletter subscriptions with consent tracking | Medium |
| **Consents** | GDPR-compliant consent management | Medium |
| **Auth Layer** | JWT-based auth replacing per-project API keys | Low (post-MVP) |

---

## 9. Open Decisions

- **Verantwortungsseite history:** Timeline of donations or only cumulative totals? *Recommendation: Totals only. Simpler, less promotional.*
- **Admin UI:** API-only or basic admin page? *Recommendation: API-only for Phase 1. Drizzle Studio for ad-hoc queries.*
- **Revenue share visibility:** Show "3% of net revenue" on Verantwortungsseite? *Recommendation: Yes, factual and understated.*
- **Notification on donation:** Email via Resend when new donation recorded? *Recommendation: Phase 2 nice-to-have.*
- **Crypto donations:** Track Coinbase Commerce donations in MMS? *Recommendation: Yes, same POST endpoint with appropriate reference.*

---

## 10. Success Criteria

- **Functional:** All 4 projects can POST donations to MMS. Public endpoint returns correct aggregated data. masem.at Verantwortungsseite displays live donation totals.
- **Performance:** Public endpoint < 200ms (cached). Internal endpoints < 500ms (cold start with Fluid Compute).
- **Architecture:** Adding a new service requires only adding files to `src/services/` — no framework changes.
- **Data integrity:** Total across MMS matches sum of individual project donation records after migration.
- **Security:** No public endpoint leaks internal data (source breakdown, API keys, config).

---

## 11. Cost Estimate

| Service | Monthly Cost | Notes |
|---------|-------------|-------|
| Vercel Pro | Already included | Part of existing masemIT Pro plan |
| NeonDB | €0 (Launch tier) | 0.5GB storage, auto-suspend |
| Upstash Redis | €0 (Free tier) | 10K commands/day |
| Custom domain | €0 | api.masem.at configured in Vercel |
| **Total incremental** | **€0/month** | All within existing plans and free tiers |
