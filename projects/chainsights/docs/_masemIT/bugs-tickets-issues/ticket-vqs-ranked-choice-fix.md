# Ticket: VQS — Handle Ranked-Choice & Weighted Voting (Option A)

**Project:** ChainSights
**Priority:** High — 12% of all voters affected (505/4.303)
**Type:** Bugfix / Data Quality
**Created:** 2026-02-12 (Party Mode: Mary, Winston, Paige)
**Owner:** Winston (Architecture) + Dev
**Reviewer:** Mary (Validation), Paige (Methodology Page)
**Depends on:** Story 9-3 (done), Story 9-2 (done)
**Blocks:** Story 9-5 (Methodology Page v1.3)

---

## Root Cause (verified 2026-02-12)

DAOs using Ranked-Choice or Weighted Voting on Snapshot return `choice` as arrays (`[1,3,2]`) or objects (`{1: 60, 2: 40}`) instead of single integers.

Current code in `src/lib/gvs/checkbox-detection.ts` filters on `typeof vote.choice === 'number'` in three functions. When non-numeric choices are encountered, voters silently receive inflated default scores:

| Function | Behavior on Ranked/Weighted | Default |
|---|---|---|
| `calculateDiversitySignal` | `singleChoiceCount < 3` → skip | signal = 0 → **Independence = 10.0** |
| `buildWhaleVoteMatrix` | Whale votes skipped | Empty matrix |
| `calculateOriginalitySignal` | `comparedCount === 0` → skip | **Originality = 10.0** |

## Affected DAOs

| DAO | Voters | % Diversity=0 | % Pattern=0 | Voting Type |
|---|---|---|---|---|
| aavegotchi.eth | 281 | 100% | 100% | Ranked-Choice |
| balancer.eth | 17 | 100% | 100% | Weighted |
| frax.eth | 10 | 100% | 100% | Weighted/Gauge |
| stakedao.eth | 10 | 100% | 20% | Mixed |
| cvx.eth | 187 | 67% | 57% | Mixed |

## Solution: Option A — N/A with Composite Recalculation

### Backend Changes

1. **Detect voting type** per voter/proposal:
   - `typeof vote.choice === 'number'` → Single-Choice ✅
   - `Array.isArray(vote.choice)` → Ranked-Choice → mark as non-scorable
   - `typeof vote.choice === 'object'` → Weighted → mark as non-scorable

2. **Set Independence and Originality to `null`** (not 0, not 10) when insufficient single-choice data:
   - `calculateDiversitySignal`: return `null` instead of `0` when `singleChoiceCount < 3`
   - `calculateOriginalitySignal`: return `null` instead of `10.0` when `comparedCount === 0`

3. **VQS Composite recalculation** when signals are null:
   - All 4 signals available → 25% each (no change)
   - Independence = null → Deliberation 33%, Focus 33%, Originality 33%
   - Originality = null → Deliberation 33%, Independence 33%, Focus 33%
   - Both null → Deliberation 50%, Focus 50%
   - Rule: Redistribute weight equally among available signals

### Frontend Changes (Matrix Details Page)

4. **VQS bar + score**: Calculate and display based on available signals only

5. **Expanded detail view**: Show "N/A" for null signals with explanation:
   ```
   Deliberation:  ████████░░  8.3
   Independence:  N/A — Ranked-choice voting
   Focus:         ████░░░░░░  4.3
   Originality:   N/A — Ranked-choice voting
   
   Voting Pattern: Deliberate  (no diversity data available)
   ```

6. **Pattern label**: Omit diversity-based descriptors when Independence is null:
   - Full data: "Deliberate, Varied" / "Rapid, Consistent"
   - Partial data: "Deliberate" / "Rapid" (speed only)

### PDF Report (€149 Governance Audit)

7. Same N/A handling in the Key Governance Participants table and any VQS references

## Validation (John)

- [ ] Aavegotchi: VQS should drop significantly from current inflated scores
- [ ] frax.eth: Median should drop from 10.0 to a realistic Deliberation+Focus score
- [ ] Uniswap (single-choice): No change — regression check
- [ ] ENS (single-choice): No change — regression check
- [ ] cvx.eth (mixed): Partial recalculation for 67% of voters, rest unchanged
- [ ] Global median should shift down from 3.7 (recalculate after fix)

## Methodology Page v1.3 (Paige)

- [ ] Add section explaining VQS signal availability per voting type
- [ ] Document the three DAO archetypes: Consensus ("Rubber Stamp"), Mixed, Pluralistic
- [ ] Add global VQS benchmark: Median 3.7, Mean 4.3, Top-Quartil ≥5.0, Green (8+) 8.3%
- [ ] Note: Benchmark values to be updated after this fix is deployed

## Also Discussed: "Consensus Penalty" Problem

Separate from the ranked-choice bug, DAOs like Uniswap (median 3.3) and ENS (median 2.4) score low because their governance is uncontroversial — not because participation quality is low.

**Decision deferred** until this fix is deployed and benchmarks recalculated. Options:
- (A) Contextualise: show DAO-specific benchmark ("VQS 2.8 — typical for consensus-driven DAOs")
- (B) Weight adjustment: reduce Independence/Originality weight when DAO-wide vote diversity is low
- (C) Document only: explain the three archetypes in Methodology Page, no scoring change

## Out of Scope (Future Epic)

- **Option B**: Actually parsing Ranked-Choice/Weighted votes for meaningful Independence and Originality signals → separate backlog item
- **Consensus Penalty** decision (see above) — depends on post-fix benchmark data

## Effort

~1-2 sessions (AI-assisted)

## Global VQS Benchmark (pre-fix, 2026-02-12)

For reference — these numbers will shift after the fix is deployed:
- **4,303 voters across 24 DAOs**
- Median: 3.7/10, Mean: 4.3/10, Max: 10.0/10
- Distribution: 71% Orange (2-4.9), 15% Yellow (5-7.9), 8% Green (8+), 6% Red (0-1.9)

## Deployment Checklist

- [ ] Code changes in `checkbox-detection.ts` (diversitySignal, originalitySignal, VQS composite)
- [ ] Frontend N/A handling in `VQSExpandPanel.tsx` + `VQSBar.tsx`
- [ ] API response extended with nullable fields in `vqs/route.ts`
- [ ] Daily job triggered to recalculate all DAOs
- [ ] Validation script run on all 5 affected DAOs
- [ ] Global VQS benchmark recalculated and documented
- [ ] Methodology page v1.3 updated (Story 9-5, same deploy or immediately after)
- [ ] `project_context.md` Decision Log updated with post-fix benchmarks
