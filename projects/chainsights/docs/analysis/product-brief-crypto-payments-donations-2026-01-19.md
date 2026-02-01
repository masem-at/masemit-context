---
stepsCompleted: [1, 2, 3, 4, 5]
inputDocuments:
  - project-context.md
  - product-brief-chainsights-2025-12-15.md
workflowType: 'product-brief'
lastStep: 5
project_name: 'Crypto Payments + Donation Tracking'
user_name: 'Mario'
date: '2026-01-19'
initiative_type: 'feature_enhancement'
parent_product: 'ChainSights'
---

# Product Brief: Crypto Payments + Donation Tracking

**Date:** 2026-01-19
**Author:** Mario
**Initiative Type:** Feature Enhancement for ChainSights
**Parent Product:** ChainSights (DAO Governance Analytics)

---

## 🚨 CRITICAL DEVELOPMENT CONSTRAINT 🚨

> **ALL CODE CHANGES FOR THIS INITIATIVE MUST GO TO `develop` BRANCH ONLY!**
>
> - ❌ NO direct commits to `main` / `master`
> - ✅ ALL work on `develop` branch
> - ✅ Staging environment (Vercel) linked to `develop`
> - ✅ Only merge to `main` after full validation on staging
>
> **This is non-negotiable.**

---

## Initiative Context

**Background:**
ChainSights is a Web3 governance analytics platform selling reports to DAOs. Current payment method is Stripe (fiat only). As a Web3-focused product, accepting crypto payments aligns with audience expectations and reduces friction.

**Related Initiative:**
Mario commits 3% of all revenue to [hoki.help](https://hoki.help) — a children's hospice donation platform (Hospiz NÖ, Lower Austria) that Mario built with his son.

---

## Party Mode Discussion Summary (2026-01-19)

**Attendees:** Winston 🏗️, Mary 📊, Sally 🎨, Maya 📣, John 📋, Bob 🏃, Anna 🧾, + full team

**Key Decisions Made:**

### 1. Crypto Payment Approach (REVISED — Stripe Crypto NOT available in Austria)

**Original Plan (ABANDONED):**
- Stripe + Crypto.com integration → ❌ NOT available for Austrian merchants

**Revised Plan (CONFIRMED):**
- **Tool:** Coinbase Commerce (separate from Stripe)
- **Auto-conversion:** YES — crypto received → auto-convert to EUR → bank account
- **Rationale:** Stripe Crypto not available in AT; Coinbase Commerce works globally
- **Supported coins:** USDC, ETH, BTC, and others supported by Coinbase
- **Trade-off:** Two payment systems instead of one (accepted complexity)

### 2. Donation Tracking (3% to hoki.help)
- **Amount:** 3% of revenue (not profit) — ~€1.47 per €49 report
- **Tracking Method:**
  - Stripe: metadata (`donation_amount`, `project`)
  - Coinbase: webhook or dashboard export
- **Transfer Cadence:** Monthly batch transfer to Hospiz NÖ
- **Tax Handling:** Annual Spendenbestätigung for deduction (Anna confirmed)

### 3. Implementation Approach
- **Philosophy:** Accept necessary complexity to achieve goal
- **Two payment rails:** Stripe (fiat) + Coinbase Commerce (crypto)
- **Monthly process:** Query both sources → sum 3% → bank transfer to Hospiz

### 4. Donation Visibility (Phased)
1. **Phase 1:** Visible on checkout ("3% supports hoki.help")
2. **Phase 2:** `/impact` page with running totals
3. **Phase 3:** Public marketing around it

---

## Executive Summary

ChainSights is a Web3 governance analytics platform. Our customers — DAO contributors, delegates, and governance researchers — live in crypto. Yet we only accept fiat payments. This creates unnecessary friction and undermines our Web3 credibility.

This initiative adds crypto payment capability via **Coinbase Commerce** (Stripe Crypto is not yet available for Austrian merchants) and implements a 3% revenue donation to hoki.help, a children's hospice platform built by the founder and his son.

**Scope:**
- Add Coinbase Commerce for crypto payments (USDC, ETH, BTC)
- Auto-convert to EUR (no volatility risk)
- Track 3% donation per transaction via metadata (Stripe) and webhook (Coinbase)
- Monthly batch transfer to Hospiz NÖ
- Phased visibility: checkout badge → impact page → public marketing

**Reality Check:** Two payment systems creates more complexity than originally hoped, but this is the only viable path for Austrian merchants wanting crypto payments today.

---

## Core Vision

### Problem Statement

ChainSights sells governance analytics to Web3-native customers who hold and transact in cryptocurrency. Forcing them to convert crypto → fiat → pay via card creates friction, undermines trust, and signals we don't understand our own audience.

### Problem Impact

- **Conversion friction:** Potential customers who prefer crypto may abandon checkout
- **Credibility gap:** "Web3 analytics platform that doesn't accept crypto" is a bad look
- **Missed alignment:** Our audience expects us to operate like they do

### Why Stripe-Only Won't Work

Stripe's Crypto.com integration (announced January 2025) is NOT available for Austrian merchants:
- Only available in select countries (US, UK, core EU)
- Austria not in initial rollout
- Unknown timeline for availability

### Proposed Solution

**Two-Rail Payment Architecture:**

```
┌─────────────────┐     ┌─────────────────┐
│     STRIPE      │     │    COINBASE     │
│ Cards/Klarna/EPS│     │    COMMERCE     │
└────────┬────────┘     └────────┬────────┘
         │                       │
         ▼                       ▼
    EUR to Bank            Auto-convert EUR
         │                       │
         └───────────┬───────────┘
                     ▼
           DONATION TRACKING
         (3% calculation monthly)
                     │
                     ▼
            BANK TRANSFER
           to Hospiz NÖ
```

**Stripe (existing):**
- Cards, Klarna, EPS (Austrian bank transfers)
- Add `donation_amount` metadata to payments
- Query dashboard/API for monthly totals

**Coinbase Commerce (new):**
- USDC, ETH, BTC, and other supported coins
- Auto-converts to EUR
- Webhook to capture payment data OR manual dashboard export
- Calculate 3% monthly

**Implement 3% donation tracking:**
- Stripe: metadata field on each payment
- Coinbase: webhook logging or manual export
- Monthly: sum both sources → bank transfer to Hospiz NÖ
- Annual: request Spendenbestätigung for tax deduction

### Key Differentiators

1. **Values-aligned business:** 3% to children's hospice isn't marketing — it's who we are
2. **Founder authenticity:** Mario built hoki.help with his son; this is personal
3. **Web3-native UX:** Accept what our customers hold
4. **Pragmatic execution:** Accept complexity when necessary to achieve the goal
5. **Transparency:** Phased public visibility of donation impact

### Success Criteria

- ✅ Crypto payment option works reliably (via Coinbase Commerce)
- ✅ Fiat payments continue working (Stripe)
- ✅ Donations tracked accurately (both systems)
- ✅ Monthly transfer process documented and executable
- ✅ "3% to hoki.help" visible on checkout

No adoption rate targets. Success = it works.

---

## Target Users

### Primary Users

**Existing ChainSights Customers**
- DAO contributors, delegates, governance researchers
- Already use ChainSights for governance reports
- **Feature impact:** Additional payment flexibility; some may prefer crypto

**Crypto-Native Professionals (Previously Blocked)**
- Hold assets primarily in crypto, minimal fiat banking
- Wanted ChainSights reports but couldn't easily pay via card
- **Feature impact:** Removes payment barrier entirely — unlocks new buyers

### Secondary Users

**DAO Treasuries**
- DAOs paying for reports from treasury funds (multisig wallets)
- Prefer paying in USDC/stablecoins — no need to off-ramp to fiat
- **Feature impact:** Enterprise/B2B opportunity — treasury-funded purchases

**Values-Aligned Buyers**
- Customers who notice and appreciate the 3% donation commitment
- Trust signal: "This company puts money where their mouth is"
- **Feature impact:** May influence purchase decision at checkout

### User Journey (Feature-Specific)

1. **Discovery:** Customer reaches checkout
2. **Choice:** Sees "Pay with Card" and "Pay with Crypto" options
3. **Trust Signal:** Notices "3% supports hoki.help" badge
4. **Payment:** Completes payment via preferred method (Stripe or Coinbase)
5. **Confirmation:** Same confirmation flow regardless of payment method
6. **Impact (Later):** Can see cumulative donation on `/impact` page

---

## Success Metrics

*Reviewed and enhanced by full team in Party Mode*

### Definition of Done ("It Works")

A payment is successful when:
- ✅ Payment goes through (Stripe or Coinbase)
- ✅ Customer receives confirmation
- ✅ Money arrives in bank account
- ✅ Donation amount is tracked

### Functional Success (Must Have)

- ✅ Crypto payments via Coinbase Commerce work reliably
- ✅ Fiat payments via Stripe continue working
- ✅ Donations tracked accurately in both systems
- ✅ Monthly transfer process documented and executable
- ✅ "3% to hoki.help" badge visible on checkout

### Observability (Built-in)

| Metric | Where to Check |
|--------|----------------|
| Payment volume (fiat) | Stripe Dashboard |
| Payment volume (crypto) | Coinbase Commerce Dashboard |
| Payment method split | Manual comparison of both dashboards |

### Monitoring & Alerts

**Webhook Health Check:**
- Log timestamp of last successful Coinbase webhook
- Alert if no webhook received in 48h AND payments expected
- Implementation: `SELECT MAX(created_at) FROM webhook_logs WHERE provider='coinbase'`

**No complex thresholds** — simple uptime monitoring sufficient.

### Post-Deploy Validation (Smoke Test)

After each deployment to `develop`:
- [ ] Test payment with Stripe (€1 or test mode)
- [ ] Test payment with Coinbase (testnet or minimal amount)
- [ ] Verify webhook received
- [ ] Verify metadata/donation amount recorded correctly

### Monthly Reconciliation Process

**Runbook: Monthly Donation Transfer**
1. Open Stripe Dashboard → filter by month → note total
2. Open Coinbase Dashboard → filter by month → note total
3. Calculate: (Stripe + Coinbase) × 3% = donation amount
4. Bank transfer to Hospiz NÖ
5. Save screenshot/receipt for tax records

*Document created by Paige (Tech Writer)*

### Nice-to-Have (Phase 2)

**`/impact` Page:**
- Show cumulative donation total
- Pull data automatically from both Stripe + Coinbase
- **Maya's input:** Only display publicly when total reaches €500+ (smaller amounts feel less impactful)
- **Daniel's input:** Ensure number is calculated automatically, not manually entered (legal compliance)

### Post-Launch UX Validation (One-time)

- Review 10 checkout sessions in LogSnag/analytics
- Confirm users find and use the crypto payment option
- Identify any UX friction points
- Not ongoing tracking — just initial validation

### What We're NOT Tracking

- ❌ No crypto adoption rate targets
- ❌ No conversion rate goals
- ❌ No A/B testing of payment methods

**Success = it works. Period.**

---

## Scope

### MVP (Phase 1) — Must Have

| Feature | Description | Effort |
|---------|-------------|--------|
| Coinbase Commerce Integration | Add crypto payment provider | Medium |
| Crypto checkout modal | Modal-based payment flow (not redirect) | Medium |
| Donation tracking (Stripe) | Add `donation_amount` + `project` metadata | Small |
| Donation tracking (Coinbase) | Webhook to capture payment + donation data | Medium |
| Checkout donation badge | "3% helps families facing childhood illness" | Small |
| Confirmation donation message | "You just helped a family. €X supports hospice care" | Small |
| `/impact` page | Show cumulative donations (visible only after €500) | Small |

### Phase 2 — Nice to Have (Later)

| Feature | Description |
|---------|-------------|
| Monthly reconciliation automation | Script to auto-generate monthly donation report |
| Detailed donation history | Track individual donations over time |
| Public marketing campaign | Promote the 3% commitment externally |

### Out of Scope

- Complex analytics dashboard
- Multiple charity options
- Donor recognition/leaderboard
- Automatic Hospiz NÖ bank transfer (manual is fine)

### Technical Constraints

- **Branch:** ALL work on `develop` only
- **Staging:** Vercel deployment from `develop` branch
- **Production:** Only after full validation on staging

### Dependencies

- Coinbase Commerce account setup
- Coinbase API keys (test + production)
- Stripe metadata fields configured

---

## Next Steps

| Step | Owner | Output |
|------|-------|--------|
| 1 | Sally + Maya | UX Spec Document |
| 2 | Winston | Architecture Document |
| 3 | Bob | Epics & Stories |
| 4 | Amelia | Implementation on `develop` |
| 5 | Murat | Test Validation |
| 6 | Team | Staging Review → Merge to Main |

---

**Product Brief Status: ✅ COMPLETE**

*Created: 2026-01-19*
*Team: Full BMAD Party Mode Session*
