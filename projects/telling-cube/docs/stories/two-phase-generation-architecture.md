# Story: Two-Phase Generation Architecture

**Status:** Ready for Development
**Priority:** High (PoC Critical - Fixes data quality issues)
**Estimated Effort:** 38h (or ~19h James-time 😄)
**Spec:** `docs/architecture/two-phase-generation-spec.md`

---

## Overview

Replace the current multi-stage Claude generation (which produces unrealistic financial ratios) with a two-phase architecture:

1. **Phase 1 (World Building):** Claude creates company, employees, customers, vendors, products, regions, cost centers
2. **Phase 2 (Monthly Course):** Claude provides monthly guidance (revenue targets, headcount, narrative, notable events)
3. **Phase 3 (Event Derivation):** Code derives events with guaranteed correct financial ratios

**Why:** After 3 test rounds, Claude consistently ignores financial constraints (scores: 4/10, 3/10, 2/10). This architecture lets Claude do what it's good at (narrative) while code handles math.

---

## User Story

As a user generating business scenarios,
I want the generated data to have realistic financial ratios,
So that the demo data is plausible for training and demonstrations.

---

## Acceptance Criteria

- [ ] All 3 tiers (Startup, Midcap, Largecap) achieve plausibility score 8+/10
- [ ] Gross margin within TierConfig range
- [ ] Net margin within TierConfig range
- [ ] Payroll share within TierConfig range
- [ ] COGS share within TierConfig range
- [ ] Other share within TierConfig range
- [ ] Events tagged with product group, region, cost center
- [ ] Generation completes with 2 Claude calls (down from 3+ stages)

---

## New Capabilities Unlocked

| Feature | Description |
|---------|-------------|
| Product Groups | Revenue breakdown by product category |
| Regions | Geographic performance analysis |
| Cost Centers | Departmental cost allocation |
| Longer Time Ranges | 1-5 years instead of 1 year |
| Lower Token Costs | 2 Claude calls vs 3+ stages |

---

## Tasks

### Phase 1: Foundation (2.75h)

| # | Task | Hours |
|---|------|-------|
| 1.1 | Create `lib/types/scenario-world.ts` with interfaces | 1h |
| 1.2 | Create `lib/types/monthly-course.ts` with interfaces | 0.5h |
| 1.3 | Add Prisma schema for `scenario_world` table | 0.5h |
| 1.4 | Add dimension columns to Event model (product_group, region, cost_center) | 0.5h |
| 1.5 | Run Prisma migration | 0.25h |

### Phase 2: World Building Generator (6h)

| # | Task | Hours | Depends On |
|---|------|-------|------------|
| 2.1 | Create `lib/prompts/world-building-prompt.ts` | 1.5h | 1.1 |
| 2.2 | Create `lib/generators/world-building-generator.ts` | 2h | 2.1 |
| 2.3 | Add JSON parsing & validation for World output | 1h | 2.2 |
| 2.4 | Store World in database | 0.5h | 1.3, 2.3 |
| 2.5 | Test with all 3 tiers | 1h | 2.4 |

### Phase 3: Monthly Course Generator (5.5h)

| # | Task | Hours | Depends On |
|---|------|-------|------------|
| 3.1 | Create `lib/prompts/monthly-course-prompt.ts` | 1.5h | 1.2 |
| 3.2 | Create `lib/generators/monthly-course-generator.ts` | 2h | 3.1 |
| 3.3 | Add JSON parsing & validation for Course output | 1h | 3.2 |
| 3.4 | Test with all 3 tiers | 1h | 3.3 |

### Phase 4: Event Derivation Engine (9h)

| # | Task | Hours | Depends On |
|---|------|-------|------------|
| 4.1 | Create `lib/derivation/sales-deriver.ts` | 2h | 1.1, 1.2 |
| 4.2 | Create `lib/derivation/payroll-deriver.ts` | 1h | 1.1, 1.2 |
| 4.3 | Create `lib/derivation/procurement-deriver.ts` | 1.5h | 1.1, 1.2 |
| 4.4 | Create `lib/derivation/other-deriver.ts` | 1h | 1.1, 1.2 |
| 4.5 | Create `lib/derivation/notable-event-deriver.ts` | 1h | 1.1, 1.2 |
| 4.6 | Create `lib/derivation/event-derivation-engine.ts` (orchestrator) | 1.5h | 4.1-4.5 |
| 4.7 | Add seasonality curves `lib/config/seasonality.ts` | 1h | - |
| 4.8 | Add financial ratio validation | 1h | 4.6 |

### Phase 5: Integration (4.5h)

| # | Task | Hours | Depends On |
|---|------|-------|------------|
| 5.1 | Update `lib/generators/scenario-generator.ts` to use new flow | 2h | 2.4, 3.3, 4.6 |
| 5.2 | Update API route `/api/generate` | 1h | 5.1 |
| 5.3 | Deprecate/remove old stage prompts | 0.5h | 5.2 |
| 5.4 | Update progress tracking for 2-phase flow | 1h | 5.2 |

### Phase 6: Testing & Validation (8.5h)

| # | Task | Hours | Depends On |
|---|------|-------|------------|
| 6.1 | Unit tests for derivation engine | 2h | 4.6 |
| 6.2 | Integration tests for full flow | 1.5h | 5.2 |
| 6.3 | Generate test scenarios (all 3 tiers) | 1h | 6.2 |
| 6.4 | Plausibility analysis (like current tests) | 2h | 6.3 |
| 6.5 | Bug fixes & iteration | 2h | 6.4 |

### Phase 7: Documentation (2h)

| # | Task | Hours | Depends On |
|---|------|-------|------------|
| 7.1 | Update architecture.md | 1h | 5.2 |
| 7.2 | Update AGENT-BRIEFING.md | 0.5h | 5.2 |
| 7.3 | Add inline code documentation | 0.5h | All |

---

## Technical Details

See full specification: `docs/architecture/two-phase-generation-spec.md`

### Key Interfaces

**ScenarioWorld (Phase 1 output):**
- company (name, story, headquarters)
- employees[] (name, role, department, salaryBand, hireDate)
- customers[] (name, type, avgOrderSize, region)
- vendors[] (name, supplies, category, isMainSupplier)
- products[] (name, category, priceRange, marginProfile)
- productGroups[] (name, revenueShare)
- regions[] (name, type, revenueShare)
- costCenters[] (name, type, costShare)

**MonthlyCourse (Phase 2 output):**
- month
- revenue (target, yoyGrowth)
- headcount (total, changes)
- mood (growing/stable/struggling)
- narrative
- notableEvents[]

### Database Changes

```sql
-- New table
CREATE TABLE scenario_world (
  id UUID PRIMARY KEY,
  scenario_id UUID REFERENCES scenario(id),
  world_data JSONB NOT NULL,
  created_at TIMESTAMP
);

-- New columns on event
ALTER TABLE event ADD COLUMN product_group VARCHAR(100);
ALTER TABLE event ADD COLUMN region VARCHAR(100);
ALTER TABLE event ADD COLUMN cost_center VARCHAR(100);
ALTER TABLE event ADD COLUMN is_notable_event BOOLEAN DEFAULT FALSE;
```

---

## Risks & Mitigations

| Risk | Impact | Mitigation |
|------|--------|------------|
| Claude JSON parsing issues | Medium | Strict schema, retry logic |
| Seasonality needs tuning | Low | Start simple, iterate |
| Edge cases in derivation | Medium | Good unit test coverage |

---

## Definition of Done

- [ ] All tasks completed
- [ ] All 3 tiers pass plausibility analysis (8+/10)
- [ ] Unit tests passing
- [ ] Integration tests passing
- [ ] No regressions in existing functionality
- [ ] Documentation updated
- [ ] Code reviewed

---

## Notes

- ROF Session decision: Unanimous support from Mary, James, Winston, Sally, Mario
- This replaces the current 3-stage prompt approach
- Old prompts in `lib/prompts/tier-scenario.ts` will be deprecated
