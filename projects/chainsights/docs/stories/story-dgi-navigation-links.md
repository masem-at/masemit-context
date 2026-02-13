# Story: DGI Navigation Links + Event Tracking

**Epic:** DGI Launch Readiness
**Priority:** BLOCKER — without this, DGI pages are unreachable for users
**Estimated Effort:** 2-3h
**Status:** Ready for Dev

---

## Context

DGI pages exist and are gated behind `DGI_PUBLIC` feature flag. But there are **zero navigation links** pointing to them. Users arriving from marketing campaigns (X threads, LinkedIn posts) have no way to discover DGI content on the site.

All new links MUST be gated behind the same `isDgiPublic()` flag so they appear automatically on launch day when `DGI_PUBLIC=true`.

**Problem:** Header is a Client Component (`'use client'`). `isDgiPublic()` reads `process.env.DGI_PUBLIC` which is server-only (no `NEXT_PUBLIC_` prefix). Solution: pass `dgiPublic` as a prop from the Server Component layout, OR use `NEXT_PUBLIC_DGI_PUBLIC` so the client can read it.

**Recommended approach:** Add `NEXT_PUBLIC_DGI_PUBLIC` env var (mirrors `DGI_PUBLIC`). Header and Footer read it client-side. This avoids prop-drilling through layout.

---

## Tasks

### Task 1: Add `NEXT_PUBLIC_DGI_PUBLIC` env var

**Files:**
- `src/lib/feature-flags.ts` (MODIFY)
- `.env.example` (MODIFY)

**AC:**
- AC1: Add `isDgiPublicClient()` function that reads `process.env.NEXT_PUBLIC_DGI_PUBLIC === 'true'` — safe for Client Components
- AC2: Existing `isDgiPublic()` stays unchanged (server-only, used by pages/routes)
- AC3: `.env.example` updated with `NEXT_PUBLIC_DGI_PUBLIC=false`
- AC4: Add to Vercel env vars (Mario does this manually, same as `DGI_PUBLIC`)

### Task 2: Header — Add "Governance Index" nav link

**File:** `src/components/header.tsx` (MODIFY)

**AC:**
- AC1: New entry in `NAV_LINKS` array: `{ href: '/governance-index', label: 'Governance Index', isDgi: true }`
- AC2: Position: after "Rankings", before "DAO Matrix"
- AC3: When `isDgiPublicClient()` returns `false`, links with `isDgi: true` are filtered out of rendered nav
- AC4: DGI link gets "NEW" badge (same pattern as Matrix — auto-expires after 14 days). Set `DGI_LAUNCH_DATE` const (leave as `new Date('2026-03-01')` placeholder — Mario updates on launch day)
- AC5: Works in both desktop nav AND mobile hamburger menu (same `NAV_LINKS` array drives both)
- AC6: `handleNavClick` fires `trackEvent('cta_click', { button_name: 'nav_governance_index', location: 'header' })`  — already works via existing pattern at line 57-62

### Task 3: Footer — Add DGI links

**File:** `src/components/footer.tsx` (MODIFY)

**AC:**
- AC1: Add "Governance Index" link to the "Resources" section (next to Quiz, Guide): `<Link href="/governance-index">`
- AC2: Add "DGI Methodology" link to the "Governance" section (next to Methodology, Opt-Out): `<Link href="/methodology/dgi">`
- AC3: Both links conditionally rendered: `{isDgiPublicClient() && (<Link ...>)}`
- AC4: Import `isDgiPublicClient` from `@/lib/feature-flags`

### Task 4: Rankings page — DGI cross-link banner

**File:** `src/app/rankings/page.tsx` (MODIFY)

**AC:**
- AC1: When `isDgiPublic()` is true, show a subtle banner/link above or below the rankings table: "Explore the DAO Governance Index — ecosystem benchmarks for 46 DAOs →" linking to `/governance-index`
- AC2: When `isDgiPublic()` is false, nothing shown
- AC3: This is a Server Component — use `isDgiPublic()` (server-side)
- AC4: `trackEvent` not needed here (server component, no client interactivity on link)

### Task 5: Analytics event tracking verification

**AC:**
- AC1: Header nav click already tracked via `handleNavClick` pattern (line 57-62) — verify new DGI link fires `cta_click` with `button_name: 'nav_governance_index'`
- AC2: No new analytics overloads needed — existing `CTAClickEventProps` with `button_name` covers it

---

## Verification Checklist

With `NEXT_PUBLIC_DGI_PUBLIC=false` / `DGI_PUBLIC=false`:
- [ ] Header: NO "Governance Index" link visible (desktop + mobile)
- [ ] Footer: NO DGI links visible
- [ ] Rankings: NO DGI banner visible
- [ ] All existing pages work unchanged

With `NEXT_PUBLIC_DGI_PUBLIC=true` / `DGI_PUBLIC=true`:
- [ ] Header: "Governance Index" link visible between Rankings and DAO Matrix, with NEW badge
- [ ] Header: Link active state works on `/governance-index`
- [ ] Header mobile: Link visible in hamburger menu
- [ ] Footer: "Governance Index" in Resources section
- [ ] Footer: "DGI Methodology" in Governance section
- [ ] Rankings: DGI cross-link banner visible
- [ ] Click on header DGI link fires analytics event
- [ ] `/governance-index` page loads correctly

---

## Launch Day Checklist

1. Set `DGI_PUBLIC=true` in Vercel
2. Set `NEXT_PUBLIC_DGI_PUBLIC=true` in Vercel
3. Update `DGI_LAUNCH_DATE` in header.tsx to actual launch date (for NEW badge expiry)
4. Redeploy
5. All links appear automatically
