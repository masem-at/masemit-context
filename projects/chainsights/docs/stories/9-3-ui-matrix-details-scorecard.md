# Story 9.3: UI — Matrix Details Scorecard + Expand Panel

Status: done

## Story

As a Matrix Details page visitor,
I want to see delegate vote quality scores in a dedicated section,
so that I can assess the quality of governance participation in a DAO.

## Acceptance Criteria

1. **AC-9.3.1: Delegate Vote Quality section** — New section on Matrix Details page below the 5 metric charts
   - Section title: "Delegate Vote Quality"
   - Shows a table of voters with VQS data
   - Table columns: Address (truncated), VQS Score (colored bar + number), Pattern Label, Votes Analyzed

2. **AC-9.3.2: VQS score display** — Colored bar + numeric score (0-10)
   - **Color gradient (Sally):** Green (8-10) → Yellow (5-7) → Orange (2-4) → Light red (0-2)
   - **No** warning symbols (`⚠️`, `🚩`, `❗`) anywhere
   - **No** judgmental labels ("checkbox voter", "low quality", "suspicious")

3. **AC-9.3.3: Expandable signal breakdown** — Click/tap on any voter row opens expand panel showing 4 signal scores:
   ```
   0x7eE0...bb6C — Vote Quality: 4.1/10

     Deliberation:  ██░░░░░░░░  2.1
     Independence:  ███░░░░░░░  3.4
     Focus:         ██████░░░░  6.2
     Originality:   ████░░░░░░  4.5

     Voting Pattern: Rapid, Consistent
   ```

4. **AC-9.3.4: Signal bars use same color gradient** — Each signal bar uses green→orange gradient per its individual score

5. **AC-9.3.5: Pattern label** — Shown as neutral tag: "Rapid, Consistent" / "Deliberate, Varied" etc.

6. **AC-9.3.6: Data source** — Fetched from `/api/dao/[spaceId]/vqs` endpoint (Story 9-2, done)

7. **AC-9.3.7: Graceful fallback** — If VQS data not yet available: section shows "Vote quality data is being calculated..." message

8. **AC-9.3.8: Mobile responsive** — Expand panel stacks vertically on small screens, table scrolls horizontally if needed

9. **AC-9.3.9: Accordion behavior** — Only one row expanded at a time (clicking another collapses the previous)

## Tasks / Subtasks

- [x] Task 1: VQS color utility (AC: 2, 4)
  - [x] 1.1 Create `src/lib/utils/vqs-colors.ts` with `vqsToColor(score: number): string` mapping 0-10 → hex
    - Green (8-10): `#22C55E` → `#16A34A`
    - Yellow (5-7): `#EAB308` → `#CA8A04`
    - Orange (2-4): `#F97316` → `#EA580C`
    - Light red (0-2): `#EF4444` → `#DC2626`
    - Use smooth interpolation, not hard steps
  - [x] 1.2 Create `vqsToTailwindClass(score: number): string` for text/bg Tailwind classes

- [x] Task 2: VQS Bar component (AC: 2, 4)
  - [x] 2.1 Create `src/components/matrix-detail/VQSBar.tsx` — horizontal progress bar
    - Props: `score: number` (0-10), `label?: string`, `compact?: boolean`
    - Width: `score / 10 * 100`%
    - Background color from `vqsToColor(score)`
    - Compact mode for table cell, full mode for expand panel
  - [x] 2.2 Show numeric score next to bar (e.g., "4.1")

- [x] Task 3: VQS Expand Panel component (AC: 3, 4, 5, 9)
  - [x] 3.1 Create `src/components/matrix-detail/VQSExpandPanel.tsx`
    - Props: voter data object from VQS API response
    - Shows 4 signal bars: Deliberation, Independence, Focus, Originality
    - Shows pattern label as neutral badge/tag
    - Shows total votes analyzed
  - [x] 3.2 Each signal bar labeled: "Deliberation", "Independence", "Focus", "Originality"
  - [x] 3.3 Smooth expand/collapse animation (CSS transition `max-height` or `grid-rows`)

- [x] Task 4: Delegate Vote Quality section (AC: 1, 6, 7, 8, 9)
  - [x] 4.1 Create `src/components/matrix-detail/DelegateVoteQuality.tsx` — main section component
    - Props: `spaceId: string`
    - Client component with `useEffect` fetch from `/api/dao/${spaceId}/vqs`
    - Loading state: skeleton shimmer
    - Empty state: "Vote quality data is being calculated..." message
    - Error state: silent fail, hide section
  - [x] 4.2 Render voter table:
    - Columns: Address (truncated `0x1234...5678`), VQS (colored bar + number), Pattern, Votes
    - Rows clickable to expand signal breakdown
    - Accordion: one open at a time
  - [x] 4.3 Show `totalVotersAnalyzed` in section subtitle: "Analyzing {N} delegates"
  - [x] 4.4 Mobile responsive: collapse Pattern + Votes columns on `< md`, expand panel stacks vertically

- [x] Task 5: Wire into Matrix Details page (AC: 1, 6)
  - [x] 5.1 In `src/app/matrix/[slug]/matrix-detail-client.tsx`: add `<DelegateVoteQuality spaceId={dao.snapshotSpace} />` below GovernanceIndex
  - [x] 5.2 Ensure `snapshotSpace` is already passed in `dao` prop (it is — verified at `page.tsx:179`)

- [x] Task 6: Tests (AC: all)
  - [x] 6.1 Unit test `vqs-colors.ts`: color mapping for edge values (0, 5, 10)
  - [x] 6.2 Unit test `VQSBar`: renders correct width and color
  - [x] 6.3 Unit test `DelegateVoteQuality`: loading state, empty state, renders voters, expand/collapse

## Dev Notes

- **CRITICAL: No "Key Governance Participants" table exists** — The epic (AC-9.3.1) references adding a column to an existing table, but this table does NOT exist on the Matrix Details page. The implementation creates a NEW "Delegate Vote Quality" section instead. This is the correct approach based on the actual codebase.

- **Epic file paths are WRONG** — The epic lists `src/app/dao/[spaceId]/components/...` but the Matrix Details page lives at `src/app/matrix/[slug]/`. All new components go in `src/components/matrix-detail/` following existing patterns (DaoHeader, MetricChart, GovernanceIndex are all there).

- **SpaceId available via props** — `MatrixDetailClient` already receives `dao.snapshotSpace` (verified at `matrix-detail-client.tsx:26` and `page.tsx:179`). The VQS API uses `spaceId` = `dao.snapshotSpace`.

- **Client-side fetch pattern** — Follow AttentionMap pattern (`attention-map.tsx`): `useEffect` + `fetch` + `useState`. NOT SWR (project doesn't use SWR). VQS data is non-blocking — page loads normally, VQS section populates async.

- **Color gradient: Sally's rules** — No warning symbols, no judgmental text. Color gradient is green→yellow→orange→light-red, purely informational. Reference: `MetricChart.tsx` has `getScoreColor()` with similar logic but for Tailwind classes. VQS needs hex colors for bars.

- **Accordion pattern** — Follow `ExpandableDAORow.tsx` pattern for expand/collapse: `useState` for expanded voter address, CSS `transition-all duration-300`. Keyboard support: Enter/Space to toggle, Escape to close.

- **Address truncation** — Use `0x${addr.slice(2,6)}...${addr.slice(-4)}` for display. Existing pattern in `ExpandableDAORow` shows addresses this way.

- **Signal descriptions simplified** — The epic AC-9.3.5 calls for raw numbers ("Avg Xs between votes"), but these raw values are NOT in the VQS API response (Story 9-2 only returns 0-10 scores). For Phase 1 simplicity, show only the 4 signal bars with numeric scores. Signal descriptions can be added later when the API is extended with raw descriptors. Do NOT call additional APIs or DB queries for this.

### Previous Story Intelligence (9-1, 9-2)

**From Story 9-1 (done):**
- Phase 2 signals stored in `voter_quality_signals` table: `cross_category_signal`, `pattern_correlation_signal`, `vqi_score`
- DB stores 0-1 range, API converts to 0-10
- `generatePatternLabel()` uses Speed × Diversity axes: "Rapid"/"Moderate"/"Deliberate" × "Consistent"/"Mixed"/"Varied"
- Code review caught: excluded wallets must be filtered from whale matrix, Phase 2 columns conditionally updated to preserve Phase 1 data

**From Story 9-2 (done):**
- VQS API: `GET /api/dao/[spaceId]/vqs`
- Response: `{ spaceId, calculationDate, totalVotersAnalyzed, voters: [{ address, vqsScore, deliberationScore, independenceScore, focusScore, originalityScore, patternLabel, totalVotesAnalyzed, checkboxScore }] }`
- Default limit: 50, max 200. Sorted by `vqi_score ASC` (lowest quality first)
- Cache: `s-maxage=3600, stale-while-revalidate=1800`
- Code review caught: `checkboxScore` should NOT be rounded (raw DB value), unused import `desc` removed

### Project Structure Notes

- New files go in `src/components/matrix-detail/` (alongside DaoHeader.tsx, MetricChart.tsx, GovernanceIndex.tsx)
- Color utility goes in `src/lib/utils/vqs-colors.ts` (new file, shared between UI and future PDF)
- Tests in `tests/unit/components/` following project convention
- All components are React client components (`'use client'`)
- Styling: Tailwind CSS, `font-montserrat` for headings, dark theme (`bg-card`, `text-gray-300`, `border-border`)
- Icons: Lucide React (already installed) — use `ChevronDown` for expand indicator

### Existing Component Patterns to Follow

**DaoHeader.tsx** — Score display with badges:
```typescript
// Score color: getScoreColor(score) → 'text-green-400' | 'text-yellow-400' | 'text-red-400'
// Layout: flex with gap, responsive grid
```

**AttentionMap.tsx** — Client-side data fetching:
```typescript
useEffect(() => {
  fetch(`/api/matrix/${slug}/attention?period=last_90d`)
    .then(res => res.json())
    .then(data => setData(data))
}, [slug])
```

**ExpandableDAORow.tsx** — Expand/collapse pattern:
```typescript
const [isExpanded, setIsExpanded] = useState(false)
// ARIA: role="button", tabIndex={0}, aria-expanded={isExpanded}
// Keyboard: onKeyDown Enter/Space → toggle, Escape → close
// Animation: transition-all duration-300
```

### References

- [Source: docs/stories/epic-checkbox-voting-phase2.md#Story 9-3] — Epic AC spec
- [Source: src/app/matrix/[slug]/matrix-detail-client.tsx] — Integration point, dao.snapshotSpace prop
- [Source: src/app/matrix/[slug]/attention-map.tsx] — Client-side fetch pattern
- [Source: src/components/matrix-detail/DaoHeader.tsx] — Score display pattern
- [Source: src/components/matrix-detail/MetricChart.tsx] — Color mapping, getScoreColor()
- [Source: src/components/matrix-detail/GovernanceIndex.tsx] — Bar visualization (bg-aqua, width scaling)
- [Source: src/components/ExpandableDAORow.tsx] — Expand/collapse pattern, keyboard support, ARIA
- [Source: src/app/api/dao/[spaceId]/vqs/route.ts] — VQS API response shape (Story 9-2)
- [Source: docs/project_context.md] — Tech stack, Tailwind, dark theme, no judgmental UI

## Dev Agent Record

### Agent Model Used

Claude Opus 4.6

### Debug Log References

No debug issues encountered.

### Completion Notes List

- **Task 1:** Created `vqs-colors.ts` with smooth color interpolation (6 color stops, linear lerp between them). `vqsToColor()` returns hex, `vqsToTailwindClass()` returns discrete Tailwind classes. 15 unit tests pass.
- **Task 2:** Created `VQSBar.tsx` — horizontal bar with inline width/color from `vqsToColor()`. Compact mode (table) vs full mode (expand panel). 7 unit tests pass.
- **Task 3:** Created `VQSExpandPanel.tsx` — shows 4 signal bars (Deliberation, Independence, Focus, Originality) via `VQSBar` + pattern label badge + votes count. Expand/collapse uses CSS `max-height` transition (300ms).
- **Task 4:** Created `DelegateVoteQuality.tsx` — main section with `useEffect` fetch from VQS API (AttentionMap pattern). Skeleton shimmer loading, empty state message, silent fail on error. Voter table with accordion expand (one-at-a-time via `expandedVoter` state). Keyboard accessible (Enter/Space/Escape). Mobile: Pattern + Votes columns hidden on `< md`, table scrolls horizontally.
- **Task 5:** Wired `<DelegateVoteQuality spaceId={dao.snapshotSpace} />` into `matrix-detail-client.tsx` below `GovernanceIndex`. Import added. `snapshotSpace` prop verified available.
- **Task 6:** 32 unit tests across 3 test files all pass. No regressions in existing VQS tests (69 VQS-related tests all green). Pre-existing test failures in auth/stripe/admin tests are NOT related to this story.

### Change Log

- 2026-02-12: Implemented Story 9-3 — Delegate Vote Quality section with VQS scorecard, colored bars, expand panel, and tests.
- 2026-02-12: Code Review (AI) — 6 issues fixed: color gradient spec compliance (H-1), unused import removed (M-1), act() test warnings resolved (M-2), VQSExpandPanel unit test added (M-3), keyboard accessibility tests added (M-4), network failure test added (M-5). Tests: 32 → 44 passing.

### File List

New files:
- src/lib/utils/vqs-colors.ts
- src/components/matrix-detail/VQSBar.tsx
- src/components/matrix-detail/VQSExpandPanel.tsx
- src/components/matrix-detail/DelegateVoteQuality.tsx
- tests/unit/lib/vqs-colors.test.ts
- tests/unit/components/vqs-bar.test.tsx
- tests/unit/components/vqs-expand-panel.test.tsx
- tests/unit/components/delegate-vote-quality.test.tsx

Modified files:
- src/app/matrix/[slug]/matrix-detail-client.tsx (added DelegateVoteQuality import + render)
- _bmad-output/implementation-artifacts/sprint-status.yaml (9-3 → in-progress → review → done)
- docs/stories/9-3-ui-matrix-details-scorecard.md (tasks checked, status → done)
