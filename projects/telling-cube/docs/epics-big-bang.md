---
stepsCompleted: ['step-01-validate-prerequisites', 'step-02-design-epics', 'step-03-create-stories', 'step-04-final-validation']
status: complete
inputDocuments:
  - docs/architecture/auth-dashboard-architecture.md
  - docs/ux/dashboard-ux-spec.md
  - docs/future/big-bang-launch-plan.md
---

# tellingCube - Big Bang Launch Epic Breakdown (Epics 4-7)

## Overview

This document provides the epic and story breakdown for the Big Bang Launch features, decomposing requirements from Winston's Architecture Spec, Sally's UX Spec, and the Big Bang Launch Plan into implementable stories.

**Note:** Epics 1-3 already exist in `docs/epics/`. This document covers Epics 4-7 for the new launch features.

**Summary:** 4 epics, 31 stories, 44 functional requirements covered.

## Requirements Inventory

### Functional Requirements

#### Epic 4: Auth & User Dashboard
| ID | Requirement |
|----|-------------|
| FR4.1 | System shall support magic link authentication via email |
| FR4.2 | Magic links shall expire after 1 hour (single-use) |
| FR4.3 | System shall create JWT sessions stored in httpOnly cookies (7-day expiry) |
| FR4.4 | System shall create users table with entitlements (tier, billing_model, max_years, has_hr_domain, generations_remaining) |
| FR4.5 | System shall create magic_links table with token, expiry, used_at tracking |
| FR4.6 | System shall create user_scenarios junction table |
| FR4.7 | Dashboard shall display welcome banner with user name and tier badge |
| FR4.8 | Dashboard shall display scenario cards (grid: 3 cols desktop, 2 tablet, 1 mobile) |
| FR4.9 | Dashboard shall show View and Download actions for each scenario |
| FR4.10 | Dashboard shall display quick stats (total generated, member since, tier) |
| FR4.11 | System shall provide /login page for returning users |
| FR4.12 | System shall provide /api/auth/request-link, /api/auth/verify, /api/auth/logout, /api/auth/me endpoints |

#### Epic 5: Scheduled Generation (QStash)
| ID | Requirement |
|----|-------------|
| FR5.1 | System shall queue generation jobs via QStash |
| FR5.2 | Scenarios table shall track status: 'queued', 'generating', 'completed', 'failed' |
| FR5.3 | Scenarios table shall track progress (0-100%) |
| FR5.4 | API /api/generate shall validate entitlements before queuing |
| FR5.5 | API /api/jobs/generate shall verify QStash signature |
| FR5.6 | System shall send completion email via Resend when generation finishes |
| FR5.7 | Dashboard shall display active generation banner with progress bar and estimated time |
| FR5.8 | Dashboard shall allow cancelling in-progress generations |
| FR5.9 | API /api/scenarios/:id/status shall return status, progress, estimatedTimeRemaining |

#### Epic 6: Hybrid Pricing & Subscriptions
| ID | Requirement |
|----|-------------|
| FR6.1 | System shall support 3 one-time Stripe products: €9 (Single), €99 (Lifetime), €299 (Lifetime+) |
| FR6.2 | System shall support 2 subscription Stripe products: €19/mo (Pro), €49/mo (Team) |
| FR6.3 | Stripe webhook shall handle checkout.session.completed for one-time purchases |
| FR6.4 | Stripe webhook shall handle subscription.created, subscription.updated, subscription.deleted |
| FR6.5 | Stripe webhook shall handle invoice.payment_failed |
| FR6.6 | Pricing page shall use two-path selection (Individual vs Team/Agency) |
| FR6.7 | Individual path shall show €9, €99, €299 options |
| FR6.8 | Team path shall show €19/mo, €49/mo options |
| FR6.9 | System shall support coupon codes: HOKIHEART, BRIAN10, PHTC2026 |
| FR6.10 | Coupon field shall be visible on pricing page |
| FR6.11 | HOKIHEART coupon shall apply 10% discount + increase charity to 5% |
| FR6.12 | Generation dialog shall show locked options based on user tier |
| FR6.13 | Upgrade prompts shall appear for Single tier users who exhaust their generation |
| FR6.14 | User entitlements shall control: maxYears (1/3/5), hasHrDomain, generationsRemaining |

#### Epic 7: Launch Polish
| ID | Requirement |
|----|-------------|
| FR7.1 | System shall create 4 new curated scenarios to complete 9 total |
| FR7.2 | Landing page shall display 9 curated tiles (replacing 3x3 matrix) |
| FR7.3 | Landing page shall display live donation counter ("€XX donated to children's hospice care") |
| FR7.4 | Footer shall display "3% of every purchase supports hoki.help" |
| FR7.5 | Purchase confirmation email shall show charity calculation (e.g., "3% of your €99 = €2.97") |
| FR7.6 | All 3 existing early supporters shall be migrated to Lifetime+ tier |
| FR7.7 | Generation dialog shall auto-open after successful purchase |
| FR7.8 | Generation dialog shall show Company Size, Industry Sector, Data Duration options |
| FR7.9 | Data Duration option shall show tier-based restrictions with lock icon |

### Non-Functional Requirements

| ID | Requirement |
|----|-------------|
| NFR4.1 | Magic links must use crypto.randomUUID for secure tokens |
| NFR4.2 | JWT must be httpOnly, secure, sameSite: lax |
| NFR4.3 | Rate limit /api/auth/request-link to 5 requests per email per hour |
| NFR4.4 | Desktop-first responsive design (B2B users) |
| NFR5.1 | QStash callbacks must verify signature |
| NFR5.2 | Progress bar must have aria-valuenow for accessibility |
| NFR5.3 | Generation time expectation: 2-3 minutes (shown in UI) |
| NFR6.1 | Stripe webhook must verify signature |
| NFR6.2 | "BEST" badge on Lifetime tier for visual recommendation |

### Additional Requirements

#### From Architecture Spec (Winston)
- Environment variables: JWT_SECRET, QSTASH_TOKEN, QSTASH_SIGNING_KEYS, STRIPE_PRICE_* for all 5 products
- Database indexes on magic_links(token), magic_links(email), users(email), users(stripe_customer_id)
- Migration SQL for existing users to Lifetime+
- Entitlements TypeScript types in lib/auth/entitlements.ts
- Helper functions: canGenerate(), canAccessYears(), canAccessHr()

#### From UX Spec (Sally)
- Tier badge colors: Free=Gray, Single=Blue, Lifetime=Emerald, Lifetime+=Purple, Pro=Teal, Team=Indigo
- Status colors: Generating=Amber, Complete=Green, Failed=Red, Queued=Gray
- Responsive breakpoints: Desktop 1024px+, Tablet 768-1023px, Mobile <768px
- Keyboard navigation for all interactive elements
- WCAG AA color contrast
- Email templates: Magic Link, Generation Complete, Purchase Confirmation

#### New Curated Scenarios to Create
| # | Size | Sector |
|---|------|--------|
| 6 | LargeCap | Consumer Staples |
| 7 | Startup | Financials |
| 8 | MidCap | Financials |
| 9 | LargeCap | Industrials |

### FR Coverage Map

| FR | Epic | Story | Description |
|----|------|-------|-------------|
| FR4.1 | Epic 4 | 4.2 | Magic link authentication |
| FR4.2 | Epic 4 | 4.2, 4.3 | 1-hour magic link expiry |
| FR4.3 | Epic 4 | 4.3 | JWT sessions in httpOnly cookies |
| FR4.4 | Epic 4 | 4.1 | Users table with entitlements |
| FR4.5 | Epic 4 | 4.1 | Magic links table |
| FR4.6 | Epic 4 | 4.1 | User scenarios junction |
| FR4.7 | Epic 4 | 4.6 | Dashboard welcome banner |
| FR4.8 | Epic 4 | 4.7 | Scenario cards grid |
| FR4.9 | Epic 4 | 4.7 | View/Download actions |
| FR4.10 | Epic 4 | 4.6 | Quick stats footer |
| FR4.11 | Epic 4 | 4.5 | /login page |
| FR4.12 | Epic 4 | 4.4, 4.8 | Auth API endpoints |
| FR5.1 | Epic 5 | 5.2 | QStash job queue |
| FR5.2 | Epic 5 | 5.1 | Scenario status tracking |
| FR5.3 | Epic 5 | 5.1 | Progress percentage |
| FR5.4 | Epic 5 | 5.2 | Entitlement validation |
| FR5.5 | Epic 5 | 5.3 | QStash signature verification |
| FR5.6 | Epic 5 | 5.5 | Completion email |
| FR5.7 | Epic 5 | 5.6 | Active generation banner |
| FR5.8 | Epic 5 | 5.7 | Cancel generation |
| FR5.9 | Epic 5 | 5.4 | Status polling endpoint |
| FR6.1 | Epic 6 | 6.1 | 3 one-time Stripe products |
| FR6.2 | Epic 6 | 6.1 | 2 subscription Stripe products |
| FR6.3 | Epic 6 | 6.2 | checkout.session.completed webhook |
| FR6.4 | Epic 6 | 6.3 | Subscription webhooks |
| FR6.5 | Epic 6 | 6.3 | Payment failed webhook |
| FR6.6 | Epic 6 | 6.4 | Two-path pricing page |
| FR6.7 | Epic 6 | 6.5 | Individual pricing options |
| FR6.8 | Epic 6 | 6.6 | Team pricing options |
| FR6.9 | Epic 6 | 6.7 | Coupon codes support |
| FR6.10 | Epic 6 | 6.7 | Coupon input field |
| FR6.11 | Epic 6 | 6.7 | HOKIHEART special coupon |
| FR6.12 | Epic 6 | 6.8 | Tier-locked options in dialog |
| FR6.13 | Epic 6 | 6.9 | Upgrade prompts |
| FR6.14 | Epic 6 | 6.8 | Entitlements control |
| FR7.1 | Epic 7 | 7.1 | 4 new curated scenarios |
| FR7.2 | Epic 7 | 7.2 | 9 tiles on landing page |
| FR7.3 | Epic 7 | 7.3 | Live donation counter |
| FR7.4 | Epic 7 | 7.4 | Charity footer badge |
| FR7.5 | Epic 7 | 7.5 | Charity in purchase email |
| FR7.6 | Epic 7 | 7.6 | Early supporter migration |
| FR7.7 | Epic 7 | 7.7 | Auto-open generation dialog |
| FR7.8 | Epic 6 | 6.8 | Generation dialog options |
| FR7.9 | Epic 6 | 6.8 | Tier restrictions with lock icon |

## Epic List

### Epic 4: Auth & User Dashboard
**Goal:** Users can authenticate via magic link and access a personal dashboard showing their scenarios and account status.

**User Outcome:** After this epic, paid users have accounts, can log in passwordlessly, and see their generated scenarios in a clean dashboard with View/Download actions.

**FRs covered:** FR4.1, FR4.2, FR4.3, FR4.4, FR4.5, FR4.6, FR4.7, FR4.8, FR4.9, FR4.10, FR4.11, FR4.12

**Standalone:** ✅ Complete auth system with dashboard - no future epics required

---

### Epic 5: Scheduled Generation (QStash)
**Goal:** Users can queue scenario generations that run asynchronously, with real-time progress tracking and email notification on completion.

**User Outcome:** After this epic, users submit generation requests that process in the background. Dashboard shows live progress, and users receive email when their scenario is ready.

**FRs covered:** FR5.1, FR5.2, FR5.3, FR5.4, FR5.5, FR5.6, FR5.7, FR5.8, FR5.9

**Depends on:** Epic 4 (user auth for entitlement validation)
**Standalone:** ✅ Complete async generation system - no future epics required

---

### Epic 6: Hybrid Pricing & Subscriptions
**Goal:** Users can purchase one-time tiers (€9/€99/€299) or subscriptions (€19mo/€49mo) through a two-path pricing page with coupon support.

**User Outcome:** After this epic, users see a clear pricing page, choose their path (Individual vs Team), purchase their tier, and have their entitlements automatically set. Coupons work, upgrades are available.

**FRs covered:** FR6.1, FR6.2, FR6.3, FR6.4, FR6.5, FR6.6, FR6.7, FR6.8, FR6.9, FR6.10, FR6.11, FR6.12, FR6.13, FR6.14

**Depends on:** Epic 4 (user model for entitlements)
**Standalone:** ✅ Complete pricing/billing system - no future epics required

---

### Epic 7: Launch Polish
**Goal:** Complete the 9 curated scenarios, add charity counter, migrate early supporters, and polish the generation dialog UX.

**User Outcome:** After this epic, the landing page shows 9 beautiful curated tiles, the charity counter displays donations, early supporters have Lifetime+ access, and the generation dialog guides users smoothly.

**FRs covered:** FR7.1, FR7.2, FR7.3, FR7.4, FR7.5, FR7.6, FR7.7, FR7.8, FR7.9

**Depends on:** Epics 4, 5, 6 (full system in place)
**Standalone:** ✅ Complete launch-ready polish - ships the product

---

## Dependency Graph

```
Epic 4: Auth & Dashboard (Foundation)
    ↓
    ├── Epic 5: Scheduled Generation (uses auth/entitlements)
    ├── Epic 6: Hybrid Pricing (uses user model)
    ↓
Epic 7: Launch Polish (final layer, uses all above)
```

---

## Epic 4: Auth & User Dashboard

### Story 4.1: Database Schema & Entitlements Model

As a **developer**,
I want **the user, magic_links, and user_scenarios tables created with proper indexes and TypeScript types**,
So that **the auth system has a solid data foundation**.

**Acceptance Criteria:**

**Given** the database migration is run
**When** the tables are created
**Then** the `users` table exists with columns: id (UUID), email (unique), tier, billing_model, max_years, has_hr_domain, generations_remaining, stripe_customer_id, stripe_subscription_id, subscription_status, subscription_ends_at, last_login_at, login_count, created_at, updated_at
**And** the `magic_links` table exists with columns: id (UUID), email, token (unique), created_at, expires_at, used_at, redirect_to, created_for
**And** the `user_scenarios` junction table exists with user_id, scenario_id, created_at
**And** indexes exist on magic_links(token), magic_links(email), users(email), users(stripe_customer_id)
**And** TypeScript types are defined in `lib/auth/entitlements.ts` for Tier, UserEntitlements, TIER_ENTITLEMENTS
**And** helper functions canGenerate(), canAccessYears(), canAccessHr() are implemented

---

### Story 4.2: Magic Link Request API

As a **user**,
I want **to request a magic link by entering my email**,
So that **I can access my account without remembering a password**.

**Acceptance Criteria:**

**Given** a user submits their email to POST /api/auth/request-link
**When** the email is valid
**Then** a magic link token is generated using crypto.randomUUID
**And** the token is stored in magic_links table with expires_at = NOW + 1 hour
**And** an email is sent via Resend with the link containing the token
**And** the email clearly states "This link expires in 1 hour"
**And** a 200 response is returned with { success: true }

**Given** a user submits their email to POST /api/auth/request-link
**When** the email has exceeded 5 requests in the last hour
**Then** a 429 response is returned with rate limit error
**And** no new magic link is created

---

### Story 4.3: Magic Link Verification & JWT Session

As a **user**,
I want **to click the magic link in my email and be logged in**,
So that **I can access my dashboard securely**.

**Acceptance Criteria:**

**Given** a user clicks a magic link with a valid token
**When** GET /api/auth/verify?token={token} is called
**Then** the token is validated (exists, not expired, not used)
**And** the token is marked as used (used_at = NOW)
**And** a JWT is created with payload { userId, email, tier, exp }
**And** the JWT is set as an httpOnly cookie (secure, sameSite: lax, 7-day expiry)
**And** the user is redirected to the redirect_to URL (default: /dashboard)

**Given** a user clicks an expired or used magic link
**When** GET /api/auth/verify is called
**Then** a 401 response is returned
**And** the user is redirected to /login with error message

---

### Story 4.4: Session Management APIs

As a **user**,
I want **to check my session status and log out**,
So that **I can manage my authentication state**.

**Acceptance Criteria:**

**Given** a logged-in user calls GET /api/auth/me
**When** the JWT cookie is valid
**Then** the response contains { user: { id, email, tier, entitlements } }
**And** if the JWT has < 1 day remaining, a refreshed JWT cookie is set

**Given** an unauthenticated user calls GET /api/auth/me
**When** no valid JWT cookie exists
**Then** a 401 response is returned

**Given** a logged-in user calls POST /api/auth/logout
**When** the request is processed
**Then** the JWT cookie is cleared
**And** a 200 response is returned

---

### Story 4.5: Login Page UI

As a **returning user**,
I want **a simple login page where I can enter my email**,
So that **I can request access to my dashboard**.

**Acceptance Criteria:**

**Given** a user visits /login
**When** the page loads
**Then** a form is displayed with an email input field and "Send Magic Link" button
**And** the tellingCube branding is visible
**And** the page is desktop-first responsive

**Given** a user enters their email and submits the form
**When** the form is submitted
**Then** the /api/auth/request-link endpoint is called
**And** a success message is displayed: "Check your email for a login link"
**And** the email input is disabled to prevent resubmission

**Given** a user enters an invalid email
**When** the form is submitted
**Then** a validation error is displayed
**And** no API call is made

---

### Story 4.6: Dashboard Layout & Welcome Banner

As a **logged-in user**,
I want **to see a personalized dashboard with my account information**,
So that **I feel welcomed and know my account status**.

**Acceptance Criteria:**

**Given** a logged-in user visits /dashboard
**When** the page loads
**Then** the welcome banner displays "Welcome back, {name}!"
**And** the tier badge is displayed (e.g., "Lifetime Member") with correct color
**And** the quick stat shows total scenarios generated
**And** the quick stats footer shows: Total Generated, Total Downloads, Member Since, Current Tier

**Given** an unauthenticated user visits /dashboard
**When** the page loads
**Then** the user is redirected to /login

**Given** the dashboard is viewed on different screen sizes
**When** desktop (1024px+)
**Then** full layout is displayed with 3-column grid
**When** tablet (768-1023px)
**Then** 2-column grid with collapsed stats
**When** mobile (<768px)
**Then** 1-column stacked layout with hamburger menu

---

### Story 4.7: Scenario Cards with View/Download

As a **logged-in user**,
I want **to see my generated scenarios as cards with View and Download actions**,
So that **I can easily access my data**.

**Acceptance Criteria:**

**Given** a user has generated scenarios
**When** the dashboard loads
**Then** scenario cards are displayed in a responsive grid
**And** each card shows: sector icon, company name, size badge, generation date
**And** each card has [View] and [Download] buttons
**And** cards are sorted by creation date (newest first)

**Given** a user clicks [View] on a scenario card
**When** the button is clicked
**Then** the user is navigated to the scenario detail page

**Given** a user clicks [Download] on a scenario card
**When** the button is clicked
**Then** the CSV/JSON export is downloaded

**Given** a user has no scenarios
**When** the dashboard loads
**Then** an empty state is displayed with "Generate your first scenario" CTA

---

### Story 4.8: Dashboard Data API

As a **frontend**,
I want **API endpoints to fetch dashboard data**,
So that **the dashboard can display user scenarios and stats**.

**Acceptance Criteria:**

**Given** a logged-in user calls GET /api/dashboard
**When** the request is authenticated
**Then** the response contains { stats: { totalGenerated, totalDownloads, memberSince, tier }, recentScenarios: [...] }

**Given** a logged-in user calls GET /api/dashboard/scenarios
**When** the request includes pagination params
**Then** the response contains paginated scenario list with { data: [...], total, page, limit }

**Given** an unauthenticated user calls dashboard endpoints
**When** no valid session exists
**Then** a 401 response is returned

---

## Epic 5: Scheduled Generation (QStash)

### Story 5.1: Scenarios Table Status Extensions

As a **developer**,
I want **the scenarios table extended with status tracking columns**,
So that **we can track generation progress and state**.

**Acceptance Criteria:**

**Given** the database migration is run
**When** the scenarios table is altered
**Then** a `status` column exists with values: 'queued', 'generating', 'completed', 'failed' (default: 'completed')
**And** a `progress` column exists as INTEGER (0-100, default: 100)
**And** a `queued_at` TIMESTAMPTZ column exists
**And** a `started_at` TIMESTAMPTZ column exists
**And** a `completed_at` TIMESTAMPTZ column exists
**And** an `error_message` TEXT column exists
**And** existing scenarios retain status='completed' and progress=100

---

### Story 5.2: Generation Queue API

As a **paid user**,
I want **to submit a generation request that gets queued for processing**,
So that **I can generate scenarios without waiting synchronously**.

**Acceptance Criteria:**

**Given** a logged-in user with valid entitlements calls POST /api/generate
**When** the request contains { size, sector, years }
**Then** the user's entitlements are validated (canGenerate, canAccessYears)
**And** a new scenario record is created with status='queued', queued_at=NOW
**And** the scenario is linked to the user via user_scenarios table
**And** a job is published to QStash with payload { scenarioId, userId, size, sector, years }
**And** the response returns { scenarioId, status: 'queued' }

**Given** a user without generation entitlements calls POST /api/generate
**When** canGenerate() returns false
**Then** a 403 response is returned with "No generations remaining"

**Given** a user requests years beyond their tier limit
**When** canAccessYears() returns false
**Then** a 403 response is returned with "Upgrade required for {years}-year data"

**Given** a Single tier user generates their 1 scenario
**When** the generation is queued
**Then** generations_remaining is decremented to 0

---

### Story 5.3: QStash Job Handler

As a **system**,
I want **a secure endpoint that processes generation jobs from QStash**,
So that **scenarios are generated asynchronously with proper verification**.

**Acceptance Criteria:**

**Given** QStash calls POST /api/jobs/generate with a valid signature
**When** the signature is verified using QSTASH_CURRENT_SIGNING_KEY or QSTASH_NEXT_SIGNING_KEY
**Then** the scenario status is updated to 'generating', started_at=NOW
**And** the existing generation logic is executed
**And** progress is updated periodically (0%, 25%, 50%, 75%, 100%)
**And** on success, status is set to 'completed', completed_at=NOW, progress=100
**And** on failure, status is set to 'failed', error_message is populated

**Given** QStash calls POST /api/jobs/generate with an invalid signature
**When** signature verification fails
**Then** a 401 response is returned
**And** no generation is started

**Given** the generation job fails
**When** QStash retries (built-in retry policy)
**Then** the job handler processes the retry correctly

---

### Story 5.4: Generation Status Polling API

As a **frontend**,
I want **an API to poll generation status**,
So that **I can show real-time progress to users**.

**Acceptance Criteria:**

**Given** a logged-in user calls GET /api/scenarios/:id/status
**When** the scenario belongs to the user
**Then** the response contains { status, progress, estimatedTimeRemaining }
**And** estimatedTimeRemaining is calculated based on progress and average generation time

**Given** the scenario status is 'queued'
**When** the endpoint is called
**Then** estimatedTimeRemaining reflects queue position estimate

**Given** the scenario status is 'generating'
**When** the endpoint is called
**Then** progress shows current percentage (0-100)
**And** estimatedTimeRemaining updates based on progress rate

**Given** a user tries to access another user's scenario status
**When** the scenario doesn't belong to them
**Then** a 404 response is returned

---

### Story 5.5: Generation Completion Email

As a **user**,
I want **to receive an email when my scenario generation is complete**,
So that **I know when to access my data without watching the screen**.

**Acceptance Criteria:**

**Given** a scenario generation completes successfully
**When** status changes to 'completed'
**Then** an email is sent via Resend to the user's email
**And** the subject is "Your tellingCube scenario is ready!"
**And** the body includes: scenario type (size x sector), years of data, event count
**And** the body includes a "View My Scenario" button linking to the scenario
**And** the body mentions hoki.help charity

**Given** a scenario generation fails
**When** status changes to 'failed'
**Then** an email is sent with subject "Generation issue - we're on it"
**And** the body apologizes and suggests trying again or contacting support

---

### Story 5.6: Active Generation Banner UI

As a **user viewing my dashboard**,
I want **to see a prominent banner when a generation is in progress**,
So that **I know the status without needing to check my email**.

**Acceptance Criteria:**

**Given** a user has a scenario with status='queued' or status='generating'
**When** the dashboard loads
**Then** an active generation banner is displayed prominently
**And** the banner shows: scenario type (e.g., "MidCap x Industrials")
**And** a progress bar is displayed with current percentage
**And** estimated time remaining is shown (e.g., "~1:30 remaining")
**And** the progress bar has aria-valuenow for accessibility

**Given** no active generations exist
**When** the dashboard loads
**Then** the active generation banner is not displayed

**Given** the generation completes while user is on dashboard
**When** status changes to 'completed'
**Then** the banner updates to show "Complete!" with a [View] button
**And** the banner auto-dismisses after 5 seconds or on click

---

### Story 5.7: Cancel Generation Feature

As a **user**,
I want **to cancel an in-progress generation**,
So that **I can stop a generation I no longer need**.

**Acceptance Criteria:**

**Given** a user has an active generation (status='queued' or 'generating')
**When** the active generation banner is displayed
**Then** a [Cancel] button is visible

**Given** a user clicks [Cancel] on an active generation
**When** the cancellation is confirmed
**Then** POST /api/scenarios/:id/cancel is called
**And** the scenario status is set to 'failed' with error_message='Cancelled by user'
**And** no completion email is sent
**And** the banner is removed from dashboard

**Given** a user tries to cancel a completed scenario
**When** the cancel endpoint is called
**Then** a 400 response is returned with "Cannot cancel completed scenario"

---

## Epic 6: Hybrid Pricing & Subscriptions

### Story 6.1: Stripe Products Configuration

As a **developer**,
I want **all Stripe products configured with proper metadata**,
So that **the payment system can identify tiers correctly**.

**Acceptance Criteria:**

**Given** Stripe dashboard is accessed
**When** products are configured
**Then** 3 one-time products exist: Single (€9), Lifetime (€99), Lifetime+ (€299)
**And** 2 subscription products exist: Pro (€19/mo), Team (€49/mo)
**And** each product has metadata.tier set to the tier name ('single', 'lifetime', 'lifetime_plus', 'pro', 'team')
**And** environment variables are configured: STRIPE_PRICE_SINGLE, STRIPE_PRICE_LIFETIME, STRIPE_PRICE_LIFETIME_PLUS, STRIPE_PRICE_PRO_MONTHLY, STRIPE_PRICE_TEAM_MONTHLY

**Given** the application starts
**When** environment variables are loaded
**Then** all 5 STRIPE_PRICE_* variables are available
**And** the application can create checkout sessions for any tier

---

### Story 6.2: One-Time Purchase Webhook Handler

As a **system**,
I want **to handle Stripe checkout.session.completed events for one-time purchases**,
So that **users get their entitlements after purchase**.

**Acceptance Criteria:**

**Given** a user completes a one-time purchase (€9, €99, or €299)
**When** Stripe sends checkout.session.completed webhook
**Then** the webhook signature is verified using STRIPE_WEBHOOK_SECRET
**And** the user is created/updated with the correct tier from session.metadata.tier
**And** billing_model is set to 'one_time'
**And** entitlements are set based on TIER_ENTITLEMENTS (maxYears, hasHrDomain, generationsRemaining)
**And** stripe_customer_id is stored
**And** a magic link email is sent to grant dashboard access

**Given** a Single tier purchase (€9)
**When** the webhook is processed
**Then** generations_remaining is set to 1
**And** max_years is set to 1

**Given** a Lifetime tier purchase (€99)
**When** the webhook is processed
**Then** generations_remaining is set to NULL (unlimited)
**And** max_years is set to 3

**Given** a Lifetime+ tier purchase (€299)
**When** the webhook is processed
**Then** generations_remaining is set to NULL (unlimited)
**And** max_years is set to 5
**And** has_hr_domain is set to true

---

### Story 6.3: Subscription Webhook Handlers

As a **system**,
I want **to handle Stripe subscription lifecycle events**,
So that **subscription users have correct entitlements throughout their subscription**.

**Acceptance Criteria:**

**Given** a user starts a subscription (Pro or Team)
**When** Stripe sends customer.subscription.created
**Then** the user is updated with tier from subscription.metadata.tier
**And** billing_model is set to 'subscription'
**And** stripe_subscription_id is stored
**And** subscription_status is set to 'active'
**And** subscription_ends_at is set to current_period_end

**Given** a subscription is updated (plan change, renewal)
**When** Stripe sends customer.subscription.updated
**Then** the user's tier, subscription_status, and subscription_ends_at are updated

**Given** a subscription is cancelled
**When** Stripe sends customer.subscription.deleted
**Then** the user's tier is set to 'free'
**And** subscription_status is set to 'canceled'
**And** entitlements revert to free tier

**Given** a subscription payment fails
**When** Stripe sends invoice.payment_failed
**Then** subscription_status is set to 'past_due'
**And** an email is sent to the user about the payment issue

---

### Story 6.4: Pricing Page Two-Path Selection

As a **visitor**,
I want **to choose between Individual and Team/Agency paths on the pricing page**,
So that **I only see pricing options relevant to my needs**.

**Acceptance Criteria:**

**Given** a user visits /pricing
**When** the page loads
**Then** two large cards are displayed: "Individual" and "Team/Agency"
**And** Individual card shows: "One-time purchase", "Pay once, use forever", "No subscriptions", "Founding member", "Lifetime access"
**And** Team/Agency card shows: "Monthly subscription", "For ongoing data needs", "Multiple seats", "Cancel anytime", "Team management"
**And** footer displays "3% of every purchase supports hoki.help - Children's hospice care in Austria"

**Given** a user clicks "Individual"
**When** the selection is made
**Then** the Individual pricing options (Step 2a) are displayed
**And** a "← Back" button is available

**Given** a user clicks "Team/Agency"
**When** the selection is made
**Then** the Team pricing options (Step 2b) are displayed
**And** a "← Back" button is available

---

### Story 6.5: Individual Pricing Options

As a **visitor who selected Individual**,
I want **to see and purchase one-time tier options**,
So that **I can become a founding member with lifetime access**.

**Acceptance Criteria:**

**Given** a user is on the Individual pricing view
**When** the page renders
**Then** three pricing cards are displayed: Single (€9), Lifetime (€99), Lifetime+ (€299)
**And** the Lifetime card has a "BEST" badge
**And** each card shows: price, one-time label, generations allowed, data years, HR domain access

**Given** the Single (€9) card
**When** displayed
**Then** it shows: "1 scenario", "1 year data", no HR domain

**Given** the Lifetime (€99) card
**When** displayed
**Then** it shows: "Unlimited", "1-3 years", no HR domain, "BEST" badge

**Given** the Lifetime+ (€299) card
**When** displayed
**Then** it shows: "Unlimited", "1-5 years", "+ HR Domain"

**Given** a user clicks [Buy Now] on any card
**When** the button is clicked
**Then** a Stripe Checkout session is created with the correct price ID
**And** metadata.tier is set to the selected tier
**And** the user is redirected to Stripe Checkout

---

### Story 6.6: Team/Agency Pricing Options

As a **visitor who selected Team/Agency**,
I want **to see and purchase subscription options**,
So that **my team can have ongoing access to data generation**.

**Acceptance Criteria:**

**Given** a user is on the Team pricing view
**When** the page renders
**Then** two pricing cards are displayed: Pro (€19/mo), Team (€49/mo)
**And** header shows "PRO SUBSCRIPTIONS"

**Given** the Pro (€19/mo) card
**When** displayed
**Then** it shows: "1 seat", "Unlimited", "1-5 years", "+ HR Domain"

**Given** the Team (€49/mo) card
**When** displayed
**Then** it shows: "5 seats", "Unlimited", "1-5 years", "+ HR Domain", "Team billing"

**Given** a user clicks [Subscribe] on either card
**When** the button is clicked
**Then** a Stripe Checkout session is created in subscription mode
**And** metadata.tier is set to the selected tier
**And** the user is redirected to Stripe Checkout

**Given** the footer area
**When** displayed
**Then** it shows "Need more seats? Contact us for enterprise pricing"

---

### Story 6.7: Coupon Code Support

As a **buyer**,
I want **to apply a discount coupon during checkout**,
So that **I can get promotional pricing**.

**Acceptance Criteria:**

**Given** a user is on any pricing view (Individual or Team)
**When** the page renders
**Then** a "Have a coupon? [Enter code]" input field is visible

**Given** a user enters coupon code "BRIAN10"
**When** the checkout session is created
**Then** the Stripe coupon is applied for 10% discount

**Given** a user enters coupon code "PHTC2026"
**When** the checkout session is created
**Then** the Stripe coupon is applied for 10% discount

**Given** a user enters coupon code "HOKIHEART"
**When** the checkout session is created
**Then** the Stripe coupon is applied for 10% discount
**And** internal tracking flags this purchase for 5% charity (instead of 3%)

**Given** a user enters an invalid coupon code
**When** validation occurs
**Then** an error message is displayed: "Invalid coupon code"
**And** the checkout does not proceed until corrected or cleared

---

### Story 6.8: Generation Dialog Tier Restrictions

As a **logged-in user**,
I want **the generation dialog to show options based on my tier**,
So that **I understand what I can generate and what requires an upgrade**.

**Acceptance Criteria:**

**Given** a user opens the generation dialog
**When** the dialog renders
**Then** Company Size options are displayed (Startup, MidCap, LargeCap) - all available
**And** Industry Sector options are displayed (Consumer Staples, Industrials, Financials) - all available
**And** Data Duration options show tier restrictions

**Given** a Single tier user (max_years=1)
**When** Data Duration options render
**Then** "1 Year" is selectable
**And** "3 Years" shows lock icon with "Requires Lifetime or higher"
**And** "5 Years" shows lock icon with "Requires Lifetime+ or Pro"

**Given** a Lifetime tier user (max_years=3)
**When** Data Duration options render
**Then** "1 Year" and "3 Years" are selectable
**And** "5 Years" shows lock icon with "Requires Lifetime+ or Pro"

**Given** a Lifetime+ or Pro/Team user (max_years=5)
**When** Data Duration options render
**Then** all duration options are selectable

**Given** a user clicks a locked option
**When** the click occurs
**Then** an [Upgrade] button appears inline
**And** clicking [Upgrade] navigates to /pricing

---

### Story 6.9: Upgrade Prompts & CTAs

As a **user with limited entitlements**,
I want **to see upgrade prompts when I hit limitations**,
So that **I know how to unlock more features**.

**Acceptance Criteria:**

**Given** a Single tier user who has used their 1 generation
**When** they visit the dashboard
**Then** a banner displays: "You've used your 1 scenario. Upgrade to Lifetime for unlimited!"
**And** an [Upgrade - €99] button is prominently displayed

**Given** a Single tier user clicks [Generate New]
**When** generations_remaining = 0
**Then** the generation dialog shows "No generations remaining"
**And** displays upgrade options: Lifetime (€99) or Pro (€19/mo)

**Given** a free user views an example scenario
**When** the example view renders
**Then** a CTA banner displays: "Want your own custom data? Generate unlimited scenarios starting at €9"
**And** a [See Pricing] button navigates to /pricing

**Given** any upgrade prompt
**When** the user clicks the upgrade button
**Then** they are navigated to /pricing with the relevant path pre-selected if applicable

---

## Epic 7: Launch Polish

### Story 7.1: Generate 4 New Curated Scenarios

As a **product owner**,
I want **4 new curated scenarios generated to complete the 9-scenario grid**,
So that **free users can explore a full variety of business sizes and sectors**.

**Acceptance Criteria:**

**Given** the existing 5 curated scenarios
**When** the 4 new scenarios are generated
**Then** scenario #6 is created: LargeCap × Consumer Staples
**And** scenario #7 is created: Startup × Financials
**And** scenario #8 is created: MidCap × Financials
**And** scenario #9 is created: LargeCap × Industrials

**Given** each new scenario
**When** generated
**Then** it has a unique, realistic company name
**And** it has 1 year of data (matching free tier)
**And** it is exported to static JSON in the curated examples folder
**And** it follows the same structure as existing curated scenarios

**Given** the complete 9-scenario set
**When** reviewed
**Then** all 9 combinations of 3 sizes × 3 sectors are covered

---

### Story 7.2: Landing Page 9 Curated Tiles

As a **free visitor**,
I want **to see 9 curated scenario tiles on the landing page**,
So that **I can explore real examples before deciding to purchase**.

**Acceptance Criteria:**

**Given** a visitor lands on the homepage
**When** the page loads
**Then** the old 3×3 matrix selector is replaced with 9 curated scenario tiles
**And** tiles are arranged in a responsive grid (3 cols desktop, 2 tablet, 1 mobile)
**And** each tile shows: company name, sector icon, size badge

**Given** each curated tile
**When** displayed
**Then** it has a hover effect indicating clickability
**And** clicking the tile navigates to the example detail view

**Given** the tile grid
**When** rendered
**Then** tiles are visually organized by size or sector (designer choice)
**And** a subtle label indicates these are "Example Scenarios"
**And** a CTA appears below: "Generate Your Own - Starting at €9"

**Given** mobile viewport
**When** the page loads
**Then** tiles stack vertically in a scrollable list
**And** each tile is full-width and easily tappable

---

### Story 7.3: Live Donation Counter

As a **visitor**,
I want **to see how much has been donated to charity**,
So that **I feel good about supporting a product that gives back**.

**Acceptance Criteria:**

**Given** a visitor views the landing page
**When** the page loads
**Then** a donation counter is prominently displayed
**And** the counter shows "€XX donated to children's hospice care"
**And** the counter links to or mentions hoki.help

**Given** the donation counter data
**When** queried
**Then** the total is calculated as 3% of all revenue (one-time + subscription)
**And** HOKIHEART purchases contribute 5% instead of 3%
**And** the total is stored/cached for efficient display

**Given** a new purchase is completed
**When** the webhook processes the payment
**Then** the donation total is updated
**And** the counter reflects the new total on next page load

**Given** the counter display
**When** rendered
**Then** it uses a subtle, tasteful design (not flashy)
**And** the amount is formatted with € symbol and appropriate decimals (e.g., "€127.50")

---

### Story 7.4: Charity Footer Badge

As a **visitor on any page**,
I want **to see that 3% supports charity in the footer**,
So that **the social impact is consistently communicated**.

**Acceptance Criteria:**

**Given** any page on the tellingCube website
**When** the footer renders
**Then** a badge/text displays "3% of every purchase supports hoki.help"
**And** "hoki.help" is a clickable link to https://hoki.help

**Given** the footer badge
**When** styled
**Then** it matches the existing footer design
**And** it's visible but not overly prominent
**And** it includes a small heart or charity icon (optional)

**Given** the pricing page specifically
**When** rendered
**Then** the charity message appears both in the two-path selector and the footer
**And** messaging is consistent across locations

---

### Story 7.5: Purchase Confirmation Email with Charity

As a **new customer**,
I want **my purchase confirmation email to show the charity contribution**,
So that **I feel the positive impact of my purchase immediately**.

**Acceptance Criteria:**

**Given** a user completes a purchase
**When** the magic link email is sent
**Then** the email template is the "Purchase Confirmation" variant (not just login)
**And** the subject is "Welcome to tellingCube! ({Tier} Member)"
**And** the body welcomes the user by name

**Given** the purchase confirmation email body
**When** rendered
**Then** it lists what the user gets with their tier (generations, years, HR domain)
**And** it shows the charity calculation: "3% of your €{amount} (€{charity_amount}) goes to hoki.help"
**And** for HOKIHEART purchases: "5% of your €{amount} (€{charity_amount}) goes to hoki.help - thank you for your extra generosity!"
**And** it thanks the user for supporting children's hospice care in Austria

**Given** specific purchase amounts
**When** calculated
**Then** €9 Single → €0.27 charity (3%)
**And** €99 Lifetime → €2.97 charity (3%)
**And** €299 Lifetime+ → €8.97 charity (3%)
**And** €99 with HOKIHEART → €4.95 charity (5%)

---

### Story 7.6: Early Supporter Migration

As an **early supporter (existing customer)**,
I want **to be automatically upgraded to Lifetime+ tier**,
So that **I get full benefits as a thank-you for early support**.

**Acceptance Criteria:**

**Given** the 3 existing customers who purchased before the Big Bang Launch
**When** the migration script runs
**Then** each customer is identified by their stripe_customer_id from invoices table
**And** a user record is created/updated for each with:
  - tier = 'lifetime_plus'
  - billing_model = 'one_time'
  - max_years = 5
  - has_hr_domain = true
  - generations_remaining = NULL (unlimited)

**Given** the migration is complete
**When** an early supporter logs in
**Then** they see "Lifetime+ Member" badge
**And** they have access to 5-year data and HR domain
**And** they have unlimited generations

**Given** the migration script
**When** executed
**Then** it is idempotent (can run multiple times safely)
**And** it logs which users were migrated
**And** it does not affect users who purchased after launch

---

### Story 7.7: Post-Purchase Generation Dialog Auto-Open

As a **new customer**,
I want **the generation dialog to automatically open after I click the magic link**,
So that **I can immediately generate my first scenario without extra clicks**.

**Acceptance Criteria:**

**Given** a user clicks the magic link from a purchase confirmation email
**When** they land on /dashboard
**Then** the generation dialog automatically opens
**And** a welcome message displays: "Welcome! Let's generate your first scenario"

**Given** a user clicks a magic link from a regular login email
**When** they land on /dashboard
**Then** the generation dialog does NOT auto-open
**And** the dashboard displays normally

**Given** the magic link redirect
**When** created_for = 'purchase_complete'
**Then** a query parameter ?welcome=true is appended to the redirect URL
**And** the dashboard checks for this parameter to trigger auto-open

**Given** the auto-open dialog
**When** displayed
**Then** all generation options are available based on user's tier
**And** the user can close the dialog if they want to explore first
**And** the ?welcome parameter is cleared from URL after dialog opens

---

## Summary

| Epic | Stories | FRs Covered |
|------|---------|-------------|
| Epic 4: Auth & Dashboard | 8 stories | 12 FRs |
| Epic 5: Scheduled Generation | 7 stories | 9 FRs |
| Epic 6: Hybrid Pricing | 9 stories | 14 FRs |
| Epic 7: Launch Polish | 7 stories | 9 FRs |
| **TOTAL** | **31 stories** | **44 FRs** |

---

*Document created by John (PM) via create-epics-and-stories workflow 2026-01-19*
