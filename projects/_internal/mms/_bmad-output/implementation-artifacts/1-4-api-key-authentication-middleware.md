# Story 1.4: API Key Authentication Middleware

Status: done

## Story

As a **project integrator**,
I want API key authentication with scoped permissions,
so that only authorized projects can access internal MMS endpoints with the correct privileges.

## Acceptance Criteria

1. **Given** middleware/auth.ts is created **When** an internal endpoint is called without an x-api-key header **Then** a 401 UNAUTHORIZED error response is returned

2. **Given** a request includes an x-api-key header **When** the key is SHA-256 hashed and looked up in the api_keys table **Then** a matching active key grants access **And** an inactive or missing key returns 401 UNAUTHORIZED

3. **Given** a valid API key is authenticated **When** the endpoint requires a specific scope (e.g., donations:write) **Then** the middleware checks the key's scopes array **And** returns 403 FORBIDDEN if the required scope is missing

4. **Given** a valid API key is used successfully **When** the request completes **Then** the key's last_used_at timestamp is updated in the database

5. **Given** a key has an expires_at value **When** the current time is past expires_at **Then** the key is treated as inactive and returns 401 UNAUTHORIZED

## Tasks / Subtasks

- [x] Task 1: Create auth middleware (AC: #1, #2, #5)
  - [x] 1.1 Create src/middleware/auth.ts with authMiddleware factory
  - [x] 1.2 Extract x-api-key header from request
  - [x] 1.3 Return 401 UNAUTHORIZED if header is missing
  - [x] 1.4 SHA-256 hash the key and lookup in api_keys table
  - [x] 1.5 Return 401 UNAUTHORIZED if key not found, is_active=false, or expired
  - [x] 1.6 Store authenticated key info in Hono context for handlers

- [x] Task 2: Implement scope checking (AC: #3)
  - [x] 2.1 Create requireScope() middleware factory that takes required scope(s)
  - [x] 2.2 Check if authenticated key's scopes array contains required scope
  - [x] 2.3 Return 403 FORBIDDEN if scope is missing

- [x] Task 3: Update last_used_at timestamp (AC: #4)
  - [x] 3.1 After successful authentication, update last_used_at in api_keys table
  - [x] 3.2 Use fire-and-forget pattern (don't block response)

- [x] Task 4: Write comprehensive tests (AC: #1, #2, #3, #4, #5)
  - [x] 4.1 Create src/middleware/auth.test.ts
  - [x] 4.2 Test missing x-api-key header returns 401
  - [x] 4.3 Test invalid key returns 401
  - [x] 4.4 Test inactive key returns 401
  - [x] 4.5 Test expired key returns 401
  - [x] 4.6 Test valid key grants access
  - [x] 4.7 Test missing scope returns 403
  - [x] 4.8 Test valid scope grants access
  - [x] 4.9 Test last_used_at is updated

## Dev Notes

### Architecture Compliance

- **Auth middleware in middleware/auth.ts** — extracts x-api-key, validates against DB, enforces scopes
- **Middleware chain order** — CORS → Rate Limit (public) → Auth (internal) → Handler
- **Error responses via lib/errors.ts** — use apiError() for 401/403 responses
- **SHA-256 hashing** — use Node.js crypto module (Web Crypto API)
- **Scopes format** — array of strings: `donations:read`, `donations:write`, `config:read`, `config:write`

### API Key Format

- Key format: `mms_{project}_{random}` (e.g., `mms_hoki_help_abc123xyz`)
- Stored as SHA-256 hash in `key_hash` column
- `key_prefix` stores first 12 chars for identification without exposing full key

### Auth Flow

```
Request
  ↓
Extract x-api-key header
  ↓
Missing? → 401 UNAUTHORIZED
  ↓
SHA-256 hash the key
  ↓
Lookup in api_keys table by key_hash
  ↓
Not found? → 401 UNAUTHORIZED
  ↓
is_active = false? → 401 UNAUTHORIZED
  ↓
expires_at < now()? → 401 UNAUTHORIZED
  ↓
Store key in context
  ↓
Update last_used_at (fire-and-forget)
  ↓
Continue to next middleware/handler
```

### Scope Check Flow

```
requireScope('donations:write')
  ↓
Get authenticated key from context
  ↓
Check if scopes array includes 'donations:write'
  ↓
Missing? → 403 FORBIDDEN
  ↓
Continue to handler
```

### Hono Context Variables

```typescript
// Set by authMiddleware
c.set('apiKey', {
  id: string,
  project: string,
  scopes: string[],
})

// Access in handlers
const apiKey = c.get('apiKey')
```

### Database Query Pattern

```typescript
// Lookup by key_hash
const result = await db
  .select()
  .from(apiKeys)
  .where(eq(apiKeys.keyHash, hashedKey))
  .limit(1)
```

### Previous Story Learnings (Story 1.1 + 1.2 + 1.3)

- ESM with `"type": "module"`, exact pinned versions
- Vitest with mocked DB module for tests
- Health check returns 503 on DB failure
- Error helpers: apiError(), formatZodError(), apiValidationError()
- Response helpers: successResponse(), listResponse(), createdResponse()
- CORS middleware already mounted globally

### References

- [Source: _bmad-output/planning-artifacts/architecture.md#Authentication & Security]
- [Source: _bmad-output/planning-artifacts/architecture.md#Middleware Chain Order]
- [Source: _bmad-output/planning-artifacts/epics.md#Story 1.4]

## Dev Agent Record

### Agent Model Used

### Debug Log References

### Completion Notes List

### File List

