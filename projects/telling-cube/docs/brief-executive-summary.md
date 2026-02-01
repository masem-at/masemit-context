# tellingCube Project Brief - Executive Summary

**Version:** 1.1 | **Date:** 2025-11-14 | **Status:** Ready for Execution | **Update:** Market Size Validated (15K IBCS members)

---

## The Opportunity

**tellingCube** is an AI-powered platform that generates mathematically consistent, multi-departmental business datasets in minutes instead of hours. The core innovation: an event-sourced architecture where Sales, Finance, HR, and Controlling views all derive from a single stream of business events—guaranteeing cross-departmental reconciliation that no competitor offers.

**Target Market:** IBCS© professionals (15,000-member community), business reporting trainers, university professors, and strategy consultants who spend several hours per workshop/course creating realistic demo datasets.

**Market Size:** ✅ **VALIDATED** - IBCS Association has 15,000 members (confirmed Nov 2024). Estimated 1,500-2,250 trainers/consultants = addressable market for MVP.

**Market Position:** Blue ocean opportunity—no Priority 1 competitors identified. Current alternatives (Excel, ChatGPT, BI samples) all suffer from consistency gaps or prohibitive costs.

---

## The Problem

- **Trainers:** Spend 3-4 hours per workshop creating demo data; reuse stale datasets or risk inconsistent numbers during live presentations
- **Professors:** Recycle outdated textbook examples; students recognize the same "bakery from 1995" semester after semester
- **Consultants:** Junior staff waste €30,000-€60,000/year in billable time fabricating demo data for client pitches

**Pain validation status:** Early conversations indicate acute pain; systematic validation planned for Weeks 2-4 (10-15 user interviews).

---

## The Solution

**MVP (2-4 weeks):**
- 3 preset scenarios (Bakery, Hotel, Tech Startup)
- 2 departmental views (Sales + Finance)
- 1 year of monthly data
- Event-sourced generation via Claude API
- Automated consistency validation
- Ephemeral generation (no user accounts)
- Stripe checkout for founding members

**Key Differentiator:** "One economic reality, multiple departmental perspectives"—when Finance shows €50K payroll, HR shows the exact employees and hours that generated that cost.

**Tech Stack:** Next.js + Vercel + NeonDB + Claude API + Stripe

---

## Business Model

**Founding Member Lifetime Pricing (MVP Phase):**
- Supporter S: €29 (1 seat)
- Supporter M: €99 (1 seat, premium scenarios)
- Supporter L: €299 (5 seats)
- Team Plus: €599 (20 seats)
- Department Partner: €999 (unlimited seats within department)

**Revenue Targets:**
- Week 4: First 5 paying members
- Week 8: 20+ members (€2,000-€6,000 revenue)
- Week 12: 100-300 members (€10,000-€45,000 revenue) ✅ **= 0.67-2% conversion from 15K IBCS community (REALISTIC)**

**Unit Economics:** €100-€150 target ARPU; €0.50-€2 COGS per generation; **<€5 CAC target** (via IBCS community post, not paid ads)

**Future Model:** Enterprise SaaS ($49-$499/month) starting Month 6-12; founding members grandfathered forever.

---

## Go-to-Market

**Primary Channel:** IBCS© Association (15,000 members) via partnership with IBCS Institute

**Launch Strategy:** Self-hosted early access (Kickstarter-style) with limited-time lifetime pricing

**Strategic Assets:**
- **Dr. Jürgen Faisst (IBCS Institute CEO):** Direct access to 15,000-member community; partnership pathway (#1 GTM asset)
- Brother (controller, 250-300 person manufacturing company): Data quality validation

**Marketing Timeline:**
- **Week 2: Jürgen Faisst strategy call** (validate trainer pain, request 3 certified consultant intros)
- Weeks 2-3: User interviews (3-5 IBCS certified consultants/trainers)
- Week 4: Soft launch to personal network + landing page test
- **Week 5: IBCS community launch** (via Jürgen post to 15K members, if partnership secured)
- Weeks 5-8: Direct outreach to high-engagement LinkedIn contributors

---

## Critical Risks & Mitigation

**🔴 CRITICAL RISKS:**
1. **AI quality insufficient** → Brother reviews Week 1-2; 3-5 finance professionals validate before launch
2. **Pain point not acute** → User interviews validate willingness to pay; ROI-focused messaging

**⚠️ HIGH RISKS:**
3. **Event-sourced architecture doesn't scale** → Performance testing Week 2-3; clear migration path to normalized schema
4. ~~**Market size insufficient**~~ → ✅ **RISK REMOVED** (15K IBCS members confirmed; 1,500-2,250 TAM validated)
5. **Timeline slips beyond 4 weeks** → Ruthless scope discipline; buffer week included; accept "good enough" for MVP
6. **Jürgen relationship doesn't convert to community access** → Fallback to direct LinkedIn outreach (slower but viable)

**Mitigation Summary:** Every risk has explicit owner (Mario), mitigation plan, and contingency approach.

---

## Success Criteria & Go/No-Go Gates

**Week 2 (Technical PoC):**
- ✅ GO: Event-sourced data reconciles; brother confirms realism; generation <5 min
- ❌ NO-GO: Can't achieve consistency OR data quality unacceptable

**Week 4 (Market Validation):**
- ✅ GO: Jürgen confirms trainer pain + intro to 3 certified consultants + 50+ email signups + willingness to pay validated
- 🟡 PIVOT: Jürgen not responsive → Fall back to LinkedIn outreach (slower path)
- ❌ NO-GO: <50% confirm pain OR <20 signups OR no willingness to pay

**Week 8 (Early Traction):**
- ✅ GO: 20+ paying members; positive feedback; <€5 CAC
- 🟡 PIVOT: 5-19 members → adjust pricing/positioning but continue
- ❌ NO-GO: <5 members → fundamental assumption broken

**Week 12 (Product-Market Fit Signal):**
- ✅ SCALE: 100+ members; NPS >40; 60%+ active users
- 🟡 ITERATE: 50-99 members → solid foundation, refine before Alpha
- ❌ REASSESS: <50 members → validate assumptions, consider pivot

---

## Resource Constraints

**Budget:** €100-150 MVP development; €300-1,100/month operational (covered by 3-10 founding members)

**Timeline:** 2-4 weeks MVP (20-30 hours/week = 40-120 total hours)

**Team:** Solo founder (Mario) with validation support (brother as controller, IBCS Institute relationship)

**Constraint Implication:** Must reach revenue breakeven quickly; ruthless scope discipline required; customer support must scale with solo founder.

---

## Key Assumptions to Validate

1. ~~**Market size sufficient**~~ (1,000+ reachable customers) → ✅ **VALIDATED** (15K IBCS members, 1,500-2,250 trainers)
2. **Pain acute enough to drive purchase** (not just "nice to have") → Week 2-3 user interviews
3. **Multi-view consistency valued** (users care about reconciliation) → Beta tester feedback Week 4-6
4. **€100-€150 ARPU realistic** (healthy tier mix) → First 20-50 customer analysis
5. **Event-sourced architecture delivers quality** (technical feasibility) → Week 1-2 PoC with brother review
6. **Solo founder can support 100-300 customers** (sustainable workload) → Support ticket tracking, automation

**Validation Approach:** Each assumption has explicit validation method, timeline, and decision point.

---

## Next Immediate Actions (Week 1)

**Day 1-2:** Set up dev environment (Next.js, Vercel, NeonDB, Claude API); implement database schema

**Day 3-5:** Build minimal event generator; create Sales + Finance views; **brother reviews for realism**

**Success Criteria:** Development operational; AI generates plausible bakery scenario; views reconcile perfectly; brother confirms "data looks realistic"

**Go/No-Go:** If successful → proceed to full MVP build; if quality issues → iterate prompts; if blockers → extend 1 week

---

## Strategic Vision (3-5 Years)

**North Star:** tellingCube becomes the industry standard for synthetic business data generation

1. **IBCS© Community's Platform of Choice** (70%+ certified trainers use tellingCube)
2. **Educational Standard** (500+ universities globally)
3. **Consultant's Productivity Tool** (10,000+ monthly users at McKinsey/BCG/Bain tier)
4. **Sustainable Business** (€500K+ ARR, 5,000 users, profitable without funding)

---

## Bottom Line

**What:** AI-powered synthetic business data with guaranteed multi-view consistency

**Why:** Saves trainers/professors/consultants hours per demo dataset; no competitor offers true cross-departmental reconciliation

**Who:** Business reporting professionals (IBCS© community primary), university educators, strategy consultants

**How:** Event-sourced architecture; self-hosted early access; founding member lifetime pricing

**When:** MVP in 2-4 weeks; soft launch Week 4; 100-300 members by Week 12

**Risk:** Honest assessment with 11 risks identified and mitigated; clear go/no-go gates at Weeks 2, 4, 8

**Validation:** Brother (controller) validates quality; 10-15 user interviews validate pain/willingness to pay; landing page test validates channel

---

**Status:** Strategic planning complete. All assumptions, risks, and plans documented in full Project Brief (1,900+ lines). Ready to execute Week 1 Day 1.

🚀 **Time to build.**
