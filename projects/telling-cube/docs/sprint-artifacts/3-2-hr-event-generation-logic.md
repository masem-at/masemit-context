# Story 3.2: HR Event Generation Logic

**Status:** done
**Epic:** 3 - HR Data Domain
**Points:** 8
**Created:** 2026-01-21

---

## Story

As a **BI developer or HR analyst**,
I want **tellingCube to generate realistic HR events based on company size and sector**,
so that **I can test HR analytics dashboards with accurate workforce dynamics data**.

---

## Acceptance Criteria

1. **AC1: Turnover Rates by Company Size**
   - [x] Startup: 15-25% annual turnover, rapid hiring phases
   - [x] MidCap: 10-15% annual turnover, steady growth
   - [x] LargeCap: 5-10% annual turnover, stable workforce

2. **AC2: Consistency Rules Enforced**
   - [x] `SUM(Payroll)` = `COUNT(Active) × Avg Salary`
   - [x] `Hires - Terminations` = `Headcount Change`
   - [x] Salary changes reflected in subsequent payroll events

3. **AC3: Multi-Year Integration**
   - [x] HR events span all years if durationYears > 1
   - [x] Realistic workforce evolution (hires, departures, promotions over time)
   - [x] Career progression reflected in promotions and salary changes

4. **AC4: Seasonal Patterns**
   - [x] Year-end performance reviews (Q4)
   - [x] New year hiring spike (Q1)
   - [x] Summer slowdown (Q3)
   - [x] Training concentrated in Q1-Q2

---

## Tasks / Subtasks

### Task 1: Create HR Event Deriver (AC: #1, #2, #3, #4)
- [x] 1.1 Create `lib/derivation/hr-event-deriver.ts`
- [x] 1.2 Define turnover rates by company tier
- [x] 1.3 Implement `deriveHREvents()` function

### Task 2: Implement Employee Lifecycle Events (AC: #1)
- [x] 2.1 Generate `EMPLOYEE_HIRED` events with metadata
- [x] 2.2 Generate `EMPLOYEE_TERMINATED` events with reasons
- [x] 2.3 Calculate turnover based on tier config

### Task 3: Implement Career Progression Events (AC: #3)
- [x] 3.1 Generate `EMPLOYEE_PROMOTED` events
- [x] 3.2 Generate `EMPLOYEE_TRANSFERRED` events
- [x] 3.3 Generate `SALARY_CHANGE` events

### Task 4: Implement Development Events (AC: #4)
- [x] 4.1 Generate `TRAINING_COMPLETED` events
- [x] 4.2 Generate `PERFORMANCE_REVIEW` events (Q4)

### Task 5: Integrate with Event Derivation Engine (AC: #2)
- [x] 5.1 Add HR deriver to `event-derivation-engine.ts`
- [x] 5.2 Ensure payroll consistency with HR headcount

---

## Dev Notes

### Turnover Rate Calculation

Annual turnover = (Terminations / Avg Headcount) × 100

For multi-year scenarios:
- Startup: 15-25% annual → ~1.5-2.5% monthly
- MidCap: 10-15% annual → ~0.8-1.2% monthly
- LargeCap: 5-10% annual → ~0.4-0.8% monthly

### Seasonal Patterns

| Quarter | Hiring | Reviews | Training | Promotions |
|---------|--------|---------|----------|------------|
| Q1      | High   | -       | High     | -          |
| Q2      | Medium | -       | Medium   | -          |
| Q3      | Low    | -       | Low      | -          |
| Q4      | Medium | High    | -        | High       |

### HR Event Metadata (from Story 3.1)

All events use the metadata interfaces defined in `lib/types/hr-events.ts`:
- HiredEventMetadata
- TerminatedEventMetadata
- PromotedEventMetadata
- TransferredEventMetadata
- SalaryChangeMetadata
- TrainingCompletedMetadata
- PerformanceReviewMetadata

### Integration Points

- `lib/derivation/event-derivation-engine.ts` - Add HR deriver call
- `lib/derivation/payroll-deriver.ts` - Already generates payroll, ensure consistency
- `lib/types/scenario-world.ts` - Employee interface with HR fields

---

## Dev Agent Record

### Agent Model Used

Claude Opus 4.5 (claude-opus-4-5-20251101)

### Completion Notes List

- [x] Story drafted
- [x] HR event deriver created with all 7 event types
- [x] Tier-specific turnover rates implemented
- [x] Seasonal patterns for hiring, reviews, training
- [x] Integration with event-derivation-engine complete
- [x] HR state tracking for multi-month consistency

### File List

**Files created:**
- `lib/derivation/hr-event-deriver.ts` - Full HR event generation logic

**Files modified:**
- `lib/derivation/event-derivation-engine.ts` - Added HR deriver integration
- `docs/sprint-artifacts/sprint-status.yaml` - Story status update

### Change Log

- 2026-01-21: Story 3.2 drafted
- 2026-01-21: Story 3.2 implemented - HR Event Generation Logic
