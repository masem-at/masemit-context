# Epic: Magic Link Authentication

**Status:** Ready for Implementation
**Priority:** Critical
**Created:** 2026-01-31
**Product Brief:** `docs/analysis/product-brief-magic-link-auth.md`
**Architecture:** `docs/architecture-magic-link-auth.md`
**Resolves:** BUG-1 (Admin access), BUG-2 (Session persistence)

---

## Problem Statement

ChainSights has no session management. Users must re-enter their email on every page visit to unlock gated content. Admin cannot access gated charts. The email unlock state is stateless and per-page. Magic Link Authentication provides lightweight, passwordless auth with persistent 30-day sessions.

---

## Architecture Decisions

- **AD-1:** `jose` for Edge-compatible JWT (Middleware runs in Edge Runtime)
- **AD-2:** Session resolution on all routes via `x-user-email` + `x-user-role` request headers
- **AD-3:** New `users` table (separate from `leads`)
- **AD-4:** DB-stored tokens for one-time-use enforcement
- **AD-5:** Session-first `getMatrixAccess()` with query param fallback during migration

---

## Story A: Database Schema & Dependency Setup

**Files:**
- `src/lib/db/schema.ts` (MODIFY: add `users` + `authTokens` tables + `userRoleEnum`)
- `package.json` (MODIFY: add `jose` dependency)

**Acceptance Criteria:**

- AC1: `userRoleEnum` pgEnum created with values `['admin', 'free', 'subscriber']`
- AC2: `users` table created with columns: `id` (uuid PK), `email` (text, unique, not null), `role` (userRoleEnum, default 'free'), `marketingOptIn` (boolean, default false), `firstAuthAt` (timestamp), `lastAuthAt` (timestamp), `createdAt` (timestamp)
- AC3: `users` table has unique index on `email`
- AC4: `authTokens` table created with columns: `id` (uuid PK), `email` (text, not null), `token` (text, unique, not null), `redirectUrl` (text), `expiresAt` (timestamp, not null), `usedAt` (timestamp, nullable), `createdAt` (timestamp)
- AC5: `authTokens` table has unique index on `token` and regular index on `email`
- AC6: Type exports added: `User`, `UserInsert`, `AuthToken`, `AuthTokenInsert`
- AC7: `jose` npm package installed
- AC8: `npm run db:generate` produces clean migration
- AC9: TypeScript compiles with no new errors

---

## Story B: Session Library (JWT Create/Verify)

**Files:**
- `src/lib/auth/session.ts` (NEW)

**Acceptance Criteria:**

- AC1: `createSessionToken(payload: { email: string; role: string }): Promise<string>` — creates a signed JWT using `jose` with `AUTH_SECRET` env, 30-day expiry, HMAC-SHA256
- AC2: `verifySessionToken(token: string): Promise<{ email: string; role: string } | null>` — verifies JWT signature and expiry, returns payload or null
- AC3: `setSessionCookie(token: string): Promise<void>` — sets `cs_session` HTTP-only cookie with Secure (prod), SameSite=Lax, 30-day maxAge, path=/
- AC4: `clearSessionCookie(): Promise<void>` — deletes `cs_session` cookie
- AC5: `getSessionFromCookie(): Promise<{ email: string; role: string } | null>` — reads and verifies `cs_session` cookie, returns payload or null
- AC6: All functions use `cookies()` from `next/headers` for cookie operations
- AC7: `AUTH_SECRET` env variable required — function throws clear error if missing
- AC8: TypeScript strict mode compliance, explicit return types on all exports

---

## Story C: Magic Link Generation & Email

**Files:**
- `src/lib/auth/magic-link.ts` (NEW)
- `src/lib/email/index.ts` (MODIFY: add `sendMagicLinkEmail`)
- `src/app/api/auth/request/route.ts` (NEW)

**Acceptance Criteria:**

- AC1: `requestMagicLink(email: string, redirectUrl: string, marketingOptIn: boolean): Promise<void>` function in `magic-link.ts`
- AC2: Function deletes all existing unused tokens for the email (`DELETE FROM auth_tokens WHERE email = ? AND used_at IS NULL`)
- AC3: Function generates token using existing `generateSecureToken()` from `src/lib/tokens.ts`
- AC4: Function inserts new row in `authTokens` with 15-minute expiry
- AC5: Function calls `sendMagicLinkEmail()` with verification URL: `${NEXT_PUBLIC_BASE_URL}/auth/verify?token=${token}`
- AC6: `sendMagicLinkEmail()` added to `src/lib/email/index.ts` — sends via Resend with subject "Your ChainSights Access Link", branded HTML body, single CTA button, expiry footer text
- AC7: `POST /api/auth/request` route validates input with Zod: `{ email: z.string().email(), redirectUrl: z.string().url(), marketingOptIn: z.boolean() }`
- AC8: Route enforces rate limit: max 3 requests per email per hour (query `authTokens` count in last hour)
- AC9: Route validates `redirectUrl` is same-origin (starts with `NEXT_PUBLIC_BASE_URL`)
- AC10: Route always returns `{ success: true }` regardless of email existence (anti-enumeration)
- AC11: Route returns 429 if rate limited with message "Too many requests. Try again later."

---

## Story D: Token Verification & Session Creation

**Files:**
- `src/app/auth/verify/page.tsx` (NEW: Server Component page route)

**Acceptance Criteria:**

- AC1: Page route at `/auth/verify` (NOT an API route — must be a navigable page)
- AC2: Reads `token` from `searchParams`
- AC3: Looks up token in `authTokens` table where `token = ? AND usedAt IS NULL`
- AC4: If token not found → renders error page with "This link is invalid or has already been used"
- AC5: If token expired (`expiresAt < NOW()`) → renders error page with "This link has expired. Please request a new one."
- AC6: If token valid → marks as used (`SET usedAt = NOW()`)
- AC7: Upserts user in `users` table: INSERT on first auth (email, role='free', marketingOptIn from token request), UPDATE `lastAuthAt` on subsequent auths
- AC8: Creates session JWT via `createSessionToken({ email, role })`
- AC9: Sets `cs_session` cookie via `setSessionCookie(jwt)`
- AC10: If `marketingOptIn` was true on the original request → upsert into `leads` table with `source: 'magic_link'`
- AC11: Redirects to `redirectUrl` stored in token (or `/matrix` as fallback)
- AC12: Error pages are styled consistently with ChainSights branding (navy background, white text)

**Note:** The `marketingOptIn` value needs to be stored alongside the token. Add `marketingOptIn` boolean column to `authTokens` table (update Story A schema accordingly).

**Schema Update for Story A:**
- AC4 updated: `authTokens` also includes `marketingOptIn` (boolean, default false)

---

## Story E: Middleware Session Resolution

**Files:**
- `src/middleware.ts` (MODIFY)

**Acceptance Criteria:**

- AC1: Before existing rate-limit logic, read `cs_session` cookie from request
- AC2: If cookie exists, verify JWT using `jose` `jwtVerify()` directly in middleware (cannot import from `src/lib/auth/session.ts` — Edge Runtime may not support all Node imports)
- AC3: If JWT valid → set `x-user-email` and `x-user-role` headers on the request via `NextResponse` headers
- AC4: If JWT invalid or expired → do NOT set headers (user treated as anonymous)
- AC5: Never block requests — session resolution is passive (read-only)
- AC6: `AUTH_SECRET` loaded from `process.env` (available in Edge Runtime)
- AC7: Existing rate-limit and security header logic unchanged
- AC8: No DB calls in middleware — JWT verification is self-contained

---

## Story F: Matrix Pages Session Integration

**Files:**
- `src/app/matrix/page.tsx` (MODIFY)
- `src/app/matrix/[slug]/page.tsx` (MODIFY)

**Acceptance Criteria:**

- AC1: Both pages import `headers` from `next/headers`
- AC2: Read `x-user-email` header as primary email source
- AC3: Fall back to `searchParams.email` query param if header not present (migration compatibility)
- AC4: Pass resolved email to `getMatrixAccess()` — function itself unchanged
- AC5: Remove `email` from `MatrixClient` props where it was only used for URL construction (session handles this now)
- AC6: Existing ISR caching (`revalidate = 3600`) continues to work — headers are per-request, not cached
- AC7: Anonymous users (no session, no query param) still see limited data as before
- AC8: Admin users with session see all charts without `?email=` in URL (BUG-1 resolved)

---

## Story G: Auth API Routes (Session Check + Logout)

**Files:**
- `src/app/api/auth/session/route.ts` (NEW)
- `src/app/api/auth/logout/route.ts` (NEW)

**Acceptance Criteria:**

- AC1: `GET /api/auth/session` reads `cs_session` cookie, verifies JWT
- AC2: If valid → returns `{ authenticated: true, email: string, role: string }`
- AC3: If invalid/missing → returns `{ authenticated: false }`
- AC4: `POST /api/auth/logout` clears `cs_session` cookie
- AC5: Logout returns `{ success: true }` and cookie is deleted
- AC6: Both routes apply standard security headers
- AC7: Session route is GET (no mutation, cacheable by browser for short TTL)

---

## Story H: UI Updates — SignInModal + UserMenu

**Files:**
- `src/components/auth/SignInModal.tsx` (NEW)
- `src/components/auth/UserMenu.tsx` (NEW)
- `src/components/matrix-detail/LockedChartOverlay.tsx` (MODIFY)
- `src/app/matrix/matrix-client.tsx` (MODIFY)

**Acceptance Criteria:**

- AC1: `SignInModal` is a dialog component with: email input, marketing opt-in checkbox (unchecked by default, label: "Send me governance insights and updates"), submit button "Send Sign-In Link"
- AC2: Modal header text: **"Sign in to unlock"**, subtext: *"We'll send you a sign-in link — no password needed."*
- AC3: On submit, POST to `/api/auth/request` with `{ email, redirectUrl: window.location.href, marketingOptIn }`
- AC4: On success, modal switches to confirmation state: "Check your email — we sent a sign-in link to {email}" with "Didn't receive it? Resend" button (respects rate limit)
- AC5: On error (429), show: "Too many requests. Please try again in a few minutes."
- AC6: `LockedChartOverlay` type='email' replaced: instead of inline email form, shows "Sign in to unlock" button that opens `SignInModal`
- AC7: `LockedChartOverlay` type='subscribe' unchanged (paid subscribe flow stays as-is)
- AC8: `UserMenu` component: shows truncated email + dropdown with "Sign out" option
- AC9: `UserMenu` calls `GET /api/auth/session` on mount to check auth state
- AC10: `UserMenu` "Sign out" calls `POST /api/auth/logout` then reloads page
- AC11: Matrix paywall overlay in `matrix-client.tsx` updated: anonymous section shows "Sign in" button (opens modal) instead of inline email form
- AC12: All analytics events updated: `trackEvent('cta_click', { button_name: 'sign_in_modal_open', location: '...' })`

---

## Story I: Header Integration

**Files:**
- `src/components/Header.tsx` or equivalent header component (MODIFY)

**Acceptance Criteria:**

- AC1: `UserMenu` component integrated into site header (right side, next to existing nav)
- AC2: When authenticated → shows `UserMenu` (email + sign out)
- AC3: When not authenticated → shows "Sign In" text button that opens `SignInModal`
- AC4: Admin users see small "Admin" badge next to email (optional, low priority)
- AC5: Mobile responsive: UserMenu works in hamburger menu
- AC6: No layout shift — UserMenu loads auth state client-side after mount

---

## Implementation Order

```
Story A → Story B → Story C → Story D → Story E → Story F → Story G → Story H → Story I
```

**Critical path:** A → B → C → D → E → F (core auth flow works end-to-end)
**UI polish:** G → H → I (can be done in parallel after F)

**Dependencies:**
- Story B depends on A (jose installed, schema ready)
- Story C depends on B (session library for token/email functions)
- Story D depends on C (token verification uses magic-link functions)
- Story E depends on B (middleware uses jose for JWT verify)
- Story F depends on E (pages read headers set by middleware)
- Story G depends on B (session check/logout use session library)
- Story H depends on C + G (modal calls auth/request, UserMenu calls auth/session)
- Story I depends on H (header integrates UserMenu)
