# Epic 8: Healthcare Sector

**Status:** In Progress (Stories 8.1-8.4 complete, blocked by reconciliation bug)
**Priority:** Medium
**Effort:** 1-2 days (initial) + 1 day (fix)
**Branch:** develop (validation only)

---

## Overview

Add Healthcare as the 4th industry sector for paid users, and replace one curated free example with a Healthcare example for visibility.

**Customer Request:** Paid user asked if we support healthcare sector.

---

## Business Value

- Expands addressable market to healthcare BI consultants, medical device vendors, health tech trainers
- Demonstrates sector extensibility (future: Technology, Energy, etc.)
- Quick win with existing architecture

---

## Scope

### In Scope
- Add Healthcare sector configuration
- Healthcare-specific HR roles (clinical)
- Replace MidCap Financials curated example with MidCap Healthcare
- Sector teaser on landing page
- Deploy to develop branch only (validation)

### Out of Scope
- Additional company sizes for Healthcare (only MidCap for now)
- Sector-specific compliance features
- Production deployment (pending Mario's validation)

---

## User Stories

### Story 8.1: Healthcare Sector Configuration

**As a** paid user (Lifetime+/Pro/Team)
**I want to** generate Healthcare sector scenarios
**So that** I can create realistic medical/diagnostics business data

**Acceptance Criteria:**
- [ ] AC1: `healthcare` added to `Sector` type in `lib/config/scenario-matrix.ts`
- [ ] AC2: `SECTOR_CONFIGS['healthcare']` defined with financial constraints per research doc
- [ ] AC3: Company names added for `midcap-healthcare` (MedTech Diagnostics AG + alternatives)
- [ ] AC4: Healthcare sector selectable in generation dialog for entitled users
- [ ] AC5: Tier check: Healthcare requires Lifetime+, Pro, or Team (same gate as HR)

**Tasks:**
- [ ] 8.1.1: Add `'healthcare'` to Sector union type
- [ ] 8.1.2: Add SECTOR_CONFIGS entry with financial constraints
- [ ] 8.1.3: Add COMPANY_NAMES entries for midcap-healthcare
- [ ] 8.1.4: Add Healthcare to SECTORS export array (UI dropdown)
- [ ] 8.1.5: Update entitlements check (if separate from HR gate)

**Reference:** `docs/research/healthcare-sector-research.md`

---

### Story 8.2: Healthcare HR Roles

**As a** user generating Healthcare scenarios
**I want to** see clinical-specific job roles in HR data
**So that** the workforce data looks realistic for a medical company

**Acceptance Criteria:**
- [ ] AC1: Healthcare scenarios use clinical role titles (Lab Technician, Radiology Tech, etc.)
- [ ] AC2: Department codes include LAB, RAD, CLIN, QA, ADMIN
- [ ] AC3: Salary ranges reflect healthcare market (per research doc)
- [ ] AC4: Executive roles include CMO, Medical Director

**Tasks:**
- [ ] 8.2.1: Add healthcare-specific role mappings in world-building prompt or config
- [ ] 8.2.2: Add healthcare department codes
- [ ] 8.2.3: Verify salary ranges applied correctly for healthcare

**Reference:** `docs/research/healthcare-sector-research.md` Section 4

---

### Story 8.3: Curated Healthcare Example

**As a** visitor (free or paid)
**I want to** see a Healthcare example on the landing page
**So that** I know tellingCube supports medical/healthcare data

**Acceptance Criteria:**
- [ ] AC1: MidCap Financials example (Regional Credit Bank) replaced with MidCap Healthcare
- [ ] AC2: New example uses "MedTech Diagnostics AG" company name
- [ ] AC3: Example JSON generated and stored in `public/examples/midcap-healthcare/`
- [ ] AC4: ExampleCards.tsx updated with Healthcare tile (icon: Heart or Stethoscope)
- [ ] AC5: Example viewable by all users (free tier included)

**Tasks:**
- [ ] 8.3.1: Generate MedTech Diagnostics AG curated example data
- [ ] 8.3.2: Save to `public/examples/midcap-healthcare/` directory
- [ ] 8.3.3: Update `generate-curated-examples.ts` to include healthcare
- [ ] 8.3.4: Update ExampleCards.tsx - swap Financials MidCap tile for Healthcare
- [ ] 8.3.5: Verify example loads correctly on landing page

---

### Story 8.4: Sector Teaser UI

**As a** landing page visitor
**I want to** see which sectors are supported
**So that** I understand the breadth of tellingCube's capabilities

**Acceptance Criteria:**
- [ ] AC1: Sector badges displayed below curated examples grid
- [ ] AC2: Shows: Consumer Staples, Industrials, Financials, Healthcare
- [ ] AC3: Each sector has icon and label
- [ ] AC4: Responsive design (stacks on mobile)

**Tasks:**
- [ ] 8.4.1: Create SectorBadges component
- [ ] 8.4.2: Add to landing page below ExampleCards
- [ ] 8.4.3: Style with sector icons (ShoppingCart, Factory, Building2, Heart)

---

### Story 8.5: Investigate Payroll Event Generation

**As a** developer
**I want to** understand why Finance payroll ≠ HR payroll
**So that** I can fix the reconciliation bug

**Background:**
- Finance calculates payroll from `operational_work` events (€12.27M)
- HR calculates payroll from employee salary × 12 (€3.83M)
- 3.2x mismatch breaks tellingCube's core reconciliation promise

**Acceptance Criteria:**
- [ ] AC1: Document where `operational_work` events are generated in two-phase-generator
- [ ] AC2: Document where employee salaries are set in world-building
- [ ] AC3: Identify the disconnect between these two sources
- [ ] AC4: Write investigation findings to this story

**Tasks:**
- [ ] 8.5.1: Trace `operational_work` event generation in `two-phase-generator.ts`
- [ ] 8.5.2: Trace employee salary assignment in world-building phase
- [ ] 8.5.3: Document root cause and proposed fix approach

---

### Story 8.6: Fix Payroll Reconciliation

**As a** user viewing Finance and HR views
**I want** the payroll figures to match
**So that** I can trust the data is internally consistent

**Acceptance Criteria:**
- [ ] AC1: `operational_work` events sum = Σ(employee.salary × months_employed)
- [ ] AC2: Finance payroll matches HR totalPayroll (within 1% tolerance for rounding)
- [ ] AC3: Fix verified on healthcare example regeneration

**Tasks:**
- [ ] 8.6.1: Modify generator to derive `operational_work` amounts from employee salaries
- [ ] 8.6.2: Ensure monthly payroll events = headcount × avg salary for that month
- [ ] 8.6.3: Add unit test verifying payroll reconciliation

---

### Story 8.7: Fix Duplicate Month Bug

**As a** user viewing HR headcount trends
**I want** each month to appear only once
**So that** the timeline chart is accurate

**Background:**
- `headcountByMonth` array has 2024-03 appearing twice
- Summary says headcount=65, but December shows 79 (drift)

**Acceptance Criteria:**
- [ ] AC1: Each month appears exactly once in headcountByMonth
- [ ] AC2: Summary headcount matches final month headcount
- [ ] AC3: Hires/terminations net correctly to headcount changes

**Tasks:**
- [ ] 8.7.1: Find duplicate month bug in `hr-queries.ts` or generation
- [ ] 8.7.2: Fix month deduplication
- [ ] 8.7.3: Ensure summary.headcount = final month headcount

---

### Story 8.8: Add Generation-Time Validation

**As a** developer running the generation script
**I want** immediate failure if Finance ≠ HR payroll
**So that** we never ship broken examples

**Acceptance Criteria:**
- [ ] AC1: Generation script validates Finance payroll ≈ HR payroll (±1%)
- [ ] AC2: Generation fails with clear error message if mismatch detected
- [ ] AC3: Validation runs before writing JSON files

**Tasks:**
- [ ] 8.8.1: Add reconciliation check function to `generate-curated-examples.ts`
- [ ] 8.8.2: Check payroll, revenue vs sales, headcount consistency
- [ ] 8.8.3: Fail fast with actionable error message

---

### Story 8.9: Regenerate Healthcare Example

**As a** visitor
**I want** the MedTech Diagnostics example to have correct data
**So that** I can trust tellingCube's data quality

**Acceptance Criteria:**
- [ ] AC1: Healthcare example regenerated with fixed generator
- [ ] AC2: Finance payroll = HR payroll (verified)
- [ ] AC3: No duplicate months
- [ ] AC4: Expansion narrative preserved in description

**Tasks:**
- [ ] 8.9.1: Run `pnpm seed:examples --index=8` with fixed generator
- [ ] 8.9.2: Verify reconciliation passes
- [ ] 8.9.3: Commit regenerated JSON files

---

### Story 8.10: Validate All Curated Examples

**As a** product owner
**I want** all 9 examples to pass reconciliation checks
**So that** the bug fix didn't break existing examples

**Acceptance Criteria:**
- [ ] AC1: All 9 examples pass Finance-HR payroll reconciliation
- [ ] AC2: No duplicate months in any example
- [ ] AC3: Summary headcounts match final month

**Tasks:**
- [ ] 8.10.1: Run validation script across all examples
- [ ] 8.10.2: Fix any failing examples
- [ ] 8.10.3: Document validation results

---

### Story 8.11: Add CI Reconciliation Test

**As a** developer
**I want** CI to catch reconciliation failures
**So that** we prevent regressions

**Acceptance Criteria:**
- [ ] AC1: CI job validates curated example JSON files
- [ ] AC2: Fails PR if Finance payroll ≠ HR payroll (±1%)
- [ ] AC3: Runs on changes to `public/examples/**` or generation scripts

**Tasks:**
- [ ] 8.11.1: Create `scripts/validate-examples.ts`
- [ ] 8.11.2: Add to CI workflow (GitHub Actions)
- [ ] 8.11.3: Test with intentionally broken data

---

## Technical Notes

### Files to Modify

| File | Story | Changes |
|------|-------|---------|
| `lib/config/scenario-matrix.ts` | 8.1 | Add healthcare sector config |
| `lib/prompts/world-building-prompt.ts` | 8.2 | Healthcare narrative + roles (if needed) |
| `scripts/generate-curated-examples.ts` | 8.3 | Add healthcare example generation |
| `public/examples/midcap-healthcare/*` | 8.3 | New curated example JSON files |
| `components/landing/ExampleCards.tsx` | 8.3, 8.4 | Swap tile, add sector badges |

### Tier Gating

Healthcare generation gated same as HR domain:
- Lifetime+ (one-time)
- Pro (subscription)
- Team (subscription)

Free/Single/Lifetime users see the curated example but cannot generate new Healthcare scenarios.

---

## Definition of Done

### Phase 1 (Stories 8.1-8.4) - ✅ Code Complete, ❌ Blocked
- [x] Healthcare sector config added
- [x] Healthcare HR roles implemented
- [x] Curated example generated
- [x] Sector teaser visible
- [ ] ❌ BLOCKED: Finance-HR payroll mismatch (€12.27M vs €3.83M)

### Phase 2 (Stories 8.5-8.11) - Reconciliation Fix
- [ ] Root cause investigated and documented (8.5)
- [ ] Payroll reconciliation fixed (8.6)
- [ ] Duplicate month bug fixed (8.7)
- [ ] Generation-time validation added (8.8)
- [ ] Healthcare example regenerated (8.9)
- [ ] All 9 examples validated (8.10)
- [ ] CI reconciliation test added (8.11)
- [ ] Mario validates regenerated data
- [ ] Production deployment approved

---

## Validation Checklist (Mario)

Before production promotion:

1. [ ] Financial ratios look realistic
2. [ ] HR roles are appropriate for medical setting
3. [ ] Department distribution makes sense
4. [ ] Salary ranges reasonable for DACH healthcare
5. [ ] No obviously fake patterns
6. [ ] Curated example looks professional

---

*Epic created by John (PM) - Ready for sprint planning*
