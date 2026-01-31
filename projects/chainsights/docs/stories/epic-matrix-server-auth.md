# Epic: Matrix Server-Side Access Control

**Status:** Ready for Implementation
**Priority:** High
**Created:** 2026-01-31
**Decision Source:** Party Mode discussion (Winston, John, Maya, Paige, Barry)

## Problem Statement

The Matrix paywall is entirely client-side. All DAO data is delivered to the browser via Server Component, and access gating happens in JavaScript. Anyone can bypass it with `?subscribed=true` in the URL. This is a credibility risk and undermines the subscription model.

## Solution

Move access control to the server. The Server Component determines the user's access level and delivers ONLY the permitted rows. Client-side code renders what it receives — no bypass possible for the paid tier.

## Architecture Decisions

- **Subscription check**: Server-side Stripe DB lookup (existing `matrix_subscriptions` table)
- **Email tier**: Query parameter `?email=x` — not tamper-proof, but acceptable (real protection is on paid tier)
- **Admin override**: `MATRIX_ADMIN_EMAILS` env variable for full access without subscription
- **Shared utility**: `getMatrixAccessLevel()` function designed for reuse by future Details Page
- **No new auth system**: No Magic Link, no sessions, no cookies. Minimal server-side checks only.

## Access Levels

| Level | Detection | Matrix Rows | Future Details Page Charts |
|-------|-----------|-------------|---------------------------|
| Anonymous | No params | 5 | 1 of 5 |
| Free (Email) | `?email=x` param | 10 | 2 of 5 |
| Subscriber | Stripe DB match | All | All 5 |
| Admin | Env whitelist | All | All 5 |

---

## Story A: Create shared access level utility

**File:** `src/lib/matrix/access.ts` (new)

### Acceptance Criteria

- [ ] AC1: Export `getMatrixAccessLevel(email?: string)` function
- [ ] AC2: Returns `{ level: 'anonymous' | 'free' | 'subscriber' | 'admin', limit: number }`
- [ ] AC3: If email matches `MATRIX_ADMIN_EMAILS` env variable (comma-separated list), return `admin` with `Infinity` limit
- [ ] AC4: If email has active subscription in `matrix_subscriptions` table (status = 'active'), return `subscriber` with `Infinity` limit
- [ ] AC5: If email is provided but no subscription, return `free` with limit `MATRIX_LIMITS.free` (10)
- [ ] AC6: If no email, return `anonymous` with limit `MATRIX_LIMITS.anonymous` (5)
- [ ] AC7: Function handles DB errors gracefully — falls back to `anonymous` on failure (never blocks page render)

---

## Story B: Server-side row limiting in Matrix page

**File:** `src/app/matrix/page.tsx` (modify)

### Acceptance Criteria

- [ ] AC1: Read `email` from searchParams
- [ ] AC2: Call `getMatrixAccessLevel(email)` to determine access
- [ ] AC3: Slice `rankings` array to `limit` rows BEFORE passing to `MatrixClient`
- [ ] AC4: Pass `accessLevel` and `totalCount` (unsliced length) as props to `MatrixClient`
- [ ] AC5: Remove `justSubscribed` prop — access is now determined server-side
- [ ] AC6: Keep `canceled` prop for Stripe cancel banner

---

## Story C: Update MatrixClient for server-driven access

**File:** `src/app/matrix/matrix-client.tsx` (modify)

### Acceptance Criteria

- [ ] AC1: Accept new props: `accessLevel: 'anonymous' | 'free' | 'subscriber' | 'admin'` and `totalCount: number`
- [ ] AC2: Remove client-side access level calculation (delete `justSubscribed`, `emailSubmitted` state for gating purposes)
- [ ] AC3: Remove `?subscribed=true` URL parameter bypass
- [ ] AC4: Sort gating uses `accessLevel` prop instead of client-side `access` variable
- [ ] AC5: Email form submission redirects to `/matrix?email={input}` (triggers server-side re-render with new access level)
- [ ] AC6: Subscribe button flow unchanged (Stripe checkout → redirect with `?subscribed=true` removed, use `?email={email}` after successful subscription)
- [ ] AC7: `hiddenCount` calculated from `totalCount - visibleData.length`
- [ ] AC8: Blurred rows still render as visual paywall hint using `hiddenCount`
- [ ] AC9: Export button only visible for `subscriber` or `admin` access levels

---

## Story D: Update Matrix subscribe flow for server-side access

**Files:** `src/app/api/matrix/subscribe/route.ts`, Stripe webhook handler

### Acceptance Criteria

- [ ] AC1: After successful Stripe subscription checkout, redirect to `/matrix?email={customer_email}` instead of `?subscribed=true`
- [ ] AC2: Stripe webhook for `checkout.session.completed` (subscription) already creates `matrix_subscriptions` entry — verify this works
- [ ] AC3: User lands on Matrix with full access because `getMatrixAccessLevel` finds their active subscription

---

## Story E: Add MATRIX_ADMIN_EMAILS env variable

### Acceptance Criteria

- [ ] AC1: Add `MATRIX_ADMIN_EMAILS` to `.env.example` with placeholder value
- [ ] AC2: Set `MATRIX_ADMIN_EMAILS=mario.semper@masem.at` in Vercel env (production + preview)
- [ ] AC3: Document in `project_context.md` under Feature Flags or a new "Admin Access" section
- [ ] AC4: Admin access works: visiting `/matrix?email=mario.semper@masem.at` shows all rows with full sorting and export

---

## Story F: Update export API for server-side access check

**File:** `src/app/api/matrix/export/route.ts` (modify)

### Acceptance Criteria

- [ ] AC1: Export API validates email parameter against `getMatrixAccessLevel()`
- [ ] AC2: Only `subscriber` or `admin` access levels can export
- [ ] AC3: Return 403 for unauthorized export attempts
- [ ] AC4: No change to export format (CSV/JSON as before)

---

## Implementation Order

1. **Story A** — Shared utility (no dependencies)
2. **Story E** — Env variable (no dependencies, needed for testing)
3. **Story B** — Server-side limiting (depends on A)
4. **Story C** — Client update (depends on B)
5. **Story D** — Subscribe flow update (depends on C)
6. **Story F** — Export protection (depends on A)

## Out of Scope

- Magic Link Auth (Phase 2 — separate epic)
- Matrix Details Page (separate epic — will reuse `getMatrixAccessLevel()`)
- Session/cookie-based auth
- Email verification
