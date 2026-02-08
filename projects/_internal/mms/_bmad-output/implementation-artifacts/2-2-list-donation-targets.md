# Story 2.2: List Donation Targets

Status: done

## Story

As a **project integrator**,
I want to list all donation targets,
so that I can display available targets or look up target IDs for creating donations.

## Acceptance Criteria

1. **Given** an authenticated request with donations:read scope **When** GET /v1/donation-targets is called **Then** all donation targets are returned in `{ "data": [...] }` envelope with 200 status **And** both active and inactive targets are included

2. **Given** donation targets exist in the database **When** the list is returned **Then** each target includes: id, name, slug, description, external_url, is_active, created_at, updated_at

3. **Given** a request without valid authentication **When** GET /v1/donation-targets is called **Then** 401 UNAUTHORIZED is returned

## Tasks / Subtasks

- [ ] Task 1: Add list query function (AC: #1, #2)
  - [ ] 1.1 Add getAllDonationTargets() to queries.ts
  - [ ] 1.2 Select all fields, order by created_at desc

- [ ] Task 2: Add list handler (AC: #1)
  - [ ] 2.1 Create handleListDonationTargets in handlers.ts
  - [ ] 2.2 Return targets in { data: [...] } envelope

- [ ] Task 3: Add route with auth (AC: #3)
  - [ ] 3.1 Add GET route with authMiddleware
  - [ ] 3.2 Require donations:read scope

- [ ] Task 4: Write tests (AC: #1, #2, #3)
  - [ ] 4.1 Test 401 without auth
  - [ ] 4.2 Test successful list with data envelope
  - [ ] 4.3 Test includes all fields

## Dev Notes

- Uses successResponse() with array data (not listResponse - no pagination for this endpoint)
- donations:read scope (not config:write) - any authenticated project can list

## Dev Agent Record

### Agent Model Used

### File List

