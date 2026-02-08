# Story 3.4: Internal Donation Summary

Status: done

## Story

As an **admin**,
I want an internal summary endpoint with per-source breakdown,
so that I can use it for admin dashboards and accounting across all projects.

## Acceptance Criteria

1. **Given** an authenticated request with donations:read scope **When** GET /v1/donations/summary is called **Then** aggregated totals are returned per target with per-source breakdown **And** response includes: target info, total_donated_cents, currency, donation_count, and breakdown by source_project

2. **Given** donations exist from multiple source projects for the same target **When** the summary is fetched **Then** the breakdown shows separate totals per source_project (e.g., hoki_help: 5000, tellingcube: 3000)

3. **Given** no donations exist **When** the summary is fetched **Then** an empty data array is returned: `{ "data": [] }`

4. **Given** a request without valid authentication **When** GET /v1/donations/summary is called **Then** 401 UNAUTHORIZED is returned

## Tasks / Subtasks

- [ ] Task 1: Add getDonationsSummary query with SQL aggregation
- [ ] Task 2: Add handleGetSummary handler
- [ ] Task 3: Add GET /summary route (before /:id route) with donations:read scope
- [ ] Task 4: Update exports
- [ ] Task 5: Write tests

## Dev Notes

- Route /summary must be defined BEFORE /:id to avoid matching summary as ID
- Aggregate by target_id and source_project
- Group results by target with breakdown array
- Use SQL SUM and COUNT for efficiency

## Dev Agent Record

### File List

- mms-api/src/services/donations/queries.ts (MODIFIED - added getDonationsSummary)
- mms-api/src/services/donations/handlers.ts (MODIFIED - added handleGetSummary)
- mms-api/src/services/donations/routes.ts (MODIFIED - added GET /summary route)
- mms-api/src/services/donations/index.ts (MODIFIED - updated exports)
- mms-api/src/services/donations/handlers.test.ts (MODIFIED - added 7 summary tests)
