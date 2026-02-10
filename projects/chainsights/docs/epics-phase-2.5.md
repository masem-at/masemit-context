---
stepsCompleted: [1, 2, 3, 4]
status: 'ready-for-development'
inputDocuments:
  - 'docs/prd-phase-2.5.md'
  - 'docs/architecture.md'
  - 'docs/_masemIT/requirements/ticket-dgi-benchmark-in-reports.md'
  - 'docs/_masemIT/requirements/ticket-pricing-page.md'
  - 'docs/_masemIT/requirements/ticket-account-dashboard.md'
workflowType: 'epics-stories'
lastStep: 1
project_name: 'chainsights'
phase: '2.5'
user_name: 'Mario'
date: '2026-02-04'
---

# chainsights Phase 2.5 - Epic Breakdown

> **⚠️ PRICING OUTDATED (as of 2026-02-05)**
> This document contains stale prices. Current pricing:
> - Free Check: €0 | Deep Dive: **€49** | Governance Audit: **€149**
> - DAO Matrix: **FREE** (paywall removed 2026-02-05)
> - ❌ €19/mo Matrix and €99/yr Matrix subscriptions are deprecated
> See `docs/project_context.md` for authoritative pricing.

## Overview

This document provides the complete epic and story breakdown for chainsights Phase 2.5, decomposing the requirements from the PRD, Architecture, and detailed tickets into implementable stories.

## Requirements Inventory

### Functional Requirements

**Epic 1: DGI Benchmark in Reports**

| ID | Requirement |
|----|-------------|
| FR1.1 | `getCurrentDGI()` must return HPR, DEI, PDI, GPI ecosystem averages |
| FR1.2 | `dgiBenchmark.componentBenchmarks` contains ecosystemAvg, daoScore, percentile, status per dimension |
| FR1.3 | Percentile calculation per dimension via `PERCENT_RANK()` |
| FR1.4 | Status logic: `above` (>110%), `below` (<90%), `average` (±10%) |
| FR1.5 | Deep Dive reports show headline benchmark (GVS vs Composite + Percentile) |
| FR1.6 | Governance Audit reports show full "How You Compare" section |
| FR1.7 | Strengths sorted by percentile DESC, improvement areas by percentile ASC |
| FR1.8 | Section gracefully hidden if benchmark data unavailable |

**Epic 2: Pricing Page**

| ID | Requirement |
|----|-------------|
| FR2.1 | `/pricing` route exists and is publicly accessible |
| FR2.2 | All current products displayed with correct prices (Free, €49, €149, €19/mo, €99/yr) |
| FR2.3 | CTAs link to correct checkout/order flows |
| FR2.4 | Navigation includes "Pricing" link |
| FR2.5 | FAQ section with expandable items |
| FR2.6 | TierCards updated with benchmark features + `[NEW]` badge |

**Epic 3: Account Dashboard**

| ID | Requirement |
|----|-------------|
| FR3.1 | `/account` route exists, protected by Magic Link auth |
| FR3.2 | Subscription status displayed (plan, price, next billing, status) |
| FR3.3 | "Manage Subscription" button opens Stripe Customer Portal |
| FR3.4 | Report history with download links |
| FR3.5 | Correct states shown: active sub, no sub, cancelled sub |
| FR3.6 | Navigation shows Account/Logout when logged in, Login when not |

### Non-Functional Requirements

| ID | Requirement | Affected Features |
|----|-------------|-------------------|
| NFR1 | Responsive design — desktop, tablet, mobile | All |
| NFR2 | Analytics tracking — `pricing_view`, `account_view`, `billing_portal_click` events | F2, F3 |
| NFR3 | SEO meta tags for `/pricing` (title, description, canonical) | F2 |
| NFR4 | Error handling — graceful fallback if Stripe Portal unavailable | F3 |
| NFR5 | Performance — PDF downloads work for historical purchases | F3 |
| NFR6 | PDF rendering — "How You Compare" section renders correctly | F1 |

### Additional Requirements

| Requirement | Source |
|-------------|--------|
| Reuse existing `TierCards` component from `/check` flow | Ticket-2 |
| Stripe Customer Portal via `billingPortal.sessions.create()` | Ticket-3 |
| Auth middleware protection for `/account` route | Architecture |
| TypeScript strict mode for new files | project_context.md |
| Drizzle ORM for all DB queries | project_context.md |

### FR Coverage Map

| FR | Epic | Description |
|----|------|-------------|
| FR1.1 | Epic 1 | `getCurrentDGI()` returns HPR, DEI, PDI, GPI ecosystem averages |
| FR1.2 | Epic 1 | `dgiBenchmark.componentBenchmarks` structure with ecosystemAvg, daoScore, percentile, status |
| FR1.3 | Epic 1 | Percentile calculation per dimension via `PERCENT_RANK()` |
| FR1.4 | Epic 1 | Status logic: above (>110%), below (<90%), average (±10%) |
| FR1.5 | Epic 1 | Deep Dive headline benchmark (GVS vs Composite + Percentile) |
| FR1.6 | Epic 1 | Governance Audit full "How You Compare" section |
| FR1.7 | Epic 1 | Strengths/improvements sorted by percentile |
| FR1.8 | Epic 1 | Graceful hiding if benchmark data unavailable |
| FR2.1 | Epic 2 | `/pricing` route publicly accessible |
| FR2.2 | Epic 2 | All products displayed with correct prices |
| FR2.3 | Epic 2 | CTAs link to correct checkout flows |
| FR2.4 | Epic 2 | Navigation includes "Pricing" link |
| FR2.5 | Epic 2 | FAQ section with expandable items |
| FR2.6 | Epic 2 | TierCards updated with benchmark features + [NEW] badge |
| FR3.1 | Epic 3 | `/account` route protected by Magic Link auth |
| FR3.2 | Epic 3 | Subscription status displayed |
| FR3.3 | Epic 3 | "Manage Subscription" opens Stripe Customer Portal |
| FR3.4 | Epic 3 | Report history with download links |
| FR3.5 | Epic 3 | Correct states for active/no/cancelled subscription |
| FR3.6 | Epic 3 | Navigation shows Account/Logout when logged in |
| NFR1 | All | Responsive design (desktop, tablet, mobile) |
| NFR2 | Epic 2, 3 | Analytics tracking events |
| NFR3 | Epic 2 | SEO meta tags for `/pricing` |
| NFR4 | Epic 3 | Error handling for Stripe Portal |
| NFR5 | Epic 3 | PDF downloads for historical purchases |
| NFR6 | Epic 1 | "How You Compare" renders correctly in PDF |

## Epic List

### Epic 1: DGI Benchmark in Reports

Enable customers to understand their DAO's governance performance in context by showing dimension-by-dimension benchmark comparisons against ecosystem averages in paid reports.

**User Outcome:** Customers see not just their score, but how they compare — "Your GVS 7.2 is Top 25%" with strengths and improvement areas clearly identified.

**FRs covered:** FR1.1, FR1.2, FR1.3, FR1.4, FR1.5, FR1.6, FR1.7, FR1.8, NFR6

**Dependencies:** None (uses existing `dgiSnapshots` data)

---

### Epic 2: Pricing Page

Provide transparent, SEO-friendly pricing information that helps visitors compare all ChainSights products before entering a checkout flow.

**User Outcome:** Visitors can browse all pricing tiers, understand what each includes, and share pricing links in Discord/forums.

**FRs covered:** FR2.1, FR2.2, FR2.3, FR2.4, FR2.5, FR2.6, NFR1, NFR2, NFR3

**Dependencies:** None (can run parallel to Epic 1). Should update TierCards with [NEW] badge after Epic 1 ships.

---

### Epic 3: Account Dashboard

Enable authenticated users to manage their subscriptions and access their report history through a self-service dashboard.

**User Outcome:** Subscribers can view/cancel subscriptions, update payment methods, and download past reports without contacting support.

**FRs covered:** FR3.1, FR3.2, FR3.3, FR3.4, FR3.5, FR3.6, NFR1, NFR2, NFR4, NFR5

**Dependencies:** Magic Link Auth (already deployed), Stripe Customer Portal API

---

## Stories

### Epic 1: DGI Benchmark in Reports

#### Story 1.1: Extend DGI Query for Component Averages

As a **report generator**,
I want **`getCurrentDGI()` to return HPR, DEI, PDI, GPI ecosystem averages**,
So that **I can compare individual DAO scores against ecosystem benchmarks**.

**Acceptance Criteria:**

**Given** the `dgiSnapshots` table contains daily snapshots with `indexType` values including 'hpr', 'dei', 'pdi', 'gpi'
**When** `getCurrentDGI()` is called
**Then** it returns an object containing `{ composite, hpr, dei, pdi, gpi }` with their latest values
**And** each value is the most recent snapshot for that `indexType`
**And** missing values return `null` (graceful degradation)

**Files:** `src/lib/dgi/queries.ts`, optionally `src/lib/dgi/types.ts`

**FRs:** FR1.1

---

#### Story 1.2: Calculate Component Benchmarks in Analysis

As a **report generator**,
I want **to calculate per-dimension benchmarks with percentiles and status**,
So that **reports can show where the DAO is strong or weak compared to the ecosystem**.

**Acceptance Criteria:**

**Given** a DAO has GVS component scores (HPR, DEI, PDI, GPI) and `getCurrentDGI()` returns ecosystem averages
**When** the analysis is generated in `/api/reports/analyze`
**Then** `dgiBenchmark.componentBenchmarks` contains an object for each dimension with:
- `ecosystemAvg`: value from `dgiSnapshots`
- `daoScore`: value from the DAO's own scores
- `percentile`: calculated via `PERCENT_RANK()` against all tracked DAOs
- `status`: "above" (>110%), "below" (<90%), or "average" (±10%)

**And** the percentile query uses all DAOs in `gvsSnapshots` for accurate ranking
**And** if ecosystem data is unavailable, `componentBenchmarks` is `null` (not empty object)

**Files:** `src/app/api/reports/analyze/route.ts`, `src/lib/dgi/types.ts`

**FRs:** FR1.2, FR1.3, FR1.4

---

#### Story 1.3: Render Headline Benchmark for Deep Dive

As a **Deep Dive customer (€49)**,
I want **to see a headline benchmark comparing my GVS to the DGI Composite**,
So that **I understand my DAO's position in the ecosystem at a glance**.

**Acceptance Criteria:**

**Given** a Deep Dive report is generated with `dgiBenchmark` data available
**When** the PDF is rendered
**Then** the benchmark section shows: "GVS {score} vs DGI Composite {composite} (Top {100-percentile}%)"
**And** the section is styled consistently with existing report design (light teal background)
**And** if `dgiBenchmark` is null, the section is gracefully hidden (no error, no placeholder)

**Files:** `src/lib/pdf/report-template.tsx`

**FRs:** FR1.5, FR1.8

---

#### Story 1.4: Render Full Benchmark Section for Governance Audit

As a **Governance Audit customer (€149)**,
I want **to see a full "How You Compare" section with dimension-by-dimension breakdown**,
So that **I can identify my DAO's specific strengths and improvement areas**.

**Acceptance Criteria:**

**Given** a Governance Audit report is generated with `dgiBenchmark.componentBenchmarks` available
**When** the PDF is rendered
**Then** the "How You Compare" section displays:
- Overall score comparison with visual bar and percentage difference
- **Strengths** subsection listing dimensions with status "above", sorted by percentile DESC
- **Improvement Areas** subsection listing dimensions with status "below", sorted by percentile ASC
- Each dimension shows: name, daoScore, ecosystemAvg, percentile badge, percentage difference

**And** dimensions with status "average" appear in neither section (or optionally in a neutral section)
**And** color coding: green for strengths, amber/orange for improvement areas
**And** percentage difference calculated as `((daoScore - ecosystemAvg) / ecosystemAvg * 100)`
**And** section is wrapped in `{tier === 'governance_audit' && ...}` conditional
**And** if `componentBenchmarks` is null, section falls back to headline-only (like Deep Dive)

**Files:** `src/lib/pdf/report-template.tsx`

**FRs:** FR1.6, FR1.7, FR1.8, NFR6

---

### Epic 2: Pricing Page

#### Story 2.1: Create Pricing Page with Report Tiers

As a **visitor**,
I want **to see all report pricing options on a dedicated page**,
So that **I can compare tiers and choose the right report for my needs**.

**Acceptance Criteria:**

**Given** I navigate to `/pricing`
**When** the page loads
**Then** I see a headline "Transparent Pricing. No Hidden Fees." with subline
**And** I see 3 report tier cards: Free Check (€0), Deep Dive (€49), Governance Audit (€149)
**And** each card lists included features as bullet points
**And** Governance Audit has a "Popular" or "Complete Analysis" badge
**And** each card has a CTA button linking to the correct flow (`/check` for reports)
**And** the layout is 3-column on desktop, stacked on mobile (responsive)

**Files:** `src/app/pricing/page.tsx` (new), reuse `TierCards` component pattern from `/check`

**FRs:** FR2.1, FR2.2, FR2.3, NFR1

---

#### Story 2.2: Add Matrix Subscription & FAQ Sections

As a **visitor**,
I want **to see subscription options and answers to common questions**,
So that **I understand the full product offering and can make an informed decision**.

**Acceptance Criteria:**

**Given** I am on `/pricing`
**When** I scroll past the report tiers
**Then** I see a "DAO Matrix" section with 2 subscription cards:
- Monthly: €19/month with features list
- Annual: €99/year with "Save 57%" badge and "= €8.25/month"

**And** each subscription card has a CTA linking to Stripe Checkout (existing Matrix products)
**And** below subscriptions, I see an FAQ section with expandable items (Accordion)
**And** FAQ includes at minimum: tier differences, upgrade path, cancellation, refunds, DAO coverage, free trial
**And** at page bottom, a footer CTA: "Not sure? Try a free Quick Check first."

**Files:** `src/app/pricing/page.tsx`

**FRs:** FR2.5, NFR1

---

#### Story 2.3: Navigation Link, SEO & Analytics

As a **visitor**,
I want **to find the pricing page easily and have it indexed by search engines**,
So that **I can discover it through navigation or Google search**.

**Acceptance Criteria:**

**Given** I am on any page of ChainSights
**When** I look at the navigation bar
**Then** I see a "Pricing" link between Matrix and Login/Account

**Given** a search engine crawls `/pricing`
**When** the page is indexed
**Then** the page has:
- Title: "Pricing — ChainSights | DAO Governance Analytics"
- Meta description with pricing highlights
- Canonical URL: `https://chainsights.one/pricing`
- Open Graph tags for social sharing

**Given** I view the `/pricing` page
**When** the page loads
**Then** a `pricing_view` analytics event is fired

**Files:** Navigation component (Header), `src/app/pricing/page.tsx` (metadata export)

**FRs:** FR2.4, NFR2, NFR3

---

#### Story 2.4: Update TierCards with Benchmark Badge

As a **visitor**,
I want **to see which features are new (DGI Benchmark)**,
So that **I understand the added value of paid reports**.

**Acceptance Criteria:**

**Given** Epic 1 (DGI Benchmark) is deployed
**When** I view tier cards on `/pricing` or `/check`
**Then** the Deep Dive card shows "DGI Benchmark comparison" with a `[NEW]` badge
**And** the Governance Audit card shows "Full How You Compare analysis" with a `[NEW]` badge
**And** the badge styling is consistent (small pill, accent color)
**And** the badge can be removed later via a simple flag or date check (auto-expire after 30 days optional)

**Files:** `TierCards` component or pricing page, `/check` page

**FRs:** FR2.6

---

### Epic 3: Account Dashboard

#### Story 3.1: Create Account Page with Auth Protection

As a **logged-in user**,
I want **to access a protected account page**,
So that **I can view my personal subscription and report information securely**.

**Acceptance Criteria:**

**Given** I am not logged in
**When** I navigate to `/account`
**Then** I am redirected to the login flow (SignInModal or login page)
**And** after successful Magic Link authentication, I am redirected back to `/account`

**Given** I am logged in via Magic Link
**When** I navigate to `/account`
**Then** I see a welcome message with my email: "Welcome back, {email}"
**And** the page layout has sections for Subscription, Reports, and Quick Actions
**And** the page is responsive (desktop, tablet, mobile)

**Files:** `src/app/account/page.tsx` (new), middleware update for `/account` protection

**FRs:** FR3.1, NFR1

---

#### Story 3.2: Display Subscription Status

As a **logged-in user**,
I want **to see my current subscription status**,
So that **I know my plan details and billing schedule**.

**Acceptance Criteria:**

**Given** I have an active Matrix subscription
**When** I view `/account`
**Then** I see a "Your Subscription" card showing:
- Product name: "DAO Matrix Access"
- Plan type: "Monthly (€19/month)" or "Annual (€99/year)"
- Status: "Active ✅"
- Next billing date: formatted date

**Given** I have a cancelled subscription still within its period
**When** I view `/account`
**Then** status shows: "Active until {date}, will not renew"

**Given** I have no subscription
**When** I view `/account`
**Then** I see "No Active Subscription" with upsell CTAs for Monthly/Annual plans

**Given** my payment has failed
**When** I view `/account`
**Then** I see a warning banner with link to update payment method

**Files:** `src/app/account/page.tsx`, query function in `src/lib/db/` or inline

**FRs:** FR3.2, FR3.5

---

#### Story 3.3: Integrate Stripe Customer Portal

As a **subscriber**,
I want **to manage my subscription through Stripe's portal**,
So that **I can update payment methods, view invoices, or cancel without contacting support**.

**Acceptance Criteria:**

**Given** I am on `/account` with an active or past subscription
**When** I click "Manage Subscription" button
**Then** the app calls `/api/billing-portal` with my session
**And** the API looks up my `stripeCustomerId` from subscriptions/orders table
**And** creates a Stripe Billing Portal session with `return_url: /account`
**And** I am redirected to the Stripe Customer Portal

**Given** no `stripeCustomerId` is found for my email
**When** I click "Manage Subscription"
**Then** I see an error message: "No billing history found. Contact support."
**And** the error is logged for debugging (NFR4)

**Files:** `src/app/api/billing-portal/route.ts` (new), `src/app/account/page.tsx`

**FRs:** FR3.3, NFR4

---

#### Story 3.4: Display Report History with Downloads

As a **user who purchased reports**,
I want **to see my report history and download past PDFs**,
So that **I can access reports I've paid for at any time**.

**Acceptance Criteria:**

**Given** I have purchased reports in the past
**When** I view `/account`
**Then** I see a "Your Reports" section listing all my reports:
- DAO name
- Report tier (Deep Dive / Governance Audit)
- Purchase date (formatted)
- [Download PDF] button/link

**And** clicking "Download PDF" downloads the file from `finalPdfUrl`
**And** reports are sorted by date (newest first)

**Given** I have no reports
**When** I view `/account`
**Then** I see "No reports yet." with CTA: "[Order Your First Report]" → `/check`

**Given** a report's PDF URL is expired or unavailable (NFR5)
**When** I click download
**Then** I see a friendly error with support contact option

**Files:** `src/app/account/page.tsx`, query joining `reports` and `orders` tables

**FRs:** FR3.4, NFR5

---

#### Story 3.5: Navigation & Analytics Integration

As a **user**,
I want **navigation to reflect my login state**,
So that **I can easily access my account or log in**.

**Acceptance Criteria:**

**Given** I am logged in
**When** I view the navigation bar
**Then** I see: `[Rankings] [DGI] [Matrix] [Pricing] | [Account] [Logout]`
**And** clicking Account navigates to `/account`
**And** clicking Logout ends my session and redirects to home

**Given** I am not logged in
**When** I view the navigation bar
**Then** I see: `[Rankings] [DGI] [Matrix] [Pricing] | [Login]`
**And** clicking Login opens the SignInModal

**Given** I view `/account`
**When** the page loads
**Then** an `account_view` analytics event is fired

**Given** I click "Manage Subscription"
**When** the portal opens
**Then** a `billing_portal_click` analytics event is fired

**Files:** Header/Navigation component, `src/app/account/page.tsx`

**FRs:** FR3.6, NFR2

---

---

### Epic 4: Feedback Bubble (Epic 17.5)

Enable anonymous, frictionless feedback collection via a floating widget on all pages, giving ChainSights qualitative user insights.

**User Outcome:** Visitors can share thoughts in 5 seconds – one click, one sentence, done.

**Business Outcome:** ChainSights finally gets qualitative data about what visitors think, need, and miss.

**FRs covered:** FR4.1-FR4.5 (see below)

**Dependencies:** None (standalone component)

**Source:** `docs/_masemIT/requirements/ticket-feedback-bubble.md`

#### Functional Requirements

| ID | Requirement |
|----|-------------|
| FR4.1 | `feedback` table stores message, page, device, country, placeholder_shown |
| FR4.2 | POST `/api/feedback` accepts message with rate limiting (10/IP/hour, 3/session) |
| FR4.3 | Floating bubble (48px, bottom-right) expands to full-width input bar on click |
| FR4.4 | Analytics events: `feedback_bubble_click`, `feedback_submitted`, `feedback_dismissed` |
| FR4.5 | Email notification to admin on new feedback submission |

---

#### Story 4.1: Create Feedback Database Schema

As a **developer**,
I want **a `feedback` table in the database**,
So that **user feedback can be stored with context for later analysis**.

**Acceptance Criteria:**

**Given** the database schema needs to be extended
**When** the migration runs
**Then** a `feedback` table exists with:
- `id` (UUID, primary key)
- `message` (TEXT, not null, max 500 chars)
- `page` (VARCHAR 200, not null)
- `referrer` (VARCHAR 500, nullable)
- `country` (VARCHAR 5, nullable – from Vercel geo headers)
- `device` (VARCHAR 20 – 'mobile' | 'desktop' | 'tablet')
- `placeholder_shown` (VARCHAR 200)
- `created_at` (TIMESTAMP, default NOW())

**And** no user identification fields (anonymous by design)

**Files:** `src/lib/db/schema.ts`, migration via `npm run db:generate`

**FRs:** FR4.1

---

#### Story 4.2: Build Feedback API Endpoint

As a **visitor**,
I want **to submit feedback via API**,
So that **my thoughts are captured without page reload**.

**Acceptance Criteria:**

**Given** I submit feedback via POST `/api/feedback`
**When** the request contains `{ message, page, placeholderShown }`
**Then** feedback is stored in database with device/country from headers
**And** response is `200 { success: true }`

**Given** message is empty or > 500 chars
**When** I submit
**Then** response is `400 { error: "..." }`

**Given** I've submitted 10+ times this hour (same IP)
**When** I submit again
**Then** response is `429 { error: "Too many submissions" }`

**Files:** `src/app/api/feedback/route.ts` (new)

**FRs:** FR4.2

---

#### Story 4.3: Build Feedback Bubble Component

As a **visitor**,
I want **a floating feedback bubble that expands on click**,
So that **I can share thoughts quickly without leaving the page**.

**Acceptance Criteria:**

**Given** I'm on any page (except /admin/*)
**When** the page loads
**Then** I see a small floating bubble (48px, bottom-right, #009BB1)

**Given** I click the bubble
**When** it expands
**Then** it animates (300ms) to a full-width input bar
**And** input is autofocused
**And** a random placeholder is shown from rotating list

**Given** I type and press Enter or click ✓
**When** feedback is submitted successfully
**Then** I see "Thanks! Your feedback shapes what we build next." for 2s
**And** bubble collapses back

**Given** I press Escape or click ✕
**When** I haven't submitted
**Then** bubble collapses without submitting

**Given** I've submitted 3x this session
**When** I try again
**Then** bubble is hidden for rest of session

**Files:** `src/components/feedback-bubble.tsx` (new)

**FRs:** FR4.3

---

#### Story 4.4: Integrate Globally & Add Analytics

As a **product owner**,
I want **the feedback bubble on all pages with analytics tracking**,
So that **I can measure engagement and collect feedback site-wide**.

**Acceptance Criteria:**

**Given** the FeedbackBubble component exists
**When** I add it to the root layout
**Then** it appears on all pages except `/admin/*`

**Given** a user interacts with the bubble
**When** they open, submit, or dismiss
**Then** analytics events fire:
- `feedback_bubble_click` (opened)
- `feedback_submitted` (sent, with message length)
- `feedback_dismissed` (closed without sending)

**Files:** `src/app/layout.tsx`, `src/components/feedback-bubble.tsx`

**FRs:** FR4.4

---

#### Story 4.5: Add Email Notification for New Feedback

As a **product owner**,
I want **an email alert when feedback is submitted**,
So that **I see it immediately without checking a dashboard**.

**Acceptance Criteria:**

**Given** feedback is successfully stored in database
**When** the API processes the request
**Then** an email is sent to `contact@masem.at` via Resend containing:
- Subject: `[ChainSights Feedback] {page}`
- Body: message, page, device, country, placeholder shown, timestamp

**And** email failure does NOT block the API response (fire-and-forget)
**And** email sending can be toggled via env var `FEEDBACK_EMAIL_ENABLED`

**Files:** `src/app/api/feedback/route.ts`

**FRs:** FR4.5

---

## Summary

| Epic | Stories | Total FRs |
|------|---------|-----------|
| Epic 1: DGI Benchmark in Reports | 4 | FR1.1-FR1.8, NFR6 |
| Epic 2: Pricing Page | 4 | FR2.1-FR2.6, NFR1-NFR3 |
| Epic 3: Account Dashboard | 5 | FR3.1-FR3.6, NFR1, NFR2, NFR4, NFR5 |
| Epic 4: Feedback Bubble (17.5) | 5 | FR4.1-FR4.5 |
| **Total** | **18 Stories** | **25 FRs + 6 NFRs** |
