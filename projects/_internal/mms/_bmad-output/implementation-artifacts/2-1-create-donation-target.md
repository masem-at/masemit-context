# Story 2.1: Create Donation Target

Status: done

## Story

As an **admin**,
I want to create new donation targets via the API,
so that new organizations can be registered as recipients for donations across all masemIT projects.

## Acceptance Criteria

1. **Given** an authenticated request with config:write scope **When** POST /v1/donation-targets is called with a valid body (name, slug, description?, external_url?, is_active?) **Then** a new donation target is created and returned in `{ "data": {...} }` envelope with 201 status

2. **Given** a valid create request **When** the slug already exists in the database **Then** a 400 VALIDATION_ERROR is returned with message indicating duplicate slug

3. **Given** a create request **When** required fields (name, slug) are missing or invalid **Then** a 400 VALIDATION_ERROR is returned with field-level details from Zod validation

4. **Given** a request without x-api-key or with insufficient scope **When** POST /v1/donation-targets is called **Then** 401 UNAUTHORIZED or 403 FORBIDDEN is returned respectively

5. **Given** is_active is not provided in the request body **When** the donation target is created **Then** is_active defaults to true

## Tasks / Subtasks

- [x] Task 1: Create service structure (AC: #1)
  - [x] 1.1 Create src/services/donation-targets/ directory
  - [x] 1.2 Create types.ts with Zod schemas for create request/response
  - [x] 1.3 Create queries.ts with insert function
  - [x] 1.4 Create handlers.ts with POST handler
  - [x] 1.5 Create routes.ts with route definitions

- [x] Task 2: Implement create endpoint (AC: #1, #5)
  - [x] 2.1 Validate request body with Zod schema
  - [x] 2.2 Insert donation target into database
  - [x] 2.3 Return created target in { data: {...} } envelope with 201
  - [x] 2.4 Default is_active to true if not provided

- [x] Task 3: Handle duplicate slug (AC: #2)
  - [x] 3.1 Check if slug exists before insert
  - [x] 3.2 Return 400 VALIDATION_ERROR if duplicate

- [x] Task 4: Apply auth middleware (AC: #4)
  - [x] 4.1 Apply authMiddleware to route
  - [x] 4.2 Apply requireScope('config:write') to POST endpoint

- [x] Task 5: Mount routes and write tests (AC: #1, #2, #3, #4, #5)
  - [x] 5.1 Mount donation-targets routes in index.ts
  - [x] 5.2 Write handler tests for all acceptance criteria (12 tests)

## Dev Notes

### Architecture Compliance

- **Feature-based structure**: src/services/donation-targets/ with routes.ts, handlers.ts, queries.ts, types.ts
- **Response envelope**: `{ "data": {...} }` for single items, use createdResponse() helper
- **Error handling**: Use apiError() and apiValidationError() from lib/errors.ts
- **Auth**: authMiddleware + requireScope('config:write') for admin endpoints
- **Middleware chain**: CORS → Auth → requireScope → Handler

### Zod Schema

```typescript
import { z } from 'zod'

export const createDonationTargetSchema = z.object({
  name: z.string().min(1).max(255),
  slug: z.string().min(1).max(100).regex(/^[a-z0-9-]+$/),
  description: z.string().optional(),
  external_url: z.string().url().max(500).optional(),
  is_active: z.boolean().default(true),
})
```

### Route Pattern

```typescript
// POST /v1/donation-targets
app.post('/v1/donation-targets', authMiddleware, requireScope('config:write'), createHandler)
```

### Query Pattern

```typescript
// Check for duplicate slug
const existing = await db
  .select({ id: donationTargets.id })
  .from(donationTargets)
  .where(eq(donationTargets.slug, data.slug))
  .limit(1)

if (existing.length > 0) {
  return apiError(c, 'VALIDATION_ERROR', 'Slug already exists')
}

// Insert new target
const [created] = await db
  .insert(donationTargets)
  .values(data)
  .returning()
```

### Previous Story Learnings

- Use createdResponse() from lib/responses.ts for 201 responses
- Zod validation with apiValidationError() for field-level errors
- Auth middleware sets c.get('apiKey') with authenticated key info
- snake_case for all API fields

### References

- [Source: _bmad-output/planning-artifacts/architecture.md#Feature Structure]
- [Source: _bmad-output/planning-artifacts/epics.md#Story 2.1]

## Dev Agent Record

### Agent Model Used
Claude Opus 4.5 (Barry - Quick Flow Solo Dev)

### Debug Log References

### Completion Notes List
- Added JSON parse error handling in handler
- Created barrel export (index.ts) for clean imports
- Feature-based structure: types, queries, handlers, routes

### File List
- src/services/donation-targets/types.ts (new)
- src/services/donation-targets/queries.ts (new)
- src/services/donation-targets/handlers.ts (new)
- src/services/donation-targets/routes.ts (new)
- src/services/donation-targets/index.ts (new)
- src/services/donation-targets/handlers.test.ts (new)
- src/index.ts (modified - mounted routes)

