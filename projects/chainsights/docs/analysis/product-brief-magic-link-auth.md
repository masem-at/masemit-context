# Product Brief: Magic Link Authentication

**Author:** Mario Semper (requirements spec) + John (PM, consolidated)
**Date:** 2026-01-31
**Status:** Product Brief Complete — Ready for Architecture
**Priority:** Critical
**Source Documents:**
- `docs/_masemIT/requirements/chainsights-bugs-auth-backlog.md` (Part 2)
- `docs/_masemIT/party-mode-auth-dao-matrix-2026-01-31.md`

---

## Vision

A lightweight, passwordless authentication layer for ChainSights. Users authenticate via a magic link sent to their email — no passwords, no signup forms, no OAuth. Creates persistent sessions that unlock gated content across all pages and visits.

---

## Problem Statement

The current email unlock mechanism is stateless — users must re-enter their email on every page visit. This creates three critical issues:

1. **BUG-1 (Admin Access):** Admin users have no way to access gated charts on detail pages. `getMatrixAccess()` requires an email query parameter, and there is no session to auto-resolve admin status.
2. **BUG-2 (Session Persistence):** After entering email to unlock charts, navigating away and returning requires unlocking again. The state is per-page, not per-session.
3. **Lead Gen Failure:** Users who experience the email-gate friction once will not do it again. This likely contributes to the 0% CTR on hero CTAs, as returning visitors already experienced this friction.

**Root Cause (Mary, Analyst):** "There is no session management. The email unlock is stateless, per-page. That's not a bug, that's missing infrastructure."

---

## Target Users

1. **Anonymous visitors** — First-time visitors who see gated content, need low-friction path to unlock
2. **Returning users** — Previously unlocked via email, expect content to still be accessible
3. **Admin (Mario)** — Needs full access for QA, testing, screenshot creation without any gating
4. **Subscribers** — Paid Matrix subscribers whose access level must persist across sessions

---

## User Flow

1. User clicks any gated CTA (unlock charts, subscribe, get report)
2. Modal appears: **"Sign in to unlock"** with subtext: *"We'll send you a sign-in link — no password needed."*
3. User enters email + optional marketing opt-in checkbox (unchecked by default, GDPR)
4. System sends email with magic link (valid for 15 minutes)
5. User clicks link in email
6. System creates session (HTTP-only cookie, 30 days)
7. User is redirected back to the page they were on
8. All gated content is now unlocked for the session duration

**Copy Decision (Paige, PM):** Replace all "Enter your email to unlock more charts" with "Sign in to unlock" — short, clear, trustworthy. The old copy "sounds like spam."

---

## Functional Requirements

### FR1: Magic Link Generation

- Generate a unique, signed token per authentication request
- Token contains: email, timestamp, redirect URL
- Token expires after 15 minutes
- One active token per email at a time (new request invalidates previous)
- Use existing Resend Pro for email delivery

### FR2: Magic Link Email

- Subject: "Your ChainSights Access Link"
- Body: Clean, branded email with single CTA button
- Link format: `https://chainsights.one/auth/verify?token={token}`
- Footer text: "This link expires in 15 minutes. If you didn't request this, ignore this email."
- This is a **brand touchpoint** — first email contact with many users

### FR3: Session Management

| Property | Value |
|----------|-------|
| Storage | HTTP-only cookie |
| Name | `cs_session` |
| Duration | 30 days |
| Secure | `true` (production) |
| SameSite | `Lax` |
| Content | Signed JWT with email + role |

### FR4: User Roles

| Role | Access Level | How Assigned |
|------|-------------|--------------|
| `admin` | All charts, all features, no gating | Manual DB flag (role column) |
| `free` | Unlocked charts (no email gate), basic Matrix view | After magic link auth |
| `subscriber` | Full Matrix, export, historical data | After Stripe subscription |
| `anonymous` | Gated charts, limited Matrix | No auth / expired session |

**Note:** These roles already exist in `MatrixAccess` type (`src/lib/db/subscriptions.ts`). Magic Link Auth must integrate with the existing `getMatrixAccess()` function, not replace it.

### FR5: Auth State Check

- On every page load, check for valid `cs_session` cookie
- If valid → resolve user role → render appropriate access level
- If expired or missing → treat as `anonymous`
- Middleware-based check (Next.js middleware or layout-level)
- Must work with existing ISR caching (1-hour revalidation)

### FR6: Lead Capture Integration

- On first magic link authentication, create/update record in `users` table
- Store: email, first_auth_at, last_auth_at, source page (which URL triggered auth)
- Marketing opt-in: stored as boolean, separate from auth consent (GDPR)
- Existing `leads` table data should be considered for migration

---

## Technical Requirements

### TR1: Database Schema

Two new tables required:

```sql
-- Users table (lightweight auth + role management)
CREATE TABLE users (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  email VARCHAR(255) UNIQUE NOT NULL,
  role VARCHAR(20) DEFAULT 'free' CHECK (role IN ('admin', 'free', 'subscriber')),
  marketing_optin BOOLEAN DEFAULT false,
  first_auth_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  last_auth_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Magic link tokens (short-lived, one per email)
CREATE TABLE auth_tokens (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  email VARCHAR(255) NOT NULL,
  token VARCHAR(255) UNIQUE NOT NULL,
  redirect_url TEXT,
  expires_at TIMESTAMP WITH TIME ZONE NOT NULL,
  used_at TIMESTAMP WITH TIME ZONE,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Indexes
CREATE INDEX idx_auth_tokens_token ON auth_tokens(token) WHERE used_at IS NULL;
CREATE INDEX idx_auth_tokens_email ON auth_tokens(email);
```

### TR2: API Endpoints

| Endpoint | Method | Purpose |
|----------|--------|---------|
| `/api/auth/request` | POST | Request magic link (input: email, redirect URL, marketing opt-in) |
| `/api/auth/verify` | GET | Verify token, create session, redirect to original page |
| `/api/auth/session` | GET | Check current session status (for client-side UI) |
| `/api/auth/logout` | POST | Clear session cookie |

### TR3: Component Changes

- Replace all "Enter your email to unlock" overlays with "Sign in to unlock" → triggers magic link modal
- Add session-aware wrapper component for gated content
- Add user menu in header (minimal): shows email + "Sign out" when authenticated
- Admin users: show "Admin" badge in nav (optional)

---

## Architecture Questions for Winston

1. **JWT Library:** `jose` (Edge-compatible) vs. alternatives for Next.js Middleware?
2. **Middleware Scope:** Only `/matrix/*` routes, or also `/rankings`, `/api/*`?
3. **Users Table vs. Leads Table:** New `users` table as specified, or extend existing `leads` table?
4. **Token Strategy:** DB-stored tokens (as in spec) vs. stateless signed URLs (HMAC)?
5. **getMatrixAccess() Integration:** How does session-based auth replace the current `?email=` query param approach? Middleware resolves session → injects email into Server Component context?

**Winston's initial notes from Party Mode:**
- Use `jose` for Edge-compatible JWT (Next.js Middleware runs in Edge Runtime)
- Token generation: `crypto.randomUUID()` + HMAC signature, no JWT for the magic link itself
- Session check via Next.js Middleware on protected routes
- DB: Neon Postgres + Drizzle ORM — `users` + `auth_tokens` tables as specified

---

## Migration Path

1. Deploy magic link auth **alongside** existing email unlock (no breaking change)
2. Existing email unlock still works but now **also creates a session**
3. After validation, remove old email unlock entirely
4. All gated content checks session cookie instead of per-page email state

**This means:** During migration, both `?email=` query param AND `cs_session` cookie are valid. `getMatrixAccess()` checks both, preferring session.

---

## Privacy & Compliance (GDPR)

- Email stored for authentication = **legitimate interest** under GDPR (no consent needed)
- Marketing opt-in = **separate and explicit** (checkbox, unchecked by default)
- User can request data deletion via email to hello@chainsights.one
- Session cookie is **functional** → no consent banner required
- Update Privacy Policy to mention magic link authentication
- No data shared with third parties (Resend is processor, not controller)

---

## Stories (from Party Mode, Sally's breakdown)

| # | Story | Resolves |
|---|-------|----------|
| A | DB Schema (users + auth_tokens tables) | Foundation |
| B | Magic Link Generation + Email via Resend | FR1, FR2 |
| C | Token Verification + Session Creation | FR3, BUG-2 |
| D | Middleware + Role Resolution | FR5 |
| E | UI Updates (Lock Overlay → "Sign in to unlock") | TR3, Copy |
| F | Lead Capture Integration | FR6 |
| G | Admin Access (session-based, no email gate) | FR4, BUG-1 |
| H | Privacy/GDPR Compliance | Compliance |

**Note:** These are preliminary story outlines. Detailed epics & stories with acceptance criteria will be created after Architecture is complete.

---

## Success Metrics

| Metric | Current | Target |
|--------|---------|--------|
| Repeat unlock rate | 0% (users don't re-enter email) | >60% return with active session |
| Admin testing friction | Cannot test paid experience | Zero-friction full access |
| Lead capture per session | 1 email per page view | 1 email per 30-day session (higher quality) |
| Session persistence | None (stateless) | 30 days |

---

## Out of Scope

- Password-based authentication (never)
- OAuth/social login (future consideration)
- Wallet-Connect login (Phase 3, separate brief)
- API key generation (Phase 3)
- User dashboard / report history (separate feature)
