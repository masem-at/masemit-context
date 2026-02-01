# Feature Spec: Multi-Year Scenarios (3-5 Years)

**Created:** 2026-01-16
**Status:** Draft for Review
**Requested By:** Brian Julius (video feedback), Jürgen Faisst (email feedback)
**Priority:** High

---

## Executive Summary

Extend tellingCube's data generation from the current 12-month period to support multi-year scenarios (3-5 years), enabling users to generate and analyze long-term business trends, strategic planning data, and year-over-year comparisons.

**Brian's Quote:**
> *"This multi-year scenario, kind of looking at a three to five year business plan is something I'm really looking forward to."*

**Jürgen's Quote:**
> *"mehr Jahre, Budgets, Forecasts, Mehrjahrespläne"*

---

## Problem Statement

### Current State
- tellingCube generates exactly 12 months of data (Jan-Dec of selected year)
- Users can compare Actual vs Plan vs Previous Year
- No support for multi-year trend analysis or strategic planning scenarios

### Pain Points
1. **Strategic Planning:** CFOs need 3-5 year projections for business plans
2. **YoY Analysis:** Current 12-month limit prevents meaningful year-over-year trends
3. **Training:** Long-term analysis requires multi-year datasets
4. **IBCS Use Cases:** Many IBCS visuals are designed for multi-year comparisons

---

## Proposed Solution

### Multi-Year Generation Options

**User Selection (New Dropdown):**
| Option | Months | Use Case |
|--------|--------|----------|
| 1 Year (current) | 12 | Quick demos, single-year analysis |
| 3 Years | 36 | YoY comparisons, medium-term trends |
| 5 Years | 60 | Strategic planning, long-term analysis |

### Data Structure Changes

**Current:**
```
timePeriodStart: 2025-01-01
timePeriodEnd: 2025-12-31
months: 12
```

**Proposed:**
```
timePeriodStart: 2021-01-01
timePeriodEnd: 2025-12-31
years: 5
months: 60
```

---

## Technical Implementation

### Phase 1: Generation Extension

**Modified Scenario Model:**
```prisma
model Scenario {
  // ... existing fields ...

  // New fields
  durationYears     Int       @default(1)  // 1, 3, or 5
  timePeriodStart   DateTime  // Start of multi-year period
  timePeriodEnd     DateTime  // End of multi-year period
}
```

**Generation Logic Changes:**
1. Scale event count by duration: `baseEvents × years`
2. Introduce year-over-year trends:
   - Growth trajectory (startup → maturity)
   - Seasonal patterns repeating annually
   - Strategic inflection points (market changes, expansions)
3. Maintain consistency across all years

### Phase 2: Business Logic - Multi-Year Patterns

**Year-Over-Year Trends to Model:**

| Pattern | Year 1 | Year 2 | Year 3 | Year 4 | Year 5 |
|---------|--------|--------|--------|--------|--------|
| **Startup** | Rapid growth 50%+ | Growth 30% | Growth 20% | Stabilize 10% | Mature 5% |
| **MidCap** | Steady 8% | Growth 10% | Expansion 15% | Integration 5% | Steady 8% |
| **LargeCap** | Stable 3% | Stable 4% | Restructure -2% | Recovery 5% | Stable 3% |

**Strategic Events (Notable Events):**
- Year 2: Market expansion / New product launch
- Year 3: Acquisition / Major client win
- Year 4: Restructuring / Cost optimization
- Year 5: Stabilization / Dividend policy

### Phase 3: Data Dimensions

**New Time Dimensions:**
```typescript
interface TimeDimensions {
  timePeriod: string;      // "2025-01" (existing)
  fiscalYear: number;      // 2025
  fiscalQuarter: string;   // "Q1"
  yearIndex: number;       // 1-5 (position in multi-year range)
  isCurrentYear: boolean;  // true for most recent year
}
```

**Comparison Columns:**
| Column | Description |
|--------|-------------|
| AC (Actual) | Current period actual |
| PY (Previous Year) | Same period, year before |
| PY-2 | Same period, 2 years ago |
| PL (Plan) | Budget/forecast |
| ΔPY | Variance vs previous year |
| ΔPY% | Variance % vs previous year |
| CAGR | Compound annual growth rate |

### Phase 4: Export Changes

**Extended JSON Export:**
```json
{
  "scenario": {
    "durationYears": 5,
    "startYear": 2021,
    "endYear": 2025
  },
  "annualSummaries": [
    { "year": 2021, "revenue": 5000000, "profit": 250000 },
    { "year": 2022, "revenue": 6500000, "profit": 450000 },
    { "year": 2023, "revenue": 8000000, "profit": 720000 },
    { "year": 2024, "revenue": 9200000, "profit": 920000 },
    { "year": 2025, "revenue": 10000000, "profit": 1100000 }
  ],
  "cagr": {
    "revenue": 0.189,  // 18.9% CAGR
    "profit": 0.445    // 44.5% CAGR
  },
  "monthlyData": [...] // All 60 months
}
```

### Phase 5: UI/Charts Extension

**New Chart Types:**
| Chart | Description |
|-------|-------------|
| Multi-Year Trend | Line chart with 3-5 year history |
| YoY Comparison | Grouped bars by year |
| CAGR Waterfall | Growth decomposition |
| Strategic Timeline | Notable events across years |

**UI Changes:**
1. Year selector/filter on charts
2. "Zoom" into specific year
3. Annual summary cards
4. CAGR badges

---

## Generation Time Considerations

### Performance Impact

| Duration | Events | Est. Generation Time |
|----------|--------|---------------------|
| 1 Year | ~3,000 | 60-90 seconds |
| 3 Years | ~9,000 | 3-4 minutes |
| 5 Years | ~15,000 | 5-7 minutes |

### Mitigation Strategies
1. **Progressive Generation:** Generate year-by-year, show progress
2. **Caching:** Store generated years, only regenerate requested
3. **Premium Feature:** Multi-year only for paid users (reduces server load)
4. **Background Processing:** Use queue for long generations

---

## Pricing Considerations

| Tier | 1 Year | 3 Years | 5 Years |
|------|--------|---------|---------|
| Free | ✅ (curated only) | ❌ | ❌ |
| Supporter S (€29) | ✅ | ✅ | ❌ |
| Supporter M (€99) | ✅ | ✅ | ✅ |
| Higher tiers | ✅ | ✅ | ✅ |

**Rationale:** Multi-year data is significantly more valuable and compute-intensive. Reserve 5-year scenarios for higher tiers.

---

## IBCS Alignment

Multi-year scenarios unlock additional IBCS reference visuals:

| IBCS Visual | Requirement | Now Possible |
|-------------|-------------|--------------|
| UN 3.2 Time series | Multi-year data | ✅ |
| UN 4.1 Scenario comparison | AC/PY/PL across years | ✅ |
| EX 2.3 Growth indicators | CAGR, trend arrows | ✅ |
| CH 2.1 Variance analysis | ΔPY, ΔPY-2 | ✅ |

---

## Effort Estimation

| Phase | Description | Complexity |
|-------|-------------|------------|
| Phase 1 | Generation extension | Medium |
| Phase 2 | Business logic (trends) | High |
| Phase 3 | Data dimensions | Medium |
| Phase 4 | Export changes | Low |
| Phase 5 | UI/Charts | High |

**Total Estimated Effort:** 3-4 weeks (solo developer)

---

## Success Metrics

1. **Adoption:** >50% of paid users generate multi-year scenarios
2. **Feedback:** "5-year planning" mentioned in positive reviews
3. **Conversion:** Multi-year as upsell driver (Free → Paid)

---

## Open Questions

1. Should generation be sequential (year by year) or parallel?
2. Allow custom year ranges (e.g., 2020-2027)?
3. Include forecast years beyond current year (AC + FC)?
4. Combine with HR domain for workforce planning scenarios?

---

## Next Steps

1. [ ] Mario reviews spec
2. [ ] Decide on pricing tier restrictions
3. [ ] Create user stories in epics
4. [ ] Design multi-year chart mockups
5. [ ] Prototype generation performance

---

*Prepared by: Mary (Business Analyst) + Winston (Architect)*
*For review by: Mario*
