# Concept: 5 Curated Reports - Free Tier Redesign

**Created:** 2026-01-16
**Status:** Concept for Review
**Proposed By:** Mario + Maya (Marketing)
**Priority:** Medium-High

---

## Executive Summary

Replace dynamic data generation for free users with 5 pre-generated, curated example reports. Free users can explore these examples to understand the product, while generation becomes a paid-only feature.

**Mario's Original Insight:**
> *"wir sollten uns überlegen, ob wir nicht 5 vorbereitete reports haben die wir random interessierten usern zeigen. generierung muss ja nicht unbedingt sein für nicht-zahlende user... das sehen sie ja am video, wie das funktioniert"*

---

## Problem Statement

### Current State
- Free users can generate unlimited scenarios
- Each generation costs server resources (Claude API, DB, compute)
- Brian's video already shows how generation works
- No differentiation between free and paid experience

### Issues
1. **Cost:** Each free generation costs ~$0.02-0.05 in API fees
2. **Abuse Potential:** Unlimited free generation invites abuse
3. **Value Leakage:** Full product available without payment
4. **No Urgency:** Why pay if free works?

---

## Proposed Solution

### "Explore Examples" vs "Generate Your Own"

| User Type | Experience |
|-----------|------------|
| **Free User** | Browse 5 curated example reports |
| **Paid User** | Generate unlimited custom scenarios |

### The 5 Curated Reports

Pre-generate and store 5 high-quality example scenarios covering the Size × Sector matrix:

| # | Size | Sector | Company Name | Highlights |
|---|------|--------|--------------|------------|
| 1 | **Startup** | Consumer Staples | "Alpine Bakery GmbH" | Rapid growth, seasonal peaks |
| 2 | **MidCap** | Industrials | "TechParts Manufacturing AG" | Steady growth, supply chain events |
| 3 | **LargeCap** | Financials | "EuroCredit Union" | Large workforce, regulatory events |
| 4 | **Startup** | Industrials | "GreenTech Solutions" | Tech startup, funding rounds |
| 5 | **MidCap** | Consumer Staples | "FreshFood Distribution" | Logistics, regional expansion |

### User Journey

```
Free User lands on tellingcube.com
        │
        ▼
┌─────────────────────────────────┐
│   "Explore 5 Example Datasets"  │
│   ┌─────┐ ┌─────┐ ┌─────┐      │
│   │  1  │ │  2  │ │  3  │ ...  │
│   └─────┘ └─────┘ └─────┘      │
└─────────────────────────────────┘
        │
        ▼ (clicks example)
┌─────────────────────────────────┐
│   Full Report View              │
│   - Finance charts              │
│   - Sales charts                │
│   - CSV download (sample)       │
│   - API preview                 │
│                                 │
│   [Generate Your Own - €29]     │
└─────────────────────────────────┘
```

---

## Technical Implementation

### Phase 1: Generate & Store Curated Data

**One-time generation:**
1. Generate 5 scenarios with current system
2. Export complete data as JSON files
3. Store in `/public/examples/` (static hosting)
4. No database queries needed for free users

**File Structure:**
```
public/
  examples/
    alpine-bakery/
      scenario.json      # Metadata
      events.json        # Raw events
      finance.json       # Aggregated finance
      sales.json         # Aggregated sales
      world.json         # Company details
    techparts-manufacturing/
      ...
    eurocredit-union/
      ...
```

### Phase 2: UI Changes

**Homepage:**
- Replace "Generate Scenario" with "Explore Examples"
- Show 5 cards with company previews
- "Want your own data? Get started for €29"

**Example View:**
- Full Finance/Sales views (read-only)
- "Sample" badge on downloads
- Prominent upgrade CTA

**Scenario Selection (Paid Users Only):**
- Accessible after login/payment
- Full 9-scenario matrix
- Custom generation

### Phase 3: API Changes

**Free Endpoints:**
```
GET /api/examples                    # List all 5 examples
GET /api/examples/{slug}/scenario    # Example metadata
GET /api/examples/{slug}/events      # Example events (truncated)
GET /api/examples/{slug}/finance     # Example finance data
GET /api/examples/{slug}/sales       # Example sales data
```

**Paid Endpoints (Unchanged):**
```
POST /api/scenarios/generate         # Requires auth
GET  /api/scenarios/{id}/...         # Requires ownership
```

---

## Benefits

### For Business
| Benefit | Impact |
|---------|--------|
| **Reduced Costs** | No API calls for free users |
| **Clear Value Prop** | "See examples" vs "Generate your own" |
| **Conversion Driver** | Examples create desire for custom data |
| **Abuse Prevention** | No free generation = no abuse |

### For Users
| Benefit | Impact |
|---------|--------|
| **Instant Access** | No waiting for generation |
| **Quality Guaranteed** | Curated = best examples |
| **Video Alignment** | Brian's video shows generation process |
| **Try Before Buy** | Full exploration without commitment |

---

## Messaging Strategy

### Homepage Copy

**Before:**
> "Generate realistic business data"

**After:**
> "Explore realistic business data"
>
> *See how tellingCube creates consistent financial datasets with 5 ready-to-explore examples. Want your own custom company? Generate unlimited scenarios for €29.*

### CTA Hierarchy

1. **Primary:** "Explore Examples" (free)
2. **Secondary:** "Generate Your Own" (€29)
3. **Tertiary:** "Watch Demo" (Brian's video)

---

## Migration Plan

### Week 1: Prepare
1. Generate 5 high-quality example scenarios
2. Export and store as static JSON
3. Design example selection UI

### Week 2: Implement
1. Build example browsing pages
2. Modify homepage flow
3. Add upgrade CTAs

### Week 3: Launch
1. A/B test with subset of traffic
2. Monitor conversion rates
3. Iterate based on data

---

## Success Metrics

| Metric | Current | Target |
|--------|---------|--------|
| Free-to-Paid Conversion | ~1% | 3-5% |
| Server Costs (Free Users) | $X/month | ~$0 |
| Time-to-Value | 60-90s (generation) | Instant |
| Bounce Rate | 65% | <50% |

---

## Risks & Mitigations

| Risk | Mitigation |
|------|------------|
| Users want specific scenario | "Generate Your Own" CTA prominent |
| Examples feel "fake" | High quality, diverse selection |
| Less engagement | Make examples interactive, explorable |
| SEO impact | Keep unique URLs, add example descriptions |

---

## Open Questions

1. Should examples include full CSV download or truncated sample?
2. Rotate examples periodically or keep static?
3. Allow API access to examples (for developers testing integrations)?
4. Show generation progress as "preview" for free users (but not actual data)?

---

## Comparison: Free Tier Options

| Option | Pros | Cons |
|--------|------|------|
| **A: Curated Examples (this proposal)** | Zero cost, instant access, quality control | Less personalization |
| B: Limited Generation (1 per month) | Some personalization | Still has costs, abuse potential |
| C: Watermarked Generation | Full experience | Confusing, still costs money |
| D: Time-Limited Trial | Urgency | Complex to implement |

**Recommendation:** Option A (Curated Examples) - simplest, cheapest, clearest value proposition.

---

## Next Steps

1. [ ] Mario reviews concept
2. [ ] Decide on 5 example companies
3. [ ] Generate and curate examples
4. [ ] Design example browser UI
5. [ ] Implement and A/B test

---

*Prepared by: Maya (Marketing) + Winston (Architect)*
*For review by: Mario*
