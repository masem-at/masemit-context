# Story 4.3: Implement Opt-Out Processing Logic

**Status:** done
**Epic:** 4 - Opt-Out Management System
**Story ID:** 4.3
**Created:** 2025-12-23
**Ultimate Context Engine Analysis:** Complete

---

## Story

**As an** admin (Mario),
**I want to** process opt-out requests and remove DAOs from public rankings,
**So that** we honor the 24-hour removal guarantee.

---

## Acceptance Criteria

### Given-When-Then Format (from Epic 4)

**Given** a pending opt-out request exists
**When** admin marks it as processed
**Then** the system:
  - Updates `daos` table: Sets `is_opted_out = true` for the specified DAO
  - Updates `opt_out_requests` table: Sets `status = 'processed'`, `processed_at = now()`
  - Next leaderboard data fetch automatically excludes opted-out DAOs (query filter)

**And** opted-out DAOs do not appear in `/rankings` page

**And** historical `gvs_snapshots` data retained internally (not deleted)

**And** future weekly GVS calculation jobs skip opted-out DAOs

**And** admin can view pending requests in admin dashboard (Epic 7)

**And** GDPR right to erasure is satisfied within 24 hours (NFR-SEC-10)

**And** audit log records the opt-out processing (FR66)

---

## Critical Developer Context

### MISSION - IMPLEMENT OPT-OUT PROCESSING BACKEND LOGIC

This story implements the **processing logic** that allows an admin to mark a pending opt-out request as processed. When processed, the matching DAO is excluded from public rankings immediately.

**What You're Building:**
1. **Processing Service** - `src/lib/dao/opt-out.ts` - Core processing function
2. **Admin API Route** - `POST /api/admin/opt-out/process` - Authenticated endpoint
3. **Database Transaction** - Atomically update both `daos` and `opt_out_requests` tables
4. **Audit Logging** - FR66 compliance with structured `[AUDIT]` logs

**What Already Exists (REUSE, DO NOT RECREATE):**
- `opt_out_requests` table - Stores pending requests (Story 4.2)
- `daos` table with `isOptedOut` column - `src/lib/db/schema.ts:324`
- Leaderboard query already filters: `WHERE daos.isOptedOut = false` - `src/lib/db/queries/leaderboard.ts:81`
- Validation schemas - `src/lib/validation/opt-out.ts` (Story 3.3)
- Email service - `src/lib/email/index.ts` (for optional processed notification)
- Audit log pattern - `console.log('[AUDIT]...')` (Story 4.2)

**What Epic 7 Will Handle (NOT THIS STORY):**
- Admin dashboard UI to view/filter pending requests
- "Mark as Processed" button UI component
- Admin authentication flow (login page, session management)
- Search/filter by DAO name or email

---

### EXISTING CODE ANALYSIS

#### Database Schema (ALREADY EXISTS)

**File:** `src/lib/db/schema.ts`

```typescript
// Opt-out requests table (Story 4.2)
export const optOutRequests = pgTable('opt_out_requests', {
  id: uuid('id').defaultRandom().primaryKey(),
  daoName: text('dao_name').notNull(),
  contactEmail: text('contact_email').notNull(),
  reason: text('reason'),
  status: optOutStatusEnum('status').default('pending').notNull(),  // 'pending' | 'processed'
  requestedAt: timestamp('requested_at').defaultNow().notNull(),
  processedAt: timestamp('processed_at'),  // NULL until processed
  createdAt: timestamp('created_at').defaultNow().notNull(),
  updatedAt: timestamp('updated_at').defaultNow().notNull(),
}, ...)

// DAOs table with opt-out flag
export const daos = pgTable('daos', {
  id: uuid('id').defaultRandom().primaryKey(),
  slug: text('slug').unique().notNull(),
  name: text('name').notNull(),
  category: daoCategoryEnum('category').notNull(),
  snapshotSpace: text('snapshot_space').notNull(),
  isOptedOut: boolean('is_opted_out').default(false).notNull(),  // <-- TOGGLE THIS
  createdAt: timestamp('created_at').defaultNow().notNull(),
  updatedAt: timestamp('updated_at').defaultNow().notNull(),
}, ...)
```

---

#### Leaderboard Query (ALREADY FILTERS)

**File:** `src/lib/db/queries/leaderboard.ts:81`

```typescript
// Already excludes opted-out DAOs!
const whereConditions = [eq(daos.isOptedOut, false)]
```

**This means:** Once `daos.isOptedOut = true`, the DAO automatically disappears from rankings. No leaderboard changes needed.

---

#### Audit Log Pattern (REUSE FROM 4.2)

**File:** `src/app/api/rankings/opt-out/route.ts`

```typescript
// FR66: Audit log for opt-out request
console.log('[AUDIT] Opt-out request submitted:', {
  requestId: insertedRequest.id,
  daoName,
  contactEmail,
  timestamp: new Date().toISOString(),
})
```

**For Processing:**
```typescript
console.log('[AUDIT] Opt-out request processed:', {
  requestId,
  daoName,
  daoId,           // If matched
  processedBy,     // Admin identifier
  timestamp: new Date().toISOString(),
})
```

---

### ARCHITECTURE DECISIONS

#### 1. DAO Matching Strategy

The opt-out request contains `daoName` (free-text from form), but the `daos` table uses `name` and `slug`. We need a matching strategy:

**Option A (Recommended): Exact Name Match**
```typescript
const dao = await db.query.daos.findFirst({
  where: eq(daos.name, optOutRequest.daoName)
})
```

**Option B: Fuzzy Match with Admin Selection**
- Return list of potential matches
- Admin selects correct DAO from dropdown
- More complex but handles typos

**Recommendation:** Start with Option A (exact match). If no match found, return error asking admin to manually identify the DAO. Epic 7 can add fuzzy matching later.

#### 2. Admin Authentication

**MVP Approach:** Bearer token in Authorization header

```typescript
// src/app/api/admin/opt-out/process/route.ts
const authHeader = request.headers.get('Authorization')
if (authHeader !== `Bearer ${process.env.ADMIN_API_KEY}`) {
  return NextResponse.json({ error: 'Unauthorized' }, { status: 401 })
}
```

**Required env var:** `ADMIN_API_KEY` (secure random string)

#### 3. Transaction Safety

Both tables must be updated atomically:

```typescript
await db.transaction(async (tx) => {
  // 1. Update opt_out_requests status
  await tx.update(optOutRequests)
    .set({ status: 'processed', processedAt: new Date() })
    .where(eq(optOutRequests.id, requestId))

  // 2. Update daos.isOptedOut (if DAO found)
  if (daoId) {
    await tx.update(daos)
      .set({ isOptedOut: true, updatedAt: new Date() })
      .where(eq(daos.id, daoId))
  }
})
```

---

### PREVIOUS STORY INTELLIGENCE

#### From Story 4.2 (Opt-Out Submission)

**Key Learnings:**
- Request stored in `opt_out_requests` with status `pending`
- `daoName` is free-text (2-100 chars, validated)
- `contactEmail` is lowercase, validated email
- Audit logging uses `[AUDIT]` prefix for FR66 compliance
- Email failures handled gracefully (don't block main operation)

**Database Record Example:**
```json
{
  "id": "550e8400-e29b-41d4-a716-446655440000",
  "daoName": "Uniswap",
  "contactEmail": "governance@uniswap.org",
  "reason": "We prefer to manage our own analytics",
  "status": "pending",
  "requestedAt": "2025-12-23T10:30:00Z",
  "processedAt": null
}
```

---

### API ROUTE IMPLEMENTATION PATTERN

**File:** `src/app/api/admin/opt-out/process/route.ts`

```typescript
import { NextRequest, NextResponse } from 'next/server'
import { getDb, isDatabaseConfigured, optOutRequests, daos } from '@/lib/db'
import { eq } from 'drizzle-orm'
import { z } from 'zod'

// Request validation schema
const processOptOutSchema = z.object({
  requestId: z.string().uuid(),
  daoId: z.string().uuid().optional(), // Admin can specify if name doesn't match
})

export async function POST(request: NextRequest) {
  // 1. Check admin authentication
  const authHeader = request.headers.get('Authorization')
  const adminKey = process.env.ADMIN_API_KEY

  if (!adminKey || authHeader !== `Bearer ${adminKey}`) {
    return NextResponse.json(
      { error: { message: 'Unauthorized' } },
      { status: 401 }
    )
  }

  // 2. Check database availability
  if (!isDatabaseConfigured()) {
    return NextResponse.json(
      { error: { message: 'Service temporarily unavailable' } },
      { status: 503 }
    )
  }

  try {
    // 3. Parse and validate request body
    const body = await request.json()
    const parseResult = processOptOutSchema.safeParse(body)

    if (!parseResult.success) {
      return NextResponse.json(
        { error: { message: 'Validation failed', details: parseResult.error.issues } },
        { status: 400 }
      )
    }

    const { requestId, daoId: providedDaoId } = parseResult.data
    const db = getDb()

    // 4. Fetch the opt-out request
    const optOutRequest = await db.query.optOutRequests.findFirst({
      where: eq(optOutRequests.id, requestId)
    })

    if (!optOutRequest) {
      return NextResponse.json(
        { error: { message: 'Opt-out request not found' } },
        { status: 404 }
      )
    }

    if (optOutRequest.status === 'processed') {
      return NextResponse.json(
        { error: { message: 'Request already processed' } },
        { status: 400 }
      )
    }

    // 5. Find matching DAO (by provided ID or name match)
    let matchedDao = null

    if (providedDaoId) {
      matchedDao = await db.query.daos.findFirst({
        where: eq(daos.id, providedDaoId)
      })
    } else {
      // Try exact name match
      matchedDao = await db.query.daos.findFirst({
        where: eq(daos.name, optOutRequest.daoName)
      })
    }

    // 6. Execute transaction
    await db.transaction(async (tx) => {
      // Update opt-out request status
      await tx.update(optOutRequests)
        .set({
          status: 'processed',
          processedAt: new Date(),
          updatedAt: new Date(),
        })
        .where(eq(optOutRequests.id, requestId))

      // Update DAO if matched
      if (matchedDao) {
        await tx.update(daos)
          .set({
            isOptedOut: true,
            updatedAt: new Date(),
          })
          .where(eq(daos.id, matchedDao.id))
      }
    })

    // 7. Audit log (FR66)
    console.log('[AUDIT] Opt-out request processed:', {
      requestId,
      daoName: optOutRequest.daoName,
      daoId: matchedDao?.id ?? null,
      daoMatched: !!matchedDao,
      contactEmail: optOutRequest.contactEmail,
      processedBy: 'admin', // Future: extract from auth token
      timestamp: new Date().toISOString(),
    })

    // 8. Return success response
    return NextResponse.json({
      success: true,
      message: matchedDao
        ? `DAO "${matchedDao.name}" removed from public rankings`
        : 'Request marked as processed (no matching DAO found in database)',
      requestId,
      daoId: matchedDao?.id ?? null,
      daoName: matchedDao?.name ?? optOutRequest.daoName,
      daoMatched: !!matchedDao,
    })

  } catch (error) {
    console.error('Opt-out processing error:', error)
    return NextResponse.json(
      { error: { message: 'Failed to process opt-out request' } },
      { status: 500 }
    )
  }
}
```

---

### OPT-OUT SERVICE (OPTIONAL EXTRACTION)

**File:** `src/lib/dao/opt-out.ts`

If you want to separate business logic from the API route:

```typescript
import { getDb, daos, optOutRequests } from '@/lib/db'
import { eq } from 'drizzle-orm'

export interface ProcessOptOutResult {
  success: boolean
  daoMatched: boolean
  daoId?: string
  daoName?: string
  error?: string
}

export async function processOptOutRequest(
  requestId: string,
  providedDaoId?: string
): Promise<ProcessOptOutResult> {
  const db = getDb()

  // Fetch request
  const optOutRequest = await db.query.optOutRequests.findFirst({
    where: eq(optOutRequests.id, requestId)
  })

  if (!optOutRequest) {
    return { success: false, daoMatched: false, error: 'Request not found' }
  }

  if (optOutRequest.status === 'processed') {
    return { success: false, daoMatched: false, error: 'Already processed' }
  }

  // Find DAO
  let matchedDao = providedDaoId
    ? await db.query.daos.findFirst({ where: eq(daos.id, providedDaoId) })
    : await db.query.daos.findFirst({ where: eq(daos.name, optOutRequest.daoName) })

  // Transaction
  await db.transaction(async (tx) => {
    await tx.update(optOutRequests)
      .set({ status: 'processed', processedAt: new Date(), updatedAt: new Date() })
      .where(eq(optOutRequests.id, requestId))

    if (matchedDao) {
      await tx.update(daos)
        .set({ isOptedOut: true, updatedAt: new Date() })
        .where(eq(daos.id, matchedDao.id))
    }
  })

  return {
    success: true,
    daoMatched: !!matchedDao,
    daoId: matchedDao?.id,
    daoName: matchedDao?.name ?? optOutRequest.daoName,
  }
}
```

---

## Implementation Strategy

### Phase 1: Create Processing Service (20 min)

1. Create `src/lib/dao/opt-out.ts`
2. Implement `processOptOutRequest()` function
3. Handle all edge cases (not found, already processed, no DAO match)
4. Use database transaction for atomicity

### Phase 2: Create Admin API Route (30 min)

1. Create `src/app/api/admin/opt-out/process/route.ts`
2. Implement Bearer token authentication
3. Validate request body with Zod
4. Call processing service
5. Add FR66 audit logging
6. Return proper responses (200, 400, 401, 404, 500, 503)

### Phase 3: Add Environment Variable (5 min)

1. Add `ADMIN_API_KEY` to `.env.local`
2. Add to Vercel environment variables
3. Document in README

### Phase 4: Testing (45 min)

1. Create unit tests for processing service
2. Create API route tests with mocked auth
3. Test scenarios:
   - Valid request with matching DAO
   - Valid request with no matching DAO
   - Already processed request
   - Request not found
   - Unauthorized access
   - Database errors

### Phase 5: Integration Test (30 min)

1. Submit opt-out via form (Story 4.1)
2. Call processing API with curl
3. Verify DAO disappears from leaderboard
4. Verify audit logs

---

## Tasks/Subtasks

### Task 1: Create Processing Service
- [x] Create `src/lib/dao/opt-out.ts`
- [x] Implement `processOptOutRequest()` function
- [x] Handle "not found" case
- [x] Handle "already processed" case
- [x] Handle "no matching DAO" case (still mark request as processed)
- [x] Use database transaction for atomic updates
- [x] Export `ProcessOptOutResult` type

### Task 2: Create Admin API Route
- [x] Create `src/app/api/admin/opt-out/process/route.ts`
- [x] Implement Bearer token authentication from `ADMIN_PASSWORD`
- [x] Add request body validation with Zod schema
- [x] Call processing service and handle result
- [x] Add FR66 audit logging with `[AUDIT]` prefix
- [x] Return proper HTTP status codes and error messages

### Task 3: Environment Configuration
- [x] `ADMIN_PASSWORD` already configured in environment (reused from admin dashboard)
- [x] No new environment variables needed - uses existing ADMIN_PASSWORD

### Task 4: Unit Tests
- [x] Create `tests/unit/lib/dao/opt-out.test.ts`
- [x] Test successful processing with matching DAO
- [x] Test successful processing without matching DAO
- [x] Test already processed request
- [x] Test request not found
- [x] Create `tests/unit/api/admin/opt-out-process.test.ts`
- [x] Test authentication (valid/invalid/missing token)
- [x] Test request validation
- [x] Test error responses

### Task 5: Verification
- [x] Run `npm run build` - verify compilation
- [x] Run `npm run test:unit` - all tests pass
- [x] Test with curl: valid processing request
- [x] Test with curl: unauthorized request
- [x] Verify DAO removed from leaderboard after processing
- [x] Verify audit logs appear in console

### Task 6: Update Story Status
- [x] Mark all tasks complete
- [x] Update status to "done"
- [x] Update sprint-status.yaml to "done"
- [x] Commit with message: `feat(story-4.3): implement opt-out processing logic`

---

## Technical Requirements

### File Structure

```
src/
├── app/
│   └── api/
│       └── admin/
│           └── opt-out/
│               └── process/
│                   └── route.ts        # Admin API route (CREATE)
├── lib/
│   └── dao/
│       └── opt-out.ts                  # Processing service (CREATE)
tests/
└── unit/
    ├── lib/
    │   └── dao/
    │       └── opt-out.test.ts         # Service tests (CREATE)
    └── api/
        └── admin/
            └── opt-out-process.test.ts # API tests (CREATE)
```

### Environment Variables Required

```bash
# Admin authentication - REQUIRED
ADMIN_API_KEY=your-secure-random-key-here  # Generate with: openssl rand -hex 32

# Database - ALREADY CONFIGURED
DATABASE_URL=postgresql://...
```

### API Request/Response Formats

**Request:**
```bash
curl -X POST http://localhost:3000/api/admin/opt-out/process \
  -H "Authorization: Bearer YOUR_ADMIN_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"requestId": "550e8400-e29b-41d4-a716-446655440000"}'
```

**Success Response (200) - DAO Matched:**
```json
{
  "success": true,
  "message": "DAO \"Uniswap\" removed from public rankings",
  "requestId": "550e8400-e29b-41d4-a716-446655440000",
  "daoId": "660e8400-e29b-41d4-a716-446655440001",
  "daoName": "Uniswap",
  "daoMatched": true
}
```

**Success Response (200) - No DAO Match:**
```json
{
  "success": true,
  "message": "Request marked as processed (no matching DAO found in database)",
  "requestId": "550e8400-e29b-41d4-a716-446655440000",
  "daoId": null,
  "daoName": "Unknown DAO",
  "daoMatched": false
}
```

**Unauthorized (401):**
```json
{
  "error": {
    "message": "Unauthorized"
  }
}
```

**Already Processed (400):**
```json
{
  "error": {
    "message": "Request already processed"
  }
}
```

**Not Found (404):**
```json
{
  "error": {
    "message": "Opt-out request not found"
  }
}
```

---

## Definition of Done

- [x] `processOptOutRequest()` function created in `src/lib/dao/opt-out.ts`
- [x] Admin API route created at `POST /api/admin/opt-out/process`
- [x] Bearer token authentication implemented
- [x] Request validation with Zod schema
- [x] Database transaction updates both tables atomically
- [x] Matching DAO has `isOptedOut = true`
- [x] Request has `status = 'processed'` and `processedAt` timestamp
- [x] No DAO match case handled gracefully
- [x] Already processed case returns 400 error
- [x] FR66 audit logging with `[AUDIT]` prefix
- [x] Unit tests for service and API route
- [x] TypeScript compiles without errors
- [x] All tests pass
- [x] Build succeeds (`npm run build`)
- [x] Manual verification with curl
- [x] Leaderboard no longer shows opted-out DAO

---

## Dev Agent Record

### Context Reference
<!-- Ultimate Context Engine analysis completed -->

### Agent Model Used
Claude Opus 4.5

### Debug Log References
- None

### Completion Notes List
- 2025-12-23: Story 4.3 created with Ultimate Context Engine analysis
- Builds on Story 4.2 (opt-out submission) - this is the processing backend
- `daos.isOptedOut` column already exists in schema
- Leaderboard query already filters opted-out DAOs - no UI changes needed
- Simple admin auth via Bearer token (Epic 7 will add proper dashboard)
- DAO matching by exact name; future: fuzzy matching in Epic 7

### File List

**Files to Create:**
- `src/app/api/admin/opt-out/process/route.ts` - Admin API route
- `src/lib/dao/opt-out.ts` - Processing service
- `tests/unit/lib/dao/opt-out.test.ts` - Service unit tests
- `tests/unit/api/admin/opt-out-process.test.ts` - API unit tests

**Files to Modify:**
- `.env.example` - Add ADMIN_API_KEY placeholder

**Files Reused (NOT modified):**
- `src/lib/db/schema.ts` - optOutRequests and daos tables
- `src/lib/db/queries/leaderboard.ts` - Already filters isOptedOut
- `src/lib/validation/opt-out.ts` - Validation patterns

---

## References

**Epic 4 Definition:** `docs/epics.md` - Opt-Out Management System
**Architecture:** `docs/architecture.md` - API patterns, admin auth
**Project Context:** `docs/project_context.md` - Coding standards
**Story 4.2:** `docs/sprint-artifacts/4-2-implement-opt-out-submission-and-email-confirmation.md`
**Database Schema:** `src/lib/db/schema.ts:294-311` (optOutRequests), `src/lib/db/schema.ts:318-329` (daos)
**Leaderboard Query:** `src/lib/db/queries/leaderboard.ts:81` (filters isOptedOut)

---

## Change Log

- **2025-12-23**: Story 4.3 created with Ultimate Context Engine analysis - comprehensive opt-out processing implementation guide

---
