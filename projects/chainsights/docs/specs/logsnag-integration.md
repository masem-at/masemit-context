# LogSnag Integration — Technical Specification

**Author:** Winston (Architect)
**Status:** Ready for Implementation
**Priority:** High — Visibility into production is critical before launch

---

## Overview

LogSnag provides real-time event tracking with push notifications. For ChainSights, it replaces `console.log` statements with structured, persistent events you can monitor from your phone.

**Why LogSnag over alternatives:**
- Simple API (one POST per event)
- Push notifications to mobile/desktop
- No complex dashboards to configure
- €12/month for Solo plan (plenty for launch)
- Zero infrastructure to maintain

---

## Architecture

```
┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐
│   API Routes    │     │   LogSnag Lib   │     │   LogSnag API   │
│                 │ ──► │   src/lib/      │ ──► │   logsnag.com   │
│   (events)      │     │   logsnag.ts    │     │                 │
└─────────────────┘     └─────────────────┘     └─────────────────┘
                                                        │
                                                        ▼
                                               ┌─────────────────┐
                                               │  Push Notif +   │
                                               │  Dashboard      │
                                               └─────────────────┘
```

---

## Channels

LogSnag uses "channels" to organize events. Keep it simple:

| Channel | Purpose | Notifications |
|---------|---------|---------------|
| `revenue` | Payments, orders | Always |
| `pipeline` | Report processing stages | On failure only |
| `growth` | Share claims, referrals | Always |
| `errors` | All failures | Always |

---

## Events to Track

### 1. Revenue Events (Channel: `revenue`)

#### `order.created`
Trigger: Stripe webhook `checkout.session.completed`

```typescript
{
  event: "Order Created",
  channel: "revenue",
  icon: "💰",
  notify: true,
  tags: {
    email: "customer@example.com",
    tier: "standard", // or "deep_dive"
    amount: "€99",
  }
}
```

---

### 2. Pipeline Events (Channel: `pipeline`)

#### `report.intake`
Trigger: Customer submits intake form

```typescript
{
  event: "Report Intake",
  channel: "pipeline",
  icon: "📋",
  notify: false,
  tags: {
    dao: "Optimism",
    space: "opcollective.eth",
    email: "customer@example.com",
  }
}
```

#### `report.fetch_started`
Trigger: Snapshot data fetch begins

```typescript
{
  event: "Fetch Started",
  channel: "pipeline",
  icon: "🔄",
  notify: false,
  tags: {
    dao: "Optimism",
    report_id: "uuid",
  }
}
```

#### `report.fetch_complete`
Trigger: Snapshot data fetched successfully

```typescript
{
  event: "Fetch Complete",
  channel: "pipeline",
  icon: "✅",
  notify: false,
  tags: {
    dao: "Optimism",
    proposals: "47",
    votes: "12,453",
  }
}
```

#### `report.analyzing`
Trigger: AI analysis begins

```typescript
{
  event: "AI Analysis Started",
  channel: "pipeline",
  icon: "🤖",
  notify: false,
  tags: {
    dao: "Optimism",
  }
}
```

#### `report.review_pending`
Trigger: PDF generated, ready for review

```typescript
{
  event: "Ready for Review",
  channel: "pipeline",
  icon: "👀",
  notify: true,  // You want to know when reports need attention
  tags: {
    dao: "Optimism",
    version: "1",
  }
}
```

#### `report.declined`
Trigger: Admin declines report

```typescript
{
  event: "Report Declined",
  channel: "pipeline",
  icon: "↩️",
  notify: true,
  tags: {
    dao: "Optimism",
    version: "2",
    feedback: "Score too high",
  }
}
```

#### `report.sent`
Trigger: Report delivered to customer

```typescript
{
  event: "Report Delivered",
  channel: "pipeline",
  icon: "📧",
  notify: true,
  tags: {
    dao: "Optimism",
    email: "customer@example.com",
    version: "1",
  }
}
```

---

### 3. Growth Events (Channel: `growth`)

#### `share.claimed`
Trigger: Customer submits Share & Save claim

```typescript
{
  event: "Share Claim Submitted",
  channel: "growth",
  icon: "🐦",
  notify: true,
  tags: {
    handle: "@username",
    dao: "Optimism",
  }
}
```

#### `share.verified`
Trigger: Admin verifies and pays claim

```typescript
{
  event: "Share Claim Paid",
  channel: "growth",
  icon: "💸",
  notify: true,
  tags: {
    handle: "@username",
    amount: "€20",
  }
}
```

---

### 4. Error Events (Channel: `errors`)

#### `pipeline.failed`
Trigger: Any pipeline stage fails

```typescript
{
  event: "Pipeline Failed",
  channel: "errors",
  icon: "🚨",
  notify: true,
  tags: {
    dao: "Optimism",
    stage: "analyzing",
    error: "AI timeout after 30s",
  }
}
```

#### `webhook.failed`
Trigger: Stripe webhook processing fails

```typescript
{
  event: "Webhook Error",
  channel: "errors",
  icon: "⚠️",
  notify: true,
  tags: {
    type: "checkout.session.completed",
    error: "Database write failed",
  }
}
```

---

## Implementation

### File: `src/lib/logsnag.ts`

```typescript
const LOGSNAG_TOKEN = process.env.LOGSNAG_TOKEN
const LOGSNAG_PROJECT = process.env.LOGSNAG_PROJECT || 'chainsights'

interface LogSnagEvent {
  channel: 'revenue' | 'pipeline' | 'growth' | 'errors'
  event: string
  icon?: string
  notify?: boolean
  tags?: Record<string, string | number>
  description?: string
}

export function isLogSnagConfigured(): boolean {
  return Boolean(LOGSNAG_TOKEN)
}

export async function trackEvent(data: LogSnagEvent): Promise<void> {
  if (!LOGSNAG_TOKEN) {
    // Silent fail in development
    console.log(`[LogSnag] ${data.channel}/${data.event}`, data.tags)
    return
  }

  try {
    await fetch('https://api.logsnag.com/v1/log', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'Authorization': `Bearer ${LOGSNAG_TOKEN}`,
      },
      body: JSON.stringify({
        project: LOGSNAG_PROJECT,
        channel: data.channel,
        event: data.event,
        icon: data.icon || '📌',
        notify: data.notify ?? false,
        tags: data.tags,
        description: data.description,
      }),
    })
  } catch (error) {
    // Never throw on tracking failure
    console.error('[LogSnag] Failed to track:', error)
  }
}

// Convenience functions
export const logRevenue = (event: string, tags: Record<string, string | number>) =>
  trackEvent({ channel: 'revenue', event, icon: '💰', notify: true, tags })

export const logPipeline = (event: string, tags: Record<string, string | number>, notify = false) =>
  trackEvent({ channel: 'pipeline', event, tags, notify })

export const logGrowth = (event: string, tags: Record<string, string | number>) =>
  trackEvent({ channel: 'growth', event, icon: '📈', notify: true, tags })

export const logError = (event: string, tags: Record<string, string | number>) =>
  trackEvent({ channel: 'errors', event, icon: '🚨', notify: true, tags })
```

---

## Integration Points

### 1. Stripe Webhook (`src/app/api/webhooks/stripe/route.ts`)

```typescript
// After order created successfully
await logRevenue('Order Created', {
  email: customerEmail,
  tier,
  amount: `€${(session.amount_total || 0) / 100}`,
})
```

### 2. Analyze Job (`src/app/api/jobs/analyze/route.ts`)

```typescript
// When analysis starts
await logPipeline('AI Analysis Started', { dao: report.daoName })

// When ready for review
await logPipeline('Ready for Review', {
  dao: report.daoName,
  version: String(report.version),
}, true) // notify = true

// On failure
await logError('Pipeline Failed', {
  dao: report.daoName,
  stage: 'analyzing',
  error: error.message,
})
```

### 3. Send Action (`src/app/api/admin/reports/[id]/send/route.ts`)

```typescript
// After successful send
await logPipeline('Report Delivered', {
  dao: report.daoName,
  email: report.customerEmail,
  version: String(report.version),
}, true)
```

### 4. Share Claim (`src/app/api/share-claim/route.ts`)

```typescript
// After claim submitted
await logGrowth('Share Claim Submitted', {
  handle: twitterHandle || 'unknown',
  dao: report.daoName,
})
```

---

## Environment Variables

```bash
# .env.local
LOGSNAG_TOKEN=your_api_token_here
LOGSNAG_PROJECT=chainsights
```

Add to `.env.example`:
```bash
# LogSnag - Event tracking (https://logsnag.com)
LOGSNAG_TOKEN=
LOGSNAG_PROJECT=chainsights
```

---

## LogSnag Setup Steps

1. Create account at https://logsnag.com
2. Create project "chainsights"
3. Create channels: `revenue`, `pipeline`, `growth`, `errors`
4. Get API token from Settings → API
5. Add to Vercel environment variables
6. Install mobile app for push notifications

---

## What You'll See

After implementation, your LogSnag dashboard shows:

**Revenue Channel:**
```
💰 Order Created
   email: mario@example.com
   tier: deep_dive
   amount: €199
   2 minutes ago
```

**Pipeline Channel:**
```
👀 Ready for Review
   dao: Optimism
   version: 1
   5 minutes ago

📧 Report Delivered
   dao: ENS
   email: customer@example.com
   1 hour ago
```

**Growth Channel:**
```
🐦 Share Claim Submitted
   handle: @daotrader
   dao: Optimism
   30 minutes ago
```

---

## Not In Scope (For Later)

- **Insights**: LogSnag can track metrics (daily revenue, reports sent). Add after launch if needed.
- **User identification**: Could link events to user IDs. Not needed yet.
- **Detailed analytics**: For deep analysis, export to a proper data warehouse later.

---

## Implementation Estimate

| Task | Complexity |
|------|------------|
| Create `src/lib/logsnag.ts` | 15 min |
| Add to Stripe webhook | 5 min |
| Add to pipeline jobs | 15 min |
| Add to share-claim | 5 min |
| Environment setup | 5 min |
| **Total** | ~45 min |

---

## Success Criteria

After implementation, you can:
1. Get push notification when order comes in
2. See real-time pipeline progress
3. Know immediately when something fails
4. Track Share & Save claims without checking admin panel

---

*"Observability isn't optional. If you can't see what's happening in production, you're flying blind."*

— Winston
