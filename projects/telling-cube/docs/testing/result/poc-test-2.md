# PoC-Test-2 Feedback, Matthias, Brother

### 1. Overall impression

- The generator produces **internally consistent** financials:  
  - Revenue, COGS, profit and margin percentages reconcile correctly.  
  - Year-over-year comparisons (AC vs PY vs PL vs FC) are coherent.
- The three scenarios (startup / midcap / largecap) are **clearly distinguishable in scale and growth**, which is great for demo purposes.
- Main improvement potential: **business realism by sector** (typical margin ranges, cost structure, cost/income ratio).

Below is detailed feedback per scenario.

---

### 2. Largecap – Financials (segment)

**Data (AC)**  
- Revenue: **€285.8m** (flat vs PY)  
- COGS: **€5.2m** (~1.8% of revenue)  
- Payroll: **€0.35m** (~0.12% of revenue)  
- Other Opex: **€18.6m** (~6.5% of revenue)  
- Net profit: **€261.6m** → **net margin ~91.6%**  
- Gross margin: **~98.2%**

**Plausibility**

- **Scale**: Plausible as a **business segment / region** of a large financial group, not as the whole listed entity.  
- **COGS & gross margin**: High gross margin fits fee/interest income – that part is okay.  
- **Cost structure** is **not realistic**:
  - Total operating cost (payroll + other) is *below 7%* of revenue → cost/income ratio <10%.
  - Real-world banks/financials typically have **40–70%** cost/income.
  - Payroll at 0.12% of revenue is far too low.

**Suggestions**

- Add sector rules for financials, e.g.:
  - `cost_income_ratio` between **40–70%** (configurable).
  - `payroll_share` between **20–40%** of revenue.

---

### 3. Midcap – Industrials (business unit)

**Data (AC)**  
- Revenue: **€5.0m** (+3% vs PY)  
- COGS (Procurement): **€1.34m** (~26.6% of revenue)  
- Other Opex: **€1.66m** (~33.0% of revenue)  
- Payroll: **€0** (0%)  
- Net profit: **€2.03m** → **net margin ~40.4%**  
- Gross margin: **~73.4%**

**Plausibility**

- **Scale**: Fits a **business unit / product line** of a midcap industrials company.  
- **Revenue growth**: Low double-digit / high single-digit growth is plausible.  
- **Margins**:
  - Gross margin >70% is **very high** for classical industrials (more typical: **25–45%**), but could be justified for a **service / spare parts / high-IP** unit.
  - Net margin ~40% is **unusually high**; typical industrials are more in the **5–15%** range.
  - Payroll at 0% is **not credible**; industrial businesses are usually staff-intensive.

**Suggestions**

- Enforce a minimum `payroll_share` (e.g. **15–30%** of revenue) for industrials.  
- Limit `net_margin` for this sector (e.g. **5–20%**), unless an explicit “high-margin niche” flag is set.  
- Optionally differentiate between **product vs service** industrials and adjust gross margin ranges accordingly.

---

### 4. Startup – Consumer Staples

**Data (AC)**  
- Revenue: **€441k** (+12% vs PY)  
- COGS (Procurement): **€187k** (~42.5% of revenue)  
- Payroll: **€107k** (~24.2% of revenue)  
- Other Opex: **€536k** (~121.6% of revenue)  
- Net profit: **–€389k** → **net margin ~–88.2%**  
- Gross margin: **~57.6%**

**Plausibility**

- **Scale & growth**: Very plausible for an early-stage consumer brand (low single-digit 100k revenue, double-digit growth).  
- **Gross margin** (~55–60%) is realistic for premium consumer staples (food, cosmetics, D2C).  
- **Loss level**:
  - Operating expenses totalling **~188% of revenue** and net margin of **–88%** are aggressive but believable for a **hyper-growth, marketing-heavy startup**.
  - Fits a narrative like: “We are intentionally burning cash to build brand and distribution.”


## Summary of UX findings (Mario)
- display company name, sector and size also in result page (some fancy header, sally think about it)
- "Tip: Save on Generation Costs" should this really be displayed to the audience?
- "Mutli-view consistency verified" also for the audience or is that more a debug feature?
- Links to CSV and API are scattered across the page, in my opionon they should be at the top (optionally bottom too) somewhere (sally find the right place)
- result page simple and clean, for each view 1 chart, 1 table, maybe 2 charts but side by side 50/50. i ll come back to this point later (I have to think about it)
