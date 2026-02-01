# Plausibility Analysis: Midcap / Industrials (Revised)

**Analysis Date:** December 12, 2025  
**Dataset:** Synthetic Business Data Generator Output  
**Declared Tier/Sector:** Midcap / Industrials

---

## 1. Tier Recognition

| Criterion | Expected (Midcap) | Observed | Status |
|-----------|-------------------|----------|--------|
| Annual Revenue | €5M - €100M | ~€32.5M (calculated) | ✅ Confirmed |
| Growth Rate | 6-15% | ~0-3% (AC ≈ PY) | ⚠️ Low but acceptable |
| Net Margin | -5% to +12% | +4.14% | ✅ Within range |

**Verdict:** This dataset correctly represents a Midcap Industrial company. Major improvement from previous iteration.

---

## 2. Plausibility Score

# 7/10

**Assessment:** Strong improvement! Margins and cost structure now align with industry benchmarks. One critical issue remains: "Other" costs at 2% is unrealistically low.

---

## 3. Detail Analysis

### Revenue Trend
✅ **Plausible Pattern**

| Metric | Expected | Observed | Assessment |
|--------|----------|----------|------------|
| Annual Revenue | €5M-€100M | ~€32.5M | ✅ Mid-range Midcap |
| YoY Growth | 6-15% | ~0-3% | ⚠️ Low but possible |
| Seasonality | Moderate cycles | Q1 peak, summer trough | ✅ Industrial pattern |
| Monthly Range | Variable | €1.5M - €5M | ✅ 3.3x swing acceptable |

**Observations:**
- Q1 strength (especially March at ~€5M) aligns with industrial project cycles
- Summer slowdown (Jul-Aug at ~€1.5M) reflects typical manufacturing/construction patterns
- Q4 recovery matches year-end project completions
- 3.3x seasonal swing is reasonable for project-based industrial work

**Subsector Alignment:**
The pattern matches Engineering/Construction or Building Materials seasonality — strong spring/early year, weak summer.

### Margin Analysis
✅ **Excellent Benchmark Alignment**

| Metric | Expected (Industrials) | Observed | Assessment |
|--------|------------------------|----------|------------|
| Gross Margin | 15-37% | 36.50% | ✅ Upper bound (Machinery-like) |
| Net Margin | -5% to +12% | 4.14% | ✅ Healthy mid-range |

**Benchmark Comparison (NYU Stern, Jan 2025):**

| Industrial Subsector | Gross Margin | Net Margin | Match? |
|----------------------|--------------|------------|--------|
| Machinery | 37.1% | 10.0% | ⚠️ GM match, NM lower |
| Building Materials | 32.1% | 10.6% | Close |
| Electrical Equipment | 30.1% | 2.6% | ✅ Good match |
| Transportation | 23.2% | 4.1% | ✅ NM matches exactly |
| Aerospace/Defense | 17.1% | 4.4% | ✅ NM matches |
| **This Dataset** | **36.50%** | **4.14%** | — |

**Assessment:** The 36.5% gross margin suggests a Machinery or specialized equipment company. The 4.14% net margin aligns perfectly with Transportation, Aerospace, or Electrical Equipment sectors. This combination is plausible for a company with good gross margins but higher operating costs.

**Margin Stability:**
| Period | Gross Margin | Net Margin |
|--------|--------------|------------|
| AC (Current) | 36.50% | 4.14% |
| PY (Prior Year) | 37.71% | 4.03% |
| PL (Plan) | 36.56% | 4.06% |
| FC (Forecast) | 37.13% | 4.03% |

Excellent consistency across periods — margins are stable with only minor fluctuations (±1pp). This reflects mature, predictable operations.

### Cost Structure
⚠️ **One Critical Issue: "Other" Too Low**

| Category | Amount | % of Revenue | Expected Range | Status |
|----------|--------|--------------|----------------|--------|
| Payroll | €9,867,000 | 30.37% | 25-40% | ✅ Perfect |
| Procurement/COGS | €20,630,795 | 63.50% | 60-85% | ✅ Within range |
| Other | €645,000 | 1.99% | 10-20% | ❌ Way too low |
| **Total Costs** | **€31,142,795** | **95.86%** | — | — |

**Mathematical Verification:**
```
Revenue (calculated):       €32,487,795   (100.00%)
- Procurement/COGS:        -€20,630,795   (-63.50%)
= Gross Profit:             €11,857,000   ( 36.50%) ✓

- Payroll:                  -€9,867,000   (-30.37%)
- Other:                      -€645,000   ( -1.99%)
= Net Profit:                €1,345,000   (  4.14%) ✓
```

**The math is internally consistent.** ✅

**The "Other" Problem:**

At only 1.99% (€645,000), "Other" must impossibly cover:

| Expense Category | Typical % of Revenue | Implied Amount |
|------------------|---------------------|----------------|
| Rent/Facilities | 2-5% | €650k - €1.6M |
| Utilities | 1-3% | €325k - €975k |
| Insurance | 1-2% | €325k - €650k |
| Depreciation | 3-8% | €975k - €2.6M |
| Professional Services | 1-2% | €325k - €650k |
| Marketing/Sales | 2-5% | €650k - €1.6M |
| IT/Technology | 1-2% | €325k - €650k |
| Travel/Transport | 1-3% | €325k - €975k |
| **Minimum Total** | **12-30%** | **€3.9M - €9.7M** |

**Current allocation:** €645,000 (1.99%)
**Realistic minimum:** €3,900,000 (12%)

The "Other" category is underfunded by approximately €3.3M - €9M.

**Impact if Corrected:**
```
If Other = 12% (minimum realistic):
- Other expense: €3,900,000
- Net Profit: €1,345,000 - €3,255,000 = -€1,910,000
- Net Margin: -5.9%

If Other = 15% (mid-range):
- Other expense: €4,875,000
- Net Profit: €1,345,000 - €4,230,000 = -€2,885,000
- Net Margin: -8.9%
```

With realistic "Other" costs, this company would be **loss-making**, which is still within the Midcap/Industrial benchmark range of -5% to +12%.

### Cash Flow
⚠️ **Magnitude Seems Excessive**

**Observations from Net Cash Flow Chart:**
- Monthly cash flows range from €3M to €9M (all positive)
- Q1 peak around €8-9M (Feb-Apr)
- Summer trough around €3-4M
- Pattern loosely follows revenue seasonality

**Concern:**
- Annual net profit is ~€1.35M (4.14% of €32.5M)
- Monthly cash flows of €3-9M seem disproportionate
- Even with depreciation add-back (~€1-2M annually), cash flows appear 20-40x higher than expected

**Expected Pattern:**
For a company with €1.35M annual profit and typical industrial depreciation:
- Net Profit: €1.35M
- Add: Depreciation: ~€1.5M (estimated 5% of revenue)
- Operating Cash Flow: ~€2.85M annually
- Monthly average: ~€240k

The €3-9M monthly cash flows shown are approximately 12-37x higher than expected.

**Possible Interpretation:**
The cash flow chart may be showing cumulative YTD figures rather than monthly, or there's a scaling issue in the visualization.

---

## 4. Consistency Checks

| Check Type | Status | Finding |
|------------|--------|---------|
| Mathematical Consistency | ✅ | All percentages calculate correctly |
| Internal Consistency | ⚠️ | "Other" too low to cover real expenses |
| Time Series Consistency | ✅ | Stable margins across AC/PY/PL/FC |
| Industry Consistency | ✅ | Margins align with Industrial benchmarks |
| Revenue vs Tier | ✅ | €32.5M correctly within Midcap range |
| Seasonality | ✅ | Pattern matches industrial cycles |
| P&L to Cash Flow | ⚠️ | Cash flow magnitude seems excessive |

---

## 5. Red Flags

### 🔴 Critical

1. **"Other" Costs at Only 2%**
   - Expected: 10-20% for industrial companies
   - Gap: 8-18 percentage points
   - Missing ~€3-6M in legitimate operating expenses
   - **Impact:** If corrected, net margin would be -6% to -9% (still within benchmark)

### 🟡 Warnings

2. **Cash Flow Magnitude**
   - Shows €3-9M monthly vs expected ~€240k average
   - Approximately 12-37x higher than P&L would suggest
   - May be visualization/scaling issue rather than data bug

3. **Low YoY Growth**
   - ~0-3% observed vs 6-15% benchmark
   - Acceptable for mature industrial company
   - Could indicate market saturation or intentional steady-state modeling

4. **Gross Margin at Upper Bound**
   - 36.5% is at the very top of 15-37% range
   - Suggests specialized machinery/equipment
   - Verify this aligns with intended subsector

---

## 6. Recommendations

### Immediate Fix Required

| Issue | Current Value | Recommended Value | Rationale |
|-------|---------------|-------------------|-----------|
| **Other %** | 1.99% | 12-18% | Cover rent, depreciation, utilities, insurance |

### Resulting Adjusted Profile

**Option A: Maintain Profitability (reduce other costs internally)**
```
Revenue:        €32,500,000  (100.0%)
- COGS:        -€20,630,000  ( 63.5%)
= Gross Profit: €11,870,000  ( 36.5%)

- Payroll:      -€8,125,000  ( 25.0%)  ← Reduced from 30%
- Other:        -€3,250,000  ( 10.0%)  ← Increased from 2%
= Net Profit:     €495,000   (  1.5%)  ← Still positive
```

**Option B: Accept Losses (keep payroll, fix other)**
```
Revenue:        €32,500,000  (100.0%)
- COGS:        -€20,630,000  ( 63.5%)
= Gross Profit: €11,870,000  ( 36.5%)

- Payroll:      -€9,870,000  ( 30.4%)  ← Keep as-is
- Other:        -€4,875,000  ( 15.0%)  ← Increased to realistic
= Net Profit:   -€2,875,000  ( -8.8%)  ← Within benchmark
```

**Option C: Adjust COGS for Higher Gross Margin**
```
Revenue:        €32,500,000  (100.0%)
- COGS:        -€17,875,000  ( 55.0%)  ← Reduced
= Gross Profit: €14,625,000  ( 45.0%)  ← Higher (specialized equipment)

- Payroll:      -€9,750,000  ( 30.0%)
- Other:        -€3,900,000  ( 12.0%)
= Net Profit:     €975,000   (  3.0%)  ← Sustainable
```

### Generator Logic Improvements

1. **Enforce Minimum "Other" Category**
   ```
   Other% = MAX(10%, calculated_value)
   # Industrial minimum: 10%
   # Adjust other categories if needed to maintain margin target
   ```

2. **Link Cash Flow to P&L**
   ```
   Monthly_CF ≈ (Net_Profit + Depreciation) / 12 ± working_capital_swing
   # For €1.35M profit + €1.5M depreciation:
   # Expected monthly CF: €240k ± €500k
   ```

3. **Create Subsector Profiles**
   
   | Subsector | COGS | Payroll | Other | Net Margin |
   |-----------|------|---------|-------|------------|
   | Machinery | 63% | 22% | 12% | 3-5% |
   | Aerospace | 83% | 10% | 5% | 2-4% |
   | Building Materials | 68% | 15% | 10% | 7-10% |

---

## 7. Summary

| Aspect | Score | Comment |
|--------|-------|---------|
| Revenue Level | ✅ 10/10 | €32.5M correctly within Midcap |
| Growth Rate | ⚠️ 6/10 | Low but acceptable |
| Gross Margin | ✅ 9/10 | 36.5% aligns with Machinery/Equipment |
| Net Margin | ✅ 9/10 | 4.14% matches Transportation/Aerospace |
| Payroll % | ✅ 10/10 | 30.4% perfect for labor-intensive industrial |
| COGS % | ✅ 9/10 | 63.5% within industrial range |
| Other % | ❌ 2/10 | 2% is ~10pp too low |
| Seasonality | ✅ 9/10 | Q1 peak, summer trough matches industrial |
| Cash Flow | ⚠️ 5/10 | Magnitude seems disconnected from P&L |
| **Overall** | **7/10** | **Major improvement — one fix needed** |

---

## 8. Comparison: Previous vs Current Midcap/Industrial

| Metric | Previous Dataset | Current Dataset | Improvement |
|--------|------------------|-----------------|-------------|
| **Score** | 3/10 | 7/10 | +4 points |
| Gross Margin | 19.9% | 36.5% | Now at benchmark |
| Net Margin | -28.3% ❌ | +4.14% ✅ | Fixed |
| COGS % | 80.1% | 63.5% | Reduced appropriately |
| Payroll % | 41.0% | 30.4% | Reduced appropriately |
| Other % | 7.2% | 2.0% | Still needs work |
| Cost Sum | 128% ❌ | 96% ✅ | Fixed |

**Key Improvements Made:**
1. ✅ Cost structure no longer exceeds 100%
2. ✅ Net margin now within -5% to +12% range
3. ✅ Payroll reduced from 41% to 30%
4. ✅ COGS reduced from 80% to 64%
5. ⚠️ "Other" still too low (was 7%, now 2% — went wrong direction!)

---

## 9. What This Dataset Gets Right

This is the first dataset that demonstrates proper understanding of:

- **Tier-appropriate revenue:** €32.5M is solidly Midcap
- **Industry-appropriate gross margins:** 36.5% matches Machinery
- **Industry-appropriate net margins:** 4.14% matches Transportation/Aerospace
- **Reasonable seasonality:** Q1 strength, summer weakness
- **Stable margin progression:** AC/PY/PL/FC all within 1pp

**The only remaining systematic issue is the "Other" category underallocation.**

---

## Appendix: Benchmark Reference

**Source:** NYU Stern (Aswath Damodaran), January 2025

### Industrial Sector — This Dataset's Position

```
Gross Margin Scale (Industrial):
Aerospace    |████                           | 17.1%
Trucking     |█████                          | 20.7%
Transport    |██████                         | 23.2%
Elec Equip   |████████                       | 30.1%
Building Mat |█████████                      | 32.1%
Machinery    |██████████                     | 37.1%
THIS DATA    |██████████                     | 36.5% ✓
             0%        20%        40%

Net Margin Scale (Industrial):
Elec Equip   |█                              | 2.6%
Eng/Const    |██                             | 3.0%
Transport    |██                             | 4.1%
THIS DATA    |██                             | 4.14% ✓
Trucking     |██                             | 4.2%
Aerospace    |██                             | 4.4%
Machinery    |█████                          | 10.0%
Building Mat |█████                          | 10.6%
             0%         5%        10%
```

---

*Analysis generated by Financial Data Quality Review*
