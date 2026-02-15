# Story 12.1: pgEnum Role Migration & RBAC Permission Module

Status: done

<!-- Note: Validation is optional. Run validate-create-story for quality check before dev-story. -->

## Story

As a **developer**,
I want the role column migrated to a pgEnum and a reusable RBAC permission module in place,
so that all subsequent stories can enforce role-based access at the DB and application level.

## Acceptance Criteria

1. **Given** the existing `users.role` column stores free text ("monteur", "meister", "buero") **When** a Drizzle migration runs **Then** a `pgEnum('user_role', ['monteur', 'meister', 'buero'])` is created **And** the `users.role` column is converted to use this enum **And** existing role values are preserved **And** the `invitations` table is created with: id, companyId, email, role (using the same pgEnum), token (unique), invitedBy, expiresAt, usedAt, createdAt **And** `voiceMessages.status` and `projects.status` are also converted to pgEnum (Tech Debt #7).

2. **Given** the migration is complete **When** `lib/auth/rbac.ts` is created **Then** it exports a `Role` type: `'monteur' | 'meister' | 'buero'` **And** it exports a `PERMISSIONS` map defining which roles can perform which actions (project:create, project:edit, project:view_all, project:view_assigned, inbox:view, inbox:assign, team:invite, team:manage, voice:record, photo:upload) **And** it exports `hasPermission(role: Role, permission: Permission): boolean` **And** it exports `requirePermission(role: Role, permission: Permission): void` that throws on denied access.

3. **Given** the RBAC module exists **When** `getAuthContext()` returns user data **Then** the `role` field is typed as `Role` (not `string`).

**FRs:** S2-FR5, S2-FR6 | **NFRs:** S2-NFR3

## Tasks / Subtasks

- [x] Task 1: Create pgEnum types and migration (AC: #1)
  - [x] 1.1 Define `userRoleEnum = pgEnum('user_role', ['monteur', 'meister', 'buero'])` in `lib/db/schema.ts`
  - [x] 1.2 Define `projectStatusEnum = pgEnum('project_status', ['active', 'paused', 'completed'])` in `lib/db/schema.ts`
  - [x] 1.3 Define `voiceMessageStatusEnum = pgEnum('voice_message_status', ['pending', 'uploaded', 'transcribed', 'done', 'partial', 'error'])` in `lib/db/schema.ts`
  - [x] 1.4 Update `users.role` column from `text("role")` to `userRoleEnum("role")`
  - [x] 1.5 Update `projects.status` column from `text("status")` to `projectStatusEnum("status")`
  - [x] 1.6 Update `voiceMessages.status` column from `text("status")` to `voiceMessageStatusEnum("status")`
  - [x] 1.7 Run `drizzle-kit generate` to produce migration SQL
  - [x] 1.8 Review generated SQL — Drizzle may generate ALTER COLUMN with USING cast, verify it preserves data
  - [x] 1.9 Run `drizzle-kit migrate` against NeonDB to apply

- [x] Task 2: Create `invitations` table schema (AC: #1)
  - [x] 2.1 Define `invitations` table in `lib/db/schema.ts` with columns: id (uuid, PK), companyId (FK → companies), email (text, notNull), role (userRoleEnum, notNull), token (text, notNull, unique), invitedBy (FK → users), expiresAt (timestamp w/tz, notNull), usedAt (timestamp w/tz, nullable), createdAt (timestamp w/tz, defaultNow)
  - [x] 2.2 Add index on `token` column for lookup performance
  - [x] 2.3 Add index on `companyId` for team listing queries
  - [x] 2.4 Export `Invitation` and `NewInvitation` types
  - [x] 2.5 Generate and apply migration (combined with Task 1 migration as 0008_next_sauron.sql)

- [x] Task 3: Create RBAC permission module (AC: #2)
  - [x] 3.1 Create `lib/auth/rbac.ts`
  - [x] 3.2 Export `Role` type derived from `userRoleEnum` (or explicit union: `'monteur' | 'meister' | 'buero'`)
  - [x] 3.3 Export `Permission` type as union of all permission strings
  - [x] 3.4 Export `PERMISSIONS` const map: `Record<Permission, readonly Role[]>` defining which roles have which permissions
  - [x] 3.5 Export `hasPermission(role: Role, permission: Permission): boolean`
  - [x] 3.6 Export `requirePermission(role: Role, permission: Permission): void` that throws `PermissionError` if denied
  - [x] 3.7 Create `PermissionError` class extending Error with `permission` and `role` fields

- [x] Task 4: Update AuthContext typing (AC: #3)
  - [x] 4.1 Update `AuthContext.role` type in `lib/auth/session.ts` to use `Role` from `lib/auth/rbac.ts`
  - [x] 4.2 Remove the manual `VALID_ROLES` array (no longer needed — pgEnum enforces at DB level)
  - [x] 4.3 Remove the runtime role validation check (DB enum guarantees valid values)
  - [x] 4.4 Verify `getAuthContext()` still works with enum-typed column

- [x] Task 5: Update Zod validation schemas (AC: #1)
  - [x] 5.1 Update `PROJECT_STATUS_OPTIONS` in `lib/validations/project.ts` to reference the enum values (kept as const array — enum provides DB enforcement)
  - [x] 5.2 Verify `updateProjectStatusSchema` still validates correctly with enum column
  - [x] 5.3 Verify all existing Zod schemas that reference status fields still work

## Dev Notes

### Architecture & Implementation Patterns

- **Drizzle pgEnum**: Import `pgEnum` from `drizzle-orm/pg-core`. Define BEFORE tables that reference them. Drizzle v0.45.1 supports pgEnum natively.
- **Migration strategy**: `drizzle-kit generate` should produce `CREATE TYPE ... AS ENUM (...)` + `ALTER TABLE ... ALTER COLUMN ... TYPE ... USING ...::enum_type`. This is a zero-downtime operation in PostgreSQL — no table rewrite needed.
- **Server Actions pattern**: All mutations use `getAuthContext()` → this continues unchanged. The RBAC module adds an additional layer on top.
- **Tenant isolation remains primary**: companyId filtering is still the first line of defense. RBAC adds role-based permissions on top.

### Critical Constraints

- **Data preservation**: The migration MUST preserve all existing role and status values. Test on a NeonDB branch first if uncertain.
- **No runtime breakage**: The `getAuthContext()` return type changes from `string` to the enum union. Since it's already typed as `"monteur" | "meister" | "buero"` in the `AuthContext` interface, downstream code should not break.
- **Enum values are immutable in PostgreSQL**: Adding new enum values later requires `ALTER TYPE ... ADD VALUE`. Drizzle handles this in migrations, but be aware.
- **Migration journal timestamps MUST be ascending**: Drizzle skips migrations with timestamps ≤ last applied. Ensure the new migration has a later timestamp than 0007.
- **invitations table is schema-only**: Story 12.2 will implement the invitation logic. This story only creates the table.

### Existing Code to Reuse

| Component/Function | Source | Reuse Approach |
|---|---|---|
| `getAuthContext()` | `lib/auth/session.ts` | Update return type, remove manual validation |
| `AuthContext` interface | `lib/auth/session.ts` | Update `role` type to use `Role` |
| `VALID_ROLES` | `lib/auth/session.ts` | **Remove** — replaced by pgEnum DB constraint |
| `PROJECT_STATUS_OPTIONS` | `lib/validations/project.ts` | Keep as const array, enum provides DB enforcement |
| `STATUS_CONFIG` | `lib/validations/project.ts` | Unchanged — UI rendering config |
| Migration pattern | `drizzle/0007_melted_exodus.sql` | Reference for manual SQL if Drizzle generates incomplete migration |

### Current State of Role/Status Columns

| Column | Current Type | Current Default | Values in Use |
|---|---|---|---|
| `users.role` | `text` | `"monteur"` | monteur, meister, buero |
| `projects.status` | `text` | `"active"` | active, paused, completed |
| `voiceMessages.status` | `text` | `"pending"` | pending, uploaded, transcribed, done, partial, error |

**Important**: The original migration (0001) created `users.role` with default `'user'`, but `schema.ts` was later changed to default `'monteur'`. The pgEnum migration should set the default to `'monteur'` explicitly.

### RBAC Permission Matrix

```
Permission              | monteur | meister | buero
------------------------|---------|---------|------
project:create          |         |    ✓    |   ✓
project:edit            |         |    ✓    |   ✓
project:view_all        |         |    ✓    |   ✓
project:view_assigned   |    ✓    |    ✓    |   ✓
inbox:view              |         |    ✓    |   ✓
inbox:assign            |         |    ✓    |   ✓
team:invite             |         |    ✓    |   ✓
team:manage             |         |    ✓    |   ✓
voice:record            |    ✓    |    ✓    |   ✓
voice:confirm_assignment|    ✓    |    ✓    |   ✓
photo:upload            |    ✓    |    ✓    |   ✓
```

[Source: docs/_masemIT/bmad-briefing-kurzum-sprint2.md#RBAC Middleware]

### Project Structure Notes

**New files:**
- `lib/auth/rbac.ts` — RBAC permission module (Role type, PERMISSIONS map, hasPermission, requirePermission)

**Modified files:**
- `lib/db/schema.ts` — Add 3 pgEnum definitions, update 3 columns, add invitations table
- `lib/auth/session.ts` — Update AuthContext.role type, remove VALID_ROLES + manual check
- `lib/validations/project.ts` — Minor: verify status enum compatibility

**Migration files (generated):**
- `drizzle/XXXX_*.sql` — pgEnum creation + column type changes + invitations table

### Testing Strategy

- **Manual test 1**: Run migration against NeonDB → verify all 3 enum types created → verify existing data preserved → verify new `invitations` table exists with correct schema
- **Manual test 2**: Log in → navigate to `/app` → verify dashboard loads (getAuthContext still works with enum role) → verify project list shows status badges correctly
- **Manual test 3**: Create a new project → verify status "active" is accepted by enum → edit project → verify status change works
- **Manual test 4**: Upload a voice message → verify all status transitions work (pending → uploaded → transcribed → done)
- **Manual test 5**: Import `hasPermission` and `requirePermission` → verify correct behavior: `hasPermission('monteur', 'project:create')` returns false, `hasPermission('meister', 'project:create')` returns true
- **Manual test 6**: TypeScript compile check — `npx tsc --noEmit` passes with zero errors after all changes

### References

- [Source: _bmad-output/planning-artifacts/epics.md#Story 12.1] — Acceptance criteria and FR mapping
- [Source: docs/_masemIT/bmad-briefing-kurzum-sprint2.md#AP 2.2] — Team invitation & RBAC requirements, permission matrix
- [Source: docs/tech-debt-sprint1-audit.md#7] — pgEnum statt freier Text (Tech Debt resolved)
- [Source: lib/db/schema.ts#users] — Current role column definition (text, default "monteur")
- [Source: lib/db/schema.ts#voiceMessages] — Current status column definition (text, default "pending")
- [Source: lib/db/schema.ts#projects] — Current status column definition (text, default "active")
- [Source: lib/auth/session.ts#AuthContext] — Current interface with role typed as union
- [Source: lib/auth/session.ts#VALID_ROLES] — Runtime validation to be removed
- [Source: lib/validations/project.ts#PROJECT_STATUS_OPTIONS] — Status values constant
- [Source: drizzle/meta/_journal.json] — Migration journal (latest: 0007)

### Previous Story Intelligence (11.1)

- **Server Component pattern**: All pages use Server Components with direct Drizzle queries — this continues unchanged
- **getAuthContext() pattern**: Consistently used across ALL stories 8.1–11.1 — the role type change must be backward-compatible
- **Raw SQL queries**: `getProjectsWithStats()` and `getDashboardProjects()` use raw SQL with `sql` template — these reference `status` as string literals (`p.status = 'active'`, `vm.status = 'done'`). PostgreSQL auto-casts enum ↔ string in comparisons, so these queries should work without changes.
- **STATUS_CONFIG rendering**: Projects list and dashboard both use `STATUS_CONFIG[project.status]` — this lookup uses string keys, which work with enum values.

### Git Intelligence

Recent commits:
- `35f118a` Plan Sprint 2 (4 epics, 10 stories)
- `e5f83ec` Document FFG research findings (ADR-007–009)
- `83a69dc` Fix Sprint 1 tech debt (bounded queries, confidence threshold, DSGVO retention)
- `fa1a345` Rebuild dashboard with project-based overview cards (Story 11.1)

Pattern: Single commit per story, descriptive message.
Suggested commit: "Add pgEnum for roles and status columns, create RBAC permission module (Story 12.1)"

### NeonDB Migration Notes

- **NeonDB project**: `delicate-field-50680020` (org: `org-fragrant-glitter-08048890`)
- **Drizzle workflow**: `drizzle-kit generate` → review SQL → `drizzle-kit migrate`
- **Journal timestamps**: Must be ascending. Drizzle auto-generates timestamps when running `generate`.
- **pgEnum in PostgreSQL**: `CREATE TYPE enum_name AS ENUM ('val1', 'val2', ...)` then `ALTER TABLE ... ALTER COLUMN ... TYPE enum_name USING column::enum_name`
- **No table rewrite**: Changing a text column to an enum is metadata-only if all existing values are valid enum values. This is safe for live tables.

## Dev Agent Record

### Agent Model Used

Claude Opus 4.6

### Debug Log References

No debug issues encountered.

### Completion Notes List

- **Task 1**: Defined 3 pgEnum types (`user_role`, `project_status`, `voice_message_status`) in schema.ts. Updated `users.role`, `projects.status`, and `voiceMessages.status` columns to use respective enums. Generated migration `0008_next_sauron.sql` with `CREATE TYPE` + `ALTER COLUMN ... USING cast`. Applied to NeonDB successfully. Verified all 3 enum types exist and existing data (1 user with role "meister") preserved.
- **Task 2**: Created `invitations` table in schema.ts with all required columns (id, companyId, email, role using userRoleEnum, token unique, invitedBy, expiresAt, usedAt, createdAt), indexes on token and companyId, FK constraints to companies and users. Exported `Invitation` and `NewInvitation` types. Combined in same migration as Task 1.
- **Task 3**: Created `lib/auth/rbac.ts` with `Role` type derived from pgEnum, `Permission` union type (11 permissions including `voice:confirm_assignment` from briefing), `PERMISSIONS` map matching the permission matrix, `hasPermission()`, `requirePermission()`, and `PermissionError` class. Verified via tsx: monteur correctly denied project:create, meister correctly granted, PermissionError thrown with role+permission fields.
- **Task 4**: Updated `AuthContext.role` to use `Role` type from rbac.ts. Removed `VALID_ROLES` array and runtime role validation check — pgEnum now enforces valid roles at DB level. Removed the `as AuthContext["role"]` cast since Drizzle returns the correct enum type.
- **Task 5**: Verified `PROJECT_STATUS_OPTIONS` const array still compatible with pgEnum column. All Zod schemas validate correctly. `npx tsc --noEmit` passes with zero errors. `npx next build` succeeds with all routes compiling.
- **Code Review Fixes**: [M1] Added `updatedAt` column to `invitations` table per architecture convention (created_at + updated_at on every table). [M2] Added compound index `idx_invitations_company_email` on `(companyId, email)` for Story 12.2 invitation lookup performance. [M3] Set up Vitest test infrastructure (vitest.config.ts, package.json scripts) and created 20 unit tests for RBAC module covering full permission matrix, PermissionError behavior, and edge cases. Generated and applied migration `0009_overconfident_the_order.sql` for M1+M2.

### Change Log

- 2026-02-15: Implemented Story 12.1 — pgEnum migration for role/status columns, invitations table schema, RBAC permission module, AuthContext typing update (Tech Debt #7 resolved)
- 2026-02-15: Code Review fixes — added updatedAt to invitations (architecture convention), compound index (companyId, email) for Story 12.2 lookup performance, Vitest test infrastructure + 20 RBAC unit tests

### File List

**New files:**
- `lib/auth/rbac.ts` — RBAC permission module (Role, Permission, PERMISSIONS, hasPermission, requirePermission, PermissionError)
- `lib/auth/rbac.test.ts` — 20 unit tests for RBAC module (hasPermission, requirePermission, PermissionError, permission matrix)
- `drizzle/0008_next_sauron.sql` — Migration: CREATE TYPE x3, ALTER COLUMN x3, CREATE TABLE invitations
- `drizzle/0009_overconfident_the_order.sql` — Migration: ADD updated_at to invitations, CREATE INDEX (company_id, email)
- `drizzle/meta/0008_snapshot.json` — Drizzle migration snapshot
- `drizzle/meta/0009_snapshot.json` — Drizzle migration snapshot
- `vitest.config.ts` — Vitest configuration with @/ path alias

**Modified files:**
- `lib/db/schema.ts` — Added 3 pgEnum definitions, updated 3 columns to use enums, added invitations table with types + updatedAt + compound index
- `lib/auth/session.ts` — AuthContext.role typed as Role, removed VALID_ROLES and runtime validation
- `drizzle/meta/_journal.json` — Added entries for migrations 0008, 0009
- `package.json` — Added vitest devDependency, test + test:watch scripts
