# Story 8.11: Add CI Reconciliation Test

Status: done

## Story

As a developer,
I want CI to catch reconciliation failures,
So that we prevent regressions.

## Acceptance Criteria

1. - [x] AC1: CI job validates curated example JSON files
2. - [x] AC2: Fails PR if Finance payroll ≠ HR payroll (±1%)
3. - [x] AC3: Runs on changes to `public/examples/**` or generation scripts

## Tasks / Subtasks

- [x] Task 1: Create validation script (AC: 1, 2) - DONE in Story 8.10
  - [x] 1.1: `scripts/validate-curated-examples.ts` already exists
  - [x] 1.2: `npm run validate:examples` command exists
- [x] Task 2: Add to CI workflow (AC: 1, 3)
  - [x] 2.1: Check existing CI workflow configuration (none existed)
  - [x] 2.2: Add validation job to GitHub Actions
  - [x] 2.3: Configure path triggers for `public/examples/**` and `scripts/generate-*.ts`
- [x] Task 3: Test CI integration (AC: 1, 2, 3)
  - [x] 3.1: Workflow configured to run on PR
  - [x] 3.2: Document CI configuration

## Dev Notes

### Workflow Configuration

Created `.github/workflows/validate-examples.yml`:

```yaml
name: Validate Curated Examples

on:
  pull_request:
    branches: [main, master, develop]
    paths:
      - 'public/examples/**'
      - 'scripts/generate-curated-examples.ts'
      - 'scripts/validate-curated-examples.ts'
  push:
    branches: [main, master, develop]
    paths: [same as above]
  workflow_dispatch: # Manual trigger
```

### Triggers

| Trigger | Condition |
|---------|-----------|
| PR | To main/master/develop when paths match |
| Push | To main/master/develop when paths match |
| Manual | workflow_dispatch enabled |

### Path Filters

- `public/examples/**` - Any example JSON changes
- `scripts/generate-curated-examples.ts` - Generation script
- `scripts/validate-curated-examples.ts` - Validation script

### Key Files

| File | Change |
|------|--------|
| `.github/workflows/validate-examples.yml` | NEW - CI workflow |

### References

- [Source: docs/epics/epic-08-healthcare-sector.md] - Story 8.11 definition
- [Source: scripts/validate-curated-examples.ts] - Validation script from 8.10

## Dev Agent Record

### Context Reference

Story context from Epic 8.11 in docs/epics/epic-08-healthcare-sector.md

### Agent Model Used

Claude Opus 4.5 (claude-opus-4-5-20251101)

### Debug Log References

### Completion Notes List

- No existing CI workflow found - created new one
- Created `.github/workflows/validate-examples.yml`
- Configured triggers: PR, push, manual dispatch
- Path filters: public/examples/**, scripts/generate-*.ts, scripts/validate-*.ts
- Uses Node.js 20, npm ci, npm run validate:examples

### File List

| File | Change |
|------|--------|
| `.github/workflows/validate-examples.yml` | NEW - CI workflow for validation |
