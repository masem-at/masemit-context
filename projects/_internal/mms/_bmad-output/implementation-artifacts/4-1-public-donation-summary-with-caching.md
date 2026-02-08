# Story 4.1: Public Donation Summary with Caching

Status: done

## Story

As a **masem.at visitor**,
I want to see aggregated donation totals per organization on the Verantwortungsseite,
so that I can see how much masemIT has contributed to each cause.

## Acceptance Criteria

1. **Given** no authentication is required **When** GET /v1/donations/public/summary is called **Then** aggregated totals per active donation target are returned with 200 status **And** response format: `{ "data": [...], "updated_at": "ISO8601" }`

2. **Given** the public summary is requested **When** donation targets exist with is_active=false **Then** inactive targets are excluded from the response

3. **Given** the public summary is requested **When** the response is returned **Then** no source_project breakdown is included (only aggregated totals) **And** no internal data (API keys, config) is exposed

4. **Given** lib/cache.ts is created with get/set/del helpers and TTL constants **When** GET /v1/donations/public/summary is called for the first time **Then** the result is fetched from DB, cached in Upstash Redis under key `mms:donations:public:summary` with 15min TTL, and returned

5. **Given** a cached value exists for the public summary **When** GET /v1/donations/public/summary is called within the TTL window **Then** the cached value is returned without DB query

6. **Given** a new donation is created via POST /v1/donations **When** the donation is successfully saved **Then** the cache key `mms:donations:public:summary` is deleted (cache-bust)

7. **Given** the cache helper encounters a Redis connection error **When** a cache read fails **Then** the request falls back to DB query (cache miss behavior, never throws)

## Tasks / Subtasks

- [ ] Task 1: Create lib/cache.ts with Upstash Redis helpers
- [ ] Task 2: Add getPublicDonationsSummary query (no source breakdown, active targets only)
- [ ] Task 3: Create public donations route with handler
- [ ] Task 4: Add caching to public summary endpoint
- [ ] Task 5: Add cache-bust to POST /v1/donations
- [ ] Task 6: Mount public route in app
- [ ] Task 7: Write tests

## Dev Notes

- Use @upstash/redis package
- TTL: 15 minutes (900 seconds)
- Cache key: mms:donations:public:summary
- No auth required for public endpoint
- Fail-open on Redis errors (return DB data)
- Include updated_at in response

## Dev Agent Record

### File List

- mms-api/src/lib/cache.ts (NEW - Upstash Redis helpers)
- mms-api/src/lib/cache.test.ts (NEW - cache constants tests)
- mms-api/src/services/donations/queries.ts (MODIFIED - added getPublicDonationsSummary)
- mms-api/src/services/donations/handlers.ts (MODIFIED - added handleGetPublicSummary, cache-bust on create)
- mms-api/src/services/donations/routes.ts (MODIFIED - added public/summary route)
- mms-api/src/services/donations/index.ts (MODIFIED - updated exports)
- mms-api/src/services/donations/handlers.test.ts (MODIFIED - added 10 public summary tests)
