# ChainSights Metric Naming Guidelines

> **Purpose**: Prevent metric confusion incidents like the Lido Discord misunderstanding (Jan 2026).
> **Audience**: All BMAD agents working on ChainSights codebase.

## Incident Summary

A user interpreted "Human Participation 10/10" from Rankings as "participation rate" when it actually measured "Sybil resistance" (% of voters that are human). This caused credibility concerns when the Deep Dive Report showed 0.52% actual participation rate.

**Root cause**: Metric naming was technically accurate but user-confusing.

## Core Principle

**Metric names must communicate WHAT is being measured, not just the technical name.**

Bad: "Human Participation Rate" (sounds like engagement)
Good: "Sybil Resistance Score" (clearly about bot detection)

## ChainSights Metric Reference

### Rankings Page (GVS System)

| Internal Name | Display Name | What It Measures |
|---------------|--------------|------------------|
| `hprScore` | **Sybil Resistance Score** | % of voters that are human (not bots) |
| `deiScore` | Delegate Engagement Index | How actively delegates participate |
| `pdiScore` | Power Dynamics Index | Voting power concentration |
| `gpiScore` | Grassroots Participation Index | Small holder participation |
| `gvsScore` | Governance Vitality Score | Weighted composite of above |

### Deep Dive Report (AI Analysis)

| Metric | What It Measures |
|--------|------------------|
| **Decentralization Score** | Power concentration (Gini, Nakamoto, Top5) |
| **Participation Rate** | % of followers who voted |
| **Top 5 Control** | % voting power held by top 5 wallets |
| **Gini Coefficient** | Inequality measure (0-1) |
| **Nakamoto Coefficient** | Min wallets for 51% control |

## Critical Distinctions

### GVS â‰  Decentralization

- **GVS** = Governance activity quality (are voters human? are delegates active?)
- **Decentralization** = Power distribution (who controls the votes?)

A DAO can have:
- High GVS + Low Decentralization (active human voters, but concentrated power)
- Low GVS + High Decentralization (distributed power, but low activity)

### Sybil Resistance â‰  Participation Rate

- **Sybil Resistance 10/10** = Of the 170 voters, all are human
- **Participation 0.52%** = Only 170 of 32,567 followers voted

Both can be true simultaneously!

## Guidelines for Agents

### When Adding New Metrics

1. **Name for the user, not the code** - Use terms non-technical users understand
2. **Include "what" in the name** - "Sybil Resistance" is clearer than "Human Participation"
3. **Add disambiguation in descriptions** - Always clarify what the metric does NOT measure
4. **Check for similar-sounding metrics** - Avoid names that could be confused with existing metrics

### When Discussing Metrics with Users

1. **Always specify which system** - "In the Rankings..." or "In the Report..."
2. **Proactively clarify confusing pairs** - If mentioning Sybil Resistance, note it's not participation rate
3. **Use concrete numbers** - "170 voters out of 32,567 followers" is clearer than "low participation"

### When Writing UI Copy

1. **Add tooltips** - Explain metrics on hover
2. **Link to methodology** - Always provide path to detailed explanation
3. **Use consistent terminology** - Don't switch between "HPR" and "Human Participation" and "Sybil Resistance"

## Files That Define These Metrics

### Rankings/GVS System
- `src/lib/gvs/hpr.ts` - Sybil Resistance calculation
- `src/lib/gvs/types.ts` - Type definitions
- `src/components/ExpandableDAORow.tsx` - Desktop UI display
- `src/components/MobileDAOCard.tsx` - Mobile UI display

### Deep Dive Reports
- `src/lib/ai/analysis.ts` - Report data generation
- `src/lib/pdf/report-template.tsx` - PDF layout

### Documentation
- `src/app/rankings/methodology/page.tsx` - Public methodology explanation
- `src/app/guide/page.tsx` - User guide (includes Rankings vs Reports section)
- `src/components/faq.tsx` - FAQ answers
- `src/app/layout.tsx` - SEO structured data

## Checklist for Future Metric Changes

- [ ] Updated all UI components with new name
- [ ] Updated FAQ answers
- [ ] Updated methodology page
- [ ] Updated structured data (layout.tsx)
- [ ] Updated guide page
- [ ] Added/updated disambiguation text
- [ ] Tested that name doesn't conflict with other metrics

---

*Last updated: January 2026 after Lido metric confusion incident.*