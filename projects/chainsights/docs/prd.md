---
stepsCompleted: [1, 2, 3, 4, 6, 7, 8, 9, 10, 11]
inputDocuments:
  - 'docs/analysis/product-brief-chainsights-2025-12-15.md'
  - 'docs/analysis/research/market-web3-analytics-europe-research-2025-12-15.md'
  - 'docs/analysis/brainstorming-session-2025-12-15.md'
  - 'docs/design/dao-rankings-leaderboard.md'
documentCounts:
  briefs: 1
  research: 1
  brainstorming: 1
  projectDocs: 0
workflowType: 'prd'
lastStep: 11
workflowStatus: 'complete'
completedDate: '2025-12-21'
---

# Product Requirements Document - chainsights

**Author:** Mario
**Date:** 2025-12-21

## Executive Summary

ChainSights is building a **DAO Rankings Leaderboard** - a public governance health scoreboard that makes DAO governance visible, competitive, and actionable. Think **CoinMarketCap for DAO Governance**: a live leaderboard displaying governance scores (x/10) with stock-market-style change indicators (+0.3, -0.5, unchanged) and strategic call-to-action buttons.

**The Core Problem:**
DAO governance health is invisible and fragmented. Communities lack benchmarks to assess their governance quality, and there's no social proof mechanism driving governance improvement across the ecosystem. Static governance reports generate minimal engagement and don't create urgency for change.

**Our Solution:**
A public leaderboard that transforms governance assessment into an engaging, viral experience:
- **Weekly rankings** showing how DAOs stack up across 5 weighted criteria (Participation 30%, Decentralization 25%, Proposal Quality 20%, Transparency 15%, Execution 10%)
- **Change indicators** that tell stories and create recurring engagement ("Arbitrum jumped 3 spots after implementing quadratic voting!")
- **Category breakdowns** providing actionable improvement paths, not just scores
- **"Claim Your Spot" CTAs** converting visibility into leads and revenue

**Target Users:**
- **Primary:** DAO communities and governance participants wanting to assess and improve governance health
- **Secondary:** DAO core teams tracking performance trends over time
- **Tertiary:** Web3 ecosystem stakeholders researching governance patterns

### What Makes This Special

**1. Movement Creates Virality**
Static scores are boring. Change indicators (+/-) generate narrative, FOMO, and social sharing. DAOs share improvements, communities tag underperformers, and rank changes become Twitter threads.

**2. Controversy as Growth Engine**
Lower-ranked DAOs will dispute rankings - that's a feature, not a bug. Criticism converts to sales ("Get the full story with our €49 detailed report") and generates organic reach. Our 5-layer defense strategy (transparency, nuance, empathy, legal protection, response playbook) ensures constructive outcomes.

**3. Hybrid Monetization Model**
Free if you opt-in to public listing, paid for private detailed reports. This maximizes both lead generation AND revenue while turning potential friction into opportunity.

**4. Social Proof Mechanics at Scale**
FOMO intensifies with every rank change. High-ranking DAOs share organically. Community members drive accountability by tagging their DAOs. The "CoinMarketCap positioning" makes the concept instantly understandable.

**5. Constructive Positioning**
Framed as a "governance health check" and ecosystem improvement tool, not a competition. Emphasis on helping DAOs improve, with clear methodology transparency and easy opt-out to maintain trust.

## Project Classification

**Technical Type:** web_app
**Domain:** Web3 Analytics / DAO Governance
**Complexity:** Medium
**Project Context:** Greenfield feature - new capability for existing ChainSights platform

**Rationale:**
- **Web App:** Public-facing browser-based leaderboard requiring responsive design, SEO optimization, and weekly update cadence
- **Web3 Domain:** Specialized analytics for DAO governance using Snapshot.org API and blockchain verification
- **Medium Complexity:** Requires weighted ranking algorithms (5 criteria), historical tracking, delta calculations, controversy management strategy, and domain knowledge of DAO governance mechanisms. Legal disclaimers needed but not heavy regulatory compliance like fintech/healthcare.
- **Integration Context:** Builds on existing Next.js 14 + TypeScript + Drizzle ORM + PostgreSQL (Neon) stack. Will integrate with existing free tier system and reporting infrastructure.

## Success Criteria

### User Success

**For DAO Communities (Primary Users):**

**Clarity & Comprehension:**
- Users land on leaderboard and immediately understand their DAO's rank and score within **<30 seconds**
- Users can explain (roughly) how GVS is calculated after reading methodology page (target: >70% comprehension in post-launch surveys)
- Category breakdown (HPR, DEI, PDI, GPI) provides actionable insights, not just numbers

**Emotional Success Moments:**
- **High-ranked DAOs:** Feel pride and share rankings voluntarily on Twitter/Discord
- **Mid-ranked DAOs:** See improvement opportunities and create action plans
- **Low-ranked DAOs:** Understand methodology is fair and either (a) accept assessment constructively or (b) opt-out without friction

**Engagement Indicators:**
- Users return weekly to check ranking changes (repeat engagement)
- Users click through to methodology page to understand scoring (>50% of leaderboard visitors)
- Users explore category breakdowns and competitor comparisons (average >2 minutes time-on-page)

**Action Triggers:**
- DAO core teams reference ChainSights rankings in internal governance discussions
- DAOs implement governance improvements based on score feedback
- Community members tag their DAOs on social media asking for scores

**Trust & Acceptance:**
- Opt-out rate <5% (indicates fair perception)
- "Thank you" messages outnumber complaint messages 3:1
- Zero perception of "biased" or "pay-to-win" rankings

---

### Business Success

**Week 1 Success Indicators:**
- Launch with 6 Featured Analysis DAOs (Gitcoin, Arbitrum, ENS, Balancer, Uniswap, Lido)
- 2-5 social media shares from DAO communities or members
- 0 opt-out requests (indicates constructive reception)
- 10+ clicks on "Get Report" CTAs from leaderboard traffic
- 5+ positive feedback responses ("this is useful")

**Month 1 Success Indicators:**
- 15-20 total DAOs on leaderboard (mix of Featured + Customer Reports)
- 10+ customer reports sold (€49-149 each)
- 60%+ customer opt-in rate for public listing
- 20%+ marketing opt-in rate on free score lookups
- 5+ paid report purchases directly attributed to leaderboard traffic
- 1-2 methodology improvement suggestions from community feedback

**Month 3 Success Indicators:**
- 30+ DAOs ranked on leaderboard
- 10+ paid reports per month from leaderboard-driven traffic
- Featured in crypto Twitter discussions or crypto newsletters
- Methodology v2 published incorporating community input
- ChainSights cited as "governance rating authority" in at least 3 external articles/podcasts

**Conversion Funnel Health:**
- Leaderboard views → Profile clicks: 40%+ CTR
- Leaderboard views → Free score check: 15%+ conversion
- Free score check → Marketing opt-in: 20%+ conversion
- Leaderboard → Paid report purchase: 1-2% conversion

**Brand Positioning:**
- Positioned as "Moody's for DAO governance" in media mentions
- DAOs reference each other's ChainSights scores in governance forums
- GVS (Governance Vitality Score) becomes quotable metric in crypto media

**Network Effects:**
- DAOs sharing rankings drives organic traffic (30%+ of ranked DAOs tweet about it)
- Community members tag underperforming DAOs → organic reach
- Lower-ranked DAOs purchase detailed reports to "tell their full story" (controversy → sales conversion)

---

### Technical Success

**Reliability & Uptime:**
- 99.5% uptime for leaderboard page (allows ~3.6 hours downtime per month)
- Weekly GVS calculation job success rate: 100% (Sunday midnight UTC execution)
- Zero data loss in historical GVS snapshots (required for week-over-week delta calculations)
- Graceful degradation if Snapshot API is down (cached data with "last updated" timestamp)

**Performance Benchmarks:**
- Leaderboard page load time: <2 seconds at p95 (95th percentile)
- GVS calculation completion time: <10 minutes per DAO (supports scaling to 100+ DAOs)
- API response time for leaderboard queries: <500ms at p95
- Cache hit rate for leaderboard queries: >95% (reduces database load)

**Data Quality & Integrity:**
- GVS calculation accuracy validated against manual calculations (>99% alignment)
- Component scores (HPR, DEI, PDI, GPI) mathematically sum to total GVS correctly (100% accuracy)
- Confidence indicators accurately reflect data completeness (High/Medium/Low thresholds enforced)
- No "black box" calculations - every score traceable to source data

**Scalability Readiness:**
- Architecture supports linear scaling from 6 DAOs (Week 1) to 100+ DAOs (Month 6+)
- Database queries optimized for large result sets (indexed rankings table)
- Weekly job can handle 100+ DAO calculations within 2-hour window
- Frontend remains responsive with 500+ ranked DAOs (pagination/infinite scroll)

**Security & Compliance:**
- Rate limiting on API endpoints (prevents abuse/scraping)
- Input validation on all user-submitted data (DAO names, email addresses)
- HTTPS-only access to leaderboard and methodology pages
- GDPR-compliant data handling (no personal data stored beyond consented emails)

---

### Measurable Outcomes

**Quantitative Metrics (3-Month Horizon):**

| Metric | Week 1 Target | Month 1 Target | Month 3 Target |
|--------|---------------|----------------|----------------|
| **DAOs Ranked** | 6 | 15-20 | 30+ |
| **Customer Reports Sold** | 0-2 | 10+ | 30+ |
| **Opt-In Rate** | N/A | 60% | 70% |
| **Leaderboard Traffic** | 500+ views | 2,000+ views | 5,000+ views |
| **Social Shares** | 2-5 | 10+ | 30+ |
| **Paid Reports from Leaderboard** | 0-1 | 5+ | 10+/month |
| **Opt-Out Requests** | 0 | <3 | <5 |
| **Media Mentions** | 0 | 1-2 | 3+ |

**Qualitative Metrics:**
- Sentiment analysis of social media mentions (target: 70%+ positive/constructive)
- "Was this helpful?" feedback on methodology page (target: 80%+ yes)
- Community perception of fairness (target: "transparent and objective" framing in >60% of discussions)

**Success Validation:**
- At least one DAO publicly shares governance improvement based on GVS feedback
- At least one external party cites ChainSights/GVS as authoritative source
- Zero legal threats or defamation claims (validates legal disclaimers effectiveness)

---

## Product Scope

### MVP - Minimum Viable Product (Launch: Week 1)

**Core Features:**
- ✅ Leaderboard page with 6 Featured Analysis DAOs
- ✅ GVS calculation engine (HPR, DEI, PDI, GPI components)
- ✅ Score display with labels (Vital/Stable/Concerning/Critical)
- ✅ Badge system (Featured Analysis vs Customer Report)
- ✅ Methodology page with complete GVS documentation
- ✅ Legal disclaimers and opt-out policy
- ✅ Basic category filter (All/DeFi/Infrastructure/Public Goods)
- ✅ Responsive design (mobile + desktop)
- ✅ Static rankings (no change indicators yet - first week baseline)

**Data Infrastructure:**
- ✅ Snapshot API integration (proposal and vote data)
- ✅ GVS scores table (stores historical calculations)
- ✅ Leaderboard entries table (current + previous scores)
- ✅ Weekly cron job (Sunday midnight UTC GVS recalculation)

**Customer Journey:**
- ✅ Order form with leaderboard opt-in checkbox
- ✅ Opt-out email handling (24-hour removal guarantee)
- ✅ "Get Scored" CTA on leaderboard page

**Out of Scope for MVP:**
- ❌ Ticker/marquee on landing page (wait until 30+ DAOs)
- ❌ Historical GVS trend charts (Week 2+)
- ❌ Free quick-check scores (Phase 2)
- ❌ API access for external integrations
- ❌ On-chain governance support (Tally, Governor contracts)
- ❌ Self-service dashboard for DAOs

---

### Growth Features (Post-MVP: Month 2-3)

**Enhanced Leaderboard:**
- Ticker/marquee on landing page showing live rankings (requires 30+ DAOs)
- Historical GVS trend visualization (line charts showing 90-day trends)
- Advanced filtering (by category, score range, change direction)
- Search functionality (find specific DAOs quickly)
- Component breakdown on hover/click (HPR/DEI/PDI/GPI details)

**Free Tier Lead Generation:**
- Free GVS quick-check (score-only lookup without full report)
- Lead capture form with marketing opt-in
- Monthly quota system (1 free check per email/wallet per month)
- "Upgrade to Full Report" upsell flow

**API & Integrations:**
- Public API endpoints for GVS queries
- API key authentication with tiered access (Free/Pro/Enterprise)
- Webhook notifications for significant score changes
- Embeddable leaderboard widget for DAO websites

**Community Features:**
- "Was this helpful?" feedback widget on methodology page
- User-submitted methodology improvement suggestions
- DAO-owned profile pages (claim your listing, add context)
- Public comments/discussion threads on rankings (moderated)

---

### Vision (Future: Month 6+)

**Advanced Analytics:**
- Predictive GVS (score trajectory modeling based on trends)
- Custom weight configuration for enterprise clients (adjust HPR/DEI/PDI/GPI importance)
- Peer group benchmarking (compare against similar DAOs automatically)
- Governance health alerts (email when score drops >10 points)

**Platform Expansion:**
- Multi-chain support (Polygon, Arbitrum, Optimism, Base)
- On-chain governance support (Tally, Governor contracts, Agora)
- Multi-platform support (beyond Snapshot)
- Hybrid governance (on-chain + off-chain combined scoring)

**Monetization Evolution:**
- GVS certification badge for DAOs (premium feature)
- White-label leaderboard licensing for DAO tooling platforms
- Enterprise dashboards with custom reporting
- Governance consulting services upsell

**Community & Trust:**
- Open methodology evolution (quarterly community input cycles)
- Methodology v2, v3 with versioned scoring
- Academic partnerships (governance research collaborations)
- Industry standard positioning ("The S&P of DAO Governance")

## User Journeys

### Journey 1: Sarah Chen - Community Member Discovering Her DAO's Health

Sarah is an active Gitcoin community member who votes on most proposals and believes in public goods funding. She's been hearing murmurs in Discord about "governance issues at other DAOs" but doesn't have a clear picture of how Gitcoin compares. One evening, while scrolling Twitter, she sees a tweet: "New DAO Governance Leaderboard just dropped - Gitcoin ranked #1!"

Curious, Sarah clicks through to ChainSights. Within 15 seconds, she understands: Gitcoin scored 74/100 (Vital) and leads the rankings. She explores the GVS breakdown - strong Human Participation (78), excellent Delegate Engagement (72), healthy Power Dynamics (70), and impressive Grassroots Index (76). The methodology page explains exactly how these scores work.

Sarah screenshots the ranking and shares it in the Gitcoin Discord: "Proud moment - we're leading in governance health! 🎉" The core team responds with appreciation, and the discussion shifts to "how do we stay #1?" Sarah feels validated that her participation matters - it's not just voting into the void, it's contributing to measurable governance health.

**This journey reveals requirements for:**
- Fast comprehension (<30 seconds to understand rankings)
- Clear score breakdown with category explanations
- Social sharing functionality (screenshots, share buttons)
- Methodology transparency (build trust through explainability)
- Mobile-responsive design (Sarah discovered on Twitter/mobile)

---

### Journey 2: Marcus DeFi - DAO Core Team Member Facing Reality

Marcus leads governance operations for a mid-sized DeFi protocol. He knows voter turnout has been declining, but the team focuses on TVL and protocol metrics. One Monday morning, he receives a DM: "Have you seen your DAO's governance score on ChainSights? You're ranked #18 with 'Concerning' label."

Marcus visits the leaderboard, sees his DAO scored 52/100, and feels defensive. But as he reads the methodology and component breakdown, he realizes the assessment is fair: participation is down to 12% (score: 35), delegate engagement is weak (score: 48), and small holders aren't participating (score: 40). The data doesn't lie.

Instead of opting out, Marcus presents the ChainSights analysis to the core team. They commission a full €49 report to get detailed recommendations. The report reveals that their governance UI is confusing and delegates aren't getting notifications. Six months later, after implementing changes, their score improves to 67/100. Marcus emails ChainSights: "Thanks for the wake-up call. We needed this."

**This journey reveals requirements for:**
- Constructive framing (criticism → improvement opportunity)
- Detailed component breakdown showing specific weaknesses
- Upsell path from free leaderboard to paid detailed report
- Opt-out option (maintain trust even when scores are low)
- Historical tracking (show improvements over time)
- Email collection for follow-up engagement

---

### Journey 3: Alex Research - Ecosystem Analyst Finding Signal

Alex is a governance researcher writing a report on DAO maturity trends for a crypto venture fund. She's been manually collecting data from 40+ DAOs - Snapshot participation rates, Gini coefficients from Etherscan, delegate data from scattered sources. It's tedious, inconsistent, and she's not confident in her comparisons because methodologies vary.

She discovers ChainSights leaderboard while researching ENS. Suddenly, she has 30+ DAOs scored with a consistent methodology (GVS). She can sort by category (DeFi vs Infrastructure vs Public Goods), compare scores, and understand trends. The methodology page gives her confidence to cite the data in her research.

Alex reaches out to ChainSights asking about API access for her analysis. She becomes an early API customer and cites ChainSights GVS in her published research: "According to ChainSights governance analysis..." The report gets picked up by crypto media, driving traffic back to the leaderboard.

**This journey reveals requirements for:**
- Sortable/filterable leaderboard (by category, score, change)
- Consistent methodology across all DAOs (apples-to-apples comparison)
- Exportable data or API access (future growth feature)
- Citation-worthy credibility (methodology transparency + disclaimers)
- B2B/Enterprise upsell path (API access, custom reporting)

---

### Journey 4: Mario (ChainSights Admin) - Managing Featured Analyses

It's Sunday morning, and Mario reviews the leaderboard before the weekly GVS recalculation job runs. He notices Lido's score dropped from 47 to 42 - a significant decline. He investigates the data: delegate engagement dropped after a key delegate became inactive. The algorithm flagged it correctly.

Mario decides to add a "Featured Analysis" badge to Lido's entry with a note: "Score reflects recent delegate inactivity - common in transitional periods." He also queues up a Tweet for Monday: "Weekly update: Notable governance shifts this week..." tagging the affected DAOs constructively.

An hour later, he receives an email: "Please remove our DAO from your leaderboard." Mario checks the admin panel, clicks "Opt-Out Request," selects the DAO, and confirms removal. The system automatically sends a confirmation email and removes the DAO within 15 minutes. Mario logs the opt-out for methodology review - if patterns emerge, the scoring might need adjustment.

**This journey reveals requirements for:**
- Admin dashboard for managing Featured Analyses
- Manual override capability (add context notes, editorial badges)
- Opt-out request handling workflow (24-hour guarantee)
- Weekly job monitoring and error handling
- Social media content generation (rankings → marketing content)
- Audit logs for compliance (opt-outs, score changes, methodology updates)

---

### Journey 5: Priya Customer - Commissioning a Report

Priya is the governance lead for a new L2 DAO. She sees competitors ranked on ChainSights and wonders where her DAO would stand. She clicks "Get Scored → €49" and fills out the order form. There's a checkbox: "☑️ List my DAO on the ChainSights Leaderboard (optional free visibility)."

Priya hesitates - what if the score is bad? But she reads: "You can request removal anytime" and checks the box. She pays €49 via Stripe.

Three days later, she receives a comprehensive PDF report: GVS score 68 (Stable), with detailed component breakdowns, benchmarking against similar L2 DAOs, and specific recommendations (e.g., "Consider implementing delegate incentives - your DEI score of 55 could improve to 70+").

Her DAO appears on the leaderboard as #9 with a "✓ Customer Report" badge. Priya shares it with the core team and on Twitter: "Just got our governance health report from @ChainSights - we're ranked #9! Here's how we're improving..." The tweet drives 15 clicks to the leaderboard and 3 new report orders.

**This journey reveals requirements for:**
- Simple order flow (name, email, DAO, payment)
- Opt-in checkbox with clear consent language
- Payment processing (Stripe integration)
- Report generation workflow (3-day SLA)
- Email delivery of PDF report
- Automatic leaderboard listing after report completion
- Customer badge display ("Customer Report" vs "Featured Analysis")
- Report quality that drives social sharing (detailed, actionable insights)

---

### Journey 6: David Controversy - DAO Representative Disputing Rankings

David is a community manager for a DAO ranked #22 with a score of 51 (Concerning). He's furious. He tweets: "@ChainSights your ranking is bullshit. Our governance is strong - you just don't understand our model."

Maya (ChainSights marketing) sees the tweet and responds within 2 hours using the playbook: "Thanks for feedback! GVS is ONE perspective based on public data. We know it's not perfect. What metrics do you think we're missing? We're refining our approach based on community input. Also, if you'd rather not be listed, we'll remove you - no hard feelings."

David calms down and actually reads the methodology page. He realizes the low score is because their delegate system isn't on-chain and ChainSights couldn't detect it. He replies: "Fair point - our governance is mostly off-chain. Can you handle that in future versions?"

Maya responds: "Great feedback! We're tracking requests for hybrid on-chain/off-chain support. Want to get a detailed report to tell your full story? €49 includes qualitative analysis." David doesn't buy immediately, but he stops criticizing and adds ChainSights to his watchlist. Three months later, when they improve on-chain governance, he commissions a report.

**This journey reveals requirements for:**
- Social media monitoring (track mentions of @ChainSights)
- Response playbook and templates (Maya's guidelines)
- Fast response time (<24 hours for criticism)
- Methodology evolution tracking (log feature requests)
- Conversion path from critic → customer (controversy → sales)
- Clear disclaimers preventing legal liability (opinions not facts)
- Opt-out mechanism as de-escalation tool

---

### Journey Requirements Summary

These journeys reveal the following capability areas needed for the DAO Rankings Leaderboard:

**Core Leaderboard Functionality:**
- Fast-loading rankings table with sort/filter by category
- GVS score display with color-coded labels (Vital/Stable/Concerning/Critical)
- Component score breakdown (HPR, DEI, PDI, GPI) on click/hover
- Badge system (Featured Analysis vs Customer Report)
- Social sharing optimization (Open Graph tags, screenshot-friendly design)
- Mobile-responsive design

**Methodology & Trust:**
- Comprehensive methodology page explaining GVS calculation
- Data source transparency and citations
- Legal disclaimers and editorial independence statements
- Confidence indicators (High/Medium/Low data quality)

**Customer Journey:**
- "Get Scored" CTA with clear pricing (€49-149)
- Order form with DAO info, email, payment
- Leaderboard opt-in checkbox with consent language
- Stripe payment integration
- Report generation workflow (automated GVS calculation + PDF export)
- Email delivery system
- Report quality (detailed, actionable, shareable)

**Admin Operations:**
- Admin dashboard for managing Featured Analyses
- Weekly GVS calculation job (Sunday midnight UTC)
- Opt-out request handling (24-hour removal)
- Manual curation tools (add context notes, editorial badges)
- Audit logs (opt-outs, score changes, data sources)
- Social media content generation support

**Controversy Management:**
- Opt-out form with simple removal process
- Social media monitoring and response workflows
- Maya's response playbook implementation
- Feedback collection mechanism ("Was this helpful?")
- Methodology evolution tracking (community feature requests)
- Conversion paths from criticism to customer engagement

**Growth & Scaling:**
- Historical score tracking (week-over-week changes)
- API access for researchers and partners (future)
- Embeddable widgets for DAO websites (future)
- Category/peer group benchmarking
- Ticker/marquee for landing page (30+ DAOs)
- Self-service dashboard for DAOs (future)

## Innovation & Novel Patterns

### Detected Innovation Areas

**1. Governance Vitality Score (GVS) - Identity-Aware Governance Analytics**

The core innovation of ChainSights is the **Governance Vitality Score (GVS)**, a proprietary composite index (0-100) that measures DAO governance health through identity-aware analytics. Unlike existing DAO analytics platforms (Dune, Nansen, DeepDAO) that count wallets and voting power, GVS reveals **actual human participation**.

**Tagline: "Wallets lie. We don't."**

**What Makes It Novel:**
- **Wallet Clustering Technology:** Groups multiple wallets controlled by the same entity to calculate true unique human participation
- **Human Participation Rate (HPR):** Measures `(Unique Humans / Active Wallets) × 100` - impossible without identity resolution
- **Four-Component Formula:** `GVS = (HPR × 0.35) + (DEI × 0.25) + (PDI × 0.20) + (GPI × 0.20)`
  - HPR: Human Participation Rate (35% weight)
  - DEI: Delegate Engagement Index (25% weight)
  - PDI: Power Dynamics Index (20% weight)
  - GPI: Grassroots Participation Index (20% weight)

**Competitive Advantage:**
Existing platforms cannot replicate GVS without building wallet clustering infrastructure. This creates a **technical moat** around ChainSights' methodology.

---

**2. Movement-Based Virality Mechanics**

Instead of static governance scores, ChainSights displays **week-over-week changes** (+0.3 ↗, -0.5 ↘, 0.0 →) inspired by stock market tickers. This transforms governance assessment from "report card" to "live scoreboard."

**Why This is Novel:**
- Creates **recurring engagement** - users return weekly to check changes
- Generates **narrative and stories** - "Arbitrum jumped 3 spots!" is shareable content
- Drives **FOMO and social proof** - rank changes become Twitter threads
- Positions governance health as **dynamic metric**, not static assessment

**Innovation Context:**
While CoinMarketCap pioneered price tickers, ChainSights is applying this pattern to **governance quality** - a non-financial, qualitative metric. No other governance platform tracks and displays week-over-week changes in this format.

---

**3. Controversy as Growth Engine**

ChainSights deliberately **embraces controversy** as a product strategy, not a risk to avoid. Lower-ranked DAOs disputing their scores drives engagement, discussion, and ultimately conversions.

**Novel Positioning:**
- **5-Layer Defense Strategy:** Transparency, Nuance, Empathy, Legal Protection, Response Playbook
- **Criticism → Sales Pipeline:** "Your score is wrong!" → "Get the full story with our detailed report (€49)"
- **Opt-Out as Trust Signal:** Easy removal (24-hour guarantee) prevents hostage perception
- **Constructive Framing:** "Governance health check," not "DAO competition"

**Why This Works:**
- Web3 culture values **transparency and open debate**
- Controversy generates **organic reach** through community discussions
- Lower scores create **actionable improvement paths** (not just criticism)
- Response playbook converts critics into **engaged customers or advocates**

**Innovation Risk:**
If defense strategy fails or methodology is perceived as biased, controversy could damage brand. Mitigation: methodology transparency, editorial independence statements, rapid response protocols.

---

**4. Hybrid Monetization Model**

ChainSights combines **public visibility** with **opt-in listing**, creating a unique value exchange:

**Free Tier Innovation:**
- Public leaderboard with Featured Analyses (ChainSights-curated)
- Customer reports appear IF customer opts-in at purchase
- Free visibility incentivizes opt-in (60-70% target rate)

**Paid Tier:**
- Detailed GVS reports (€49-149) with component breakdowns
- Historical tracking and benchmarking
- Actionable recommendations

**Why This is Novel:**
- **Free ≠ Limited:** Free leaderboard shows full GVS scores, not "teaser" data
- **Opt-In ≠ Required:** Customers choose visibility (privacy-friendly)
- **Listing = Marketing Asset:** Customers' reports become content marketing for ChainSights

**Market Context:**
- Moody's/S&P: Ratings are published without company consent (regulatory model)
- Glassdoor: Companies claim profiles but all reviews are public
- ChainSights: Hybrid - Featured Analyses (editorial) + Customer Opt-Ins (consented)

This balances **editorial independence** (Featured Analyses) with **customer agency** (opt-in choice).

---

### Market Context & Competitive Landscape

**Existing DAO Analytics Platforms:**

| Platform | Focus | What They Measure | What They Miss |
|----------|-------|-------------------|----------------|
| **DeepDAO** | Treasury + participation | Token holders, voting power, treasury size | Human vs wallet distinction, delegate quality |
| **Dune Analytics** | Custom queries | On-chain metrics (raw data) | Identity resolution, composite health scores |
| **Nansen** | Wallet labeling | "Smart money" tracking, token flows | Governance-specific metrics, delegate engagement |
| **Tally** | Governance UI | On-chain voting interface + basic stats | Off-chain governance, human participation |
| **Snapshot** | Voting platform | Proposal creation + vote recording | Analysis layer, quality assessment, benchmarking |

**ChainSights Differentiation:**
- **Only platform** with identity-aware governance analytics (wallet clustering)
- **Only platform** with single composite governance health score (GVS)
- **Only platform** framing governance as competitive leaderboard with change indicators
- **Only platform** positioning controversy as growth mechanic

**Market Gap:**
Current platforms provide **data** (voting records, token distributions). ChainSights provides **insight** (governance health assessment). Analogy: Dune is Bloomberg Terminal, ChainSights is Moody's.

---

### Validation Approach

**1. GVS Methodology Validation**

**Pre-Launch:**
- **Manual calibration** against 20 known DAOs (10 healthy, 10 problematic)
- **Expert review** with governance researchers (validate formula makes sense)
- **Reference benchmarks:** Expected GVS ranges documented in spec (e.g., Gitcoin 70-85, Lido 40-55)

**Post-Launch:**
- **Correlation analysis:** Does GVS correlate with qualitative governance reputation?
- **Predictive validation:** Do governance incidents (attacks, voter apathy crises) correspond to low GVS?
- **Community feedback:** Track methodology improvement suggestions

**Success Criteria:**
- >90% alignment between GVS classification (Vital/Stable/Concerning/Critical) and expert assessment
- <5% opt-out rate in first 3 months (indicates fairness perception)
- Zero successful legal challenges to methodology

---

**2. Leaderboard Engagement Validation**

**Hypothesis:** Movement-based display drives recurring engagement

**Metrics to Track:**
- **Repeat visitor rate:** % of users who return weekly to check changes
- **Time-on-page increase:** Do +/- indicators increase engagement time vs static scores?
- **Social sharing:** Are rank changes shared more than static scores?

**Validation Timeline:**
- Week 1: Baseline (static rankings, no deltas)
- Week 2+: Enable week-over-week changes
- Month 1: Compare engagement metrics (before/after deltas)

**Success Criteria:**
- 30%+ repeat visitor rate (users returning weekly)
- 2x social sharing rate for DAOs with rank changes vs unchanged
- >2 minutes average time-on-page

---

**3. Controversy-to-Conversion Validation**

**Hypothesis:** Criticism converts to sales through 5-layer defense strategy

**Metrics to Track:**
- **Criticism volume:** # of negative social media mentions
- **Response effectiveness:** % of critics who engage constructively after response
- **Conversion rate:** % of controversial DAOs who purchase detailed reports within 30 days
- **Opt-out rate:** Do criticized DAOs opt-out or engage?

**Success Criteria:**
- <10% opt-out rate among criticized DAOs (most engage, don't exit)
- 5-10% conversion rate from criticism → paid report purchase
- 70%+ positive/constructive sentiment in follow-up engagement

---

### Risk Mitigation

**Innovation Risk 1: GVS Formula Rejection**

**Risk:** Community rejects GVS methodology as "flawed" or "biased"

**Indicators:**
- >10% opt-out rate in first month
- Widespread criticism across multiple DAOs
- Media framing as "controversial rankings" (negative)

**Mitigation:**
- **Transparency:** Full methodology disclosure + open formula
- **Versioning:** Methodology v1.0 → v2.0 based on feedback (quarterly updates)
- **Advisory council:** Governance experts review methodology changes
- **Confidence indicators:** Show data quality (High/Medium/Low) to set expectations
- **Alternative views:** Offer category-specific benchmarking (DeFi vs Infrastructure)

**Fallback:**
- Pivot to "Governance Insights Platform" (remove "leaderboard" language)
- Offer custom GVS weight configuration for enterprise clients
- Add qualitative narrative alongside quantitative scores

---

**Innovation Risk 2: Controversy Backfires**

**Risk:** Defense strategy fails; ChainSights becomes "toxic" in DAO communities

**Indicators:**
- Coordinated criticism from multiple DAOs
- Calls for boycott or blacklisting
- Legal threats or defamation claims
- Influencers amplifying negative narrative

**Mitigation:**
- **Legal firewall:** Disclaimers prevent defamation liability (opinions, not facts)
- **Rapid response:** <24 hour response time to criticism (Maya's playbook)
- **Olive branch:** Proactive outreach to controversial DAOs before public listing
- **Opt-out guarantee:** 24-hour removal prevents hostage perception
- **Constructive framing:** Every response focuses on improvement, not shame

**Fallback:**
- Pause Featured Analyses (only Customer Reports with opt-in)
- Require pre-approval from all DAOs before listing (pivot to fully consensual model)
- Add "Request Analysis" feature (DAOs apply to be ranked)

---

**Innovation Risk 3: Wallet Clustering Accuracy**

**Risk:** Identity resolution fails; HPR (Human Participation Rate) is inaccurate

**Indicators:**
- Low confidence scores (<50% identity resolution) for most DAOs
- Community disputes about "unique humans" calculations
- DAOs prove clustering is wrong (e.g., "we have 100 humans, not 50")

**Mitigation:**
- **Confidence thresholds:** Don't publish GVS if identity resolution <50%
- **Transparency:** Show confidence level + data completeness for every score
- **Manual review:** Human-in-loop validation for controversial cases
- **Data sources:** Multiple signals (ENS, delegate registries, behavioral analysis)
- **Improvement cycle:** Clustering algorithm improves with more data

**Fallback:**
- Downweight HPR component (35% → 20%) if clustering unreliable
- Offer "Verified GVS" tier where DAOs submit delegate/participant lists
- Pivot to delegate-focused metrics (DEI, PDI) which don't require wallet clustering

---

**Innovation Risk 4: Competitive Response**

**Risk:** Competitors (DeepDAO, Nansen) add governance health scores

**Indicators:**
- Competitor launches similar composite score
- Competitor adds wallet clustering capability
- Competitor frames as "leaderboard"

**Mitigation:**
- **Speed to market:** Launch before competitors mobilize
- **Brand positioning:** Own "Moody's for DAOs" narrative first
- **Proprietary data:** Wallet clustering is technical moat (6-12 month lead time)
- **Network effects:** More DAOs = better benchmarking = stronger moat
- **Methodology evolution:** v2, v3 keep innovating (maintain lead)

**Competitive Advantage Timeline:**
- Month 1-3: Proprietary GVS + leaderboard format (unique)
- Month 4-9: Competitors may start copying
- Month 10+: ChainSights has historical data, brand recognition, methodology refinement

**Defensibility:**
- **GVS trademark:** Protect brand name
- **Methodology versioning:** Stay ahead with v2, v3 improvements
- **Customer lock-in:** Historical tracking data (switching cost)
- **Editorial authority:** "Original" governance leaderboard (first-mover advantage)

## Web Application Specific Requirements

### Project-Type Overview

The DAO Rankings Leaderboard is a **Next.js 14 App Router web application** delivering server-side rendered (SSR) content with client-side interactivity. The architecture balances **SEO requirements** (public content marketing tool) with **dynamic user experiences** (sorting, filtering, change indicators).

**Key Architectural Decisions:**
- **Hybrid SSR/CSR:** Server-side render initial leaderboard data for SEO + crawlers, client-side hydrate for interactivity (sort, filter, hover states)
- **Static Generation:** Methodology page and legal disclaimers are fully static (build-time generation)
- **Incremental Static Regeneration (ISR):** Leaderboard page uses ISR with 1-hour revalidation (balance freshness vs performance)
- **Weekly Data Updates:** GVS scores update Sunday midnight UTC, but page remains accessible 24/7 with cached data

**Browser-Based Experience:**
- No native app requirements (mobile users access via browser)
- No CLI requirements (admin operations via web dashboard)
- Progressive Web App (PWA) features optional (future enhancement for mobile users)

---

### Browser Support Matrix

**Supported Browsers (Primary):**

| Browser | Minimum Version | Market Share (Target Users) | Testing Priority |
|---------|-----------------|----------------------------|------------------|
| **Chrome** | v120+ (Dec 2023) | 45% crypto users | **HIGH** |
| **Firefox** | v120+ (Nov 2023) | 15% crypto users | **HIGH** |
| **Safari** | v17+ (iOS 17, macOS Sonoma) | 25% crypto users | **HIGH** |
| **Edge** | v120+ (Chromium-based) | 10% crypto users | **MEDIUM** |
| **Brave** | v1.60+ (Chromium fork) | 5% crypto users | **MEDIUM** |

**Browser Requirements Rationale:**
- **Modern ES6+ JavaScript:** ChainSights uses Next.js 14 + TypeScript (requires ES2020+ support)
- **CSS Grid/Flexbox:** Leaderboard table layout requires modern CSS
- **Web Crypto API:** Future wallet authentication features
- **LocalStorage:** Caching user preferences (sort order, filters)

**Unsupported Browsers:**
- ❌ Internet Explorer (all versions) - EOL 2022, <0.5% crypto user share
- ❌ Safari <15 (iOS <15) - Lacks modern JS features, security risks
- ❌ Chrome <100 - Outdated security patches

**Browser Testing Strategy:**
- **Manual Testing:** Chrome (primary), Safari iOS (mobile), Firefox (desktop)
- **Automated Testing:** Playwright cross-browser matrix (Chrome, Firefox, WebKit)
- **Error Tracking:** Sentry captures browser-specific JS errors with version tagging

---

### Responsive Design Requirements

**Target Devices & Breakpoints:**

| Device Class | Breakpoint | Viewport | Usage Pattern | Design Priority |
|--------------|------------|----------|---------------|-----------------|
| **Mobile** | 320px - 767px | Portrait phone | Quick checks, social sharing | **HIGH** |
| **Tablet** | 768px - 1023px | iPad, tablet | Extended browsing, comparisons | **MEDIUM** |
| **Desktop** | 1024px+ | Laptop, monitor | Deep analysis, multi-tab research | **HIGH** |
| **Wide Desktop** | 1440px+ | Large monitors | Admin operations, data entry | **LOW** |

**Mobile-First Design Principles:**

**Leaderboard Page (Mobile):**
- **Vertical scrolling table:** Rank, DAO name, score, badge visible without horizontal scroll
- **Collapsible rows:** Tap to expand component scores (HPR, DEI, PDI, GPI)
- **Fixed header:** Rank column remains visible during scroll
- **Touch-optimized:** 44px minimum touch target size for CTAs
- **Social sharing:** Native share API integration ("Share this ranking")

**Methodology Page (Mobile):**
- **Progressive disclosure:** Collapsible accordion sections for ranking criteria
- **Sticky navigation:** Jump-to-section menu remains accessible
- **Readability:** 16px base font size, 1.6 line-height for long-form content

**Tablet Optimizations:**
- **Two-column layout:** Methodology content uses wider viewport
- **Side-by-side comparisons:** Show 2-3 DAOs simultaneously for benchmarking
- **Hybrid touch/mouse:** Support both touch and cursor interactions

**Desktop Enhancements:**
- **Full-width table:** Display all leaderboard columns without scrolling (Rank, DAO, Score, Change, Category, Badge)
- **Hover states:** Component score tooltips on hover (no click required)
- **Keyboard navigation:** Tab through table rows, Enter to expand details
- **Multi-column methodology:** Side-by-side criteria explanations

**Responsive Testing Requirements:**
- **Manual testing:** iPhone 14 (390px), iPad Air (820px), MacBook Pro (1440px)
- **Automated testing:** Playwright viewport matrix (375px, 768px, 1280px, 1920px)
- **Visual regression testing:** Percy.io snapshots across breakpoints

---

### Performance Targets

**Page Load Performance:**

| Metric | Target (p95) | Acceptable (p99) | Rationale |
|--------|--------------|------------------|-----------|
| **First Contentful Paint (FCP)** | <1.5s | <2.5s | Users see leaderboard skeleton within 1.5s |
| **Largest Contentful Paint (LCP)** | <2.0s | <3.0s | Full leaderboard table visible within 2s |
| **Time to Interactive (TTI)** | <3.0s | <4.5s | Users can interact (sort, filter) within 3s |
| **Cumulative Layout Shift (CLS)** | <0.1 | <0.2 | No layout shifts during ranking table load |
| **Total Blocking Time (TBT)** | <200ms | <400ms | Smooth interactions, no jank |

**Asset Optimization:**

| Asset Type | Budget | Optimization Strategy |
|------------|--------|----------------------|
| **JavaScript Bundle** | <150KB gzipped | Code splitting, tree shaking, lazy loading |
| **CSS** | <30KB gzipped | Critical CSS inline, Tailwind purge |
| **Images** | <500KB total | Next.js Image optimization, WebP format |
| **Fonts** | <50KB WOFF2 | Subset Montserrat + Inter, self-hosted |
| **API Responses** | <100KB per query | Paginated results, gzip compression |

**Performance Budget Enforcement:**
- **Build-time checks:** Next.js bundle analyzer fails if bundle >200KB
- **Lighthouse CI:** Automated performance audits on PR merges (minimum score: 85)
- **Real User Monitoring (RUM):** Vercel Analytics tracks Core Web Vitals
- **Performance regression alerts:** Alert if LCP increases >20% week-over-week

**Caching Strategy:**
- **Static pages:** Methodology, disclaimers (Cache-Control: public, max-age=86400)
- **Leaderboard data:** ISR with 1-hour revalidation (stale-while-revalidate)
- **API responses:** Redis cache with 15-minute TTL (reduce Snapshot API load)
- **Images:** CDN cache with 1-year TTL (immutable assets)

**Database Query Optimization:**
- **Indexed queries:** Rankings table indexed on `score DESC, dao_name ASC`
- **Connection pooling:** Neon serverless Postgres with 10-connection pool
- **Query timeouts:** 5-second timeout with graceful degradation
- **N+1 prevention:** Batch fetch component scores with JOIN queries

---

### SEO Strategy

**Critical SEO Requirements:**

The leaderboard is a **content marketing asset** - organic discovery is essential for growth. SEO optimizations directly impact lead generation and brand awareness.

**On-Page SEO:**

**Leaderboard Page (`/rankings`):**
- **Title Tag:** "DAO Governance Rankings | Live Leaderboard by ChainSights" (55 chars)
- **Meta Description:** "Compare DAO governance health scores across 30+ DAOs. Weekly updated rankings with Governance Vitality Score (GVS). Discover top-performing DAOs." (155 chars)
- **H1 Tag:** "DAO Governance Leaderboard"
- **Semantic HTML:** `<table>` with proper `<thead>`, `<tbody>`, `<th>` structure (crawlable by search engines)
- **Schema.org Markup:** ItemList schema with ranked DAOs as ListItem entities
- **Open Graph Tags:** og:title, og:description, og:image (social sharing optimization)
- **Twitter Card:** summary_large_image with leaderboard preview

**Methodology Page (`/rankings/methodology`):**
- **Title Tag:** "How DAO Rankings Work | GVS Methodology | ChainSights" (58 chars)
- **Meta Description:** "Transparent methodology for DAO governance scoring. Learn how we calculate Governance Vitality Score (GVS) with 5 weighted criteria." (134 chars)
- **Content SEO:** 2,000+ word long-form content with keyword targeting (DAO governance, governance health, DAO rankings, decentralization metrics)
- **Internal Linking:** Link to leaderboard page, individual DAO profiles (when available)

**Individual DAO Profile Pages (Future):**
- **URL Structure:** `/rankings/dao/[dao-name]` (e.g., `/rankings/dao/gitcoin`)
- **Title Tag:** "[DAO Name] Governance Score | ChainSights Rankings" (e.g., "Gitcoin Governance Score | ChainSights Rankings")
- **Unique Content:** DAO-specific GVS breakdown, historical trend charts, category context
- **Schema.org:** Organization schema with DAO details + score entity

**Technical SEO:**

| Requirement | Implementation | Impact |
|-------------|----------------|--------|
| **Sitemap** | `/sitemap.xml` with all public pages | Crawl efficiency |
| **Robots.txt** | Allow all, disallow `/admin/*` | Protect admin routes |
| **Canonical URLs** | `<link rel="canonical">` on all pages | Prevent duplicate content |
| **Mobile-Friendly** | Viewport meta tag, responsive design | Mobile search ranking |
| **HTTPS Only** | Force HTTPS redirect, HSTS header | Trust signal + ranking boost |
| **Page Speed** | Core Web Vitals optimization | Ranking factor (Google) |
| **Structured Data** | JSON-LD schema for rankings, FAQs | Rich snippets eligibility |

**Content Strategy for SEO:**

**Target Keywords (Primary):**
- "DAO governance rankings" (10-100 monthly searches, low competition)
- "DAO governance health" (10-100 monthly searches)
- "Governance Vitality Score" (brand term, will grow with awareness)
- "Best DAO governance" (100-1K monthly searches, medium competition)

**Target Keywords (Secondary):**
- "[DAO Name] governance score" (e.g., "Gitcoin governance score")
- "DAO decentralization metrics"
- "DAO voter participation"
- "Compare DAO governance"

**Content Plan:**
- **Weekly updates:** "DAO Rankings Update: Week of [Date]" blog posts (fresh content signal)
- **Deep dives:** "Top 10 DAOs for Governance Health in 2025" (long-form, linkable assets)
- **Controversy content:** "Why [DAO] Disagreed with Their Governance Score" (leverages disputes for traffic)

**Link Building Strategy:**
- **Organic mentions:** DAOs sharing rankings → inbound links
- **Media outreach:** Pitch "DAO governance state of the industry" to crypto media
- **Guest posts:** Mario writes for governance-focused publications with ChainSights attribution
- **Academic citations:** Offer GVS data to researchers → .edu backlinks

**SEO Measurement:**
- **Google Search Console:** Track impressions, clicks, CTR for target keywords
- **Organic traffic goal:** 1,000+ organic visits per month by Month 6
- **SERP tracking:** Monitor rankings for "DAO governance" and "DAO rankings"
- **Backlink monitoring:** Ahrefs or similar (track inbound links from DAO websites, media)

---

### Accessibility Level

**Target: WCAG 2.1 Level AA Compliance**

**Rationale:**
- **Web3 community values:** Decentralization principles extend to inclusive design
- **Legal protection:** WCAG AA reduces discrimination liability (EU Accessibility Act 2025)
- **SEO benefit:** Accessible markup improves crawlability and semantic structure
- **Brand positioning:** "Open governance rankings for everyone" includes accessibility

**Accessibility Requirements by Component:**

**Leaderboard Table:**
- **Semantic HTML:** `<table>`, `<caption>`, `<th scope="col">` for screen reader navigation
- **ARIA labels:** `aria-label="DAO Rankings Table"` on table wrapper
- **Keyboard navigation:** Tab through rows, Enter to expand component scores
- **Focus indicators:** Visible focus outline on interactive elements (2px solid aqua)
- **Sort indicators:** `aria-sort="ascending"` when columns are sorted
- **Screen reader announcements:** "Currently sorted by score, descending" on sort action

**Change Indicators (+/- Scores):**
- **Color + icon:** Don't rely solely on color (green/red) - include ↗/↘ arrows
- **ARIA live regions:** Announce "Gitcoin increased by 0.3 points" when filtering updates results
- **Alt text:** `aria-label="Increased by 0.3 points"` on change badges

**Forms (Order, Opt-Out):**
- **Label association:** `<label for="dao-name">` properly linked to inputs
- **Error messages:** `aria-describedby="dao-name-error"` for validation errors
- **Required indicators:** `aria-required="true"` + visual asterisk
- **Focus management:** Move focus to first error on form submission failure

**Interactive Elements:**
- **Button vs Link semantics:** `<button>` for actions (sort, filter), `<a>` for navigation
- **Touch targets:** Minimum 44×44px for mobile (WCAG 2.5.5)
- **Hover + Focus states:** Identical styling for mouse and keyboard users
- **Disabled states:** `disabled` attribute + `aria-disabled="true"` with explanation

**Visual Design:**

| Requirement | Implementation | WCAG Criterion |
|-------------|----------------|----------------|
| **Color contrast** | 4.5:1 text-to-background (7:1 for headings) | 1.4.3 (AA) |
| **Text resize** | Layout intact at 200% zoom | 1.4.4 (AA) |
| **Focus visible** | 2px aqua outline on focus | 2.4.7 (AA) |
| **Reflow** | No horizontal scroll at 320px width | 1.4.10 (AA) |
| **Text spacing** | Support user-defined line height/spacing | 1.4.12 (AA) |

**Screen Reader Testing:**
- **NVDA (Windows):** Test with Chrome (30% screen reader market share)
- **VoiceOver (macOS/iOS):** Test with Safari (25% screen reader market share)
- **Automated testing:** axe-core in Playwright tests (catches 60% of issues)
- **Manual audit:** WAVE browser extension for visual WCAG violations

**Accessibility Documentation:**
- **Accessibility statement:** `/accessibility` page documenting WCAG compliance
- **Feedback mechanism:** Email for accessibility issues (hello@chainsights.one)
- **Regular audits:** Quarterly accessibility review (self-audit + external when budget allows)

---

### Implementation Considerations

**Next.js 14 App Router Specifics:**

**Route Structure:**
```
/app
  /rankings
    /page.tsx             # Main leaderboard (/rankings)
    /methodology
      /page.tsx           # Methodology page (/rankings/methodology)
    /opt-out
      /page.tsx           # Opt-out form (/rankings/opt-out)
    /dao
      /[slug]
        /page.tsx         # Individual DAO profiles (future)
```

**Server Components vs Client Components:**
- **Server Components (default):** Leaderboard data fetching, methodology content (SEO-optimized)
- **Client Components:** Sort/filter controls, hover states, form inputs (use `'use client'`)
- **Hybrid pattern:** Server component fetches data → passes to client component for interactivity

**Data Fetching Patterns:**
- **Leaderboard data:** `fetch()` in Server Component with `revalidate: 3600` (ISR 1-hour cache)
- **GVS calculations:** Server-side API route `/api/gvs/calculate` (not exposed publicly)
- **Historical data:** Batch fetch on-demand when user views trends (lazy loading)

**Environment Configuration:**
- **`NEXT_PUBLIC_*` variables:** Only for client-side code (e.g., `NEXT_PUBLIC_SITE_URL`)
- **Server-only secrets:** Database URLs, API keys (never expose to client bundle)
- **Vercel environment variables:** Production, Preview, Development isolation

**Build & Deployment:**
- **Build command:** `npm run build` (generates optimized production bundle)
- **Output:** `.next` directory with static HTML + JS chunks
- **Deployment target:** Vercel (zero-config Next.js hosting)
- **Preview deployments:** Every PR gets unique URL for testing
- **Production:** `main` branch auto-deploys to `chainsights.one/rankings`

**Error Handling:**
- **API failures:** Graceful degradation - show cached data with "Last updated [timestamp]"
- **404 pages:** Custom `/not-found.tsx` with branded design + navigation back to leaderboard
- **500 errors:** Sentry error tracking + user-friendly error message
- **Stale data alerts:** If GVS data >7 days old, show warning banner

**Security Considerations:**
- **Rate limiting:** Vercel Edge Middleware (100 requests/minute per IP)
- **Input sanitization:** Validate all user inputs (DAO names, emails) to prevent XSS
- **CORS:** Restrict API endpoints to same-origin only (no public API access yet)
- **Content Security Policy (CSP):** Restrict script sources to prevent injection attacks
- **HTTPS enforcement:** Automatic via Vercel (HSTS header enabled)

**Monitoring & Observability:**
- **Vercel Analytics:** Real-time traffic, Core Web Vitals, conversion tracking
- **Sentry:** Error tracking with source maps (catch production issues)
- **Database monitoring:** Neon dashboard for query performance, connection pooling
- **Uptime monitoring:** UptimeRobot pings `/rankings` every 5 minutes (alerts on downtime)
- **Weekly job monitoring:** Cron job logs + Slack notifications on failure

## Project Scoping & Phased Development

### MVP Strategy & Philosophy

**MVP Approach: Experience MVP - Viral Proof of Concept**

ChainSights is adopting an **Experience MVP** strategy focused on delivering the complete leaderboard experience with minimal but working functionality. The goal is to prove the **"movement-based virality"** hypothesis: that displaying DAO governance scores with week-over-week change indicators will drive organic social sharing and recurring engagement.

**Strategic Rationale:**

**Why Experience MVP over alternatives:**

1. **Problem-Solving MVP would be too limited:** Just showing scores without rankings, changes, or CTAs wouldn't validate the core hypothesis (controversy → engagement → sales)

2. **Platform MVP would delay learning:** Building comprehensive API, self-service dashboard, and historical tracking infrastructure before validating market fit risks wasted effort

3. **Revenue MVP requires proof:** Need initial traction and social proof before DAOs will pay for reports - free leaderboard builds authority first

**The Experience MVP Strategy:**
- Launch with **6 Featured Analysis DAOs** (Gitcoin, Arbitrum, ENS, Balancer, Uniswap, Lido)
- Deliver the **complete visual experience** (leaderboard, change indicators, CTAs, methodology page)
- Prove **one core loop:** View leaderboard → Share socially → Drive traffic → Convert to paid reports
- **Learn fast:** Will controversy drive sales? Do high-ranked DAOs share? Is opt-out rate <5%?
- **Iterate based on data:** Week 2-4 add customer reports, refine methodology based on feedback

**What Makes Users Say "This is Useful":**
- **DAO Community Members:** "Finally, I can compare our governance to competitors!"
- **DAO Core Teams:** "This gives us a benchmark to track improvement over time"
- **Governance Researchers:** "Consistent scoring methodology across 30+ DAOs is gold"

**What Makes Investors/Partners Say "This Has Potential":**
- Week 1 social sharing from ranked DAOs (proof of organic reach)
- <5% opt-out rate (proof methodology is fair and accepted)
- First paid report purchases from leaderboard traffic (proof of conversion funnel)
- Media mentions ("Moody's for DAO governance" positioning sticks)

**Fastest Path to Validated Learning:**
Launch leaderboard → Monitor social engagement → Track opt-out rate → Measure conversion → Refine methodology → Expand coverage

---

### Resource Requirements

**MVP Team Structure:**

**Core Team (Week 1-4):**
- **Mario (Founder/Developer):** Full-stack development, GVS calculation engine, leaderboard implementation
- **Maya (Marketing Agent):** Social media monitoring, response playbook execution, content generation
- **UX Designer (Contract/Part-time):** Leaderboard UI/UX, methodology page layout (5-10 hours)

**Build Effort Estimates:**

| Component | Estimated Hours | Complexity |
|-----------|----------------|------------|
| **GVS Calculation Engine** | 8-12 hours | Medium - formula logic, data fetching, confidence scoring |
| **Leaderboard Page** | 4-6 hours | Low - table display, sort/filter, responsive design |
| **Methodology Page** | 2-3 hours | Low - static content with existing copy |
| **Weekly Cron Job** | 3-4 hours | Medium - scheduled GVS recalc, delta calculation |
| **Opt-Out Form** | 2-3 hours | Low - simple form with email handling |
| **Legal Disclaimers** | 1-2 hours | Low - text integration, schema updates |
| **Testing & QA** | 4-6 hours | Medium - cross-browser, mobile, edge cases |
| **Deployment & Monitoring** | 2-3 hours | Low - Vercel deploy, Sentry setup |

**Total MVP Effort:** 26-39 hours (~1 week full-time)

**Post-MVP Team Scaling:**
- **Month 2-3:** Add part-time data analyst for GVS methodology refinement
- **Month 4+:** Add developer for API development, self-service dashboard
- **Month 6+:** Consider FTE developer if scaling to 100+ DAOs

**Critical Skills Required:**
- **TypeScript/Next.js:** ChainSights stack (Mario has)
- **Data Analysis:** GVS component weighting, statistical validation (need consultant)
- **Community Management:** Handling criticism/controversy (Maya handles)
- **Legal/Compliance:** GDPR, disclaimers, opt-out (consult as needed)

---

### MVP Feature Set (Phase 1)

**Core User Journeys Supported:**

**Primary Journey - Sarah (Community Member Discovering Her DAO's Health):**
- ✅ View leaderboard with DAO's rank and score
- ✅ Understand GVS breakdown (HPR, DEI, PDI, GPI components)
- ✅ Read methodology page to understand scoring
- ✅ Share ranking on social media (screenshot-friendly design)

**Secondary Journey - Marcus (DAO Core Team Facing Reality):**
- ✅ See DAO's score and ranking (including "Concerning" label if low)
- ✅ View component breakdown showing specific weaknesses
- ✅ Click "Get Report" CTA to commission detailed analysis
- ✅ Opt-out if desired (24-hour removal guarantee)

**Tertiary Journey - Mario (Admin Managing Featured Analyses):**
- ✅ Weekly GVS calculation job runs automatically
- ✅ Admin can manually trigger recalculations
- ✅ Opt-out requests processed via email (manual for MVP)
- ✅ Monitor leaderboard performance via Vercel Analytics

**Out of Scope for MVP:**
- ❌ Alex (Researcher) API access journey - Phase 2
- ❌ Priya (Customer) self-service report ordering - manual handling for MVP
- ❌ David (Controversy) automated response system - Maya handles manually

---

### Must-Have Capabilities (MVP)

**Absolute MVP Necessities:**

**1. GVS Calculation Engine**
- ✅ Calculate all 4 components (HPR, DEI, PDI, GPI)
- ✅ Weighted formula: `GVS = (HPR × 0.35) + (DEI × 0.25) + (PDI × 0.20) + (GPI × 0.20)`
- ✅ Confidence scoring based on data completeness
- ✅ Handle missing data gracefully (skip DAOs with <50% confidence)
- **Without this:** No scores to display - product fails

**2. Leaderboard Display**
- ✅ Ranked table (Rank, DAO Name, Score, Category, Badge)
- ✅ Score labels (Vital/Stable/Concerning/Critical with color coding)
- ✅ Badge system (Featured Analysis vs Customer Report)
- ✅ Basic category filter (All/DeFi/Infrastructure/Public Goods)
- ✅ Mobile responsive design
- **Without this:** No user interface - product fails

**3. Methodology Page**
- ✅ Ranking criteria explanation (5 weighted factors)
- ✅ Data sources transparency
- ✅ Update frequency disclosure (weekly)
- ✅ Legal disclaimers (opinions, not facts)
- ✅ Opt-out policy (24-hour removal guarantee)
- **Without this:** No trust/credibility - legal liability risk

**4. Historical Tracking (Week 2+)**
- ✅ Store GVS snapshots with timestamps
- ✅ Calculate week-over-week deltas (+/- change)
- ✅ Display change indicators (↗/↘/→ with color coding)
- ⚠️ **Not needed Week 1:** Launch with static baseline, add deltas Week 2
- **Without this (Week 2+):** No virality mechanic - engagement limited

**5. Opt-Out Mechanism**
- ✅ Opt-out form page (`/rankings/opt-out`)
- ✅ Email handling workflow (24-hour removal guarantee)
- ✅ Admin interface to process removals
- ✅ Confirmation email to requester
- **Without this:** Legal/PR risk - hostage perception

---

### Can Be Manual Initially (Automation Later)

**MVP Manual Processes:**

| Process | MVP Approach (Manual) | Phase 2 Automation |
|---------|----------------------|-------------------|
| **Customer Report Orders** | Email → Manual invoice via Stripe → Manual report generation | Self-service order form with auto-payment |
| **Opt-Out Requests** | Email → Mario manually removes DAO from database | Admin dashboard with 1-click removal |
| **Featured Analysis Selection** | Mario manually selects 6 DAOs for Week 1 | Algorithm suggests high-impact DAOs to analyze |
| **Social Media Monitoring** | Maya manually searches "@ChainSights" mentions | Social listening tool with alerts |
| **Methodology Updates** | Mario manually edits methodology page | Version-controlled methodology with changelog |

**Manual-First Benefits:**
- **Faster launch:** Skip automation overhead (save 10-15 hours)
- **Learn patterns:** Understand workflows before automating
- **Flexibility:** Easy to iterate based on feedback
- **Resource efficient:** Don't automate premature processes

**When to Automate:**
- Customer reports: When >5 orders per week (Month 2-3)
- Opt-outs: When >2 requests per week (Month 3-4)
- Featured Analysis: When covering >50 DAOs (Month 4-6)

---

### Post-MVP Features (Phase 2 & Beyond)

**Phase 2: Growth Features (Month 2-3)**

**Enhanced Leaderboard Experience:**
- Historical GVS trend charts (90-day line graphs)
- Advanced filtering (by score range, change direction, multiple categories)
- Search functionality (find specific DAOs instantly)
- Component breakdown on hover (tooltip showing HPR/DEI/PDI/GPI details)
- Ticker/marquee on landing page (requires 30+ DAOs)

**Free Tier Lead Generation:**
- Free GVS quick-check (score-only lookup without full report)
- Lead capture form with marketing opt-in
- Monthly quota system (1 free check per email/wallet per month)
- "Upgrade to Full Report" upsell flow with pricing

**Customer Journey Automation:**
- Self-service order form (DAO name, email, payment via Stripe)
- Automated report generation (GVS calculation + PDF template)
- Email delivery system (send report within 3 days)
- Customer badge auto-applied to leaderboard with opt-in

**Community Feedback:**
- "Was this helpful?" feedback widget on methodology page
- User-submitted methodology improvement suggestions
- Public comments/discussion threads on rankings (moderated)

**Success Criteria for Phase 2:**
- 20-30 DAOs ranked on leaderboard
- 10+ customer reports sold per month
- 60%+ customer opt-in rate for public listing
- <5% opt-out rate (fairness validation)

---

**Phase 3: Expansion (Month 6+)**

**API & Integrations:**
- Public API endpoints for GVS queries (`/api/v1/dao/{slug}/gvs`)
- API key authentication with tiered access (Free/Pro/Enterprise)
- Webhook notifications for significant score changes
- Embeddable leaderboard widget for DAO websites

**Advanced Analytics:**
- Predictive GVS (score trajectory modeling based on trends)
- Custom weight configuration for enterprise clients (adjust HPR/DEI/PDI/GPI importance)
- Peer group benchmarking (compare against similar DAOs automatically)
- Governance health alerts (email when score drops >10 points)

**Platform Expansion:**
- Multi-chain support (Polygon, Arbitrum, Optimism, Base)
- On-chain governance support (Tally, Governor contracts, Agora)
- Multi-platform support (beyond Snapshot)
- Hybrid governance (on-chain + off-chain combined scoring)

**Self-Service DAO Dashboard:**
- DAOs claim their profile pages
- Track historical GVS trends over 12+ months
- Upload context/explanations for score changes
- Respond to community feedback publicly

**Success Criteria for Phase 3:**
- 100+ DAOs ranked across multiple chains
- API adopted by 5+ DAO tooling platforms
- 50+ active API users (researchers, analysts, platforms)
- ChainSights cited as "industry standard" for governance assessment

---

### Risk Mitigation Strategy

**Technical Risks:**

**Risk: GVS Calculation Accuracy**
- **Risk Level:** HIGH - If formula is wrong, entire product fails
- **Mitigation:**
  - Manual calibration against 20 known DAOs before launch
  - Expert review with governance researchers (validate methodology)
  - Confidence scoring (don't publish <50% confidence scores)
  - Continuous validation against qualitative reputation
- **Contingency:** If methodology rejected, pivot to "Governance Insights Platform" (remove leaderboard language, focus on narrative reports)

**Risk: Data Availability**
- **Risk Level:** MEDIUM - Snapshot API downtime or missing data could break calculations
- **Mitigation:**
  - Graceful degradation (show cached data with "Last updated" timestamp)
  - Multiple data sources (Snapshot + on-chain verification where possible)
  - Confidence indicators (High/Medium/Low based on data completeness)
  - Manual data collection fallback for high-value DAOs
- **Contingency:** If data unavailable, pause weekly updates and show "Under Maintenance" banner

**Risk: Scalability**
- **Risk Level:** LOW initially, MEDIUM at 100+ DAOs
- **Mitigation:**
  - Database indexing (rankings table indexed on score, category)
  - Caching strategy (1-hour ISR, Redis for API responses)
  - Efficient GVS calculation (batch processing, parallel fetching)
  - Weekly job timeout limits (skip DAOs that exceed 5-minute calculation)
- **Contingency:** If performance degrades, paginate leaderboard (Top 50, Rest) or add loading states

---

**Market Risks:**

**Risk: DAO Controversy Backfires**
- **Risk Level:** HIGH - Criticism could damage brand if handled poorly
- **Mitigation:**
  - 5-layer defense strategy (Transparency, Nuance, Empathy, Legal, Response Playbook)
  - Maya's response playbook (<24 hour response time)
  - Opt-out guarantee (24-hour removal prevents hostage perception)
  - Constructive framing ("governance health check," not "competition")
  - Legal disclaimers (opinions, not facts - prevent defamation liability)
- **Indicators to Watch:**
  - Opt-out rate >10% (indicates methodology rejection)
  - Coordinated criticism from multiple DAOs
  - Legal threats or defamation claims
  - Negative sentiment >30% of social mentions
- **Contingency:** If backfires, pause Featured Analyses (only Customer Reports with opt-in), add pre-approval requirement, or pivot to fully consensual model

**Risk: Market Timing**
- **Risk Level:** MEDIUM - DAO governance interest could be declining (bear market)
- **Mitigation:**
  - Launch during high-governance activity periods (avoid holidays, major crypto events)
  - Target evergreen DAOs (Gitcoin, ENS, Compound) not hype-cycle projects
  - Frame as "long-term governance health" not "trading signal"
  - Build relationships with governance-focused communities (forums, Discord)
- **Indicators to Watch:**
  - Social sharing rate <5% of ranked DAOs
  - Paid report conversion <1% of leaderboard visitors
  - Zero media interest in Week 1-2
- **Contingency:** If low engagement, pivot to B2B (sell reports to VCs/funds), add educational content (governance guides), or partner with DAO tooling platforms

**Risk: Competitive Response**
- **Risk Level:** MEDIUM - DeepDAO, Nansen could copy approach
- **Mitigation:**
  - **Speed to market:** Launch before competitors mobilize (4-week window)
  - **Brand positioning:** Own "Moody's for DAOs" narrative first
  - **Proprietary data:** Wallet clustering is technical moat (6-12 month lead time for competitors)
  - **Network effects:** More DAOs = better benchmarking = stronger moat
  - **Methodology evolution:** v2, v3 keep innovating (maintain lead)
- **Indicators to Watch:**
  - Competitor announces similar governance scoring
  - Competitor adds leaderboard feature
  - Loss of traffic to competitor platforms
- **Contingency:** If competitors launch similar features, emphasize GVS proprietary methodology, lean into editorial authority ("original" governance leaderboard), accelerate API/integrations (create lock-in)

---

**Resource Risks:**

**Risk: Solo Founder Bandwidth**
- **Risk Level:** MEDIUM - Mario is sole developer for MVP
- **Mitigation:**
  - **Manual-first approach:** Skip automation overhead (save 10-15 hours)
  - **Lean MVP scope:** 26-39 hours total build (1 week full-time)
  - **Contract specialists:** UX designer (5-10 hours), legal consultant (as needed)
  - **Automated monitoring:** Sentry, Vercel Analytics, UptimeRobot reduce manual monitoring
- **Indicators to Watch:**
  - MVP launch delayed >2 weeks from target
  - Support requests exceed 2 hours/day (unsustainable)
  - Critical bugs unresolved >48 hours
- **Contingency:** If overwhelmed, cut Phase 2 features (delay API, dashboard), hire part-time developer ($3K-5K/month), or pause Featured Analysis expansion (focus only on Customer Reports)

**Risk: Customer Report Quality**
- **Risk Level:** MEDIUM - Manual report generation could have inconsistent quality
- **Mitigation:**
  - Report template with standardized sections (GVS breakdown, benchmarking, recommendations)
  - Quality checklist before sending (verify calculations, review narrative, spell-check)
  - 3-day SLA allows time for thorough review
  - Early customer feedback loop (ask for testimonials, improvement suggestions)
- **Indicators to Watch:**
  - Customer satisfaction <80% ("Was this helpful?" feedback)
  - Refund requests >5% of orders
  - Negative reviews citing "low quality"
- **Contingency:** If quality issues, add QA reviewer (contract analyst), create detailed report writing guide, or reduce price ($99 → $79) until quality stabilizes

---

### MVP Launch Checklist

**Pre-Launch (Week 0):**
- [ ] GVS calculation engine tested with 20 DAOs (accuracy validated)
- [ ] Leaderboard page deployed to production (responsive, no bugs)
- [ ] Methodology page complete with legal disclaimers
- [ ] Opt-out form functional (tested email workflow)
- [ ] 6 Featured Analysis DAOs selected and scored
- [ ] Maya's response playbook finalized
- [ ] Vercel Analytics + Sentry monitoring active

**Week 1 Launch:**
- [ ] Leaderboard goes live (static baseline, no deltas yet)
- [ ] Social media announcement (Mario + Maya)
- [ ] Monitor opt-out requests (target: 0 in Week 1)
- [ ] Track social sharing (target: 2-5 shares)
- [ ] Watch for criticism (Maya responds <24 hours)

**Week 2 Iteration:**
- [ ] Add week-over-week change indicators (+/- deltas)
- [ ] Calculate first set of ranking changes
- [ ] Publish "Week 1 Rankings Update" blog post
- [ ] Add 5-10 more DAOs to leaderboard

**Week 3-4 Validation:**
- [ ] Analyze engagement metrics (repeat visitors, time-on-page, social shares)
- [ ] Review opt-out rate (target: <5%)
- [ ] Measure paid report conversions (target: 1-2 in first month)
- [ ] Collect methodology feedback (implement quick wins)

## Functional Requirements

This section defines the **capability contract** for the DAO Rankings Leaderboard. Every feature built must trace back to one of these requirements. UX designers will design interactions for these capabilities, architects will build systems to support them, and developers will implement them.

**Capability Organization:** Requirements are grouped by logical capability areas (what users can do), not by technology or implementation layer.

**Requirement Format:** FR#: [Actor] can [capability] [context/constraint if needed]

---

### Governance Scoring & Calculation

**Core GVS Engine Capabilities:**

- **FR1:** System can calculate Governance Vitality Score (GVS) for any DAO using the weighted formula: (HPR × 0.35) + (DEI × 0.25) + (PDI × 0.20) + (GPI × 0.20)
- **FR2:** System can calculate Human Participation Rate (HPR) component by identifying unique human participants among voting wallets
- **FR3:** System can calculate Delegate Engagement Index (DEI) component by measuring weighted delegate participation with recency factors
- **FR4:** System can calculate Power Dynamics Index (PDI) component by analyzing leadership turnover, Gini trends, and new entrant rates
- **FR5:** System can calculate Grassroots Participation Index (GPI) component by measuring small holder participation, delegation breadth, and vote diversity
- **FR6:** System can assign confidence scores to GVS calculations based on data completeness (High/Medium/Low)
- **FR7:** System can exclude DAOs from ranking when data confidence is below 50%
- **FR8:** System can fetch governance data from Snapshot.org API for proposal and voting records
- **FR9:** System can fetch token distribution data from Ethereum blockchain for on-chain verification
- **FR10:** System can store historical GVS snapshots with timestamps for week-over-week comparisons
- **FR11:** System can calculate week-over-week GVS deltas (+/- change values)
- **FR12:** System can execute automated weekly GVS recalculation job (Sunday midnight UTC)

---

### Leaderboard Display & Navigation

**Public Leaderboard Viewing:**

- **FR13:** Users can view ranked list of DAOs sorted by GVS score (highest to lowest)
- **FR14:** Users can see each DAO's rank, name, GVS score, score label, category, and badge type
- **FR15:** Users can see score labels with color coding (Vital: 80-100, Stable: 60-79, Concerning: 40-59, Critical: 0-39)
- **FR16:** Users can see badge types indicating analysis source (Featured Analysis vs Customer Report)
- **FR17:** Users can see week-over-week change indicators (+/- score change with directional arrows and color coding)
- **FR18:** Users can filter leaderboard by DAO category (All/DeFi/Infrastructure/Public Goods)
- **FR19:** Users can sort leaderboard by different columns (rank, score, change, category)
- **FR20:** Users can expand DAO entries to view component score breakdown (HPR, DEI, PDI, GPI with individual scores)
- **FR21:** Users can view "last updated" timestamp showing when GVS scores were last calculated
- **FR22:** Users can navigate to individual DAO profile pages (future enhancement - Phase 3)

**Responsive & Accessible Display:**

- **FR23:** Users can access leaderboard on mobile devices with vertical scrolling table layout
- **FR24:** Users can access leaderboard on desktop with full-width table showing all columns
- **FR25:** Users can navigate leaderboard using keyboard (tab through rows, enter to expand)
- **FR26:** Screen reader users can access leaderboard with semantic HTML structure and ARIA labels

---

### Methodology Transparency & Education

**Methodology Disclosure:**

- **FR27:** Users can view comprehensive methodology page explaining how GVS is calculated
- **FR28:** Users can see detailed ranking criteria with weights (Voter Participation 30%, Decentralization 25%, Proposal Quality 20%, Transparency 15%, Execution 10%)
- **FR29:** Users can see data sources listed (Snapshot.org API, Ethereum Blockchain, Public Governance Forums)
- **FR30:** Users can see update frequency disclosure (weekly, Sunday midnight UTC)
- **FR31:** Users can see methodology version number and last updated date
- **FR32:** Users can view methodology change log documenting version history and updates

**Legal & Policy Transparency:**

- **FR33:** Users can view legal disclaimers stating rankings are opinions, not facts, and are educational tools
- **FR34:** Users can view opt-out policy explaining 24-hour removal guarantee
- **FR35:** Users can view accessibility statement documenting WCAG 2.1 Level AA compliance
- **FR36:** Users can contact ChainSights via email for methodology feedback or accessibility issues

---

### Customer Journey Management

**Report Ordering (Manual MVP, Self-Service Phase 2):**

- **FR37:** Customers can request governance report for their DAO via email or order form
- **FR38:** Customers can opt-in to public leaderboard listing during report purchase
- **FR39:** Customers can pay for reports via Stripe payment integration
- **FR40:** Customers can receive detailed PDF governance report within 3-day SLA
- **FR41:** Customers can receive automatic leaderboard listing after report completion (if opted-in)

**Opt-Out Request Handling:**

- **FR42:** DAO representatives can submit opt-out request via web form
- **FR43:** DAO representatives can submit opt-out request via email to hello@chainsights.one
- **FR44:** System can remove DAO from leaderboard within 24 hours of opt-out request
- **FR45:** System can send confirmation email to requester when DAO is removed

**Free Tier Lead Generation (Phase 2):**

- **FR46:** Users can request free GVS quick-check (score-only lookup) for any DAO
- **FR47:** Users can opt-in to marketing communications during free score request
- **FR48:** System can enforce monthly quota limit (1 free check per email per month)
- **FR49:** Users can upgrade from free score to paid full report

---

### Administrative Operations

**Featured Analysis Management:**

- **FR50:** Admin can manually select DAOs for Featured Analysis
- **FR51:** Admin can trigger manual GVS recalculation for specific DAOs
- **FR52:** Admin can add context notes or editorial badges to Featured Analysis entries
- **FR53:** Admin can view weekly GVS calculation job logs and status
- **FR54:** Admin can monitor opt-out requests and process removals
- **FR55:** System can notify admin via Slack when weekly job fails or encounters errors

**Content & Social Management:**

- **FR56:** Admin can generate weekly ranking update content from GVS data changes
- **FR57:** Admin can monitor social media mentions of ChainSights and leaderboard
- **FR58:** Admin can access response playbook for handling criticism or controversy
- **FR59:** Admin can track engagement metrics (views, social shares, repeat visitors, time-on-page)

---

### Legal & Compliance

**Data Protection & Privacy:**

- **FR60:** System can handle user data in GDPR-compliant manner (no personal data stored beyond consented emails)
- **FR61:** System can validate and sanitize all user inputs to prevent XSS and injection attacks
- **FR62:** System can enforce HTTPS-only access to all pages
- **FR63:** System can implement rate limiting (100 requests/minute per IP) to prevent abuse

**Legal Protection:**

- **FR64:** System can display legal disclaimers on all public ranking pages
- **FR65:** System can document methodology changes with advance notice to maintain transparency
- **FR66:** System can maintain audit logs for opt-outs, score changes, and methodology updates

---

### Social & Marketing Integration

**Social Sharing & Virality:**

- **FR67:** Users can share leaderboard rankings on social media platforms (Twitter, Discord, Telegram)
- **FR68:** System can generate Open Graph tags for optimized social media previews
- **FR69:** System can generate Twitter Card metadata for leaderboard and DAO profile pages
- **FR70:** Users can screenshot leaderboard with mobile-friendly layout (no horizontal scroll needed)

**Call-to-Action & Conversion:**

- **FR71:** Users can click "Get Report" CTA buttons next to each DAO entry
- **FR72:** Users can click "Claim Your Spot" CTA for DAOs not yet ranked
- **FR73:** Users can access pricing information from leaderboard CTAs
- **FR74:** System can track CTA click-through rates and conversion metrics

---

### Performance & Reliability

**Caching & Optimization:**

- **FR75:** System can cache leaderboard data with 1-hour revalidation (Incremental Static Regeneration)
- **FR76:** System can cache API responses with 15-minute TTL to reduce database load
- **FR77:** System can serve cached data with "last updated" timestamp when live data unavailable

**Error Handling & Graceful Degradation:**

- **FR78:** System can gracefully degrade when Snapshot API is unavailable (show cached data)
- **FR79:** System can display user-friendly error messages when GVS calculation fails
- **FR80:** System can show "Under Maintenance" banner when weekly updates are paused
- **FR81:** System can track errors with source maps via Sentry for production debugging

---

### Search Engine Optimization

**Content Discovery:**

- **FR82:** System can generate sitemap.xml with all public leaderboard pages
- **FR83:** System can serve robots.txt allowing crawling of public pages while blocking admin routes
- **FR84:** System can generate structured data (Schema.org ItemList) for leaderboard entries
- **FR85:** System can set canonical URLs on all pages to prevent duplicate content indexing
- **FR86:** System can serve optimized title tags and meta descriptions for SEO

---

### Analytics & Monitoring

**Usage Tracking:**

- **FR87:** System can track Core Web Vitals (LCP, FCP, TTI, CLS, TBT) via Vercel Analytics
- **FR88:** System can track user engagement metrics (repeat visitors, time-on-page, bounce rate)
- **FR89:** System can track conversion funnel metrics (leaderboard views → CTA clicks → report purchases)
- **FR90:** System can monitor uptime with 5-minute ping intervals

**Data Quality Monitoring:**

- **FR91:** System can validate GVS calculation accuracy against manual calculations
- **FR92:** System can track data completeness for each DAO (confidence scoring)
- **FR93:** Admin can review weekly job execution time and identify performance bottlenecks

## Non-Functional Requirements

Non-functional requirements define **how well** the system must perform. These specify quality attributes that cut across all functional capabilities. This section consolidates quality targets documented throughout the PRD into measurable acceptance criteria.

**Note:** Only categories relevant to the DAO Rankings Leaderboard are included. Categories not listed (e.g., offline mode, native platform support) don't apply to this web-based product.

---

### Performance

**Page Load Performance (Public-Facing Pages):**

| Metric | Target (p95) | Acceptable (p99) | Test Method |
|--------|--------------|------------------|-------------|
| **First Contentful Paint (FCP)** | <1.5s | <2.5s | Lighthouse CI, Vercel Analytics |
| **Largest Contentful Paint (LCP)** | <2.0s | <3.0s | Core Web Vitals tracking |
| **Time to Interactive (TTI)** | <3.0s | <4.5s | Lighthouse audit |
| **Cumulative Layout Shift (CLS)** | <0.1 | <0.2 | Visual stability metric |
| **Total Blocking Time (TBT)** | <200ms | <400ms | Main thread responsiveness |

**Rationale:** As a content marketing tool, fast page loads directly impact SEO rankings, social sharing effectiveness, and user engagement. Users landing from social media expect immediate content visibility.

**Backend Performance:**

| Operation | Target | Acceptable | Failure Condition |
|-----------|--------|------------|-------------------|
| **GVS Calculation (per DAO)** | <5 minutes | <10 minutes | Timeout after 10 min |
| **Weekly Job (50 DAOs)** | <2 hours | <4 hours | Alert if exceeds 4 hours |
| **API Response Time** | <500ms (p95) | <1s (p99) | Error if >5s |
| **Database Query** | <100ms (p95) | <500ms (p99) | Timeout after 5s |

**Rationale:** Weekly job must complete within a reasonable window (Sunday midnight → Sunday 4am) to ensure Monday rankings are fresh. Long-running calculations shouldn't block user-facing pages.

**Asset Budgets:**

- **JavaScript Bundle:** <150KB gzipped (fail build if exceeded)
- **CSS:** <30KB gzipped
- **Fonts:** <50KB WOFF2 (Montserrat + Inter subsets)
- **Images:** <500KB total per page
- **API Response Payload:** <100KB per query

**Enforcement:** Next.js bundle analyzer fails CI/CD pipeline if budgets exceeded.

---

### Security

**Data Protection:**

- **Encryption in Transit:** All traffic served over HTTPS with HSTS header (max-age=31536000)
- **Encryption at Rest:** Database encryption enabled via Neon managed Postgres
- **Input Validation:** All user inputs (DAO names, emails, form fields) sanitized to prevent XSS, SQL injection, and command injection
- **Rate Limiting:** 100 requests per minute per IP address via Vercel Edge Middleware
- **Content Security Policy:** Restrict script sources to prevent code injection attacks
- **CORS:** API endpoints restricted to same-origin only (no cross-origin requests until public API in Phase 3)

**Authentication & Authorization (Admin Only - MVP):**

- **Admin Access:** Protected routes require authentication (implementation: Vercel basic auth or session-based)
- **No User Authentication:** Public leaderboard requires no login (deliberately barrier-free)

**GDPR Compliance:**

- **Data Minimization:** No personal data stored beyond consented email addresses for reports/opt-outs
- **Right to Erasure:** Opt-out mechanism guarantees 24-hour removal from public rankings
- **Data Retention:** Email addresses deleted 90 days after report delivery (unless marketing opt-in)
- **Cookie Consent:** Analytics cookies only (Vercel Analytics) with disclosure
- **Privacy Policy:** Linked from footer, explains data collection and usage

**Vulnerability Management:**

- **Dependency Updates:** Automated Dependabot alerts for npm package vulnerabilities
- **Security Headers:** X-Content-Type-Options, X-Frame-Options, X-XSS-Protection configured
- **Error Handling:** No sensitive data leaked in error messages (stack traces hidden in production)

---

### Scalability

**User Growth Scenarios:**

| Timeframe | Expected DAOs Ranked | Expected Monthly Visitors | Expected Reports/Month | System Readiness |
|-----------|---------------------|---------------------------|------------------------|------------------|
| **Week 1** | 6 | 500 | 0-2 | MVP deployment |
| **Month 1** | 15-20 | 2,000 | 10 | Current architecture sufficient |
| **Month 3** | 30+ | 5,000 | 30 | Cache optimization, database indexing |
| **Month 6** | 50-75 | 10,000 | 50 | Consider Redis, optimize weekly job |
| **Month 12** | 100+ | 25,000 | 100+ | Evaluate dedicated API server, CDN |

**Horizontal Scalability:**

- **Vercel Serverless:** Automatic horizontal scaling for Next.js pages (no manual intervention)
- **Database:** Neon Postgres serverless with automatic connection pooling (scales to demand)
- **Caching:** Redis cache added when API load exceeds 10,000 requests/day (Month 6+)

**Vertical Scalability:**

- **GVS Calculation:** Parallel processing of DAOs (calculate 10 DAOs simultaneously instead of sequential)
- **Database Queries:** Indexed on frequently queried columns (score, category, rank)
- **Asset Delivery:** Next.js Image optimization with automatic WebP conversion

**Degradation Strategy:**

If load exceeds capacity:
1. **Increase cache TTL:** 1-hour → 6-hour ISR revalidation (reduce database hits)
2. **Paginate leaderboard:** Show Top 50, then "Load More" for rest
3. **Pause weekly job:** Manual recalculation only for critical DAOs

**Capacity Alerts:**

- Alert if database connection pool >80% utilization
- Alert if API response time p95 exceeds 1 second for 5 minutes
- Alert if weekly job duration increases >50% week-over-week

---

### Accessibility

**Compliance Target: WCAG 2.1 Level AA**

**Rationale:** Web3 community values decentralization and inclusivity. Accessible design aligns with mission, reduces legal risk (EU Accessibility Act 2025), and improves SEO through semantic markup.

**Visual Accessibility:**

- **Color Contrast:** Minimum 4.5:1 for body text, 7:1 for headings (measured with WebAIM Contrast Checker)
- **Focus Indicators:** Visible 2px aqua outline on all interactive elements
- **Text Resizing:** Layout remains functional at 200% browser zoom
- **No Information from Color Alone:** Change indicators use arrows (↗/↘/→) + color

**Keyboard Accessibility:**

- **Full Keyboard Navigation:** Tab through all interactive elements in logical order
- **Skip Links:** "Skip to main content" link for screen reader users
- **Focus Management:** Focus moves to error messages on form submission failure
- **No Keyboard Traps:** Users can navigate away from all components using only keyboard

**Screen Reader Accessibility:**

- **Semantic HTML:** Proper use of `<table>`, `<thead>`, `<tbody>`, `<th scope>` for leaderboard
- **ARIA Labels:** `aria-label` on all icons and non-text controls
- **ARIA Live Regions:** Announce dynamic content changes (e.g., "Leaderboard filtered by DeFi category")
- **Alt Text:** Descriptive alt text for all images (if any)

**Mobile Accessibility:**

- **Touch Targets:** Minimum 44×44px for all buttons and links (WCAG 2.5.5)
- **Orientation:** Layout works in both portrait and landscape
- **Motion Sensitivity:** Respect `prefers-reduced-motion` media query (disable animations)

**Testing & Validation:**

- **Automated Testing:** axe-core in Playwright tests (catches 60% of issues)
- **Manual Testing:** NVDA (Windows) + VoiceOver (macOS/iOS) screen reader testing
- **Accessibility Statement:** Public page at `/accessibility` with contact for issues

---

### Reliability & Availability

**Uptime Targets:**

- **Public Pages:** 99.5% uptime (allows ~3.6 hours downtime per month)
- **Weekly Job:** 100% success rate (zero acceptable failures)
- **Admin Dashboard:** 99% uptime (acceptable - only Mario accesses)

**Data Integrity:**

- **Zero Data Loss:** Historical GVS snapshots must never be lost (critical for delta calculations)
- **Calculation Accuracy:** GVS scores must align with manual calculations >99% of the time
- **Score Consistency:** Component scores (HPR+DEI+PDI+GPI) must mathematically sum to GVS (100% accuracy)

**Error Recovery:**

- **Snapshot API Failure:** Serve cached data with "Last updated [timestamp]" notice
- **Database Failure:** Display maintenance page within 30 seconds, alert admin immediately
- **Weekly Job Failure:** Retry once automatically, then Slack alert to admin if second failure
- **Payment Failure:** Customer notified via email, manual invoice fallback

**Monitoring & Alerting:**

- **Uptime Monitoring:** UptimeRobot pings `/rankings` every 5 minutes
- **Error Tracking:** Sentry captures and alerts on production errors (>10 errors/hour = immediate alert)
- **Weekly Job:** Cron job logs success/failure, Slack notification on failure
- **Performance Degradation:** Alert if LCP increases >20% week-over-week

**Backup & Recovery:**

- **Database Backups:** Neon automatic daily backups (30-day retention)
- **Code Versioning:** Git history allows rollback to any previous deploy
- **Deployment Rollback:** Vercel 1-click rollback to previous stable deployment

---

### Maintainability

**Code Quality:**

- **TypeScript:** 100% TypeScript (no JavaScript files except config)
- **Type Safety:** `strict: true` in tsconfig.json, zero `any` types in core logic
- **Linting:** ESLint with Next.js recommended rules, zero warnings in production build
- **Code Coverage:** Minimum 60% test coverage for GVS calculation engine (critical logic)

**Documentation:**

- **README.md:** Setup instructions, architecture overview, deployment guide
- **Inline Comments:** Complex GVS calculations documented with formula explanations
- **API Documentation:** JSDocs for all public functions and components
- **ADRs (Architecture Decision Records):** Document major technical decisions (e.g., why Neon over RDS)

**Dependency Management:**

- **Regular Updates:** npm audit run weekly, critical vulnerabilities patched within 7 days
- **Dependency Pinning:** Exact versions in package.json (no `^` or `~` for production dependencies)
- **Deprecation Monitoring:** Track deprecated npm packages, plan migration before EOL

**Refactoring Safety:**

- **Test Suite:** Unit tests for GVS calculation, integration tests for API routes
- **Visual Regression:** Percy.io snapshots for leaderboard page (detect unintended UI changes)
- **Deployment Preview:** Every PR gets Vercel preview URL for manual QA before merge

---

### Integration

**External System Dependencies:**

| System | Purpose | SLA Requirement | Fallback Strategy |
|--------|---------|-----------------|-------------------|
| **Snapshot.org API** | Proposal and vote data | 99% uptime | Serve cached data, show staleness indicator |
| **Ethereum RPC** | Token distribution data | 95% uptime | Skip on-chain verification if unavailable |
| **Stripe API** | Payment processing | 99.9% uptime | Manual invoice fallback |
| **Vercel Platform** | Hosting and deployment | 99.9% uptime | No fallback (dependency) |
| **Neon Postgres** | Database | 99.95% uptime | No fallback (dependency) |

**API Reliability:**

- **Retry Logic:** 3 retries with exponential backoff for transient failures
- **Timeout Handling:** 30-second timeout for Snapshot API calls, graceful failure message
- **Circuit Breaker:** After 5 consecutive Snapshot API failures, skip external calls for 5 minutes

**Data Format Compatibility:**

- **Snapshot API:** Handle schema changes gracefully (ignore unknown fields, validate required fields)
- **Stripe Webhooks:** Verify webhook signatures to prevent spoofing
- **CSV Export (Future):** UTF-8 encoding, RFC 4180 compliant

---

### SEO & Content Discoverability

**Search Engine Optimization:**

- **Crawlability:** All public pages crawlable by search engines (robots.txt allows)
- **Indexability:** No `noindex` tags on leaderboard or methodology pages
- **Sitemap.xml:** Generated automatically, includes all public DAO ranking pages
- **Structured Data:** Schema.org ItemList markup for leaderboard (rich snippets eligibility)
- **Page Speed:** Core Web Vitals in "Good" range (Google ranking factor)

**Social Media Optimization:**

- **Open Graph Tags:** og:title, og:description, og:image for all public pages
- **Twitter Cards:** summary_large_image with leaderboard preview
- **Shareable URLs:** Clean, readable URLs (`/rankings`, `/rankings/methodology`)

**Content Freshness:**

- **Weekly Updates:** New content signal for search engines (GVS recalculations)
- **Last Modified:** `lastmod` in sitemap.xml reflects actual content updates
- **Canonical URLs:** Prevent duplicate content penalties

**Target:** 1,000+ organic visits per month by Month 6 (baseline: 0 at launch)
