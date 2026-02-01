# Epic: /check Flow Page — Modal-Free UX Refactor

**Date:** 2026-02-01
**Status:** IMPLEMENTED
**Source:** `docs/_masemIT/bugs/issue-free-check-flow-page-refactor.md`

## Summary

Replaced 4-5 chained modals (DAO select, tier select, email, Stripe, results) with a single `/check` page using progressive disclosure. All CTAs site-wide now redirect to `/check` instead of opening modals.

## Stories

### Story 1-5: /check Page + Progressive Disclosure Engine

- Created `src/app/check/page.tsx` (Server Component)
- Created `src/app/check/check-flow-client.tsx` (Client Component — main flow)
- Created `src/app/check/tier-cards.tsx` (Tier selection cards)
- Created `src/app/check/results-section.tsx` (Step 4 results display)
- Created `src/app/check/types.ts` (Shared QuickCheckResult type)

**State Machine:** SELECT_DAO -> SELECT_TIER -> YOUR_DETAILS -> YOUR_RESULTS
Each step: LOCKED | ACTIVE | COMPLETED

**Features:**
- Progressive disclosure with smooth-scroll between steps
- URL params: `?dao=lido-snapshot.eth` for deep-links
- Auto-confirm DAO from URL param
- Session detection: admin (@masem.at) skips Stripe, email pre-filled
- SSE streaming for on-the-fly DAO analysis
- Tier cards: Green (Free), Yellow (Deep Dive), Red (Governance Audit)
- Upsell from Free results to Deep Dive

### Story 6: CTA Migration

Replaced all 8 modal-opening CTAs with `router.push('/check')`:
- `src/components/hero.tsx`
- `src/components/hero/HeroVariantA.tsx`
- `src/components/hero/HeroVariantB.tsx`
- `src/components/header.tsx`
- `src/app/pricing/pricing-cards.tsx`
- `src/components/ExpandableDAORow.tsx` (uses `?dao=` param)
- `src/components/sample-report.tsx`

### Story 7: Modal Cleanup

Deleted:
- `src/components/report-selection-modal.tsx`
- `src/components/quick-check/QuickCheckEmailModal.tsx`
- `src/components/quick-check/QuickCheckResultsModal.tsx`

Updated `src/components/quick-check/index.ts` to remove deleted exports.

### Story 8: Analytics Events

Added 11 new `check_*` event types to `src/lib/analytics.ts` with typed props interfaces:
- `check_page_view`, `check_step_completed`, `check_step_changed`
- `check_dao_selected`, `check_tier_selected`
- `check_email_entered`, `check_payment_started`, `check_payment_completed`, `check_payment_failed`
- `check_result_shown`, `check_upgrade_clicked`

## Stripe Architecture

Kept Stripe Hosted Checkout for paid tiers. The `/check` page collects email in Step 3, then redirects to Stripe Checkout via existing `POST /api/checkout`. Returns to `/success` page.

**Backlog:** Inline Stripe Payment Element (replace redirect with embedded payment form).

## Verification Checklist

- [ ] Free Check (known DAO): `/check` -> select DAO -> Free -> email -> instant results
- [ ] Free Check (unknown DAO): enter space -> Free -> email -> SSE stream in Step 4
- [ ] Deep Dive: select DAO -> Deep Dive -> email -> Stripe checkout -> /success
- [ ] Admin: session detected -> email pre-filled -> no Stripe -> instant results
- [ ] Deep links: `/check?dao=aave.eth` -> Step 1 pre-filled, auto-advance
- [ ] Mobile: all steps render vertically
- [ ] No old links: search for `ReportSelectionModal` -> 0 source imports
- [ ] Events: all `check_*` events firing
