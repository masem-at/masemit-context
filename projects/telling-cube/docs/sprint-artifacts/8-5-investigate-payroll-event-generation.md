# Story 8.5: Investigate Payroll Event Generation

Status: ready-for-dev

## Story

As a developer,
I want to understand why Finance payroll ≠ HR payroll,
so that I can fix the reconciliation bug.

## Acceptance Criteria

1. - [x] AC1: Document where `operational_work` events are generated in two-phase-generator
2. - [x] AC2: Document where employee salaries are set in world-building
3. - [x] AC3: Identify the disconnect between these two sources
4. - [x] AC4: Write investigation findings to this story

## Tasks / Subtasks

- [x] Task 1: Trace `operational_work` event generation (AC: 1)
  - [x] 1.1: Find payroll-deriver.ts
  - [x] 1.2: Document derivePayrollEvents function
  - [x] 1.3: Trace targetPayrollAmount source
- [x] Task 2: Trace employee salary assignment (AC: 2)
  - [x] 2.1: Find world-building-generator.ts
  - [x] 2.2: Document employee salary population
  - [x] 2.3: Trace hr-queries.ts calculation
- [x] Task 3: Document root cause (AC: 3, 4)
  - [x] 3.1: Write investigation findings
  - [x] 3.2: Propose fix approach

## Dev Notes

### Investigation Findings

#### The Problem
- **Finance payroll**: €12,266,100 (from `operational_work` events)
- **HR payroll**: €3,828,000 (from employee salary × 12)
- **Mismatch factor**: 3.2x

#### Root Cause: Dual Calculation Paths

The system has TWO independent payroll calculation methods that don't reconcile:

**Path A: Finance (Event-Based)**
```
Revenue Target (€37.16M)
    ↓
calculateMonthlyTargets() [event-derivation-engine.ts:77-103]
    ↓ payrollShare = 33% (from SECTOR_CONFIGS.healthcare.payrollShare)
    ↓
targets.payroll = €12.27M
    ↓
derivePayrollEvents() [payroll-deriver.ts:241-317]
    ↓ scaleFactor = targetPayroll / actualPayroll = 3.2
    ↓
operational_work events with SCALED amounts
    ↓
Finance queries sum operational_work = €12.27M ✓
```

**Path B: HR (World-Based)**
```
World Building (Claude generates employees)
    ↓
Employee records with salary field (UNSCALED)
    ↓ avgSalary = €4,908/month
    ↓
hr-queries.ts:173-175
    ↓ totalPayroll = Σ(employee.salary) × 12
    ↓
HR summary shows €3.83M ✗
```

#### Key Code Locations

| File | Lines | Function | Purpose |
|------|-------|----------|---------|
| `lib/derivation/event-derivation-engine.ts` | 77-103 | `calculateMonthlyTargets()` | Derives payroll target from revenue × payrollShare |
| `lib/derivation/event-derivation-engine.ts` | 118 | `targets = calculateMonthlyTargets()` | Passes target to payroll deriver |
| `lib/derivation/payroll-deriver.ts` | 241-317 | `derivePayrollEvents()` | Creates scaled operational_work events |
| `lib/derivation/payroll-deriver.ts` | 264-266 | `scaleFactor` calculation | Scales employee salaries to match financial target |
| `lib/queries/hr-queries.ts` | 173-175 | `getHRSummary()` | Sums UNSCALED employee salaries |
| `lib/queries/finance-queries.ts` | 93-97 | `getPLSummary()` | Sums operational_work events (SCALED) |

#### The Scaling Disconnect

In `payroll-deriver.ts:264-266`:
```typescript
const scaleFactor = targetPayrollAmount > 0 && actualPayroll > 0
  ? targetPayrollAmount / actualPayroll
  : 1
```

For healthcare:
- `targetPayrollAmount` = €12.27M (from financial constraints)
- `actualPayroll` = €3.83M (from employee salaries)
- `scaleFactor` = 3.2

Events are created with scaled amounts:
```typescript
const scaledSalary = Math.round(salary * scaleFactor * 100) / 100
events.push({
  eventType: 'operational_work',
  amountEur: scaledSalary,  // SCALED to 3.2x
})
```

But HR queries directly use `employee.salary` from world data (UNSCALED).

### Proposed Fix Approach

**Option A: Scale Employee Salaries in World Data**
- After world-building, apply scaleFactor to employee.salary field
- HR queries would then match Finance naturally
- Risk: May produce unrealistic individual salaries

**Option B: HR Queries Use Event Data**
- Modify `getHRSummary()` to sum `payroll` or `operational_work` events
- HR becomes event-sourced like Finance
- Maintains single source of truth (events)
- Recommended approach

**Option C: Validate and Adjust at Generation Time**
- Add post-generation validation
- If mismatch > 1%, adjust world employee salaries to reconcile
- More complex but ensures both views are correct

### Recommended Fix (Story 8.6)

**Implement Option B**: Make HR payroll event-sourced.

Modify `lib/queries/hr-queries.ts`:
```typescript
// Instead of:
const totalPayroll = totalSalary * 12

// Use:
const payrollEvents = events.filter(e => e.eventType === 'payroll')
const totalPayroll = payrollEvents.reduce((sum, e) => sum + e.amountEur, 0) * 12
```

This ensures Finance and HR both derive from the same scaled event data.

### Additional Bugs Found

1. **Duplicate month bug**: `headcountByMonth` has 2024-03 appearing twice
   - Likely in `getHeadcountByMonth()` in hr-queries.ts
   - Needs investigation in Story 8.7

2. **Headcount drift**: Summary says 65, December shows 79
   - Employee lifecycle events add to headcount without updating world data
   - HR summary uses world employees, not event-derived headcount

### Project Structure Notes

- All derivation logic: `lib/derivation/`
- All queries: `lib/queries/`
- Financial constraints: `lib/config/scenario-matrix.ts`
- Generation orchestration: `lib/generators/two-phase-generator.ts`

### References

- [Source: lib/derivation/payroll-deriver.ts:241-317] - derivePayrollEvents with scaleFactor
- [Source: lib/derivation/event-derivation-engine.ts:77-103] - calculateMonthlyTargets
- [Source: lib/queries/hr-queries.ts:173-175] - HR payroll calculation
- [Source: lib/queries/finance-queries.ts:93-97] - Finance payroll calculation
- [Source: docs/epics/epic-08-healthcare-sector.md] - Epic context

## Dev Agent Record

### Context Reference

Investigation completed via code tracing. No story context XML generated.

### Agent Model Used

Claude Opus 4.5 (Bob - Scrum Master)

### Completion Notes List

- Root cause identified: Dual calculation paths with scaling disconnect
- Fix approach documented: Make HR event-sourced (Option B)
- Additional bugs catalogued for Stories 8.7+

### File List

Files analyzed:
- `lib/derivation/payroll-deriver.ts`
- `lib/derivation/event-derivation-engine.ts`
- `lib/derivation/hr-event-deriver.ts`
- `lib/queries/hr-queries.ts`
- `lib/queries/finance-queries.ts`
- `lib/config/scenario-matrix.ts`
