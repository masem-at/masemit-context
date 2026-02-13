# BUG TICKET v4: Governance Audit PDF — Final Issues

**Priority:** High (blocks Lido forum outreach)
**Affected:** PDF Report Generation Pipeline
**Report Reviewed:** Lido Governance Audit (Generated: February 10, 2026, post-fix v3)
**Reporter:** Mario

---

## Fixed Since v3

- ✅ BUG-PDF-12: Score now consistently 3/10 (D) across generations
- ✅ BUG-PDF-13: Yellow comma replaced with "—" dash for Stable indicator

---

## Open Bugs

### BUG-PDF-11 (STILL OPEN): Example text still sticks to recommendation text

**Severity:** High (visual quality — makes the report look unprofessional)
**Page:** 12 (Strategic Recommendations)
**Status:** NOT FIXED — same issue as v3

**The Problem:** The grey italic "Example:" lines sit directly against the bold recommendation text above with zero vertical spacing. They visually merge into one block of text. This is especially bad when the recommendation text wraps to multiple lines — the example starts on the very next line with no breathing room.

**Visual proof (Screenshot, first HIGH card):**
```
...mandatory participation thresholds and reg-
ular reporting requirements
Example: Compound's delegate accountability framework requires quarterly reports and minimum
participation rates
```

The "Example:" line needs to be visually separated from the recommendation above it.

**Fix — specific guidance for the developer:**

Option A (preferred): Add `margin-top: 8px` (or equivalent in PDF units) to the Example text element. This creates a clear visual gap.

Option B: Add a blank line (`\n`) between recommendation body and example text in the template string before rendering.

Option C: Increase the `line-height` of the Example text so it naturally creates more space above.

**Acceptance criteria:** There must be a visible gap between the last line of the recommendation text and the "Example:" line. The gap should be approximately the height of one line of text (~8-12px).

---

### BUG-PDF-15 (NEW): Blank pages at Page 3 and Page 13

**Severity:** High (looks broken)
**Pages:** 3, 13

**Expected:** No blank pages in the report. Content flows continuously.

**Actual:** 
- **Page 3** contains only the footer "ChainSights Governance Audit Page 2" — the rest of the page is completely empty. The Participation Analysis content from Page 2 seems to overflow but the actual text ends on Page 2, leaving Page 3 as a ghost page.
- **Page 13** contains only the footer "ChainSights Governance Audit Page 9" — same issue after the Strategic Recommendations section.

**Likely Cause:** A page break is being inserted after certain sections (Participation Analysis, Strategic Recommendations) even when the content fits on the current page. This could be:
1. A forced `page-break-after: always` CSS rule on these sections
2. A content height miscalculation that thinks the content overflows when it doesn't
3. The footer being rendered as a separate page element instead of part of the current page

**Fix:** 
- Check for explicit page-break rules after the Participation Analysis and Recommendations sections
- If using a "content + footer" pattern: ensure the footer is attached to the content page, not rendered as its own page
- Remove any forced page breaks that create empty pages

**Impact:** A 14-page report with 2 blank pages looks like a broken PDF. Reduces it to 12 pages of actual content, which is cleaner anyway.

---

### BUG-PDF-14 (STILL OPEN): Page numbering incorrect

**Severity:** Medium
**Pages:** Throughout

**Current state (14 physical pages):**

| Physical Page | Content | Footer Shows |
|--------------|---------|-------------|
| 1 | Cover | (no page number) |
| 2 | Executive Detail | Page 2 |
| 3 | **BLANK** | Page 2 |
| 4 | Power Distribution | (no page number visible) |
| 5 | Key Participants cont. | Page 3 |
| 6 | Voting Patterns | (no page number visible) |
| 7 | Methodology | Page 4 |
| 8 | How You Compare | Page 5 |
| 9 | Historical Trends | Page 6 |
| 10 | Peer Comparison | Page 7 |
| 11 | Attention Analysis | Page 8 |
| 12 | Recommendations | (no page number) |
| 13 | **BLANK** | Page 9 |
| 14 | About | Page 10 |

**Issues:**
1. Blank pages inherit the previous page's number
2. Some content pages have no page number at all (Pages 4, 6, 12)
3. Numbering doesn't match physical page count

**Fix:** After removing blank pages (BUG-PDF-15), re-verify that every content page gets a sequential page number in the footer. The cover and About page can be excluded from numbering.

---

## Priority Order

| # | Bug | Severity | Effort | Blocks Lido? |
|---|-----|----------|--------|-------------|
| 1 | **BUG-PDF-11: Example spacing** | High | ~30min | YES |
| 2 | **BUG-PDF-15: Blank pages** | High | ~1-2h | YES |
| 3 | **BUG-PDF-14: Page numbering** | Medium | ~30min (after #2) | Should fix |

**Total estimated effort:** ~2-3h

---

## Cumulative Status (All Bugs)

| Bug | Description | Status |
|-----|-------------|--------|
| BUG-PDF-1 | Attention Map missing | ✅ Fixed |
| BUG-PDF-2 | Letter grade missing | ✅ Fixed |
| BUG-PDF-3 | Text overlap recommendations | ✅ Fixed |
| BUG-PDF-4 | Inconsistent footer naming | ✅ Fixed |
| BUG-PDF-5 | Cover/header mismatch | ✅ Fixed |
| BUG-PDF-6 | Empty VOTERS/CONCENTRATION | ✅ Fixed |
| BUG-PDF-7 | Score confusion text | ✅ Fixed |
| BUG-PDF-8 | No visualization | ✅ Fixed |
| BUG-PDF-9 | Page 10 footer | ✅ Fixed |
| BUG-PDF-10 | Recommendations spacing | ✅ Partially (overlap gone) |
| **BUG-PDF-11** | **Example text sticks** | **❌ OPEN — 3rd ticket** |
| BUG-PDF-12 | Score 3/10 vs 4/10 | ✅ Fixed |
| BUG-PDF-13 | Yellow comma icon | ✅ Fixed |
| **BUG-PDF-14** | **Page numbering wrong** | **❌ OPEN** |
| **BUG-PDF-15** | **Blank pages (NEW)** | **❌ OPEN** |

---

## Note to Developer

BUG-PDF-11 has been reported in v2 and v3 already. Please specifically test the spacing between recommendation body text and "Example:" lines before marking as fixed. A simple visual check: if you can't tell where the recommendation ends and the example begins at a glance, the spacing is insufficient.
