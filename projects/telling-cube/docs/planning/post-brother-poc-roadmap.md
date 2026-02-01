# Post-Brother PoC Roadmap
## Multi-Agent Strategic Planning Session

**Date:** 2025-11-21 (Original) | **Updated:** 2025-12-14
**Session Type:** Planning Round (All Key Roles)
**Context:** tellingCube MVP is 100% complete. Brother validation complete (GO with changes).

> **Status Update (2025-12-14):**
> - ✅ Brother PoC validation: COMPLETE (2025-12-08) - GO with changes
> - ✅ Scenario 3 issues: RESOLVED via new two-phase architecture
> - ✅ Architecture refactored: Size × Sector matrix (9 scenarios)
> - ✅ Launch prep: Demo video, YouTube metadata, launch cookbook complete
> - ✅ Faisst preview: Email sent to IBCS Institute CEO
> - 🔲 Pending: Domain connection to Vercel, then LAUNCH

---

## 🎯 Session Goal

Define the **post-PoC strategy** across 3 scenarios:
1. **Brother says GO** → Launch immediately
2. **Brother says GO with changes** → Fix then launch
3. **Brother says NO-GO** → Pivot strategy

Each role presents their perspective and recommended next steps.

---

## 🏗️ **Architect (Winston): Technical Roadmap**

### If Brother Says GO ✅

**Immediate (Week 1):**

1. **Production Hardening (Priority: HIGH)**
   - Add rate limiting (Upstash Redis setup: 2 hours)
   - Configure Sentry error tracking (1 hour)
   - Set up Vercel Analytics (15 min)
   - Enable uptime monitoring (UptimeRobot: 15 min)

2. **Domain & Email (Priority: HIGH)**
   - Verify noreply@tellingcube.com in Resend (30 min)
   - Update Stripe webhook URLs to production (15 min)
   - Final smoke test with live Stripe mode (1 hour)

3. **Performance Optimization (Priority: MEDIUM)**
   - Implement API response caching (Next.js ISR: 2 hours)
   - Add database query monitoring (Prisma metrics: 1 hour)
   - Bundle size optimization (lazy load Recharts: 1 hour)

**Short-term (Weeks 2-4):**

4. **Scalability Prep**
   - NeonDB connection pooling tuning (if >100 concurrent users)
   - Claude API fallback to OpenAI (redundancy: 4 hours)
   - Background job queue for generation (if >500 scenarios/day: 8 hours)

5. **Technical Debt Cleanup**
   - Resolve 4 TODOs in codebase (2 hours)
   - Add unit tests for validation logic (4 hours)
   - E2E tests with Playwright (8 hours)

**Long-term (Months 2-3):**

6. **Enterprise Features**
   - White-label API access (REST endpoints: 2 weeks)
   - Data export (CSV/Excel: 1 week)
   - User accounts & saved scenarios (Auth.js integration: 2 weeks)

### If Brother Says GO with Changes ⚠️

**Response Plan:**

**Category 1: IBCS Styling Issues**
- Chart color corrections (1-2 hours)
- Message-driven headline adjustments (1 hour)
- Table formatting fixes (2 hours)

**Category 2: Data Realism Issues**
- Prompt engineering refinements (4-6 hours per scenario)
- Validation rule adjustments (2 hours)
- Re-test with brother (1 session)

**Category 3: UI/UX Polish**
- Layout tweaks (2-4 hours)
- Mobile responsive fixes (2-4 hours)
- Accessibility improvements (4 hours)

**Estimated Fix Time:** 1-3 days depending on severity

### If Brother Says NO-GO ❌

**Pivot Options:**

**Option A: AI Model Switch**
- Try Claude Opus instead of Haiku (higher quality, 4x cost)
- Test OpenAI GPT-4o (different prompt style)
- Estimated: 1 week to re-engineer prompts

**Option B: Simplify Data Complexity**
- Generate quarterly instead of monthly (less detail)
- Reduce event types from 7 to 4 (focus on core financials)
- Estimated: 3-5 days to redesign

**Option C: Hybrid Approach**
- Keep AI for master data, use deterministic formulas for events
- Pros: Perfect consistency, faster generation
- Cons: Less "realistic" variability
- Estimated: 2 weeks to rebuild

---

## 📈 **Product Manager (PM): Product Roadmap**

### If Brother Says GO ✅

**Phase 1: Soft Launch (Weeks 1-4)**

**Goals:**
- Acquire 20-50 founding members
- Validate willingness to pay
- Gather feedback for iteration

**Launch Checklist:**
1. **Pre-Launch (Week 1)**
   - Write launch announcement (LinkedIn, email)
   - Prepare demo video (2-min Loom walkthrough)
   - Set up customer support email (support@tellingcube.com)
   - Create founder's tier (€0 for first 10 testers?)

2. **Launch Day (Week 2, Day 1)**
   - LinkedIn post (personal + company page)
   - Email to warm leads (professors, trainers, consultants)
   - ProductHunt launch? (optional, high visibility)
   - Monitor Sentry/Vercel for errors

3. **Post-Launch (Weeks 2-4)**
   - Daily monitoring: generation success rate, payment conversions
   - Weekly user interviews (5-10 founding members)
   - Iterate on pricing/messaging based on feedback

**Phase 2: Public Beta (Months 2-3)**

**Feature Priorities (based on user feedback):**

**Tier 1: Must-Have**
- Custom scenario inputs (user-defined industry/parameters)
- Data export (CSV/Excel)
- User accounts (save/retrieve scenarios)

**Tier 2: High-Value**
- Additional views (HR, Controlling)
- Multi-year scenarios (2-3 years)
- Comparison mode (side-by-side scenarios)

**Tier 3: Nice-to-Have**
- AI-generated narratives (explain the data)
- API access (white-label integration)
- Collaboration features (team sharing)

**Phase 3: Scale (Months 4-6)**

**Growth Levers:**
- Content marketing (IBCS blog posts)
- Partnerships (business schools, training providers)
- Referral program (20% commission for affiliates?)
- Enterprise sales (direct outreach to Fortune 500)

### If Brother Says GO with Changes ⚠️

**Modified Launch Timeline:**

- Fix critical issues (1-3 days)
- Re-validate with brother (1 session)
- Delay launch by 1 week
- Everything else proceeds as planned

**Risk Mitigation:**
- Communicate delay to warm leads ("polishing for you")
- Use extra time to prep launch materials

### If Brother Says NO-GO ❌

**Pivot Decision Tree:**

**Option A: Narrow Target Market**
- If data is "good enough for training" but not "controller-grade"
- Target: University professors only (lower quality bar)
- Pricing: Lower tiers (€9-€49)

**Option B: Different Use Case**
- Pivot from "demo data" to "learning tool" (interactive scenarios)
- Add educational content (explain P&L, cash flow)
- Target: Business students, not professionals

**Option C: Consult Further Experts**
- Get 5 more controllers to test (maybe brother is too strict?)
- If 3/5 say "good enough", launch with caveats
- If 0/5 say good, major rework needed

---

## 🎨 **UX Expert: User Experience Roadmap**

### If Brother Says GO ✅

**Immediate UX Improvements (Weeks 1-2):**

1. **Onboarding Flow**
   - Add 30-second explainer video on landing page
   - First-time user tooltip tour (Intro.js or similar)
   - "How to interpret IBCS charts" guide

2. **Results Page Enhancements**
   - Add "Share" button (generate shareable link)
   - Print-friendly view (remove nav, optimize for PDF)
   - "Download scenario summary" (1-page PDF report)

3. **Mobile Experience**
   - Test on 5 different devices (iOS/Android)
   - Optimize chart touch interactions
   - Add swipe gestures for tab switching

**Short-term UX Research (Weeks 3-4):**

4. **User Testing Sessions**
   - 5 moderated sessions with founding members
   - Track: time to first scenario, confusion points, aha moments
   - Tools: Hotjar heatmaps, Fullstory session recordings

5. **A/B Testing**
   - Landing page headline variants (3 options)
   - Pricing tier naming (Supporter vs Starter?)
   - CTA button copy ("Generate Scenario" vs "Try Demo")

**Long-term UX Vision (Months 2-3):**

6. **Advanced Interactions**
   - Interactive charts (click bar → drill down to events)
   - Scenario comparison mode (side-by-side 2 scenarios)
   - Custom color themes (let users pick brand colors)

7. **Personalization**
   - "My Scenarios" dashboard
   - Favoriting/tagging scenarios
   - Recently viewed scenarios

### If Brother Says GO with Changes ⚠️

**UX-Driven Fixes:**

**If IBCS Issues:**
- Work with brother to document exact violations
- Create IBCS checklist component-by-component
- Iterate until 5/5 compliance score

**If Usability Issues:**
- Simplify navigation (fewer clicks)
- Add more contextual help (tooltips)
- Improve error messages (less technical jargon)

### If Brother Says NO-GO ❌

**UX Pivot:**

**Option A: Guided Mode**
- Add step-by-step wizard ("What industry?" → "How many employees?" → etc.)
- Give users control over data generation (less black box)

**Option B: Educational Focus**
- Redesign as "learn business metrics" tool
- Add explanations alongside charts ("What is COGS?")
- Gamify learning (quizzes, badges)

---

## 🧪 **QA Engineer: Quality Roadmap**

### If Brother Says GO ✅

**Immediate QA Actions (Week 1):**

1. **Production Monitoring Setup**
   - Sentry alert rules (error rate >5%)
   - Vercel uptime monitoring (ping every 5 min)
   - Database performance dashboard (query times)

2. **Regression Test Suite**
   - Automate brother's test scenarios (Playwright)
   - Run on every deployment (CI/CD integration)
   - Track: generation success rate, validation pass rate

3. **Load Testing**
   - Simulate 50 concurrent users (k6 or Locust)
   - Test: API rate limits, database connection pool
   - Goal: Identify breaking point before real traffic

**Ongoing QA (Weeks 2-4):**

4. **User-Reported Bug Tracking**
   - Set up public issue tracker (GitHub Issues or Linear)
   - Triage process (P0: fix in 24h, P1: 3 days, P2: 1 week)
   - Weekly bug review meeting

5. **Data Quality Monitoring**
   - Sample 10% of generated scenarios daily
   - Run validation checks (reconciliation, realism heuristics)
   - Alert if validation pass rate drops below 95%

**Long-term QA Strategy (Months 2-3):**

6. **Test Automation**
   - Unit tests: 80% coverage target
   - Integration tests: All API endpoints
   - E2E tests: 5 critical user flows

7. **Performance Benchmarks**
   - Track: P50, P95, P99 generation times
   - Goal: P95 < 90s (current: ~70s)
   - Alert if degradation >20%

### If Brother Says GO with Changes ⚠️

**QA Response:**

1. **Bug Fix Validation**
   - Create test cases for each issue brother found
   - Verify fixes with before/after screenshots
   - Re-run full regression suite

2. **Brother Re-Test**
   - Schedule follow-up session (30 min)
   - Focus on fixed areas only
   - Document: "Issue X fixed, verified by [brother's name]"

### If Brother Says NO-GO ❌

**QA Pivot:**

**Root Cause Analysis:**
- Document all brother's findings (categorize)
- Identify: Is it AI quality? Prompts? Validation logic?
- Data-driven decision: Which pivot (A/B/C) addresses most issues?

**Re-Test Plan:**
- After pivot, run full test suite again
- Get 3 additional expert validators (not just brother)
- Set quality bar: "4/5 experts must approve"

---

## 💰 **Scrum Master (SM): Execution Roadmap**

### If Brother Says GO ✅

**Sprint Planning (2-Week Sprints):**

**Sprint 1 (Post-Launch Weeks 1-2): Stabilization**

| Story | Owner | Effort | Priority |
|-------|-------|--------|----------|
| Rate limiting (Upstash) | Dev | 3 SP | P0 |
| Sentry setup | Dev | 2 SP | P0 |
| Domain email verification | Dev | 1 SP | P0 |
| Stripe live mode switch | PM | 1 SP | P0 |
| Launch announcement | Marketing | 2 SP | P0 |
| User interview framework | UX | 2 SP | P1 |
| Load testing | QA | 3 SP | P1 |

**Total:** 14 story points (team capacity: ~15 SP/sprint)

**Sprint 2 (Weeks 3-4): Feedback Iteration**

| Story | Owner | Effort | Priority |
|-------|-------|--------|----------|
| Custom scenario inputs (design) | UX | 5 SP | P1 |
| Data export (CSV) | Dev | 5 SP | P1 |
| User accounts (planning) | Architect | 3 SP | P1 |
| A/B testing setup | Marketing | 2 SP | P2 |

**Total:** 15 story points

**Sprint 3+ (Month 2): Feature Expansion**
- Based on founding member feedback
- Prioritize using RICE scoring (Reach, Impact, Confidence, Effort)

### If Brother Says GO with Changes ⚠️

**Emergency Sprint:**

**Sprint 0.5 (Fix Week):**
- Daily standups (15 min)
- Brother's issues → Jira tickets (categorized P0/P1/P2)
- Fix P0 issues (blockers) in Days 1-2
- Fix P1 issues (critical) in Days 3-4
- Re-test Friday, launch Monday

**Velocity Impact:**
- Delays Sprint 1 by 1 week
- Deprioritize P2 tasks to catch up

### If Brother Says NO-GO ❌

**Pivot Sprint:**

**Week 1: Assessment & Decision**
- Monday: Emergency meeting (all stakeholders)
- Tuesday-Wednesday: Architect presents 3 pivot options (A/B/C)
- Thursday: Team votes on pivot direction
- Friday: Create new backlog for chosen pivot

**Week 2-3: Pivot Implementation**
- Depends on chosen pivot (1-2 weeks estimated)
- Daily standups to track progress
- End of Week 3: Re-test with brother + 2 other experts

---

## 📊 **Product Owner (PO): Backlog Prioritization**

### If Brother Says GO ✅

**Product Backlog (Next 3 Months):**

**Epic 4: Launch & Growth (Priority: P0)**
- Story 4.1: Stripe live mode activation ✅
- Story 4.2: Launch announcement (LinkedIn + email)
- Story 4.3: Customer support inbox setup
- Story 4.4: First 10 user interviews
- Story 4.5: Pricing optimization (based on feedback)

**Epic 5: Data Export & Sharing (Priority: P1)**
- Story 5.1: CSV export for Finance view
- Story 5.2: CSV export for Sales view
- Story 5.3: PDF report generation (1-page summary)
- Story 5.4: Shareable links (public view, no login)
- Story 5.5: Print-friendly layout

**Epic 6: Custom Scenarios (Priority: P1)**
- Story 6.1: Custom scenario input form (industry, employees, revenue)
- Story 6.2: Prompt engineering for custom scenarios
- Story 6.3: Validation for custom inputs (edge cases)
- Story 6.4: Save custom scenario templates

**Epic 7: User Accounts (Priority: P2)**
- Story 7.1: Auth.js integration (email/password)
- Story 7.2: "My Scenarios" dashboard
- Story 7.3: Save/delete scenarios
- Story 7.4: Team sharing (multi-user access)

**Epic 8: Additional Views (Priority: P2)**
- Story 8.1: HR View (headcount, payroll breakdown)
- Story 8.2: Controlling View (KPIs, variance analysis)
- Story 8.3: Operations View (procurement, inventory)

**Epic 9: API & Integrations (Priority: P3)**
- Story 9.1: REST API for scenario generation
- Story 9.2: API authentication (OAuth2)
- Story 9.3: Webhook support (notify on generation complete)
- Story 9.4: White-label embedding (iframe)

**Backlog Grooming:**
- Weekly refinement sessions (Wednesdays)
- Acceptance criteria written by PO
- Effort estimated by Dev + Architect
- Priority set by PM + PO

### If Brother Says GO with Changes ⚠️

**Backlog Adjustment:**

- Insert "Brother Feedback Fixes" epic at Priority P0
- Push all other epics down by 1 week
- Re-estimate completion dates

### If Brother Says NO-GO ❌

**Backlog Reset:**

- Archive current backlog (save for reference)
- Create new "Pivot" epic (based on chosen direction)
- Re-prioritize from scratch
- Timeline: Add 2-4 weeks to original plan

---

## 🎯 **Consensus Recommendations**

### **All Agents Agree:**

**If Brother Says GO:**
1. **Launch within 7 days** (don't overthink it)
2. **Start with soft launch** (warm leads only, no paid ads yet)
3. **Iterate based on feedback** (2-week sprints)
4. **Focus on stability first** (rate limiting, monitoring)
5. **Then add features** (custom scenarios, export)

**If Brother Says GO with Changes:**
1. **Fix within 3-5 days** (don't let scope creep)
2. **Re-validate with brother** (30-min session)
3. **Launch 1 week later** (communicate delay to leads)

**If Brother Says NO-GO:**
1. **Don't panic** (PoC failed, but product concept validated)
2. **Get 3 more expert opinions** (maybe brother is outlier?)
3. **If 3/4 experts say no → pivot** (choose Option A/B/C)
4. **Timeline: +2-4 weeks** (depending on pivot complexity)

---

## 📅 **Decision Timeline**

**Today (Nov 21):**
- ✅ Send testing guide to brother
- Schedule test session (when?)

**Brother's Test Session:**
- Duration: 45 min
- Outcome: GO / GO with changes / NO-GO

**If GO → Launch Timeline:**
- Day 1-2: Production hardening (rate limiting, Sentry)
- Day 3-4: Domain setup, Stripe live mode
- Day 5: Final smoke test
- Day 6: Launch announcement prep
- Day 7: **GO LIVE** 🚀

**If GO with changes → Fix Timeline:**
- Day 1: Triage issues (P0/P1/P2)
- Day 2-3: Fix P0 issues
- Day 4: Re-test with brother
- Day 5-6: Fix P1 issues (if any)
- Day 7: Launch prep
- Day 10: **GO LIVE** 🚀

**If NO-GO → Pivot Timeline:**
- Week 1: Assess + choose pivot direction
- Week 2-3: Implement pivot
- Week 4: Re-test with 3 experts
- Week 5: Launch (if approved)

---

## 🎤 **Individual Agent Closing Statements**

### Architect:
*"The technical foundation is solid. If brother approves, we're production-ready in 48 hours. If he wants changes, I need specifics - not vague 'make it better'. If he rejects, I advocate for Pivot Option A (better AI model) over starting from scratch."*

### PM:
*"This is a validation moment, not a perfection moment. If brother says 'good enough for demos', that's our go-ahead. We'll iterate based on real customer feedback, not one person's opinion. Launch fast, learn fast."*

### UX Expert:
*"Brother's feedback is gold - he's our target user proxy. If he finds the charts confusing, users will too. I'm prepared to redesign, but I want to understand WHY before jumping to solutions."*

### QA:
*"I'm confident in the system's stability. 95%+ success rate, all gates passed. Brother's testing the DATA QUALITY, not the software quality. Those are different concerns. We can fix data quality with prompt engineering in days, not weeks."*

### SM:
*"Team morale is high - we shipped 32 stories in 8 weeks. If brother says change course, I want a clear pivot plan before we start coding. No 'let's try stuff and see' - that kills momentum."*

### PO:
*"Brother's vote is important but not final. If he says no, I want 4 more expert testers before we pivot. User research 101: one data point is not a trend. That said, if he says go, we launch FAST."*

---

## ✅ **River's Action Items**

**Before Brother's Test:**
- [ ] Schedule test session (date/time?)
- [ ] Send him the testing guide (print or email?)
- [ ] Clarify: Do you want to be present during his test?

**After Brother's Test:**
- [ ] Document his feedback (use the guide's rating system)
- [ ] Share results with agents (call another Lagerfeuer)
- [ ] Make GO/CHANGE/NO-GO decision
- [ ] Agents execute chosen roadmap

---

**Session Complete.** All agents standing by for brother's verdict. 🔥

**Next milestone:** Brother's test session → GO/NO-GO decision → Execute roadmap

**Estimated time to launch (if GO):** 7 days
**Estimated time to launch (if CHANGE):** 10 days
**Estimated time to pivot (if NO-GO):** 3-4 weeks

---

**Version:** 1.0
**Contributors:** Architect, PM, UX Expert, QA, SM, PO
**Approved by:** River (BMad Orchestrator)