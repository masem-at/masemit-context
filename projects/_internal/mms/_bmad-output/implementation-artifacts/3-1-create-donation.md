# Story 3.1: Create Donation

Status: done

## Story

As a **project integrator**,
I want to record donations via the API,
so that every donation from my project is centrally tracked in MMS.

## Acceptance Criteria

1. **Given** an authenticated request with donations:write scope **When** POST /v1/donations is called with valid body (target_id, source_project, amount_cents, currency?, donation_type, reference?, donor_note?, donated_at) **Then** a new donation is created and returned in `{ "data": {...} }` envelope with 201 status

2. **Given** a create request **When** target_id references a non-existent or inactive donation target **Then** a 400 VALIDATION_ERROR is returned

3. **Given** a create request **When** amount_cents is not a positive integer **Then** a 400 VALIDATION_ERROR is returned with field-level detail

4. **Given** a create request without currency **When** the donation is created **Then** currency defaults to "EUR"

5. **Given** a valid create request **When** source_project is not a valid enum value (hoki_help, tellingcube, chainsights, manual) **Then** a 400 VALIDATION_ERROR is returned

6. **Given** a create request **When** required fields (target_id, source_project, amount_cents, donation_type, donated_at) are missing **Then** a 400 VALIDATION_ERROR is returned with Zod validation details

## Tasks / Subtasks

- [ ] Task 1: Create donations service structure (types.ts, queries.ts, handlers.ts, routes.ts, index.ts)
- [ ] Task 2: Add Zod schema with validations (positive integer, enums, UUID)
- [ ] Task 3: Add query to check target exists and is active
- [ ] Task 4: Add createDonation query
- [ ] Task 5: Add handler with target validation
- [ ] Task 6: Add POST route with donations:write scope
- [ ] Task 7: Mount routes in main app
- [ ] Task 8: Write tests

## Dev Notes

- source_project enum: hoki_help, tellingcube, chainsights, manual
- donation_type enum: direct, revenue_share
- currency defaults to "EUR"
- Must validate target is active (not just exists)
- Use successResponse with 201 status

## Dev Agent Record

### File List

- mms-api/src/services/donations/types.ts (NEW)
- mms-api/src/services/donations/queries.ts (NEW)
- mms-api/src/services/donations/handlers.ts (NEW)
- mms-api/src/services/donations/routes.ts (NEW)
- mms-api/src/services/donations/index.ts (NEW)
- mms-api/src/services/donations/handlers.test.ts (NEW)
- mms-api/src/index.ts (MODIFIED - added donations routes)
