# ChainSights: Bugs, Auth Requirement & Backlog

**Project:** ChainSights (chainsights.one)  
**Date:** 2026-01-31  
**Author:** Mario Semper  
**Version:** 1.0  

---

## Part 1: Bugs (Fix Immediately)

### BUG-1: Admin has no access to gated charts

**Severity:** High  
**Page:** DAO Detail View (e.g., `/matrix/aave`)

**Problem:** When logged in as admin, the detail view still shows locked chart placeholders with "Enter your email to unlock more charts" instead of the actual charts. Admin should have full access to all charts without any gating.

**Expected Behavior:** Admin users see all charts (HPR, DEI, PDI, GPI) fully rendered, identical to a paid/unlocked user view.

**Impact:** Blocks testing, QA, and screenshot creation for marketing. Mario cannot verify what the paid experience looks like.

**Fix:** Check admin/auth status before rendering chart lock overlay. If user is admin → render all charts.

---

### BUG-2: Email unlock does not persist across navigation

**Severity:** High  
**Page:** DAO Detail View → DAO Matrix → DAO Detail View

**Problem:** After entering email to unlock charts on a DAO detail page, navigating back to the Matrix and then to any DAO detail page (same or different) requires unlocking again. The email unlock state is not persisted.

**Expected Behavior:** Once a user unlocks via email, the unlock state persists for the entire session (minimum) or for 30 days (ideal).

**Impact:** Completely breaks the user experience. Users who unlock once and then navigate will not unlock again — they will leave. This likely contributes to the 0% CTR on hero CTAs, as returning visitors already experienced this friction.

**Fix:** Store unlock state in a session cookie or localStorage after successful email submission. Check this state before rendering lock overlay.

---

### BUG-3: "DAO Governance Index" section visible on detail page

**Severity:** Medium  
**Page:** DAO Detail View (bottom section)

**Problem:** The "DAO Governance Index" section at the bottom of each DAO detail page shows:
- Rank within category (e.g., "#4 of 16 DeFi DAOs · #4 overall of 21")
- Bar comparison: DAO score vs. "DeFi Average" (e.g., Lido 7.4 vs DeFi Average 5.5)
- "Excels at" / "Needs work" badges (e.g., "Excels at: Human Participation Rate")

**Why this must be hidden:**
1. The sector averages (e.g., "DeFi Average 5.5") are not based on a defined benchmark methodology. We have not yet defined what constitutes the "DeFi" category benchmark, which DAOs are included, or how the average is weighted.
2. The "Excels at" / "Needs work" badges give away analysis insights that should be part of the paid report, not visible for free.
3. Showing unvalidated benchmarks undermines credibility — our core promise is "Wallets lie. We don't." We cannot show numbers we can't defend.

**Fix:** Hide the entire "DAO Governance Index" section on all DAO detail pages. Remove from rendering until the DAO S&P Benchmark (see Part 3) is properly defined.

---

## Part 2: Requirement — Magic Link Authentication

### Problem Statement

The current email unlock mechanism is stateless — users must re-enter their email on every page visit. This creates unacceptable UX friction and renders the lead generation flow ineffective. Additionally, there is no session management, meaning even admin users lack proper access.

A lightweight authentication layer is needed that:
- Persists user state across pages and sessions
- Provides admin access without email gating
- Enables future features (subscriptions, saved preferences, API keys)
- Stays lightweight (no password management, no OAuth complexity)

### Objective

Implement Magic Link authentication as the primary auth mechanism for ChainSights. Users authenticate via a link sent to their email — no passwords, no signup forms, no OAuth.

### User Flow

1. User clicks any gated CTA (unlock charts, subscribe, get report)
2. Modal appears: "Enter your email to continue"
3. User submits email
4. System sends email with magic link (valid for 15 minutes)
5. User clicks link in email
6. System creates session (cookie-based, 30 days)
7. User is redirected back to the page they were on
8. All gated content is now unlocked for the session duration

### Functional Requirements

#### FR1: Magic Link Generation

- Generate a unique, signed token per authentication request
- Token contains: email, timestamp, redirect URL
- Token expires after 15 minutes
- One active token per email at a time (new request invalidates old)
- Use existing Resend Pro for email delivery

#### FR2: Magic Link Email

- Subject: "Your ChainSights Access Link"
- Body: Clean, branded email with single CTA button
- Link format: `https://chainsights.one/auth/verify?token={token}`
- Include: "This link expires in 15 minutes. If you didn't request this, ignore this email."

#### FR3: Session Management

| Property | Value |
|----------|-------|
| Storage | HTTP-only cookie |
| Name | `cs_session` |
| Duration | 30 days |
| Secure | `true` (production) |
| SameSite | `Lax` |
| Content | Signed JWT with email + role |

#### FR4: User Roles

| Role | Access Level | How Assigned |
|------|-------------|--------------|
| `admin` | All charts, all features, no gating | Manual DB flag |
| `free` | Unlocked charts (no email gate), basic Matrix view | After magic link auth |
| `subscriber` | Full Matrix, export, historical data | After Stripe subscription |
| `anonymous` | Gated charts, limited Matrix | No auth |

#### FR5: Auth State Check

- On every page load, check for valid `cs_session` cookie
- If valid → resolve user role → render appropriate access level
- If expired or missing → treat as `anonymous`
- Middleware-based check (Next.js middleware or layout-level)

#### FR6: Lead Capture Integration

- On first magic link authentication, create/update record in `leads` table
- Store: email, first_auth_at, last_auth_at, source (which page triggered auth)
- Marketing opt-in: include checkbox in magic link request modal (unchecked by default, GDPR)

### Technical Requirements

#### TR1: Database

```sql
-- Users table (lightweight)
CREATE TABLE users (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  email VARCHAR(255) UNIQUE NOT NULL,
  role VARCHAR(20) DEFAULT 'free' CHECK (role IN ('admin', 'free', 'subscriber')),
  marketing_optin BOOLEAN DEFAULT false,
  first_auth_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  last_auth_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Magic link tokens
CREATE TABLE auth_tokens (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  email VARCHAR(255) NOT NULL,
  token VARCHAR(255) UNIQUE NOT NULL,
  redirect_url TEXT,
  expires_at TIMESTAMP WITH TIME ZONE NOT NULL,
  used_at TIMESTAMP WITH TIME ZONE,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Index for token lookup
CREATE INDEX idx_auth_tokens_token ON auth_tokens(token) WHERE used_at IS NULL;
CREATE INDEX idx_auth_tokens_email ON auth_tokens(email);
```

#### TR2: API Endpoints

| Endpoint | Method | Purpose |
|----------|--------|---------|
| `/api/auth/request` | POST | Request magic link (input: email, redirect) |
| `/api/auth/verify` | GET | Verify token, create session (input: token) |
| `/api/auth/session` | GET | Check current session status |
| `/api/auth/logout` | POST | Clear session cookie |

#### TR3: Component Changes

- Replace all "Enter your email to unlock" overlays with "Sign in to unlock" → triggers magic link flow
- Add session-aware wrapper component for gated content
- Add user menu (minimal): shows email + "Sign out" when authenticated
- Admin users: add small "Admin" badge in nav (optional)

### Migration Path

1. Deploy magic link auth alongside existing email unlock
2. Existing email unlock still works but now also creates a session
3. After validation, remove old email unlock entirely
4. All gated content checks session instead of per-page email state

### Privacy & Compliance

- Email stored for authentication and lead generation (legitimate interest under GDPR)
- Marketing opt-in separate and explicit (not pre-checked)
- User can request data deletion via email to hello@chainsights.one
- Session cookie is functional → no consent banner required
- Update Privacy Policy to mention magic link authentication

---

## Part 3: Backlog — DAO S&P Governance Benchmark

**Status:** Backlog (not ready for implementation)  
**Blocked by:** Benchmark methodology definition

### Context

The DAO detail page currently shows a "DAO Governance Index" section that compares a DAO's score against a sector average (e.g., "Lido 7.4 vs DeFi Average 5.5"). This section has been hidden (see BUG-3) because the benchmark is not yet properly defined.

### What Needs to Be Defined Before Implementation

1. **Category Taxonomy:** Which categories exist? (DeFi, Infrastructure, Public Goods, Social, Gaming, etc.) What are the inclusion criteria per category?

2. **Benchmark Composition:** Which DAOs form the benchmark for each category? All DAOs in the category? Only those above a minimum GVS threshold? Weighted by TVL, token market cap, or equal-weight?

3. **Benchmark Calculation:** Simple average? Median (more robust against outliers)? Weighted average? Rolling window (last N weeks)?

4. **Minimum Sample Size:** How many DAOs must be in a category before a benchmark is meaningful? (Suggestion: minimum 5 DAOs per category)

5. **Update Frequency:** Weekly (aligned with GVS updates)? Daily?

6. **Display Format:** Bar comparison (current)? Percentile rank? Both?

7. **"Excels at" / "Needs work" Logic:** Based on deviation from benchmark? Top/bottom component score? How much deviation = "excels" vs. "needs work"?

### Analogy

Think of this like building the S&P 500 for DAO governance:
- The S&P has clear inclusion criteria (US, market cap, profitability, liquidity)
- It uses a defined weighting method (market-cap weighted)
- It has sector indices (S&P 500 Financials, Tech, etc.)
- We need the same rigor for our governance benchmarks

### Dependency

This item remains in backlog until Mario and Claude define the benchmark methodology together. Once defined, a separate implementation requirement will be created.

### Interim Solution

The "DAO Governance Index" section stays hidden on all detail pages. Rankings and Matrix continue to work without sector benchmarks — they show absolute scores, not relative comparisons.
