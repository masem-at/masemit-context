# ChainSights DAO Governance Index (DGI)
## BMAD Product Requirement — v1.1

**Author:** Claude (Co-Founder, masemIT)  
**Date:** 2026-02-01  
**Status:** Approved — Ready for BMAD Sprint Planning  
**Backlog Reference:** bmm-workflow-status.yaml → `dao-sp-benchmark`  
**Approved by:** Mario Semper (2026-02-01)

| Version | Date | Changes |
|---------|------|---------|
| v0.1 | 2026-02-01 | Initial proposal draft |
| v1.0 | 2026-02-01 | Decisions D1-D5 confirmed, BMAD epics added |
| v1.1 | 2026-02-01 | Team feedback integrated: D6-D9 added, Story 1.4 → Admin UI, pricing corrected (€49/€149), manual curation confirmed, Epic 1/2 separated |

---

## 1. Why a Benchmark Index?

The GVS (Governance Vitality Score) tells you how healthy **one** DAO is. But what users really want to know is: *"Is this good or bad compared to others?"*

Right now on the Matrix Details Page we have individual GVS component charts — but no reference line. No context. It's like showing someone their blood pressure without saying what "normal" is.

**The DAO Governance Index (DGI) solves this** by providing a benchmark that answers:
- "Where does 'Stable' actually start compared to the ecosystem?"
- "Is a DEI of 58 good for a DeFi DAO?"
- "How does governance health trend across the entire DAO ecosystem?"

Think of it as: **GVS = Individual DAO Score. DGI = The Market Average.**

---

## 2. Design Principles

### 2.1 What We Borrow From S&P

| S&P Concept | Our Adaptation |
|-------------|----------------|
| Composite index from constituents | DGI = weighted average of all tracked DAO GVS scores |
| Sector indices (Tech, Finance, etc.) | Category indices (DeFi, Infrastructure, Public Goods, Social) |
| Market-cap weighting | Equal-weighted (confirmed — see D2) |
| Quarterly rebalancing | Monthly rebalancing (governance moves faster than stocks) |
| Inclusion criteria | Minimum data quality thresholds |
| Transparent methodology | Public methodology page, like our existing GVS methodology |

### 2.2 What We Do Differently

S&P tracks stock **prices** — a single dimension. We track governance **health** — a multi-dimensional composite. This means our index is actually closer to an **ESG Rating Benchmark** than a price index. Key differences:

- **No "price" to track** — we benchmark scores (0-100), not market values
- **Slower cadence** — governance changes weekly/monthly, not by the second
- **Smaller universe** — we start with 50 DAOs (see D3)
- **Quality gating** — only DAOs with sufficient data quality (confidence ≥ Medium) enter the index

---

## 3. Index Architecture

### 3.1 The DGI Family

```
DGI (DAO Governance Index)
├── DGI Composite        ← All qualifying DAOs
├── DGI DeFi             ← DeFi DAOs (Uniswap, Aave, Balancer, etc.)
├── DGI Infrastructure   ← Protocol/Infra (ENS, Arbitrum, Optimism, etc.)
├── DGI Public Goods     ← Public Goods (Gitcoin, etc.)
└── DGI Component Indices
    ├── DGI-HPR          ← Ecosystem-wide Human Participation benchmark
    ├── DGI-DEI          ← Ecosystem-wide Delegate Engagement benchmark
    ├── DGI-PDI          ← Ecosystem-wide Power Dynamics benchmark
    └── DGI-GPI          ← Ecosystem-wide Grassroots Participation benchmark
```

### 3.2 Index Calculation

**DGI Composite (Equal-Weighted — recommended for MVP):**

```
DGI = (1/n) × Σ GVS_i    for all qualifying DAOs i=1..n
```

**DGI Component Indices:**

```
DGI-HPR = (1/n) × Σ HPR_i
DGI-DEI = (1/n) × Σ DEI_i
DGI-PDI = (1/n) × Σ PDI_i
DGI-GPI = (1/n) × Σ GPI_i
```

**DGI Category Indices (e.g., DGI DeFi):**

```
DGI_DeFi = (1/n_defi) × Σ GVS_j    for all qualifying DeFi DAOs j
```

### 3.3 Weighting: Equal-Weighted (Confirmed)

| Approach | Status | Rationale |
|----------|--------|-----------|
| **Equal-Weighted** | ✅ **Phase 1 (Now)** | Philosophically consistent, no extra data dependencies, simple, defensible |
| **Treasury-Weighted** | 🔮 Phase 3+ | Requires new data pipeline (Multi-Sig addresses, on-chain balances, token prices). Treasury data NOT available via Snapshot API. |
| **Voter-Weighted** | ❌ Rejected | Circular bias with GVS (high HPR DAOs get more weight AND score higher). Gaming risk. |

Equal-weighted means every qualifying DAO contributes equally to the index. This aligns with ChainSights' core message: governance health matters regardless of treasury size.

---

## 4. Inclusion Criteria

Not every DAO we track should enter the index. Quality standards:

### 4.1 Minimum Requirements

| Criterion | Threshold | Rationale |
|-----------|-----------|-----------|
| **Confidence Level** | ≥ Medium | Low-confidence scores would distort the index |
| **Proposal Count** | ≥ 10 in last 90 days | Need enough governance activity to measure |
| **Active Voters** | ≥ 50 unique addresses | Micro-DAOs with 5 voters skew averages |
| **Data Freshness** | GVS calculated within last 14 days | Stale data = stale index |
| **Not Opted Out** | Must be in public leaderboard | Respect opt-outs |

### 4.2 Curation Process (Manual)

- **Owner:** Mario (sole curator — "das Committee bist du")
- **Tooling:** Admin UI at `/admin` with DGI Universe Management
- **Frequency:** As needed, no fixed schedule
- **Inclusion Criteria serve as guidelines** for Mario's decisions, not as automated rules
- **Announcement:** Changes communicated when relevant (can be content pieces)

### 4.3 Universe Bootstrap (Confirmed: Batch to 50)

**Approach:** Review current tracked DAOs → remove irrelevant → add top Snapshot DAOs → target 50.

**Process:**
1. Export current DAO list from ChainSights DB
2. Evaluate each: Does it meet inclusion criteria? Is it relevant?
3. Remove dead/dormant/irrelevant DAOs
4. Identify top Snapshot spaces by voter count that we don't track yet
5. Add until universe reaches ~50 qualifying DAOs
6. Run batch GVS calculation (existing pipeline, no additional AI costs)
7. Categorize each into DeFi / Infrastructure / Public Goods / Social

**Cost:** Zero additional AI costs — uses existing GVS weekly/daily cron.

---

## 5. Visual Integration on Matrix Details Page

### 5.1 Benchmark Line on Charts

Currently the Matrix Details Page shows 5 charts per DAO. The DGI adds a **reference line** to each:

```
Score
100 ┤
 80 ┤     ████
 67 ┤─────────────────── DGI Composite: 62 ← benchmark line
 60 ┤  ██ ████ ████
 40 ┤  ████████████
 20 ┤  ████████████
  0 ┤──────────────────
     HPR  DEI  PDI  GPI  GVS
     
     ████ = This DAO's scores
     ──── = DGI Benchmark (ecosystem average)
```

### 5.2 Percentile Positioning

On the DAO header/summary:

```
┌──────────────────────────────────────────┐
│  ENS DAO                                 │
│  GVS: 67 (Stable)                        │
│  📊 Top 28% of all tracked DAOs          │
│  📊 Top 15% of Infrastructure DAOs       │
│  ↗️  +3 positions vs. last month          │
└──────────────────────────────────────────┘
```

### 5.3 Tiered Access for Benchmark Data

Following the existing Matrix Details gating logic:

| Access Level | What They See |
|--------------|---------------|
| **Anonymous** | GVS score + DGI Composite line only |
| **Free (email)** | + Category benchmark + percentile |
| **Paid** | + All component benchmarks + historical trend + full breakdown |

This makes the benchmark a **lead-gen tool**: "Want to know how your DAO compares to DeFi average? Sign up free."

---

## 6. DGI as Standalone Product

Beyond the Matrix Details integration, the DGI has standalone value:

### 6.1 DGI Dashboard (Future)

A public page at `/governance-index` showing:
- DGI Composite trend over time (weekly data points)
- Category indices comparison
- "State of DAO Governance" narrative (AI-generated monthly)
- Top movers (biggest GVS changes this month)

### 6.2 DGI Newsletter / Social Content

The DGI generates natural content hooks:
- "DGI dropped 3 points this month — here's why delegate engagement is falling across the ecosystem"
- "DeFi DAOs now govern better than Infrastructure DAOs for the first time"
- "The DGI-HPR hit an all-time low. Are DAOs losing their humans?"

This is exactly the kind of thought-leadership content that builds ChainSights' authority.

### 6.3 DGI in Governance Reports

Every paid report (Deep Dive €49 / Governance Audit €149) now includes:

> *"ENS DAO's GVS of 67 places it in the 72nd percentile of the DAO Governance Index (DGI Composite: 62). Within Infrastructure DAOs specifically (DGI Infrastructure: 59), ENS ranks in the 85th percentile, indicating above-average governance health for its category."*

This adds massive perceived value to reports — context turns a number into a story.

---

## 7. BMAD Implementation Epics

### Epic 1: DGI Foundation (Priority: HIGH)

**Goal:** Calculate and store the DGI index daily.

| Story | Description | Acceptance Criteria | Effort |
|-------|-------------|---------------------|--------|
| **1.1** Universe Setup | Curate initial 50 DAO list with category assignments | 50 DAOs in DB with category field (DeFi/Infra/PG/Social), all meeting inclusion criteria | 2h |
| **1.2** Batch GVS Calculation | Run GVS for all 50 DAOs using existing pipeline | All 50 DAOs have current GVS scores, confidence ≥ Medium | 2h |
| **1.3** DGI Calculation Engine | Daily cron job that computes DGI Composite + Category + Component indices | `dgi_snapshots` table with daily records: composite, 4 category indices, 4 component indices | 4h |
| **1.4** Admin UI: DGI Universe Management | CRUD for DGI universe in /admin dashboard: add/remove DAO, assign category (dropdown), toggle included status | Admin page with DAO table: name, category dropdown (DeFi/Infra/PG/Social), included toggle, last GVS date. All changes manual by Mario. | 4h |
| **1.5** DGI API Endpoint | `GET /api/dgi` returns current index values + history | JSON response with composite, categories, components, trend data | 2h |

**DB Schema:**
```sql
CREATE TABLE dgi_snapshots (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  snapshot_date DATE NOT NULL,
  index_type TEXT NOT NULL,        -- 'composite', 'defi', 'infrastructure', 'public_goods', 'social', 'hpr', 'dei', 'pdi', 'gpi'
  value DECIMAL(5,2) NOT NULL,     -- 0.00 - 100.00
  dao_count INTEGER NOT NULL,      -- number of DAOs in this calculation
  created_at TIMESTAMPTZ DEFAULT NOW(),
  UNIQUE(snapshot_date, index_type)
);

CREATE TABLE dao_index_membership (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  dao_id TEXT NOT NULL,
  category TEXT NOT NULL,           -- 'defi', 'infrastructure', 'public_goods', 'social'
  included_since DATE NOT NULL,
  excluded_at DATE,
  exclusion_reason TEXT,
  grace_period_until DATE,
  created_at TIMESTAMPTZ DEFAULT NOW()
);
```

**Estimated Total: ~14h dev**

---

### Epic 2: Matrix Details Integration (Priority: HIGH)

**Goal:** Show DGI benchmark context on existing Matrix Details Page.

| Story | Description | Acceptance Criteria | Effort |
|-------|-------------|---------------------|--------|
| **2.1** Benchmark Line on Charts | Add DGI reference line to all 5 Recharts components (GVS + 4 components) | Dashed line at ecosystem average on each chart, labeled "DGI: XX" | 3h |
| **2.2** Percentile Calculation | Calculate each DAO's position within the distribution | Percentile shown in DAO header: "Top X% of all DAOs" + "Top X% of [Category] DAOs" | 2h |
| **2.3** Rank Movement | Show position change vs. previous month | "↗️ +3 positions" or "↘️ -2 positions" or "→ unchanged" | 1h |
| **2.4** Tiered Access Gating | Benchmark data follows existing access tiers | Anonymous: GVS + Composite line only. Free: + Category + percentile. Paid: + all components + history | 2h |

**Estimated Total: ~8h dev**

---

### Epic 3: DGI Public Page (Priority: MEDIUM)

**Goal:** Standalone `/governance-index` dashboard for thought leadership and SEO.

| Story | Description | Acceptance Criteria | Effort |
|-------|-------------|---------------------|--------|
| **3.1** DGI Dashboard Page | Public page showing DGI Composite trend, category comparison, top movers | Responsive page at `/governance-index` with ISR caching | 6h |
| **3.2** Chart Granularity Toggle | User can switch between daily/weekly/monthly view | Default: daily. After 3-6mo: weekly toggle. After 12mo: monthly. Session-persistent (account or cookie). | 3h |
| **3.3** State of Governance Narrative | Auto-generated monthly summary text | AI-generated paragraph updated monthly: key trends, top movers, ecosystem insights | 3h |

**Estimated Total: ~12h dev**

---

### Epic 4: Report & Content Integration (Priority: MEDIUM)

**Goal:** DGI context in paid reports and social content pipeline.

| Story | Description | Acceptance Criteria | Effort |
|-------|-------------|---------------------|--------|
| **4.1** Report Template Update | Add DGI benchmark section to governance report template | Every report includes: percentile rank, category comparison, benchmark narrative | 2h |
| **4.2** Social Content Hooks | Monthly DGI summary formatted for LinkedIn/Twitter | Template-based: "DGI Monthly: [trend], [top mover], [insight]" | 1h |

**Estimated Total: ~3h dev**

---

### Sprint Priority Order

| Order | Epic | Reason |
|-------|------|--------|
| 1️⃣ | Epic 1 (Foundation) | No index = nothing to display. Must run ≥2 days before Epic 2. |
| 2️⃣ | Epic 2 (Matrix Integration) | Starts after ≥2 DGI data points exist (need delta for trend). |
| 3️⃣ | Epic 4 (Reports) | Quick win, adds value to paid product |
| 4️⃣ | Epic 3 (Public Page) | SEO/thought leadership — can follow after core is solid |

**Total Estimated Effort: ~37h dev across all epics**

---

## 8. Confirmed Decisions

| # | Decision | Resolution | Rationale |
|---|----------|------------|-----------|
| **D1** | Naming | **DGI (DAO Governance Index)** | Professional, S&P-adjacent positioning. Short, clear, intentional. |
| **D2** | Weighting | **Equal-Weighted** | Philosophically consistent with "Wallets lie. We don't." — governance health shouldn't be dominated by the richest DAOs. Treasury data not available via Snapshot (would require new data pipeline). Voter-weighted has circular bias with GVS. Treasury-weighted can be added as DGI-TW variant in Phase 3. |
| **D3** | Universe Size | **Batch to 50 DAOs** | Review current tracked DAOs, remove irrelevant ones, add top Snapshot DAOs to reach 50. No additional AI costs — uses existing GVS calculation pipeline. |
| **D4** | Category Taxonomy | **Single category per DAO, 4 categories** (DeFi, Infrastructure, Public Goods, Social) | One primary category per DAO, manually assigned. Sufficient for 50 DAOs at launch. New categories announced individually when justified — each is a content piece. |
| **D5** | Update Cadence | **Daily calculation, flexible chart display** | DGI calculated daily (aligns with existing GVS cron). Charts default to daily granularity. After 3-6 months: user can toggle to weekly (session-persistent). After 12 months: monthly toggle added. Persistence via user account (if logged in) or cookie/query-param (anonymous). |
| **D6** | Scope / Sprint Strategy | **Epic 1 first, then Epic 2 separately** | Foundation must produce ≥2 daily DGI values before Matrix integration makes sense (need delta for trend display). Epic 2 starts after DGI has been running for at least 2 days. |
| **D7** | Universe Curation | **Manual only, by Mario** | All DAO additions, removals, and category assignments happen manually via Admin UI. Inclusion criteria serve as guidelines, not automated rules. No automated rebalancing. |
| **D8** | Tiered Access UX | **Lock-icons + prominent percentile** | Dezente Lock-Icons neben Premium-Benchmark-Daten, consistent with existing LockedChartOverlay. Percentile ("Top X% of all DAOs") prominent in DAO header, not hidden in tooltip. |
| **D9** | Documentation | **Methodology page + internal taxonomy doc** | Public DGI methodology page (analog to GVS methodology). Internal taxonomy doc for category assignments. Methodology page can follow after code. |

---

## 9. What This Is NOT

Important to be clear about boundaries:

- **Not a financial index.** We don't track token prices or market caps. No investment advice.
- **Not a rating agency.** We don't "rate" DAOs like Moody's rates bonds. We measure governance health with transparent methodology.
- **Not a ranking.** The DGI is a benchmark average, not a competition. (The leaderboard is the competition part.)
- **Not a substitute for GVS.** GVS measures individual DAOs. DGI provides ecosystem context for GVS.

---

## 10. Summary

| Element | Confirmed |
|---------|-----------|
| **Name** | DAO Governance Index (DGI) |
| **What it is** | Ecosystem benchmark for governance health |
| **Calculation** | Equal-weighted average of all qualifying DAO GVS scores |
| **Universe** | 50 DAOs at launch (batch from top Snapshot spaces) |
| **Categories** | Composite + DeFi + Infrastructure + Public Goods + Social |
| **Component Indices** | DGI-HPR, DGI-DEI, DGI-PDI, DGI-GPI |
| **Inclusion Criteria** | Confidence ≥ Medium, ≥10 proposals, ≥50 voters, fresh data |
| **Rebalancing** | Manual by Mario via Admin UI (no automated schedule) |
| **Update Cadence** | Daily calculation; chart granularity toggle (daily → weekly → monthly) |
| **First Integration** | Benchmark line on Matrix Details Page charts (Epic 2, after ≥2 DGI values exist) |
| **Lead-Gen Angle** | Tiered access to benchmark data (anon → free → paid) |
| **Content Angle** | Monthly "State of DAO Governance" narratives |
| **Total Effort** | ~37h dev across 4 epics |
| **Sprint Priority** | Foundation (solo) → Matrix Integration (after ≥2 DGI data points) → Reports → Public Page |

---

## ⚠️ KNOWN ISSUE: Pricing Inconsistencies Across Codebase

**Status:** Open — requires dedicated sweep  
**Correct Pricing (as of 2026-01-28):**

| Tier | Price |
|------|-------|
| Quick Check | Free (email required) |
| Deep Dive | €49 |
| Governance Audit | €149 |

**Known locations with potentially stale prices (previously €99/€199, now €49/€149):**
- [ ] `gvs-requirement.md` — Section 8.2 references "Free Tier / Pro Tier / Enterprise" API pricing
- [ ] Stripe product/price configuration
- [ ] Report PDF templates (check hardcoded price references)
- [ ] Landing page / pricing page components
- [ ] CTA buttons and checkout flow copy
- [ ] Social media response playbook (dao-rankings-leaderboard.md references "€49")
- [ ] DM templates and outreach scripts
- [ ] README / docs that reference pricing

**Recommendation:** Dedicated 1h sweep — grep codebase for "€99", "€199" and fix all occurrences to €49/€149.

---

*ChainSights — Wallets lie. We don't.*
