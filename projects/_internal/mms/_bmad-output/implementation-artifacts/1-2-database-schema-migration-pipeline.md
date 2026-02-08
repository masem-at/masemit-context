# Story 1.2: Database Schema & Migration Pipeline

Status: done

## Story

As a **developer**,
I want the complete database schema defined and migrated to NeonDB,
so that all tables are ready for the application and the migration workflow is established.

## Acceptance Criteria

1. **Given** NeonDB is provisioned in Frankfurt (eu-central-1) **When** db/index.ts is created with Drizzle ORM + Neon HTTP driver **Then** the database client connects successfully via DATABASE_URL

2. **Given** the Drizzle client is configured **When** db/schema.ts is created **Then** it defines all 4 tables (donation_targets, donations, donation_config, api_keys) with correct columns, types, and constraints **And** it defines 2 PostgreSQL enums via pgEnum() (source_project_enum, donation_type_enum) **And** all tables use UUID primary keys with gen_random_uuid() default **And** all tables include timestamptz created_at with defaultNow() **And** mutable tables include timestamptz updated_at

3. **Given** the schema is defined **When** `npx drizzle-kit generate` is run **Then** SQL migration files are generated in the drizzle/ folder **And** `npx drizzle-kit migrate` applies the migration to NeonDB successfully

4. **Given** the migration is applied **When** drizzle.config.ts is inspected **Then** it points to the correct schema file and NeonDB connection

5. **Given** the database is connected **When** `GET /health` is called **Then** the response includes DB connectivity status: `{ "status": "ok", "db": "connected" }`

## Tasks / Subtasks

- [x] Task 1: Create Drizzle client with Neon HTTP driver (AC: #1)
  - [x] 1.1 Create src/db/index.ts with Drizzle + Neon HTTP driver using DATABASE_URL
  - [x] 1.2 Use simplified `drizzle(process.env.DATABASE_URL!)` pattern from drizzle-orm/neon-http
  - [x] 1.3 Export db instance for use in queries

- [x] Task 2: Define complete database schema (AC: #2)
  - [x] 2.1 Create src/db/schema.ts as single source of truth for ALL tables
  - [x] 2.2 Define source_project_enum via pgEnum(): hoki_help, tellingcube, chainsights, manual
  - [x] 2.3 Define donation_type_enum via pgEnum(): direct, revenue_share
  - [x] 2.4 Define donation_targets table (8 columns)
  - [x] 2.5 Define donations table (11 columns)
  - [x] 2.6 Define donation_config table (9 columns)
  - [x] 2.7 Define api_keys table (10 columns) with dedicated apiKeyProjectEnum
  - [x] 2.8 Add all indexes: target_id, source_project, donated_at, slug unique, key_hash unique, composite source+target

- [x] Task 3: Create Drizzle Kit configuration (AC: #4)
  - [x] 3.1 Create drizzle.config.ts pointing to src/db/schema.ts and DATABASE_URL
  - [x] 3.2 Configure output directory as ./drizzle for migration files

- [x] Task 4: Update health check with DB connectivity (AC: #5)
  - [x] 4.1 Update src/index.ts health endpoint to check DB connectivity via simple query
  - [x] 4.2 Return `{ "status": "ok", "db": "connected" }` on success
  - [x] 4.3 Return `{ "status": "degraded", "db": "disconnected" }` on DB failure (graceful)

- [x] Task 5: Write tests for schema and health check (AC: #2, #5)
  - [x] 5.1 Create src/db/schema.test.ts verifying all table definitions exist and have correct columns
  - [x] 5.2 Update src/index.test.ts for new health check response format (with mocked DB)
  - [x] 5.3 Test DB client initialization (mock DATABASE_URL)

## Dev Notes

### Architecture Compliance

- **Single schema file:** `src/db/schema.ts` — ALL table definitions in one file, never split across services
- **Drizzle + Neon HTTP:** Use simplified pattern `drizzle(process.env.DATABASE_URL!)` from `drizzle-orm/neon-http`
- **PostgreSQL enums:** Use `pgEnum()` — type safety at DB level. New enum values require migration.
- **Primary keys:** UUID v4 via `gen_random_uuid()` — no sequential exposure
- **Timestamps:** `timestamptz` with `defaultNow()` for created_at. `updated_at` must be explicitly set in queries (no DB trigger)
- **Amount storage:** Integer cents — eliminates floating-point arithmetic issues
- **Currency:** ISO 4217 varchar(3), default 'EUR'

### Critical Constraints

- **api_keys.project enum:** Needs extended enum with masem_at and admin in addition to source_project_enum values. Architecture uses separate text-based approach or extended enum. Use a dedicated pgEnum for api key projects: hoki_help, tellingcube, chainsights, masem_at, admin
- **Indexes are critical:** Must create indexes listed in requirements for query performance
- **Foreign keys:** donations.target_id → donation_targets.id, donation_config.target_id → donation_targets.id
- **Migration workflow:** `drizzle-kit generate` → SQL files committed to Git → `drizzle-kit migrate` in production

### Drizzle Schema Patterns

```typescript
import { pgTable, uuid, varchar, text, boolean, integer, timestamp, decimal, pgEnum, index, uniqueIndex } from 'drizzle-orm/pg-core'

// Enums
export const sourceProjectEnum = pgEnum('source_project_enum', ['hoki_help', 'tellingcube', 'chainsights', 'manual'])
export const donationTypeEnum = pgEnum('donation_type_enum', ['direct', 'revenue_share'])

// UUID PK pattern
id: uuid('id').primaryKey().defaultRandom()

// Timestamps pattern
created_at: timestamp('created_at', { withTimezone: true }).notNull().defaultNow()
updated_at: timestamp('updated_at', { withTimezone: true })
```

### Previous Story Learnings (Story 1.1)

- Project uses ESM (`"type": "module"` in package.json)
- Absolute imports via tsconfig paths `src/*`
- Production dependencies are pinned to exact versions (no caret `^`)
- Vitest with Hono `app.request()` for endpoint testing
- `.gitkeep` files in empty directories

### Database Table Summary

| Table | Columns | Key Features |
|-------|---------|-------------|
| donation_targets | 8 | slug unique, is_active soft-delete |
| donations | 11 | FK to targets, enum columns, cents integer |
| donation_config | 9 | FK to targets, decimal percentage, validity period |
| api_keys | 11 | Extended project enum, hash storage, scopes array |

### References

- [Source: _bmad-output/planning-artifacts/architecture.md#Data Architecture]
- [Source: _bmad-output/planning-artifacts/architecture.md#Naming Patterns]
- [Source: _bmad-output/_masemIT/requirements/mms-donations-requirement.md#Data Model]
- [Source: _bmad-output/planning-artifacts/epics.md#Story 1.2]

## Dev Agent Record

### Agent Model Used

Claude Opus 4.5 (claude-opus-4-5-20251101)

### Debug Log References

- 16/16 tests pass (12 schema tests + 4 health check tests)
- DB module mocked in health check tests to avoid real DB dependency
- Separate apiKeyProjectEnum created for api_keys.project (includes masem_at, admin beyond source_project_enum)

### Completion Notes List

- src/db/index.ts: Drizzle client with Neon HTTP driver, schema-aware mode
- src/db/schema.ts: Complete schema with 3 enums, 4 tables, 6 indexes
- drizzle.config.ts: Points to schema.ts, output to ./drizzle
- Health check now returns DB connectivity status (ok/degraded)
- Migration generation (AC #3) requires DATABASE_URL — `npx drizzle-kit generate` and `npx drizzle-kit migrate` to be run manually by developer with DB access

### Code Review Fixes Applied (2026-02-03)

- Fixed: Health check returns 503 (not 200) when DB is disconnected
- Fixed: Added `import 'dotenv/config'` to drizzle.config.ts for local CLI usage
- Fixed: Renamed `sql` variable to `neonClient` in db/index.ts to avoid shadowing drizzle-orm's `sql` tag
- Fixed: Schema tests extended with column type, notNull, and constraint verification (18 tests)
- Fixed: Removed redundant .gitkeep from db/ directory

### File List

- mms-api/src/db/index.ts (new)
- mms-api/src/db/schema.ts (new)
- mms-api/src/db/schema.test.ts (new)
- mms-api/drizzle.config.ts (new)
- mms-api/src/index.ts (modified — health check with DB)
- mms-api/src/index.test.ts (modified — mocked DB, new test cases)

