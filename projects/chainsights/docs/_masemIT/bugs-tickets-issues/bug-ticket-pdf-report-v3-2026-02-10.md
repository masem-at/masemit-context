# BUG TICKET v3: Governance Audit PDF — Final Polish Before Lido Outreach

**Priority:** High (blocks Lido forum outreach)
**Affected:** PDF Report Generation Pipeline
**Report Reviewed:** Lido Governance Audit (Generated: February 10, 2026, post-fix v2)
**Reporter:** Mario

---

## Fixed Since v2

- ✅ BUG-PDF-6: Attention Analysis columns — solved by removing empty columns, showing horizontal bars
- ✅ BUG-PDF-7: Score confusion — explanation text added on Page 2
- ✅ BUG-PDF-8: Visualization — horizontal bars added to Attention chapter
- ✅ BUG-PDF-9: Footer Page 10 — now shows page number

---

## Open Bugs

### BUG-PDF-11: Strategic Recommendations — Example text sticks to recommendation text above

**Severity:** High (visual quality)
**Page:** 11 (Strategic Recommendations)

**Expected:** Each recommendation card has clear visual separation between the recommendation text and the "Example:" line below it. The example should feel like a subordinate annotation, not glued to the main text.

**Actual:** The "Example:" lines in grey italic sit directly against the recommendation text above with zero or near-zero spacing. This makes the cards look cramped and unprofessional. See screenshot — the grey example text visually merges with the black recommendation text.

**Fix:** Add vertical margin/padding between the recommendation text and the "Example:" line. Suggested: 8-12px gap between recommendation body and example line. This should be a simple CSS/styling change in the PDF template.

**Visual reference (current):**
```
Launch delegation campaigns to reduce whale concentration by encouraging token distribution to
professional delegates
Example: Uniswap's delegate program successfully distributed voting power across professional governance participants
```

**Should look like:**
```
Launch delegation campaigns to reduce whale concentration by encouraging token distribution to
professional delegates

Example: Uniswap's delegate program successfully distributed voting power across professional governance participants
```

---

### BUG-PDF-12: Decentralization Score inconsistent between PDF generations (3/10 vs 4/10)

**Severity:** High (data integrity)
**Pages:** 1, 2

**Description:** Two PDFs generated on the same date (February 10, 2026) for the same DAO (Lido) show different Decentralization Scores:

- PDF Generation #1: **4/10 (C)**
- PDF Generation #2: **3/10 (D)**

This is either:
1. **A race condition** — the score changed between generations due to new data being ingested mid-day
2. **A rounding/boundary issue** — the raw score sits exactly at the boundary between 3 and 4, and minor calculation differences tip it one way or the other
3. **A caching issue** — one generation uses cached data, the other uses fresh data

**Impact:** If a customer generates two reports on the same day and gets different scores, trust is destroyed. For Lido outreach specifically: we can't have the score jump around.

**Fix:**
1. Investigate why the score differs between generations on the same day
2. If it's a boundary issue: implement rounding stability (e.g., once a score is published, it stays until a meaningful threshold is crossed — not on ±0.1 fluctuations)
3. If it's a data freshness issue: pin the report to a specific data snapshot timestamp and display it ("Data snapshot: Feb 10, 2026 00:00 UTC")
4. Add the raw decimal score to the report for transparency (e.g., "3.5/10" instead of rounding to 3 or 4)

---

### BUG-PDF-13: Cover page trend icon renders as yellow comma

**Severity:** Medium (visual quality)
**Page:** 1

**Expected:** A clear trend indicator icon next to "Stable" — e.g., a horizontal arrow (→), a dash (—), or a stability icon.

**Actual:** Renders as what looks like a large yellow comma or apostrophe character: **'**. This appears to be a font rendering issue — likely an icon font glyph that doesn't render correctly in the PDF generation library.

**Fix:** Either:
- Replace the icon font glyph with an inline SVG arrow
- Use a Unicode character that renders reliably in PDF (→ or —)
- Remove the icon entirely and rely on the "Stable" text label

---

### BUG-PDF-14: Page numbering incorrect / duplicated

**Severity:** Low
**Pages:** Throughout (10, 11, 12)

**Actual page numbering in footer:**
- Pages 1-9: Correct (Page 2 through Page 7)
- Page 10 (Attention Analysis): "Page 8"
- Page 11 (Recommendations): "Page 8" ← duplicate!
- Page 12 (About): "Page 9"

**Expected:** Sequential page numbers without duplicates.

**Fix:** The page counter likely wasn't incremented when the Attention Analysis chapter was inserted. Review the page generation loop and ensure each page gets a unique sequential number.

---

## Priority Order

| # | Bug | Severity | Blocks Lido? | Effort |
|---|-----|----------|-------------|--------|
| 1 | **BUG-PDF-11: Example text spacing** | High | YES — looks unprofessional | ~30min |
| 2 | **BUG-PDF-12: Score inconsistency 3/10 vs 4/10** | High | YES — data trust issue | ~2-3h |
| 3 | **BUG-PDF-13: Yellow comma trend icon** | Medium | Cosmetic but noticeable | ~30min |
| 4 | **BUG-PDF-14: Duplicate page numbers** | Low | Minor | ~15min |

**Must fix before Lido outreach:** BUG-PDF-11, BUG-PDF-12
**Should fix before Lido outreach:** BUG-PDF-13, BUG-PDF-14

**Total estimated effort:** ~3.5-4.5h

---

## Cumulative Status (All Bugs Since v1)

| Bug | Status | Fixed In |
|-----|--------|----------|
| BUG-PDF-1: Attention Map missing | ✅ Fixed | v2 |
| BUG-PDF-2: Letter grade missing | ✅ Fixed | v2 |
| BUG-PDF-3: Text overlap recommendations | ✅ Fixed | v2 (overlap gone, spacing still tight) |
| BUG-PDF-4: Inconsistent footer naming | ✅ Fixed | v2 |
| BUG-PDF-5: Cover/header mismatch | ✅ Fixed | v2 |
| BUG-PDF-6: Empty VOTERS/CONCENTRATION | ✅ Fixed | v3 |
| BUG-PDF-7: Score confusion text | ✅ Fixed | v3 |
| BUG-PDF-8: No visualization | ✅ Fixed | v3 (horizontal bars) |
| BUG-PDF-9: Page 10 footer | ✅ Fixed | v3 |
| BUG-PDF-10: Recommendations spacing | 🔄 Partially | Overlap gone, Example spacing still tight → BUG-PDF-11 |
| **BUG-PDF-11: Example text sticks** | ❌ Open | — |
| **BUG-PDF-12: Score 3/10 vs 4/10** | ❌ Open | — |
| **BUG-PDF-13: Yellow comma icon** | ❌ Open | — |
| **BUG-PDF-14: Duplicate page numbers** | ❌ Open | — |
