# Story 10.3: Direct DM Campaign to Governance Leads

Status: ready-for-dev

## Story

As the **ChainSights founder**,
I want to **send personalized DMs to governance leads who have already shown interest**,
so that I can **convert warm engagement into conversations that lead to our first paying customer**.

## Acceptance Criteria

### AC1: Target Selection (Warm Leads Only)
- [ ] Only DM people who ALREADY engaged with ChainSights content
- [ ] Sources: Forum replies to Story 10.1 posts, Twitter replies to launch thread, Lido conversation participants
- [ ] Minimum 3 warm leads identified
- [ ] Each lead has documented prior engagement (screenshot/link)

### AC2: Message Personalization
- [ ] Each DM references their SPECIFIC previous engagement
- [ ] Example: "Hey [Name], saw your question about [specific topic] in the Lido forum..."
- [ ] NO generic "Hey, thought you might be interested" messages
- [ ] Each message includes ONE specific insight relevant to THEIR DAO

### AC3: Value-First Content
- [ ] NO sales pitch in initial DM
- [ ] Lead with additional value (expand on their question, offer more data)
- [ ] Soft CTA only: "Happy to share more details if helpful"
- [ ] NO pricing, NO report pitch until THEY ask

### AC4: Message Delivery
- [ ] DMs sent via appropriate channel (Twitter DM, Discord DM, or forum PM)
- [ ] Match channel to where they originally engaged
- [ ] Screenshot/record of sent messages for tracking
- [ ] Send within 48 hours of their engagement (while conversation is fresh)

### AC5: Response Tracking
- [ ] Tracking: Lead Name | DAO | Prior Engagement | DM Sent | Response | Next Step
- [ ] Follow-up plan: If no response in 5 days, DO NOT bump (they saw it and chose not to reply)
- [ ] If response received: Prioritize reply within 4 hours

## Tasks / Subtasks

- [ ] Task 1: Identify Warm Leads (AC: #1)
  - [ ] 1.1 Review Story 10.1 forum post responses - who replied?
  - [ ] 1.2 Review Story 10.2 Twitter thread - who engaged?
  - [ ] 1.3 Check Lido forum conversation - who asked questions?
  - [ ] 1.4 Document each lead with: Name, DAO, Original Engagement, Channel
  - [ ] 1.5 Verify minimum 3 leads before proceeding

- [ ] Task 2: Research Each Lead (AC: #2)
  - [ ] 2.1 For each lead, find their DAO's current GVS score
  - [ ] 2.2 Identify ONE specific insight about their DAO governance
  - [ ] 2.3 Review their original question/comment for context
  - [ ] 2.4 Note what additional value you can provide

- [ ] Task 3: Draft Personalized Messages (AC: #2, #3)
  - [ ] 3.1 Write 3+ unique DMs (NOT copy-paste with name swap)
  - [ ] 3.2 Each DM opens with reference to THEIR specific engagement
  - [ ] 3.3 Each DM includes relevant insight for THEIR DAO
  - [ ] 3.4 Each DM ends with soft value-offer CTA
  - [ ] 3.5 Review against DO NOT list (no pitch, no pricing, no criticism)

- [ ] Task 4: Send and Track (AC: #4, #5)
  - [ ] 4.1 Send each DM via their preferred channel
  - [ ] 4.2 Screenshot each sent message
  - [ ] 4.3 Log in tracking document
  - [ ] 4.4 Set calendar reminder for response check (24h, 72h)

## Dev Notes

### CRITICAL CONTEXT - READ THIS FIRST

**This story is ONLY for WARM leads.** Do NOT cold DM anyone.

### Previous DM Failure Analysis (From project-context.md)

**Week 51 Cold DM Campaign - TOTAL FAILURE:**
- Targets: Sinkas (Arbitrum), Lefteris (Optimism), Nick Johnson (ENS), kpk (karpatkey), Wintermute, Areta
- Results: 0 responses, 0 engagement, complete silence

**Why It Failed:**
1. **No credibility** - Unknown sender with no social proof
2. **Felt like criticism** - They could see we publicly roasted DAOs
3. **Wrong tier** - Too high-profile for cold start
4. **No warm intro** - Cold DM in noisy space = ignored

### What's Different This Time

Story 10.3 is designed to ONLY target people who already engaged:

| Criteria | Cold DM (Failed) | Warm DM (This Story) |
|----------|------------------|----------------------|
| Prior engagement | None | Required |
| Target tier | High-profile | Mid-tier |
| Opening | Generic | Reference their comment |
| Pitch | Implied | None |
| Success rate | 0% | TBD (higher expected) |

### Example Messages

**GOOD (Warm, Value-First):**
```
Hey [Name],

Saw your question in the Lido forum about how we measure delegate concentration.

Quick answer: We look at voting power distribution across the top 20 delegates, not just the top 5. Lido's top 5 control 51.1% vs ENS at 31% - but the full picture shows Lido's Nakamoto coefficient at 5 (meaning 5 delegates needed to reach 50%).

Happy to share more details on how we calculate this if useful for your governance work.

- Mario
```

**BAD (Cold, Sales-y):**
```
Hey [Name],

I built ChainSights, a governance analytics platform. We score DAOs on governance health.

Your DAO scores 6.5/10. Want a detailed report for €49?

- Mario
```

### Warm Lead Sources

1. **Story 10.1 Forum Posts**
   - Check: docs/marketing/outreach-drafts-story-10-1.md
   - Look for: Forum replies, questions, engagement

2. **Story 10.2 Twitter Thread**
   - Check: Twitter @ChainSights_one thread responses
   - Look for: Replies, quote tweets, questions

3. **Lido Forum Conversation**
   - Known: Lido DAO Operations showed interest
   - Asked for comparative data (positive signal)

### DO NOT List

- [ ] Do NOT DM anyone who hasn't engaged first
- [ ] Do NOT mention pricing in initial DM
- [ ] Do NOT ask for a call/meeting in initial DM
- [ ] Do NOT send generic messages with just name swapped
- [ ] Do NOT follow up if no response (they chose not to reply)
- [ ] Do NOT combine with public criticism (learned from Week 51)

### Success Metrics

| Metric | Target | Stretch |
|--------|--------|---------|
| Warm Leads Identified | 3 | 5+ |
| DMs Sent | 3 | 5+ |
| Response Rate | 33% (1 reply) | 66% (2+ replies) |
| Conversation Started | 1 | 2+ |
| Report Interest Expressed | 1 (stretch) | - |

### Time Estimate

| Task | Time |
|------|------|
| Identify warm leads | 30 min |
| Research each lead | 30 min |
| Draft personalized DMs | 45 min |
| Send and track | 15 min |
| **Total** | ~2 hours |

### Project Structure Notes

- Output tracking: Can use simple markdown or spreadsheet
- Reference: docs/marketing/outreach-drafts-story-10-1.md (working templates)
- Reference: docs/marketing/launch-thread-FINAL.md (Twitter thread for engagement check)

### References

- [Source: project-context.md#WHAT-WE'VE-TRIED-FAILURES] - Week 51 DM failure analysis
- [Source: project-context.md#LESSONS-LEARNED] - DO/DON'T lists
- [Source: docs/sprint-artifacts/10-1-seed-outreach-to-3-target-daos.md] - Seed outreach approach
- [Source: docs/sprint-artifacts/10-2-create-twitter-launch-thread.md] - Twitter thread (check for engagement)

## Dev Agent Record

### Context Reference

Story created from Epic 10: Marketing Sprint 0 - First Revenue Focus.
This is a MARKETING story, not a code story. No development work required.

### Agent Model Used

### Debug Log References

### Completion Notes List

### File List

