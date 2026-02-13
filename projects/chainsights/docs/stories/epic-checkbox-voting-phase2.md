# Epic 9: Checkbox Voting Detection — Phase 2 (Delegate Scorecard)

**Source:** `docs/analysis/product-brief-checkbox-voting-detection-2026-02-11.md`
**Origin:** BCV (Lido Forum) community feedback — "distinguish informed participation from checkbox voting"
**Decision:** Approved 2026-02-12 (Party Mode: Mary, John, Paige, Winston, Sally)
**Depends On:** Epic Phase 1 (done — checkbox-detection.ts, GPI integration, voter_quality_signals table)
**Status:** Ready for Dev

---

## Overview

Phase 2 adds **visible** vote quality analytics. Phase 1 (backend-only) adjusted GPI scoring to penalize checkbox voters. Phase 2 introduces two new signals (Focus, Originality), computes a composite **Vote Quality Score (VQS)** per delegate, and displays it on the Matrix Details page + Governance Audit PDF.

**What BCV asked for:** "Distinguish informed participation from delegates voting on everything regardless of expertise."

**What Phase 2 delivers:** Per-delegate quality scores with 4 signals, visible on the DAO detail page and in paid reports.

---

## Architecture Decisions (from Party Mode)

| Decision | Owner | Rationale |
|----------|-------|-----------|
| DAO Size Normalization for Signal 3 | Mary (BA) | DAOs with <10 proposals: Focus weight = 0%. Linear scale 10→25 proposals. Small DAOs shouldn't penalize generalists. |
| Pre-computed Whale Vote Matrix for Signal 4 | Winston (Arch) | `Map<proposalId, whaleChoices[]>` for Top 3 whales. Per voter: O(1) per proposal. Total: O(n×m) not O(n²). |
| No warning symbols in UI | Sally (UX) | No `⚠️`, `🚩`, `❗`. Color gradient green→yellow→orange. Neutral labels: "Rapid, Consistent" / "Deliberate, Varied". |
| Methodology docs ship WITH code | Paige (TW) | No deployment without methodology page update. v1.3 version marker. |
| Phase 3 NOT scheduled | John (PM) | No GVS recomposition until community has adopted VQS. Phase 1+2 first. |

---

## Story 9-1: Signal 3 (Focus) + Signal 4 (Originality) + VQS Composite

**As a** GVS pipeline
**I need** two new quality signals and a composite VQS score per voter
**So that** we can display meaningful delegate quality data on the UI

### Acceptance Criteria

- [ ] AC-9.1.1: `calculateFocusSignal(voterVotes, proposalCategoryMap, totalProposalCount)` in `src/lib/gvs/checkbox-detection.ts`
  - Counts how many categories a voter votes in (using Attention Map category data)
  - `categoryConcentration = maxCategoryVotes / totalVoterVotes`
  - `focusScore = categoryConcentration * 10` (0-10 scale, 10 = focused specialist)
  - **DAO Size Normalization (Mary):** Signal 3 dynamic weight based on `totalProposalCount`:
    - `< 10 proposals` → weight = 0% (exempt — not enough categories to specialize)
    - `10–25 proposals` → weight scales linearly from 0% to 25%
    - `> 25 proposals` → full 25% weight
- [ ] AC-9.1.2: `calculateOriginalitySignal(voterVotes, whaleVoteMatrix)` in `src/lib/gvs/checkbox-detection.ts`
  - **Pre-computed Whale Vote Matrix (Winston):** `Map<proposalId, number[]>` — top 3 whale choices per proposal
  - Built once per DAO at start of VQS calculation, NOT per voter
  - Per voter: for each proposal, check if voter's choice matches ANY whale choice → alignment ratio
  - `originalityScore = (1 - alignmentRatio) * 10` (0-10 scale, 10 = fully independent)
  - Only `typeof choice === 'number'` votes compared (ranked/weighted excluded — optimistic)
- [ ] AC-9.1.3: `calculateVQS(signals)` → composite Vote Quality Score (0-10)
  - Default weights: Deliberation 25%, Independence 25%, Focus 25%, Originality 25%
  - When Focus weight is reduced (small DAO), redistribute proportionally across other 3 signals
  - Maps Phase 1 signals to 0-10 scale: `deliberationScore = (1 - speedSignal) * 10`, `independenceScore = (1 - diversitySignal) * 10`
- [ ] AC-9.1.4: `VQSResult` type with all 4 signal scores + composite + pattern label
  - Pattern label generated from signals (Sally's framing): "Rapid, Consistent" / "Deliberate, Varied" / "Focused, Independent" etc.
  - No judgmental text — purely descriptive of observed patterns
- [ ] AC-9.1.5: Phase 1 `CheckboxSignals` type extended with optional Phase 2 fields: `focusScore?`, `originalityScore?`, `vqsScore?`, `patternLabel?`
- [ ] AC-9.1.6: `voter_quality_signals` table columns populated: `cross_category_signal`, `pattern_correlation_signal`, `vqi_score` (already exist as nullable from Phase 1 migration)
- [ ] AC-9.1.7: VQS calculation integrated into `calculateGPI()` flow — called after Phase 1 checkbox signals
- [ ] AC-9.1.8: `saveCheckboxSignals()` updated to also persist Phase 2 columns

### Technical Notes

- Attention Map categories come from proposal classification (already live). Need access to `proposalCategoryMap: Map<proposalId, string>` — check how Attention Map stores categories.
- Whale Vote Matrix construction: identify top 3 voters by cumulative `vp` across all proposals, then build `Map<proposalId, number[]>` of their choices.
- Existing `calculateCheckboxSignals()` returns a `Map<voter, CheckboxSignals>` — extend this to include Phase 2 signals, or create separate `calculatePhase2Signals()` that enriches the existing map.
- Pattern labels (Sally): combine 2 axes — Speed (Rapid/Moderate/Deliberate) × Diversity (Consistent/Mixed/Varied). E.g., "Rapid, Consistent" for high speed + low diversity.

### File List

- `src/lib/gvs/checkbox-detection.ts` — Add Signal 3, Signal 4, VQS composite
- `src/lib/gvs/types.ts` — VQSResult type, extend GPIResult metadata
- `src/lib/gvs/gpi.ts` — Integrate Phase 2 signals into pipeline
- `src/lib/gvs/checkbox-storage.ts` — Persist Phase 2 columns
- `src/lib/gvs/index.ts` — Export new types and functions

---

## Story 9-2: API Endpoint for VQS Data

**As a** frontend developer
**I need** an API endpoint that returns per-voter VQS data for a DAO
**So that** the Matrix Details page can display delegate quality scores

### Acceptance Criteria

- [ ] AC-9.2.1: `GET /api/dao/[spaceId]/vqs` returns VQS data for a DAO
- [ ] AC-9.2.2: Response shape:
  ```json
  {
    "spaceId": "lido-snapshot.eth",
    "calculationDate": "2026-02-12",
    "totalVotersAnalyzed": 245,
    "voters": [
      {
        "address": "0x7eE0...",
        "vqsScore": 4.1,
        "deliberationScore": 2.1,
        "independenceScore": 3.4,
        "focusScore": 6.2,
        "originalityScore": 4.5,
        "patternLabel": "Rapid, Consistent",
        "totalVotesAnalyzed": 47,
        "checkboxScore": 0.72
      }
    ]
  }
  ```
- [ ] AC-9.2.3: Query `voter_quality_signals` table for latest `snapshot_date` per space
- [ ] AC-9.2.4: Returns only voters with >= 3 votes (same threshold as Phase 1)
- [ ] AC-9.2.5: Sorted by `vqs_score` ascending (lowest quality first — most interesting for analysis)
- [ ] AC-9.2.6: Optional `?limit=N` parameter (default 50, max 200)
- [ ] AC-9.2.7: No auth required (VQS data is public — same as GVS scores on Matrix)

### Technical Notes

- Standard Next.js route handler pattern matching existing `/api/dao/[spaceId]/...` endpoints.
- Single DB query with WHERE + ORDER BY + LIMIT — should be fast with existing indexes.
- Consider adding `idx_vqs_vqi_score` index on `(space_id, vqi_score)` if performance needs it.

### File List

- `src/app/api/dao/[spaceId]/vqs/route.ts` — API route handler
- `src/lib/db/schema.ts` — Add Drizzle query helpers if needed

---

## Story 9-3: UI — Matrix Details Scorecard + Expand Panel

**As a** Matrix Details page visitor
**I need** to see delegate vote quality scores
**So that** I can assess the quality of governance participation in a DAO

### Acceptance Criteria

- [ ] AC-9.3.1: "Vote Quality" column added to Key Governance Participants table on Matrix Details page
- [ ] AC-9.3.2: VQS displayed as colored bar + numeric score (0-10)
  - **Color gradient (Sally):** Green (8-10) → Yellow (5-7) → Orange (2-4) → Light red (0-2)
  - **No** warning symbols (`⚠️`, `🚩`, `❗`) anywhere
  - **No** judgmental labels ("checkbox voter", "low quality", "suspicious")
- [ ] AC-9.3.3: Click/tap on any voter row opens expand panel showing 4 signal breakdown:
  ```
  0x7eE0...bb6C — DELEGATE — Vote Quality: 4.1/10

    Deliberation:  ██░░░░░░░░  2.1  Avg 23s between votes
    Independence:  ███░░░░░░░  3.4  92% same-direction votes
    Focus:         ██████░░░░  6.2  Active in 6 of 8 categories
    Originality:   ████░░░░░░  4.5  78% alignment with top holders

    Voting Pattern: Rapid, Consistent
  ```
- [ ] AC-9.3.4: Each signal bar uses the same green→orange color gradient (per signal score)
- [ ] AC-9.3.5: Signal descriptions are factual/numerical, NOT interpretive:
  - Deliberation: "Avg Xs between votes" (raw number)
  - Independence: "X% same-direction votes" (raw number)
  - Focus: "Active in X of Y categories" (raw number)
  - Originality: "X% alignment with top holders" (raw number)
- [ ] AC-9.3.6: Pattern label shown as neutral tag: "Rapid, Consistent" / "Deliberate, Varied" etc.
- [ ] AC-9.3.7: Data fetched from `/api/dao/[spaceId]/vqs` endpoint (Story 9-2)
- [ ] AC-9.3.8: Graceful fallback if VQS data not yet available: column shows "—" with tooltip "Quality data calculating..."
- [ ] AC-9.3.9: Mobile responsive — expand panel stacks vertically on small screens

### Technical Notes

- The Key Governance Participants table is on the Matrix Details page (Epic 3). It already shows: Address, Type, Voting Power, Participation.
- VQS column adds to the right. Expand panel below the row (accordion style, one open at a time).
- Use `useSWR` or `useEffect` to fetch VQS data separately from main page data (non-blocking).
- Color gradient implementation: utility function `vqsToColor(score: number): string` mapping 0-10 to hex colors.

### File List

- `src/app/dao/[spaceId]/components/VoteQualityColumn.tsx` — New column component
- `src/app/dao/[spaceId]/components/VQSExpandPanel.tsx` — Signal breakdown expand panel
- `src/app/dao/[spaceId]/components/VQSBar.tsx` — Colored progress bar component
- `src/app/dao/[spaceId]/page.tsx` or relevant parent — Wire VQS column into existing table
- `src/lib/utils/vqs-colors.ts` — Color gradient utility

---

## Story 9-4: PDF Report — VQS in Governance Audit

**As a** Governance Audit report reader (EUR 149 tier)
**I need** vote quality analysis in the PDF report
**So that** I get actionable insights about delegate governance quality

### Acceptance Criteria

- [ ] AC-9.4.1: New "Vote Quality Analysis" section in Governance Audit PDF (after existing Governance Participants section)
- [ ] AC-9.4.2: Section shows top 10 voters by voting power with their VQS scores and pattern labels
- [ ] AC-9.4.3: Table format:
  ```
  Delegate Vote Quality Analysis

  Address         Type       VQS    Pattern
  0x4af8...1F6A   WHALE      6.2    Moderate, Mixed
  0x7eE0...bb6C   DELEGATE   4.1    Rapid, Consistent
  0x55Bc...f0fA   TOP VOTER  8.7    Deliberate, Varied
  ...
  ```
- [ ] AC-9.4.4: Summary paragraph generated by AI analysis prompt:
  - Avg VQS across all voters
  - Count of voters with VQS < 4 (low quality signals)
  - Most common pattern label
  - **Framing (Sally):** Neutral observations, NOT accusations. E.g., "X% of active delegates show rapid, consistent voting patterns" not "X% are checkbox voters"
- [ ] AC-9.4.5: VQS data passed to AI analysis prompt alongside existing GVS data
- [ ] AC-9.4.6: Color coding in PDF: same green→orange gradient as UI (PDF-compatible colors)
- [ ] AC-9.4.7: Only included in Governance Audit tier (EUR 149), NOT in free Quick Check or Deep Dive

### Technical Notes

- PDF is generated via `src/lib/pdf/report-template.tsx` using @react-pdf/renderer.
- AI analysis prompt is in `src/lib/ai/analysis.ts` — extend with VQS data context.
- VQS data can be fetched during report generation via direct DB query (no API needed, server-side).
- Keep PDF section concise — max 1 page for VQS analysis.

### File List

- `src/lib/pdf/report-template.tsx` — Add VQS section after Governance Participants
- `src/lib/ai/analysis.ts` — Add VQS context to AI analysis prompt
- `src/lib/pdf/types.ts` — Extend report data types with VQS data

---

## Story 9-5: Methodology Page + Version Marker v1.3

**As a** ChainSights user
**I need** transparent methodology documentation for vote quality scoring
**So that** I understand how VQS is calculated and trust the scores

### Acceptance Criteria

- [ ] AC-9.5.1: New "Vote Quality Score (VQS)" section on `/rankings/methodology` page
- [ ] AC-9.5.2: Section explains:
  - What VQS measures (quality of governance participation)
  - The 4 signals: Deliberation, Independence, Focus, Originality
  - How signals are combined (weighted average, dynamic Focus weight for small DAOs)
  - Score interpretation (0-10 scale, pattern labels)
  - What VQS does NOT measure (it doesn't judge correctness of votes, only engagement patterns)
- [ ] AC-9.5.3: Framing follows Sally's rules:
  - "VQS measures voting patterns, not voting quality" / "Shows engagement style, not correctness"
  - No judgmental language about low-scoring delegates
  - Explicit disclaimer: "A low VQS does not mean a delegate is 'bad' — it indicates rapid, consistent voting patterns that may warrant closer examination."
- [ ] AC-9.5.4: Version marker updated to **v1.3** (current is v1.2 from Epic 8 wallet filtering)
- [ ] AC-9.5.5: Changelog entry added:
  ```
  v1.3 (2026-02-XX) — Vote Quality Score
  - Added Vote Quality Score (VQS) for per-delegate quality analysis
  - 4 signals: Deliberation, Independence, Focus, Originality
  - VQS visible on Matrix Details page and in Governance Audit reports
  - Phase 1 checkbox detection continues to adjust GPI scoring (invisible)
  ```
- [ ] AC-9.5.6: Methodology page ships on same deployment as code (Paige's requirement)

### Technical Notes

- Methodology page is `src/app/rankings/methodology/page.tsx`.
- Current version is v1.2 (added in Epic 8 for wallet filtering).
- Keep methodology section accessible — avoid technical jargon, explain with examples.

### File List

- `src/app/rankings/methodology/page.tsx` — Add VQS section + v1.3 changelog

---

## Story Dependencies

```
Story 9-1 (Signals + VQS)
    ↓
Story 9-2 (API Endpoint)
    ↓
Story 9-3 (UI Scorecard)  ←  Core visible feature

Story 9-1 (Signals + VQS)
    ↓
Story 9-4 (PDF Report)    ←  Parallel with 9-3

Story 9-5 (Methodology)   ←  Ships WITH 9-3 (same deployment)
```

**Critical path:** 9-1 → 9-2 → 9-3 + 9-5 (same deploy)
**Parallel:** 9-4 can start after 9-1, parallel with 9-2/9-3

---

## Definition of Done

- [ ] All 4 signals calculated and stored in `voter_quality_signals` table
- [ ] VQS visible on Matrix Details page with color gradient and expand panel
- [ ] VQS included in Governance Audit PDF reports
- [ ] Methodology page updated to v1.3 with VQS documentation
- [ ] No warning symbols or judgmental language anywhere in UI/reports
- [ ] Pattern labels are neutral and descriptive
- [ ] DAO size normalization working for Signal 3 (Focus)
- [ ] Whale Vote Matrix pre-computed, not O(n^2)
- [ ] Validation: run VQS calculation for 10+ DAOs, verify reasonable scores
