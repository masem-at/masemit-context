# Party Mode: Auth & DAO Matrix — Decision Log

**Date:** 2026-01-31
**Attendees:** Mary (Analyst), Paige (PM), Winston (Architect), John (PM), Sally (SM)
**Facilitator:** Mario Semper
**Input Document:** `docs/_masemIT/requirements/chainsights-bugs-auth-backlog.md`

---

## Topics Discussed

### 1. BUG-1: Admin has no access to gated charts (Detail Page)
- **Root Cause:** No session management. `getMatrixAccess()` requires email as query param.
- **Decision:** Do NOT fix separately. Will be resolved by Magic Link Auth (Story 7: Admin Access).
- **Rationale:** Temporary admin-email-hardcoding would be throwaway code. Magic Link Auth uses the same `getMatrixAccess()` function, so the fix is structural.

### 2. BUG-2: Email unlock does not persist across navigation
- **Root Cause:** Email unlock is stateless, per-page. No session cookies.
- **Decision:** Do NOT fix separately. Will be resolved by Magic Link Auth (Story 3: Token Verification + Session).
- **Rationale:** There is no meaningful temporary fix without session management. Magic Link Auth provides 30-day cookie-based sessions.

### 3. BUG-3: GovernanceIndex section visible on detail page
- **Severity:** HIGH — Reputational risk
- **Decision:** Fix immediately. Remove entire GovernanceIndex section from all DAO detail pages.
- **Rationale:**
  - Category averages are not based on a defined benchmark methodology
  - "Excels at" / "Needs work" badges give away paid report insights for free
  - Showing unvalidated numbers contradicts brand promise ("Wallets lie. We don't.")
- **Implementation:** Remove GovernanceIndex JSX + import from `matrix-detail-client.tsx`
- **Status:** Implementing today

### 4. Matrix Random Sort for Anonymous/Free Users
- **Problem:** Current default sort shows lowest-scoring DAOs first (bad first impression)
- **Decision:** Implement server-side seeded shuffle for anonymous and free users
- **Implementation Details (Winston):**
  - Use date-based seed (UTC day) for deterministic daily shuffle
  - Fisher-Yates shuffle with seeded PRNG in Server Component
  - Paid/admin users: keep normal sorted order (highest GVS first)
  - Sort preference for paid users persists via client-side state
- **Status:** Implementing today

### 5. Magic Link Authentication
- **Decision:** Create as separate Epic with 8 stories
- **Resolves:** BUG-1, BUG-2, and provides foundation for subscriptions, saved preferences, API keys
- **Stories (Sally's breakdown):**
  1. DB Schema (users + auth_tokens tables)
  2. Magic Link Generation + Email via Resend
  3. Token Verification + Session (30-day cookie)
  4. Middleware + Role Resolution
  5. UI Updates (Lock Overlay → "Sign in to unlock")
  6. Lead Capture Integration
  7. Admin Access (resolves BUG-1)
  8. Privacy/GDPR Compliance
- **Copy Note (Paige):** Change "Enter your email to unlock" → "Sign in to unlock" with subtext "We'll send you a sign-in link — no password needed."
- **Status:** Next sprint, requires Product Brief → Architecture → Epics & Stories pipeline

### 6. DAO S&P Governance Benchmark
- **Decision:** Remains in backlog, blocked by methodology definition
- **Prerequisite:** Mario and Claude must define benchmark methodology (category taxonomy, composition, calculation method, minimum sample size, update frequency, display format)
- **Interim:** GovernanceIndex section stays hidden (see BUG-3 fix)

---

## Action Items

| # | Item | Owner | When | Status |
|---|------|-------|------|--------|
| 1 | Remove GovernanceIndex from detail pages | Barry (Dev) | Today | In Progress |
| 2 | Implement Matrix random sort for anon/free | Barry (Dev) | Today | In Progress |
| 3 | Create Magic Link Auth Product Brief | John (PM) | Next Sprint | Pending |
| 4 | Create Magic Link Auth Architecture | Winston (Architect) | After Brief | Pending |
| 5 | Create Magic Link Auth Epics & Stories | John (PM) | After Architecture | Pending |
| 6 | Define DAO S&P Benchmark Methodology | Mario + Claude | TBD | Backlog |

---

## Key Quotes

- **Mary:** "The root cause is clear — there is no session management. The email unlock is stateless, per-page. That's not a bug, that's missing infrastructure."
- **Winston:** "Option (a) is throwaway code. Option (b) is the first step of Magic Link Auth. I'd recommend (b)."
- **Paige:** "'Enter your email to unlock more charts' sounds like spam. After Magic Link Auth it should say 'Sign in to unlock' — short, clear, trustworthy."
- **John:** "BUG-3 is a reputation risk. We fix that today. BUG-1 and BUG-2 get solved structurally via Magic Link Auth."
