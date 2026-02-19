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

- [ ] "Last Proposal" column visible in Governance Index table on desktop
- [ ] Column is sortable (ascending/descending)
- [ ] Relative time format: `2d ago` (green), `3w ago` (default), `3mo ago` (amber), `2y ago` (red), `—` (no data)
- [ ] Column header has tooltip explaining the metric
- [ ] Detail view shows full date + proposal title + relative time
- [ ] Info note displayed for `score_basis: 'historical'` with appropriate wording
- [ ] New backfill chart marker appears on MetricChart trend charts
- [ ] **BUG FIX:** Both v1.1 and v1.2 marker tooltips appear on hover
- [ ] Mobile: column hidden, data accessible in expandable card view
- [ ] API response includes `lastProposalDate`, `lastProposalTitle`, `scoreBasis`

---

## Affected Files

| File | Change |
|------|--------|
| `src/lib/db/queries/leaderboard.ts` | Add `lastProposalDate`, `lastProposalTitle`, `scoreBasis` to query + interface |
| `src/app/api/matrix/route.ts` | Pass new fields through |
| `src/lib/utils/relative-time.ts` | New utility: `formatLastProposal()` |
| Rankings table component | Add "Last Proposal" column with sorting |
| DAO detail view component | Add last proposal section + info note |
| `src/components/MetricChart.tsx` | Add v1.2 marker, fix tooltip bug for v1.1 |
