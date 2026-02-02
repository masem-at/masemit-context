# DGI Launch Runbook

> Standalone checklist for launching the DAO Governance Index publicly.
> All code is production-ready. Launch = feature flag flip + marketing.

**Created:** 2026-02-02
**Owner:** Mario

---

## Pre-Launch (48h before)

- [ ] Verify daily GVS job ran successfully for ≥2 consecutive days (check `/admin` job logs)
- [ ] Confirm ≥40 DAOs in `daoIndexMembership` with confidence ≥ medium
- [ ] Test as non-admin: `/governance-index` shows Coming Soon, `/methodology/dgi` shows Coming Soon
- [ ] Test as admin: both pages render full content
- [ ] Set `DGI_LAUNCH_DATE` const in `src/components/header.tsx` to actual launch date (controls "NEW" badge expiry)
- [ ] Prepare marketing content (X thread, LinkedIn, Discord — see `docs/_masemIT/requirements/chainsights-dgi-promotion-strategy.md`)
- [ ] Verify `/dgi` redirects to `/governance-index` (shareable short URL for campaigns)

## Launch Day

### 1. Flip Feature Flags (Vercel Dashboard → Settings → Environment Variables)

Set **both** to `true` and redeploy:

| Variable | Value |
|----------|-------|
| `DGI_PUBLIC` | `true` |
| `NEXT_PUBLIC_DGI_PUBLIC` | `true` |

Trigger redeploy (or push any commit).

### 2. Verify (within 5 minutes)

- [ ] `/governance-index` — DGI dashboard loads publicly
- [ ] `/methodology/dgi` — methodology page loads publicly
- [ ] `/dgi` — redirects to `/governance-index`
- [ ] `/api/dgi` — returns JSON with composite score + categories
- [ ] Header: "Governance Index" link visible with "NEW" badge
- [ ] Footer: "Governance Index" + "DGI Methodology" links visible
- [ ] Rankings page: DGI cross-link banner visible
- [ ] Matrix detail pages: DGI benchmark lines visible on charts
- [ ] Mobile: nav links render correctly

### 3. Publish Content

Recommended timing: Tuesday or Wednesday, 14:00–16:00 UTC.

Order:
1. X thread (EN)
2. LinkedIn post (EN)
3. Farcaster cast
4. Discord announcements (DAO-specific)
5. LinkedIn (DE)
6. X (DE)

### 4. Monitor

- [ ] Check Vercel logs for errors after deploy
- [ ] Verify daily-recalculation cron runs at 00:00 UTC next day
- [ ] Check analytics for `cta_click` events on new DGI nav links

## Post-Launch (Week 1)

- [ ] Daily DGI calculations running without errors
- [ ] Start 4-week content rotation:
  - Week 1: Movers & shakers
  - Week 2: Category deep dive
  - Week 3: Biggest mover story
  - Week 4: Methodology / insight
- [ ] Monitor engagement: page views on `/governance-index`, subscribe conversions
- [ ] Respond to community feedback (Discord, X)

## Rollback

If critical issues are found after launch:

1. Set `DGI_PUBLIC=false` and `NEXT_PUBLIC_DGI_PUBLIC=false` in Vercel
2. Redeploy
3. DGI pages revert to Coming Soon, nav links hide, API returns 404
4. Admin access remains unaffected

No data loss — DGI snapshots continue being calculated daily regardless of public visibility.

---

## Reference

| Resource | Path |
|----------|------|
| Feature flags | `src/lib/feature-flags.ts` |
| DGI calculation | `src/lib/dgi/calculate.ts` |
| DGI queries | `src/lib/dgi/queries.ts` |
| Daily cron job | `src/app/api/jobs/daily-recalculation/route.ts` |
| DGI API | `src/app/api/dgi/route.ts` |
| Coming Soon page | `src/components/dgi/DGIComingSoon.tsx` |
| Promotion strategy | `docs/_masemIT/requirements/chainsights-dgi-promotion-strategy.md` |
| Admin DGI management | `/admin/dgi` |
