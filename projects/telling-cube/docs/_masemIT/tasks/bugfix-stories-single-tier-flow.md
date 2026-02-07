# Bug-Fix Stories: €9 Single Tier Purchase Flow

**Priority:** 🔴 CRITICAL
**Reported:** 2026-02-07
**Reporter:** Mario (manual QA, real purchase)
**Affects:** All new €9 Single tier customers
**Impact:** First-time buyers cannot generate their scenario after purchase — destroys the highest-motivation moment

---

## Bug Summary

The €9 Single tier post-purchase flow has 3 related issues that create a broken first-time experience:

1. **"Generate your first scenario" button → redirects to landing page** (not generation flow)
2. **Dashboard is empty** after login via magic link email (no guidance for Single tier)
3. **Overall flow is fragmented** — user bounces between confirmation page, email, landing page, and dashboard without clear path

---

## Story BF-1: Fix Post-Purchase "Generate" Button Redirect

**Severity:** 🔴 CRITICAL
**Type:** Bug Fix

**As a** new €9 customer who just completed payment,
**I want** the "Generate your first scenario" button to take me directly into the generation flow,
**so that** I can immediately use what I just paid for.

### Current (Broken) Behavior

1. User completes Stripe checkout → arrives at confirmation page
2. Confirmation page shows "Generate your first scenario" button
3. Clicking button → redirects to **landing page** (public homepage)
4. User is not authenticated (no session exists after Stripe redirect)
5. User sees the landing page as if they never purchased anything

### Root Cause (Investigate)

Most likely: The confirmation page button links to `/` or `/generate` without authentication context. After Stripe redirect, no session/JWT is set — the user is technically "anonymous" despite having just paid.

### Expected Behavior

1. User completes Stripe checkout → arrives at confirmation page
2. Confirmation page shows "Generate your first scenario" button
3. Clicking button → triggers magic link email **OR** auto-creates session → opens generation dialog
4. User is authenticated and can generate immediately

### Acceptance Criteria

1. After successful Stripe payment, clicking "Generate your first scenario" does NOT redirect to the public landing page
2. The button either:
   - **(Option A — Preferred)** Auto-logs the user in (session created from Stripe webhook email) and opens the generation dialog directly
   - **(Option B — Fallback)** Sends a magic link to the user's email AND shows clear messaging: "Check your email for your access link — click it to start generating!"
3. If Option A: Session is created server-side during Stripe webhook processing, and the confirmation page URL includes a token or session parameter
4. If Option B: The confirmation page clearly communicates the next step so the user doesn't feel lost
5. The flow works on both desktop and mobile browsers
6. Tested with a real Stripe test-mode purchase end-to-end

### Technical Notes

- Check `/api/stripe/webhook` → `checkout.session.completed` handler: does it create a user session or only send the welcome email?
- Check confirmation page component: where does the "Generate" button link to? Does it pass any auth context?
- Consider: Can we set an httpOnly cookie during the Stripe success redirect? The `success_url` from Stripe could include a one-time token.
- Alternative: The confirmation page could auto-trigger a magic link send and show "Check your email to get started"

### Estimate: 3-5 hours

---

## Story BF-2: Single Tier Dashboard — Empty State with Clear Guidance

**Severity:** ⚠️ HIGH
**Type:** UX Bug / Enhancement

**As a** €9 Single tier customer arriving at my dashboard for the first time,
**I want** to see clear guidance on what to do next (or my generated scenario),
**so that** I don't see an empty page and wonder if something went wrong.

### Current (Broken) Behavior

1. User receives welcome email → clicks magic link → arrives at dashboard
2. Dashboard is completely empty (no scenarios, no guidance)
3. User doesn't know what to do
4. No indication of tier, remaining generations, or next steps

### Expected Behavior

Dashboard shows a contextual empty state for Single tier users:

**Before generation:**
- Tier badge: "Single (1 Generation)"
- Prominent CTA: "Generate Your Scenario" button that opens generation dialog
- Brief text: "You have 1 scenario generation included. Choose your company profile and get started!"

**After generation (scenario complete):**
- The generated scenario card with view/download options
- "0/1 Generations remaining" indicator
- Upgrade CTA: "Need more scenarios? Upgrade to Lifetime — €99" (subtle, not pushy)

**After generation (in progress):**
- Progress indicator: "Your scenario is being generated... ~2 minutes remaining"
- "We'll email you when it's ready"

### Acceptance Criteria

1. Single tier dashboard shows "Generate Your Scenario" CTA when no scenarios exist (not an empty grid)
2. Tier badge is visible: "Single" with "1 Generation" indicator
3. After scenario is generated, the dashboard shows the scenario card with all view/download options
4. After the 1 generation is used, the "Generate" button is replaced with or accompanied by a clear upgrade path
5. Upgrade CTA links to pricing page with Lifetime tier highlighted
6. The empty state is specific to Single tier (other tiers show their normal grid with "Generate" button)

### Design Guidance

```
┌─────────────────────────────────────────────┐
│  Dashboard                    [Single Plan]  │
│                                              │
│  ┌─────────────────────────────────────────┐ │
│  │                                         │ │
│  │   🎲  Ready to generate your scenario?  │ │
│  │                                         │ │
│  │   You have 1 generation included.       │ │
│  │   Pick a company profile and go!        │ │
│  │                                         │ │
│  │        [ Generate Your Scenario ]       │ │
│  │                                         │ │
│  └─────────────────────────────────────────┘ │
│                                              │
│  ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ │
│  Want unlimited scenarios?                   │
│  Upgrade to Lifetime → €99                   │
└─────────────────────────────────────────────┘
```

### Estimate: 2-3 hours

---

## Story BF-3: Streamline Post-Purchase Flow (End-to-End)

**Severity:** ⚠️ HIGH
**Type:** UX Improvement

**As a** new customer (any tier),
**I want** a seamless flow from payment to first scenario generation,
**so that** I never feel lost or wonder "what now?"

### Current (Broken) Flow

```
Payment ✓ → Confirmation Page
              ├─ "Generate" button → Landing Page (BROKEN!)
              └─ Welcome email arrives
                   └─ Magic link → Dashboard (empty)
                        └─ User confused
                              └─ Eventually generates
                                   └─ "Scenario ready" email
                                        └─ Click → View scenario
```

**Problems:** 5 steps, 2 emails, 1 dead end. User bounces between pages.

### Expected (Fixed) Flow

```
Payment ✓ → Confirmation Page (auto-session OR "check email" messaging)
              └─ "Generate Now" → Generation Dialog (authenticated!)
                   └─ Pick scenario → Generation starts
                        └─ Dashboard shows progress
                              └─ Email when ready (backup)
                                   └─ Scenario viewable in dashboard
```

**Target:** 3 steps, 1 email (notification only), zero dead ends.

### Acceptance Criteria

1. The complete flow from Stripe payment success to "scenario generating" takes no more than 3 clicks
2. The user is never redirected to the public landing page after a successful purchase
3. The user is authenticated before or immediately after arriving at the generation step
4. The generation dialog opens automatically (or with one click) after first login post-purchase
5. The "scenario ready" email is a convenience notification, not a required step to access the scenario
6. Flow tested for all 5 tiers: Single (€9), Lifetime (€99), Lifetime+ (€299), Pro (€19/mo), Team (€49/mo)
7. Flow tested on both desktop Chrome and mobile Safari

### Implementation Notes

Consider these approaches (in order of preference):

**Approach 1: Auto-session on Stripe return**
- Stripe `success_url` includes a one-time provisioning token
- Confirmation page validates token → creates session → user is logged in
- "Generate" button opens generation dialog directly
- Pro: Seamless, zero friction
- Con: Security consideration (token in URL)

**Approach 2: Smart confirmation page**
- Confirmation page detects no session → auto-triggers magic link
- Shows: "✓ Payment complete! We've sent you a login link. Check your email to start generating."
- Magic link → Dashboard with auto-open generation dialog (via URL param like `?action=generate`)
- Pro: Secure, uses existing magic link infra
- Con: Extra step (email)

**Approach 3: Combine both**
- Try auto-session first; if fails, fall back to magic link messaging
- Best UX with safe fallback

### Estimate: 4-6 hours (depends on approach chosen)

---

## Story BF-4: Add `?action=generate` Deep Link Support to Dashboard

**Severity:** MEDIUM
**Type:** Enhancement (supports BF-1 and BF-3)

**As a** system routing a new customer to the dashboard,
**I want** to pass a URL parameter that auto-opens the generation dialog,
**so that** first-time users land directly in the generation flow without extra clicks.

### Acceptance Criteria

1. Dashboard page accepts `?action=generate` query parameter
2. When present AND user has remaining generations, the generation dialog opens automatically on page load
3. When present but user has no remaining generations (Single tier used up), show upgrade prompt instead
4. Parameter is consumed on first load (doesn't re-trigger on page refresh via URL cleanup)
5. Works in combination with magic link redirect: `/dashboard?action=generate`

### Technical Notes

- Magic link verification endpoint should support a `redirect` parameter
- Welcome email "Get started" button URL: `/api/auth/verify?token=xxx&redirect=/dashboard?action=generate`
- Confirmation page "Generate" button: same pattern

### Estimate: 1-2 hours

---

## Priority & Sequencing

| Order | Story | Severity | Estimate | Dependency |
|-------|-------|----------|----------|------------|
| 1 | **BF-1** | 🔴 CRITICAL | 3-5h | None — fix first |
| 2 | **BF-4** | MEDIUM | 1-2h | Enables BF-2 and BF-3 |
| 3 | **BF-2** | ⚠️ HIGH | 2-3h | After BF-4 |
| 4 | **BF-3** | ⚠️ HIGH | 4-6h | After BF-1 + BF-4 |

**Total Estimate:** 10-16 hours

**Recommendation:** Fix BF-1 immediately (it's actively losing customers). BF-4 is small and enables the others. Then BF-2 and BF-3 together as a polished flow.

---

## Testing Checklist (All Stories)

- [ ] Real Stripe test-mode purchase: €9 Single tier, end-to-end
- [ ] Real Stripe test-mode purchase: €99 Lifetime tier, end-to-end
- [ ] Desktop Chrome: full flow
- [ ] Mobile Safari: full flow
- [ ] New user (no prior account): full flow
- [ ] Returning user (already has account): purchase → dashboard updated
- [ ] Magic link email: arrives within 60 seconds
- [ ] Generation complete email: arrives and link works
- [ ] Dashboard shows correct tier badge and generation limits
- [ ] Upgrade CTA visible and links to correct pricing page
