# Story 3.4: Double-Opt-In Email & Confirmation Flow

Status: done

## Story

As a **visitor (Stefan)**,
I want to receive a confirmation email after signing up and verify my interest with one click,
so that I know my registration was received and I'm officially on the waitlist.

As a **founder (Mario)**,
I want to be notified when a lead confirms via double-opt-in,
so that I can personally reach out to qualified pilot companies.

## Acceptance Criteria

1. **Given** a waitlist entry was successfully created (Story 3.3) **When** the API route completes the DB insert **Then** a double-opt-in email is sent via Resend (`lib/email.ts`) **And** the email is sent from `RESEND_FROM_EMAIL` (mario@kurzum.app) **And** the email contains a confirmation link: `{NEXT_PUBLIC_APP_URL}/confirmed?token={confirmation_token}` **And** the email includes an unsubscribe link using the MMS unsubscribe token **And** the email subject is "kurzum. — Bitte bestätige deine Anmeldung" **And** the email text is in informal German (duzen)

2. **Given** a visitor clicks the confirmation link **When** `/confirmed?token=xxx` is loaded **Then** the page looks up the `waitlist_entry` by `confirmation_token` **And** if the token is valid and not yet confirmed: `confirmed_at` is set to `now()`, MMS party status is updated to "prospect" via `updatePartyStatus()`, a notification email is sent to Mario, and the page displays "Du bist dabei!" **And** if the token is already confirmed: the page displays "Du bist bereits bestätigt." **And** if the token is invalid or not found: the page displays "Link ungültig"

3. **Given** the Resend API is unavailable **When** email sending fails **Then** the waitlist entry is still saved (email failure does NOT block registration) **And** the error is logged for manual follow-up **And** the user sees the form confirmation (not an error)

4. **Given** the route is deployed **When** `pnpm build` is run **Then** zero errors, zero type errors, zero lint errors

## Tasks / Subtasks

- [x] Task 1: Create Resend email client (AC: #1, #3)
  - [x] 1.1 Create `lib/email.ts` — initialize Resend client with `RESEND_API_KEY` env var
  - [x] 1.2 Implement `sendConfirmationEmail({ to, name, confirmationToken, unsubscribeToken })` → sends double-opt-in email, returns `{ success: boolean }`
  - [x] 1.3 Implement `sendFounderNotification({ entryId, name, email, companyName, gewerk, companySize })` → sends notification to `RESEND_FROM_EMAIL`, returns `{ success: boolean }`
  - [x] 1.4 Both functions: try/catch with `console.error` on failure, never throw — return `{ success: false }` on error

- [x] Task 2: Integrate email sending into waitlist API route (AC: #1, #3)
  - [x] 2.1 Modify `app/api/waitlist/route.ts` — after DB insert, call `sendConfirmationEmail()` with `entry.confirmationToken` and `mmsResult.unsubscribeToken`
  - [x] 2.2 Email failure must NOT prevent success response — fire-and-forget pattern with error logging
  - [x] 2.3 Pass `data.name` to email for personalization

- [x] Task 3: Create confirmation page (AC: #2)
  - [x] 3.1 Create `app/confirmed/page.tsx` as Server Component
  - [x] 3.2 Read `token` from `searchParams`
  - [x] 3.3 Look up `waitlist_entry` by `confirmation_token` using Drizzle `db.select().where(eq(confirmationToken, token))`
  - [x] 3.4 Handle 3 states: valid+unconfirmed → confirm, valid+already-confirmed → show "already confirmed", invalid → show "link invalid"
  - [x] 3.5 Page uses kurzum brand design (logo, colors, typography), standalone layout with "Zurück zu kurzum.app" link

- [x] Task 4: Implement confirmation logic (AC: #2)
  - [x] 4.1 On valid+unconfirmed: UPDATE `waitlist_entries SET confirmed_at = now()` WHERE `confirmation_token = token`
  - [x] 4.2 On valid+unconfirmed: call `updatePartyStatus(entry.mmsPartyId, "prospect")` — catch errors, don't block page render
  - [x] 4.3 On valid+unconfirmed: call `sendFounderNotification()` with lead details — catch errors, don't block page render
  - [x] 4.4 Show "Du bist dabei!" confirmation with brand design

- [x] Task 5: Update FormConfirmation with email hint (AC: #1)
  - [x] 5.1 Modify `components/landing/form-confirmation.tsx` — add spam folder hint: "Schau in dein E-Mail-Postfach für die Bestätigung." and "Keine E-Mail? Schau im Spam-Ordner nach."

- [x] Task 6: Verify Build & Lint (AC: #4)
  - [x] 6.1 Run `pnpm build` — zero errors
  - [x] 6.2 Run `pnpm lint` — zero errors
  - [x] 6.3 TypeScript produces zero type errors

## Dev Notes

### Critical: Existing Files — USE, DO NOT RECREATE

| File | Exports | How to Use |
|------|---------|------------|
| `app/api/waitlist/route.ts` | `POST()` handler | MODIFY — add email sending after DB insert |
| `lib/mms.ts` | `updatePartyStatus(partyId, status)` | Call on confirmation to set status to "prospect" |
| `lib/db/index.ts` | `db` (Drizzle instance) | Query + update `waitlist_entries` |
| `lib/db/schema.ts` | `waitlistEntries`, `WaitlistEntry` | Import table for select/update queries |
| `lib/api-response.ts` | `errorResponse()`, `successResponse()` | Used in route.ts (already imported) |
| `components/landing/form-confirmation.tsx` | `FormConfirmation` | MODIFY — add email hint text |

### Critical: Resend SDK Usage (v6.9.2)

```typescript
// lib/email.ts — NEW
import { Resend } from "resend";

const resend = new Resend(process.env.RESEND_API_KEY);

export async function sendConfirmationEmail(params: {
  to: string;
  name: string;
  confirmationToken: string;
  unsubscribeToken: string | null;
}): Promise<{ success: boolean }> {
  try {
    const confirmUrl = `${process.env.NEXT_PUBLIC_APP_URL}/confirmed?token=${params.confirmationToken}`;
    const unsubscribeUrl = params.unsubscribeToken
      ? `${process.env.NEXT_PUBLIC_APP_URL}/api/unsubscribe?token=${params.unsubscribeToken}`
      : null;

    await resend.emails.send({
      from: process.env.RESEND_FROM_EMAIL!,
      to: params.to,
      subject: "kurzum. — Bitte bestätige deine Anmeldung",
      html: `...`, // Build HTML inline — no @react-email needed
    });
    return { success: true };
  } catch (error) {
    console.error("Failed to send confirmation email:", error instanceof Error ? error.name : "Unknown error");
    return { success: false };
  }
}
```

**DO NOT install @react-email** — use plain HTML string for email body. Keep it simple, this is a landing page with one email template.

### Critical: Resend `emails.send()` API

```typescript
await resend.emails.send({
  from: "Mario von kurzum <mario@kurzum.app>",   // Personal sender
  to: "stefan@example.com",
  subject: "kurzum. — Bitte bestätige deine Anmeldung",
  html: "<html>...</html>",                       // Plain HTML string
  headers: {
    "List-Unsubscribe": `<${unsubscribeUrl}>`,   // If unsubscribe token available
  },
});
```

**IMPORTANT:** `from` format is `"Display Name <email>"`. Use `"Mario von kurzum <mario@kurzum.app>"` for personal feel per UX spec.

### Critical: Waitlist Route Modification (Fire-and-Forget Email)

```typescript
// In app/api/waitlist/route.ts — ADD after DB insert, BEFORE success response:

// 5c. Send confirmation email (fire-and-forget — failure does NOT block success)
sendConfirmationEmail({
  to: data.email,
  name: data.name,
  confirmationToken: entry.confirmationToken,
  unsubscribeToken: mmsResult.unsubscribeToken,
}).catch((err) => {
  console.error("Email send failed:", err instanceof Error ? err.name : "Unknown error");
});

// 6. Success — always returned regardless of email outcome
return successResponse({ entryId: entry.id });
```

**The `entry.confirmationToken` is already returned from the DB insert** (Story 3.3 `.returning()` includes it). The `mmsResult.unsubscribeToken` is already available from the MMS `createParty()` response.

### Critical: Confirmation Page — Server Component with searchParams

```typescript
// app/confirmed/page.tsx — NEW (Server Component)
import { db } from "@/lib/db";
import { waitlistEntries } from "@/lib/db/schema";
import { eq } from "drizzle-orm";
import { updatePartyStatus } from "@/lib/mms";
import { sendFounderNotification } from "@/lib/email";

export default async function ConfirmedPage({
  searchParams,
}: {
  searchParams: Promise<{ token?: string }>;
}) {
  const { token } = await searchParams;    // Next.js 16: searchParams is a Promise
  // ...lookup, update, render
}
```

**CRITICAL: Next.js 16 `searchParams` is a `Promise`** — must `await` it. This is a breaking change from Next.js 14/15 where it was a plain object.

### Critical: Three Confirmation States

| State | Condition | Display | Side Effects |
|-------|-----------|---------|-------------|
| **Success** | Token found, `confirmed_at` is null | "Du bist dabei!" | UPDATE `confirmed_at`, MMS → "prospect", notify Mario |
| **Already confirmed** | Token found, `confirmed_at` is not null | "Du bist bereits bestätigt." | None |
| **Invalid** | Token not found or not provided | "Link ungültig" | None |

### Critical: Confirmation Page UX (from UX Spec)

- Simple standalone page with kurzum brand design (logo, colors, typography)
- "Zurück zu kurzum.app" link — NO full site navigation (dead-end page by design)
- No upsell, no social sharing prompts, no additional CTAs
- Success state: "Du bist dabei! Wir melden uns persönlich bei dir. Voraussichtlicher Start: Herbst 2026."

### Critical: Email HTML Template (Inline, No React-Email)

Email must be:
- From: "Mario von kurzum" (personal, not "noreply@")
- Subject: "kurzum. — Bitte bestätige deine Anmeldung"
- Body: Short, personal, one CTA button "Anmeldung bestätigen"
- Tone: Same as landing page — duzen, direct, no marketing fluff
- Include unsubscribe link in footer and `List-Unsubscribe` header
- Inline CSS only (no external stylesheets in email)
- Use brand colors: Orange CTA button (#F97316), Anthracite text (#1C1917)

### Critical: Drizzle Update Syntax

```typescript
// Update confirmed_at on valid token
await db
  .update(waitlistEntries)
  .set({ confirmedAt: new Date() })
  .where(eq(waitlistEntries.confirmationToken, token));
```

**DO NOT use** `.where(eq(waitlistEntries.confirmationToken, token)).set(...)` — `.set()` comes BEFORE `.where()` in Drizzle ORM.

### Critical: MMS updatePartyStatus on Confirmation

```typescript
// Already implemented in lib/mms.ts (Story 3.2):
await updatePartyStatus(entry.mmsPartyId, "prospect");
```

This calls `PATCH /v1/parties/:id` with `{ status: "prospect" }`. Catch errors — don't block page render if MMS is down.

Party status progression: `lead` (signup) → `prospect` (confirmed) → `customer` (pilot start)

### Critical: Environment Variables Required

| Variable | Value | Used By |
|----------|-------|---------|
| `RESEND_API_KEY` | `re_xxx` | `lib/email.ts` — Resend SDK authentication |
| `RESEND_FROM_EMAIL` | `mario@kurzum.app` | `lib/email.ts` — sender address |
| `NEXT_PUBLIC_APP_URL` | `https://kurzum.app` | `lib/email.ts` — confirmation link base URL |

All already defined in `.env.example`. No new env vars needed.

### Critical: FormConfirmation Update

Current `form-confirmation.tsx` shows:
- "Danke!" + "Wir melden uns persönlich bei dir." + "Voraussichtlicher Start: Herbst 2026."

Must add (per UX spec):
- "Schau in dein E-Mail-Postfach für die Bestätigung."
- "Keine E-Mail? Schau im Spam-Ordner nach." (smaller text, muted)

### Previous Story Learnings (Story 3.1 + 3.2 + 3.3 + Reviews)

1. **Zod v4 `.safeParse()`** — `.error.issues[0]`, NOT `.error.errors[0]`
2. **Next.js 16 `searchParams`** — is a `Promise`, MUST `await` it
3. **Drizzle update syntax** — `.update().set().where()` order (set before where)
4. **Drizzle `eq` import** — from `"drizzle-orm"` (already used in route.ts)
5. **Error logging** — `error.name` not `error.message` (prevent PII leaks)
6. **Fire-and-forget pattern** — `.catch()` on async calls that shouldn't block response
7. **MMS throws on error** — `updatePartyStatus()` throws if API returns non-2xx, must catch
8. **`satisfies` keyword** — Used in `api-response.ts` for type-safe response construction
9. **Duplicate check** — route.ts already handles duplicate MMS party entries (Story 3.3 review fix)

### Installed Package Versions

| Package | Version | Notes |
|---------|---------|-------|
| `resend` | 6.9.2 | `new Resend(apiKey)`, `resend.emails.send()` |
| `drizzle-orm` | 0.45.1 | `.update().set().where()`, `.select().where()` |
| `next` | 16.1.6 | `searchParams` is `Promise`, must `await` |
| `zod` | 4.3.6 | Not directly needed in this story |

### Project Structure (After This Story)

```
app/
├── api/
│   └── waitlist/
│       └── route.ts              # MODIFIED: add sendConfirmationEmail after DB insert
├── confirmed/
│   └── page.tsx                  # NEW: confirmation page (Server Component)
lib/
├── email.ts                      # NEW: Resend email client (sendConfirmationEmail, sendFounderNotification)
├── mms.ts                        # UNCHANGED: updatePartyStatus already exists
├── db/
│   ├── index.ts                  # UNCHANGED
│   └── schema.ts                 # UNCHANGED
components/
└── landing/
    └── form-confirmation.tsx     # MODIFIED: add email + spam hint text
```

### Architecture Compliance Checklist

- [ ] Confirmation page: `app/confirmed/page.tsx` (kebab-case directory)
- [ ] Email client: `lib/email.ts` (kebab-case file)
- [ ] Server Component for confirmation page — no `"use client"`
- [ ] Email failure does NOT block registration (fire-and-forget)
- [ ] Error messages: German for users, English for `console.error`
- [ ] No PII in logs (log `error.name` not `error.message`)
- [ ] No `any` types — full TypeScript strict mode
- [ ] Imports use `@/` alias for cross-directory
- [ ] Email text in informal German (duzen)
- [ ] `searchParams` awaited (Next.js 16 async API)

### References

- [Source: _bmad-output/planning-artifacts/epics.md#Story 3.4]
- [Source: _bmad-output/planning-artifacts/architecture.md#Data Flow — Confirmation Step]
- [Source: _bmad-output/planning-artifacts/architecture.md#Email (Resend)]
- [Source: _bmad-output/planning-artifacts/ux-design-specification.md#Journey 2: Double-Opt-In Confirmation]
- [Source: _bmad-output/planning-artifacts/ux-design-specification.md#FormConfirmation Component]
- [Source: _bmad-output/planning-artifacts/ux-design-specification.md#Confirmation Page Navigation]
- [Source: _bmad-output/implementation-artifacts/3-3-waitlist-api-route.md#Code Review Fixes]
- [Source: app/api/waitlist/route.ts — confirmationToken already returned from .returning()]
- [Source: lib/mms.ts — updatePartyStatus already implemented]

## Dev Agent Record

### Agent Model Used

Claude Opus 4.6

### Debug Log References

No blocking issues encountered during implementation.

### Completion Notes List

- Created `lib/email.ts` with `sendConfirmationEmail()` and `sendFounderNotification()` using Resend SDK v6.9.2
- Email HTML templates built inline with brand colors (Orange #F97316 CTA, Anthracite #1C1917 text), inline CSS only, no @react-email dependency
- Confirmation email: personal sender "Mario von kurzum", `List-Unsubscribe` header, unsubscribe link in footer
- Integrated fire-and-forget email sending in `route.ts` — email failure never blocks API success response
- Created `app/confirmed/page.tsx` as Server Component with `searchParams` properly awaited (Next.js 16 async API)
- Three confirmation states implemented: confirmed ("Du bist dabei!"), already-confirmed ("Du bist bereits bestätigt."), invalid ("Link ungültig")
- Confirmation triggers: `confirmed_at` DB update, MMS party status → "prospect", founder notification email
- Added `getParty()` to `lib/mms.ts` to fetch name/email from MMS for founder notification (since name/email are not stored in waitlist_entries)
- Updated `form-confirmation.tsx` with email hint + spam folder note
- All error handling follows project patterns: `error.name` (no PII), fire-and-forget with `.catch()`, never throws from email functions
- `pnpm build` and `pnpm lint` both pass with zero errors

### Implementation Plan

1. Task 1: Created `lib/email.ts` — Resend client + two email functions with try/catch safety
2. Task 2: Modified `route.ts` — added `sendConfirmationEmail()` after DB insert with fire-and-forget pattern
3. Task 3+4: Created `app/confirmed/page.tsx` — Server Component with token lookup, 3 confirmation states, MMS + email side effects
4. Task 5: Updated `form-confirmation.tsx` — email hint + spam folder note
5. Task 6: Verified build + lint — zero errors

### File List

- `lib/email.ts` — NEW: Resend email client (sendConfirmationEmail, sendFounderNotification, HTML template builders)
- `app/confirmed/page.tsx` — NEW: Confirmation page (Server Component, 3 states, brand design)
- `app/api/waitlist/route.ts` — MODIFIED: Added sendConfirmationEmail import and fire-and-forget call after DB insert
- `lib/mms.ts` — MODIFIED: Added getParty() function to fetch party display_name and email by partyId
- `components/landing/form-confirmation.tsx` — MODIFIED: Added email hint and spam folder note

### Change Log

- 2026-02-11: Implemented Story 3.4 — Double-Opt-In Email & Confirmation Flow. Created Resend email client, confirmation page, integrated email sending into waitlist API route, updated FormConfirmation with email/spam hints.
- 2026-02-11: Code Review (Claude Opus 4.6) — 1 HIGH, 3 MEDIUM, 3 LOW issues found. Fixed: H1 race condition (atomic UPDATE), M1 non-blocking getParty(), M2 email fallback text, M3 sql`now()` consistency, L3 `.limit(1)`. Remaining LOWs (L1 escapeHtml single quotes, L2 List-Unsubscribe-Post header) deferred — no security risk.
