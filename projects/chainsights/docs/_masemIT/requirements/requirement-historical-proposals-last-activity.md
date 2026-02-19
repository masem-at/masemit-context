# ChainSights: Historical Proposal Backfill & Last Activity Tracking

## Product Requirement Specification

**Version:** 1.0  
**Date:** 2026-02-19  
**Author:** Mario Semper (PO)  
**Status:** Ready for Development  
**Priority:** High  
**Relates to:** GVS Requirement v1.0 (gvs-requirement.md)

---

## 1. Problem Statement

### 1.1 Default Value Problem

Currently, 17 of 47 tracked DAOs (Ranks 29-47) display default component scores of `5.0 / 5.0 / 5.0` with a DGI of `3.3`. These defaults appear because the DAOs had no proposals within the current calculation window (rolling 90 days).

**Impact:**
- Destroys credibility of the entire Governance Index — visitors see Optimism, Curve, dYdX, Yearn, NounsDAO all showing identical scores
- Indistinguishable from calculated scores — users cannot tell if 5.0 is a real score or a placeholder
- Major DAOs like Optimism (Rank 29), Safe (Rank 30), Curve Finance (Rank 36) should not show dummy data

### 1.2 Data Loss Problem

When a DAO has no proposals within the rolling calculation window, previously calculated real scores are lost and replaced with defaults. There is no historical persistence of calculated scores.

### 1.3 Missing Activity Context

Users cannot distinguish between:
- A DAO with DGI 3.3 because governance is weak
- A DAO with DGI 3.3 because it has been inactive for 8 months

The `last_proposal` date is a critical context signal that is currently not tracked or displayed.

---

## 2. Solution Overview

Three changes, executed in order:

| # | Change | Type | Effort |
|---|--------|------|--------|
| 1 | **Historical Proposal Backfill Script** | One-time script + ongoing logic | Medium |
| 2 | **Persist `last_proposal_date` in DB** | Schema change + API update | Small |
| 3 | **Display `Last Proposal` in UI** | Frontend (Index table + Detail view) | Small |

---

## 3. Detailed Requirements

### 3.1 Historical Proposal Backfill Script

**Goal:** For every tracked DAO that currently shows default values (5.0/5.0/5.0), fetch historical proposals from Snapshot API regardless of age, calculate real DGI component scores, and persist them.

#### 3.1.1 Identification Logic

```
SELECT * FROM dao_scores 
WHERE delegate_engagement = 5.0 
  AND power_dynamics = 5.0 
  AND grassroots_participation = 5.0
```

All DAOs matching this pattern have default values and need backfill.

#### 3.1.2 Snapshot API Query

For each identified DAO, query the Snapshot GraphQL API:

```graphql
query {
  proposals(
    where: { space: "{space_id}" }
    orderBy: "created"
    orderDirection: desc
    first: 100
  ) {
    id
    title
    created
    end
    state
    votes
    scores
    scores_total
    choices
  }
}
```

Also fetch the associated votes for the retrieved proposals:

```graphql
query {
  votes(
    where: { proposal: "{proposal_id}" }
    first: 1000
  ) {
    voter
    choice
    vp
    created
  }
}
```

#### 3.1.3 Score Calculation

Use the **existing DGI/GVS calculation logic** to compute scores based on the fetched historical data. No changes to the scoring algorithm — only the data input changes (historical proposals instead of only recent ones).

#### 3.1.4 Persistence

Store the calculated scores in the existing `dao_scores` table, replacing the default values. Add a flag or timestamp indicating the data basis:

- `score_basis: 'historical'` — scored on proposals older than 90 days
- `score_basis: 'current'` — scored on proposals within rolling window (normal)

#### 3.1.5 Execution

- **One-time run** for initial backfill of all 17 DAOs with defaults
- **Ongoing:** When the daily/weekly recalculation job finds no proposals in the rolling window, it should **keep the last calculated score** instead of reverting to defaults. The score should only be replaced when new data produces a new calculation.

#### 3.1.6 Edge Cases

| Scenario | Handling |
|----------|----------|
| DAO has 0 proposals ever on Snapshot | Keep defaults, but mark as `score_basis: 'no_data'` |
| DAO has <3 proposals total | Calculate with available data, set confidence to `low` |
| Snapshot API rate limiting | Implement backoff; process DAOs sequentially with 2s delay |
| Proposal has 0 votes | Skip proposal in calculation |

---

### 3.2 Last Proposal Date Tracking

#### 3.2.1 Schema Change

Add to the DAO data model (table `daos` or equivalent):

```sql
ALTER TABLE daos ADD COLUMN last_proposal_date TIMESTAMP;
ALTER TABLE daos ADD COLUMN last_proposal_title VARCHAR(500);
ALTER TABLE daos ADD COLUMN last_proposal_id VARCHAR(100);
```

#### 3.2.2 Population

- **Backfill script** (3.1) populates `last_proposal_date` for all 47 DAOs as it fetches proposals
- **Daily/weekly recalculation job** updates `last_proposal_date` whenever new proposals are detected
- Value is the `created` timestamp of the most recent proposal in the Snapshot space

#### 3.2.3 API Response

Add `last_proposal` to the DAO score API response:

```json
{
  "dao_id": "curve.eth",
  "dgi": 3.3,
  "score_basis": "historical",
  "last_proposal": {
    "date": "2025-06-14T12:00:00Z",
    "title": "CRV Emissions Adjustment Q3",
    "id": "0xabc123...",
    "days_ago": 250
  },
  "components": { ... }
}
```

---

### 3.3 UI Changes

#### 3.3.1 Governance Index Table (Index Page)

Add a new column **"Last Proposal"** to the ranking table:

| Position | Before | After |
|----------|--------|-------|
| Column placement | After DGI score columns | New column after the last component score, before the trend arrow |

**Display format:**
- Recent (< 7 days): `2d ago` (green text)
- Recent (7-30 days): `3w ago` (default text color)
- Stale (1-6 months): `3mo ago` (yellow/amber text)
- Inactive (> 6 months): `11mo ago` (red text)
- No data: `—`

**Optional enhancement:** DAOs inactive > 6 months get a subtle visual indicator, e.g. reduced row opacity (0.7) or a small `⏸` icon.

#### 3.3.2 DAO Detail View

In the DAO detail/expanded view, add a **Last Proposal** section:

```
Last Proposal: June 14, 2025 — "CRV Emissions Adjustment Q3"
              (250 days ago)
```

If `score_basis` is `'historical'`, show an info note:

```
ℹ️ Scores based on most recent governance activity (June 2025). 
   No proposals detected in the last 90 days.
```

This is transparent and explains why scores might look different from actively governed DAOs.

#### 3.3.3 Sorting

The "Last Proposal" column should be **sortable** (ascending/descending). Default sort remains by DGI score.

#### 3.3.4 Chart Version Marker for Backfill

Add a new vertical marker line to all MetricChart trend charts (same pattern as the existing "v1.1" methodology marker) at the date the backfill is deployed.

- **Label:** `Backfill` (or short identifier, e.g. `v1.2`)
- **Style:** Vertical dashed line, consistent with existing v1.1 marker
- **Tooltip:** On hover, explain: *"Historical scores backfilled — default placeholder scores replaced with real governance data."*
- **Placement:** All MetricChart instances (Matrix detail pages, DAO detail views)
- **Implementation:** Extend the existing hardcoded marker array in `MetricChart.tsx`

> **BUG (existing):** The v1.1 methodology marker tooltip does not appear on hover. This must be fixed as part of this work — both v1.1 and the new backfill marker must show their tooltips correctly.

---

## 4. Acceptance Criteria

### 4.1 Backfill Script

- [ ] All 17 DAOs with default values (5.0/5.0/5.0) have been recalculated with real historical data
- [ ] No DAO shows 5.0/5.0/5.0 unless it truly has zero proposals on Snapshot
- [ ] `score_basis` flag correctly distinguishes `current` vs `historical` vs `no_data`
- [ ] Script is idempotent — running it twice produces same results
- [ ] Script logs which DAOs were processed and how many proposals were fetched

### 4.2 Last Proposal Tracking

- [ ] All 47 DAOs have a `last_proposal_date` populated
- [ ] Daily job updates `last_proposal_date` when new proposals appear
- [ ] API response includes `last_proposal` object

### 4.3 UI

- [ ] "Last Proposal" column visible in Governance Index table on desktop
- [ ] Column is sortable
- [ ] Relative time format with color coding as specified
- [ ] Detail view shows full date + proposal title
- [ ] Historical score info note displays for `score_basis: 'historical'`
- [ ] Mobile: Column can be hidden or shown in expandable card view

### 4.4 Data Integrity

- [ ] Daily recalculation job no longer reverts to defaults when no proposals in window
- [ ] Previously calculated scores persist until replaced by new calculations
- [ ] No data loss on recalculation runs

---

## 5. Technical Notes

### 5.1 Snapshot API Reference

- GraphQL endpoint: `https://hub.snapshot.org/graphql`
- Rate limits: ~30 requests/minute (respect `Retry-After` headers)
- Docs: See `snapshot-api-docu.md` in project context

### 5.2 Affected Files (likely)

- Scoring engine / calculation job
- Database schema (migration needed)
- API route returning DAO scores
- Governance Index page component
- DAO detail view component

### 5.3 Dependencies

- No new external dependencies
- Uses existing Snapshot API integration
- Uses existing DGI calculation logic

---

## 6. Out of Scope

- Changes to the DGI scoring algorithm itself
- Adding new DAOs to the tracked set
- Open-universe free check feature
- API endpoint changes beyond adding `last_proposal` and `score_basis`

---

## 7. Risks

| Risk | Mitigation |
|------|------------|
| Some Snapshot spaces may have API changes | Use existing Snapshot integration; test with 2-3 DAOs first |
| Historical proposals may produce very different scores than expected | Review results for top 5 DAOs manually before deploying |
| Migration may briefly show inconsistent data | Run backfill script before deploying UI changes |

---

## 8. Rollout Plan

1. **Step 1:** DB migration — add columns (`last_proposal_date`, `last_proposal_title`, `last_proposal_id`, `score_basis`)
2. **Step 2:** Run backfill script — populate all 47 DAOs with real scores and last proposal data
3. **Step 3:** Update daily job — prevent default fallback, update `last_proposal` on new proposals
4. **Step 4:** Deploy UI — add column to index, add section to detail view
5. **Step 5:** Verify — check all 47 DAOs show real data, no more 5.0/5.0/5.0 defaults

---

## 9. Addendum: Party Mode Review Decisions (2026-02-19)

Discussed by John (PM), Winston (Architect), Mary (Analyst), Paige (Tech Writer).

### 9.1 Verified Data

- **Affected DAOs: 17** (not 19 as originally estimated) — confirmed via DB query
- All 17 show identical pattern: HPR=0, DEI=5, PDI=5, GPI=5 → GVS=3.25
- **Bug:** All 17 report `confidence_level: high` and `data_completeness: 100` despite being pure defaults
- Test DAOs validated against Snapshot API:
  - Optimism: 93 proposals, last Jan 2023 (~3y ago), 62k votes
  - Curve: 100+ proposals, last Apr 2022 (~4y ago), 90 votes
  - NounsDAO: 21 proposals, last Mar 2023 (~3y ago), 37 votes
- All three have migrated governance off Snapshot (on-chain governance)

### 9.2 Architectural Decisions

| Decision | Rationale |
|----------|-----------|
| **Recalculation job = daily** | Confirmed by PO |
| **Daily job "never revert" = separate PR** | Behavioral change in production, must be independently testable (Winston) |
| **Backfill script = reusable CLI command** | Not a throwaway script — needed when onboarding new DAOs (Winston) |
| **Confidence-Stufen = separate follow-up** | Paige's 3-tier wording as basis: "Based on full activity" / "Based on limited data" / "No governance data available" |
| **Priority: eliminate dummy data first, UX-polish second** | PO decision |
| **`score_basis: 'historical'` is permanent for migrated DAOs** | These DAOs will never return to `current` on Snapshot |
| **`referenceDate` parameter for historical calculation** | **CRITICAL.** DEI recency decay (weight 0.1 for >90d) and PDI month-bucketing both use `Date.now()`, which crushes historical scores to near-zero. Fix: new `GVSOptions.referenceDate` param = date of last proposal. Backfill calculates "as of" the DAO's last active period. Daily job continues using `Date.now()`. No algorithm change — only the reference point shifts. Affects: `dei.ts` (recency weight), `pdi.ts` (month bucket boundaries), `hpr.ts`/`gpi.ts` (cutoff timestamp). |

### 9.5 Dry-Run Validation Results (2026-02-19)

| DAO | GVS | HPR | DEI | PDI | GPI | Duration | Notes |
|-----|-----|-----|-----|-----|-----|----------|-------|
| Optimism | 3.25 → **5.75** | 0→9.91 | 5→0.95 | 5→4.60 | 5→5.64 | 444s | 93 proposals, 2264 voters, Gini=0.95 |
| Curve | 3.25 → **5.64** | 0→10.0 | 5→0.16 | 5→5.01 | 5→5.49 | 121s | 100 proposals, 188 voters, Gini=0.71 |
| NounsDAO | 3.25 → **5.47** | 0→10.0 | 5→0.21 | 5→3.39 | 5→6.22 | 2.5s | 21 proposals, 45 voters, Gini=0.62 |

**Note:** DEI scores (0.16–0.95) are artificially low due to `Date.now()` recency decay. With `referenceDate` fix, DEI will be significantly higher. These results will be re-validated after implementing the `referenceDate` parameter.

### 9.3 Deliverable Structure

| Deliverable | Scope | Dependencies |
|-------------|-------|--------------|
| **A: Schema + Backfill** | DB migration + CLI backfill script for all 17 DAOs | None |
| **B: Daily Job Refactor** | "Never revert to defaults" guard | Can run parallel to A |
| **C: UI** | Last Proposal column, detail view, chart marker, tooltip bug fix | Depends on A |
| **Follow-up: Confidence Wording** | 3-tier text framework for score trustworthiness | After C |

### 9.4 Chart Marker & Bug Fix

- New backfill marker on MetricChart trend charts (same pattern as v1.1)
- **BUG:** Existing v1.1 marker tooltip does not appear — must be fixed

---

*ChainSights – Wallets lie. We don't.*
