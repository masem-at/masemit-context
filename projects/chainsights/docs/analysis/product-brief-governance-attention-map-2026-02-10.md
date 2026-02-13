---
stepsCompleted: [1, 2, 3, 4, 5]
inputDocuments:
  - 'Lido Forum Thread: Governance Health - Perfect Voter Quality, Room for Participation'
  - 'docs/analysis/product-brief-chainsights-2025-12-15.md'
  - 'docs/prd.md'
  - 'docs/architecture.md'
workflowType: 'product-brief'
lastStep: 5
project_name: 'chainsights'
user_name: 'Mario'
date: '2026-02-10'
feature_context: 'Governance Attention Map - Topic-Level Participation Analysis'
community_validated: true
validation_source: 'Lido DAO Forum (Jenya_K, mizuki, Stefa_k, BCV)'
---

# Product Brief: Governance Attention Map

**Date:** 2026-02-10
**Author:** Mario

---

## Community Validation Context

This feature was validated through direct engagement with Lido DAO governance participants in the Lido Forum thread "Governance Health: Perfect Voter Quality, Room for Participation" (Feb 2026). The thread reached Top 6 Hot Threads with 11+ replies from active governance contributors.

### Key Community Inputs

| Contributor | Role | Input | Feature Implication |
|-------------|------|-------|---------------------|
| **mizuki** | DAO Operations | "Rational apathy" — contributors triage, not disengage. Proposed "Attention Incentives" to allocate grants/delegate rewards to under-discussed high-risk areas | Topic-level participation tracking, resource allocation signal |
| **Stefa_k** | Node Operator | Key decisions happen in private group chats and closed calls — public record doesn't capture full governance picture | Visibility Gap metric, confidence indicator per topic |
| **BCV** | Community Member | Delegates vote on everything regardless of expertise — "checkbox voting" vs. informed participation | Delegate Topic Specialization Score |
| **Jenya_K** | DAO Operations Workstream | Etherscan labels as starting point for treasury/CEX wallet identification | Data quality improvement (separate but related) |

### Validated Demand Signals

- Multiple governance operators confirmed the need unprompted
- mizuki framed it as "Resource Allocation Signal" — operational, not just analytical
- Stefa_k confirmed the off-chain visibility gap is "a problem for DAOs in general"
- BCV identified delegate quality vs. quantity as unresolved challenge

---

## Executive Summary

The **Governance Attention Map** is a topic-level participation analysis feature for ChainSights that reveals where a DAO's governance attention concentrates — and where dangerous gaps exist. By classifying Snapshot proposals into topic categories and measuring participation patterns per category, it transforms raw voting data into an operational governance steering tool.

**Core Insight:** Overall participation rate is a misleading metric. A DAO with 60% participation might have 90% engagement on treasury votes and 5% on critical infrastructure upgrades. The Attention Map makes this gap visible and actionable.

**Tagline:** "Know where your DAO is paying attention — and where it isn't."

**Positioning:** Moves ChainSights from measuring *how much* governance happens to measuring *where* governance happens. No competitor offers this level of topic-granularity in governance analytics.

---

## Core Vision

### Problem Statement

DAO governance participation is measured as a single aggregate number — but governance isn't monolithic. DAOs make decisions across fundamentally different domains (treasury, protocol upgrades, grants, operations, parameter changes), each requiring different expertise and carrying different risk profiles.

Current analytics (including our own DGI) treat all proposals equally. A 60% participation rate looks healthy — until you realize treasury debates attract 90% of voters while critical infrastructure upgrades pass with 5% participation. This "rational apathy" pattern creates invisible governance risk.

### Problem Impact

| Stakeholder | Impact |
|-------------|--------|
| **DAO Operations Teams** | No visibility into which governance areas are under-resourced. Cannot allocate delegate incentives or grants effectively. |
| **Delegates** | No signal about where their expertise is most needed. Default to voting on everything ("checkbox voting") instead of specializing. |
| **Token Holders** | Critical decisions pass with minimal review. Governance failures appear sudden but were predictable from attention patterns. |
| **DAO Ecosystem** | Information asymmetry between governance areas remains invisible. Resource allocation is based on loudness, not risk. |

### Why Existing Solutions Fall Short

| Solution | What It Measures | What It Misses |
|----------|-----------------|----------------|
| **DeepDAO** | Aggregate participation rates | No topic-level breakdown |
| **Snapshot (native)** | Per-proposal vote counts | No categorization, no cross-proposal analysis |
| **Boardroom** | Delegate activity | No topic specialization metrics |
| **ChainSights DGI (current)** | Overall governance health score | Treats all proposals equally |

**The Gap:** No tool answers the question "Where is our DAO paying attention, and where are we blind?"

### Proposed Solution

A **Governance Attention Map** that:

1. **Classifies** every Snapshot proposal into governance topic categories using AI
2. **Measures** participation patterns per topic (unique voters, delegate concentration, vote diversity)
3. **Visualizes** attention distribution as a heatmap/treemap on the Matrix Details Page
4. **Identifies** "rational apathy zones" — high-risk topics with low participation
5. **Tracks** attention shifts over time (monthly trends)

### Key Differentiators

| Differentiator | Why It Matters |
|----------------|----------------|
| **Topic-level granularity** | First tool to break governance participation down by subject area |
| **"Rational Apathy" detection** | Names and quantifies an invisible governance risk |
| **Operational, not just analytical** | Designed as input for delegate incentives and grant allocation |
| **Community-validated** | Built from real operator feedback, not product assumptions |
| **AI-powered classification** | Scales across DAOs without manual tagging |

### Why Now

1. **Community demand validated** — Lido governance operators confirmed the need in public forum
2. **Data infrastructure exists** — We already fetch Snapshot proposals for DGI; classification is incremental
3. **Competitive moat** — No competitor is building this; first-mover advantage on topic-level analytics
4. **DGI enhancement** — Strengthens our core scoring methodology with a new dimension

---

## Target Users

### Primary User: DAO Operations Lead

**Profile:**
- Responsible for governance health and process optimization
- Allocates delegate incentives and grant budgets
- Reports governance metrics to core team and community
- Needs actionable data, not just dashboards

**Trigger Moment:**
> "We just passed a critical infrastructure vote with 3% participation, and nobody noticed until a community member flagged it on the forum. I need to know where our attention gaps are *before* things break."

**What Success Looks Like:**
> "I can show the core team exactly which governance areas need more delegate attention, and we can allocate grants accordingly."

### Secondary User: Active Delegate

**Profile:**
- Votes on multiple DAOs, limited bandwidth
- Wants to focus expertise where it matters most
- Currently votes on everything (checkbox voting) because there's no signal about where they're most needed

**Trigger Moment:**
> "I'm voting on 15 proposals this week across 3 DAOs. I have no idea which ones actually need my expertise vs. which ones already have enough participation."

### Tertiary User: Governance Researcher

**Profile:**
- Studies DAO governance patterns across protocols
- Publishes research and recommendations
- Needs comparative data across DAOs

**Trigger Moment:**
> "I want to compare how different DAOs allocate governance attention. Do DeFi protocols consistently under-participate on security votes?"

### User Journey

| Stage | Experience | Success Moment |
|-------|-----------|----------------|
| **Discovery** | Sees Attention Map on DAO's Matrix Details page | "I've never seen governance broken down like this" |
| **Understanding** | Reads topic breakdown, identifies attention gaps | "Treasury gets 80% attention, infra upgrades only 8%" |
| **Action** | Uses data to propose delegate incentive reallocation | "We're redirecting grants to under-watched areas" |
| **Monitoring** | Returns monthly to track attention shifts | "Infra participation went from 8% to 25% after incentives" |

---

## Success Metrics

### User Success Metrics

| Metric | Target | Measurement |
|--------|--------|-------------|
| **Comprehension** | Users understand attention distribution within 30 seconds | Heatmap clarity, no explanation needed |
| **Actionability** | At least 1 DAO uses data for delegate/grant allocation within 3 months | Direct feedback or public forum reference |
| **Engagement** | Attention Map is viewed on 40%+ of Matrix Details page visits | Vercel Analytics click tracking |

### Business Objectives

| Objective | Target | Rationale |
|-----------|--------|-----------|
| **DGI differentiation** | Feature no competitor offers | Competitive moat |
| **Report upsell** | 10%+ of Attention Map viewers click paid report CTA | Revenue driver |
| **Community credibility** | Referenced in 2+ DAO governance discussions | Brand positioning |
| **Lido relationship** | Positive feedback from Lido forum contributors on early preview | Strategic partnership potential |

### Key Performance Indicators

| KPI | Target | Timeline |
|-----|--------|----------|
| **DAOs with classified proposals** | 20+ | Month 1 |
| **Classification accuracy** | >85% (spot-checked) | Ongoing |
| **Matrix Details engagement lift** | +20% time-on-page | Month 2 |
| **Paid report conversions from feature** | 5+ attributable | Month 3 |

---

## Technical Approach

### Data Source: Snapshot API

**What we already have:**
- Proposal fetching (title, body, scores, votes, timestamps, author, state)
- Per-DAO proposal history
- Voter participation data

**What we need to add:**
- AI-based topic classification of proposal title + body
- Category storage in database
- Per-category aggregation logic

### Proposal Classification

**Approach:** AI classification (Claude API) of proposal title + body into standardized categories.

**Universal Category Taxonomy (v1):**

| Category | Description | Examples |
|----------|-------------|---------|
| **Treasury** | Fund allocation, budget, spending | "Allocate 500K ARB to incentives program" |
| **Protocol Upgrades** | Technical changes, migrations, implementations | "Upgrade to v3 contracts", "0x02 migration" |
| **Grants** | Funding programs, bounties, ecosystem support | "Season 5 Grants Council budget" |
| **Operations** | Working groups, processes, governance meta | "Restructure operations workstream" |
| **Parameter Changes** | Risk parameters, fees, thresholds | "Adjust liquidation ratio to 150%" |
| **Node/Validator Operations** | Staking, node operators, validator sets | "CSM operator onboarding", "Minipool changes" |
| **Partnerships & Integrations** | Cross-protocol collaborations | "Partner with Protocol X for liquidity" |
| **Social/Meta** | Governance process changes, signaling | "Change voting period to 7 days" |

**Classification Confidence:** Each classification includes a confidence score (High/Medium/Low). Low-confidence classifications are flagged for review.

**DAO-Specific Overrides:** Some DAOs have unique categories (e.g., Lido's CSM). The system supports DAO-specific category extensions on top of the universal taxonomy.

### New Database Schema

```sql
-- Proposal classifications
CREATE TABLE proposal_categories (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  dao_id VARCHAR(255) NOT NULL,
  proposal_id VARCHAR(255) NOT NULL,
  category VARCHAR(100) NOT NULL,
  confidence DECIMAL(3,2) NOT NULL, -- 0.00 to 1.00
  classified_at TIMESTAMP NOT NULL DEFAULT NOW(),
  classifier_version VARCHAR(20) NOT NULL DEFAULT 'v1',
  UNIQUE(dao_id, proposal_id)
);

-- Per-category aggregation (materialized)
CREATE TABLE category_attention (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  dao_id VARCHAR(255) NOT NULL,
  category VARCHAR(100) NOT NULL,
  period VARCHAR(20) NOT NULL, -- 'monthly', 'all-time'
  period_start DATE,
  proposal_count INTEGER NOT NULL,
  unique_voters INTEGER NOT NULL,
  avg_participation_rate DECIMAL(5,2),
  delegate_concentration DECIMAL(5,2), -- top 5 delegates' vote share
  calculated_at TIMESTAMP NOT NULL DEFAULT NOW(),
  UNIQUE(dao_id, category, period, period_start)
);
```

### Metrics per Category

| Metric | Description | Formula |
|--------|-------------|---------|
| **Proposal Count** | Number of proposals in category | COUNT(proposals) |
| **Unique Voters** | Distinct wallets voting on category | COUNT(DISTINCT voter) |
| **Participation Rate** | Category voters vs. total DAO voter pool | unique_voters / total_dao_voters × 100 |
| **Delegate Concentration** | Top 5 delegates' share of category votes | top5_delegate_votes / total_category_votes × 100 |
| **Attention Share** | Category's share of total DAO governance activity | category_votes / total_dao_votes × 100 |
| **Checkbox Voting Indicator** | Delegates who vote identically across all categories vs. selectively | cross_category_voting_pattern analysis |

### UI Concept

**Location:** Matrix Details Page — new section below existing charts.

**Free Tier (Matrix Details Page):**
- Full treemap/heatmap visible showing all categories with attention share percentages
- Categories 4+ are **blurred/greyed out** with a "Get Full Report" overlay
- This "visible but locked" approach triggers "I want to see more" better than simply showing fewer bars (Sally's recommendation)
- Rational Apathy zones visible as red-tinted areas even in blurred state — creates "Oh shit" moment

**Paid Tier (PDF Report — new chapter "Governance Attention Analysis"):**
- Complete Attention Map with all categories unlocked
- Participation rate per category
- Delegate concentration per category
- "Rational Apathy Alert" — highlighted categories where participation is significantly below DAO average (<20% = red badge)
- Monthly trend (attention shift over time)
- Actionable recommendations per under-attended category

**Mobile:** Vertical stacked bars with color-coded participation indicators (sortable list). Treemap doesn't work on small screens — bars with blurred lower entries maintain the teaser effect.

### Classification Pipeline

```
1. Daily GVS job fetches proposals (existing)
2. New proposals → Claude API classification (batch)
3. Classification stored in proposal_categories table
4. Monthly aggregation job → category_attention table
5. Matrix Details Page reads from category_attention
```

**API Cost Estimate:** ~$0.01-0.02 per proposal classification (Claude Haiku). For 50 DAOs × 20 proposals/month = 1,000 classifications = ~$10-20/month.

---

## MVP Scope

### Core Features (Phase 1)

| # | Feature | Description | Priority |
|---|---------|-------------|----------|
| 1 | **Proposal Classification Service** | AI classification of proposals into categories | MUST |
| 2 | **DB Schema + Migration** | proposal_categories and category_attention tables | MUST |
| 3 | **Classification Pipeline** | Batch classification during daily GVS job | MUST |
| 4 | **Category Aggregation Job** | Monthly calculation of per-category metrics | MUST |
| 5 | **Attention Map UI (Free)** | Treemap with blurred categories 4+ on Matrix Details | MUST |
| 6 | **Attention Map PDF Chapter** | Full analysis as new chapter in Governance Audit report | SHOULD |
| 7 | **Rational Apathy Alert** | Flag categories with below-average participation | SHOULD |
| 8 | **Historical Proposals Backfill** | Classify existing proposal history | SHOULD |

### Effort Estimates

| Component | Estimated Hours | Complexity |
|-----------|----------------|------------|
| AI Classification Prompt Engineering + Testing | 4h | Medium |
| DB Schema + Migration | 2h | Low |
| Classification Pipeline Integration | 3h | Medium |
| Category Aggregation Logic | 3h | Medium |
| Attention Map UI — Treemap with blur effect (Free Tier) | 4h | Medium |
| Attention Map PDF Chapter (Paid Report) | 3h | Medium |
| Rational Apathy Alert Logic | 2h | Low |
| Historical Backfill Script | 2h | Low |
| Testing + QA | 3h | Medium |
| **Total** | **~26h** | |

### Out of Scope for MVP

| Feature | Rationale | Target Phase |
|---------|-----------|--------------|
| **Checkbox Voting Indicator** | Complex cross-category analysis, needs more data validation | Phase 2 |
| **Delegate Topic Specialization Score** | Requires delegate identity resolution across categories | Phase 2 |
| **DAO-Specific Category Overrides** | Universal taxonomy first, customization later | Phase 2 |
| **Attention Incentive Recommendations** | Needs validated model before prescribing actions | Phase 3 |
| **Cross-DAO Category Comparison** | Requires standardized taxonomy validation across 20+ DAOs | Phase 3 |

---

## Monetization Strategy

### Tiered Access (Consistent with Matrix Details)

Matrix Details remains FREE — no new paywall. The Attention Map follows the same principle: basic insights are free, deep analysis is in the paid report.

**Current Pricing Structure (Reference):**

| Tier | Price | Key Features |
|------|-------|-------------|
| **Free Check** | Free | GVS score, 4 KPI breakdown, category comparison, instant results |
| **Deep Dive Report** | €49 | AI-powered analysis, specific recommendations, DGI Benchmark, PDF download |
| **Governance Audit** | €149 | 12-week historical trends, peer comparison matrix, "How You Compare" analysis, executive summary, professional PDF |

| Tier | What They See | Conversion Goal |
|------|--------------|-----------------|
| **Matrix Details (FREE)** | Treemap/heatmap showing all categories with attention share (%). Categories 4+ are blurred/greyed out with "Get Full Report" overlay. Teaser creates "I want to see more" trigger. | → Paid report interest |
| **Paid Report (Governance Audit €149)** | New chapter "Governance Attention Analysis": full Attention Map with all categories, delegate concentration per topic, Rational Apathy Alerts, monthly trends, actionable recommendations | Revenue |

**Decision: Add-on, not standalone.** The Attention Map is a new chapter in the existing Governance Audit (€149), not a separate report tier. This keeps the pricing structure simple and adds value to the premium product without fragmenting the offering. The €49 Deep Dive does not include the Attention Map — this creates a clear upgrade incentive from Deep Dive → Governance Audit.

### Lido Early Access (Community Goodwill)

- Within **24 hours of feature launch**, tag mizuki, Jenya_K, Stefa_k, BCV in the Lido Forum thread
- Position as "Built with your input — here's what it looks like for Lido"
- Show the full Matrix Details view (free tier) — no pre-launch preview, ship something real
- Goal: Testimonial or public reference from Lido governance contributor
- Timeline is critical — the forum thread momentum has an expiration date

### Revenue Projection

- Attention Map as new chapter in Governance Audit: increases perceived value of existing €149 report and creates clear upgrade path from Deep Dive (€49)
- Expected impact: +15-20% conversion rate lift on Governance Audit purchases (stronger differentiation vs. Deep Dive)
- Target: 5 additional Governance Audit sales attributable to Attention Map feature in first 3 months = €745 incremental revenue

---

## Risk Assessment

### Technical Risks

| Risk | Likelihood | Impact | Mitigation |
|------|-----------|--------|------------|
| **AI classification inaccuracy** | Medium | High | Confidence scoring, manual spot-checks, low-confidence flagging |
| **Category taxonomy too generic** | Medium | Medium | Start universal, add DAO-specific overrides in Phase 2 |
| **Snapshot API rate limits on backfill** | Low | Low | Batch processing with delays, we already handle this |

### Product Risks

| Risk | Likelihood | Impact | Mitigation |
|------|-----------|--------|------------|
| **Feature not used** | Low (community-validated) | Medium | Start with Lido preview, iterate based on feedback |
| **Off-chain governance gap (Stefa_k's point)** | High | Medium | Add confidence indicator: "Based on public Snapshot data only" |
| **Competitors copy approach** | Low (short-term) | Low | First-mover advantage, build on it quickly |

### Mitigation: The Off-Chain Visibility Problem

Stefa_k correctly identified that public Snapshot data doesn't capture the full governance picture. Our mitigation:

1. **Transparent disclaimer:** "This analysis is based on public Snapshot voting data. Private discussions, closed calls, and off-chain coordination are not captured."
2. **Visibility Gap indicator:** Show what % of governance decisions are made via Snapshot vs. estimated total (where data is available)
3. **Frame as feature:** "The gap between public and private governance is itself a governance health indicator"

---

## Future Vision

### Phase 2 Enhancements

- **Checkbox Voting Indicator:** Identify delegates who vote identically across all categories vs. those who selectively engage
- **Delegate Topic Specialization Score:** Rate each delegate's expertise concentration
- **DAO-Specific Category Overrides:** Allow DAOs to define custom categories

### Phase 3 Expansion

- **Attention Incentive Engine:** Recommend specific grant/reward allocations based on attention gaps
- **Cross-DAO Benchmarking:** Compare attention distribution across similar DAOs ("DeFi DAOs spend 40% of governance on treasury, your DAO spends 80%")
- **Predictive Alerts:** "Infrastructure participation trending down 15% month-over-month — attention needed"
- **Integration with Delegate Platforms:** Feed specialization data to delegation UIs (Karma, Agora)

### Long-Term Vision

The Governance Attention Map evolves ChainSights from a governance *scorer* to a governance *navigator*. Instead of just saying "your governance is healthy/unhealthy," we show *where* to focus improvements — making ChainSights the operational backbone of DAO governance optimization.

> "From passive transparency to active governance steering." — mizuki, Lido DAO Operations

---

## Team Feedback Resolution (2026-02-10)

| # | Open Point | Raised By | Decision |
|---|-----------|-----------|----------|
| 1 | Free/Paid Split vs. Matrix Details Paywall | John | Matrix Details stays FREE. Attention Map basics (treemap with blur) on Matrix Details. Full analysis only in Paid PDF Report (Governance Audit €149). |
| 2 | Standalone report or Add-on? | Mary | **Add-on.** New chapter in existing Governance Audit (€149). No separate tier — keeps pricing simple. Clear upgrade path from Deep Dive (€49). |
| 3 | Lido Early Preview Timeline | John | **Within 24h of launch.** No pre-launch preview. Ship something real, then tag forum contributors. Momentum has expiration date. |
| 4 | Free Tier UX | Sally | Blurred/greyed treemap (not fewer bars). Full map visible but categories 4+ locked. Triggers "I want to see more." |
| 5 | Classification Quality Gate | John | Validate with 50-100 manually classified proposals before launch. Must hit 85% accuracy. |

---
