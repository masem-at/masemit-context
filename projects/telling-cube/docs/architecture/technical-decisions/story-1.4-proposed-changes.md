# Story 1.4 Proposed Changes - Batched Generation

**Date:** 2025-11-17
**Reason:** Architectural decision to implement quarterly batched generation (TDR-001)
**Approval Status:** Pending Mario (Product Stakeholder)

## Summary of Changes

Due to Claude Sonnet 4.5's practical output token limit (~5,400 tokens), we're implementing quarterly batched generation for Stage 3. This requires updating Story 1.4 acceptance criteria and documentation.

## Detailed Changes

### 1. Acceptance Criteria - AC5 (Line 19)

**Current:**
```
5. Manual test demonstrates: calling bakery generator returns 150-300 events for 1 year with realistic values (validated by console log review)
```

**Proposed:**
```
5. Manual test demonstrates: calling bakery generator returns 48-72 events for 1 year with realistic values (validated by console log review)
   - Events generated via quarterly batches (Q1-Q4) to respect Claude API token limits
   - Each quarter generates 12-18 events
   - All quarters aggregate into final events array
```

**Rationale:** Reflects batched generation approach and realistic output constraints.

---

### 2. Tasks - Stage 3 Prompt (Line 40)

**Current:**
```
  - [ ] Request 150-300 total events spread across 12 months
```

**Proposed:**
```
  - [ ] Create `getBakeryStage3QuarterlyPrompt()` function for quarterly event generation
  - [ ] Request 12-18 events per quarter (Q1-Q4)
  - [ ] Events constrained to 3-month date range per batch
```

**Rationale:** Adds quarterly prompt function and clarifies batching approach.

---

### 3. Tasks - Generator Orchestration (Lines 47-51)

**Current:**
```
  - [ ] Create async function `generateBakeryScenario()` that:
    - Calls Stage 1 prompt → parse JSON → masterData
    - Pass masterData to Stage 2 prompt → parse JSON → trends
    - Pass masterData + trends to Stage 3 prompt → parse JSON → events
    - Return events array
```

**Proposed:**
```
  - [ ] Create async function `generateBakeryScenario()` that:
    - Calls Stage 1 prompt → parse JSON → masterData
    - Pass masterData to Stage 2 prompt → parse JSON → trends
    - Loop through 4 quarters (Q1-Q4):
      - Pass masterData + trends + quarter to Stage 3 quarterly prompt → parse JSON → quarter events
      - Aggregate quarter events into events array
    - Return aggregated events array with combined token usage
```

**Rationale:** Details the quarterly batching loop logic.

---

### 4. Tasks - Manual Testing (Lines 78-82)

**Current:**
```
  - [ ] Verify 150-300 events generated
  - [ ] Verify all 7 event types present
  - [ ] Verify events spread across 12 months (timePeriod check)
  - [ ] Verify realistic values (employee salaries 2000-4000 EUR, product prices 2-15 EUR, etc.)
  - [ ] Verify JSON structure matches Prisma Event schema
```

**Proposed:**
```
  - [ ] Verify 48-72 events generated (12-18 per quarter × 4 quarters)
  - [ ] Verify all 7 event types present across all quarters
  - [ ] Verify events spread across 12 months (timePeriod check, all 4 quarters represented)
  - [ ] Verify realistic values (employee salaries 2000-4000 EUR, product prices 2-15 EUR, etc.)
  - [ ] Verify JSON structure matches Prisma Event schema
  - [ ] Verify quarterly batch logs show progress (Q1, Q2, Q3, Q4 completion messages)
```

**Rationale:** Updates validation criteria for batched approach.

---

### 5. Dev Notes - Stage 3 Architecture (Lines 114-119)

**Current:**
```
**Stage 3: Generate Granular Events (45-60s)**
- **Prompt Goal:** Create 150-300 detailed business events matching the Event model schema
- **Claude Temperature:** 0.5 (needs consistency and mathematical accuracy)
- **Input:** Master data + Trends from Stages 1 & 2
- **Output:** JSON array of Event objects
- **Why Last:** Events must align with master data and trends for multi-view consistency
```

**Proposed:**
```
**Stage 3: Generate Granular Events (90-120s, 4 batches)**
- **Prompt Goal:** Create 48-72 detailed business events matching the Event model schema
- **Claude Temperature:** 0.5 (needs consistency and mathematical accuracy)
- **Input:** Master data + Trends from Stages 1 & 2 + Quarter specification
- **Output:** JSON array of Event objects (12-18 per quarter)
- **Why Batched:** Claude Sonnet 4.5 has ~5,400 token output limit; quarterly batching ensures reliable completion
- **Why Last:** Events must align with master data and trends for multi-view consistency
```

**Rationale:** Documents batching rationale and updated timing expectations.

---

### 6. Dev Notes - Event Distribution (Lines 187-194)

**Current:**
```
**Event Distribution (for 1 year, 150-300 events total):**
- Employee lifecycle: 5-10 events (hires, promotions, terminations)
- Sales transactions: 60-100 events (daily sales aggregated weekly/monthly)
- Cash movements: 40-60 events (inflows from sales, outflows for expenses)
- Operational work: 30-50 events (employee hours worked, shifts)
- Procurement: 20-40 events (ingredient purchases, packaging orders)
- Asset lifecycle: 5-10 events (equipment purchases, maintenance)
- Planning/Budgeting: 10-20 events (monthly budgets, forecasts)
```

**Proposed:**
```
**Event Distribution (for 1 year, 48-72 events total across 4 quarters):**
- Employee lifecycle: 3-4 events (hires, promotions, terminations)
- Sales transactions: 20-28 events (monthly sales summaries by product)
- Cash movements: 12-16 events (inflows from sales, outflows for expenses)
- Operational work: 10-14 events (employee hours worked, monthly summaries)
- Procurement: 8-12 events (ingredient purchases, packaging orders)
- Asset lifecycle: 2-4 events (equipment purchases, maintenance)
- Planning/Budgeting: 5-8 events (monthly budgets, quarterly forecasts)

**Per Quarter Target:** 12-18 events per quarter (Q1-Q4)
```

**Rationale:** Reflects realistic event counts with quarterly batching.

---

### 7. Dev Notes - Cost Estimates (Lines 278-287)

**Current:**
```
**Per-Scenario Generation (Claude Sonnet 4.5 - Actual Model):**
- Input costs (prompts): ~2000 tokens × $3.00/1M = $0.006
- Stage 1 (Master Data): ~500 output tokens × $15.00/1M = $0.0075
- Stage 2 (Trends): ~1000 output tokens × $15.00/1M = $0.015
- Stage 3 (Events): ~3000 output tokens × $15.00/1M = $0.045
- **Total per scenario:** ~$0.0735
```

**Proposed:**
```
**Per-Scenario Generation (Claude Sonnet 4.5 - Actual Model, with Batching):**
- Input costs (prompts): ~6000 tokens × $3.00/1M = $0.018
- Stage 1 (Master Data): ~500 output tokens × $15.00/1M = $0.0075
- Stage 2 (Trends): ~1000 output tokens × $15.00/1M = $0.015
- Stage 3 (Events, 4 batches): ~3000 output tokens × 4 × $15.00/1M = $0.18
- **Total per scenario:** ~$0.10-0.15

**Note:** Batched generation adds ~40% to costs but ensures 100% reliability vs frequent truncation failures.
```

**Rationale:** Updates cost estimates to reflect quarterly batching overhead.

---

### 8. Dev Notes - Testing Checklist (Line 302)

**Current:**
```
4. Verify Stage 3 returns 150-300 events with:
```

**Proposed:**
```
4. Verify Stage 3 quarterly batches return 48-72 total events with:
   - Q1 (Jan-Mar): 12-18 events
   - Q2 (Apr-Jun): 12-18 events
   - Q3 (Jul-Sep): 12-18 events
   - Q4 (Oct-Dec): 12-18 events
   And:
```

**Rationale:** Adds quarterly batch validation to test checklist.

---

### 9. Add New Dev Notes Section (After line 313)

**Add after "Testing" section:**

```markdown
### Claude API Token Limit Constraint (Discovered during Implementation)

**Issue:** Claude Sonnet 4.5 has a practical output token limit of ~5,400 tokens when generating large structured JSON arrays, even when max_tokens is set higher (e.g., 8,192).

**Evidence:**
- Multiple test runs with event targets of 150-300, 80-120, and 50-70 all truncated at ~21,400 characters
- Consistent truncation at line ~890 of generated JSON (~100 events)
- Error: "Unterminated string in JSON" or "Unexpected end of JSON input"

**Root Cause:** Internal model constraints on sustained structured output generation.

**Solution:** Quarterly batched generation (see TDR-001)

**Reference:** `docs/architecture/technical-decisions/tdr-001-batched-event-generation.md`
```

**Rationale:** Documents the constraint discovery for future reference.

---

### 10. Change Log Entry (After line 320)

**Add new row to Change Log table:**

```markdown
| 2025-11-17 | 1.2 | Updated AC to reflect quarterly batched generation (48-72 events). Added TDR-001 reference. Updated cost estimates. | Winston (Architect) / Mario (Stakeholder) |
```

**Rationale:** Tracks this architectural change in story history.

---

## Summary of Impact

**Acceptance Criteria:** 1 modified (AC5)
**Tasks:** 3 modified (Stage 3 prompt, orchestration, testing)
**Dev Notes:** 5 sections updated (Stage 3 architecture, event distribution, costs, testing, new constraint section)
**Change Log:** 1 entry added

**Total Lines Changed:** ~15-20 lines modified/added

## Approval

- [ ] Mario (Product Stakeholder) - Approved
- [ ] Winston (Architect) - Recommends approval
- [ ] Sarah (PO) - To be notified if formal approval needed

## Next Steps

Once approved:
1. Winston updates Story 1.4 with these changes
2. Winston hands off to James (Dev) with:
   - Updated story file
   - TDR-001 reference
   - Implementation code samples (already provided)
3. James implements quarterly batching
4. James runs validation test
5. James updates Dev Agent Record with results
