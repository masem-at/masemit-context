# Story 8.4: Report + Methodology Transparency

Status: review

## Story

As a **report reader**,
I want **to know that wallet filtering was applied**,
so that **I understand the methodology and trust the scores**.

## Acceptance Criteria

1. **AC-4.1:** PDF report methodology section updated with transparency note: "This analysis distinguishes between human participants and protocol wallets. Known treasury, exchange, and bridge wallets are identified via public label data and excluded from participation metrics."
2. **AC-4.2:** Excluded wallet count shown in methodology section (e.g., "3 wallets excluded")
3. **AC-4.3:** `/rankings/methodology` page updated with wallet filtering explanation section
4. **AC-4.4:** AI analysis prompt updated to mention that data is pre-filtered (so LLM doesn't mention treasury wallets that were already excluded)

## Tasks / Subtasks

- [x] **Task 1: PDF Methodology Text** (AC: 4.1, 4.2)
  - [x] 1.1 Update `methodology` string in `src/lib/ai/analysis.ts` to include wallet filtering transparency note
  - [x] 1.2 Include `excludedWallets` count from PowerDistribution in methodology text (conditional — only shows if >0)

- [x] **Task 2: AI Prompt Update** (AC: 4.4)
  - [x] 2.1 Add note to `ANALYSIS_PROMPT` that data is pre-filtered for protocol wallets
  - [x] 2.2 Add `Wallets Excluded` line to `formatDataForAnalysis()` Power Distribution section (conditional)

- [x] **Task 3: Methodology Page** (AC: 4.3)
  - [x] 3.1 Add "Data Quality: Wallet Filtering" section to `/rankings/methodology` with Filter icon
  - [x] 3.2 Section explains: what gets filtered (treasury, CEX, bridge, protocol) and what stays (fund/VC, individual, multisig)
  - [x] 3.3 Info box explaining eth-labels source (115K+ addresses) + manual overrides
  - [x] 3.4 Updated methodology version to v1.2 (February 2026)
  - [x] 3.5 Updated change log with v1.2 entry

- [x] **Task 4: Verification** (all ACs)
  - [x] 4.1 TypeScript check passes (zero production errors)

## Dev Notes

### Methodology Text (PDF)

The `methodology` string in `analyzeGovernanceData()` now includes a "Data Quality" paragraph between the intro and classification rules. Uses conditional template literal: `excludedWallets` count is only shown when >0.

### AI Prompt

Added to `ANALYSIS_PROMPT` IMPORTANT section: "The data has been pre-filtered: known treasury, exchange, bridge, and protocol wallets have already been excluded. Do NOT mention these wallet types as participants."

Also added `Wallets Excluded` line to `formatDataForAnalysis()` output (conditional on `excludedWallets > 0`).

### Methodology Page

New section placed before "Data Sources" — uses the `Filter` icon from lucide-react. Matches existing card/section pattern. Language follows the epic's guidance: neutral ("distinguishes between human participants and protocol wallets") not negative ("removes wallets").

### References

- [Source: docs/stories/epic-wallet-label-filtering.md — Story 4 ACs]
- [Source: src/lib/ai/analysis.ts — methodology string, ANALYSIS_PROMPT, formatDataForAnalysis]
- [Source: src/app/rankings/methodology/page.tsx — GVS methodology page]

## Dev Agent Record

### Agent Model Used

Claude Opus 4.6 (claude-opus-4-6)

### Debug Log References

- TypeScript check: zero errors in production code

### Completion Notes List

- **Task 1:** Updated methodology string in `analyzeGovernanceData()` — added "Data Quality" paragraph with transparency note (AC-4.1) and conditional excluded wallet count (AC-4.2).
- **Task 2:** Updated `ANALYSIS_PROMPT` with instruction to not mention pre-filtered wallet types. Updated `formatDataForAnalysis()` to show excluded wallet count in Power Distribution section.
- **Task 3:** Added "Data Quality: Wallet Filtering" Card section to methodology page. Explains what gets filtered vs kept. Info box about eth-labels + manual overrides. Version bumped to v1.2, changelog updated.
- **Task 4:** TypeScript compilation: zero errors in modified files.

### File List

- `src/lib/ai/analysis.ts` — MODIFIED: methodology string, ANALYSIS_PROMPT, formatDataForAnalysis
- `src/app/rankings/methodology/page.tsx` — MODIFIED: New Data Quality section, Filter import, version bump, changelog
- `docs/stories/8-4-report-methodology-transparency.md` — MODIFIED: Status → review, tasks marked complete
