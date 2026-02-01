# Story 4.4: Session Management APIs

**Epic:** 4 - Auth & User Dashboard (Phase 2)
**Status:** ready-for-dev
**Created:** 2026-01-21
**Depends On:** Story 4.3 (JWT Session)

## User Story

As a **user**,
I want **to check my session status and log out**,
So that **I can manage my authentication state**.

## Acceptance Criteria

### AC1: Get Current User (GET /api/auth/me)
**Given** a logged-in user calls GET /api/auth/me
**When** the JWT cookie is valid
**Then** the response contains { user: { id, email, tier, entitlements } }
**And** if the JWT has < 1 day remaining, a refreshed JWT cookie is set

### AC2: Unauthenticated Access
**Given** an unauthenticated user calls GET /api/auth/me
**When** no valid JWT cookie exists
**Then** a 401 response is returned

### AC3: Logout (POST /api/auth/logout)
**Given** a logged-in user calls POST /api/auth/logout
**When** the request is processed
**Then** the JWT cookie is cleared
**And** a 200 response is returned

## Technical Details

### Source References
- Architecture: `docs/architecture/auth-dashboard-architecture.md` (Section 2)
- Epic Definition: `docs/epics-big-bang.md` (Story 4.4)

### API Endpoints

#### GET /api/auth/me
```
Response (200 - Authenticated):
{
  "user": {
    "id": "uuid",
    "email": "user@example.com",
    "tier": "lifetime",
    "entitlements": {
      "tier": "lifetime",
      "billingModel": "one_time",
      "maxYears": 3,
      "hasHrDomain": false,
      "generationsRemaining": "unlimited",
      "isSubscriptionActive": false
    }
  }
}

Response (401 - Unauthenticated):
{
  "error": "unauthenticated",
  "message": "Not logged in"
}
```

#### POST /api/auth/logout
```
Response (200):
{
  "success": true,
  "message": "Logged out successfully"
}
Set-Cookie: tellingcube_auth=; Path=/; HttpOnly; Max-Age=0
```

### Files to Create/Modify
1. `app/api/auth/me/route.ts` - Get current user endpoint (NEW)
2. `app/api/auth/logout/route.ts` - Logout endpoint (NEW)
3. `lib/auth/session.ts` - Session helper functions (NEW)

### Implementation Notes
- Parse JWT from cookie header (not body)
- Refresh JWT if less than 1 day remaining (using shouldRefreshJwt)
- Clear cookie by setting Max-Age=0
- Return full user entitlements for frontend use

## Tasks

- [ ] Create lib/auth/session.ts with getSession helper
- [ ] Create app/api/auth/me/route.ts endpoint
- [ ] Create app/api/auth/logout/route.ts endpoint
- [ ] Implement JWT refresh logic in /me endpoint

## Definition of Done

- [ ] All acceptance criteria pass
- [ ] GET /api/auth/me returns user info when authenticated
- [ ] GET /api/auth/me returns 401 when not authenticated
- [ ] POST /api/auth/logout clears the cookie
- [ ] JWT is refreshed when close to expiry
- [ ] TypeScript compiles without errors
