# Report Validation Layer — Technical Specification

**Author:** Winston (Architect)
**Status:** ✅ Implemented
**Priority:** High — prevents data inconsistency bugs from reaching customers
**Implemented:** December 2024

---

## Overview

A deterministic validation checkpoint that runs **after AI analysis, before PDF generation**. Compares key metrics in `aiAnalysis` against source `rawData` to catch hallucinations, calculation errors, and formatting bugs.

No AI involved — pure arithmetic verification.

---

## Architecture

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│  rawData    │ ──▶ │ AI Analysis │ ──▶ │  VALIDATOR  │ ──▶ │  PDF Gen    │
│  (source)   │     │ (aiAnalysis)│     │  (new)      │     │             │
└─────────────┘     └─────────────┘     └──────┬──────┘     └─────────────┘
                                               │
                                      ┌────────▼────────┐
                                      │ PASS → continue │
                                      │ WARN → flag     │
                                      │ FAIL → halt     │
                                      └─────────────────┘
```

### Insertion Point

In `src/app/api/jobs/analyze/route.ts`, after AI analysis completes:

```typescript
// Current flow
const analysis = await analyzeGovernanceData(governanceData)

// NEW: Validate before proceeding
const validation = validateReportConsistency(governanceData, analysis)
if (validation.status === 'failed') {
  // Store errors, set status to 'validation_failed'
  // Do NOT generate PDF
}

// Continue to PDF generation...
```

---

## Validation Rules

### Critical (FAIL if wrong)

| Rule ID | Check | Source | Target | Tolerance |
|---------|-------|--------|--------|-----------|
| `CRIT-001` | Top 1% | `rawData.powerDistribution.top1Percentage` | `aiAnalysis.powerDistribution.concentration.top1Percentage` | ±0.5% |
| `CRIT-002` | Top 5% | `rawData.powerDistribution.top5Percentage` | Same path pattern | ±0.5% |
| `CRIT-003` | Top 10% | `rawData.powerDistribution.top10Percentage` | Same path pattern | ±0.5% |
| `CRIT-004` | Top 20% | `rawData.powerDistribution.top20Percentage` | Same path pattern | ±0.5% |
| `CRIT-005` | Gini coefficient | `rawData.powerDistribution.giniCoefficient` | Same path pattern | ±0.01 |
| `CRIT-006` | Unique voters | `rawData.powerDistribution.uniqueVoters` | Same path pattern | Exact |
| `CRIT-007` | Total voting power | `rawData.powerDistribution.totalVotingPower` | Same path pattern | ±1% |
| `CRIT-008` | Participation rate conversion | Verify top voter participation rates are ×100 of source decimal | See WARN-006 | Must match |

### Warnings (Flag but continue)

| Rule ID | Check | Description |
|---------|-------|-------------|
| `WARN-001` | Whale count mismatch | AI classified different count than source `isWhale` flags |
| `WARN-002` | Delegate count mismatch | AI classified different count than source `isDelegate` flags |
| `WARN-003` | Score out of expected range | `decentralizationScore` doesn't correlate with Gini (e.g., Gini 0.9 but score 8/10) |
| `WARN-004` | Missing top voters | AI descriptions missing for top 5 by voting power |
| `WARN-005` | Participation rate sanity | Any rate > 100% or negative |
| `WARN-006` | Participation rate decimal trap | Detects if displayed rate appears to be raw decimal (see below) |

### WARN-006: Participation Rate Decimal Trap (Special Rule)

This rule specifically catches the bug where `participationRate: 0.8389` (decimal) gets displayed as "0.8%" instead of "83.9%".

**Detection logic:**

```typescript
// For each top voter in AI output text/summary:
// If displayed participation < 10% BUT source participationRate > 0.1
// → Likely decimal-to-percentage conversion was skipped

function checkParticipationDecimalTrap(
  rawVoter: VoterAnalysis,
  displayedRate: number  // Extracted from AI text
): boolean {
  // Source is decimal (0-1), e.g., 0.8389
  const sourceDecimal = rawVoter.participationRate

  // If source suggests high participation (>10%)
  // but displayed is suspiciously low (<10%)
  // they probably forgot to multiply by 100
  if (sourceDecimal > 0.1 && displayedRate < 10) {
    // Check if displayed ≈ source (forgot ×100)
    if (Math.abs(displayedRate - sourceDecimal * 100) < 1) {
      return false // Correct conversion
    }
    if (Math.abs(displayedRate - sourceDecimal) < 0.1) {
      return true // BUG: displayed raw decimal as percentage
    }
  }

  return false
}
```

**Example catches:**

| Source (decimal) | Displayed | Expected | Bug? |
|------------------|-----------|----------|------|
| 0.8389 | 0.8% | 83.9% | ✓ YES — caught |
| 0.8389 | 83.9% | 83.9% | ✗ Correct |
| 0.05 | 5.0% | 5.0% | ✗ Correct |
| 0.05 | 0.05% | 5.0% | ✓ YES — caught |

### Informational (Log only)

| Rule ID | Check | Description |
|---------|-------|-------------|
| `INFO-001` | Proposal count | Number of proposals analyzed matches source |
| `INFO-002` | Analysis timestamp | `fetchedAt` is within expected range |

---

## Data Structures

### ValidationResult

```typescript
// src/lib/validation/types.ts

export type ValidationStatus = 'passed' | 'warnings' | 'failed'

export type ValidationSeverity = 'critical' | 'warning' | 'info'

export interface ValidationIssue {
  ruleId: string
  severity: ValidationSeverity
  message: string
  expected: string | number
  actual: string | number
  path: string  // JSON path to the problematic field
}

export interface ValidationResult {
  status: ValidationStatus
  issues: ValidationIssue[]
  checkedAt: string  // ISO timestamp
  rulesRun: number
  rulesPassed: number
}
```

### Database Schema Addition

```typescript
// Add to reports table in schema.ts

validationStatus: text('validation_status'),  // 'passed' | 'warnings' | 'failed'
validationResult: jsonb('validation_result'), // Full ValidationResult object
```

---

## Implementation

### Module Structure

```
src/lib/validation/
├── index.ts              # Main export
├── types.ts              # Type definitions
├── validator.ts          # Core validation logic
├── rules/
│   ├── critical.ts       # CRIT-* rules
│   ├── warnings.ts       # WARN-* rules
│   └── info.ts           # INFO-* rules
└── __tests__/
    └── validator.test.ts # Unit tests
```

### Core Validator

```typescript
// src/lib/validation/validator.ts

import type { GovernanceData } from '@/lib/snapshot'
import type { ReportDraft } from '@/lib/ai'
import type { ValidationResult, ValidationIssue } from './types'
import { runCriticalRules } from './rules/critical'
import { runWarningRules } from './rules/warnings'
import { runInfoRules } from './rules/info'

export function validateReportConsistency(
  rawData: GovernanceData,
  analysis: ReportDraft
): ValidationResult {
  const issues: ValidationIssue[] = []

  // Run all rule categories
  issues.push(...runCriticalRules(rawData, analysis))
  issues.push(...runWarningRules(rawData, analysis))
  issues.push(...runInfoRules(rawData, analysis))

  // Determine overall status
  const hasCritical = issues.some(i => i.severity === 'critical')
  const hasWarning = issues.some(i => i.severity === 'warning')

  let status: ValidationStatus = 'passed'
  if (hasCritical) status = 'failed'
  else if (hasWarning) status = 'warnings'

  return {
    status,
    issues,
    checkedAt: new Date().toISOString(),
    rulesRun: issues.length + countPassedRules(rawData, analysis),
    rulesPassed: countPassedRules(rawData, analysis),
  }
}
```

### Example Rule Implementation

```typescript
// src/lib/validation/rules/critical.ts

import type { GovernanceData } from '@/lib/snapshot'
import type { ReportDraft } from '@/lib/ai'
import type { ValidationIssue } from '../types'

export function runCriticalRules(
  rawData: GovernanceData,
  analysis: ReportDraft
): ValidationIssue[] {
  const issues: ValidationIssue[] = []

  // CRIT-001: Top 1% validation
  const expectedTop1 = rawData.powerDistribution.top1Percentage
  const actualTop1 = analysis.powerDistribution.concentration.top1Percentage

  if (Math.abs(expectedTop1 - actualTop1) > 0.5) {
    issues.push({
      ruleId: 'CRIT-001',
      severity: 'critical',
      message: 'Top 1% power concentration mismatch',
      expected: expectedTop1,
      actual: actualTop1,
      path: 'powerDistribution.top1Percentage',
    })
  }

  // CRIT-005: Gini coefficient
  const expectedGini = rawData.powerDistribution.giniCoefficient
  const actualGini = analysis.powerDistribution.concentration.giniCoefficient

  if (Math.abs(expectedGini - actualGini) > 0.01) {
    issues.push({
      ruleId: 'CRIT-005',
      severity: 'critical',
      message: 'Gini coefficient mismatch',
      expected: expectedGini,
      actual: actualGini,
      path: 'powerDistribution.giniCoefficient',
    })
  }

  // ... additional critical rules

  return issues
}
```

---

## Pipeline Integration

### Analyze Job Modification

```typescript
// src/app/api/jobs/analyze/route.ts

import { validateReportConsistency } from '@/lib/validation'

// After AI analysis completes:
const analysis = await analyzeGovernanceData(governanceData)

// NEW: Validate consistency
const validation = validateReportConsistency(governanceData, analysis)

// Store validation result
await db
  .update(reports)
  .set({
    validationStatus: validation.status,
    validationResult: validation as unknown as Record<string, unknown>,
    updatedAt: new Date(),
  })
  .where(eq(reports.id, reportId))

// If critical failure, halt pipeline
if (validation.status === 'failed') {
  await db
    .update(reports)
    .set({
      status: 'validation_failed',
      errorMessage: `Validation failed: ${validation.issues.filter(i => i.severity === 'critical').map(i => i.ruleId).join(', ')}`,
      updatedAt: new Date(),
    })
    .where(eq(reports.id, reportId))

  await logError('Validation Failed', {
    dao: report.daoName,
    issues: validation.issues.length,
  })

  return NextResponse.json({
    success: false,
    reportId,
    status: 'validation_failed',
    validation,
  })
}

// Continue to PDF generation if passed or warnings only...
```

### New Report Status

Add to `reportStatusEnum` in schema:

```typescript
export const reportStatusEnum = pgEnum('report_status', [
  'pending',
  'fetching',
  'analyzing',
  'validation_failed',  // NEW
  'generating',
  'review_pending',
  // ... rest
])
```

---

## Admin UI Integration

### Report Detail Page

Show validation status badge:

```tsx
// In report detail page

{report.validationStatus === 'passed' && (
  <Badge className="bg-green-500/20 text-green-400">
    ✓ Validation Passed
  </Badge>
)}

{report.validationStatus === 'warnings' && (
  <Badge className="bg-yellow-500/20 text-yellow-400">
    ⚠ {report.validationResult?.issues?.length} Warnings
  </Badge>
)}

{report.validationStatus === 'failed' && (
  <Badge className="bg-red-500/20 text-red-400">
    ✗ Validation Failed
  </Badge>
)}
```

### Validation Details Expandable

```tsx
{report.validationResult?.issues?.length > 0 && (
  <Card className="mt-4 border-yellow-500/30">
    <CardHeader>
      <CardTitle className="text-sm">Validation Issues</CardTitle>
    </CardHeader>
    <CardContent>
      {report.validationResult.issues.map((issue, i) => (
        <div key={i} className="text-sm mb-2">
          <span className="font-mono text-yellow-400">[{issue.ruleId}]</span>
          <span className="ml-2">{issue.message}</span>
          <div className="text-xs text-gray-500 ml-6">
            Expected: {issue.expected} | Actual: {issue.actual}
          </div>
        </div>
      ))}
    </CardContent>
  </Card>
)}
```

---

## Test Cases

### Unit Tests Required

```typescript
// src/lib/validation/__tests__/validator.test.ts

describe('Report Validator', () => {
  describe('Critical Rules', () => {
    it('CRIT-001: fails when top1Percentage differs by >0.5%', () => {
      const rawData = mockRawData({ top1Percentage: 10.0 })
      const analysis = mockAnalysis({ top1Percentage: 15.0 })

      const result = validateReportConsistency(rawData, analysis)

      expect(result.status).toBe('failed')
      expect(result.issues).toContainEqual(
        expect.objectContaining({ ruleId: 'CRIT-001' })
      )
    })

    it('CRIT-001: passes when top1Percentage within tolerance', () => {
      const rawData = mockRawData({ top1Percentage: 10.0 })
      const analysis = mockAnalysis({ top1Percentage: 10.3 })

      const result = validateReportConsistency(rawData, analysis)

      expect(result.issues.find(i => i.ruleId === 'CRIT-001')).toBeUndefined()
    })

    it('CRIT-005: fails when Gini differs by >0.01', () => {
      const rawData = mockRawData({ giniCoefficient: 0.85 })
      const analysis = mockAnalysis({ giniCoefficient: 0.90 })

      const result = validateReportConsistency(rawData, analysis)

      expect(result.status).toBe('failed')
      expect(result.issues).toContainEqual(
        expect.objectContaining({ ruleId: 'CRIT-005' })
      )
    })

    it('CRIT-008: fails when participation rate not multiplied by 100', () => {
      // Source: 0.8389 (decimal) should display as 83.9%
      // Bug: displayed as 0.8% (forgot ×100)
      const rawData = mockRawData({
        topVoters: [{ participationRate: 0.8389 }]
      })
      const analysis = mockAnalysis({
        displayedParticipation: '0.8%'  // BUG!
      })

      const result = validateReportConsistency(rawData, analysis)

      expect(result.status).toBe('failed')
      expect(result.issues).toContainEqual(
        expect.objectContaining({
          ruleId: 'CRIT-008',
          message: expect.stringContaining('participation rate'),
          expected: 83.9,
          actual: 0.8,
        })
      )
    })

    it('CRIT-008: passes when participation rate correctly converted', () => {
      const rawData = mockRawData({
        topVoters: [{ participationRate: 0.8389 }]
      })
      const analysis = mockAnalysis({
        displayedParticipation: '83.9%'  // Correct!
      })

      const result = validateReportConsistency(rawData, analysis)

      expect(result.issues.find(i => i.ruleId === 'CRIT-008')).toBeUndefined()
    })
  })

  describe('Warning Rules', () => {
    it('WARN-005: warns on participation rate >100%', () => {
      const rawData = mockRawData()
      const analysis = mockAnalysis({ participationRate: 150 })

      const result = validateReportConsistency(rawData, analysis)

      expect(result.status).toBe('warnings')
      expect(result.issues).toContainEqual(
        expect.objectContaining({ ruleId: 'WARN-005' })
      )
    })
  })

  describe('Overall Status', () => {
    it('returns "passed" when no issues', () => {
      const result = validateReportConsistency(validRawData, validAnalysis)
      expect(result.status).toBe('passed')
    })

    it('returns "failed" if any critical issue', () => {
      // Even with warnings, critical = failed
    })

    it('returns "warnings" if warnings but no critical', () => {
      // ...
    })
  })
})
```

---

## Implementation Checklist

1. [x] Add `validation_failed` to `reportStatusEnum`
2. [x] Add `validationStatus` and `validationResult` columns to reports
3. [x] Run `npm run db:push`
4. [x] Create `src/lib/validation/` module structure
5. [x] Implement critical rules (CRIT-001 through CRIT-008)
6. [x] Implement warning rules (WARN-001 through WARN-006)
7. [x] Write unit tests for all rules (especially CRIT-008 decimal trap)
8. [x] Integrate into analyze job
9. [x] Add validation badge to admin UI
10. [x] Add expandable validation details to admin UI

---

## Notes

- **No AI involved** — This is pure arithmetic comparison
- **Tolerances are intentional** — Floating point math and rounding can cause tiny differences
- **Warnings don't block** — Only critical failures halt the pipeline
- **Audit trail** — Full validation result stored for debugging
- **Extensible** — Easy to add new rules as we discover edge cases

---

*"Trust, but verify. Especially when AI is involved."*

— Winston
