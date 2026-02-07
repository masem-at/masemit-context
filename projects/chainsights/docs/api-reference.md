# ChainSights API Reference

> Auto-generated from source code scan on 2026-02-07.

---

## Table of Contents

- [Public API Routes](#public-api-routes)
- [Auth Routes](#auth-routes)
- [Matrix Routes](#matrix-routes)
- [Payment & Billing Routes](#payment--billing-routes)
- [Webhook Routes](#webhook-routes)
- [Admin Routes](#admin-routes)
- [Job / Cron Routes](#job--cron-routes)
- [GDPR Routes](#gdpr-routes)

---

## Public API Routes

### POST /api/quick-check

Free governance health check for any DAO. Three response paths: (A) known DAO with immediate JSON, (B) cached on-the-fly result from last 24h, (C) unknown DAO with SSE stream and on-the-fly GVS calculation.

| Field | Details |
|---|---|
| **Auth** | None (session optional -- logged-in users bypass IP rate limit) |
| **Rate Limit** | 10 req/IP/hour (anonymous), 1 check per email+DAO per 30 days, 5 on-the-fly checks per email per day |

**Request Body:**

```json
{
  "email": "string (required, or falls back to session email)",
  "snapshot_space": "string (required, e.g. 'ens.eth')",
  "marketing_optin": "boolean (optional)"
}
```

**Response (Path A/B -- JSON):**

```json
{
  "success": true,
  "data": {
    "dao_name": "string",
    "gvs_score": "number (0-100)",
    "gvs_label": "Vital | Stable | Concerning | Critical",
    "hpr_score": "number | null",
    "dei_score": "number | null",
    "pdi_score": "number | null",
    "gpi_score": "number | null",
    "category": "string",
    "category_comparison": "string",
    "data_source": "rankings | cached_on_the_fly | reports"
  }
}
```

**Response (Path C -- SSE stream):** Progressive `data:` events with steps: `validating`, `counting`, `calculating`, `scoring`, `done` (or `error`).

---

### POST /api/feedback

Submit anonymous user feedback. Rate limited, sends email notification.

| Field | Details |
|---|---|
| **Auth** | None |
| **Rate Limit** | 10 per IP per hour |

**Request Body:**

```json
{
  "message": "string (1-500 chars, required)",
  "page": "string (max 200 chars, required)",
  "placeholderShown": "string (optional, max 200 chars)"
}
```

**Response:** `{ "success": true }`

---

### GET /api/stats

Public aggregate stats for social proof (risks identified, voting power analyzed, DAOs analyzed). Cached in-memory for 1 hour.

| Field | Details |
|---|---|
| **Auth** | None |
| **Cache** | `public, max-age=3600` |

**Response:**

```json
{
  "risksIdentified": "number",
  "votingPowerAnalyzed": "number",
  "daosAnalyzed": "number",
  "lastUpdated": "ISO8601"
}
```

---

### GET /api/stats/health-checks

Returns count of free health checks performed, for social proof display.

| Field | Details |
|---|---|
| **Auth** | None |

**Response:**

```json
{
  "count": "number",
  "display": "string | null (e.g. '1200+')",
  "timestamp": "ISO8601"
}
```

---

### POST /api/share-claim

Submit a share-and-save reward claim after sharing a report on X/Twitter.

| Field | Details |
|---|---|
| **Auth** | None (verified by email + order lookup) |

**Request Body:**

```json
{
  "email": "string (required)",
  "tweetUrl": "string (required, valid X/Twitter status URL)"
}
```

**Response:**

```json
{
  "success": true,
  "message": "string",
  "claimId": "string"
}
```

---

### POST /api/quiz-response

Store product quiz answers and recommended tier. Silently succeeds if DB is unavailable.

| Field | Details |
|---|---|
| **Auth** | None |

**Request Body:**

```json
{
  "answers": "object (required)",
  "recommendedTier": "string (optional)",
  "referrer": "string (optional)"
}
```

**Response:** `{ "success": true }`

---

### POST /api/subscribe

Subscribe to Rankings Watch email notifications (double opt-in).

| Field | Details |
|---|---|
| **Auth** | None |

**Request Body:**

```json
{
  "email": "string (valid email, required)"
}
```

**Response:**

```json
{
  "success": true,
  "message": "Check your email to confirm subscription"
}
```

---

### GET /api/subscribe/confirm

Confirm an email subscription via token link. Redirects to `/rankings`.

| Field | Details |
|---|---|
| **Auth** | None (token-based) |

**Query Params:** `?token=xxx`

**Response:** Redirect to `/rankings?message=confirmed`

---

### GET /api/subscribe/unsubscribe

Unsubscribe from email notifications. Redirects to `/rankings`.

| Field | Details |
|---|---|
| **Auth** | None (token-based) |

**Query Params:** `?token=xxx`

**Response:** Redirect to `/rankings?message=unsubscribed`

---

### POST /api/rankings/opt-out

Submit a DAO opt-out request from public rankings.

| Field | Details |
|---|---|
| **Auth** | None |

**Request Body:**

```json
{
  "daoName": "string (required)",
  "contactEmail": "string (required)",
  "reason": "string (optional)"
}
```

**Response:**

```json
{
  "success": true,
  "message": "Opt-out request submitted successfully",
  "requestId": "string",
  "emailSent": "boolean"
}
```

---

### GET /api/donations/total

Returns cumulative donation total for impact page display.

| Field | Details |
|---|---|
| **Auth** | None |
| **Cache** | `public, s-maxage=300, stale-while-revalidate=60` |

**Query Params:** `?project=chainsights` (optional, defaults to `chainsights`)

**Response:**

```json
{
  "total": "number",
  "count": "number",
  "currency": "EUR",
  "lastUpdated": "ISO8601"
}
```

---

### GET /api/download/[token]

Download a PDF or JSON governance report via secure time-limited token. No authentication required.

| Field | Details |
|---|---|
| **Auth** | None (token-based, 30-day expiry) |

**Response:** PDF binary (Content-Type: `application/pdf`) or JSON file, depending on token type. Returns `410 Gone` if token expired.

---

### POST /api/reports/free-rating

Legacy free governance rating endpoint. Looks up GVS scores for a known DAO. Supports anonymous and identified requests with per-identifier monthly quota.

| Field | Details |
|---|---|
| **Auth** | None |
| **Rate Limit** | 1 per identifier per month |

**Request Body:**

```json
{
  "dao": "string (snapshot space ID, required)",
  "anonymous": "boolean (optional)",
  "identifier": "string (email or wallet, required if not anonymous)",
  "identifierType": "'email' | 'wallet'",
  "consentGiven": "boolean",
  "consentTimestamp": "string",
  "marketingOptIn": "boolean (optional)",
  "publicListing": "boolean (optional)"
}
```

**Response:**

```json
{
  "success": true,
  "rating": {
    "score": "number",
    "giniCoefficient": "number",
    "nakamotoCoefficient": "number",
    "ranking": "number",
    "totalDaos": "number"
  },
  "quotaRemaining": 0,
  "nextResetDate": "ISO8601"
}
```

---

### GET /api/dao/snapshot-available

Check if a DAO has Snapshot governance data available (rankings, reports, or live Snapshot API).

| Field | Details |
|---|---|
| **Auth** | None |

**Query Params:** `?dao=ens.eth` (required)

**Response:**

```json
{
  "available": true,
  "source": "rankings | reports | snapshot_api",
  "lastSnapshot": "ISO8601 | null",
  "dataQuality": "good | limited | on_the_fly"
}
```

---

### POST /api/account-lookup

Request an account management link by email. Always returns same response to prevent email enumeration.

| Field | Details |
|---|---|
| **Auth** | None |
| **Rate Limit** | 3 per email per hour |

**Request Body:**

```json
{
  "email": "string (valid email, required)"
}
```

**Response:** `{ "message": "If an account exists for this email, we've sent you a link to manage your subscription." }`

---

### POST /api/intake

Submit the DAO intake form after a purchase. Links order to a new report and triggers the processing pipeline via QStash.

| Field | Details |
|---|---|
| **Auth** | None (linked via Stripe session ID) |

**Request Body:**

```json
{
  "sessionId": "string (Stripe checkout session ID)",
  "daoName": "string (required)",
  "snapshotSpace": "string (required)",
  "chain": "string (optional, default: 'ethereum')",
  "email": "string (required)",
  "additionalContext": "string (optional)"
}
```

**Response:**

```json
{
  "success": true,
  "message": "Intake form submitted successfully",
  "reportId": "string",
  "orderId": "string"
}
```

---

### POST /api/revalidate

On-demand ISR (Incremental Static Regeneration) cache invalidation. Called internally by cron jobs.

| Field | Details |
|---|---|
| **Auth** | Bearer token (`REVALIDATE_SECRET`) |

**Query Params:** `?path=/rankings` (required)

**Response:**

```json
{
  "revalidated": true,
  "path": "string",
  "timestamp": "ISO8601"
}
```

---

### GET /api/dgi

Returns current DGI (Decentralized Governance Index) values. Optionally includes 30-day history.

| Field | Details |
|---|---|
| **Auth** | None |
| **Feature Flag** | Only available when DGI feature flag is enabled |

**Query Params:** `?history=true` (optional)

**Response:**

```json
{
  "current": { "composite": "number", "categories": {}, "components": {} },
  "history": "array (if ?history=true)"
}
```

---

## Auth Routes

### POST /api/auth/request

Request a magic link for passwordless authentication. Always returns `{ success: true }` to prevent email enumeration.

| Field | Details |
|---|---|
| **Auth** | None |
| **Rate Limit** | Built-in via `requestMagicLink` |

**Request Body:**

```json
{
  "email": "string (valid email, required)",
  "redirectUrl": "string (same-origin URL, required)",
  "marketingOptIn": "boolean (required)"
}
```

**Response:** `{ "success": true }`

---

### GET /api/auth/verify

Verify a magic link token, create a session cookie, and redirect. Called when user clicks the magic link in their email.

| Field | Details |
|---|---|
| **Auth** | None (token-based) |

**Query Params:** `?token=xxx`

**Response:** Redirect to original page (or `/matrix` as fallback). Sets `cs_session` cookie.

---

### POST /api/auth/logout

Clear the session cookie.

| Field | Details |
|---|---|
| **Auth** | None (clears existing session) |

**Response:** `{ "success": true }`

---

### GET /api/auth/session

Check current authentication state from the `cs_session` cookie. Safe for client-side polling.

| Field | Details |
|---|---|
| **Auth** | None (reads cookie) |
| **Cache** | `no-store, no-cache, must-revalidate` |

**Response:**

```json
{
  "authenticated": true,
  "email": "string",
  "role": "admin | free | subscriber"
}
```

or `{ "authenticated": false }`

---

## Matrix Routes

### GET /api/matrix

Returns DAO data for the Matrix table with access-level enforcement.

| Field | Details |
|---|---|
| **Auth** | Optional (email query param for access resolution) |

**Query Params:**

| Param | Description |
|---|---|
| `email` | User email for access check |
| `category` | Filter: `all`, `defi`, `infrastructure`, `public_goods`, `social` |
| `sortBy` | `rank`, `score`, `change`, `category` |
| `sortOrder` | `asc`, `desc` |
| `search` | Search by name or snapshot space |

**Response:**

```json
{
  "daos": [],
  "meta": {
    "access": "anonymous | free | subscriber | admin",
    "totalCount": "number",
    "visibleCount": "number",
    "hiddenCount": "number",
    "canExport": "boolean",
    "canViewHistorical": "boolean"
  }
}
```

---

### POST /api/matrix/subscribe

Create a Stripe Checkout session for a Matrix subscription.

| Field | Details |
|---|---|
| **Auth** | None |

**Request Body:**

```json
{
  "plan": "'matrix_monthly' | 'matrix_yearly' (required)",
  "email": "string (optional, pre-fills checkout)"
}
```

**Response:**

```json
{
  "url": "string (Stripe checkout URL)",
  "sessionId": "string"
}
```

---

### GET /api/matrix/export

Export Matrix data as CSV or JSON. Subscriber/admin only.

| Field | Details |
|---|---|
| **Auth** | Required (subscriber or admin, via email header or query param) |

**Query Params:**

| Param | Description |
|---|---|
| `email` | User email (or `x-user-email` header) |
| `format` | `json` (default) or `csv` |

**Response:** CSV or JSON file download with `Content-Disposition` header.

---

## Payment & Billing Routes

### POST /api/checkout

Create an embedded Stripe Checkout session for a one-time report purchase (Deep Dive or Governance Audit).

| Field | Details |
|---|---|
| **Auth** | None |

**Request Body:**

```json
{
  "tier": "'deep_dive' | 'governance_audit' (optional, default: 'deep_dive')",
  "daoName": "string (optional, for prefill)",
  "daoSlug": "string (optional)",
  "snapshotSpace": "string (optional)"
}
```

**Response:** `{ "clientSecret": "string (Stripe embedded checkout client secret)" }`

---

### POST /api/crypto-checkout

Create a Coinbase Commerce charge for crypto payment.

| Field | Details |
|---|---|
| **Auth** | None |

**Request Body:**

```json
{
  "orderId": "string (required)",
  "orderTotal": "number (positive, required)",
  "customerEmail": "string (required)",
  "daoName": "string (required)"
}
```

**Response:**

```json
{
  "chargeId": "string",
  "checkoutUrl": "string"
}
```

---

### POST /api/billing-portal

Create a Stripe Billing Portal session for invoice/subscription management.

| Field | Details |
|---|---|
| **Auth** | None (requires valid Stripe customer ID) |

**Request Body:**

```json
{
  "stripeCustomerId": "string (required)"
}
```

**Response:** `{ "url": "string (Stripe portal URL)" }`

---

## Webhook Routes

### POST /api/webhooks/stripe

Stripe webhook handler. Processes checkout completions, payment intents, subscription lifecycle events, and invoice payments. Verifies signature via `stripe-signature` header.

| Field | Details |
|---|---|
| **Auth** | Stripe webhook signature (`STRIPE_WEBHOOK_SECRET`) |

**Handled Events:**

| Event | Action |
|---|---|
| `checkout.session.completed` | Creates order record, tracks revenue |
| `payment_intent.succeeded` | Records donation (3% or 5% with CHOKIHEART promo) |
| `payment_intent.payment_failed` | Logs failure |
| `customer.subscription.created` | Upserts subscription record |
| `customer.subscription.updated` | Updates subscription status |
| `customer.subscription.deleted` | Marks subscription canceled |
| `invoice.paid` | Records donation for subscription renewals |

**Response:** `{ "received": true }`

---

### POST /api/webhooks/coinbase

Coinbase Commerce webhook handler. Processes confirmed crypto payments and records donations. Verifies signature via `x-cc-webhook-signature` header.

| Field | Details |
|---|---|
| **Auth** | Coinbase webhook signature (`COINBASE_COMMERCE_WEBHOOK_SECRET`) |

**Handled Events:**

| Event | Action |
|---|---|
| `charge:confirmed` | Records donation for ChainSights payments |
| `charge:failed` | Logs failure |
| `charge:expired` | Logs expiry |

**Response:** `{ "received": true }`

---

## Admin Routes

All admin routes are protected by `isAdminAuthenticated()` which verifies a magic link session with `role: 'admin'`.

### GET /api/admin/reports/[id]

Fetch a single report by ID.

**Response:** Full report object (JSON).

---

### PATCH /api/admin/reports/[id]

Update report draft content.

**Request Body:**

```json
{
  "draftContent": "object"
}
```

**Response:** Updated report object.

---

### POST /api/admin/reports/[id]/fetch

Fetch Snapshot governance data for a report. Updates report status through `fetching` -> `pending`.

**Response:** Updated report object with `rawData` populated.

---

### POST /api/admin/reports/[id]/analyze

Run AI analysis (Anthropic Claude) on fetched governance data. Updates status through `analyzing` -> `draft`.

**Response:** Updated report with `aiAnalysis` and `draftContent`.

---

### POST /api/admin/reports/[id]/approve

Approve a report for delivery. Requires PDF to already be generated.

**Response:** Updated report with status `approved`.

---

### POST /api/admin/reports/[id]/decline

Decline a report with feedback. Re-runs AI analysis with feedback injection, generates new PDF. Max 3 iterations.

**Request Body:**

```json
{
  "templates": ["string array of feedback template IDs"],
  "freeText": "string (free-form feedback)"
}
```

**Response:**

```json
{
  "success": true,
  "report": {},
  "version": "number",
  "remainingIterations": "number"
}
```

---

### POST /api/admin/reports/[id]/send

Send report delivery email with secure download links. Schedules share reminder via QStash.

**Response:** Updated report with status `sent` and `deliveredAt` timestamp.

---

### GET /api/admin/reported-daos

List all reported DAOs (from free checks and paid reports). Returns up to 50 entries.

**Response:** `{ "daos": [] }`

---

### POST /api/admin/test-report

Create a test report (no payment) and trigger the processing pipeline.

**Request Body:**

```json
{
  "daoName": "string (required)",
  "snapshotSpace": "string (required)",
  "chain": "string (optional)",
  "email": "string (optional, default: contact@masem.at)",
  "tier": "'deep_dive' | 'governance_audit' (optional)"
}
```

**Response:**

```json
{
  "success": true,
  "reportId": "string",
  "orderId": "string",
  "spaceName": "string"
}
```

---

### GET /api/admin/daos

List all non-opted-out DAOs with DGI membership status.

**Response:** `{ "daos": [{ id, name, slug, snapshotSpace, category, isDgiMember, ... }] }`

---

### POST /api/admin/daos

Bulk-add DAOs from structured rows (max 200 per request).

**Request Body:**

```json
{
  "rows": [
    {
      "snapshotSpace": "string",
      "name": "string",
      "category": "'defi' | 'infrastructure' | 'public_goods' | 'social'"
    }
  ]
}
```

**Response:**

```json
{
  "success": true,
  "summary": { "created": 0, "duplicates": 0, "errors": 0, "total": 0 },
  "results": []
}
```

---

### PATCH /api/admin/daos/[daoId]

Update DAO fields and/or toggle DGI membership.

**Request Body (all optional):**

```json
{
  "name": "string",
  "category": "string",
  "badgeType": "string | null",
  "discordUrl": "string",
  "forumUrl": "string",
  "twitterUrl": "string",
  "twitterHandle": "string",
  "governanceForumUrl": "string",
  "telegramUrl": "string",
  "engagementStatus": "'not_started' | 'active' | 'warm' | 'pending' | 'blocked'",
  "engagementNotes": "string",
  "isDgiMember": "boolean",
  "dgiCategory": "string"
}
```

**Response:** `{ "success": true, "dao": {}, "dgi": { "action": "added | excluded | category_updated" } }`

---

### DELETE /api/admin/daos/[daoId]

Hard-delete a DAO. FK cascades handle related records.

**Request Body:** `{ "confirm": true }` (required)

**Response:** `{ "success": true, "message": "DAO deleted" }`

---

### POST /api/admin/recalculate

Manually trigger GVS recalculation for a single DAO or all non-opted-out DAOs. Also triggers DGI recalculation when processing all DAOs.

**Request Body:**

```json
{
  "daoId": "string (UUID, for single DAO)",
  "all": "boolean (true to process all DAOs)"
}
```

**Response:**

```json
{
  "success": true,
  "jobLogId": "string",
  "daosProcessed": "number",
  "daosTotal": "number",
  "durationMs": "number",
  "cacheInvalidated": "boolean",
  "results": []
}
```

---

### POST /api/admin/recalculate-dgi

Manually trigger DGI index recalculation using latest GVS snapshots. No request body required.

**Response:**

```json
{
  "success": true,
  "daoCount": "number",
  "indicesWritten": "number",
  "durationMs": "number"
}
```

---

### GET /api/admin/diagnose

Diagnostic endpoint for debugging GVS calculation and Snapshot API connectivity. Tests circuit breaker, space fetch, proposal fetch. Optional `?fullTest=true` for full GVS calculation, `?batchTest=true` for batch test.

**Response:** Diagnostic results object with test statuses, durations, and state information.

---

### GET /api/admin/job-logs

Fetch job execution history for admin monitoring.

**Query Params:**

| Param | Description |
|---|---|
| `limit` | Max results (default 30, max 100) |
| `id` | Specific job log ID |

**Response:** `{ "data": [], "meta": { "count": 0, "limit": 30 } }`

---

### GET /api/admin/costs/summary

Aggregated AI cost metrics for a selected period.

**Query Params:** `period`, `startDate`, `endDate`, `operationType`

**Response:** `{ "data": { totalCost, totalTokens, operationCount, ... } }`

---

### GET /api/admin/costs/details

Paginated list of individual AI operations.

**Query Params:** `period`, `startDate`, `endDate`, `operationType`, `page`, `limit`

**Response:** `{ "data": { operations, pagination } }`

---

### GET /api/admin/costs/history

Time-series AI cost data for charts.

**Query Params:** `period`, `startDate`, `endDate`, `operationType`, `granularity` (`day` | `week` | `month`)

**Response:** `{ "data": [] }`

---

### GET /api/admin/opt-out/all

List all opt-out requests (pending and processed) for the admin dashboard.

**Response:** `{ "requests": [] }`

---

### POST /api/admin/opt-out/process

Process a pending opt-out request. Marks the matching DAO as opted out from rankings.

**Request Body:**

```json
{
  "requestId": "string (UUID, required)",
  "daoId": "string (UUID, optional -- admin-specified DAO match)"
}
```

**Response:**

```json
{
  "success": true,
  "requestId": "string",
  "daoId": "string | null",
  "daoName": "string",
  "daoMatched": "boolean"
}
```

---

### GET /api/admin/free-tier/export

Export all free-tier query log records as CSV.

**Response:** CSV file download (`text/csv`).

---

### POST /api/admin/gdpr/export

Export all data associated with an email address (GDPR access request).

**Request Body:** `{ "email": "string" }`

**Response:** JSON with orders, reports, share rewards, and summary.

---

### POST /api/admin/gdpr/delete

Delete/anonymize all data for an email address (GDPR erasure). Orders are anonymized (retained for 7-year tax compliance); reports and share rewards are fully deleted.

**Request Body:**

```json
{
  "email": "string (required)",
  "confirm": true
}
```

**Response:**

```json
{
  "success": true,
  "result": {
    "ordersAnonymized": "number",
    "reportsDeleted": "number",
    "shareRewardsDeleted": "number"
  }
}
```

---

### PATCH /api/admin/engagement/[daoId]

Update DAO engagement settings (community links, status, notes).

**Request Body (all optional):**

```json
{
  "twitterHandle": "string",
  "forumUrl": "string",
  "governanceForumUrl": "string",
  "discordUrl": "string",
  "telegramUrl": "string",
  "farcasterUrl": "string",
  "engagementStatus": "'not_started' | 'active' | 'warm' | 'pending' | 'blocked'",
  "engagementNotes": "string"
}
```

**Response:** `{ "success": true, "dao": {} }`

---

### GET /api/admin/engagement/[daoId]/log

List all engagement log entries for a DAO (newest first).

**Response:** `{ "success": true, "entries": [] }`

---

### POST /api/admin/engagement/[daoId]/log

Add a new engagement log entry and update `last_engagement_at`.

**Request Body:**

```json
{
  "type": "'forum_post' | 'forum_reply' | 'x_reply' | 'x_post' | 'discord' | 'dm' | 'note'",
  "platform": "'x' | 'forum' | 'discord' | 'telegram' | 'other'",
  "content": "string (optional)",
  "url": "string (optional)",
  "contactName": "string (optional)",
  "outcome": "'pending' | 'no_response' | 'replied' | 'positive' | 'negative' | 'blocked' (default: 'pending')"
}
```

**Response:** `{ "success": true, "entry": {}, "lastEngagementAt": "ISO8601" }`

---

### POST /api/admin/engagement/report

Generate an engagement report PDF for a date range.

**Request Body:**

```json
{
  "week": "number (optional, ISO week)",
  "year": "number (optional)",
  "startDate": "string (optional, ISO8601)",
  "endDate": "string (optional, ISO8601)",
  "highlights": ["string array (optional)"]
}
```

**Response:** PDF binary download.

---

## Job / Cron Routes

### POST /api/jobs/fetch-snapshot

Pipeline step: Fetch Snapshot governance data for a report. Triggered by QStash. Enqueues the analyze step on success.

| Field | Details |
|---|---|
| **Auth** | QStash signature (`upstash-signature` header) |

**Request Body:** `{ "reportId": "string" }`

**Response:** `{ "success": true, "reportId": "string" }`

---

### POST /api/jobs/analyze

Pipeline step: Run AI analysis, validate consistency, generate PDF, calculate GVS, enrich with DGI benchmarks. Full pipeline from analysis through to `review_pending` status.

| Field | Details |
|---|---|
| **Auth** | QStash signature (`upstash-signature` header) |

**Request Body:** `{ "reportId": "string" }`

**Response:** `{ "success": true, "reportId": "string", "status": "review_pending" }`

---

### POST /api/jobs/share-reminder

Send a share-and-save reminder email 24h after report delivery. Skipped if Share & Save feature is disabled.

| Field | Details |
|---|---|
| **Auth** | QStash signature |

**Request Body:** `{ "reportId": "string" }`

**Response:** `{ "success": true, "reportId": "string" }`

---

### POST /api/jobs/gvs-calculate

Calculate GVS for a single DAO (called by daily recalculation orchestrator via QStash). Includes anomaly detection and Slack alerts.

| Field | Details |
|---|---|
| **Auth** | QStash signature |
| **Max Duration** | 300s (5 min) |

**Request Body:**

```json
{
  "daoId": "string",
  "daoName": "string",
  "snapshotSpace": "string",
  "jobLogId": "string (optional)"
}
```

**Response:** `{ "success": true, "daoId": "string", "gvsScore": "number", "durationMs": "number" }`

---

### GET /api/jobs/daily-recalculation

Vercel Cron: Daily GVS recalculation for all non-opted-out DAOs. Runs at midnight UTC. Includes DGI recalculation post-hook.

| Field | Details |
|---|---|
| **Auth** | Bearer token (`CRON_SECRET`) |
| **Schedule** | `0 0 * * *` (daily midnight UTC) |
| **Max Duration** | 300s (5 min) |

**Response:** Job result with `daosProcessed`, `daosTotal`, `failures`, `durationMs`.

---

### GET /api/jobs/dgi-calculation

Vercel Cron: DGI index recalculation. Runs 30 minutes after GVS recalculation.

| Field | Details |
|---|---|
| **Auth** | Bearer token (`CRON_SECRET`) |
| **Schedule** | `0 30 0 * * *` (00:30 UTC daily) |
| **Max Duration** | 60s |

**Response:** `{ "success": true, "indicesWritten": "number", "daoCount": "number", "durationMs": "number" }`

---

### GET /api/jobs/monthly-aggregation

Vercel Cron: Monthly GVS aggregation. Runs on the 1st of each month at 1am UTC.

| Field | Details |
|---|---|
| **Auth** | Bearer token (`CRON_SECRET`) |
| **Schedule** | `0 1 1 * *` (1st of month, 01:00 UTC) |
| **Max Duration** | 300s |

**Response:** Job result with `daosProcessed`, `daosTotal`, `failures`, `durationMs`.

---

### GET /api/jobs/gdpr-cleanup

Vercel Cron: GDPR data retention cleanup and GVS snapshot cleanup (30-day retention). Runs weekly on Sundays at 3am UTC.

| Field | Details |
|---|---|
| **Auth** | Vercel Cron header or Bearer token (`CRON_SECRET`) |
| **Schedule** | `0 3 * * 0` (Sundays 03:00 UTC) |

**Response:**

```json
{
  "message": "All cleanup jobs completed successfully",
  "gdpr": { "freeQueryLogsAnonymized": 0, "leadsCleanedUp": 0 },
  "snapshots": { "deletedCount": 0, "retentionDays": 30 }
}
```

---

### GET /api/cron/scan-forums

Vercel Cron: Scan DAO governance forums for engagement signals (RSS). Runs 3x daily.

| Field | Details |
|---|---|
| **Auth** | Bearer token (`CRON_SECRET`) |
| **Schedule** | `0 8,14,20 * * *` (08:00, 14:00, 20:00 UTC) |
| **Max Duration** | 300s |

**Response:**

```json
{
  "success": true,
  "durationMs": "number",
  "totalDaos": "number",
  "daosWithForums": "number",
  "signalsCreated": "number",
  "details": []
}
```

---

### GET /api/cron/recovery

Vercel Cron: Recovery email sender for orphan orders (paid but intake form not submitted). Runs hourly.

| Field | Details |
|---|---|
| **Auth** | Bearer token (`CRON_SECRET`) |
| **Schedule** | `0 * * * *` (hourly) |

**Response:**

```json
{
  "success": true,
  "orphanOrdersFound": "number",
  "emailsSent": "number",
  "emailsFailed": "number"
}
```

---

### POST /api/cron/send-rankings-emails

Rankings Watch: Send daily notification emails to confirmed subscribers about new DAOs.

| Field | Details |
|---|---|
| **Auth** | Bearer token (`CRON_SECRET`) |

**Response:**

```json
{
  "success": true,
  "emailsSent": "number",
  "errors": "number",
  "newDAOs": "number",
  "subscribers": "number"
}
```

---

### POST /api/cron/webhook-health

Webhook health monitoring. Checks last webhook timestamp for Stripe and Coinbase. Alerts if stale (48+ hours).

| Field | Details |
|---|---|
| **Auth** | Bearer token (`CRON_SECRET`) |

**Response:**

```json
{
  "status": "healthy | alert",
  "providers": {
    "coinbase": { "lastWebhook": "ISO8601 | null", "ageHours": "number | null", "stale": "boolean" },
    "stripe": { "lastWebhook": "ISO8601 | null", "ageHours": "number | null", "stale": "boolean" }
  }
}
```

---

## GDPR Routes

User-facing GDPR endpoints (no admin auth required).

### POST /api/gdpr/export

Export all free-tier query records for an identifier (email or wallet). GDPR-compliant data access request.

| Field | Details |
|---|---|
| **Auth** | None (identifier-based lookup) |

**Request Body:**

```json
{
  "identifier": "string (email or wallet address, required)",
  "identifierType": "'email' | 'wallet' (required)"
}
```

**Response:**

```json
{
  "success": true,
  "dataController": { "name": "ChainSights", "email": "hello@chainsights.one" },
  "dataSubject": { "identifierType": "string", "identifierHashPrefix": "string" },
  "records": [],
  "exportedAt": "ISO8601"
}
```

---

### POST /api/gdpr/delete

Request deletion of free-tier query records. Marks records for deletion within 30 days.

| Field | Details |
|---|---|
| **Auth** | None (identifier-based lookup) |

**Request Body:**

```json
{
  "identifier": "string (required)",
  "identifierType": "'email' | 'wallet' (required)"
}
```

**Response:**

```json
{
  "success": true,
  "message": "Deletion request received. Your data will be permanently deleted within 30 days.",
  "recordsMarked": "number"
}
```
