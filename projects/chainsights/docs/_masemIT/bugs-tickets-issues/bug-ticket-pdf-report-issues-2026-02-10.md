# BUG TICKET: Governance Audit PDF Report — Multiple Issues

**Priority:** High
**Affected:** PDF Report Generation Pipeline
**Report Reviewed:** Lido Governance Audit (Generated: February 10, 2026)
**Reporter:** Mario

---

## Summary

The Governance Audit PDF for Lido has multiple issues: missing content that exists on the website, layout bugs, and missing UI elements. These need to be fixed before sending reports to customers or sharing with the Lido forum community.

---

## Bug List

### BUG-PDF-1: Governance Attention Map / Treemap Missing

**Severity:** High
**Page:** N/A (completely absent)

**Expected:** The Governance Attention Map (proposal categorization treemap) is live on the Matrix Details web page. The Governance Audit PDF (€149) should include a dedicated chapter "Governance Attention Analysis" showing:
- Treemap visualization of governance topics (Treasury, Protocol Upgrades, Grants, Operations, etc.)
- Participation rate per category
- Delegate concentration per category
- Rational Apathy indicators (categories with significantly below-average participation)

**Actual:** The Attention Map chapter is completely missing from the PDF. No proposal categorization data is included.

**Fix:** Add new PDF section/chapter that pulls the Attention Map data from the existing classification pipeline and renders it in the report. Should be placed after "How You Compare" and before "Historical Trends Analysis".

---

### BUG-PDF-2: Rating Letter Grade Missing

**Severity:** Medium
**Page:** 1 (Cover/Executive Summary)

**Expected:** The Decentralization Score (3/10) should include a letter grade (e.g., A, B, C, D, F) alongside the numeric score, consistent with website display.

**Actual:** Only shows "3/10" and "Stable" label. No letter grade visible.

**Fix:** Add letter grade display next to the numeric score on the cover page.

---

### BUG-PDF-3: Text Overlap on Strategic Recommendations Page

**Severity:** High
**Page:** 8 (Strategic Recommendations)

**Expected:** All recommendation cards should be properly spaced with clear separation between items. Each recommendation should have readable text without overlap.

**Actual:** Text elements overlap each other, making the recommendations partially unreadable. Layout/spacing breaks on this page.

**Fix:** Review spacing calculations in the PDF generation for the recommendations section. Likely a fixed-height issue — recommendations with longer text overflow into the next card. Needs dynamic height calculation or increased spacing between cards.

---

### BUG-PDF-4: Inconsistent Page Numbering

**Severity:** Low
**Pages:** Throughout

**Expected:** Consistent footer format across all pages.

**Actual:** Some pages show "ChainSights Governance Truth Report Page X" and others show "ChainSights Governance Audit Page X". Should be consistent — "Governance Audit" for the €149 tier.

**Fix:** Standardize footer to "ChainSights Governance Audit" across all pages.

---

### BUG-PDF-5: Cover Page Label Mismatch

**Severity:** Low  
**Page:** 1

**Expected:** Cover says "Governance Audit Report" (correct for €149 tier).

**Actual:** Page 2 header says "Governance Truth Report" instead of "Governance Audit Report".

**Fix:** Align page 2 header with cover page — should say "Governance Audit Report" or just "Governance Audit".

---

## Priority Order

| # | Bug | Severity | Effort | Do First? |
|---|-----|----------|--------|-----------|
| 1 | BUG-PDF-1: Attention Map missing | High | Medium (~4h) | Yes — new feature value |
| 2 | BUG-PDF-3: Text overlap on recommendations | High | Small (~1-2h) | Yes — readability |
| 3 | BUG-PDF-2: Letter grade missing | Medium | Small (~30min) | Yes — quick fix |
| 4 | BUG-PDF-4: Inconsistent page footer | Low | Small (~30min) | Nice to have |
| 5 | BUG-PDF-5: Cover/header mismatch | Low | Small (~15min) | Nice to have |

**Total estimated effort:** ~6-7 hours

---

## Notes

- The Attention Map (BUG-PDF-1) is the most impactful fix — it adds the new Governance Attention Analysis chapter that differentiates the €149 Governance Audit from the €49 Deep Dive
- Text overlap (BUG-PDF-3) is customer-facing and makes the report look unprofessional — fix before sending any reports
- The naming inconsistency (BUG-PDF-4 + BUG-PDF-5) suggests the PDF template was originally built for the "Governance Truth Report" product name and wasn't fully updated when the tiers were restructured
