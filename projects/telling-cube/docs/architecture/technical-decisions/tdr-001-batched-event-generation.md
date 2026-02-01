# TDR-001: Batched Event Generation for Claude API

**Status:** Accepted
**Date:** 2025-11-17
**Decision Maker:** Winston (Architect) / Mario (Product Stakeholder)
**Context:** Story 1.4 - Bakery Scenario Prompt Engineering

## Problem Statement

During implementation of the 3-stage AI prompt pipeline for bakery scenario generation, we discovered that Claude Sonnet 4.5 has a practical output token limit of ~5,400 tokens when generating structured JSON arrays, significantly less than the documented max_tokens limit of 8,192.

**Observed Behavior:**
- Requested: 150-300 events → Generated: ~100 events (truncated at ~21,400 chars)
- Requested: 80-120 events → Generated: ~100 events (truncated at ~21,400 chars)
- Requested: 50-70 events → Generated: ~100 events (truncated at ~21,400 chars)

**Root Cause:**
Claude Sonnet 4.5 appears to stop generating after approximately 5,400 output tokens for large JSON arrays, regardless of max_tokens setting, likely due to internal attention/context constraints.

## Decision

Implement **quarterly batched generation** for Stage 3 event generation:

- Split 12-month period into 4 quarters (Q1-Q4)
- Generate 12-18 events per quarter via separate API calls
- Aggregate results into final events array
- Target: 48-72 total events (sufficient for PoC validation)

## Rationale

### Alternatives Considered

**Option 1: Accept Reduced Scope**
- ❌ Less data for validation
- ✅ Simplest implementation
- ❌ Doesn't scale to production needs

**Option 2: Batched Generation (Chosen)**
- ✅ Respects API limits while maintaining quality
- ✅ Scales to larger datasets
- ✅ Adds minimal complexity (~4 API calls)
- ✅ Natural business boundaries (quarters)
- ❌ Slightly higher API costs (~$0.10-0.15 vs $0.07)

**Option 3: Month-by-Month Generation**
- ✅ Maximum granularity
- ❌ 12 API calls (3x more than quarterly)
- ❌ Higher latency and cost
- ❌ Over-engineering for PoC

### Why Quarterly Batches?

1. **Token Budget:** 12-18 events per quarter = ~2,500-3,500 tokens (well within 5,400 limit)
2. **Business Logic:** Quarters are natural business reporting boundaries
3. **Scalability:** Easy to extend to multi-year scenarios
4. **Balance:** Optimal trade-off between API calls and reliability

## Implementation

### Architecture Changes

**Modified Pipeline:**
```
Stage 1 (Master Data) → Stage 2 (Trends) → Stage 3 (Quarterly Batches)
                                            ├─ Q1 (Jan-Mar): 12-18 events
                                            ├─ Q2 (Apr-Jun): 12-18 events
                                            ├─ Q3 (Jul-Sep): 12-18 events
                                            └─ Q4 (Oct-Dec): 12-18 events
                                            → Aggregate: 48-72 events total
```

### Code Changes

1. **New Function:** `getBakeryStage3QuarterlyPrompt(masterData, trends, quarter, quarterMonths)`
   - Location: `lib/prompts/bakery-scenario.ts`
   - Generates quarter-specific prompts with 3-month date constraints

2. **Modified Function:** `generateBakeryScenario()`
   - Location: `lib/generators/bakery-generator.ts`
   - Loops through 4 quarters
   - Aggregates events and token usage
   - Logs per-quarter progress

### Expected Outcomes

- **Total Events:** 48-72 (vs original 150-300)
- **API Calls:** 6 total (1 + 1 + 4)
- **Token Usage:** ~15,000-20,000 tokens
- **Cost per Scenario:** ~$0.10-0.15
- **Success Rate:** Near 100% (no truncation)
- **Latency:** ~90-120 seconds (vs ~60-90 for single-batch)

## Consequences

### Positive

- ✅ **Reliability:** Eliminates JSON truncation errors
- ✅ **Scalability:** Pattern extends to production use cases
- ✅ **Quality:** Better event distribution across time periods
- ✅ **Maintainability:** Clean separation of batch logic

### Negative

- ⚠️ **Higher Cost:** ~40% increase in API costs ($0.07 → $0.10)
- ⚠️ **Longer Runtime:** ~30-40% increase in generation time
- ⚠️ **Complexity:** Additional loop and aggregation logic

### Neutral

- Event count reduced from 150-300 → 48-72 (still sufficient for PoC validation)
- Story 1.4 AC requires update to reflect new target

## Monitoring & Validation

**Success Criteria:**
- ✅ All 4 quarters generate successfully without truncation
- ✅ Events distributed across all 12 months
- ✅ All 7 event types represented
- ✅ Financial values realistic and mathematically consistent
- ✅ Total events: 48-72

**Failure Modes:**
- Individual quarter truncation (requires further event reduction)
- Inconsistent event distribution across quarters
- Token usage exceeds budget expectations

## Future Considerations

- **Production Scale:** For multi-year scenarios, consider monthly batching
- **Parallel Generation:** Quarters could be generated in parallel for speed
- **Caching:** Consider caching master data + trends for repeated generation
- **Alternative Models:** Evaluate Claude Opus or Haiku if output limits differ

## References

- Story 1.4: Bakery Scenario Prompt Engineering
- Claude Structured Outputs Documentation: https://docs.claude.com/en/docs/build-with-claude/structured-outputs
- Architecture Document: `docs/architecture.md`

## Approval

- **Architect:** Winston ✅
- **Product Stakeholder:** Mario ✅ (Approved 2025-11-17)
- **Implementation:** James (Dev Agent) - In Progress
