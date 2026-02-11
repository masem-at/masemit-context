# Story 3.2: Database Schema & Integration Clients

Status: done

## Story

As a **founder (Mario)**,
I want the database schema and external service clients ready,
so that the waitlist API route can store leads and sync them with my CRM.

## Acceptance Criteria

1. **Given** no database schema exists yet **When** the database layer is set up **Then** `lib/db/index.ts` creates a Drizzle instance connected to NeonDB via `@neondatabase/serverless` **And** `lib/db/schema.ts` defines the `waitlist_entries` table with all columns per architecture spec (id, mms_party_id, mms_contact_point_id, company_name, gewerk, company_size, phone, message, utm_source, utm_medium, utm_campaign, confirmation_token, confirmed_at, created_at, updated_at) **And** indexes are created: `idx_waitlist_entries_mms_party_id`, `idx_waitlist_entries_confirmation_token` **And** drizzle-kit generates a migration file committed to git **And** `drizzle.config.ts` is configured for NeonDB

2. **Given** the database is ready **When** the MMS API client is implemented (`lib/mms.ts`) **Then** it calls `POST /v1/parties` with: email, displayName, source_project "kurzum_app", marketing_consent true, consent IP and user agent **And** it returns partyId, contactPointId, created boolean, and unsubscribeToken **And** it authenticates via `x-api-key` header using `MMS_API_KEY` env variable **And** MMS errors are caught and logged without exposing details to the user

3. **Given** security utilities are needed **When** the Turnstile verification helper is implemented (`lib/turnstile.ts`) **Then** it verifies Cloudflare Turnstile tokens server-side via the siteverify API **And** it returns a boolean indicating valid/invalid **And** it uses `TURNSTILE_SECRET_KEY` env variable

4. **Given** rate limiting is needed **When** the rate limiter is implemented (`lib/rate-limit.ts`) **Then** it uses `@upstash/ratelimit` with a sliding window of 5 requests per IP per hour **And** it uses `UPSTASH_REDIS_REST_URL` and `UPSTASH_REDIS_REST_TOKEN` env variables

5. **Given** a unified API response format is needed **When** `lib/api-response.ts` is implemented **Then** it exports the `ApiResponse<T>` type and helper functions for success/error responses **And** it defines standard error codes: `VALIDATION_ERROR`, `BOT_DETECTED`, `RATE_LIMITED`, `INTERNAL_ERROR`

## Tasks / Subtasks

- [x] Task 1: Update Drizzle DB Instance (AC: #1)
  - [x] 1.1 Update `lib/db/index.ts` — import schema, pass to drizzle instance for relational queries
  - [x] 1.2 Verify `drizzle.config.ts` — ensure `out` path is `./drizzle` and schema path is `./lib/db/schema.ts`

- [x] Task 2: Define waitlist_entries Schema (AC: #1)
  - [x] 2.1 Update `lib/db/schema.ts` — define `waitlist_entries` table using Drizzle `pgTable`
  - [x] 2.2 Define ALL 15 columns with correct Drizzle types (see Dev Notes for exact schema)
  - [x] 2.3 Add `idx_waitlist_entries_mms_party_id` index on `mms_party_id`
  - [x] 2.4 Add `idx_waitlist_entries_confirmation_token` index on `confirmation_token`
  - [x] 2.5 Export inferred types: `export type WaitlistEntry = typeof waitlistEntries.$inferSelect` and `export type NewWaitlistEntry = typeof waitlistEntries.$inferInsert`

- [x] Task 3: Generate Migration (AC: #1)
  - [x] 3.1 Run `pnpm drizzle-kit generate` to create SQL migration file
  - [x] 3.2 Verify generated SQL matches expected schema (inspect migration file)
  - [x] 3.3 Migration file must be committed to git (in `./drizzle/` directory)

- [x] Task 4: Create ApiResponse Helper (AC: #5)
  - [x] 4.1 Create `lib/api-response.ts` with `ApiResponse<T>` type
  - [x] 4.2 Define `ErrorCode` union type: `VALIDATION_ERROR | BOT_DETECTED | RATE_LIMITED | INTERNAL_ERROR | NOT_FOUND`
  - [x] 4.3 Export `successResponse<T>(data: T)` helper — returns `{ success: true, data }`
  - [x] 4.4 Export `errorResponse(code: ErrorCode, message: string)` helper — returns `{ success: false, error: { code, message } }`
  - [x] 4.5 Export HTTP status code mapper: `errorCodeToStatus(code: ErrorCode): number`

- [x] Task 5: Create Turnstile Verification Helper (AC: #3)
  - [x] 5.1 Create `lib/turnstile.ts` with `verifyTurnstileToken(token: string, ip?: string): Promise<boolean>`
  - [x] 5.2 POST to `https://challenges.cloudflare.com/turnstile/v0/siteverify` with `secret` + `response` + optional `remoteip`
  - [x] 5.3 Use `TURNSTILE_SECRET_KEY` env variable
  - [x] 5.4 Return `true` if `outcome.success === true`, `false` otherwise
  - [x] 5.5 Catch and log network errors, return `false` (fail closed)

- [x] Task 6: Create Rate Limiter (AC: #4)
  - [x] 6.1 Create `lib/rate-limit.ts` with `ratelimit` export using `@upstash/ratelimit`
  - [x] 6.2 Configure `Ratelimit.slidingWindow(5, "1 h")` — 5 per IP per hour
  - [x] 6.3 Use `Redis.fromEnv()` for automatic env variable binding (`UPSTASH_REDIS_REST_URL`, `UPSTASH_REDIS_REST_TOKEN`)
  - [x] 6.4 Export `checkRateLimit(ip: string): Promise<{ success: boolean; remaining: number }>`

- [x] Task 7: Create MMS API Client (AC: #2)
  - [x] 7.1 Create `lib/mms.ts` with typed MMS API functions
  - [x] 7.2 Define `MMS_BASE_URL = "https://api.masem.at/v1"`
  - [x] 7.3 Implement `createParty(data)` — POST `/v1/parties` with email, displayName, source_project, marketing_consent, consent IP/UA
  - [x] 7.4 Implement `updatePartyStatus(partyId, status)` — PATCH `/v1/parties/:id` (needed by Story 3.4)
  - [x] 7.5 Authenticate via `x-api-key` header using `MMS_API_KEY` env variable
  - [x] 7.6 Type the response: `{ partyId: string; contactPointId: string; created: boolean; unsubscribeToken: string | null }`
  - [x] 7.7 Catch errors, log in English (`console.error`), throw typed errors for API route to handle

- [x] Task 8: Verify Build & Lint (AC: all)
  - [x] 8.1 Run `pnpm build` — zero errors
  - [x] 8.2 Run `pnpm lint` — zero errors
  - [x] 8.3 TypeScript produces zero type errors

## Dev Notes

### Critical: Existing Files — DO NOT RECREATE

These files already exist from Story 1.1 initialization. **MODIFY** them, do NOT delete and recreate:

| File | Current State | Action |
|------|--------------|--------|
| `lib/db/index.ts` | Minimal drizzle instance (no schema import) | **UPDATE** — add schema import |
| `lib/db/schema.ts` | Empty placeholder comment | **UPDATE** — add waitlist_entries table |
| `drizzle.config.ts` | Configured for NeonDB, `out: './drizzle'` | **VERIFY** — should be correct as-is |
| `lib/validations/waitlist.ts` | Complete Zod schema (Story 3.1) | **DO NOT TOUCH** — already done |

### Critical: Exact Drizzle ORM Setup (v0.45.1)

The existing `lib/db/index.ts` uses the simplified API pattern. Update it to include schema:

```typescript
// lib/db/index.ts — UPDATE (do not recreate)
import { drizzle } from "drizzle-orm/neon-http";
import * as schema from "./schema";

export const db = drizzle(process.env.DATABASE_URL!, { schema });
```

The `{ schema }` parameter enables Drizzle's relational query API (`db.query.waitlistEntries.findMany()`). The simplified `drizzle(url)` pattern internally creates a `neon()` HTTP client — no need to import `@neondatabase/serverless` directly.

### Critical: Exact waitlist_entries Schema

```typescript
// lib/db/schema.ts — UPDATE (do not recreate)
import { pgTable, serial, text, uuid, timestamp, index } from "drizzle-orm/pg-core";

export const waitlistEntries = pgTable(
  "waitlist_entries",
  {
    id: serial("id").primaryKey(),
    mmsPartyId: uuid("mms_party_id").notNull(),
    mmsContactPointId: uuid("mms_contact_point_id").notNull(),
    companyName: text("company_name").notNull(),
    gewerk: text("gewerk").notNull(),
    companySize: text("company_size").notNull(),
    phone: text("phone"),
    message: text("message"),
    utmSource: text("utm_source"),
    utmMedium: text("utm_medium"),
    utmCampaign: text("utm_campaign"),
    confirmationToken: uuid("confirmation_token").defaultRandom().notNull().unique(),
    confirmedAt: timestamp("confirmed_at", { withTimezone: true }),
    createdAt: timestamp("created_at", { withTimezone: true }).defaultNow().notNull(),
    updatedAt: timestamp("updated_at", { withTimezone: true }).defaultNow().notNull(),
  },
  (table) => [
    index("idx_waitlist_entries_mms_party_id").on(table.mmsPartyId),
    index("idx_waitlist_entries_confirmation_token").on(table.confirmationToken),
  ]
);

export type WaitlistEntry = typeof waitlistEntries.$inferSelect;
export type NewWaitlistEntry = typeof waitlistEntries.$inferInsert;
```

**NAMING CONVENTION:** Table name is `snake_case` plural (`waitlist_entries`), column names are `snake_case` in DB but `camelCase` in TypeScript (Drizzle handles the mapping via the string argument).

### Critical: ApiResponse Type Pattern

```typescript
// lib/api-response.ts — NEW
export type ErrorCode =
  | "VALIDATION_ERROR"
  | "BOT_DETECTED"
  | "RATE_LIMITED"
  | "INTERNAL_ERROR"
  | "NOT_FOUND";

export type ApiResponse<T> =
  | { success: true; data: T }
  | { success: false; error: { code: ErrorCode; message: string } };

export function successResponse<T>(data: T): Response {
  return Response.json({ success: true, data } satisfies ApiResponse<T>);
}

export function errorResponse(
  code: ErrorCode,
  message: string,
  status?: number
): Response {
  return Response.json(
    { success: false, error: { code, message } } satisfies ApiResponse<never>,
    { status: status ?? errorCodeToStatus(code) }
  );
}

function errorCodeToStatus(code: ErrorCode): number {
  const map: Record<ErrorCode, number> = {
    VALIDATION_ERROR: 400,
    BOT_DETECTED: 403,
    RATE_LIMITED: 429,
    INTERNAL_ERROR: 500,
    NOT_FOUND: 404,
  };
  return map[code];
}
```

### Critical: Turnstile Server-Side Verification

```typescript
// lib/turnstile.ts — NEW
const VERIFY_URL = "https://challenges.cloudflare.com/turnstile/v0/siteverify";

export async function verifyTurnstileToken(
  token: string,
  ip?: string
): Promise<boolean> {
  try {
    const body: Record<string, string> = {
      secret: process.env.TURNSTILE_SECRET_KEY!,
      response: token,
    };
    if (ip) body.remoteip = ip;

    const res = await fetch(VERIFY_URL, {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify(body),
    });
    const data = await res.json();
    return data.success === true;
  } catch (error) {
    console.error("Turnstile verification failed:", error);
    return false;
  }
}
```

**CRITICAL:** Tokens expire after 300 seconds (5 min). Each token can only be validated ONCE. Do NOT retry verification with the same token.

### Critical: Upstash Rate Limiter (v2.0.8)

```typescript
// lib/rate-limit.ts — NEW
import { Ratelimit } from "@upstash/ratelimit";
import { Redis } from "@upstash/redis";

const ratelimit = new Ratelimit({
  redis: Redis.fromEnv(),
  limiter: Ratelimit.slidingWindow(5, "1 h"),
  prefix: "kurzum:waitlist",
});

export async function checkRateLimit(
  ip: string
): Promise<{ success: boolean; remaining: number }> {
  const { success, remaining } = await ratelimit.limit(ip);
  return { success, remaining };
}
```

`Redis.fromEnv()` automatically reads `UPSTASH_REDIS_REST_URL` and `UPSTASH_REDIS_REST_TOKEN`. The `prefix` prevents key collisions if Redis is shared.

### Critical: MMS API Client

```typescript
// lib/mms.ts — NEW
const MMS_BASE_URL = "https://api.masem.at/v1";

interface CreatePartyRequest {
  email: string;
  displayName: string;
  sourceProject: string;
  marketingConsent: boolean;
  consentIp: string;
  consentUserAgent: string;
}

export interface CreatePartyResponse {
  partyId: string;
  contactPointId: string;
  created: boolean;
  unsubscribeToken: string | null;
}

export async function createParty(
  data: CreatePartyRequest
): Promise<CreatePartyResponse> {
  const res = await fetch(`${MMS_BASE_URL}/parties`, {
    method: "POST",
    headers: {
      "Content-Type": "application/json",
      "x-api-key": process.env.MMS_API_KEY!,
    },
    body: JSON.stringify({
      email: data.email,
      display_name: data.displayName,
      source_project: data.sourceProject,
      marketing_consent: data.marketingConsent,
      consent_ip: data.consentIp,
      consent_user_agent: data.consentUserAgent,
    }),
  });

  if (!res.ok) {
    const errorBody = await res.text().catch(() => "Unknown error");
    console.error(`MMS API error (${res.status}):`, errorBody);
    throw new Error(`MMS API request failed with status ${res.status}`);
  }

  const body = await res.json();
  return {
    partyId: body.party_id,
    contactPointId: body.contact_point_id,
    created: body.created,
    unsubscribeToken: body.unsubscribe_token ?? null,
  };
}

export async function updatePartyStatus(
  partyId: string,
  status: "lead" | "prospect" | "customer"
): Promise<void> {
  const res = await fetch(`${MMS_BASE_URL}/parties/${partyId}`, {
    method: "PATCH",
    headers: {
      "Content-Type": "application/json",
      "x-api-key": process.env.MMS_API_KEY!,
    },
    body: JSON.stringify({ status }),
  });

  if (!res.ok) {
    const errorBody = await res.text().catch(() => "Unknown error");
    console.error(`MMS API status update error (${res.status}):`, errorBody);
    throw new Error(`MMS API status update failed with status ${res.status}`);
  }
}
```

**NAMING:** MMS API uses `snake_case` (e.g. `display_name`, `source_project`). The client translates to/from `camelCase` at the boundary. **Authentication:** `x-api-key` header with `MMS_API_KEY` env var. **Error handling:** Log in English, throw typed error. The API route (Story 3.3) will catch and return German user-facing messages.

### Critical: Migration Generation

After defining the schema, run:
```bash
pnpm drizzle-kit generate
```

This creates a SQL migration in `./drizzle/`. Inspect it to verify the generated SQL matches the schema. **DO NOT** run `drizzle-kit push` against production — only `generate` to create version-controlled migration files.

### Critical: Environment Variables (Story 3.2 uses)

| Variable | Used By | Notes |
|----------|---------|-------|
| `DATABASE_URL` | `lib/db/index.ts` | Already in `.env.example` |
| `MMS_API_KEY` | `lib/mms.ts` | Already in `.env.example` |
| `TURNSTILE_SECRET_KEY` | `lib/turnstile.ts` | Already in `.env.example` |
| `UPSTASH_REDIS_REST_URL` | `lib/rate-limit.ts` | Already in `.env.example` |
| `UPSTASH_REDIS_REST_TOKEN` | `lib/rate-limit.ts` | Already in `.env.example` |

All variables are already in `.env.example`. **Reminder:** `.env.local` does not exist yet — the developer must create it with real values for local testing.

### Previous Story Learnings (Story 3.1 + Code Review)

1. **ESLint `no-empty-object-type`** — Do NOT export empty interfaces. Only export interfaces that have properties.
2. **`as any` cast for zodResolver** — Known `@hookform/resolvers@5.2.2` + `zod@4.3.6` type mismatch. Runtime works fine.
3. **Turnstile `options` prop** — `theme`/`language`/`size` go in `options={{ ... }}`, NOT as top-level props.
4. **aria-describedby** — Always link error messages to inputs via `id` + `aria-describedby`.
5. **`min-h-12`** — Use `min-h-12` (48px) for touch targets, not `min-h-touch`.
6. **Phone validation** — Uses `refine()` with regex, not native Zod phone validator (doesn't exist in v4).
7. **Error messages** — German for users, English for logs. Be specific: "Die Anmeldung konnte nicht gesendet werden."

### Installed Package Versions

| Package | Version | Notes |
|---------|---------|-------|
| `drizzle-orm` | 0.45.1 | Use `drizzle-orm/neon-http` driver |
| `drizzle-kit` | 0.31.9 (dev) | For migration generation |
| `@neondatabase/serverless` | 1.0.2 | Used internally by drizzle-orm/neon-http |
| `@upstash/ratelimit` | 2.0.8 | `Ratelimit.slidingWindow()` API |
| `@upstash/redis` | 1.36.2 | `Redis.fromEnv()` for auto config |
| `resend` | 6.9.2 | NOT used in this story (Story 3.4) |
| `zod` | 4.3.6 | Schema already exists from Story 3.1 |

### Project Structure (After This Story)

```
lib/
├── db/
│   ├── index.ts               # MODIFIED: Add schema import for relational queries
│   ├── schema.ts              # MODIFIED: Add waitlist_entries table + types
│   └── (migrations via drizzle-kit go to ./drizzle/)
├── validations/
│   └── waitlist.ts            # UNCHANGED: Zod schema from Story 3.1
├── api-response.ts            # NEW: ApiResponse<T> type + helpers
├── turnstile.ts               # NEW: Cloudflare Turnstile server verification
├── rate-limit.ts              # NEW: Upstash rate limiter (5/IP/hour)
├── mms.ts                     # NEW: MMS API client (party creation + status)
└── utils.ts                   # UNCHANGED: cn() utility
drizzle/
└── XXXX_*.sql                 # NEW: Generated migration file(s)
drizzle.config.ts              # VERIFY: Should be correct as-is
```

### Architecture Compliance Checklist

- [x] File naming: kebab-case (`api-response.ts`, `rate-limit.ts`, `turnstile.ts`)
- [x] Function naming: camelCase (`createParty`, `verifyTurnstileToken`, `checkRateLimit`)
- [x] Type naming: PascalCase (`ApiResponse`, `ErrorCode`, `WaitlistEntry`, `CreatePartyResponse`)
- [x] DB naming: snake_case tables/columns (`waitlist_entries`, `mms_party_id`, `created_at`)
- [x] Imports: `@/` for cross-directory (`@/lib/db`, `@/lib/validations/waitlist`)
- [x] Error messages: English in `console.error`, German in user responses
- [x] ApiResponse wrapper: ALL responses use `ApiResponse<T>` — never raw JSON
- [x] No hardcoded secrets — all via `process.env`
- [x] No `any` types — full TypeScript strict mode
- [x] Server-only code — none of these files should have `"use client"`

### References

- [Source: _bmad-output/planning-artifacts/epics.md#Story 3.2]
- [Source: _bmad-output/planning-artifacts/architecture.md#Data Architecture — Database Schema]
- [Source: _bmad-output/planning-artifacts/architecture.md#Data Architecture — ORM: Drizzle]
- [Source: _bmad-output/planning-artifacts/architecture.md#API & Communication Patterns — Response Format]
- [Source: _bmad-output/planning-artifacts/architecture.md#External Integrations — MMS API]
- [Source: _bmad-output/planning-artifacts/architecture.md#Security — Bot Protection & Rate Limiting]
- [Source: _bmad-output/planning-artifacts/architecture.md#Implementation Patterns — Naming Conventions]
- [Source: _bmad-output/implementation-artifacts/3-1-waitlist-form-with-client-side-validation.md#Dev Agent Record]
- [Drizzle ORM + Neon: https://orm.drizzle.team/docs/connect-neon]
- [Upstash Ratelimit: https://upstash.com/docs/redis/sdks/ratelimit-ts/algorithms]
- [Cloudflare Turnstile Server Validation: https://developers.cloudflare.com/turnstile/get-started/server-side-validation/]

## Dev Agent Record

### Agent Model Used

Claude Opus 4.6

### Debug Log References

- Migration generated: `drizzle/0000_rich_serpent_society.sql` (15 columns, 2 indexes, UNIQUE on confirmation_token)
- `pnpm lint` — zero errors
- `pnpm build` — zero errors, zero type errors

### Completion Notes List

- All 8 tasks (30+ subtasks) completed in single pass
- `errorCodeToStatus` exported (Dev Notes had it as non-exported, but Task 4.5 requires export)
- No `"use client"` directives — all files are server-only
- No `any` types used
- All env vars accessed via `process.env` (no hardcoded secrets)

### Code Review Fixes (2026-02-11)

- **H1**: Added `UNAUTHORIZED` + `FORBIDDEN` to `ErrorCode` union + status mapping (architecture compliance)
- **H2**: Added `lookupParty(email)` to MMS client (architecture spec requires 3 functions)
- **M1**: Exported `CreatePartyRequest` interface for reuse in Story 3.3
- **M2**: Added try/catch to `checkRateLimit()` — fail open on Redis errors
- **M3**: Typed MMS raw response via `MmsPartyRaw` interface + shared `parsePartyResponse()` helper
- **L1, L2**: Accepted as-is (low risk, no fix needed)

### Implementation Plan

Tasks executed in order: 1 → 2 → (4, 5, 6, 7 parallel — new files, no deps) → 3 (migration) → 8 (build/lint)

### File List

| File | Action | Description |
|------|--------|-------------|
| `lib/db/index.ts` | MODIFIED | Added schema import for relational queries |
| `lib/db/schema.ts` | MODIFIED | Added waitlist_entries table (15 cols, 2 indexes) + exported types |
| `lib/api-response.ts` | CREATED | ApiResponse<T> type, ErrorCode, successResponse, errorResponse helpers |
| `lib/turnstile.ts` | CREATED | Server-side Turnstile token verification (fail closed) |
| `lib/rate-limit.ts` | CREATED | Upstash rate limiter (5/IP/hour sliding window) |
| `lib/mms.ts` | CREATED | MMS API client (createParty, updatePartyStatus) |
| `drizzle/0000_rich_serpent_society.sql` | CREATED | SQL migration for waitlist_entries table |
| `drizzle/meta/0000_snapshot.json` | CREATED | Drizzle migration snapshot |
| `drizzle/meta/_journal.json` | CREATED | Drizzle migration journal |
| `_bmad-output/implementation-artifacts/sprint-status.yaml` | MODIFIED | 3-2 status: in-progress → review |
| `_bmad-output/implementation-artifacts/3-2-database-schema-and-integration-clients.md` | MODIFIED | All tasks checked, Dev Agent Record filled |
