# Story 3.2: List Donations with Filtering & Pagination

Status: done

## Story

As a **project integrator**,
I want to query donations with filters and pagination,
so that I can review donation history for my project or specific targets.

## Acceptance Criteria

1. **Given** an authenticated request with donations:read scope **When** GET /v1/donations is called without filters **Then** donations are returned paginated in `{ "data": [...], "meta": { "page": 1, "limit": 50, "total": N } }` envelope

2. **Given** query parameter target_id is provided **When** the list is fetched **Then** only donations for that target are returned

3. **Given** query parameter source_project is provided **When** the list is fetched **Then** only donations from that source project are returned

4. **Given** query parameter donation_type is provided **When** the list is fetched **Then** only donations of that type (direct or revenue_share) are returned

5. **Given** query parameters from and/or to are provided (ISO 8601) **When** the list is fetched **Then** only donations within the date range (based on donated_at) are returned

6. **Given** query parameters page and limit are provided **When** limit exceeds 100 **Then** limit is capped at 100 **And** meta reflects the actual page, limit, and total count

7. **Given** multiple filters are combined **When** the list is fetched **Then** all filters are applied with AND logic

## Tasks / Subtasks

- [ ] Task 1: Add query params schema for filters and pagination
- [ ] Task 2: Add getDonations query with filters and count
- [ ] Task 3: Add handleListDonations handler
- [ ] Task 4: Add GET route with donations:read scope
- [ ] Task 5: Update exports
- [ ] Task 6: Write tests

## Dev Notes

- Default limit: 50, max limit: 100
- Default page: 1
- Filters: target_id, source_project, donation_type, from, to
- Date range filter on donated_at field
- Use listResponse() for paginated output
- Order by donated_at DESC

## Dev Agent Record

### File List

- mms-api/src/services/donations/types.ts (MODIFIED - added listDonationsQuerySchema)
- mms-api/src/services/donations/queries.ts (MODIFIED - added getDonations with filters)
- mms-api/src/services/donations/handlers.ts (MODIFIED - added handleListDonations)
- mms-api/src/services/donations/routes.ts (MODIFIED - added GET route)
- mms-api/src/services/donations/index.ts (MODIFIED - updated exports)
- mms-api/src/services/donations/handlers.test.ts (MODIFIED - added 19 list tests)
