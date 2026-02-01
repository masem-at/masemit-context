# Story 4.3: Magic Link Verification & JWT Session

**Epic:** 4 - Auth & User Dashboard (Phase 2)
**Status:** ready-for-dev
**Created:** 2026-01-21
**Depends On:** Story 4.2 (Magic Link Request API)

## User Story

As a **user**,
I want **to click the magic link in my email and be logged in**,
So that **I can access my dashboard securely**.

## Acceptance Criteria

### AC1: Valid Token Verification
**Given** a user clicks a magic link with a valid token
**When** GET /api/auth/verify?token={token} is called
**Then** the token is validated (exists, not expired, not used)
**And** the token is marked as used (used_at = NOW)
**And** a JWT is created with payload { userId, email, tier, exp }
**And** the JWT is set as an httpOnly cookie (secure, sameSite: lax, 7-day expiry)
**And** the user is redirected to the redirect_to URL (default: /dashboard)

### AC2: Invalid/Expired Token
**Given** a user clicks an expired or used magic link
**When** GET /api/auth/verify is called
**Then** a 401 response is returned
**And** the user is redirected to /login with error message

### AC3: User Login Tracking
**Given** a valid magic link is verified
**When** the user is logged in
**Then** last_login_at is updated to NOW
**And** login_count is incremented

## Technical Details

### Source References
- Architecture: `docs/architecture/auth-dashboard-architecture.md` (Section 2)
- Epic Definition: `docs/epics-big-bang.md` (Story 4.3)

### API Endpoint
```
GET /api/auth/verify?token={token}

Success: 302 Redirect to /dashboard (or redirect_to)
Set-Cookie: auth_token=<jwt>; HttpOnly; Secure; SameSite=Lax; Max-Age=604800

Error: 302 Redirect to /login?error=invalid_token
```

### JWT Payload
```typescript
{
  userId: string    // User UUID
  email: string     // User email
  tier: Tier        // User tier (free, single, lifetime, etc.)
  exp: number       // Expiry timestamp (7 days from now)
  iat: number       // Issued at timestamp
}
```

### Files to Create/Modify
1. `app/api/auth/verify/route.ts` - Verification endpoint (NEW)
2. `lib/auth/jwt.ts` - JWT signing and verification (NEW)
3. `lib/auth/magic-link.ts` - Already has verifyMagicLink (from 4.2)

### Environment Variables Required
```
JWT_SECRET=<32+ character secret>
```

### Implementation Notes
- Use jose library for JWT (already in package.json)
- Cookie name: `tellingcube_auth` (consistent naming)
- Cookie settings: httpOnly=true, secure=true, sameSite='lax', maxAge=7*24*60*60
- Handle edge case: token exists but user was deleted

## Tasks

- [ ] Create lib/auth/jwt.ts with signJwt and verifyJwt functions
- [ ] Create app/api/auth/verify/route.ts endpoint
- [ ] Implement redirect handling for success/error cases
- [ ] Test with valid, expired, and used tokens

## Definition of Done

- [ ] All acceptance criteria pass
- [ ] Valid token creates session and redirects to dashboard
- [ ] Invalid token redirects to login with error
- [ ] JWT cookie is set correctly (httpOnly, secure, sameSite)
- [ ] User login stats are updated
- [ ] TypeScript compiles without errors
