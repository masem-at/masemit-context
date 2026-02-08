# Story 2.3: Update Donation Target

Status: done

## Story

As an **admin**,
I want to update existing donation targets,
so that I can correct information, change URLs, or soft-disable targets without deleting data.

## Acceptance Criteria

1. **Given** an authenticated request with config:write scope **When** PATCH /v1/donation-targets/:id is called with partial body (any of: name, slug, description, external_url, is_active) **Then** only the provided fields are updated and the full target is returned in `{ "data": {...} }` envelope

2. **Given** an update request **When** the target ID does not exist **Then** a 404 NOT_FOUND error is returned

3. **Given** an update request with a new slug **When** the slug already exists on a different target **Then** a 400 VALIDATION_ERROR is returned indicating duplicate slug

4. **Given** a valid update **When** the target is saved **Then** updated_at is set to the current timestamp

5. **Given** an update sets is_active to false **When** the target is queried later **Then** it still exists in list results but is marked as inactive

## Tasks / Subtasks

- [ ] Task 1: Add update types and queries
- [ ] Task 2: Add update handler with partial validation
- [ ] Task 3: Check duplicate slug on different target
- [ ] Task 4: Set updated_at timestamp
- [ ] Task 5: Add PATCH route with auth
- [ ] Task 6: Write tests

## Dev Agent Record

### File List

