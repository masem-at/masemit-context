# Story 4.1: Database Schema & Entitlements Model

**Epic:** 4 - Auth & User Dashboard (Phase 2)
**Status:** ready-for-dev
**Created:** 2026-01-21

## User Story

As a **developer**,
I want **the user, magic_links, and user_scenarios tables created with proper indexes and TypeScript types**,
So that **the auth system has a solid data foundation**.

## Acceptance Criteria

### AC1: Users Table
**Given** the database migration is run
**When** the tables are created
**Then** the `users` table exists with columns:
- id (UUID, primary key)
- email (TEXT, unique, not null)
- tier (TEXT, default 'free') - values: 'free', 'single', 'lifetime', 'lifetime_plus', 'pro', 'team'
- billing_model (TEXT, default 'none') - values: 'none', 'one_time', 'subscription'
- max_years (INTEGER, default 1) - values: 1, 3, 5
- has_hr_domain (BOOLEAN, default false)
- generations_remaining (INTEGER, nullable) - NULL = unlimited
- stripe_customer_id (TEXT, nullable)
- stripe_subscription_id (TEXT, nullable)
- subscription_status (TEXT, nullable) - 'active', 'canceled', 'past_due'
- subscription_ends_at (TIMESTAMPTZ, nullable)
- last_login_at (TIMESTAMPTZ, nullable)
- login_count (INTEGER, default 0)
- created_at (TIMESTAMPTZ, default now())
- updated_at (TIMESTAMPTZ, default now())

### AC2: Magic Links Table
**Given** the database migration is run
**When** the tables are created
**Then** the `magic_links` table exists with columns:
- id (UUID, primary key)
- email (TEXT, not null)
- token (TEXT, unique, not null)
- created_at (TIMESTAMPTZ, default now())
- expires_at (TIMESTAMPTZ, not null) - created_at + 1 hour
- used_at (TIMESTAMPTZ, nullable)
- redirect_to (TEXT, default '/dashboard')
- created_for (TEXT) - 'login', 'purchase_complete'

### AC3: User Scenarios Junction Table
**Given** the database migration is run
**When** the tables are created
**Then** the `user_scenarios` junction table exists with:
- user_id (UUID, FK to users)
- scenario_id (UUID, FK to scenarios)
- created_at (TIMESTAMPTZ, default now())
- Primary key: (user_id, scenario_id)

### AC4: Database Indexes
**Given** the database migration is run
**When** the tables are created
**Then** indexes exist on:
- magic_links(token)
- magic_links(email)
- users(email)
- users(stripe_customer_id)

### AC5: TypeScript Types
**Given** the types are needed
**When** lib/auth/entitlements.ts is created
**Then** the following types are defined:
- `Tier` type: 'free' | 'single' | 'lifetime' | 'lifetime_plus' | 'pro' | 'team'
- `BillingModel` type: 'none' | 'one_time' | 'subscription'
- `UserEntitlements` interface with all entitlement fields
- `TIER_ENTITLEMENTS` constant mapping each tier to its entitlements

### AC6: Helper Functions
**Given** entitlement checks are needed
**When** the helper functions are called
**Then** the following functions work correctly:
- `canGenerate(user)` - returns true if user has generations remaining
- `canAccessYears(user, years)` - returns true if user's maxYears >= requested years
- `canAccessHr(user)` - returns true if user.hasHrDomain is true

## Technical Details

### Source References
- Architecture: `docs/architecture/auth-dashboard-architecture.md` (Section 1, 3)
- Epic Definition: `docs/epics-big-bang.md` (Story 4.1)

### Files to Create/Modify
1. `prisma/schema.prisma` - Add User, MagicLink, UserScenario models
2. `lib/auth/entitlements.ts` - TypeScript types and helpers (NEW)
3. `prisma/migrations/*` - Generated migration

### Implementation Notes
- Use Prisma for schema definition (consistent with existing codebase)
- Use `@default(uuid())` for UUIDs (Prisma standard)
- Add `@@map()` directives to use snake_case table names (existing pattern)
- The `generations_remaining` being NULL means unlimited - important distinction

## Tasks

- [ ] Add User model to Prisma schema
- [ ] Add MagicLink model to Prisma schema
- [ ] Add UserScenario model to Prisma schema
- [ ] Create lib/auth/entitlements.ts with types
- [ ] Implement canGenerate helper
- [ ] Implement canAccessYears helper
- [ ] Implement canAccessHr helper
- [ ] Run prisma generate and migrate

## Definition of Done

- [ ] All acceptance criteria pass
- [ ] Migration runs successfully on local DB
- [ ] TypeScript compiles without errors
- [ ] Helper functions have basic unit test coverage
