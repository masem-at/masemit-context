# Tech Spec: ChainSights Semi-Automated Pipeline

**Date:** December 15, 2025
**Author:** Mario (masemIT e.U.)
**Status:** Approved for implementation
**Prerequisite:** Spike validation (Snapshot API data fetch)

---

## 1. Executive Summary

Build a semi-automated pipeline that transforms customer orders into delivered Governance Truth Reports. The pipeline fetches DAO governance data from Snapshot, analyzes it with Claude AI, generates a draft for manual review, and delivers the final PDF via email.

**Key Principle:** Manual review for MVP. Each report teaches us what the AI gets wrong.

---

## 2. Decisions Made

| Question | Decision | Rationale |
|----------|----------|-----------|
| **Data Source** | Snapshot first (100%) | 80%+ of DAOs use Snapshot; clean GraphQL API; off-chain = more voters = more data |
| **Manual Review** | Yes, admin dashboard with approve flow | Don't trust AI blindly; learn from each report; €100-200/hr effective rate is fine |
| **Report Template** | Start with skeleton, iterate after real data | Until we see real Snapshot data, we don't know what insights are interesting |

**V2 Consideration:** On-chain Governor DAOs (Compound, Uniswap) can be added later. Bigger DAOs, but slower sales cycle.

---

## 3. Pipeline Architecture

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│   Payment   │     │   Intake    │     │  Data Fetch │     │ AI Analysis │
│   (Stripe)  │ ──► │   (Form)    │ ──► │  (Snapshot) │ ──► │  (Claude)   │
└─────────────┘     └─────────────┘     └─────────────┘     └─────────────┘
                                                                   │
                                                                   ▼
┌─────────────┐     ┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│  Delivered  │     │  PDF Gen    │     │   Approve   │     │   DRAFT     │
│   (Email)   │ ◄── │ (React-PDF) │ ◄── │  (Manual)   │ ◄── │  (Review)   │
└─────────────┘     └─────────────┘     └─────────────┘     └─────────────┘
```

### Flow Detail

1. **Payment** → Stripe checkout completes, webhook fires
2. **Intake** → Customer submits DAO name, Snapshot space ID, email
3. **Data Fetch** → QStash triggers Snapshot GraphQL query
4. **AI Analysis** → Claude analyzes governance patterns, generates draft
5. **Draft Review** → Admin dashboard shows draft for Mario's review
6. **Approve** → Mario edits if needed, clicks "Approve & Send"
7. **PDF Gen** → React-PDF generates final report
8. **Deliver** → Resend sends email with PDF attachment + Share & Save offer

---

## 4. Spike Validation (BEFORE Full Build)

### Purpose
Validate that wallet clustering is possible with Snapshot data alone, or if we need Covalent for funding source analysis.

### Target
- **DAO:** ENS (large, active, well-known)
- **Data:** Last 10 proposals + all votes

### Questions to Answer

| Question | Why It Matters |
|----------|----------------|
| What fields does Snapshot return per vote? | Do we get wallet addresses, timestamps, voting power? |
| Can we see delegation relationships? | Critical for "who actually controls votes" |
| Is there funding source data? | Or do we need Covalent/Etherscan? |
| How many unique voters per proposal? | Validates our "300 wallets, 5 entities" premise |
| What does "voting power" actually mean? | Token-weighted? Delegate-weighted? |

### Success Criteria
- [ ] Can identify unique wallet addresses per proposal
- [ ] Can see voting power per wallet
- [ ] Can determine if delegation data is available
- [ ] Understand what additional data sources might be needed

---

## 5. Infrastructure

| Service | Purpose | Status |
|---------|---------|--------|
| **NeonDB** | PostgreSQL database | Ready |
| **Drizzle ORM** | Type-safe database queries | To install |
| **Stripe** | Payment processing | Configured |
| **Resend** | Email delivery (unlimited/day) | Ready |
| **QStash** | Background job processing | Ready |
| **Cloudflare Turnstile** | Bot protection | Ready |
| **Vercel** | Hosting (Pro plan) | Ready |

---

## 6. Database Schema

### Tables

#### `orders`
| Column | Type | Description |
|--------|------|-------------|
| id | uuid | Primary key |
| stripe_session_id | string | Stripe checkout session |
| stripe_payment_intent | string | Payment intent ID |
| status | enum | pending, paid, processing, draft_ready, approved, delivered, failed |
| amount_cents | int | Payment amount (4900 for €49) |
| created_at | timestamp | Order creation time |
| updated_at | timestamp | Last update time |

#### `reports`
| Column | Type | Description |
|--------|------|-------------|
| id | uuid | Primary key |
| order_id | uuid | FK to orders |
| dao_name | string | DAO display name |
| snapshot_space | string | Snapshot space ID (e.g., "ens.eth") |
| chain | string | Blockchain (ethereum, polygon, etc.) |
| customer_email | string | Delivery email |
| customer_name | string | Optional customer name |
| raw_data | jsonb | Raw Snapshot API response |
| ai_analysis | jsonb | Claude's analysis output |
| draft_content | jsonb | Editable draft fields |
| final_pdf_url | string | Stored PDF URL |
| status | enum | fetching, analyzing, draft, approved, generated, sent |
| created_at | timestamp | |
| updated_at | timestamp | |
| delivered_at | timestamp | When email was sent |

#### `share_rewards`
| Column | Type | Description |
|--------|------|-------------|
| id | uuid | Primary key |
| order_id | uuid | FK to orders |
| tweet_url | string | Submitted tweet link |
| twitter_handle | string | Customer's Twitter |
| follower_count | int | At time of submission |
| engagement | int | Likes + retweets |
| sentiment | enum | positive, mixed, critical |
| verified | boolean | Manual verification done |
| reward_amount_cents | int | 2000 for €20 |
| reward_status | enum | pending, approved, paid |
| paid_at | timestamp | When refund processed |
| created_at | timestamp | |

---

## 7. Admin Dashboard Requirements

### Pages Needed

#### 7.1 Orders Queue (`/admin/orders`)
- List of all orders with status badges
- Filter by status (pending, processing, draft_ready, etc.)
- Click to view report details

#### 7.2 Report Review (`/admin/reports/[id]`)
- **Header:** DAO name, customer email, order date
- **Raw Data Panel:** Collapsible view of Snapshot data
- **Draft Preview:** Rendered report sections (editable)
- **Edit Fields:**
  - Executive summary (textarea)
  - Key findings (editable list)
  - Recommendations (editable list)
  - Decentralization score (number input)
- **Actions:**
  - "Save Draft" button
  - "Approve & Generate PDF" button
  - "Approve & Send" button (generates + emails)

#### 7.3 Sent History (`/admin/sent`)
- List of delivered reports
- Customer, DAO, delivery date
- Link to download PDF
- Share & Save status

#### 7.4 Share Rewards (`/admin/rewards`)
- Pending verifications
- Tweet preview
- "Verify & Approve" / "Reject" buttons
- Reward payment status

### Authentication
- Simple: Environment variable with admin password
- Or: Clerk/NextAuth if time permits
- MVP: Password-protected routes

---

## 8. Report Template Skeleton

Based on Product Brief, to be refined after seeing real data.

### Section Structure

| Section | Content | Data Source |
|---------|---------|-------------|
| **Cover Page** | DAO name, date, ChainSights branding | Order data |
| **Executive Summary** | TL;DR of findings, decentralization score (1-10) | AI generated |
| **Participation Analysis** | Unique voters vs. wallet count, confidence level | Snapshot + AI |
| **Power Distribution** | Top entities, concentration %, whale identification | Snapshot + AI |
| **Voting Patterns** | Participation trends, delegate analysis | Snapshot + AI |
| **Recommendations** | What they can do to improve | AI generated |
| **Methodology** | How we clustered, confidence explanation | Static + dynamic |
| **Share & Save** | €20 cashback offer | Static |

### Visual Elements
- Decentralization score gauge (1-10)
- Pie chart: voting power distribution
- Bar chart: participation over time
- Table: top 10 voting entities

---

## 9. API Endpoints

### Existing
- `POST /api/checkout` — Create Stripe session
- `POST /api/intake` — Receive intake form (update to save to DB)

### New Endpoints

| Endpoint | Method | Purpose |
|----------|--------|---------|
| `/api/webhooks/stripe` | POST | Handle Stripe payment events |
| `/api/admin/orders` | GET | List all orders |
| `/api/admin/reports/[id]` | GET | Get report details |
| `/api/admin/reports/[id]` | PATCH | Update draft content |
| `/api/admin/reports/[id]/approve` | POST | Approve and generate PDF |
| `/api/admin/reports/[id]/send` | POST | Send email with PDF |
| `/api/admin/rewards` | GET | List share rewards |
| `/api/admin/rewards/[id]/verify` | POST | Verify and approve reward |
| `/api/jobs/fetch-snapshot` | POST | QStash webhook: fetch data |
| `/api/jobs/analyze` | POST | QStash webhook: run AI analysis |
| `/api/jobs/generate-pdf` | POST | QStash webhook: create PDF |

---

## 10. Environment Variables

```env
# Database
DATABASE_URL=postgresql://...@neon.tech/chainsights

# Stripe
STRIPE_SECRET_KEY=sk_live_...
STRIPE_WEBHOOK_SECRET=whsec_...

# Resend
RESEND_API_KEY=re_...

# QStash
QSTASH_URL=https://qstash.upstash.io
QSTASH_TOKEN=...
QSTASH_CURRENT_SIGNING_KEY=...
QSTASH_NEXT_SIGNING_KEY=...

# Claude AI
ANTHROPIC_API_KEY=sk-ant-...

# Admin
ADMIN_PASSWORD=...

# App
NEXT_PUBLIC_BASE_URL=https://chainsights.one
```

---

## 11. Implementation Order

### Phase 0: Spike (30 min)
1. Fetch Snapshot data for ENS
2. Log to console, analyze structure
3. Determine if clustering is feasible
4. Document findings

### Phase 1: Foundation (3-4 hrs)
1. Set up Drizzle ORM + NeonDB schema
2. Implement Stripe webhook
3. Update intake form → save to database
4. Basic order status tracking

### Phase 2: Data Pipeline (4-5 hrs)
1. Snapshot GraphQL client
2. QStash job for data fetching
3. Store raw data in database
4. Error handling + retries

### Phase 3: AI Analysis (3-4 hrs)
1. Claude API integration
2. Analysis prompt engineering
3. Store analysis results
4. Draft generation

### Phase 4: Admin Dashboard (4-5 hrs)
1. Admin authentication
2. Orders queue page
3. Report review/edit page
4. Approve + send flow

### Phase 5: Delivery (3-4 hrs)
1. React-PDF report generation
2. PDF storage (Vercel Blob or S3)
3. Resend email with attachment
4. Share & Save page in PDF

### Phase 6: Polish (2-3 hrs)
1. Error states + notifications
2. Share rewards tracking
3. Testing full flow
4. Deploy to production

---

## 12. Acceptance Criteria

- [ ] Customer can pay and submit intake form
- [ ] Snapshot data is fetched automatically after payment
- [ ] AI generates draft analysis
- [ ] Admin can review, edit, and approve reports
- [ ] PDF is generated with correct branding
- [ ] Email is delivered with PDF attachment
- [ ] Share & Save offer appears on last page
- [ ] Full flow works end-to-end for one real DAO

---

## 13. Out of Scope (V2)

- On-chain Governor DAO support
- Automated tweet verification
- Tiered share rewards
- Customer self-service portal
- Multiple report types
- Subscription model

---

*Document created: December 15, 2025*
*Status: Ready for spike validation, then implementation*
