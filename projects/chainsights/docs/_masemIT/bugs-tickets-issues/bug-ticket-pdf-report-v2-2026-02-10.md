# BUG TICKET v2: Governance Audit PDF — Remaining Issues

**Priority:** High (blocks Lido forum outreach)
**Affected:** PDF Report Generation Pipeline
**Report Reviewed:** Lido Governance Audit (Generated: February 10, 2026, post-fix)
**Reporter:** Mario
**Predecessor:** bug-ticket-pdf-report-issues-2026-02-10.md

---

## Fixed Since v1

- ✅ BUG-PDF-2: Letter Grade now shows "4/10 (C)" on Page 2
- ✅ BUG-PDF-1: Governance Attention Analysis chapter added (Page 10)
- ✅ BUG-PDF-4/5: Footer/header naming mostly consistent now

---

## Open Bugs

### BUG-PDF-6: Attention Analysis — VOTERS and TOP 5 CONCENTRATION columns empty

**Severity:** BLOCKER
**Page:** 10

**Expected:** Each category row shows the number of unique voters and the top 5 delegate concentration percentage, matching the data available on the Matrix Details web page.

**Actual:** All 8 category rows show "—" for both VOTERS and TOP 5 CONCENTRATION columns. The table looks broken — only PROPOSALS and ATTENTION % have values.

**Impact:** This is the feature we're about to showcase to Lido governance contributors. A table full of dashes makes the report look unfinished and undermines credibility. **This blocks the Lido forum outreach.**

**Likely Cause:** The PDF generation pulls from `category_attention` table but the `unique_voters` and `delegate_concentration` fields are either not populated or not mapped to the PDF template.

**Fix:** Ensure the aggregation job populates all fields in `category_attention`, and the PDF template renders them. If the data genuinely isn't available yet, remove the columns from the PDF rather than showing dashes.

---

### BUG-PDF-7: Score confusion — Decentralization Score (4/10) vs GVS (6.9)

**Severity:** Medium
**Pages:** 1, 2, 7

**Expected:** The reader understands what each score means and why they differ.

**Actual:**
- Page 1 (Cover): "4/10 Decentralization Score"
- Page 2: "Decentralization Score 4/10 (C)"
- Page 7: "GVS 6.9 vs DeFi Average 5.9"

The report uses two different scoring systems without explaining the relationship. A reader will ask: "Is my score 4 or 6.9?"

**Fix:** Add a short explanation on Page 2, e.g.: "The Decentralization Score (4/10) measures power distribution specifically. The Governance Vitality Score (GVS: 6.9) is a composite score across all governance dimensions including participation, delegate engagement, and process quality."

Alternatively, add a small score summary box on Page 1 or 2 showing both scores side by side with labels.

---

### BUG-PDF-8: Attention Analysis — no visualization, just a table

**Severity:** Medium
**Page:** 10

**Expected:** The Governance Attention Analysis includes a treemap or visual chart matching the website experience, showing proportional governance attention distribution at a glance.

**Actual:** Only a flat text table. No treemap, no bar chart, no visual hierarchy. On the website this is a treemap — in the PDF it's just rows and columns.

**Impact:** The treemap is the signature visual of this feature. Without it, the PDF chapter feels like raw data rather than an insight. The visual "oh, treasury only gets 6%!" moment is lost.

**Fix:** Add a simple visualization above the table — either a horizontal stacked bar chart (easiest to render in PDF) or a treemap. The chart should show proportional category sizes with color coding.

---

### BUG-PDF-9: Page 10 footer inconsistent

**Severity:** Low
**Page:** 10

**Expected:** Footer reads "ChainSights Governance Audit Page 8" (sequential page number).

**Actual:** Footer reads "ChainSights Governance Audit Governance Attention Analysis" — uses section title instead of page number. Inconsistent with all other pages.

**Fix:** Use sequential page number like all other pages.

---

### BUG-PDF-10: Strategic Recommendations formatting still tight

**Severity:** Low
**Page:** 11

**Expected:** Recommendation cards have clear visual separation with comfortable spacing.

**Actual:** Cards are visually cramped. Not overlapping anymore (fixed from v1), but spacing between cards is minimal. The "Example:" lines run close to the next card's header.

**Fix:** Increase vertical padding between recommendation cards by ~10-15px. Low priority — functional but not polished.

---

## Priority Order

| # | Bug | Severity | Blocks Lido? | Effort |
|---|-----|----------|-------------|--------|
| 1 | **BUG-PDF-6: Empty VOTERS/CONCENTRATION columns** | BLOCKER | YES | ~2h |
| 2 | **BUG-PDF-7: Score confusion (4/10 vs 6.9)** | Medium | No, but confusing | ~1h |
| 3 | **BUG-PDF-8: No visualization in Attention chapter** | Medium | No, but weak | ~3h |
| 4 | **BUG-PDF-9: Page 10 footer** | Low | No | ~15min |
| 5 | **BUG-PDF-10: Recommendations spacing** | Low | No | ~30min |

**Must fix before Lido outreach:** BUG-PDF-6
**Should fix before Lido outreach:** BUG-PDF-7, BUG-PDF-8
**Nice to have:** BUG-PDF-9, BUG-PDF-10

**Total estimated effort:** ~6-7h

---

## Context: Why This Matters Now

We have a ready-to-post Lido forum reply offering free Governance Audit reports (€149 value) to 4 community contributors who helped shape the Attention Map feature. The forum thread momentum has an expiration date. Sending a report with empty data columns or confusing scores would damage credibility at exactly the wrong moment.

**Minimum bar for Lido outreach:** BUG-PDF-6 fixed, BUG-PDF-7 at least acknowledged in a footnote if not fully resolved.
