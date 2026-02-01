
# tellingCube – Event‑Driven Simulation Architecture (Draft) - Post-PoC(!!)

## 0. Context & Goals

**Goal**  
Extend the event‑driven business simulation engine (“tellingCube”) that generates **internally consistent synthetic data** for a company across multiple views:

- Finance (P&L, cashflow, tax)
- Controlling (cost centers, CAPEX/OPEX)
- Sales (products, regions, channels)
- HR (headcount, hires/exits, payroll)
- ...

All views must be derived from the **same underlying events**, not from independent prompts.

The solution combines:

- an **event‑centric data model** (event store + projections),
- a **multi‑stage LLM prompt pipeline** (scenario → events → views),
- optional **fine‑tuning** of an LLM for more stability and domain‑specific behaviour.

---

## 1. Data Layer – Event‑Centric Design

### 1.1 Design Approach

Use a hybrid between:

1. A **single raw event store** (append‑only, minimal schema) – similar to event sourcing.
2. **Derived fact tables** per business process (Sales, HR, Expense, etc.) following Kimball‑style dimensional modelling.

This avoids a large “centipede” fact table with 100+ sparsely populated columns, but still keeps a **single source of truth at event level**.

### 1.2 Raw Event Store (Single Table)

**Table: `events`** (append‑only)

- `event_id` (PK)
- `company_id`
- `scenario_id`
- `event_type`  
  e.g. `sale`, `hr_hire`, `hr_exit`, `expense`, `capex`, `tax_payment`, …
- `event_ts` / `event_month`
- `source` (e.g. `synthetic_llm`, `manual`, `import`)
- `payload` (JSON / JSONB)

**Payload examples**

- `sale`  
  `product`, `customerSegment`, `region`, `quantity`, `unitPrice`, `revenue`, `cogs`, `salesChannel`

- `hr_hire`  
  `employeeId`, `department`, `role`, `monthlySalary`, `fte`

- `expense`  
  `costCenter`, `account`, `amount`, `isCapex`

- `tax_payment`  
  `taxType`, `baseAmount`, `taxRate`, `taxAmount`

**Why a single event table?**

- Simplifies the LLM interface: the “event prompt” only needs to output **one JSON event format**.
- Ingestion is straightforward: one table, one insert path.
- Works well with projections: any number of downstream views can be materialised from the same log.

### 1.3 Derived Fact & Dimension Tables (Analytics Layer)

On top of `events`, define **dimensional models** (Kimball style).

**Core dimensions**

- `dim_company`
- `dim_date` (month/quarter/year, calendar and fiscal)
- `dim_product`
- `dim_customer_segment`
- `dim_employee`
- `dim_cost_center`
- `dim_scenario` (baseline, growth, recession, …)

**Example fact tables**

- `fact_sales_monthly`  
  Grain: company + scenario + product + month  
  Measures: `revenue`, `cogs`, `gross_margin`, `quantity`, …

- `fact_hr_monthly`  
  Grain: company + scenario + department + month  
  Measures: `opening_headcount`, `hires`, `exits`, `closing_headcount`, `payroll_cost`

- `fact_expense_monthly`  
  Grain: company + scenario + cost_center + month  
  Measures: `opex`, `capex`

- `fact_finance_monthly`  
  Grain: company + scenario + month  
  Measures: `revenue`, `cogs`, `opex`, `ebitda`, `depreciation`, `ebit`,
  `interest`, `tax`, `net_income`, `cash_change`

**Population strategy**

Fact tables are populated via **ETL/ELT projections** from `events`:

- `fact_sales_monthly` = aggregate of `events` where `event_type = 'sale'` grouped by month/product.
- HR headcount snapshots built by replaying `hr_hire` + `hr_exit` events over time.
- Finance facts calculated from sales, expenses, capex and tax events.

### 1.4 Design Trade‑Offs (for Discussion)

- **Single events table vs. multiple typed tables**  
  - Recommended: single `events` table + JSON payload for flexibility and simple ingestion.  
  - Typed fact tables for analytics are created **downstream**, not at ingestion.

---

## 2. LLM & Prompt Architecture for tellingCube

The LLM side is organised as a **prompt pipeline** (prompt chaining): each stage takes structured input and produces structured JSON output that flows into the next stage.

### 2.1 Stage 0 – System / Role Prompt

(Defined once per deployed model)

- Role: “structured business simulation engine”.
- Core principles:
  - think in **events over time**,
  - keep numbers **realistic and internally consistent**,
  - when asked for data: **return JSON only**, following the given schema.
- Awareness: multiple views (Finance, HR, Sales, Controlling, ...) must all derive from the **same events**.

### 2.2 Stage 1 – Scenario Prompt (Company & Year Definition)

**Input**

- GICS sector, region, currency  
- Company stage / size (e.g. mid‑cap, revenue run‑rate, employee count)  
- Scenario (baseline, growth, mild recession, …)  
- Time horizon (e.g. 12 months / 8 quarters)

**Output JSON**

```json
{
  "companyProfile": {
    "name": "ExampleTech AG",
    "gicsSector": "Information Technology",
    "region": "Europe",
    "currency": "EUR",
    "companyStage": "mid-cap",
    "scenario": "baseline"
  },
  "year": 2025,
  "narrative": [
    { "month": "2025-03", "event": "Launch of Product B in DACH region." },
    { "month": "2025-07", "event": "Cost optimisation in operations." }
  ],
  "startingState": {
    "revenueRunRate": 120000000,
    "employees": 420,
    "netDebt": 20000000,
    "cash": 8000000
  }
}
```

This JSON acts as the **seed** for event generation.

### 2.3 Stage 2 – Event Prompt (Event Ledger Generation)

**Purpose**

Generate all **monthly business events** for the selected year that drive Finance, Sales, HR, Controlling, Tax etc., based on the scenario.

**Input**

Scenario JSON from Stage 1.

**Output JSON**

```json
{
  "companyProfile": { ... },
  "events": [
    {
      "eventType": "sale",
      "month": "2025-01",
      "date": "2025-01-15",
      "product": "A",
      "customerSegment": "SMB",
      "region": "AT",
      "quantity": 35,
      "unitPrice": 2000,
      "revenue": 70000,
      "cogs": 28000,
      "salesChannel": "Direct"
    },
    {
      "eventType": "hr_hire",
      "month": "2025-01",
      "date": "2025-01-10",
      "employeeId": "E001",
      "role": "Sales Manager",
      "department": "Sales",
      "costCenter": "Sales",
      "monthlySalary": 4500,
      "fte": 1.0
    },
    {
      "eventType": "expense",
      "month": "2025-01",
      "date": "2025-01-05",
      "costCenter": "Marketing",
      "account": "Online Ads",
      "amount": 15000
    },
    {
      "eventType": "tax_payment",
      "month": "2025-04",
      "date": "2025-04-15",
      "taxType": "VAT",
      "baseAmount": 200000,
      "taxRate": 0.2,
      "taxAmount": 40000
    }
  ]
}
```

**Key instructions for this stage**

- Cover at least these `eventType` values:
  - `sale`, `hr_hire`, `hr_exit`, `expense`, `capex`, `tax_payment`
- Ensure that:
  - every month in the horizon is represented,
  - revenue and headcount evolve smoothly from `startingState`,
  - financial figures are realistic for the sector and company size.
- Return **ONLY valid JSON**, matching the schema that maps directly into the `events` table.

The resulting JSON is persisted to the `events` table (one row per event).

### 2.4 Stage 3 – Views / Cubes

From this point, all views (Finance, HR, Sales, Controlling, ...) are derived from the same `events` data.

There are two options:

#### Option A – Deterministic Views (Recommended Baseline)

- Build **SQL/ETL projections** from `events` into:
  - `fact_sales_monthly`
  - `fact_hr_monthly`
  - `fact_expense_monthly`
  - `fact_finance_monthly`
- Reporting and cubes (tellingCube) read from these fact tables using conformed dimensions.

This gives full control, traceability and repeatability – the LLM is only responsible for generating events, not for aggregating or doing accounting logic.

#### Option B – LLM‑Assisted Views (Optional)

For narrative or exploratory features, an LLM can take the `events` JSON as input and produce:

- a Finance summary JSON (and/or natural‑language explanation),
- a HR summary JSON,
- a Sales or Controlling narrative.

Example: Finance view prompt

- Input: `events` JSON for one company/scenario.
- Instructions: aggregate to a monthly P&L structure, return JSON only.

These outputs can be used as an additional layer on top of the deterministic cubes (e.g. for storytelling, commentary, anomaly explanations).

---

## 3. Fine‑Tuning Strategy (Optional)

### Phase 1 – Prompt‑Only

- Start with a strong LLM (hosted or open‑source) and rely on:
  - robust prompt templates (Stages 0–2),
  - JSON schema validation and automatic retries on invalid output.
- No model training required; faster iteration and simpler operations.

### Phase 2 – Parameter‑Efficient Fine‑Tuning

When higher stability or sector‑specific accuracy is needed:

- Collect training pairs:
  - **Scenario → Events** (high‑quality “golden” event ledgers),
  - optionally **Events → Aggregated Views**.
- Fine‑tune an open‑source base model using **PEFT/LoRA** to specialise it as a “business event simulator”.
- Benefits:
  - Higher probability of schema‑correct JSON,
  - More realistic behaviour per GICS sector,
  - Less dependence on long, complex prompts.

Fine‑tuning is an optimisation step; the architecture does not depend on it from day one.

---

## 4. Open Questions for Architecture

1. **Event Schema Governance**  
   - How strict is the JSON schema (validation, versioning)?  
   - Do we align with CloudEvents‑style envelopes for interoperability?

2. **Projection Layer**  
   - Implementation via SQL views, materialised views, batch ETL or streaming pipelines?  
   - Recompute strategy: on demand vs. scheduled.

3. **Model Hosting & Costing**  
   - Single provider vs. hybrid (managed + self‑hosted open‑source).  
   - Latency and cost constraints per simulation run.

4. **Versioning & Reproducibility**  
   - Versioning of:
     - scenarios,
     - event generations,
     - LLM model versions,
     - prompt templates.  
   - Ability to reproduce a given simulation exactly for audit or demo.

5. **Security & Tenancy**  
   - Multi‑tenant separation at `company_id` / `scenario_id`.  
   - Data retention policies for synthetic vs. real data (future extension).

---

This document is intended as a **discussion starter** for architecture around tellingCube’s data model and LLM integration. It focuses on:
- a single raw event store as the backbone,
- derived analytical fact tables for Finance/HR/Sales/Controlling,
- and a three‑stage LLM prompt pipeline (scenario → events → views).
