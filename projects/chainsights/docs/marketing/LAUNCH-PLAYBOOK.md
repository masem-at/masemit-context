# ChainSights Rankings Launch Playbook
## 6-Hour Sprint to Market

Created: December 24, 2025
Epic Milestone: 8/9 Epics Complete 🎉

---

## Overview

This is the execution plan for launching the ChainSights DAO Rankings platform. Everything is tested, built, and ready. Now we execute.

**Core Message:** Free DAO governance rankings are live. Real identity-first analysis. No signup required.

---

## Hour 1-2: Pre-Launch Testing & Content Prep

### ✅ Site Health Check (30 minutes)
- [x] All payment CTAs open health check modal (NOT direct Stripe)
- [x] Rankings page loads and displays properly
- [x] Analysis Type column shows correctly
- [x] Quiz and Guide pages have headers
- [ ] Test mobile experience on rankings
- [ ] Verify all navigation links work
- [ ] Check page load speeds

### 📸 Screenshot Collection (30 minutes)
Create screenshots for social proof:
- [ ] Rankings page full view
- [ ] Example DAO with expanded details (pick top 3)
- [ ] Component score breakdown visualization
- [ ] Health status indicators (Vital, Stable, Concerning, Critical)
- [ ] Mobile view of rankings

### ✍️ Launch Thread Draft (60 minutes)
Write and save draft in `docs/marketing/launch-thread-FINAL.md`:

**Thread Structure:**
1. Hook: "We analyzed governance for 10 major DAOs. Here's what we found 👇"
2. Problem: Wallet counts lie. Real participation is 1/10 what it looks like
3. Example: "ENS: 142K followers, only 302 REAL voters (0.2%)"
4. Solution: ChainSights rankings - identity-first analysis
5. What makes us different: Cluster wallets, identify delegates vs whales
6. Live now: https://chainsights.one/rankings
7. Call to action: Check your DAO's rank

---

## Hour 3-4: Launch Thread & Initial Posts

### 🐦 Twitter/X Launch Thread (30 minutes)
- [ ] Post launch thread from @ChainSights_one account
- [ ] Pin thread to profile
- [ ] Add to highlights

**Sample Tweet 1 (Hook):**
```
🚨 We analyzed governance for 10 major DAOs.

What we found will change how you think about "decentralization"

🧵 Thread 👇
```

**Tweet 2 (Problem):**
```
Most DAO dashboards show wallet counts.

But wallets ≠ people.

One person can control 50+ wallets.
One whale can have 1000x the power of a community member.

The numbers you see? They're lying to you.
```

**Tweet 3 (Example):**
```
Take @ensdomains:

• 142,908 Twitter followers
• 302 REAL unique voters
• That's 0.2% participation

And the top 20 power holders? All delegates.
Good sign for legitimacy.
Bad sign? They vote on <1% of proposals.
```

**Tweet 4 (Solution):**
```
That's why we built ChainSights Rankings

↳ Identity-first governance analysis
↳ Cluster wallets to find real people
↳ Distinguish delegates from whales
↳ Track REAL participation rates

Live now: https://chainsights.one/rankings
```

**Tweet 5 (CTA):**
```
Check where your DAO ranks 👇

https://chainsights.one/rankings

Free. No signup. Updated weekly.

RT if you think your community needs to see this.
```

### 📱 LinkedIn Post (30 minutes)
Adapt Twitter thread for LinkedIn (more professional tone):
- [ ] Post from company page or personal profile
- [ ] Include 2-3 ranking screenshots
- [ ] Tag relevant DAOs/protocols
- [ ] Shorter, more direct copy

**LinkedIn Opening:**
```
Most DAO governance dashboards are lying to you.

Here's what we found after analyzing 10 major DAOs...
```

### 🎯 Product Hunt (30 minutes) - OPTIONAL Day 1
Only if team bandwidth allows:
- [ ] Submit to Product Hunt
- [ ] Prep hunter outreach if needed
- [ ] Coordinate upvote campaign

---

## Hour 5-6: Community Engagement

### 🎪 Forum Posts (Target: 2-3 High-Quality Posts)

**Priority Forums:**
1. **Arbitrum Forum** (arbitrum.foundation)
2. **Optimism Forum** (gov.optimism.io)
3. **ENS Forum** (discuss.ens.domains)

**Post Template:**
```
Title: [DAO Name] Governance Rankings Now Live - Free Identity-First Analysis

Hey [DAO Name] community,

We just launched free governance rankings for major DAOs, and [DAO NAME] is included.

What makes this different: We cluster wallets to identify REAL participation, not just wallet counts. We also distinguish between:
- Whale concentration (token holders)
- Delegate concentration (trusted community members)

Because those are very different things.

[DAO NAME]'s current ranking: [RANK]/10 (link)

Key findings:
• [Metric 1]
• [Metric 2]
• [Metric 3]

The full rankings + methodology: https://chainsights.one/rankings

Would love the community's feedback. Especially if we're missing context that would improve the analysis.

--
*Disclosure: This is a commercial service (paid deep-dive reports available), but all rankings are free and public.*
```

**Response Strategy:**
- Monitor posts hourly during launch day
- Respond to ALL comments within 2 hours
- Be humble, curious, not defensive
- Ask questions about their governance context
- Offer to add missing context

---

## Hour 6+: Monitoring & Amplification

### 📊 Track Metrics
Monitor in Vercel Analytics:
- [ ] Traffic to /rankings
- [ ] CTA clicks (modal opens)
- [ ] Time on site
- [ ] Bounce rate
- [ ] Top referring sources

### 🔁 Engagement Loop
For 24 hours post-launch:
- [ ] Respond to all comments/replies within 2 hours
- [ ] Share positive feedback as quote tweets
- [ ] Join relevant Twitter Spaces if happening
- [ ] Monitor DAO Discord servers for mentions

### 📈 Amplification Opportunities
- [ ] Reply to governance-related threads with "We just analyzed this - here's what we found: [link]"
- [ ] Share specific DAO insights: "Did you know [DAO] has X% concentration? [link to ranking]"
- [ ] Use relevant hashtags: #DeFi #DAO #Web3Governance #Decentralization

---

## Post-Launch: Days 2-7

### Day 2-3: Content Marketing
- [ ] Write blog post: "How We Calculate Governance Vitality Scores"
- [ ] Create explainer thread: "Why wallet counts don't matter"
- [ ] Share individual DAO spotlights: "Let's talk about [DAO]'s governance"

### Day 4-5: Outreach
- [ ] Email relevant newsletters (Bankless, The Defiant)
- [ ] Reach out to governance podcasts
- [ ] DM governance leads of ranked DAOs (use dm-outreach-guide.md)

### Day 6-7: Iteration
- [ ] Collect all feedback
- [ ] Identify top 3 complaints/questions
- [ ] Plan improvements for next week
- [ ] Update methodology if needed

---

## Critical Success Factors

### ✅ DO THIS:
1. **Be humble** - "We might have missed context"
2. **Show receipts** - Link to methodology, data sources
3. **Distinguish commercial from free** - Rankings free, reports paid
4. **Respond fast** - Within 2 hours during launch window
5. **Celebrate others** - "X DAO is doing governance RIGHT"

### ❌ DON'T DO THIS:
1. **Don't spam** - Max 1 post per forum
2. **Don't be defensive** - Criticism is data
3. **Don't oversell** - Let the data speak
4. **Don't ignore feedback** - Every comment deserves a response
5. **Don't claim perfection** - Acknowledge limitations upfront

---

## Emergency Contacts & Backup Plans

### If Site Goes Down:
- Check Vercel dashboard
- Check database connection (Neon)
- Fallback: "We're experiencing high traffic - back soon"

### If Negative Feedback Escalates:
- Stay calm
- Acknowledge concern
- Ask clarifying questions
- Offer to take discussion to DMs
- Don't argue in public

### If A DAO Requests Removal:
- Follow opt-out process immediately
- Document request
- Respond within 24 hours
- Process within 48 hours
- Update rankings page

---

## Success Metrics - First Week

**Minimum Viable Success:**
- 500+ unique visitors to /rankings
- 10+ engaged community responses (not just views)
- 0 major technical issues
- 0 reputation-damaging incidents

**Good Success:**
- 2,000+ unique visitors
- 50+ engaged responses
- 3-5 inbound leads for paid reports
- Positive sentiment ratio >70%

**Exceptional Success:**
- 5,000+ unique visitors
- 100+ engaged responses
- 10+ inbound leads
- Featured in major crypto newsletter
- Partnership inquiry from a ranked DAO

---

## Post-Launch Review (After 7 Days)

Schedule time to review:
1. What worked? (Double down)
2. What didn't? (Stop doing)
3. What surprised us? (Investigate)
4. What should we do next? (Prioritize)

Document findings in `docs/marketing/launch-retrospective.md`

---

## Notes from Maya (Marketing Strategist)

**Context is everything.** Every DAO has unique governance challenges. Our job isn't to judge - it's to illuminate.

**Bad concentration isn't always bad.** Sometimes a DAO SHOULD have concentrated power (early stage, emergency actions). Our analysis shows the numbers. Communities provide the context.

**Listen more than you talk.** Especially in the first week. Every piece of feedback is golden. Save it all.

**This is a marathon, not a sprint.** We're building trust in a space that's been burned by analytics platforms before. Be patient. Be honest. Be helpful.

---

**Ready? Let's launch.** 🚀

Last updated: December 24, 2025
