---
stepsCompleted: [1, 2, 3, 4, 5]
inputDocuments:
  - 'Lido Forum Thread: Jenya_K feedback on Etherscan labels'
  - 'Product Brief: Governance Attention Map (community validation context)'
  - 'Etherscan API Documentation'
  - 'dawsbot/eth-labels GitHub Dataset'
workflowType: 'product-brief'
lastStep: 5
project_name: 'chainsights'
user_name: 'Mario'
date: '2026-02-11'
feature_context: 'Wallet Label Filtering — Data Quality Improvement'
community_validated: true
validation_source: 'Lido DAO Forum (Jenya_K, DAO Operations Workstream)'
---

# Product Brief: Wallet Label Filtering

**Date:** 2026-02-11
**Author:** Mario

---

## Community Validation Context

This feature was suggested by **Jenya_K** (DAO Operations Workstream, Lido DAO) in the Lido Forum thread "Governance Health: Perfect Voter Quality, Room for Participation" (Feb 2026).

When asked about how to identify treasury and CEX wallets for exclusion from governance analysis, Jenya_K responded: **"For 1st check, at least the etherscan labels."**

This is a pragmatic, operator-validated approach to improving data quality across all ChainSights governance metrics.

---

## Executive Summary

**Wallet Label Filtering** is a data quality improvement for ChainSights' GVS scoring pipeline that identifies and appropriately handles non-human governance participants (treasury wallets, CEX wallets, multisig contracts, protocol-owned wallets) using public Etherscan label data. By filtering or flagging these wallets, all governance metrics become more accurate — HPR, DEI, PDI, and GPI all benefit from cleaner input data.

**Core Insight:** A treasury wallet voting with 10M tokens isn't "governance participation" — it's protocol mechanics. Counting it as a human participant inflates HPR, distorts power concentration metrics, and makes governance health scores unreliable.

**Tagline:** "Cleaner data in, better scores out."

**Positioning:** Invisible infrastructure improvement — users don't see the filter, they see better scores. Reinforces ChainSights' "Wallets lie. We don't." positioning by actually following through on identity-first analytics.

---

## Core Vision

### Problem Statement

ChainSights' GVS pipeline currently treats every voting wallet equally. But not all wallets represent human governance participants:

- **Treasury wallets** vote with protocol-owned funds (not individual decisions)
- **CEX wallets** may vote with customer deposits (not representing individual governance choices)
- **Multisig wallets** (e.g., Gnosis Safe) represent committees, not individual participants
- **Protocol contracts** may participate in governance mechanically

These non-human participants distort every governance metric:

| Metric | Distortion |
|--------|-----------|
| **HPR (Human Participation Rate)** | Treasury/CEX wallets counted as "unique humans" — inflates participation |
| **Power Distribution (Gini/Nakamoto)** | Protocol wallets skew concentration metrics — may make centralization look worse (or better) than reality |
| **DEI (Delegate Engagement)** | Multisig committees counted as single delegates — distorts engagement rates |
| **GPI (Grassroots Participation)** | CEX wallets with large holdings counted as "whales" — masks actual holder behavior |

### Problem Impact

| Stakeholder | Impact |
|-------------|--------|
| **DAO Operations Teams** | Governance scores don't reflect reality — can't make informed decisions |
| **ChainSights Credibility** | Sophisticated governance operators (like Jenya_K) immediately spot inaccurate data |
| **Report Customers** | Paying €99-149 for reports with inflated/distorted metrics |
| **DGI Rankings** | DAOs ranked unfairly due to treasury/CEX wallet noise |

### Why Existing Approach Falls Short

Current ChainSights pipeline:
1. Fetches all voter wallets from Snapshot
2. Classifies as WHALE / DELEGATE / TOP VOTER based on token holdings vs. delegated power
3. Calculates GVS components on the full dataset

**What's missing:** Step 1.5 — identify and flag non-human wallets before scoring.

### Proposed Solution

Add a **wallet label enrichment step** to the GVS pipeline that:

1. **Looks up** every voter wallet against a public label dataset
2. **Tags** wallets with their type (Treasury, CEX, Multisig, Protocol, Unknown)
3. **Filters** tagged wallets from human-focused metrics (HPR, GPI)
4. **Separately reports** non-human wallet activity (for transparency)
5. **Stores** labels for reuse (no re-lookup needed for known wallets)

### Key Differentiators

| Differentiator | Why It Matters |
|----------------|----------------|
| **Identity-first analytics, for real** | "Wallets lie. We don't." — now backed by actual identity filtering |
| **Community-validated approach** | Built on Jenya_K's recommendation (DAO Operations Workstream) |
| **Invisible quality improvement** | Users see better scores, not a new feature — trust increases silently |
| **Cross-DAO benefit** | Every scored DAO gets more accurate metrics automatically |

### Why Now

1. **Credibility at stake** — we're about to send reports to Lido governance operators who will spot unfiltered treasury wallets immediately
2. **Free data source available** — dawsbot/eth-labels provides 115K+ labeled addresses at zero cost
3. **Low effort, high impact** — a few hours of work improves every GVS score across the platform
4. **Reinforces positioning** — every other analytics platform (DeepDAO, Dune, Nansen) counts wallets, not humans. This deepens our moat.

---

## Target Users

### Primary: ChainSights Pipeline (Internal)

This is an **internal data quality improvement**. There's no user-facing feature or UI. The beneficiaries are:

- **Every DAO on the DGI** — more accurate scores
- **Every report customer** — cleaner analysis
- **ChainSights credibility** — scores that withstand expert scrutiny

### Secondary: Power Users Who Read Reports

Sophisticated governance operators (like Jenya_K) who review reports and can tell when treasury wallets are polluting the data. They won't see the filter directly, but they'll notice the improved accuracy.

---

## Success Metrics

### Data Quality Metrics

| Metric | Target | Measurement |
|--------|--------|-------------|
| **Wallets labeled** | >80% of top 50 voter wallets per DAO identified | Label coverage report |
| **Non-human wallets found** | At least 1-5 per DAO in top 50 DAOs | Flagged wallet count |
| **Score impact** | Measurable GVS change in 20%+ of DAOs after filtering | Before/after comparison |

### Business Objectives

| Objective | Target | Rationale |
|-----------|--------|-----------|
| **No credibility incidents** | Zero cases where operators spot unfiltered treasury wallets | Trust protection |
| **Report quality** | Reports withstand Lido-level scrutiny | Revenue protection |
| **Methodology strength** | Documented in methodology page | Competitive moat |

---

## Technical Approach

### Data Source: dawsbot/eth-labels (Primary)

**Why not Etherscan API directly?**
The Etherscan Nametag API (`getaddresstag`) is **Pro Plus only** — not available in the Free tier we're using. However, the `dawsbot/eth-labels` project on GitHub provides the same data as a public dataset:

- **115K+ labeled accounts** across EVM chains
- **54K+ labeled tokens**
- **Free public API** available at no cost
- **MCP server** available for Claude Code integration
- **Labels include:** Exchange, Treasury, Bridge, Multisig, Protocol, Fund, etc.
- **Updated regularly** via automated Etherscan scraping

**API Endpoint:** `https://eth-labels.com/api/v1/labels/{address}`

**Fallback:** If eth-labels is down or missing data, maintain a local JSON override file with manually curated labels for known high-value wallets (Lido Treasury, Coinbase, Binance, etc.).

### Alternative/Supplementary Sources

| Source | Coverage | Cost | Use Case |
|--------|----------|------|----------|
| **dawsbot/eth-labels** | 115K+ addresses | Free | Primary lookup |
| **Manual curated list** | Top 50-100 known wallets | Free (maintenance time) | Fallback + high-confidence override |
| **Etherscan Nametag API** | Most comprehensive | Pro Plus ($199/mo) | Future upgrade if needed |
| **Gnosis Safe API** | All Safe multisigs | Free | Identify multisig wallets specifically |

### Label Taxonomy

Wallet labels mapped to governance relevance:

| Label Category | Examples | Governance Treatment |
|---------------|---------|---------------------|
| **TREASURY** | DAO treasury, protocol-owned | **Exclude** from HPR, flag in power distribution |
| **CEX** | Binance, Coinbase, Kraken | **Exclude** from HPR and GPI |
| **MULTISIG** | Gnosis Safe, committee wallets | **Flag** as committee, don't count as individual |
| **BRIDGE** | Cross-chain bridge contracts | **Exclude** from all metrics |
| **PROTOCOL** | DeFi protocol contracts | **Exclude** from all metrics |
| **FUND/VC** | a16z, Paradigm, etc. | **Keep** but flag as institutional (they're real participants) |
| **INDIVIDUAL** | ENS-named personal wallets | **Keep** as normal participant |
| **UNKNOWN** | No label found | **Keep** as normal participant (default) |

**Important:** FUND/VC wallets are **not filtered** — they represent real governance decisions by actual entities. Only mechanical/protocol wallets are excluded.

### Pipeline Integration

```
CURRENT PIPELINE:
1. Fetch voter wallets from Snapshot API
2. Classify (WHALE / DELEGATE / TOP VOTER)
3. Calculate GVS components
4. Generate report

NEW PIPELINE:
1. Fetch voter wallets from Snapshot API
1.5 NEW: Label enrichment
    a. Check local cache for known labels
    b. Look up unknown wallets via eth-labels API
    c. Store results in wallet_labels table
    d. Tag wallets with governance treatment (EXCLUDE / FLAG / KEEP)
2. Classify (WHALE / DELEGATE / TOP VOTER) — excluding tagged wallets
3. Calculate GVS components — with filtered dataset
4. Generate report — include "Filtered Wallets" transparency section
```

### New Database Schema

```sql
-- Wallet label cache
CREATE TABLE wallet_labels (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  address VARCHAR(42) NOT NULL, -- 0x-prefixed Ethereum address
  label_source VARCHAR(50) NOT NULL, -- 'eth-labels', 'manual', 'gnosis-safe-api'
  nametag VARCHAR(255), -- e.g., "Coinbase 10", "Lido Treasury"
  label_category VARCHAR(50) NOT NULL, -- TREASURY, CEX, MULTISIG, BRIDGE, PROTOCOL, FUND, INDIVIDUAL, UNKNOWN
  governance_treatment VARCHAR(20) NOT NULL DEFAULT 'KEEP', -- EXCLUDE, FLAG, KEEP
  labels JSONB, -- raw labels array from source, e.g., ["Exchange", "Coinbase"]
  confidence VARCHAR(20) NOT NULL DEFAULT 'HIGH', -- HIGH (confirmed label), MEDIUM (inferred), LOW (heuristic)
  first_seen_at TIMESTAMP NOT NULL DEFAULT NOW(),
  last_verified_at TIMESTAMP NOT NULL DEFAULT NOW(),
  UNIQUE(address)
);

-- Index for fast lookups during GVS calculation
CREATE INDEX idx_wallet_labels_address ON wallet_labels(address);
CREATE INDEX idx_wallet_labels_category ON wallet_labels(label_category);
```

### Caching Strategy

1. **First GVS run:** Look up all voter wallets → store in `wallet_labels` table
2. **Subsequent runs:** Check cache first → only look up new/unknown wallets
3. **Cache refresh:** Re-verify labels monthly (wallets don't change type often)
4. **Manual overrides:** Admin can add/update labels via direct DB insert (no UI needed for MVP)

### Report Transparency

Add a small section to the PDF report methodology:

> **Wallet Filtering:** This analysis excludes known non-human participants (treasury wallets, exchange wallets, bridge contracts) identified via public Etherscan label data. [X] wallets were excluded from participation metrics. Institutional investors (VCs, funds) are included as legitimate governance participants.

This addresses Stefa_k's concern about transparency — we show what we filter and why.

---

## MVP Scope

### Core Features (Phase 1)

| # | Feature | Description | Priority |
|---|---------|-------------|----------|
| 1 | **eth-labels API integration** | Fetch labels for wallet addresses | MUST |
| 2 | **wallet_labels DB table** | Cache labels locally | MUST |
| 3 | **Label-to-governance mapping** | Map label categories to EXCLUDE/FLAG/KEEP | MUST |
| 4 | **GVS pipeline filter** | Exclude tagged wallets before scoring | MUST |
| 5 | **Manual override file** | JSON file for high-confidence manual labels | MUST |
| 6 | **Report transparency note** | Add filtered wallet count to methodology section | SHOULD |
| 7 | **Before/after score comparison** | Log score changes after filtering for validation | SHOULD |

### Effort Estimates

| Component | Estimated Hours | Complexity |
|-----------|----------------|------------|
| eth-labels API client + error handling | 2h | Low |
| wallet_labels DB schema + migration | 1h | Low |
| Label category mapping logic | 1h | Low |
| GVS pipeline integration (filter step) | 2h | Medium |
| Manual override JSON file (top 50 wallets) | 1h | Low |
| Report methodology text update | 0.5h | Low |
| Before/after score logging | 1h | Low |
| Testing + QA (verify scores improve, no regressions) | 2h | Medium |
| **Total** | **~10-11h** | |

### Out of Scope for MVP

| Feature | Rationale | Target Phase |
|---------|-----------|--------------|
| **Gnosis Safe API integration** | Extra data source, not needed for v1 | Phase 2 |
| **Etherscan Pro API** | $199/mo, not justified yet | Phase 2 (if eth-labels insufficient) |
| **UI for label management** | Manual DB/JSON overrides sufficient for now | Phase 2 |
| **Per-DAO custom exclusion lists** | Universal filtering first | Phase 2 |
| **Filtered wallet detail page** | Transparency note in report sufficient | Phase 2 |

---

## Monetization Strategy

**This feature has no direct monetization** — it's an infrastructure improvement that makes the existing product better.

**Indirect revenue impact:**
- Reports withstand expert scrutiny → fewer refunds, more testimonials
- DGI scores more accurate → higher trust, more social sharing
- Methodology page updated → stronger competitive positioning
- Lido relationship strengthened → Jenya_K's suggestion implemented

---

## Risk Assessment

### Technical Risks

| Risk | Likelihood | Impact | Mitigation |
|------|-----------|--------|------------|
| **eth-labels API downtime** | Low | Low | Local cache + manual override file as fallback |
| **Incomplete label coverage** | Medium | Medium | Manual override for top 50 known wallets; UNKNOWN defaults to KEEP |
| **False positive filtering** | Low | High | Only EXCLUDE confirmed categories (Treasury, CEX, Bridge); FUND/VC stays KEEP |
| **Score changes confuse users** | Medium | Medium | Log before/after; gradual rollout; methodology page explains filtering |

### Product Risks

| Risk | Likelihood | Impact | Mitigation |
|------|-----------|--------|------------|
| **Scores change visibly after filtering** | High (that's the point) | Medium | Communicate as "methodology improvement" not "score change" |
| **DAO disputes exclusion of wallet** | Low | Medium | Governance treatment mapping is transparent; easy to adjust |
| **eth-labels data stale** | Low | Low | Monthly re-verification; manual overrides for critical wallets |

### Mitigation: Score Change Communication

When scores change after enabling wallet filtering:
1. **Don't retroactively change historical snapshots** — mark them as "pre-filter" methodology
2. **Add methodology version** — "GVS v1.1: Wallet label filtering enabled"
3. **Blog post** — "Why Your DAO's Score Might Have Changed" (transparency)

---

## Future Vision

### Phase 2 Enhancements

- **Gnosis Safe API integration:** Automatically detect multisig wallets without relying on Etherscan labels
- **Per-DAO custom exclusion lists:** Allow DAOs to submit their own treasury/operational wallet addresses for exclusion
- **Label confidence scoring:** Weight metrics by label confidence (HIGH = exclude, MEDIUM = partial weight, LOW = full weight)
- **Admin UI for label management:** Web interface for reviewing and updating wallet labels

### Phase 3 Expansion

- **Cross-chain label support:** Extend to Arbitrum, Optimism, Polygon using eth-labels multichain data
- **Automated label discovery:** Heuristics to identify unlabeled treasury/multisig wallets based on behavior patterns
- **Community label contributions:** Allow governance operators to submit labels (like Jenya_K suggested)
- **Etherscan Pro integration:** Upgrade to Etherscan Pro Plus if label volume justifies the cost

### Long-Term Vision

Wallet label filtering is the first step toward ChainSights' core promise: **identity-first analytics**. Today we filter known non-human wallets. Tomorrow we cluster wallets by entity. Eventually we resolve true human governance participation at scale — making "Wallets lie. We don't." not just a tagline but a technical reality.

---
