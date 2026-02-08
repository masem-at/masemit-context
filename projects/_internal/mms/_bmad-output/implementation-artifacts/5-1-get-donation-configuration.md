# Story 5.1: Get Donation Configuration

Status: done

## Story

As an **admin**,
I want to query the current revenue-share configuration,
so that I can review which percentages are configured per project and target.

## Acceptance Criteria

1. **Given** an authenticated request with config:read scope **When** GET /v1/donation-config is called **Then** all donation config entries are returned in `{ "data": [...] }` envelope with 200 status **And** each entry includes: id, source_project, target_id, percentage, is_active, valid_from, valid_until, created_at, updated_at

2. **Given** configs exist with different validity periods **When** the list is returned **Then** both active and expired configs are included (valid_until in the past) **And** the is_active flag is independent of validity period

3. **Given** a request without valid authentication or without config:read scope **When** GET /v1/donation-config is called **Then** 401 UNAUTHORIZED or 403 FORBIDDEN is returned respectively

## Tasks / Subtasks

- [ ] Task 1: Create donation-config service structure (types, queries, handlers, routes, index)
- [ ] Task 2: Add getAllDonationConfigs query
- [ ] Task 3: Add handleListDonationConfigs handler
- [ ] Task 4: Add GET route with config:read scope
- [ ] Task 5: Mount routes in app
- [ ] Task 6: Write tests

## Dev Notes

- Uses config:read scope (not donations:read)
- Return all configs including expired ones (valid_until in past)
- is_active flag is separate from validity period
- Order by created_at DESC

## Dev Agent Record

### File List

- mms-api/src/services/donation-config/types.ts (NEW)
- mms-api/src/services/donation-config/queries.ts (NEW)
- mms-api/src/services/donation-config/handlers.ts (NEW)
- mms-api/src/services/donation-config/routes.ts (NEW)
- mms-api/src/services/donation-config/index.ts (NEW)
- mms-api/src/services/donation-config/handlers.test.ts (NEW)
- mms-api/src/index.ts (MODIFIED - added donation-config routes)
