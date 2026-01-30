# Story 11.11: Webhook Health Monitoring

Status: done

## Story

**As a** operator
**I want** alerts if Coinbase webhooks stop working
**So that** I can investigate payment issues quickly

## Acceptance Criteria

1. - [x] System tracks last webhook timestamp per provider (via existing `getLastWebhookTimestamp()`)
2. - [x] Alert triggered if no webhook in 48h AND payments expected
3. - [x] Alert sent via existing notification channel (LogSnag + Slack)
4. - [x] Cron endpoint at `/api/cron/webhook-health` with CRON_SECRET auth
5. - [x] Monitoring process documented

## Tasks / Subtasks

- [x] Task 1: Create cron endpoint (AC: #4)
  - [x] 1.1 Create `src/app/api/cron/webhook-health/route.ts`
  - [x] 1.2 Add CRON_SECRET authorization check
  - [x] 1.3 Return JSON response with status

- [x] Task 2: Implement health check logic (AC: #1, #2)
  - [x] 2.1 Call `getLastWebhookTimestamp('coinbase')` from donations queries
  - [x] 2.2 Call `getLastWebhookTimestamp('stripe')` for comparison
  - [x] 2.3 Define STALE_THRESHOLD_HOURS = 48
  - [x] 2.4 Check if last webhook exceeds threshold
  - [x] 2.5 Skip alert if no donations exist yet (new system)

- [x] Task 3: Integrate with notification systems (AC: #3)
  - [x] 3.1 Send LogSnag alert via `logError()` when stale
  - [x] 3.2 Send Slack alert via `sendSlackAlert()` when stale
  - [x] 3.3 Include provider name and last timestamp in alert

- [x] Task 4: Write unit tests (AC: #1-4)
  - [x] 4.1 Test healthy state (recent webhook) returns success
  - [x] 4.2 Test stale webhook triggers alert
  - [x] 4.3 Test no donations (new system) skips alert
  - [x] 4.4 Test CRON_SECRET authorization
  - [x] 4.5 Test response format

- [x] Task 5: Document monitoring process (AC: #5)
  - [x] 5.1 Add Dev Notes section with monitoring documentation
  - [x] 5.2 Document Vercel cron schedule recommendation

## Dev Notes

### CRITICAL CONSTRAINT

> **ALL CODE CHANGES GO TO `develop` BRANCH ONLY!**

### Existing Infrastructure

The following already exists and should be reused:

**Donations Queries (`src/lib/donations/queries.ts`):**
```typescript
// Already implemented in Story 11-2
export async function getLastWebhookTimestamp(provider: string): Promise<Date | null>
```

**LogSnag (`src/lib/logsnag.ts`):**
```typescript
export const logError = (event: string, tags: Record<string, string | number>) =>
  trackEvent({ channel: 'errors', event, icon: '🚨', notify: true, tags })
```

**Slack Alerts (`src/lib/monitoring/alerts.ts`):**
```typescript
export async function sendSlackAlert(message: string, type: 'success' | 'alert' = 'alert'): Promise<boolean>
```

### Cron Pattern

Follow existing pattern from `src/app/api/cron/send-rankings-emails/route.ts`:

```typescript
// POST /api/cron/webhook-health
// Authorization: Bearer {CRON_SECRET}

export async function POST(request: NextRequest): Promise<NextResponse> {
  // Verify cron secret
  const authHeader = request.headers.get('authorization')
  const expectedAuth = `Bearer ${process.env.CRON_SECRET}`

  if (authHeader !== expectedAuth) {
    return NextResponse.json({ error: 'Unauthorized' }, { status: 401 })
  }

  // Health check logic here...
}
```

### Threshold Logic

```typescript
const STALE_THRESHOLD_HOURS = 48

const lastCoinbase = await getLastWebhookTimestamp('coinbase')
const lastStripe = await getLastWebhookTimestamp('stripe')

// If no donations exist yet, system is new - don't alert
if (!lastCoinbase && !lastStripe) {
  return { status: 'healthy', reason: 'No donations yet' }
}

// Check each provider
const now = new Date()
const thresholdMs = STALE_THRESHOLD_HOURS * 60 * 60 * 1000

if (lastCoinbase) {
  const ageMs = now.getTime() - lastCoinbase.getTime()
  if (ageMs > thresholdMs) {
    // Trigger alert for Coinbase
  }
}
```

### Alert Message Format

```
🚨 Coinbase webhook stale - Last received: {timestamp} ({hours}h ago)
```

### Vercel Cron Schedule

Recommend daily check at 9:00 AM UTC:

```json
// vercel.json
{
  "crons": [
    {
      "path": "/api/cron/webhook-health",
      "schedule": "0 9 * * *"
    }
  ]
}
```

### Response Format

```typescript
// Healthy response
{
  status: 'healthy',
  providers: {
    coinbase: { lastWebhook: '2026-01-21T10:00:00Z', ageHours: 2 },
    stripe: { lastWebhook: '2026-01-21T08:00:00Z', ageHours: 4 }
  }
}

// Alert triggered response
{
  status: 'alert',
  alerts: ['coinbase'],
  providers: {
    coinbase: { lastWebhook: '2026-01-19T10:00:00Z', ageHours: 50, stale: true },
    stripe: { lastWebhook: '2026-01-21T08:00:00Z', ageHours: 4, stale: false }
  }
}
```

### Files to Create

- `src/app/api/cron/webhook-health/route.ts` - Cron endpoint
- `tests/unit/api/cron/webhook-health.test.ts` - Unit tests

### Files to Modify

- `vercel.json` - Add cron schedule (optional, can be done manually)

### Dependencies

- **Story 11-2:** Donation queries library - DONE (provides `getLastWebhookTimestamp()`)
- **Story 11-4:** Coinbase webhook handler - DONE (creates donation records)

### References

- [Epic: Story 11](./epic-crypto-payments-donations.md#story-11-webhook-health-monitoring)
- [Existing Cron Pattern](../../src/app/api/cron/send-rankings-emails/route.ts)
- [LogSnag Integration](../../src/lib/logsnag.ts)
- [Monitoring Alerts](../../src/lib/monitoring/alerts.ts)

## Dev Agent Record

### Context Reference

Story created by create-story workflow with context from:
- Epic file (docs/sprint-artifacts/epic-crypto-payments-donations.md)
- Existing cron patterns (src/app/api/cron/send-rankings-emails/route.ts)
- Notification systems (src/lib/logsnag.ts, src/lib/monitoring/alerts.ts)
- Donations queries library (src/lib/donations/queries.ts)

### Agent Model Used

Claude Opus 4.5

### Debug Log References

### Completion Notes List

1. **TDD Approach**: Tests written first (RED), then implementation (GREEN)
2. **Threshold Logic**: STALE_THRESHOLD_HOURS = 48, uses strict `>` comparison for grace period at boundary
3. **Dual Notification**: Alerts sent to both LogSnag and Slack for redundancy
4. **Provider Independence**: Each provider (Coinbase, Stripe) checked independently
5. **New System Handling**: If no donations exist yet, returns healthy without alerting
6. **Response Format**: Includes provider details with lastWebhook timestamp and ageHours
7. **Unit Tests**: 11 tests covering authorization, healthy state, stale alerts, new system, response format

### File List

- `src/app/api/cron/webhook-health/route.ts` (new) - Cron endpoint for webhook health monitoring
- `tests/unit/api/cron/webhook-health.test.ts` (new) - 11 unit tests

## Senior Developer Review

**Review Date:** 2026-01-21
**Reviewer:** Claude Opus 4.5 (Adversarial Code Review)

### Issues Found

| Severity | Count | Status |
|----------|-------|--------|
| HIGH | 0 | - |
| MEDIUM | 0 | - |
| LOW | 1 | Noted |

### Issues Noted (Low Priority)

| Severity | Issue | Notes |
|----------|-------|-------|
| LOW | Missing test for 500 error case | Could add test for database failure scenario; not blocking |

### Verification

- All 11 unit tests passing
- Build verified successful
- No TypeScript errors
- Follows existing cron patterns

**Recommendation:** APPROVED - Story meets all acceptance criteria with clean implementation.

## Change Log

- 2026-01-21: Story created by Bob (Scrum Master) via autonomous create-story workflow
- 2026-01-21: Implementation completed by Dev Agent (Claude Opus 4.5) - all 11 tests passing, build verified
- 2026-01-21: Code review completed - no blocking issues, story approved and marked done
