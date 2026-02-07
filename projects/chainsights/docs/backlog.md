# ChainSights — Active Backlog

> **SINGLE SOURCE OF TRUTH for all open work items.**
> Every agent MUST load this file before asking "what's next?"
> Update this file after completing items. Never let work items exist only in conversation.

**Last Updated:** 2026-02-06
**Updated By:** Mario + Claude (Barry/Dev, John/PM, Mary/Analyst, Paige/Writer)

---

## CRITICAL / Blockers

_(none currently)_

## HIGH Priority

- [ ] **DGI Benchmark in Reports** — Deep Dive (€49): headline comparison ("Your DAO scores 7.2 — DeFi average is 5.8"). Audit (€149): full "How You Compare" with percentiles, category breakdown, strengths/weaknesses vs peers.
- [x] ~~**GovernanceIndex Re-enabled**~~ — DONE 2026-02-06 (BUG-3 reversed, category averages displayed)
- [x] ~~**Free Check Rate Limit 10/h**~~ — DONE 2026-02-06 (`ANON_IP_LIMIT` 2→10)
- [x] ~~**Share & Save Reactivated**~~ — DONE 2026-02-06 (Mario enables `SHARE_REWARDS_ENABLED=true`)
- [x] ~~**Landing Page Engagement Tracking**~~ — DONE 2026-02-02
- [x] ~~**LogSnag Retry**~~ — DONE 2026-02-02
- [x] ~~**DGI Coming Soon Page**~~ — DONE 2026-02-02
- [x] ~~**DGI GVS Calculation**~~ — DONE 2026-02-02 (daily job already processed all 44 DAOs)
- [x] ~~**DGI Public Landing**~~ — DONE 2026-02-02 (`/dgi` redirects to `/governance-index`)
- [x] ~~**DGI Launch Runbook**~~ — DONE 2026-02-02 (`docs/launch-runbook-dgi.md`)
- [x] ~~**Stripe Payment Inline on /check**~~ — DONE 2026-02-02 (Embedded Checkout via `@stripe/react-stripe-js`)
- [x] ~~**Admin Dashboard Stats Cleanup**~~ — DONE 2026-02-02

## MEDIUM Priority

- [ ] **Spendenabwicklung auf MMS Service umstellen** — Lokale `donations`-Tabelle + Logik durch zentralen MMS Donations Service (`https://api.masem.at/docs/donations`) ersetzen. Betroffen: Stripe-Webhook (`handlePaymentIntentSucceeded`), Coinbase-Webhook (`handleChargeConfirmed`), `/api/donations/total`, `HeroDonationCounter`, `ImpactContent`, `DonationBadge`, `DonationImpactBox`. MMS übernimmt Berechnung (Prozentsatz), Speicherung, Target-Verteilung. ChainSights ruft `POST /v1/donations` statt lokaler DB-Insert auf und `GET /v1/donations/summary` für Totals. Env: `MMS_API_URL` + `MMS_API_KEY` bereits gesetzt. CHOKIHEART-Logik (5%) über `donation_percentage_override` abbildbar.
- [ ] **DAO S&P Governance Benchmark** — GovernanceIndex re-enabled (2026-02-06). Remaining: DGI Benchmark in Reports (moved to HIGH), public methodology page, Mirror whitepaper (Mario handles).
- [ ] **Mirror Account + Whitepaper** — Create chainsights.mirror.xyz. Publish DGI methodology whitepaper. Needed for Wikipedia citation path.
- [ ] **Social Media Assets** — LinkedIn + X images in `public/images/linkedin/` and `public/images/x/` (untracked). Finalize and commit.
- [x] ~~**DGI Launch Content**~~ — DONE 2026-02-06 (LinkedIn, X, Discord posts published by Mario)
- [x] ~~**Share & Save Re-evaluation**~~ — DONE 2026-02-06 (Decision: reactivate. Mario enables flag.)

## LOW Priority / Backlog

- [ ] **API Access (Phase 3)** — Public API for researchers. Not built.
- [ ] **Farcaster Frames** — Interactive DGI ranking frames. Explore feasibility.
- [ ] **Product Hunt Launch** — After community is established.
- [ ] **Wikipedia Entry** — After 2-3 independent secondary sources cite DGI. Earliest Q2 2026.
- [ ] **Monthly DGI Reports on Mirror** — Recurring content, start month 2 post-launch.

## DONE (Recently Completed)

- [x] GovernanceIndex re-enabled on Matrix Detail pages — BUG-3 reversed, category averages displayed — 2026-02-06
- [x] Free Check Rate Limit increased to 10/IP/hour — was 2, now 10 since Matrix is free — 2026-02-06
- [x] Share & Save reactivated — Mario enables `SHARE_REWARDS_ENABLED=true` — 2026-02-06
- [x] DGI Launch Content — LinkedIn, X, Discord posts published — 2026-02-06
- [x] Farcaster URL field in Engagement Hub — DB column, table icon, edit dialog, API, report platform — 2026-02-06
- [x] Dynamic Sitemap with DAO pages — `/matrix/{slug}` entries, ISR 24h revalidation — 2026-02-06
- [x] Bing IndexNow on build — `scripts/indexnow.mjs`, production-only, non-fatal — 2026-02-06
- [x] Feedback Bubble Email Fix — await Resend call (serverless fire-and-forget bug) — 2026-02-06
- [x] Engagement Report current week picker — loop starts at i=0 instead of i=1 — 2026-02-06
- [x] Admin Dashboard Stats Cleanup — removed unused featuredDAOs prop, badge shows total DAO count — 2026-02-02
- [x] Stripe Payment Inline — Embedded Checkout replaces Hosted Checkout redirect on /check — 2026-02-02
- [x] DGI Launch Runbook — `docs/launch-runbook-dgi.md` with pre-launch, launch day, post-launch, rollback — 2026-02-02
- [x] DGI Public Landing — `/dgi` redirects to `/governance-index` for shareable launch URLs — 2026-02-02
- [x] DGI Coming Soon Page — branded teaser with email capture replaces silent redirect/404 — 2026-02-02
- [x] DGI GVS Calculation — all 44 DAOs scored by daily job, no manual action needed — 2026-02-02
- [x] LogSnag Retry — 1x retry in catch block for Vercel cold-start SocketError drops — 2026-02-02
- [x] Landing Page Engagement Tracking — scroll depth, time-on-page, section visibility via EngagementTracker component — 2026-02-02
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
