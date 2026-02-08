---
stepsCompleted: [1, 2, 3, 4, 5, 6, 7, 8]
lastStep: 8
status: 'complete'
completedAt: '2026-02-02'
inputDocuments:
  - _bmad-output/_masemIT/requirements/mms-donations-requirement.md
  - _bmad-output/project-bible.md
  - _bmad-output/_masemIT/company-info.md
workflowType: 'architecture'
project_name: 'mms'
user_name: 'Sempre'
date: '2026-02-02'
---

# Architecture Decision Document

_This document builds collaboratively through step-by-step discovery. Sections are appended as we work through each architectural decision together._

## Project Context Analysis

### Requirements Overview

**Functional Requirements:**
- Donations CRUD with filtering, pagination, and date-range queries
- Donation Targets management (organizations receiving donations)
- Donation Config management (revenue-share percentages per project/target)
- Public aggregated summary endpoint (cached, no auth, no source breakdown)
- Internal detailed summary endpoint (with source breakdown for admin/accounting)
- API Key management with project-scoped permissions

**Non-Functional Requirements:**
- Performance: Public endpoint < 200ms (cached), Internal < 500ms (incl. cold start)
- Security: SHA-256 hashed API keys, scope-based authorization, CORS restrictions, rate limiting (100 req/min public)
- Caching: 15min TTL on public summary with cache-bust on POST
- Data Integrity: Amounts in cents (integer arithmetic), ISO 4217 currency codes
- GDPR: Frankfurt region (eu-central-1), no personal data exposed on public endpoints
- Extensibility: Service pattern must be replicable for future MMS services

**Scale & Complexity:**

- Primary domain: API/Backend (pure REST, no frontend)
- Complexity level: Low-Medium (Phase 1), architecture designed for Medium-High (multi-service future)
- Estimated architectural components: 4 middleware layers, 1 service module (donations), 1 DB layer, 1 cache layer

### Technical Constraints & Dependencies

- Runtime: Vercel Pro with Fluid Compute (Node.js, ~115ms cold starts)
- Database: NeonDB Launch tier, Frankfurt region, serverless HTTP driver (no persistent connections)
- Cache: Upstash Redis free tier (10K commands/day ceiling)
- Framework: Express 5 (mature, reliable Node.js runtime support on Vercel)
- ORM: Drizzle ORM with drizzle-kit for migrations
- Domain: api.masem.at (Vercel custom domain)
- Cost constraint: Zero incremental monthly cost (all within existing plans/free tiers)

### Cross-Cutting Concerns Identified

1. **Authentication & Authorization** — API key validation + scope enforcement, reusable across all future services
2. **Error Handling** — Standardized error response format with typed error codes, consistent across all endpoints
3. **Input Validation** — Zod schemas for all request bodies, shareable validation patterns
4. **CORS Management** — Different policies for public vs. internal endpoints
5. **Rate Limiting** — Upstash-based, configurable per endpoint category
6. **Caching Strategy** — Cache key patterns, invalidation logic, must scale beyond single-key approach
7. **Database Patterns** — Drizzle schema conventions (UUIDs, timestamps, soft-deletes), migration workflow
8. **Enum Management** — PostgreSQL enums require migrations for new values; trade-off between type safety and flexibility

## Starter Template Evaluation

### Primary Technology Domain

API/Backend (pure REST) based on project requirements analysis.

### Starter Options Considered

| Option | Assessment |
|--------|-----------|
| Express 5 | Mature, reliable, excellent Vercel Node.js runtime support. **Selected after Hono migration issues.** |
| Hono (original choice) | Body parsing issues on Vercel Node.js runtime — POST requests hung indefinitely |
| Fastify | Good option, but Express is more familiar and proven |

### Selected Framework: Express 5

**Rationale for Selection:**
- Mature, battle-tested framework with excellent Node.js runtime support
- Express 5 has native Promise support and improved error handling
- Reliable body parsing with `express.json()` middleware
- Large ecosystem and well-documented patterns

**Note:** Originally Hono was selected, but body parsing issues on Vercel's Node.js runtime (POST requests hanging) required migration to Express on 2026-02-05.

**Initialization:**

```bash
npm init -y && npm install express cors
```

### Architectural Decisions Provided by Starter

**Language & Runtime:**
- TypeScript with strict mode (NodeNext module resolution)
- Node.js runtime on Vercel (Fluid Compute)

**Build Tooling:**
- Vercel-native build pipeline with TypeScript compilation
- ESM modules with explicit `.js` extensions in imports

**Code Organization:**
- `src/index.ts` — Express app entry point (default export)
- `src/dev.ts` — Local development server
- `api/index.ts` — Vercel serverless entry point

**Development Experience:**
- `npm run dev` (tsx watch) for local development

### Additional Dependencies (Post-Init)

```bash
npm install drizzle-orm@0.45.1 @neondatabase/serverless@1.0.2 @upstash/redis@1.36.1 @upstash/ratelimit@2.0.8 zod resend
npm install -D drizzle-kit @types/node dotenv vitest
```

### Verified Package Versions (2026-02-05)

| Package | Version | Notes |
|---------|---------|-------|
| express | 5.2.1 | Express 5 with native Promise support |
| cors | 2.8.6 | CORS middleware for Express |
| drizzle-orm | 0.45.1 | Stable (v1.0 beta available — stay on stable for production) |
| drizzle-kit | ~0.31.x | Aligned with drizzle-orm stable |
| @neondatabase/serverless | 1.0.2 | GA, requires Node.js >= 19 |
| @upstash/redis | 1.36.1 | HTTP-based, serverless-native |
| @upstash/ratelimit | 2.0.8 | Includes new dynamic limits feature |
| zod | 4.3.6 | Input validation |
| resend | 6.9.1 | Email notifications |
| vitest | 4.0.18 | Test runner |

**Note:** Project initialization using this command should be the first implementation story.

## Core Architectural Decisions

### Decision Priority Analysis

**Critical Decisions (Block Implementation):**
- Migration workflow, enum strategy, auth pattern, error handling, response format

**Important Decisions (Shape Architecture):**
- Testing strategy, CI/CD pipeline, caching patterns, CORS configuration

**Deferred Decisions (Post-MVP):**
- Structured logging (pino), dedicated dev environment, GitHub Actions CI, API key rotation mechanism

### Data Architecture

| Decision | Choice | Rationale |
|----------|--------|-----------|
| Migration Workflow | `drizzle-kit generate` → SQL files in Git → `drizzle-kit migrate` | Audit trail, rollback capability, version control for production system |
| Enum Strategy | PostgreSQL enums via `pgEnum()` | Type safety at DB level; new source projects are rare, migration cost acceptable |
| Primary Keys | UUID v4 via `gen_random_uuid()` | No sequential exposure, safe for distributed systems |
| Timestamps | `timestamptz` with `defaultNow()`, `updated_at` auto-managed | Timezone-aware, consistent across all tables |
| Amount Storage | Integer (cents) | Eliminates floating-point arithmetic issues |
| Currency | ISO 4217 varchar(3), default 'EUR' | Standards-compliant, extensible |

### Authentication & Security

| Decision | Choice | Rationale |
|----------|--------|-----------|
| Auth Method | API key via `x-api-key` header | Simple, stateless, sufficient for service-to-service auth |
| Key Storage | SHA-256 hash in DB, prefix for identification | Never store plaintext; prefix enables quick visual identification |
| Key Format | `mms_{project}_{random}` | Self-documenting, easy to revoke per project |
| Scope Enforcement | Array-based scopes per key: `donations:read`, `donations:write`, `config:read`, `config:write` | Granular, extensible for future services |
| CORS | Public: `masem.at` only. Internal: known project domains | Principle of least privilege |
| Rate Limiting | Upstash `@upstash/ratelimit` sliding window, 100 req/min on public endpoints | Serverless-native, no state to manage |

### API & Communication Patterns

| Decision | Choice | Rationale |
|----------|--------|-----------|
| API Style | REST, versioned via URL prefix `/v1` | Simple, well-understood, appropriate for CRUD |
| Response Envelope (list) | `{ data: [...], meta: { page, limit, total } }` | Consistent pagination metadata |
| Response Envelope (single) | `{ data: {...} }` | Consistent unwrapping pattern |
| Error Format | `{ error: { code, message, details? } }` | Typed codes enable programmatic handling |
| Pagination | Offset-based (`page` + `limit`, max 100) | Simple, sufficient for expected data volumes |
| Input Validation | Zod schemas per endpoint, validated in handlers | Type inference + runtime validation in one |
| Content Type | JSON only (`application/json`) | Pure API, no form data needed |

### Infrastructure & Deployment

| Decision | Choice | Rationale |
|----------|--------|-----------|
| Hosting | Vercel Pro with Fluid Compute | Already available, auto-scaling, ~115ms cold starts |
| CI/CD | Vercel Git Integration (auto-deploy on push to main) | Zero config, Preview Deployments for PRs |
| Environments | Production only + Vercel Preview Deployments | Lean setup; NeonDB branching for migration testing when needed |
| Domain | `api.masem.at` via Vercel custom domain | Clean separation from project domains |
| Health Check | `GET /health` → checks DB + Redis connectivity | Central service must verify its dependencies |
| Monitoring | Vercel Runtime Logs + Analytics (Pro plan) | Sufficient for Phase 1; structured logging deferred |
| Environment Vars | Vercel Environment Variables (encrypted) | `DATABASE_URL`, `UPSTASH_REDIS_REST_URL`, `UPSTASH_REDIS_REST_TOKEN`, API key secrets |

### Testing Strategy

| Decision | Choice | Rationale |
|----------|--------|-----------|
| Test Runner | Vitest | Fast, native ESM/TS, excellent Express compatibility |
| Unit Tests | Drizzle queries + handler logic with mocked DB | Fast feedback, isolated |
| Integration Tests | Supertest with Express app | Full endpoint testing without HTTP server overhead |
| Test Coverage Target | Critical paths: auth middleware, donation CRUD, cache invalidation | Pragmatic, not 100% coverage for coverage's sake |

### Decision Impact Analysis

**Implementation Sequence:**
1. Project scaffold + Vercel deploy (health check)
2. DB schema + migrations
3. Auth middleware
4. Error handling + validation patterns
5. Donation endpoints (CRUD + summary)
6. Caching layer
7. Rate limiting
8. Tests

**Cross-Component Dependencies:**
- Auth middleware must exist before any internal endpoint
- DB schema must be migrated before any query logic
- Error handling patterns used by all handlers — establish first
- Cache layer depends on working donation queries to cache

## Implementation Patterns & Consistency Rules

### Critical Conflict Points Identified

12 areas where AI agents could make different choices — all resolved below.

### Naming Patterns

**Database Naming (PostgreSQL Convention):**
- Tables: `snake_case`, plural → `donation_targets`, `api_keys`
- Columns: `snake_case` → `source_project`, `amount_cents`, `created_at`
- Foreign keys: `{singular_table}_id` → `target_id` (references `donation_targets`)
- Indexes: `{table}_{column(s)}_idx` → `donations_target_id_idx`
- Enums: `snake_case` → `source_project_enum`, `donation_type_enum`
- Enum values: `snake_case` → `hoki_help`, `revenue_share`

**API Naming (REST Convention):**
- Endpoints: `kebab-case`, plural nouns → `/v1/donations`, `/v1/donation-targets`
- Route params: `:id` → `/v1/donations/:id`
- Query params: `snake_case` → `?source_project=hoki_help&target_id=uuid`
- Custom headers: `x-api-key` (lowercase with hyphens)

**Code Naming (TypeScript Convention):**
- Files: `kebab-case.ts` → `rate-limit.ts`, `donation-targets.ts`
- Functions: `camelCase` → `getDonationSummary()`, `validateApiKey()`
- Types/Interfaces: `PascalCase` → `Donation`, `ApiKeyScope`, `CreateDonationInput`
- Constants: `UPPER_SNAKE_CASE` → `MAX_PAGE_SIZE`, `CACHE_TTL_SECONDS`
- Zod schemas: `camelCase` + `Schema` suffix → `createDonationSchema`, `paginationSchema`
- Enums in code: match DB values → `source_project_enum` values are `'hoki_help' | 'tellingcube' | ...`

**JSON Response Fields:**
- `snake_case` in API responses → matches DB columns, no transformation layer needed
- Example: `{ "amount_cents": 2500, "source_project": "hoki_help", "created_at": "..." }`

### Structure Patterns

**Project Organization (Feature-based):**

```
src/
├── index.ts              # Express app entry, composes all routes
├── middleware/            # Shared middleware (auth, cors, rate-limit)
│   └── {name}.ts
├── services/
│   └── {service-name}/   # One folder per service domain
│       ├── routes.ts     # Hono route definitions
│       ├── handlers.ts   # Request/response logic
│       ├── queries.ts    # Drizzle DB queries
│       └── types.ts      # Zod schemas + TS types
├── db/
│   ├── index.ts          # Drizzle + Neon client init
│   ├── schema.ts         # ALL table definitions (single file)
│   └── seed.ts           # Seed data scripts
└── lib/
    ├── cache.ts          # Upstash Redis helper
    └── errors.ts         # Error response helpers
```

**Test Organization (Co-located):**

```
src/
├── middleware/
│   ├── auth.ts
│   └── auth.test.ts      # Test next to source
├── services/donations/
│   ├── handlers.ts
│   ├── handlers.test.ts
│   ├── queries.ts
│   └── queries.test.ts
```

**Rules:**
- Tests are ALWAYS co-located with source files as `{name}.test.ts`
- No separate `__tests__/` directory
- Each service folder is self-contained — all files for that domain live there
- `schema.ts` is the SINGLE source of truth for all tables — never split across services
- `lib/` contains only truly shared utilities, not service-specific helpers

### Format Patterns

**API Response Formats:**

Success (single):
```json
{ "data": { "id": "uuid", "amount_cents": 2500 } }
```

Success (list):
```json
{
  "data": [{ "id": "uuid", "amount_cents": 2500 }],
  "meta": { "page": 1, "limit": 50, "total": 127 }
}
```

Error:
```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "amount_cents must be a positive integer",
    "details": [{ "field": "amount_cents", "message": "Expected positive integer" }]
  }
}
```

**Date Format:**
- Always ISO 8601 strings in UTC → `"2026-02-02T10:00:00Z"`
- Never Unix timestamps in API responses
- DB stores as `timestamptz`, serialized to ISO string

**Null Handling:**
- Nullable fields included in response with `null` value, never omitted
- Example: `{ "donor_note": null }` not `{}` (missing field)

### Process Patterns

**Error Handling:**
- All errors go through `lib/errors.ts` helper functions
- Error codes are typed constants: `VALIDATION_ERROR`, `UNAUTHORIZED`, `FORBIDDEN`, `NOT_FOUND`, `RATE_LIMITED`, `INTERNAL_ERROR`
- Handlers NEVER throw raw errors — always use error helpers
- DB errors caught in queries layer, wrapped as `INTERNAL_ERROR`
- Zod validation errors caught in handlers, formatted as `VALIDATION_ERROR` with field details

**Middleware Chain Order:**
1. CORS
2. Rate limiting (public routes only)
3. Auth (internal routes only)
4. Route handler

**Import Patterns:**
- Absolute imports from `src/` root — no relative `../../` paths
- Group imports: external packages → internal modules → types
- Barrel exports (`index.ts`) ONLY at service level, not in `lib/`

**Cache Patterns:**
- Cache key format: `mms:{service}:{scope}:{identifier}` → `mms:donations:public:summary`
- Cache helper always returns `T | null` — never throws on cache miss
- Cache-bust: delete key on write operations, never update in place
- TTL constants defined in `lib/cache.ts`, not hardcoded in handlers

### Enforcement Guidelines

**All AI Agents MUST:**
1. Follow naming conventions exactly — no "creative improvements"
2. Place files in the defined structure — no new top-level directories without approval
3. Use error helpers from `lib/errors.ts` — no custom error formatting
4. Write tests co-located with source — no separate test directories
5. Use Zod schemas for ALL input validation — no manual checks
6. Return responses through the envelope format — no direct object returns
7. Use `snake_case` for all JSON fields — no camelCase in API responses

**Anti-Patterns (NEVER do this):**
- Creating `src/utils/` or `src/helpers/` — use `lib/` or service-specific
- Splitting schema across service folders
- Using `camelCase` in JSON responses
- Hardcoding cache TTL values in handlers
- Importing with relative paths like `../../lib/cache`
- Throwing raw errors instead of using error helpers
- Returning raw DB rows without response envelope

## Project Structure & Boundaries

### Complete Project Directory Structure

```
mms-api/
├── README.md
├── package.json
├── tsconfig.json
├── drizzle.config.ts               # Drizzle Kit configuration
├── vitest.config.ts                 # Vitest test configuration
├── vercel.json                      # Vercel project config (rewrites, headers)
├── .env.example                     # Template for required env vars
├── .gitignore
│
├── drizzle/                         # Generated SQL migration files (Git-tracked)
│   └── NNNN_migration_name.sql
│
└── src/
    ├── index.ts                     # Express app entry: compose middleware + service routes + health check + default export
    │
    ├── middleware/
    │   ├── auth.ts                  # API key validation, scope checking, x-api-key header extraction
    │   ├── auth.test.ts
    │   ├── cors.ts                  # CORS config: public (masem.at) vs internal (project domains)
    │   ├── cors.test.ts
    │   ├── rate-limit.ts            # Upstash sliding window rate limiter
    │   └── rate-limit.test.ts
    │
    ├── services/
    │   └── donations/
    │       ├── routes.ts            # All donation route definitions: public + internal
    │       ├── handlers.ts          # Request/response logic for each endpoint
    │       ├── handlers.test.ts
    │       ├── queries.ts           # Drizzle DB queries: CRUD, aggregation, filtering
    │       ├── queries.test.ts
    │       └── types.ts             # Zod schemas + inferred TS types for all donation endpoints
    │
    ├── db/
    │   ├── index.ts                 # Drizzle client init with Neon HTTP driver
    │   ├── schema.ts                # ALL table definitions: donation_targets, donations, donation_config, api_keys + enums
    │   └── seed.ts                  # Seed: HoKi target, initial API keys, test donations
    │
    └── lib/
        ├── cache.ts                 # Upstash Redis: get/set/del helpers, TTL constants, key builders
        ├── cache.test.ts
        ├── errors.ts                # Error response factory: apiError(), typed error codes, Zod error formatter
        └── errors.test.ts
```

### Architectural Boundaries

**API Layer Boundaries:**

```
Public (no auth)          Internal (x-api-key required)
─────────────────         ──────────────────────────────
GET /health               POST /v1/donations
GET /v1/donations/        GET  /v1/donations
    public/summary        GET  /v1/donations/:id
                          GET  /v1/donations/summary
                          GET  /v1/donation-targets
                          POST /v1/donation-targets
                          PATCH /v1/donation-targets/:id
                          GET  /v1/donation-config
                          PUT  /v1/donation-config/:project
```

**Data Flow:**

```
Request → Middleware (CORS → Rate Limit → Auth) → Handler → Query → DB
                                                     ↓
                                                   Cache (read/invalidate)
                                                     ↓
                                                  Response Envelope
```

**Service Boundaries:**
- Each service in `src/services/` is self-contained: routes + handlers + queries + types
- Services NEVER import from other services directly
- Shared concerns (auth, cache, errors) live in `middleware/` or `lib/`
- DB schema is centralized in `db/schema.ts` — services access it through queries only

**Data Boundaries:**
- `db/schema.ts` → single source of truth for all table definitions
- `db/index.ts` → single Drizzle client instance, exported for queries
- Service queries import from `db/` — handlers never touch DB directly
- Cache sits between handlers and responses — queries are always fresh

### Requirements to Structure Mapping

| Requirement Area | Primary Files |
|-----------------|---------------|
| Donation CRUD + Filtering + Pagination | `services/donations/routes.ts`, `handlers.ts`, `queries.ts`, `types.ts` |
| Public Summary (Verantwortungsseite) | `services/donations/routes.ts` (public prefix), `lib/cache.ts` |
| Auth & API Key System | `middleware/auth.ts`, `db/schema.ts` (api_keys table) |
| Donation Targets + Config | `services/donations/handlers.ts` (separate handler functions) |
| Rate Limiting | `middleware/rate-limit.ts` |
| CORS | `middleware/cors.ts` |
| Error Handling | `lib/errors.ts` |
| Caching + Invalidation | `lib/cache.ts` |
| DB Schema + Migrations | `db/schema.ts`, `drizzle/` folder |
| Seed Data (HoKi, API keys) | `db/seed.ts` |

### Integration Points

| Integration | Direction | Boundary File |
|-------------|-----------|---------------|
| NeonDB (PostgreSQL) | Outbound | `src/db/index.ts` |
| Upstash Redis | Outbound | `src/lib/cache.ts` |
| masem.at (consumer) | Inbound | Public API endpoints |
| hoki.help (consumer) | Inbound | Internal API endpoints |
| tellingcube.com (consumer) | Inbound | Internal API endpoints |
| chainsights.one (consumer) | Inbound | Internal API endpoints |

### Environment Variables Required

```
DATABASE_URL=              # NeonDB connection string (neon-http)
UPSTASH_REDIS_REST_URL=    # Upstash Redis HTTP endpoint
UPSTASH_REDIS_REST_TOKEN=  # Upstash Redis auth token
```

### Development Workflow

- **Local Development:** `vercel dev` → Hono app with Vercel runtime emulation
- **Migrations:** `npx drizzle-kit generate` → `npx drizzle-kit migrate` → commit SQL files
- **Testing:** `npx vitest` (watch mode) or `npx vitest run` (CI)
- **Deploy:** Push to `main` → Vercel auto-deploys to `api.masem.at`
- **Preview:** Push to branch → Vercel creates Preview Deployment URL

### Future Service Extension Pattern

Adding a new service (e.g., Users):
1. Add tables to `src/db/schema.ts`
2. Generate migration with `drizzle-kit generate`
3. Create service folder (`src/services/users/`) with routes/handlers/queries/types
4. Mount routes in `src/index.ts`

## Architecture Validation Results

### Coherence Validation

**Decision Compatibility:** All technology choices verified compatible:
- Express 5.2.1 + Vercel Node.js runtime → mature, reliable support
- Drizzle 0.45.1 + @neondatabase/serverless 1.0.2 → officially documented neon-http driver combo
- Upstash Redis 1.36.1 + Ratelimit 2.0.8 → same vendor, HTTP-based, no connection overhead
- Vitest + Supertest → standard Express testing pattern
- No version conflicts or incompatible patterns detected

**Pattern Consistency:** `snake_case` consistently used across all layers (DB → API Response → Query Params). No transformation layer required. Naming is coherent across all boundaries.

**Structure Alignment:** Feature-based organization with clear boundaries. Each service is self-contained. `schema.ts` as single source of truth prevents schema drift.

### Requirements Coverage Validation

| Requirement | Coverage |
|-------------|----------|
| Donations CRUD | services/donations/ + Drizzle queries |
| Filtering + Pagination | Query params + offset pagination in queries.ts |
| Public Summary (cached) | Cache layer + public route prefix |
| Internal Summary (source breakdown) | Separate handler, auth-protected |
| Donation Targets CRUD | Same service folder, separate handlers |
| Donation Config | Same service folder |
| API Key Auth + Scopes | middleware/auth.ts + api_keys table |
| Rate Limiting (100/min) | middleware/rate-limit.ts + Upstash |
| CORS (public vs internal) | middleware/cors.ts |
| Cache Invalidation | Cache-bust in POST handler |
| Performance (< 200ms / < 500ms) | Upstash cache + Fluid Compute |
| GDPR (Frankfurt) | NeonDB eu-central-1 |
| Zero incremental cost | All within existing plans/free tiers |

**No gaps found.** Every requirement maps to specific files and patterns.

### Implementation Readiness Validation

**Decision Completeness:** All critical decisions documented with verified versions. Implementation patterns are comprehensive. Consistency rules are clear and enforceable with concrete examples.

**Structure Completeness:** Complete project tree with every file and its purpose defined. All integration points specified. Component boundaries well-defined.

**Pattern Completeness:** All 12 potential conflict points addressed. Naming conventions cover DB, API, Code, and JSON layers. Process patterns (error handling, caching, middleware chain) fully specified.

### Gap Analysis Results

**Critical Gaps:** None.

**Important Observations:**
1. **`updated_at` Auto-Update** — Drizzle has no built-in auto-update trigger. Resolution: explicitly set `updated_at: new Date()` in queries.ts on every UPDATE operation. Simpler than a DB trigger, keeps logic visible.
2. **API Key Generation** — initial keys created via `db/seed.ts` which outputs plaintext once. No rotation mechanism for Phase 1 (deferred).

**Deferred (Post-MVP):**
- API Documentation (OpenAPI/Swagger)
- Structured Logging (pino)
- API Key Rotation Mechanism
- GitHub Actions CI Pipeline

### Architecture Completeness Checklist

**Requirements Analysis:**
- [x] Project context thoroughly analyzed
- [x] Scale and complexity assessed
- [x] Technical constraints identified
- [x] Cross-cutting concerns mapped

**Architectural Decisions:**
- [x] Critical decisions documented with versions
- [x] Technology stack fully specified
- [x] Integration patterns defined
- [x] Performance considerations addressed

**Implementation Patterns:**
- [x] Naming conventions established
- [x] Structure patterns defined
- [x] Process patterns documented
- [x] Enforcement guidelines with anti-patterns

**Project Structure:**
- [x] Complete directory structure defined
- [x] Component boundaries established
- [x] Integration points mapped
- [x] Requirements to structure mapping complete

### Architecture Readiness Assessment

**Overall Status:** READY FOR IMPLEMENTATION

**Confidence Level:** High — proven serverless stack (Express+Drizzle+Neon), clear patterns, manageable scope.

**Key Strengths:**
- Minimal stack with zero overhead — every dependency has a clear purpose
- Consistent patterns across all layers (snake_case, envelope, error format)
- Clear extensibility path for future services
- Zero incremental monthly cost

**Areas for Future Enhancement:**
- OpenAPI spec generation
- Structured logging (pino)
- Automated API key rotation
- GitHub Actions CI pipeline

### Implementation Handoff

**AI Agent Guidelines:**
- Follow all architectural decisions exactly as documented
- Use implementation patterns consistently across all components
- Respect project structure and boundaries
- Refer to this document for all architectural questions

**First Implementation Priority:**

Initialize Express project with required dependencies, then set up DB schema as defined in this document.
