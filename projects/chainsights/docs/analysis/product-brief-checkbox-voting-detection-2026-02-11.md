---
stepsCompleted: [1, 2, 3, 4, 5]
inputDocuments:
  - 'Lido Forum Thread: BCV feedback on checkbox voting'
  - 'Product Brief: Governance Attention Map (category data foundation)'
  - 'Product Brief: Wallet Label Filtering (pipeline context)'
  - 'Engagement Hub Ticket: Discourse/Forum Scanner (Phase 3b)'
  - 'Snapshot API: Vote timestamps, choices, proposals'
workflowType: 'product-brief'
lastStep: 5
project_name: 'chainsights'
user_name: 'Mario'
date: '2026-02-11'
feature_context: 'Checkbox Voting Detection — Vote Quality Analytics'
community_validated: true
validation_source: 'Lido DAO Forum (BCV, governance contributor)'
---

# Product Brief: Checkbox Voting Detection

**Date:** 2026-02-11
**Author:** Mario

---

## Community Validation Context

This feature was shaped by **BCV** in the Lido Forum thread "Governance Health: Perfect Voter Quality, Room for Participation" (Feb 2026).

BCV raised the question of **distinguishing informed participation from delegates voting on everything regardless of expertise**. This is a fundamental governance quality problem: high participation rates can mask low-quality engagement.

Our response in the forum: *"Checkbox voting detection — distinguishing informed participation from delegates voting on everything regardless of expertise is on our roadmap. Your observation shaped how we think about vote quality vs. quantity."*

---

## Executive Summary

**Checkbox Voting Detection** adds vote quality analysis to ChainSights, identifying voters who participate broadly but shallowly — "checking boxes" rather than making informed governance decisions. The feature rolls out in three phases: improving GPI scoring (invisible backend), adding per-delegate quality scores (visible UI), and eventually introducing a new GVS dimension (Vote Quality Index).

**Core Insight:** A delegate who votes "For" on every proposal in 30 seconds isn't governing — they're performing participation theater. Current metrics reward this behavior. Checkbox Voting Detection corrects that.

**Tagline:** "Participation is easy. Governance is hard. We measure the difference."

**Positioning:** No other DAO analytics platform distinguishes vote quality from vote quantity. DeepDAO counts participation rates. Tally shows vote history. ChainSights shows whether votes are actually informed. This deepens the "identity-first analytics" moat — not just who votes, but how well they vote.

---

## Core Vision

### Problem Statement

Current DAO governance metrics treat all votes equally. A delegate who:
- Reads the proposal, participates in forum discussion, votes after deliberation
- Blindly votes "For" on everything in a 2-minute session

...both get the **same participation score**. In fact, the checkbox voter often scores **higher** because they never miss a vote.

This creates perverse incentives:
- Delegates optimise for participation rate, not governance quality
- DAOs that reward participation inadvertently reward checkbox behavior
- Governance health scores inflate because "everyone votes" even if nobody thinks

### The Five Signals

We identified five measurable signals that distinguish informed voting from checkbox behavior:

| # | Signal | Description | Data Source | Complexity |
|---|--------|-------------|-------------|-----------|
| 1 | **Voting Speed** | Time between consecutive votes within a session. Voting on 5 proposals in 2 minutes = checkbox | Snapshot API (vote timestamps) | Low |
| 2 | **Vote Diversity** | Ratio of For/Against/Abstain across all votes. 100% "For" across 50 proposals = suspicious | Snapshot API (vote choices) | Low |
| 3 | **Cross-Category Voting** | Does someone vote on topics outside their expertise? A validator expert voting on treasury proposals | Attention Map categories (already live) | Medium |
| 4 | **Pattern Correlation** | Does someone always vote the same as the top 3 whales? "Follow the leader" behavior | Snapshot API (all votes) | Medium |
| 5 | **Forum Participation** | Did someone comment on the forum before voting? | Discourse API (Engagement Hub Phase 3b) | Medium* |

*Signal 5 drops from "High" to "Medium" complexity because the Discourse/Forum Scanner is already planned in the Engagement Hub (Phase 3b). We can reuse that infrastructure.

### Why Now

1. **Attention Map is live** — category data is the foundation for Signal 3 (Cross-Category Voting)
2. **BCV's feedback creates accountability** — we told the Lido community this is on our roadmap
3. **Wallet Label Filtering showed minimal impact** — Checkbox Voting Detection is the next data quality step with potentially much higher impact
4. **Engagement Hub Discourse integration is planned** — Signal 5 infrastructure coming anyway
5. **No competitor does this** — first mover advantage in vote quality analytics

---

## Three-Phase Rollout

### Phase 1: GPI Enhancement (Backend — Invisible to Users)

**Goal:** Improve the Grassroots Participation Index to penalise checkbox voting behavior instead of rewarding it.

**Scope:** Signals 1 (Voting Speed) + 2 (Vote Diversity)

**How it works:**
- Calculate a **Checkbox Score (0-1)** per voter using Signals 1+2
- 0 = clearly informed voter (varied votes, deliberate timing)
- 1 = clearly checkbox voter (all "For", rapid-fire timing)
- Weight the voter's contribution to GPI by `(1 - checkbox_score)`
- Checkbox voters' participation counts less toward GPI

**What changes for users:** GPI scores shift — DAOs with many checkbox voters drop, DAOs with quality participation rise. No new UI, no new score label. Just better numbers.

**Data needed:**
```
Per voter:
- vote_timestamps[]     → time deltas between consecutive votes
- vote_choices[]        → For/Against/Abstain distribution
- proposals_voted_on[]  → total participation breadth
```

**Checkbox Score Formula (v1):**

```
speed_signal = count(votes with delta < 60s) / total_votes
diversity_signal = max_choice_ratio  // e.g., 48/50 "For" = 0.96
checkbox_score = (speed_signal * 0.5) + (diversity_signal * 0.5)
```

Thresholds:
- `checkbox_score > 0.8` → strong checkbox behavior, weight = 0.2
- `checkbox_score 0.5-0.8` → moderate, weight = 0.6
- `checkbox_score < 0.5` → informed voter, weight = 1.0

**Effort:** ~8-10h
**Dependencies:** None (all data available from Snapshot API)
**Risk:** GPI scores change — need methodology marker if impact is significant

---

### Phase 2: Delegate Scorecard (Visible Feature)

**Goal:** Show per-delegate vote quality scores on the Matrix Details page. This is the feature that BCV asked for.

**Scope:** Signals 1-4 (Speed, Diversity, Cross-Category, Pattern Correlation)

**How it works:**
- Each delegate/top voter gets a **Vote Quality Score (VQS)** on their profile
- VQS is composed of the four signal scores, weighted and normalised to 0-10
- Displayed on the Matrix Details page under the "Key Governance Participants" section
- Included in the Governance Audit PDF report (€149 tier)

**Vote Quality Score Components:**

| Component | Weight | Measures |
|-----------|--------|----------|
| **Deliberation** (Signal 1) | 25% | Time spent before voting — penalises rapid-fire sessions |
| **Independence** (Signal 2) | 25% | Vote diversity — penalises always voting the same way |
| **Focus** (Signal 3) | 25% | Category concentration — rewards domain expertise, penalises voting on everything |
| **Originality** (Signal 4) | 25% | Pattern independence — penalises "follow the whale" behavior |

**UI Concept — Matrix Details Page:**

```
Key Governance Participants

Address          Type       Voting Power   Participation   Vote Quality
0x4af8...1F6A   WHALE      12,818,472     87%            ██████░░░░ 6.2
0x7eE0...bb6C   DELEGATE    5,016,798     94%            ████░░░░░░ 4.1 ⚠️
0x55Bc...f0fA   TOP VOTER   3,475,103     23%            █████████░ 8.7
```

The ⚠️ flag appears for high-participation but low-quality voters — exactly BCV's checkbox voting concern.

**UI Concept — Delegate Detail (click to expand):**

```
0x7eE0...bb6C — DELEGATE — Vote Quality: 4.1/10

  Deliberation:  ██░░░░░░░░  2.1  "Votes within 30s of each other"
  Independence:  ███░░░░░░░  3.4  "92% For votes across all proposals"
  Focus:         ██████░░░░  6.2  "Votes across 6 of 8 categories"
  Originality:   ████░░░░░░  4.5  "78% correlation with top whale"
  
  Pattern: This delegate shows signs of checkbox voting — 
  high participation (94%) but low quality signals.
```

**Effort:** ~20-25h
**Dependencies:** Phase 1 (checkbox score calculation), Attention Map categories
**Revenue Impact:** Differentiator for €149 Governance Audit — "see not just who votes, but how well"

---

### Phase 3: Vote Quality Index (New GVS Dimension)

**Goal:** Introduce VQI as a fifth dimension of the Governance Vitality Score, alongside HPR, DEI, PDI, and GPI.

**Scope:** All 5 Signals including Forum Participation

**How it works:**
- VQI aggregates the Vote Quality Scores of all voters in a DAO, weighted by voting power
- A DAO where high-power voters are all checkbox voters gets a low VQI
- A DAO where power holders show diverse, deliberate voting patterns gets a high VQI
- VQI becomes 15-20% of GVS, with other dimensions rebalanced

**New GVS Composition:**

```
Current:
GVS = HPR (35%) + DEI (25%) + PDI (20%) + GPI (20%)

Phase 3:
GVS = HPR (30%) + DEI (20%) + PDI (15%) + GPI (15%) + VQI (20%)
```

**Forum Participation Signal:**
- Leverage the Discourse/Forum Scanner from Engagement Hub Phase 3b
- For each voter: check if they posted/replied in the relevant forum thread before voting
- Forum participation before vote → positive signal
- Vote without any forum activity → neutral (not negative — lurking is valid)
- This is the strongest quality signal but requires forum data per DAO

**Effort:** ~15-20h (assuming Engagement Hub Phase 3b is done)
**Dependencies:** Phase 2, Engagement Hub Discourse integration
**Risk:** Changing GVS composition affects ALL rankings. Requires:
  - Communication: blog post "Why Your DAO's Score Changed"
  - Methodology versioning (v2.0)
  - Possibly a transition period showing both old and new GVS

---

## Technical Architecture

### Data Model

```sql
-- Per-voter checkbox signals (calculated during GVS job)
CREATE TABLE voter_quality_signals (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  space_id VARCHAR(100) NOT NULL,
  voter_address VARCHAR(42) NOT NULL,
  calculation_date DATE NOT NULL,
  
  -- Signal 1: Voting Speed
  avg_vote_delta_seconds FLOAT,        -- avg time between consecutive votes
  rapid_vote_count INT,                -- votes with delta < 60s
  total_session_votes INT,             -- votes in same session
  speed_score FLOAT NOT NULL,          -- 0-10 (10 = deliberate)
  
  -- Signal 2: Vote Diversity
  for_ratio FLOAT,                     -- % of For votes
  against_ratio FLOAT,                 -- % of Against votes
  abstain_ratio FLOAT,                 -- % of Abstain votes
  diversity_score FLOAT NOT NULL,      -- 0-10 (10 = diverse)
  
  -- Signal 3: Cross-Category Voting
  categories_voted_on INT,             -- out of total categories
  primary_category VARCHAR(100),       -- most voted category
  category_concentration FLOAT,        -- % in primary category
  focus_score FLOAT NOT NULL,          -- 0-10 (10 = focused)
  
  -- Signal 4: Pattern Correlation
  whale_correlation FLOAT,             -- correlation with top 3 whales
  unique_dissents INT,                 -- times voted differently from majority
  originality_score FLOAT NOT NULL,    -- 0-10 (10 = independent)
  
  -- Signal 5: Forum Participation (Phase 3)
  forum_posts_before_vote INT,
  forum_score FLOAT,                   -- 0-10 (10 = active forum participant)
  
  -- Aggregate
  vote_quality_score FLOAT NOT NULL,   -- weighted composite 0-10
  checkbox_flag BOOLEAN DEFAULT false, -- true if VQS < 3.0
  
  created_at TIMESTAMP NOT NULL DEFAULT NOW(),
  UNIQUE(space_id, voter_address, calculation_date)
);

CREATE INDEX idx_vqs_space_date ON voter_quality_signals(space_id, calculation_date);
CREATE INDEX idx_vqs_checkbox ON voter_quality_signals(checkbox_flag) WHERE checkbox_flag = true;
```

### Pipeline Integration

```
CURRENT GVS PIPELINE:
1. Fetch proposals + votes (cached)
2. Label enrichment (wallet filtering)
3. Classify voters (WHALE/DELEGATE/TOP VOTER)
4. Calculate HPR, DEI, PDI, GPI
5. Aggregate GVS

NEW PIPELINE (Phase 1):
1. Fetch proposals + votes (cached)
2. Label enrichment (wallet filtering)
3. Classify voters
3.5 NEW: Calculate voter_quality_signals per voter
    a. Compute speed_score from vote timestamps
    b. Compute diversity_score from vote choices
    c. Store in voter_quality_signals table
4. Calculate HPR, DEI, PDI, GPI — with GPI using checkbox weights
5. Aggregate GVS

NEW PIPELINE (Phase 2 — adds to 3.5):
    c. Compute focus_score from Attention Map categories
    d. Compute originality_score from vote correlation analysis
    e. Compute vote_quality_score composite
    f. Flag checkbox voters (VQS < 3.0)

NEW PIPELINE (Phase 3 — adds to 3.5):
    g. Check forum participation via Discourse API
    h. Compute forum_score
    i. Calculate VQI dimension from aggregate voter quality
5. Calculate HPR, DEI, PDI, GPI, VQI
6. Aggregate GVS (new weights)
```

### Attention Map Integration (Signal 3)

The Governance Attention Map already classifies proposals into categories (Operations, Protocol Upgrades, Grants, etc.). For Cross-Category Voting:

```typescript
// For each voter, check how many categories they vote in
const voterCategories = votes.reduce((acc, vote) => {
  const category = proposalCategoryMap[vote.proposalId];
  acc[category] = (acc[category] || 0) + 1;
  return acc;
}, {});

const totalCategories = Object.keys(voterCategories).length;
const maxCategoryVotes = Math.max(...Object.values(voterCategories));
const concentration = maxCategoryVotes / totalVotes;

// High concentration = focused voter (good)
// Low concentration = votes on everything (checkbox signal)
const focusScore = concentration * 10;
```

### Engagement Hub Reuse (Signal 5)

The Engagement Hub Phase 3b plans a Forum RSS Scanner that monitors Discourse forums. For Checkbox Voting Detection:

```
Engagement Hub Forum Scanner:
  → Monitors new topics/replies in governance categories
  → Stores: author, timestamp, topic, DAO

Checkbox Voting Detection reuses this data:
  → For each voter address:
    1. Resolve address to forum username (if available via Discourse profile or ENS)
    2. Check: did this user post in the relevant proposal's forum thread?
    3. Check: was the post BEFORE the vote timestamp?
    4. forum_score = posts_before_vote / total_votes * 10
```

**Note:** Address-to-forum-username resolution is imperfect. Not all voters have linked accounts. Phase 3 should treat missing forum data as neutral (not penalising), and only give a positive signal when forum participation IS confirmed.

---

## Success Metrics

### Phase 1

| Metric | Target | Measurement |
|--------|--------|-------------|
| Checkbox voters identified | >5% of voters per DAO flagged | voter_quality_signals table |
| GPI score shift | Measurable change in 30%+ of DAOs | Before/after comparison |
| No false positives | Manual review of top 10 flagged voters confirms checkbox behavior | Human QA |

### Phase 2

| Metric | Target | Measurement |
|--------|--------|-------------|
| Delegate Scorecard engagement | 20%+ of Matrix Details visitors view quality scores | Analytics event |
| Report upsell | Mention in 3+ governance forum threads | Community tracking |
| BCV satisfaction | Positive response from BCV in Lido thread | Forum check |

### Phase 3

| Metric | Target | Measurement |
|--------|--------|-------------|
| VQI adoption | Community references VQI as governance metric | Forum/Twitter monitoring |
| GVS composition accepted | No major pushback on ranking changes | Community feedback |
| Forum data coverage | >50% of top 20 DAOs have forum data | Data coverage report |

---

## Effort Summary

| Phase | Scope | Effort | Dependencies |
|-------|-------|--------|-------------|
| **Phase 1** | GPI enhancement (Signals 1+2) | ~8-10h | None |
| **Phase 2** | Delegate Scorecard (Signals 1-4) | ~20-25h | Phase 1, Attention Map |
| **Phase 3** | VQI as GVS dimension (Signals 1-5) | ~15-20h | Phase 2, Engagement Hub Phase 3b |
| **Total** | | **~43-55h** | |

---

## Risk Assessment

### Technical Risks

| Risk | Likelihood | Impact | Mitigation |
|------|-----------|--------|------------|
| **Voting timestamps too coarse** | Medium | Medium | Snapshot provides block-level timestamps; test resolution across DAOs |
| **Vote diversity metric flags legitimate aligned voting** | Medium | High | Use multiple signals in combination; single-signal flags are warnings not penalties |
| **Cross-category scoring unfair to generalist delegates** | Medium | Medium | "Focus" signal rewards specialisation but doesn't penalise breadth heavily (25% weight) |
| **Forum-to-wallet address resolution unreliable** | High | Medium | Treat as positive-only signal; missing data = neutral, not negative |

### Product Risks

| Risk | Likelihood | Impact | Mitigation |
|------|-----------|--------|------------|
| **Delegates feel "judged" or "graded"** | High | High | Frame as transparency, not judgment. "We show the data, the community decides what it means." |
| **DAOs push back on checkbox flagging** | Medium | Medium | Make VQS optional/hideable in early phases; let adoption grow organically |
| **GVS composition change (Phase 3) creates ranking drama** | High | High | Transition period, blog post, methodology page update, community feedback period |

### Mitigation: Framing

The biggest risk is how this is perceived. Key framing decisions:

- ✅ "Vote Quality Score" — neutral, descriptive
- ❌ "Checkbox Voter Flag" — accusatory, confrontational
- ✅ "Signals suggest rapid voting pattern" — observational
- ❌ "This delegate is a checkbox voter" — judgmental

The UI should **show signals and let users draw conclusions**, not label people.

---

## Monetization Impact

| Phase | Revenue Impact |
|-------|---------------|
| **Phase 1** | Indirect — better GPI scores improve product trust |
| **Phase 2** | Direct — Delegate Scorecard is a €149 Audit differentiator. "See not just who votes, but how well." |
| **Phase 3** | Strategic — VQI as unique metric cements ChainSights as the governance quality platform. Potential for API access tier. |

**Potential sales angle for €149 Audit:**
> "Your governance has 94% delegate participation. Sounds great, right? Our Vote Quality analysis shows 40% of those votes happen in under 60 seconds with no vote diversity. That's checkbox voting — and it's masking a participation crisis."

---

## Team Feedback Resolution (Feb 11, 2026)

### John (PM) — Before/After Measurement for Phase 1
**Concern:** How do we measure Phase 1 success if users see nothing?
**Resolution:** Validation script (same pattern as Wallet Label Filtering Epic 8) that compares GPI before/after checkbox weighting across all DGI DAOs. Run before deployment, document impact, keep as regression test.

### John (PM) — Phase 3 Timing
**Concern:** Changing GVS weights is a breaking change, too soon after other methodology updates.
**Resolution:** Phase 3 is NOT scheduled until Phase 2 is live and the community has adopted Vote Quality Scores. No GVS recomposition until we have community buy-in. Phase 1+2 first, Phase 3 when the feature is understood and accepted.

### Mary (BA) — DAO Size Normalisation for Signal 3
**Concern:** Small DAOs with <10 proposals/month — "voting on everything" is normal, not suspicious.
**Resolution:** Normalise Focus Score against total proposal count:
- DAOs with <10 proposals in analysis period: Signal 3 weight = 0% (neutral)
- DAOs with 10-25 proposals: Signal 3 weight scales linearly 0% → 25%
- DAOs with >25 proposals: Signal 3 at full 25% weight
Small DAOs aren't penalised for having generalist voters when there aren't enough proposals to specialise.

### Winston (Architect) — O(n²) Performance on Signal 4
**Concern:** Pattern Correlation comparing every voter against top whales doesn't scale.
**Resolution:** Pre-computed Whale Vote Matrix (architecture constraint):
1. Start of GVS job: build `Map<proposalId, whaleChoices[]>` for Top 3 whales
2. Per voter: compare against lookup table — O(1) per proposal
3. Total: O(n × m) instead of O(n × m × k)
Must be implemented this way regardless of development speed.

### Sally (UX) — Framing and Visual Representation
**Concern:** ⚠️ symbols and "checkbox voter" labels feel accusatory.
**Resolution:** Strict framing rules for Phase 2 UI:
- ❌ No warning symbols (⚠️, 🚩, ❗)
- ❌ No judgmental labels ("checkbox voter", "low quality", "suspicious")
- ✅ Colour gradient: Green → Yellow → Orange (spectrum, not binary)
- ✅ Neutral pattern descriptions: "Rapid, Consistent" / "Deliberate, Varied"
- ✅ Expanded view shows raw signal data, not interpretive text

Updated delegate detail concept:
```
0x7eE0...bb6C — DELEGATE — Vote Quality: 4.1/10

  Deliberation:  ██░░░░░░░░  2.1  Avg 23s between votes
  Independence:  ███░░░░░░░  3.4  92% same-direction votes
  Focus:         ██████░░░░  6.2  Active in 6 of 8 categories
  Originality:   ████░░░░░░  4.5  78% alignment with top holders

  Voting Pattern: Rapid, Consistent
```

### Paige (Technical Writer) — Methodology Documentation Timing
**Concern:** Documentation must ship WITH code, not after.
**Resolution:** Phase 1 deployment checklist:
1. ✅ Code deployed
2. ✅ Methodology page updated (same day)
3. ✅ Validation script results documented
4. ✅ Internal changelog updated
No deployment without methodology page update. One unit of work.

### Effort Re-Calibration
Effort estimates re-calibrated for AI-assisted development velocity (Epic 8: estimated 2-3 weeks, completed in 28 minutes):

| Phase | Original Estimate | Re-Calibrated |
|-------|------------------|---------------|
| Phase 1 | 8-10h | 1-2 sessions |
| Phase 2 | 20-25h | 2-4 sessions |
| Phase 3 | 15-20h | 2-3 sessions |

"Sessions" = focused Claude-assisted development blocks (~1-2h each).

---

## Future Vision

### Beyond Phase 3

- **Delegate Reputation System** — historical VQS over time, showing whether delegates improve or decline in quality
- **Checkbox Voting Alerts** — real-time notification when a known checkbox voter submits rapid-fire votes
- **Governance Health Recommendations** — AI-powered suggestions based on checkbox voting prevalence ("Consider requiring mandatory discussion periods")
- **Cross-DAO Delegate Comparison** — delegates active in multiple DAOs get a quality profile across their entire governance portfolio
- **Community-Contributed Signals** — allow DAOs to submit custom quality signals relevant to their governance model

### The Long-Term Story

ChainSights started by measuring **who** votes (identity-first analytics). Then **how power is distributed** (DGI, power concentration). Then **what they vote on** (Attention Map). Now **how well they vote** (Checkbox Detection → VQI).

The next step: **why they vote that way** — understanding voter motivation through forum analysis, delegation patterns, and social graph analytics. Each feature builds on the last, creating an increasingly deep governance intelligence layer that no competitor can easily replicate.

---
