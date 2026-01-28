# Response to BMAD Team - Naming & Fixes

**From:** Mario (Product Owner)
**Date:** 2026-01-28

---

## 1. Naming Decision ✅

Sally's approach is right, but with adjustments:

| Internal Name | User-Facing Name | Price | CTA Text |
|---------------|------------------|-------|----------|
| `quick_check` | **Free Check** | €0 (email required) | "Get Free Check" |
| `deep_dive` | **Deep Dive** | €49 | "Get Deep Dive – €49" |
| `governance_audit` | **Governance Audit** | €149 | "Get Governance Audit – €149" |

**Why "Deep Dive" instead of "Governance Report"?**
- More memorable and differentiated
- "Governance Report" is generic – anyone can make a "report"
- "Deep Dive" implies thoroughness and exclusivity

**Rules:**
- Use these exact names EVERYWHERE
- CTAs always include the price (except Free Check)
- Never mix: "Order a report", "Get your report", etc.

**Amelia:** Please grep for these strings and replace:
- "Quick Check" → "Free Check" (user-facing only, internal code can stay `quick_check`)
- "Order a report" → Remove or replace with tier-specific CTA
- "Get your report" → Replace with tier-specific CTA
- Any "€99" references → "€49"

---

## 2. Share & Save – Feature Flag ✅

**Decision:** Hide temporarily, don't delete.

Winston is right. Create a constant:

```typescript
// src/lib/config/features.ts
export const FEATURES = {
  SHARE_REWARDS_ENABLED: false,
} as const
```

**Touchpoints to update (per Amelia's list):**
- `src/app/success/page.tsx`
- `src/lib/email/index.ts` (getReportDeliveryEmailHtml, getShareReminderEmailHtml)
- `src/app/api/jobs/share-reminder/route.ts`
- Report PDF template (the "Share & Save: Get EUR20 Back" section)

**Why temporary?** The idea is good, but we need to validate that people actually buy first. Once funnel works, we reactivate Share & Save as a growth lever.

---

## 3. Quiz Link – Go to Report Modal ✅

**Decision:** Quiz "Let us help" should link to the Report Selection Modal, NOT the landing page.

**Why?** User just completed the quiz → they're engaged and interested. Don't send them back to start. Direct path to conversion.

**Implementation:** Link to `/rankings?openModal=true` or trigger modal directly (however the modal is currently implemented).

---

## 4. Data Issues – BUGS 🐛 (for Murat)

**Point 2 - No donations entries: BUG CONFIRMED**

I DID use CHOKIHEART promo on both test purchases. There should be 2 entries in the donations table, but there are 0.

→ **Webhook is likely not firing correctly** or the donation tracking logic in `handlePaymentIntentSucceeded` has a bug. Please investigate urgently.

**Point 3 - No quick check entries: BUG CONFIRMED**

I did multiple Quick Checks during testing. There should be entries, but there are 0 in both tables.

→ **Question:** Which table should Quick Check write to?
- `free_query_log` (old)?
- `quick_checks` (new)?

Either way, something is broken – no entries in either table after multiple tests.

**⚠️ PRIORITY: These bugs must be fixed before launch:**
1. No Quick Check data = no leads = the whole point of Free Check is broken
2. No donation tracking = can't track hoki.help contributions from CHOKIHEART promo

---

## 5. Action Plan Approved ✅

Mary's phased approach is correct:

**Phase 1: DECISIONS** ← Done (see above)

**Phase 2: AUDIT**
- [ ] Grep all hardcoded product names
- [ ] List all Share & Save touchpoints  
- [ ] Verify donations webhook
- [ ] Confirm quick_checks table structure

**Phase 3: IMPLEMENTATION**
- [ ] Create `src/lib/config/features.ts` with SHARE_REWARDS_ENABLED
- [ ] Create `src/lib/config/products.ts` with naming constants
- [ ] Replace all hardcoded strings
- [ ] Fix quiz CTA link → modal
- [ ] Test all flows (Free Check, Deep Dive, Governance Audit)

---

## Summary

| Question | Decision |
|----------|----------|
| Naming | Free Check / Deep Dive (€49) / Governance Audit (€149) |
| Share & Save | Feature flag `SHARE_REWARDS_ENABLED = false`, hide everywhere |
| Quiz link | → Report Selection Modal, not landing page |

Let's get this tight before we announce. Thanks team! 🚀
