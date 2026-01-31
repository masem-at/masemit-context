# Architecture: Magic Link Authentication

**Author:** Winston (Architect)
**Date:** 2026-01-31
**Status:** Architecture Complete — Ready for Epics & Stories
**Input:** `docs/analysis/product-brief-magic-link-auth.md`

---

## Architecture Decisions

### AD-1: JWT Library → `jose`

**Decision:** Use `jose` (Edge-compatible JOSE implementation).

**Rationale:**
- Next.js Middleware runs in Edge Runtime — `jsonwebtoken` uses Node.js `crypto` module and **will not work** in Edge.
- `jose` is the de facto standard for JWT in Edge/Serverless environments.
- Lightweight (~10KB), zero dependencies, maintained by Auth.js team.
- Already used by NextAuth.js, Auth.js — proven in production at scale.

**Impact:**
- New dependency: `jose` (npm install)
- Used in: Middleware (session verification), API routes (session creation/verification)
- Session cookie `cs_session` contains a signed JWT: `{ email, role, iat, exp }`
- Signing key: `AUTH_SECRET` env variable (32+ byte random string)

**Rejected:**
- `jsonwebtoken` — Not Edge-compatible. Would require separate code paths for Middleware vs API routes.
- Plain JSON cookie (current admin pattern) — Not signed, trivially forgeable. The admin panel is internal-only so the risk is acceptable there, but user-facing auth needs proper signing.

---

### AD-2: Middleware Scope → All routes, lightweight check

**Decision:** Extend existing Middleware to resolve session on **all routes** (not just `/matrix/*`).

**Rationale:**
- The `cs_session` cookie is set on the root domain. Middleware already runs on all routes (current matcher: `/((?!_next/static|_next/image|favicon.ico).*)`).
- Session resolution is a single JWT verification (~0.1ms on Edge). No DB call needed.
- Resolved user info is passed downstream via `x-user-email` and `x-user-role` request headers.
- Server Components read these headers to determine access level — no query params needed.
- This replaces the current `?email=` query param pattern completely (after migration).

**Implementation:**
```
Middleware Flow:
1. Read `cs_session` cookie
2. If present → verify JWT signature + expiry with jose
3. If valid → set x-user-email + x-user-role headers on request
4. If invalid/missing → no headers (anonymous)
5. Continue to existing rate-limit + security header logic
```

**Why not just `/matrix/*`?**
- Future features (report history, saved preferences) need auth on other routes.
- The cost of checking a cookie on every request is negligible.
- Consistent architecture: one place to check auth, one pattern for all pages.

---

### AD-3: New `users` table — Do NOT extend `leads`

**Decision:** Create a new `users` table as specified in the Product Brief. Keep `leads` table separate.

**Rationale:**
- **Separation of concerns:** `leads` is a marketing/analytics table. `users` is an auth table. Different lifecycles, different access patterns.
- The `leads` table tracks engagement metrics (`freeChecksCount`, `paidReportsCount`, `lastActivityAt`) that don't belong in auth.
- `users` table needs a `role` column with CHECK constraint — adding this to `leads` would muddy its purpose.
- The `leads` table has 267 lines of existing code depending on its shape. Changing it risks regressions.
- `users` rows are created on magic link verification. `leads` rows are created on Quick Check completion. Different events.

**Migration note:** On first magic link auth, if the email exists in `leads`, do NOT migrate data. The tables serve different purposes and can coexist. The `leads.source` column can track `'magic_link'` as a new source value when auth triggers lead creation.

**Schema (Drizzle ORM):**
```typescript
// New enum for user roles
export const userRoleEnum = pgEnum('user_role', ['admin', 'free', 'subscriber'])

// Users table
export const users = pgTable('users', {
  id: uuid('id').defaultRandom().primaryKey(),
  email: text('email').notNull().unique(),
  role: userRoleEnum('role').default('free').notNull(),
  marketingOptIn: boolean('marketing_opt_in').default(false).notNull(),
  firstAuthAt: timestamp('first_auth_at').defaultNow().notNull(),
  lastAuthAt: timestamp('last_auth_at').defaultNow().notNull(),
  createdAt: timestamp('created_at').defaultNow().notNull(),
}, (table) => ({
  emailIdx: uniqueIndex('users_email_idx').on(table.email),
}))

// Auth tokens table
export const authTokens = pgTable('auth_tokens', {
  id: uuid('id').defaultRandom().primaryKey(),
  email: text('email').notNull(),
  token: text('token').notNull().unique(),
  redirectUrl: text('redirect_url'),
  expiresAt: timestamp('expires_at').notNull(),
  usedAt: timestamp('used_at'),
  createdAt: timestamp('created_at').defaultNow().notNull(),
}, (table) => ({
  tokenIdx: uniqueIndex('auth_tokens_token_idx').on(table.token),
  emailIdx: index('auth_tokens_email_idx').on(table.email),
}))
```

---

### AD-4: Token Strategy → DB-stored tokens (not stateless HMAC)

**Decision:** Store magic link tokens in the `auth_tokens` DB table.

**Rationale:**
- **One-time use enforcement:** DB-stored tokens can be marked as `used_at` after verification. Stateless HMAC tokens cannot be invalidated after use — the link would work repeatedly until expiry.
- **One-active-per-email:** New token request can `DELETE FROM auth_tokens WHERE email = ?` to invalidate previous. Not possible with stateless tokens.
- **Audit trail:** DB records show when tokens were created, used, and by which email. Useful for debugging and abuse detection.
- **Existing pattern:** The `emailSubscribers` table already uses `confirmationToken` for double opt-in — same concept.
- The DB overhead is minimal: 1 INSERT on request, 1 SELECT + UPDATE on verify. Tokens are short-lived (15 min) and can be cleaned up with a simple cron.

**Token format:** `crypto.randomBytes(32).toString('hex')` — reuse existing `generateSecureToken()` from `src/lib/tokens.ts`.

**Rejected:**
- Stateless HMAC-signed URLs: Cannot enforce one-time use or one-active-per-email. Simpler but less secure.

---

### AD-5: `getMatrixAccess()` Integration → Session-first, query param fallback during migration

**Decision:** Extend `getMatrixAccess()` to check session cookie first, then fall back to `?email=` query param during migration.

**Rationale:**
- The function signature stays the same: `getMatrixAccess(email?: string | null): Promise<MatrixAccess>`
- But instead of only getting `email` from query params, the Server Component now gets it from Middleware-injected headers.
- During migration, both paths work. After migration, query param support is removed.

**New flow (Server Component):**
```
1. Middleware resolves cs_session → injects x-user-email, x-user-role headers
2. Server Component reads headers via headers() from next/headers
3. Passes email to getMatrixAccess() — same function, same logic
4. getMatrixAccess() checks admin whitelist → subscription → free (unchanged)
```

**Key change:** `getMatrixAccess()` itself doesn't change. What changes is **how the email gets to it**:
- Before: `searchParams.email` (query param)
- After: `headers().get('x-user-email')` (from Middleware)
- Migration: Check headers first, fall back to query param

**Admin resolution improvement:**
- Currently: Admin needs `?email=admin@example.com` in URL (BUG-1)
- After: Middleware reads session cookie → injects email → `getMatrixAccess()` checks `MATRIX_ADMIN_EMAILS` → returns `admin`
- No URL param needed. Session persists across navigation. BUG-1 solved.

---

## Component Architecture

```
┌─────────────────────────────────────────────────┐
│                   Middleware                      │
│  1. Read cs_session cookie                       │
│  2. Verify JWT (jose)                            │
│  3. Inject x-user-email + x-user-role headers    │
│  4. Rate limiting (existing)                     │
│  5. Security headers (existing)                  │
└──────────────────────┬──────────────────────────┘
                       │
        ┌──────────────┼──────────────┐
        ▼              ▼              ▼
┌──────────────┐ ┌──────────┐ ┌──────────────┐
│ /matrix/*    │ │ /api/auth│ │ Other pages  │
│ Server Comp  │ │ Routes   │ │ (future)     │
│              │ │          │ │              │
│ headers() →  │ │ /request │ │ headers() →  │
│ email, role  │ │ /verify  │ │ email, role  │
│ ↓            │ │ /session │ │              │
│ getMatrix-   │ │ /logout  │ │              │
│ Access()     │ │          │ │              │
└──────────────┘ └──────────┘ └──────────────┘
```

---

## Auth Flow Sequence

### Magic Link Request
```
User → LockedChartOverlay "Sign in to unlock"
  → Modal with email input + marketing opt-in checkbox
  → POST /api/auth/request { email, redirectUrl, marketingOptIn }
  → Server:
    1. Validate email (Zod)
    2. Rate limit: max 3 requests per email per hour
    3. Delete existing tokens for this email
    4. Generate token (generateSecureToken())
    5. Insert into auth_tokens (email, token, redirectUrl, expiresAt: +15min)
    6. Send email via Resend with magic link
    7. Return { success: true, message: "Check your email" }
  → UI shows "Check your email" confirmation
```

### Magic Link Verification
```
User clicks link in email → GET /auth/verify?token={token}
  → This is a PAGE route (not API) — renders loading state then redirects
  → Server Component:
    1. Lookup token in auth_tokens WHERE token = ? AND used_at IS NULL
    2. If not found or expired → render error page
    3. Mark token as used (SET used_at = NOW())
    4. Upsert user in users table (create if new, update last_auth_at if existing)
    5. Create JWT: { email, role } signed with AUTH_SECRET, exp: 30 days
    6. Set cs_session cookie (HTTP-only, Secure, SameSite=Lax, 30 days)
    7. If marketing opt-in → upsert into leads table
    8. Redirect to redirectUrl (or /matrix as default)
```

### Session Check (Client-side)
```
Any Client Component that needs auth state:
  → GET /api/auth/session
  → Server reads cs_session cookie, verifies JWT
  → Returns { authenticated: true, email, role } or { authenticated: false }
  → Used for: user menu display, conditional UI elements
```

---

## API Endpoints Detail

### POST `/api/auth/request`

**Input (Zod validated):**
```typescript
{
  email: string       // Valid email format
  redirectUrl: string  // URL to redirect after verification (same-origin only)
  marketingOptIn: boolean  // GDPR marketing consent
}
```

**Rate limit:** 3 requests per email per hour (prevents spam). Uses existing rate-limit infrastructure.

**Response:** `{ success: true }` (always — don't leak whether email exists)

### GET `/auth/verify` (Page route, not API)

**Why a page route?** The user clicks a link in their email. This must be a navigable page that:
1. Shows a loading state
2. Processes the token server-side
3. Sets the cookie
4. Redirects to the original page

An API route cannot set cookies and redirect in the same response reliably across all browsers.

**Input:** `?token={token}` query parameter

**Success:** Redirect to `redirectUrl` with session cookie set
**Failure:** Render error page ("Link expired or already used")

### GET `/api/auth/session`

**Response:**
```typescript
{ authenticated: true, email: string, role: 'admin' | 'free' | 'subscriber' }
// or
{ authenticated: false }
```

### POST `/api/auth/logout`

**Action:** Delete `cs_session` cookie. Return `{ success: true }`.

---

## File Structure

```
src/
├── middleware.ts                          # MODIFY: Add session resolution
├── lib/
│   ├── auth/
│   │   ├── session.ts                    # NEW: JWT create/verify with jose
│   │   └── magic-link.ts                 # NEW: Token generation, email sending
│   ├── db/
│   │   ├── schema.ts                     # MODIFY: Add users + authTokens tables
│   │   └── subscriptions.ts             # MODIFY: Session-first email resolution
│   ├── email/
│   │   └── index.ts                      # MODIFY: Add sendMagicLinkEmail()
│   └── tokens.ts                         # REUSE: generateSecureToken()
├── app/
│   ├── auth/
│   │   └── verify/
│   │       └── page.tsx                  # NEW: Token verification page
│   ├── api/
│   │   └── auth/
│   │       ├── request/
│   │       │   └── route.ts              # NEW: Magic link request
│   │       ├── session/
│   │       │   └── route.ts              # NEW: Session check
│   │       └── logout/
│   │           └── route.ts              # NEW: Logout
│   └── matrix/
│       ├── page.tsx                      # MODIFY: Read email from headers
│       └── [slug]/
│           └── page.tsx                  # MODIFY: Read email from headers
└── components/
    ├── matrix-detail/
    │   └── LockedChartOverlay.tsx         # MODIFY: "Sign in" flow
    ├── auth/
    │   ├── SignInModal.tsx                # NEW: Email + opt-in modal
    │   └── UserMenu.tsx                  # NEW: Email + Sign out dropdown
    └── matrix/
        └── matrix-client.tsx             # MODIFY: Auth-aware paywall overlay
```

---

## Environment Variables

| Variable | Purpose | Required |
|----------|---------|----------|
| `AUTH_SECRET` | JWT signing key (32+ bytes, `openssl rand -hex 32`) | Yes |
| `RESEND_API_KEY` | Email delivery (already configured) | Yes |
| `MATRIX_ADMIN_EMAILS` | Admin whitelist (already configured) | Yes |
| `NEXT_PUBLIC_BASE_URL` | Magic link URL base (already configured) | Yes |

---

## Migration Strategy

### Phase 1: Deploy alongside existing (no breaking change)

1. Add `users` + `auth_tokens` tables (DB migration)
2. Deploy auth API routes + verify page
3. Middleware checks session cookie BUT `?email=` still works
4. `getMatrixAccess()` prefers session email, falls back to query param
5. LockedChartOverlay shows "Sign in" for new users, email form still works

### Phase 2: Remove legacy email param

1. Remove `?email=` support from Matrix pages
2. Remove email form from LockedChartOverlay (only "Sign in" button)
3. Clean up query param handling in `searchParams`

**Rollback:** If issues arise, revert Middleware changes. Cookie is ignored, query params still work. Zero data loss.

---

## Security Considerations

- **JWT signing:** HMAC-SHA256 via `jose`. Secret in `AUTH_SECRET` env variable.
- **Cookie security:** HTTP-only (no JS access), Secure (HTTPS only in prod), SameSite=Lax (CSRF protection).
- **Token one-time use:** DB-enforced via `used_at` column. Cannot replay magic links.
- **Token expiry:** 15 minutes. Short window limits interception risk.
- **Rate limiting:** 3 magic link requests per email per hour. Prevents abuse.
- **No password storage:** Magic link = no passwords = no password leaks.
- **Same-origin redirect:** `redirectUrl` must match `NEXT_PUBLIC_BASE_URL` origin. Prevents open redirect.
- **Email enumeration:** `/api/auth/request` always returns success. Don't leak whether email exists.

---

## Dependencies

| Package | Version | Purpose |
|---------|---------|---------|
| `jose` | latest | JWT sign/verify (Edge-compatible) |

No other new dependencies needed. Resend, Drizzle, Zod all already installed.
