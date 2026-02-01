# Two-Phase Generation Architecture

**Author:** Winston (Architect)
**Date:** December 2024
**Status:** Draft - Ready for Review
**ROF Session:** Claude Test Round 3

---

## Executive Summary

After 3 rounds of testing, we've proven that Claude cannot reliably generate events with correct financial ratios. This spec defines a new architecture where:

1. **Claude** provides narrative guidance (what it's good at)
2. **Code** derives events with correct math (guaranteed ratios)

**Result:** 2 Claude calls instead of 3+ stages, deterministic financial accuracy, support for product groups/regions/cost centers.

---

## Problem Statement

| Round | Startup | Midcap | Largecap | Root Cause |
|-------|---------|--------|----------|------------|
| 1 | 5/10 | 6/10 | 4/10 | Cash flow bugs, ratio violations |
| 2 | 5/10 | 6/10 | 4/10 | Payroll %, gross margin wrong |
| 3 | 4/10 | 3/10 | 2/10 | COGS+Payroll >100%, Other bloated |

**Core Issue:** Claude treats cost categories independently without sum constraints.

---

## Solution: Two-Phase Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│ PHASE 1: World Building (1 Claude call)                         │
│ ─────────────────────────────────────────                       │
│ Input:  Tier, Sector, optional parameters                       │
│ Output: Company, Employees[], Customers[], Vendors[],           │
│         Products[], ProductGroups[], Regions[], CostCenters[]   │
│ Token cost: ~1 call                                             │
└─────────────────────────────────────────────────────────────────┘
                              ↓
                       Stored in DB
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│ PHASE 2: Monthly Course (1 Claude call for ALL months)          │
│ ─────────────────────────────────────────                       │
│ Input:  World from Phase 1, Tier constraints, time range        │
│ Output: MonthlyCourse[] (12-36 months in one response)          │
│ Token cost: ~1 call                                             │
└─────────────────────────────────────────────────────────────────┘
                              ↓
                    No DB storage needed
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│ PHASE 3: Event Derivation (Code only - NO Claude)               │
│ ─────────────────────────────────────────                       │
│ Input:  World + MonthlyCourse[] + TierConfig (ratios)           │
│ Output: Events[] with guaranteed correct financial ratios       │
│ Token cost: 0                                                   │
└─────────────────────────────────────────────────────────────────┘
                              ↓
                       Stored in DB
```

**Total Claude calls: 2** (vs current 3+ stage approach)

---

## Phase 1: World Building Schema

### TypeScript Interface

```typescript
interface ScenarioWorld {
  company: {
    name: string
    foundedYear: number
    story: string                // 2-3 sentences founding narrative
    headquarters: string         // "Wien", "Graz", etc.
  }

  employees: Array<{
    id: string                   // Generated UUID
    name: string
    role: string                 // "Bäckermeister", "Controller", etc.
    department: string           // Maps to cost center
    salaryBand: "junior" | "mid" | "senior" | "executive"
    hireDate: string             // "2022-03"
    region?: string              // If multi-region
  }>

  customers: Array<{
    id: string
    name: string
    type: "regular" | "occasional" | "key_account"
    avgOrderSize: number         // Rough EUR guidance
    region: string               // Which region they belong to
    preferredProducts?: string[] // Product categories they typically buy
  }>

  vendors: Array<{
    id: string
    name: string
    supplies: string             // "flour", "packaging", "equipment"
    category: string             // Maps to COGS category
    isMainSupplier: boolean
  }>

  products: Array<{
    id: string
    name: string
    category: string             // Product group name
    priceRange: [number, number] // [min, max] EUR
    marginProfile: "low" | "medium" | "high"
  }>

  productGroups: Array<{
    name: string
    revenueShare: number         // % of total revenue (must sum to 100)
  }>

  regions: Array<{
    name: string
    type: "headquarters" | "branch" | "sales_office"
    revenueShare: number         // % of total revenue (must sum to 100)
  }>

  costCenters: Array<{
    name: string
    type: "production" | "sales" | "admin" | "management" | "rd"
    costShare: number            // % of opex excl. COGS (must sum to 100)
  }>
}
```

### Pool Sizes by Tier

| Pool | Startup | Midcap | Largecap |
|------|---------|--------|----------|
| Employees | 8-15 | 15-25 | 20-30 |
| Customers | 10-20 | 15-30 | 20-40 |
| Vendors | 5-10 | 8-15 | 10-20 |
| Products | 3-6 | 5-8 | 6-10 |
| Product Groups | 2-4 | 3-5 | 4-6 |
| Regions | 1 | 1-3 | 2-5 |
| Cost Centers | 2-3 | 3-5 | 4-6 |

### Example Output (Startup Bakery)

```yaml
company:
  name: "Bäckerei Sonnenschein"
  foundedYear: 2022
  story: "Founded by master baker Hans Gruber after 15 years at a traditional Viennese bakery. Started with a small shop in Floridsdorf, known for artisanal sourdough."
  headquarters: "Wien"

employees:
  - id: "emp-001"
    name: "Hans Gruber"
    role: "Geschäftsführer & Bäckermeister"
    department: "Geschäftsführung"
    salaryBand: "executive"
    hireDate: "2022-01"
  - id: "emp-002"
    name: "Maria Hofer"
    role: "Bäckerin"
    department: "Produktion"
    salaryBand: "mid"
    hireDate: "2022-03"
  # ... more employees

customers:
  - id: "cust-001"
    name: "Café Landtmann"
    type: "key_account"
    avgOrderSize: 450
    region: "Wien"
    preferredProducts: ["Gebäck", "Torten"]
  - id: "cust-002"
    name: "Familie Müller"
    type: "regular"
    avgOrderSize: 25
    region: "Wien"
  # ... more customers

vendors:
  - id: "vend-001"
    name: "Müller Mehl GmbH"
    supplies: "Mehl, Getreide"
    category: "Rohstoffe"
    isMainSupplier: true
  - id: "vend-002"
    name: "Eier Huber"
    supplies: "Eier, Milchprodukte"
    category: "Rohstoffe"
    isMainSupplier: false
  # ... more vendors

products:
  - id: "prod-001"
    name: "Bauernbrot"
    category: "Brot"
    priceRange: [3.50, 4.50]
    marginProfile: "medium"
  - id: "prod-002"
    name: "Croissant"
    category: "Gebäck"
    priceRange: [1.80, 2.50]
    marginProfile: "high"
  - id: "prod-003"
    name: "Hochzeitstorte"
    category: "Torten"
    priceRange: [150, 400]
    marginProfile: "high"

productGroups:
  - name: "Brot"
    revenueShare: 45
  - name: "Gebäck"
    revenueShare: 35
  - name: "Torten"
    revenueShare: 20

regions:
  - name: "Wien"
    type: "headquarters"
    revenueShare: 100

costCenters:
  - name: "Produktion"
    type: "production"
    costShare: 60
  - name: "Verkauf"
    type: "sales"
    costShare: 25
  - name: "Verwaltung"
    type: "admin"
    costShare: 15
```

---

## Phase 2: Monthly Course Schema

### TypeScript Interface

```typescript
interface MonthlyCourse {
  month: string                  // "2024-03"

  revenue: {
    target: number               // EUR target for month
    yoyGrowth: number            // % vs same month last year
  }

  headcount: {
    total: number
    changes?: Array<{
      action: "hire" | "departure"
      role: string
      reason?: string            // "expansion", "resignation", "retirement"
    }>
  }

  mood: "growing" | "stable" | "struggling"

  narrative: string              // 1-2 sentences about the month

  notableEvents?: Array<{
    type: "new_client" | "lost_client" | "equipment" | "expansion" |
          "cost_saving" | "marketing_campaign" | "seasonal_peak" | "incident"
    description: string
    impactEstimate?: number      // Rough EUR impact (optional)
  }>

  cashFlowNote?: string          // Special cash timing (delayed payment, prepayment, etc.)
}

// Full Phase 2 output
interface Phase2Output {
  timeRange: {
    start: string                // "2024-01"
    end: string                  // "2024-12"
  }
  months: MonthlyCourse[]
}
```

### Example Output (12 months)

```yaml
timeRange:
  start: "2024-01"
  end: "2024-12"

months:
  - month: "2024-01"
    revenue:
      target: 85000
      yoyGrowth: 45
    headcount:
      total: 10
    mood: "growing"
    narrative: "Strong January after holiday season. New year brings optimism and returning regular customers."
    notableEvents:
      - type: "seasonal_peak"
        description: "Three Kings Day (Dreikönigstag) brings extra pastry orders"
        impactEstimate: 5000

  - month: "2024-02"
    revenue:
      target: 72000
      yoyGrowth: 42
    headcount:
      total: 10
    mood: "stable"
    narrative: "Typical February slowdown. Focus on cost control and recipe development."

  - month: "2024-03"
    revenue:
      target: 95000
      yoyGrowth: 48
    headcount:
      total: 11
      changes:
        - action: "hire"
          role: "Bäcker"
          reason: "expansion"
    mood: "growing"
    narrative: "Easter preparations drive strong March. Hired additional baker to meet demand."
    notableEvents:
      - type: "new_client"
        description: "Signed contract with Hotel Sacher for Easter brunch supply"
        impactEstimate: 8000

  # ... 9 more months
```

---

## Phase 3: Event Derivation Engine

### Overview

The Event Derivation Engine transforms Phase 1 (World) + Phase 2 (Monthly Course) into concrete events with correct financial ratios.

### Derivation Rules

#### 1. Sales Events

```typescript
function deriveSalesEvents(
  month: MonthlyCourse,
  world: ScenarioWorld,
  config: TierConfig
): Event[] {
  const targetRevenue = month.revenue.target
  const events: Event[] = []

  // Distribute by product group
  for (const group of world.productGroups) {
    const groupRevenue = targetRevenue * (group.revenueShare / 100)

    // Distribute by region
    for (const region of world.regions) {
      const regionRevenue = groupRevenue * (region.revenueShare / 100)

      // Create daily sales with seasonality
      const dailySales = applySeasonality(regionRevenue, month.month, config.sector)

      for (const [day, amount] of dailySales) {
        // Pick random customer from region
        const customer = pickRandomCustomer(world.customers, region.name)
        // Pick random product from group
        const product = pickRandomProduct(world.products, group.name)

        events.push({
          eventType: 'sales_transaction',
          date: `${month.month}-${day}`,
          amount: amount,
          description: `${product.name} - ${customer.name}`,
          productGroup: group.name,
          region: region.name,
          customerId: customer.id,
          productId: product.id
        })
      }
    }
  }

  return events
}
```

#### 2. Payroll Events

```typescript
function derivePayrollEvents(
  month: MonthlyCourse,
  world: ScenarioWorld,
  config: TierConfig
): Event[] {
  const events: Event[] = []

  // Get active employees for this month
  const activeEmployees = world.employees.filter(
    emp => emp.hireDate <= month.month
  )

  // Calculate salary by band from TierConfig
  for (const employee of activeEmployees) {
    const salaryRange = config.salaryRanges[employee.salaryBand]
    const salary = randomInRange(salaryRange.min, salaryRange.max)

    events.push({
      eventType: 'payroll',
      date: `${month.month}-28`,  // Payday
      amount: salary,
      description: `Gehalt ${employee.name} (${employee.role})`,
      costCenter: employee.department,
      employeeId: employee.id
    })
  }

  return events
}
```

#### 3. COGS/Procurement Events

```typescript
function deriveProcurementEvents(
  month: MonthlyCourse,
  world: ScenarioWorld,
  config: TierConfig
): Event[] {
  const targetCOGS = month.revenue.target *
    midpoint(config.financialConstraints.cogsShare)

  const events: Event[] = []

  // Distribute across vendors
  for (const vendor of world.vendors) {
    const vendorShare = vendor.isMainSupplier ? 0.4 : 0.1
    const vendorAmount = targetCOGS * vendorShare

    // Split into 2-4 deliveries per month
    const deliveryCount = randomInt(2, 4)
    const deliveryAmount = vendorAmount / deliveryCount

    for (let i = 0; i < deliveryCount; i++) {
      const day = randomInt(1, 28)
      events.push({
        eventType: 'procurement',
        date: `${month.month}-${day.toString().padStart(2, '0')}`,
        amount: deliveryAmount,
        description: `${vendor.supplies} - ${vendor.name}`,
        vendorId: vendor.id,
        costCenter: 'Produktion'
      })
    }
  }

  return events
}
```

#### 4. Other/SG&A Events

```typescript
function deriveOtherEvents(
  month: MonthlyCourse,
  world: ScenarioWorld,
  config: TierConfig
): Event[] {
  const targetOther = month.revenue.target *
    midpoint(config.financialConstraints.otherShare)

  const events: Event[] = []

  // Fixed monthly costs
  const fixedCosts = [
    { name: 'Miete', share: 0.35, day: 1 },
    { name: 'Versicherung', share: 0.15, day: 1 },
    { name: 'Energie', share: 0.20, day: 15 },
    { name: 'Telefon/Internet', share: 0.05, day: 10 },
    { name: 'Buchhaltung', share: 0.10, day: 20 },
    { name: 'Sonstiges', share: 0.15, day: 15 }
  ]

  // Distribute by cost center
  for (const costCenter of world.costCenters) {
    const ccAmount = targetOther * (costCenter.costShare / 100)

    for (const cost of fixedCosts) {
      events.push({
        eventType: 'other_expense',
        date: `${month.month}-${cost.day.toString().padStart(2, '0')}`,
        amount: ccAmount * cost.share,
        description: cost.name,
        costCenter: costCenter.name
      })
    }
  }

  return events
}
```

#### 5. Notable Events

```typescript
function deriveNotableEvents(
  month: MonthlyCourse,
  world: ScenarioWorld
): Event[] {
  const events: Event[] = []

  for (const notable of month.notableEvents || []) {
    switch (notable.type) {
      case 'new_client':
        // Add extra sales event
        events.push({
          eventType: 'sales_transaction',
          date: `${month.month}-15`,
          amount: notable.impactEstimate || 5000,
          description: `Neukunde: ${notable.description}`,
          isNotableEvent: true
        })
        break

      case 'equipment':
        // Add procurement/investment event
        events.push({
          eventType: 'procurement',
          date: `${month.month}-10`,
          amount: notable.impactEstimate || 3000,
          description: `Investition: ${notable.description}`,
          isNotableEvent: true
        })
        break

      case 'lost_client':
        // Reduce revenue (handled in adjustment)
        break

      // ... other types
    }
  }

  return events
}
```

### Financial Ratio Validation

After derivation, validate that ratios match TierConfig:

```typescript
function validateDerivedEvents(
  events: Event[],
  config: TierConfig
): ValidationResult {
  const totals = calculateTotals(events)

  const checks = [
    {
      name: 'Gross Margin',
      actual: (totals.revenue - totals.cogs) / totals.revenue,
      expected: config.financialConstraints.grossMargin,
    },
    {
      name: 'Net Margin',
      actual: totals.netProfit / totals.revenue,
      expected: config.financialConstraints.netMargin,
    },
    {
      name: 'Payroll Share',
      actual: totals.payroll / totals.revenue,
      expected: config.financialConstraints.payrollShare,
    },
    {
      name: 'COGS Share',
      actual: totals.cogs / totals.revenue,
      expected: config.financialConstraints.cogsShare,
    },
    {
      name: 'Other Share',
      actual: totals.other / totals.revenue,
      expected: config.financialConstraints.otherShare,
    },
  ]

  return {
    passed: checks.every(c =>
      c.actual >= c.expected.min && c.actual <= c.expected.max
    ),
    checks
  }
}
```

---

## Seasonality Curves

### By Sector

```typescript
const SEASONALITY_CURVES: Record<string, number[]> = {
  // Index 0-11 = Jan-Dec, values are multipliers (1.0 = average)

  'consumer-staples': [
    1.1,  // Jan - post-holiday
    0.9,  // Feb - slow
    1.0,  // Mar - Easter prep
    1.2,  // Apr - Easter
    1.0,  // May
    0.9,  // Jun - summer start
    0.8,  // Jul - vacation
    0.8,  // Aug - vacation
    1.0,  // Sep - back to normal
    1.0,  // Oct
    1.1,  // Nov - pre-Christmas
    1.3,  // Dec - Christmas peak
  ],

  'financials': [
    1.1,  // Jan - year start
    1.0,  // Feb
    1.1,  // Mar - Q1 close
    0.9,  // Apr
    1.0,  // May
    1.1,  // Jun - Q2 close
    0.8,  // Jul - summer
    0.8,  // Aug - summer
    1.1,  // Sep - Q3 close
    1.0,  // Oct
    1.0,  // Nov
    1.1,  // Dec - year close
  ],

  'industrials': [
    0.9,  // Jan - slow start
    1.0,  // Feb
    1.1,  // Mar
    1.1,  // Apr
    1.1,  // May
    1.0,  // Jun
    0.8,  // Jul - plant shutdowns
    0.7,  // Aug - vacation
    1.1,  // Sep - ramp up
    1.1,  // Oct
    1.0,  // Nov
    0.9,  // Dec - year end
  ],
}
```

---

## Database Schema Updates

### New Table: scenario_world

```sql
CREATE TABLE scenario_world (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  scenario_id UUID REFERENCES scenario(id) ON DELETE CASCADE,
  world_data JSONB NOT NULL,  -- Full ScenarioWorld object
  created_at TIMESTAMP DEFAULT NOW()
);
```

### Updated Event Table

Add optional dimension columns:

```sql
ALTER TABLE event ADD COLUMN product_group VARCHAR(100);
ALTER TABLE event ADD COLUMN region VARCHAR(100);
ALTER TABLE event ADD COLUMN cost_center VARCHAR(100);
ALTER TABLE event ADD COLUMN is_notable_event BOOLEAN DEFAULT FALSE;
```

---

## Implementation Tasks for James

### Phase 1: Foundation (Priority: High)

| # | Task | Estimated Hours | Dependencies |
|---|------|-----------------|--------------|
| 1.1 | Create `lib/types/scenario-world.ts` with interfaces | 1h | None |
| 1.2 | Create `lib/types/monthly-course.ts` with interfaces | 0.5h | None |
| 1.3 | Add Prisma schema for `scenario_world` table | 0.5h | None |
| 1.4 | Add dimension columns to Event model | 0.5h | None |
| 1.5 | Run Prisma migration | 0.25h | 1.3, 1.4 |

### Phase 2: World Building Generator (Priority: High)

| # | Task | Estimated Hours | Dependencies |
|---|------|-----------------|--------------|
| 2.1 | Create `lib/prompts/world-building-prompt.ts` | 1.5h | 1.1 |
| 2.2 | Create `lib/generators/world-building-generator.ts` | 2h | 2.1 |
| 2.3 | Add JSON parsing & validation for World output | 1h | 2.2 |
| 2.4 | Store World in database | 0.5h | 1.3, 2.3 |
| 2.5 | Test with all 3 tiers | 1h | 2.4 |

### Phase 3: Monthly Course Generator (Priority: High)

| # | Task | Estimated Hours | Dependencies |
|---|------|-----------------|--------------|
| 3.1 | Create `lib/prompts/monthly-course-prompt.ts` | 1.5h | 1.2 |
| 3.2 | Create `lib/generators/monthly-course-generator.ts` | 2h | 3.1 |
| 3.3 | Add JSON parsing & validation for Course output | 1h | 3.2 |
| 3.4 | Test with all 3 tiers | 1h | 3.3 |

### Phase 4: Event Derivation Engine (Priority: High)

| # | Task | Estimated Hours | Dependencies |
|---|------|-----------------|--------------|
| 4.1 | Create `lib/derivation/sales-deriver.ts` | 2h | 1.1, 1.2 |
| 4.2 | Create `lib/derivation/payroll-deriver.ts` | 1h | 1.1, 1.2 |
| 4.3 | Create `lib/derivation/procurement-deriver.ts` | 1.5h | 1.1, 1.2 |
| 4.4 | Create `lib/derivation/other-deriver.ts` | 1h | 1.1, 1.2 |
| 4.5 | Create `lib/derivation/notable-event-deriver.ts` | 1h | 1.1, 1.2 |
| 4.6 | Create `lib/derivation/event-derivation-engine.ts` (orchestrator) | 1.5h | 4.1-4.5 |
| 4.7 | Add seasonality curves `lib/config/seasonality.ts` | 1h | None |
| 4.8 | Add financial ratio validation | 1h | 4.6 |

### Phase 5: Integration (Priority: High)

| # | Task | Estimated Hours | Dependencies |
|---|------|-----------------|--------------|
| 5.1 | Update `lib/generators/scenario-generator.ts` to use new flow | 2h | 2.4, 3.3, 4.6 |
| 5.2 | Update API route `/api/generate` | 1h | 5.1 |
| 5.3 | Deprecate/remove old stage prompts | 0.5h | 5.2 |
| 5.4 | Update progress tracking for 2-phase flow | 1h | 5.2 |

### Phase 6: Testing & Validation (Priority: High)

| # | Task | Estimated Hours | Dependencies |
|---|------|-----------------|--------------|
| 6.1 | Unit tests for derivation engine | 2h | 4.6 |
| 6.2 | Integration tests for full flow | 1.5h | 5.2 |
| 6.3 | Generate test scenarios (all 3 tiers) | 1h | 6.2 |
| 6.4 | Plausibility analysis (like current tests) | 2h | 6.3 |
| 6.5 | Bug fixes & iteration | 2h | 6.4 |

### Phase 7: Documentation (Priority: Medium)

| # | Task | Estimated Hours | Dependencies |
|---|------|-----------------|--------------|
| 7.1 | Update architecture.md | 1h | 5.2 |
| 7.2 | Update AGENT-BRIEFING.md | 0.5h | 5.2 |
| 7.3 | Add inline code documentation | 0.5h | All |

---

## Summary

| Phase | Hours | Description |
|-------|-------|-------------|
| Foundation | 2.75h | Types, schema, migration |
| World Building | 6h | Phase 1 generator |
| Monthly Course | 5.5h | Phase 2 generator |
| Event Derivation | 9h | Core derivation engine |
| Integration | 4.5h | Wire it all together |
| Testing | 8.5h | Tests and validation |
| Documentation | 2h | Update docs |
| **Total** | **38.25h** | |

**With James efficiency factor:** ~19h actual 😄

---

## Risks & Mitigations

| Risk | Impact | Mitigation |
|------|--------|------------|
| Claude JSON parsing issues | Medium | Strict schema, retry logic |
| Seasonality needs tuning | Low | Start with simple curves, iterate |
| Edge cases in derivation | Medium | Good unit test coverage |
| Notable events complexity | Low | Start simple, expand later |

---

## Success Criteria

After implementation, all 3 tiers should achieve:

| Metric | Target |
|--------|--------|
| Gross Margin | Within TierConfig range |
| Net Margin | Within TierConfig range |
| Payroll Share | Within TierConfig range |
| COGS Share | Within TierConfig range |
| Other Share | Within TierConfig range |
| Plausibility Score | 8+/10 |

---

## Appendix: Comparison with Current Approach

| Aspect | Current | New |
|--------|---------|-----|
| Claude calls | 3+ stages | 2 |
| Token usage | High | Lower |
| Financial accuracy | 2-6/10 | 8+/10 (guaranteed) |
| Time range | 1 year | 1-5 years |
| Product groups | No | Yes |
| Regions | No | Yes |
| Cost centers | No | Yes |
| Testability | Low | High |
| Debugging | Hard | Easy |
