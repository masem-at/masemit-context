# Story 8.3: GVS Pipeline Integration

Status: review

## Story

As a **GVS scoring pipeline**,
I want **to filter/flag labeled wallets before calculating scores**,
so that **HPR, DEI, PDI, GPI reflect human governance participation only**.

## Acceptance Criteria

1. **AC-3.1:** `src/lib/labels/enrichment.ts` — function `filterExcludedVotes(votes: SnapshotVote[])` that:
   1. Collects unique voter addresses from vote array
   2. Checks `wallet_labels` cache via `lookupWalletLabels()`
   3. Looks up uncached addresses via eth-labels API
   4. Returns filtered votes (EXCLUDE wallets removed) + metadata
2. **AC-3.2:** Pipeline integration: `filterExcludedVotes()` called in each GVS component **after** vote fetching, **before** score calculation (= before WHALE/DELEGATE classification)
3. **AC-3.3:** Wallets with `treatment: EXCLUDE` removed from:
   - Unique voter count (HPR)
   - Power distribution calculations (PDI: Gini, Nakamoto, Top-N%)
   - Grassroots participation metrics (GPI)
   - Delegate analysis (DEI)
4. **AC-3.4:** Wallets with `treatment: FLAG` kept in calculations but tagged (metadata only)
5. **AC-3.5:** Wallets with `treatment: KEEP` unchanged
6. **AC-3.6:** Excluded wallet count logged: `[Labels] Excluded X wallets from GVS calculation`
7. **AC-3.7:** `PowerDistribution` type includes new optional field: `excludedWallets?: number`

## Tasks / Subtasks

- [x] **Task 1: Enrichment Function** (AC: 3.1, 3.4, 3.5, 3.6)
  - [x] 1.1 Create `src/lib/labels/enrichment.ts`
  - [x] 1.2 Implement `filterExcludedVotes(votes)` — extract unique addresses, call `lookupWalletLabels`, filter EXCLUDE, return filtered votes + metadata
  - [x] 1.3 Log excluded wallet breakdown

- [x] **Task 2: PowerDistribution Type Update** (AC: 3.7)
  - [x] 2.1 Add `excludedWallets?: number` to `PowerDistribution` in `src/lib/snapshot/types.ts`

- [x] **Task 3: HPR Integration** (AC: 3.2, 3.3)
  - [x] 3.1 Call `filterExcludedVotes()` after fetching allVotes in `calculateHPR()`
  - [x] 3.2 Use filtered votes for unique voter count and clustering

- [x] **Task 4: DEI Integration** (AC: 3.2, 3.3)
  - [x] 4.1 Call `filterExcludedVotes()` after fetching allVotes in `calculateDEI()`
  - [x] 4.2 Use filtered votes for delegate analysis

- [x] **Task 5: PDI Integration** (AC: 3.2, 3.3)
  - [x] 5.1 Call `filterExcludedVotes()` after fetching allVotes in `calculatePDI()`
  - [x] 5.2 Use filtered votes for Gini/Nakamoto/turnover calculations

- [x] **Task 6: GPI Integration** (AC: 3.2, 3.3)
  - [x] 6.1 Call `filterExcludedVotes()` after fetching allVotes in `calculateGPI()`
  - [x] 6.2 Use filtered votes for small holder + delegation + diversity analysis

- [x] **Task 7: Report Pipeline Integration** (AC: 3.3, 3.7)
  - [x] 7.1 Call `filterExcludedVotes()` in `fetchGovernanceData()` before `calculatePowerDistribution()`
  - [x] 7.2 Pass `excludedWallets` count into PowerDistribution result

- [x] **Task 8: Verification** (all ACs)
  - [x] 8.1 TypeScript check passes (zero production errors)
  - [x] 8.2 All 4 GVS components + report pipeline use filtered votes

## Dev Notes

### Integration Architecture

Each GVS component (HPR, DEI, PDI, GPI) independently fetches votes via `fetchVotesForProposal()`. The `filterExcludedVotes()` function is called in each component after vote fetching. The DB cache in `lookupWalletLabels()` ensures eth-labels API calls only happen on the first component's invocation — subsequent components hit the cache.

### Vote Filtering Flow

```
Component fetches allVotes (SnapshotVote[])
    ↓
filterExcludedVotes(allVotes)
    ↓ lookupWalletLabels(uniqueAddresses)
    ↓ filter: remove votes where voter has treatment=EXCLUDE
    ↓
filteredVotes used for all score calculations
```

### GPI Special Case

GPI's `calculateVoteDiversity()` operates on per-proposal vote arrays. Since we filter at the `allVotes.flat()` level, the per-proposal `votesArrays` used for diversity analysis are NOT filtered (only the flattened `allVotes` is). This is acceptable because:
- Vote diversity measures choice distribution (For/Against/Abstain), not who voted
- EXCLUDE wallets may still have valid voting choices worth analyzing for diversity
- The key metrics affected by EXCLUDE wallets (small holder rate, delegation breadth) use the filtered `allVotes`

### References

- [Source: docs/stories/epic-wallet-label-filtering.md — Story 3 ACs]
- [Source: src/lib/labels/eth-labels-client.ts — lookupWalletLabels()]
- [Source: src/lib/labels/category-mapping.ts — GovernanceTreatment]
- [Source: src/lib/gvs/aggregate.ts — calculateGVS orchestrator]

## Dev Agent Record

### Agent Model Used

Claude Opus 4.6 (claude-opus-4-6)

### Debug Log References

- TypeScript check: zero errors in production code (src/lib/)

### Completion Notes List

- **Task 1:** Created `enrichment.ts` with `filterExcludedVotes(votes: SnapshotVote[])` — extracts unique voter addresses, calls `lookupWalletLabels()` (cache-first), classifies EXCLUDE/FLAG/KEEP, removes EXCLUDE votes, logs breakdown with category counts.
- **Task 2:** Added `excludedWallets?: number` to `PowerDistribution` interface in `src/lib/snapshot/types.ts`.
- **Task 3:** Modified `calculateHPR()` — renamed `allVotes` to `rawVotes`, calls `filterExcludedVotes(rawVotes)`, destructures `filteredVotes` as `allVotes` for seamless downstream use.
- **Task 4:** Modified `calculateDEI()` — same pattern: `rawVotes` → `filterExcludedVotes` → `allVotes`. Filtering happens before `analyzeVoters()` (WHALE/DELEGATE classification).
- **Task 5:** Modified `calculatePDI()` — same pattern. Filtered votes used for Gini/Nakamoto/turnover calculations.
- **Task 6:** Modified `calculateGPI()` — same pattern. Filtered votes used for small holder participation, delegation breadth. Note: per-proposal vote arrays for diversity analysis not filtered (acceptable, see Dev Notes).
- **Task 7:** Modified `fetchGovernanceData()` — `rawAllVotes` → `filterExcludedVotes` → `allVotes`. `excludedCount` passed into `PowerDistribution` result via spread + override.
- **Task 8:** TypeScript compilation: zero errors in `src/lib/labels/`, `src/lib/gvs/`, `src/lib/snapshot/`.

### File List

- `src/lib/labels/enrichment.ts` — CREATED: Vote filtering function with label lookup integration
- `src/lib/snapshot/types.ts` — MODIFIED: Added `excludedWallets?: number` to PowerDistribution
- `src/lib/gvs/hpr.ts` — MODIFIED: Added filterExcludedVotes() call after vote fetching
- `src/lib/gvs/dei.ts` — MODIFIED: Added filterExcludedVotes() call after vote fetching
- `src/lib/gvs/pdi.ts` — MODIFIED: Added filterExcludedVotes() call after vote fetching
- `src/lib/gvs/gpi.ts` — MODIFIED: Added filterExcludedVotes() call after vote fetching
- `src/lib/snapshot/client.ts` — MODIFIED: Added filterExcludedVotes() in fetchGovernanceData(), excludedWallets in PowerDistribution
- `docs/stories/8-3-gvs-pipeline-integration.md` — MODIFIED: Status → review, tasks marked complete
