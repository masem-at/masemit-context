# ChainSights Phase 3.5: Open DAO Matrix (Freemium Pivot)

**Status:** ✅ Complete (All Stories Done)
**Created:** 2026-02-05
**Updated:** 2026-02-06
**Priority:** High
**Effort:** Small-Medium
**Source:** docs/_masemIT/requirements/ticket-open-dao-matrix.md

---

## Overview

Strategic pivot from paywall-first to freemium model. Raw data becomes free, analysis/interpretation costs money.

**Why now?** Radiant Capital proved that open data gets shared (45 referrals in one day). A DAO delegate who can't see their score won't share the link. One who sees their #1 rank shares it everywhere.

**Core Principle:** The data is free. The insights cost money.

---

## Epic 21: Open DAO Matrix

**Priority:** High
**Effort:** Small-Medium
**Dependencies:** None (frontend-only changes)

### Business Context

Current state:
- Matrix shows 5 DAOs, rest hidden behind "Sign in to unlock"
- Detail pages show only GVS chart, others locked
- Logged-in (non-paying) users see more but not everything
- Paying subscribers see everything + CSV/JSON export

Problem:
- Best potential multipliers (DAO delegates, governance enthusiasts) blocked by paywall
- Forum criticism of paywall = losing trust and reach
- People who can't see their score don't share links

Solution:
- All 46 DAOs visible in Matrix (no login required)
- All 5 charts visible on Detail pages (no login required)
- CSV/JSON export free for everyone (subscription model removed)
- Login/auth hidden (kept for admin access only)
- Report CTAs guide users to paid analysis (only paid feature)

### What Changes

#### Free for Everyone (no login)
| Feature | Before | After |
|---------|--------|-------|
| Matrix — all DAOs visible | 5 visible, rest hidden | All 46 visible |
| Matrix — sorting + filtering | Login required | Free |
| Detail — HPR/DEI/PDI/GPI charts | Locked | Open |
| Detail — Benchmark vs Avg | Locked | Open |
| CSV/JSON Export | €19/mo subscription | **Free** |
| Rankings page | Tiered access | **Free as "Top 10 DAOs"** |
| Login/Sign-in | Visible | **Hidden** (admin only) |

#### Stays Behind Paywall
| Feature | Gate |
|---------|------|
| AI-powered Reports | One-time (€0/€49/€149) |

**Subscription model completely removed.** Only revenue = Report purchases.

### Acceptance Criteria

- [x] DAO Matrix shows all 46 DAOs without login
- [x] Sorting and filtering work without login
- [x] DAO Detail pages show all 5 charts without login
- [x] Report CTA banner on every DAO Detail page
- [x] "Sign in to unlock" removed from all content pages
- [x] All new analytics events fire correctly
- [x] Feature flag `OPEN_MATRIX` for quick rollback
- [x] OG Meta tags per DAO page
- [x] CSV/JSON Export free for everyone (no gate)
- [x] Rankings renamed to "Top 10 DAOs" at `/top-daos`
- [x] Vanity URLs work for exact slugs (`/{slug}` → `/matrix/{slug}`)
- [x] **Vanity URL aliases** for human-friendly names (e.g., `/radiant-capital` → `radiantcapital-eth`) — **Story 21-5**
- [x] **Clickable Matrix rows** — entire row navigates to detail page — **Story 21-6**
- [x] Login button hidden (admin access preserved)
- [x] Subscription infrastructure removed/disabled

---

## Stories

### Story 21-1: Remove Matrix Paywall

**As a** visitor,
**I want** to see all DAOs in the Matrix without signing in,
**So that** I can explore governance scores freely and find my DAO.

#### Acceptance Criteria

**Given** I visit `/matrix` without being logged in
**When** the page loads
**Then** I see all 46 DAOs in the table (not just 5)
**And** I can sort by any column (GVS, HPR, DEI, PDI, GPI, 7d trend)
**And** I can filter by category
**And** I can search for DAOs
**And** there is NO "X more DAOs hidden" banner
**And** there is NO "Sign in to unlock" button

**Given** I click CSV or JSON export without subscription
**When** the modal appears
**Then** I see "Subscribe for full export access — €19/mo" with CTA
**And** `export_gate_hit` event fires with format and dao_count

**Given** I click on a DAO row
**When** I'm redirected to the detail page
**Then** `matrix_dao_click` event fires with dao slug and rank

#### Technical Notes

Remove/modify in Matrix page:
- Row limiting logic (`visibleDAOs = allDAOs` instead of `slice(0, 5)`)
- "X more DAOs hidden" banner
- "Sign in to unlock" button
- Row blur overlay
- Login gate for sorting

Keep:
- Export buttons with 🔒 icon for non-subscribers
- Subscription modal on export click

**Files:** `src/app/matrix/page.tsx`, Matrix client component

---

### Story 21-2: Remove Detail Page Chart Locks + Add Report CTA

**As a** visitor,
**I want** to see all governance charts for any DAO,
**So that** I can understand the full picture before deciding on a paid report.

#### Acceptance Criteria

**Given** I visit `/matrix/{dao-slug}` without being logged in
**When** the page loads
**Then** I see ALL 5 charts: GVS, HPR, DEI, PDI, GPI
**And** I see the Benchmark vs Average comparison
**And** there is NO "Sign in to unlock more charts" overlay
**And** there is NO blur on any charts
**And** `dao_detail_view` event fires with dao, score, category, rank

**Given** I scroll to the bottom of a DAO detail page
**When** the Report CTA banner comes into view
**Then** I see a banner with:
- Personalized headline using the DAO's strongest metric, e.g.:
  - "Your HPR is 9.0 (A+) — but what does that mean for your DAO?"
  - "Your DEI dropped 12% this month — find out why."
  - "You're #3 in DeFi — what's holding you back from #1?"
- Subline: "Our AI-powered reports analyze the numbers and give you specific recommendations."
- 3 CTAs: [Get Free Check] [Deep Dive — €49] [Full Audit — €149]
**And** `dao_detail_report_cta_view` event fires

**Given** I click a Report CTA
**When** I'm redirected to `/check`
**Then** `dao_detail_report_cta_click` event fires with dao and tier
**And** the DAO is pre-selected in the check flow (if possible)

#### Technical Notes

Remove from Detail pages:
- `showChart` gating logic (all charts now `true`)
- "Sign in to unlock" overlays
- Blur CSS on locked charts
- Subscribe modal triggers on chart view

Add:
- `ReportCtaBanner` component at page bottom
- Pre-fill DAO in `/check` via query param (optional)

**Files:** `src/app/matrix/[slug]/page.tsx`, new `src/components/report-cta-banner.tsx`

---

### Story 21-3: Analytics Events + OG Meta Tags

**As a** product owner,
**I want** comprehensive analytics on the open Matrix,
**So that** I can measure engagement and optimize the funnel.

#### Acceptance Criteria

**Analytics Events:**

| Event | Trigger | Properties |
|-------|---------|------------|
| `matrix_dao_click` | Click DAO row in Matrix | dao, rank, source |
| `matrix_sort` | Sort column | column, direction |
| `matrix_filter` | Category filter | category |
| `matrix_search` | Search | query, results |
| `export_gate_hit` | Export click without sub | format, dao_count |
| `dao_detail_view` | Detail page load | dao, dgi_score, category, rank |
| `dao_chart_interact` | Chart hover/click | dao, chart, action |
| `dao_detail_report_cta_view` | CTA scrolls into view | dao |
| `dao_detail_report_cta_click` | CTA click | dao, tier |

**OG Meta Tags (per DAO):**

**Given** someone shares a DAO detail page URL
**When** it renders in Discord/X/LinkedIn
**Then** the preview shows:
- Title: "{DAO Name} — DGI Score {score} ({grade}) | ChainSights"
- Description: "{DAO} ranks #{rank}. HPR: {hpr}, DEI: {dei}, PDI: {pdi}, GPI: {gpi}"
- Image: Default ChainSights OG image (custom per-DAO images = future)

**Feature Flag:**

- `OPEN_MATRIX=true` in env enables freemium mode
- `OPEN_MATRIX=false` reverts to old paywall behavior
- Default: `true` (but allows quick rollback)

**Files:** Analytics calls in Matrix + Detail pages, metadata in Detail page, env config

---

### Story 21-4: Rankings → Top 10 + Subscription Sunset

**As a** visitor,
**I want** a simple "Top 10 DAOs" entry point and no login friction,
**So that** I can quickly see the best DAOs and access all data without barriers.

#### Part A: Rankings → Top 10 DAOs

**Given** I visit `/top-daos`
**When** the page loads
**Then** I see "Top 10 DAOs by Governance Health" as the title
**And** I see exactly 10 DAOs ranked by GVS score
**And** there is NO login gate or paywall
**And** I see a CTA at the bottom: "Want to see all 46 DAOs? [View Full DAO Matrix →]"
**And** `top_daos_view` event fires

**Given** I visit `/rankings`
**When** the redirect occurs
**Then** I am 301 redirected to `/top-daos`

**Given** I visit `/{dao-slug}` (e.g., `/radiant-capital`)
**When** the redirect occurs
**Then** I am 301 redirected to `/matrix/{dao-slug}`

**Navigation:**
- Nav label changes from "Rankings" to "Top 10"

**OG Meta Tags:**
- Title: "Top 10 DAOs by Governance Health | ChainSights"
- Description: "The highest-scoring DAOs ranked by DGI. See who leads in governance health."

#### Part B: Subscription Sunset

**Given** I am on any page
**When** I look at the header
**Then** there is NO "Sign In" button visible
**And** there is NO "UserMenu" component visible

**Given** I click CSV or JSON export in the Matrix
**When** the export triggers
**Then** the file downloads immediately (no modal, no gate)
**And** `matrix_export` event fires with format and dao_count

**Given** I visit `/account`
**When** the page loads
**Then** I am redirected to `/pricing` (or see a minimal page)

**Given** I am an admin and visit `/admin`
**When** I need to authenticate
**Then** Magic Link auth still works (hidden but functional)
**And** `/admin/*` routes remain protected

#### Technical Notes

**Part A - Rankings → Top 10:**
- Rename page component and route
- Update nav label in Header component
- Add redirects in `next.config.js` or middleware:
  - `/rankings` → `/top-daos` (301)
  - `/{dao-slug}` → `/matrix/{dao-slug}` (301) for all 46 DAO slugs
- Limit display to top 10 DAOs
- Add "View Full Matrix" CTA at bottom
- Update OG meta tags

**Part B - Subscription Sunset:**
- Hide SignInModal trigger in Header (`visible: false` or conditional render)
- Hide UserMenu component
- Remove export gate logic in Matrix (no modal, direct download)
- Remove `export_gate_hit` event, add `matrix_export` event
- Remove or redirect `/account` page
- Keep Magic Link auth infrastructure (for admin)
- Deactivate/remove Stripe subscription webhook handlers
- Remove subscription-related code paths (but keep code commented/flagged for potential future use)

**Files:**
- `src/components/Header.tsx` — hide login button
- `src/app/matrix/matrix-client.tsx` — remove export gate
- `src/app/rankings/` → rename to `src/app/top-daos/`
- `next.config.js` or `middleware.ts` — redirects
- `src/app/account/page.tsx` — redirect or simplify
- Stripe webhook handlers — deactivate subscription events

---

### Story 21-5: Vanity URL Aliases

**As a** visitor or DAO sharing their ChainSights link,
**I want** human-friendly URLs like `/radiant-capital` to work,
**So that** I don't need to know the technical Snapshot slug.

#### Problem

Current vanity URLs only work with exact Snapshot-based slugs:
- ✅ `chainsights.one/radiantcapital-eth` → works (exact slug match)
- ❌ `chainsights.one/radiant-capital` → 404 (human-friendly, but not in DB)

Users expect intuitive URLs based on DAO names, not Snapshot space IDs.

#### Solution

Add `slug_alias` column to `daos` table. Middleware checks both:
1. `slug` (original, e.g., `radiantcapital-eth`)
2. `slug_alias` (human-friendly, e.g., `radiant-capital`)

Both redirect to `/matrix/{slug}` (canonical URL uses original slug).

#### Acceptance Criteria

**Given** I visit `/{slug}` where `slug` matches a DAO's `slug` column
**When** the middleware checks
**Then** I am 301 redirected to `/matrix/{slug}`

**Given** I visit `/{alias}` where `alias` matches a DAO's `slug_alias` column
**When** the middleware checks
**Then** I am 301 redirected to `/matrix/{slug}` (using the canonical slug, not the alias)

**Given** I visit `/radiant-capital` and that alias exists for `radiantcapital-eth`
**When** the redirect occurs
**Then** I land on `/matrix/radiantcapital-eth`

#### Technical Notes

**Database:**
- Add `slug_alias VARCHAR(100)` to `daos` table (nullable, unique if set)
- Index on `slug_alias` for fast lookups

**Middleware update:**
```typescript
// Check both slug and slug_alias
const dao = await findDaoBySlugOrAlias(segment)
if (dao) {
  return redirect(`/matrix/${dao.slug}`, 301) // Always use canonical slug
}
```

**Initial aliases to populate:**
| DAO | slug | slug_alias |
|-----|------|------------|
| Radiant Capital | radiantcapital-eth | radiant-capital |
| Arbitrum | arbitrumfoundation-eth | arbitrum |
| Optimism | opcollective-eth | optimism |
| Uniswap | uniswap | uni |
| Aave | aave-eth | aave |
| ... | ... | ... |

**Files:**
- `drizzle/0019_*.sql` — migration adding `slug_alias`
- `src/lib/db/schema.ts` — add column to schema
- `src/lib/db/queries/leaderboard.ts` — update `daoSlugExists()` to check both
- `src/middleware.ts` — no change needed if query handles both

---

### Story 21-6: Clickable Matrix Rows

**As a** visitor browsing the DAO Matrix,
**I want** to click anywhere on a row to navigate to the detail page,
**So that** I don't have to precisely click the DAO name link.

#### Acceptance Criteria

**Given** I am on `/matrix`
**When** I hover over any row
**Then** I see a pointer cursor indicating clickability

**Given** I click anywhere on a DAO row (not just the name)
**When** the click event fires
**Then** I am navigated to `/matrix/{slug}`
**And** a `matrix_dao_click` analytics event fires

#### Technical Notes

- Add `cursor-pointer` to `<tr>` elements
- Add `onClick` handler using `useRouter().push()`
- Use `e.stopPropagation()` on the name Link to prevent double navigation
- Analytics event already exists, just moved to row level

**Files:**
- `src/app/matrix/matrix-client.tsx` — row click handler

**Status:** ✅ Done (2026-02-06)

---

## New Funnel

```
top_daos_view (entry point for casuals)
  ↓
matrix_view (full exploration)
  → matrix_dao_click
    → dao_detail_view
      → dao_chart_interact
        → dao_detail_report_cta_view
          → dao_detail_report_cta_click
            → quick_check_start
              → quick_check_complete
                → checkout_complete (€49/€149 reports only)
```

**No login required anywhere in funnel. Only payment = report purchase.**

---

## SEO Benefit

46 DAO detail pages become indexable landing pages:
- `chainsights.one/matrix/radiant-capital` → "Radiant Capital Governance Score"
- `chainsights.one/matrix/ens` → "ENS Governance Score"
- `chainsights.one/matrix/lido` → "Lido Governance Score"

Each shareable with rich previews = organic reach multiplier.

---

## Out of Scope

- Comparison Mode (future, if needed)
- Share buttons on detail pages (future)
- Custom OG images per DAO (future)
- Report generation or content changes
- "Governance Index" → "Methodology" rename (separate ticket)
- Category-specific Top 10 (e.g., "Top 10 DeFi DAOs")

---

## Dependencies

| Dependency | Status |
|------------|--------|
| Matrix page exists | ✅ |
| Detail pages exist | ✅ |
| Analytics infrastructure | ✅ |
| Subscription gate logic | ✅ |

---

## Rollout Plan

**Done (Stories 21-1 to 21-3):**
1. ✅ Add `OPEN_MATRIX` feature flag
2. ✅ Remove paywall logic (Matrix + Detail pages)
3. ✅ Add Report CTA banner on Detail pages
4. ✅ Implement new analytics events
5. ✅ Add OG meta tags per DAO

**Done (Story 21-4):**
6. ✅ Rename Rankings → Top 10 DAOs
7. ✅ Add redirects (Rankings + Vanity URLs via dynamic DB lookup for exact slugs)
8. ✅ Remove export gate (CSV/JSON free)
9. ✅ Hide login button + UserMenu (`SHOW_AUTH_UI` flag)
10. ✅ Redirect /account to /pricing
11. ✅ Deployed and monitoring

**Done (Story 21-5):**
12. ✅ Add `slug_alias` column to `daos` table for human-friendly vanity URLs
13. ✅ Update middleware to check both `slug` AND `slug_alias`
14. ✅ Populate aliases for existing DAOs (e.g., `radiant-capital` → `radiantcapital-eth`)

**Done (Story 21-6):**
15. ✅ Make entire Matrix rows clickable (not just DAO name)
16. ✅ Add pointer cursor on row hover
17. ✅ Analytics event fires on row click
