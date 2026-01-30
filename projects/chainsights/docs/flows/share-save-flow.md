# Share & Save €20 — Technical Flow Documentation

**Last Updated:** December 17, 2025
**Status:** Implemented & Live-Ready
**Purpose:** Marketing reference for the viral referral mechanism

---

## Overview

Share & Save is ChainSights' viral growth mechanism. Customers get €20 back when they tweet about their report experience. This turns every customer into a marketing channel while generating authentic social proof.

**Key Value Prop:** "Whatever you share — praise, critique, or somewhere in between — we honor the €20. Because truth is what we do."

---

## Customer Journey Flow

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│  Customer   │     │   Report    │     │  Customer   │     │    Admin    │
│   Buys      │ ──► │  Delivered  │ ──► │   Tweets    │ ──► │  Verifies   │
│  €99/€199   │     │  (w/ offer) │     │ @ChainSights│     │   Tweet     │
└─────────────┘     └─────────────┘     └─────────────┘     └─────────────┘
                                                                   │
                    ┌──────────────────────────────────────────────┘
                    │
                    ▼
            ┌─────────────┐     ┌─────────────┐
            │   Stripe    │     │  Customer   │
            │  Refunds    │ ──► │  Gets €20   │
            │    €20      │     │   Back!     │
            └─────────────┘     └─────────────┘
```

---

## Touchpoints

### 1. PDF Report — Page 4 (Last Page)

The offer appears on the final page of every Governance Truth Report:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

💰 SHARE & SAVE: Get €20 Back

Found this report valuable? Share your experience on Twitter,
mention @ChainSights_one, and get €20 back.

Whatever you share — praise, critique, or somewhere
in between — we'll honor the cashback.

Because truth is what we do.

Details in your delivery email.

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

**Code:** `src/lib/pdf/report-template.tsx` (Page 4)

---

### 2. Delivery Email

When the report is sent, the delivery email includes a prominent Share & Save CTA:

**Subject:** `Your Governance Truth Report for [DAO Name] is ready`

**CTA Button:** "🎁 Claim Your €20 →" → links to `https://chainsights.one/share`

**Copy highlights:**
- "Share your experience on X/Twitter and get €20 back"
- "Whatever you share — praise, critique, or somewhere in between — we honor it"
- "Because truth is what we do"

**Code:** `src/lib/email/index.ts` — `sendReportEmail()`

---

### 3. Reminder Email (24h After Delivery)

A follow-up email is sent 24 hours after report delivery:

**Subject:** `Still thinking about your report? Get €20 back.`

**Content:**
1. Reminder of their report
2. 3-step process to claim
3. Share ideas (prompts, not templates)
4. CTA button to claim page

**Code:** `src/lib/email/index.ts` — `sendShareReminderEmail()`

---

## Claim Flow — User Experience

### Step 1: Customer Visits `/share` Page

URL: `https://chainsights.one/share`

**Page includes:**
- "How It Works" — 3-step visual guide
- Requirements checklist
- One-click tweet template button (pre-filled tweet)
- Claim form (email + tweet URL)

**Tweet Template (Pre-filled):**
```
Just got my DAO governance report from @ChainSights_one - eye-opening insights about voting power distribution and participation trends!

If you're running a DAO, this is a must-have.

https://chainsights.one
```

**Code:** `src/app/share/page.tsx`

---

### Step 2: Customer Submits Claim Form

**Required fields:**
- Email (must match purchase email)
- Tweet URL (must be valid x.com or twitter.com status URL)

**Validation rules:**
1. Email must exist in orders table
2. Report must be delivered (`deliveredAt` not null)
3. Tweet URL not already claimed
4. Only one claim per order

**Success message:** "We'll verify your tweet and process your €20 refund within 48 hours."

**Code:** `src/app/api/share-claim/route.ts`

---

### Step 3: Admin Verification

**Admin Dashboard:** `/admin`

Shows "Pending Share Claims" card with:
- Customer email
- Twitter handle (auto-extracted from URL)
- Link to view tweet
- Claim date

**Verification criteria (manual, ~30 sec):**
- Tweet exists and is public
- Mentions @ChainSights_one
- References governance/analytics/ChainSights
- Not spam/bot behavior

**Note:** Negative tweets still qualify — authenticity over praise.

**Code:** `src/app/admin/page.tsx` — Pending Claims section

---

### Step 4: Stripe Refund

Admin processes €20 partial refund via Stripe dashboard.

**Future:** Mark claim as verified in admin panel (update `shareRewards.verified = true`)

---

## Database Schema

**Table:** `share_rewards`

| Column | Type | Description |
|--------|------|-------------|
| `id` | UUID | Primary key |
| `orderId` | UUID | FK to orders |
| `reportId` | UUID | FK to reports |
| `tweetUrl` | text | The submitted tweet URL |
| `twitterHandle` | text | Auto-extracted from URL |
| `followerCount` | int | (future) For tiered rewards |
| `engagement` | int | (future) Likes + retweets |
| `sentiment` | enum | (future) positive/neutral/negative |
| `verified` | boolean | Admin verified? |
| `verifiedAt` | timestamp | When verified |
| `rewardAmountCents` | int | Default: 2000 (€20) |
| `rewardStatus` | enum | pending/approved/paid/rejected |
| `stripeRefundId` | text | Stripe refund ID |
| `paidAt` | timestamp | When refund processed |
| `createdAt` | timestamp | Claim submitted |

**Code:** `src/lib/db/schema.ts`

---

## URLs & Endpoints

| URL | Purpose |
|-----|---------|
| `/share` | Customer claim form |
| `/api/share-claim` | POST: Submit claim |
| `/admin` | Admin dashboard (shows pending claims) |

---

## Key Files

| File | Purpose |
|------|---------|
| `src/app/share/page.tsx` | Claim page UI |
| `src/app/api/share-claim/route.ts` | Claim submission API |
| `src/lib/db/schema.ts` | `shareRewards` table |
| `src/lib/email/index.ts` | Delivery + reminder emails |
| `src/lib/pdf/report-template.tsx` | Page 4 share offer |
| `src/app/admin/page.tsx` | Pending claims view |

---

## Brand Policy

> **"Whatever you share — praise, critique, or somewhere in between — we'll honor the cashback."**

This is brand-defining. It communicates:
- Confidence in product quality
- Alignment with "Wallets lie. We don't."
- Authentic testimonials > polished marketing

---

## Metrics to Track (Future)

| Metric | Target | Why |
|--------|--------|-----|
| Share Rate | 20-30% | Customer satisfaction |
| Time to Share | <48h | Is PDF placement working? |
| Referral Conversions | Track | Direct ROI |
| Sentiment Distribution | Monitor | Product feedback loop |

---

## Related Documentation

- Strategy doc: `docs/_masemIT/ChainSights_Share_Save_Strategy.md`
- Email templates: `docs/_masemIT/ChainSights_Email_Templates.md`
- Tweet templates: `docs/_masemIT/ChainSights_Tweet_Templates.md`

---

*Generated: December 17, 2025*
