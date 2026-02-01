# DAO Rankings Leaderboard - Feature Documentation

**Created:** December 21, 2025
**Status:** Approved for Implementation
**Team Decision:** Option 3 (Hybrid CTA)

---

## Table of Contents

1. [Overview](#overview)
2. [Original Question](#original-question)
3. [Team Analysis](#team-analysis)
4. [Decision: Option 3](#decision-option-3)
5. [Confrontation Defense Strategy](#confrontation-defense-strategy)
6. [Implementation Requirements](#implementation-requirements)
7. [Social Media Response Playbook](#social-media-response-playbook)
8. [Launch Strategy](#launch-strategy)

---

## Overview

Mario proposed creating a public DAO leaderboard displaying governance scores in a "stock market" style with:
- DAO names
- Scores (x/10)
- **Change indicators** (+0.3 / -0.5 / unchanged)
- **Call-to-action buttons** next to each entry

**Concept:** CoinMarketCap for DAO Governance

---

## Original Question

> "What's wrong with displaying our key figure x/10 as a kind of stock market list with +/- and a CTA next to it? Don't worry, I don't want to turn everything upside down, I'm just asking for a friend. 😉😅"

**Team Assessment:** Nothing wrong with it - it's actually BRILLIANT for virality and engagement.

---

## Team Analysis

### Maya (Marketing) - ✅ HELL YES

**Why This Works:**

1. **Social Proof Through Competition**
   - "Uniswap is ranked #3? Let me check my DAO..."
   - "We dropped from #5 to #8?! We need to improve!"
   - FOMO intensifies with every rank change

2. **Virality Mechanics**
   - DAOs share when they rank high
   - DAOs share when they improve (+0.5!)
   - Community members tag DAOs: "@DAO you're ranked #12, step it up!"

3. **The +/- Hook**
   - Static scores = boring
   - **Movement = story = virality**
   - "Breaking: Arbitrum jumps 3 spots after implementing quadratic voting"

4. **CTA Effectiveness**
   - Context: "Your DAO is #8"
   - CTA: "Get Full Report" ← natural next step
   - **Conversion rate will be 5-10x higher** than cold traffic

### Sally (Solutions Architect) - ✅ YES

**Technical Requirements:**
- Historical data tracking (scores over time)
- Refresh cadence (weekly recommended)
- Ranking algorithm (weighted criteria)
- Caching layer (Redis or Next.js cache)

**Data Availability:**
- ✅ Score data exists in `freeQueryLog`
- ✅ Timestamp tracking
- ❌ No historical tracking yet (need to add)
- ❌ No delta calculation logic (need to build)

**Effort Estimate:** 4-6 hours for proper implementation

### Winston (Tech Lead) - ✅ YES

**Build Complexity Options:**

**Easy Version (Static Snapshot):**
- Show current scores
- Manual update weekly
- No delta tracking
- **2 hours build time**

**Medium Version (Semi-Automated):**
- Scheduled job recalculates nightly
- Stores historical snapshots
- Auto-calculates deltas
- **6 hours build time**

**Full Version (Real-Time):**
- Live score updates
- Trend charts (7-day, 30-day)
- Category breakdowns
- Social sharing buttons
- **20+ hours build time**

### Paige (UX Designer) - ⚠️ YES WITH CAVEATS

**What Could Go Wrong:**

1. **Not Enough DAOs Yet**
   - Leaderboard with 3 DAOs looks pathetic
   - **Wait until 15-20 DAOs minimum**

2. **Controversial Rankings**
   - DAOs might get pissed: "Your algo is bullshit!"
   - Need clear methodology disclosure (✅ DONE)

3. **Stale Data Problem**
   - If scores don't update frequently, +/- loses meaning
   - Need weekly updates minimum

4. **Gaming the System**
   - DAOs will optimize for metrics
   - Not necessarily bad, but design for it

### Mary (Business Analyst) - ✅ YES

**Strategic Positioning:**
- Frame as "Moody's for DAOs"
- Constructive governance assessment
- Ecosystem improvement tool

**Prerequisites:**
- 15-20 DAOs with scores
- Weekly update commitment
- Clear opt-out policy
- Methodology transparency

---

## Decision: Option 3 (Hybrid CTA)

**Mario's Choice:** Option 3 - "Claim Your Spot"

### CTA Strategy

**Option 3: "Claim Your Spot" (Hybrid)**
- Free if you opt-in to public listing
- Paid for private detailed report
- **This maximizes both lead gen AND revenue**

**Why This Wins:**
- Low friction for lead generation
- Revenue opportunity for detailed analysis
- Turns controversy into sales ("let us tell your full story")

---

## Confrontation Defense Strategy

### Layer 1: Transparency

**Methodology Page:** `/rankings/methodology`
- Full criteria disclosure (5 categories with weights)
- Data sources (Snapshot, blockchain, forums)
- Update frequency (weekly)
- Clear opt-out policy
- Limitations & disclaimers

**Status:** ✅ CREATED

### Layer 2: Nuance

Show more than just a number:
- **Confidence intervals:** "8.5/10 (±0.3)"
- **Category breakdown:**
  - Participation: 9/10 ✅ Strong
  - Decentralization: 7/10 ⚠️ Room for improvement
  - Transparency: 8/10 ✅ Good
  - Execution: 6/10 ⚠️ Could be better
- **Peer comparison:** "Your DAO: 7.2/10, Similar DAOs (DeFi): Avg 7.8/10"

### Layer 3: Empathy

**Constructive Framing:**
- ❌ Don't say: "Ranked #1", "Best DAO", "Failed governance"
- ✅ Do say: "Governance Health Score", "Strong performance", "Opportunities for improvement"

**Tone:**
- Constructive, not judgmental
- Data-driven, not opinion-based
- Improvement-focused, not punitive

### Layer 4: Legal Protection

**Disclaimers:**
```
DISCLAIMER: Rankings are educational tools based on publicly
available data and should not be considered financial, legal,
or investment advice. Scores reflect quantitative metrics and
may not capture the full complexity of governance structures.
Rankings are opinions, not statements of fact. We make no
guarantees about accuracy or completeness.

By appearing on this list, DAOs have either (1) explicitly
opted-in, or (2) are included based on public data analysis.
Any DAO can request removal at any time.
```

**Why This Protects:**
- ✅ "Opinions" = protected speech
- ✅ "Educational tool" = not commercial slander
- ✅ "Opt-out available" = consent-friendly
- ✅ "Public data" = defensible source

### Layer 5: Response Playbook

**Maya handles escalations** with pre-written templates.

---

## Social Media Response Playbook

### Scenario 1: "Your ranking is bullshit!"

**Response Template:**
```
Hey! Thanks for the feedback. Our ranking is ONE perspective
based on public data. We know it's not perfect - governance
is complex. What metrics do you think we're missing? We're
constantly refining our approach. Also, if you don't want to
be listed, we'll remove you - no hard feelings!
```

**Key Elements:**
- ✅ Acknowledge imperfection
- ✅ Ask for their input
- ✅ Remind them of opt-out
- ✅ Stay friendly, not defensive

### Scenario 2: "You're hurting our reputation!"

**Response Template:**
```
We hear you. That's not our intent - we're trying to help
the ecosystem improve. A few things: (1) Your score is based
on public data anyone can verify, (2) We show the methodology
so people understand what we're measuring, (3) We highlight
what you're doing WELL, not just weaknesses, (4) You can
always opt-out. Would you be open to a call to discuss how
we can make this more constructive?
```

**Key Elements:**
- ✅ Empathy first
- ✅ Point to transparency
- ✅ Emphasize improvement, not shame
- ✅ Offer dialogue
- ✅ Easy opt-out

### Scenario 3: "Your weights are wrong!"

**Response Template:**
```
That's a fair point! Different people value different things.
Our weights reflect what we currently believe drives healthy
governance, but we're open to adjusting as we learn. Here's
why we weighted it this way: [reasoning]. What would you
weight differently and why? We publish methodology updates
every quarter based on community feedback.
```

**Key Elements:**
- ✅ Validate their perspective
- ✅ Explain your reasoning
- ✅ Show willingness to evolve
- ✅ Invite constructive input

### Scenario 4: "You don't understand our governance model!"

**Response Template (with sales pivot):**
```
You're probably right! Every DAO is unique, and quantitative
metrics can miss important context. That's exactly why we
offer detailed reports (€49) - they include qualitative
analysis and nuance. The public ranking is a high-level
snapshot. Want to do a deep-dive report? We'll tell your
full story, not just a number.
```

**Key Elements:**
- ✅ Acknowledge uniqueness
- ✅ **Pivot to paid offering** 🎯
- ✅ Position report as "telling their story"
- ✅ Turn criticism into sales opportunity

---

## Implementation Requirements

### Phase 1: Pre-Launch (Before Publishing Rankings)

**Priority:** HIGH

1. ✅ **Methodology Page**
   - Location: `/rankings/methodology`
   - Status: CREATED

2. ⏳ **Opt-Out Form**
   - Location: `/rankings/opt-out`
   - Status: PENDING

3. ⏳ **Legal Disclaimers**
   - Add to ranking page header
   - Status: PENDING

4. ⏳ **Social Media Response Templates**
   - Document in internal wiki
   - Train Maya on response protocol
   - Status: PENDING

### Phase 2: Build Rankings Page

**Technical Requirements:**

1. **Database Schema Changes**
   - Add `ranking_snapshots` table:
     ```sql
     CREATE TABLE ranking_snapshots (
       id UUID PRIMARY KEY,
       dao_id TEXT,
       score DECIMAL(3,1),
       rank INTEGER,
       category_scores JSONB,
       snapshot_date DATE,
       created_at TIMESTAMP
     );
     ```

2. **Ranking Calculation Logic**
   - Weighted scoring algorithm (see methodology)
   - Category breakdown computation
   - Delta calculation (current vs previous week)

3. **Frontend Components**
   - Leaderboard table with sorting
   - Change indicators (+/-/→)
   - Category breakdown tooltips
   - CTA buttons (Claim Your Spot)

4. **Update Job**
   - Cron job: Every Sunday midnight UTC
   - Fetch latest data from Snapshot
   - Calculate scores
   - Store snapshot
   - Update rankings

**Build Complexity:** Start with Easy Version (2 hours)

### Phase 3: Launch Strategy

**Pre-Launch Outreach:**

1. **Contact Top 5 Ranked DAOs Personally**
   - Email template:
     ```
     Hey [DAO name],

     We're launching a DAO governance leaderboard next week,
     and you ranked highly (#3 out of 20)! Wanted to give
     you a heads up before we publish.

     Your score: 8.5/10
     - Participation: 9/10 ✅
     - Decentralization: 8/10 ✅
     - Transparency: 9/10 ✅
     - Proposal Quality: 8/10 ✅
     - Execution: 7/10 ⚠️

     Full methodology: [link]

     If you'd rather not be listed, just let us know and
     we'll remove you - no questions asked. Otherwise, we'll
     publish next Monday.

     Thoughts?
     ```

   - **Goal:** Turn them into advocates
   - Give them chance to opt-out preemptively

2. **Publish Methodology First, Rankings Second**
   - Day 1: Tweet about methodology page
   - Day 2-3: Build anticipation ("ranking 20 DAOs...")
   - Day 4: Launch rankings

3. **Emphasize "Beta" Label**
   - "v1.0 - we're learning!"
   - Invite feedback openly

**Post-Launch Monitoring:**

1. Monitor Twitter/Discord for mentions
2. Respond within 24 hours to criticism
3. Document all feedback for methodology v2
4. Quarterly transparency reports

---

## Ranking Criteria (Detailed)

### 1. Voter Participation (30%)

**Metrics:**
- % of token holders who voted (last 10 proposals)
- Average voter turnout per proposal
- Trend over time (improving/declining)

**Scoring:**
- 10/10: >40% turnout
- 7-9/10: 20-40% turnout
- 4-6/10: 10-20% turnout
- 0-3/10: <10% turnout

### 2. Decentralization (25%)

**Metrics:**
- Gini coefficient (0 = perfect equality, 1 = total inequality)
- Nakamoto coefficient (# entities needed to control 51%)
- Top 10 holders % of total supply

**Scoring:**
- 10/10: Gini <0.5, Nakamoto >100
- 7-9/10: Gini 0.5-0.7, Nakamoto 50-100
- 4-6/10: Gini 0.7-0.85, Nakamoto 20-50
- 0-3/10: Gini >0.85, Nakamoto <20

### 3. Proposal Quality (20%)

**Metrics:**
- Pass/reject ratio (too high = rubber-stamping)
- Average discussion comments per proposal
- Proposal quality score (AI analysis of clarity)

**Scoring:**
- 10/10: 60-80% pass rate, >50 comments/proposal
- 7-9/10: 80-90% pass rate, 20-50 comments
- 4-6/10: >90% pass rate, 10-20 comments
- 0-3/10: >95% pass rate, <10 comments

### 4. Transparency (15%)

**Metrics:**
- On-chain voting %
- Public discussion before vote
- Governance documentation quality

**Scoring:**
- 10/10: 100% on-chain, public forums active
- 7-9/10: >80% on-chain, some public discussion
- 4-6/10: 50-80% on-chain, minimal discussion
- 0-3/10: <50% on-chain, no public discussion

### 5. Execution (10%)

**Metrics:**
- % of passed proposals implemented
- Time from approval to implementation
- Treasury execution efficiency

**Scoring:**
- 10/10: >90% implemented, <30 days average
- 7-9/10: 70-90% implemented, 30-60 days
- 4-6/10: 50-70% implemented, 60-90 days
- 0-3/10: <50% implemented, >90 days

---

## UI Mockup

### Leaderboard Layout

```
┌─────────────────────────────────────────────────────────────┐
│  🏆 DAO Governance Leaderboard                              │
│  Updated: Dec 21, 2025 • Next update: Dec 28, 2025         │
│  [Methodology] [How Rankings Work]                          │
├──────┬───────────────┬─────────┬─────────┬──────────────────┤
│ Rank │ DAO           │ Score   │ Change  │ Action           │
├──────┼───────────────┼─────────┼─────────┼──────────────────┤
│  #1  │ Uniswap       │ 9.2/10  │ +0.3 ↗  │ [Get Report]     │
│      │               │         │ (green) │ [Claim Spot]     │
│      │               ├─────────┴─────────┴──────────────────┤
│      │               │ Participation: 9/10 • Decent: 10/10  │
├──────┼───────────────┼─────────┬─────────┬──────────────────┤
│  #2  │ Aave          │ 8.8/10  │ -0.1 ↘  │ [Get Report]     │
│      │               │         │ (red)   │ [Claim Spot]     │
│      │               ├─────────┴─────────┴──────────────────┤
│      │               │ Participation: 8/10 • Decent: 9/10   │
├──────┼───────────────┼─────────┬─────────┬──────────────────┤
│  #3  │ Compound      │ 8.5/10  │ +0.0 →  │ [Get Report]     │
│      │               │         │ (gray)  │ [Claim Spot]     │
└──────┴───────────────┴─────────┴─────────┴──────────────────┘

📊 Don't see your DAO? [Get Free Score] (Lead Gen CTA)

⚠️ DISCLAIMER: Rankings are educational tools based on public
data. Not financial advice. [Full Methodology]
```

### Color Coding

- **Green (+0.3 ↗):** Improved since last week
- **Red (-0.1 ↘):** Declined since last week
- **Gray (+0.0 →):** No change

---

## Success Metrics

### Week 1 Targets

- 15-20 DAOs ranked
- 5+ social media shares from DAO communities
- 0 opt-out requests (good sign of acceptance)
- 10+ clicks on "Get Report" CTAs

### Month 1 Targets

- 50+ DAOs ranked
- 20% marketing opt-in rate on free scores
- 5+ paid report purchases from leaderboard traffic
- 1-2 methodology improvement suggestions implemented

### Month 3 Targets

- 100+ DAOs ranked
- Featured in crypto Twitter discussions
- Methodology v2 published with community input
- 10+ paid reports/month from leaderboard

---

## Risk Mitigation

### Risk 1: DAO Backlash

**Mitigation:**
- ✅ Clear methodology page
- ✅ Easy opt-out
- ✅ Maya handles confrontations
- ✅ Pre-launch outreach to top DAOs

### Risk 2: Data Staleness

**Mitigation:**
- Weekly automated updates
- Show last update timestamp
- "Beta" label to set expectations

### Risk 3: Gaming the System

**Mitigation:**
- AI-powered quality checks
- Manual review for suspicious spikes
- Quarterly methodology adjustments

### Risk 4: Legal Liability

**Mitigation:**
- ✅ Disclaimers: "Opinions, not facts"
- ✅ "Educational tool" framing
- ✅ Public data only
- ✅ Opt-out policy

---

## Team Consensus

**All team members approved this feature:**

- **Mary (BA):** ✅ Strategic value confirmed
- **Sally (Architect):** ✅ Technically feasible
- **Winston (Tech Lead):** ✅ Build complexity manageable
- **Paige (UX):** ✅ With proper disclaimers
- **Maya (Marketing):** ✅ Viral growth engine

**Mario's Decision:** Approved - Option 3 (Hybrid CTA)

**Next Steps:**
1. Build opt-out form
2. Implement Easy Version (2 hours)
3. Wait for 15-20 DAOs with scores
4. Execute launch strategy

---

## Maya's Promise

> "I got your back, Mario. When the first angry DAO tweets at you, forward it to me. I'll handle it. This is literally what I do. 😎"

---

**Document Status:** Complete
**Last Updated:** December 21, 2025
**Next Review:** After first 10 DAOs ranked
