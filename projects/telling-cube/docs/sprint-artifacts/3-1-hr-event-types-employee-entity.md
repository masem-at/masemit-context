# Story 3.1: HR Event Types & Employee Entity

**Status:** done
**Epic:** 3 - HR Data Domain
**Points:** 5
**Created:** 2026-01-21
**Completed:** 2026-01-21

---

## Story

As a **BI developer or HR analyst**,
I want **tellingCube to generate HR-specific events and employee entities**,
so that **I can test HR analytics dashboards with realistic workforce data**.

---

## Acceptance Criteria

1. **AC1: New HR Event Types Implemented**
   - [x] `EMPLOYEE_HIRED` - New employee joining the company
   - [x] `EMPLOYEE_TERMINATED` - Employee leaving (resignation, termination, retirement)
   - [x] `EMPLOYEE_PROMOTED` - Role/level change within company
   - [x] `EMPLOYEE_TRANSFERRED` - Department or region change
   - [x] `SALARY_CHANGE` - Compensation adjustment
   - [x] `TRAINING_COMPLETED` - Training/certification completion
   - [x] `PERFORMANCE_REVIEW` - Periodic performance evaluation

2. **AC2: Employee Entity Added to ScenarioWorld**
   - [x] Employee interface extended with HR-specific fields:
     - `firstName`, `lastName` (synthetic names via faker)
     - `department`, `costCenter`, `region`
     - `role`, `level` (Junior/Mid/Senior/Lead/Manager/Director)
     - `hireDate`, `terminationDate` (optional)
     - `salary`, `currency`
     - `birthYear` (NOT full DOB - privacy consideration)
     - `gender` ('M' | 'F' | 'D')
     - `skills[]`
     - `reportsTo` (manager ID, optional)

3. **AC3: Database Schema Updated**
   - [x] No Prisma schema changes needed - Event model already supports HR events via:
     - `eventType` field for new HR types
     - `counterpartyId` for employee ID
     - `amountEur` for salary/cost amounts
     - `metadata` JSON for HR-specific attributes

---

## Tasks / Subtasks

### Task 1: Define HR Event Types (AC: #1)
- [x] 1.1 Add HR event type constants to `lib/types/event-types.ts` (create if needed)
- [x] 1.2 Document event type metadata structure for each HR event type

### Task 2: Extend Employee Interface (AC: #2)
- [x] 2.1 Update `lib/types/scenario-world.ts` - extend `Employee` interface with new fields
- [x] 2.2 Update `SalaryBand` type to include HR levels (Junior/Mid/Senior/Lead/Manager/Director)
- [x] 2.3 Add `HREmployee` interface if backward compatibility needed, or extend existing

### Task 3: Update World Building Generator (AC: #2)
- [x] 3.1 Update `lib/generators/world-building-generator.ts` validation for new Employee fields
- [x] 3.2 Add validation for `reportsTo` cross-references (must reference valid employee ID)
- [x] 3.3 Update auto-fix logic for new HR employee fields

### Task 4: Update World Building Prompt (AC: #2)
- [x] 4.1 Update `lib/prompts/world-building-prompt.ts` to request HR employee fields from Claude
- [x] 4.2 Add guidelines for realistic HR data (turnover patterns, salary distributions)

### Task 5: Add HR Event Metadata Types (AC: #1)
- [x] 5.1 Create `lib/types/hr-events.ts` with metadata interfaces for each HR event type
- [x] 5.2 Define `HiredEventMetadata`, `TerminatedEventMetadata`, `PromotedEventMetadata`, etc.

---

## Dev Notes

### Architecture Pattern: Event Sourcing

tellingCube uses **event-sourced architecture** where all data views (Finance, Sales, HR) derive from a single `Event` table. HR events will follow the same pattern:

```typescript
// Event model already supports HR via existing fields:
model Event {
  eventType         String    // 'EMPLOYEE_HIRED', 'EMPLOYEE_TERMINATED', etc.
  counterpartyId    String?   // Employee ID
  amountEur         Decimal?  // Salary amount
  metadata          Json      // HR-specific attributes (reason, newLevel, etc.)
  costCenter        String?   // Department
  region            String?   // Location
}
```

### Existing Employee Interface (lib/types/scenario-world.ts)

Current structure (keep backward compatible):
```typescript
export interface Employee {
  id: string
  name: string              // Will split to firstName + lastName
  role: string
  department: string        // Maps to cost center
  salaryBand: SalaryBand    // 'junior' | 'specialist' | 'manager' | 'executive'
  hireDate: string          // "2022-03" format
  region?: string
}
```

New fields to add:
```typescript
export interface Employee {
  // Existing fields...

  // New HR fields
  firstName: string         // Synthetic via faker
  lastName: string          // Synthetic via faker
  name: string              // Full name (computed: firstName + lastName)
  level: EmployeeLevel      // Junior/Mid/Senior/Lead/Manager/Director
  salary: number            // Monthly salary EUR
  currency: string          // Default 'EUR'
  birthYear: number         // For age distribution (NOT full DOB)
  gender: 'M' | 'F' | 'D'
  skills: string[]
  reportsTo?: string        // Manager employee ID
  terminationDate?: string  // If departed (YYYY-MM format)
}
```

### HR Event Metadata Structures

```typescript
// EMPLOYEE_HIRED
interface HiredEventMetadata {
  previousEmployer?: string
  recruitmentChannel?: string  // 'referral' | 'job_board' | 'agency'
  startingSalary: number
}

// EMPLOYEE_TERMINATED
interface TerminatedEventMetadata {
  reason: 'resignation' | 'termination' | 'retirement' | 'end_of_contract'
  finalSalary: number
  tenure: number  // months
}

// EMPLOYEE_PROMOTED
interface PromotedEventMetadata {
  previousRole: string
  newRole: string
  previousLevel: EmployeeLevel
  newLevel: EmployeeLevel
  salaryChange: number  // EUR difference
}

// SALARY_CHANGE
interface SalaryChangeMetadata {
  previousSalary: number
  newSalary: number
  reason: 'promotion' | 'annual_review' | 'market_adjustment' | 'performance'
  effectiveDate: string
}
```

### Sector-Specific Turnover Rates (from spec)

| Company Size | Turnover Rate | Hiring Pattern |
|--------------|---------------|----------------|
| Startup      | 15-25%        | Rapid hiring phases |
| MidCap       | 10-15%        | Steady growth |
| LargeCap     | 5-10%         | Large workforce, clear hierarchy |

### Project Structure Notes

**Files to modify:**
- `lib/types/scenario-world.ts` - Extend Employee interface
- `lib/generators/world-building-generator.ts` - Add validation for new fields
- `lib/prompts/world-building-prompt.ts` - Update Claude prompt

**Files to create:**
- `lib/types/hr-events.ts` - HR event metadata types
- `lib/types/event-types.ts` - Centralized event type constants (if not exists)

### Privacy Considerations (GDPR)

- **NO** full birthdates - only `birthYear` for age brackets
- **NO** real addresses or contact info
- All names are **synthetic** (faker.js style)
- Data clearly marked as "synthetic/demo data"
- Option for fully anonymized names (Employee_001)

### Multi-Year Integration (Epic 2 Dependency)

HR events must integrate with multi-year scenarios:
- If `durationYears` > 1, HR events span all years
- Realistic workforce evolution (hires, departures, promotions over time)
- Consistency: Finance payroll must match HR total compensation

### References

- [Source: docs/future/spec-hr-data-domain.md] - Full HR domain specification
- [Source: docs/epics/epic-03-hr-data-domain.md] - Epic and story details
- [Source: docs/architecture.md#FR18] - HR Domain architecture notes
- [Source: lib/types/scenario-world.ts] - Current Employee interface
- [Source: prisma/schema.prisma] - Event model structure

---

## Dev Agent Record

### Context Reference

Story created via create-story workflow with comprehensive artifact analysis.

### Agent Model Used

Claude Opus 4.5 (claude-opus-4-5-20251101)

### Debug Log References

- TypeScript build: PASS (no errors)
- ESLint: PASS (only pre-existing warnings)

### Completion Notes List

- [x] Story drafted and ready for dev
- [x] Created lib/types/event-types.ts with HR_EVENT_TYPES constants
- [x] Created lib/types/hr-events.ts with metadata interfaces for all 7 HR event types
- [x] Extended Employee interface in lib/types/scenario-world.ts with HR fields
- [x] Added EmployeeLevel and EmployeeGender types
- [x] Updated world-building-generator.ts with validation for new fields
- [x] Updated world-building-prompt.ts with HR data guidelines
- [x] Fixed payroll-deriver.ts to support new Employee interface

### File List

**New Files:**
- `lib/types/event-types.ts` - HR event type constants (EMPLOYEE_HIRED, EMPLOYEE_TERMINATED, etc.)
- `lib/types/hr-events.ts` - HR event metadata interfaces

**Modified Files:**
- `lib/types/scenario-world.ts` - Extended Employee interface with HR fields
- `lib/generators/world-building-generator.ts` - Added HR field validation and auto-fix
- `lib/prompts/world-building-prompt.ts` - Updated prompt for HR employee data
- `lib/derivation/payroll-deriver.ts` - Added helper functions for new Employee fields

### Change Log

- 2026-01-21: Story 3.1 implemented - HR Event Types & Employee Entity
- 2026-01-21: Code review completed - fixed unused import, added missing type guards, fixed gender default bias
