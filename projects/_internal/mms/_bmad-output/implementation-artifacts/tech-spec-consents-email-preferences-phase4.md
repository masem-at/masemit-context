---
title: 'Consents & Email Preferences Service (Phase 4)'
slug: 'consents-email-preferences-phase4'
created: '2026-02-08'
status: 'review-complete'
stepsCompleted: [1, 2, 3, 4, 5, 6]
tech_stack: ['Express 5', 'Drizzle ORM 0.45', 'NeonDB (PostgreSQL)', 'Zod', 'Vitest', 'TypeScript']
files_to_modify:
  - 'mms-api/src/db/schema.ts'
  - 'mms-api/src/middleware/auth.ts'
  - 'mms-api/src/index.ts'
  - 'mms-api/src/docs/routes.ts'
  - 'mms-api/src/services/parties/types.ts'
  - 'mms-api/src/services/parties/handlers.ts'
  - 'mms-api/src/services/parties/queries.ts'
  - 'mms-api/src/services/parties/routes.ts'
  - 'mms-api/src/services/parties/index.ts'
  - 'mms-api/src/lib/crypto.ts'
files_to_create:
  - 'mms-api/src/services/consents/types.ts'
  - 'mms-api/src/services/consents/routes.ts'
  - 'mms-api/src/services/consents/handlers.ts'
  - 'mms-api/src/services/consents/queries.ts'
  - 'mms-api/src/services/consents/index.ts'
code_patterns:
  - 'Service 5-file pattern: types.ts, routes.ts, handlers.ts, queries.ts, index.ts'
  - 'Zod safeParse() for input validation, apiValidationError() for error response'
  - 'format*() helpers convert DB rows to API responses (Date → ISO string)'
  - 'getProjectFilter(req) for project isolation via API key'
  - 'db.transaction() for multi-step operations'
  - 'pgEnum for DB-level type safety, mirrored as z.enum in types.ts'
  - 'successResponse/createdResponse helpers wrap data in { data: T }'
  - 'apiError() with ERROR_CODES for consistent error responses'
  - 'isUniqueViolation(err) checks PostgreSQL error code 23505'
  - 'Barrel exports from index.ts'
test_patterns:
  - 'Vitest with vi.mock() for db and cache modules'
  - 'Tests co-located as *.test.ts alongside source files'
  - 'setupMocks() helper for chaining db.select/insert/update mocks'
  - 'npm test to run, npm run test:watch for dev'
---

# Tech-Spec: Consents & Email Preferences Service (Phase 4)

**Created:** 2026-02-08

## Overview

### Problem Statement

MMS lacks centralized GDPR-compliant consent tracking and email preference management across masemIT projects. There is no audit trail for consent changes, no mechanism for managing email subscriptions, and no unsubscribe flow. This creates legal risk (GDPR violations) and prevents unified consent management.

### Solution

Add `con_consents` and `con_email_preferences` tables with immutable consent records providing a full GDPR audit trail. Implement API endpoints for consent recording/querying and public token-based email preference management. Extend the existing `POST /v1/parties` handler to automatically create consent records and email preferences with unsubscribe tokens when `marketing_consent: true` is provided.

### Scope

**In Scope:**
- `con_consents` table with CHECK constraint, partial indexes, immutable insert pattern
- `con_email_preferences` table with unsubscribe tokens
- New PostgreSQL enums: `consent_type_enum`, `consent_scope_enum`
- `POST /v1/consents` — Record consent (grant or revoke)
- `GET /v1/parties/:id/consents` — Get all consents for a party
- `GET /v1/consents/check` — Check specific consent status
- `GET /v1/email-preferences/:token` — Get preferences by unsubscribe token (public)
- `POST /v1/email-preferences/unsubscribe` — Process unsubscribe (public)
- `PATCH /v1/email-preferences/:token` — Update preferences (public)
- Extend `POST /v1/parties` to create consent record + email preference + unsubscribe token
- New auth scopes: `consents:read`, `consents:write`
- API documentation at `/docs/consents`

**Out of Scope:**
- Unsubscribe HTML page / UI (each project implements their own)
- i18n / translations (project responsibility)
- Cookie consent banner UI
- Party merge functionality
- Admin API key with `parties:read:all` scope

## Context for Development

### Codebase Patterns

- **Service structure:** Each service follows 5-file pattern: `types.ts` (Zod schemas + response interfaces), `routes.ts` (Express Router with middleware chain), `handlers.ts` (request validation + orchestration), `queries.ts` (DB operations with format helpers), `index.ts` (barrel export)
- **Validation:** Zod `safeParse()` in handlers, `apiValidationError(res, parsed.error)` for 400 responses
- **Response format:** All responses wrapped in `{ data: T }` via `successResponse()`/`createdResponse()`. Dates always as ISO strings, never raw Date objects
- **Error handling:** `apiError(res, code, message, details?)` with standard ERROR_CODES. `isUniqueViolation(err)` for PostgreSQL 23505 duplicate errors
- **Project isolation:** `getProjectFilter(req)` extracts project from `req.apiKey!.project`, returns `undefined` for admin keys (no filter)
- **DB enums:** Defined as `pgEnum()` in `schema.ts`, mirrored as `z.enum()` in service `types.ts`
- **Transactions:** `db.transaction(async (tx) => { ... })` for multi-insert operations
- **Immutability:** Consent records are **never updated** — always INSERT new records for full GDPR audit trail
- **Auth middleware chain:** `authMiddleware` → `requireScope('scope:name')` → handler
- **Public endpoints:** No auth middleware — used for token-based access (email preferences)

### Files to Reference

| File | Purpose |
| ---- | ------- |
| `src/db/schema.ts` | All table definitions + pgEnums. Add `con_consents`, `con_email_preferences`, new enums here |
| `src/middleware/auth.ts` | `ApiScope` type union + `authMiddleware` + `requireScope()`. Add `consents:read \| consents:write` |
| `src/index.ts` | Express app setup, route registration. Add `app.use('/v1/consents', ...)` and `app.use('/v1/email-preferences', ...)` |
| `src/docs/routes.ts` | Inline markdown docs. Add consents doc + index entry |
| `src/lib/errors.ts` | `apiError()`, `apiValidationError()`, `formatZodError()`, `ERROR_CODES` — no changes needed |
| `src/lib/responses.ts` | `successResponse()`, `createdResponse()`, `listResponse()` — no changes needed |
| `src/lib/crypto.ts` | `sha256Hash()`, `generateSecureRandomString()`. Add `generateHexToken()` |
| `src/services/parties/types.ts` | `createPartySchema` — extend with optional consent fields |
| `src/services/parties/handlers.ts` | `handleCreateParty()` — extend to create consent + email preference |
| `src/services/parties/queries.ts` | `createPartyWithContactPoint()` — extend transaction to include consent records |
| `src/services/parties/routes.ts` | Add `GET /:id/consents` route |
| `src/services/parties/index.ts` | Barrel export — add new exports |

### Technical Decisions

- **Consent immutability:** Full audit trail for GDPR. Query latest consent via `ORDER BY created_at DESC LIMIT 1`. Never UPDATE consent records.
- **CHECK constraint:** `CHECK (party_id IS NOT NULL OR device_id IS NOT NULL)` — every consent must link to either a party or device
- **Unsubscribe tokens:** 64-char hex string via `crypto.getRandomValues(new Uint8Array(32))` — adapting existing `crypto.ts` pattern (Web Crypto API, not Node crypto)
- **consent_text_version:** String only (e.g., "v1.0"), actual text lives in Git repos
- **Separate auth scopes:** `consents:read` / `consents:write` — independent from `parties:read/write`
- **Phase 5 eliminated:** Unsubscribe page is project responsibility, MMS only provides the API
- **Two route groups:** `consentsRoutes` (auth-protected) mounted at `/v1/consents`, `emailPreferencesRoutes` (public, no auth) mounted at `/v1/email-preferences`. Both exported from same service.
- **Party consents route:** `GET /v1/parties/:id/consents` registered in `parties/routes.ts` but handler imported from consents service
- **sourceProjectEnum reuse:** `con_consents.project` column reuses existing `source_project_enum` for consistency
- **Scope-based filtering:** Consents filtered by project in queries, same pattern as parties. Admin key sees all.

## Implementation Plan

### Tasks

- [x] **Task 1: Add hex token generator to crypto utilities**
  - File: `src/lib/crypto.ts`
  - Action: Add `generateHexToken(bytes: number): string` function using Web Crypto API pattern matching existing code
  - Implementation: `crypto.getRandomValues(new Uint8Array(bytes))` → map to hex → join. Default 32 bytes = 64 char hex string
  - Notes: Follows existing `generateSecureRandomString()` pattern but outputs hex instead of alphanumeric

- [x] **Task 2: Add consent enums and tables to database schema**
  - File: `src/db/schema.ts`
  - Action: Add the following after the Party Tables section:
    - `consentTypeEnum` pgEnum: `['cookie_analytics', 'cookie_marketing', 'marketing_email', 'transactional_email']`
    - `consentScopeEnum` pgEnum: `['global', 'project']`
    - `preferenceTypeEnum` pgEnum: `['newsletter', 'product_updates', 'transactional']`
    - `conConsents` pgTable with columns: `id` (UUID PK), `party_id` (UUID FK nullable → ptyParties), `device_id` (varchar(64) nullable), `contact_point_id` (UUID FK nullable → ptyContactPoints), `consent_type` (consentTypeEnum NOT NULL), `scope` (consentScopeEnum NOT NULL default 'project'), `project` (sourceProjectEnum nullable), `granted` (boolean NOT NULL), `granted_at` (timestamp nullable), `revoked_at` (timestamp nullable), `ip_address` (varchar(45) nullable), `user_agent` (text nullable), `consent_text_version` (varchar(50) nullable), `created_at` (timestamp NOT NULL defaultNow)
    - Indexes on `conConsents`: `idx_con_consents_party` partial on `party_id` WHERE `party_id IS NOT NULL`, `idx_con_consents_device` partial on `device_id` WHERE `device_id IS NOT NULL`, `idx_con_consents_lookup` on `(party_id, consent_type, created_at DESC)`
    - CHECK constraint: `sql\`CHECK (party_id IS NOT NULL OR device_id IS NOT NULL)\``
    - `conEmailPreferences` pgTable with columns: `id` (UUID PK), `contact_point_id` (UUID FK NOT NULL → ptyContactPoints), `preference_type` (preferenceTypeEnum NOT NULL), `subscribed` (boolean NOT NULL default true), `unsubscribed_at` (timestamp nullable), `unsubscribe_token` (varchar(64) NOT NULL), `created_at` (timestamp NOT NULL defaultNow)
    - Indexes on `conEmailPreferences`: `idx_con_email_pref_token` uniqueIndex on `unsubscribe_token`, `idx_con_email_pref_cp` on `contact_point_id`
  - Notes: Follow exact column naming and type patterns from existing tables. All timestamps with `{ withTimezone: true }`

- [x] **Task 3: Generate and apply database migration**
  - Action: Run `npm run db:generate` then `npm run db:migrate` from `mms-api/` directory
  - Notes: Verify migration file is created with correct SQL. Check that CHECK constraint and partial indexes are included.

- [x] **Task 4: Extend auth middleware with consent scopes**
  - File: `src/middleware/auth.ts`
  - Action: Add `'consents:read' | 'consents:write'` to the `ApiScope` type union on line 16
  - Notes: No other changes needed — `requireScope()` and `authMiddleware` remain unchanged

- [x] **Task 5: Create consents service types**
  - File: `src/services/consents/types.ts` (NEW)
  - Action: Create with:
    - Zod enum mirrors: `consentTypeEnum` z.enum, `consentScopeEnum` z.enum, `preferenceTypeEnum` z.enum
    - Re-export `sourceProjectEnum` from parties types for convenience
    - `recordConsentSchema` z.object: `party_id` (uuid optional), `device_id` (string max 64 optional), `contact_point_id` (uuid optional), `consent_type` (consentTypeEnum), `scope` (consentScopeEnum default 'project'), `project` (sourceProjectEnum optional), `granted` (boolean), `ip_address` (string max 45 optional), `user_agent` (string optional), `consent_text_version` (string max 50 optional). Add `.refine()` to enforce party_id OR device_id present
    - `checkConsentQuerySchema` z.object: `party_id` (uuid optional), `device_id` (string optional), `consent_type` (consentTypeEnum), `project` (sourceProjectEnum optional). Add `.refine()` requiring party_id or device_id
    - `unsubscribeSchema` z.object: `token` (string min 1), `preference_type` (preferenceTypeEnum)
    - `updateEmailPreferenceSchema` z.object: `preference_type` (preferenceTypeEnum), `subscribed` (boolean)
    - Response interfaces: `Consent` (all fields as strings/booleans, dates as ISO strings or null), `EmailPreference` (id, contact_point_id, preference_type, subscribed, unsubscribed_at, unsubscribe_token, created_at), `ConsentCheckResult` (consent_type, granted, last_updated)
    - Type exports for all inputs: `RecordConsentInput`, `CheckConsentQuery`, `UnsubscribeInput`, `UpdateEmailPreferenceInput`

- [x] **Task 6: Create consents service queries**
  - File: `src/services/consents/queries.ts` (NEW)
  - Action: Create with:
    - Imports: `db` from `../../db/index.js`, schema tables from `../../db/schema.js`, drizzle operators
    - `formatConsent()` helper: converts DB row to `Consent` response type (dates → ISO strings)
    - `formatEmailPreference()` helper: converts DB row to `EmailPreference` response type
    - `recordConsent(data: RecordConsentInput): Promise<Consent>` — INSERT into `conConsents`, set `granted_at = new Date()` if granted, `revoked_at = new Date()` if not granted. Return formatted consent
    - `getConsentsForParty(partyId: string, project?: SourceProject): Promise<Consent[]>` — SELECT from `conConsents` WHERE party_id = partyId, optionally filter by project (include scope='global' results too). ORDER BY created_at DESC
    - `checkConsent(query: CheckConsentQuery): Promise<ConsentCheckResult | null>` — SELECT latest consent matching party_id/device_id + consent_type + optional project. ORDER BY created_at DESC LIMIT 1. Return `{ consent_type, granted, last_updated }` or null
    - `getEmailPreferencesByToken(token: string): Promise<EmailPreference | null>` — SELECT from `conEmailPreferences` WHERE unsubscribe_token = token. LIMIT 1. Return formatted or null
    - `processUnsubscribe(token: string, preferenceType: string): Promise<EmailPreference | null>` — SELECT preference by token AND preference_type. If not found return null. UPDATE subscribed=false, unsubscribed_at=new Date(). Return formatted result
    - `updateEmailPreference(token: string, data: UpdateEmailPreferenceInput): Promise<EmailPreference | null>` — SELECT preference by token AND preference_type. If not found return null. UPDATE subscribed=data.subscribed, set/clear unsubscribed_at based on subscribed value. Return formatted result
    - `createEmailPreferenceForContact(contactPointId: string, preferenceType: string): Promise<{ unsubscribe_token: string }>` — Generate hex token via `generateHexToken(32)`, INSERT into `conEmailPreferences`. Return token. Used by parties handler extension.
  - Notes: All queries follow existing pattern from `parties/queries.ts`. Consent records are NEVER updated — `recordConsent` always INSERTs. Email preferences CAN be updated (subscribed toggle).

- [x] **Task 7: Create consents service handlers**
  - File: `src/services/consents/handlers.ts` (NEW)
  - Action: Create with:
    - Imports: types from `./types.js`, queries from `./queries.js`, error/response helpers from `../../lib/`
    - `getProjectFilter(req)` helper — same pattern as parties handlers
    - `handleRecordConsent(req, res)` — Validate body with `recordConsentSchema`. If `party_id` provided, verify party exists and belongs to project via `getPartyById()` import from parties. If `contact_point_id` provided, verify it belongs to the party. INSERT consent. Return 201 with formatted consent.
    - `handleGetPartyConsents(req, res)` — Validate `:id` param as UUID. Get project filter. Query consents for party. Return 200 with array.
    - `handleCheckConsent(req, res)` — Validate query params with `checkConsentQuerySchema`. Query latest consent. Return 200 with result, or 404 if no consent found.
    - `handleGetEmailPreferences(req, res)` — Extract `:token` param. Query by token. Return 200 with preference, or 404 if invalid token.
    - `handleUnsubscribe(req, res)` — Validate body with `unsubscribeSchema`. Process unsubscribe. Return 200 (idempotent — if already unsubscribed, still return 200). Return 404 if token invalid.
    - `handleUpdateEmailPreferences(req, res)` — Extract `:token` param. Validate body with `updateEmailPreferenceSchema`. Update preference. Return 200 with updated preference, or 404 if token invalid.
  - Notes: Error handling follows exact pattern from `parties/handlers.ts` — try/catch, console.error for unexpected errors, apiError for expected errors

- [x] **Task 8: Create consents service routes**
  - File: `src/services/consents/routes.ts` (NEW)
  - Action: Create with two routers:
    - `consentsRouter` (auth-protected):
      - `POST /` → `authMiddleware`, `requireScope('consents:write')`, `handleRecordConsent`
      - `GET /check` → `authMiddleware`, `requireScope('consents:read')`, `handleCheckConsent` (BEFORE /:id routes if any)
    - `emailPreferencesRouter` (public, NO auth):
      - `GET /:token` → `handleGetEmailPreferences`
      - `POST /unsubscribe` → `handleUnsubscribe` (BEFORE /:token to avoid matching "unsubscribe" as token)
      - `PATCH /:token` → `handleUpdateEmailPreferences`
    - Export both as `consentsRoutes` and `emailPreferencesRoutes`
  - Notes: Email preferences routes are PUBLIC — no `authMiddleware`. Order matters: `/unsubscribe` before `/:token`.

- [x] **Task 9: Create consents service barrel export**
  - File: `src/services/consents/index.ts` (NEW)
  - Action: Create barrel export for all handlers, queries, types, and both route exports
  - Notes: Follow exact pattern from `parties/index.ts`

- [x] **Task 10: Extend parties service for consent creation on party creation**
  - Files: `src/services/parties/types.ts`, `src/services/parties/queries.ts`, `src/services/parties/handlers.ts`, `src/services/parties/index.ts`
  - Action in `types.ts`:
    - Extend `createPartySchema` with optional fields: `marketing_consent` (boolean optional), `consent_ip` (string max 45 optional), `consent_user_agent` (string optional), `consent_text_version` (string max 50 optional)
    - Extend `CreatePartyResponse` interface with optional `unsubscribe_token: string | null`
  - Action in `queries.ts`:
    - Extend `createPartyWithContactPoint()` to accept extended input
    - Inside the transaction: if `marketing_consent` is true, INSERT a `con_consents` record (consent_type='marketing_email', scope='project', granted=true, with ip/user_agent/version) AND INSERT a `con_email_preferences` record (preference_type='newsletter', subscribed=true, generate unsubscribe token)
    - Return `unsubscribe_token` in the result (or null if no consent)
    - Import `conConsents`, `conEmailPreferences` from schema and `generateHexToken` from crypto
  - Action in `handlers.ts`:
    - Update `handleCreateParty` response to include `unsubscribe_token` from query result
    - For existing party (find-or-create returns existing): set `unsubscribe_token: null` in response
  - Action in `index.ts`:
    - Add any new type exports
  - Notes: The transaction ensures party + contact point + consent + email preference are all created atomically. If party already exists (find-or-create), no new consent is created — just returns existing party with `unsubscribe_token: null`.

- [x] **Task 11: Register routes in main application**
  - File: `src/index.ts`
  - Action:
    - Add imports: `import { consentsRoutes, emailPreferencesRoutes } from './services/consents/routes.js'`
    - Add route registrations after parties routes:
      - `app.use('/v1/consents', consentsRoutes)`
      - `app.use('/v1/email-preferences', emailPreferencesRoutes)`
  - File: `src/services/parties/routes.ts`
  - Action:
    - Add import: `import { handleGetPartyConsents } from '../consents/handlers.js'`
    - Add route: `router.get('/:id/consents', authMiddleware, requireScope('consents:read'), handleGetPartyConsents)`
    - Place AFTER the `GET /:id` route
  - Notes: `GET /v1/parties/:id/consents` stays under parties URL namespace but uses handler from consents service. Requires `consents:read` scope (not `parties:read`).

- [x] **Task 12: Add API documentation**
  - File: `src/docs/routes.ts`
  - Action:
    - Add `CONSENTS_DOC` markdown constant covering:
      - Authentication section (scopes: `consents:read`, `consents:write`)
      - Enums section (consent_type, consent_scope, preference_type)
      - Consent endpoints: POST /v1/consents, GET /v1/parties/:id/consents, GET /v1/consents/check
      - Email Preferences endpoints (public): GET /v1/email-preferences/:token, POST /v1/email-preferences/unsubscribe, PATCH /v1/email-preferences/:token
      - Request/Response examples for each endpoint
      - Note about consent immutability
      - Note about public endpoints (no auth required for email preferences)
    - Add index entry: `{ name: 'consents', path: '/docs/consents', description: 'Consents & Email Preferences API - GDPR consent tracking and email subscription management' }`
    - Add route: `router.get('/consents', ...)` serving `CONSENTS_DOC`
  - Notes: Follow exact documentation style from existing DONATIONS_DOC and PARTIES_DOC

- [x] **Task 13: Run database migration**
  - Action: From `mms-api/` run `npm run db:generate` then `npm run db:migrate`
  - Notes: Verify generated SQL includes CHECK constraint, partial indexes, and all enum types. If Drizzle doesn't support CHECK constraints natively, add raw SQL in migration file.

- [x] **Task 14: Update seed script for consent scopes**
  - File: `src/db/seed.ts`
  - Action: Update seed API keys to include `consents:read` and `consents:write` scopes where appropriate
  - Notes: At minimum, the admin key should have all consent scopes. Project keys should have both scopes too.

- [x] **Task 15: Write tests**
  - File: `src/services/consents/handlers.test.ts` (NEW)
  - Action: Write Vitest tests covering:
    - `POST /v1/consents`: valid consent recording, missing party_id AND device_id (400), party not found (404), consent with device_id only (cookie consent)
    - `GET /v1/parties/:id/consents`: valid party returns consents, party not found returns 404, project isolation works
    - `GET /v1/consents/check`: valid check returns latest consent, no consent found returns 404
    - `GET /v1/email-preferences/:token`: valid token returns preference, invalid token returns 404
    - `POST /v1/email-preferences/unsubscribe`: valid unsubscribe, already unsubscribed (idempotent 200), invalid token (404)
    - `PATCH /v1/email-preferences/:token`: valid update (subscribe/unsubscribe), invalid token (404)
  - Notes: Follow test pattern from existing `handlers.test.ts` files. Mock db and crypto modules.

### Acceptance Criteria

**Consent Recording:**
- [x] AC 1: Given a valid API key with `consents:write` scope, when POST /v1/consents is called with party_id and consent_type, then a new consent record is created and returned with 201 status
- [x] AC 2: Given a valid API key, when POST /v1/consents is called with device_id (no party_id) for cookie consent, then a consent record is created linked to the device
- [x] AC 3: Given a request without both party_id and device_id, when POST /v1/consents is called, then a 400 validation error is returned
- [x] AC 4: Given a consent is revoked (granted=false), when the record is created, then revoked_at is set and granted_at is null. The original granted consent record remains unchanged (immutability).

**Consent Querying:**
- [x] AC 5: Given a party with multiple consents, when GET /v1/parties/:id/consents is called, then all consents are returned ordered by created_at DESC
- [x] AC 6: Given a non-admin API key, when querying consents, then only consents for the key's project (plus scope='global') are returned
- [x] AC 7: Given a party_id and consent_type, when GET /v1/consents/check is called, then the latest consent status is returned (granted true/false + last_updated timestamp)
- [x] AC 8: Given no consent exists for the query, when GET /v1/consents/check is called, then 404 is returned

**Email Preferences (Public):**
- [x] AC 9: Given a valid unsubscribe_token, when GET /v1/email-preferences/:token is called (no auth), then the email preference is returned with 200
- [x] AC 10: Given an invalid token, when any email preference endpoint is called, then 404 is returned
- [x] AC 11: Given a valid token and preference_type, when POST /v1/email-preferences/unsubscribe is called, then subscribed is set to false and unsubscribed_at is set
- [x] AC 12: Given an already unsubscribed preference, when POST /v1/email-preferences/unsubscribe is called again, then 200 is returned (idempotent)
- [x] AC 13: Given a valid token, when PATCH /v1/email-preferences/:token is called with subscribed=true, then the user is re-subscribed and unsubscribed_at is cleared

**Party Creation Integration:**
- [x] AC 14: Given POST /v1/parties with marketing_consent=true, when a new party is created, then a consent record (marketing_email, granted=true) AND an email preference (newsletter, subscribed=true) with unsubscribe_token are created in the same transaction
- [x] AC 15: Given POST /v1/parties with marketing_consent=true, when the party already exists (find-or-create), then no new consent/preference is created and unsubscribe_token is null in the response
- [x] AC 16: Given POST /v1/parties without marketing_consent, when a party is created, then no consent or email preference records are created

**Auth & Security:**
- [x] AC 17: Given an API key without `consents:write` scope, when POST /v1/consents is called, then 403 Forbidden is returned
- [x] AC 18: Given no API key, when email preference endpoints are called (GET/POST/PATCH), then they work without auth (public endpoints)
- [x] AC 19: Given a non-admin API key, when querying party consents, then consents from other projects are not visible (except scope='global')

## Additional Context

### Dependencies

- **Phase 2 (Parties) must be deployed** — con_consents has FK to pty_parties and pty_contact_points. Phase 2 is already complete and deployed.
- **No new npm packages required** — all functionality uses existing dependencies (Drizzle, Zod, Express, Web Crypto API)
- **Database migration required** — new tables + enums need to be generated and applied via Drizzle

### Testing Strategy

- **Unit tests:** Handler tests with mocked DB (Vitest + vi.mock), covering all endpoints and error paths. Follow existing pattern from `donations/handlers.test.ts`.
- **Manual testing:** Use curl/httpie against local dev server (`npm run dev`). Test full flow: create party with consent → query consent → check consent → get email preferences → unsubscribe → re-subscribe
- **Migration verification:** After `db:migrate`, verify tables exist and constraints work by inserting test data via Drizzle Studio (`npm run db:studio`)
- **Integration check:** Verify `POST /v1/parties` with `marketing_consent: true` creates all records atomically (party + contact point + consent + email preference)

### Notes

- **GDPR Critical:** Consent immutability is not negotiable. The implementation MUST never UPDATE con_consents records. Always INSERT new records to maintain full audit trail.
- **CHECK constraint in Drizzle:** Drizzle ORM may not natively support table-level CHECK constraints in `pgTable()`. If so, add the CHECK constraint as raw SQL in the generated migration file before applying.
- **Token collision:** While 64-char hex tokens (256 bits of entropy) have negligible collision risk, the unique index on `unsubscribe_token` provides a safety net. If a collision occurs on insert, the unique violation is caught and retried.
- **Future consideration:** When party merge is implemented, consent records should be migrated to the surviving party_id. Email preference tokens remain valid since they reference contact_point_id, not party_id directly.
- **Seed data:** After migration, update seed script to include consent scopes in test API keys. This is needed for local development and testing.

### Adversarial Review (Step 5–6)

Review completed with 18 findings. 6 classified as false positives/deferred, 12 fixed automatically:

| ID | Severity | Finding | Resolution |
|----|----------|---------|------------|
| F1 | Critical | TOCTOU race in processUnsubscribe/updateEmailPreference | Wrapped in `db.transaction()` |
| F2 | Critical | Public endpoints missing rate limiting | Added Express rate limiter (20 req/min per IP, Upstash Redis, fail-open) |
| F3 | High | project not enforced from auth key in handleRecordConsent | Non-admin keys always override project from API key |
| F4 | High | scope:'project' allowed without project value | Handler validates project required when scope='project' |
| F5 | High | Missing unique constraint on (contact_point_id, preference_type) | Added `con_email_pref_cp_type_uniq` constraint in schema + Neon migration |
| F6 | Medium | device_id missing .max(64) in checkConsentQuerySchema | Added `.max(64)` |
| F7 | Medium | ip_address not validated as IP | Added `.max(45).regex(/^[\d.:a-fA-F]+$/)` (Zod 4 lacks `.ip()`) |
| F8 | Medium | user_agent no max length | Added `.max(2048)` |
| F9 | Medium | Unsafe `as` type casts in queries | Changed to `PreferenceType` type parameter, removed all `as` casts |
| F10 | Medium | checkConsent AND vs OR for party_id/device_id | Changed to `or()` for identifier conditions |
| F11 | Low | unsubscribe_token leaked in GET response | Stripped from GET /email-preferences/:token response via destructuring |
| F12 | Low | No audit trail for email preference changes | Added `console.info` logging with IP, user-agent, preference details |

**False Positives / Deferred (not fixed):**
- F13: consent_text_version not enforced → Business policy, not enforcement
- F14: No pagination on getConsentsForParty → Low volume, premature optimization
- F15: unsubscribe_token collision handling → 256-bit entropy + unique index, negligible risk
- F16: Missing OpenAPI spec → Out of scope for this phase
- F17: No integration tests → Separate test infrastructure work
- F18: Rate-limit.ts still Hono-based → Pre-existing, separate migration task
