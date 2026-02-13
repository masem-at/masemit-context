# Story 8.5: Before/After Validation + Score Logging

Status: review

## Story

As a **ChainSights operator**,
I need **to see the impact of wallet filtering on GVS scores**,
so that **I can validate the improvement and communicate changes**.

## Acceptance Criteria

1. **AC-5.1:** One-time validation script: run GVS calculation for Lido with and without filtering, log both scores
2. **AC-5.2:** Log output shows per-component impact: HPR before/after, PDI before/after, DEI before/after, GPI before/after, GVS before/after
3. **AC-5.3:** Log shows which wallets were excluded with their labels (for manual verification)
4. **AC-5.4:** Run validation for at least 3 DAOs (Lido + 2 others) to confirm cross-DAO benefit
5. **AC-5.5:** Results documented for potential blog post / changelog

## Tasks / Subtasks

- [x] **Task 1: Global Skip Toggle** (AC: 5.1)
  - [x] 1.1 Add `_skipFiltering` variable and `setSkipLabelFiltering()` export to `src/lib/labels/enrichment.ts`
  - [x] 1.2 Wire up `_skipFiltering` check at top of `filterExcludedVotes()` — returns all votes unchanged when true

- [x] **Task 2: Validation Script** (AC: 5.1, 5.2, 5.3, 5.4, 5.5)
  - [x] 2.1 Create `scripts/validate-wallet-filtering.ts` with dotenv + relative imports pattern
  - [x] 2.2 For each DAO: run `calculateGVS()` with skip=true (before), then skip=false (after)
  - [x] 2.3 Log per-component comparison table (AC-5.2)
  - [x] 2.4 After GVS runs, fetch votes + lookup labels to list EXCLUDE wallets with address/name/category (AC-5.3)
  - [x] 2.5 Default DAOs: Lido, ENS, Gitcoin (AC-5.4)
  - [x] 2.6 Write JSON results to `data/wallet-filtering-validation.json` (AC-5.5)
  - [x] 2.7 Support `--dao=spaceId` flag for single-DAO runs

- [x] **Task 3: Verification** (all ACs)
  - [x] 3.1 TypeScript check passes (zero production errors)

## Dev Notes

### Skip Toggle

The `setSkipLabelFiltering(true)` function sets a module-level flag. When active, `filterExcludedVotes()` returns all votes unchanged (zero excluded, zero flagged) without making any API calls. This allows the validation script to compare GVS scores with identical data.

### Script Pattern

Follows `scripts/generate-rankings.ts` pattern: `dotenv` for `.env.local`, relative imports from `../src/lib/...`, runs via `npx tsx scripts/validate-wallet-filtering.ts`.

### Label Cache

Labels are cached in the DB after the first GVS run (skip=true still processes votes normally through each component). The second run (skip=false) benefits from the label cache — `lookupWalletLabels()` hits DB instead of API. The AC-5.3 listing also hits cache.

### References

- [Source: docs/stories/epic-wallet-label-filtering.md — Story 5 ACs]
- [Source: src/lib/labels/enrichment.ts — filterExcludedVotes, setSkipLabelFiltering]
- [Source: scripts/validate-wallet-filtering.ts — validation script]

## Dev Agent Record

### Agent Model Used

Claude Opus 4.6 (claude-opus-4-6)

### Debug Log References

- TypeScript check: zero errors in production code and script

### Completion Notes List

- **Task 1:** Added `_skipFiltering` toggle and `setSkipLabelFiltering()` to enrichment.ts. Wired up the check at top of `filterExcludedVotes()` — when true, returns early with unfiltered votes.
- **Task 2:** Created `scripts/validate-wallet-filtering.ts` — runs calculateGVS before/after for 3 DAOs, logs per-component impact, lists excluded wallets with labels, writes JSON results.
- **Task 3:** TypeScript compilation: zero errors in modified/created files.

### File List

- `src/lib/labels/enrichment.ts` — MODIFIED: Added skip check in filterExcludedVotes()
- `scripts/validate-wallet-filtering.ts` — CREATED: Before/after validation script
- `docs/stories/8-5-before-after-validation.md` — CREATED: Story file → review
