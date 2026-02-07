# Feature Requirements: Controlling View

**Document Version:** 1.1
**Created:** 2026-02-07
**Updated:** 2026-02-07 (Party Mode review — all open questions resolved)
**Author:** Mario Semper (Product Owner)
**Status:** Ready for Implementation
**Target Release:** Next feature release (post Big Bang v2.0)

---

## Executive Summary

Add a **Controlling View** as the fourth departmental perspective in tellingCube, completing the "4 Perspectives" product story: Finance + Sales + HR + **Controlling**. The view focuses on Plan vs. Actual variance analysis, cost center breakdowns, and KPI dashboards — the core toolkit for controllers and BI consultants.

**Tier Availability:** Lifetime+ (€299) / Pro (€19/mo) / Team (€49/mo) only.

---

## Background & Strategic Context

### Why Now

- **Jürgen Faisst (IBCS Institute CEO)** explicitly identified Plan vs. Actual and variance analysis as a critical gap in December 2025
- The Controlling View rounds out the multi-view consistency story — tellingCube's core differentiator
- BI Consultants (Priority 1 audience) and Software Vendors (Priority 2) expect controlling perspectives in realistic business data
- Strengthens the upgrade path from Lifetime (€99) → Lifetime+ (€299)

### Feature Stacking (Tier Differentiation)

| Tier | Views Available |
|------|----------------|
| Single (€9) / Lifetime (€99) | Finance + Sales |
| Lifetime+ (€299) / Pro / Team | Finance + Sales + HR + **Controlling** |

This creates a clear "want the full picture? upgrade" narrative.

---

## Technical Decision: Option A — Derived Comparison Data

### Context

Winston (Architect) confirmed the following about Plan/Budget data in tellingCube:

1. **Generated events are Actuals only** — all `sales_transaction`, `operational_work`, `procurement`, `other_expense`, `cash_movement`, and HR events are AC data
2. **Planning events exist but are minimal** — `planning_budgeting` events have `amountEur = 0` with values only in metadata fields
3. **PY/PL/FC values are calculated at query time** — `lib/utils/comparison-data.ts` derives comparison values deterministically:
   - **PY** = AC / (1 + historical growth rate) — reverse-engineered
   - **PL** = PY × (1 + ambitious plan factor) — synthetically derived
   - **FC** = PY × (1 + realistic forecast factor) — synthetically derived
   - All using industry-specific growth rates

### Decision: Use Option A (Derived Data)

**We will build the Controlling View on top of the existing `comparison-data.ts` logic.** No changes to event generation required.

**Rationale:**
- The derived PL/FC values are deterministic, branchenspezifisch, and "look right" — sufficient for demos, POCs, and training
- No risk of breaking existing Finance/Sales/HR views
- Significantly faster to ship (view layer + API only, no generation refactoring)
- Upgrade path remains open: Option B (real Plan events) can be added later as a Premium/Enterprise feature if demand validates it

**What this means for the team:**
- **No changes to event generation** (`lib/generators/`, prompts, etc.)
- **No schema changes** for events table
- **Reuse `comparison-data.ts`** as the data foundation for all PL/FC/PY values
- **New work is purely:** API endpoint + view components + charts

**Scope boundary — FC explicitly excluded from v1:** Forecast (FC) data exists in `comparison-data.ts` but is NOT displayed in this release. The Controlling View shows AC vs. PL only. FC can be added in a future iteration.

---

## Key Definitions

### Terminology

- **"Plan" (PL)** is the standard term throughout the Controlling View — consistent with IBCS notation. "Budget" is only used when referring specifically to cost budgets. All user-facing labels, API fields, and documentation use "Plan".

### Cost Center

- **Cost Center = Department** — mapped to `organizationUnit` from event data. No separate cost center concept. A simple `GROUP BY organizationUnit` on cost-related events provides the breakdown.

---

## Functional Requirements

### FR-CV-1: Controlling View API Endpoint

The system shall provide an API endpoint `/api/views/controlling` that returns controlling-specific aggregated data for a given scenario.

**Request:** `POST /api/views/controlling` with `{ scenarioId: string }`

**Month format:** All `month` fields use ISO format (`"2026-01"`). Display formatting is handled by the frontend.

**Response structure (full — Lifetime+/Pro/Team):**
```json
{
  "planVsActual": {
    "revenue": { "actual": number, "plan": number, "varianceAbs": number, "variancePct": number, "favorable": boolean },
    "totalCosts": { "actual": number, "plan": number, "varianceAbs": number, "variancePct": number, "favorable": boolean },
    "profit": { "actual": number, "plan": number, "varianceAbs": number, "variancePct": number, "favorable": boolean }
  },
  "monthlyVariances": [
    { "month": "2026-01", "revenueAC": number, "revenuePL": number, "varianceAbs": number, "variancePct": number }
  ],
  "costCenterBreakdown": [
    { "costCenter": string, "actual": number, "plan": number, "varianceAbs": number, "variancePct": number }
  ],
  "kpis": {
    "grossMargin": { "actual": number, "plan": number },
    "netMargin": { "actual": number, "plan": number },
    "costRatio": { "actual": number, "plan": number },
    "revenuePerEmployee": { "actual": number, "plan": number }
  }
}
```

**Response structure (teaser — Single/Lifetime):**
```json
{
  "planVsActual": { ... },
  "monthlyVariances": null,
  "costCenterBreakdown": null,
  "kpis": null
}
```

Non-eligible tiers receive only the `planVsActual` summary (used for the visible teaser cards). Detail data (monthly, cost centers, KPIs) is withheld server-side.

### FR-CV-2: Plan vs. Actual Summary

The Controlling View shall display a Plan vs. Actual comparison for key financial metrics:
- Revenue (AC vs. PL)
- Total Costs (AC vs. PL)
- Profit / EBIT (AC vs. PL)

Each metric shows: Actual value, Plan value, absolute variance, percentage variance, and favorable/unfavorable indicator.

### FR-CV-3: Monthly Variance Analysis

The Controlling View shall display a time-series variance analysis showing:
- Monthly AC vs. PL revenue comparison
- Cumulative variance trend over the scenario period
- Visual highlighting of months with significant deviations (>10% variance — uniform threshold, not tier-specific)

### FR-CV-4: Waterfall Chart (Variance Bridge)

The Controlling View shall include a waterfall/bridge chart showing the variance decomposition:
- Starting point: Plan value
- Individual variance contributors (positive and negative)
- Ending point: Actual value

This is the signature IBCS-style controlling visualization.

**Implementation:** Custom Recharts implementation using stacked bars (invisible base + visible delta). If implementation proves too complex, fallback to grouped bar chart (AC vs PL side by side).

**Mobile (<640px):** Replace waterfall chart with a compact variance table (Plan | Actual | Δ | Δ%) — waterfall charts are not usable on small screens.

### FR-CV-5: Cost Center Breakdown

The Controlling View shall display cost allocation by cost center with:
- **Cost Center = `organizationUnit`** (department) from event data
- AC vs. PL per cost center
- Variance (absolute + percentage) per cost center
- Sorted by largest absolute variance (biggest deviations first)

### FR-CV-6: KPI Dashboard Cards

The Controlling View shall display key performance indicators:
- Gross Margin (AC vs. PL)
- Net Margin (AC vs. PL)
- Cost Ratio (Total Costs / Revenue, AC vs. PL)
- Revenue per Employee (AC vs. PL)

Each KPI shows actual, plan, and a directional indicator (↑ favorable / ↓ unfavorable).

**Note:** Revenue per Employee is always available because the Controlling View is only accessible to tiers that include HR data. In the unlikely edge case of missing HR data, hide the KPI card rather than showing an error.

**Mobile layout:** KPI cards displayed as a 2×2 grid (not horizontal scroll).

### FR-CV-7: Tier Gating

The Controlling View shall only be accessible to users with Lifetime+, Pro, or Team tiers.

- Users on Single or Lifetime tiers see a **partial teaser**: the 3 Plan vs. Actual summary cards are shown with real data (Revenue, Costs, Profit), while everything below (charts, waterfall, cost centers, KPIs) is blurred with an upgrade CTA overlay
- The API returns only `planVsActual` for non-eligible tiers (detail data withheld server-side)
- CTA text: **"Unlock Plan vs. Actual Analysis — Upgrade to Lifetime+"** with link to pricing page
- Consistent with existing HR View tier gating implementation

### FR-CV-8: Navigation Integration

The Controlling View shall be accessible as a new tab alongside existing views:
- Tab order: Finance | Sales | HR | **Controlling**
- Tab appears for all users but shows lock icon for non-eligible tiers
- Active tab styling consistent with existing view tabs

---

## IBCS Visualization Guidelines

The Controlling View should follow IBCS notation conventions at the same level as existing views ("halbwegs passen" — doesn't need to be 100% perfect, but recognizably IBCS-inspired):

| Element | IBCS Convention | Implementation |
|---------|-----------------|----------------|
| Actual (AC) | Solid fill, dark/black | Solid bars/lines |
| Plan (PL) | Outlined, white fill with border | Outlined bars |
| Favorable variance | — | Green accent or standard dark |
| Unfavorable variance | — | Red accent |
| Waterfall | Connected bars showing buildup | Standard waterfall chart |

**Note:** FC and PY are not displayed in v1. Reuse existing IBCS styling patterns from Finance/Sales views. The goal is visual consistency across all four views, not pixel-perfect IBCS certification.

---

## Non-Functional Requirements

### Performance
- Controlling View API response: < 500ms (same as existing views)
- Chart rendering: < 300ms after data load
- No additional database queries beyond existing event queries + `comparison-data.ts` calculations

### Data Consistency
- All PL values must be deterministic — same scenario always produces identical Controlling View
- Revenue variance in Controlling View must match Finance View revenue figures exactly
- Cost figures must reconcile with Finance View cost breakdown
- **Shared utility function** with Finance View for base aggregation — no duplicated logic

### Responsive Design
- Desktop (>1024px): Full dashboard layout with charts side-by-side where appropriate
- Tablet (640-1024px): Stacked charts, full width
- Mobile (<640px): Stacked single-column, KPI cards as 2×2 grid, waterfall replaced by variance table

---

## UI/UX Notes

### Layout Suggestion (Desktop)

```
┌─────────────────────────────────────────────────┐
│  [Finance] [Sales] [HR] [Controlling]           │
├─────────────────────────────────────────────────┤
│                                                 │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐       │
│  │ Revenue  │ │  Costs   │ │  Profit  │       │
│  │ AC vs PL │ │ AC vs PL │ │ AC vs PL │       │
│  └──────────┘ └──────────┘ └──────────┘       │
│                                                 │
│  ┌────────────────────────────────────────┐     │
│  │  Monthly Variance Analysis (Bar Chart) │     │
│  └────────────────────────────────────────┘     │
│                                                 │
│  ┌────────────────────────────────────────┐     │
│  │  Variance Waterfall / Bridge Chart     │     │
│  │  (Desktop/Tablet: chart)               │     │
│  │  (Mobile: compact variance table)      │     │
│  └────────────────────────────────────────┘     │
│                                                 │
│  ┌─────────────────┐  ┌──────────────────┐     │
│  │  Cost Center     │  │  KPI Cards       │     │
│  │  Breakdown Table │  │  (2×2 grid)      │     │
│  └─────────────────┘  └──────────────────┘     │
│                                                 │
└─────────────────────────────────────────────────┘
```

### Locked State (Single/Lifetime Tiers)

```
┌─────────────────────────────────────────────────┐
│  [Finance] [Sales] [HR] [🔒 Controlling]        │
├─────────────────────────────────────────────────┤
│                                                 │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐       │  ← VISIBLE (real data)
│  │ Revenue  │ │  Costs   │ │  Profit  │       │
│  │ AC vs PL │ │ AC vs PL │ │ AC vs PL │       │
│  └──────────┘ └──────────┘ └──────────┘       │
│                                                 │
│  ░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░  │  ← BLURRED
│  ░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░  │
│  ░░░░                                    ░░░  │
│  ░░░  🔒 Unlock Plan vs. Actual         ░░░  │
│  ░░░     Analysis — Upgrade to           ░░░  │
│  ░░░     Lifetime+ →                     ░░░  │
│  ░░░░                                    ░░░  │
│  ░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░  │
│  ░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░  │
└─────────────────────────────────────────────────┘
```

---

## Acceptance Criteria

### Must Pass

- [ ] Controlling View displays correctly for all 9 curated example scenarios
- [ ] PL values derived from `comparison-data.ts` are consistent and deterministic
- [ ] Revenue figures match between Controlling View and Finance View (zero discrepancy)
- [ ] Waterfall chart correctly visualizes variance bridge from Plan to Actual
- [ ] Tier gating works: Single/Lifetime see teaser (summary cards visible, rest blurred), Lifetime+/Pro/Team see full view
- [ ] API returns only `planVsActual` for non-eligible tiers (detail data withheld server-side)
- [ ] Responsive layout works on desktop, tablet, and mobile
- [ ] Mobile: waterfall replaced by variance table, KPIs in 2×2 grid
- [ ] IBCS notation applied (solid AC, outlined PL, variance coloring)
- [ ] Tab navigation works seamlessly alongside Finance/Sales/HR

### Should Pass

- [ ] Cost center breakdown sorted by largest absolute variance
- [ ] KPI cards show directional indicators (favorable/unfavorable)
- [ ] Locked state includes blurred preview with visible summary cards (not just an error message)
- [ ] Loading/skeleton state while data fetches
- [ ] Variance highlighting at >10% threshold (uniform)

### Nice to Have

- [ ] Tooltip on variance bars showing detailed breakdown
- [ ] Print-friendly view for the controlling dashboard
- [ ] "Export as CSV" for controlling data (if export infrastructure supports it)

---

## Out of Scope

- **Real Plan event generation** (Option B) — deferred to future release based on demand
- **Custom budget input** — users cannot set their own plan values
- **Forecast (FC) display** — FC data exists in `comparison-data.ts` but is not shown in v1. Future iteration candidate.
- **Drill-down into cost center details** — flat cost center list (grouped by `organizationUnit`), no hierarchical drill-down
- **Controlling View for free/unauthenticated users** — only available to paying users on eligible tiers
- **Tier-specific variance thresholds** — uniform 10% for all scenario sizes

---

## Dependencies

| Dependency | Status | Notes |
|------------|--------|-------|
| `lib/utils/comparison-data.ts` | Exists | Core data source for PL values |
| Existing view tab infrastructure | Exists | Finance/Sales/HR tabs already implemented |
| Tier gating logic | Exists | HR View already implements tier-based access |
| Chart components (Recharts) | Exists | Needs new waterfall chart component (custom) |
| IBCS styling patterns | Exists | Reuse from existing views |
| Event `organizationUnit` field | Exists | Used as cost center identifier |

### New Component Needed

- **Waterfall/Bridge Chart** — custom Recharts implementation using stacked bars (invisible base + visible delta). Fallback: grouped bar chart (AC vs PL side by side) if waterfall proves too complex.

---

## Estimation Notes

Given that this is purely a view-layer feature (no generation changes, no schema changes):

| Component | Estimated Effort |
|-----------|-----------------|
| API endpoint (`/api/views/controlling`) | 3-4h |
| Plan vs. Actual summary component | 2-3h |
| Monthly variance chart | 2-3h |
| Waterfall/bridge chart | 4-6h (highest uncertainty) |
| Cost center breakdown table | 2-3h |
| KPI cards | 1-2h |
| Tier gating + locked state (teaser pattern) | 2-3h |
| Tab navigation integration | 1h |
| Responsive design (incl. mobile waterfall fallback) | 2-3h |
| Testing across 9 scenarios | 3-4h |
| **Total** | **22-32h** |

**Biggest risk:** Waterfall chart implementation. If Recharts custom stacked bars prove unreliable, fall back to grouped bar chart.

---

## References

- PRD v2.0: Epic 4, Story 4.2 (Plan vs. Actual), Story 4.3 (IBCS Visualization)
- Jürgen Faisst feedback (December 2025): "Plan vs. Actual comparisons" as HIGH priority
- Product Brief v2.0: Controlling View listed under "Next" on What's Next page
- Winston's architecture assessment (2026-02-07): Option A vs. Option B analysis
- Party Mode review (2026-02-07): All 8 open questions resolved by Product Owner

---

## Appendix: Party Mode Review Decisions (2026-02-07)

| # | Question | Decision |
|---|----------|----------|
| 1 | Cost Center definition | Cost Center = Department = `organizationUnit` from events |
| 2 | Revenue per Employee without HR data | Not applicable — Controlling View tiers always have HR. Edge case: hide KPI card |
| 3 | FC values in PvA table | No — AC vs. PL only in v1. FC deferred to future iteration |
| 4 | Variance threshold tier-specific? | No — uniform 10% for all sizes |
| 5 | Locked state detail level | Summary cards visible (real data), charts/details blurred |
| 6 | Mobile waterfall fallback | Compact variance table replaces waterfall on <640px |
| 7 | Month format in API | ISO format (`2026-01`), frontend formats for display |
| 8 | Terminology | "Plan" throughout (IBCS-consistent), not "Budget" |

*This document is ready for the BMAD team to begin architecture review and implementation planning.*
