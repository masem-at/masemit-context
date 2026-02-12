# Story 5.2: Magic Link Authentication

Status: done

## Story

As a **user (Stefan)**,
I want to log in by entering my email and clicking a link from my inbox,
So that I can access the protected app area without remembering a password.

## Acceptance Criteria

1. **Given** the app shell and schema exist (Story 5.1), **When** the login page at `/login` is fully implemented, **Then** it displays: kurzum logo, tagline "Sprich statt tipp.", email input (autofocus, type="email"), Turnstile widget, "Login-Link senden" button (Orange, full-width), and hint text. The button is disabled until email is valid. Turnstile uses invisible mode (managed widget as fallback).

2. **Given** the login form is submitted, **When** POST `/api/auth/login` receives email + Turnstile token, **Then** Turnstile token is verified server-side, rate limit is checked (5 magic links per email per hour via Upstash), user is created if email doesn't exist yet (auto-registration), a magic link token (UUID v4, 15-min expiry) is stored in `magic_links` table, a login email is sent via Resend, and the user is redirected to `/login/check-email`.

3. **Given** the user received the email, **When** they click the magic link → GET `/api/auth/verify?token=xxx`, **Then** the token is looked up in `magic_links`, validated (not expired, not used), marked as used (`usedAt = now()`), a session is created in `sessions` table (UUID token, 7-day expiry), a httpOnly/Secure/SameSite=Lax cookie is set, `users.lastLoginAt` is updated, and the user is redirected to `/app`.

4. **Given** an expired or invalid magic link token, **When** verification fails, **Then** the user is redirected to `/login?error=expired` or `/login?error=invalid`, and the login page shows a warm amber banner with the appropriate German message.

5. **Given** the user wants to log out, **When** they click the logout button, **Then** POST `/api/auth/logout` deletes the session from DB, clears the session cookie, and redirects to `/login`.

6. **Given** the login email is needed, **When** `sendLoginEmail()` is called, **Then** the email uses kurzum branding (same style as confirmation email), subject "kurzum. — Dein Login-Link", contains a prominent Orange "Jetzt einloggen" button, explains 15-min expiry and single-use, and all dynamic content uses `escapeHtml()`.

7. **Given** the check-email page exists, **When** a user visits `/login/check-email`, **Then** it shows the email icon, "Prüfe dein Postfach!" headline, the target email address, expiry hint (15 min), "Neue E-Mail anfordern" button, and spam folder hint.

8. **Given** error states on the login page, **When** query parameters contain `?error=expired` or `?error=invalid` or `?error=rate-limited`, **Then** a warm amber banner (NOT aggressive red) displays the appropriate German message.

## Tasks / Subtasks

- [x] **Task 1: Create Auth Validation Schemas** (AC: #1, #2)
  - [x] 1.1 Create `lib/validations/auth.ts` with `loginSchema` (email via `z.email()`, turnstileToken string) and `verifyTokenSchema` (token UUID string)
  - [x] 1.2 Export TypeScript types: `LoginData`, `VerifyTokenData`

- [x] **Task 2: Extend Auth Helpers** (AC: #2, #3, #5)
  - [x] 2.1 Add to `lib/auth/session.ts`: `createSession(userId: string)` — inserts into sessions table (7-day expiry), returns `{ token, expiresAt }`
  - [x] 2.2 Add to `lib/auth/session.ts`: `deleteSession(token: string)` — deletes session row from DB
  - [x] 2.3 Create `lib/auth/magic-link.ts`: `createMagicLink(userId: string)` — inserts into magic_links table (15-min expiry), returns `{ token }`
  - [x] 2.4 Add to `lib/auth/magic-link.ts`: `verifyMagicLink(token: string)` — lookup token, validate not expired + not used, atomically mark `usedAt = now()`, return `{ userId }` or null
  - [x] 2.5 Create `lib/auth/user.ts`: `findOrCreateUser(email: string)` — find by email or create with `name = email-prefix` (users.name is notNull!), return user record

- [x] **Task 3: Create Rate Limiter for Auth** (AC: #2)
  - [x] 3.1 Add to `lib/rate-limit.ts`: new `authRateLimiter` with prefix `kurzum:auth`, 5 requests per email per hour (sliding window)

- [x] **Task 4: Implement Login API Route** (AC: #2)
  - [x] 4.1 Create `app/api/auth/login/route.ts` — POST handler following existing waitlist route pattern:
    1. Content-Type check
    2. Parse JSON body
    3. Turnstile verification (reuse `lib/turnstile.ts`)
    4. Rate limit check by email (NOT IP — per the architecture spec)
    5. Zod validation with `loginSchema`
    6. `findOrCreateUser(email)` — upsert user
    7. `createMagicLink(userId)` — generate token
    8. `sendLoginEmail(email, name, token)` — fire-and-forget via `.catch()`
    9. Return success response (redirect to `/login/check-email` handled client-side)

- [x] **Task 5: Implement Verify API Route** (AC: #3, #4)
  - [x] 5.1 Create `app/api/auth/verify/route.ts` — GET handler:
    1. Extract `token` from query params
    2. `verifyMagicLink(token)` — validates + marks used atomically
    3. If invalid/expired → redirect to `/login?error=expired` or `?error=invalid`
    4. `createSession(userId)` — generates session
    5. Update `users.lastLoginAt = now()`
    6. Set cookie: `session={token}`, `httpOnly`, `Secure`, `SameSite=Lax`, `Path=/`, `maxAge=7d`
    7. Redirect to `/app`

- [x] **Task 6: Upgrade Logout Route** (AC: #5)
  - [x] 6.1 Update `app/api/auth/logout/route.ts` — read session cookie, call `deleteSession(token)` to remove from DB, clear cookie, redirect to `/login`

- [x] **Task 7: Create Login Email Template** (AC: #6)
  - [x] 7.1 Add `sendLoginEmail(email, name, magicLinkToken)` to `lib/email.ts` — follow existing `sendConfirmationEmail` pattern:
    - From: `"Mario von kurzum <notification@kurzum.app>"` (same as confirmation)
    - Subject: `"kurzum. — Dein Login-Link"`
    - HTML: Inline CSS, kurzum logo, greeting, Orange "Jetzt einloggen" button with link to `/api/auth/verify?token=xxx`, 15-min expiry note, ignore-if-not-you text, masemIT footer
    - All dynamic content through `escapeHtml()`

- [x] **Task 8: Implement Login Page (Client Component)** (AC: #1, #8)
  - [x] 8.1 Replace placeholder `app/login/page.tsx` with a layout wrapper (Server Component) that handles session check + redirect + reads `searchParams` for error state
  - [x] 8.2 Create `components/app/login-form.tsx` (Client Component) — email input + Turnstile + submit button using React Hook Form + Zod resolver:
    - Email input: `autofocus`, `type="email"`, `autocomplete="email"`
    - Turnstile: `@marsidev/react-turnstile`, invisible mode, options `{ theme: "light", language: "de" }`
    - Button: Orange full-width, disabled until email valid + Turnstile ready
    - On submit: POST to `/api/auth/login`, on success redirect to `/login/check-email`
    - On error: show inline German error message
  - [x] 8.3 Implement error banner in login page — reads `?error=` param, shows warm amber/orange banner with contextual German message

- [x] **Task 9: Implement Check-Email Page** (AC: #7)
  - [x] 9.1 Update `app/login/check-email/page.tsx` — display email address (from query param or session storage), 15-min expiry hint, "Neue E-Mail anfordern" button (disabled for 60s after click), spam folder hint

- [ ] **Task 10: Manual Smoke Tests** (AC: all)
  - [ ] 10.1 Test happy path: enter email → receive email → click link → arrive at /app
  - [ ] 10.2 Test expired token: wait 15+ min or manually expire → verify error redirect
  - [ ] 10.3 Test rate limiting: send 6+ magic links for same email → verify rate limit error
  - [ ] 10.4 Test logout: click logout → verify redirect to /login + session deleted from DB
  - [ ] 10.5 Test auto-redirect: visit /login with valid session → should redirect to /app
  - [ ] 10.6 Test protected routes: visit /app without session → should redirect to /login

## Dev Notes

### Critical Design Decisions from Story 5.1 Review

**users.name is notNull** — The architecture spec had it nullable, but the schema uses `.notNull()`. When auto-creating users from just an email in `findOrCreateUser()`, use the email prefix (everything before `@`) as the default name. Example: `stefan@elektro-huber.at` → name = `"stefan"`. Users can update their name later.

**Token columns use `uuid("token").defaultRandom()`** — Both `magic_links.token` and `sessions.token` are auto-generated UUID columns. When inserting, do NOT manually generate a UUID — just omit the token field and let PostgreSQL generate it. Use `.returning()` to get the generated token back:
```typescript
const [link] = await db.insert(magicLinks).values({
  userId,
  expiresAt: new Date(Date.now() + 15 * 60 * 1000),
}).returning();
// link.token is the auto-generated UUID
```

**Logout stub exists** — `app/api/auth/logout/route.ts` was created as a stub in the 5.1 code review. It currently only clears the cookie. Task 6 upgrades it to also delete the session from DB.

### Session Cookie Configuration

From architecture spec Section 2:
```typescript
// Cookie settings for session token
response.cookies.set("session", sessionToken, {
  httpOnly: true,
  secure: process.env.NODE_ENV === "production",
  sameSite: "lax",
  path: "/",
  maxAge: 7 * 24 * 60 * 60, // 7 days in seconds
});
```

### Rate Limiting: Per Email, NOT Per IP

The architecture spec says "5 magic links per email per hour" — this is different from the waitlist rate limiter which uses IP. The auth rate limiter should use the email address as the identifier:
```typescript
export const authRateLimiter = new Ratelimit({
  redis: Redis.fromEnv(),
  limiter: Ratelimit.slidingWindow(5, "1 h"),
  prefix: "kurzum:auth",
});
// Usage: await authRateLimiter.limit(email)
```

### Existing Code Patterns to Follow

**API Route Pipeline** (from `app/api/waitlist/route.ts`):
1. Content-Type check → reject non-JSON
2. Parse body with try/catch
3. Turnstile verification
4. Rate limit check
5. Zod validation with `safeParse`
6. Business logic
7. Return `successResponse()` or `errorResponse()`

**Email Pattern** (from `lib/email.ts`):
- Use `new Resend(process.env.RESEND_API_KEY)`
- HTML with inline CSS only, no @react-email
- `escapeHtml()` for all dynamic content
- Fire-and-forget: `.catch(console.error)`, never block API response
- From address: `"Mario von kurzum <notification@kurzum.app>"`

**Turnstile Pattern** (from `lib/turnstile.ts`):
- Extract token from parsed body (NOT from raw request)
- Call `verifyTurnstileToken(token, ip)`
- On failure: return `errorResponse("BOT_DETECTED", ...)`

**Zod v4 Pattern** (from `lib/validations/waitlist.ts`):
- `z.email()` is top-level in Zod v4 (NOT `z.string().email()`)
- German error messages in schema: `z.email({ message: "Bitte gib eine gültige E-Mail-Adresse ein." })`

### Magic Link Verification — Atomic Pattern

Use the same atomic UPDATE pattern from Story 3.4 to prevent TOCTOU race conditions:
```typescript
// Atomically verify + mark as used in one query
const [link] = await db.update(magicLinks)
  .set({ usedAt: sql`now()` })
  .where(and(
    eq(magicLinks.token, token),
    gt(magicLinks.expiresAt, sql`now()`),
    isNull(magicLinks.usedAt),
  ))
  .returning({ userId: magicLinks.userId });

if (!link) return null; // expired, used, or not found
return { userId: link.userId };
```

### Login Page Architecture

The login page needs BOTH Server Component and Client Component parts:

**Server Component wrapper** (`app/login/page.tsx`):
- Reads `cookies()` for session check → redirect to `/app` if valid
- Reads `searchParams` for `?error=` query param (Promise in Next.js 16!)
- Passes error state as prop to the client component

**Client Component form** (`components/app/login-form.tsx`):
- React Hook Form + Zod resolver for email validation
- Turnstile widget from `@marsidev/react-turnstile`
- State management for submission, loading, errors
- On success: `router.push('/login/check-email')` (client-side redirect)

### Error Banner Design

From UX spec Section 2, State 4:
- **Token abgelaufen**: "Dieser Link ist abgelaufen. Fordere einen neuen an."
- **Token ungültig**: "Dieser Link ist ungültig."
- **Rate Limit**: "Zu viele Anfragen. Bitte warte ein paar Minuten."
- **Server-Fehler**: "Etwas ist schiefgelaufen. Versuch es nochmal."

Design: Warm amber/orange banner (`--brand-warning` or amber-based), NOT aggressive red. Calm pragmatism.

### Check-Email Page Enhancement

From UX spec Section 2, State 2:
- Show the email address the link was sent to (pass via query param: `/login/check-email?email=stefan@elektro-huber.at`)
- "Der Link ist 15 Minuten gültig."
- "Neue E-Mail anfordern" button — ghost/outline style, disabled for 60s after click (client-side cooldown)
- "Keine E-Mail? Schau im Spam-Ordner nach." — subtle hint

### Login Email Template (HTML)

Follow the exact pattern from `sendConfirmationEmail` in `lib/email.ts`. Key structure:
```html
<!-- Same inline CSS template as confirmation email -->
<div style="font-family: Inter, sans-serif; max-width: 520px; margin: 0 auto; padding: 32px 20px;">
  <!-- Logo: kurzum. -->
  <div style="text-align: center; margin-bottom: 24px;">
    <span style="font-size: 24px; font-weight: 700; color: #1C1917;">kurzum</span>
    <span style="font-size: 24px; font-weight: 700; color: #F97316;">.</span>
  </div>
  <!-- Greeting -->
  <p>Hallo {name},</p>
  <p>hier ist dein Login-Link für kurzum.app:</p>
  <!-- CTA Button -->
  <a href="{loginUrl}" style="display: block; background: #F97316; color: #fff; text-align: center; padding: 14px 24px; border-radius: 8px; text-decoration: none; font-weight: 600;">
    Jetzt einloggen
  </a>
  <!-- Expiry note -->
  <p style="color: #78716C; font-size: 14px;">Der Link ist 15 Minuten gültig und kann nur einmal verwendet werden.</p>
  <p style="color: #78716C; font-size: 14px;">Falls du keinen Login angefordert hast, ignoriere diese E-Mail.</p>
  <!-- Footer -->
  <hr style="border: none; border-top: 1px solid #E7E5E4; margin: 24px 0;" />
  <p style="color: #A8A29E; font-size: 12px;">© 2026 masemIT e.U. — kurzum.app</p>
</div>
```

### Server vs Client Component Map

| File | Type | Reason |
|------|------|--------|
| `app/login/page.tsx` | Server Component | Session check, searchParams access |
| `components/app/login-form.tsx` | Client Component | Form state, Turnstile, useRouter |
| `app/login/check-email/page.tsx` | Server Component with client parts | Display email, resend cooldown |
| `app/api/auth/login/route.ts` | API Route (server) | Auth pipeline |
| `app/api/auth/verify/route.ts` | API Route (server) | Token verification |
| `app/api/auth/logout/route.ts` | API Route (server) | Session deletion |
| `lib/auth/session.ts` | Server utility | DB operations |
| `lib/auth/magic-link.ts` | Server utility | DB operations |
| `lib/auth/user.ts` | Server utility | DB operations |
| `lib/validations/auth.ts` | Shared | Used by client + server |

### Project Structure Notes

**New files to create:**
```
lib/
├── validations/
│   └── auth.ts                    # Zod schemas (NEW)
├── auth/
│   ├── session.ts                 # EXTEND with createSession, deleteSession
│   ├── magic-link.ts              # NEW — createMagicLink, verifyMagicLink
│   └── user.ts                    # NEW — findOrCreateUser
├── email.ts                       # EXTEND with sendLoginEmail
├── rate-limit.ts                  # EXTEND with authRateLimiter

app/
├── api/auth/
│   ├── login/route.ts             # NEW — POST magic link request
│   ├── verify/route.ts            # NEW — GET token verification
│   └── logout/route.ts            # MODIFY — upgrade stub to delete session
├── login/
│   ├── page.tsx                   # MODIFY — functional login page
│   └── check-email/
│       └── page.tsx               # MODIFY — enhanced check-email page

components/
├── app/
│   └── login-form.tsx             # NEW — Client Component login form
```

**Files to modify:**
- `lib/auth/session.ts` — add `createSession()`, `deleteSession()`
- `lib/email.ts` — add `sendLoginEmail()`
- `lib/rate-limit.ts` — add `authRateLimiter`
- `app/api/auth/logout/route.ts` — upgrade stub
- `app/login/page.tsx` — replace placeholder with functional form
- `app/login/check-email/page.tsx` — add email display + resend

**Alignment with architecture:**
- `lib/auth/` per architecture-sprint0-supplement.md Section 2
- API routes under `app/api/auth/` per architecture Section 3
- Magic link flow matches sequence diagram in Section 2 exactly

### References

- [Auth Flow: docs/architecture-sprint0-supplement.md#2-authentication-flow-magic-link]
- [Session Schema: docs/architecture-sprint0-supplement.md#2 → Session Schema]
- [Security Constraints: docs/architecture-sprint0-supplement.md#2 → Security Constraints]
- [Route Structure: docs/architecture-sprint0-supplement.md#3-route-structure]
- [Login UX States: docs/ux-sprint0-app-specs.md#2-login-flow]
- [Email Template: docs/ux-sprint0-app-specs.md#10-magic-link-e-mail-template]
- [Error Banner Design: docs/ux-sprint0-app-specs.md#2 → State 4]
- [Check-Email UX: docs/ux-sprint0-app-specs.md#2 → State 2]
- [Color Tokens: docs/ux-sprint0-app-specs.md#8-color-usage-im-app-bereich]
- [Existing Email Pattern: lib/email.ts]
- [Existing API Route Pattern: app/api/waitlist/route.ts]
- [Existing Turnstile Pattern: lib/turnstile.ts]
- [Existing Rate Limit Pattern: lib/rate-limit.ts]
- [Previous Story: _bmad-output/implementation-artifacts/5-1-database-schema-extension-and-app-shell.md]
- [Epics: _bmad-output/planning-artifacts/epics.md#story-52-magic-link-authentication]
- [FFG Rules: CLAUDE.md + docs/_masemIT/kurzum-bmad-do-dont-liste.md]

## Dev Agent Record

### Agent Model Used

Claude Opus 4.6

### Debug Log References

- Zod v4 + @hookform/resolvers type mismatch: Cast schema `as any` (not result). Known issue, runtime works correctly.

### Completion Notes List

- **Tasks 1-9 code-complete.** All implementation tasks done, build passes, lint clean.
- **Task 10 (Manual Smoke Tests)** requires manual testing with running dev server + real services (NeonDB, Resend, Upstash, Turnstile). Cannot be automated by dev agent.
- Followed all architecture patterns: atomic UPDATE for magic link verification (TOCTOU prevention), fire-and-forget email, per-email rate limiting, Server/Client Component split.
- Auto-registration via `findOrCreateUser()` uses email prefix as default name (users.name is notNull).
- Token columns use PostgreSQL auto-generated UUIDs via `.returning()`.
- Login API pipeline mirrors waitlist route pattern exactly.
- Error banner uses warm amber design (not aggressive red) per UX spec.
- Check-email page includes ResendButton with 60s cooldown + Turnstile protection.

### Change Log

- 2026-02-12: Implement magic link authentication flow (Tasks 1-9). Full login/verify/logout cycle with Turnstile, rate limiting, email delivery, and error handling.
- 2026-02-12: Code review (Opus 4.6) — 1 HIGH + 4 MEDIUM issues fixed: H1 findOrCreateUser race condition (onConflictDoNothing), M1 logout error handling (try/catch), M2+M3 verify route token validation + error type distinction (verifyTokenSchema + DB lookup), M4 nul artifact deleted + gitignored. 3 LOW issues documented as known.

### File List

**New files:**
- `lib/validations/auth.ts` — Zod schemas for login and token verification
- `lib/auth/magic-link.ts` — createMagicLink, verifyMagicLink (atomic UPDATE)
- `lib/auth/user.ts` — findOrCreateUser (auto-registration)
- `app/api/auth/login/route.ts` — POST handler for magic link requests
- `app/api/auth/verify/route.ts` — GET handler for token verification + session creation
- `components/app/login-form.tsx` — Client Component with RHF + Turnstile + error handling
- `app/login/check-email/resend-button.tsx` — Client Component with 60s cooldown + Turnstile

**Modified files:**
- `lib/auth/session.ts` — Added createSession(), deleteSession()
- `lib/rate-limit.ts` — Added authRateLimiter (per email, 5/hour)
- `lib/email.ts` — Added sendLoginEmail() with kurzum-branded template
- `app/api/auth/logout/route.ts` — Upgraded stub to delete session from DB
- `app/login/page.tsx` — Replaced placeholder with functional login page (Server Component + error banner)
- `app/login/check-email/page.tsx` — Enhanced with email display, expiry hint, resend button, spam hint
