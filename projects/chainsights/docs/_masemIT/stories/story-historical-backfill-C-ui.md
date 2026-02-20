# Story C: UI — Last Proposal Column, Detail View, Chart Marker

**Parent Requirement:** `docs/_masemIT/requirements/requirement-historical-proposals-last-activity.md`
**Priority:** P1 — User-facing visibility of last activity + score transparency
**Effort:** Medium (1-2 days)
**Dependencies:** Story A (schema + backfill must be deployed first)

---

## Goal

Add a "Last Proposal" column to the Governance Index table, show last proposal details in the DAO detail view, add a backfill chart marker, and fix the existing v1.1 tooltip bug.

---

## Subtasks

- [x] C1: API Response — Add `lastProposal` and `scoreBasis`
- [x] C2: "Last Proposal" Column in Governance Index Table
- [x] C3: DAO Detail View — Last Proposal Section
- [x] C4: Chart Marker for Backfill
- [x] C5: Bug Fix — v1.1 Marker Tooltip Not Appearing

### C1: API Response — Add `lastProposal` and `scoreBasis`

**File:** `src/lib/db/queries/leaderboard.ts`

Extend `LeaderboardRanking` interface:

```typescript
interface LeaderboardRanking {
  // ... existing fields ...
  lastProposalDate: Date | null
  lastProposalTitle: string | null
  scoreBasis: 'current' | 'historical' | 'no_data' | null
}
```

Query the new `daos.last_proposal_date`, `daos.last_proposal_title` and `gvs_snapshots.score_basis` columns and include in response.

**File:** `src/app/api/matrix/route.ts` — Pass new fields through.

### C2: "Last Proposal" Column in Governance Index Table

**Placement:** After the last component score column, before the trend arrow/sparkline.

**Display format utility** (`src/lib/utils/relative-time.ts`):

```typescript
function formatLastProposal(date: Date | null): { text: string; colorClass: string } {
  if (!date) return { text: '—', colorClass: 'text-muted-foreground' }

  const daysAgo = Math.floor((Date.now() - date.getTime()) / (1000 * 60 * 60 * 24))

  if (daysAgo < 7)   return { text: `${daysAgo}d ago`, colorClass: 'text-green-500' }
  if (daysAgo < 30)   return { text: `${Math.floor(daysAgo / 7)}w ago`, colorClass: 'text-foreground' }
  if (daysAgo < 365)  return { text: `${Math.floor(daysAgo / 30)}mo ago`, colorClass: 'text-amber-500' }
  return { text: `${Math.floor(daysAgo / 365)}y ago`, colorClass: 'text-red-500' }
}
```

**Column header:** "Last Proposal" with tooltip icon: *"Recency of governance activity on Snapshot"*

**Sortable:** Yes, ascending/descending. Default sort remains GVS score.

**Mobile:** Hidden by default, visible in expandable card view.

### C3: DAO Detail View — Last Proposal Section

In the detail/expanded view, add:

```
Last Proposal: January 12, 2023 — "Special Voting Cycle #9b: Protocol Delegation Elections"
               (3 years ago)
```

If `score_basis === 'historical'`, show info note:

```
ℹ️ This DAO's last Snapshot proposal was in January 2023.
   Scores reflect its most recent Snapshot governance activity.
```

**Wording note:** Do NOT imply the DAO is inactive — many have migrated to on-chain governance. Focus on the data source limitation.

### C4: Chart Marker for Backfill

**File:** `src/components/MetricChart.tsx` (or equivalent chart component)

Add a new vertical marker at the backfill deployment date to the existing methodology markers array:

```typescript
{ date: '2026-02-XX', label: 'v1.2', tooltip: 'Historical scores backfilled — default placeholder scores replaced with real governance data.' }
```

Same visual style as the existing v1.1 marker (vertical dashed line).

### C5: Bug Fix — v1.1 Marker Tooltip Not Appearing

**BUG:** The existing v1.1 methodology version marker tooltip does not appear on hover.

**Investigate and fix:** Check the Recharts tooltip/reference line interaction. Both v1.1 and the new v1.2 marker must show their tooltips correctly on hover.

---

## Acceptance Criteria

- [x] "Last Proposal" column visible in Governance Index table on desktop
- [x] Column is sortable (ascending/descending)
- [x] Relative time format: `2d ago` (green), `3w ago` (default), `3mo ago` (amber), `2y ago` (red), `—` (no data)
- [x] Column header has tooltip explaining the metric
- [x] Detail view shows full date + proposal title + relative time
- [x] Info note displayed for `score_basis: 'historical'` with appropriate wording
- [x] New backfill chart marker appears on MetricChart trend charts
- [x] **BUG FIX:** Both v1.1 and v1.2 marker tooltips appear on hover
- [x] Mobile: column hidden, data accessible in expandable card view
- [x] API response includes `lastProposalDate`, `lastProposalTitle`, `scoreBasis`

---

## Affected Files

| File | Change |
|------|--------|
| `src/lib/db/queries/leaderboard.ts` | Add `lastProposalDate`, `lastProposalTitle`, `scoreBasis` to query + interface |
| `src/app/api/matrix/route.ts` | Pass new fields through (sort type updated) |
| `src/lib/utils/relative-time.ts` | New utility: `formatLastProposal()`, `formatFullDate()` |
| `src/components/RankingsTable.tsx` | Add "Last Proposal" column header with tooltip |
| `src/components/ExpandableDAORow.tsx` | Add "Last Proposal" cell + import utility |
| `src/components/MobileDAOCard.tsx` | Add last proposal in expanded section |
| `src/components/matrix-detail/DaoHeader.tsx` | Add last proposal section + historical info note |
| `src/components/matrix-detail/MetricChart.tsx` | Add v1.2 marker, fix tooltip bug with custom SVG label |
| `src/app/matrix/[slug]/page.tsx` | Pass lastProposalDate, lastProposalTitle, scoreBasis through |
| `src/app/matrix/[slug]/matrix-detail-client.tsx` | Update dao prop type |
| `src/app/rankings/page.tsx` | Update sortBy type union + validSortBy whitelist |
| `tests/unit/lib/utils/relative-time.test.ts` | New: 7 unit tests for formatLastProposal, formatFullDate |
| `tests/unit/components/expandable-dao-row.test.tsx` | Fix stale labels + add Story C Last Proposal tests |
| `tests/unit/components/mobile-dao-card.test.tsx` | Fix stale labels + add Story C Last Proposal tests |

---

## Dev Agent Record

### Implementation Plan

- C1: Extended `LeaderboardRanking` with 3 new fields, sourced from `dao.lastProposalDate/lastProposalTitle` and `snapshot.scoreBasis`. Added `lastProposal` sort option.
- C2: Created `relative-time.ts` utility. Added column to `RankingsTable` (header with (i) tooltip), `ExpandableDAORow` (desktop cell), `MobileDAOCard` (expanded section).
- C3: Extended `DaoHeader` props with lastProposal/scoreBasis. Added full date + title display and blue info banner for `score_basis: 'historical'`.
- C4: Added v1.2 entry to `METHODOLOGY_VERSIONS` array with date `2026-02-20`.
- C5: Root cause: Recharts `ReferenceLine` label is plain SVG text — no hover interaction. Fix: Custom `MethodologyLabel` component with SVG `<title>` for native browser tooltips + styled pill background for visual clarity.

### Debug Log

- No issues encountered during implementation.
- Pre-existing test failures in `expandable-dao-row.test.tsx` and `mobile-dao-card.test.tsx` — tests reference old component labels ("Human Participation Rate" vs current "Sybil Resistance Score"). Not caused by Story C changes.

### Completion Notes

- All 5 subtasks implemented and verified via TypeScript compilation (0 source errors).
- New unit tests: 7 tests for `formatLastProposal` and `formatFullDate` — all passing.
- Column placement: After Health Status, before Category (desktop). In expanded section (mobile).
- Info note wording follows story guidance: focuses on data source limitation, doesn't imply DAO inactivity.

---

## Change Log

| Date | Change |
|------|--------|
| 2026-02-20 | Story C implemented: Last Proposal column, detail view section, v1.2 chart marker, v1.1 tooltip bug fix |
| 2026-02-20 | Code Review: Fixed 5 issues (2 HIGH, 3 MEDIUM). H1: Added lastProposal to validSortBy. H2: Fixed stale test labels. M1: Fixed info note grammar. M2: Added Story C UI tests. M3: Added test files to File List. |

---

## Status

done
