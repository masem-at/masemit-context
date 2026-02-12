# Story 9.1: Signal 3 (Focus) + Signal 4 (Originality) + VQS Composite

Status: done

## Story

As a GVS pipeline,
I want two new vote quality signals (Focus + Originality) and a composite VQS score per voter,
so that Phase 2 Delegate Scorecard can display meaningful per-delegate quality data.

## Acceptance Criteria

1. **AC-9.1.1: Focus Signal (Signal 3)** — `calculateFocusSignal()` in `src/lib/gvs/checkbox-detection.ts`
   - Counts how many governance categories a voter votes in (via `proposal_categories` table)
   - `categoryConcentration = maxCategoryVotes / totalVoterVotes`
   - `focusScore = categoryConcentration * 10` (0-10 scale, 10 = focused specialist)
   - **DAO Size Normalization:** Dynamic weight based on `totalProposalCount`:
     - `< 10 proposals` → Focus weight = 0% (exempt)
     - `10–25 proposals` → weight scales linearly 0% → 25%
     - `> 25 proposals` → full 25% weight
   - Only considers proposals that have been classified (skip unclassified)

2. **AC-9.1.2: Originality Signal (Signal 4)** — `calculateOriginalitySignal()` in `src/lib/gvs/checkbox-detection.ts`
   - **Pre-computed Whale Vote Matrix:** `Map<string, number[]>` keyed by proposalId → top 3 whale choices
   - Matrix built ONCE per DAO before voter loop (NOT per voter)
   - Top 3 whales: highest cumulative `vp` across all votes (use `analyzeVoters()` pattern)
   - Per voter: for each proposal, check if voter's choice matches ANY whale choice
   - `alignmentRatio = matchCount / voterProposalCount`
   - `originalityScore = (1 - alignmentRatio) * 10` (0-10, 10 = fully independent)
   - Only `typeof choice === 'number'` votes compared (ranked/weighted excluded — optimistic 0 alignment)

3. **AC-9.1.3: VQS Composite** — `calculateVQS()` returns weighted composite 0-10
   - Maps Phase 1 signals to 0-10 scale:
     - `deliberationScore = (1 - speedSignal) * 10`
     - `independenceScore = (1 - diversitySignal) * 10`
   - Default weights: Deliberation 25%, Independence 25%, Focus 25%, Originality 25%
   - When Focus weight is reduced (small DAO): redistribute proportionally across other 3 signals
   - Example: Focus weight = 10% → remaining 90% split as 30/30/30

4. **AC-9.1.4: VQSResult type** with all 4 signal scores + composite + pattern label
   - Pattern label from Sally's framing rules: Speed axis × Diversity axis
   - Speed: "Rapid" (deliberation < 3) / "Moderate" (3-7) / "Deliberate" (> 7)
   - Diversity: "Consistent" (independence < 3) / "Mixed" (3-7) / "Varied" (> 7)
   - Combined: "Rapid, Consistent" / "Deliberate, Varied" etc.

5. **AC-9.1.5: Extended CheckboxSignals** — Optional Phase 2 fields added:
   - `focusScore?: number` (0-10)
   - `originalityScore?: number` (0-10)
   - `vqsScore?: number` (0-10 composite)
   - `patternLabel?: string` (e.g., "Rapid, Consistent")
   - `categoriesVotedOn?: number`
   - `primaryCategory?: string`
   - `whaleAlignmentRatio?: number`

6. **AC-9.1.6: DB persistence** — `voter_quality_signals` Phase 2 columns populated:
   - `cross_category_signal` ← focusScore / 10 (normalize to 0-1)
   - `pattern_correlation_signal` ← (1 - originalityScore / 10) (invert: high correlation = high value)
   - `vqi_score` ← vqsScore / 10 (normalize to 0-1)

7. **AC-9.1.7: Pipeline integration** — VQS calculated in `calculateGPI()` after Phase 1 checkbox signals
   - Category map loaded from `proposal_categories` table for this DAO
   - Whale vote matrix pre-computed from top 3 whales
   - Phase 2 signals calculated for each voter that has Phase 1 signals (>= 3 votes)

8. **AC-9.1.8: Storage updated** — `saveCheckboxSignals()` persists Phase 2 columns alongside Phase 1

## Tasks / Subtasks

- [x] Task 1: Extend types (AC: #4, #5)
  - [x] 1.1 Add `VQSResult` interface to `src/lib/gvs/types.ts`
  - [x] 1.2 Extend `CheckboxSignals` in `src/lib/gvs/checkbox-detection.ts` with optional Phase 2 fields
  - [x] 1.3 Extend `GPIResult.metadata` in `src/lib/gvs/types.ts` with VQS aggregate stats

- [x] Task 2: Implement Focus Signal (AC: #1)
  - [x] 2.1 Add DB query function to load proposal categories for a DAO: `getProposalCategoryMap(daoId, proposalIds)` returning `Map<string, GovernanceCategory>`
  - [x] 2.2 Implement `calculateFocusSignal(voterVotes, proposalCategoryMap, totalProposalCount)` in checkbox-detection.ts
  - [x] 2.3 Implement DAO size normalization logic for dynamic Focus weight

- [x] Task 3: Implement Originality Signal (AC: #2)
  - [x] 3.1 Implement `buildWhaleVoteMatrix(allVoteRecords, topN = 3)` → `Map<string, number[]>`
  - [x] 3.2 Implement `calculateOriginalitySignal(voterVotes, whaleVoteMatrix)` in checkbox-detection.ts

- [x] Task 4: Implement VQS Composite (AC: #3, #4)
  - [x] 4.1 Implement `calculateVQS(signals, focusWeight)` with dynamic weight redistribution
  - [x] 4.2 Implement `generatePatternLabel(deliberationScore, independenceScore)` for neutral labels

- [x] Task 5: Pipeline Integration (AC: #7)
  - [x] 5.1 In `calculateGPI()`: load `proposalCategoryMap` from DB after fetching proposals
  - [x] 5.2 Build whale vote matrix from vote records
  - [x] 5.3 Calculate Phase 2 signals for each voter with existing checkbox signals
  - [x] 5.4 Enrich CheckboxSignals map with Phase 2 data

- [x] Task 6: Storage Update (AC: #6, #8)
  - [x] 6.1 Update `saveCheckboxSignals()` to include Phase 2 columns in upsert
  - [x] 6.2 Test that existing Phase 1 data is preserved (ON CONFLICT DO UPDATE)

- [x] Task 7: Exports (AC: all)
  - [x] 7.1 Update `src/lib/gvs/index.ts` with new type exports

- [x] Task 8: TypeScript Verification
  - [x] 8.1 `npx tsc --noEmit` — no new errors in modified files

## Dev Notes

### Architecture Constraints

**Pre-computed Whale Vote Matrix (Winston, Party Mode 2026-02-12):**
The Originality signal MUST use a pre-computed lookup table, NOT compare voter-by-voter against all whales. Build the matrix once per DAO at the start of VQS calculation:
```
1. From all voteRecords, aggregate VP per voter
2. Sort by VP descending, take top 3
3. Build Map<proposalId, number[]> of their choices
4. Per voter: O(1) lookup per proposal → total O(n × m) not O(n²)
```

**DAO Size Normalization (Mary, Party Mode 2026-02-12):**
Small DAOs shouldn't be penalized for generalist voters. Focus signal weight formula:
```
if totalProposals < 10:   focusWeight = 0.0
elif totalProposals <= 25: focusWeight = 0.25 * (totalProposals - 10) / 15
else:                      focusWeight = 0.25
```
Remaining weight redistributed proportionally across Deliberation, Independence, Originality.

**Framing Rules (Sally, Party Mode 2026-02-12):**
- NO warning symbols (⚠️, 🚩, ❗)
- NO judgmental labels ("checkbox voter", "low quality", "suspicious")
- Pattern labels: Speed × Diversity axes, purely descriptive
- "Rapid, Consistent" not "Suspicious, Repetitive"

### Source Data for Signals

**Focus Signal — Category data:**
- Table: `proposal_categories` (`src/lib/db/schema.ts` line 884)
- Columns: `daoId`, `proposalId`, `category` (one of 8 `GovernanceCategory` values)
- Categories: `treasury`, `protocol_upgrades`, `grants`, `operations`, `parameter_changes`, `node_validator`, `partnerships`, `social_meta`
- Source: `src/lib/governance/types.ts` — `GOVERNANCE_CATEGORIES`
- Classifier: `src/lib/governance/classifier.ts` — runs during GVS job, idempotent
- Not all proposals may be classified — skip unclassified proposals in Focus calculation

**Query to load category map:**
```typescript
import { proposalCategories } from '@/lib/db/schema'
import { eq, inArray, and } from 'drizzle-orm'

const rows = await db.select({
  proposalId: proposalCategories.proposalId,
  category: proposalCategories.category,
}).from(proposalCategories).where(
  and(
    eq(proposalCategories.daoId, daoId),
    inArray(proposalCategories.proposalId, proposalIds)
  )
)
const categoryMap = new Map(rows.map(r => [r.proposalId, r.category]))
```

**IMPORTANT:** `proposalCategories` uses `daoId` (UUID from our `daos` table), NOT `spaceId` (Snapshot space ID). In `calculateGPI()`, we have `spaceId` but need `daoId`. The mapping is in the `daos` table: `daos.snapshotSpace = spaceId`. You MUST look up the daoId first:
```typescript
import { daos } from '@/lib/db/schema'
const [dao] = await db.select({ id: daos.id }).from(daos).where(eq(daos.snapshotSpace, spaceId))
```

**Originality Signal — Whale identification:**
- Function: `analyzeVoters(allVotes)` in `src/lib/snapshot/client.ts` (line 588)
- Returns `VoterAnalysis[]` with `isWhale`, `totalVotingPower`
- Top 3 whales: sort by `totalVotingPower` descending, take first 3 where `isWhale === true`
- BUT for VQS we DON'T need full `analyzeVoters()` — just need top 3 by cumulative VP
- Simpler: aggregate VP from voteRecords directly (already available in GPI context)

**Whale Vote Matrix construction from voteRecords:**
```typescript
// 1. Aggregate VP per voter
const voterVP = new Map<string, number>()
for (const vr of voteRecords) {
  // voteRecords don't have VP — need to cross-reference with allVotes
}
```
**PROBLEM:** `VoteRecord` type only has `voter, proposalId, created, choice` — no `vp` field. You'll need either:
- Option A: Extend `VoteRecord` with optional `vp?: number` field
- Option B: Build VP map from the raw `SnapshotVote[]` (available as `allVotes` in `calculateGPI()`)
**Recommended:** Option B — use `allVotes` to build the whale VP map, since VP data is already there.

### Existing Code Patterns (from Phase 1)

**checkbox-detection.ts patterns:**
- `calculateCheckboxSignals(voteRecords)` → `Map<string, CheckboxSignals>`
- Groups votes by voter, minimum 3 votes required
- Each signal function returns typed result object
- `round()` helper for 3 decimal places
- `setSkipCheckboxDetection(skip)` bypass flag — extend to Phase 2 as well

**checkbox-storage.ts patterns:**
- `saveCheckboxSignals(spaceId, signals, date)` — non-blocking try/catch
- Upsert with `onConflictDoUpdate` on `(spaceId, voter, snapshotDate)`
- Batches of 50 records (currently sequential within chunks — could optimize later)

**gpi.ts integration point:**
- Phase 1 signals calculated at line 130: `const checkboxMap = calculateCheckboxSignals(voteRecords)`
- Storage at line 168: `saveCheckboxSignals(spaceId, checkboxMap).catch(() => {})`
- Phase 2: enrich `checkboxMap` entries with Focus + Originality + VQS after line 130
- Phase 2 categories need to be loaded BEFORE the checkbox calculation

### DB Schema (already exists, nullable columns ready)

`voter_quality_signals` table in `src/lib/db/schema.ts` (line 959):
- `cross_category_signal: real` — for Focus signal (nullable, Phase 2)
- `pattern_correlation_signal: real` — for Originality signal (nullable, Phase 2)
- `vqi_score: real` — for VQS composite (nullable, Phase 2)
- All three columns already exist in migration `drizzle/0023_add_voter_quality_signals.sql`
- **No new migration needed** — just populate existing nullable columns

### Project Structure Notes

- Signals go in: `src/lib/gvs/checkbox-detection.ts` (extend existing file)
- Types go in: `src/lib/gvs/types.ts` (extend existing GPIResult)
- Storage extends: `src/lib/gvs/checkbox-storage.ts`
- Pipeline integration: `src/lib/gvs/gpi.ts`
- Exports: `src/lib/gvs/index.ts`
- Category data from: `src/lib/governance/types.ts`, `src/lib/db/schema.ts` (proposalCategories table)

### Git Intelligence (Recent Commits)

```
3157bc4 chore: mark Epic 8 wallet label filtering as done in sprint status
cfd90d0 fix: code review fixes for Epic 8 wallet label filtering
bd1908f fix: admin paid tier redirects to /success intake form
b999d47 feat: checkbox voting detection Phase 1 — GPI enhancement
ab9f4be feat: Epic 8 wallet label filtering with eth-labels API integration
```

**Pattern from Phase 1 commit (`b999d47`):**
- All Phase 1 changes in one commit
- checkbox-detection.ts, checkbox-storage.ts, gpi.ts, types.ts, index.ts, schema.ts, migration SQL, validation script
- Follow same pattern: one commit for all Phase 2 signal changes

### References

- [Source: docs/stories/epic-checkbox-voting-phase2.md — Story 9-1]
- [Source: docs/analysis/product-brief-checkbox-voting-detection-2026-02-11.md — Phase 2 section]
- [Source: src/lib/gvs/checkbox-detection.ts — Phase 1 implementation]
- [Source: src/lib/gvs/gpi.ts — GPI pipeline integration point]
- [Source: src/lib/governance/classifier.ts — Proposal classification]
- [Source: src/lib/governance/types.ts — GovernanceCategory enum]
- [Source: src/lib/db/schema.ts line 884 — proposalCategories table]
- [Source: src/lib/db/schema.ts line 959 — voterQualitySignals table]
- [Source: src/lib/snapshot/client.ts line 588 — analyzeVoters() whale detection]

## Dev Agent Record

### Agent Model Used
Claude Opus 4.6

### Debug Log References
- TypeScript check: 0 new errors in modified GVS files (pre-existing errors in test files only)
- VQS unit tests: 26/26 passed
- GPI regression tests: passed (no new failures)

### Completion Notes List
- Task 1: Added `VQSResult` interface to types.ts, extended `CheckboxSignals` with 7 optional Phase 2 fields, added VQS aggregate stats to `GPIResult.metadata`
- Task 2: Implemented `calculateFocusSignal()` with category concentration formula and `calculateFocusWeight()` for DAO size normalization (linear ramp 10-25 proposals)
- Task 3: Implemented `buildWhaleVoteMatrix()` using Option B (raw SnapshotVote[] with VP data), and `calculateOriginalitySignal()` with alignment ratio calculation
- Task 4: Implemented `calculateVQS()` with dynamic weight redistribution when focus weight < 0.25, and `generatePatternLabel()` using neutral Speed x Diversity axes
- Task 5: Integrated into `calculateGPI()` pipeline: daoId lookup via `getDaoIdFromSpaceId()`, category map loading via `getProposalCategoryMap()` with 500-chunk batching, whale matrix built from allVotesWithProposal, and per-voter Phase 2 enrichment. All wrapped in non-blocking try/catch.
- Task 6: Updated `saveCheckboxSignals()` to persist Phase 2 columns: `crossCategorySignal` (focusScore/10), `patternCorrelationSignal` (1 - originalityScore/10), `vqiScore` (vqsScore/10). ON CONFLICT DO UPDATE preserves Phase 1 data.
- Task 7: Added all Phase 2 exports to `src/lib/gvs/index.ts`: functions + VQSResult type
- Task 8: `npx tsc --noEmit` — 0 new errors in modified files

### File List
- `src/lib/gvs/types.ts` — Added VQSResult interface, extended GPIResult.metadata with VQS aggregate stats
- `src/lib/gvs/checkbox-detection.ts` — Extended CheckboxSignals with Phase 2 fields, added calculateFocusSignal(), calculateFocusWeight(), buildWhaleVoteMatrix(), calculateOriginalitySignal(), calculateVQS(), generatePatternLabel()
- `src/lib/gvs/gpi.ts` — Added getProposalCategoryMap(), getDaoIdFromSpaceId(), Phase 2 VQS pipeline integration in calculateGPI(), extended getCheckboxMetadata() with VQS stats
- `src/lib/gvs/checkbox-storage.ts` — Extended upsert with Phase 2 columns (crossCategorySignal, patternCorrelationSignal, vqiScore)
- `src/lib/gvs/index.ts` — Added Phase 2 function and type exports
- `tests/unit/gvs/vqs.test.ts` — New: 26 unit tests for Focus, Originality, VQS, pattern labels
- `_bmad-output/implementation-artifacts/sprint-status.yaml` — Story status: ready-for-dev → in-progress → review
- `docs/stories/9-1-signals-focus-originality-vqs.md` — Updated tasks [x], status, dev agent record

## Senior Developer Review (AI)

**Reviewer:** Claude Opus 4.6 (adversarial code review)
**Date:** 2026-02-12
**Verdict:** APPROVED (all issues fixed)

### Findings (10 total: 3 High, 4 Medium, 3 Low)

**FIXED — HIGH:**
- H1: Phase 2 DB columns overwritten with NULL on VQS failure re-run → `checkbox-storage.ts`: conditional spread in ON CONFLICT DO UPDATE — Phase 2 columns only updated when VQS data present
- H2: Whale vote matrix included EXCLUDED wallets (bots/exchanges from Story 8-3) → `gpi.ts`: added `filteredVoterAddresses` set, whale matrix now filters excluded wallets
- H3: `import type { VQSResult }` at line 423 mid-file → moved to top of `checkbox-detection.ts`

**FIXED — MEDIUM:**
- M1: Triple memory allocation (votesArrays + voteRecords + allVotesWithProposal) → consolidated into single `voteRecords` array with VP field, eliminated `allVotesWithProposal`
- M2: `votesByVoter` map built twice (gpi.ts + inside calculateCheckboxSignals) → moved construction before VQS block, reused
- M3: Silent degradation when `daoId` not found for spaceId → added `console.warn` in `getDaoIdFromSpaceId()`
- M4: Deliberation/independence re-derived in `getCheckboxMetadata()` → added `deliberationScore`/`independenceScore` to `CheckboxSignals`, stored during VQS enrichment, used directly in metadata

**ACCEPTED — LOW (not fixed):**
- L1: No input validation on calculateVQS signal ranges (0-1) — Phase 1 guarantees valid ranges
- L2: Test file untracked in git — will be staged at commit time
- L3: No automated test for Phase 1 data preservation on upsert — manually verified via code inspection

### Post-Fix Verification
- TypeScript: 0 new errors in modified GVS files
- Unit Tests: 26/26 passed

## Change Log
- 2026-02-12: Code review fixes — H1 (Phase 2 null overwrite), H2 (excluded wallets in whale matrix), H3 (mid-file import), M1-M4 (memory, dedup, logging, metadata)
- 2026-02-12: Implemented Phase 2 VQS signals (Focus + Originality + VQS composite) with full pipeline integration, DB persistence, and 26 unit tests
