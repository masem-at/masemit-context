---
stepsCompleted: [1, 2, 3, 4]
lastStep: 4
status: 'complete'
completedAt: '2026-02-02'
inputDocuments:
  - _bmad-output/_masemIT/requirements/mms-donations-requirement.md
  - _bmad-output/planning-artifacts/architecture.md
workflowType: 'epics-and-stories'
project_name: 'mms'
user_name: 'Sempre'
date: '2026-02-02'
---

# MMS - Epic Breakdown

## Overview

This document provides the complete epic and story breakdown for MMS, decomposing the requirements from the Requirements Document and Architecture into implementable stories.

## Requirements Inventory

### Functional Requirements

FR1: Create donation targets (POST /v1/donation-targets) with name, slug, description, external_url, is_active
FR2: List donation targets (GET /v1/donation-targets)
FR3: Update donation targets (PATCH /v1/donation-targets/:id) — partial update of any field
FR4: Create donations (POST /v1/donations) with target_id, source_project, amount_cents, currency, donation_type, reference, donor_note, donated_at
FR5: List donations with filtering and pagination (GET /v1/donations) — filter by target_id, source_project, donation_type, date range (from/to); paginated with page/limit (default 50, max 100)
FR6: Get single donation by ID (GET /v1/donations/:id)
FR7: Get internal donation summary with per-source breakdown (GET /v1/donations/summary) — authenticated, for admin/accounting
FR8: Get public donation summary aggregated per active target (GET /v1/donations/public/summary) — no auth, no source breakdown, cached 15min TTL
FR9: Get donation config (GET /v1/donation-config) — revenue-share percentages per project/target
FR10: Update donation config per project (PUT /v1/donation-config/:project) — set percentage, target, validity period
FR11: API key authentication via x-api-key header with SHA-256 hash validation against api_keys table
FR12: Scope-based authorization per API key (donations:read, donations:write, config:read, config:write)
FR13: Health check endpoint (GET /health) verifying DB + Redis connectivity
FR14: Cache invalidation — bust public summary cache key on POST /v1/donations
FR15: Seed data — HoKi NÖ as first donation target, generate API keys for all projects (hoki_help, tellingcube, chainsights, masem_at, admin)

### NonFunctional Requirements

NFR1: Performance — Public endpoint response < 200ms (cached via Upstash Redis)
NFR2: Performance — Internal endpoints < 500ms including Vercel Fluid Compute cold start (~115ms)
NFR3: Rate limiting — Public endpoints: 100 req/min sliding window via Upstash Ratelimit
NFR4: CORS — Public endpoints allow masem.at only; Internal endpoints restrict to known project domains (hoki.help, tellingcube.com, chainsights.one)
NFR5: GDPR — NeonDB Frankfurt region (eu-central-1), no personal data exposed on public endpoints
NFR6: Data integrity — Amounts stored as integer cents (no floating point), ISO 4217 currency codes
NFR7: Security — API keys stored as SHA-256 hashes only, plaintext shown once at generation
NFR8: Extensibility — Adding a new service requires only new files in src/services/ and route mounting in index.ts, no framework changes
NFR9: Cost — Zero incremental monthly cost, all within existing Vercel Pro, NeonDB Launch, Upstash Free tiers

### Additional Requirements

- Starter template: `npm create hono@latest mms-api -- --template vercel` as first implementation step
- Pinned dependencies: drizzle-orm@0.45.1, @neondatabase/serverless@1.0.2, @upstash/redis@1.36.1, @upstash/ratelimit@2.0.8, zod, resend, vitest (dev)
- Migration workflow: `drizzle-kit generate` → SQL files committed to Git → `drizzle-kit migrate` in production
- PostgreSQL enums via Drizzle `pgEnum()` for source_project_enum and donation_type_enum
- snake_case consistently across all layers: DB columns, API response fields, query parameters
- Feature-based project structure: src/services/{name}/ with routes.ts, handlers.ts, queries.ts, types.ts
- Co-located tests: {name}.test.ts next to source files, no separate test directories
- Middleware chain order: CORS → Rate Limit (public) → Auth (internal) → Handler
- Response envelope: `{ data, meta }` for lists, `{ data }` for single items, `{ error: { code, message, details } }` for errors
- Cache key pattern: `mms:{service}:{scope}:{identifier}` (e.g., `mms:donations:public:summary`)
- CI/CD: Vercel Git Integration — auto-deploy on push to main, Preview Deployments for branches
- Environment variables: DATABASE_URL, UPSTASH_REDIS_REST_URL, UPSTASH_REDIS_REST_TOKEN (via Vercel encrypted env vars)
- updated_at must be explicitly set to `new Date()` in queries.ts on every UPDATE (no DB trigger)
- API key generation via db/seed.ts with one-time plaintext output to console
- Single schema.ts file for all table definitions — never split across services
- Absolute imports from src/ root — no relative ../../ paths

### FR Coverage Map

| FR | Epic | Description |
|----|------|-------------|
| FR1 | Epic 2 | Create donation targets |
| FR2 | Epic 2 | List donation targets |
| FR3 | Epic 2 | Update donation targets |
| FR4 | Epic 3 | Create donations |
| FR5 | Epic 3 | List donations with filtering/pagination |
| FR6 | Epic 3 | Get single donation by ID |
| FR7 | Epic 3 | Internal donation summary with source breakdown |
| FR8 | Epic 4 | Public donation summary (cached, no auth) |
| FR9 | Epic 5 | Get donation config |
| FR10 | Epic 5 | Update donation config per project |
| FR11 | Epic 1 | API key authentication via x-api-key header |
| FR12 | Epic 1 | Scope-based authorization |
| FR13 | Epic 1 | Health check endpoint (DB + Redis) |
| FR14 | Epic 4 | Cache invalidation on POST /v1/donations |
| FR15 | Epic 1 | Seed data (HoKi target + API keys for all projects) |

## Epic List

### Epic 1: Service Foundation & Authentication
Projects can securely connect to the MMS API. The health check confirms the service is operational. This epic establishes the foundation for all subsequent epics: project scaffold, Vercel deployment, database schema with migrations, authentication middleware, error handling patterns, and seed data.
**FRs covered:** FR11, FR12, FR13, FR15

### Epic 2: Donation Target Management
Admins can create, list, and update donation targets (organizations that receive donations) — the prerequisite for recording any donation.
**FRs covered:** FR1, FR2, FR3

### Epic 3: Donation Recording & Tracking
Projects can submit donations and query their donation history with filtering, pagination, and detailed internal summary with per-source breakdown.
**FRs covered:** FR4, FR5, FR6, FR7

### Epic 4: Public Donation Transparency
masem.at can display live aggregated donation data on the Verantwortungsseite — cached, rate-limited, no source breakdown. Public CORS for masem.at only.
**FRs covered:** FR8, FR14

### Epic 5: Revenue Share Configuration
Admins can configure and query revenue-share percentages per project and donation target, enabling automated donation tracking based on net revenue.
**FRs covered:** FR9, FR10

---

## Epic 1: Service Foundation & Authentication

Projects can securely connect to the MMS API. The health check confirms the service is operational. This epic establishes the foundation for all subsequent epics: project scaffold, Vercel deployment, database schema with migrations, authentication middleware, error handling patterns, and seed data.

### Story 1.1: Project Scaffold & Initial Deployment

As a **developer**,
I want a fully scaffolded Hono project deployed to Vercel with a working health check,
So that the MMS API is live at api.masem.at and ready for feature development.

**Acceptance Criteria:**

**Given** no project exists yet
**When** the Hono Vercel template is initialized with `npm create hono@latest mms-api -- --template vercel`
**Then** the project structure is created with src/index.ts as entry point
**And** all dependencies are installed at pinned versions (drizzle-orm@0.45.1, @neondatabase/serverless@1.0.2, @upstash/redis@1.36.1, @upstash/ratelimit@2.0.8, zod, resend, vitest)

**Given** the project is scaffolded
**When** `GET /health` is called
**Then** a 200 response with `{ "status": "ok" }` is returned

**Given** the project is configured with vercel.json and tsconfig.json
**When** pushed to the main branch
**Then** Vercel auto-deploys and api.masem.at responds to requests

**Given** the project is deployed
**When** a developer checks the repository
**Then** .env.example lists all required environment variables (DATABASE_URL, UPSTASH_REDIS_REST_URL, UPSTASH_REDIS_REST_TOKEN)
**And** .gitignore excludes node_modules, .env, .vercel

### Story 1.2: Database Schema & Migration Pipeline

As a **developer**,
I want the complete database schema defined and migrated to NeonDB,
So that all tables are ready for the application and the migration workflow is established.

**Acceptance Criteria:**

**Given** NeonDB is provisioned in Frankfurt (eu-central-1)
**When** db/index.ts is created with Drizzle ORM + Neon HTTP driver
**Then** the database client connects successfully via DATABASE_URL

**Given** the Drizzle client is configured
**When** db/schema.ts is created
**Then** it defines all 4 tables (donation_targets, donations, donation_config, api_keys) with correct columns, types, and constraints
**And** it defines 2 PostgreSQL enums via pgEnum() (source_project_enum, donation_type_enum)
**And** all tables use UUID primary keys with gen_random_uuid() default
**And** all tables include timestamptz created_at with defaultNow()
**And** mutable tables include timestamptz updated_at

**Given** the schema is defined
**When** `npx drizzle-kit generate` is run
**Then** SQL migration files are generated in the drizzle/ folder
**And** `npx drizzle-kit migrate` applies the migration to NeonDB successfully

**Given** the migration is applied
**When** drizzle.config.ts is inspected
**Then** it points to the correct schema file and NeonDB connection

**Given** the database is connected
**When** `GET /health` is called
**Then** the response includes DB connectivity status: `{ "status": "ok", "db": "connected" }`

### Story 1.3: Error Handling & Response Patterns

As a **developer**,
I want standardized error handling and response envelope helpers,
So that all endpoints return consistent, predictable responses following the architecture patterns.

**Acceptance Criteria:**

**Given** lib/errors.ts is created
**When** an error response is needed
**Then** the apiError() factory produces `{ "error": { "code": "...", "message": "...", "details": [...] } }`
**And** typed error codes are available: VALIDATION_ERROR (400), UNAUTHORIZED (401), FORBIDDEN (403), NOT_FOUND (404), RATE_LIMITED (429), INTERNAL_ERROR (500)

**Given** lib/errors.ts includes a Zod error formatter
**When** a Zod validation fails
**Then** the formatter converts Zod issues into the standard error details array with field-level messages

**Given** response envelope helpers exist
**When** a handler returns a single resource
**Then** it is wrapped as `{ "data": {...} }`
**When** a handler returns a list
**Then** it is wrapped as `{ "data": [...], "meta": { "page": N, "limit": N, "total": N } }`

**Given** middleware/cors.ts is created
**When** a request comes from masem.at to a public endpoint
**Then** CORS headers allow the request
**When** a request comes from a known project domain (hoki.help, tellingcube.com, chainsights.one) to an internal endpoint
**Then** CORS headers allow the request
**When** a request comes from an unknown origin
**Then** CORS headers reject the request

### Story 1.4: API Key Authentication Middleware

As a **project integrator**,
I want API key authentication with scoped permissions,
So that only authorized projects can access internal MMS endpoints with the correct privileges.

**Acceptance Criteria:**

**Given** middleware/auth.ts is created
**When** an internal endpoint is called without an x-api-key header
**Then** a 401 UNAUTHORIZED error response is returned

**Given** a request includes an x-api-key header
**When** the key is SHA-256 hashed and looked up in the api_keys table
**Then** a matching active key grants access
**And** an inactive or missing key returns 401 UNAUTHORIZED

**Given** a valid API key is authenticated
**When** the endpoint requires a specific scope (e.g., donations:write)
**Then** the middleware checks the key's scopes array
**And** returns 403 FORBIDDEN if the required scope is missing

**Given** a valid API key is used successfully
**When** the request completes
**Then** the key's last_used_at timestamp is updated in the database

**Given** a key has an expires_at value
**When** the current time is past expires_at
**Then** the key is treated as inactive and returns 401 UNAUTHORIZED

### Story 1.5: Seed Data & API Key Generation

As an **admin**,
I want initial seed data with HoKi NÖ as the first donation target and API keys for all projects,
So that the system is ready for immediate use after deployment.

**Acceptance Criteria:**

**Given** db/seed.ts is created
**When** the seed script is executed
**Then** HoKi NÖ is created as a donation target with: name="HoKi NÖ — Kinderhospizteam", slug="hoki-noe", description, external_url="https://www.hospiz-noe.at/projekte/hospizteam-fuer-jugendliche/", is_active=true

**Given** the seed script runs
**When** API keys are generated for all 5 projects (hoki_help, tellingcube, chainsights, masem_at, admin)
**Then** each key follows the format `mms_{project}_{random}`
**And** keys are stored as SHA-256 hashes in the api_keys table with correct key_prefix
**And** plaintext keys are output to console exactly once

**Given** the seed script generates API keys
**When** scopes are assigned
**Then** hoki_help, tellingcube, chainsights keys get: [donations:read, donations:write]
**And** masem_at key gets: [donations:read]
**And** admin key gets: [donations:read, donations:write, config:read, config:write]

**Given** the seed script has run
**When** the seed is run again
**Then** it does not create duplicate entries (idempotent or skip-if-exists)

---

## Epic 2: Donation Target Management

Admins can create, list, and update donation targets (organizations that receive donations) — the prerequisite for recording any donation.

### Story 2.1: Create Donation Target

As an **admin**,
I want to create new donation targets via the API,
So that new organizations can be registered as recipients for donations across all masemIT projects.

**Acceptance Criteria:**

**Given** an authenticated request with config:write scope
**When** POST /v1/donation-targets is called with a valid body (name, slug, description?, external_url?, is_active?)
**Then** a new donation target is created and returned in `{ "data": {...} }` envelope with 201 status

**Given** a valid create request
**When** the slug already exists in the database
**Then** a 400 VALIDATION_ERROR is returned with message indicating duplicate slug

**Given** a create request
**When** required fields (name, slug) are missing or invalid
**Then** a 400 VALIDATION_ERROR is returned with field-level details from Zod validation

**Given** a request without x-api-key or with insufficient scope
**When** POST /v1/donation-targets is called
**Then** 401 UNAUTHORIZED or 403 FORBIDDEN is returned respectively

**Given** is_active is not provided in the request body
**When** the donation target is created
**Then** is_active defaults to true

### Story 2.2: List Donation Targets

As a **project integrator**,
I want to list all donation targets,
So that I can display available targets or look up target IDs for creating donations.

**Acceptance Criteria:**

**Given** an authenticated request with donations:read scope
**When** GET /v1/donation-targets is called
**Then** all donation targets are returned in `{ "data": [...] }` envelope with 200 status
**And** both active and inactive targets are included

**Given** donation targets exist in the database
**When** the list is returned
**Then** each target includes: id, name, slug, description, external_url, is_active, created_at, updated_at

**Given** a request without valid authentication
**When** GET /v1/donation-targets is called
**Then** 401 UNAUTHORIZED is returned

### Story 2.3: Update Donation Target

As an **admin**,
I want to update existing donation targets,
So that I can correct information, change URLs, or soft-disable targets without deleting data.

**Acceptance Criteria:**

**Given** an authenticated request with config:write scope
**When** PATCH /v1/donation-targets/:id is called with partial body (any of: name, slug, description, external_url, is_active)
**Then** only the provided fields are updated and the full target is returned in `{ "data": {...} }` envelope

**Given** an update request
**When** the target ID does not exist
**Then** a 404 NOT_FOUND error is returned

**Given** an update request with a new slug
**When** the slug already exists on a different target
**Then** a 400 VALIDATION_ERROR is returned indicating duplicate slug

**Given** a valid update
**When** the target is saved
**Then** updated_at is set to the current timestamp

**Given** an update sets is_active to false
**When** the target is queried later
**Then** it still exists in list results but is marked as inactive

---

## Epic 3: Donation Recording & Tracking

Projects can submit donations and query their donation history with filtering, pagination, and detailed internal summary with per-source breakdown.

### Story 3.1: Create Donation

As a **project integrator**,
I want to record donations via the API,
So that every donation from my project is centrally tracked in MMS.

**Acceptance Criteria:**

**Given** an authenticated request with donations:write scope
**When** POST /v1/donations is called with valid body (target_id, source_project, amount_cents, currency?, donation_type, reference?, donor_note?, donated_at)
**Then** a new donation is created and returned in `{ "data": {...} }` envelope with 201 status

**Given** a create request
**When** target_id references a non-existent or inactive donation target
**Then** a 400 VALIDATION_ERROR is returned

**Given** a create request
**When** amount_cents is not a positive integer
**Then** a 400 VALIDATION_ERROR is returned with field-level detail

**Given** a create request without currency
**When** the donation is created
**Then** currency defaults to "EUR"

**Given** a valid create request
**When** source_project is not a valid enum value (hoki_help, tellingcube, chainsights, manual)
**Then** a 400 VALIDATION_ERROR is returned

**Given** a create request
**When** required fields (target_id, source_project, amount_cents, donation_type, donated_at) are missing
**Then** a 400 VALIDATION_ERROR is returned with Zod validation details

### Story 3.2: List Donations with Filtering & Pagination

As a **project integrator**,
I want to query donations with filters and pagination,
So that I can review donation history for my project or specific targets.

**Acceptance Criteria:**

**Given** an authenticated request with donations:read scope
**When** GET /v1/donations is called without filters
**Then** donations are returned paginated in `{ "data": [...], "meta": { "page": 1, "limit": 50, "total": N } }` envelope

**Given** query parameter target_id is provided
**When** the list is fetched
**Then** only donations for that target are returned

**Given** query parameter source_project is provided
**When** the list is fetched
**Then** only donations from that source project are returned

**Given** query parameter donation_type is provided
**When** the list is fetched
**Then** only donations of that type (direct or revenue_share) are returned

**Given** query parameters from and/or to are provided (ISO 8601)
**When** the list is fetched
**Then** only donations within the date range (based on donated_at) are returned

**Given** query parameters page and limit are provided
**When** limit exceeds 100
**Then** limit is capped at 100
**And** meta reflects the actual page, limit, and total count

**Given** multiple filters are combined
**When** the list is fetched
**Then** all filters are applied with AND logic

### Story 3.3: Get Single Donation

As a **project integrator**,
I want to retrieve a specific donation by ID,
So that I can verify or display details of an individual donation record.

**Acceptance Criteria:**

**Given** an authenticated request with donations:read scope
**When** GET /v1/donations/:id is called with a valid UUID
**Then** the donation is returned in `{ "data": {...} }` envelope with 200 status
**And** all fields are included (id, target_id, source_project, amount_cents, currency, donation_type, reference, donor_note, donated_at, created_at, updated_at)

**Given** a request with a non-existent ID
**When** GET /v1/donations/:id is called
**Then** a 404 NOT_FOUND error is returned

**Given** a request with an invalid UUID format
**When** GET /v1/donations/:id is called
**Then** a 400 VALIDATION_ERROR is returned

### Story 3.4: Internal Donation Summary

As an **admin**,
I want an internal summary endpoint with per-source breakdown,
So that I can use it for admin dashboards and accounting across all projects.

**Acceptance Criteria:**

**Given** an authenticated request with donations:read scope
**When** GET /v1/donations/summary is called
**Then** aggregated totals are returned per target with per-source breakdown
**And** response includes: target info, total_donated_cents, currency, donation_count, and breakdown by source_project

**Given** donations exist from multiple source projects for the same target
**When** the summary is fetched
**Then** the breakdown shows separate totals per source_project (e.g., hoki_help: 5000, tellingcube: 3000)

**Given** no donations exist
**When** the summary is fetched
**Then** an empty data array is returned: `{ "data": [] }`

**Given** a request without valid authentication
**When** GET /v1/donations/summary is called
**Then** 401 UNAUTHORIZED is returned

---

## Epic 4: Public Donation Transparency

masem.at can display live aggregated donation data on the Verantwortungsseite — cached, rate-limited, no source breakdown. Public CORS for masem.at only.

### Story 4.1: Public Donation Summary with Caching

As a **masem.at visitor**,
I want to see aggregated donation totals per organization on the Verantwortungsseite,
So that I can see how much masemIT has contributed to each cause.

**Acceptance Criteria:**

**Given** no authentication is required
**When** GET /v1/donations/public/summary is called
**Then** aggregated totals per active donation target are returned with 200 status
**And** response format: `{ "data": [{ "target": { "id", "name", "slug", "description", "external_url" }, "total_donated_cents", "currency", "last_donation_at", "donation_count" }], "updated_at": "ISO8601" }`

**Given** the public summary is requested
**When** donation targets exist with is_active=false
**Then** inactive targets are excluded from the response

**Given** the public summary is requested
**When** the response is returned
**Then** no source_project breakdown is included (only aggregated totals)
**And** no internal data (API keys, config) is exposed

**Given** lib/cache.ts is created with get/set/del helpers and TTL constants
**When** GET /v1/donations/public/summary is called for the first time
**Then** the result is fetched from DB, cached in Upstash Redis under key `mms:donations:public:summary` with 15min TTL, and returned

**Given** a cached value exists for the public summary
**When** GET /v1/donations/public/summary is called within the TTL window
**Then** the cached value is returned without DB query
**And** response time is < 200ms (NFR1)

**Given** a new donation is created via POST /v1/donations
**When** the donation is successfully saved
**Then** the cache key `mms:donations:public:summary` is deleted (cache-bust)
**And** the next public summary request fetches fresh data from DB

**Given** the cache helper encounters a Redis connection error
**When** a cache read fails
**Then** the request falls back to DB query (cache miss behavior, never throws)

### Story 4.2: Rate Limiting for Public Endpoints

As a **service operator**,
I want rate limiting on public endpoints,
So that the API is protected from abuse and excessive traffic.

**Acceptance Criteria:**

**Given** middleware/rate-limit.ts is created using @upstash/ratelimit
**When** a public endpoint receives requests
**Then** a sliding window rate limiter allows up to 100 requests per minute per IP

**Given** a client exceeds 100 requests per minute
**When** the next request arrives
**Then** a 429 RATE_LIMITED error response is returned with the standard error envelope

**Given** rate limiting is configured
**When** internal (authenticated) endpoints are called
**Then** rate limiting is NOT applied to internal endpoints

**Given** the health check endpoint exists
**When** GET /health is called after this story
**Then** the response includes Redis connectivity status: `{ "status": "ok", "db": "connected", "redis": "connected" }`

**Given** Upstash Redis is unreachable
**When** a rate limit check fails
**Then** the request is allowed through (fail-open) to avoid blocking legitimate traffic

---

## Epic 5: Revenue Share Configuration

Admins can configure and query revenue-share percentages per project and donation target, enabling automated donation tracking based on net revenue.

### Story 5.1: Get Donation Configuration

As an **admin**,
I want to query the current revenue-share configuration,
So that I can review which percentages are configured per project and target.

**Acceptance Criteria:**

**Given** an authenticated request with config:read scope
**When** GET /v1/donation-config is called
**Then** all donation config entries are returned in `{ "data": [...] }` envelope with 200 status
**And** each entry includes: id, source_project, target_id, percentage, is_active, valid_from, valid_until, created_at, updated_at

**Given** configs exist with different validity periods
**When** the list is returned
**Then** both active and expired configs are included (valid_until in the past)
**And** the is_active flag is independent of validity period

**Given** a request without valid authentication or without config:read scope
**When** GET /v1/donation-config is called
**Then** 401 UNAUTHORIZED or 403 FORBIDDEN is returned respectively

### Story 5.2: Update Donation Configuration

As an **admin**,
I want to set revenue-share percentages per project and target,
So that automated donation amounts can be calculated based on net revenue.

**Acceptance Criteria:**

**Given** an authenticated request with config:write scope
**When** PUT /v1/donation-config/:project is called with valid body (target_id, percentage, is_active?, valid_from, valid_until?)
**Then** the config entry is created or updated and returned in `{ "data": {...} }` envelope

**Given** a valid config request
**When** percentage is provided
**Then** it is stored as decimal(5,2) (e.g., 3.00 = 3%)
**And** percentage must be between 0.00 and 100.00 or a VALIDATION_ERROR is returned

**Given** a config for the same source_project and target_id already exists and is active
**When** a new config is submitted
**Then** the previous config is deactivated (is_active=false) and the new one is created
**And** this ensures only one active config per project/target combination

**Given** the :project path parameter is not a valid source_project enum value
**When** PUT /v1/donation-config/:project is called
**Then** a 400 VALIDATION_ERROR is returned

**Given** target_id references a non-existent donation target
**When** the config is submitted
**Then** a 400 VALIDATION_ERROR is returned

**Given** valid_until is provided
**When** valid_until is before valid_from
**Then** a 400 VALIDATION_ERROR is returned

**Given** a valid config update
**When** the config is saved
**Then** updated_at is set to the current timestamp
