# tellingCube Product Brief

**Version:** 2.0
**Created:** 2026-01-21
**Author:** Mary (Business Analyst) with strategic input from Mario Semper
**Status:** ACTIVE - Supersedes all previous positioning

---

## Executive Summary

tellingCube is a **Realistic Business Data Generator** that produces mathematically consistent synthetic data for demos, testing, and training. Unlike AI-generated data that hallucinates numbers or random generators that produce obvious garbage, tellingCube's event-sourcing architecture ensures every view—Finance, Sales, HR—derives from the same transaction stream. The numbers always reconcile.

**One-Liner:**
> "Realistic business data that actually reconciles."

**Elevator Pitch (30 seconds):**
> "tellingCube generates consistent financial scenarios in minutes. Every transaction—sales, payroll, invoices—flows through one event stream. Finance, Sales, and HR always match. Because they come from the same source. No more random garbage. No more weeks of data prep. 3 minutes instead of 3 hours."

---

## The Problem

### What We're Solving

Professionals who need realistic business data for demos, testing, or training face a painful choice:

| Option | Problem |
|--------|---------|
| **AI (ChatGPT, Claude)** | Hallucinates numbers. P&L doesn't reconcile with balance sheet. |
| **Random Generators** | Obviously fake. Clients notice immediately. |
| **Real Data** | NDAs, GDPR, compliance issues. Can't use it. |
| **Manual (Excel)** | Takes 3-4 hours minimum. Still full of errors. |

### The Pain in Their Words

**Brian Julius (55K LinkedIn followers):**
> "One of the things I've always had trouble with is developing realistic data that's consistent. That when you put it in visuals, it doesn't look crazy. The numbers add up."

**Koteswara Rao (Data Professional):**
> "We all struggle for synthetic data specially to make hands dirty and also to build a POC."

**Jürgen Faisst (CEO, IBCS Institute):**
> "Relevant for software vendors who today build demos on unrealistic, overly simplistic, or inconsistent data, as well as for BI consultants who want to create mockups based on real-world models."

---

## The Solution

### How tellingCube Works

tellingCube uses an **event-sourcing architecture**:

```
                    ┌─────────────────┐
                    │  EVENT STREAM   │
                    │  (Single Source)│
                    └────────┬────────┘
                             │
         ┌───────────────────┼───────────────────┐
         │                   │                   │
         ▼                   ▼                   ▼
   ┌──────────┐        ┌──────────┐        ┌──────────┐
   │ FINANCE  │        │  SALES   │        │    HR    │
   │   VIEW   │        │   VIEW   │        │   VIEW   │
   └──────────┘        └──────────┘        └──────────┘
```

Every transaction—sales, payroll, invoices—flows through one event stream. All views derive from the same data. That's why the numbers always match:

- Revenue matches across Sales and Finance
- Payroll matches HR headcount
- COGS matches purchasing events
- Every € is traceable to a source event

### Key Differentiators

| vs Competitor | Our Advantage |
|---------------|---------------|
| vs AI (ChatGPT/Claude) | "AI hallucinates. Numbers don't reconcile." |
| vs Excel/Manual | "3-4 hours in Excel. 3 minutes with tellingCube." |
| vs Random Generators | "Not random garbage. Event-based logic." |
| vs Real Data | "No NDAs, no GDPR issues. Use it anywhere." |

---

## Target Audiences

### Priority Order

| Priority | Segment | Use Case | Pain Point |
|----------|---------|----------|------------|
| **1** | BI Consultants | Client demos, POCs, prototypes | "I need realistic data for demos but can't show real client data" |
| **2** | Software Vendors | Product demos, sales enablement | "Our demo data looks fake and customers notice" |
| **3** | Trainers/Educators | Workshops, courses, exercises | "I need scenarios that are didactic AND realistic" |
| **4** | Developers/QA | Testing, development, QA | "I need test data covering edge cases without production data" |
| **5** | Data Scientists | ML training, experimentation | "Synthetic data for ML must be statistically valid" |

### Audience-Specific Hooks

| Audience | Hook |
|----------|------|
| BI Consultants | "Stop using fake data for client demos" |
| Software Vendors | "Your demo data looks fake. Customers notice." |
| Trainers | "Realistic scenarios for workshops, ready in 3 minutes" |
| Developers | "Test data that covers edge cases. No production data risk." |
| Data Scientists | "Synthetic data that's statistically valid" |

---

## Key Messages

Use these consistently across all communications:

1. **"Realistic business data that actually reconciles"** - Primary message
2. **"3 minutes instead of 3 hours"** - Time savings
3. **"AI hallucinates. tellingCube doesn't."** - vs AI positioning
4. **"Every € is traceable to a source event"** - Technical credibility
5. **"3% goes to children's hospice"** - hoki.help charity angle

---

## Social Proof

### Hierarchy (use in this order)

1. **Brian Julius (55K followers):**
   > "The ivory-billed woodpecker of the data world. Specific use, but one distinctive click instantly, reliably meets the exact need."

2. **Jürgen Faisst (IBCS Institute):**
   > "Relevant for software vendors and BI consultants who want to create mockups based on real-world models."

3. **Early User Testimonials** (as they come in)

4. **hoki.help Charity Model** - 3% of revenue donated to children's hospice Austria

---

## Product Capabilities

### What tellingCube Does

| Feature | Description |
|---------|-------------|
| **Multi-Year Scenarios** | 1, 3, or 5 year data generation |
| **Multiple Domains** | Finance, Sales, HR views |
| **9 Company Profiles** | Startup → MidCap → LargeCap |
| **3 Industries** | Consumer Staples, Industrials, Financials |
| **Export Options** | CSV files + JSON API |
| **Mathematical Consistency** | Guaranteed reconciliation |
| **Preview Charts** | Visual data quality check |
| **Fast Generation** | ~3 minutes per scenario |

### IBCS Positioning

**IBCS is a FEATURE, not the PRODUCT.**

| Acceptable | Not Acceptable |
|------------|----------------|
| "Charts follow IBCS® visualization standards" (small print) | "IBCS-ready business data" |
| Mentioning Jürgen Faisst for credibility (sparingly) | "You've mastered IBCS standards..." |
| | IBCS in headlines or main selling points |
| | Leading with IBCS as the main topic |

---

## Pricing Model

### Tiers

| Tier | Price | Includes |
|------|-------|----------|
| **Single** | €9 | 1 generation, 1 year scenarios |
| **Lifetime** | €99 | Unlimited generations, 1-3 year scenarios |
| **Lifetime+** | €299 | Unlimited, 1-5 years, HR domain |
| **Pro** | €19/mo | Unlimited, 1-5 years, HR domain, 1 seat |
| **Team** | €49/mo | Unlimited, 1-5 years, HR domain, 5 seats |

### Charity Component

**3% of all revenue donated to hoki.help** (Children's hospice Austria)

- Live counter on website shows cumulative donations
- Authenticity signal and trust builder
- Differentiator: "Buy software, help children"

---

## Business Model

### Revenue Streams

1. **One-time purchases** - Single, Lifetime, Lifetime+
2. **Subscriptions** - Pro, Team (recurring monthly)
3. **Future: Enterprise** - Custom pricing for large organizations

### Key Metrics

| Metric | Target |
|--------|--------|
| Landing → Example Click | Baseline + 50% |
| Example → Generation | Baseline + 30% |
| Generation → Purchase | Baseline + 20% |
| Bounce Rate | Baseline - 20% |

---

## Competitive Landscape

### Direct Alternatives

| Competitor | Their Approach | Our Advantage |
|------------|----------------|---------------|
| ChatGPT/Claude | Generate data via prompts | Numbers reconcile |
| Mockaroo/Faker | Random data generation | Event-based logic, not random |
| Excel templates | Manual creation | 100x faster |
| Production data | Copy/anonymize real data | No compliance risk |

### Positioning Statement

> "For BI consultants and software vendors who need realistic demo data, tellingCube is a synthetic data generator that produces mathematically consistent business scenarios. Unlike AI tools that hallucinate numbers or random generators that produce obvious garbage, tellingCube's event-sourcing architecture guarantees that Finance, Sales, and HR views always reconcile—because they derive from the same transaction stream."

---

## Implementation Phases

### Phase 1: Copy/Messaging (This Week)
- Landing page new messaging
- Chart labels update
- Navigation update
- Meta/SEO update

### Phase 2: Auth + Dashboard (1-2 weeks)
- Magic Link authentication
- User dashboard
- Scenario history
- Tier display

### Phase 3: Async + Pricing (1 week)
- QStash integration
- New Stripe products
- Email notifications
- Progress tracking

### Phase 4: New Features (1-2 weeks)
- HR Domain
- Multi-year logic (1/3/5 years)
- hoki.help counter
- Coupon system

---

## Success Criteria

### Launch Success

1. **Conversion rate improvement** after repositioning
2. **New audience acquisition** (BI consultants, software vendors signing up)
3. **Reduced confusion** in user feedback
4. **Social proof accumulation** (testimonials, case studies)

### Product Success

1. **Mathematical consistency** - 100% reconciliation pass rate
2. **Generation speed** - <5 minutes for 90% of requests
3. **User satisfaction** - "Realistic" rating from beta testers

---

## Risk Factors

| Risk | Mitigation |
|------|------------|
| New positioning doesn't resonate | A/B test messaging, measure conversion |
| Technical issues at scale | Phased rollout, feature flags |
| Competitor copies approach | Speed to market, community building |
| IBCS audience feels abandoned | Maintain IBCS as feature, communicate clearly |

---

## Document References

| Document | Purpose |
|----------|---------|
| `docs/tellingcube-strategic-briefing.md` | Strategic positioning decision |
| `docs/tellingcube-repositioning-strategy.md` | Detailed implementation plan |
| `docs/prd.md` | Technical requirements (to be updated) |
| `docs/project-context.md` | Agent coding guidelines |

---

**Document Control**

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 2.0 | 2026-01-21 | Mary (BA) | Complete repositioning from IBCS to Data Reconciliation |

---

*This Product Brief supersedes all previous positioning guidance. All agents should align their work with this document.*
