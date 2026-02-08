# Story 1.3: Error Handling & Response Patterns

Status: done

## Story

As a **developer**,
I want standardized error handling and response envelope helpers,
so that all endpoints return consistent, predictable responses following the architecture patterns.

## Acceptance Criteria

1. **Given** lib/errors.ts is created **When** an error response is needed **Then** the apiError() factory produces `{ "error": { "code": "...", "message": "...", "details": [...] } }` **And** typed error codes are available: VALIDATION_ERROR (400), UNAUTHORIZED (401), FORBIDDEN (403), NOT_FOUND (404), RATE_LIMITED (429), INTERNAL_ERROR (500)

2. **Given** lib/errors.ts includes a Zod error formatter **When** a Zod validation fails **Then** the formatter converts Zod issues into the standard error details array with field-level messages

3. **Given** response envelope helpers exist **When** a handler returns a single resource **Then** it is wrapped as `{ "data": {...} }` **When** a handler returns a list **Then** it is wrapped as `{ "data": [...], "meta": { "page": N, "limit": N, "total": N } }`

4. **Given** middleware/cors.ts is created **When** a request comes from masem.at to a public endpoint **Then** CORS headers allow the request **When** a request comes from a known project domain (hoki.help, tellingcube.com, chainsights.one) to an internal endpoint **Then** CORS headers allow the request **When** a request comes from an unknown origin **Then** CORS headers reject the request

## Tasks / Subtasks

- [x] Task 1: Create error response helpers (AC: #1)
  - [x] 1.1 Create src/lib/errors.ts with typed error codes and HTTP status mapping
  - [x] 1.2 Implement apiError() factory returning Hono JSON response with correct status code
  - [x] 1.3 Define all error codes: VALIDATION_ERROR (400), UNAUTHORIZED (401), FORBIDDEN (403), NOT_FOUND (404), RATE_LIMITED (429), INTERNAL_ERROR (500)

- [x] Task 2: Create Zod error formatter (AC: #2)
  - [x] 2.1 Implement formatZodError() converting Zod issues to `{ field, message }` details array
  - [x] 2.2 Integrate with apiError() for VALIDATION_ERROR responses

- [x] Task 3: Create response envelope helpers (AC: #3)
  - [x] 3.1 Implement successResponse() wrapping single resources as `{ data }`
  - [x] 3.2 Implement listResponse() wrapping lists as `{ data, meta: { page, limit, total } }`

- [x] Task 4: Create CORS middleware (AC: #4)
  - [x] 4.1 Create src/middleware/cors.ts using Hono's cors() middleware
  - [x] 4.2 Configure public origins: masem.at (with https://)
  - [x] 4.3 Configure internal origins: hoki.help, tellingcube.com, chainsights.one (with https://)
  - [x] 4.4 Mount CORS middleware in src/index.ts

- [x] Task 5: Write comprehensive tests (AC: #1, #2, #3, #4)
  - [x] 5.1 Create src/lib/errors.test.ts testing all error codes, status mapping, and response format
  - [x] 5.2 Test Zod error formatting with various validation failure scenarios
  - [x] 5.3 Test response envelope helpers for single and list responses
  - [x] 5.4 Create src/middleware/cors.test.ts testing allowed and rejected origins

## Dev Notes

### Architecture Compliance

- **Error helpers in lib/errors.ts** — all errors go through these helpers, handlers NEVER throw raw errors
- **Response envelope pattern** — `{ data }` single, `{ data, meta }` list, `{ error: { code, message, details } }` error
- **CORS in middleware/cors.ts** — different policies for public vs internal endpoints
- **snake_case in all JSON responses** — field names must be snake_case
- **Null handling** — nullable fields included with `null` value, never omitted

### Error Code to HTTP Status Mapping

| Error Code | HTTP Status | Usage |
|---|---|---|
| VALIDATION_ERROR | 400 | Invalid input, Zod failures |
| UNAUTHORIZED | 401 | Missing/invalid API key |
| FORBIDDEN | 403 | Valid key but insufficient scope |
| NOT_FOUND | 404 | Resource doesn't exist |
| RATE_LIMITED | 429 | Rate limit exceeded |
| INTERNAL_ERROR | 500 | Unexpected server errors |

### CORS Configuration

- **Public endpoints** (`/health`, `/v1/donations/public/*`): Allow `https://masem.at`
- **Internal endpoints** (all others): Allow `https://hoki.help`, `https://tellingcube.com`, `https://chainsights.one`
- **Rejected origins**: Return no CORS headers (browser blocks request)

### Zod Error Format Example

```typescript
// Zod issues: [{ path: ['amount_cents'], message: 'Expected number' }]
// Formatted: [{ field: 'amount_cents', message: 'Expected number' }]
```

### Previous Story Learnings (Story 1.1 + 1.2)

- ESM with `"type": "module"`, exact pinned versions
- Vitest with mocked DB module for endpoint tests
- Health check now returns 503 on DB failure
- Drizzle schema with 4 tables, 3 enums fully defined
- `.gitkeep` files for empty directories (middleware/, services/donations/, lib/)

### References

- [Source: _bmad-output/planning-artifacts/architecture.md#API & Communication Patterns]
- [Source: _bmad-output/planning-artifacts/architecture.md#Process Patterns]
- [Source: _bmad-output/planning-artifacts/architecture.md#Naming Patterns]
- [Source: _bmad-output/planning-artifacts/epics.md#Story 1.3]

## Dev Agent Record

### Agent Model Used
Claude Opus 4.5

### Debug Log References
- 59/59 Tests bestanden

### Completion Notes List
- Alle 6 Error Codes implementiert (VALIDATION_ERROR, UNAUTHORIZED, FORBIDDEN, NOT_FOUND, RATE_LIMITED, INTERNAL_ERROR)
- apiError() Factory mit korrektem HTTP Status Mapping
- formatZodError() konvertiert Zod Issues zu field/message Details
- successResponse(), listResponse(), createdResponse() Envelope Helpers
- CORS Middleware mit Public (masem.at) und Internal (hoki.help, tellingcube.com, chainsights.one) Origins
- Helper Functions für zukünftige Route-spezifische CORS Policies
- Code Review: 0 kritische, 0 medium, 0 low Issues

### File List
- src/lib/errors.ts (neu)
- src/lib/errors.test.ts (neu)
- src/lib/responses.ts (neu)
- src/lib/responses.test.ts (neu)
- src/middleware/cors.ts (neu)
- src/middleware/cors.test.ts (neu)
- src/index.ts (aktualisiert - CORS Middleware eingebunden)

