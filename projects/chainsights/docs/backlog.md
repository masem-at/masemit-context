# ChainSights — Active Backlog

> **SINGLE SOURCE OF TRUTH for all open work items.**
> Every agent MUST load this file before asking "what's next?"
> Update this file after completing items. Never let work items exist only in conversation.

**Last Updated:** 2026-02-02
**Updated By:** Mario + Claude

---

## CRITICAL / Blockers

_(none currently)_

## HIGH Priority

- [ ] **Landing Page Engagement Tracking** — Scroll depth, time-on-page, section visibility. Requirement: `docs/_masemIT/requirements/feature-landing-engagement-tracking.md`. Needed to diagnose why 96% of visitors bounce from landing page.
- [ ] **LogSnag Retry** — Server-side fetch to LogSnag API drops with SocketError on Vercel cold starts. Add 1x retry in `src/lib/logsnag.ts`.
- [ ] **DGI Coming Soon Page** — Soft-landing for direct URL access to gated DGI pages (instead of silent redirect). Optional: email capture for launch notification.
- [ ] **DGI GVS Calculation** — Run initial GVS for all DGI DAOs (new ones have no scores). Verify daily job picks them up.
- [ ] **DGI Public Landing** — Decide: dedicated `/dgi` page or just show on `/rankings`? Need a shareable URL for the launch campaign.
- [ ] **DGI Launch Runbook** — Extract launch checklist from story into standalone `docs/launch-runbook-dgi.md` with pre-launch, launch day, post-launch, rollback sections.
- [ ] **Stripe Payment Inline on /check** — Currently uses Stripe Hosted Checkout (redirect). Plan was inline Payment Element in Step 3. Status: deferred, Hosted Checkout works but is less seamless.
- [ ] **Admin Dashboard Stats Cleanup** — `featuredDAOs` prop still passed to dashboard. Clean up unused props now that featured/DGI are consolidated.

## MEDIUM Priority

- [ ] **DAO S&P Governance Benchmark** — Define benchmark methodology (category averages, inclusion criteria, weighting). Blocked by: Mario's methodology decisions. See `docs/_masemIT/requirements/chainsights-bugs-auth-backlog.md` Part 3.
- [ ] **Mirror Account + Whitepaper** — Create chainsights.mirror.xyz. Publish DGI methodology whitepaper. Needed for Wikipedia citation path.
- [ ] **Social Media Assets** — LinkedIn + X images in `public/images/linkedin/` and `public/images/x/` (untracked). Finalize and commit.
- [ ] **DGI Launch Content** — Fill in X thread template placeholders with real data (scores, insights). See `docs/_masemIT/requirements/chainsights-dgi-promotion-strategy.md`.
- [ ] **Share & Save Re-evaluation** — Currently disabled. Decide: reactivate, modify, or remove code entirely?

## LOW Priority / Backlog

- [ ] **API Access (Phase 3)** — Public API for researchers. Not built.
- [ ] **Farcaster Frames** — Interactive DGI ranking frames. Explore feasibility.
- [ ] **Product Hunt Launch** — After community is established.
- [ ] **Wikipedia Entry** — After 2-3 independent secondary sources cite DGI. Earliest Q2 2026.
- [ ] **Monthly DGI Reports on Mirror** — Recurring content, start month 2 post-launch.

## DONE (Recently Completed)

- [x] Admin DGI Bypass — DGI gates skip for admin role (server + client), Mario can test production-like — 2026-02-02
- [x] Fix cta_click tracking — trackAndNavigate() prevents race condition where router.push kills XHR — 2026-02-02
- [x] DGI Navigation Links + Event Tracking — Header, Footer, Rankings cross-link, all gated behind feature flag — 2026-02-02
- [x] DGI Hidden Until Launch — feature flag `DGI_PUBLIC`, guards on 5 pages/routes — 2026-02-02
- [x] Anon IP Rate Limiting — 2/IP/hour for anonymous `/api/quick-check` — 2026-02-02
- [x] Admin GVS Trigger — Recalc GVS buttons (per-DAO + all) on `/admin/daos` — 2026-02-02
- [x] Consolidate admin DAO management (`/admin/daos`) — 2026-02-02
- [x] Bulk DAO import script + DGI CSV — 2026-02-01
- [x] Unified admin auth (magic link only) — 2026-02-01
- [x] `/check` Flow Page refactor (modals → progressive disclosure) — 2026-02-01
- [x] Open-Universe Free Check (any Snapshot space) — 2026-02-01
- [x] Magic Link Authentication — 2026-02-01
- [x] DAO Governance Index (DGI) Epics 1-4 — 2026-02-01

---

## Notes

- This file MUST be committed to git and kept up-to-date.
- Agents: read `docs/backlog.md` + `docs/project_context.md` at session start.
- Mario: add items here directly when you think of them. This is YOUR list.
