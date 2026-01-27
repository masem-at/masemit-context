# Story 12.4: Implement Anomaly Detection with Alerts

Status: done

## Story

As an **admin**,
I want **immediate alerts when a DAO's score changes significantly**,
So that I can **investigate unexpected governance changes** before they affect reports or outreach.

## Acceptance Criteria

### AC1: Anomaly Detection Logic
- [x] Score changes exceeding threshold trigger anomaly detection
- [x] Default threshold: ±0.5 points
- [x] Threshold configurable via `GVS_ANOMALY_THRESHOLD` env var
- [x] Both increases and decreases detected

### AC2: Anomaly Recording
- [x] Anomalies recorded with: DAO name, old score, new score, delta, timestamp
- [x] Anomalies stored in database (job_logs JSON or dedicated table)
- [x] Historical anomalies queryable for dashboard display

### AC3: Slack Notifications
- [x] Slack alert sent when anomaly detected
- [x] Message format: "DAO: old_score → new_score (delta) ⚠️ SIGNIFICANT CHANGE"
- [x] Multiple anomalies batched into single message (if several in one run)

### AC4: Email Notifications
- [x] Email alert sent when anomaly detected
- [x] Email includes all anomalies from the job run
- [x] Clear formatting with DAO names, scores, and deltas

### AC5: Integration with Daily Job
- [x] Anomaly detection runs after each DAO calculation
- [x] Does not block job execution on detection failure
- [x] Anomaly count included in job completion summary

## Tasks / Subtasks

- [x] Task 1: Create Anomaly Detection Module (AC: #1, #2)
  - [x] 1.1 Create `src/lib/gvs/anomaly-detection.ts`
  - [x] 1.2 Implement `detectAnomaly(daoId, currentScore, previousScore, threshold)` function
  - [x] 1.3 Return anomaly object or null if within threshold
  - [x] 1.4 Add unit tests for threshold detection

- [x] Task 2: Add Environment Configuration (AC: #1)
  - [x] 2.1 Add `GVS_ANOMALY_THRESHOLD` to env vars (default: 0.5)
  - [x] 2.2 Document in README or .env.example
  - [x] 2.3 Validate threshold is positive number

- [x] Task 3: Implement Anomaly Storage (AC: #2)
  - [x] 3.1 Option A: Add anomalies array to job_logs result JSON
  - [x] 3.2 Option B: Create `gvs_anomalies` table (if cleaner)
  - [x] 3.3 Implement `saveAnomaly()` function
  - [x] 3.4 Implement `getRecentAnomalies(hours)` for dashboard

- [x] Task 4: Implement Slack Alerts (AC: #3)
  - [x] 4.1 Create `formatAnomalySlackMessage(anomalies)` function
  - [x] 4.2 Call existing `sendSlackAlert()` with formatted message
  - [x] 4.3 Include severity indicator (⚠️ for drops, 📈 for increases)
  - [x] 4.4 Test with mock anomaly data

- [x] Task 5: Implement Email Alerts (AC: #4)
  - [x] 5.1 Create `generateAnomalyEmailHtml(anomalies)` function
  - [x] 5.2 Call existing `sendEmailAlert()` with HTML content
  - [x] 5.3 Include table of all anomalies with scores
  - [x] 5.4 Test email rendering

- [x] Task 6: Integrate with Daily Job (AC: #5)
  - [x] 6.1 Call `detectAnomaly()` after each DAO's `calculateDelta()`
  - [x] 6.2 Collect all anomalies during job run
  - [x] 6.3 Send batch notification at end of job (if any anomalies)
  - [x] 6.4 Include anomaly count in job result logging
  - [x] 6.5 Handle detection errors gracefully (log, continue)

## Dev Notes

### Files to Create/Modify

```
src/lib/gvs/anomaly-detection.ts         # NEW - detection logic
src/lib/monitoring/alerts.ts             # Add anomaly message formatters
src/lib/jobs/daily-recalculation.ts      # Integrate detection
.env.example                             # Add GVS_ANOMALY_THRESHOLD
```

### Anomaly Detection Logic

```typescript
// src/lib/gvs/anomaly-detection.ts

export interface GVSAnomaly {
  daoId: string
  daoName: string
  snapshotSpace: string
  previousScore: number
  currentScore: number
  delta: number
  direction: 'up' | 'down'
  detectedAt: Date
  threshold: number
}

export function detectAnomaly(
  daoId: string,
  daoName: string,
  snapshotSpace: string,
  currentScore: number,
  previousScore: number | null,
  threshold: number = 0.5
): GVSAnomaly | null {
  // No previous score = new DAO, not an anomaly
  if (previousScore === null) return null

  const delta = currentScore - previousScore

  // Check if change exceeds threshold
  if (Math.abs(delta) < threshold) return null

  return {
    daoId,
    daoName,
    snapshotSpace,
    previousScore,
    currentScore,
    delta,
    direction: delta > 0 ? 'up' : 'down',
    detectedAt: new Date(),
    threshold
  }
}
```

### Slack Message Format

```
⚠️ GVS Anomaly Alert

Significant score changes detected:

• Lido: 7.5 → 5.8 (-1.7) ⬇️
• Balancer: 5.0 → 5.8 (+0.8) ⬆️

Threshold: ±0.5 | Detected: 2026-01-22 00:05 UTC
Dashboard: https://chainsights.one/admin
```

### Email HTML Template

```html
<h2>⚠️ GVS Anomaly Alert</h2>
<p>The following DAOs had significant score changes:</p>
<table>
  <tr><th>DAO</th><th>Previous</th><th>Current</th><th>Change</th></tr>
  <tr><td>Lido</td><td>7.5</td><td>5.8</td><td style="color:red">-1.7</td></tr>
</table>
<p>Threshold: ±0.5</p>
<p><a href="https://chainsights.one/admin">View Dashboard</a></p>
```

### Storage Option Comparison

**Option A: Store in job_logs JSON**
- Pros: No schema change, keeps anomalies with job context
- Cons: Harder to query across jobs

**Option B: Dedicated gvs_anomalies table**
- Pros: Easy to query, separate from job logs
- Cons: Another table to maintain

**Recommendation:** Start with Option A (job_logs JSON), migrate to dedicated table if querying becomes complex.

### Environment Variables

```env
# Anomaly detection threshold (absolute score change)
# Default: 0.5 (detects changes > ±0.5 points)
GVS_ANOMALY_THRESHOLD=0.5
```

### Dependency on Story 12-1

This story requires Story 12-1 (daily job) to be completed first, as anomaly detection is integrated into the daily calculation job.

### Testing Strategy

1. **Unit Tests:**
   - `detectAnomaly()` with change above threshold → returns anomaly
   - `detectAnomaly()` with change below threshold → returns null
   - `detectAnomaly()` with no previous score → returns null
   - Message formatting functions

2. **Integration Tests:**
   - Full job with mock data triggering anomaly
   - Verify Slack webhook called (mock)
   - Verify email sent (mock)

### Example Scenario (Lido Case)

```
Previous snapshot: 2026-01-19, score: 7.5
Current calculation: 2026-01-22, score: 5.8
Delta: -1.7
Threshold: 0.5
Result: ANOMALY DETECTED ⚠️

Alert sent:
- Slack: "Lido: 7.5 → 5.8 (-1.7) ⚠️ SIGNIFICANT CHANGE"
- Email: Full HTML report
```

### References

- [Source: src/lib/monitoring/alerts.ts] - Existing Slack/email functions
- [Source: src/lib/jobs/daily-recalculation.ts] - Job structure to integrate with
- [Source: src/lib/gvs/storage.ts] - Delta calculation for comparison

## Dev Agent Record

### Context Reference

Story created from Epic 12: Daily GVS Monitoring & Historical Tracking.
Priority: URGENT - directly solves the Lido 7.5→5.8 undetected drop issue.

### Agent Model Used

Claude Opus 4.5 (claude-opus-4-5-20251101)

### Debug Log References

None

### Completion Notes List

- All 5 Acceptance Criteria implemented and verified
- Anomaly detection uses configurable threshold (default ±0.5)
- Detection integrated non-blocking into daily recalculation job
- Slack and email formatters create well-formatted alerts with direction indicators
- Storage uses Option A (job_logs JSON) per story recommendation
- Database-level time filtering for getRecentAnomalies() efficiency
- Dynamic URL configuration via NEXT_PUBLIC_BASE_URL environment variable
- 20 unit tests all passing
- Adversarial code review performed and all issues fixed

### File List

- `src/lib/gvs/anomaly-detection.ts` (NEW) - Anomaly detection logic, Slack/Email formatters
- `src/lib/jobs/daily-recalculation.ts` (MODIFIED) - Integrated anomaly detection into daily job
- `src/lib/jobs/job-logs.ts` (MODIFIED) - Added anomaly storage and getRecentAnomalies()
- `src/lib/db/schema.ts` (MODIFIED) - Added JobAnomalyEntry interface, anomalies column to jobLogs
- `drizzle/0013_add_anomalies_column.sql` (NEW) - Migration for anomalies JSONB column
- `.env.example` (MODIFIED) - Added GVS_ANOMALY_THRESHOLD config
- `tests/unit/gvs/anomaly-detection.test.ts` (NEW) - 20 unit tests for anomaly detection
