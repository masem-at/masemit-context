# Story 12.3: Implement 30-Day Rolling Cleanup Job

Status: done

## Story

As an **admin**,
I want **daily snapshots older than 30 days automatically deleted**,
So that **database storage remains efficient** while monthly aggregates preserve long-term history.

## Acceptance Criteria

### AC1: Cleanup Cron Job Execution
- [x] Cron job runs weekly on Sundays at 3am UTC
- [x] Schedule: `0 3 * * 0` (integrated with existing gdpr-cleanup)
- [x] Job only deletes `snapshot_type='daily'` records
- [x] Monthly snapshots (`snapshot_type='monthly'`) never deleted

### AC2: 30-Day Retention Window
- [x] Daily snapshots older than 30 days are deleted
- [x] Retention period configurable via `GVS_DAILY_RETENTION_DAYS` env var
- [x] Default: 30 days
- [x] Deletion is based on `snapshot_date` column

### AC3: Safe Deletion Logic
- [x] Job verifies monthly snapshot count before/after deletion
- [x] Logs critical error if monthly count changes unexpectedly
- [x] Transaction-safe: single DELETE statement
- [x] Dry-run mode available via `SNAPSHOT_CLEANUP_DRY_RUN=true`

### AC4: Logging and Monitoring
- [x] Job logs: records deleted, time taken, any errors
- [x] Slack notification on completion (optional, configurable)
- [x] Job result stored in `job_logs` table
- [x] Errors do not crash job; logged and reported

### AC5: GDPR Compliance
- [x] Cleanup integrated with existing GDPR deletion job
- [x] Does not interfere with opt-out data handling
- [x] Audit trail maintained in job_logs

## Tasks / Subtasks

- [x] Task 1: Create Cron Job Route (AC: #1)
  - [x] 1.1 Extended `/api/jobs/gdpr-cleanup/route.ts` to include snapshot cleanup
  - [x] 1.2 Uses existing GET handler with CRON_SECRET auth
  - [x] 1.3 Uses existing `0 3 * * 0` cron schedule (no change to vercel.json)
  - [x] 1.4 Integrated with existing GDPR cleanup for single weekly job

- [x] Task 2: Implement Cleanup Logic (AC: #2, #3)
  - [x] 2.1 Created `src/lib/jobs/snapshot-cleanup.ts`
  - [x] 2.2 Implemented `executeSnapshotCleanup(retentionDays, dryRun)` function
  - [x] 2.3 Queries for daily snapshots older than retention window
  - [x] 2.4 Single DELETE statement (transaction-safe)
  - [x] 2.5 Returns SnapshotCleanupResult object

- [x] Task 3: Add Configuration (AC: #2)
  - [x] 3.1 Added `GVS_DAILY_RETENTION_DAYS` env var (default: 30)
  - [x] 3.2 Added to `.env.example` with documentation
  - [x] 3.3 Validates retention is positive integer (falls back to 30)

- [x] Task 4: Implement Safety Checks (AC: #3)
  - [x] 4.1 Verifies monthly snapshot count before/after deletion
  - [x] 4.2 Logs CRITICAL error if monthly count changes unexpectedly
  - [x] 4.3 Implemented dry-run mode via `SNAPSHOT_CLEANUP_DRY_RUN=true`
  - [x] 4.4 Dry-run logs what would be deleted without deleting

- [x] Task 5: Add Logging and Monitoring (AC: #4, #5)
  - [x] 5.1 Logs cleanup start, progress, completion
  - [x] 5.2 Stores result in job_logs table via createJobLog/updateJobLog
  - [x] 5.3 Optional Slack notification on completion
  - [x] 5.4 Includes deleted count, retention days, errors in result

- [x] Task 6: Testing (AC: #1-5)
  - [x] 6.1 Unit tests for cleanup logic with mock data (16 tests)
  - [x] 6.2 Tests retention window calculation and configuration
  - [x] 6.3 Tests monthly snapshots are preserved (count verification)
  - [ ] 6.4 Integration test full job execution (deferred - requires DB)

## Dev Notes

### Files to Create/Modify

```
src/lib/jobs/snapshot-cleanup.ts              # NEW - cleanup logic
src/app/api/jobs/gdpr-cleanup/route.ts        # Extend or create
vercel.json                                    # Add cron schedule
.env.example                                   # Add retention config
```

### Cleanup Logic

```typescript
// src/lib/jobs/snapshot-cleanup.ts

import { db } from '@/lib/db'
import { gvsSnapshots } from '@/lib/db/schema'
import { and, eq, lt } from 'drizzle-orm'

interface CleanupResult {
  deletedCount: number
  retentionDays: number
  cutoffDate: Date
  dryRun: boolean
  errors: string[]
  duration: number
}

export async function executeSnapshotCleanup(
  retentionDays: number = 30,
  dryRun: boolean = false
): Promise<CleanupResult> {
  const startTime = Date.now()
  const errors: string[] = []

  // Calculate cutoff date
  const cutoffDate = new Date()
  cutoffDate.setDate(cutoffDate.getDate() - retentionDays)
  cutoffDate.setHours(0, 0, 0, 0)

  console.log(`[Cleanup] Starting snapshot cleanup`)
  console.log(`[Cleanup] Retention: ${retentionDays} days`)
  console.log(`[Cleanup] Cutoff date: ${cutoffDate.toISOString()}`)
  console.log(`[Cleanup] Dry run: ${dryRun}`)

  try {
    // Count records to delete
    const toDelete = await db
      .select({ id: gvsSnapshots.id })
      .from(gvsSnapshots)
      .where(
        and(
          eq(gvsSnapshots.snapshotType, 'daily'),
          lt(gvsSnapshots.snapshotDate, cutoffDate)
        )
      )

    const deleteCount = toDelete.length
    console.log(`[Cleanup] Found ${deleteCount} daily snapshots to delete`)

    if (!dryRun && deleteCount > 0) {
      // Delete in batches of 100
      const batchSize = 100
      for (let i = 0; i < toDelete.length; i += batchSize) {
        const batch = toDelete.slice(i, i + batchSize)
        const ids = batch.map(r => r.id)

        await db
          .delete(gvsSnapshots)
          .where(
            and(
              eq(gvsSnapshots.snapshotType, 'daily'),
              lt(gvsSnapshots.snapshotDate, cutoffDate)
            )
          )

        console.log(`[Cleanup] Deleted batch ${Math.floor(i / batchSize) + 1}`)
      }
    }

    return {
      deletedCount: dryRun ? 0 : deleteCount,
      retentionDays,
      cutoffDate,
      dryRun,
      errors,
      duration: Date.now() - startTime
    }
  } catch (error) {
    const errorMsg = error instanceof Error ? error.message : 'Unknown error'
    errors.push(errorMsg)
    console.error(`[Cleanup] Error: ${errorMsg}`)

    return {
      deletedCount: 0,
      retentionDays,
      cutoffDate,
      dryRun,
      errors,
      duration: Date.now() - startTime
    }
  }
}
```

### Cron Configuration

```json
// In vercel.json
{
  "crons": [
    {
      "path": "/api/jobs/daily-recalculation",
      "schedule": "0 0 * * *"
    },
    {
      "path": "/api/jobs/monthly-aggregation",
      "schedule": "0 1 1 * *"
    },
    {
      "path": "/api/jobs/gdpr-cleanup",
      "schedule": "0 3 * * 0"
    }
  ]
}
```

### Environment Variables

```env
# Snapshot retention configuration
# Daily snapshots older than this are deleted
# Default: 30 days
GVS_DAILY_RETENTION_DAYS=30

# Dry run mode for cleanup job
# Set to "true" to log what would be deleted without deleting
# DRY_RUN=true
```

### API Route Handler

```typescript
// src/app/api/jobs/gdpr-cleanup/route.ts

import { NextRequest, NextResponse } from 'next/server'
import { executeSnapshotCleanup } from '@/lib/jobs/snapshot-cleanup'
import { logJobResult } from '@/lib/monitoring/job-logs'

export async function GET(request: NextRequest) {
  // Verify cron secret
  const authHeader = request.headers.get('authorization')
  if (authHeader !== `Bearer ${process.env.CRON_SECRET}`) {
    return NextResponse.json({ error: 'Unauthorized' }, { status: 401 })
  }

  const retentionDays = parseInt(process.env.GVS_DAILY_RETENTION_DAYS || '30')
  const dryRun = process.env.DRY_RUN === 'true'

  const result = await executeSnapshotCleanup(retentionDays, dryRun)

  // Log job result
  await logJobResult({
    jobName: 'snapshot-cleanup',
    status: result.errors.length > 0 ? 'partial' : 'success',
    result: result,
    executedAt: new Date()
  })

  return NextResponse.json({
    success: result.errors.length === 0,
    ...result
  })
}
```

### Job Execution Flow

```
Every Sunday @ 3am UTC
        │
        ▼
┌───────────────────┐
│ Calculate cutoff  │
│ date (30 days ago)│
└─────────┬─────────┘
          │
          ▼
┌───────────────────┐
│ Query daily       │
│ snapshots before  │
│ cutoff date       │
└─────────┬─────────┘
          │
          ▼
┌───────────────────┐
│ Delete in batches │
│ (preserves        │
│ monthly snapshots)│
└─────────┬─────────┘
          │
          ▼
┌───────────────────┐
│ Log results       │
│ Optional Slack    │
└───────────────────┘
```

### Data Retention Summary

| Type | Retention | Cleanup |
|------|-----------|---------|
| Daily snapshots | 30 days | Weekly cleanup job |
| Monthly snapshots | Indefinite (12+ months) | Never deleted |
| Job logs | 90 days (existing) | Existing GDPR job |

### Dependency

This story can run in parallel with Stories 12-4 and 12-5, but should be implemented after Story 12-1 (daily snapshots) is working to have data to clean up.

### Testing Strategy

1. **Unit Tests:**
   - `executeSnapshotCleanup()` deletes correct records
   - Monthly snapshots are never deleted
   - Dry run mode doesn't delete anything
   - Retention window calculated correctly

2. **Integration Tests:**
   - Full job execution with test data
   - Verify only daily snapshots deleted
   - Verify monthly snapshots preserved

### Safety Considerations

- Always test with `DRY_RUN=true` first in production
- Monitor job_logs for unexpected behavior
- Consider adding admin UI to trigger manual cleanup with preview

### References

- [Source: src/lib/jobs/daily-recalculation.ts] - Job structure reference
- [Source: src/app/api/jobs/] - Existing cron job patterns
- [GDPR compliance requirements from Epic 8]

## Dev Agent Record

### Context Reference

Story created from Epic 12: Daily GVS Monitoring & Historical Tracking.
Ensures database efficiency while preserving long-term analysis capability.

### Agent Model Used

Claude Opus 4.5 (claude-opus-4-5-20251101)

### Debug Log References

N/A

### Completion Notes List

- Created `src/lib/jobs/snapshot-cleanup.ts` with cleanup logic
- Extended `src/app/api/jobs/gdpr-cleanup/route.ts` to run both GDPR + snapshot cleanup
- Added `GVS_DAILY_RETENTION_DAYS` and `SNAPSHOT_CLEANUP_DRY_RUN` env vars to .env.example
- Implemented safety check: verifies monthly snapshot count before/after deletion
- All 16 unit tests passing (retention config, dry run, deletion, error handling)
- Integration test deferred - requires database setup

### File List

- `src/lib/jobs/snapshot-cleanup.ts` (NEW - 195 lines)
- `src/app/api/jobs/gdpr-cleanup/route.ts` (MODIFIED - added snapshot cleanup)
- `.env.example` (MODIFIED - added cleanup config vars)
- `tests/unit/jobs/snapshot-cleanup.test.ts` (NEW - 16 tests)

### Code Review (2026-01-23)

**Reviewer:** Claude Opus 4.5 (Adversarial Mode)

**Issues Found & Fixed:**
- HIGH #1: TypeScript error - used `'warning'` instead of `'alert'` for sendSlackAlert severity - FIXED

**Result:** All issues resolved. Story approved for merge.

