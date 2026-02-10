---
type: 'epics-stories'
feature: 'governance-attention-map'
status: 'ready-for-development'
author: 'Winston (Architect) + Party Mode'
date: '2026-02-10'
inputDocuments:
  - 'docs/analysis/product-brief-governance-attention-map-2026-02-10.md'
  - 'docs/architecture-governance-attention-map.md'
  - 'docs/architecture.md'
  - 'docs/project_context.md'
---

# Governance Attention Map — Epics & Stories

## Overview

4 Epics, 11 Stories. Estimated ~26h total.
Implementation order: Epic 1 → Epic 4 (backfill) → Epic 2 → Epic 3.
Epic 4 can run parallel to Epic 2 once Epic 1 is complete.

### Pricing Context (from project_context.md)

- Free Check: €0 | Deep Dive: €49 | Governance Audit: €149
- DAO Matrix: FREE (no paywall)
- Attention Map on Matrix Details: FREE (blurred categories 4+)
- Full Attention Analysis: Governance Audit PDF only (€149)

---

## Epic List

| # | Epic | Stories | Effort | Priority |
|---|------|---------|--------|----------|
| 1 | Classification Pipeline | 4 | ~10h | MUST — foundation for everything |
| 2 | Attention Map UI | 3 | ~7h | MUST — the visible feature |
| 3 | PDF Report Chapter | 2 | ~5h | SHOULD — monetization driver |
| 4 | Backfill & Validation | 2 | ~4h | SHOULD — data quality |

---

## Epic 1: Classification Pipeline

Build the AI-powered proposal classification service and integrate it into the existing daily GVS job.

**User Outcome:** Every Snapshot proposal gets automatically classified into a governance topic category.

**Architecture Refs:** AD-1 (pipeline placement), AD-2 (classification model), AD-3 (aggregation)

---

### Story 1.1: Database Schema Migration

As a **developer**,
I want **`proposal_categories` and `category_attention` tables in the database**,
So that **classified proposals and aggregated attention data can be stored and queried**.

**Acceptance Criteria:**

**Given** the database schema needs new tables
**When** the migration runs
**Then** `proposal_categories` table exists with:
- `id` (UUID, primary key)
- `dao_id` (UUID, FK → daos.id, CASCADE)
- `proposal_id` (VARCHAR 255, not null)
- `category` (VARCHAR 100, not null)
- `confidence` (NUMERIC 3,2, not null)
- `classifier_version` (VARCHAR 20, default 'v1')
- `classified_at` (TIMESTAMP WITH TZ, default NOW())
- Unique index on `(dao_id, proposal_id)`
- Index on `(dao_id, category)`

**And** `category_attention` table exists with:
- `id` (UUID, primary key)
- `dao_id` (UUID, FK → daos.id, CASCADE)
- `category` (VARCHAR 100, not null)
- `period` (VARCHAR 20, not null) — 'all_time' or 'last_90d'
- `proposal_count` (INTEGER, not null)
- `unique_voters` (INTEGER, not null)
- `avg_participation_rate` (NUMERIC 5,2, nullable)
- `delegate_concentration` (NUMERIC 5,2, nullable) — Phase 2, nullable for now
- `attention_share` (NUMERIC 5,2, nullable)
- `calculated_at` (TIMESTAMP WITH TZ, default NOW())
- Unique index on `(dao_id, category, period)`

**Files:** `src/lib/db/schema.ts`, migration via `npm run db:generate`

**Effort:** 1h

---

### Story 1.2: Proposal Classification Service

As a **developer**,
I want **a service that classifies Snapshot proposals into governance categories using AI**,
So that **proposals can be automatically categorized for attention analysis**.

**Acceptance Criteria:**

**Given** a list of Snapshot proposals (title + body + space name)
**When** `classifyProposals()` is called
**Then** each proposal receives a `category` from the taxonomy and a `confidence` score (0.00-1.00)

**And** the taxonomy contains exactly 8 categories:
`treasury`, `protocol_upgrades`, `grants`, `operations`, `parameter_changes`, `node_validator`, `partnerships`, `social_meta`

**And** the classifier uses Claude Haiku via the Anthropic SDK
**And** proposal body is truncated to 2000 characters (cost control)
**And** the response is validated as JSON with `{ category, confidence }`
**And** invalid responses default to `social_meta` with confidence `0.30`
**And** API errors are caught per-proposal (one failure doesn't block the batch)
**And** `classifierVersion` is set to `'v1'`

**Given** `classifyNewProposals(daoId, daoName, proposals)` is called
**When** some proposals are already classified
**Then** only unclassified proposals are sent to the AI (skip existing)
**And** the function returns `{ classified: number, skipped: number, errors: number }`

**Files:**
- `src/lib/governance/classifier.ts` (new)
- `src/lib/governance/classifier-prompt.ts` (new)
- `src/lib/governance/types.ts` (new)

**Effort:** 4h

---

### Story 1.3: Category Aggregation Logic

As a **developer**,
I want **a function that computes per-category attention metrics for a DAO**,
So that **the Attention Map UI and PDF can display aggregated data**.

**Acceptance Criteria:**

**Given** a DAO has classified proposals in `proposal_categories`
**When** `refreshCategoryAttention(daoId)` is called
**Then** for each category, the following metrics are computed:
- `proposalCount` — COUNT of proposals in category
- `uniqueVoters` — SUM of `votes` field from Snapshot proposals (approximate)
- `avgParticipationRate` — average voter turnout per proposal in category
- `attentionShare` — category proposal count / total proposals × 100
- `delegateConcentration` — NULL for MVP (Phase 2)

**And** metrics are computed for both periods: `all_time` and `last_90d`
**And** results are upserted into `category_attention` (ON CONFLICT → UPDATE)
**And** `calculatedAt` is set to NOW()
**And** categories with 0 proposals are NOT stored (no empty rows)

**Files:**
- `src/lib/governance/attention.ts` (new)

**Effort:** 3h

---

### Story 1.4: Daily Job Integration

As a **developer**,
I want **the classification and aggregation to run automatically during the daily GVS job**,
So that **attention data stays fresh without manual intervention**.

**Acceptance Criteria:**

**Given** the daily GVS job runs for a DAO via `gvs-calculate/route.ts`
**When** GVS calculation and snapshot save complete successfully
**Then** `classifyNewProposals()` is called with the fetched proposals
**And** `refreshCategoryAttention()` is called after classification

**And** classification failure does NOT block GVS job completion (non-blocking)
**And** classification errors are logged to `jobLogs.anomalies` array
**And** classification timing is included in the job result metrics

**Given** there are no new proposals since last classification
**When** the job runs
**Then** classification is skipped (0 classified, N skipped)
**And** aggregation still runs (refreshes with current data)

**Files:**
- `src/app/api/jobs/gvs-calculate/route.ts` (modify)

**Effort:** 2h

---

## Epic 2: Attention Map UI

Display the Governance Attention Map on the Matrix Details page with blurred teaser for categories 4+.

**User Outcome:** Visitors see where a DAO's governance attention concentrates — and get a teaser to unlock the full analysis.

**Architecture Refs:** AD-4 (treemap), AD-6 (API)

---

### Story 2.1: Attention API Endpoint

As a **developer**,
I want **a GET endpoint that returns attention data for a DAO**,
So that **the frontend can render the Attention Map**.

**Acceptance Criteria:**

**Given** `GET /api/matrix/[slug]/attention` is called with a valid DAO slug
**When** the DAO has attention data in `category_attention`
**Then** response is `200` with:
```json
{
  "categories": [
    { "category": "treasury", "label": "Treasury & Funding", "color": "#3B82F6",
      "proposalCount": 12, "uniqueVoters": 340, "participationRate": 72.5,
      "attentionShare": 35.2 }
  ],
  "totalProposals": 34,
  "totalClassified": 34,
  "classificationCoverage": 100,
  "period": "last_90d",
  "calculatedAt": "2026-02-10T00:00:00Z"
}
```

**And** categories are sorted by `attentionShare` DESC
**And** the `label` and `color` are resolved from `GOVERNANCE_CATEGORIES` constant

**Given** the DAO has no attention data
**When** the endpoint is called
**Then** response is `200` with `{ categories: [], totalProposals: 0 }`

**Given** the slug doesn't match any DAO
**When** the endpoint is called
**Then** response is `404`

**Files:**
- `src/app/api/matrix/[slug]/attention/route.ts` (new)
- `src/lib/governance/types.ts` (extend with API response type)

**Effort:** 2h

---

### Story 2.2: Treemap Component with Blur Overlay

As a **visitor on the Matrix Details page**,
I want **a visual attention map showing where a DAO focuses governance**,
So that **I can see attention distribution at a glance and get intrigued to unlock the full report**.

**Acceptance Criteria:**

**Given** I view a DAO's Matrix Details page (`/matrix/[slug]`)
**When** attention data is available
**Then** I see an "Governance Attention Map" section below existing charts
**And** the treemap shows all categories sized by `attentionShare`
**And** each cell displays category label + percentage

**Given** categories are sorted by attentionShare
**When** there are 4+ categories
**Then** categories ranked 1-3 are fully visible with brand colors
**And** categories ranked 4+ are blurred (`filter: blur(8px)`, opacity 0.6)
**And** an overlay appears over the blurred area:
- Text: "See where the real gaps are"
- CTA button: "Get Full Report" → `/check?dao={snapshotSpace}&tier=governance_audit`

**Given** there is no attention data for this DAO
**When** I view the page
**Then** the Attention Map section is NOT rendered (graceful absence, no empty state)

**And** the section has a subtle disclaimer: "Based on public Snapshot voting data"
**And** analytics event `attention_map_view` fires on section visible (intersection observer)
**And** analytics event `attention_map_cta_click` fires on CTA click

**Files:**
- `src/app/matrix/[slug]/attention-map.tsx` (new, client component)
- `src/app/matrix/[slug]/attention-blur.tsx` (new, client component)
- `src/app/matrix/[slug]/page.tsx` (modify — add section)

**Effort:** 4h

---

### Story 2.3: Mobile Responsive Layout

As a **mobile visitor**,
I want **the attention data displayed as a vertical bar chart**,
So that **I can understand the data on small screens**.

**Acceptance Criteria:**

**Given** I view the Attention Map on a screen < 768px width
**When** the component renders
**Then** instead of a treemap, I see vertical bars sorted by attentionShare DESC
**And** bars 1-3 are fully visible with category colors
**And** bars 4+ are blurred with the same overlay CTA as desktop
**And** each bar shows category label + percentage

**Files:**
- `src/app/matrix/[slug]/attention-map.tsx` (extend)

**Effort:** 1h

---

## Epic 3: PDF Report Chapter

Add "Governance Attention Analysis" as a new chapter in the Governance Audit (€149) PDF report.

**User Outcome:** Governance Audit customers receive a complete attention breakdown with rational apathy alerts and AI-generated recommendations.

**Architecture Refs:** AD-5 (PDF integration)

---

### Story 3.1: Include Attention Data in Report Analysis

As a **report generator**,
I want **attention data included in the Governance Audit AI analysis prompt**,
So that **the AI can generate insights and recommendations about attention patterns**.

**Acceptance Criteria:**

**Given** a Governance Audit report is being generated
**When** attention data exists for the DAO in `category_attention`
**Then** the AI analysis prompt includes:
- "Governance Attention Distribution" table (all categories with metrics)
- Additional instruction: "Analyze governance attention patterns. Identify rational apathy zones where participation is significantly below the DAO average. Provide 2-3 specific, actionable recommendations to improve attention in under-watched areas."

**And** the AI response includes a new field `attentionAnalysis`:
- `summary` — 2-3 sentence overview of attention patterns
- `apathyZones` — array of `{ category, participationRate, risk }` for under-watched areas
- `recommendations` — array of category-specific improvement suggestions

**Given** no attention data exists for the DAO
**When** the report is generated
**Then** the attention section is omitted from the prompt
**And** `attentionAnalysis` is `null` in the response

**Files:**
- `src/lib/ai/analysis.ts` (modify — extend prompt for governance_audit tier)
- `src/lib/ai/types.ts` (extend ReportDraft with attentionAnalysis)

**Effort:** 2h

---

### Story 3.2: PDF Attention Analysis Pages

As a **Governance Audit customer (€149)**,
I want **a "Governance Attention Analysis" chapter in my PDF report**,
So that **I can see where my DAO focuses governance attention and where gaps exist**.

**Acceptance Criteria:**

**Given** a Governance Audit PDF is being rendered with `attentionAnalysis` data
**When** the PDF is generated
**Then** a new page "Where Your DAO Pays Attention" is inserted after Peer Comparison:
- Category table with columns: Category, Proposals, Participation Rate, Attention Share
- Rows sorted by attentionShare DESC
- Categories with participation < 20% of DAO average marked with red "Low Attention" badge
- AI-generated summary box (2-3 sentences) styled in brand teal background
- Per-category recommendations for flagged apathy zones

**And** the page follows existing PDF styling (navy background, Helvetica, brand colors)
**And** category names use human-readable labels (not snake_case)
**And** the page is wrapped in `{tier === 'governance_audit' && attentionAnalysis && ...}` conditional

**Given** `attentionAnalysis` is null
**When** the PDF is rendered
**Then** the page is not included (no empty page, no placeholder)

**Files:**
- `src/lib/pdf/attention-section.tsx` (new — extracted component)
- `src/lib/pdf/report-template.tsx` (modify — add page)

**Effort:** 3h

---

## Epic 4: Backfill & Validation

Classify historical proposals and validate classification quality before launch.

**User Outcome:** Attention Map launches with historical data for all tracked DAOs, not just data collected from launch day.

---

### Story 4.1: Historical Proposal Backfill Script

As a **developer**,
I want **a script to classify historical proposals for all tracked DAOs**,
So that **the Attention Map has data from day one**.

**Acceptance Criteria:**

**Given** the classification service (Story 1.2) is deployed
**When** `npx tsx scripts/backfill-classifications.ts` is run
**Then** for each DAO in the `daos` table:
- Fetch last 50 closed proposals from Snapshot API
- Skip proposals already in `proposal_categories`
- Classify in batches of 10, with 1s delay between batches
- Insert results into `proposal_categories`
- Call `refreshCategoryAttention()` after all proposals classified

**And** Snapshot API rate limit respected (max 50 req/min)
**And** Claude API rate limit respected (~10 classifications/min)
**And** progress logged: `[DAO_NAME] Classified 42/50, skipped 8 (already classified)`
**And** total cost logged at end: `Total: 2,127 classifications, estimated cost: $21.27`
**And** script aborts if estimated cost exceeds $75 (safety cap)
**And** script is idempotent (safe to re-run)

**Files:**
- `scripts/backfill-classifications.ts` (new)

**Effort:** 2h

---

### Story 4.2: Classification Accuracy Validation

As a **product owner**,
I want **classification accuracy validated against manual labels**,
So that **I can trust the Attention Map data before launch**.

**Acceptance Criteria:**

**Given** the classifier is running
**When** 100 proposals across 5 diverse DAOs are manually classified
**Then** the AI classifier achieves ≥85% agreement with manual labels
**And** disagreements are reviewed and documented
**And** the prompt is refined if accuracy is below threshold

**And** accuracy results are documented in `docs/project_context.md` Feature Flags table:
- Feature: Governance Attention Map
- Status: VALIDATED (or NEEDS_REFINEMENT)
- Accuracy: XX%

**Deliverable:** Spreadsheet or markdown table with 100 rows: proposal_id, manual_label, ai_label, match (y/n)

**Files:**
- `scripts/validate-classifications.ts` (new — optional, can be manual)
- `docs/project_context.md` (update Feature Flags)

**Effort:** 2h

---

## Implementation Order

```
Week 1:
┌─────────────────────────┐
│ Story 1.1: DB Migration  │ ← Start here (unblocks everything)
└────────┬────────────────┘
         │
┌────────▼────────────────┐
│ Story 1.2: Classifier    │ ← Core AI service
└────────┬────────────────┘
         │
┌────────▼────────────────┐     ┌─────────────────────────┐
│ Story 1.3: Aggregation   │     │ Story 4.1: Backfill      │
└────────┬────────────────┘     │ (can start once 1.2 done)│
         │                      └─────────────────────────┘
┌────────▼────────────────┐
│ Story 1.4: Job Integration│
└────────┬────────────────┘
         │
Week 2:  │
┌────────▼────────────────┐     ┌─────────────────────────┐
│ Story 2.1: API Endpoint  │     │ Story 4.2: Validation    │
└────────┬────────────────┘     └─────────────────────────┘
         │
┌────────▼────────────────┐
│ Story 2.2: Treemap + Blur│ ← The visible feature
│ Story 2.3: Mobile        │
└────────┬────────────────┘
         │
Week 3:  │
┌────────▼────────────────┐
│ Story 3.1: AI Prompt Ext │
│ Story 3.2: PDF Pages     │
└─────────────────────────┘
```

**Dependencies:**
- 1.1 → 1.2 → 1.3 → 1.4 (sequential pipeline build)
- 4.1 depends on 1.2 (needs classifier)
- 2.1 depends on 1.3 (needs aggregation data)
- 2.2 depends on 2.1 (needs API)
- 3.1 depends on 1.3 (needs attention data in DB)
- 3.2 depends on 3.1 (needs AI analysis output)
- 4.2 can run anytime after 4.1

---

## Summary

| Epic | Stories | Effort | FRs Covered |
|------|---------|--------|-------------|
| Epic 1: Classification Pipeline | 1.1, 1.2, 1.3, 1.4 | ~10h | Classification, storage, automation |
| Epic 2: Attention Map UI | 2.1, 2.2, 2.3 | ~7h | Free tier visualization, blur teaser, mobile |
| Epic 3: PDF Report Chapter | 3.1, 3.2 | ~5h | Governance Audit analysis, monetization |
| Epic 4: Backfill & Validation | 4.1, 4.2 | ~4h | Historical data, quality gate |
| **Total** | **11 Stories** | **~26h** | |

---

*All stories reference `docs/architecture-governance-attention-map.md` for technical decisions. See `docs/project_context.md` for pricing and naming rules.*
