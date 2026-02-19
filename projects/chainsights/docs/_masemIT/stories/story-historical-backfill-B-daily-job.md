# Story B: Daily Job Refactor — Never Revert to Defaults

**Parent Requirement:** `docs/_masemIT/requirements/requirement-historical-proposals-last-activity.md`
**Priority:** P0 — Prevents re-introduction of dummy data
**Effort:** Small (half day)
**Dependencies:** None (can run parallel to Story A)
**Blocks:** Nothing (independent)

---

## Goal

Modify the daily GVS recalculation job so it never overwrites real scores with default values when no proposals are found in the rolling window. Previously calculated scores must persist until replaced by a new calculation with real data.

---

## Problem

Currently, the daily job (`/api/jobs/gvs-calculate`) runs `calculateGVS()` for every DAO. When a DAO has no proposals within the lookback window, the components return neutral defaults (DEI=5, PDI=5, GPI=5, HPR=0) and a new snapshot is written with `GVS=3.25`. This overwrites any previously calculated real scores.

After Story A backfills historical scores, the daily job would **immediately revert them to defaults** on the next run. This story prevents that.

---

## Implementation

### B1: Guard in `/api/jobs/gvs-calculate/route.ts`

Before calling `saveGVSSnapshot()`, check if the result contains only default values:

```typescript
function isDefaultResult(gvsResult: GVSResult): boolean {
  const hpr = gvsResult.components.hpr
  const dei = gvsResult.components.dei
  const pdi = gvsResult.components.pdi
  const gpi = gvsResult.components.gpi

  // Check if all available components returned neutral/zero scores
  // indicating no proposals were found in the lookback window
  const hprIsDefault = hpr === null || (hpr.score === 0 && hpr.metadata.proposalsAnalyzed === 0)
  const deiIsDefault = dei === null || (dei.score === 5 && dei.metadata.proposalsAnalyzed === 0)
  const pdiIsDefault = pdi === null || (pdi.score === 5 && pdi.metadata.monthsAnalyzed === 0)
  const gpiIsDefault = gpi === null || (gpi.score === 5 && gpi.metadata.proposalsAnalyzed === 0)

  return hprIsDefault && deiIsDefault && pdiIsDefault && gpiIsDefault
}
```

**When `isDefaultResult()` is true:**
- Do NOT write a new snapshot
- Log: `[GVS] Skipping snapshot for ${daoName}: no proposals in window, keeping last real score`
- Still update `last_proposal_date` check (fetch latest proposal date)
- Return `{ skipped: true, reason: 'no_new_data' }` in job response

**When `isDefaultResult()` is false:**
- Normal behavior: write snapshot with `score_basis: 'current'`
- Update `last_proposal_date` if new proposals detected

### B2: Update Daily Job Metrics

Add `skippedCount` to the job response for monitoring:

```json
{
  "success": true,
  "daosProcessed": 47,
  "daosCalculated": 30,
  "daosSkipped": 17,
  "failures": 0
}
```

### B3: Update `last_proposal_date` on Every Run

Even when a GVS snapshot is skipped, the daily job should still check for new proposals and update `last_proposal_date` on the `daos` table. This ensures the "Last Proposal" column stays current.

---

## Acceptance Criteria

- [ ] Daily job does not write default-value snapshots (5.0/5.0/5.0) to the database
- [ ] Previously calculated scores (from backfill or normal calculation) persist when no new proposals exist
- [ ] Job logs clearly indicate which DAOs were skipped and why
- [ ] Job response includes `daosSkipped` count
- [ ] `last_proposal_date` still updated even for skipped DAOs
- [ ] DAOs that receive new proposals get normal `score_basis: 'current'` snapshots
- [ ] No impact on DAOs that currently have active proposals (regression test)

---

## Affected Files

| File | Change |
|------|--------|
| `src/app/api/jobs/gvs-calculate/route.ts` | Add `isDefaultResult()` guard before `saveGVSSnapshot()` |
| `src/lib/jobs/daily-recalculation.ts` | Add `skippedCount` to metrics |

---

## Testing

- **Unit test:** `isDefaultResult()` with various component combinations
- **Integration test:** Mock a DAO with no proposals, verify no snapshot written
- **Regression test:** DAO with active proposals still gets snapshots normally

---

## Separate PR

This is intentionally a separate PR from Story A because:
1. It changes **production behavior** of the daily job
2. Must be independently testable and reviewable
3. Can be merged and deployed independently of the backfill
4. If it causes issues, can be reverted without affecting the backfill
