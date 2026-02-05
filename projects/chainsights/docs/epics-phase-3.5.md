# ChainSights Phase 3.5: Open DAO Matrix (Freemium Pivot)

**Status:** Ready for Development
**Created:** 2026-02-05
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
- CSV/JSON export remains behind subscription gate
- Report CTAs guide users to paid analysis

### What Changes

#### Free for Everyone (no login)
| Feature | Before | After |
|---------|--------|-------|
| Matrix — all DAOs visible | 5 visible, rest hidden | All 46 visible |
| Matrix — sorting + filtering | Login required | Free |
| Detail — HPR/DEI/PDI/GPI charts | Locked | Open |
| Detail — Benchmark vs Avg | Locked | Open |

#### Stays Behind Paywall
| Feature | Gate |
|---------|------|
| CSV Export | Subscription (€19/mo) |
| JSON Export | Subscription (€19/mo) |
| Historical data > 4 weeks | Subscription |
| AI-powered Reports | One-time (€49/€149) |

### Acceptance Criteria

- [x] DAO Matrix shows all 46 DAOs without login
- [x] Sorting and filtering work without login
- [ ] DAO Detail pages show all 5 charts without login
- [x] CSV/JSON Export stays behind subscription gate (🔒 icon)
- [ ] Report CTA banner on every DAO Detail page
- [x] "Sign in to unlock" removed from all content pages
- [x] All new analytics events fire correctly
- [x] Feature flag `OPEN_MATRIX` for quick rollback
- [ ] OG Meta tags per DAO page (can be follow-up)

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

## New Funnel

```
matrix_dao_click
  → dao_detail_view
    → dao_chart_interact
      → dao_detail_report_cta_view
        → dao_detail_report_cta_click
          → quick_check_start
            → quick_check_complete
              → checkout_complete
```

---

## SEO Benefit

46 DAO detail pages become indexable landing pages:
- `chainsights.one/matrix/radiant-capital` → "Radiant Capital Governance Score"
- `chainsights.one/matrix/ens` → "ENS Governance Score"
- `chainsights.one/matrix/lido` → "Lido Governance Score"

Each shareable with rich previews = organic reach multiplier.

---

## Out of Scope

- Comparison Mode (stays subscription feature)
- Share buttons on detail pages (future)
- Custom OG images per DAO (future)
- Changes to pricing/tiers
- Report generation or content

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

1. Add `OPEN_MATRIX` feature flag
2. Remove paywall logic (Matrix + Detail pages)
3. Add Report CTA banner on Detail pages
4. Update Export buttons with 🔒 icon
5. Implement new analytics events
6. Add OG meta tags per DAO
7. Deploy with flag = true
8. Monitor: traffic, engagement, conversions vs. before
