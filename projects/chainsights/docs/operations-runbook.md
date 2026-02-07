# ChainSights Operations Runbook

> Production operations guide for the ChainSights platform.
> Covers deployment, scheduled jobs, webhooks, monitoring, troubleshooting, and rollback procedures.

**Last Updated:** 2026-02-07
**Owner:** Mario

---

## Table of Contents

1. [Architecture Overview](#1-architecture-overview)
2. [Deployment Process](#2-deployment-process)
3. [Environment Variables Checklist](#3-environment-variables-checklist)
4. [Database Operations](#4-database-operations)
5. [Scheduled Jobs (Cron)](#5-scheduled-jobs-cron)
6. [QStash Background Jobs](#6-qstash-background-jobs)
7. [Webhook Endpoints](#7-webhook-endpoints)
8. [Monitoring and Alerting](#8-monitoring-and-alerting)
9. [Feature Flags](#9-feature-flags)
10. [Common Issues and Fixes](#10-common-issues-and-fixes)
11. [Rollback Procedures](#11-rollback-procedures)
12. [Operational Scripts](#12-operational-scripts)
13. [Contacts and Escalation](#13-contacts-and-escalation)

---

## 1. Architecture Overview

| Component         | Technology               | Provider       |
|-------------------|--------------------------|----------------|
| Framework         | Next.js 14 (App Router)  | Vercel         |
| Database          | PostgreSQL (serverless)  | Neon           |
| ORM               | Drizzle ORM              | --             |
| Background Jobs   | QStash                   | Upstash        |
| Payments (fiat)   | Stripe (Checkout + Webhooks) | Stripe     |
| Payments (crypto) | Coinbase Commerce        | Coinbase       |
| AI Analysis       | Claude (Anthropic SDK)   | Anthropic      |
| Email             | Resend                   | Resend         |
| Error Tracking    | Sentry (Next.js SDK)     | Sentry         |
| Event Tracking    | LogSnag                  | LogSnag        |
| Alerting          | Slack Webhooks + Resend  | Slack / Resend |
| SEO Indexing      | IndexNow API             | Bing           |
| Analytics         | Self-hosted (masem.at)   | --             |

**Production URL:** `https://chainsights.one`
**Admin Dashboard:** `https://chainsights.one/admin`

---

## 2. Deployment Process

ChainSights uses **git-based continuous deployment** via Vercel.

### Standard Deployment

1. Push or merge to `master` branch.
2. Vercel automatically builds and deploys.
3. Build pipeline: `next build && node scripts/indexnow.mjs`
   - The IndexNow script submits all public URLs to Bing for re-indexing.
   - IndexNow only runs when `VERCEL_ENV=production`; skipped for preview deploys.
   - IndexNow failures are non-fatal and will not break the build.
4. Vercel serves the new deployment atomically (zero-downtime swap).

### Preview Deployments

- Every push to a non-`master` branch creates a preview deployment.
- Preview deployments share the same environment variables unless overridden per-branch in Vercel.
- **WARNING:** Preview deploys use the same database unless `DATABASE_URL` is overridden. Be careful with destructive operations.

### Manual Redeployment

- Vercel Dashboard > Deployments > choose a deployment > Redeploy.
- Or trigger via CLI: `vercel --prod`.

### Build Settings

- **Build Command:** `next build && node scripts/indexnow.mjs`
- **Output Directory:** `.next`
- **Install Command:** `npm install`
- **Node Version:** 20.x

---

## 3. Environment Variables Checklist

All environment variables are managed in **Vercel Dashboard > Settings > Environment Variables**.

### Required for Core Functionality

| Variable | Purpose | Where to Get |
|----------|---------|--------------|
| `DATABASE_URL` | Neon PostgreSQL connection string | Neon Console |
| `STRIPE_SECRET_KEY` | Stripe API key (server-side) | Stripe Dashboard |
| `STRIPE_WEBHOOK_SECRET` | Stripe webhook signing secret | Stripe Dashboard > Webhooks |
| `NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY` | Stripe publishable key (client-side) | Stripe Dashboard |
| `ANTHROPIC_API_KEY` | Claude AI API key for report analysis | Anthropic Console |
| `ADMIN_PASSWORD` | Admin dashboard authentication | Self-generated |

### Required for Pipeline / Background Jobs

| Variable | Purpose | Where to Get |
|----------|---------|--------------|
| `QSTASH_TOKEN` | QStash client token | Upstash Console |
| `QSTASH_CURRENT_SIGNING_KEY` | QStash signature verification | Upstash Console |
| `QSTASH_NEXT_SIGNING_KEY` | QStash key rotation | Upstash Console |
| `CRON_SECRET` | Authentication for Vercel Cron jobs | `openssl rand -hex 32` |
| `REVALIDATE_SECRET` | ISR cache revalidation authentication | `openssl rand -hex 32` |

### Required for Email

| Variable | Purpose | Where to Get |
|----------|---------|--------------|
| `RESEND_API_KEY` | Email sending via Resend | Resend Dashboard |
| `EMAIL_FROM` | Sender address (default: `ChainSights <hello@chainsights.one>`) | -- |

### Required for Monitoring

| Variable | Purpose | Where to Get |
|----------|---------|--------------|
| `NEXT_PUBLIC_SENTRY_DSN` | Sentry DSN (client-side) | Sentry > Project Settings |
| `SENTRY_DSN` | Sentry DSN (server-side) | Sentry > Project Settings |
| `SENTRY_ORG` | Sentry organization slug | Sentry Dashboard |
| `SENTRY_PROJECT` | Sentry project slug | Sentry Dashboard |
| `SENTRY_AUTH_TOKEN` | Sentry auth token for source map uploads | Sentry > Auth Tokens |

### Optional

| Variable | Purpose | Default |
|----------|---------|---------|
| `LOGSNAG_TOKEN` | LogSnag event tracking | Logs to console if unset |
| `LOGSNAG_PROJECT` | LogSnag project name | `chainsights` |
| `SLACK_WEBHOOK_URL` | Slack alerts for job failures | Alerts disabled if unset |
| `NEXT_PUBLIC_BASE_URL` | Base URL for email links | `https://chainsights.one` |
| `COINBASE_COMMERCE_API_KEY` | Coinbase Commerce API key | Coinbase Commerce Dashboard |
| `COINBASE_COMMERCE_WEBHOOK_SECRET` | Coinbase webhook verification | Coinbase Commerce Dashboard |
| `GVS_DAILY_RETENTION_DAYS` | Daily snapshot retention | `30` |
| `GVS_ANOMALY_THRESHOLD` | Score change threshold for alerts | `0.5` |
| `SNAPSHOT_CLEANUP_DRY_RUN` | Test cleanup without deleting | `false` |
| `MATRIX_ADMIN_EMAILS` | Comma-separated admin emails | -- |
| `FEEDBACK_EMAIL_ENABLED` | Email notifications on feedback | `true` |

### Feature Flags

| Variable | Purpose | Default |
|----------|---------|---------|
| `DGI_PUBLIC` | Show DGI pages to public (server-side) | `false` |
| `NEXT_PUBLIC_DGI_PUBLIC` | Show DGI pages to public (client-side) | `false` |

---

## 4. Database Operations

### Connection

The database is a **Neon serverless PostgreSQL** instance. Connection uses the `@neondatabase/serverless` driver.

**Neon Console:** Log in at [console.neon.tech](https://console.neon.tech)

### Migrations

Migrations use **Drizzle Kit**. Schema is defined in `src/lib/db/schema.ts`.

```bash
# Generate migration files from schema changes
npm run db:generate

# Apply pending migrations to the database
npm run db:migrate

# Push schema directly (development only, skips migration files)
npm run db:push

# Open Drizzle Studio (visual database browser)
npm run db:studio
```

**Production Migration Process:**

1. Make schema changes in `src/lib/db/schema.ts`.
2. Run `npm run db:generate` to create migration SQL.
3. Review generated SQL in `drizzle/` directory.
4. Commit the migration files.
5. Run `npm run db:migrate` against production (use `DATABASE_URL` pointing to production).
6. Deploy the code that depends on the schema change.

**CAUTION:** Always run migrations before deploying code that depends on new columns/tables. Otherwise the deployed code will fail with missing column errors.

### Backups

Neon provides automatic branching and point-in-time recovery.

- **Point-in-Time Recovery:** Neon retains full WAL history. You can create a branch from any point in time via the Neon Console.
- **Before Risky Operations:** Create a Neon branch first as a backup:
  1. Neon Console > Project > Branches > Create Branch.
  2. Name it `backup-YYYY-MM-DD-description`.
  3. Perform the risky operation.
  4. If it fails, restore from the branch.

### Neon Cold Starts

Neon serverless computes can suspend after inactivity. The first connection takes 3-5 seconds ("cold start"). The codebase handles this with `withDbRetry()` from `src/lib/db/retry.ts`:

- Retries up to 3 times with exponential backoff (starting at 3000ms).
- Detects cold start error patterns: `Control plane request failed`, `Connection terminated unexpectedly`, `connect ETIMEDOUT`, etc.
- All cron jobs use `withDbRetry()` for resilience.

---

## 5. Scheduled Jobs (Cron)

All cron jobs are defined in `vercel.json` and authenticated via `CRON_SECRET`.

### Cron Schedule Overview

| Schedule | Path | Job | Description |
|----------|------|-----|-------------|
| `0 0 * * *` (daily, 00:00 UTC) | `/api/jobs/daily-recalculation` | Daily GVS Recalculation | Enqueues GVS calculation for all active DAOs via QStash |
| `45 0 * * *` (daily, 00:45 UTC) | `/api/jobs/dgi-calculation` | DGI Calculation | Calculates Decentralized Governance Index after GVS completes |
| `30 0 * * *` (daily, 00:30 UTC) | `/api/cron/send-rankings-emails` | Rankings Watch Emails | Sends email notifications about new DAOs to subscribers |
| `0 1 1 * *` (1st of month, 01:00 UTC) | `/api/jobs/monthly-aggregation` | Monthly GVS Aggregation | Aggregates daily snapshots into monthly (avg/min/max) |
| `0 3 * * 0` (Sundays, 03:00 UTC) | `/api/jobs/gdpr-cleanup` | GDPR + Snapshot Cleanup | Anonymizes expired data, deletes daily snapshots older than 30 days |
| `0 * * * *` (hourly) | `/api/cron/recovery` | Order Recovery Emails | Finds paid orders without completed intake forms, sends recovery email |
| `0 8,14,20 * * *` (3x daily) | `/api/cron/scan-forums` | Forum RSS Scanner | Scans DAO governance forums for engagement signals |

### Daily GVS Recalculation (00:00 UTC)

**What it does:**
1. Fetches all non-opted-out DAOs from the database.
2. Enqueues each DAO to QStash for independent GVS calculation.
3. DAOs are staggered: 2 DAOs every 30 seconds to avoid Snapshot API rate limits.
4. Each DAO is processed independently by `/api/jobs/gvs-calculate`.

**What to check:**
- Admin Dashboard > Jobs: look for `daily-gvs-recalculation` job log.
- Status should be `completed` with `daosProcessed` matching `daosTotal`.
- Slack channel: success/failure notifications.
- If >50% DAOs fail, the job is marked as `failed`.

**Max Duration:** 300 seconds (5 minutes, Vercel Pro).

**Common failures:**
- Snapshot API rate limiting (mitigated by staggered delays).
- QStash token expired or misconfigured.
- Neon cold start (retried automatically).

### DGI Calculation (00:45 UTC)

**What it does:**
1. Calculates the Decentralized Governance Index from latest GVS scores.
2. Invalidates cache for `/rankings` and `/matrix` pages.
3. Sends Slack notification.

**Depends on:** Daily GVS recalculation must complete first. Scheduled 45 minutes later to allow all QStash jobs to finish.

**Max Duration:** 60 seconds.

### Monthly Aggregation (1st of month, 01:00 UTC)

**What it does:**
1. For each active DAO, aggregates daily GVS snapshots from the previous month.
2. Creates a `monthly` snapshot with avg/min/max scores.
3. Monthly snapshots are never deleted by cleanup.

**What to check:**
- Admin Dashboard > Jobs: `monthly-gvs-aggregation` log.
- Verify monthly snapshots exist in the database.

### GDPR + Snapshot Cleanup (Sundays, 03:00 UTC)

**What it does:**
1. **GDPR Cleanup:** Anonymizes free query logs and cleans up expired leads.
2. **Snapshot Cleanup:** Deletes daily GVS snapshots older than retention period (default: 30 days). Monthly snapshots are **never** deleted.

**What to check:**
- Admin Dashboard > Jobs: `snapshot-cleanup` log.
- Verify `monthlySnapshotsPreserved` count has not changed.

**Dry Run:** Set `SNAPSHOT_CLEANUP_DRY_RUN=true` to log what would be deleted without actually deleting.

### Order Recovery (Hourly)

**What it does:**
- Finds orders where payment succeeded (status=`paid`) but intake form was not completed (no reportId).
- Sends a recovery email with a prefilled link to complete the form.
- Only sends for orders between 30 minutes and 24 hours old.
- Only sends once (tracks `recoveryEmailSentAt`).

### Forum Scanner (3x Daily)

**What it does:**
- Scans governance forum RSS feeds for DAOs with configured forum URLs.
- Creates engagement signals in the database.
- Used by the Signal Scanner / Engagement Hub features.

**Max Duration:** 300 seconds.

---

## 6. QStash Background Jobs

QStash is used for jobs that exceed Vercel's function timeout or need to be distributed across many invocations.

### Pipeline Jobs (Report Generation)

When a customer purchases a report and submits the intake form, a pipeline of QStash jobs runs:

1. **`fetch-snapshot`** (`POST /api/jobs/fetch-snapshot`) -- Fetches governance data from Snapshot.org API. On success, enqueues `analyze`.
2. **`analyze`** (`POST /api/jobs/analyze`) -- Runs AI analysis with Claude, validates consistency, calculates GVS, generates PDF, enriches with DGI benchmarks. Sets report status to `review_pending`.
3. **`share-reminder`** (`POST /api/jobs/share-reminder`) -- Scheduled 24 hours after delivery. Sends share reminder email (only when `SHARE_REWARDS_ENABLED` is true).

### GVS Calculate Job

**`gvs-calculate`** (`POST /api/jobs/gvs-calculate`) -- Calculates GVS for a single DAO. Called by the daily recalculation cron via QStash with staggered delays (2 DAOs per 30 seconds).

Each invocation:
1. Calculates GVS score.
2. Saves daily snapshot.
3. Calculates delta from previous snapshot.
4. Runs anomaly detection (alerts via Slack if score changes >0.5 points).

**Max Duration:** 300 seconds per DAO.

### QStash Signature Verification

All QStash endpoints verify the `upstash-signature` header using `src/lib/jobs/verify.ts`. In development (without QStash configured), verification is skipped.

---

## 7. Webhook Endpoints

### Stripe Webhook

**Endpoint:** `POST /api/webhooks/stripe`

**Events handled:**

| Event | Handler | Purpose |
|-------|---------|---------|
| `checkout.session.completed` | `handleCheckoutCompleted` | Creates order record, tracks revenue in LogSnag, sends analytics event |
| `payment_intent.succeeded` | `handlePaymentIntentSucceeded` | Calculates and records donation (3% or 5% with CHOKIHEART promo) |
| `payment_intent.payment_failed` | -- | Logs failure |
| `customer.subscription.created` | `handleSubscriptionCreated` | Creates subscription record |
| `customer.subscription.updated` | `handleSubscriptionUpdated` | Updates subscription status |
| `customer.subscription.deleted` | `handleSubscriptionDeleted` | Marks subscription as canceled |
| `invoice.paid` | `handleInvoicePaid` | Records donation for Matrix subscription renewals |

**Signature Verification:** Uses `stripe.webhooks.constructEvent()` with `STRIPE_WEBHOOK_SECRET`.

**Troubleshooting Stripe Webhooks:**

1. **Signature verification failures:**
   - Check `STRIPE_WEBHOOK_SECRET` matches the secret shown in Stripe Dashboard > Developers > Webhooks.
   - Ensure the webhook endpoint URL in Stripe matches `https://chainsights.one/api/webhooks/stripe`.
   - Check Vercel logs for `Webhook signature verification failed`.

2. **Missing events:**
   - Stripe Dashboard > Developers > Webhooks > select endpoint > Recent deliveries.
   - Check for failed deliveries and response codes.
   - Stripe retries failed webhooks for up to 72 hours.

3. **Testing locally:**
   - Use Stripe CLI: `stripe listen --forward-to localhost:3000/api/webhooks/stripe`.
   - Use test mode keys (`sk_test_...`).

### Coinbase Commerce Webhook

**Endpoint:** `POST /api/webhooks/coinbase`

**Events handled:**

| Event | Handler | Purpose |
|-------|---------|---------|
| `charge:confirmed` | `handleChargeConfirmed` | Records donation for crypto payments |
| `charge:failed` | -- | Logs failure |
| `charge:expired` | -- | Logs expiry |

**Signature Verification:** Uses `x-cc-webhook-signature` header with `COINBASE_COMMERCE_WEBHOOK_SECRET`.

**Troubleshooting:**
- Always returns 200 to prevent Coinbase retries on handler errors.
- Check `COINBASE_COMMERCE_WEBHOOK_SECRET` in Coinbase Commerce Dashboard > Settings > Webhook subscriptions.

---

## 8. Monitoring and Alerting

### Sentry (Error Tracking)

**Configuration files:**
- `sentry.client.config.ts` -- Browser errors
- `sentry.server.config.ts` -- Server-side (Node.js) errors
- `sentry.edge.config.ts` -- Edge runtime errors

**Key settings:**
- **Sample Rate:** 10% (`tracesSampleRate: 0.1`) for cost efficiency.
- **Environment:** Auto-detected from `VERCEL_ENV`.
- **Production Only:** Errors are only sent when `NODE_ENV=production`.
- **PII Scrubbing:** All events pass through `scrubSentryEvent()` before sending (GDPR compliant).
- **Ignored Errors:** Browser extensions, network errors, user-initiated aborts.

**Source Maps:** Uploaded to Sentry during build via `@sentry/nextjs` webpack plugin. Source maps are hidden from clients.

**Where to look:** Sentry Dashboard > Issues. Filter by environment (production/preview).

### LogSnag (Business Event Tracking)

**Configuration:** `src/lib/logsnag.ts`

**Channels:**
| Channel | Events | Notifications |
|---------|--------|---------------|
| `revenue` | Order Created, Subscription Created | Push notification |
| `pipeline` | Analysis Started, Ready for Review, Governance Audit Analysis | Silent (except Ready for Review) |
| `growth` | Growth events | Push notification |
| `errors` | Pipeline Failed, Validation Failed, Webhook Errors | Push notification |

**Fallback:** If `LOGSNAG_TOKEN` is not set, events are logged to console.

### Slack Alerts

**Configuration:** `src/lib/monitoring/alerts.ts`

**Triggers:**
- Daily GVS recalculation success/failure.
- DGI calculation success/failure.
- Monthly aggregation success/failure.
- Snapshot cleanup results.
- GVS anomaly detection (score change > threshold).
- QStash not configured.

**Format:** `{emoji} ChainSights: {message}` where emoji is checkmark for success, siren for alerts.

**Setup:** Configure `SLACK_WEBHOOK_URL` in Vercel. Get webhook URL from Slack > Apps > Incoming Webhooks.

### Email Alerts

**Configuration:** Same module (`src/lib/monitoring/alerts.ts`), uses Resend.

**Triggers:** Job success/failure notifications sent to `hello@chainsights.one`.

### Admin Dashboard

**URL:** `https://chainsights.one/admin`

**Sections:**
- **Jobs:** View recent job logs with status, duration, DAO counts, errors, and anomalies.
- **Reports:** View/manage generated reports in the pipeline.
- **DGI:** Manage DGI index membership and configuration.
- **Orders:** View payment history and order status.

---

## 9. Feature Flags

### Server-Side Feature Flags (Environment Variables)

| Flag | Purpose | Current Default |
|------|---------|-----------------|
| `DGI_PUBLIC` | DGI pages visible to non-admin users | `false` |
| `NEXT_PUBLIC_DGI_PUBLIC` | Client-side mirror of DGI_PUBLIC | `false` |
| `OPEN_MATRIX` | DAO Matrix free for everyone | `true` |

### Code-Level Feature Flags

Defined in `src/lib/config/features.ts`:

| Flag | Status | Purpose |
|------|--------|---------|
| `SHARE_REWARDS_ENABLED` | `false` | Share-and-Save cashback program |
| `DAO_MATRIX_ENABLED` | `false` | DAO Matrix subscription product |
| `API_ACCESS_ENABLED` | `false` | Programmatic API access |

### DGI Feature Gate

DGI visibility is controlled by `src/lib/feature-flags.ts`:
- `isDgiPublic()` -- server-side check.
- `isDgiPublicClient()` -- client-side check.
- `isDgiVisible()` -- returns true if DGI_PUBLIC=true OR user is admin.
- Admins always see DGI pages regardless of the flag.

---

## 10. Common Issues and Fixes

### Neon Database Cold Start Failures

**Symptom:** API routes return 500 with `Control plane request failed` or `Connection terminated unexpectedly`.

**Fix:** Automatic -- `withDbRetry()` retries up to 3 times with 3-second initial backoff. If persistent, check Neon Console for compute status.

### Snapshot API Rate Limiting

**Symptom:** Daily GVS recalculation shows >50% DAO failures. Error messages reference rate limits or 429 responses.

**Fix:** The stagger is currently 2 DAOs per 30 seconds. If failures persist:
1. Check Snapshot.org API status.
2. Increase `DELAY_BETWEEN_BATCHES_SECONDS` in `src/lib/jobs/qstash.ts`.
3. Reduce `BATCH_SIZE` from 2 to 1.

### QStash Signature Verification Failures

**Symptom:** Jobs return 401 `Invalid signature`.

**Fix:**
1. Verify `QSTASH_CURRENT_SIGNING_KEY` and `QSTASH_NEXT_SIGNING_KEY` in Vercel match Upstash Console.
2. After key rotation in Upstash, update both keys in Vercel and redeploy.

### Stripe Webhook Not Receiving Events

**Symptom:** Orders are paid in Stripe but not appearing in the database.

**Fix:**
1. Stripe Dashboard > Developers > Webhooks: check endpoint status.
2. Verify endpoint URL is `https://chainsights.one/api/webhooks/stripe`.
3. Check for failed deliveries and response codes.
4. Verify `STRIPE_WEBHOOK_SECRET` matches.
5. Re-send failed events from Stripe Dashboard.

### Report Pipeline Stuck

**Symptom:** Report status stuck at `fetching` or `analyzing`.

**Fix:**
1. Check Vercel Function Logs for the specific report ID.
2. Check QStash Console (Upstash) for failed/pending messages.
3. If a job is stuck, manually re-trigger via Admin Dashboard or re-enqueue via QStash.
4. If Anthropic API is down, the `analyze` step will fail -- check Anthropic status page.

### Build Failures

**Symptom:** Vercel build fails.

**Fix:**
1. Check build logs in Vercel Dashboard.
2. Most common: TypeScript type errors, missing environment variables at build time.
3. The IndexNow script is non-fatal -- it exits with code 0 even on failure.

### DGI Pages Showing "Coming Soon"

**Symptom:** DGI pages show Coming Soon instead of actual data.

**Fix:** Set both `DGI_PUBLIC=true` and `NEXT_PUBLIC_DGI_PUBLIC=true` in Vercel environment variables and redeploy.

### Cache Not Updating After Job Runs

**Symptom:** Rankings or Matrix pages show stale data after daily recalculation.

**Fix:**
1. The DGI calculation cron (00:45 UTC) automatically invalidates cache for `/rankings` and `/matrix`.
2. Manually invalidate: call `POST /api/revalidate?path=/rankings` with `Authorization: Bearer {REVALIDATE_SECRET}`.
3. Check that `REVALIDATE_SECRET` is set in environment.

---

## 11. Rollback Procedures

### Code Rollback (Vercel)

1. **Vercel Dashboard > Deployments.**
2. Find the last known good deployment.
3. Click the three-dot menu > **Promote to Production**.
4. The previous deployment is live within seconds.
5. Fix the issue on a branch, test, then merge to `master`.

### Feature Flag Rollback

For features gated by environment variables:

1. Go to **Vercel Dashboard > Settings > Environment Variables**.
2. Change the flag value (e.g., `DGI_PUBLIC=false`).
3. Redeploy (or the next push triggers it).
4. No data loss -- gated features only affect visibility.

### Database Rollback

For schema changes gone wrong:

1. **Neon Console > Branches > Create Branch** from a point before the migration.
2. Point `DATABASE_URL` to the backup branch.
3. Redeploy.
4. Fix the migration, then switch back to the main branch.

For data-level rollback:
1. Use Neon's point-in-time restore to create a branch at the desired timestamp.
2. Query the backup branch for correct data.
3. Manually fix the main branch.

### Job Rollback

If a cron job caused data corruption:

1. Check the job log in Admin Dashboard for the exact timestamp and DAOs affected.
2. Use Neon branching to restore data from before the job ran.
3. Disable the cron by removing it from `vercel.json` and deploying, or by setting an invalid `CRON_SECRET`.

---

## 12. Operational Scripts

All scripts are in the `scripts/` directory. Most require `.env.local` with `DATABASE_URL`.

### IndexNow URL Submission

Automatically submits all public URLs to Bing after production builds.

```bash
node scripts/indexnow.mjs
```

- Runs automatically as part of `npm run build`.
- Only executes when `VERCEL_ENV=production`.
- Submits static pages + all DAO detail pages from the database.
- Non-fatal: build succeeds even if IndexNow fails.

### Generate Rankings

Generates governance scores for configured DAOs using AI analysis. Writes to `data/rankings.json` and appends to `data/rankings-history.json`.

```bash
npm run rankings:generate
npm run rankings:generate -- --limit=3  # Test with fewer DAOs
```

Requires: `ANTHROPIC_API_KEY` in `.env.local`.

### Seed Leaderboard

Populates database with DAOs and calculates live GVS scores.

```bash
npm run seed:leaderboard
npm run seed:leaderboard -- --limit=3  # Test with fewer DAOs
```

Reads from `config/ranking-daos.json`. Calls Snapshot API + GVS calculation for each DAO.

### Seed Test Data

Populates database with mock DAOs and GVS scores for development.

```bash
npm run seed:test
```

### Bulk Add DAOs

Inserts DAOs from a CSV file and optionally triggers GVS calculation.

```bash
npx tsx --env-file=.env.local scripts/bulk-add-daos.ts path/to/file.csv
npx tsx --env-file=.env.local scripts/bulk-add-daos.ts path/to/file.csv --dry-run
npx tsx --env-file=.env.local scripts/bulk-add-daos.ts path/to/file.csv --skip-gvs
```

CSV format: `space_id, name, category`.

### Find Target DAOs

Searches Snapshot.org for potential DAOs to add.

```bash
npm run find-daos
```

### Set Featured Badges

Updates `badge_type='featured_analysis'` for all DAOs without a badge.

```bash
npx tsx scripts/set-featured-badges.ts
```

### Add Community Links

Populates Discord, Forum, and Twitter URLs for DAOs.

```bash
npx tsx scripts/add-community-links.ts
```

### Other Scripts

| Script | Purpose |
|--------|---------|
| `scripts/migrate-crypto-payments.ts` | One-time migration for crypto payment data |
| `scripts/generate-sample-report.ts` | Generates a sample PDF report for testing |
| `scripts/convert-og-image.mjs` | Converts OG images for social sharing |
| `scripts/snapshot-spike.mjs` | Debug/test script for Snapshot API calls |

---

## 13. Contacts and Escalation

### Service Status Pages

| Service | Status Page |
|---------|-------------|
| Vercel | [vercel.com/status](https://www.vercel-status.com) |
| Neon | [neonstatus.com](https://neonstatus.com) |
| Stripe | [status.stripe.com](https://status.stripe.com) |
| Sentry | [status.sentry.io](https://status.sentry.io) |
| Upstash (QStash) | [status.upstash.com](https://status.upstash.com) |
| Anthropic | [status.anthropic.com](https://status.anthropic.com) |
| Snapshot.org | -- (no public status page; check X/Twitter for outages) |

### Admin Access

- **Vercel Dashboard:** Project > Settings
- **Neon Console:** [console.neon.tech](https://console.neon.tech)
- **Stripe Dashboard:** [dashboard.stripe.com](https://dashboard.stripe.com)
- **Upstash Console:** [console.upstash.com](https://console.upstash.com)
- **Sentry Dashboard:** [sentry.io](https://sentry.io)
- **LogSnag Dashboard:** [app.logsnag.com](https://app.logsnag.com)
- **Resend Dashboard:** [resend.com](https://resend.com)

---

## Appendix: File Reference

| Resource | Path |
|----------|------|
| Vercel cron config | `vercel.json` |
| Next.js config | `next.config.js` |
| Drizzle config | `drizzle.config.ts` |
| Database schema | `src/lib/db/schema.ts` |
| Database retry helper | `src/lib/db/retry.ts` |
| QStash client | `src/lib/jobs/qstash.ts` |
| QStash signature verification | `src/lib/jobs/verify.ts` |
| Daily recalculation logic | `src/lib/jobs/daily-recalculation.ts` |
| Monthly aggregation logic | `src/lib/jobs/monthly-aggregation.ts` |
| Snapshot cleanup logic | `src/lib/jobs/snapshot-cleanup.ts` |
| Job logs service | `src/lib/jobs/job-logs.ts` |
| Stripe webhook handler | `src/app/api/webhooks/stripe/route.ts` |
| Coinbase webhook handler | `src/app/api/webhooks/coinbase/route.ts` |
| Monitoring alerts | `src/lib/monitoring/alerts.ts` |
| LogSnag integration | `src/lib/logsnag.ts` |
| Sentry PII scrubbing | `src/lib/monitoring/sentry-scrubbing.ts` |
| Feature flags (env) | `src/lib/feature-flags.ts` |
| Feature flags (code) | `src/lib/config/features.ts` |
| Environment template | `.env.example` |
| DGI launch runbook | `docs/launch-runbook-dgi.md` |
