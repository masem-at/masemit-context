---
title: 'Legal & Compliance Pages + Consent Banner (Epic 7)'
slug: 'epic-7-legal-compliance'
created: '2026-02-21'
status: 'ready-for-dev'
stepsCompleted: [1, 2, 3, 4]
tech_stack: ['Next.js 15 App Router', 'TypeScript', 'Tailwind v4 + shadcn/ui (OKLCH)', 'Auth.js v5', 'MMS API (x-api-key)', 'MMS Consents API']
files_to_modify: ['src/app/layout.tsx', 'src/lib/analytics.ts', 'src/components/landing/landing-footer.tsx', 'src/components/dashboard/nav-config.ts', 'src/components/admin/admin-nav-config.ts', 'src/components/admin/admin-sidebar.tsx', '.env.example']
files_to_create: ['src/app/(landing)/legal/imprint/page.tsx', 'src/app/(landing)/legal/privacy/page.tsx', 'src/app/(landing)/legal/terms/page.tsx', 'src/app/api/consent/route.ts', 'src/components/shared/consent-provider.tsx', 'src/components/shared/consent-banner.tsx', 'src/lib/consent.ts']
code_patterns: ['SSG static pages under (landing)/', 'Route Handler: auth guard + mmsApi() + MmsApiError', 'React Context Provider in shared/', 'localStorage for client state', 'NavItem[] array pattern for sidebar links', 'shadcn/ui Button/Dialog/Sheet available']
test_patterns: ['No test framework configured — no tests in scope']
---

# Tech-Spec: Legal & Compliance Pages + Consent Banner (Epic 7)

**Created:** 2026-02-21

## Overview

### Problem Statement

PayWatcher ist feature-complete (Epic 1-6, 27 Stories done) aber ohne Legal Pages und Cookie Consent Banner nicht DSGVO-konform. Ohne Impressum (§5 ECG), Datenschutzerklaerung (DSGVO Art. 13/14) und Consent Banner (ePrivacy-Richtlinie) ist ein Go-Live in Oesterreich/EU nicht moeglich.

### Solution

3 statische Legal Pages (Impressum, Privacy Policy, Terms of Service), ein Cookie Consent Banner mit Dual-Storage (localStorage fuer sofortige UI + MMS Consents API fuer DSGVO Audit Trail), und conditional Analytics Loading basierend auf Consent-Status.

### Scope

**In Scope:**
- Impressum Page (`/legal/imprint`) — DE + EN
- Privacy Policy Page (`/legal/privacy`) — EN
- Terms of Service Page (`/legal/terms`) — EN
- Consent Banner Component + ConsentProvider (React Context)
- Route Handler `/api/consent` → MMS POST /v1/consents (fire-and-forget)
- Simple In-Memory Rate Limiting fuer `/api/consent`
- Analytics conditional rendering in Root Layout
- `trackEvent()` Consent-Check in `lib/analytics.ts`
- Footer-Links Update (Landing)
- Sidebar-Links Update (Dashboard + Admin)
- `CONSENT_TEXT_VERSION = "v1.0"` als Konstante

**Out of Scope:**
- Granulare Cookie-Kategorien ueber Essential/Analytics hinaus
- Dedizierte Consent Management Settings-Page
- Multi-Language Legal Pages (nur EN, Impressum zusaetzlich DE)
- Upstash/Redis Rate Limiting
- SLA oder advanced Consent UI (Customize-View)

## Context for Development

### Codebase Patterns

- **Route Handlers**: Auth guard at top with early-return (`"error" in result`), `MmsApiError` typed errors, `snakeToCamel` transform on responses. MMS calls via `mmsApi<T>(path, options, apiKey?)` from `src/lib/api/client.ts` — uses `x-api-key` header, base URL from `MMS_API_URL` env.
- **Layouts**: Landing uses `(landing)/layout.tsx` with `LandingHeader` + `LandingFooter` + `AnalyticsProvider`. Dashboard layout at `(dashboard)/layout.tsx` with `SidebarNav` + `DashboardHeader`. Admin layout at `(admin)/layout.tsx` with `AdminSidebar` + `AdminHeader`. Both Dashboard/Admin use `QueryProvider` wrapper.
- **Analytics**: Custom self-hosted tracker (`analytics.masem.at/tracker.js`), loaded via `<Script>` in root layout line 102-108 (production-only via `VERCEL_ENV === "production"`). `lib/analytics.ts` wraps `window.masemit.track()`. `AnalyticsProvider` component tracks page_view, section_view (IntersectionObserver), cta_click events.
- **Shared Components**: Lean folder at `src/components/shared/` — ThemeProvider, QueryProvider, CodeBlock, CopyButton, ThemeToggle. ConsentProvider + ConsentBanner will go here.
- **Styling**: Tailwind v4 + shadcn/ui, OKLCH color space, dark mode default. Custom type scale (`text-display` through `text-caption`). PayWatcher brand tokens as CSS vars (`--pw-accent`, `--pw-bg`, etc.). Status colors: `success`, `warning`, `error`, `neutral`.
- **Sidebar Nav**: Dashboard uses `NavItem[]` from `nav-config.ts` (href, label, icon). Admin uses `AdminNavItem[]` from `admin-nav-config.ts` (same shape). Both use Lucide icons.
- **UI Components Available**: Button, Dialog, Sheet, AlertDialog, Card, Badge, Input, Label, Select, Tabs, Table, Skeleton, Sonner (toasts), DropdownMenu

### Files to Reference

| File | Purpose |
| ---- | ------- |
| `src/app/layout.tsx` | Root layout — Analytics Script tag to make conditional |
| `src/app/(landing)/layout.tsx` | Landing layout — has Footer + AnalyticsProvider |
| `src/components/landing/landing-footer.tsx` | Footer — needs Legal links update |
| `src/lib/analytics.ts` | Analytics wrapper — needs Consent check |
| `src/components/landing/analytics-provider.tsx` | Landing analytics — uses trackEvent |
| `src/components/dashboard/nav-config.ts` | Dashboard sidebar nav items |
| `src/components/admin/admin-nav-config.ts` | Admin sidebar nav items |
| `src/lib/api/client.ts` | MMS API client — `mmsApi<T>()` with `x-api-key` header |
| `src/lib/api/errors.ts` | `MmsApiError` typed error class |
| `src/lib/auth.ts` | `requireAuth()` — session + tenant API key |
| `src/lib/admin.ts` | `requireAdminAuth()` + `isAdmin()` |
| `src/app/api/admin/payments/route.ts` | Reference: Route Handler pattern (auth guard, param whitelist, mmsApi, error handling) |
| `src/app/(dashboard)/layout.tsx` | Dashboard layout — SidebarNav + DashboardHeader |
| `src/app/(admin)/layout.tsx` | Admin layout — AdminSidebar + AdminHeader |
| `src/components/admin/admin-sidebar.tsx` | Admin sidebar component — needs Legal link |
| `src/app/globals.css` | Design tokens, type scale, PayWatcher brand vars |
| `.env.example` | Env vars — no new env var needed (reusing MMS_API_KEY) |
| `docs/_masemIT/requirements/requirement-paywatcher-legal-compliance-2026-02-21.md` | Source requirement with all legal content |

### Technical Decisions

- **Rate Limiting**: Simple In-Memory rate limiter (Map + sliding window) for `/api/consent` — no Upstash/Redis needed for MVP
- **Consent Storage**: Dual — localStorage (`pw_consent`, `pw_device_id`) for immediate UI + MMS Consents API for DSGVO audit trail
- **MMS API Key**: Existing `MMS_API_KEY` will get `consents:write` scope added — no separate env var needed
- **Consent Text Version**: `CONSENT_TEXT_VERSION = "v1.0"` as constant, bump on Privacy Policy content changes
- **Legal Pages**: SSG, public, no auth — under `app/(landing)/legal/`
- **Footer vs Sidebar**: Landing page gets footer links, Dashboard/Admin get sidebar links

## Implementation Plan

### Stories

---

#### Story 7-1: Legal Pages Layout + Impressum

**Goal:** Create the legal pages route structure and the Impressum page with all §5 ECG mandatory information.

- [ ] Task 1: Create legal page layout (shared wrapper for all legal pages)
  - File: `src/app/(landing)/legal/layout.tsx` (NEW)
  - Action: Create a simple layout that wraps children in a centered, readable container. Use `max-w-3xl mx-auto px-4 py-12`. No header/footer needed — inherited from `(landing)/layout.tsx`. Add `prose prose-invert` classes for markdown-style typography.

- [ ] Task 2: Create Impressum page
  - File: `src/app/(landing)/legal/imprint/page.tsx` (NEW)
  - Action: Create SSG page with static metadata (`title: "Imprint | PayWatcher"`). Content in German (primary) with English section below. Include all §5 ECG mandatory fields:
    - masemIT e.U., Mario Semper, Alleegasse 26, 3851 Kautzen, Austria
    - FN 661236g, ATU82330407
    - contact@masem.at, https://masem.at
    - Unternehmensgegenstand: IT-Dienstleistungen und Softwareentwicklung
    - Zustaendige Behoerde: BH Waidhofen an der Thaya
    - Mitglied der WKO Niederoesterreich
    - Anwendbare Rechtsvorschriften: Gewerbeordnung (www.ris.bka.gv.at)
    - Verbraucherstreitbeilegung: https://ec.europa.eu/consumers/odr
  - Notes: Plain text page, consistent dark mode styling. Use `text-body` / `text-body-sm` / `text-h2` type scale. Links use `text-primary hover:underline`.

**Acceptance Criteria:**
- [ ] AC 7-1.1: Given a user navigates to `/legal/imprint`, when the page loads, then all §5 ECG mandatory fields are displayed (company name, address, FN, UID, contact, authority, WKO membership, applicable law, ODR link).
- [ ] AC 7-1.2: Given any legal page, when rendered, then the content is centered (`max-w-3xl`), readable, and consistent with the dark mode theme.

---

#### Story 7-2: Privacy Policy

**Goal:** Create the Privacy Policy page with all DSGVO Art. 13/14 mandatory information.

- [ ] Task 1: Create Privacy Policy page
  - File: `src/app/(landing)/legal/privacy/page.tsx` (NEW)
  - Action: Create SSG page with metadata (`title: "Privacy Policy | PayWatcher"`). Content in English with note "Eine deutsche Fassung ist auf Anfrage erhältlich." All 10 sections from requirement:
    1. **Data Controller** — masemIT e.U. contact details
    2. **Data We Collect** — table with data type, when, legal basis, retention (from requirement Section 2.2 detail table)
    3. **Legal Basis** — Art. 6(1)(a) consent (analytics), Art. 6(1)(b) contract (login, API), Art. 6(1)(f) legitimate interest (security logs)
    4. **Cookies & Tracking** — Essential (auth session, CSRF, consent preference, device_id) always active + Analytics (masemIT Analytics) only with consent
    5. **Sub-Processors** — Vercel (Hosting, USA, EU SCCs), Neon (DB, EU Frankfurt), Resend (Email, USA, EU SCCs), QStash/Upstash (Queue/Cache)
    6. **Data Sharing** — No third-party sharing beyond sub-processors, no data sales
    7. **Retention Periods** — Auth tokens: session. Request access: 6 months. Payments: 7 years (AT law). Analytics: 26 months.
    8. **Your Rights** — Access, rectification, erasure, restriction, portability, objection, complaint to DSB (dsb.gv.at)
    9. **International Transfers** — USA transfers via EU SCCs (Vercel, Resend)
    10. **Changes** — Last updated date, changes published on this page
  - Notes: Developer-audience tone (like Stripe/Resend). Use tables for structured data. `text-body-sm` for table content. Section headers with `text-h3` and `id` attributes for anchor linking. Last updated date hardcoded as `February 2026`.

**Acceptance Criteria:**
- [ ] AC 7-2.1: Given a user navigates to `/legal/privacy`, when the page loads, then all 10 DSGVO-required sections are present with complete content.
- [ ] AC 7-2.2: Given the privacy policy page, when rendered, then a data processing table lists all 8 data types with legal basis and retention period for each.
- [ ] AC 7-2.3: Given the privacy policy page, when rendered, then all sub-processors are listed with their location and transfer mechanism.

---

#### Story 7-3: Terms of Service

**Goal:** Create the Terms of Service page with the critical verification-not-custody distinction.

- [ ] Task 1: Create Terms of Service page
  - File: `src/app/(landing)/legal/terms/page.tsx` (NEW)
  - Action: Create SSG page with metadata (`title: "Terms of Service | PayWatcher"`). Content in English. All 12 sections from requirement:
    1. **Service Description** — Verification layer for stablecoin payments. Not a payment processor, not a custodian, not a money transmitter.
    2. **Eligibility** — 18+, business-capable. B2B service.
    3. **Account & API Keys** — Managed onboarding. API key confidential, user responsible. Misuse = deactivation.
    4. **Acceptable Use** — No illegal activities, no money laundering, no terrorism financing, no sanctions evasion, no fraud. Accounts suspended without warning on suspicion.
    5. **Service Level** — "As-is" MVP. No SLA. Best-effort availability. Planned maintenance announced.
    6. **Fees & Billing** — Per pricing page. Flat fee per verification. No hidden costs. Changes with 30 days notice.
    7. **Limitation of Liability** — Verifies payments, no guarantee of blockchain data correctness. No liability for: missed payments, reorgs, smart contract bugs, fund loss. Liability capped at fees paid in last 12 months.
    8. **Intellectual Property** — API, docs, dashboard = masemIT IP. User retains rights to own data.
    9. **Termination** — Either party, any time. On termination: API key deactivated, data deleted per retention policy.
    10. **Governing Law** — Austrian law, court jurisdiction Waidhofen an der Thaya (B2B). Consumers: statutory jurisdiction.
    11. **Changes** — 30 days notice via email. Continued use = acceptance.
    12. **Contact** — contact@masem.at
  - Notes: **CRITICAL** — The verification-not-custody disclaimer must be prominently displayed as a blockquote/callout near the top (after Service Description). Use a styled `<blockquote>` or a `Card` component with border-left accent. Text from requirement: "PayWatcher is a payment **verification** service, not a payment **processor**..."

**Acceptance Criteria:**
- [ ] AC 7-3.1: Given a user navigates to `/legal/terms`, when the page loads, then all 12 sections are present with complete content.
- [ ] AC 7-3.2: Given the terms page, when rendered, then the verification-not-custody disclaimer is prominently displayed as a highlighted callout.
- [ ] AC 7-3.3: Given the terms page, when rendered, then the liability limitation section clearly states PayWatcher does not guarantee blockchain data correctness.

---

#### Story 7-4: Consent Banner + MMS Integration + Analytics Conditional Loading

**Goal:** Implement the cookie consent system with dual storage (localStorage + MMS audit trail) and make analytics loading conditional on consent.

- [ ] Task 1: Create consent utility library
  - File: `src/lib/consent.ts` (NEW)
  - Action: Create utility with:
    ```typescript
    export const CONSENT_TEXT_VERSION = "v1.0"

    export interface ConsentState {
      essential: true  // always true
      analytics: boolean
      timestamp: string
    }

    export function getConsentState(): ConsentState | null
    // Read from localStorage key "pw_consent", parse JSON, return null if not set

    export function setConsentState(analytics: boolean): ConsentState
    // Write to localStorage "pw_consent" with { essential: true, analytics, timestamp: new Date().toISOString() }

    export function getDeviceId(): string
    // Read from localStorage "pw_device_id", generate UUID via crypto.randomUUID() if not exists, persist and return

    export function isAnalyticsConsented(): boolean
    // Return getConsentState()?.analytics === true

    export function clearConsentState(): void
    // Remove "pw_consent" from localStorage (for re-opening banner)
    ```

- [ ] Task 2: Create ConsentProvider (React Context)
  - File: `src/components/shared/consent-provider.tsx` (NEW)
  - Action: Create `"use client"` provider component:
    - React Context with `{ consentState: ConsentState | null, updateConsent: (analytics: boolean) => void, resetConsent: () => void }`
    - On mount: read `getConsentState()` from localStorage
    - `updateConsent(analytics)`: calls `setConsentState(analytics)`, updates context state, fires POST to `/api/consent` (fire-and-forget via `fetch` with no `await`), sets `consentState`
    - `resetConsent()`: calls `clearConsentState()`, sets `consentState` to `null` (which triggers banner to show)
    - Export `useConsent()` hook
    - **Conditionally render analytics Script tag**: When `consentState?.analytics === true` AND `process.env.VERCEL_ENV === "production"`, render the `<Script src="https://analytics.masem.at/tracker.js" ... />` tag. Otherwise render nothing.
  - Notes: The Script tag moves from `src/app/layout.tsx` INTO this provider. This is the single source of truth for analytics loading.

- [ ] Task 3: Create ConsentBanner component
  - File: `src/components/shared/consent-banner.tsx` (NEW)
  - Action: Create `"use client"` banner component:
    - Uses `useConsent()` hook
    - Only renders when `consentState === null` (no consent decision yet)
    - Position: `fixed bottom-0 left-0 right-0 z-50` (full-width bottom bar)
    - Dark theme consistent: `bg-card border-t border-border`
    - Content: Short text explaining cookies (2 sentences max). Link to `/legal/privacy`.
    - Two buttons, **equally styled** (no dark patterns):
      - "Accept All" → `updateConsent(true)`
      - "Essential Only" → `updateConsent(false)`
    - Both buttons use `Button` from shadcn/ui, same variant (`outline` or both `default`), same size
    - Animation: slide-up on appear, fade-out on dismiss
  - Notes: No "Customize" view for MVP. Keep it simple: two buttons, one text, one link.

- [ ] Task 4: Create consent API route handler
  - File: `src/app/api/consent/route.ts` (NEW)
  - Action: Create POST handler (NO auth guard — public endpoint):
    ```typescript
    // 1. Rate limiting: simple in-memory Map<string, number[]>
    //    Key: IP address, Value: array of timestamps
    //    Limit: 10 requests per minute per IP
    //    Clean up entries older than 60 seconds on each request

    // 2. Parse request body:
    //    { device_id: string, consent_type: string, granted: boolean, consent_text_version: string }

    // 3. Extract from request: IP (x-forwarded-for or request headers), User-Agent

    // 4. Check if user is logged in: auth() session (optional — don't fail if not logged in)
    //    If logged in: extract party_id from session

    // 5. POST to MMS: mmsApi('/v1/consents', {
    //      method: 'POST',
    //      body: JSON.stringify({
    //        party_id: session?.user?.partyId ?? undefined,  // only if logged in
    //        device_id,
    //        consent_type,        // "cookie_analytics"
    //        scope: "project",
    //        project: "paywatcher",
    //        granted,
    //        ip_address,
    //        user_agent,
    //        consent_text_version
    //      })
    //    })

    // 6. Return 200 OK (fire-and-forget: don't fail UI if MMS call fails)
    //    Catch all MMS errors → log server-side, still return 200
    ```
  - Notes: This route is public — no `requireAuth()`. The `auth()` call is optional enrichment (adds `party_id` if user happens to be logged in). Rate limit by IP to prevent abuse.

- [ ] Task 5: Update root layout — move Analytics Script to ConsentProvider
  - File: `src/app/layout.tsx` (MODIFY)
  - Action:
    - Remove the existing `<Script>` tag (lines 102-108)
    - Import and wrap children with `<ConsentProvider>` inside the `<ThemeProvider>`
    - Add `<ConsentBanner />` inside the ConsentProvider
    - The ConsentProvider handles conditional Script rendering internally
    - Result structure:
      ```tsx
      <ThemeProvider ...>
        <ConsentProvider>
          {children}
          <Toaster position="bottom-right" />
          <ConsentBanner />
        </ConsentProvider>
      </ThemeProvider>
      ```

- [ ] Task 6: Update analytics.ts — add consent check
  - File: `src/lib/analytics.ts` (MODIFY)
  - Action:
    - Import `isAnalyticsConsented` from `@/lib/consent`
    - Add consent check as first guard in `trackEvent()`:
      ```typescript
      export function trackEvent(name: string, properties?: Record<string, string>) {
        if (typeof window === "undefined") return
        if (!isAnalyticsConsented()) return          // ← NEW: consent check
        if (process.env.NODE_ENV !== "production") {
          console.debug("[analytics]", name, properties)
          return
        }
        window.masemit?.track(name, properties)
      }
      ```
  - Notes: Double protection — Script tag not loaded without consent AND trackEvent bails out. Belt and suspenders.

**Acceptance Criteria:**
- [ ] AC 7-4.1: Given a first-time visitor on any page, when the page loads, then the consent banner appears at the bottom with "Accept All" and "Essential Only" buttons of equal prominence.
- [ ] AC 7-4.2: Given a user clicks "Accept All", when the banner dismisses, then `localStorage.pw_consent` contains `{ essential: true, analytics: true, timestamp: "..." }` and the analytics `<Script>` tag is rendered in the DOM.
- [ ] AC 7-4.3: Given a user clicks "Essential Only", when the banner dismisses, then `localStorage.pw_consent` contains `{ essential: true, analytics: false, timestamp: "..." }` and NO analytics `<Script>` tag is in the DOM.
- [ ] AC 7-4.4: Given a consent decision is made, when the decision is saved, then a POST request is sent to `/api/consent` with `device_id`, `consent_type: "cookie_analytics"`, `granted`, and `consent_text_version: "v1.0"`.
- [ ] AC 7-4.5: Given the `/api/consent` route receives a valid POST, when processing, then it forwards the consent record to MMS `POST /v1/consents` with `scope: "project"`, `project: "paywatcher"`, IP address, and user-agent.
- [ ] AC 7-4.6: Given a logged-in user makes a consent decision, when the POST reaches `/api/consent`, then the MMS record includes `party_id` from the auth session.
- [ ] AC 7-4.7: Given a returning visitor with existing consent, when any page loads, then the consent banner does NOT appear.
- [ ] AC 7-4.8: Given analytics consent is NOT granted, when `trackEvent()` is called, then the function returns early without calling `window.masemit.track()`.
- [ ] AC 7-4.9: Given more than 10 requests per minute from the same IP to `/api/consent`, when the 11th request arrives, then a 429 status is returned.
- [ ] AC 7-4.10: Given the MMS API call fails in `/api/consent`, when the error is caught, then the route still returns 200 OK (fire-and-forget) and the error is logged server-side.

---

#### Story 7-5: Footer Links + Sidebar Navigation Update

**Goal:** Add legal page links to the landing footer and cookie settings trigger to all areas. Add legal links to dashboard/admin sidebars.

- [ ] Task 1: Update landing footer with legal links
  - File: `src/components/landing/landing-footer.tsx` (MODIFY)
  - Action:
    - Replace the existing dead `/impressum` link with `/legal/imprint`
    - Add links: `/legal/privacy` (Privacy), `/legal/terms` (Terms)
    - Add "Cookie Settings" button that triggers `resetConsent()` from `useConsent()` hook
    - Reorganize footer links into two groups: "Product" (GitHub, Docs, Status) and "Legal" (Imprint, Privacy, Terms, Cookie Settings)
    - Mark component as `"use client"` (needed for `useConsent()` hook)
  - Notes: "Cookie Settings" is NOT a link — it's a button styled like a link that calls `resetConsent()` which clears localStorage and re-shows the banner.

- [ ] Task 2: Add legal links to dashboard sidebar
  - File: `src/components/dashboard/nav-config.ts` (MODIFY)
  - Action: Add a `legalItems` export below `navItems`:
    ```typescript
    import { Scale } from "lucide-react"

    export const legalItems: NavItem[] = [
      { href: "/legal/imprint", label: "Legal", icon: Scale },
    ]
    ```
  - Notes: Single "Legal" link pointing to imprint — from there users can navigate to Privacy/Terms. Keeps sidebar clean.

- [ ] Task 3: Add legal links to admin sidebar
  - File: `src/components/admin/admin-nav-config.ts` (MODIFY)
  - Action: Same pattern as dashboard — add `legalItems` export:
    ```typescript
    import { Scale } from "lucide-react"

    export const adminLegalItems: AdminNavItem[] = [
      { href: "/legal/imprint", label: "Legal", icon: Scale },
    ]
    ```

- [ ] Task 4: Render legal items in dashboard sidebar
  - File: `src/components/dashboard/sidebar-nav.tsx` (MODIFY)
  - Action: Import `legalItems` from `nav-config.ts`. Render them as a separate section at the bottom of the sidebar (below the main nav items), with a thin `border-t border-border` separator and `mt-auto` to push to bottom.

- [ ] Task 5: Render legal items in admin sidebar
  - File: `src/components/admin/admin-sidebar.tsx` (MODIFY)
  - Action: Import `adminLegalItems` from `admin-nav-config.ts`. Same pattern as dashboard — separate section at bottom with border separator.

**Acceptance Criteria:**
- [ ] AC 7-5.1: Given the landing page footer, when rendered, then links to Imprint, Privacy Policy, and Terms of Service are visible and navigate to the correct `/legal/*` routes.
- [ ] AC 7-5.2: Given the landing page footer, when "Cookie Settings" is clicked, then the consent banner re-appears (localStorage cleared, consent state reset to null).
- [ ] AC 7-5.3: Given the dashboard sidebar, when rendered, then a "Legal" link is visible at the bottom that navigates to `/legal/imprint`.
- [ ] AC 7-5.4: Given the admin sidebar, when rendered, then a "Legal" link is visible at the bottom that navigates to `/legal/imprint`.

---

### Acceptance Criteria (Summary — All Stories)

| AC | Story | Criterion |
|----|-------|-----------|
| 7-1.1 | 7-1 | Impressum shows all §5 ECG fields |
| 7-1.2 | 7-1 | Legal pages use consistent centered layout with dark mode |
| 7-2.1 | 7-2 | Privacy policy has all 10 DSGVO sections |
| 7-2.2 | 7-2 | Data processing table lists all 8 data types |
| 7-2.3 | 7-2 | Sub-processors listed with location + transfer mechanism |
| 7-3.1 | 7-3 | Terms has all 12 sections |
| 7-3.2 | 7-3 | Verification-not-custody disclaimer prominently displayed |
| 7-3.3 | 7-3 | Liability limitation section is clear |
| 7-4.1 | 7-4 | Consent banner shows on first visit |
| 7-4.2 | 7-4 | "Accept All" enables analytics + Script tag |
| 7-4.3 | 7-4 | "Essential Only" disables analytics, no Script tag |
| 7-4.4 | 7-4 | Consent POST sent to `/api/consent` |
| 7-4.5 | 7-4 | MMS receives consent record with all fields |
| 7-4.6 | 7-4 | Logged-in user consent includes party_id |
| 7-4.7 | 7-4 | Returning visitor: no banner |
| 7-4.8 | 7-4 | trackEvent respects consent |
| 7-4.9 | 7-4 | Rate limiting: 429 after 10/min |
| 7-4.10 | 7-4 | MMS failure: still returns 200 |
| 7-5.1 | 7-5 | Footer has all legal links |
| 7-5.2 | 7-5 | Cookie Settings re-opens banner |
| 7-5.3 | 7-5 | Dashboard sidebar has Legal link |
| 7-5.4 | 7-5 | Admin sidebar has Legal link |

## Additional Context

### Dependencies

- MMS Consents API (`api.masem.at`) — already live, needs `consents:write` scope on existing API key
- masemIT Analytics (`analytics.masem.at/tracker.js`) — already integrated, needs conditional loading
- `auth()` from Auth.js v5 — used optionally in consent route for `party_id` enrichment

### Testing Strategy

- **No automated tests** — no test framework configured in the project
- **Manual testing checklist:**
  1. Visit `/legal/imprint`, `/legal/privacy`, `/legal/terms` — verify content renders, dark mode, links work
  2. Clear localStorage → visit any page → verify banner appears
  3. Click "Accept All" → verify Script tag in DOM, localStorage set, network request to `/api/consent`
  4. Click "Essential Only" → verify NO Script tag, localStorage set, network request to `/api/consent`
  5. Reload page → verify banner does NOT reappear
  6. Click "Cookie Settings" in footer → verify banner reappears
  7. Check dashboard sidebar → "Legal" link present at bottom
  8. Check admin sidebar → "Legal" link present at bottom
  9. Verify `/api/consent` returns 429 after rapid requests (curl loop)
  10. Verify MMS failure doesn't break consent flow (temporarily invalid API key)

### Notes

- **Impressum** content must comply with §5 ECG (Austrian law) — all mandatory fields from requirement
- **Privacy Policy** must cover DSGVO Art. 13/14 information obligations — all 8 data types documented
- **Terms** must prominently state: PayWatcher = verification service, NOT payment processor/custodian
- **Consent Banner**: no dark patterns — both buttons equally styled and sized
- **Implementation Order**: 7-1 → 7-5 → 7-2 → 7-3 → 7-4 (layout + footer first, then content pages, then the big consent story last)
- **Content Source**: All legal text content comes from `docs/_masemIT/requirements/requirement-paywatcher-legal-compliance-2026-02-21.md` — the requirement IS the content spec
