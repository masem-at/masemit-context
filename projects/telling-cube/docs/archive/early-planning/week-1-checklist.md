# Week 1 Task Checklist - tellingCube MVP

**Goal:** Build technical proof of concept demonstrating event-sourced architecture generates realistic, consistent multi-view data

**Time Commitment:** 20-30 hours total (2-3 hours/weekday + 5-8 hours/weekend day)

**Critical Milestone:** Brother (controller) validates data quality and realism

---

## Pre-Week Setup

- [ ] Block calendar time for Week 1 (mark specific hours for development)
- [ ] Inform family of commitment this week
- [ ] Prepare workspace (quiet environment, minimize distractions)
- [ ] Review full Project Brief once (30 min read)
- [ ] Verify access to all required accounts:
  - [ ] Anthropic Claude API account
  - [ ] Vercel account
  - [ ] NeonDB account (sign up if needed)
  - [ ] GitHub account
  - [ ] Stripe account (for later, but verify access)

---

## Day 1-2: Technical Foundation (6-8 hours total)

### Environment Setup

- [ ] Create new GitHub repository `telling-cube`
  - [ ] Initialize with README.md
  - [ ] Add .gitignore for Next.js
  - [ ] Create `main` branch

- [ ] Initialize Next.js 14 project with TypeScript
  ```bash
  npx create-next-app@latest telling-cube --typescript --tailwind --app --no-src-dir
  cd telling-cube
  ```
  - [ ] Verify app runs locally (`npm run dev`)
  - [ ] Commit initial setup to Git

- [ ] Install core dependencies
  ```bash
  npm install @anthropic-ai/sdk
  npm install prisma @prisma/client
  npm install uuid
  npm install @types/uuid
  ```
  - [ ] Test imports work (no errors)

- [ ] Set up Vercel project
  - [ ] Connect GitHub repository to Vercel
  - [ ] Configure automatic deployments (main branch)
  - [ ] Verify preview deployment works
  - [ ] Note deployment URL

- [ ] Set up NeonDB database
  - [ ] Create new NeonDB project "tellingCube-dev"
  - [ ] Copy connection string
  - [ ] Add to `.env.local` file:
    ```
    DATABASE_URL="postgresql://..."
    CLAUDE_API_KEY="sk-ant-..."
    ```
  - [ ] Add `.env.local` to `.gitignore` (verify it's not committed!)

### Database Schema Implementation

- [ ] Initialize Prisma
  ```bash
  npx prisma init
  ```

- [ ] Create database schema in `prisma/schema.prisma`
  ```prisma
  model Event {
    id              String   @id @default(uuid())
    scenarioId      String   @map("scenario_id")
    eventType       String   @map("event_type")
    eventDate       DateTime @map("event_date")

    // Dimensions
    organizationUnit String?  @map("organization_unit")
    productId       String?  @map("product_id")
    counterpartyId  String?  @map("counterparty_id")
    assetId         String?  @map("asset_id")

    // Financial attributes
    amount          Float?
    quantity        Float?
    unitPrice       Float?  @map("unit_price")

    // Metadata
    eventNarrative  String?  @map("event_narrative")
    createdAt       DateTime @default(now()) @map("created_at")

    @@index([scenarioId])
    @@index([eventType])
    @@index([eventDate])
    @@map("events")
  }
  ```

- [ ] Run Prisma migration
  ```bash
  npx prisma migrate dev --name init
  ```
  - [ ] Verify migration succeeded (check NeonDB dashboard)

- [ ] Test basic database operations
  - [ ] Create test API route `app/api/test-db/route.ts`
  - [ ] Insert test event
  - [ ] Query test event
  - [ ] Delete test event
  - [ ] Verify CRUD works ✅

### Claude API Setup

- [ ] Verify Claude API key
  - [ ] Log in to Anthropic Console
  - [ ] Check API tier and rate limits
  - [ ] Copy API key to `.env.local`

- [ ] Test Claude API connection
  - [ ] Create test API route `app/api/test-claude/route.ts`
  - [ ] Send simple prompt: "Generate a list of 3 business events"
  - [ ] Log response
  - [ ] Verify API works ✅

**End of Day 1-2 Checkpoint:**
- ✅ Next.js app running locally
- ✅ Vercel deployment working
- ✅ NeonDB connected and tested
- ✅ Prisma migrations successful
- ✅ Claude API responding

**Time Check:** If >8 hours spent, acceptable (learning curve). If significantly over, simplify Day 3-5 scope.

---

## Day 3-5: AI Generation Proof of Concept (12-15 hours total)

### Prompt Engineering (3-4 hours)

- [ ] Create prompt template file `lib/prompts/bakery-scenario.ts`

- [ ] Draft initial bakery prompt (use hybrid pipeline approach)
  - [ ] Stage 1: Business context generation
    ```
    Generate a realistic bakery business context:
    - Business name and location
    - 10 employees (owner, bakers, sales staff)
    - 5 core products (bread, pastries, cakes, etc.)
    - 3 main suppliers (flour, dairy, packaging)
    - 20 regular customers
    Return as structured JSON.
    ```

  - [ ] Stage 2: Monthly time-series trends
    ```
    Based on the bakery context, generate 12 months of high-level trends:
    - Revenue trend (with seasonal variations)
    - Employee changes (hires, terminations)
    - Supplier cost trends
    Return as monthly summary JSON.
    ```

  - [ ] Stage 3: Granular event generation
    ```
    Generate detailed business events for each month:
    - Employee lifecycle events (hire, terminate, hours worked)
    - Sales transactions (product, quantity, customer, price)
    - Supplier purchases (ingredient, quantity, cost)
    - Cash movements (customer payments, supplier payments, payroll)
    Ensure all events reconcile mathematically.
    Return as events array JSON.
    ```

- [ ] Test prompts in Claude API manually
  - [ ] Run Stage 1 → verify context makes sense
  - [ ] Run Stage 2 → verify trends are plausible
  - [ ] Run Stage 3 → verify events look realistic
  - [ ] Iterate prompts until output quality good

### Event Generator Implementation (4-5 hours)

- [ ] Create event generator utility `lib/generators/event-generator.ts`
  - [ ] Function: `generateBakeryScenario()`
  - [ ] Call Claude API with 3-stage prompts
  - [ ] Parse JSON responses
  - [ ] Transform to Prisma Event format
  - [ ] Return array of events

- [ ] Create API route `app/api/generate/route.ts`
  - [ ] POST endpoint accepts `{ scenarioType: "bakery" }`
  - [ ] Call `generateBakeryScenario()`
  - [ ] Insert events to database using Prisma
  - [ ] Return scenario ID

- [ ] Test generation end-to-end
  - [ ] Call `/api/generate` via Postman or curl
  - [ ] Verify ~150-300 events inserted (12 months × ~12-25 events/month)
  - [ ] Check NeonDB dashboard for data
  - [ ] Verify no errors ✅

### Consistency Validation (2-3 hours)

- [ ] Create validation utility `lib/validators/consistency-validator.ts`

- [ ] Implement validation rules:
  - [ ] Revenue Reconciliation Rule
    ```typescript
    // Finance total revenue should equal sum of all sales events
    financeRevenue === sumOfSalesEvents(amount)
    ```

  - [ ] Payroll Alignment Rule
    ```typescript
    // Finance payroll expense should equal sum of employee hours × rates
    financePayroll === sumOfEmployeeHours(quantity × unitPrice)
    ```

  - [ ] COGS Matching Rule
    ```typescript
    // Finance COGS should equal sum of procurement events for materials
    financeCOGS === sumOfProcurementEvents(amount, type='material')
    ```

- [ ] Create validation API route `app/api/validate/route.ts`
  - [ ] POST endpoint accepts `{ scenarioId }`
  - [ ] Run all validation rules
  - [ ] Return `{ valid: boolean, errors: [] }`

- [ ] Test validation
  - [ ] Generate scenario
  - [ ] Run validation
  - [ ] Verify passes (or identify issues to fix in prompts)

### View Generation (4-5 hours)

- [ ] Create view query utilities `lib/queries/view-queries.ts`

- [ ] Implement Sales View SQL
  ```sql
  -- Revenue by month
  SELECT
    DATE_TRUNC('month', event_date) as month,
    SUM(amount) as revenue
  FROM events
  WHERE scenario_id = ? AND event_type = 'sale'
  GROUP BY month
  ORDER BY month;

  -- Revenue by product
  SELECT
    product_id,
    SUM(amount) as revenue,
    SUM(quantity) as units_sold
  FROM events
  WHERE scenario_id = ? AND event_type = 'sale'
  GROUP BY product_id;
  ```
  - [ ] Write Prisma query equivalents
  - [ ] Test queries return data

- [ ] Implement Finance View SQL
  ```sql
  -- P&L Summary
  SELECT
    SUM(CASE WHEN event_type = 'sale' THEN amount ELSE 0 END) as revenue,
    SUM(CASE WHEN event_type = 'payroll' THEN amount ELSE 0 END) as payroll,
    SUM(CASE WHEN event_type = 'procurement' THEN amount ELSE 0 END) as cogs
  FROM events
  WHERE scenario_id = ?;

  -- Monthly cash flow
  SELECT
    DATE_TRUNC('month', event_date) as month,
    SUM(CASE WHEN event_type IN ('customer_payment') THEN amount ELSE 0 END) as cash_in,
    SUM(CASE WHEN event_type IN ('supplier_payment', 'payroll') THEN amount ELSE 0 END) as cash_out
  FROM events
  WHERE scenario_id = ?
  GROUP BY month;
  ```
  - [ ] Write Prisma query equivalents
  - [ ] Test queries return data

- [ ] Create view API routes
  - [ ] `app/api/views/sales/route.ts`
  - [ ] `app/api/views/finance/route.ts`
  - [ ] Both accept `scenarioId` query param
  - [ ] Return formatted JSON

- [ ] Test view endpoints
  - [ ] Generate scenario
  - [ ] Call `/api/views/sales?scenarioId=...`
  - [ ] Call `/api/views/finance?scenarioId=...`
  - [ ] Verify data returned ✅

### Manual Consistency Check (1 hour)

- [ ] Generate complete bakery scenario
- [ ] Export Sales view data
- [ ] Export Finance view data
- [ ] Manually verify in Excel/Google Sheets:
  - [ ] Total sales revenue matches between views
  - [ ] Employee count × avg hours × rate ≈ payroll expense
  - [ ] Procurement costs align with COGS
- [ ] Document any discrepancies
- [ ] Fix prompts/queries if needed

**End of Day 3-5 Checkpoint:**
- ✅ AI generates 1 year of bakery events
- ✅ Events stored in database
- ✅ Sales + Finance views queryable
- ✅ Basic consistency validation passing
- ✅ Generation time <5 minutes

---

## 🎯 CRITICAL MILESTONE: Brother Review

### Schedule Review Session (1-2 hours)

- [ ] Contact brother and schedule 30-60 min review session
- [ ] Prepare materials:
  - [ ] Generate 2-3 bakery scenarios
  - [ ] Export Sales view (revenue by month, by product)
  - [ ] Export Finance view (P&L, cash flow)
  - [ ] Prepare questions:
    - "Do the revenue numbers look realistic for a 10-person bakery?"
    - "Does the P&L structure make sense?"
    - "Do you see any obvious red flags (margins, growth rates, ratios)?"
    - "Would you trust this data if you saw it in a workshop?"

- [ ] Conduct review session
  - [ ] Walk through generated scenario
  - [ ] Show Sales view
  - [ ] Show Finance view
  - [ ] Demonstrate consistency (revenue matches, payroll aligns)
  - [ ] Ask for feedback
  - [ ] Take notes on issues/concerns

- [ ] Document feedback
  - [ ] Create file `docs/week-1-validation.md`
  - [ ] Record brother's assessment: Realistic? Issues? Recommendations?
  - [ ] Note specific changes needed (if any)

**Success Criteria:**
- ✅ Brother confirms: "Data looks realistic enough for demo purposes"
- ✅ No critical quality issues (impossible margins, nonsensical patterns)
- ✅ Multi-view consistency impressed him

**If feedback is negative:**
- [ ] Identify root cause (prompt quality, validation gaps, data structure)
- [ ] Iterate on prompts (add more business rules, constraints)
- [ ] Consider template-driven approach if AI quality insufficient
- [ ] Schedule follow-up review after fixes

---

## End of Week 1: Success Criteria Check

### Technical Success ✅
- [ ] Next.js + Vercel + NeonDB + Claude API all operational
- [ ] Database schema implemented and tested
- [ ] AI generates plausible 1-year bakery scenario
- [ ] Generation completes in <5 minutes
- [ ] Sales + Finance views render from same event data
- [ ] Views reconcile perfectly (validation passing)
- [ ] Code committed to GitHub

### Quality Success ✅
- [ ] Brother (controller) confirms data looks realistic
- [ ] No obvious hallucinations or impossible numbers
- [ ] P&L structure makes sense
- [ ] Cash flow patterns plausible

### Decision: Proceed to Full MVP Build? ✅

**If ALL criteria met:**
- ✅ **GO:** Proceed to Week 2-3 (full MVP development)
- Document success in `docs/week-1-validation.md`
- Celebrate! You've proven the core concept works 🎉

**If quality issues exist:**
- 🟡 **ITERATE:** Extend Week 1 by 2-3 days, fix prompts, re-validate with brother
- Document issues and iteration plan

**If technical blockers exist:**
- 🟡 **EXTEND:** Add 1 week buffer, seek technical help if needed
- Reassess scope (can we simplify further?)

**If fundamental concept broken:**
- ❌ **PIVOT:** Event-sourced approach not working; consider template-driven alternative
- Schedule deep-dive session to reassess architecture

---

## Week 1 Wrap-Up Tasks

- [ ] Update Project Brief with any learnings from Week 1
- [ ] Commit all code to GitHub (verify nothing sensitive committed)
- [ ] Back up database (NeonDB snapshot if available)
- [ ] Document any technical decisions made during implementation
- [ ] Share progress with family (show them something working!)
- [ ] Plan Week 2 tasks based on Week 1 outcomes
- [ ] Rest! Take evening off before Week 2 starts

---

## Notes & Learnings

*Use this space to capture insights, challenges, solutions as you work through Week 1*

**Day 1-2 Notes:**
- What took longer than expected?
- What was easier than expected?
- Any tools/libraries that were particularly helpful?

**Day 3-5 Notes:**
- Prompt engineering challenges?
- Claude API surprises (good or bad)?
- Database query performance observations?

**Brother Review Notes:**
- Key feedback:
- Surprises (positive or negative):
- Action items from review:

**Overall Week 1 Reflections:**
- Total time spent:
- Biggest win:
- Biggest challenge:
- Confidence level for Week 2 (1-10):

---

**Week 1 Status:** [ ] Complete | [ ] In Progress | [ ] Needs Extension

**Ready for Week 2?** [ ] Yes | [ ] Not Yet

🚀 **Let's build!**
