# tellingCube – Post-PoC Event-Driven Architecture

**Version:** 1.0 (Draft)
**Status:** Planning / Discussion
**Authors:** Mario (Concept), Winston (Architecture Review)
**Date:** 2025-12-10

---

## 0. Executive Summary

This document defines the target architecture for tellingCube's data generation engine post-PoC. It extends the current event-sourcing foundation with:

- A **single raw event store** with JSONB payloads for flexibility
- **Kimball-style dimensional modeling** for analytics
- A **three-stage LLM prompt pipeline** (Scenario → Events → Views)
- **Deterministic SQL projections** for all numerical views (Option A)
- **Optional LLM narrative layer** for commentary and insights (Option B)

**Core Principle:** LLM generates events, deterministic code aggregates views.

---

## 1. Data Layer – Event-Centric Design

### 1.1 Design Philosophy

Use a hybrid approach:

1. **Single raw event store** (append-only, JSONB payload) – event sourcing style
2. **Derived fact tables** per business process – Kimball-style dimensional modeling

This avoids a "centipede" fact table with 100+ sparse columns while maintaining a **single source of truth at event level**.

### 1.2 Raw Event Store

**Table: `events`** (append-only, single table)

```sql
CREATE TABLE events (
  event_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  company_id UUID NOT NULL,
  scenario_id UUID NOT NULL,
  event_type VARCHAR(50) NOT NULL,
  event_ts TIMESTAMP NOT NULL,
  event_month VARCHAR(7) NOT NULL,  -- '2024-01' for fast grouping
  source VARCHAR(20) DEFAULT 'synthetic_llm',
  payload JSONB NOT NULL,
  created_at TIMESTAMP DEFAULT NOW()
);

-- Essential indexes for query performance
CREATE INDEX idx_events_scenario ON events(scenario_id);
CREATE INDEX idx_events_type ON events(event_type);
CREATE INDEX idx_events_month ON events(event_month);
CREATE INDEX idx_events_company ON events(company_id);
```

**Supported Event Types:**

| event_type | Description | Key Payload Fields |
|------------|-------------|-------------------|
| `sale` | Revenue transaction | product, region, quantity, unitPrice, revenue, cogs, channel |
| `hr_hire` | Employee onboarding | employeeId, department, role, monthlySalary, fte |
| `hr_exit` | Employee departure | employeeId, department, exitReason |
| `expense` | Operating expense | costCenter, account, amount, isCapex |
| `procurement` | Material/supply purchase | supplier, category, amount |
| `tax_payment` | Tax obligation | taxType, baseAmount, taxRate, taxAmount |
| `cash_movement` | Non-operational cash flow | category, amount, direction |

**Why Single Table + JSONB:**

- ✅ Simplifies LLM integration (one insert path)
- ✅ Schema evolution without migrations (add fields to JSON)
- ✅ Cross-event queries without UNIONs
- ✅ Flexibility at ingestion, structure at analytics layer

**Validation Strategy:** Zod schemas validate payload structure on insert (already implemented in PoC).

### 1.3 Derived Fact Tables (Analytics Layer)

Fact tables are populated via **SQL projections** from the raw event store.

**Core Dimensions:**

```sql
dim_company       -- Company metadata
dim_date          -- Month/quarter/year (calendar + fiscal)
dim_product       -- Product catalog
dim_region        -- Geographic hierarchy
dim_department    -- Org structure
dim_cost_center   -- Cost allocation
dim_scenario      -- baseline, growth, recession, plan, forecast
```

**Fact Tables:**

```sql
-- Sales Performance
fact_sales_monthly (
  company_id, scenario_id, product_id, region_id, month,
  revenue, cogs, gross_margin, quantity, transactions
)

-- HR Metrics
fact_hr_monthly (
  company_id, scenario_id, department_id, month,
  opening_headcount, hires, exits, closing_headcount, payroll_cost
)

-- Operating Expenses
fact_expense_monthly (
  company_id, scenario_id, cost_center_id, month,
  opex, capex
)

-- Financial Summary (P&L)
fact_finance_monthly (
  company_id, scenario_id, month,
  revenue, cogs, gross_profit, opex, ebitda,
  depreciation, ebit, interest, tax, net_income, cash_change
)
```

**Population Strategy:**

- `fact_sales_monthly` = aggregate events WHERE event_type = 'sale'
- `fact_hr_monthly` = replay hr_hire + hr_exit events over time
- `fact_finance_monthly` = calculated from sales + expense + tax events

**Implementation:** Materialized views in PostgreSQL/NeonDB with REFRESH on demand after generation.

---

## 2. LLM Prompt Architecture

The LLM pipeline is organized as **prompt chaining**: each stage produces structured JSON that flows to the next.

### 2.1 Stage 0 – System Prompt (Role Definition)

Defined once per deployment:

```
Role: "Structured business simulation engine"

Core Principles:
- Think in events over time
- Keep numbers realistic and internally consistent
- Return JSON only, following the provided schema
- Multiple views (Finance, HR, Sales) derive from the SAME events
```

### 2.2 Stage 1 – Scenario Prompt (Company Seed)

**Input:** User selections (industry, size, scenario type, time horizon)

**Output JSON:**

```json
{
  "companyProfile": {
    "name": "Alpine Bakery GmbH",
    "gicsSector": "Consumer Staples",
    "region": "Europe",
    "currency": "EUR",
    "companyStage": "small-business",
    "scenario": "baseline"
  },
  "year": 2024,
  "narrative": [
    { "month": "2024-03", "event": "Easter seasonal peak" },
    { "month": "2024-07", "event": "Summer tourism boost" }
  ],
  "startingState": {
    "revenueRunRate": 120000,
    "employees": 8,
    "cash": 25000
  }
}
```

This JSON becomes the **seed** for event generation.

### 2.3 Stage 2 – Event Prompt (Event Ledger Generation)

**Input:** Scenario JSON from Stage 1

**Output:** Array of business events for the time horizon

```json
{
  "events": [
    {
      "eventType": "sale",
      "month": "2024-01",
      "product": "Sourdough Bread",
      "region": "Vienna",
      "quantity": 450,
      "unitPrice": 4.50,
      "revenue": 2025,
      "cogs": 810
    },
    {
      "eventType": "hr_hire",
      "month": "2024-02",
      "employeeId": "E009",
      "role": "Baker",
      "department": "Production",
      "monthlySalary": 2800
    }
  ]
}
```

**Generation Strategy (Current):** Quarterly batching to manage token limits and ensure temporal consistency.

**Validation:** Each event validated against Zod schema before database insert.

### 2.4 Stage 3 – Views (Projection Layer)

**Decision: Option A (Deterministic) as foundation, Option B (LLM) as enhancement**

#### Option A – Deterministic SQL Projections (Required)

All numerical views are derived via SQL aggregation:

```
events → SQL/Materialized Views → fact_sales, fact_hr, fact_finance
```

**Benefits:**
- ✅ Reproducible (same input = same output)
- ✅ Auditable ("Revenue = SUM(events WHERE type='sale')")
- ✅ Fast (milliseconds, no API calls)
- ✅ Debuggable (trace the SQL)

**This is non-negotiable for data that must reconcile across views.**

#### Option B – LLM Narrative Layer (Optional Enhancement)

For storytelling and insights on top of deterministic data:

- Executive commentary ("March revenue dropped 12% due to...")
- Anomaly explanations ("HR costs spiked due to 3 senior hires")
- Scenario comparisons ("In recession scenario, margin contracts to...")

**Architecture:**

```
┌─────────────────────────────────────────────────────┐
│                    events (raw)                      │
└─────────────────────┬───────────────────────────────┘
                      │
        ┌─────────────┴─────────────┐
        ▼                           ▼
┌───────────────────┐     ┌───────────────────────────┐
│ Option A          │     │ Option B                  │
│ Deterministic SQL │     │ LLM Narrative Layer       │
│                   │     │                           │
│ fact_sales        │     │ "Explain this trend"      │
│ fact_hr           │     │ "Summarize P&L drivers"   │
│ fact_finance      │     │ "Compare scenarios"       │
└─────────┬─────────┘     └─────────────┬─────────────┘
          │                             │
          ▼                             ▼
┌─────────────────────────────────────────────────────┐
│              UI / Dashboard / Export                 │
│  Numbers from (A)  +  Commentary from (B)           │
└─────────────────────────────────────────────────────┘
```

---

## 3. IBCS Scenario Support

The architecture must support IBCS© scenario types:

| Scenario | Abbr | Description | Implementation |
|----------|------|-------------|----------------|
| Actual | AC | What really happened | Generated events for current year |
| Previous Year | PY | Last year's actuals | Calculated from AC via growth rates OR separate generation |
| Plan | PL | Budget set before period | Calculated OR separate scenario generation |
| Forecast | FC | Updated expectation | Calculated OR separate scenario generation |

**Current PoC Approach:** AC generated, PY/PL/FC calculated at projection layer using industry growth rates.

**Post-PoC Options:**

1. **Keep calculated approach** – Simple, consistent, fast
2. **Generate separate scenarios** – More realistic variation, higher cost (4x generation time)

**Recommendation:** Keep calculated approach for MVP+, consider separate generation for enterprise tier.

---

## 4. Fine-Tuning Strategy (Future)

### Phase 1 – Prompt-Only (Current + Post-PoC)

- Strong base LLM (Claude) with robust prompt templates
- JSON schema validation with automatic retries
- No model training required

### Phase 2 – Parameter-Efficient Fine-Tuning (Future)

When higher stability or sector-specific accuracy is needed:

- Collect training pairs (Scenario → Events golden datasets)
- Fine-tune open-source model using PEFT/LoRA
- Benefits: Higher schema compliance, sector-specific realism

**Fine-tuning is an optimization, not a requirement.**

---

## 5. Open Questions (For Future Discussion)

### 5.1 Event Schema Governance
- Validation strictness and versioning strategy
- CloudEvents-style envelopes? (Recommendation: Not for MVP, evaluate later)

### 5.2 Projection Layer Implementation
- SQL views vs. materialized views vs. batch ETL
- Recommendation: Materialized views with on-demand REFRESH

### 5.3 Model Hosting & Costing
- Single provider vs. hybrid (managed + self-hosted)
- Cost constraints per simulation run

### 5.4 Versioning & Reproducibility
- Version: scenarios, event generations, LLM model versions, prompt templates
- Ability to reproduce simulation exactly for audit/demo

### 5.5 Security & Tenancy
- Multi-tenant separation at company_id / scenario_id level
- Data retention policies

---

## 6. Implementation Phases

| Phase | Scope | Effort |
|-------|-------|--------|
| **PoC** (Current) | Single company, single region, calculated PY/PL/FC | ✅ Done |
| **Phase 1** | Add company_id dimension (multi-company) | Small |
| **Phase 2** | Add region dimension with hierarchical generation | Medium |
| **Phase 3** | Full Kimball dimensional model (fact tables) | Medium |
| **Phase 4** | LLM narrative layer (Option B) | Medium |
| **Phase 5** | Fine-tuning evaluation | Large |

---

## 7. References

- Original concept: `docs/_masemIT/ideas/after-poc-architecture-(data-generation).md`
- Current architecture: `docs/architecture.md`
- IBCS standards: `docs/standards/ibcs-standards.md`
- Tech stack: `docs/architecture/tech-stack.md`
