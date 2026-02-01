# Story 4.2: Magic Link Request API

**Epic:** 4 - Auth & User Dashboard (Phase 2)
**Status:** ready-for-dev
**Created:** 2026-01-21
**Depends On:** Story 4.1 (Database Schema)

## User Story

As a **user**,
I want **to request a magic link by entering my email**,
So that **I can access my account without remembering a password**.

## Acceptance Criteria

### AC1: Successful Magic Link Request
**Given** a user submits their email to POST /api/auth/request-link
**When** the email is valid
**Then** a magic link token is generated using crypto.randomUUID
**And** the token is stored in magic_links table with expires_at = NOW + 1 hour
**And** an email is sent via Resend with the link containing the token
**And** the email clearly states "This link expires in 1 hour"
**And** a 200 response is returned with { success: true }

### AC2: Rate Limiting
**Given** a user submits their email to POST /api/auth/request-link
**When** the email has exceeded 5 requests in the last hour
**Then** a 429 response is returned with rate limit error
**And** no new magic link is created

### AC3: New User Creation
**Given** a user requests a magic link
**When** the email doesn't exist in the users table
**Then** a new user record is created with tier='free'
**And** the magic link is still generated and sent

### AC4: Email Format
**Given** the magic link email is sent
**When** rendered
**Then** it includes the tellingCube branding
**And** it includes a prominent "Sign in to tellingCube" button
**And** it includes the magic link URL
**And** it states "This link expires in 1 hour"
**And** it has a text fallback for email clients that don't render HTML

## Technical Details

### Source References
- Architecture: `docs/architecture/auth-dashboard-architecture.md` (Section 2)
- Epic Definition: `docs/epics-big-bang.md` (Story 4.2)

### API Endpoint
```
POST /api/auth/request-link
Content-Type: application/json

Request Body:
{
  "email": "user@example.com"
}

Success Response (200):
{
  "success": true,
  "message": "Check your email for a login link"
}

Rate Limit Response (429):
{
  "error": "rate_limit_exceeded",
  "message": "Too many requests. Please try again later."
}
```

### Files to Create/Modify
1. `app/api/auth/request-link/route.ts` - API endpoint (NEW)
2. `lib/auth/magic-link.ts` - Magic link generation logic (NEW)
3. `lib/email/templates/magic-link.tsx` - Email template (NEW)

### Implementation Notes
- Use `crypto.randomUUID()` for token generation (secure, no dependencies)
- Rate limiting: Count magic_links by email in last hour
- Use Resend for email delivery (already configured)
- Magic link URL format: `{BASE_URL}/api/auth/verify?token={token}`
- Set `created_for: 'login'` for login requests (vs 'purchase_complete')

## Tasks

- [ ] Create app/api/auth/request-link/route.ts
- [ ] Create lib/auth/magic-link.ts with generateMagicLink function
- [ ] Implement rate limiting check (5 per email per hour)
- [ ] Create lib/email/templates/magic-link.tsx email template
- [ ] Update lib/email/send-email.ts to support magic link template
- [ ] Add input validation (email format)

## Definition of Done

- [ ] All acceptance criteria pass
- [ ] API returns 200 with valid email
- [ ] API returns 429 when rate limited
- [ ] Email is delivered via Resend
- [ ] Magic link record created in database
- [ ] TypeScript compiles without errors
