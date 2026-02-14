# Story 8.1: Multi-Tenant Data Model & Auth Context

Status: done

## Story

As a **Meister**,
I want my data to be isolated per company,
so that each construction business sees only its own projects and messages.

## Acceptance Criteria

1. **Given** the database has existing Sprint 0 tables (users, sessions, magic_links, voice_messages)
   **When** the migration runs
   **Then** a `companies` table exists with id, name, createdAt columns
   **And** the `users` table has a non-nullable `companyId` foreign key referencing companies
   **And** the `users` table has a non-nullable `role` column with default "monteur" and allowed values "monteur", "meister", "buero"
   **And** a seed company "masemIT Testbetrieb" exists and Mario's user is linked to it with role "meister"
   **And** the `projects` table exists with id, companyId, name, customerName, address, description, status, createdAt, updatedAt
   **And** the `project_members` table exists with projectId, userId
   **And** the `voice_messages` table has new nullable columns: projectId, companyId, aiConfidence, assignedBy, aiReasoning, llmPromptVersion, assignedAt, aiSuggestedProjectId
   **And** existing voice_messages have companyId backfilled from their user's company

2. **Given** a user with a valid session
   **When** calling `getAuthContext()`
   **Then** it returns `{ userId, companyId, role }` from the session + user record
   **And** it throws if no valid session exists

3. **Given** the middleware is extended
   **When** a request hits `/app/*` or `/api/voice/*`
   **Then** the middleware validates that the user has a companyId
   **And** the `x-user-id` header injection from Sprint 0 is removed (replaced by getAuthContext)

**FRs:** S1-FR27, S1-FR28, S1-FR29 | **NFRs:** S1-NFR5, S1-NFR6

## Tasks / Subtasks

- [x] Task 1: Create `companies` table migration (AC: #1)
  - [x] 1.1 Add `companies` table to `lib/db/schema.ts` with columns: id (uuid, defaultRandom), name (text, notNull), createdAt (timestamptz, defaultNow)
  - [x] 1.2 Run `pnpm drizzle-kit generate` to create migration SQL
  - [x] 1.3 Run migration against NeonDB with `pnpm drizzle-kit migrate`

- [x] Task 2: Extend `users` table with companyId + role (AC: #1)
  - [x] 2.1 Add `companyId` column (uuid, nullable initially, FK → companies.id)
  - [x] 2.2 Add `role` column (text, notNull, default "monteur") — allowed values: "monteur", "meister", "buero"
  - [x] 2.3 Generate and run migration

- [x] Task 3: Seed test company + backfill Mario's user (AC: #1)
  - [x] 3.1 Create seed migration: INSERT "masemIT Testbetrieb" into companies
  - [x] 3.2 UPDATE Mario's user: set companyId to seed company, role to "meister"
  - [x] 3.3 NOTE: This is a data migration — use raw SQL in the migration file, NOT Drizzle's JS API

- [x] Task 4: Make companyId NOT NULL on users (AC: #1)
  - [x] 4.1 ALTER TABLE users ALTER COLUMN companyId SET NOT NULL
  - [x] 4.2 Generate and run migration
  - [x] 4.3 VERIFY: All existing users have companyId set before this migration

- [x] Task 5: Create `projects` table (AC: #1)
  - [x] 5.1 Add `projects` table to schema.ts: id (uuid), companyId (uuid FK, notNull), name (text, notNull), customerName (text, nullable), address (text, nullable), description (text, nullable), status (text, notNull, default "active"), createdAt (timestamptz), updatedAt (timestamptz)
  - [x] 5.2 Add indexes: idx_projects_company_id on companyId, idx_projects_status on (companyId, status)
  - [x] 5.3 Generate and run migration

- [x] Task 6: Create `project_members` table (AC: #1)
  - [x] 6.1 Add `project_members` table to schema.ts: projectId (uuid FK → projects.id), userId (uuid FK → users.id), composite PK (projectId, userId)
  - [x] 6.2 Generate and run migration

- [x] Task 7: Extend `voice_messages` with assignment columns (AC: #1)
  - [x] 7.1 Add nullable columns to voice_messages: projectId (uuid FK → projects.id), companyId (uuid FK → companies.id), aiConfidence (real), assignedBy (text), aiReasoning (text), llmPromptVersion (text), assignedAt (timestamptz), aiSuggestedProjectId (uuid FK → projects.id)
  - [x] 7.2 Add indexes: idx_voice_messages_company_id, idx_voice_messages_project_id, idx_voice_messages_assignment (companyId, projectId, status) — compound index for Inbox queries
  - [x] 7.3 Backfill: UPDATE voice_messages SET companyId = (SELECT companyId FROM users WHERE users.id = voice_messages.userId) — include in migration SQL
  - [x] 7.4 Generate and run migration

- [x] Task 8: Implement `getAuthContext()` helper (AC: #2)
  - [x] 8.1 Extend `lib/auth/session.ts` with `getAuthContext()` function
  - [x] 8.2 Function reads session cookie via `cookies()` from `next/headers`
  - [x] 8.3 Validates session token against DB (reuse existing `validateSession` logic)
  - [x] 8.4 Joins users table to fetch companyId + role
  - [x] 8.5 Returns typed `AuthContext = { userId: string; companyId: string; role: "monteur" | "meister" | "buero" }`
  - [x] 8.6 Throws descriptive error if no valid session (caught by error boundary)

- [x] Task 9: Update middleware — remove x-user-id, validate companyId (AC: #3)
  - [x] 9.1 Remove `x-user-id` header injection from middleware.ts
  - [x] 9.2 Add companyId existence check: if user has no companyId → redirect to error page or /login
  - [x] 9.3 Middleware continues to protect `/app/*` and `/api/voice/*` routes
  - [x] 9.4 NOTE: Middleware only validates session exists + user has companyId. It does NOT inject headers — all context via `getAuthContext()`

- [x] Task 10: Update `POST /api/voice` route to use getAuthContext() (AC: #3)
  - [x] 10.1 Replace `request.headers.get("x-user-id")` with `const { userId, companyId } = await getAuthContext()`
  - [x] 10.2 Add `companyId` to the voice_messages INSERT
  - [x] 10.3 Verify all existing functionality still works (STT + LLM pipeline unchanged)

- [x] Task 11: Update app layout to use getAuthContext() (AC: #3)
  - [x] 11.1 Replace `headers().get("x-user-id")` in `app/(app)/layout.tsx` with `getAuthContext()`
  - [x] 11.2 User name display continues to work
  - [x] 11.3 Update any other files that read `x-user-id` header

- [x] Task 12: Update schema type exports (AC: #1)
  - [x] 12.1 Export types: Company, NewCompany, Project, NewProject, ProjectMember
  - [x] 12.2 Update VoiceMessage type to include new assignment columns
  - [x] 12.3 Add AuthContext type export from session.ts

- [x] Task 13: Verify + test (all ACs)
  - [x] 13.1 Run `pnpm build` — zero TypeScript errors
  - [x] 13.2 Verify magic link login still works end-to-end
  - [x] 13.3 Verify voice recording + STT + LLM pipeline still works
  - [x] 13.4 Verify dashboard shows existing voice messages (companyId backfill correct)
  - [x] 13.5 Verify getAuthContext() returns correct { userId, companyId, role } for Mario's account

## Dev Notes

### Critical Architecture Patterns

**1. Migration Sequence — MUST follow this exact order:**
```
Migration 001: Create companies table
Migration 002: Add companyId (nullable) + role to users
Migration 003: Seed company "masemIT Testbetrieb", backfill Mario user
Migration 004: ALTER users SET companyId NOT NULL
Migration 005: Create projects table (with companyId NOT NULL)
Migration 006: Create project_members table
Migration 007: Add assignment columns to voice_messages (all nullable)
```
[Source: architecture-sprint1.md#Decision 4]

**2. getAuthContext() replaces x-user-id header injection:**
```typescript
// lib/auth/session.ts — EXTEND existing module
interface AuthContext {
  userId: string;
  companyId: string;
  role: "monteur" | "meister" | "buero";
}

export async function getAuthContext(): Promise<AuthContext> {
  // 1. Read session cookie (next/headers)
  // 2. Validate session against DB (existing validateSession)
  // 3. Join users table for companyId + role
  // 4. Throw if no valid session
}
```
[Source: architecture-sprint1.md#Decision 3]

**3. companyId Filtering Pattern (THE most important Sprint 1 rule):**
Every DB query on tenant-scoped tables MUST filter by companyId. This story establishes the foundation — subsequent stories MUST follow this pattern.

Tenant-scoped tables: companies, users, projects, project_members, voice_messages
NOT tenant-scoped: waitlist_entries, sessions, magic_links
[Source: architecture-sprint1.md#companyId Filtering Pattern]

**4. Drizzle Migration — NOT push:**
Use `drizzle-kit generate` + `drizzle-kit migrate`. Never `drizzle-kit push` for production schema changes.
[Source: architecture-sprint1.md#Decision 4]

**5. voice_messages.status behavior:**
When adding assignment columns, the existing status values remain unchanged: pending → uploaded → transcribed → done/partial/error. Assignment columns are orthogonal to status — a message can be "done" (summary complete) with projectId=null (unassigned, appears in Inbox).
[Source: architecture-sprint1.md#Pipeline Error Handling Pattern]

### Existing Codebase — What to Modify

| File | What Changes | Why |
|------|-------------|-----|
| `lib/db/schema.ts` | Add companies, projects, project_members tables. Extend users (+companyId, +role). Extend voiceMessages (+8 assignment columns) | Core data model changes |
| `lib/auth/session.ts` | Add `getAuthContext()` function. Keep existing `validateSession`, `createSession`, `deleteSession` | New auth context helper |
| `middleware.ts` | Remove `x-user-id` header set. Add companyId existence check | Header injection replaced by getAuthContext() |
| `app/api/voice/route.ts` | Replace `request.headers.get("x-user-id")` with `getAuthContext()`. Add companyId to INSERT | Auth context migration |
| `app/(app)/layout.tsx` | Replace `headers().get("x-user-id")` with `getAuthContext()` | Auth context migration |
| `drizzle.config.ts` | No changes needed — already configured correctly | - |

### Existing Code Patterns to Follow

**Current session.ts pattern (extend, don't rewrite):**
```typescript
// Existing exports — keep all of these:
export async function validateSession(token: string)  // → { userId } | null
export async function createSession(userId: string)   // → { token, expiresAt }
export async function deleteSession(token: string)    // → void

// NEW: Add getAuthContext() alongside existing functions
```

**Current schema.ts pattern (extend, don't rewrite):**
```typescript
// Existing tables — DON'T touch their columns:
export const waitlistEntries = pgTable(...)  // Keep unchanged
export const users = pgTable(...)            // ADD companyId + role columns
export const magicLinks = pgTable(...)       // Keep unchanged
export const sessions = pgTable(...)         // Keep unchanged
export const voiceMessages = pgTable(...)    // ADD assignment columns
```

**Current middleware pattern:**
```typescript
// Currently sets: requestHeaders.set("x-user-id", session.userId)
// Sprint 1: REMOVE this header injection entirely
// Middleware should only: validate session exists → redirect if not
// Add: check that user has companyId set (post-migration safety net)
```

**Cookie name:** `session` — consistent across login, logout, middleware, and getAuthContext()

### Critical Anti-Patterns to Avoid

```typescript
// ❌ WRONG: companyId from client input
export async function createProject(data: { name: string; companyId: string }) {
  await db.insert(projects).values(data);  // Client could send any companyId!
}

// ❌ WRONG: Missing companyId filter on tenant-scoped query
const allProjects = await db.select().from(projects);  // Data leak!

// ❌ WRONG: Using push instead of generate+migrate
// pnpm drizzle-kit push  ← NEVER in production

// ❌ WRONG: Changing voice_messages status based on assignment
await db.update(voiceMessages).set({ status: "error" });  // NO! Status stays "done"
```

### Library/Framework Specifics

| Library | Version | Key Notes |
|---------|---------|-----------|
| Drizzle ORM | 0.45.1 | 3rd arg for indexes returns array: `(table) => [index().on(table.col)]` |
| drizzle-kit | latest | `generate` creates SQL, `migrate` runs it. Config in `drizzle.config.ts` |
| Next.js | 16.1.6 | `cookies()` from `next/headers` is async in Next.js 16 |
| @neondatabase/serverless | via drizzle | `drizzle(url, { schema })` simplified API |

**Drizzle schema patterns established in this project:**
- UUID PKs: `uuid().primaryKey().defaultRandom()`
- Timestamps: `timestamp({ withTimezone: true }).defaultNow().notNull()`
- FKs: `uuid().references(() => otherTable.id)`
- Indexes: 3rd arg `(table) => [index('name').on(table.column)]`

**Next.js 16 cookies() — ASYNC:**
```typescript
import { cookies } from "next/headers";
const cookieStore = await cookies();
const token = cookieStore.get("kurzum_session")?.value;
```

### Project Structure Notes

**New files created by this story:**
- None — all changes are in existing files

**Modified files:**
- `lib/db/schema.ts` — Major: 3 new tables, 2 extended tables
- `lib/auth/session.ts` — Medium: getAuthContext() function
- `middleware.ts` — Medium: remove x-user-id, add companyId check
- `app/api/voice/route.ts` — Medium: use getAuthContext()
- `app/(app)/layout.tsx` — Small: use getAuthContext()

**Migration files generated by Drizzle:**
- `drizzle/XXXX_*.sql` — 7 migration files (naming auto-generated by drizzle-kit)

**No new directories needed** — all changes within existing structure.

### Previous Story Intelligence

**From Story 7.2 (Audio File Upload):**
- Voice API route accepts multipart/form-data, validates MIME type and file size
- `NEXT_PUBLIC_ENABLE_UPLOAD` env var pattern for feature gating
- Audio duration extraction with `AudioContext.decodeAudioData()`

**From Story 7.1 (Dashboard):**
- Dashboard page (`app/(app)/page.tsx`) is a Server Component that fetches data via Drizzle
- Uses `headers()` to get `x-user-id` → this MUST be updated to `getAuthContext()`
- Voice message cards group by day, use relative timestamps
- `VoiceSummary` interface: `{ status, material[], nextSteps, problems?, project }`

**Key takeaway:** The dashboard currently queries `voice_messages WHERE userId = ?`. After this story, it should query `WHERE companyId = ?` (tenant isolation) and `WHERE userId = ?` only for user-specific views.

### Git Intelligence

Recent commit patterns show research-oriented messages:
- `fbcd1f2` Implement audio file upload for research pipeline testing (Story 7.2)
- `602c31a` Implement Mistral LLM summarization pipeline for voice messages (Story 6.3)
- `9675512` Integrate Mistral Voxtral STT pipeline with contextBias for Elektro-Fachbegriffe (Story 6.2)

Commit message style: `"Implement/Evaluate <feature description> (Story X.Y)"`
This story's commit should follow: `"Implement multi-tenant data model and auth context helper (Story 8.1)"`

### References

- [Source: architecture-sprint1.md#Decision 3 — Session Helper for Auth Context]
- [Source: architecture-sprint1.md#Decision 4 — Schema Migration Strategy]
- [Source: architecture-sprint1.md#companyId Filtering Pattern]
- [Source: architecture-sprint1.md#Pipeline Error Handling Pattern]
- [Source: architecture-sprint1.md#Implementation Sequence]
- [Source: prd-sprint1.md#Data Architecture Preparation (S1-FR27–29)]
- [Source: prd-sprint1.md#Tenant Isolation]
- [Source: epics.md#Story 8.1 Acceptance Criteria]
- [Source: docs/architecture-sprint0-supplement.md — Sprint 0 auth patterns]

## Dev Agent Record

### Agent Model Used

Claude Opus 4.6

### Debug Log References

- Drizzle migration tracking fix: Previous sprints used `drizzle-kit push` (no `__drizzle_migrations` table). Created tracking table manually, computed SHA-256 hashes for existing migrations, and registered them to enable `drizzle-kit migrate` workflow going forward.
- Journal timestamp ordering: Manual seed migration (0004) was given a timestamp higher than auto-generated 0005/0006, causing Drizzle to skip them. Fixed by manually applying skipped migrations and correcting all journal timestamps to ascending order.
- `lib/auth/user.ts` type error: `findOrCreateUser()` INSERT lacked required `companyId` after NOT NULL constraint. Fixed by defaulting new users to seed company `DEFAULT_COMPANY_ID`.

### Completion Notes List

- 6 Drizzle migrations generated and applied (0002–0007) + 1 manual seed migration (0004)
- 3 new tables: companies, projects, project_members
- 2 extended tables: users (+companyId, +role default change), voice_messages (+8 assignment columns, +3 indexes)
- `getAuthContext()` implemented in lib/auth/session.ts — reads session cookie, validates against DB, returns { userId, companyId, role }
- Middleware updated: x-user-id header injection removed, companyId existence check added
- All consumers of x-user-id migrated to getAuthContext(): voice API route, app layout, dashboard page
- Backfill successful: all 7 existing voice_messages have companyId set from their user's company
- `pnpm build` passes with zero TypeScript errors

### Change Log

- 2026-02-14: Implement multi-tenant data model and auth context helper (Story 8.1)
  - Created companies, projects, project_members tables
  - Extended users with companyId (NOT NULL FK) and role (default "monteur")
  - Extended voice_messages with 8 assignment columns and 3 indexes
  - Seeded "masemIT Testbetrieb" company, linked Mario as meister
  - Backfilled voice_messages.companyId from users
  - Implemented getAuthContext() replacing x-user-id header injection
  - Updated middleware, voice API, app layout, dashboard to use getAuthContext()
  - Fixed user creation to include default companyId
- 2026-02-14: Code Review fixes (adversarial review)
  - [C1] Removed `/drizzle/meta/` from .gitignore — journal + snapshots must be committed
  - [H1] Added VALID_ROLES runtime validation in getAuthContext() to prevent unsafe type cast
  - [H2] Reduced auth DB queries from 4 to 2 per request — single JOIN in middleware and getAuthContext()
  - [M2] Added TEMPORARY comment on DEFAULT_COMPANY_ID in findOrCreateUser()
  - [M3] Fixed cookie name documentation (was "kurzum_session", actual is "session")

### File List

- `lib/db/schema.ts` — Major: added companies, projects, project_members tables; extended users (+companyId, +role); extended voiceMessages (+8 assignment columns, +3 indexes); exported new types
- `lib/auth/session.ts` — Added AuthContext interface and getAuthContext() function; review: single JOIN query + VALID_ROLES runtime validation
- `lib/auth/user.ts` — Added DEFAULT_COMPANY_ID, companyId to findOrCreateUser INSERT; review: added TEMPORARY comment
- `middleware.ts` — Removed x-user-id header injection, added companyId validation; review: single JOIN query (2→1 DB round-trip)
- `.gitignore` — Review: removed `/drizzle/meta/` exclusion to enable journal+snapshot tracking
- `app/api/voice/route.ts` — Replaced x-user-id header with getAuthContext(), added companyId to INSERT
- `app/app/layout.tsx` — Replaced x-user-id header with getAuthContext()
- `app/app/page.tsx` — Replaced x-user-id header with getAuthContext()
- `drizzle/0002_curved_elektra.sql` — Migration: CREATE companies table
- `drizzle/0003_common_absorbing_man.sql` — Migration: ALTER users ADD company_id + role default change
- `drizzle/0004_seed_company_backfill.sql` — Migration: Seed masemIT Testbetrieb, backfill Mario's user
- `drizzle/0005_small_senator_kelly.sql` — Migration: ALTER users SET company_id NOT NULL
- `drizzle/0006_wooden_exodus.sql` — Migration: CREATE projects + project_members tables
- `drizzle/0007_melted_exodus.sql` — Migration: ALTER voice_messages ADD assignment columns + backfill companyId
- `drizzle/meta/_journal.json` — Updated with all 8 migration entries
