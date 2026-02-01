# Project Brief: tellingCube

**Document Version:** 1.0
**Created:** 2025-11-13
**Status:** In Progress

---

## Executive Summary

**tellingCube** is an AI-powered synthetic business data generation platform that creates mathematically consistent, multi-departmental datasets in minutes instead of hours. Unlike existing solutions that produce random or inconsistent data, tellingCube generates realistic business scenarios from a single event-sourced foundation, ensuring that Sales, Finance, HR, and Controlling views all reflect the same underlying economic reality.

**Primary Problem:** Based on early conversations with target users, business reporting trainers, consultants, and educators currently spend several hours manually creating realistic demo datasets for workshops, courses, and client presentations. Existing tools either generate random inconsistent data or require extensive manual reconciliation across departmental views.

**Target Market:** Initial focus on business reporting professionals (including those following IBCS© standards), university professors teaching business subjects, and strategy consultants who need believable demonstration scenarios.

**Key Value Proposition:** Generate 1 year of realistic, multi-view business data in minutes instead of hours, with guaranteed mathematical consistency across all departmental perspectives—enabling trainers and consultants to focus on teaching and analysis rather than data fabrication.

---

## Problem Statement

### Current State & Pain Points

Business reporting professionals—trainers, educators, and consultants—face a recurring challenge: they need realistic, believable business data for demonstrations, workshops, and training scenarios, but they cannot use real company data due to confidentiality constraints.

**The current manual process involves:**

1. **Inventing a business scenario** (industry, size, product mix, employee count)
2. **Creating foundational data** (employee roster, product catalog, customer list)
3. **Generating transactional events** (sales, purchases, payroll, expenses) for months or years
4. **Ensuring cross-departmental consistency** (revenue must match sales × price; payroll must align with headcount; expenses must reconcile)
5. **Formatting data** for specific reporting tools or dashboard templates

Based on user conversations, this process consumes **several hours per scenario**, with more complex multi-year, multi-department scenarios taking an entire day.

### Impact of the Problem

**For Business Trainers:**
- 3-4 hours of preparation time per workshop reduces capacity to serve more clients
- Reusing old datasets makes workshops feel stale and less engaging
- Inconsistent data creates awkward moments when dashboard numbers don't reconcile

**For University Professors:**
- Limited time to create diverse scenarios means students see the same "bakery example" semester after semester
- Outdated Excel files from textbooks fail to engage modern students
- Teaching "how to spot data inconsistencies" becomes harder when example data is itself inconsistent

**For Strategy Consultants:**
- Junior consultants waste billable hours ($150-$500/hour) fabricating demo data
- Client presentations suffer when scenarios aren't tailored to the client's industry
- Pitch decks with generic data fail to impress prospects

**Quantified Impact:**
- A trainer running 20 workshops/year spends **60-80 hours annually** just on data preparation
- A consulting firm with 5 consultants creating monthly client demos wastes **$30,000-$50,000/year** in billable time
- University departments recycle the same 3-5 datasets across multiple courses, reducing educational variety

### Why Existing Solutions Fall Short

From the competitive analysis, current alternatives include:

1. **Manual Excel/Google Sheets Creation**
   - Time-consuming (hours per scenario)
   - Error-prone (formulas break, inconsistencies creep in)
   - No multi-view consistency guaranteed

2. **ChatGPT / General AI Prompting**
   - Generates data in isolation (Sales data doesn't reconcile with Finance data)
   - No structural coherence across departments
   - Requires extensive manual validation and cleanup

3. **BI Tool Sample Datasets**
   - Generic, unrealistic patterns (too perfect, no business narrative)
   - Fixed scenarios (can't customize industry, size, timeframe)
   - Not designed for training/education use cases

4. **Business Simulation Platforms**
   - Expensive enterprise tools ($5,000-$50,000/year)
   - Designed for decision-making practice, not data generation
   - Overkill for trainers who just need realistic datasets

5. **Anonymized Real Data**
   - Legal and compliance risks (even anonymized data may violate confidentiality)
   - Hard to obtain for specific industries
   - Cannot be customized or adapted to teaching scenarios

**None of these solutions offer the combination of:**
- AI-powered speed (minutes not hours)
- Multi-departmental mathematical consistency
- Customizable scenarios (industry, size, timeframe)
- Professional-grade realism
- Affordable pricing for individual trainers/professors

### Urgency & Importance

**Why Now?**

1. **AI Enablement:** Modern LLMs (Claude, GPT-4) make sophisticated synthetic data generation feasible for the first time at reasonable cost
2. **Remote Training Explosion:** Post-pandemic shift to online workshops and virtual training increases demand for shareable, reproducible demo scenarios
3. **IBCS© Standards Growth:** Growing adoption of international reporting standards creates demand for compliant demo data
4. **Consultant Efficiency Pressure:** Economic uncertainty drives consulting firms to optimize billable time and reduce non-billable overhead
5. **Education Technology Gap:** Universities seeking engaging, modern teaching tools to compete for students

**The window is open:** Solo founders can now build AI-powered tools that would have required teams of engineers 3 years ago. First-mover advantage in this niche market is significant.

---

## Proposed Solution

### Core Concept & Approach

tellingCube solves the synthetic data problem through **event-sourced architecture**: instead of generating random numbers for each department independently, it creates a single stream of realistic business events, then transforms those events into department-specific views.

**The fundamental insight:** Every department in a company observes the same economic reality through different lenses. When an employee is hired, HR sees a new headcount, Finance sees increased payroll expense, and Controlling sees a variance from the hiring plan. These aren't separate data points—they're different perspectives on one event.

**How it works:**

1. **User provides a simple prompt:** "Generate data for a bakery with 10 employees over 1 year"

2. **AI generates 7 types of business events:**
   - Employee Lifecycle Events (hired, promoted, terminated)
   - Operational Work Performed (hours worked, units produced)
   - Sales Events (orders, deliveries, revenue recognition)
   - Cash Movements (payments received/made, payroll)
   - Procurement & Expenses (materials, licenses, services)
   - Asset Lifecycle Events (purchased, depreciated, sold)
   - Planning & Budgeting Events (planned revenue, headcount, costs)

3. **Events are dimensioned across 5 axes:**
   - Time (when)
   - Organization (which unit/department)
   - Product/Service/Project (what)
   - Counterparty (who: customer/supplier/employee)
   - Asset/Resource (which resource)

4. **System generates multiple consistent views:**
   - **Sales View:** Revenue by product, customer acquisition, sales trends
   - **Finance View:** P&L, cash flow, balance sheet implications
   - **HR View:** Headcount, payroll, working hours
   - **Controlling View:** Plan vs. actual, variance analysis, KPIs

**Mathematical guarantee:** Because all views derive from the same event stream, they automatically reconcile. If Finance shows €50,000 payroll expense, HR will show the exact employees and hours that generated that cost.

### Key Differentiators from Existing Solutions

**vs. Manual Excel Creation:**
- **Speed:** Minutes instead of hours
- **Consistency:** Built-in reconciliation across all views
- **Scalability:** Generate 1 year or 5 years with equal ease

**vs. ChatGPT / General AI:**
- **Structural coherence:** Single event stream ensures mathematical consistency
- **Multi-view support:** Purpose-built for departmental perspectives
- **Business logic:** Understands causal relationships (hire employee → payroll increases)

**vs. BI Sample Datasets:**
- **Customizable:** Choose industry, size, timeframe, complexity
- **Realistic narratives:** AI generates plausible business stories, not random patterns
- **Training-focused:** Designed specifically for education and consulting use cases

**vs. Business Simulations:**
- **Lightweight:** Just data generation, not full simulation platform
- **Affordable:** $29-$999 one-time vs. $5K-$50K/year
- **Fast:** Instant generation vs. multi-step simulation runs

**vs. Anonymized Real Data:**
- **Zero compliance risk:** Fully synthetic, no confidentiality concerns
- **Unlimited customization:** Tailor to specific teaching scenarios
- **Reproducible:** Share scenarios with colleagues or students freely

### Why We Believe This Solution Can Succeed

**1. Architectural Advantage**

The event-sourced foundation provides a structural approach to guaranteeing cross-departmental consistency. Early architectural validation suggests this is feasible with modern database and API technology.

**2. AI-Powered Timing**

Modern LLMs (Claude Sonnet, GPT-4) demonstrate capability to understand complex business logic. Initial testing will validate whether they can generate plausible event sequences at the quality level required.

**3. Focused Market Entry**

Early conversations with business reporting professionals (including IBCS© community members) indicate acute pain around demo data creation. Systematic validation will confirm pain severity and willingness to pay.

**4. Founding Member Model**

Self-hosted early access with lifetime pricing follows patterns from successful indie SaaS products (Plausible Analytics, AppSumo deals). This approach:
- Generates immediate revenue for validation
- Builds community of advocates who provide feedback
- Creates urgency without platform fees
- De-risks development by validating demand before scaling

**5. Executable Scope**

Aggressive scope discipline enables rapid prototyping:
- **Prototype MVP:** 3 preset scenarios (Bakery, Hotel, Tech Startup), 2 views (Sales + Finance), 1 year monthly data
- **No user accounts:** Ephemeral generation, zero friction
- **Simple tech stack:** Next.js + Vercel + NeonDB + Claude API
- **Founding member checkout:** Stripe integration, immediate revenue

### Validation Plan

#### **What We Know (Validated)**

✅ **Market Gap Confirmed:**
- Competitive analysis shows no direct competitors offering multi-view consistent synthetic data
- Existing alternatives (manual Excel, ChatGPT, BI samples) have documented limitations

✅ **Technical Feasibility:**
- Event-sourced architecture is proven pattern in software engineering
- LLMs demonstrate capability for complex business reasoning
- Tech stack components (Next.js, Vercel, NeonDB) are production-ready

✅ **Market Access:**
- Established relationship with IBCS Institute CEO (Dr. Jürgen Faisst)
- Permission to engage IBCS© community via LinkedIn #IBCS
- Access to business reporting professional networks

#### **What We Believe (Hypotheses to Validate)**

⚠️ **Hypothesis 1: Event-Sourced Architecture Delivers Quality**
- *Belief:* Single event stream will generate realistic, consistent multi-view data
- *Risk:* Technical complexity, AI quality issues, performance bottlenecks
- *Validation approach:* Build minimal prototype with 1 scenario, 2 views; verify reconciliation

⚠️ **Hypothesis 2: Target Market Has Acute Pain**
- *Belief:* Trainers/professors/consultants spend several hours creating demo data and will pay to save time
- *Risk:* Pain might be overstated; users might reuse old datasets; time savings not valued enough
- *Validation approach:* Conduct 10-15 structured user interviews across all three segments

⚠️ **Hypothesis 3: Pricing is Appropriate**
- *Belief:* $29-$999 lifetime pricing will attract founding members and generate meaningful revenue
- *Risk:* Price could be too high (no buyers) or too low (leaves money on table; damages credibility)
- *Validation approach:* Landing page A/B testing, direct outreach with pricing discussion

⚠️ **Hypothesis 4: IBCS© Community is Viable Channel**
- *Belief:* IBCS© LinkedIn community will engage with tellingCube messaging and convert to customers
- *Risk:* Community might be too small, not engaged, or not buyers of tools
- *Validation approach:* Measure community size, test engagement with value-adding posts, track conversion

#### **Validation Timeline & Resources**

**Week 1-2: Technical Proof of Concept**
- Build minimal event generator for bakery scenario (1 year, monthly data)
- Generate Sales + Finance views from same event stream
- **Validation resource:** Brother (controller, 250-300 person manufacturing company) reviews output for realism
- **Success criteria:** Data reconciles perfectly; controller confirms "looks believable"

**Week 2-3: Market Validation**
- Conduct 10-15 user interviews (5 trainers, 5 professors, 5 consultants)
- Questions: Time spent on demo data? Current process? Willingness to pay?
- **Validation resource:** LinkedIn outreach to IBCS© community, university business school contacts
- **Success criteria:** At least 70% confirm pain point; at least 50% indicate willingness to pay $50-$300

**Week 3-4: Channel & Pricing Validation**
- Create landing page with 3 pricing tiers
- Run LinkedIn ad campaign ($100-200 budget) targeting IBCS© community
- Track email signups and tier interest
- **Success criteria:** 50+ email signups; clear tier preference emerges; <$5 cost per lead

**Week 4-6: Prototype Build & Beta Testing**
- Build 3 preset scenarios with 2 views each
- Deploy to Vercel with Stripe checkout
- Invite 10-20 early users for feedback
- **Validation resource:** Brother tests manufacturing scenario quality
- **Success criteria:** First 5 paying founding members; positive feedback on data quality

#### **Go/No-Go Decision Points**

**After Technical PoC (Week 2):**
- ✅ GO if: Data reconciles; controller confirms realism; generation time <5 minutes
- ❌ NO-GO if: Can't achieve consistency; data quality unacceptable; major technical blockers

**After Market Validation (Week 3):**
- ✅ GO if: >70% confirm pain; >50% willing to pay; clear value proposition resonates
- ❌ NO-GO if: Pain not validated; no willingness to pay; value prop doesn't resonate

**After Channel/Pricing Test (Week 4):**
- ✅ GO if: >50 email signups; <$5 per lead; clear tier preference
- ❌ PIVOT if: Low signups but positive interviews → change channel strategy
- ❌ NO-GO if: No engagement across all channels

### High-Level Vision

**Short-term (Prototype → Alpha, Weeks 1-8):**
- Prove the multi-view consistency concept works technically
- Validate that early adopters will pay for this solution
- Achieve 50-100 founding members

**Medium-term (Beta → MVP, Months 3-6):**
- Add HR and Controlling views (4 total departmental perspectives)
- Expand scenario library (10+ industries)
- Enable custom prompts (user-defined scenarios)
- Introduce export functionality (CSV, Excel, JSON)

**Long-term (Post-MVP, Months 6-12):**
- AI-generated business narratives (story → events)
- Scenario variant comparison (base vs. shock vs. improvement)
- Template marketplace (IBCS©-inspired dashboards)
- Enterprise SaaS model ($2,500-$20,000/year for large organizations)
- Explore IBCS© certification partnership

---

## Target Users

### Primary User Segment: Business Reporting Trainers

**Demographic/Firmographic Profile:**
- Independent trainers or small training firms (1-5 employees)
- Specialization: Business reporting, financial analysis, controlling, data visualization
- Many hold IBCS© certification or follow IBCS© standards in their training materials
- Geographic distribution: DACH region (Germany, Austria, Switzerland) primarily, with growing international presence
- Age range: 35-55 years old
- Annual revenue: €80,000-€250,000 (solo trainers to small firms)

**Current Behaviors & Workflows:**
- Conduct 15-30 workshops per year for corporate clients
- Workshop topics: IBCS©-compliant reporting, dashboard design, financial storytelling, controlling fundamentals
- Preparation time: 3-5 hours per workshop, with 1-2 hours spent on demo data creation
- Data creation process:
  1. Reuse old datasets (60% of time) - leading to staleness
  2. Manually create new scenarios in Excel (30% of time) - time-consuming
  3. Adapt client data with anonymization (10% of time) - risky and limited
- Tools used: Excel, PowerPoint, various BI tools (Power BI, Tableau, Qlik), IBCS© certified software
- Pain point frequency: Every 2-3 workshops require fresh data scenarios

**Specific Needs & Pain Points:**
- **Time pressure:** Billable hours lost to non-value-adding data preparation
- **Scenario variety:** Clients expect industry-relevant examples (can't show bakery to automotive client)
- **Data quality:** Inconsistent numbers create awkward questions during live workshops ("Why doesn't the headcount match payroll?")
- **Scalability:** Growing client base demands more diverse scenarios
- **IBCS© compliance:** Data must support teaching international reporting standards (variance analysis, plan vs. actual, waterfall charts)
- **Shareability:** Need datasets students can take home and practice with

**Goals They're Trying to Achieve:**
- **Primary goal:** Deliver high-quality, engaging workshops that lead to repeat bookings and referrals
- **Efficiency goal:** Minimize non-billable preparation time to maximize revenue
- **Professional credibility:** Demonstrate expertise through realistic, industry-relevant examples
- **Scalability goal:** Serve more clients without proportional time increase
- **Teaching effectiveness:** Help students understand reporting concepts through believable data scenarios

**Willingness to Pay Indicators:**
- Expense software tools <€500/year easily (professional development budget)
- Value time savings highly (€100-€300/hour billable rate)
- Early conversations suggest strong interest in $99-$299 tier (Supporter M/L)

### Secondary User Segment: University Professors (Business/Finance/Controlling)

**Demographic/Firmographic Profile:**
- Teaching positions at universities, business schools, vocational colleges (Fachhochschulen)
- Subjects: Business administration, finance, controlling, accounting, business analytics
- Class sizes: 30-150 students per semester
- Geographic: Global, with focus on European and North American institutions
- Age range: 30-65 years old
- Academic level: Lecturers, assistant professors, full professors

**Current Behaviors & Workflows:**
- Prepare course materials 2-4 weeks before semester start
- Create 3-8 case study scenarios per course
- Data creation process:
  1. Recycle textbook datasets (70% of time) - outdated, students recognize them
  2. Adapt previous semester's data (20% of time) - minor changes, still stale
  3. Manually create new scenarios (10% of time) - only when absolutely necessary
- Tools used: Excel, Google Sheets, LMS platforms (Moodle, Canvas), statistical software (R, SPSS)
- Budget constraints: Departmental software budget €1,000-€5,000/year shared across faculty

**Specific Needs & Pain Points:**
- **Student engagement:** Modern students demand contemporary, relevant examples (not "the same bakery from 1995")
- **Learning variety:** Need diverse industries to show concepts apply universally
- **Assessment creation:** Exams and assignments require fresh datasets to prevent cheating
- **Limited time:** Research, teaching, admin duties leave little time for data creation
- **Technical barriers:** Not all professors comfortable with advanced Excel or data generation tools
- **Reproducibility:** Students need datasets they can re-download for homework and practice
- **Multi-semester use:** Need scenarios that stay relevant across multiple course iterations

**Goals They're Trying to Achieve:**
- **Primary goal:** Effective teaching that results in strong student evaluations and learning outcomes
- **Engagement goal:** Make finance/controlling concepts accessible and interesting to students
- **Efficiency goal:** Streamline course preparation to focus time on research and mentoring
- **Modern curriculum:** Keep course materials contemporary and industry-relevant
- **Assessment integrity:** Prevent academic dishonesty through unique, rotating datasets

**Willingness to Pay Indicators:**
- Departmental budgets support software purchases (especially if multi-faculty use)
- Prefer one-time purchases vs. annual subscriptions (budget predictability)
- Early conversations suggest Department Partner tier ($999 for unlimited departmental use) appeals strongly
- Can justify expense with "improves teaching quality for 100+ students/year"

### Tertiary User Segment: Strategy Consultants

**Demographic/Firmographic Profile:**
- Consultants at strategy/management consulting firms (McKinsey-tier to boutique firms)
- Firm sizes: 5-500 employees
- Seniority: Junior consultants to partners (primarily senior consultants/managers who create deliverables)
- Industries served: Cross-industry (need versatile demo data)
- Geographic: Global, concentrated in business hubs (London, NYC, Frankfurt, Zurich, Singapore)
- Billable rates: €150-€500/hour depending on seniority

**Current Behaviors & Workflows:**
- Create 2-5 client presentations per month
- Pitch deck preparation: 5-10 hours per pitch, with 1-2 hours on demo data
- Data creation process:
  1. Junior consultants manually fabricate datasets (50% of time)
  2. Adapt anonymized client data from previous projects (30% of time) - compliance risk
  3. Use generic industry benchmarks (20% of time) - lacks specificity
- Tools used: Excel, PowerPoint, specialized industry databases, data visualization tools
- Client expectations: Industry-specific, realistic scenarios that demonstrate consultant expertise

**Specific Needs & Pain Points:**
- **Billable time waste:** Junior consultants spending 10-20 hours/month on non-billable data fabrication at €150-300/hour = €30,000-€60,000/year opportunity cost
- **Pitch quality:** Generic demo data fails to impress sophisticated clients
- **Industry customization:** Manufacturing client expects manufacturing data, not retail examples
- **Compliance concerns:** Using even anonymized client data creates legal/ethical gray areas
- **Speed to proposal:** RFP deadlines demand rapid turnaround (can't spend days creating data)
- **Credibility signal:** Polished, realistic demo data signals professionalism and attention to detail

**Goals They're Trying to Achieve:**
- **Primary goal:** Win client engagements through impressive, tailored pitch materials
- **Efficiency goal:** Maximize billable hours, minimize non-billable overhead
- **Quality goal:** Demonstrate deep industry understanding through relevant examples
- **Risk mitigation:** Avoid compliance issues from using client data
- **Team productivity:** Enable junior consultants to focus on analysis, not data fabrication

**Willingness to Pay Indicators:**
- Firms easily expense software <€1,000 per seat
- ROI calculation: If saves 15 hours/year at €200/hour = €3,000 value from €299 investment
- Early conversations suggest Team Plus ($599 for 20 seats) or custom enterprise pricing
- Decision-makers value credibility signal ("too cheap = not serious tool")

### User Segment Prioritization

**Phase 1 Launch (Weeks 1-8):**
- **Primary focus:** Business reporting trainers (Supporters M/L tiers)
- **Rationale:**
  - Smallest, most accessible segment
  - Acute pain point with frequent occurrence (every 2-3 workshops)
  - Active in IBCS© LinkedIn community (easy to reach)
  - Moderate pricing ($99-$299) matches value perception
  - Quick decision-makers (no procurement process)

**Phase 2 Expansion (Months 3-6):**
- **Add focus:** University professors (Department Partner tier)
- **Rationale:**
  - Moderate segment size, accessible via academic networks
  - Slower buying cycle (semester planning) but higher lifetime value
  - Department-wide licensing creates larger deal sizes ($999)
  - Word-of-mouth within faculty creates expansion opportunities

**Phase 3 Scale (Months 6-12):**
- **Add focus:** Strategy consultants (Team Plus tier, enterprise custom)
- **Rationale:**
  - Larger segment, requires different sales motion
  - Higher willingness to pay but longer sales cycles
  - Enterprise pricing unlocks significant revenue potential
  - Requires more polished product and case studies from earlier segments

---

## Goals & Success Metrics

### Business Objectives

**Revenue & Growth:**
- **Founding Member Revenue (12 weeks):** Generate €5,000-€40,000 from 50-300 founding members by end of Week 12
  - *Measurement:* Stripe payment tracking, founding member count dashboard
  - *Target breakdown:* 20 members by Week 4, 100 by Week 8, 300 by Week 12 (stretch goal)

- **Customer Acquisition Cost (CAC):** Maintain CAC <€50 per founding member during early access phase
  - *Measurement:* Total marketing spend / number of paying customers
  - *Channels:* LinkedIn ads, organic IBCS© community engagement, direct outreach

- **Conversion Rate:** Achieve 10-15% landing page visitor-to-founding-member conversion
  - *Measurement:* Analytics tracking (visitors → email signups → purchases)
  - *Benchmark:* Industry standard for SaaS landing pages is 5-10%; founding member urgency should drive higher

**Product Validation:**
- **Technical Proof of Concept (Week 2):** Successfully demonstrate event-sourced architecture generates mathematically consistent multi-view data
  - *Measurement:* Controller validation confirms Sales + Finance views reconcile perfectly
  - *Success criteria:* Zero consistency errors in generated scenarios

- **Data Quality Threshold (Week 6):** 80%+ of early beta testers rate generated scenarios as "realistic" or "very realistic"
  - *Measurement:* Post-generation survey (1-5 scale)
  - *Validation:* Brother (controller) + 3-5 finance/controlling professionals review output

**Market Validation:**
- **User Interview Validation (Week 3):** 70%+ of interviewed target users confirm pain point and indicate willingness to pay
  - *Measurement:* Structured interview responses (10-15 interviews across 3 segments)
  - *Success criteria:* Clear "yes, I would pay $X" from at least 50% of respondents

- **Channel Validation (Week 4):** IBCS© LinkedIn community engagement demonstrates viable distribution channel
  - *Measurement:* 50+ email signups from $100-200 ad spend (<$4 cost per lead)
  - *Engagement:* 100+ post impressions, 10+ meaningful comments/DMs from target users

### User Success Metrics

**Time Savings (Primary Value Metric):**
- **Data Generation Speed:** Users can generate complete 1-year, 2-view scenario in <5 minutes (vs. several hours manually)
  - *Measurement:* Time from prompt submission to usable dataset download/view
  - *Target:* 90% of generations complete in <3 minutes

- **Workshop Preparation Time Reduction:** Trainers report 30-50% reduction in demo data preparation time
  - *Measurement:* User survey at 30-day mark: "How much time did tellingCube save you?"
  - *Target:* Average reported savings of 2+ hours per workshop

**Data Quality (Secondary Value Metric):**
- **Multi-View Consistency:** 100% of generated scenarios pass mathematical reconciliation checks
  - *Measurement:* Automated validation (Finance payroll = HR headcount × hours × rates)
  - *Target:* Zero reconciliation errors in production

- **Scenario Believability:** 75%+ of end users (workshop attendees, students) don't question data realism
  - *Measurement:* Survey of trainers/professors: "Did students/attendees notice the data was synthetic?"
  - *Target:* <25% report "students questioned data validity"

**Adoption & Engagement:**
- **Repeat Usage:** Founding members generate 3+ scenarios within first 30 days
  - *Measurement:* Usage analytics per user account
  - *Target:* 60%+ of paying customers are "active users" (3+ generations/month)

- **Scenario Diversity:** Users explore beyond single industry (try 2+ preset scenarios)
  - *Measurement:* Track which presets are generated (Bakery, Hotel, Tech Startup)
  - *Target:* 70%+ of users try at least 2 different scenarios

**Satisfaction & Retention:**
- **Net Promoter Score (NPS):** Achieve NPS >40 among founding members
  - *Measurement:* 90-day survey: "How likely are you to recommend tellingCube?" (0-10 scale)
  - *Calculation:* % Promoters (9-10) minus % Detractors (0-6)
  - *Benchmark:* SaaS average NPS is ~30; >40 indicates strong product-market fit

- **Feature Request Engagement:** 40%+ of founding members provide product feedback
  - *Measurement:* Survey responses, feature request submissions, feedback emails
  - *Target:* Active community participation signals investment in product success

### Key Performance Indicators (KPIs)

| KPI | Definition | Target (Week 12) | Measurement Method |
|-----|------------|------------------|-------------------|
| **Founding Members** | Total paying customers (all tiers) | 100-300 | Stripe customer count |
| **MRR Equivalent** | If lifetime revenue were monthly subscription | €2,000-€8,000 | Total revenue ÷ 12 months |
| **Avg. Revenue Per User (ARPU)** | Average founding member payment | €100-€150 | Total revenue ÷ customers |
| **Landing Page CVR** | Visitor → email signup conversion | 20-30% | Analytics (signups/visitors) |
| **Email → Purchase CVR** | Email signup → paying customer | 30-50% | Stripe purchases / email list |
| **Time to First Value** | Prompt → usable scenario | <3 min (90% of cases) | Generation time logs |
| **Data Consistency Rate** | % scenarios passing validation | 100% | Automated validation checks |
| **User Satisfaction** | % users rating 4-5 stars | 80%+ | Post-generation survey |
| **CAC Payback** | Time to recoup acquisition cost | Immediate (one-time purchase) | Revenue - CAC |
| **Churn Rate** | % requesting refunds | <5% | Refund requests / total customers |

### Success Milestones & Go/No-Go Gates

**Week 2 Milestone: Technical Validation**
- ✅ GO: Event-sourced architecture works; data reconciles; controller approves quality
- ❌ NO-GO: Can't achieve consistency OR data quality unacceptable OR generation takes >10 minutes

**Week 4 Milestone: Market Validation**
- ✅ GO: 70%+ user interviews confirm pain; 50+ email signups; willingness to pay validated
- ❌ NO-GO: <50% confirm pain OR <20 email signups OR no one indicates willingness to pay

**Week 8 Milestone: Early Traction**
- ✅ GO: 20+ paying founding members; positive feedback; clear tier preference; <€5 CAC
- 🟡 PIVOT: 5-19 members → adjust pricing/positioning/channels but continue
- ❌ NO-GO: <5 members after 8 weeks → fundamental assumption broken

**Week 12 Milestone: Product-Market Fit Signal**
- ✅ SCALE: 100+ founding members; NPS >40; 60%+ active users; feature requests flowing
- 🟡 ITERATE: 50-99 members → solid foundation, refine product before Alpha expansion
- ❌ REASSESS: <50 members → validate assumptions, consider major pivot

---

## MVP Scope

### Core Features (Must Have)

**1. Three Preset Industry Scenarios**
- **Feature:** Bakery, Hotel, and Tech Startup scenarios with predefined parameters
- **Rationale:** Eliminates "blank canvas anxiety," removes UI complexity for custom inputs, proves concept works across different business models
- **User experience:** User lands on homepage, clicks one of three buttons ("Generate Bakery Scenario" | "Generate Hotel Scenario" | "Generate Tech Startup Scenario")
- **Technical implementation:**
  - Hardcoded scenario parameters (employee count, product mix, revenue range)
  - AI prompt templates optimized for each industry
  - No user input form required

**2. Event-Sourced Data Generation**
- **Feature:** Generate 7 types of business events across 5 dimensions for 1 year (monthly granularity)
- **Rationale:** This is the architectural foundation that guarantees multi-view consistency—the core differentiator
- **Event types generated:**
  - Employee Lifecycle (hires, terminations, promotions)
  - Sales Transactions (orders, deliveries, revenue recognition)
  - Cash Movements (customer payments, supplier payments, payroll)
  - Operational Work (hours worked, units produced)
  - Procurement & Expenses (materials, licenses, services)
  - Asset Lifecycle (purchases, depreciation)
  - Planning & Budgeting (planned revenue, costs, headcount)
- **Dimensions:** Time (12 months), Organization (simplified structure), Product (3-5 products), Counterparty (customers/suppliers/employees), Asset (if applicable)
- **Technical implementation:**
  - Single `events` table in NeonDB with wide schema
  - Claude API generates event stream via structured prompt
  - Server-side validation ensures mathematical consistency

**3. Two Departmental Views (Sales + Finance)**
- **Feature:** Interactive dashboards showing same data from Sales and Finance perspectives
- **Rationale:** Two views sufficient to prove "one reality, multiple perspectives" concept; reduces MVP complexity by 50% vs. 4 views
- **Sales View displays:**
  - Revenue by month (line chart)
  - Revenue by product (bar chart)
  - Top customers (table)
  - Sales trend analysis
- **Finance View displays:**
  - P&L summary (revenue, costs, profit)
  - Monthly cash flow
  - Cost breakdown by category
  - Margin analysis
- **Technical implementation:**
  - Dynamic SQL queries transform events → aggregated view data
  - React components render charts (using Chart.js or Recharts)
  - Side-by-side layout for easy comparison

**4. Mathematical Consistency Guarantee**
- **Feature:** Automated validation ensures Sales revenue matches Finance revenue, payroll costs align with employee headcount, etc.
- **Rationale:** This is the "wow" moment—users can instantly verify multi-view consistency
- **Validation rules:**
  - Finance total revenue = sum(Sales events × price)
  - Finance payroll expense = sum(Employee hours × hourly rates)
  - Finance COGS = sum(Procurement events for materials)
- **User experience:** "✓ Multi-view consistency verified" badge displayed after generation
- **Technical implementation:**
  - Server-side validation function runs after event generation
  - Comparison logic checks cross-view reconciliation
  - Generation fails if consistency check doesn't pass (retry with adjusted AI parameters)

**5. Ephemeral Generation (No User Accounts)**
- **Feature:** Users can generate scenarios without signing up or logging in
- **Rationale:** Zero friction to try product; removes 2-3 weeks of auth development; encourages experimentation
- **User experience:**
  - Click scenario button → loading animation (30-60 seconds) → view results
  - No "my scenarios" dashboard, no saved history
  - Each visit generates fresh data
- **Technical implementation:**
  - Session-based data storage (temporary, cleared after 24 hours)
  - No user database, no authentication system
  - Scenarios not persisted beyond session

**6. Founding Member Checkout (Stripe Integration)**
- **Feature:** Call-to-action with Stripe payment for 5 pricing tiers
- **Rationale:** Enables immediate revenue generation; validates willingness to pay
- **User experience:**
  - After viewing generated scenario, CTA appears: "Want unlimited access? Become a Founding Member"
  - Pricing tier cards displayed (Supporter S/M/L, Team Plus, Department Partner)
  - Click tier → Stripe Checkout → payment → confirmation email
- **What founding members get (MVP):**
  - Unlimited scenario generations (no rate limits)
  - Priority email support
  - Early access to new features (Alpha/Beta)
  - Founding member badge
  - Grandfathered lifetime pricing
- **Technical implementation:**
  - Stripe Checkout integration (hosted payment page)
  - 5 Stripe products configured with pricing
  - Email delivery via SendGrid or Resend (welcome email + access instructions)

**7. Simple Landing Page**
- **Feature:** Single-page website explaining value proposition with scenario generation
- **Rationale:** MVP doesn't need multi-page site; focus on conversion
- **Page sections:**
  - Hero: "Generate realistic business data in minutes instead of hours"
  - Problem statement (2-3 sentences)
  - Solution demo (3 scenario buttons)
  - Multi-view consistency proof (side-by-side dashboards)
  - Founding member CTA with pricing tiers
  - FAQ (3-5 common questions)
- **Technical implementation:**
  - Next.js app with TailwindCSS
  - Responsive design (mobile-friendly)
  - Fast loading (<2 seconds)

### Out of Scope for MVP

**Authentication & User Management:**
- ❌ User accounts, login/logout
- ❌ Password reset flows
- ❌ User dashboards
- ❌ "My Scenarios" saved library
- *Why deferred:* Adds 2-3 weeks development time; not needed to prove core value proposition
- *When to add:* Alpha phase (Weeks 8-12) when paying customers need persistent access

**Custom Scenario Inputs:**
- ❌ User-defined prompts ("Generate data for my specific company")
- ❌ Industry selection dropdown
- ❌ Parameter customization (employee count, timeframe, product mix)
- *Why deferred:* UI complexity, prompt engineering challenges, edge case handling
- *When to add:* Beta phase (Months 3-4) after validating preset scenarios work

**Data Export Functionality:**
- ❌ CSV download
- ❌ Excel export
- ❌ JSON API access
- *Why deferred:* Not needed to demonstrate multi-view consistency; can view in-browser
- *When to add:* Alpha phase as first paid feature unlock

**Additional Views:**
- ❌ HR View (headcount, payroll, working hours)
- ❌ Controlling View (plan vs. actual, variance analysis)
- *Why deferred:* 2 views sufficient to prove concept; each view adds 1-2 weeks development
- *When to add:* Alpha phase (prioritize based on founding member feedback)

**Advanced Features:**
- ❌ Multi-year scenarios (5+ years of data)
- ❌ AI-generated business narratives (story mode)
- ❌ Scenario variant comparison (base vs. shock vs. improvement)
- ❌ Modular event building blocks (drag-and-drop events)
- ❌ Template marketplace
- ❌ Real-time collaboration
- *Why deferred:* "Nice to have" features that don't impact core value delivery
- *When to add:* MVP+ and beyond, based on validated user demand

**Performance Optimizations:**
- ❌ Materialized database views
- ❌ Caching layer
- ❌ CDN for static assets
- *Why deferred:* Premature optimization; handle <100 concurrent users fine without it
- *When to add:* When performance metrics show >5 second generation times

**Marketing & Growth:**
- ❌ Referral program
- ❌ Affiliate system
- ❌ In-app onboarding tutorial
- ❌ Email drip campaigns
- *Why deferred:* Focus on product validation before growth mechanics
- *When to add:* Post-MVP when retention and satisfaction metrics are strong

### MVP Success Criteria

**Technical Success:**
- ✅ All 3 preset scenarios generate successfully 95%+ of the time
- ✅ Generation completes in <5 minutes for 90% of requests
- ✅ 100% of generated scenarios pass multi-view consistency validation
- ✅ Zero data reconciliation errors between Sales and Finance views
- ✅ Site loads in <2 seconds, responsive on mobile and desktop

**User Success:**
- ✅ 10+ beta testers successfully generate scenarios without confusion
- ✅ 80%+ of beta testers rate generated data "realistic" or "very realistic" (4-5 stars)
- ✅ Controller validation confirms financial data is plausible and consistent
- ✅ No critical bugs blocking core user flow (scenario selection → generation → viewing)

**Business Success:**
- ✅ First 5 paying founding members within 2 weeks of launch
- ✅ Landing page achieves >10% visitor-to-email-signup conversion
- ✅ At least one founding member from each tier (S, M, L) validates pricing structure
- ✅ <€5 customer acquisition cost for first 20 customers

**Go/No-Go for Alpha Phase:**
- ✅ PROCEED if: MVP works reliably, 20+ founding members, positive feedback, clear feature requests
- 🟡 ITERATE if: MVP works but <20 members → improve marketing/positioning before Alpha
- ❌ STOP if: Technical issues persist, <5 members after 8 weeks, negative feedback on core concept

---

## Post-MVP Vision

### Phase 2 Features (Alpha - Weeks 8-16)

**Priority determined by founding member feedback and validation metrics**

**1. User Accounts & Authentication**
- **What:** Basic user registration, login/logout, password reset
- **Why:** Founding members need persistent access to unlock their lifetime benefits
- **User value:** Save generated scenarios, return to previous work, manage account
- **Technical scope:** Auth0 or NextAuth.js integration, user database table, session management
- **Success metric:** 80%+ of founding members create accounts within first week of Alpha launch

**2. HR & Controlling Views**
- **What:** Add two additional departmental perspectives (4 total views)
- **Why:** Completes the "multi-view consistency" vision; appeals to broader audience
- **HR View displays:**
  - Headcount trends over time
  - Payroll expenses by month
  - Employee lifecycle events (hires, promotions, terminations)
  - Working hours analysis
- **Controlling View displays:**
  - Plan vs. actual comparisons
  - Variance analysis (budget vs. actuals)
  - KPI dashboards
  - Cost center breakdowns
- **Success metric:** 60%+ of users explore all 4 views after launch

**3. Data Export Functionality**
- **What:** CSV, Excel, and JSON download options
- **Why:** Trainers/professors need to share datasets with students; consultants need to import into presentations
- **Export options:**
  - Raw events table (all business events)
  - View-specific exports (Sales data only, Finance data only, etc.)
  - Combined multi-view export
- **Watermarking (free tier):** "Generated by tellingCube" footer on free exports
- **Success metric:** 70%+ of founding members use export feature within 30 days

**4. Scenario Library (5-10 Additional Industries)**
- **What:** Expand from 3 to 10-15 preset scenarios
- **Industries to add (prioritized by demand):**
  - Manufacturing company
  - Retail store
  - Professional services firm (consulting, legal, accounting)
  - E-commerce business
  - Healthcare clinic
  - Restaurant/café
  - SaaS startup
- **Success metric:** No single scenario dominates >40% of generations (indicates healthy diversity)

**5. Saved Scenarios Dashboard**
- **What:** "My Scenarios" page showing generation history
- **Features:**
  - List of all generated scenarios with metadata (industry, date, parameters)
  - Re-open previous scenarios
  - Delete old scenarios
  - Share scenarios via link (view-only)
- **Success metric:** 50%+ of users return to view previously generated scenarios

### Long-Term Vision (Beta & MVP - Months 3-12)

**Beta Phase (Months 3-6): Customization & Intelligence**

**6. Custom Scenario Prompts**
- **What:** User-defined inputs instead of just presets
- **Input form:**
  - Industry (dropdown + custom text)
  - Company size (employee count range)
  - Timeframe (1-5 years, monthly/quarterly granularity)
  - Product/service count
  - Optional: Revenue range, growth rate, seasonal patterns
- **AI prompt engineering:** Dynamic prompt generation based on user inputs
- **Validation:** Prevent unrealistic combinations (e.g., 1 employee but $10M revenue)
- **Success metric:** 40%+ of generations use custom prompts vs. presets

**7. AI-Generated Business Narratives (Story Mode)**
- **What:** LLM generates believable business story first, then creates events matching the narrative
- **Example workflow:**
  1. User requests "Bakery scenario"
  2. AI generates story: "Family bakery founded in 2020. Expanded product line in Year 2 with specialty cakes. Hired 2 bakers in Q3. Supply chain issues drove ingredient costs up 15% in Q4. Holiday season sales spike in December."
  3. AI then generates events consistent with story
- **User value:** More realistic, explainable scenarios; great for teaching cause-and-effect
- **Success metric:** 30%+ of users prefer story mode vs. direct generation

**8. Scenario Variant Comparison**
- **What:** Generate multiple versions of same base scenario (base, optimistic, pessimistic, shock event)
- **Use cases:**
  - "What if we hired 3 more employees?"
  - "What if inflation increased costs 10%?"
  - "What if we lost our biggest customer?"
- **Display:** Side-by-side comparison across all views
- **Success metric:** 25%+ of users generate at least one scenario variant

**MVP Phase (Months 6-12): Scale & Ecosystem**

**9. Template Marketplace (IBCS©-Inspired Dashboards)**
- **What:** Community-contributed dashboard templates built on tellingCube data
- **Model:**
  - Users create and share dashboard templates
  - Premium templates sold for $5-$25 (revenue split: 70% creator, 30% tellingCube)
  - Free templates drive engagement
- **Content types:**
  - Power BI dashboard templates
  - Tableau workbooks
  - Excel pivot table templates
  - IBCS©-compliant report templates
- **Success metric:** 50+ templates available; 10+ creators earning $100+/month

**10. Multi-Year & Granular Timescales**
- **What:** Support 1-10 year scenarios with weekly or quarterly granularity
- **Use cases:**
  - Long-term business evolution (startup → scale-up over 5 years)
  - Weekly sales data for retail scenarios
  - Quarterly financial reporting
- **Technical challenge:** Performance optimization for large datasets
- **Success metric:** 20%+ of scenarios use multi-year or weekly granularity

**11. Enterprise SaaS Model**
- **What:** Transition from lifetime pricing to recurring revenue for new customers
- **Pricing tiers:**
  - **Professional:** $49/month - 5 users, unlimited scenarios, all views, priority support
  - **Team:** $149/month - 25 users, custom branding, SSO, dedicated success manager
  - **Enterprise:** $499+/month - Unlimited users, API access, on-premise deployment option, custom scenarios, SLA
- **Grandfather clause:** Founding members keep lifetime access forever
- **Success metric:** $10,000+ MRR from new customers by Month 12

**12. Advanced Features (Post-MVP)**
- **Modular Event Building Blocks:** Drag-and-drop interface to manually add/remove events
- **Real-Time Collaboration:** Multiple users editing same scenario simultaneously
- **API Access:** Programmatic scenario generation for integrations
- **White-Label Licensing:** Training firms can rebrand tellingCube for their clients
- **IBCS© Certification Partnership:** Official certification as IBCS©-compliant tool

### Expansion Opportunities (12+ Months)

**Geographic Expansion:**
- Localization (German, French, Spanish interfaces)
- Currency support (EUR, USD, GBP, CHF)
- Regional accounting standards (GAAP, IFRS, HGB)
- Country-specific scenarios (UK Ltd, German GmbH, US LLC)

**Vertical-Specific Solutions:**
- **Healthcare Edition:** Patient volumes, insurance billing, staffing regulations
- **Manufacturing Edition:** Production lines, inventory management, supply chain
- **Non-Profit Edition:** Grants, donations, program expenses, outcome metrics
- **Government Edition:** Budget cycles, procurement rules, departmental accounting

**Educational Partnerships:**
- University site licenses (unlimited access for entire business school)
- Curriculum integration (textbook partnerships, case study collaborations)
- Student certification program ("tellingCube Certified Data Analyst")
- Professor training workshops

**Consulting Firm Partnerships:**
- McKinsey, BCG, Bain custom scenario packages
- Boutique consulting firm white-label deals
- Industry-specific scenario packs (automotive, pharma, finance)
- RFP response automation tools

### North Star Vision (3-5 Years)

**tellingCube becomes the industry standard for synthetic business data generation, recognized as:**

1. **The IBCS© Community's Data Platform of Choice**
   - Partnership with IBCS Institute (official certification achieved)
   - 70%+ of IBCS© certified trainers use tellingCube
   - Case studies featured in IBCS© publications and events

2. **The Educational Standard for Business Schools**
   - Adopted by 500+ universities globally
   - Integrated into leading business textbooks
   - Student licenses drive next generation of business professionals who expect tellingCube quality

3. **The Consultant's Productivity Tool**
   - 10,000+ consulting professionals using tellingCube monthly
   - Recognized brand among McKinsey/BCG/Bain analysts
   - Industry benchmark: "tellingCube data" synonymous with "realistic demo scenarios"

4. **A Sustainable, Profitable Business**
   - $500,000+ ARR from mix of enterprise SaaS, marketplace commissions, and partnerships
   - 5,000+ active users (founding members + subscribers)
   - Team of 3-5 (founder + 2-4 contractors/employees)
   - Profitable without external funding (bootstrapped success story)

---

## Technical Considerations

*Note: These are initial technical decisions for MVP. Architecture decisions are not final and will evolve based on validation and scale requirements.*

### Platform Requirements

**Target Platforms:**
- **Primary:** Web application (browser-based, no native app required)
- **Deployment:** Cloud-hosted SaaS (Vercel)
- **Access:** Public internet, HTTPS-only

**Browser/OS Support:**
- **Desktop browsers:** Chrome, Firefox, Safari, Edge (latest 2 versions)
- **Mobile browsers:** iOS Safari, Chrome Mobile (responsive design, not native apps)
- **Screen sizes:** 1024px+ (desktop), 768px+ (tablet), 375px+ (mobile)
- **No IE11 support:** Modern JavaScript features required

**Performance Requirements:**
- **Page load time:** <2 seconds for landing page (Lighthouse score >90)
- **Time to First Value (TTFV):** <5 minutes from prompt to usable scenario (target: <3 min for 90% of cases)
- **Concurrent users (MVP):** Support 100 simultaneous generations without degradation
- **Database query performance:** View rendering queries <500ms
- **Uptime target (MVP):** 95% (upgradable to 99.9% for enterprise SaaS phase)

**Accessibility:**
- WCAG 2.1 Level AA compliance (where feasible for MVP)
- Keyboard navigation support
- Screen reader compatibility for key user flows
- Color contrast ratios meet accessibility standards

### Technology Preferences

**Frontend:**
- **Framework:** Next.js 14+ (React-based)
  - *Rationale:* Server-side rendering (SSR) for SEO, built-in API routes, Vercel-optimized, TypeScript support
  - *Alternatives considered:* Remix (newer, less ecosystem), plain React (more setup overhead)
- **Styling:** TailwindCSS 3+
  - *Rationale:* Rapid prototyping, small bundle size, utility-first approach matches MVP speed requirement
  - *Brand colors:* tellingCube palette (to be defined during design phase)
- **Charting Library:** Recharts or Chart.js
  - *Rationale:* React-native components (Recharts) vs. lightweight Canvas-based (Chart.js)
  - *Decision deferred to:* Week 1 of implementation (prototype both, choose based on rendering performance)
- **State Management:** React Context API (MVP), Zustand or Redux (if needed in Alpha)
  - *Rationale:* Avoid premature complexity; Context sufficient for ephemeral session state

**Backend:**
- **Server Framework:** Next.js API Routes (serverless functions)
  - *Rationale:* Unified codebase (frontend + backend), automatic deployment with Vercel, zero server management
  - *Endpoints needed:*
    - `POST /api/generate` - Trigger AI scenario generation
    - `GET /api/scenario/:id` - Retrieve generated scenario data
    - `GET /api/views/:scenarioId/:viewType` - Get transformed view data (sales, finance, etc.)
    - `POST /api/validate` - Run consistency validation
- **Runtime:** Node.js 18+ (Vercel default)
- **Language:** TypeScript
  - *Rationale:* Type safety for complex event schema, better IDE support, catches errors pre-runtime

**Database:**
- **Primary Database:** NeonDB (serverless PostgreSQL)
  - *Rationale:* Serverless (pay-per-use), PostgreSQL-compatible (familiar SQL), generous free tier, scales automatically
  - *Connection:* Vercel Postgres integration (native support)
  - *Backup strategy (MVP):* NeonDB automatic daily backups
  - *Migration path:* Standard PostgreSQL means easy migration to RDS/self-hosted if needed later

**AI/LLM:**
- **Primary Provider:** Anthropic Claude API (Sonnet 3.5 or 4)
  - *Rationale:* Superior business logic understanding, longer context windows, structured output support
  - *Estimated cost:* $0.50-$2 per scenario generation (depending on prompt complexity and output tokens)
- **Fallback Provider (Alpha+):** OpenAI GPT-4
  - *Rationale:* Redundancy if Claude API has outages; cost comparison testing
- **Prompt Strategy:** Hybrid Structured Pipeline (Option F from brainstorming)
  1. AI generates business context
  2. AI generates high-level time-series trends
  3. AI generates granular events within envelopes
  4. System validates and patches inconsistencies

**Payment Processing:**
- **Provider:** Stripe Checkout
  - *Rationale:* Industry standard, handles PCI compliance, supports one-time payments, easy webhook integration
  - *Products configured:** 5 founding member tiers (S/M/L/Team Plus/Department Partner)
  - *Post-payment flow:** SendGrid/Resend email with access instructions

**Email Delivery:**
- **Provider:** Resend or SendGrid
  - *Rationale:* Simple API, generous free tier (Resend: 100 emails/day free), React Email templates support
  - *Emails needed:* Welcome email, payment confirmation, password reset (Alpha+)

**Hosting/Infrastructure:**
- **Platform:** Vercel
  - *Rationale:* Zero-config Next.js deployment, automatic SSL, preview deployments, serverless by default
  - *Plan:* Pro tier ($20/month) for production (Hobby tier works for MVP testing)
  - *CDN:** Vercel Edge Network (automatic)
- **Domain:** tellingcube.com (to be registered)
  - *DNS:* Vercel DNS or Cloudflare (TBD)

**Monitoring & Analytics:**
- **Error Tracking:** Sentry (free tier: 5K events/month)
- **Analytics:** Vercel Analytics + Plausible (privacy-friendly alternative to Google Analytics)
- **Logging:** Vercel Logs (basic), upgrade to Axiom or LogFlare if needed

### Architecture Considerations

**Repository Structure:**
- **Monorepo:** Single repository for frontend + backend
  - *Rationale:* Simplified deployment, shared TypeScript types, faster iteration for solo developer
  - *Structure:*
    ```
    /telling-cube
      /app           (Next.js 14 app directory)
        /api         (API routes)
        /components  (React components)
        /lib         (shared utilities)
      /prisma        (database schema & migrations)
      /public        (static assets)
      /types         (TypeScript type definitions)
    ```

**Database Schema (MVP - Single Table Approach):**

**Events Table (Wide Schema):**
```sql
CREATE TABLE events (
  id UUID PRIMARY KEY,
  scenario_id UUID NOT NULL,
  event_type VARCHAR(50),      -- 'employee_hire', 'sale', 'payment', etc.
  event_date DATE,

  -- Dimensions
  organization_unit VARCHAR(100),
  product_id VARCHAR(50),
  counterparty_id VARCHAR(50),  -- customer/supplier/employee ID
  asset_id VARCHAR(50),

  -- Financial attributes
  amount DECIMAL(12,2),
  quantity DECIMAL(10,2),
  unit_price DECIMAL(10,2),

  -- Metadata
  event_narrative TEXT,         -- AI-generated explanation
  created_at TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_scenario ON events(scenario_id);
CREATE INDEX idx_event_type ON events(event_type);
CREATE INDEX idx_date ON events(event_date);
```

**Why single wide table for MVP:**
- Simpler to implement (1-2 days vs. 1-2 weeks for normalized schema)
- Easier AI prompt engineering (single JSON structure to generate)
- Flexible schema (add columns without complex migrations)
- Adequate performance for <10,000 events per scenario

**Evolution path:**
- **Alpha:** Add `users` table, `scenarios` metadata table
- **Beta:** Consider event type-specific tables if performance degrades
- **MVP:** Implement materialized views for dashboard queries

**Service Architecture:**
- **MVP:** Monolithic Next.js app (frontend + API routes in same deployment)
  - *Rationale:* Simplicity, no microservices complexity, suitable for <1,000 users
- **Alpha+:** Extract AI generation to separate serverless function if latency/cost requires optimization
- **Enterprise:** Potential separate API service for white-label/API access customers

**Integration Requirements:**
- **MVP:** Stripe webhooks (payment confirmation)
- **Alpha:** Email service API (Resend/SendGrid)
- **Beta:** Potential OAuth providers (Google, Microsoft for SSO)
- **Future:** Export integrations (Power BI connector, Tableau connector)

**Security/Compliance:**

**MVP Security:**
- HTTPS-only (enforced by Vercel)
- API rate limiting (100 requests/min per IP via Vercel edge config)
- Input validation (prevent SQL injection, XSS)
- Stripe handles PCI compliance (no card data touches our servers)
- Session tokens (httpOnly cookies, 7-day expiration)

**Data Privacy:**
- **GDPR Considerations:**
  - All generated data is synthetic (no personal information)
  - Email collection requires consent (checkbox on signup)
  - Right to deletion (manual process for MVP, automated in Alpha)
- **Data Retention:**
  - Ephemeral scenarios deleted after 24 hours
  - Paying customer scenarios retained indefinitely (with deletion option)
- **No tracking:** Privacy-friendly analytics (Plausible), no third-party ad trackers

**Alpha+ Security Enhancements:**
- Auth0 or NextAuth.js (secure authentication)
- CSRF protection
- SQL injection prevention via Prisma ORM
- Content Security Policy (CSP) headers
- Automated security scanning (Snyk or Dependabot)

**Compliance Roadmap:**
- **MVP:** Basic security, synthetic data only
- **Alpha:** GDPR-compliant data handling, privacy policy, terms of service
- **Enterprise:** SOC 2 Type II consideration (if enterprise deals require it)
- **Future:** ISO 27001 (if serving regulated industries like finance/healthcare)

---

## Constraints & Assumptions

### Constraints

**Budget:**
- **MVP Development:** €0 initial investment (using free tiers)
  - Vercel Hobby tier: €0/month (sufficient for development and early testing)
  - NeonDB free tier: €0/month (500 MB storage, adequate for MVP)
  - Claude API: Pay-per-use (~€50-100 for testing/validation phase)
  - Domain registration: ~€15/year (tellingcube.com)
  - **Total MVP budget:** €100-150 for 2-4 week development phase

- **Post-Launch Operational (Months 1-3):**
  - Vercel Pro: €20/month
  - NeonDB Scale tier (if needed): €0-20/month (usage-based)
  - Claude API costs: €0.50-2 per generation × estimated 500 generations/month = €250-1,000/month
  - Stripe fees: 2.9% + €0.30 per transaction
  - Email (Resend/SendGrid): €0-15/month
  - **Total monthly operational:** €300-1,100/month (covered by founding member revenue if >3-10 members)

- **Constraint implication:** Must reach revenue breakeven quickly; cannot sustain >€1K/month burn for long period

**Timeline:**
- **MVP Development:** 2-4 weeks (target: ship by end of Week 4)
  - Assumes 20-30 hours/week dedicated time (evenings + weekends)
  - Total development hours: 40-120 hours
  - Buffer for unexpected blockers: 1 additional week maximum
- **Market Validation:** Weeks 2-4 (parallel to development)
- **Launch & Early Access:** Week 4-8
- **Alpha Phase:** Weeks 8-16 (if validation successful)

- **Constraint implication:** Aggressive timeline requires ruthless scope discipline; any feature not in "Core Features (Must Have)" gets deferred

**Resources:**
- **Team size:** Solo founder (Mario)
  - Full-stack development (frontend + backend + database)
  - Product management & strategy
  - Marketing & sales
  - Customer support
- **Available time:** 20-30 hours/week
  - Not full-time (maintaining current commitments)
  - Weekday evenings (2-3 hours/day, 5 days = 10-15 hours)
  - Weekends (5-8 hours each day, 2 days = 10-15 hours)

- **Support resources:**
  - Brother (controller in 250-300 person manufacturing company): Data quality validation, financial realism review
  - IBCS Institute CEO relationship: Market access, community validation
  - Personal network: Beta testers, early user interviews

- **Constraint implication:** Must prioritize mercilessly; cannot build everything; need automation wherever possible; customer support must be sustainable at scale of 1

**Technical:**
- **Skill constraints:**
  - Strengths: Full-stack development, modern web technologies
  - Learning curve: Potentially new to Next.js 14 app directory, Vercel deployment patterns, NeonDB specifics
  - Time buffer: +20-30% for learning/experimentation

- **API rate limits:**
  - Claude API: Tier-dependent (need to verify current tier limits)
  - Stripe API: Generally generous for MVP scale
  - Vercel functions: 100GB-hours/month on Hobby (sufficient for MVP)

- **Database limits:**
  - NeonDB free tier: 500 MB storage, 3 GB data transfer/month
  - Upgrade trigger: >400 MB data (leave 100 MB buffer)
  - Estimated scenario size: ~1-5 MB (100-500 scenarios before upgrade needed)

- **Third-party dependencies:**
  - Claude API availability: Critical dependency; no fallback in MVP
  - Vercel uptime: Critical for hosting; 99%+ SLA on Pro tier
  - Stripe availability: Critical for payments; industry-leading uptime

- **Constraint implication:** Technical architecture must account for API failures gracefully; monitoring essential to catch issues before users do

### Key Assumptions

**Market Assumptions:**

1. **Target market size is sufficient:**
   - Assumption: At least 1,000 business reporting trainers, 5,000 university business professors, and 10,000 strategy consultants globally have this pain point
   - Risk if wrong: Market too small to reach 100-300 founding members
   - Validation: LinkedIn research, IBCS© community size measurement, user interviews

2. **Pain point is acute enough to drive purchase:**
   - Assumption: "Several hours per demo dataset" pain is real and frequent (not occasional annoyance)
   - Risk if wrong: Users acknowledge problem but won't pay to solve it
   - Validation: Willingness-to-pay questions in user interviews, landing page conversion rates

3. **IBCS© community is accessible and engaged:**
   - Assumption: LinkedIn #IBCS provides viable distribution channel; community will engage with tellingCube content
   - Risk if wrong: Channel doesn't convert, need alternative go-to-market strategy
   - Validation: Week 3-4 landing page test, LinkedIn ad campaign results

4. **No major competitor will launch similar product in next 6 months:**
   - Assumption: Blue ocean window stays open long enough to establish position
   - Risk if wrong: Well-funded competitor could out-execute and capture market
   - Validation: Ongoing competitive monitoring; speed to market is primary defense

**User Assumptions:**

5. **Users value multi-view consistency:**
   - Assumption: "Data that reconciles across departments" is a recognized pain point, not just internal insight
   - Risk if wrong: Users don't care about consistency; just want "plausible-looking numbers"
   - Validation: User interview feedback, beta tester reactions to consistency demo

6. **Users will tolerate ephemeral generation (MVP):**
   - Assumption: Try-before-buy works even without save functionality; creates urgency to become founding member
   - Risk if wrong: Users expect persistence before paying; conversion suffers
   - Validation: Landing page analytics (do users generate multiple times?), conversion rate tracking

7. **Founding member model creates urgency:**
   - Assumption: Limited-time lifetime pricing drives faster purchase decisions than standard subscription
   - Risk if wrong: Users procrastinate ("I'll buy later"); FOMO doesn't work
   - Validation: Time-to-purchase metrics, cohort conversion analysis

**Technical Assumptions:**

8. **Event-sourced architecture delivers quality:**
   - Assumption: Single event stream → multiple views approach generates realistic, consistent data
   - Risk if wrong: Technical complexity too high; quality suffers; can't achieve consistency
   - Validation: Week 1-2 technical proof of concept with controller review

9. **AI can generate plausible business scenarios:**
   - Assumption: Claude API (or GPT-4) can understand business logic well enough to create believable events
   - Risk if wrong: Generated data is obviously fake; users lose trust
   - Validation: Week 2-3 quality testing with finance professionals, beta tester feedback

10. **Single wide table performs adequately:**
    - Assumption: <10,000 events per scenario, <500ms query times achievable with indexed wide schema
    - Risk if wrong: Performance degrades; must refactor to normalized schema (adds 2-3 weeks)
    - Validation: Performance testing during MVP development, query profiling

11. **Serverless architecture scales to 100 concurrent users:**
    - Assumption: Vercel + NeonDB can handle MVP traffic without manual infrastructure management
    - Risk if wrong: Cold starts cause timeouts; database connections exhausted; user experience degrades
    - Validation: Load testing before public launch, monitoring during early access

**Business Assumptions:**

12. **€100-€150 ARPU is realistic:**
    - Assumption: Healthy mix across pricing tiers (not everyone chooses $29 Supporter S)
    - Risk if wrong: Average is closer to €50; need 2-6x more customers for same revenue
    - Validation: Early purchase data (first 20-50 customers), tier preference analysis

13. **10-15% landing page conversion is achievable:**
    - Assumption: Founding member urgency + clear value prop drives above-average conversion
    - Risk if wrong: Conversion is 2-5% (industry standard); need 3-5x more traffic
    - Validation: Week 4 landing page test, A/B testing messaging

14. **Solo founder can support 100-300 customers:**
    - Assumption: Product is self-service enough; support load manageable at <5 hours/week
    - Risk if wrong: Support requests overwhelm; quality/responsiveness suffers; churn increases
    - Validation: Early customer support ticket volume, time tracking, automation opportunities

15. **Lifetime pricing doesn't cannibalize future revenue:**
    - Assumption: Founding members represent <10% of eventual TAM; plenty of future subscription customers remain
    - Risk if wrong: Given away too much value; limits future revenue potential
    - Validation: Market size research, cohort retention analysis, enterprise pipeline development

**Go-to-Market Assumptions:**

16. **Word-of-mouth will drive growth post-launch:**
    - Assumption: Satisfied trainers/professors share with colleagues; viral coefficient >0.5
    - Risk if wrong: Each customer is isolated; must continually acquire via paid channels
    - Validation: NPS score, referral tracking, organic vs. paid customer mix

17. **€50 CAC is sustainable:**
    - Assumption: LinkedIn ads, organic content, direct outreach keeps acquisition cost low
    - Risk if wrong: CAC is €100-200; payback period extends; margins compress
    - Validation: Channel-specific CAC tracking, optimization experiments

18. **Brother's validation carries weight with target market:**
    - Assumption: "Controller-approved" credibility signal resonates with business reporting professionals
    - Risk if wrong: Users don't care about third-party validation; quality speaks for itself
    - Validation: A/B test messaging with/without validation claim, testimonial impact analysis

---

## Risks & Open Questions

### Key Risks

**Technical Risks**

**RISK 1: Event-Sourced Architecture Doesn't Scale** ⚠️ **HIGH SEVERITY, MEDIUM LIKELIHOOD**
- **Description:** Single wide table becomes performance bottleneck; query times exceed 2-5 seconds; user experience degrades
- **Impact:** Must refactor to normalized schema or materialized views; adds 2-4 weeks development time; delays Alpha launch
- **Mitigation:**
  - Performance testing during MVP development (Week 2-3)
  - Database query profiling with realistic data volumes
  - Design schema with clear migration path to normalized structure
  - Limit MVP scope to 2 views (reduces query complexity)
- **Contingency:** If identified early (Week 2), pivot to normalized schema; if discovered post-launch, implement materialized views incrementally
- **Owner:** Mario (technical validation)

**RISK 2: AI Generation Quality Insufficient** 🔴 **CRITICAL SEVERITY, MEDIUM LIKELIHOOD**
- **Description:** Claude API generates unrealistic scenarios (impossible margins, nonsensical patterns, obvious hallucinations); users lose trust
- **Impact:** Product value proposition collapses; negative word-of-mouth; high churn; pivot required
- **Mitigation:**
  - Week 1-2: Controller (brother) review of generated scenarios
  - Structured prompt engineering with explicit business rules
  - Validation layer catches mathematical inconsistencies
  - Quality testing with 3-5 finance professionals before launch
  - Fallback to GPT-4 if Claude quality insufficient
- **Contingency:** If quality unacceptable, explore template-driven approach (predefined patterns, AI fills parameters only)
- **Owner:** Mario (prompt engineering) + Brother (quality validation)

**RISK 3: Claude API Rate Limits or Outages** ⚠️ **HIGH SEVERITY, LOW LIKELIHOOD**
- **Description:** Claude API becomes unavailable or rate-limited; scenario generation fails; users cannot use product
- **Impact:** Service downtime; frustrated users; refund requests; reputation damage
- **Mitigation:**
  - Verify Claude API tier limits before launch
  - Implement graceful error messaging ("Generation temporarily unavailable, please try again in 5 minutes")
  - Monitor API usage proactively
  - Alpha: Add GPT-4 fallback for redundancy
- **Contingency:** If persistent issues, negotiate higher tier with Anthropic or switch primary provider to OpenAI
- **Owner:** Mario (monitoring + failover implementation)

**Market Risks**

**RISK 4: Market Size Insufficient** ⚠️ **HIGH SEVERITY, MEDIUM LIKELIHOOD**
- **Description:** Fewer than 500 reachable potential customers globally; cannot achieve 100-300 founding members
- **Impact:** Revenue targets unmet; business model unsustainable; pivot or shutdown required
- **Mitigation:**
  - Week 2-3: LinkedIn research to count IBCS© community size
  - Week 3: User interviews validate market breadth across segments
  - Week 4: Landing page test measures actual interest
  - Broaden TAM definition if needed (add BI professionals, data analysts)
- **Contingency:** If IBCS© community too small (<1,000), pivot to broader "business data professionals" positioning
- **Owner:** Mario (market research + validation)
- **Go/No-Go:** If <20 email signups in Week 4 test, reassess viability

**RISK 5: Pain Point Not Acute** 🔴 **CRITICAL SEVERITY, MEDIUM-HIGH LIKELIHOOD**
- **Description:** Users acknowledge problem exists but don't prioritize solving it; "nice to have" not "must have"
- **Impact:** Low conversion rates (<5%); long sales cycles; CAC exceeds LTV; business unsustainable
- **Mitigation:**
  - User interviews with willingness-to-pay questions (Week 2-3)
  - Landing page messaging emphasizes ROI ("Save 60-80 hours/year = €6,000-€24,000 in billable time")
  - Testimonials from early adopters highlighting time savings
  - Free trial lowers barrier to trying (ephemeral generation)
- **Contingency:** If conversion <5% after Week 8, pivot messaging to different value prop or target different segment
- **Owner:** Mario (customer development + messaging)
- **Go/No-Go:** If <50% of interviewees indicate willingness to pay, fundamental problem

**RISK 6: Competitor Launches Similar Product** ⚠️ **MEDIUM SEVERITY, LOW-MEDIUM LIKELIHOOD**
- **Description:** Well-funded startup or established BI vendor launches multi-view synthetic data tool
- **Impact:** Market share pressure; pricing pressure; differentiation challenge; slower growth
- **Mitigation:**
  - Speed to market (ship MVP in 2-4 weeks, not 3-6 months)
  - Build moat through IBCS© community relationships
  - Founding member lifetime pricing locks in early adopters
  - Focus on niche (business reporting trainers) vs. broad market
- **Contingency:** Double down on community engagement; emphasize founder story and direct support; compete on service not features
- **Owner:** Mario (competitive monitoring)

**Execution Risks**

**RISK 7: Timeline Slips Beyond 4 Weeks** ⚠️ **MEDIUM SEVERITY, MEDIUM-HIGH LIKELIHOOD**
- **Description:** Technical blockers, learning curve, or scope creep extends MVP development to 6-8 weeks
- **Impact:** Delayed revenue; increased opportunity cost; motivation risk; market window narrows
- **Mitigation:**
  - Ruthless scope discipline (use MVP "Out of Scope" list as guardrail)
  - Time-box learning (max 1 day per new technology)
  - Daily progress tracking (what shipped today?)
  - Accept "good enough" for MVP (polish in Alpha)
  - Buffer week built into timeline
- **Contingency:** If Week 3 shows <50% progress, cut additional features (reduce to 2 preset scenarios, simplify dashboard views)
- **Owner:** Mario (project management + scope control)

**RISK 8: Solo Founder Burnout** ⚠️ **MEDIUM SEVERITY, MEDIUM LIKELIHOOD**
- **Description:** 20-30 hours/week pace unsustainable; quality/health/relationships suffer; project abandoned
- **Impact:** Project stalls; incomplete MVP; no launch; wasted investment
- **Mitigation:**
  - Realistic timeline (not claiming "ship in 1 week")
  - Maintain work-life balance (weekends not 100% work)
  - Celebrate small wins (each completed section)
  - Accountability (share progress with family/friends)
  - Permission to take breaks
- **Contingency:** If burnout risk emerges, extend timeline by 1-2 weeks; recruit contractor for specific tasks (design, testing)
- **Owner:** Mario (self-awareness + support network)

**RISK 9: Customer Support Overwhelms Solo Founder** ⚠️ **MEDIUM SEVERITY, LOW LIKELIHOOD (MVP), MEDIUM (ALPHA)**
- **Description:** Support requests exceed 5 hours/week; product development stalls; quality suffers
- **Impact:** Slower iteration; customer dissatisfaction; churn increases; founder stress
- **Mitigation:**
  - Comprehensive FAQ on landing page (deflect common questions)
  - Email support only (no phone/chat commitment)
  - Response SLA: 24-48 hours (manage expectations)
  - Track common questions → improve product/documentation
  - Founding member tier benefits include "priority support" (not "instant support")
- **Contingency:** If support >10 hours/week, hire part-time VA for tier-1 support (€10-15/hour)
- **Owner:** Mario (support process + automation)

**Financial Risks**

**RISK 10: Operational Costs Exceed Revenue** ⚠️ **MEDIUM SEVERITY, LOW-MEDIUM LIKELIHOOD**
- **Description:** Claude API costs (€0.50-2 per generation) exceed revenue per customer; business loses money on each transaction
- **Impact:** Unsustainable unit economics; must raise prices or reduce costs; founding members grandfathered
- **Mitigation:**
  - Track generation costs closely from Day 1
  - Set rate limits for free tier (prevent abuse)
  - Optimize prompts to reduce token usage
  - Negotiate volume pricing with Anthropic if usage scales
  - Pricing tiers designed with margin buffer (€29-€999 >> €2 COGS)
- **Contingency:** If COGS >30% of revenue, implement usage caps or switch to cheaper LLM for simple scenarios
- **Owner:** Mario (financial monitoring + optimization)

**RISK 11: Low ARPU (Average Revenue Per User)** ⚠️ **MEDIUM SEVERITY, MEDIUM LIKELIHOOD**
- **Description:** 80%+ of founding members choose €29 tier; ARPU €40-50 instead of target €100-150
- **Impact:** Need 2-3x more customers for same revenue; CAC payback period extends; margins compress
- **Mitigation:**
  - Tier benefits clearly differentiate value (S vs M vs L)
  - Position €99 tier as "most popular" (anchoring)
  - Testimonials from M/L tier customers
  - Limit S tier feature set to incentivize upgrades
- **Contingency:** Adjust tiers in real-time based on first 20 purchases; raise prices for later cohorts
- **Owner:** Mario (pricing optimization)

### Open Questions

**Product Questions:**

1. **What is the optimal number of preset scenarios for MVP?**
   - Current plan: 3 (Bakery, Hotel, Tech Startup)
   - Question: Is 3 enough variety? Should we have 5? 7?
   - Decision point: Week 1 (based on development velocity)
   - Decision maker: Mario

2. **Should MVP include basic CSV export despite "out of scope" decision?**
   - Trade-off: Adds 1-2 days development but unlocks practical use cases
   - Risk: Trainers/professors may need export to share with students
   - Decision point: Week 3 (based on beta tester feedback)
   - Decision maker: Mario

3. **What chart library should we use: Recharts or Chart.js?**
   - Recharts: React-native, easier integration, larger bundle
   - Chart.js: Lightweight, Canvas-based, more setup
   - Decision point: Day 1 of frontend development
   - Decision maker: Mario (prototype both, choose based on performance)

**Market Questions:**

4. **What is the actual size of the IBCS© LinkedIn community?**
   - Research needed: Count #IBCS posts, engaged followers, active contributors
   - Timeline: Week 2
   - Owner: Mario

5. **Which user segment converts best: trainers, professors, or consultants?**
   - Hypothesis: Trainers convert fastest (acute pain, fast decision-making)
   - Validation: Track segment-specific conversion rates in first 50 customers
   - Timeline: Weeks 4-8
   - Owner: Mario

6. **What is realistic time savings for target users?**
   - Claim: "Several hours → minutes"
   - Question: Is it 2 hours? 4 hours? 8 hours? Varies by use case?
   - Validation: User interviews + 30-day follow-up survey
   - Timeline: Weeks 2-3 (interviews), Weeks 8-12 (actual usage data)
   - Owner: Mario

**Technical Questions:**

7. **What is the optimal AI prompt structure for consistent event generation?**
   - Options: Single mega-prompt vs. multi-stage generation
   - Current plan: Hybrid structured pipeline (context → trends → events)
   - Question: Does this actually work or is simpler approach better?
   - Validation: Week 1-2 prompt experimentation
   - Owner: Mario

8. **How do we handle AI generation failures gracefully?**
   - Scenarios: API timeout, rate limit, poor quality output, consistency validation fails
   - Question: Retry logic? Show error message? Fallback to cached scenario?
   - Decision point: Week 2 (error handling implementation)
   - Decision maker: Mario

9. **Should we use Prisma ORM or raw SQL?**
   - Prisma: Type-safe, migrations handled, slower queries
   - Raw SQL: Faster, more control, more error-prone
   - Decision point: Week 1 (schema implementation)
   - Decision maker: Mario (lean toward Prisma for type safety)

**Business Questions:**

10. **What is the right founding member cap: 300? 500? Unlimited?**
    - Trade-off: Scarcity (300) vs. revenue maximization (unlimited)
    - Question: Does artificial scarcity actually drive urgency?
    - Decision point: Week 4 (before launch)
    - Decision maker: Mario

11. **Should we offer refunds? What's the policy?**
    - Options: 30-day money-back guarantee, no refunds (lifetime deal), case-by-case
    - Question: Do refunds increase trust (higher conversion) or create support burden?
    - Decision point: Week 3 (before Stripe setup)
    - Decision maker: Mario

12. **How long should founding member pricing remain open?**
    - Options: First 300 members, first 90 days, until Alpha launch
    - Question: What creates more urgency: number cap or time limit?
    - Decision point: Week 4 (launch messaging)
    - Decision maker: Mario

### Areas Needing Further Research

**Before MVP Launch (Weeks 1-4):**

1. **Claude API tier limits and pricing tiers**
   - Action: Review API documentation, contact Anthropic if needed
   - Timeline: Week 1
   - Owner: Mario

2. **IBCS© community engagement patterns**
   - Action: Analyze #IBCS LinkedIn posts (frequency, engagement, key influencers)
   - Timeline: Week 2
   - Owner: Mario

3. **Competitive landscape monitoring**
   - Action: Set up Google Alerts, Product Hunt tracking, AngelList searches for "synthetic data" + "business data generation"
   - Timeline: Ongoing from Week 1
   - Owner: Mario

**During MVP Phase (Weeks 4-8):**

4. **User interview insights synthesis**
   - Action: Conduct 10-15 interviews, identify patterns, validate assumptions
   - Timeline: Weeks 2-4
   - Owner: Mario

5. **Beta tester feedback collection**
   - Action: Recruit 10-20 beta testers, gather structured feedback
   - Timeline: Weeks 4-6
   - Owner: Mario

6. **Landing page A/B testing**
   - Action: Test messaging variants, pricing presentation, CTA copy
   - Timeline: Weeks 4-8
   - Owner: Mario

**Post-Launch (Weeks 8+):**

7. **Cohort retention analysis**
   - Action: Track founding member engagement at 30/60/90 days
   - Timeline: Weeks 12-20
   - Owner: Mario

8. **Channel effectiveness comparison**
   - Action: Measure ROI of LinkedIn ads vs. organic vs. direct outreach
   - Timeline: Weeks 8-12
   - Owner: Mario

9. **IBCS© certification pathway research**
   - Action: Understand requirements, timeline, costs for official certification
   - Timeline: Months 6-12
   - Owner: Mario

---

## Appendices

### A. Supporting Strategic Documents

This Project Brief synthesizes insights from comprehensive strategic research and planning documents created during the discovery and analysis phase:

**1. Competitive Analysis (v2.0 - IBCS©-Compliant)**
- **Location:** `docs/competitor-analysis.md`
- **Summary:** Comprehensive analysis of the synthetic business data market identifying tellingCube as a blue ocean opportunity with no Priority 1 competitors. Analyzes 5 current alternatives (Excel, ChatGPT, BI samples, simulations, anonymized data) and defines clear differentiation strategy based on event-sourced multi-view consistency.
- **Key Findings:**
  - No direct competitors offering mathematically consistent multi-departmental views
  - Current alternatives all suffer from consistency gaps or high cost
  - IBCS© community provides accessible early adopter channel
  - Pricing positioned between manual Excel (time cost) and enterprise simulations ($5K-$50K/year)

**2. Go-to-Market Plan (v1.0 - IBCS©-Compliant)**
- **Location:** `docs/go-to-market-plan.md`
- **Summary:** Detailed 12-week launch strategy targeting 300 founding members with self-hosted early access model. Includes week-by-week timeline, channel strategy (LinkedIn primary), messaging templates, revenue forecasts (€5K-€140K in 12 weeks), and IBCS© community engagement approach.
- **Key Components:**
  - Landing page content and conversion funnel design
  - LinkedIn post templates (compliant with IBCS Institute guidelines)
  - Email outreach sequences for direct sales
  - Partnership pathway with IBCS Institute (future certification goal)
  - Founding member benefits and tier positioning

**3. Brainstorming Session Results (v2.0 - IBCS©-Compliant)**
- **Location:** `docs/brainstorming-session-results.md`
- **Summary:** Structured ideation session using 6 brainstorming techniques (First Principles, Morphological Analysis, SCAMPER, Analogical Thinking, Role Playing, Assumption Reversal) that generated 40+ ideas and validated core architectural decisions.
- **Key Decisions:**
  - Event-sourced architecture (7 event types × 5 dimensions) as foundational approach
  - Hybrid structured pipeline for AI generation (Option F)
  - Aggressive scope cuts (1 year vs. 5 years, 2 views vs. 4 views, 3 presets vs. custom prompts) to enable 2-4 week timeline
  - Self-hosted Kickstarter model with lifetime founding member pricing
  - Technical stack selection (Next.js, Vercel, NeonDB, Claude API)

**4. Original Concept Document**
- **Location:** `tellingCube.md` (parent directory)
- **Summary:** Initial concept document from discussion with IBCS Institute CEO Dr. Jürgen Faisst outlining the vision for an "AI-based magical cube" generating realistic multidimensional business data for demos, trainings, and events.
- **Core Insight:** IBCS©-compliant dashboards and reports require realistic, business-sound data that doesn't exist in current tools.

**5. AI Generation Strategy Analysis**
- **Location:** `answers.md` (parent directory)
- **Summary:** Detailed evaluation of 6 AI data generation approaches (one-shot, multi-stage, template-driven, constraint-based, story-first, hybrid pipeline) with trade-offs and recommendations.
- **Recommendation:** Hybrid Structured Pipeline (Option F) - combines AI context generation, time-series trends, granular events, validation, and corrective patching for best balance of creativity, structure, and mathematical soundness.

### B. Key Relationships & Resources

**IBCS Institute Partnership:**
- **Contact:** Dr. Jürgen Faisst (CEO, IBCS Institute)
- **Status:** Initial meeting completed; permission granted to use #IBCS hashtag and mention CEO name in compliant context
- **Trademark Compliance:** Use "inspired by IBCS©" language; pursue future IBCS© certification (6-12 month goal)
- **Community Access:** LinkedIn #IBCS community, IBCS© events and publications
- **Strategic Value:** Credibility signal, distribution channel, future partnership pathway

**Data Quality Validation Resource:**
- **Contact:** Mario's brother (controller in 250-300 person manufacturing company)
- **Role:** Financial scenario quality review, realism validation, controlling professional perspective
- **Availability:** Week 1-2 (technical PoC review), Week 4-6 (beta testing), ongoing ad-hoc consultation
- **Strategic Value:** "Controller-approved" credibility, catches unrealistic scenarios before launch, represents target user perspective

### C. Reference Materials

**Technology Documentation:**
- Next.js 14 Documentation: https://nextjs.org/docs
- Vercel Deployment Guide: https://vercel.com/docs
- NeonDB PostgreSQL: https://neon.tech/docs
- Anthropic Claude API: https://docs.anthropic.com/claude/reference
- Stripe Checkout: https://stripe.com/docs/checkout

**Market Research Sources:**
- IBCS Institute: https://www.ibcs.com
- LinkedIn #IBCS community analysis
- User interview transcripts (to be collected Weeks 2-4)

---

## Next Steps

### Immediate Actions (Week 1)

**Day 1-2: Technical Foundation**
1. Set up development environment
   - Initialize Next.js 14 project with TypeScript
   - Configure Vercel project and preview deployments
   - Set up NeonDB database and connection
   - Install core dependencies (TailwindCSS, chart library TBD, Prisma)

2. Design database schema
   - Implement single `events` table with wide schema
   - Create indexes for scenario_id, event_type, event_date
   - Test basic CRUD operations

3. Configure Claude API access
   - Verify API tier and rate limits
   - Set up API key securely (environment variables)
   - Test basic generation with sample prompt

**Day 3-5: AI Generation Proof of Concept**
4. Build minimal event generator
   - Create prompt template for bakery scenario
   - Generate 1 year of monthly business events
   - Validate output structure and completeness

5. Implement consistency validation
   - Build validation rules (revenue reconciliation, payroll alignment, COGS matching)
   - Test with generated bakery data
   - **Critical milestone:** Brother reviews output for financial realism

6. Generate Sales + Finance views
   - Write SQL queries to transform events → Sales metrics
   - Write SQL queries to transform events → Finance P&L
   - Verify mathematical consistency between views

**Success Criteria for Week 1:**
- ✅ Development environment fully operational
- ✅ Database schema implemented and tested
- ✅ AI can generate plausible 1-year bakery scenario
- ✅ Sales + Finance views reconcile perfectly
- ✅ Brother confirms "data looks realistic"

**Go/No-Go Decision Point:**
- If PoC successful → Proceed to full MVP build (Week 2-4)
- If quality issues → Iterate on prompts, consider template-driven approach
- If technical blockers → Extend timeline by 1 week, seek technical help if needed

---

### Week 2-3: MVP Development

**Frontend Development:**
7. Build landing page
   - Hero section with value proposition
   - 3 preset scenario buttons (Bakery, Hotel, Tech Startup)
   - Problem statement and solution explanation
   - Founding member pricing tier cards
   - FAQ section

8. Create dashboard view components
   - Sales view (revenue charts, product breakdown, customer table)
   - Finance view (P&L summary, cash flow, cost analysis)
   - Side-by-side layout
   - Responsive design (mobile, tablet, desktop)

**Backend Development:**
9. Implement API routes
   - `POST /api/generate` - scenario generation endpoint
   - `GET /api/scenario/:id` - retrieve scenario data
   - `GET /api/views/:scenarioId/:viewType` - view transformation
   - `POST /api/validate` - consistency validation

10. Complete AI prompt templates
    - Bakery scenario prompt (validated in Week 1)
    - Hotel scenario prompt
    - Tech Startup scenario prompt
    - Error handling and retry logic

**Integration:**
11. Stripe Checkout setup
    - Create 5 products (Supporter S/M/L, Team Plus, Department Partner)
    - Configure checkout flow
    - Test payment webhook
    - Email delivery (welcome email via Resend/SendGrid)

---

### Week 3-4: Validation & Launch Preparation

**Market Validation:**
12. Conduct user interviews (10-15 total)
    - 5 business reporting trainers
    - 5 university professors
    - 5 strategy consultants
    - Validate pain point, willingness to pay, feature priorities

13. IBCS© community research
    - Count LinkedIn #IBCS community size
    - Identify key influencers
    - Analyze engagement patterns
    - Draft initial outreach messages

**Beta Testing:**
14. Recruit 10-20 beta testers
    - Personal network, LinkedIn connections, IBCS© contacts
    - Provide early access to MVP
    - Collect structured feedback (survey)
    - Track usage analytics

**Landing Page Testing:**
15. Deploy landing page with email signup
    - Run LinkedIn ad campaign ($100-200 budget)
    - Measure email signup conversion
    - Track cost per lead
    - A/B test messaging if time permits

**Launch Preparation:**
16. Final polish and bug fixes
    - Address beta tester feedback
    - Performance optimization (page load <2s, generation <5min)
    - Error handling refinement
    - Content copywriting review

---

### Week 4: Soft Launch

**Day 1 (Launch Day):**
17. Deploy to production
    - Vercel production deployment
    - NeonDB production database
    - Stripe live mode activated
    - Monitoring (Sentry, analytics) enabled

18. Announce to personal network
    - LinkedIn post (personal profile)
    - Email to beta testers and interview participants
    - Direct outreach to 20-30 warm contacts

**Days 2-7 (Early Access Week 1):**
19. Monitor and respond
    - Track first generations (success rate, quality issues)
    - Respond to support emails (<24 hour SLA)
    - Fix critical bugs immediately
    - Celebrate first paying customer!

20. Expand outreach
    - Post to IBCS© LinkedIn community (value-adding content, not spam)
    - Engage with #IBCS posts
    - Direct message 50 IBCS© professionals
    - Monitor conversion funnel

---

### Weeks 5-8: Early Access Growth

**Weekly Rhythm:**
- Monday: Review metrics (signups, conversions, generations, support tickets)
- Tuesday-Thursday: Product improvements based on feedback
- Friday: Marketing content (LinkedIn posts, case studies, testimonials)
- Weekend: Outreach and community engagement

**Key Metrics to Track:**
- Founding member count (target: 20+ by Week 8)
- Landing page conversion rate (target: >10%)
- Email → purchase conversion (target: >30%)
- CAC (target: <€50)
- Generation success rate (target: >95%)
- User satisfaction (NPS target: >40)

**Go/No-Go Decision (End of Week 8):**
- ✅ **Proceed to Alpha** if: 20+ members, positive feedback, clear product-market fit signals
- 🟡 **Iterate** if: 5-19 members → improve messaging/positioning, extend early access
- ❌ **Reassess** if: <5 members → fundamental assumptions broken, consider pivot

---

### Handoff to Next Phase

**If proceeding to Alpha (Week 8+):**

This Project Brief provides the strategic foundation for tellingCube. The next phase requires:

1. **Alpha Feature Prioritization**
   - Review founding member feedback
   - Identify top 3-5 requested features
   - Create Alpha roadmap (user accounts, exports, HR/Controlling views, scenario library)

2. **Technical Architecture Evolution**
   - Assess performance bottlenecks
   - Plan database schema migrations
   - Evaluate need for materialized views
   - Implement authentication system

3. **Go-to-Market Scaling**
   - Analyze channel effectiveness (organic vs. paid)
   - Expand beyond IBCS© community if needed
   - Build referral program
   - Create case studies from early customers

4. **Business Model Refinement**
   - Analyze ARPU by tier
   - Adjust pricing based on early data
   - Plan enterprise SaaS transition
   - Explore IBCS© certification pathway

**Recommended Next Documents:**
- Alpha PRD (Product Requirements Document)
- Technical Architecture Document
- Marketing Playbook
- Customer Support Handbook

---

**Project Brief Complete**

**Created:** 2025-11-14
**Version:** 1.0
**Author:** Business Analyst Mary (with Mario)
**Status:** Ready for Execution

---

*This Project Brief serves as the comprehensive strategic foundation for tellingCube. All key decisions, assumptions, risks, and plans have been documented. It's time to build.*

🚀 **Let's ship this!**

---

## Addendum: Strategic Pivot (December 2025)

**Version:** 1.1
**Date:** 2025-12-18
**Source:** Direct feedback from Jürgen Faisst (CEO, IBCS Institute)
**Status:** Validated Market Insight - Requires PRD/Architecture Updates

---

### Executive Summary

Following direct feedback from Jürgen Faisst (IBCS Institute CEO) in December 2025, we have identified a critical **market positioning refinement**. While impressed by TellingCube's technical capabilities, Jürgen questioned commercial viability for the originally targeted audience (IBCS trainers). However, his feedback revealed an **untapped market opportunity** and validated a key pain point.

---

### Key Feedback Analysis

#### What Jürgen Said

| Feedback Point (German) | Translation | Strategic Implication |
|-------------------------|-------------|----------------------|
| "Angenehm überrascht" | Pleasantly surprised | Product exceeds expectations |
| "Für den Anfang sieht das recht vielversprechend aus" | Promising start | Technical validation from IBCS authority |
| "Skeptisch ob es außerhalb des Open Source Gedankens einen Markt gibt" | Skeptical about commercial viability outside open source | Challenge to original market assumptions |
| "Unsere Trainer reproduzieren [standardisierte] Schulungen" | Our trainers reproduce standardized curricula | Trainers NOT the primary market |
| "Datenanbindung in unserer virtuellen Organisation zu kompliziert" | Data connection too complex for their virtual organization | **CRITICAL INSIGHT: Data connectivity is the unsolved pain** |

#### Critical Discovery

The IBCS Institute **built their own enterprise data model** but **cannot use it** because data connection is too complex. This reveals:

1. The problem is NOT data model generation (that's already solved)
2. The problem IS making data connectivity effortless
3. If the IBCS Institute itself struggles with this, practitioners certainly do too

---

### Market Positioning Pivot

#### Previous Positioning (v1.0)

| Element | Original |
|---------|----------|
| **Target Market** | Business reporting trainers, professors, consultants |
| **Value Proposition** | Generate realistic business data in minutes instead of hours |
| **Core Problem** | Time spent creating demo data |
| **Key Differentiator** | Multi-view mathematical consistency |

#### Updated Positioning (v1.1)

| Element | Updated |
|---------|---------|
| **Target Market** | **IBCS practitioners post-certification** (CFOs, controllers, BI analysts, consultants implementing IBCS) |
| **Value Proposition** | **IBCS-compliant reporting made effortlessly simple** |
| **Core Problem** | **Data connectivity complexity** - the "last mile" problem |
| **Key Differentiator** | Zero-config data connection + IBCS compliance + multi-view consistency |

---

### Target Audience Refinement

#### Why Trainers Are NOT the Primary Market

Jürgen explained: IBCS trainers use standardized curricula with pre-built examples. They don't need custom data models because:
- Training materials are certified and consistent across trainers
- Existing examples are "good enough" for educational purposes
- Workshop+ is the only training variant with potential use cases

#### Who IS the Primary Market

**The 10,000+ IBCS certification alumni** who:
- Understand IBCS standards from their training
- Now need to **implement IBCS-compliant reports** in their organizations
- Struggle with the "last mile" - connecting data to IBCS-compliant visualizations
- Are CFOs, controllers, BI analysts, and consultants

**User Journey:**
```
1. Complete IBCS training → Understand the standards
2. Return to organization → Need to implement IBCS
3. Face reality → Data connection is complex, time-consuming
4. TellingCube → Solves the connectivity problem instantly
```

---

### Updated Value Proposition

#### Previous
> "Generate 1 year of realistic, multi-view business data in minutes instead of hours, with guaranteed mathematical consistency across all departmental perspectives."

#### Updated
> "IBCS-compliant business reports in 5 minutes. Upload your data, get instantly reconciled Sales, Finance, and Controlling views - no complex configuration required."

**The shift:** From "data generation speed" to "connectivity simplicity + IBCS compliance"

---

### Feature Gap Analysis (From Jürgen's Feedback)

| Feature Gap | Current State | Required Enhancement | Priority |
|-------------|---------------|----------------------|----------|
| **Multi-year support** | 1 year only | 3-5 years | HIGH |
| **Budget/Forecast data** | Not available | Plan vs. Actual comparisons | HIGH |
| **Hierarchies** | Flat dimensions | Multi-level hierarchies in org, products | MEDIUM |
| **HR domain** | Not available | Headcount, payroll, FTE views | MEDIUM |
| **ESG domain** | Not available | Sustainability metrics | LOW |
| **IBCS visualization** | Basic | Full IBCS notation compliance | HIGH |

---

### Action Items

#### Immediate (Week of 2025-12-18)

1. ✅ **Document SPA** → Strategic Party Analysis saved to `docs/marketing/launch/spa-ibcs-ceo-feedback-2025-12-18.md`
2. ☐ **Update PRD** → Incorporate positioning pivot and feature requirements
3. ☐ **Update Architecture** → Add "effortless data connectivity" as core design principle
4. ☐ **Create new epics** → Multi-year support, budgets/forecasts, hierarchies, domains

#### Short-term

5. ☐ **Follow up with Brian Julius** → Warm referral from Jürgen for market intelligence
6. ☐ **Await quote permission** → Request sent to use Jürgen's positive impressions in marketing
7. ☐ **Research IBCS certification alumni** → Validate practitioner market segment size

---

### Updated Assumptions

#### Validated Assumptions

| Assumption | Status | Evidence |
|------------|--------|----------|
| Technical capability impresses IBCS authority | ✅ VALIDATED | "Angenehm überrascht" from Jürgen |
| Multi-view consistency works | ✅ VALIDATED | Jürgen noted the completeness |
| Jürgen relationship is strategic asset | ✅ VALIDATED | Direct feedback, Brian Julius referral |

#### Revised Assumptions

| Previous Assumption | New Assumption | Validation Required |
|--------------------|----------------|---------------------|
| Trainers are primary market | Practitioners are primary market | Interview 5 IBCS alumni |
| Time savings is key value | Connectivity simplicity is key value | User testing with practitioners |
| 3 preset scenarios sufficient | Multi-year, budget/forecast needed | Feature usage analytics |

---

### Quote Permission Request

In the response to Jürgen, Mario requested permission to quote his positive impressions:

> "Dürfte ich Ihre positiven Eindrücke ('angenehm überrascht', 'vielversprechend') zitieren – etwa als 'Feedback vom IBCS Institute'?"

**Status:** Awaiting response

**Potential marketing value:**
- "IBCS Institute CEO: 'Pleasantly surprised by what has been created'"
- "Validated by the creators of IBCS standards"

---

### References

- **Original Feedback Email:** `docs/_masemIT/mail_ceo_ibcs.md`
- **Strategic Party Analysis:** `docs/marketing/launch/spa-ibcs-ceo-feedback-2025-12-18.md`
- **Brian Julius:** Referral contact for enterprise data model sources

---

**Addendum Author:** Strategic Party Mode (all agents)
**Review Status:** Pending Mario's confirmation before PRD updates

