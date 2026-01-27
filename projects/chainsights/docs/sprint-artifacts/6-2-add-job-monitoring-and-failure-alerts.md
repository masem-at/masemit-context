# Story 6.2: Add Job Monitoring and Alerts

**Status:** done
**Epic:** 6 - Weekly Automation & Historical Tracking
**Story ID:** 6.2
**Created:** 2025-12-23
**Updated:** 2025-12-23
**Ultimate Context Engine Analysis:** Complete

---

## Story

**As an** admin (Mario),
**I want to** be notified when the weekly job completes (success or failure) and view job execution history,
**So that** I can confirm rankings are updated and intervene quickly when needed.

---

## Acceptance Criteria

### Given-When-Then Format

**AC1: Job Log Database Table**
```
Given the weekly GVS recalculation job runs
When the job completes or fails
Then job execution is logged to database in `job_logs` table:
  - id (uuid, PK)
  - job_name (text): 'weekly-gvs-recalculation'
  - started_at (timestamp)
  - completed_at (timestamp, nullable)
  - status (enum): 'running' | 'completed' | 'failed'
  - daos_processed (integer)
  - daos_total (integer)
  - errors (jsonb, nullable): Array of error objects [{daoId, daoName, error}]
  - duration_ms (integer)
  - cache_invalidated (boolean)
```

**AC2: Job Logging Integration**
```
Given the executeWeeklyRecalculation() function runs
When the job starts
Then a new job_logs record is created with status='running'
When the job completes
Then the record is updated with final status, daos_processed, errors, duration_ms
```

**AC3: Success Notification - Email**
```
Given the weekly job completes successfully
When job status is 'completed' (≤50% failures)
Then an email notification is sent to hello@chainsights.one via Resend
And email includes: Success summary, DAOs processed count, duration, timestamp
And email has green/success styling
```

**AC4: Failure Notification - Email**
```
Given the weekly job fails (>50% DAOs failed OR fatal error)
When the job status is 'failed'
Then an email notification is sent to hello@chainsights.one via Resend
And email includes: Error summary, failed DAO count, timestamp, link to admin dashboard
And email has red/failure styling
```

**AC5: Failure Notification - Slack (Already Exists)**
```
Given the weekly job has >20% failure rate
When the job completes
Then the existing Slack alert is triggered (AC7 from Story 6.1)
And the alert includes DAO failure details
```

**AC6: Job History Query**
```
Given the admin dashboard needs job history
When the admin views the jobs section
Then the last 30 job executions can be retrieved via API
And results include all job_logs fields sorted by started_at DESC
```

**AC7: NFR-REL-13 Compliance**
```
Given the job monitoring system
When any job execution occurs
Then 100% of executions are logged to database
And no execution data is lost
And logs are queryable for admin dashboard (Epic 7)
```

---

## Critical Developer Context

### MISSION: Add Database Persistence for Job Logs

**CRITICAL CONTEXT**: Story 6.1 already implemented the weekly job with console logging and Slack alerts. This story adds **database persistence** for job logs and **email notifications** for failures.

**What Already Exists from Story 6.1 (DO NOT RECREATE):**
- `src/lib/jobs/weekly-recalculation.ts` - Full job orchestration
  - `executeWeeklyRecalculation()` - Main function
  - `JobExecutionLog` interface - Already defined (lines 30-38)
  - `JobResult` interface - Already defined (lines 43-51)
  - Console logging throughout
- `src/lib/monitoring/alerts.ts` - `sendSlackAlert()` for failure alerts
- `src/app/api/jobs/weekly-recalculation/route.ts` - Cron endpoint
- `vercel.json` - Cron schedule configuration
- Slack webhook integration (>20% failure rate)

**What You're Building (NEW):**
1. Database schema: `src/lib/db/schema.ts` - Add `job_logs` table
2. Job log service: `src/lib/jobs/job-logs.ts` - CRUD operations
3. Email alerting: `src/lib/monitoring/alerts.ts` - Add `sendEmailAlert()`
4. API endpoint: `src/app/api/admin/job-logs/route.ts` - Query job history
5. Integration: Update `weekly-recalculation.ts` to persist logs

---

### Existing Code to Modify

#### 1. Weekly Recalculation Service (Key Integration Point)
**File:** `src/lib/jobs/weekly-recalculation.ts`

The `JobExecutionLog` interface already exists (lines 30-38):
```typescript
export interface JobExecutionLog {
  jobName: 'weekly-gvs-recalculation'
  startedAt: Date
  completedAt: Date | null
  daosProcessed: number
  daosTotal: number
  failures: Array<{ daoId: string; daoName: string; error: string }>
  status: 'running' | 'completed' | 'failed'
}
```

**Key integration points:**
- Line 91-93: Job start logging → INSERT INTO job_logs
- Line 196-206: Job completion logging → UPDATE job_logs
- Line 229: Slack alert already sent → ADD email alert for 'failed' status

#### 2. Alerts Module
**File:** `src/lib/monitoring/alerts.ts`

Currently only has `sendSlackAlert()`. Add `sendEmailAlert()` using Resend SDK:
```typescript
import { Resend } from 'resend'

export async function sendEmailAlert(subject: string, html: string): Promise<boolean> {
  const resend = new Resend(process.env.RESEND_API_KEY)

  await resend.emails.send({
    from: process.env.EMAIL_FROM || 'ChainSights <hello@chainsights.one>',
    to: 'hello@chainsights.one',
    subject: `🚨 ChainSights Alert: ${subject}`,
    html,
  })
}
```

---

### Database Schema Design

**New Table: `job_logs`**
```typescript
// Add to src/lib/db/schema.ts

export const jobStatusEnum = pgEnum('job_status', ['running', 'completed', 'failed'])

export const jobLogs = pgTable('job_logs', {
  id: uuid('id').defaultRandom().primaryKey(),
  jobName: text('job_name').notNull(), // 'weekly-gvs-recalculation'
  startedAt: timestamp('started_at').notNull(),
  completedAt: timestamp('completed_at'),
  status: jobStatusEnum('status').default('running').notNull(),
  daosProcessed: integer('daos_processed').default(0).notNull(),
  daosTotal: integer('daos_total').default(0).notNull(),
  errors: jsonb('errors').$type<Array<{ daoId: string; daoName: string; error: string }>>(),
  durationMs: integer('duration_ms'),
  cacheInvalidated: boolean('cache_invalidated').default(false).notNull(),
  createdAt: timestamp('created_at').defaultNow().notNull(),
  updatedAt: timestamp('updated_at').defaultNow().notNull(),
}, (table) => ({
  jobNameIdx: index('job_logs_job_name_idx').on(table.jobName),
  startedAtIdx: index('job_logs_started_at_idx').on(table.startedAt.desc()),
}))

export type JobLog = typeof jobLogs.$inferSelect
export type JobLogInsert = typeof jobLogs.$inferInsert
```

---

## Implementation Tasks

### Task 1: Add job_logs Table to Schema
- [x] Open `src/lib/db/schema.ts`
- [x] Add `jobStatusEnum` enum: 'running', 'completed', 'failed'
- [x] Add `jobLogs` table with all fields from AC1
- [x] Add indexes for jobName and startedAt (DESC for recent queries)
- [x] Export `JobLog` and `JobLogInsert` types
- [x] Run `npm run db:generate` to create migration
- [x] Run `npm run db:push` to apply migration

### Task 2: Create Job Logs Service
- [x] Create `src/lib/jobs/job-logs.ts`
- [x] Implement `createJobLog(jobName: string): Promise<JobLog>` - Creates initial 'running' record
- [x] Implement `updateJobLog(id: string, result: JobResult): Promise<JobLog>` - Updates with final status
- [x] Implement `getRecentJobLogs(limit: number): Promise<JobLog[]>` - For admin dashboard
- [x] Implement `getJobLogById(id: string): Promise<JobLog | null>` - For detail view
- [x] Add proper error handling and logging

### Task 3: Add Email Alerting
- [x] Open `src/lib/monitoring/alerts.ts`
- [x] Add `sendEmailAlert(subject: string, html: string, type): Promise<boolean>` function
- [x] Add `generateJobSuccessEmailHtml()` for AC3 success notifications
- [x] Add `generateJobFailureEmailHtml()` for AC4 failure notifications
- [x] Use Resend SDK (already configured in project)
- [x] Email from: `process.env.EMAIL_FROM` (default: ChainSights <hello@chainsights.one>)
- [x] Email to: hello@chainsights.one
- [x] Handle missing RESEND_API_KEY gracefully (log and return false)
- [x] Add proper error handling

### Task 4: Integrate Logging into Weekly Job
- [x] Open `src/lib/jobs/weekly-recalculation.ts`
- [x] Import `createJobLog`, `updateJobLog` from job-logs.ts
- [x] At job start: Call `createJobLog('weekly-gvs-recalculation')`
- [x] Store the job log ID for later update
- [x] At job completion: Call `updateJobLog(jobLogId, result)`
- [x] At job failure: Call `sendEmailAlert()` with failure details
- [x] On success: Call `sendEmailAlert()` with success details (AC3)
- [x] Ensure log is updated even on fatal errors and empty DAO case

### Task 5: Create Admin API Endpoint
- [x] Create `src/app/api/admin/job-logs/route.ts`
- [x] Implement GET handler to fetch recent job logs
- [x] Add query param: `?limit=30` (default 30, max 100)
- [x] Add query param: `?id=UUID` for single job log
- [x] Add admin authentication (Bearer ADMIN_PASSWORD)
- [x] Return JSON: `{ data: JobLog[], meta: { count, limit } }`

### Task 6: Write Tests
- [x] Create `tests/unit/jobs/job-logs.test.ts`
- [x] Test createJobLog creates record with 'running' status
- [x] Test updateJobLog updates all fields correctly
- [x] Test getRecentJobLogs returns correct order and limit
- [x] Test generateJobFailureEmailHtml generates valid HTML
- [x] Test generateJobSuccessEmailHtml generates valid HTML
- [x] Test sendEmailAlert handles missing API key
- [x] Mock database and Resend SDK
- [x] Update `tests/unit/jobs/weekly-recalculation.test.ts` with new mocks

### Task 7: Verify and Complete
- [x] Run `npm run build` - verify no TypeScript errors
- [x] Run `npm run lint` - verify no linting errors
- [x] Run tests: `npm run test:unit -- --run tests/unit/jobs/`
- [x] Code review fixes applied
- [x] Update this story status to "review"
- [x] Update sprint-status.yaml

---

## Technical Requirements

### File Structure
```
src/
├── app/
│   └── api/
│       └── admin/
│           └── job-logs/
│               └── route.ts           # NEW: Admin endpoint for job history
├── lib/
│   ├── db/
│   │   └── schema.ts                  # MODIFY: Add job_logs table
│   ├── jobs/
│   │   ├── weekly-recalculation.ts    # MODIFY: Add log persistence
│   │   └── job-logs.ts                # NEW: Job log CRUD operations
│   └── monitoring/
│       └── alerts.ts                  # MODIFY: Add email alerting
tests/
└── unit/
    └── jobs/
        └── job-logs.test.ts           # NEW: Unit tests
```

### API Contracts

**GET /api/admin/job-logs**
```typescript
// Headers
Authorization: Bearer ${ADMIN_PASSWORD}

// Query Params
?limit=30  // Optional, default 30

// Response (Success)
{
  "data": [
    {
      "id": "uuid",
      "jobName": "weekly-gvs-recalculation",
      "startedAt": "2025-12-22T00:00:00.000Z",
      "completedAt": "2025-12-22T00:05:32.000Z",
      "status": "completed",
      "daosProcessed": 10,
      "daosTotal": 10,
      "errors": null,
      "durationMs": 332000,
      "cacheInvalidated": true,
      "createdAt": "2025-12-22T00:00:00.000Z",
      "updatedAt": "2025-12-22T00:05:32.000Z"
    }
  ]
}

// Response (Unauthorized)
{
  "error": {
    "message": "Unauthorized",
    "code": "UNAUTHORIZED",
    "statusCode": 401
  }
}
```

### Email Template for Failure Alert
```html
<h1>🚨 Weekly GVS Recalculation Failed</h1>
<p><strong>Status:</strong> Failed</p>
<p><strong>Timestamp:</strong> ${completedAt}</p>
<p><strong>Duration:</strong> ${durationMs}ms</p>
<p><strong>DAOs Processed:</strong> ${daosProcessed}/${daosTotal}</p>
<p><strong>Failures:</strong> ${failures.length}</p>

<h2>Failed DAOs:</h2>
<ul>
  ${failures.map(f => `<li>${f.daoName}: ${f.error}</li>`).join('')}
</ul>

<p><a href="https://chainsights.one/admin/jobs">View in Admin Dashboard</a></p>
```

---

## Previous Story Intelligence (6.1)

**Key Learnings from Story 6.1:**
1. `JobExecutionLog` interface already defined - reuse it
2. Console logging structure already in place - mirror for DB
3. Slack alerting works (>20% threshold) - email adds 'failed' status alert
4. `JobResult` type has all needed fields for DB persistence
5. Error handling is robust with try-catch - maintain pattern

**Code Patterns from 6.1:**
- Use Drizzle ORM for all database operations
- Return structured results with status field
- Log errors but don't throw (graceful degradation)
- Use environment variables for optional features

**Production Verification (6.1):**
- Job completed successfully: 10/10 DAOs in ~8 seconds
- Slack integration tested and working
- ISR cache invalidation confirmed

---

## Definition of Done

- [ ] job_logs table created and migrated to database
- [ ] Job log service with CRUD operations implemented
- [ ] Email alerting added for failed jobs
- [ ] Weekly recalculation job persists all executions to DB
- [ ] Admin API endpoint returns job history
- [ ] Unit tests for job-logs service
- [ ] All existing 6.1 tests still pass
- [ ] Build and lint pass
- [ ] Ready for Epic 7 (Admin Dashboard) to display job logs

---

## Dev Agent Record

### Context Reference
<!-- Ultimate Context Engine analysis completed -->

### Agent Model Used
Claude Opus 4.5

### Debug Log References
- None

### Completion Notes List
- 2025-12-23: Story 6.2 created with Ultimate Context Engine analysis
- Story 6.1 (dependency) is complete and verified in production
- Existing `JobExecutionLog` and `JobResult` interfaces reused
- Minimal changes to working 6.1 code - adds persistence layer
- Email alerting via Resend (already configured in project)

### File List

**Files Created:**
- `src/lib/jobs/job-logs.ts` - Job log CRUD service
- `src/app/api/admin/job-logs/route.ts` - Admin API endpoint
- `tests/unit/jobs/job-logs.test.ts` - Unit tests
- `drizzle/0006_majestic_dracula.sql` - Migration file (auto-generated)

**Files Modified:**
- `src/lib/db/schema.ts` - Add job_logs table, jobStatusEnum, and types
- `src/lib/monitoring/alerts.ts` - Add sendEmailAlert(), generateJobSuccessEmailHtml(), generateJobFailureEmailHtml()
- `src/lib/jobs/weekly-recalculation.ts` - Integrate log persistence and email notifications
- `tests/unit/jobs/weekly-recalculation.test.ts` - Add mocks for new alert functions

---

## References

**Story 6.1:** `docs/sprint-artifacts/6-1-implement-weekly-gvs-recalculation-cron-job.md` - Foundation for this story
**Epic 6 Definition:** `docs/epics.md` - Weekly Automation & Historical Tracking
**Architecture:** `docs/architecture.md` - Database patterns, job scheduling
**Project Context:** `docs/project_context.md` - Coding standards, Drizzle patterns
**Resend Docs:** https://resend.com/docs/send-with-nodejs
**NFR-REL-13:** Zero job execution data loss requirement

---

## Change Log

- **2025-12-23**: Story 6.2 created with Ultimate Context Engine analysis
- Story builds directly on completed Story 6.1 implementation
- Reuses existing interfaces and patterns for minimal change
- Adds database persistence for admin dashboard (Epic 7) enablement
