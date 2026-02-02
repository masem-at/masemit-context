# ChainSights — Active Backlog

> **SINGLE SOURCE OF TRUTH for all open work items.**
> Every agent MUST load this file before asking "what's next?"
> Update this file after completing items. Never let work items exist only in conversation.

**Last Updated:** 2026-02-02
**Updated By:** Mario + Claude

---

## CRITICAL / Blockers

- [ ] **DGI Hidden Until Launch** — DGI content (rankings filter, DGI badge, DGI category columns) must NOT be visible to public users. Admin-only until launch day. Add feature flag or env var `DGI_PUBLIC=false`. On launch day: flip to `true`. Affects: `/rankings` page, DAO detail pages, any public-facing DGI references.
- [ ] **Anon IP Rate Limiting** — Anonymous users (no session) can hammer the API. Need IP-based rate limiting for `/api/quick-check` and other public endpoints. Current rate limiting is email-based only (5/email/day for on-the-fly). Anon users bypass this entirely.
- [ ] **Admin DAO Management — GVS Trigger** — New DAOs added via bulk-add have NO GVS scores. Need "Trigger GVS Recalc" button on `/admin/daos` or auto-trigger after bulk-add.

## HIGH Priority

- [ ] **DGI GVS Calculation** — Run initial GVS for all DGI DAOs (new ones have no scores). Verify daily job picks them up.
- [ ] **DGI Public Landing** — Decide: dedicated `/dgi` page or just show on `/rankings`? Need a shareable URL for the launch campaign.
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
