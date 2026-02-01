# Story 2.3: Multi-Year Event Generation Logic

**Status:** done
**Epic:** 2 - Multi-Year Scenarios (3-5 Years)
**Points:** 8
**Created:** 2026-01-21

---

## Story

As a **tellingCube system**,
I want **the generation pipeline to create consistent events across multiple years with realistic YoY trends**,
so that **users can analyze long-term business trends with mathematically consistent multi-year data**.

---

## Acceptance Criteria

### AC1: Events Scaled by Duration
**Given** a scenario with `durationYears = 3`
**When** the generation completes
**Then** approximately `baseEvents × 3` events are created
**And** events span 36 months (3 years)

### AC2: Year-over-Year Growth Patterns
**Given** a Startup scenario with 5 years
**When** revenue is calculated per year
**Then** growth follows pattern: 50% → 30% → 20% → 10% → 5%

**Given** a MidCap scenario with 5 years
**When** revenue is calculated per year
**Then** growth follows pattern: 8% → 10% → 15% → 5% → 8%

**Given** a LargeCap scenario with 5 years
**When** revenue is calculated per year
**Then** growth follows pattern: 3% → 4% → -2% → 5% → 3%

### AC3: Seasonal Patterns Repeat Annually
**Given** a 3-year scenario with seasonal revenue patterns
**When** monthly revenue is analyzed
**Then** Q4 is consistently highest across all years
**And** Q1 shows post-holiday dip in all years
**And** seasonal amplitude is consistent year-over-year

### AC4: Strategic Inflection Points
**Given** a 5-year scenario
**When** notable events are generated
**Then** Year 2 includes market expansion or product launch event
**And** Year 3 includes acquisition or major client event
**And** Year 4 includes restructuring or optimization event

### AC5: Mathematical Consistency Across Years
**Given** a multi-year scenario
**When** financial totals are calculated
**Then** Profit = Revenue - COGS - OpEx for each year
**And** Assets = Liabilities + Equity at year end
**And** YoY variance calculations are correct

---

## Tasks / Subtasks

- [ ] **Task 1: Update Worker to Handle durationYears** (AC: 1)
  - [ ] Read `durationYears` from QStash payload (already passed from route.ts)
  - [ ] Fetch `durationYears` from scenario record if not in payload
  - [ ] Calculate `timeRange` based on `durationYears` (e.g., 2024-01 to 2026-12 for 3Y)
  - [ ] Pass extended `timeRange` to `generateTwoPhaseScenario`

- [ ] **Task 2: Update Two-Phase Generator for Multi-Year** (AC: 1, 5)
  - [ ] Parse timeRange to calculate number of years
  - [ ] Update progress tracking for multi-year generation
  - [ ] Log duration and event count scaling

- [ ] **Task 3: Update Monthly Course Generator for YoY Trends** (AC: 2, 3)
  - [ ] Add `durationYears` parameter to generator
  - [ ] Define YoY growth patterns per tier in configuration
  - [ ] Apply growth multiplier per year
  - [ ] Ensure seasonal patterns repeat with consistent amplitude
  - [ ] Update Claude prompt to generate multi-year monthly course

- [ ] **Task 4: Add Strategic Inflection Points** (AC: 4)
  - [ ] Define inflection point templates per year
  - [ ] Add logic to inject notable events at strategic years
  - [ ] Update world building to reference multi-year strategy

- [ ] **Task 5: Update Event Derivation for Multi-Year** (AC: 1, 5)
  - [ ] Ensure derivation engine handles 36/60 month periods
  - [ ] Apply YoY growth factors to monthly targets
  - [ ] Validate mathematical consistency across years

- [ ] **Task 6: Testing and Validation** (AC: 5)
  - [ ] Generate 3-year scenario and verify event counts
  - [ ] Generate 5-year scenario and verify event counts
  - [ ] Validate YoY growth rates match configured patterns
  - [ ] Run consistency validator on multi-year output

---

## Dev Notes

### Current Architecture

The generation pipeline uses a two-phase approach:
1. **Phase 1: World Building** - Creates company, employees, customers via Claude
2. **Phase 2: Monthly Course** - Creates revenue targets, narrative per month via Claude
3. **Phase 3: Event Derivation** - Deterministic code creates financial events

### Key Files to Modify

| File | Change |
|------|--------|
| `app/api/generate/worker/route.ts` | Read durationYears, calculate extended timeRange |
| `lib/generators/two-phase-generator.ts` | Handle multi-year timeRange |
| `lib/generators/monthly-course-generator.ts` | Add YoY growth logic, multi-year prompts |
| `lib/derivation/event-derivation-engine.ts` | Scale for 36/60 months |

### Current timeRange Handling

Worker passes hardcoded timeRange (`worker/route.ts:130`):
```typescript
timeRange: { start: '2024-01', end: '2024-12' },
```

Two-phase generator accepts but defaults (`two-phase-generator.ts:74`):
```typescript
timeRange = { start: '2024-01', end: '2024-12' },
```

### Required Changes

**1. Worker - Calculate timeRange from durationYears:**
```typescript
// Read durationYears from payload or scenario
const scenario = await prisma.scenario.findUnique({
  where: { id: scenarioId },
  select: { durationYears: true }
})
const durationYears = body.durationYears || scenario?.durationYears || 1

// Calculate timeRange
const currentYear = new Date().getFullYear()
const startYear = currentYear
const endYear = currentYear + durationYears - 1
const timeRange = {
  start: `${startYear}-01`,
  end: `${endYear}-12`
}
```

**2. YoY Growth Configuration:**
```typescript
const YOY_GROWTH_PATTERNS: Record<CompanyTier, number[]> = {
  startup: [0.50, 0.30, 0.20, 0.10, 0.05],    // 50% → 5%
  midcap: [0.08, 0.10, 0.15, 0.05, 0.08],     // 8% → 15% → 8%
  largecap: [0.03, 0.04, -0.02, 0.05, 0.03],  // Mature with dip
}
```

**3. Strategic Inflection Points:**
```typescript
const INFLECTION_EVENTS: Record<number, string> = {
  2: 'Market expansion / New product launch',
  3: 'Acquisition / Major client win',
  4: 'Restructuring / Cost optimization',
  5: 'Stabilization / Dividend policy',
}
```

### Performance Considerations

From `docs/future/spec-multi-year-scenarios.md`:
| Duration | Events | Est. Generation Time |
|----------|--------|---------------------|
| 1 Year | ~3,000 | 60-90 seconds |
| 3 Years | ~9,000 | 3-4 minutes |
| 5 Years | ~15,000 | 5-7 minutes |

The worker has `maxDuration = 800` (13.3 minutes), which provides sufficient headroom.

### Dependencies

- Story 2.1 (done): Schema has `durationYears` field
- Story 2.2 (done): API accepts `durationYears`, passes to QStash worker

---

## References

- [Source: docs/epics/epic-02-multi-year-scenarios.md#Story 2.3]
- [Source: docs/future/spec-multi-year-scenarios.md#Phase 2: Business Logic]
- [Source: app/api/generate/worker/route.ts:130] - Current hardcoded timeRange
- [Source: lib/generators/two-phase-generator.ts:74] - Default timeRange
- [Source: lib/generators/monthly-course-generator.ts] - Monthly course generation

---

## Dev Agent Record

### Context Reference

Story context created via BMM create-story workflow 2026-01-21

### Agent Model Used

Claude Opus 4.5 (claude-opus-4-5-20251101)

### Debug Log References

N/A - No errors encountered

### Completion Notes List

1. **Worker Route Updated** (`app/api/generate/worker/route.ts`)
   - Added `durationYears` to WorkerRequest interface
   - Fetches durationYears from QStash payload or scenario record
   - Calculates dynamic timeRange based on durationYears (e.g., 2026-01 to 2028-12 for 3Y)
   - Passes timeRange and durationYears to generateTwoPhaseScenario

2. **Two-Phase Generator Updated** (`lib/generators/two-phase-generator.ts`)
   - Added `durationYears` to GenerationOptions interface
   - Extracts and logs totalMonths (12/36/60)
   - Passes durationYears to generateMonthlyCourse

3. **Monthly Course Generator Updated** (`lib/generators/monthly-course-generator.ts`)
   - Added `durationYears` parameter (default: 1)
   - Increased maxTokens for multi-year: 8192 → 16384 → 24576
   - Passes durationYears to prompt generator

4. **Monthly Course Prompt Updated** (`lib/prompts/monthly-course-prompt.ts`)
   - Added YOY_GROWTH_BY_TIER configuration: startup (50%→5%), midcap (8%→8%), largecap (3%→-2%→3%)
   - Added INFLECTION_EVENTS for strategic events by year
   - Added multi-year guidance section to prompt with YoY growth trajectories
   - Instructs Claude on seasonal pattern repetition and narrative arc

5. **Event Derivation Engine** - No changes needed
   - Already iterates over `course.months.length` dynamically
   - Handles 12, 36, or 60 months automatically

### File List

| File | Change |
|------|--------|
| `app/api/generate/worker/route.ts` | Added durationYears handling, dynamic timeRange calculation |
| `lib/generators/two-phase-generator.ts` | Added durationYears to options, pass to monthly course generator |
| `lib/generators/monthly-course-generator.ts` | Added durationYears param, dynamic maxTokens |
| `lib/prompts/monthly-course-prompt.ts` | Added YoY growth patterns, inflection events, multi-year prompt guidance |
