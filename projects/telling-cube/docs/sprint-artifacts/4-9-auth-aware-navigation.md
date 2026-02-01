# Story 4.9: Auth-Aware Navigation

**Status:** done
**Epic:** 4 - Auth & User Dashboard
**Points:** 3
**Created:** 2026-01-21

---

## Story

As a **logged-in user**,
I want **seamless navigation between dashboard and marketing pages with visible auth state**,
so that **I can easily access my dashboard from anywhere and logout when needed**.

---

## Acceptance Criteria

1. **AC1: Middleware Redirect for Logged-In Users**
   - [x] Logged-in user visiting `/` is redirected to `/dashboard`
   - [x] Redirect happens server-side (no flash of landing page)
   - [x] Anonymous users see landing page normally

2. **AC2: Dashboard Header Shows "Home" Link**
   - [x] Dashboard header includes "Home" link pointing to `/`
   - [x] Link bypasses middleware redirect (explicit navigation via `/?home=true`)
   - [x] Positioned appropriately in header navigation

3. **AC3: Dashboard Link on Non-Dashboard Pages**
   - [x] When logged in, all pages show "Dashboard" link in header
   - [x] Link points to `/dashboard`
   - [x] Only visible when user has active session

4. **AC4: Logout Available Everywhere**
   - [x] Logout button/link visible in header when logged in
   - [x] Works on landing page, API docs, and all other pages
   - [x] Clears session and redirects to landing page

5. **AC5: Anonymous User Experience**
   - [x] Anonymous users see "Login" link instead of Dashboard/Logout
   - [x] No Dashboard link shown when not logged in
   - [x] Clean transition between auth states

---

## Tasks / Subtasks

### Task 1: Create Auth-Aware Header Component (AC: #3, #4, #5)
- [x] 1.1 Create or update shared header to check session state
- [x] 1.2 Conditionally render Dashboard/Logout vs Login links
- [x] 1.3 Use consistent styling with existing header

### Task 2: Implement Middleware Redirect (AC: #1)
- [x] 2.1 Create/update `middleware.ts` for root path
- [x] 2.2 Check session cookie validity
- [x] 2.3 Redirect to `/dashboard` if valid session
- [x] 2.4 Allow explicit `/` navigation to bypass (query param or cookie)

### Task 3: Update Dashboard Header (AC: #2)
- [x] 3.1 Add "Home" link to dashboard header
- [x] 3.2 Ensure link works (doesn't loop back to dashboard)

### Task 4: Integration Testing (AC: all)
- [x] 4.1 Test anonymous user flow
- [x] 4.2 Test logged-in user redirect
- [x] 4.3 Test logout from various pages
- [x] 4.4 Test "Home" link from dashboard

---

## Dev Notes

### Session Check Pattern

From existing auth implementation (Epic 4):
```typescript
// Check for valid session cookie
const sessionToken = request.cookies.get('session')?.value
const isLoggedIn = sessionToken && isValidJWT(sessionToken)
```

### Middleware Location

Next.js middleware at `middleware.ts` in project root:
```typescript
import { NextResponse } from 'next/server'
import type { NextRequest } from 'next/server'

export function middleware(request: NextRequest) {
  const { pathname } = request.nextUrl

  if (pathname === '/' && hasValidSession(request)) {
    return NextResponse.redirect(new URL('/dashboard', request.url))
  }

  return NextResponse.next()
}

export const config = {
  matcher: ['/'],
}
```

### Header Component Pattern

Current Header at `components/layout/Header.tsx` needs auth awareness:
```typescript
// Need to check session state
// Server component can read cookies directly
// Client component needs to fetch session status
```

### Bypass Redirect for "Home" Link

Option A: Query parameter `/?home=true` skips redirect
Option B: Different route `/home` that shows landing
Option C: Cookie flag set when clicking "Home"

**Recommendation:** Option A is simplest - middleware checks for `?home=true`

### Files to Reference

- `components/layout/Header.tsx` - Current header (needs auth awareness)
- `app/dashboard/page.tsx` - Dashboard layout
- `lib/auth/session.ts` - Session utilities (if exists)
- `middleware.ts` - May need to create

---

## Dev Agent Record

### Agent Model Used

Claude Opus 4.5 (Amelia - Dev Agent)

### Completion Notes List

- [x] Story created by Bob (SM)
- [x] Auth-aware header implemented
- [x] Middleware redirect working
- [x] All navigation flows tested
- [x] Code review passed (adversarial)

### File List

**Files modified:**
- `middleware.ts` - Added auth redirect logic with JWT verification (Edge-compatible)
- `components/layout/Header.tsx` - Added auth-aware navigation (Dashboard/Logout vs Login)
- `components/dashboard/DashboardContent.tsx` - Added Home link to dashboard header

### Change Log

- 2026-01-21: Story 4.9 created via Party Mode (John, Bob, Mary, Winston, Sally)
- 2026-01-23: Story 4.9 implemented by Amelia (Dev Agent)
