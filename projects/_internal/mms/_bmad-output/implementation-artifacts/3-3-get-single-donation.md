# Story 3.3: Get Single Donation

Status: done

## Story

As a **project integrator**,
I want to retrieve a specific donation by ID,
so that I can verify or display details of an individual donation record.

## Acceptance Criteria

1. **Given** an authenticated request with donations:read scope **When** GET /v1/donations/:id is called with a valid UUID **Then** the donation is returned in `{ "data": {...} }` envelope with 200 status **And** all fields are included (id, target_id, source_project, amount_cents, currency, donation_type, reference, donor_note, donated_at, created_at, updated_at)

2. **Given** a request with a non-existent ID **When** GET /v1/donations/:id is called **Then** a 404 NOT_FOUND error is returned

3. **Given** a request with an invalid UUID format **When** GET /v1/donations/:id is called **Then** a 400 VALIDATION_ERROR is returned

## Tasks / Subtasks

- [ ] Task 1: Add findDonationById query
- [ ] Task 2: Add handleGetDonation handler with UUID validation
- [ ] Task 3: Add GET /:id route with donations:read scope
- [ ] Task 4: Update exports
- [ ] Task 5: Write tests

## Dev Notes

- Validate UUID format before database query
- Use successResponse() for single item
- Return 404 NOT_FOUND when donation doesn't exist

## Dev Agent Record

### File List

- mms-api/src/services/donations/types.ts (MODIFIED - added donationIdSchema)
- mms-api/src/services/donations/queries.ts (MODIFIED - added findDonationById)
- mms-api/src/services/donations/handlers.ts (MODIFIED - added handleGetDonation)
- mms-api/src/services/donations/routes.ts (MODIFIED - added GET /:id route)
- mms-api/src/services/donations/index.ts (MODIFIED - updated exports)
- mms-api/src/services/donations/handlers.test.ts (MODIFIED - added 6 tests)
