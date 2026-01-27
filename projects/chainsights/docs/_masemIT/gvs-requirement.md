# ChainSights Governance Vitality Score (GVS)

## Product Requirement Specification

**Version:** 1.0  
**Date:** 2025-12-21  
**Author:** ChainSights Team  
**Status:** Draft

---

## 1. Executive Summary

The Governance Vitality Score (GVS) is a proprietary composite index that measures the health and authenticity of DAO governance. Unlike existing metrics that count wallets and voting power, GVS reveals **actual human participation** – directly leveraging ChainSights' core differentiator: *"Wallets lie. We don't."*

The GVS provides a single, quotable number (0-100) that answers: **"How alive and healthy is this DAO's governance, really?"**

---

## 2. Strategic Context

### 2.1 Market Gap

Current analytics platforms (Dune, Nansen, DeepDAO) provide:
- Wallet counts and voting power distribution
- Proposal pass rates and participation percentages
- Token holder concentration metrics

**What they cannot provide:**
- Distinction between wallets and actual humans
- Delegate engagement quality vs. mere existence
- Power distribution dynamics over time
- Grassroots vs. whale-driven governance

### 2.2 ChainSights Advantage

GVS is only possible because ChainSights performs **identity-aware analytics** – clustering wallets to individuals and distinguishing between:
- **Whale concentration** (capital-based power)
- **Delegate concentration** (trust-based power)

This positions GVS as an **exclusive, defensible metric** that competitors cannot easily replicate.

### 2.3 Business Value

| Value Driver | Description |
|--------------|-------------|
| **Differentiation** | Unique metric unavailable elsewhere |
| **Report Enhancement** | Adds headline-worthy score to governance reports |
| **PR & Virality** | Quotable scores drive social sharing and media pickup |
| **Benchmarking** | Enables DAO-to-DAO comparison, driving engagement |
| **Upsell Path** | Free score lookup → Paid detailed breakdown |

---

## 3. Score Definition

### 3.1 Overall Formula

```
GVS = (HPR × 0.35) + (DEI × 0.25) + (PDI × 0.20) + (GPI × 0.20)
```

**Output:** Integer score from 0-100

### 3.2 Score Interpretation

| Range | Label | Interpretation |
|-------|-------|----------------|
| 80-100 | **Vital** | Healthy, decentralized governance with broad human participation |
| 60-79 | **Stable** | Functional governance with some concentration risks |
| 40-59 | **Concerning** | Significant oligarchic tendencies or participation apathy |
| 0-39 | **Critical** | Governance captured by few actors or largely inactive |

### 3.3 Visual Representation

```
[Critical]----[Concerning]----[Stable]----[Vital]
    0              40            60          80    100
    🔴              🟠            🟡          🟢
```

---

## 4. Component Specifications

### 4.1 Human Participation Rate (HPR)

**Weight:** 35%  
**Purpose:** Measures actual human participation vs. raw wallet counts

#### Definition

```
HPR = (Unique Human Participants / Active Voting Wallets) × 100
```

#### Calculation Details

| Element | Definition |
|---------|------------|
| **Unique Human Participants** | Distinct individuals identified through wallet clustering, delegate registration, ENS profiles, and behavioral analysis |
| **Active Voting Wallets** | Wallets that voted on at least one proposal in the measurement period |
| **Measurement Period** | Rolling 90 days |

#### Normalization

| Raw HPR | Normalized Score (0-100) |
|---------|--------------------------|
| ≥90% | 100 |
| 70-89% | 80-99 (linear) |
| 50-69% | 50-79 (linear) |
| 30-49% | 25-49 (linear) |
| <30% | 0-24 (linear) |

#### Data Sources

- Snapshot voting records
- On-chain governance transactions
- Delegate registry data (e.g., ENS delegates, Agora, Tally)
- ChainSights wallet clustering engine

#### Edge Cases

| Scenario | Handling |
|----------|----------|
| New DAO (<30 days) | Insufficient data flag; exclude from score |
| No delegate system | Use direct wallet→human clustering only |
| Anonymous-heavy DAO | Lower confidence indicator; score still calculated |

---

### 4.2 Delegate Engagement Index (DEI)

**Weight:** 25%  
**Purpose:** Measures how actively delegates exercise their entrusted power

#### Definition

```
DEI = Weighted Average of (Delegate Votes Cast / Eligible Proposals) × Activity Recency Factor
```

#### Calculation Details

| Element | Definition |
|---------|------------|
| **Delegate** | Address holding delegated voting power from ≥1 other address |
| **Eligible Proposals** | Proposals occurring while delegate held power |
| **Activity Recency Factor** | Multiplier favoring recent activity (decay over 90 days) |
| **Weighting** | By delegated voting power (larger delegates weighted more) |

#### Recency Decay Function

```
Recency Factor = e^(-days_since_last_vote / 30)
```

- Vote yesterday: Factor ≈ 0.97
- Vote 30 days ago: Factor ≈ 0.37
- Vote 90 days ago: Factor ≈ 0.05

#### Normalization

| Weighted Participation | Normalized Score |
|------------------------|------------------|
| ≥95% with high recency | 100 |
| 80-94% | 75-99 |
| 60-79% | 50-74 |
| 40-59% | 25-49 |
| <40% | 0-24 |

#### Bonus Modifiers

| Behavior | Modifier |
|----------|----------|
| Delegate provides on-chain/off-chain vote rationale | +5 points (capped) |
| Delegate votes against majority (independent thinking) | +2 points per instance (capped at +10) |

#### Edge Cases

| Scenario | Handling |
|----------|----------|
| DAO without delegation | DEI = 50 (neutral); rely on other components |
| Single dominant delegate | Flag as concentration risk in report |
| Delegate with <1% power | Exclude from weighted calculation |

---

### 4.3 Power Dynamics Index (PDI)

**Weight:** 20%  
**Purpose:** Measures healthy turnover and fluidity in governance power distribution

#### Definition

```
PDI = (Leadership Turnover Score + Gini Trend Score + New Entrant Score) / 3
```

#### Sub-Components

##### 4.3.1 Leadership Turnover Score

Measures changes in top delegate positions over time.

```
Turnover Rate = (Position Changes in Top 20 Delegates) / 20 × 100
```

| Measurement | Period |
|-------------|--------|
| Comparison intervals | Current vs. 90 days ago |
| Position change | Any movement in/out of Top 20, or rank change ≥5 positions |

| Turnover Rate | Score |
|---------------|-------|
| 30-50% | 100 (healthy churn) |
| 20-29% or 51-60% | 75 |
| 10-19% or 61-70% | 50 |
| <10% (frozen) or >70% (chaotic) | 25 |

##### 4.3.2 Gini Trend Score

Measures whether power concentration is improving or worsening.

```
Gini Trend = Gini Coefficient (90 days ago) - Gini Coefficient (current)
```

| Trend | Interpretation | Score |
|-------|----------------|-------|
| Positive (>0.05) | Decentralizing | 100 |
| Neutral (-0.05 to +0.05) | Stable | 60 |
| Negative (<-0.05) | Centralizing | 20 |

##### 4.3.3 New Entrant Score

Measures fresh participation entering governance.

```
New Entrant Rate = (First-time voters in period / Total voters) × 100
```

| Rate | Score |
|------|-------|
| ≥15% | 100 |
| 10-14% | 75 |
| 5-9% | 50 |
| <5% | 25 |

---

### 4.4 Grassroots Participation Index (GPI)

**Weight:** 20%  
**Purpose:** Measures community-driven vs. whale-dominated governance

#### Definition

```
GPI = (Small Holder Participation Rate × 0.5) + (Delegation Breadth × 0.3) + (Vote Diversity × 0.2)
```

#### Sub-Components

##### 4.4.1 Small Holder Participation Rate

```
SHPR = (Participating Small Holders / Total Small Holders) × 100
```

| Definition | Threshold |
|------------|-----------|
| Small Holder | Wallet holding <0.1% of total voting power |
| Participating | Voted or delegated in measurement period |

##### 4.4.2 Delegation Breadth

```
Breadth = Unique Delegators / Total Token Holders with Voting Rights
```

Measures how many token holders actively choose a delegate.

##### 4.4.3 Vote Diversity

```
Diversity = 1 - (Top 10 Voters Power Share / 100)
```

If Top 10 control 80% of votes → Diversity = 0.20 → Score contribution = 20

#### Normalization

Each sub-component normalized to 0-100, then combined per weights.

---

## 5. Data Requirements

### 5.1 Required Data Points

| Data Category | Source | Update Frequency |
|---------------|--------|------------------|
| Proposal metadata | Snapshot API, on-chain events | Real-time |
| Vote records | Snapshot API, governance contracts | Real-time |
| Delegation events | On-chain logs, delegate registries | Real-time |
| Wallet clustering | ChainSights identity engine | Daily |
| Token holdings | Blockchain state, Covalent/The Graph | Daily |
| Delegate profiles | ENS, Agora, Tally, DAO-specific registries | Weekly |

### 5.2 Supported Governance Platforms

**Phase 1 (MVP):**
- Snapshot (off-chain voting)

**Phase 2:**
- Tally (on-chain governance)
- Agora
- Governor Bravo/Alpha contracts

### 5.3 Supported Chains

**Phase 1:** Ethereum Mainnet  
**Phase 2:** Polygon, Arbitrum, Optimism, Base

---

## 6. Output Specification

### 6.1 Score Object

```json
{
  "dao_id": "ens.eth",
  "dao_name": "ENS DAO",
  "snapshot_space": "ens.eth",
  "score_date": "2025-12-21",
  "measurement_period": {
    "start": "2025-09-23",
    "end": "2025-12-21"
  },
  "gvs": {
    "total": 67,
    "label": "Stable",
    "components": {
      "hpr": {
        "score": 72,
        "weight": 0.35,
        "weighted_contribution": 25.2,
        "raw_value": 0.68,
        "description": "68% of voting wallets map to unique humans"
      },
      "dei": {
        "score": 58,
        "weight": 0.25,
        "weighted_contribution": 14.5,
        "raw_value": 0.61,
        "description": "Top delegates voted on 61% of proposals (30-day recency weighted)"
      },
      "pdi": {
        "score": 65,
        "weight": 0.20,
        "weighted_contribution": 13.0,
        "sub_scores": {
          "leadership_turnover": 55,
          "gini_trend": 70,
          "new_entrants": 70
        },
        "description": "Moderate power fluidity; slight decentralization trend"
      },
      "gpi": {
        "score": 71,
        "weight": 0.20,
        "weighted_contribution": 14.2,
        "sub_scores": {
          "small_holder_participation": 65,
          "delegation_breadth": 78,
          "vote_diversity": 72
        },
        "description": "Good grassroots engagement; delegation widely used"
      }
    }
  },
  "confidence": {
    "level": "high",
    "factors": {
      "data_completeness": 0.95,
      "identity_resolution_rate": 0.72,
      "proposal_count": 23
    }
  },
  "flags": [],
  "generated_at": "2025-12-21T14:30:00Z",
  "version": "1.0"
}
```

### 6.2 Comparison Object (for benchmarking)

```json
{
  "dao_id": "ens.eth",
  "gvs": 67,
  "percentile_rank": 72,
  "category_rank": 3,
  "category": "Protocol DAOs",
  "comparison_set_size": 45,
  "comparison_date": "2025-12-21"
}
```

---

## 7. Report Integration

### 7.1 Score Display in Governance Reports

The GVS appears as a **headline metric** at the top of every ChainSights Governance Report:

```
┌─────────────────────────────────────────────────────────┐
│  GOVERNANCE VITALITY SCORE                              │
│                                                         │
│     ██████████████████████░░░░░░░░  67 / 100           │
│                                                         │
│     Label: STABLE                                       │
│     Percentile: Top 28% of Protocol DAOs                │
│                                                         │
│  Components:                                            │
│  ├─ Human Participation:  72  ████████████████████░░░  │
│  ├─ Delegate Engagement:  58  ███████████████░░░░░░░░  │
│  ├─ Power Dynamics:       65  █████████████████░░░░░░  │
│  └─ Grassroots Index:     71  ██████████████████░░░░░  │
└─────────────────────────────────────────────────────────┘
```

### 7.2 Narrative Integration

Each report includes an auto-generated narrative paragraph:

> *"ENS DAO achieves a Governance Vitality Score of 67 (Stable), placing it in the top 28% of Protocol DAOs. While human participation is healthy at 72, delegate engagement shows room for improvement – top delegates voted on only 61% of eligible proposals in the past 90 days. Power dynamics are moderately fluid, with a slight trend toward decentralization. Grassroots participation is strong, indicating broad community involvement beyond whale-dominated voting."*

---

## 8. API Specification

### 8.1 Endpoints

#### Get Current GVS

```
GET /api/v1/gvs/{dao_id}
```

**Response:** Score Object (see 6.1)

#### Get Historical GVS

```
GET /api/v1/gvs/{dao_id}/history?period=90d
```

**Response:** Array of Score Objects with timestamps

#### Get GVS Comparison

```
GET /api/v1/gvs/{dao_id}/compare?category=protocol
```

**Response:** Comparison Object (see 6.2)

#### Get GVS Leaderboard

```
GET /api/v1/gvs/leaderboard?category=all&limit=50
```

**Response:** Ranked list of DAOs by GVS

### 8.2 Authentication

All GVS endpoints require API key authentication.

**Free Tier:** 100 requests/month, current score only  
**Pro Tier:** 10,000 requests/month, full history and comparison  
**Enterprise:** Unlimited, webhook notifications on score changes

---

## 9. Validation & Quality Assurance

### 9.1 Score Sanity Checks

| Check | Threshold | Action |
|-------|-----------|--------|
| HPR > 100% | Impossible | Error; investigate clustering |
| GVS components sum ≠ total | Math error | Recalculate |
| Sudden GVS drop >30 points | Anomaly | Flag for manual review |
| Zero proposals in period | Insufficient data | Exclude from scoring |

### 9.2 Confidence Indicators

Each score includes a confidence level:

| Level | Criteria |
|-------|----------|
| **High** | ≥20 proposals, ≥70% identity resolution, ≥95% data completeness |
| **Medium** | 10-19 proposals, 50-69% identity resolution, 80-94% data |
| **Low** | <10 proposals, <50% identity resolution, or <80% data |

### 9.3 Manual Calibration

For initial launch, manually verify GVS scores against:
- Known healthy DAOs (expected: 70+)
- Known problematic DAOs (expected: <50)
- Edge cases (single-delegate DAOs, dormant DAOs)

---

## 10. Privacy & Compliance

### 10.1 Data Handling

| Principle | Implementation |
|-----------|----------------|
| **Pseudonymity preserved** | Scores aggregate to DAO level; individual wallet scores not exposed |
| **Public data only** | All inputs derived from public blockchain and Snapshot data |
| **GDPR compliance** | No personal data collected; wallet addresses are pseudonymous identifiers |
| **Right to explanation** | Component breakdown provides score transparency |

### 10.2 Transparency

- Methodology documentation publicly available
- Component weights disclosed
- Raw data sources cited in reports

---

## 11. Roadmap

### 11.1 Phase 1: MVP (4 weeks)

- [ ] Implement HPR calculation with basic wallet clustering
- [ ] Implement DEI calculation for Snapshot-based DAOs
- [ ] Implement PDI with leadership turnover and Gini trend
- [ ] Implement GPI with small holder and delegation metrics
- [ ] Create score aggregation and normalization engine
- [ ] Build API endpoints (current score only)
- [ ] Integrate into Governance Report template

### 11.2 Phase 2: Enhancement (8 weeks)

- [ ] Add historical score tracking and trend visualization
- [ ] Implement comparison and benchmarking features
- [ ] Expand to on-chain governance (Tally, Governor contracts)
- [ ] Add confidence scoring and data quality indicators
- [ ] Create public leaderboard page

### 11.3 Phase 3: Advanced (12+ weeks)

- [ ] Multi-chain support (L2s)
- [ ] Predictive GVS (score trajectory modeling)
- [ ] Custom weight configuration for enterprise clients
- [ ] Webhook alerts on significant score changes
- [ ] GVS certification badge for DAOs

---

## 12. Success Metrics

| Metric | Target | Measurement |
|--------|--------|-------------|
| Score accuracy | >90% alignment with expert assessment | Manual validation of 20 DAOs |
| Report engagement | GVS section most-viewed | Analytics on report interactions |
| Social mentions | GVS cited in 10+ DAO discussions/month | Social monitoring |
| API adoption | 50+ API calls to GVS endpoints/month | Usage analytics |
| Upsell conversion | 5% of free score lookups → paid report | Funnel tracking |

---

## 13. Open Questions

| Question | Owner | Due Date |
|----------|-------|----------|
| Should component weights be adjustable per DAO category? | Product | TBD |
| How to handle DAOs with hybrid on-chain/off-chain voting? | Engineering | TBD |
| Minimum proposal count for valid score? | Product | TBD |
| Public vs. gated access to leaderboard? | Business | TBD |

---

## 14. Appendix

### A. Glossary

| Term | Definition |
|------|------------|
| **Delegate** | Address that holds voting power delegated by other token holders |
| **Gini Coefficient** | Measure of inequality (0 = perfect equality, 1 = perfect inequality) |
| **Snapshot** | Off-chain voting platform used by most DAOs |
| **Wallet Clustering** | Process of grouping multiple wallets controlled by the same entity |
| **Voting Power** | Token-weighted influence in governance decisions |

### B. Reference DAOs for Calibration

| DAO | Expected GVS Range | Rationale |
|-----|-------------------|-----------|
| ENS | 60-75 | Large, active, but delegate-concentrated |
| Uniswap | 50-65 | Low participation, whale-heavy |
| Gitcoin | 70-85 | Strong community, diverse participation |
| Optimism | 65-80 | Active delegates, growing ecosystem |
| Lido | 40-55 | Known concentration concerns |

### C. Version History

| Version | Date | Changes |
|---------|------|---------|
| 1.0 | 2025-12-21 | Initial specification |

---

*ChainSights – Wallets lie. We don't.*
