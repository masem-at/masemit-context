# ChainSights Launch Plan

**Date:** December 15, 2025
**Author:** Mario (masemIT e.U.)
**Status:** Approved

---

## Strategic Decision

**Approach:** Series Teaser + Gitcoin First

Instead of launching with a single "exposé" report that could be perceived as attacking a DAO, we launch with a **neutral comparison** of multiple DAOs, positioning ChainSights as a data analyst — not a critic.

---

## Why Gitcoin First

| Reason | Strategic Value |
|--------|-----------------|
| **Self-critical community** | They WANT to improve. Won't shoot the messenger. |
| **Values alignment** | "Public goods" people care about transparency. That's our brand. |
| **Twitter presence** | Kevin Owocki, Scott Moore = 100K+ potential eyeballs |
| **Grants ecosystem** | These people fund DAOs. They need to evaluate governance. Potential customers! |
| **Safe learning** | Less defensive than ENS, more credible than tiny DAOs |

---

## Why NOT ENS First

ENS at 95% concentration is a bomb.

Risks of leading with ENS:
- Get labeled "that guy who attacks DAOs"
- Defensive community reaction
- Overshadows the product with drama

**Better approach:** Tease ENS numbers in comparison, let the data speak without making it personal.

---

## The Series Teaser Strategy

### Reframing

| Old Approach | New Approach |
|--------------|--------------|
| "ENS is centralized! Look at this exposé!" | "We analyzed 3 major DAOs. Here's what governance concentration actually looks like." |

### The Comparison Hook

| DAO | Top 20 Wallet Concentration |
|-----|----------------------------|
| ENS | 94.9% |
| Gitcoin | ??% |
| Aave | ??% |

> "Want your DAO analyzed? €99. chainsights.one"

### Why This Works

1. **Not attacking** — presenting data neutrally
2. **Comparing** — more interesting than single report
3. **Teasing** — "Want the full breakdown? Buy the report."
4. **Positioning** — neutral analyst, not critic

---

## Launch Sequence

| Step | Action | Purpose |
|------|--------|---------|
| 1 | Build MVP with Gitcoin as primary test case | Validate pipeline end-to-end |
| 2 | Run analysis on ENS + Aave (comparison data) | Gather teaser content |
| 3 | First public content: "3 DAOs Compared" Twitter thread | Launch awareness |
| 4 | Full Gitcoin report as free sample download | Lead magnet, prove value |
| 5 | ENS/Aave reports as paid products or marketing | Revenue + credibility |

---

## The Twitter Thread (Draft)

```
🧵 I analyzed governance concentration in 3 major DAOs.

The results surprised me.

Thread 👇

1/ We talk about "decentralization" constantly in Web3.

But how decentralized is governance, really?

I pulled voting data from @gitcoin @ENS_DAO @AaveAave

2/ Here's what I found:

Top 20 wallets control:
- ENS: 94.9%
- Gitcoin: [X]%
- Aave: [Y]%

3/ But wait — wallets ≠ people.

What if some of those 20 wallets... are the same person?

That's what I'm building @ChainSights_one to find out.

4/ Full Gitcoin analysis (free):
[link to PDF]

Want your DAO analyzed?
€99. chainsights.one

Wallets lie. We don't.
```

---

## Risk Mitigation

### What if Gitcoin's numbers are actually GOOD?

Example: Top 20 = only 50%

**This is still a story:**

> "ENS: 95% concentrated. Gitcoin: 50% concentrated. What's the difference? Let's dig in."

Contrast is content. Either way, we win.

### What if a DAO reacts negatively?

- Data is public (Snapshot API)
- We're reporting facts, not opinions
- Our methodology is transparent
- "Honor criticism" policy applies to us too

---

## Content Calendar (Post-Launch)

| Week | Content |
|------|---------|
| Week 1 | "3 DAOs Compared" thread + free Gitcoin report |
| Week 2 | Deep-dive thread on one insight (delegate vs whale) |
| Week 3 | Customer testimonial (if Share & Save works) |
| Week 4 | "What we learned from 10 reports" thread |

---

## Success Metrics

| Metric | Target | Why |
|--------|--------|-----|
| Thread impressions | 10K+ | Validates audience interest |
| Free report downloads | 100+ | Proves lead magnet works |
| Paid report purchases | 3+ in first month | Revenue validation |
| Share & Save participation | 20%+ of customers | Viral loop validation |

---

## Key Insight from Spike

The `vp_by_strategy` field from Snapshot reveals **delegate vs whale** breakdown:

```json
"vp_by_strategy": [0.1, 267598.4]
                   │      │
                   │      └── Delegated power (267K!)
                   └── Own tokens (0.1)
```

**This is novel.** Dune doesn't show this. Nansen doesn't show this.

| Type | Pattern | Meaning |
|------|---------|---------|
| **Delegates** | High delegated, low own | Trusted voices, experts |
| **Whales** | High own, low delegated | Big investors, potential manipulators |
| **Delegate-Whales** | High both | The real power holders |

This becomes a key differentiator in our reports.

---

## Brand Positioning

**We are:** Neutral data analysts presenting governance reality

**We are NOT:** Critics, attackers, or "exposé" journalists

**Tagline alignment:** "Wallets lie. We don't." = We report truth, even uncomfortable truth

**Criticism policy:** We honor criticism of our own work too. Transparency IS the brand.

---

*Document created: December 15, 2025*
*Decision validated through: Spike data analysis + strategic discussion*
