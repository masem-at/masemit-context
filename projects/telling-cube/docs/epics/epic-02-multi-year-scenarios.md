# Epic 2: Multi-Year Scenarios (3-5 Years)

**Created:** 2026-01-18
**Status:** Ready for Development
**Sprint:** 2
**Priority:** High
**Estimated Effort:** 3-4 weeks
**Dependencies:** None (independent epic)

---

## Goal

Extend tellingCube's data generation from 12 months to multi-year scenarios (1/3/5 years), enabling long-term business trend analysis, strategic planning data, and year-over-year comparisons.

## Business Value

- **Brian's Request:** "Multi-year scenario, three to five year business plan"
- **Jürgen's Request:** "mehr Jahre, Budgets, Forecasts, Mehrjahrespläne"
- **IBCS Unlock:** Multi-year visuals (UN 3.2, UN 4.1, EX 2.3, CH 2.1)
- **Upsell Driver:** 5-year scenarios as premium feature

## Source Spec

- `docs/future/spec-multi-year-scenarios.md`

---

## Stories

### Story 2.1: Extend Scenario Model for Multi-Year
**Points:** 3

**Description:**
Add duration fields to scenario model and update generation to support multi-year periods.

**Acceptance Criteria:**
- [ ] `durationYears` field added to Scenario model (1, 3, or 5)
- [ ] `timePeriodStart` and `timePeriodEnd` calculated correctly
- [ ] Database migration created and tested
- [ ] Existing 1-year scenarios continue to work

---

### Story 2.2: Year Duration Selector UI
**Points:** 2

**Description:**
Add dropdown for users to select 1, 3, or 5 year duration on generation page.

**Acceptance Criteria:**
- [ ] Dropdown with options: "1 Year", "3 Years", "5 Years"
- [ ] Default selection is "1 Year"
- [ ] Selection persists to scenario generation
- [ ] Tier restrictions applied (see pricing matrix)
- [ ] Tooltip explains generation time implications

---

### Story 2.3: Multi-Year Event Generation Logic
**Points:** 8

**Description:**
Extend AI generation to create consistent events across multiple years with realistic YoY trends.

**Acceptance Criteria:**
- [ ] Events scaled by duration: `baseEvents × years`
- [ ] Year-over-year growth patterns implemented:
  - Startup: 50% → 30% → 20% → 10% → 5%
  - MidCap: 8% → 10% → 15% → 5% → 8%
  - LargeCap: 3% → 4% → -2% → 5% → 3%
- [ ] Seasonal patterns repeat annually
- [ ] Strategic inflection points added (Year 2, 3, 4)
- [ ] All years mathematically consistent

---

### Story 2.4: Time Dimension Extensions
**Points:** 3

**Description:**
Add fiscal year, quarter, and year-index dimensions to data model.

**Acceptance Criteria:**
- [ ] `fiscalYear` field on all aggregations
- [ ] `fiscalQuarter` (Q1-Q4) calculated
- [ ] `yearIndex` (1-5) for position in range
- [ ] `isCurrentYear` boolean flag
- [ ] Comparison columns: PY, PY-2, CAGR

---

### Story 2.5: Extended JSON/CSV Export
**Points:** 3

**Description:**
Update export to include multi-year data structure and annual summaries.

**Acceptance Criteria:**
- [ ] Export includes `durationYears`, `startYear`, `endYear`
- [ ] `annualSummaries` array with yearly totals
- [ ] `cagr` object with growth rates
- [ ] All 36/60 months in `monthlyData`
- [ ] CSV export has year column

---

### Story 2.6: Multi-Year Chart - Trend Line
**Points:** 5

**Description:**
Create line chart showing 3-5 year trends for key metrics.

**Acceptance Criteria:**
- [ ] Line chart with year granularity
- [ ] Metrics: Revenue, Profit, Costs
- [ ] AC vs PL comparison lines
- [ ] CAGR annotation on chart
- [ ] Year selector/zoom capability
- [ ] IBCS styling (grayscale, clean axes)

---

### Story 2.7: YoY Comparison Chart
**Points:** 5

**Description:**
Create grouped bar chart comparing same metrics across years.

**Acceptance Criteria:**
- [ ] Grouped bars by year (max 5 groups)
- [ ] ΔPY variance indicators
- [ ] ΔPY% badges
- [ ] Click-to-drill into specific year
- [ ] IBCS-inspired styling

---

### Story 2.8: Progressive Generation UI
**Points:** 3

**Description:**
Show progress as multi-year scenarios generate (can take 5-7 minutes).

**Acceptance Criteria:**
- [ ] Progress bar shows year-by-year progress
- [ ] "Generating Year 2 of 5..." status message
- [ ] Estimated time remaining
- [ ] Cancel option available
- [ ] Generated years viewable before completion

---

### Story 2.9: Tier-Based Access Control
**Points:** 2

**Description:**
Restrict multi-year options based on pricing tier.

**Acceptance Criteria:**
- [ ] Free: No multi-year (curated examples only)
- [ ] Supporter S (€29): 1 + 3 years
- [ ] Supporter M+ (€99+): 1 + 3 + 5 years
- [ ] Locked options show upgrade CTA
- [ ] Existing purchases respected

---

## Definition of Done

- [ ] All stories completed and tested
- [ ] 3-year and 5-year scenarios generate correctly
- [ ] YoY trends are realistic and consistent
- [ ] Charts display multi-year data properly
- [ ] Export includes all years
- [ ] Generation time acceptable (<7 min for 5 years)
- [ ] Deployed to develop branch for testing

---

## Technical Notes (from Winston)

**Architecture Decision:** Multi-Year is a horizontal extension (time dimension). HR Domain (Epic 3) will build on this infrastructure - HR events will span all generated years if Multi-Year is active.

**Performance:** Consider background queue for 5-year generations.

---

*Epic created by: Bob (Scrum Master)*
*Reviewed by: John (PM), Winston (Architect)*
*Source spec by: Mary (Analyst), Winston (Architect)*
