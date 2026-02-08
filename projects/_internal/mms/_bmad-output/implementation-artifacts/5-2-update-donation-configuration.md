# Story 5.2: Update Donation Configuration

Status: done

## Story

As an **admin**,
I want to set revenue-share percentages per project and target,
so that automated donation amounts can be calculated based on net revenue.

## Acceptance Criteria

1. **Given** an authenticated request with config:write scope **When** PUT /v1/donation-config/:project is called with valid body (target_id, percentage, is_active?, valid_from, valid_until?) **Then** the config entry is created or updated and returned in `{ "data": {...} }` envelope

2. **Given** a valid config request **When** percentage is provided **Then** it is stored as decimal(5,2) (e.g., 3.00 = 3%) **And** percentage must be between 0.00 and 100.00 or a VALIDATION_ERROR is returned

3. **Given** a config for the same source_project and target_id already exists and is active **When** a new config is submitted **Then** the previous config is deactivated (is_active=false) and the new one is created

4. **Given** the :project path parameter is not a valid source_project enum value **When** PUT /v1/donation-config/:project is called **Then** a 400 VALIDATION_ERROR is returned

5. **Given** target_id references a non-existent donation target **When** the config is submitted **Then** a 400 VALIDATION_ERROR is returned

6. **Given** valid_until is provided **When** valid_until is before valid_from **Then** a 400 VALIDATION_ERROR is returned

7. **Given** a valid config update **When** the config is saved **Then** updated_at is set to the current timestamp

## Tasks / Subtasks

- [ ] Task 1: Add upsert schema with validation (percentage range, dates)
- [ ] Task 2: Add upsertDonationConfig query (deactivate existing, create new)
- [ ] Task 3: Add handleUpsertDonationConfig handler with validations
- [ ] Task 4: Add PUT /:project route with config:write scope
- [ ] Task 5: Update exports
- [ ] Task 6: Write tests

## Dev Notes

- source_project from path param must match enum: hoki_help, tellingcube, chainsights, manual
- Percentage: 0.00-100.00 with decimal(5,2)
- Validate target_id exists
- Deactivate previous active config for same project/target
- valid_until must be >= valid_from if provided

## Dev Agent Record

### File List

- mms-api/src/services/donation-config/types.ts (MODIFIED - added upsertDonationConfigSchema)
- mms-api/src/services/donation-config/queries.ts (MODIFIED - added findTargetById, deactivateExistingConfig, createDonationConfig)
- mms-api/src/services/donation-config/handlers.ts (MODIFIED - added handleUpsertDonationConfig)
- mms-api/src/services/donation-config/routes.ts (MODIFIED - added PUT /:project route)
- mms-api/src/services/donation-config/index.ts (MODIFIED - updated exports)
- mms-api/src/services/donation-config/handlers.test.ts (MODIFIED - added 13 PUT tests)
