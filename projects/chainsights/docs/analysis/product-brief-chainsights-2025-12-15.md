---
stepsCompleted: [1, 2, 3, 4, 5]
inputDocuments:
  - 'docs/analysis/research/market-web3-analytics-europe-research-2025-12-15.md'
  - 'docs/analysis/brainstorming-session-2025-12-15.md'
  - 'docs/_masemIT/ChainSights_blue_hat_and_more.md'
  - 'docs/_masemIT/ChainSights_Panel_Discussion_Responses.md'
  - 'docs/_masemIT/ChainSights_User_Discovery.md'
workflowType: 'product-brief'
lastStep: 5
project_name: 'chainsights'
user_name: 'Mario'
date: '2025-12-15'
party_mode_insights: true
---

# Product Brief: ChainSights

**Date:** 2025-12-15
**Author:** Mario

---

## Executive Summary

**ChainSights** is an identity-first Web3 analytics platform that tells the truth about who's actually participating in blockchain ecosystems.

While existing tools count wallets, ChainSights counts *people* — exposing the gap between reported metrics and reality. Our first product is automated governance reports for DAOs, delivered as one-time purchases (€99-199) to validate demand before building a full platform.

**Core Philosophy:** "Wallets lie. We don't."

**Lead Differentiator:** Transparency is the feature. We don't claim perfect identity resolution — we claim *honest* identity resolution with confidence levels.

**Target Market:** DAOs needing governance truth, with expansion to DApp funnel analytics.

**Bridge Product:** Automated Governance Truth Reports (€99-199, semi-automated with manual quality review)

**Founder Fit:** Built in Austria by a founder who understands European business compliance (insurance industry background), positioned authentically for MiCA-era Web3.

---

## Core Vision

### Problem Statement

Web3 analytics tools count wallets, not people. This creates a fundamental truth problem:

- **DAOs claim decentralization** while 3 people with 50 wallets control governance
- **DApp funnels look broken** when it's actually the same user retrying
- **Growth metrics are inflated 40-60%** because nobody asks: *how many humans is this, really?*

The entire industry is built on a lie of convenience. Wallet counts are easy to measure. Actual participation is hard. So everyone measures wallets and pretends they're measuring people.

### Problem Impact

| Stakeholder | Impact of the Problem |
|-------------|----------------------|
| **DAOs** | False decentralization claims; governance manipulation undetected; regulatory risk (MiCA) |
| **DApp Teams** | Misdiagnosed funnel problems; wasted optimization effort; inflated user metrics misleading investors |
| **NFT Projects** | Whale concentration invisible; market manipulation undetected; community health misjudged |
| **The Industry** | Eroded trust; regulatory scrutiny; unsustainable metrics culture |

### Why Existing Solutions Fall Short

| Solution | What They Do | Why It's Not Enough |
|----------|--------------|---------------------|
| **Dune Analytics** | SQL queries on raw blockchain data | Counts wallets, not people. Requires SQL expertise. |
| **Nansen** | 350M labeled wallets, smart money tracking | Labels wallets, doesn't cluster them into identities. |
| **Flipside** | Free community analytics | No identity resolution. Protocol-dependent model. |
| **Glassnode** | Bitcoin/ETH on-chain metrics | Market focus, not governance/participation focus. |

**The Gap:** No one asks "how many unique humans does this represent?" because the answer is uncomfortable. It makes metrics smaller. It exposes inconvenient truths.

### Proposed Solution

**ChainSights** provides identity-aware analytics that tell the truth about who's actually participating — including the confidence level of that truth.

**Phase 1: Automated Governance Reports (Bridge Product)**
- User submits DAO contract address
- System fetches on-chain governance data (Covalent/The Graph)
- AI analyzes with heuristics + LLM reasoning
- Semi-automated report generation with manual quality review
- Professional PDF delivered within 24 hours

**Report Contents:**
- True participation score (unique voters vs. wallet count)
- Wallet clustering analysis (estimated unique entities)
- Decentralization assessment (power concentration)
- MiCA-readiness indicator
- Confidence levels for all identity claims
- Recommendations for improvement
- Shareable Summary page (for public transparency)

**Phase 2: Platform (If Validated)**
- Dashboard interface for ongoing monitoring
- Real-time alerts on governance anomalies
- API access for integration
- Expanded to DApp funnel analytics

### Key Differentiators

| Differentiator | Why It Matters |
|----------------|----------------|
| **Transparency is the feature** | "~47 unique voters (confidence: medium)" is more valuable than "47 voters" presented as false certainty. We lead with honest uncertainty. |
| **Identity-first philosophy** | Every metric is identity-aware by default. Not a feature — a worldview. |
| **"Wallets lie. We don't."** | Memorable, provocative, true. Category-defining positioning. |
| **Diagnosis, not judgment** | Report emotional arc: Curiosity → Uncomfortable clarity → Normalization → Empowerment → Gratitude. Trusted advisor, not auditor. |
| **Report-first, not dashboard-first** | Lower barrier (€99 one-time vs. subscription). Validates demand before building. Learning > scale. |
| **Built in EU, for EU** | Austrian founder with compliance background. Authentic MiCA positioning, not marketing speak. |

### Why Now

1. **MiCA regulation** is creating compliance demand for verifiable decentralization metrics
2. **Price compression** in analytics market (Nansen cut prices 62-95%) means differentiation is essential
3. **No competitor** offers identity-aware metrics — the gap is open
4. **AI capabilities** make automated analysis feasible for a solo founder

### Founder's Vision

> "I love the idea that a DAO could come to me thinking they're decentralized, and I show them: 'Actually, 3 entities control 61% of your voting power. Here's the evidence.'
>
> That's not just analytics. That's accountability. It's saying what everyone suspects but nobody proves.
>
> The report isn't judgment. It's a diagnosis. And a diagnosis is the first step to getting better."
>
> — Mario, Founder

### Strategic Decisions (Panel-Validated)

| Decision | Rationale |
|----------|-----------|
| **Pricing: €99 (validation) / €199 (premium)** | Low enough to try, high enough to signal quality. "If I can't sell at €99, I won't sell at €49 either. Trust is the objection, not price." |
| **Automation: Semi-automated with manual review** | Learning > scale at this stage. Document every manual fix — patterns become automation roadmap. |
| **Emotional Design: Diagnosis arc** | Empowerment over shame drives action. Goal: "We're sharing this with our community because transparency matters." |

---

## Target Users

### Primary User: Elena — The Governance Lead

**Profile:**
- **Name:** Elena, 32
- **Role:** Governance Lead / Core Contributor
- **Organization:** Mid-sized DeFi protocol DAO
- **DAO Size:** 50-500 active governance participants
- **Treasury:** €50K - €2M

**Her Day:**
- Morning: Checks Discord for governance discussions, replies to proposals
- Midday: Prepares a governance update for the community call
- Afternoon: Reviews voting results from last week's proposal
- Evening: Writes a forum post explaining why a controversial proposal passed with "only" 12% participation

**Her Frustration:**
Elena keeps getting the same question from the community:
> *"Why do the same 5 wallets always decide everything? Is this even decentralized?"*

She *suspects* it's not actually 5 people — it's maybe 2-3 people with multiple wallets. But she can't prove it. Last month, a Twitter thread accused her DAO of being "controlled by insiders." It went semi-viral. She had no data to refute it — or confirm it.

**Her Trigger Moment:**
She's preparing for a community AMA where she knows the "centralization" question will come up. She thinks:
> *"I wish I had an actual report that shows the real participation numbers. Something I could share publicly. Something that either proves we're decentralized — or tells us we need to fix it."*

**Why She's the Ideal Buyer:**
- Elena is both **buyer AND user** — she has discretionary budget authority
- If a DAO requires a governance proposal for €99, that DAO is too decentralized for MVP sales cycle
- €99 is a "rounding error" in a €100K+ treasury — the question isn't "can she afford it" but "does she believe it's valuable"

### Primary Segment: Protocol DAOs (DeFi)

| DAO Type | Fit | Why |
|----------|-----|-----|
| **Protocol DAOs (DeFi)** | ✅ Best | Token voting, whale concentration is *the* issue, MiCA pressure, treasury exists |
| **Investment DAOs** | ✅ Good | Treasury decisions matter, want to know who's really deciding |
| **Service DAOs** | 🟡 Okay | Governance matters less, more about work allocation |
| **Social DAOs** | ❌ Weak | Less formal governance, less treasury, less pain |

**Why Protocol DAOs First:**
- Clearest governance pain (token voting = whale problem)
- Have treasuries (DeFi protocols often sit on millions)
- MiCA specifically targets them (decentralization claims matter for regulation)
- Often accused of centralization on Twitter (reputation risk)

### Sweet Spot: Medium DAOs (50-500 members)

| Size | Problem Intensity | Budget | Decision Speed | Verdict |
|------|-------------------|--------|----------------|---------|
| Tiny (5-20) | Low — they know each other | Low | Fast | ❌ Don't need us |
| Small (20-50) | Emerging — starting to feel it | Some | Fast | 🟡 Maybe |
| **Medium (50-500)** | **High — can't track manually** | **Yes** | **Medium** | ✅ **Sweet spot** |
| Large (1000+) | Very high | Yes | Slow (weeks) | 🟡 Later, not MVP |

### Secondary Users (Emerging)

Watch for these personas in actual sales — don't design for them yet:
- **Investors** doing due diligence before buying governance tokens
- **Contributors** wanting ammunition for a governance proposal
- **Journalists** researching "centralization" stories
- **Community members** reading publicly shared reports

### User Journey

| Stage | Elena's Experience | Success Moment |
|-------|-------------------|----------------|
| **Awareness** | Twitter criticism → Googles "DAO governance analysis" | Finds ChainSights |
| **Consideration** | Sees sample report sections, "Wallets lie" positioning | "This is exactly what I need" |
| **Purchase** | €99 via Stripe (expenses it) or USDC | Frictionless checkout |
| **Delivery** | Report arrives within 24 hours | Professional quality surprises her |
| **Consumption** | Opens with anxiety → Reads with relief or awakening | Feels proud she ordered it |
| **Outcome** | Uses in community call, shares publicly | Earns respect for transparency |
| **Advocacy** | Recommends to 2 other governance leads | Viral loop begins |

**Emotional Design:**
- Report opens with: *"Whatever this report reveals, you took the brave step of seeking truth. That matters."*
- Two success framings:
  - **Vindication:** "Share this publicly and earn trust."
  - **Awakening:** "You know something your competitors don't. Now you can act."
- Either outcome = Elena feels **proud** she ordered it

### Anti-Personas (Who We Don't Serve)

| Anti-Persona | Why Not |
|--------------|---------|
| **Tiny DAOs (5-20)** | They know each other personally. No identity mystery to solve. |
| **Enterprise DAOs (1000+)** | Sales cycle too long, need procurement process. Later market. |
| **Non-governance use cases** | NFT trading, DeFi yield tracking — not MVP focus. |
| **Price shoppers** | If €99 feels expensive, trust is the real objection. Can't solve with discounts. |
| **DAOs requiring proposal for €99** | Too decentralized for MVP sales cycle. Come back when they have discretionary budgets. |

### Market Size (Rough Validation)

- 16,000+ DAOs globally
- ~30% are Protocol DAOs = ~4,800
- ~50% are medium-sized = ~2,400 target DAOs
- 1-2 Elenas per DAO = ~2,400-4,800 potential buyers
- €99-199 per report = **€240K-480K addressable market for reports alone**

This is more than enough to validate the model.

---

## Success Metrics

### User Success Metrics

| Metric | Definition | Target |
|--------|------------|--------|
| **Report Utility** | Buyer references report in community communication | 70%+ |
| **Time to Value** | Report delivered and key insight visible | <24 hours, first page |
| **Problem Resolution** | Report provides clear decentralization assessment with evidence | 90%+ |
| **Shareability** | Buyer shares report publicly with community | 50%+ |

**The "Worth It" Moment:** Elena reads the executive summary and thinks: "Finally, I have *data* instead of opinions. I can actually show this to the community."

### Business Objectives

**3-Month Validation Phase:**
- Prove demand exists with paying strangers (not friends/family)
- Learn what automation works vs. needs manual intervention
- Build case studies for future sales

**Go/No-Go Decision:**

| Signal | Action |
|--------|--------|
| 10 paid reports + 3 referrals | ✅ GO — Build platform |
| 10 paid reports, 0 referrals | 🟡 PIVOT — Investigate value gap |
| <5 paid reports in 3 months | ❌ NO-GO — Wrong positioning or market |

### Key Performance Indicators

| KPI | Target | Measurement |
|-----|--------|-------------|
| **Paid Reports** | 10 in 3 months | Stripe/crypto payment count |
| **Referral Rate** | 30%+ (3+ from referrals) | Track source in intake form |
| **Conversion Rate** | 2-5% | Landing page visits → purchases |
| **Delivery SLA** | 100% <24 hours | Time from payment to delivery |
| **Gross Margin** | >70% | Revenue minus time cost (€50/hr) |

**Not Measured (Intentionally):**
- Social media followers
- Website traffic
- Email signups

*Philosophy: Only measure actions that predict or prove willingness to pay.*

---

## MVP Scope

### Core Features (Bridge Product)

| Feature | Description | User Value |
|---------|-------------|------------|
| **Landing Page** | Value proposition, sample report sections, payment | Discovery + trust |
| **Payment System** | Stripe (fiat) + USDC (crypto) | Frictionless checkout |
| **Intake Form** | DAO contract address + optional context | Clear expectations |
| **Data Pipeline** | Governance data via Covalent/The Graph | Automated collection |
| **AI Analysis Engine** | Heuristics + LLM for identity clustering | Core intelligence |
| **Report Generation** | Professional PDF with findings + recommendations | Deliverable value |
| **Manual QA Review** | Founder review before delivery | Quality assurance |
| **Delivery System** | Email PDF within 24 hours | Fast turnaround |

**MVP User Flow:**
Landing page → Sample sections → Payment (€99) → Intake form → Pipeline + QA → PDF delivered <24h

### Out of Scope for MVP

| Feature | Rationale |
|---------|-----------|
| Dashboard/Platform | Validates demand only after reports sell |
| Real-time Monitoring | Overkill for one-time governance check |
| Multi-chain Support | Ethereum-first reduces complexity |
| API Access | No integration customers yet |
| Subscription Model | One-time validates willingness to pay |
| Automated Delivery | Manual review = learning opportunity |
| User Accounts | Unnecessary friction for €99 purchase |

*Philosophy: If Elena doesn't need it for her first report, it's not MVP.*

### MVP Success Criteria

| Gate | Target | Decision Trigger |
|------|--------|------------------|
| Demand Validation | 10 paid reports | Proceed to platform |
| Viral Validation | 3+ referral purchases | Word-of-mouth works |
| Delivery Feasibility | 100% <24h delivery | Process sustainable |
| Unit Economics | >70% gross margin | Pricing validated |

### Future Vision

**Phase 2 (Post-Validation):** Platform with dashboard, real-time monitoring, multi-DAO comparison

**Phase 3 (Expansion):** DApp funnel analytics, NFT whale analysis, protocol integrations

**Phase 4 (Ecosystem):** "ChainSights Verified" badge, public leaderboard, MiCA compliance support

**Long-term Vision:** The default source of truth for identity-aware on-chain analytics — "Bloomberg Terminal for Web3 participation metrics"

---
