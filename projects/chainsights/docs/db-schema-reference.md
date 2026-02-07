# ChainSights Database Schema Reference

> Auto-generated from `src/lib/db/schema.ts`
> Last updated: 2026-02-07

---

## Table of Contents

- [Enum Definitions](#enum-definitions)
- [Core Tables](#core-tables)
  - [daos](#daos)
  - [gvs_snapshots](#gvs_snapshots)
  - [reported_daos](#reported_daos)
- [Reports Tables](#reports-tables)
  - [orders](#orders)
  - [reports](#reports)
  - [quick_checks](#quick_checks)
- [Auth Tables](#auth-tables)
  - [users](#users)
  - [auth_tokens](#auth_tokens)
- [Payments Tables](#payments-tables)
  - [donations](#donations)
  - [subscriptions](#subscriptions)
  - [share_rewards](#share_rewards)
- [Engagement Tables](#engagement-tables)
  - [engagement_log](#engagement_log)
  - [signals](#signals)
  - [feedback](#feedback)
- [DGI Tables](#dgi-tables)
  - [dgi_snapshots](#dgi_snapshots)
  - [dao_index_membership](#dao_index_membership)
- [Leads & Subscribers Tables](#leads--subscribers-tables)
  - [leads](#leads)
  - [email_subscribers](#email_subscribers)
  - [free_query_log](#free_query_log)
  - [quiz_responses](#quiz_responses)
  - [opt_out_requests](#opt_out_requests)
- [Operations Tables](#operations-tables)
  - [job_logs](#job_logs)

---

## Enum Definitions

### `order_status`
Tracks the lifecycle of an order from creation to delivery.

| Value | Description |
|-------|-------------|
| `pending` | Order created, awaiting payment |
| `paid` | Payment confirmed |
| `processing` | Report generation in progress |
| `draft_ready` | Draft report is ready for review |
| `approved` | Report approved by admin |
| `delivered` | Final report delivered to customer |
| `failed` | Order processing failed |

### `tier`
Product tier for report orders.

| Value | Description |
|-------|-------------|
| `standard` | Standard report |
| `deep_dive` | Deep Dive report |
| `governance_audit` | Full Governance Audit |

### `payment_method`
Supported payment methods.

| Value | Description |
|-------|-------------|
| `stripe` | Stripe (credit card) payment |
| `crypto` | Cryptocurrency payment |

### `report_status`
Detailed lifecycle status for report generation pipeline.

| Value | Description |
|-------|-------------|
| `pending` | Report created, waiting to start |
| `fetching` | Fetching data from Snapshot API |
| `analyzing` | AI analysis in progress |
| `validation_failed` | AI analysis failed validation checks |
| `generating` | PDF being created |
| `review_pending` | Awaiting admin review (replaces legacy 'draft') |
| `declined` | Feedback submitted, re-analyzing |
| `draft` | Legacy status, kept for backwards compatibility |
| `approved` | Report approved |
| `generated` | PDF generated successfully |
| `sent` | Report sent to customer |
| `failed` | Report generation failed |

### `reward_status`
Status for Share & Save cashback rewards.

| Value | Description |
|-------|-------------|
| `pending` | Reward claim submitted |
| `approved` | Reward approved by admin |
| `paid` | Cashback paid out |
| `rejected` | Reward claim rejected |

### `sentiment`
Sentiment classification for social media engagement.

| Value | Description |
|-------|-------------|
| `positive` | Positive sentiment |
| `mixed` | Mixed sentiment |
| `critical` | Critical/negative sentiment |

### `identifier_type`
Type of user identifier for free query tracking.

| Value | Description |
|-------|-------------|
| `email` | Email address |
| `wallet` | Crypto wallet address |

### `dao_category`
Category classification for DAOs.

| Value | Description |
|-------|-------------|
| `defi` | DeFi protocol |
| `infrastructure` | Infrastructure / L2 |
| `public_goods` | Public goods / grants |
| `social` | Social / community |

### `confidence_level`
Data confidence level for GVS scores, driven by data completeness.

| Value | Description |
|-------|-------------|
| `high` | High confidence (sufficient data) |
| `medium` | Medium confidence |
| `low` | Low confidence (limited data) |

### `dao_badge_type_enum`
Badge types that can be assigned to DAOs.

| Value | Description |
|-------|-------------|
| `featured_analysis` | Featured analysis badge |
| `customer_report` | Customer report badge (paid report exists) |

### `engagement_status`
Tracking status for DAO engagement outreach.

| Value | Description |
|-------|-------------|
| `not_started` | No engagement initiated |
| `active` | Actively engaged |
| `warm` | Warm lead, positive signals |
| `pending` | Awaiting response |
| `blocked` | Engagement blocked or unresponsive |

### `snapshot_type_enum`
Distinguishes daily from monthly aggregated snapshots.

| Value | Description |
|-------|-------------|
| `daily` | Daily snapshot |
| `monthly` | Monthly aggregated snapshot |

### `dgi_index_type`
Types of DGI (DAO Governance Index) indices.

| Value | Description |
|-------|-------------|
| `composite` | Overall composite index |
| `defi` | DeFi sector index |
| `infrastructure` | Infrastructure sector index |
| `public_goods` | Public goods sector index |
| `social` | Social sector index |
| `hpr` | Human Participation Rate sub-index |
| `dei` | Delegate Engagement Index sub-index |
| `pdi` | Power Dynamics Index sub-index |
| `gpi` | Grassroots Participation Index sub-index |

### `job_status`
Execution state for background jobs.

| Value | Description |
|-------|-------------|
| `running` | Job currently executing |
| `completed` | Job finished successfully |
| `failed` | Job failed |

### `opt_out_status`
Status for DAO opt-out requests from public rankings.

| Value | Description |
|-------|-------------|
| `pending` | Request submitted, awaiting processing |
| `processed` | Request has been processed |

### `donation_status`
Transfer state for donation tracking.

| Value | Description |
|-------|-------------|
| `confirmed` | Donation confirmed from payment |
| `transferred` | Donation transferred to recipient |

### `subscription_status`
Stripe subscription lifecycle states.

| Value | Description |
|-------|-------------|
| `active` | Subscription active |
| `canceled` | Subscription canceled |
| `past_due` | Payment past due |
| `unpaid` | Subscription unpaid |
| `trialing` | In trial period |

### `subscription_product`
Products available for subscription.

| Value | Description |
|-------|-------------|
| `matrix` | DAO Matrix access |
| `api` | API access (future) |

### `user_role`
User roles for authentication and authorization.

| Value | Description |
|-------|-------------|
| `admin` | Administrator |
| `free` | Free tier user |
| `subscriber` | Paid subscriber |

### `engagement_log_type`
Types of engagement interactions logged.

| Value | Description |
|-------|-------------|
| `forum_post` | Posted on governance forum |
| `forum_reply` | Replied on governance forum |
| `x_reply` | Replied on X/Twitter |
| `x_post` | Posted on X/Twitter |
| `discord` | Discord interaction |
| `dm` | Direct message |
| `note` | Internal note |

### `engagement_log_platform`
Platforms where engagement occurs.

| Value | Description |
|-------|-------------|
| `x` | X / Twitter |
| `forum` | Governance forum |
| `discord` | Discord |
| `telegram` | Telegram |
| `farcaster` | Farcaster |
| `other` | Other platform |

### `engagement_log_outcome`
Outcome of an engagement interaction.

| Value | Description |
|-------|-------------|
| `pending` | Awaiting response |
| `no_response` | No response received |
| `replied` | Received a reply |
| `positive` | Positive outcome |
| `negative` | Negative outcome |
| `blocked` | Communication blocked |

### `device_type`
Device type for feedback tracking.

| Value | Description |
|-------|-------------|
| `mobile` | Mobile device |
| `desktop` | Desktop browser |
| `tablet` | Tablet device |

### `signal_source`
Source of governance signals.

| Value | Description |
|-------|-------------|
| `forum_rss` | Discourse/Commonwealth RSS feeds (Phase 3b) |
| `x_search` | X/Twitter API search (Phase 3a, blocked) |
| `discord_webhook` | Discord bot webhook (Phase 3c, future) |

### `signal_type`
Classification of governance signals.

| Value | Description |
|-------|-------------|
| `governance_discussion` | General governance topic |
| `delegation` | Delegation-related discussion |
| `frustration` | Frustration with governance |
| `question` | Question about governance |
| `proposal` | New proposal or temp check |
| `mention` | ChainSights or DGI mentioned |
| `info` | Info dumps (updates, reports, summaries) -- max relevance score 3.5 |

### `signal_priority`
Priority levels for signals.

| Value | Description |
|-------|-------------|
| `high` | Immediate action recommended |
| `medium` | Good opportunity |
| `low` | Nice to have |

### `signal_status`
Processing status for signals.

| Value | Description |
|-------|-------------|
| `new` | Just detected |
| `reviewed` | Admin has seen it |
| `acted` | Action taken (reply sent, etc.) |
| `dismissed` | Not relevant / ignored |

---

## Core Tables

### `daos`

**Purpose:** Master table storing all tracked DAOs, their metadata, community links, and engagement tracking fields.

| Column | Type | Constraints | Default | Description |
|--------|------|-------------|---------|-------------|
| `id` | `uuid` | PK | `gen_random_uuid()` | Primary key |
| `slug` | `text` | UNIQUE, NOT NULL | -- | URL-friendly identifier derived from Snapshot space (e.g., `radiantcapital-eth`) |
| `slug_alias` | `varchar(100)` | -- | `NULL` | Human-friendly vanity URL alias (e.g., `radiant-capital`) (Story 21-5) |
| `name` | `text` | NOT NULL | -- | Display name |
| `category` | `dao_category` | NOT NULL | -- | DAO category (defi, infrastructure, public_goods, social) |
| `snapshot_space` | `text` | NOT NULL | -- | Snapshot.org space ID |
| `badge_type` | `dao_badge_type_enum` | -- | `NULL` | Badge: null (none), `featured_analysis`, or `customer_report` |
| `is_opted_out` | `boolean` | NOT NULL | `false` | Whether DAO opted out of public rankings |
| `discord_url` | `text` | -- | `NULL` | Discord invite link (UTM added at display time) |
| `forum_url` | `text` | -- | `NULL` | Governance forum URL (UTM added at display time) |
| `twitter_url` | `text` | -- | `NULL` | Twitter/X handle URL (UTM added at display time) |
| `twitter_handle` | `varchar(100)` | -- | `NULL` | Twitter/X handle without @ (Engagement Hub) |
| `governance_forum_url` | `varchar(500)` | -- | `NULL` | Dedicated governance forum if different from `forum_url` |
| `telegram_url` | `varchar(500)` | -- | `NULL` | Telegram group URL |
| `farcaster_url` | `varchar(500)` | -- | `NULL` | Farcaster profile/channel URL |
| `engagement_status` | `engagement_status` | -- | `not_started` | Engagement tracking status |
| `engagement_notes` | `text` | -- | `NULL` | Free-text context notes for engagement |
| `last_engagement_at` | `timestamp` | -- | `NULL` | Last engagement interaction timestamp |
| `created_at` | `timestamp` | NOT NULL | `now()` | Row creation timestamp |
| `updated_at` | `timestamp` | NOT NULL | `now()` | Last update timestamp |

**Indexes:**
- `daos_slug_idx` -- UNIQUE on `slug`
- `daos_slug_alias_idx` -- on `slug_alias` (vanity URL lookups)
- `idx_dao_badge_type` -- on `badge_type`
- `idx_dao_engagement_status` -- on `engagement_status`

**Referenced by:** `gvs_snapshots.dao_id`, `dao_index_membership.dao_id`, `engagement_log.dao_id`, `signals.dao_id`

---

### `gvs_snapshots`

**Purpose:** Stores historical Governance Vitality Score (GVS) snapshots for each DAO. Supports both daily and monthly aggregated snapshots for retention policies.

| Column | Type | Constraints | Default | Description |
|--------|------|-------------|---------|-------------|
| `id` | `uuid` | PK | `gen_random_uuid()` | Primary key |
| `dao_id` | `uuid` | NOT NULL, FK | -- | References `daos.id` (ON DELETE CASCADE) |
| `gvs_score` | `numeric` | NOT NULL | -- | Governance Vitality Score 0-10 (weighted: HPR 35%, DEI 25%, PDI 20%, GPI 20%) |
| `hpr_score` | `numeric` | -- | `NULL` | Human Participation Rate component (0-10) |
| `dei_score` | `numeric` | -- | `NULL` | Delegate Engagement Index component (0-10) |
| `pdi_score` | `numeric` | -- | `NULL` | Power Dynamics Index component (0-10) |
| `gpi_score` | `numeric` | -- | `NULL` | Grassroots Participation Index component (0-10) |
| `nakamoto_coefficient` | `integer` | -- | `NULL` | Voting power concentration metric (from PDI calculation) |
| `confidence_level` | `confidence_level` | NOT NULL | -- | Data confidence (high, medium, low) |
| `data_completeness` | `numeric` | NOT NULL | -- | Percentage 0-100 (drives confidence level) |
| `snapshot_type` | `text` | -- | `'daily'` | `daily` or `monthly` for retention policies |
| `monthly_avg_score` | `numeric(4,2)` | -- | `NULL` | Average of all daily scores in month (monthly snapshots only) |
| `monthly_min_score` | `numeric(4,2)` | -- | `NULL` | Minimum score of the month (monthly snapshots only) |
| `monthly_max_score` | `numeric(4,2)` | -- | `NULL` | Maximum score of the month (monthly snapshots only) |
| `snapshot_date` | `timestamp` | NOT NULL | -- | Date of the snapshot |
| `created_at` | `timestamp` | NOT NULL | `now()` | Row creation timestamp |
| `updated_at` | `timestamp` | NOT NULL | `now()` | Last update timestamp |

**Indexes:**
- `gvs_snapshots_dao_id_idx` -- on `dao_id`
- `gvs_snapshots_snapshot_date_idx` -- on `snapshot_date`
- `gvs_snapshots_dao_date_idx` -- compound on `(dao_id, snapshot_date DESC)` for historical queries
- `idx_gvs_snapshots_type_date` -- on `(snapshot_type, snapshot_date)` for cleanup queries

**Foreign Keys:**
- `dao_id` -> `daos.id` (ON DELETE CASCADE)

---

### `reported_daos`

**Purpose:** Tracks which DAOs have been analyzed via the report system (legacy from pre-GVS era). Also caches Free Check results for the Open-Universe feature.

| Column | Type | Constraints | Default | Description |
|--------|------|-------------|---------|-------------|
| `space_id` | `text` | PK | -- | Primary key (Snapshot space ID) |
| `name` | `text` | NOT NULL | -- | DAO name |
| `reports_generated` | `integer` | NOT NULL | `1` | Number of reports generated |
| `last_score` | `integer` | -- | `NULL` | Most recent score |
| `avg_score` | `real` | -- | `NULL` | Average score across reports |
| `total_score_sum` | `integer` | -- | `NULL` | Sum of all scores (for running average) |
| `last_reported_at` | `timestamp` | -- | `now()` | Last report generation time |
| `created_at` | `timestamp` | -- | `now()` | Row creation timestamp |
| `last_free_check_at` | `timestamp` | -- | `NULL` | Last Free Check timestamp (Open-Universe) |
| `free_check_count` | `integer` | -- | `0` | Number of free checks performed |
| `confidence_level` | `varchar(10)` | -- | `NULL` | Data confidence level |
| `proposal_count` | `integer` | -- | `NULL` | Number of proposals found |
| `cached_gvs` | `jsonb` | -- | `NULL` | Cached GVS calculation results |

---

## Reports Tables

### `orders`

**Purpose:** Tracks all orders for governance reports, supporting both Stripe and crypto payments. Includes DAO prefill data for abandoned cart recovery flows.

| Column | Type | Constraints | Default | Description |
|--------|------|-------------|---------|-------------|
| `id` | `uuid` | PK | `gen_random_uuid()` | Primary key |
| `payment_method` | `payment_method` | NOT NULL | `'stripe'` | Payment method (stripe or crypto) |
| `stripe_session_id` | `text` | UNIQUE | `NULL` | Stripe Checkout session ID |
| `stripe_payment_intent` | `text` | -- | `NULL` | Stripe PaymentIntent ID |
| `stripe_customer_id` | `text` | -- | `NULL` | Stripe Customer ID |
| `crypto_tx_hash` | `text` | UNIQUE | `NULL` | Crypto transaction hash |
| `crypto_wallet_address` | `text` | -- | `NULL` | Crypto wallet address |
| `customer_email` | `text` | NOT NULL | -- | Customer email address |
| `customer_name` | `text` | -- | `NULL` | Customer name |
| `tier` | `tier` | NOT NULL | `'standard'` | Report tier (standard, deep_dive, governance_audit) |
| `status` | `order_status` | NOT NULL | `'pending'` | Order lifecycle status |
| `amount_cents` | `integer` | NOT NULL | -- | Order amount in cents |
| `currency` | `text` | NOT NULL | `'eur'` | Currency code |
| `dao_name` | `text` | -- | `NULL` | DAO name (prefill for recovery) |
| `dao_slug` | `text` | -- | `NULL` | DAO slug (prefill for recovery) |
| `snapshot_space` | `text` | -- | `NULL` | Snapshot space (prefill for recovery) |
| `recovery_email_sent_at` | `timestamp` | -- | `NULL` | When recovery email was sent |
| `created_at` | `timestamp` | NOT NULL | `now()` | Order creation timestamp |
| `updated_at` | `timestamp` | NOT NULL | `now()` | Last update timestamp |

**Referenced by:** `reports.order_id`, `share_rewards.order_id`

---

### `reports`

**Purpose:** The actual governance reports. Contains the full pipeline state from data fetching through AI analysis to PDF generation, including revision/decline loops and secure download tokens.

| Column | Type | Constraints | Default | Description |
|--------|------|-------------|---------|-------------|
| `id` | `uuid` | PK | `gen_random_uuid()` | Primary key |
| `order_id` | `uuid` | NOT NULL, FK | -- | References `orders.id` |
| `dao_name` | `text` | NOT NULL | -- | DAO name from intake form |
| `snapshot_space` | `text` | NOT NULL | -- | Snapshot.org space ID (e.g., `ens.eth`) |
| `chain` | `text` | NOT NULL | `'ethereum'` | Blockchain network |
| `customer_email` | `text` | NOT NULL | -- | Customer email |
| `customer_name` | `text` | -- | `NULL` | Customer name |
| `raw_data` | `jsonb` | -- | `NULL` | Raw Snapshot API response |
| `ai_analysis` | `jsonb` | -- | `NULL` | Claude AI analysis output |
| `draft_content` | `jsonb` | -- | `NULL` | Editable draft fields |
| `final_pdf_url` | `text` | -- | `NULL` | URL to generated PDF |
| `version` | `integer` | NOT NULL | `1` | Current version (increments on decline/regenerate) |
| `revision_history` | `jsonb` | -- | `NULL` | Array of `{version, feedback, declinedAt}` |
| `pdf_token` | `text` | -- | `NULL` | Secure token for PDF download |
| `json_token` | `text` | -- | `NULL` | Secure token for JSON download |
| `token_expires_at` | `timestamp` | -- | `NULL` | Download token expiration |
| `status` | `report_status` | NOT NULL | `'pending'` | Report pipeline status |
| `error_message` | `text` | -- | `NULL` | Error details if failed |
| `validation_status` | `text` | -- | `NULL` | Validation result: `passed`, `failed`, or `warnings` |
| `validation_result` | `jsonb` | -- | `NULL` | Full ValidationResult object |
| `input_tokens` | `integer` | -- | `NULL` | AI input token usage |
| `output_tokens` | `integer` | -- | `NULL` | AI output token usage |
| `created_at` | `timestamp` | NOT NULL | `now()` | Report creation timestamp |
| `updated_at` | `timestamp` | NOT NULL | `now()` | Last update timestamp |
| `fetched_at` | `timestamp` | -- | `NULL` | When data was fetched |
| `analyzed_at` | `timestamp` | -- | `NULL` | When AI analysis completed |
| `approved_at` | `timestamp` | -- | `NULL` | When report was approved |
| `generated_at` | `timestamp` | -- | `NULL` | When PDF was generated |
| `delivered_at` | `timestamp` | -- | `NULL` | When report was delivered |

**Foreign Keys:**
- `order_id` -> `orders.id`

**Referenced by:** `share_rewards.report_id`

---

### `quick_checks`

**Purpose:** Caches Quick Check (free tier) results per email+DAO combination. Used for rate limiting and preventing duplicate computations.

| Column | Type | Constraints | Default | Description |
|--------|------|-------------|---------|-------------|
| `id` | `uuid` | PK | `gen_random_uuid()` | Primary key |
| `email` | `text` | NOT NULL | -- | User email |
| `dao_slug` | `text` | NOT NULL | -- | DAO slug identifier |
| `gvs_score` | `numeric(5,2)` | -- | `NULL` | Overall GVS score |
| `hpr_score` | `numeric(5,2)` | -- | `NULL` | Human Participation Rate |
| `dei_score` | `numeric(5,2)` | -- | `NULL` | Delegate Engagement Index |
| `pdi_score` | `numeric(5,2)` | -- | `NULL` | Power Dynamics Index |
| `gpi_score` | `numeric(5,2)` | -- | `NULL` | Grassroots Participation Index |
| `is_on_the_fly` | `boolean` | -- | `false` | Whether this was an on-the-fly calculation |
| `created_at` | `timestamp` | NOT NULL | `now()` | Creation timestamp |

**Indexes:**
- `quick_checks_email_dao_idx` -- UNIQUE on `(email, dao_slug)`
- `idx_quick_checks_rate_limit` -- on `(dao_slug, created_at)` for rate limit queries

---

## Auth Tables

### `users`

**Purpose:** Lightweight user table for magic link authentication and role management.

| Column | Type | Constraints | Default | Description |
|--------|------|-------------|---------|-------------|
| `id` | `uuid` | PK | `gen_random_uuid()` | Primary key |
| `email` | `text` | NOT NULL, UNIQUE | -- | User email address |
| `role` | `user_role` | NOT NULL | `'free'` | User role (admin, free, subscriber) |
| `marketing_opt_in` | `boolean` | NOT NULL | `false` | Marketing consent flag |
| `first_auth_at` | `timestamp` | NOT NULL | `now()` | First authentication timestamp |
| `last_auth_at` | `timestamp` | NOT NULL | `now()` | Most recent authentication |
| `created_at` | `timestamp` | NOT NULL | `now()` | Row creation timestamp |

**Indexes:**
- `users_email_idx` -- UNIQUE on `email`

---

### `auth_tokens`

**Purpose:** Short-lived magic link tokens for passwordless authentication. Tokens are single-use and expire after a set time.

| Column | Type | Constraints | Default | Description |
|--------|------|-------------|---------|-------------|
| `id` | `uuid` | PK | `gen_random_uuid()` | Primary key |
| `email` | `text` | NOT NULL | -- | Email the token was sent to |
| `token` | `text` | NOT NULL, UNIQUE | -- | Magic link token value |
| `redirect_url` | `text` | -- | `NULL` | URL to redirect after authentication |
| `marketing_opt_in` | `boolean` | NOT NULL | `false` | Marketing consent at time of request |
| `expires_at` | `timestamp` | NOT NULL | -- | Token expiration time |
| `used_at` | `timestamp` | -- | `NULL` | When the token was consumed |
| `created_at` | `timestamp` | NOT NULL | `now()` | Token creation timestamp |

**Indexes:**
- `auth_tokens_token_idx` -- UNIQUE on `token`
- `auth_tokens_email_idx` -- on `email`

---

## Payments Tables

### `donations`

**Purpose:** Tracks the 3% donation from all payment sources (Stripe + Coinbase). Supports multi-project donation tracking.

| Column | Type | Constraints | Default | Description |
|--------|------|-------------|---------|-------------|
| `id` | `uuid` | PK | `gen_random_uuid()` | Primary key |
| `order_id` | `text` | NOT NULL | -- | Reference to the originating order |
| `payment_provider` | `text` | NOT NULL | -- | `stripe` or `coinbase` |
| `payment_id` | `text` | NOT NULL | -- | Stripe `payment_intent_id` or Coinbase `charge_id` |
| `order_amount` | `numeric(10,2)` | NOT NULL | -- | Total order amount |
| `donation_amount` | `numeric(10,2)` | NOT NULL | -- | Calculated donation amount (3% of order) |
| `currency` | `text` | NOT NULL | `'EUR'` | Currency code |
| `project` | `text` | NOT NULL | `'chainsights'` | Project identifier (multi-project support) |
| `customer_email` | `text` | -- | `NULL` | Customer email |
| `dao_name` | `text` | -- | `NULL` | DAO name from the order |
| `status` | `donation_status` | NOT NULL | `'confirmed'` | Transfer status (confirmed, transferred) |
| `transferred_at` | `timestamp` | -- | `NULL` | When donation was transferred |
| `created_at` | `timestamp` | NOT NULL | `now()` | Creation timestamp |
| `updated_at` | `timestamp` | NOT NULL | `now()` | Last update timestamp |

**Indexes:**
- `idx_donations_created_at` -- on `created_at`
- `idx_donations_project` -- on `project`
- `idx_donations_status` -- on `status`
- `idx_donations_payment_id` -- UNIQUE on `(payment_provider, payment_id)` (prevents duplicate donations)

---

### `subscriptions`

**Purpose:** Tracks Stripe recurring subscriptions for product access (DAO Matrix, future API access).

| Column | Type | Constraints | Default | Description |
|--------|------|-------------|---------|-------------|
| `id` | `uuid` | PK | `gen_random_uuid()` | Primary key |
| `email` | `text` | NOT NULL | -- | Subscriber email |
| `stripe_customer_id` | `text` | NOT NULL | -- | Stripe Customer ID |
| `stripe_subscription_id` | `text` | NOT NULL, UNIQUE | -- | Stripe Subscription ID |
| `product` | `subscription_product` | NOT NULL | -- | Product type (matrix, api) |
| `stripe_price_id` | `text` | NOT NULL | -- | Stripe Price ID |
| `status` | `subscription_status` | NOT NULL | `'active'` | Subscription status |
| `current_period_start` | `timestamp` | -- | `NULL` | Current billing period start |
| `current_period_end` | `timestamp` | -- | `NULL` | Current billing period end |
| `cancel_at_period_end` | `boolean` | NOT NULL | `false` | Whether subscription cancels at period end |
| `created_at` | `timestamp` | NOT NULL | `now()` | Creation timestamp |
| `updated_at` | `timestamp` | NOT NULL | `now()` | Last update timestamp |

**Indexes:**
- `idx_subscriptions_email` -- on `email`
- `idx_subscriptions_stripe_sub` -- UNIQUE on `stripe_subscription_id`
- `idx_subscriptions_stripe_customer` -- on `stripe_customer_id`
- `idx_subscriptions_product_status` -- on `(product, status)`

---

### `share_rewards`

**Purpose:** Tracks Share & Save cashback program. Customers share their report on Twitter and receive a cashback reward after verification.

| Column | Type | Constraints | Default | Description |
|--------|------|-------------|---------|-------------|
| `id` | `uuid` | PK | `gen_random_uuid()` | Primary key |
| `order_id` | `uuid` | NOT NULL, FK | -- | References `orders.id` |
| `report_id` | `uuid` | FK | `NULL` | References `reports.id` |
| `tweet_url` | `text` | NOT NULL | -- | URL of the tweet |
| `twitter_handle` | `text` | -- | `NULL` | Twitter handle |
| `follower_count` | `integer` | -- | `NULL` | Follower count at time of submission |
| `engagement` | `integer` | -- | `NULL` | Engagement count (likes + retweets) |
| `sentiment` | `sentiment` | -- | `NULL` | Tweet sentiment (positive, mixed, critical) |
| `verified` | `boolean` | NOT NULL | `false` | Whether the tweet was verified by admin |
| `verified_at` | `timestamp` | -- | `NULL` | Verification timestamp |
| `verified_by` | `text` | -- | `NULL` | Admin who verified |
| `reward_amount_cents` | `integer` | NOT NULL | `2000` | Reward amount in cents (default: 20 EUR) |
| `reward_status` | `reward_status` | NOT NULL | `'pending'` | Reward payout status |
| `stripe_refund_id` | `text` | -- | `NULL` | Stripe refund ID for cashback |
| `paid_at` | `timestamp` | -- | `NULL` | When cashback was paid |
| `notes` | `text` | -- | `NULL` | Admin notes |
| `created_at` | `timestamp` | NOT NULL | `now()` | Creation timestamp |
| `updated_at` | `timestamp` | NOT NULL | `now()` | Last update timestamp |

**Foreign Keys:**
- `order_id` -> `orders.id`
- `report_id` -> `reports.id`

---

## Engagement Tables

### `engagement_log`

**Purpose:** Tracks all interactions with DAOs as part of the Engagement Hub (Epic 19). Each log entry represents a single outreach action (forum post, tweet reply, DM, etc.).

| Column | Type | Constraints | Default | Description |
|--------|------|-------------|---------|-------------|
| `id` | `uuid` | PK | `gen_random_uuid()` | Primary key |
| `dao_id` | `uuid` | NOT NULL, FK | -- | References `daos.id` (ON DELETE CASCADE) |
| `type` | `engagement_log_type` | NOT NULL | -- | Interaction type (forum_post, x_reply, etc.) |
| `platform` | `engagement_log_platform` | NOT NULL | -- | Platform (x, forum, discord, etc.) |
| `content` | `text` | -- | `NULL` | Description of the interaction |
| `url` | `varchar(500)` | -- | `NULL` | Link to the post/reply |
| `contact_name` | `varchar(100)` | -- | `NULL` | Name of the person contacted |
| `outcome` | `engagement_log_outcome` | NOT NULL | `'pending'` | Interaction outcome |
| `created_at` | `timestamp` | NOT NULL | `now()` | Creation timestamp |

**Indexes:**
- `idx_engagement_log_dao_id` -- on `dao_id`
- `idx_engagement_log_created_at` -- on `created_at`
- `idx_engagement_log_dao_created` -- compound on `(dao_id, created_at DESC)`

**Foreign Keys:**
- `dao_id` -> `daos.id` (ON DELETE CASCADE)

---

### `signals`

**Purpose:** Stores detected governance engagement opportunities from external sources (forum RSS feeds, future X search and Discord webhooks). Part of Phase 3 Signal Scanner.

| Column | Type | Constraints | Default | Description |
|--------|------|-------------|---------|-------------|
| `id` | `uuid` | PK | `gen_random_uuid()` | Primary key |
| `dao_id` | `uuid` | NOT NULL, FK | -- | References `daos.id` |
| `source` | `signal_source` | NOT NULL | -- | Signal source (forum_rss, x_search, discord_webhook) |
| `signal_type` | `signal_type` | NOT NULL | -- | Signal classification |
| `priority` | `signal_priority` | NOT NULL | `'medium'` | Priority level (high, medium, low) |
| `status` | `signal_status` | NOT NULL | `'new'` | Processing status (new, reviewed, acted, dismissed) |
| `title` | `varchar(500)` | NOT NULL | -- | Signal title / headline |
| `content` | `text` | -- | `NULL` | Full text or excerpt |
| `url` | `varchar(500)` | NOT NULL, UNIQUE | -- | URL to original content |
| `author` | `varchar(100)` | -- | `NULL` | Author of the original content |
| `hook_suggestion` | `text` | -- | `NULL` | AI-generated engagement suggestion (Phase 4) |
| `relevance_score` | `real` | -- | `NULL` | AI relevance score 0-1 (Phase 4) |
| `published_at` | `timestamp` | -- | `NULL` | When the original content was posted |
| `detected_at` | `timestamp` | NOT NULL | `now()` | When the signal was detected |
| `reviewed_at` | `timestamp` | -- | `NULL` | When admin reviewed |
| `acted_at` | `timestamp` | -- | `NULL` | When action was taken |

**Indexes:**
- `idx_signals_dao_id` -- on `dao_id`
- `idx_signals_status` -- on `status`
- `idx_signals_priority_status` -- on `(priority, status)`
- `idx_signals_detected_at` -- on `detected_at DESC`
- `idx_signals_url_unique` -- UNIQUE on `url` (prevents duplicate signals)

**Foreign Keys:**
- `dao_id` -> `daos.id`

---

### `feedback`

**Purpose:** Stores anonymous user feedback submitted via the floating feedback bubble (Epic 17.5). No authentication required.

| Column | Type | Constraints | Default | Description |
|--------|------|-------------|---------|-------------|
| `id` | `uuid` | PK | `gen_random_uuid()` | Primary key |
| `message` | `text` | NOT NULL | -- | Feedback message (max 500 chars, validated in API) |
| `page` | `varchar(200)` | NOT NULL | -- | Page path where feedback was submitted |
| `referrer` | `varchar(500)` | -- | `NULL` | Referrer URL |
| `country` | `varchar(5)` | -- | `NULL` | Country code (from Vercel geo headers) |
| `device` | `device_type` | -- | `NULL` | Device type (mobile, desktop, tablet) |
| `placeholder_shown` | `varchar(200)` | -- | `NULL` | Which placeholder text was displayed |
| `created_at` | `timestamp` | NOT NULL | `now()` | Submission timestamp |

**Indexes:**
- `idx_feedback_created_at` -- on `created_at`
- `idx_feedback_page` -- on `page`

---

## DGI Tables

### `dgi_snapshots`

**Purpose:** Stores daily DAO Governance Index values. The DGI is a composite index (and sub-indices by category and metric) that aggregates GVS scores across all tracked DAOs.

| Column | Type | Constraints | Default | Description |
|--------|------|-------------|---------|-------------|
| `id` | `uuid` | PK | `gen_random_uuid()` | Primary key |
| `snapshot_date` | `timestamp` | NOT NULL | -- | Date of the index snapshot |
| `index_type` | `dgi_index_type` | NOT NULL | -- | Index type (composite, defi, infrastructure, etc.) |
| `value` | `numeric(5,2)` | NOT NULL | -- | Index value |
| `dao_count` | `integer` | NOT NULL | -- | Number of DAOs included in calculation |
| `created_at` | `timestamp` | NOT NULL | `now()` | Creation timestamp |

**Indexes:**
- `dgi_snapshots_date_type_idx` -- UNIQUE on `(snapshot_date, index_type)`
- `dgi_snapshots_type_idx` -- on `index_type`

---

### `dao_index_membership`

**Purpose:** Tracks which DAOs are included in the DGI universe, with history of inclusions and exclusions. Supports grace periods for temporarily excluded DAOs.

| Column | Type | Constraints | Default | Description |
|--------|------|-------------|---------|-------------|
| `id` | `uuid` | PK | `gen_random_uuid()` | Primary key |
| `dao_id` | `uuid` | NOT NULL, FK | -- | References `daos.id` (ON DELETE CASCADE) |
| `category` | `dao_category` | NOT NULL | -- | DAO category at time of inclusion |
| `included_since` | `timestamp` | NOT NULL | `now()` | When the DAO was added to the index |
| `excluded_at` | `timestamp` | -- | `NULL` | When the DAO was removed (NULL = active) |
| `exclusion_reason` | `text` | -- | `NULL` | Reason for exclusion |
| `grace_period_until` | `timestamp` | -- | `NULL` | Manual note field for grace period end |
| `created_at` | `timestamp` | NOT NULL | `now()` | Creation timestamp |

**Indexes:**
- `dao_index_membership_dao_id_idx` -- on `dao_id`
- `dao_index_membership_active_idx` -- on `(dao_id, excluded_at)` for filtering active members

**Foreign Keys:**
- `dao_id` -> `daos.id` (ON DELETE CASCADE)

---

## Leads & Subscribers Tables

### `leads`

**Purpose:** Marketing email list with explicit GDPR consent. Aggregates lead data from free checks and paid reports.

| Column | Type | Constraints | Default | Description |
|--------|------|-------------|---------|-------------|
| `id` | `uuid` | PK | `gen_random_uuid()` | Primary key |
| `email` | `text` | NOT NULL, UNIQUE | -- | Lead email address |
| `marketing_opt_in` | `boolean` | NOT NULL | -- | Marketing consent flag |
| `marketing_opt_in_timestamp` | `timestamp` | NOT NULL | -- | When consent was given |
| `unsubscribed` | `boolean` | -- | `false` | Whether lead has unsubscribed |
| `unsubscribed_at` | `timestamp` | -- | `NULL` | When they unsubscribed |
| `last_activity_at` | `timestamp` | -- | `now()` | Last known activity |
| `free_checks_count` | `integer` | -- | `1` | Number of free checks performed |
| `paid_reports_count` | `integer` | -- | `0` | Number of paid reports purchased |
| `source` | `text` | -- | `'free_tier'` | Lead source |
| `created_at` | `timestamp` | NOT NULL | `now()` | Creation timestamp |
| `updated_at` | `timestamp` | NOT NULL | `now()` | Last update timestamp |

---

### `email_subscribers`

**Purpose:** Rankings Watch email subscription list. Users subscribe to receive periodic DAO rankings updates via double opt-in confirmation flow.

| Column | Type | Constraints | Default | Description |
|--------|------|-------------|---------|-------------|
| `id` | `uuid` | PK | `gen_random_uuid()` | Primary key |
| `email` | `text` | NOT NULL, UNIQUE | -- | Subscriber email |
| `subscribed_at` | `timestamp` | NOT NULL | `now()` | Subscription timestamp |
| `confirmed` | `boolean` | NOT NULL | `false` | Double opt-in confirmed |
| `confirmation_token` | `text` | UNIQUE | `NULL` | Token for email confirmation |
| `confirmation_sent_at` | `timestamp` | -- | `NULL` | When confirmation email was sent |
| `unsubscribe_token` | `text` | NOT NULL, UNIQUE | -- | Token for one-click unsubscribe |
| `unsubscribed` | `boolean` | NOT NULL | `false` | Whether subscriber has unsubscribed |
| `unsubscribed_at` | `timestamp` | -- | `NULL` | When they unsubscribed |
| `created_at` | `timestamp` | NOT NULL | `now()` | Creation timestamp |
| `updated_at` | `timestamp` | NOT NULL | `now()` | Last update timestamp |

**Indexes:**
- `idx_email_subscribers_email` -- on `email`
- `idx_email_subscribers_confirmed` -- on `confirmed`
- `idx_email_subscribers_unsubscribed` -- on `unsubscribed`

---

### `free_query_log`

**Purpose:** Tracks free tier (Quick Check) usage and quota enforcement. Stores per-user, per-DAO check results with GDPR compliance fields.

| Column | Type | Constraints | Default | Description |
|--------|------|-------------|---------|-------------|
| `id` | `uuid` | PK | `gen_random_uuid()` | Primary key |
| `identifier_hash` | `text` | NOT NULL | -- | SHA-256 hash of identifier (legacy, backward compat) |
| `identifier_type` | `identifier_type` | NOT NULL | -- | Type of identifier (email or wallet) |
| `email` | `text` | -- | `NULL` | Plain email for report delivery (NULL for wallet-only) |
| `dao_id` | `text` | NOT NULL | -- | DAO identifier |
| `rating_score` | `real` | NOT NULL | -- | GVS rating score |
| `gini_coefficient` | `real` | -- | `NULL` | Gini coefficient value |
| `nakamoto_coefficient` | `integer` | -- | `NULL` | Nakamoto coefficient value |
| `ranking_position` | `integer` | -- | `NULL` | Position in rankings |
| `consent_given` | `boolean` | NOT NULL | `false` | GDPR consent flag |
| `consent_timestamp` | `timestamp` | NOT NULL | -- | When consent was given |
| `marketing_opt_in` | `boolean` | -- | `false` | Marketing consent flag |
| `marketing_opt_in_timestamp` | `timestamp` | -- | `NULL` | When marketing consent was given |
| `created_at` | `timestamp` | NOT NULL | `now()` | Creation timestamp |
| `month_key` | `text` | NOT NULL | -- | `YYYY-MM` format for monthly quota tracking |
| `email_sent` | `boolean` | -- | `false` | Whether result email was sent |
| `email_sent_at` | `timestamp` | -- | `NULL` | When email was sent |
| `deletion_requested` | `boolean` | -- | `false` | GDPR deletion request flag |
| `deletion_requested_at` | `timestamp` | -- | `NULL` | When deletion was requested |
| `public_listing` | `boolean` | -- | `false` | Opt-in for public rankings |
| `listed_since` | `timestamp` | -- | `NULL` | When public listing started |

---

### `quiz_responses`

**Purpose:** Stores anonymous quiz responses from the product recommendation quiz. Used for analytics to understand user needs.

| Column | Type | Constraints | Default | Description |
|--------|------|-------------|---------|-------------|
| `id` | `uuid` | PK | `gen_random_uuid()` | Primary key |
| `answers` | `jsonb` | NOT NULL | -- | Quiz answers as JSON |
| `recommended_tier` | `text` | -- | `NULL` | Tier recommendation result |
| `completed` | `boolean` | NOT NULL | `true` | Whether quiz was completed |
| `referrer` | `text` | -- | `NULL` | Referrer URL |
| `created_at` | `timestamp` | NOT NULL | `now()` | Submission timestamp |

---

### `opt_out_requests`

**Purpose:** Tracks DAO opt-out requests from public rankings. DAOs can request to be removed from the leaderboard.

| Column | Type | Constraints | Default | Description |
|--------|------|-------------|---------|-------------|
| `id` | `uuid` | PK | `gen_random_uuid()` | Primary key |
| `dao_name` | `text` | NOT NULL | -- | Name of the DAO requesting opt-out |
| `contact_email` | `text` | NOT NULL | -- | Contact email for the request |
| `reason` | `text` | -- | `NULL` | Optional feedback / reason |
| `status` | `opt_out_status` | NOT NULL | `'pending'` | Request status (pending, processed) |
| `requested_at` | `timestamp` | NOT NULL | `now()` | When the request was submitted |
| `processed_at` | `timestamp` | -- | `NULL` | When the request was processed |
| `created_at` | `timestamp` | NOT NULL | `now()` | Creation timestamp |
| `updated_at` | `timestamp` | NOT NULL | `now()` | Last update timestamp |

**Indexes:**
- `opt_out_requests_email_idx` -- on `contact_email`
- `opt_out_requests_status_idx` -- on `status`

---

## Operations Tables

### `job_logs`

**Purpose:** Tracks all background job executions for admin monitoring. Records job duration, success/failure, per-DAO errors, and anomaly detection results.

| Column | Type | Constraints | Default | Description |
|--------|------|-------------|---------|-------------|
| `id` | `uuid` | PK | `gen_random_uuid()` | Primary key |
| `job_name` | `text` | NOT NULL | -- | Job identifier (e.g., `weekly-gvs-recalculation`) |
| `started_at` | `timestamp` | NOT NULL | -- | Job start timestamp |
| `completed_at` | `timestamp` | -- | `NULL` | Job completion timestamp |
| `status` | `job_status` | NOT NULL | `'running'` | Execution state (running, completed, failed) |
| `daos_processed` | `integer` | NOT NULL | `0` | Number of DAOs successfully processed |
| `daos_total` | `integer` | NOT NULL | `0` | Total number of DAOs to process |
| `errors` | `jsonb` | -- | `NULL` | Array of `{daoId, daoName, error}` objects |
| `anomalies` | `jsonb` | -- | `NULL` | Array of anomaly detection objects (Story 12.4) |
| `duration_ms` | `integer` | -- | `NULL` | Job duration in milliseconds |
| `cache_invalidated` | `boolean` | NOT NULL | `false` | Whether cache was invalidated after job |
| `created_at` | `timestamp` | NOT NULL | `now()` | Creation timestamp |
| `updated_at` | `timestamp` | NOT NULL | `now()` | Last update timestamp |

**Indexes:**
- `job_logs_job_name_idx` -- on `job_name`
- `job_logs_started_at_idx` -- on `started_at`

**JSONB Types:**

`errors` array entries:
```typescript
interface JobErrorEntry {
  daoId: string
  daoName: string
  error: string
}
```

`anomalies` array entries:
```typescript
interface JobAnomalyEntry {
  daoId: string
  daoName: string
  snapshotSpace: string
  previousScore: number
  currentScore: number
  delta: number
  direction: 'up' | 'down'
  detectedAt: string    // ISO timestamp
  threshold: number
}
```

---

## Entity Relationship Summary

```
orders ──────< reports
orders ──────< share_rewards
reports ─────< share_rewards

daos ────────< gvs_snapshots
daos ────────< dao_index_membership
daos ────────< engagement_log
daos ────────< signals
```

Key relationships:
- An **order** can have one or more **reports** (version/regeneration loop)
- An **order** can have one or more **share_rewards**
- A **report** can have zero or more **share_rewards**
- A **DAO** has many **GVS snapshots** (daily + monthly history)
- A **DAO** has one active **index membership** record (with exclusion history)
- A **DAO** has many **engagement log** entries
- A **DAO** has many **signals** (detected governance opportunities)

---

## TypeScript Type Exports

The schema file exports inferred select and insert types for all tables:

| Table | Select Type | Insert Type |
|-------|-------------|-------------|
| `orders` | `Order` | `NewOrder` |
| `reports` | `Report` | `NewReport` |
| `share_rewards` | `ShareReward` | `NewShareReward` |
| `reported_daos` | `ReportedDao` | `NewReportedDao` |
| `quiz_responses` | `QuizResponse` | `NewQuizResponse` |
| `free_query_log` | `FreeQueryLog` | `NewFreeQueryLog` |
| `leads` | `Lead` | `NewLead` |
| `quick_checks` | `QuickCheck` | `QuickCheckInsert` |
| `opt_out_requests` | `OptOutRequest` | `NewOptOutRequest` |
| `daos` | `DAO` | `DAOInsert` |
| `gvs_snapshots` | `GVSSnapshot` | `GVSSnapshotInsert` |
| `dgi_snapshots` | `DGISnapshot` | `DGISnapshotInsert` |
| `dao_index_membership` | `DaoIndexMembership` | `DaoIndexMembershipInsert` |
| `job_logs` | `JobLog` | `JobLogInsert` |
| `email_subscribers` | `EmailSubscriber` | `EmailSubscriberInsert` |
| `donations` | `Donation` | `DonationInsert` |
| `subscriptions` | `Subscription` | `SubscriptionInsert` |
| `users` | `User` | `UserInsert` |
| `auth_tokens` | `AuthToken` | `AuthTokenInsert` |
| `feedback` | `Feedback` | `FeedbackInsert` |
| `engagement_log` | `EngagementLog` | `EngagementLogInsert` |
| `signals` | `Signal` | `SignalInsert` |

Additionally, the following union/literal types are exported:
- `EngagementStatus`
- `EngagementLogType`
- `EngagementLogPlatform`
- `EngagementLogOutcome`
- `SignalSource`
- `SignalType`
- `SignalPriority`
- `SignalStatus`
- `RevisionEntry` (interface for `reports.revision_history` JSONB)
