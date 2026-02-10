# Architecture: Governance Attention Map

**Author:** Winston (Architect)
**Date:** 2026-02-10
**Status:** Ready for Review
**Input:** `docs/analysis/product-brief-governance-attention-map-2026-02-10.md`
**Depends on:** `docs/architecture.md` (base architecture)

---

## 1. Context & Constraints

### What We're Building

A topic-level participation analysis feature that classifies Snapshot proposals into governance categories and measures per-category attention patterns. Surfaces as:
- **Free:** Treemap on Matrix Details page (categories 4+ blurred)
- **Paid:** New chapter "Governance Attention Analysis" in Governance Audit PDF (€149)

### Architectural Constraints

| Constraint | Source | Impact |
|-----------|--------|--------|
| Vercel serverless, 300s timeout | Existing infra | Classification must be batched, not real-time |
| QStash for parallel DAO processing | Existing pattern | Classification piggybacks on existing daily job |
| Drizzle ORM, snake_case identifiers | project_context.md rule | Schema must follow conventions |
| PostgreSQL/Neon serverless | Existing DB | Cold start considerations for new queries |
| recharts already in dependency tree | Existing frontend | Use for treemap visualization |
| Claude API for AI tasks | Existing pattern (report analysis) | Use Claude Haiku for classification (cost-efficient) |

### Existing Infrastructure We Leverage

```
Daily GVS Job → QStash → per-DAO calculation
  └── fetchGovernanceData() already fetches proposals (title, body, votes)
  └── Proposals are already in memory during GVS calculation
  └── We add classification as a STEP within the existing flow — zero new infra
```

---

## 2. Architectural Decisions

### AD-1: Classification Pipeline Placement

**Decision:** Classify proposals inside the existing per-DAO QStash job (`gvs-calculate/route.ts`), not as a separate cron job.

**Rationale:**
- Proposals are already fetched and in memory during GVS calculation
- No new cron job, no new QStash queue, no new scheduling complexity
- Only NEW proposals get classified (idempotent — skip if already in `proposal_categories`)
- Failure in classification does NOT block GVS score calculation (fire-and-forget with logging)

**Flow:**
```
gvs-calculate/route.ts (existing per-DAO job)
  │
  ├─ fetchGovernanceData()        ← existing
  ├─ calculateGVS()               ← existing
  ├─ saveGVSSnapshot()            ← existing
  │
  ├─ classifyNewProposals()       ← NEW (after GVS, non-blocking)
  │   ├─ Filter: only proposals NOT in proposal_categories
  │   ├─ Batch classify via Claude Haiku API
  │   └─ Insert into proposal_categories
  │
  └─ refreshCategoryAttention()   ← NEW (after classification)
      └─ Recalculate aggregates for this DAO → category_attention
```

**Timeout Budget:** GVS calculation typically takes 30-90s. Classification of 2-5 new proposals adds ~5-15s. Well within 300s QStash limit.

### AD-2: AI Classification Model

**Decision:** Claude Haiku via Anthropic SDK (already a project dependency).

**Rationale:**
- ~$0.01-0.02 per proposal (title + body ≈ 500-2000 tokens)
- 50 DAOs × ~5 new proposals/week = 250 classifications/week ≈ $2.50-5.00/week
- Anthropic SDK already configured in `src/lib/ai/`
- Haiku is fast enough for batch classification (< 2s per proposal)

**Prompt Structure:**
```
System: You are a governance proposal classifier. Classify the proposal into
exactly ONE category from the taxonomy. Return JSON only.

User: {
  "title": "...",
  "body": "..." (first 2000 chars),
  "space": "..." (DAO name for context)
}

Response: {
  "category": "treasury",
  "confidence": 0.92
}
```

**Taxonomy (v1):** `treasury`, `protocol_upgrades`, `grants`, `operations`, `parameter_changes`, `node_validator`, `partnerships`, `social_meta`

### AD-3: Aggregation Strategy

**Decision:** Recalculate category_attention per-DAO during the daily job (not a separate monthly job).

**Rationale:**
- Aggregation is cheap (SQL GROUP BY on classified proposals)
- Daily refresh means Matrix Details always shows current data
- No need for a separate scheduler, monitoring, or failure handling
- "Monthly" period is just a WHERE clause on proposal dates, not a separate run cadence

**Aggregation Windows:**
- `all_time` — all classified proposals for the DAO
- `last_90d` — rolling 90-day window (more actionable than all-time)

### AD-4: Treemap Visualization Library

**Decision:** `recharts` Treemap component.

**Rationale:**
- Already in `package.json` (used for other charts)
- Supports custom content rendering (needed for blur overlay)
- SSR-compatible with Next.js
- Good enough for 8 categories — no need for d3 complexity

**Blur Implementation:** CSS `filter: blur(8px)` on TreemapCell for categories ranked 4+, with absolute-positioned overlay div containing CTA button.

### AD-5: PDF Integration

**Decision:** New page(s) in `report-template.tsx`, conditional on `tier === 'governance_audit'`.

**Rationale:**
- Follows existing pattern (executive summary, benchmark pages are also tier-conditional)
- @react-pdf/renderer supports the layout we need (treemap as styled divs, not SVG chart)
- No new PDF library needed

**PDF Sections:**
1. Category breakdown table (all categories, participation rate, delegate concentration)
2. Rational Apathy Alerts (red-highlighted categories with < 20% participation)
3. Monthly trend summary (if sufficient data exists)
4. Actionable recommendations per under-attended category (AI-generated)

### AD-6: API Design

**Decision:** Single new API endpoint: `GET /api/matrix/[slug]/attention`

**Rationale:**
- Matrix Details page already fetches per-DAO data via slug
- One endpoint returns both the treemap data and the detail breakdown
- No auth needed (free tier sees all data; blur is client-side)

**Response Shape:**
```typescript
{
  categories: Array<{
    category: string        // 'treasury', 'protocol_upgrades', etc.
    label: string           // 'Treasury', 'Protocol Upgrades', etc.
    proposalCount: number
    uniqueVoters: number
    participationRate: number  // 0-100
    delegateConcentration: number  // 0-100, top 5 delegates' share
    attentionShare: number  // 0-100, % of total DAO votes
  }>
  totalProposals: number
  totalClassified: number
  classificationCoverage: number  // 0-100
  period: 'last_90d' | 'all_time'
  calculatedAt: string  // ISO timestamp
}
```

---

## 3. Database Schema (Drizzle ORM)

### New Tables

```typescript
// src/lib/db/schema.ts — additions

import { pgTable, uuid, varchar, numeric, timestamp, integer, uniqueIndex } from 'drizzle-orm/pg-core'

// Proposal topic classifications (AD-1, AD-2)
export const proposalCategories = pgTable('proposal_categories', {
  id: uuid('id').defaultRandom().primaryKey(),
  daoId: uuid('dao_id').notNull().references(() => daos.id, { onDelete: 'cascade' }),
  proposalId: varchar('proposal_id', { length: 255 }).notNull(),
  category: varchar('category', { length: 100 }).notNull(),
  confidence: numeric('confidence', { precision: 3, scale: 2 }).notNull(),
  classifierVersion: varchar('classifier_version', { length: 20 }).notNull().default('v1'),
  classifiedAt: timestamp('classified_at', { withTimezone: true }).notNull().defaultNow(),
}, (table) => ({
  daoProposalIdx: uniqueIndex('idx_proposal_categories_dao_proposal')
    .on(table.daoId, table.proposalId),
  categoryIdx: index('idx_proposal_categories_category')
    .on(table.daoId, table.category),
}))

// Per-category aggregated metrics (AD-3)
export const categoryAttention = pgTable('category_attention', {
  id: uuid('id').defaultRandom().primaryKey(),
  daoId: uuid('dao_id').notNull().references(() => daos.id, { onDelete: 'cascade' }),
  category: varchar('category', { length: 100 }).notNull(),
  period: varchar('period', { length: 20 }).notNull(),  // 'all_time', 'last_90d'
  proposalCount: integer('proposal_count').notNull(),
  uniqueVoters: integer('unique_voters').notNull(),
  avgParticipationRate: numeric('avg_participation_rate', { precision: 5, scale: 2 }),
  delegateConcentration: numeric('delegate_concentration', { precision: 5, scale: 2 }),
  attentionShare: numeric('attention_share', { precision: 5, scale: 2 }),
  calculatedAt: timestamp('calculated_at', { withTimezone: true }).notNull().defaultNow(),
}, (table) => ({
  daoCategoryPeriodIdx: uniqueIndex('idx_category_attention_dao_cat_period')
    .on(table.daoId, table.category, table.period),
}))
```

### Migration

Generate via existing workflow: `npm run db:generate` → Drizzle Kit produces SQL migration.

---

## 4. New File Structure

```
src/lib/governance/
  └── classifier.ts          ← AI classification service (AD-2)
  └── classifier-prompt.ts   ← System prompt + taxonomy definition
  └── attention.ts           ← Aggregation logic (AD-3)
  └── types.ts               ← CategoryClassification, AttentionData types

src/app/api/matrix/[slug]/attention/
  └── route.ts               ← GET endpoint (AD-6)

src/app/matrix/[slug]/
  └── attention-map.tsx       ← Treemap client component (AD-4)
  └── attention-blur.tsx      ← Blur overlay + CTA component

src/lib/pdf/
  └── report-template.tsx     ← Add attention analysis pages (AD-5)
  └── attention-section.tsx   ← Extracted component for PDF attention chapter

scripts/
  └── backfill-classifications.ts  ← One-time historical backfill
```

### Integration Points (Existing Files Modified)

| File | Change |
|------|--------|
| `src/app/api/jobs/gvs-calculate/route.ts` | Add `classifyNewProposals()` + `refreshCategoryAttention()` after GVS save |
| `src/lib/db/schema.ts` | Add `proposalCategories` + `categoryAttention` tables |
| `src/app/matrix/[slug]/page.tsx` | Add `<AttentionMap>` component below existing charts |
| `src/lib/pdf/report-template.tsx` | Add attention analysis pages for governance_audit tier |
| `src/lib/ai/analysis.ts` | Include attention data in Governance Audit AI prompt context |

---

## 5. Classification Service Design

### `src/lib/governance/classifier.ts`

```typescript
interface ClassificationResult {
  proposalId: string
  category: string
  confidence: number
}

// Classify a batch of proposals for a DAO
async function classifyProposals(
  proposals: SnapshotProposal[],
  daoName: string
): Promise<ClassificationResult[]>

// Check which proposals are already classified (skip those)
async function getUnclassifiedProposals(
  daoId: string,
  proposalIds: string[]
): Promise<string[]>

// Main entry point called from gvs-calculate job
async function classifyNewProposals(
  daoId: string,
  daoName: string,
  proposals: SnapshotProposal[]
): Promise<{ classified: number, skipped: number, errors: number }>
```

**Error Handling:**
- Individual proposal classification failure → log warning, skip, continue batch
- Claude API timeout → retry once with 5s delay, then skip
- Full batch failure → log error, do NOT block GVS job completion
- All errors logged to jobLogs anomalies array

**Cost Control:**
- Proposal body truncated to 2000 chars (sufficient for classification, saves tokens)
- Only closed proposals classified (active/pending proposals may change)
- Batch API calls where possible (multiple proposals per request if < 4000 total tokens)

### Category Taxonomy

```typescript
export const GOVERNANCE_CATEGORIES = {
  treasury: { label: 'Treasury & Funding', color: '#3B82F6' },
  protocol_upgrades: { label: 'Protocol Upgrades', color: '#8B5CF6' },
  grants: { label: 'Grants & Bounties', color: '#10B981' },
  operations: { label: 'Operations', color: '#F59E0B' },
  parameter_changes: { label: 'Parameter Changes', color: '#EF4444' },
  node_validator: { label: 'Node/Validator Ops', color: '#06B6D4' },
  partnerships: { label: 'Partnerships', color: '#EC4899' },
  social_meta: { label: 'Social & Meta-Governance', color: '#6B7280' },
} as const

export type GovernanceCategory = keyof typeof GOVERNANCE_CATEGORIES
```

---

## 6. Aggregation Logic

### `src/lib/governance/attention.ts`

```typescript
// Called after classification in the daily job
async function refreshCategoryAttention(daoId: string): Promise<void>
```

**SQL Logic (via Drizzle):**

```sql
-- Per-category metrics (computed per period)
SELECT
  pc.category,
  COUNT(DISTINCT pc.proposal_id) AS proposal_count,
  COUNT(DISTINCT v.voter) AS unique_voters,
  AVG(p.votes::numeric / NULLIF(total_dao_voters, 0) * 100) AS avg_participation_rate,
  -- delegate concentration: top 5 delegates' vote share per category
  attention_share  -- category votes / total DAO votes * 100
FROM proposal_categories pc
JOIN snapshot proposal data...
WHERE pc.dao_id = $1
  AND (period filter)
GROUP BY pc.category
```

**Implementation Note:** Since we don't store per-proposal voter data in our DB (it's fetched live from Snapshot), the aggregation uses the proposal-level `votes` and `scores_total` fields from `gvsSnapshots` metadata or re-fetches summary stats. For MVP, we compute attention share from proposal counts weighted by voter turnout — not a full per-voter cross-category analysis.

**Simplified MVP Aggregation:**
- `proposalCount` — COUNT of proposals per category
- `uniqueVoters` — SUM of `votes` field per category (approximate, not deduplicated across proposals)
- `avgParticipationRate` — AVG of `(votes / total_dao_voter_pool)` per proposal in category
- `delegateConcentration` — deferred to Phase 2 (requires per-vote data storage)
- `attentionShare` — `category_proposal_count / total_proposals * 100`

This is pragmatic: we get the treemap and rational apathy detection without storing per-vote data. Phase 2 adds per-voter resolution for delegate concentration.

---

## 7. UI Architecture

### Matrix Details Page Integration

```
src/app/matrix/[slug]/page.tsx (Server Component)
  │
  ├── Existing: DAO header, GVS score, component charts
  │
  └── NEW: <AttentionMapSection daoSlug={slug} />
        │
        └── Client Component (src/app/matrix/[slug]/attention-map.tsx)
              │
              ├── useSWR('/api/matrix/{slug}/attention')
              ├── <Treemap> from recharts
              │     └── Custom cell renderer with:
              │           - Categories 1-3: full color, label, percentage
              │           - Categories 4+: blur(8px) filter, greyed overlay
              │
              └── <AttentionBlurOverlay>
                    └── "Unlock Full Analysis" CTA → /check?dao={slug}&tier=governance_audit
```

**Responsive Design (AD-4):**
- Desktop: Treemap (min-width 768px)
- Mobile: Vertical bar chart (sorted by attention share) with blur on items 4+

### Blur UX Details

```css
/* Categories 4+ */
.attention-cell-blurred {
  filter: blur(8px);
  opacity: 0.6;
  pointer-events: none;
}

/* Overlay */
.attention-overlay {
  position: absolute;
  inset: 0;
  display: flex;
  align-items: center;
  justify-content: center;
  background: rgba(15, 44, 85, 0.5);  /* navy/50 */
  backdrop-filter: blur(4px);
}
```

Categories are sorted by `attentionShare` DESC. The top 3 are fully visible. Everything below is blurred with one overlay CTA.

---

## 8. PDF Architecture

### New Pages in Governance Audit Report

**Placement:** After "Peer Comparison" page (current page 7), before "Strategic Recommendations" (current page 8).

**Page: Governance Attention Analysis**

| Section | Content |
|---------|---------|
| Header | "Where Your DAO Pays Attention" |
| Category Table | All categories with proposal count, participation rate, attention share |
| Rational Apathy Alerts | Red-highlighted rows where participation < 20% of DAO average |
| Insight Box | AI-generated 2-3 sentence summary of attention patterns |

**Data Flow:**
```
Report analysis job
  ├── Fetch categoryAttention for DAO (from DB)
  ├── Include in AI prompt context:
  │     "Governance Attention Distribution: {category_table}"
  │     "Generate 2-3 insights about where this DAO focuses governance attention
  │      and where dangerous gaps exist."
  │
  └── report-template.tsx renders attention page with:
        - Styled table (navy background, category colors)
        - Red badge on low-participation categories
        - AI insight box
```

**PDF Layout:** Uses `@react-pdf/renderer` styled divs (no SVG treemap in PDF — treemap is web-only). PDF uses a clean table with color-coded bars.

---

## 9. Backfill Strategy

### One-Time Historical Classification

**Script:** `scripts/backfill-classifications.ts`

```
1. Fetch all DAOs from `daos` table
2. For each DAO:
   a. Fetch last 50 closed proposals from Snapshot API
   b. Filter out already-classified proposals
   c. Batch classify (10 at a time, 1s delay between batches)
   d. Insert into proposal_categories
   e. Refresh category_attention for DAO
3. Rate limit: 50 req/min to Snapshot, ~10 classifications/min to Claude
4. Estimated: 50 DAOs × 50 proposals = 2,500 classifications ≈ $25-50 one-time cost
5. Estimated runtime: ~30-45 minutes
```

**Run via:** `npx tsx scripts/backfill-classifications.ts` (local or CI)

---

## 10. Quality Gate

### Classification Accuracy Validation (AD-2)

Before launch:
1. Manually classify 100 proposals across 5 diverse DAOs
2. Run AI classifier on same 100 proposals
3. Compare: must achieve **≥85% agreement**
4. Review disagreements, refine prompt if needed
5. Document accuracy baseline in `docs/project_context.md`

### Performance Budget

| Operation | Target | Measured Where |
|-----------|--------|---------------|
| Classification (per proposal) | < 3s | Claude API response time |
| Aggregation (per DAO) | < 500ms | DB query time |
| Attention API endpoint | < 200ms | Vercel function duration |
| Treemap render (client) | < 100ms | Browser paint |

---

## 11. Risk Mitigations

| Risk | Mitigation | Owner |
|------|-----------|-------|
| Classification adds too much time to daily job | Non-blocking: failures logged but don't block GVS | Dev |
| Claude API rate limits | Batch proposals, max 10 per DAO per run | Dev |
| Taxonomy too generic for some DAOs | v1 universal taxonomy, DAO-specific overrides in Phase 2 | Product |
| Neon cold start on new queries | Add DB indexes, test query plans before launch | Dev |
| Backfill API costs spike | Budget cap: abort if > $75. Manual review after 500 classifications | Dev |

---

## 12. Phase 2 Hooks (Designed But Not Built)

These are explicitly NOT in MVP but the schema and code structure accommodate them:

| Future Feature | How We Prepare |
|---------------|---------------|
| **Delegate Concentration per Category** | `delegate_concentration` column exists in `category_attention` (nullable, populated in Phase 2) |
| **DAO-Specific Category Overrides** | `classifier_version` column allows v2 prompts with custom taxonomies |
| **Checkbox Voting Indicator** | Requires per-vote storage (new table), not in current schema |
| **Cross-DAO Category Comparison** | `category_attention` table already supports multi-DAO queries |
| **Monthly Trend Tracking** | Add `period_start` date to `category_attention`, query by month |

---

*This architecture document is the technical source of truth for the Governance Attention Map implementation. All implementation decisions should reference this document. See `docs/project_context.md` for pricing and naming rules.*
