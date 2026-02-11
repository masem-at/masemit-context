# Story 3.3: Waitlist API Route

Status: done

## Story

As a **founder (Mario)**,
I want every waitlist signup securely validated, bot-protected, rate-limited, and stored in both MMS and local DB,
so that I receive only legitimate leads with GDPR-compliant consent tracking.

## Acceptance Criteria

1. **Given** the database, MMS client, Turnstile, and rate limiter are ready (Story 3.2) **When** `POST /api/waitlist` receives a request **Then** processing follows this exact sequence: (1) Cloudflare Turnstile token verification → 403 BOT_DETECTED if invalid, (2) Upstash rate limit check (5 per IP per hour) → 429 RATE_LIMITED if exceeded, (3) Server-side Zod validation of request body → 400 VALIDATION_ERROR if invalid, (4) MMS API: create/find party with consent data, (5) Local DB: INSERT `waitlist_entries` with mms_party_id + business data + UTM params, (6) Return `ApiResponse<{ entryId: number }>` with success: true

2. **Given** UTM parameters are present in the request **When** the entry is stored **Then** `utm_source`, `utm_medium`, `utm_campaign` are extracted from the request body **And** stored in the corresponding columns of `waitlist_entries`

3. **Given** any step in the pipeline fails **When** an error occurs **Then** the API returns the appropriate `ApiResponse` error format with correct HTTP status code **And** errors are logged with `console.error` in English (no PII in logs) **And** user-facing error messages are in German **And** non-JSON requests are rejected with 400

4. **Given** the route is deployed **When** `pnpm build` is run **Then** zero errors, zero type errors, zero lint errors

## Tasks / Subtasks

- [x] Task 1: Create server-side validation schema (AC: #1, #3)
  - [x] 1.1 Create `lib/validations/waitlist-api.ts` — extend the existing form schema with `turnstileToken` and UTM fields for the API request body
  - [x] 1.2 Define `waitlistApiSchema` that includes: all `waitlistFormSchema` fields + `turnstileToken: z.string().min(1)` + `utmSource: z.string().nullable().optional()` + `utmMedium: z.string().nullable().optional()` + `utmCampaign: z.string().nullable().optional()`
  - [x] 1.3 Export `type WaitlistApiData = z.infer<typeof waitlistApiSchema>`

- [x] Task 2: Create the API Route Handler (AC: #1, #2, #3)
  - [x] 2.1 Create `app/api/waitlist/route.ts` with `export async function POST(request: Request)`
  - [x] 2.2 Extract client IP from `request.headers.get("x-forwarded-for")` — split on comma, take first, fallback to `"unknown"`
  - [x] 2.3 Reject non-JSON Content-Type with `errorResponse("VALIDATION_ERROR", "Content-Type must be application/json", 400)`
  - [x] 2.4 Parse request body with `request.json()` — catch JSON parse errors as VALIDATION_ERROR

- [x] Task 3: Implement Security Pipeline (AC: #1, #3)
  - [x] 3.1 Step 1 — Turnstile verification: call `verifyTurnstileToken(body.turnstileToken, ip)` → return `errorResponse("BOT_DETECTED", ...)` if false
  - [x] 3.2 Step 2 — Rate limit: call `checkRateLimit(ip)` → return `errorResponse("RATE_LIMITED", ...)` if not success
  - [x] 3.3 Step 3 — Zod validation: call `waitlistApiSchema.safeParse(body)` → return `errorResponse("VALIDATION_ERROR", ...)` with first error message if not success

- [x] Task 4: Implement Business Logic Pipeline (AC: #1, #2)
  - [x] 4.1 Step 4 — MMS: call `createParty({ email: data.email, displayName: data.name, sourceProject: "kurzum_app", marketingConsent: true, consentIp: ip, consentUserAgent: request.headers.get("user-agent") ?? "unknown" })`
  - [x] 4.2 Step 5 — DB insert: use `db.insert(waitlistEntries).values({ mmsPartyId: mmsResult.partyId, mmsContactPointId: mmsResult.contactPointId, companyName: data.companyName, gewerk: data.gewerk, companySize: data.companySize, phone: data.phone ?? null, message: data.message ?? null, utmSource: data.utmSource ?? null, utmMedium: data.utmMedium ?? null, utmCampaign: data.utmCampaign ?? null }).returning({ id: waitlistEntries.id, confirmationToken: waitlistEntries.confirmationToken })`
  - [x] 4.3 Step 6 — Return success: `successResponse({ entryId: entry.id })`

- [x] Task 5: Implement Error Handling (AC: #3)
  - [x] 5.1 Wrap MMS + DB steps in try/catch — catch MMS errors and DB errors separately
  - [x] 5.2 Log errors in English with `console.error` — include error message but no PII (no email, no name)
  - [x] 5.3 Return `errorResponse("INTERNAL_ERROR", "Die Anmeldung konnte nicht gespeichert werden. Bitte versuche es später erneut.")` for any unhandled error
  - [x] 5.4 Add outer try/catch around entire handler as safety net

- [x] Task 6: Verify Build & Lint (AC: #4)
  - [x] 6.1 Run `pnpm build` — zero errors
  - [x] 6.2 Run `pnpm lint` — zero errors
  - [x] 6.3 TypeScript produces zero type errors

## Dev Notes

### Critical: Existing Files from Story 3.2 — USE, DO NOT RECREATE

These files were created in Story 3.2 and are your building blocks. Import and USE them:

| File | Exports | How to Use |
|------|---------|------------|
| `lib/api-response.ts` | `successResponse<T>()`, `errorResponse()`, `errorCodeToStatus()`, `ErrorCode`, `ApiResponse<T>` | Return all responses via these helpers |
| `lib/turnstile.ts` | `verifyTurnstileToken(token, ip?)` → `Promise<boolean>` | Call in step 1, returns false = bot |
| `lib/rate-limit.ts` | `checkRateLimit(ip)` → `Promise<{ success, remaining }>` | Call in step 2, fail-open on Redis errors |
| `lib/mms.ts` | `createParty(data)` → `Promise<CreatePartyResponse>`, `CreatePartyRequest`, `CreatePartyResponse` | Call in step 4 |
| `lib/db/index.ts` | `db` (Drizzle instance with schema) | `db.insert(waitlistEntries).values(...)` |
| `lib/db/schema.ts` | `waitlistEntries`, `WaitlistEntry`, `NewWaitlistEntry` | Import table for insert |
| `lib/validations/waitlist.ts` | `waitlistFormSchema`, `WaitlistFormData`, `GEWERK_OPTIONS`, `COMPANY_SIZE_OPTIONS` | Base schema to extend |

### Critical: Exact Request Body from Client

The `waitlist-form.tsx` (Story 3.1) sends this exact JSON body to `POST /api/waitlist`:

```typescript
{
  // From WaitlistFormData (Zod schema):
  name: string,          // "Martin Maier"
  email: string,         // "martin@maier-elektro.at"
  companyName: string,   // "Maier Elektrotechnik"
  gewerk: string,        // "Elektrotechnik" | "Sanitär/Heizung/Lüftung" | "Sonstige"
  companySize: string,   // "1-5" | "6-10" | "11-20" | "21-50" | "50+"
  phone?: string,        // "+43 664 123 4567" (optional)
  message?: string,      // "Bin sehr interessiert" (optional)

  // Added by form submission:
  turnstileToken: string, // Cloudflare Turnstile token
  utmSource: string | null, // from URL params
  utmMedium: string | null,
  utmCampaign: string | null,
}
```

### Critical: Server-Side Validation Schema

Create a NEW file `lib/validations/waitlist-api.ts` that EXTENDS the existing schema:

```typescript
// lib/validations/waitlist-api.ts — NEW
import { z } from "zod";
import { waitlistFormSchema } from "./waitlist";

export const waitlistApiSchema = waitlistFormSchema.extend({
  turnstileToken: z.string().min(1, { message: "Turnstile token is required." }),
  utmSource: z.string().nullable().optional(),
  utmMedium: z.string().nullable().optional(),
  utmCampaign: z.string().nullable().optional(),
});

export type WaitlistApiData = z.infer<typeof waitlistApiSchema>;
```

**WHY A SEPARATE FILE:** The form schema (`waitlistFormSchema`) is shared between client and server. The API schema adds server-only fields (`turnstileToken`, UTM). Keeping them separate avoids shipping server validation logic to the client bundle.

### Critical: Exact Pipeline Implementation

```typescript
// app/api/waitlist/route.ts — NEW
import { verifyTurnstileToken } from "@/lib/turnstile";
import { checkRateLimit } from "@/lib/rate-limit";
import { createParty } from "@/lib/mms";
import { db } from "@/lib/db";
import { waitlistEntries } from "@/lib/db/schema";
import { waitlistApiSchema } from "@/lib/validations/waitlist-api";
import { successResponse, errorResponse } from "@/lib/api-response";

export async function POST(request: Request) {
  try {
    // 0. Content-Type check
    const contentType = request.headers.get("content-type");
    if (!contentType?.includes("application/json")) {
      return errorResponse("VALIDATION_ERROR", "Content-Type must be application/json.");
    }

    // 0b. Extract IP
    const forwarded = request.headers.get("x-forwarded-for");
    const ip = forwarded?.split(",")[0].trim() ?? "unknown";

    // 0c. Parse JSON body
    let rawBody: unknown;
    try {
      rawBody = await request.json();
    } catch {
      return errorResponse("VALIDATION_ERROR", "Ungültiger Request-Body.");
    }

    // 1. Turnstile verification
    const { turnstileToken, ...formFields } = rawBody as Record<string, unknown>;
    if (typeof turnstileToken !== "string" || !turnstileToken) {
      return errorResponse("BOT_DETECTED", "Bot-Schutz-Token fehlt.");
    }
    const isHuman = await verifyTurnstileToken(turnstileToken, ip);
    if (!isHuman) {
      return errorResponse("BOT_DETECTED", "Bot-Schutz-Überprüfung fehlgeschlagen. Bitte lade die Seite neu und versuche es erneut.");
    }

    // 2. Rate limit
    const rateResult = await checkRateLimit(ip);
    if (!rateResult.success) {
      return errorResponse("RATE_LIMITED", "Zu viele Anfragen. Bitte warte eine Stunde und versuche es erneut.");
    }

    // 3. Zod validation
    const parsed = waitlistApiSchema.safeParse(rawBody);
    if (!parsed.success) {
      const firstError = parsed.error.issues[0];
      return errorResponse("VALIDATION_ERROR", firstError.message);
    }
    const data = parsed.data;

    // 4. MMS: create/find party
    const mmsResult = await createParty({
      email: data.email,
      displayName: data.name,
      sourceProject: "kurzum_app",
      marketingConsent: true,
      consentIp: ip,
      consentUserAgent: request.headers.get("user-agent") ?? "unknown",
    });

    // 5. DB insert
    const [entry] = await db
      .insert(waitlistEntries)
      .values({
        mmsPartyId: mmsResult.partyId,
        mmsContactPointId: mmsResult.contactPointId,
        companyName: data.companyName,
        gewerk: data.gewerk,
        companySize: data.companySize,
        phone: data.phone ?? null,
        message: data.message ?? null,
        utmSource: data.utmSource ?? null,
        utmMedium: data.utmMedium ?? null,
        utmCampaign: data.utmCampaign ?? null,
      })
      .returning({
        id: waitlistEntries.id,
        confirmationToken: waitlistEntries.confirmationToken,
      });

    // 6. Success
    return successResponse({ entryId: entry.id });
  } catch (error) {
    console.error("Waitlist API error:", error instanceof Error ? error.message : error);
    return errorResponse(
      "INTERNAL_ERROR",
      "Die Anmeldung konnte nicht gespeichert werden. Bitte versuche es später erneut."
    );
  }
}
```

### Critical: Zod v4 `.safeParse()` Error Access

In Zod v4 (4.3.6), `safeParse` returns `{ success: false, error: ZodError }`. Access issues via:

```typescript
const parsed = schema.safeParse(data);
if (!parsed.success) {
  const firstError = parsed.error.issues[0];
  // firstError.message = "Bitte gib deinen Namen ein."
  // firstError.path = ["name"]
}
```

**DO NOT** use `parsed.error.errors` (Zod v3 API) — use `parsed.error.issues` (Zod v4).

### Critical: Drizzle `.returning()` Syntax

```typescript
const [entry] = await db
  .insert(waitlistEntries)
  .values({ ... })
  .returning({
    id: waitlistEntries.id,
    confirmationToken: waitlistEntries.confirmationToken,
  });
// entry = { id: number, confirmationToken: string }
```

The `.returning()` call returns an array. Destructure `[entry]` to get the single row. The `confirmationToken` is returned for Story 3.4 (double-opt-in email) but NOT included in the API response to the client.

### Critical: IP Extraction on Vercel

On Vercel, the client IP is available via `x-forwarded-for` header. It may contain multiple IPs (comma-separated) if the request passed through proxies. Always take the FIRST value:

```typescript
const forwarded = request.headers.get("x-forwarded-for");
const ip = forwarded?.split(",")[0].trim() ?? "unknown";
```

**DO NOT** use `request.ip` — it does not exist on the standard `Request` object. Next.js provides IP via headers.

### Critical: German Error Messages

All user-facing error messages MUST be in informal German (duzen):

| Error Code | German Message |
|------------|---------------|
| `BOT_DETECTED` (no token) | "Bot-Schutz-Token fehlt." |
| `BOT_DETECTED` (failed) | "Bot-Schutz-Überprüfung fehlgeschlagen. Bitte lade die Seite neu und versuche es erneut." |
| `RATE_LIMITED` | "Zu viele Anfragen. Bitte warte eine Stunde und versuche es erneut." |
| `VALIDATION_ERROR` | From Zod schema (already German in `waitlistFormSchema`) |
| `VALIDATION_ERROR` (JSON) | "Ungültiger Request-Body." |
| `VALIDATION_ERROR` (content-type) | "Content-Type must be application/json." (technical, not user-facing) |
| `INTERNAL_ERROR` | "Die Anmeldung konnte nicht gespeichert werden. Bitte versuche es später erneut." |

### Previous Story Learnings (Story 3.1 + 3.2 + Reviews)

1. **Zod v4 `.safeParse()`** — Access errors via `.error.issues[0]`, NOT `.error.errors[0]`
2. **ESLint `no-empty-object-type`** — Do NOT export empty interfaces
3. **Error messages** — German for users, English for `console.error`. Be specific.
4. **Rate limiter fail-open** — `checkRateLimit()` returns `{ success: true, remaining: -1 }` if Redis is down
5. **Turnstile fail-closed** — `verifyTurnstileToken()` returns `false` on network error
6. **MMS throws on error** — `createParty()` throws `Error` if MMS API returns non-2xx. Must be caught.
7. **`as any` NOT needed here** — Server-side Zod usage has no type mismatch issues
8. **`satisfies` keyword** — Used in `api-response.ts` for type-safe response construction

### Installed Package Versions

| Package | Version | Notes |
|---------|---------|-------|
| `drizzle-orm` | 0.45.1 | `db.insert().values().returning()` API |
| `zod` | 4.3.6 | `z.email()` top-level, `.safeParse()` with `.error.issues` |
| `@upstash/ratelimit` | 2.0.8 | Already instantiated in `lib/rate-limit.ts` |
| `@upstash/redis` | 1.36.2 | Auto-config via `Redis.fromEnv()` |
| `next` | 16.1.6 | App Router, `app/api/*/route.ts` pattern |

### Project Structure (After This Story)

```
app/
└── api/
    └── waitlist/
        └── route.ts              # NEW: POST handler with security pipeline
lib/
├── validations/
│   ├── waitlist.ts               # UNCHANGED: Client+server form schema
│   └── waitlist-api.ts           # NEW: Server-only API schema (extends form schema)
├── api-response.ts               # UNCHANGED: ApiResponse helpers
├── turnstile.ts                  # UNCHANGED: Turnstile verification
├── rate-limit.ts                 # UNCHANGED: Upstash rate limiter
├── mms.ts                        # UNCHANGED: MMS API client
├── db/
│   ├── index.ts                  # UNCHANGED: Drizzle instance
│   └── schema.ts                 # UNCHANGED: waitlist_entries table
└── utils.ts                      # UNCHANGED: cn() utility
```

### Architecture Compliance Checklist

- [x] Route file: `app/api/waitlist/route.ts` (kebab-case directory)
- [x] All responses use `successResponse()` or `errorResponse()` — never raw `Response.json()`
- [x] Security pipeline order: Turnstile → Rate Limit → Validation → Business Logic
- [x] Error messages: German for users, English for `console.error`
- [x] No PII in logs (no email, no name, no phone)
- [x] No `any` types — full TypeScript strict mode
- [x] No `"use client"` — this is a server-only route
- [x] Imports use `@/` alias for cross-directory
- [x] Response body uses camelCase (`entryId`, NOT `entry_id`)

### References

- [Source: _bmad-output/planning-artifacts/epics.md#Story 3.3]
- [Source: _bmad-output/planning-artifacts/architecture.md#API & Communication Patterns]
- [Source: _bmad-output/planning-artifacts/architecture.md#Security — Bot Protection & Rate Limiting]
- [Source: _bmad-output/planning-artifacts/architecture.md#Implementation Patterns — Naming Conventions]
- [Source: _bmad-output/implementation-artifacts/3-2-database-schema-and-integration-clients.md#Code Review Fixes]
- [Source: components/landing/waitlist-form.tsx#onSubmit — Exact client request body]

## Dev Agent Record

### Agent Model Used

Claude Opus 4.6 (claude-opus-4-6)

### Debug Log References

- No debug issues encountered — build and lint passed on first attempt

### Completion Notes List

- Created `lib/validations/waitlist-api.ts` extending `waitlistFormSchema` with `turnstileToken` + UTM fields
- Created `app/api/waitlist/route.ts` with full security pipeline: Content-Type check → IP extraction → JSON parse → Turnstile → Rate Limit → Zod validation → MMS createParty → DB insert → successResponse
- Turnstile token extracted from raw body before Zod validation to enable early rejection
- Outer try/catch as safety net returns INTERNAL_ERROR with German user-facing message
- `console.error` logs in English with no PII
- Note: MMS + DB errors are caught by the outer try/catch rather than separate inner catches — functionally equivalent since both return INTERNAL_ERROR
- `confirmationToken` returned from DB insert but NOT exposed in API response (reserved for Story 3.4 double-opt-in)

### Code Review Fixes (2026-02-11)

- **H1 (Duplicate Entry):** Added duplicate check via `db.select().where(eq(mmsPartyId))` before insert — returns existing entryId if MMS party already on waitlist
- **H3 (PII in Error Logs):** Changed `console.error` to log `error.name` instead of `error.message` to prevent potential PII leaks from DB errors
- **M1 (Pipeline Order):** Architecture feedback documented — Turnstile → Rate Limit order follows spec but Rate Limit first would be more secure (no code change, architecture decision)
- **M2 (Redundant ?? null):** Removed unnecessary `?? null` on phone, message, and UTM fields — Drizzle handles `undefined` correctly for nullable columns
- Added `import { eq } from "drizzle-orm"` for duplicate check query

### Implementation Plan

1. Task 1: Created `lib/validations/waitlist-api.ts` — server-side schema extending form schema
2. Tasks 2–5: Created `app/api/waitlist/route.ts` with complete POST handler
3. Task 6: Verified `pnpm build` (zero errors, route shows as `ƒ /api/waitlist`) and `pnpm lint` (zero errors)

### File List

| File | Action | Description |
|------|--------|-------------|
| `lib/validations/waitlist-api.ts` | CREATED | Server-only API schema extending waitlistFormSchema with turnstileToken + UTM |
| `app/api/waitlist/route.ts` | CREATED | POST handler with 6-step security + business logic pipeline |
