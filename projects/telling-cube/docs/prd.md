# tellingCube Product Requirements Document (PRD)

**Document Version:** 2.0
**Created:** 2025-11-14
**Last Updated:** 2026-01-21
**Status:** Big Bang Launch - Phase 1 Ready

> **REPOSITIONING NOTICE (2026-01-21):** This PRD has been updated to reflect the new positioning. tellingCube is now positioned as a "Realistic Business Data Generator" - NOT an "IBCS tool." See `docs/product-brief.md` for full positioning details.

---

## Goals and Background Context

### Goals

The tellingCube Big Bang Launch aims to deliver the following outcomes:

- **Enable rapid synthetic data generation**: Users can generate 1-5 years of realistic, multi-departmental business data in under 5 minutes
- **Guarantee cross-departmental consistency**: All views (Sales, Finance, HR) derive from the same event stream, ensuring mathematical reconciliation - "The numbers always add up"
- **Validate new positioning**: Test "Realistic business data that actually reconciles" messaging with target audiences (BI Consultants, Software Vendors, Trainers, Developers)
- **Achieve market expansion**: Move beyond IBCS niche to broader synthetic data market (millions vs thousands)
- **Establish foundation for scaling**: Dashboard, auth, async generation, multi-year scenarios, HR domain
- **Deliver phased rollout**: Phase 1 (Copy) → Phase 2 (Auth+Dashboard) → Phase 3 (Async+Pricing) → Phase 4 (Features)

### Background Context

tellingCube addresses a universal problem in synthetic business data: **data that reconciles**.

**The Problem:**
- AI (ChatGPT, Claude) hallucinates numbers that don't reconcile
- Random generators produce obvious garbage
- Real data has compliance/NDA issues
- Manual creation in Excel takes 3-4 hours and is error-prone

**The Solution:**
tellingCube's **event-sourced architecture** generates a unified stream of business events. All departmental views (Finance, Sales, HR) query this single source of truth. When Finance shows €50K payroll, HR shows the exact employees that generated that cost. Every € is traceable to a source event.

**Market Validation:**
- Brian Julius (55K LinkedIn followers) called it "the ivory-billed woodpecker of the data world"
- Jürgen Faisst (IBCS Institute CEO) said it's "relevant for software vendors and BI consultants"
- Early users signed up immediately citing "we all struggle for synthetic data to build POCs"

**Target Audiences (Priority Order):**
1. BI Consultants - Client demos, POCs, prototypes
2. Software Vendors - Product demos that look real
3. Trainers/Educators - Workshops with realistic scenarios
4. Developers/QA - Test data without production data risks
5. Data Scientists - Statistically valid synthetic data for ML

### Change Log

| Date | Version | Description | Author |
|------|---------|-------------|--------|
| 2025-11-14 | 1.0 | Initial PRD draft for tellingCube MVP | John (PM) |
| 2025-12-18 | 1.1 | Strategic pivot based on Jürgen Faisst (IBCS CEO) feedback | John (PM) + Party Mode |
| 2026-01-21 | 2.0 | **MAJOR REPOSITIONING** - From "IBCS tool" to "Realistic Business Data Generator". New target audiences, messaging, Big Bang launch phases. | Mary (BA) + Party Mode |

---

### Strategic Repositioning Reference (v2.0)

**Source:** Brian Julius video + market analysis (January 2026)
**Full Analysis:** See `docs/tellingcube-strategic-briefing.md` and `docs/tellingcube-repositioning-strategy.md`

**Key Changes in v2.0:**

| Aspect | Previous (v1.1) | Updated (v2.0) |
|--------|-----------------|----------------|
| **Primary Target** | IBCS practitioners | BI Consultants, Software Vendors, Trainers, Developers |
| **Core Problem** | IBCS implementation complexity | Synthetic data doesn't reconcile |
| **Value Proposition** | IBCS compliance made simple | Realistic business data that actually reconciles |
| **IBCS Role** | Primary positioning | Feature only (bonus, not product) |
| **Market Size** | ~10,000 IBCS practitioners | Millions of data professionals |

**Critical Insight:** Brian Julius (55K followers) bought tellingCube and made a video. He never mentioned IBCS. He said: "The numbers add up." That's what people actually buy. IBCS was solving the right problem for the wrong audience.

**Evidence:**
- Brian Julius: "One of the things I've always had trouble with is developing realistic data that's consistent."
- Jürgen Faisst (publicly): "Relevant for software vendors and BI consultants"
- Koteswara Rao: "We all struggle for synthetic data to build POCs" - signed up immediately

---

## Requirements

### Functional Requirements

**FR1:** The system shall provide three preset industry scenario options: Bakery, Hotel, and Tech Startup, accessible via dedicated generation buttons on the landing page.

**FR2:** The system shall generate 1 year of business events (monthly granularity) using an event-sourced architecture with 7 event types: Employee Lifecycle, Sales Transactions, Cash Movements, Operational Work, Procurement & Expenses, Asset Lifecycle, and Planning & Budgeting.

**FR3:** The system shall generate events across 5 dimensions: Time (12 months), Organization (simplified structure), Product (3-5 products per scenario), Counterparty (customers/suppliers/employees), and Asset (if applicable to scenario).

**FR4:** The system shall provide a Sales View displaying: revenue by month (line chart), revenue by product (bar chart), top customers (table), and sales trend analysis.

**FR5:** The system shall provide a Finance View displaying: P&L summary (revenue, costs, profit), monthly cash flow, cost breakdown by category, and margin analysis.

**FR6:** The system shall perform automated mathematical consistency validation ensuring: Finance total revenue equals sum of Sales events, Finance payroll expense equals sum of Employee hours × rates, and Finance COGS equals sum of Procurement events for materials.

**FR7:** The system shall display a "✓ Multi-view consistency verified" indicator after successful generation and validation.

**FR8:** The system shall operate in ephemeral mode with session-based data storage, requiring no user authentication or account creation for scenario generation.

**FR9:** The system shall integrate Stripe Checkout for founding member purchases across 5 pricing tiers: Supporter S (€29), Supporter M (€99), Supporter L (€299), Team Plus (€599), and Department Partner (€999).

**FR10:** The system shall send a confirmation email to founding members after successful payment, including access instructions and welcome message.

**FR11:** The system shall provide a single-page landing page with the following sections: Hero (value proposition), Problem statement, Solution demo (3 scenario buttons), Multi-view consistency proof (side-by-side dashboards), Founding member CTA with pricing tiers, and FAQ.

**FR12:** The system shall display generated scenarios in-browser with interactive charts and tables, without requiring data export or download for MVP.

**FR13:** The system shall automatically clear session-based scenario data after 24 hours to minimize storage costs.

**FR14:** The system shall use Claude API with structured prompts optimized for each industry scenario to generate realistic, contextually appropriate business events.

---

### Functional Requirements - Roadmap (v1.1 Addition)

*The following requirements are derived from Jürgen Faisst (IBCS CEO) feedback and represent the next phase of development after MVP validation. They are documented here to inform architecture decisions and epic planning.*

**FR15:** The system shall support multi-year scenario generation (3-5 years) with configurable time granularity (monthly or quarterly).
- *Priority:* HIGH
- *Rationale:* Jürgen noted the need for "mehr Jahre" (more years) to support realistic long-term business planning scenarios.
- *Dependency:* Requires database schema consideration during architecture phase.

**FR16:** The system shall generate Plan vs. Actual data including budgets, forecasts, and variance analysis.
- *Priority:* HIGH
- *Rationale:* Core IBCS use case is comparing planned vs. actual figures; current MVP only generates actual data.
- *Dependency:* Requires additional event types (PLAN_BUDGET, FORECAST) in event schema.

**FR17:** The system shall support multi-level hierarchies in structural dimensions (organization units, product categories, cost centers).
- *Priority:* MEDIUM
- *Rationale:* Jürgen noted need for "Hierarchien in den strukturellen Dimensionen" for realistic enterprise scenarios.
- *Dependency:* May require normalized dimension tables vs. flat event schema.

**FR18:** The system shall provide an HR domain view displaying headcount trends, payroll analysis, FTE calculations, and workforce metrics.
- *Priority:* MEDIUM
- *Rationale:* Mentioned as gap in current scope; extends multi-view concept beyond Sales/Finance.
- *Dependency:* HR view queries can leverage existing employee lifecycle events.

**FR19:** The system shall provide an ESG (Environmental, Social, Governance) domain view with sustainability metrics.
- *Priority:* LOW
- *Rationale:* Emerging requirement noted by Jürgen; not critical for initial practitioner adoption.
- *Dependency:* Requires new event types and dimension attributes for ESG tracking.

**FR20:** The system shall implement full IBCS® notation compliance in all visualizations including semantic notation (actual/plan/forecast indicators), unification guidelines, and proper chart selection.
- *Priority:* HIGH
- *Rationale:* Jürgen noted need for "Feinschliff für die IBCS-Visualisierung" (refinement for IBCS visualization).
- *Dependency:* Requires chart component updates and IBCS standards reference implementation.

---

### Non-Functional Requirements

**NFR1:** Performance - The system shall complete scenario generation (from prompt submission to viewable results) in under 5 minutes for 90% of requests, with a target of under 3 minutes.

**NFR2:** Performance - The landing page shall load in under 2 seconds on desktop and mobile devices.

**NFR3:** Reliability - The system shall successfully generate valid scenarios at a rate of 95% or higher, with automatic retry mechanisms for transient failures.

**NFR4:** Data Quality - The system shall achieve a 100% pass rate on automated consistency validation checks (zero reconciliation errors between views).

**NFR5:** Data Quality - The system shall generate scenarios rated "realistic" or "very realistic" (4-5 stars) by 80% or more of beta testers and validation experts.

**NFR6:** Security - The system shall not persist user-generated data beyond session scope, and shall not implement authentication or user accounts in MVP phase.

**NFR7:** Scalability - The system shall handle up to 100 concurrent users without performance degradation, supporting the Week 12 target of 100-300 founding members.

**NFR8:** Cost Efficiency - Claude API usage shall remain within budgeted constraints of €0.50-€2.00 per generation, with monitoring and circuit breakers to prevent runaway costs.

**NFR9:** Responsive Design - The system shall provide a fully responsive user interface optimized for desktop (1920×1080), tablet (768×1024), and mobile (375×667) viewports.

**NFR10:** Availability - The system shall be deployed on Vercel with target uptime of 99%+ during MVP phase, leveraging Vercel's default SLA.

**NFR11:** Maintainability - The codebase shall use TypeScript for type safety, follow Next.js 14 App Router conventions, and include inline comments for complex event generation and validation logic.

**NFR12:** Observability - The system shall log generation times, validation results, and error rates to enable performance monitoring and quality assessment during MVP validation period.

---

## User Interface Design Goals

### Overall UX Vision

tellingCube's user experience prioritizes **immediate value demonstration** over feature richness. Users should understand within 10 seconds that tellingCube solves their data reconciliation problem.

**Core UX Principles:**
- **Lead with the problem**: "Synthetic data is garbage. AI hallucinates. Real data has compliance issues."
- **Instant proof**: Explore curated examples without signup - see the data quality immediately
- **Trust through consistency**: Show the "✓ Data reconciles" badge prominently
- **Professional aesthetics**: Clean, modern design signaling B2B SaaS quality
- **Clear value proposition**: "3 minutes instead of 3 hours"

**Key Messaging on Landing Page:**
- **Hero:** "Realistic Business Data That Actually Reconciles"
- **Subline:** "Generate consistent financial scenarios in minutes. Finance, Sales, HR—every number adds up."
- **Social Proof:** Brian Julius quote prominently displayed
- **CTA Primary:** "Explore Examples" (free, no signup)
- **CTA Secondary:** "Generate Your Own - €9"

The experience should immediately communicate: "This is the tool that makes your demo data look real."

### Key Interaction Paradigms

**1. One-Click Generation (Primary Interaction)**
- Large, prominent scenario buttons on landing page: "Generate Bakery Scenario" | "Generate Hotel Scenario" | "Generate Tech Startup Scenario"
- Single click triggers generation (no configuration required)
- Loading state with progress indicator and estimated time remaining
- Automatic transition to results view when complete

**2. Passive Data Consumption (Results Interaction)**
- Users scroll through pre-rendered charts and tables (no configuration UI)
- Side-by-side layout enables instant visual comparison between Sales and Finance views
- Hover states on charts reveal detailed tooltips with exact values
- No "edit" or "customize" interactions (out of MVP scope)

**3. Call-to-Action Conversion (Monetization Interaction)**
- After viewing results, prominent CTA: "Want unlimited access? Become a Founding Member"
- Pricing tier cards with clear differentiation (seats, features, price)
- Click tier → Stripe Checkout modal → payment → confirmation
- Non-intrusive (users can freely generate scenarios without payment wall)

**4. Mobile-Responsive Adaptation**
- Desktop: Side-by-side view comparison (Sales left, Finance right)
- Mobile: Stacked views with sticky tabs for quick switching
- Charts scale responsively while maintaining readability

### Core Screens and Views

**1. Landing Page / Homepage**
- Hero section with value proposition headline
- Brief problem/solution copy (2-3 sentences)
- Three large scenario generation buttons (CTA buttons, primary action)
- "How it works" section with visual diagram (optional, below fold)
- Founding member pricing tiers (below fold)
- FAQ accordion (bottom of page)

**2. Generation/Loading State**
- Full-screen overlay or dedicated loading page
- Animated progress indicator (spinner or progress bar)
- Status text: "Generating 1 year of bakery data..." → "Creating Sales view..." → "Validating consistency..."
- Estimated time remaining: "~45 seconds remaining"
- Cancel button (returns to homepage, aborts generation)

**3. Results View (Split Dashboards)**
- **Desktop Layout:** Side-by-side split (50/50)
  - Left panel: Sales View
  - Right panel: Finance View
  - Top bar: Scenario info (Bakery, 10 employees, 12 months) + consistency badge
- **Content Structure (each view):**
  - View title (e.g., "Sales View")
  - 3-4 charts (line charts, bar charts)
  - 1-2 summary tables
  - Key metrics cards (total revenue, average, etc.)
- **Bottom CTA:** Founding Member call-to-action banner

**4. Checkout/Payment Page (Stripe Hosted)**
- Stripe Checkout handles all payment UI (out of our scope)
- Redirect from our site → Stripe → redirect back with confirmation

**5. Confirmation Page**
- Thank you message
- "You're a Founding Member!" headline
- Access instructions: "Generate unlimited scenarios anytime"
- Email confirmation notice
- CTA: "Generate Another Scenario"

### Accessibility: None (MVP Constraint)

**MVP Decision:** No specific accessibility compliance (WCAG AA/AAA) for initial launch due to timeline and resource constraints.

**Rationale:**
- MVP timeline (2-4 weeks) cannot accommodate full accessibility audit and remediation
- Target users (business professionals) predominantly use desktop in professional settings
- Solo developer lacks specialized accessibility expertise

**Post-MVP Commitment:**
- **Alpha Phase (Weeks 8-12):** Basic accessibility review (keyboard navigation, color contrast, alt text)
- **Beta Phase (Months 3-6):** WCAG AA compliance for core user flows
- **Enterprise Phase:** Full WCAG AA compliance to serve university customers (academic institutions often require accessibility)

**MVP Accessibility Baseline:**
- Semantic HTML5 (headings, sections, buttons)
- Sufficient color contrast for readability (not formally audited)
- Keyboard-navigable buttons and links (browser defaults)

### Branding

**Brand Identity:**
- **Name:** tellingCube (lowercase "t", capital "C" — memorable, slightly quirky)
- **Primary Tagline:** "Realistic business data that actually reconciles"
- **Secondary Taglines:**
  - "3 minutes instead of 3 hours"
  - "AI hallucinates. tellingCube doesn't."
  - "Every € is traceable to a source event"

**Color Palette:**
- **Primary color:** #2563EB (blue) — trust, data, technology
- **Secondary color:** #E5EAF5 (light gray/white-blue) — backgrounds, subtle elements
- **Accent color:** #10B981 (turquoise green) — innovation, freshness, success states
- **Additional colors:**
  - **Error/Warning:** #EF4444 (red) — validation errors, failed generation
  - **Neutral text:** #1F2937 (dark gray) — body text
  - **Muted text:** #6B7280 (medium gray) — secondary text, labels
  - **Border/Divider:** #E5E7EB (light gray) — card borders, separators
  - **White:** #FFFFFF — card backgrounds, main content areas

**Typography:**
- **Primary font:** Inter (recommended) — clean, modern, excellent readability for data-heavy UIs
- **Fallback font:** IBM Plex Sans — equally professional alternative
- **Font weights:**
  - Headings: 600-700 (Semibold/Bold)
  - Body text: 400 (Regular)
  - Emphasis/labels: 500 (Medium)
- **Scale:**
  - H1: 48px (Hero headline)
  - H2: 36px (Section titles)
  - H3: 24px (Card titles, view names)
  - Body: 16px (Main content)
  - Small: 14px (Labels, metadata)

**Icon/Visual Element:**
- **Concept:** Stylized cube grid or data cube representing multi-dimensional data
- **Usage:** Favicon, loading spinner, consistency badge, email signature
- **Style:** Minimalist, geometric, works in monochrome and color

**Chart Color Palette (for data visualizations):**
- **Primary series:** #2563EB (blue) — main data line/bar
- **Secondary series:** #10B981 (green) — comparison data
- **Tertiary series:** #8B5CF6 (purple) — third data series if needed
- **Quaternary series:** #F59E0B (amber) — fourth series
- **Positive indicator:** #10B981 (green) — profit, growth
- **Negative indicator:** #EF4444 (red) — loss, decline

**Button Styles:**
- **Primary action (CTA):** #2563EB background, white text, hover: #1E40AF (darker blue)
- **Secondary action:** #E5EAF5 background, #2563EB text, hover: #D1D9EB
- **Success state:** #10B981 background, white text
- **Destructive action:** #EF4444 background, white text

**Logo (MVP):**
- **Placeholder:** Text-based "tellingCube.com" using Inter font
- **Post-MVP:** Custom logo design with cube grid icon

**Brand Voice:**
- Professional but human (not corporate-stiff)
- Direct and benefit-focused (data reconciles, save time)
- Slightly technical (target users understand business terminology)
- Confident: "AI hallucinates. We don't."
- Authentic: 3% to charity, built by a solo founder

**IBCS Mention Guidelines:**
- ✅ "Charts follow IBCS® visualization standards" (small print, feature)
- ❌ "IBCS-ready business data" (not acceptable)
- ❌ IBCS in headlines or primary messaging (not acceptable)

### Target Device and Platforms: Web Responsive

**Primary Platform:** Web browser (desktop and mobile)

**Desktop (Primary Focus):**
- **Target resolution:** 1920×1080 (most common)
- **Minimum supported:** 1366×768
- **Browsers:** Chrome, Firefox, Safari, Edge (latest 2 versions)
- **Rationale:** Target users (trainers, consultants, professors) primarily work on laptops/desktops in professional settings

**Mobile (Important, Must Be Fully Functional):**
- **Target devices:** iPhone (375×667 and up), Android (360×640 and up)
- **Layout adaptation:**
  - Stacked views with sticky tab navigation for switching between Sales/Finance
  - Touch-optimized chart interactions
  - Collapsible tables for narrow screens
- **Performance considerations:**
  - Optimize chart rendering for mobile (fewer data points if needed)
  - Lazy-load below-the-fold content
  - Minimize JavaScript bundle size
- **Rationale:** Users frequently share scenarios with colleagues on mobile, review data while traveling, or use tablets during workshops; mobile experience must be polished and fully functional, not just "acceptable"

**Responsive Breakpoints:**
- **Mobile:** < 640px (single column, stacked views)
- **Tablet:** 640px - 1024px (may show side-by-side in landscape)
- **Desktop:** > 1024px (full side-by-side layout)

**Tablets:**
- iPad and Android tablets fully supported
- Landscape mode: Side-by-side views (desktop-like)
- Portrait mode: Stacked views with tab switching (mobile-like)

**Out of Scope:**
- Native mobile apps (iOS, Android)
- Desktop applications (Electron)
- Browser extensions
- Offline functionality

---

## Technical Assumptions

These technical decisions will guide the Architect and constrain implementation choices. All decisions are optimized for MVP speed, solo developer productivity, and cost efficiency.

### Repository Structure: Monorepo

**Decision:** Single monorepo containing frontend, backend, and database schema in one repository.

**Structure:**
```
/telling-cube
  /app              (Next.js 14 App Router)
    /api            (API routes: generation, views, validation)
    /(routes)       (Page routes: home, results, confirmation)
  /components       (React components: charts, cards, layouts)
  /lib              (Shared utilities: AI prompts, validation, queries)
  /prisma           (Database schema, migrations)
  /public           (Static assets: images, fonts)
  /types            (TypeScript type definitions)
  package.json
  tsconfig.json
  tailwind.config.js
  next.config.js
```

**Rationale:**
- **Simplified deployment:** Single Vercel deployment for entire application
- **Shared TypeScript types:** Event schema, API contracts shared between frontend/backend
- **Faster iteration:** No need to coordinate changes across multiple repos
- **Solo developer productivity:** Single codebase easier to manage for one person
- **Immediate code reuse:** Utilities and types accessible across all layers

### Service Architecture: Monolith

**Decision:** Monolithic Next.js 14 application with frontend and backend API routes in a single deployment.

**Architecture Components:**
- **Frontend:** React components (Server Components + Client Components)
- **Backend:** Next.js API routes (`/app/api/*`)
  - `/api/generate` - Scenario generation endpoint
  - `/api/views/sales` - Sales view queries
  - `/api/views/finance` - Finance view queries
  - `/api/validate` - Consistency validation
  - `/api/stripe/webhook` - Payment webhooks
- **Database:** Direct Prisma Client connections from API routes to NeonDB
- **AI Generation:** Claude API called directly from `/api/generate` route

**Rationale:**
- **Simplicity:** No service orchestration, API gateways, or inter-service communication
- **Performance:** No network hops between services (single process)
- **Cost:** Single deployment on Vercel (no multiple container/function costs)
- **Adequate scale:** Handles 100-300 users easily; Vercel serverless scales automatically
- **Fast MVP delivery:** No microservices architecture complexity

**Evolution Path:**
- **Alpha+:** Extract AI generation to separate serverless function if generation time/cost becomes bottleneck
- **Enterprise:** Separate API service for white-label customers requiring API-only access

### Testing Requirements: Unit + Integration (Pragmatic Approach)

**Decision:** Pragmatic testing strategy focused on critical paths and data consistency.

**Testing Layers:**

**1. Unit Tests (High Priority):**
- **Validation logic:** Mathematical consistency checks (Revenue reconciliation, Payroll alignment, COGS matching)
  - Test framework: Jest
  - Coverage target: 100% for validation functions (these are critical!)
  - Example: `lib/validators/consistency-validator.test.ts`
- **Query utilities:** View aggregation logic (Sales queries, Finance queries)
  - Coverage target: 80%+ for query functions
  - Example: `lib/queries/view-queries.test.ts`
- **Event transformations:** Prisma model mappings
  - Coverage target: 70%+

**2. Integration Tests (Medium Priority):**
- **API route testing:** End-to-end API behavior
  - Framework: Next.js test helpers + Supertest
  - Test scenarios:
    - `/api/generate` with mocked Claude API → returns scenario ID
    - `/api/views/sales` → returns correct aggregated data
    - `/api/validate` → catches inconsistencies
  - Coverage: Core happy paths + critical error cases
- **Database integration:** Real Prisma queries against test database
  - Use in-memory SQLite or test NeonDB instance
  - Verify complex aggregations work correctly

**3. Manual Testing (High Priority for MVP):**
- **Scenario quality validation:** Human review of generated data (Brother + beta testers)
- **UI/UX testing:** Click through entire user journey on desktop + mobile
- **Cross-browser testing:** Chrome, Firefox, Safari, Edge (manual spot-checks)
- **Payment flow:** Stripe test mode end-to-end checkout

**4. E2E Tests (Post-MVP):**
- **Deferred to Alpha:** Playwright or Cypress for full user journey automation
- **Rationale:** High maintenance cost, slow to write, manual testing sufficient for MVP velocity

**Testing Practices:**
- **TDD for validation logic:** Write tests first for mathematical consistency rules
- **Test data factories:** Reusable event generators for tests
- **CI/CD integration:** GitHub Actions runs tests on every PR
- **No test coverage mandates:** Focus on high-risk areas, not arbitrary % targets

**What's NOT tested in MVP:**
- ❌ UI component unit tests (React Testing Library deferred to Alpha)
- ❌ Visual regression tests (Percy/Chromatic deferred)
- ❌ Load/performance tests (deferred until scale issues appear)
- ❌ Security penetration tests (basic security review only)

**Rationale:**
- **Risk-based approach:** Test what matters most (data consistency, core flows)
- **Solo developer efficiency:** Full testing pyramid takes weeks; pragmatic approach takes days
- **Quality gate:** Brother's validation + beta testing catches issues automated tests would miss
- **Fast iteration:** Don't let test writing slow MVP delivery

### Additional Technical Assumptions and Requests

**Frontend Framework & Libraries:**
- **Framework:** Next.js 14+ (App Router, React Server Components)
- **Styling:** TailwindCSS with custom config for brand colors
- **Component library:** shadcn/ui (unstyled, customizable components) or Headless UI
- **Charts:** Recharts or Chart.js (lightweight, responsive)
- **Forms:** React Hook Form (minimal re-renders, good DX)
- **HTTP client:** Native `fetch` API (no axios needed for Next.js)

**Backend & API:**
- **Runtime:** Node.js 18+ (Vercel default)
- **Language:** TypeScript (strict mode enabled)
- **ORM:** Prisma (type-safe database access, migration management)
- **Validation:** Zod (runtime type validation for API inputs)
- **API design:** RESTful JSON endpoints (no GraphQL for MVP)

**Database:**
- **Provider:** NeonDB (serverless PostgreSQL)
- **Schema strategy:** Single wide `events` table (see Database Schema in FR2)
- **Indexes:** scenario_id, event_type, event_date
- **Migrations:** Prisma Migrate (version-controlled schema changes)
- **Connection pooling:** Prisma's built-in pooling + NeonDB connection pooler

**AI/LLM Integration:**
- **Provider:** Anthropic Claude API (Sonnet 3.5 or Claude 4)
- **Prompt strategy:** 3-stage structured pipeline
  1. Generate business context (company details, entities)
  2. Generate monthly trends (revenue patterns, headcount changes)
  3. Generate granular events (individual transactions)
- **Fallback:** OpenAI GPT-4 (Alpha+ for redundancy)
- **Estimated cost:** €0.50-€2 per generation
- **Rate limiting:** Circuit breaker pattern if API fails repeatedly

**Payment Processing:**
- **Provider:** Stripe Checkout (hosted payment pages)
- **Products:** 5 pricing tiers pre-configured in Stripe Dashboard
- **Webhooks:** `/api/stripe/webhook` for payment confirmation
- **Email:** Resend for confirmation emails

**Deployment & Infrastructure:**
- **Hosting:** Vercel (Pro tier €20/month for production)
- **Domain:** tellingcube.com (to be registered)
- **SSL:** Automatic via Vercel
- **CDN:** Vercel Edge Network (automatic)
- **Environment variables:** Managed via Vercel dashboard
  - `DATABASE_URL` (NeonDB connection string)
  - `CLAUDE_API_KEY` (Anthropic API key)
  - `STRIPE_SECRET_KEY` (Stripe secret)
  - `STRIPE_WEBHOOK_SECRET` (Stripe webhook signing)
  - `RESEND_API_KEY` (Email service)

**Monitoring & Observability:**
- **Error tracking:** Sentry (free tier: 5K events/month)
- **Analytics:** Plausible (privacy-friendly) + Vercel Analytics
- **Logging:** Vercel Logs (basic) or Axiom (if needed)
- **Performance monitoring:** Vercel Web Vitals
- **Uptime monitoring:** UptimeRobot (free tier)

**Development Tools:**
- **Version control:** Git + GitHub
- **CI/CD:** GitHub Actions (run tests, deploy to Vercel)
- **Code quality:** ESLint + Prettier (enforced via pre-commit hooks)
- **Type checking:** TypeScript strict mode
- **Package manager:** npm or pnpm (pnpm preferred for speed)

**Security & Compliance:**
- **HTTPS:** Enforced by Vercel
- **Rate limiting:** 100 requests/min per IP (Vercel Edge Config or custom middleware)
- **Input validation:** Zod schemas for all API inputs
- **SQL injection prevention:** Prisma parameterized queries (never raw SQL)
- **XSS prevention:** React's automatic escaping + CSP headers
- **CORS:** Restricted to own domain
- **Session management:** httpOnly cookies, 7-day expiration
- **PCI compliance:** Delegated to Stripe (no card data stored)

**Data Management:**
- **Ephemeral data:** Scenarios deleted after 24 hours (cron job or lazy cleanup)
- **Backups:** NeonDB automatic daily backups
- **Data export (post-MVP):** CSV/Excel generation for paid users

**Performance Budgets:**
- **Landing page load:** <2 seconds (mobile 3G)
- **Generation time:** <5 minutes (90% of cases)
- **Chart rendering:** <500ms for initial display
- **JavaScript bundle:** <200KB gzipped
- **Database query time:** <200ms for view aggregations

**Browser Support:**
- **Desktop:** Chrome, Firefox, Safari, Edge (last 2 versions)
- **Mobile:** iOS Safari 15+, Chrome Android 100+
- **No support:** IE11, Opera Mini

**Accessibility (Minimum Baseline):**
- Semantic HTML5
- Keyboard navigation (browser defaults)
- Sufficient color contrast (informal check)
- Alt text for images
- ARIA labels for interactive elements (where needed)

---

## Epic List (Big Bang Launch - v2.0)

> **Note:** The original Epics 1-3 (Foundation, Dashboards, Monetization) have been COMPLETED. The MVP is live. The following epics represent the Big Bang Launch phases.

### Epic 5: Phase 1 - Copy & Messaging Update
**Goal:** Update all user-facing copy to reflect the new positioning ("Realistic business data that actually reconciles"). Remove IBCS from primary positioning. This is a LOW-RISK phase with zero technical changes to the core engine.

**Timeline:** This Week (1-2 days)

### Epic 6: Phase 2 - Auth & Dashboard
**Goal:** Implement Magic Link authentication, user dashboard with scenario history, and tier display. Enable users to save and revisit their generated scenarios.

**Timeline:** 1-2 weeks

### Epic 7: Phase 3 - Async Generation & Pricing
**Goal:** Implement QStash-based async generation, new Stripe products for 5 pricing tiers, email notifications, and progress tracking. Enable longer generations without timeout issues.

**Timeline:** 1 week

### Epic 8: Phase 4 - New Features
**Goal:** Add HR domain, multi-year logic (1/3/5 years), hoki.help donation counter, and coupon system. Complete the "Lifetime+" tier features.

**Timeline:** 1-2 weeks

---

## Original Epics (MVP - Completed)

> **Status:** These epics were completed in the MVP phase (Nov-Dec 2025). They are preserved here for reference.

### Epic 1: Foundation & Event Generation Engine (COMPLETED)
**Goal:** Establish project infrastructure (Next.js, database, CI/CD) and implement core AI-powered event generation for three preset scenarios, delivering the ability to generate realistic 1-year business event streams.

### Epic 2: Multi-View Dashboards & Consistency Proof (COMPLETED)
**Goal:** Transform raw event data into interactive Sales and Finance views with automated mathematical consistency validation, proving the core differentiator of "one reality, multiple perspectives."

### Epic 3: User Journey & Monetization (COMPLETED)
**Goal:** Complete the end-to-end user experience from landing page to founding member conversion, including responsive design, payment flow, and production polish.

### Epic 4: Advanced Features (v1.1 - PARTIALLY COMPLETED)
**Goal:** Extended scenarios (9 company profiles), IBCS-inspired visualizations, data export features. Some features moved to Phase 4 (HR domain, multi-year).

---

## Epic 1: Foundation & Event Generation Engine

**Goal:** Establish project infrastructure (Next.js, database, CI/CD) and implement core AI-powered event generation for three preset scenarios. By the end of this epic, the system can generate complete, validated event datasets for Bakery, Hotel, and Tech Startup scenarios—proving technical feasibility and enabling Week 1-2 validation with your brother.

### Story 1.1: Project Foundation & Deployment Pipeline

**As a** developer,
**I want** a fully configured Next.js 14 project with TypeScript, TailwindCSS, and automated Vercel deployment,
**so that** I have a solid foundation to build features and can deploy changes continuously.

#### Acceptance Criteria

1. Next.js 14 project initialized with App Router, TypeScript, and TailwindCSS configured with brand colors (#2563EB, #E5EAF5, #10B981)
2. Git repository created with `.gitignore` excluding `.env.local`, `node_modules`, and `.next`
3. GitHub repository connected to Vercel with automatic deployments on push to `main` branch
4. Landing page route (`/`) displays "tellingCube.com" text placeholder and renders successfully at deployed Vercel URL
5. `.env.local.example` file created documenting required environment variables (DATABASE_URL, CLAUDE_API_KEY, etc.)

---

### Story 1.2: Database Schema & Connection Setup

**As a** developer,
**I want** a PostgreSQL database with Prisma ORM and the events table schema,
**so that** I can store and query business event data efficiently.

#### Acceptance Criteria

1. NeonDB project created and connection string stored in `.env.local` (not committed to Git)
2. Prisma initialized with `schema.prisma` containing Event model matching FR2 specification (id, scenarioId, eventType, eventDate, dimensions, financial attributes, metadata)
3. Initial Prisma migration created and applied successfully to NeonDB (events table exists with indexes on scenario_id, event_type, event_date)
4. Test API route (`/api/test-db`) successfully inserts a sample event, queries it back, and returns the result as JSON
5. Database connection error handling implemented (returns meaningful error if DATABASE_URL missing or invalid)

---

### Story 1.3: Claude API Integration & Test

**As a** developer,
**I want** Claude API integrated with proper error handling and retry logic,
**so that** I can reliably call the AI generation service for scenario creation.

#### Acceptance Criteria

1. `@anthropic-ai/sdk` installed and configured with API key from environment variable `CLAUDE_API_KEY`
2. Utility function `lib/ai/claude-client.ts` created that accepts prompt and returns Claude response with TypeScript types
3. Test API route (`/api/test-claude`) sends simple prompt ("Generate a list of 3 business events") and returns Claude's response
4. Error handling implemented for API failures (network errors, rate limits, invalid API key) with user-friendly error messages
5. Circuit breaker pattern implemented: after 3 consecutive failures, stop retrying and return error (prevents runaway API costs)

---

### Story 1.4: Bakery Scenario Prompt Engineering

**As a** developer,
**I want** a 3-stage prompt pipeline that generates realistic bakery business context, trends, and events,
**so that** Claude produces structured, consistent event data for the Bakery scenario.

#### Acceptance Criteria

1. Prompt template file `lib/prompts/bakery-scenario.ts` created with three stage prompts:
   - Stage 1: Business context generation (bakery name, 10 employees, 5 products, 3 suppliers, 20 customers) returning structured JSON
   - Stage 2: Monthly trends (12 months revenue trend, employee changes, supplier costs) returning monthly summary JSON
   - Stage 3: Granular events (employee lifecycle, sales, procurement, payments, hours worked) returning events array JSON
2. Utility function `lib/generators/bakery-generator.ts` executes all 3 stages sequentially, passing context between stages
3. JSON parsing implemented with error handling for malformed Claude responses (retry with adjusted prompt if parse fails)
4. Output transformation implemented: Claude JSON → Prisma Event model format (matching schema from Story 1.2)
5. Manual test demonstrates: calling bakery generator returns 150-300 events for 1 year with realistic values (validated by console log review)

---

### Story 1.5: Event Storage & Scenario Management

**As a** developer,
**I want** functions to store generated events and manage scenarios by unique ID,
**so that** generated data persists and can be retrieved for viewing.

#### Acceptance Criteria

1. Utility function `lib/data/event-repository.ts` created with `insertEvents(scenarioId: string, events: Event[])` method using Prisma batch insert
2. Scenario ID generation implemented using UUID v4 (unique identifier for each generation)
3. Function `getEventsByScenario(scenarioId: string)` retrieves all events for a given scenario, ordered by event_date
4. Function `deleteScenarioEvents(scenarioId: string)` removes all events for a scenario (for cleanup/testing)
5. Test demonstrates: inserting 200 events, retrieving them by scenario ID, and verifying count and order are correct

---

### Story 1.6: Bakery Scenario Generation API Endpoint

**As a** user,
**I want** an API endpoint that generates a complete Bakery scenario,
**so that** I can trigger scenario creation and receive a scenario ID to view results.

#### Acceptance Criteria

1. API route `/api/generate` created accepting POST requests with JSON body `{ scenarioType: "bakery" }`
2. Endpoint calls `generateBakeryScenario()` from Story 1.4, receives events array
3. Generated events stored in database using `insertEvents()` from Story 1.5 with new scenario ID
4. API returns JSON response: `{ scenarioId: "uuid", eventCount: number, status: "success", generatedAt: timestamp }`
5. Generation completes in under 5 minutes for 90% of requests (logged for monitoring), with timeout after 5 minutes returning error

---

### Story 1.7: Hotel & Tech Startup Scenario Generation

**As a** user,
**I want** Hotel and Tech Startup scenarios available alongside Bakery,
**so that** I can generate diverse business data across different industries.

#### Acceptance Criteria

1. Prompt template `lib/prompts/hotel-scenario.ts` created with 3-stage pipeline adapted for hotel business (rooms, bookings, housekeeping staff, F&B operations)
2. Prompt template `lib/prompts/tech-startup-scenario.ts` created with 3-stage pipeline adapted for SaaS startup (subscriptions, developers, cloud costs, churn)
3. `/api/generate` endpoint updated to accept `scenarioType: "bakery" | "hotel" | "techStartup"` and route to appropriate generator
4. All three scenarios successfully generate 150-300 events each when tested via API calls
5. Manual review confirms Hotel and Tech Startup data is contextually appropriate (hotel has room bookings not product sales; startup has MRR not physical inventory)

---

### Story 1.8: Mathematical Consistency Validation

**As a** system,
**I want** automated validation that ensures generated events are mathematically consistent,
**so that** multi-view reconciliation is guaranteed before presenting data to users.

#### Acceptance Criteria

1. Validation function `lib/validators/consistency-validator.ts` created with three checks:
   - Revenue Reconciliation: Sum of all 'sale' event amounts equals expected total revenue
   - Payroll Alignment: Sum of 'payroll' event amounts aligns with employee count × hours × rates
   - COGS Matching: Sum of 'procurement' events (materials only) equals expected cost of goods sold
2. Function `validateScenario(scenarioId: string)` runs all checks and returns `{ valid: boolean, errors: string[] }`
3. `/api/generate` endpoint automatically calls validation after event insertion; if validation fails, logs errors and returns warning (but still returns scenario ID for debugging)
4. Unit tests created covering: valid scenario passes all checks, invalid scenario (manually corrupted data) fails with specific error messages
5. Test coverage for validation logic reaches 100% (critical business logic)

---

## Epic 2: Multi-View Dashboards & Consistency Proof

**Goal:** Transform raw event data into interactive Sales and Finance views with automated mathematical consistency validation. By the end of this epic, users can view side-by-side dashboards with charts and tables, and the system displays the "✓ Multi-view consistency verified" badge—proving the core differentiator of "one reality, multiple perspectives."

### Story 2.1: Sales View Data Layer & API

**As a** developer,
**I want** SQL query functions and an API endpoint that aggregate event data into Sales view metrics,
**so that** I can provide revenue, product, and customer analysis data to the frontend.

#### Acceptance Criteria

1. Query utility file `lib/queries/sales-queries.ts` created with functions:
   - `getRevenueByMonth(scenarioId)`: Returns array of `{ month: Date, revenue: number }` aggregated from 'sale' events
   - `getRevenueByProduct(scenarioId)`: Returns array of `{ productId: string, revenue: number, unitsSold: number }`
   - `getTopCustomers(scenarioId, limit: number)`: Returns top N customers by total revenue
   - `getSalesTrendAnalysis(scenarioId)`: Returns month-over-month growth rates
2. All query functions use Prisma with proper type safety (no raw SQL in MVP)
3. API route `/api/views/sales` created accepting GET request with query param `scenarioId`
4. Endpoint returns JSON with structure: `{ revenueByMonth: [], revenueByProduct: [], topCustomers: [], trendAnalysis: {} }`
5. Manual test with generated scenario returns realistic aggregated data (verified via Postman or curl)

---

### Story 2.2: Finance View Data Layer & API

**As a** developer,
**I want** SQL query functions and an API endpoint that aggregate event data into Finance view metrics,
**so that** I can provide P&L, cash flow, and cost analysis data to the frontend.

#### Acceptance Criteria

1. Query utility file `lib/queries/finance-queries.ts` created with functions:
   - `getPLSummary(scenarioId)`: Returns `{ revenue: number, payroll: number, cogs: number, profit: number }`
   - `getMonthlyCashFlow(scenarioId)`: Returns array of `{ month: Date, cashIn: number, cashOut: number, netCash: number }`
   - `getCostBreakdown(scenarioId)`: Returns cost categories (payroll, procurement, other) with amounts
   - `getMarginAnalysis(scenarioId)`: Returns gross margin, net margin percentages
2. All query functions use Prisma with proper aggregations (SUM, GROUP BY)
3. API route `/api/views/finance` created accepting GET request with query param `scenarioId`
4. Endpoint returns JSON with structure: `{ plSummary: {}, monthlyCashFlow: [], costBreakdown: {}, marginAnalysis: {} }`
5. Manual test verifies Finance revenue matches Sales revenue (consistency check from Epic 1 validation)

---

### Story 2.3: Chart Library Setup & Base Components

**As a** developer,
**I want** a chart library configured with reusable base components styled with brand colors,
**so that** I can quickly build consistent visualizations across Sales and Finance views.

#### Acceptance Criteria

1. Recharts library installed and configured (or Chart.js if preferred for lighter bundle size)
2. Base chart components created in `components/charts/`:
   - `LineChart.tsx`: Accepts data prop, renders responsive line chart with #2563EB primary color
   - `BarChart.tsx`: Accepts data prop, renders responsive bar chart with brand color palette
   - `DataTable.tsx`: Accepts columns and rows, renders sortable table with TailwindCSS styling
3. Chart components use Inter font and brand colors (#2563EB, #10B981, #E5EAF5)
4. Charts are responsive (adapt to container width) and display tooltips on hover showing exact values
5. Storybook or standalone test page demonstrates all three components with sample data

---

### Story 2.4: Sales View Dashboard UI

**As a** user,
**I want** a Sales dashboard displaying revenue charts and customer tables,
**so that** I can analyze sales performance from the generated scenario.

#### Acceptance Criteria

1. React component `components/views/SalesView.tsx` created that:
   - Fetches data from `/api/views/sales` using scenario ID
   - Displays loading state while fetching (spinner or skeleton)
   - Renders four visualizations:
     - Revenue by Month (LineChart component)
     - Revenue by Product (BarChart component)
     - Top Customers (DataTable component)
     - Sales Trend Analysis (text summary with month-over-month % change)
2. Component displays "Sales View" heading with view icon
3. Key metrics cards shown at top: Total Revenue, Average Monthly Revenue, Total Units Sold
4. Error handling implemented: if API call fails, display user-friendly error message
5. Manual test with generated Bakery scenario shows realistic sales data with charts rendering correctly

---

### Story 2.5: Finance View Dashboard UI

**As a** user,
**I want** a Finance dashboard displaying P&L, cash flow, and cost breakdowns,
**so that** I can analyze financial performance from the generated scenario.

#### Acceptance Criteria

1. React component `components/views/FinanceView.tsx` created that:
   - Fetches data from `/api/views/finance` using scenario ID
   - Displays loading state while fetching
   - Renders four visualizations:
     - P&L Summary (card with revenue, costs, profit breakdown)
     - Monthly Cash Flow (LineChart with dual lines: cash in vs. cash out)
     - Cost Breakdown by Category (BarChart showing payroll, COGS, other expenses)
     - Margin Analysis (cards showing gross margin %, net margin %)
2. Component displays "Finance View" heading with view icon
3. Color coding implemented: positive values (profit, cash in) use #10B981 green, negative/costs use neutral colors
4. Error handling implemented: if API call fails, display user-friendly error message
5. Manual test with generated Bakery scenario shows P&L that reconciles with Sales view revenue

---

### Story 2.6: Results Page with Side-by-Side Layout

**As a** user,
**I want** a results page showing Sales and Finance views side-by-side,
**so that** I can compare both perspectives simultaneously and understand multi-view consistency.

#### Acceptance Criteria

1. Page route `/results/[scenarioId]` created using Next.js dynamic routing
2. Desktop layout (>1024px): Side-by-side 50/50 split with SalesView on left, FinanceView on right
3. Scenario metadata displayed at top: scenario type (Bakery/Hotel/Tech Startup), generation date, time period (1 year, 12 months)
4. Both views load simultaneously (parallel data fetching) for optimal performance
5. Page is accessible via direct URL (e.g., `/results/abc-123-uuid`) and renders correctly when shared

---

### Story 2.7: Consistency Badge & Validation Display

**As a** user,
**I want** a visible indicator that multi-view consistency has been verified,
**so that** I trust the data quality and understand tellingCube's unique value proposition.

#### Acceptance Criteria

1. Consistency badge component `components/ConsistencyBadge.tsx` created displaying "✓ Multi-view consistency verified" with green (#10B981) checkmark icon
2. Badge prominently displayed at top of results page (above or between views)
3. API endpoint `/api/validate` from Epic 1 Story 1.8 called when results page loads
4. If validation passes: show green badge; if validation fails: show warning badge with "⚠ Validation issues detected" in amber color
5. Tooltip or expandable section explains what consistency means: "Revenue, payroll, and costs reconcile perfectly across Sales and Finance views"

---

### Story 2.8: Mobile Responsive Adaptation

**As a** user,
**I want** the results page to work seamlessly on mobile devices,
**so that** I can review generated scenarios on my phone or tablet.

#### Acceptance Criteria

1. Mobile layout (<640px): Stacked views with sticky tab navigation at top ("Sales" | "Finance" tabs)
2. Tapping tab switches between full-screen SalesView and FinanceView (smooth transition)
3. Charts scale responsively: line/bar charts maintain readability on narrow screens (minimum 320px width)
4. Tables adapt to mobile: horizontal scroll for wide tables or responsive columns that stack on small screens
5. Manual testing on iPhone (375×667) and Android (360×640) confirms all charts, tables, and interactions work smoothly without layout breaking

---

## Epic 3: User Journey & Monetization

**Goal:** Complete the end-to-end user experience from landing page to founding member conversion, including responsive design, payment flow, and production polish. By the end of this epic, the MVP is production-ready and can convert visitors to paying customers—enabling the Week 4 soft launch and founding member acquisition.

### Story 3.1: Landing Page Hero & Scenario Selection

**As a** visitor,
**I want** a clear landing page explaining tellingCube's value with easy access to scenario generation,
**so that** I understand what the product does and can try it immediately.

#### Acceptance Criteria

1. Home page route (`/`) created with hero section containing:
   - Headline: "Realistic business data in minutes, not hours"
   - Subheadline: 2-3 sentence value proposition explaining time savings and multi-view consistency
   - Text logo placeholder: "tellingCube.com" styled with Inter font and #2563EB color
2. Three large scenario generation buttons displayed prominently:
   - "Generate Bakery Scenario" (primary button style)
   - "Generate Hotel Scenario" (primary button style)
   - "Generate Tech Startup Scenario" (primary button style)
3. Clicking any scenario button triggers POST request to `/api/generate` with appropriate scenarioType and navigates to loading page
4. Responsive layout: buttons stack vertically on mobile (<640px), display in row on desktop (>1024px)
5. Page loads in under 2 seconds on desktop (verified with Lighthouse or WebPageTest)

---

### Story 3.2: Generation Loading State & Progress Indicator

**As a** user,
**I want** visual feedback during scenario generation showing progress and estimated time,
**so that** I understand the system is working and don't abandon the process.

#### Acceptance Criteria

1. Loading page route `/generate/loading` created with:
   - Full-screen centered layout with tellingCube.com logo
   - Animated spinner or progress indicator (using #2563EB and #10B981 colors)
   - Status text that updates: "Generating 1 year of [scenario] data..." → "Creating Sales view..." → "Validating consistency..."
   - Estimated time remaining: "~45 seconds remaining" (dynamically calculated)
2. Page polls `/api/generate` endpoint (if using async generation) or shows loading state until API returns scenarioId
3. On successful generation: automatically redirects to `/results/[scenarioId]`
4. Cancel button displayed at bottom: returns user to homepage and aborts generation (optional for MVP, can log cancellation)
5. Error handling: if generation fails or times out (>5 minutes), display user-friendly error with "Try Again" button

---

### Story 3.3: Problem & Solution Copy Section

**As a** visitor,
**I want** to read a brief explanation of the problem tellingCube solves,
**so that** I understand why I should use this tool and how it helps me.

#### Acceptance Criteria

1. Section added to landing page (below hero, above scenario buttons or below buttons):
   - **Problem statement:** 2-3 sentences describing pain of manual data creation for trainers/consultants/professors
   - **Solution statement:** 2-3 sentences explaining event-sourced architecture and multi-view consistency benefit
2. Section uses clean typography (Inter font) with sufficient spacing and readability
3. Optional visual diagram or icon illustrating "One reality, multiple perspectives" concept (can be simple CSS/SVG, not required to be complex illustration)
4. Responsive: text stacks vertically on mobile, may have side-by-side layout on desktop if diagram is included
5. Copy is concise and benefit-focused (not technical jargon), reviewed for clarity

---

### Story 3.4: Stripe Checkout Integration

**As a** visitor,
**I want** to purchase a founding member tier via secure payment,
**so that** I can get unlimited scenario access and support the product.

#### Acceptance Criteria

1. Stripe account configured with 5 products (Supporter S: €29, M: €99, L: €299, Team Plus: €599, Department Partner: €999)
2. API route `/api/stripe/create-checkout` created accepting POST with `{ tier: "supporterS" | "supporterM" | ... }`
3. Endpoint creates Stripe Checkout Session and returns checkout URL
4. User clicking tier button on landing page calls API and redirects to Stripe hosted checkout page
5. Stripe success URL redirects to `/confirmation` page with session_id parameter; cancel URL redirects back to landing page

---

### Story 3.5: Founding Member Pricing & CTA Section

**As a** visitor,
**I want** to see founding member pricing tiers and benefits,
**so that** I can choose the right tier and become a paying customer.

#### Acceptance Criteria

1. Pricing section added to landing page (below problem/solution section or at bottom):
   - Section heading: "Become a Founding Member"
   - Subheading explaining lifetime access and grandfathered pricing
2. Five pricing tier cards displayed with responsive grid (stack on mobile, 2-3 columns on tablet, 3-5 columns on desktop):
   - Each card shows: tier name, price (€29, €99, €299, €599, €999), seat count, key features
   - Primary CTA button on each card: "Get [Tier Name]" triggers Stripe checkout from Story 3.4
3. Founding member benefits listed: Unlimited generations, priority support, early access to new features, lifetime grandfathered pricing
4. Visual hierarchy: cards use #2563EB for headers, #10B981 for CTA buttons
5. Mobile responsive: cards stack vertically on narrow screens with touch-friendly button sizing

---

### Story 3.6: Payment Confirmation & Thank You Page

**As a** founding member,
**I want** a confirmation page after successful payment,
**so that** I know my payment succeeded and understand how to access the product.

#### Acceptance Criteria

1. Confirmation page route `/confirmation` created accepting `session_id` query parameter
2. Page displays:
   - "You're a Founding Member!" headline with celebratory styling
   - Thank you message confirming purchase
   - Access instructions: "Generate unlimited scenarios anytime by visiting tellingcube.com"
   - "Check your email for confirmation and receipt" message
3. Primary CTA button: "Generate Your First Scenario" links back to homepage
4. Page verifies Stripe session_id is valid by calling Stripe API; if invalid, shows error
5. Basic styling matches landing page (consistent branding, responsive layout)

---

### Story 3.7: Email Confirmation via Resend

**As a** founding member,
**I want** to receive a confirmation email after payment,
**so that** I have a receipt and access instructions in my inbox.

#### Acceptance Criteria

1. Resend configured with API key in environment variables (`RESEND_API_KEY`)
2. Email template created with:
   - Subject: "Welcome to tellingCube - Founding Member Confirmed"
   - Body: Thank you message, tier purchased, amount paid, access instructions ("Visit tellingcube.com anytime")
   - Footer: Support email contact
3. Stripe webhook endpoint `/api/stripe/webhook` created to listen for `checkout.session.completed` event
4. On successful payment event: extract customer email from Stripe session and send confirmation email via Resend
5. Manual test in Stripe test mode: complete checkout → confirmation email received within 1 minute

---

### Story 3.8: FAQ Section

**As a** visitor,
**I want** answers to common questions about tellingCube,
**so that** I can make an informed decision without contacting support.

#### Acceptance Criteria

1. FAQ section added to landing page (at bottom, above footer):
   - Section heading: "Frequently Asked Questions"
2. Accordion component displays 5-7 common questions:
   - "What is tellingCube?" → Brief product explanation
   - "How does multi-view consistency work?" → Explanation of event-sourced architecture in plain language
   - "Do I need to create an account?" → Explain ephemeral mode for MVP
   - "What scenarios are available?" → List Bakery, Hotel, Tech Startup
   - "Is my payment secure?" → Mention Stripe handles payment processing
   - "Can I get a refund?" → State refund policy (e.g., 30-day refund for founding members)
   - "How do I contact support?" → Provide email address
3. Accordion interaction: clicking question expands answer, clicking again collapses (accessible via keyboard)
4. Responsive: works on mobile and desktop
5. Copy is clear, benefit-focused, and builds trust

---

### Story 3.9: Error Handling & Production Polish

**As a** developer,
**I want** comprehensive error handling and production-ready polish,
**so that** the MVP is stable, secure, and provides a professional user experience.

#### Acceptance Criteria

1. Global error boundary implemented: catches React errors and displays user-friendly fallback UI instead of white screen
2. API error responses standardized: all endpoints return consistent JSON structure `{ error: { message: string, code: string } }`
3. Rate limiting implemented on `/api/generate` endpoint: max 10 generations per IP per hour (prevents abuse, reduces costs)
4. Environment variable validation: app fails fast at startup if required env vars (DATABASE_URL, CLAUDE_API_KEY, STRIPE_SECRET_KEY) are missing
5. Logging implemented using Sentry for production errors: API failures, validation errors, payment webhook issues logged for monitoring

---

### Story 3.10: SEO, Meta Tags & Performance Optimization

**As a** developer,
**I want** proper SEO meta tags and performance optimizations,
**so that** the landing page ranks well in search engines and loads quickly.

#### Acceptance Criteria

1. Landing page includes SEO meta tags:
   - `<title>`: "tellingCube - Realistic Business Data in Minutes"
   - `<meta name="description">`: Value proposition (140-160 characters)
   - Open Graph tags (og:title, og:description, og:image) for social sharing
   - Favicon configured with simple cube icon or text-based logo
2. Performance optimizations applied:
   - Images (if any) use Next.js Image component with lazy loading
   - Font optimization: Inter font loaded with `font-display: swap`
   - JavaScript bundle analyzed: ensure <200KB gzipped total
3. Lighthouse audit run on landing page: achieves scores of 90+ for Performance, Accessibility, Best Practices, SEO
4. Responsive meta viewport tag ensures proper mobile rendering
5. Sitemap.xml and robots.txt created for search engine crawlers

---

## Epic 4: Advanced Features (v1.1 Roadmap)

**Goal:** Extend TellingCube beyond MVP to address practitioner needs identified by Jürgen Faisst (IBCS CEO). This epic implements the feature gaps required for commercial viability with IBCS practitioners post-certification. This epic is **post-MVP** and should be prioritized after successful market validation.

**Source:** Jürgen Faisst feedback (December 2025) - see `docs/marketing/launch/spa-ibcs-ceo-feedback-2025-12-18.md`

**Priority Order:**
1. Multi-year support + Plan/Budget data (HIGH - core practitioner need)
2. Full IBCS visualization compliance (HIGH - brand differentiation)
3. HR domain view (MEDIUM - extends multi-view concept)
4. Dimension hierarchies (MEDIUM - enterprise realism)
5. ESG domain view (LOW - emerging requirement)

---

### Story 4.1: Multi-Year Scenario Generation

**As a** business controller,
**I want** to generate scenarios spanning 3-5 years with monthly or quarterly granularity,
**so that** I can create realistic long-term planning and trend analysis examples.

#### Acceptance Criteria

1. Scenario generation UI includes year range selector (1-5 years, default: 3 years)
2. Time granularity toggle available: Monthly (default) or Quarterly
3. Event generation prompts adapted to handle multi-year narratives (business growth, seasonality patterns)
4. Database schema supports date ranges beyond 12 months without performance degradation
5. All views (Sales, Finance) correctly aggregate and display multi-year data with year-over-year comparisons
6. Generation time scales linearly (3 years ≈ 3x generation time of 1 year)

**Dependencies:** FR15, database schema review
**Estimate:** 8-12 hours

---

### Story 4.2: Plan vs. Actual Data Generation

**As an** IBCS practitioner,
**I want** scenarios that include both planned/budgeted figures and actual results,
**so that** I can demonstrate variance analysis and plan vs. actual comparisons.

#### Acceptance Criteria

1. Event schema extended with `data_type` field: 'ACTUAL' | 'PLAN' | 'FORECAST'
2. Claude prompts generate realistic budget/plan figures (typically created before actuals, with reasonable variance)
3. Finance view displays plan vs. actual comparison with variance highlighting
4. IBCS notation applied: solid fill for actuals, outlined for plan/forecast (per IBCS standards)
5. Variance calculation logic: absolute variance, percentage variance, favorable/unfavorable indicators
6. Controlling view added with plan vs. actual dashboard as primary focus

**Dependencies:** FR16, FR20
**Estimate:** 12-16 hours

---

### Story 4.3: IBCS Visualization Compliance

**As an** IBCS-certified professional,
**I want** all charts and visualizations to follow IBCS® notation standards,
**so that** the output is immediately usable in professional presentations without modification.

#### Acceptance Criteria

1. Semantic notation implemented:
   - Actuals: solid black/dark gray bars
   - Plan/Budget: outlined bars (white fill, dark border)
   - Forecast: hatched pattern bars
   - Previous year: lighter shade with "PY" indicator
2. Chart type selection follows IBCS guidelines:
   - Time series: horizontal bar charts or line charts
   - Structure: vertical bar charts
   - Deviation: waterfall charts for variance
3. Scaling and labeling follow IBCS unification guidelines
4. Color usage minimized (black/gray primary, color only for highlighting)
5. Chart components library documented with IBCS compliance notes
6. Reference implementation validated against IBCS® cheatsheet (`docs/standards/ibcs-cheatsheet.md`)

**Dependencies:** FR20, existing chart components from Epic 2
**Estimate:** 8-12 hours

---

### Story 4.4: HR Domain View

**As a** business analyst,
**I want** an HR view showing headcount, payroll, and workforce metrics,
**so that** I can demonstrate how personnel costs connect to financial performance.

#### Acceptance Criteria

1. HR View API endpoint created: `/api/views/hr`
2. HR View displays:
   - Headcount trend by month (FTE and headcount)
   - Payroll expense breakdown by department/role
   - New hires vs. terminations timeline
   - Average cost per employee metrics
3. HR view mathematically reconciles with Finance view (payroll expense matches)
4. UI includes HR tab alongside Sales and Finance views
5. Employee lifecycle events visualized as timeline or Gantt-style chart

**Dependencies:** FR18, existing employee lifecycle events from Epic 1
**Estimate:** 6-8 hours

---

### Story 4.5: Dimension Hierarchies

**As an** enterprise user,
**I want** multi-level hierarchies in organization, product, and cost center dimensions,
**so that** scenarios reflect realistic corporate structures with drill-down capabilities.

#### Acceptance Criteria

1. Dimension tables created: `organizations`, `products`, `cost_centers` with parent-child relationships
2. Events reference dimension IDs instead of flat strings
3. Views support drill-down: Region → Country → Site → Department
4. Aggregation logic correctly rolls up child values to parent levels
5. UI includes expand/collapse controls for hierarchical data display
6. Sample hierarchies included in preset scenarios (e.g., Bakery: single location; Tech Startup: 3 departments)

**Dependencies:** FR17, database schema updates
**Estimate:** 10-14 hours

---

### Story 4.6: ESG Domain View (Future)

**As a** sustainability-focused organization,
**I want** an ESG view showing environmental and social metrics,
**so that** I can demonstrate integrated financial and sustainability reporting.

#### Acceptance Criteria

1. ESG event types added: carbon emissions, energy usage, diversity metrics, community investment
2. ESG View API endpoint: `/api/views/esg`
3. ESG View displays:
   - Carbon footprint by month/quarter
   - Energy consumption trends
   - Diversity metrics (if employee data supports)
   - Sustainability score/rating
4. ESG data correlates with operational events (e.g., production volume → emissions)
5. Optional inclusion in scenario generation (checkbox to include ESG data)

**Dependencies:** FR19, new event types
**Estimate:** 8-10 hours
**Priority:** LOW - defer until ESG reporting demand validated

---

### Epic 4 Summary

| Story | Priority | Estimate | Dependencies |
|-------|----------|----------|--------------|
| 4.1 Multi-Year Generation | HIGH | 8-12h | FR15 |
| 4.2 Plan vs. Actual | HIGH | 12-16h | FR16, FR20 |
| 4.3 IBCS Visualization | HIGH | 8-12h | FR20 |
| 4.4 HR Domain View | MEDIUM | 6-8h | FR18 |
| 4.5 Dimension Hierarchies | MEDIUM | 10-14h | FR17 |
| 4.6 ESG Domain View | LOW | 8-10h | FR19 |

**Total Epic 4 Estimate:** 52-72 hours (post-MVP, phased implementation)

---

## Epic 5: Phase 1 - Copy & Messaging Update (Big Bang Launch)

**Goal:** Update all user-facing copy to reflect the new positioning. Remove IBCS from primary positioning. Zero technical changes to core engine.

**Timeline:** This Week (1-2 days)
**Risk Level:** LOW - Copy changes only, git revert if needed

---

### Story 5.1: Landing Page Hero Update

**As a** visitor,
**I want** the landing page hero to immediately communicate the value proposition,
**so that** I understand within 10 seconds that tellingCube solves my data reconciliation problem.

#### Acceptance Criteria

1. Hero headline changed to: "Realistic Business Data That Actually Reconciles"
2. Subline changed to: "Generate consistent financial scenarios in minutes. Finance, Sales, HR—every number adds up."
3. CTA Primary: "Explore Examples" (scrolls to scenario grid)
4. CTA Secondary: "Generate Your Own - €9"
5. Brian Julius quote displayed as social proof: "The ivory-billed woodpecker of the data world"
6. hoki.help charity counter visible

**Files to Update:** Landing page hero component
**Estimate:** 1-2 hours

---

### Story 5.2: Problem Section Addition

**As a** visitor,
**I want** to see the problem tellingCube solves clearly articulated,
**so that** I recognize my own pain and understand why this tool exists.

#### Acceptance Criteria

1. Problem section added below hero with heading: "The Synthetic Data Problem"
2. Four pain point cards displayed:
   - AI Hallucinates: "Ask ChatGPT for P&L data. The numbers won't match the balance sheet."
   - Random Generators: "Fake names, random numbers, obviously synthetic."
   - Real Data: "NDAs, GDPR, compliance. You can't use it for demos."
   - Manual (Excel): "3-4 hours minimum. And still full of errors."
3. Transition text: "You need test data that looks real, reconciles perfectly, and is ready in minutes—not days."

**Files to Update:** Landing page, new ProblemSection component
**Estimate:** 2-3 hours

---

### Story 5.3: Solution Section Update

**As a** visitor,
**I want** to understand how tellingCube solves the problem,
**so that** I trust the technical approach and believe it will work for me.

#### Acceptance Criteria

1. Solution section heading: "How tellingCube Works"
2. Visual diagram showing: Events → Finance View / Sales View / HR View (same source)
3. Explanation text: "Every transaction—sales, payroll, invoices—flows through one event stream. All views derive from the same data. That's why the numbers always match."
4. Proof points listed:
   - Revenue matches across Sales and Finance
   - Payroll matches HR headcount
   - COGS matches purchasing events
   - Every € is traceable to a source event

**Files to Update:** Landing page solution section
**Estimate:** 2-3 hours

---

### Story 5.4: Use Cases Section Addition

**As a** visitor,
**I want** to see specific use cases for my profession,
**so that** I understand how tellingCube applies to my work.

#### Acceptance Criteria

1. Use cases section heading: "Who Uses tellingCube"
2. Five use case cards displayed:
   - BI Consultants: "Client demos and POCs without revealing real data"
   - Software Vendors: "Product demos that look like real business data"
   - Trainers: "Workshops with realistic, consistent scenarios"
   - Developers & QA: "Test data that covers edge cases"
   - Data Scientists: "Statistically valid synthetic data for ML"
3. Each card has icon, title, and 1-sentence description

**Files to Update:** Landing page, new UseCasesSection component
**Estimate:** 2 hours

---

### Story 5.5: Chart Labels Update (Result Page)

**As a** user viewing results,
**I want** chart labels to focus on data quality rather than IBCS,
**so that** the value proposition is consistent with the new positioning.

#### Acceptance Criteria

1. Chart section header changed from "Charts demonstrate data quality · Inspired by IBCS©" to "Data Preview · Quality at a glance"
2. Chart legend text changed from "Styling inspired by IBCS©: AC = Actual..." to "AC = Actual, PL = Plan. Charts follow IBCS® standards."
3. Remove or minimize any "All charts are inspired by IBCS©" messaging
4. Keep IBCS mention only as small footnote if present

**Files to Update:** Result page chart components
**Estimate:** 1 hour

---

### Story 5.6: Navigation Update

**As a** user,
**I want** navigation to reflect the new positioning,
**so that** IBCS is not prominently featured as the main value proposition.

#### Acceptance Criteria

1. Remove "IBCS" from main navigation if present
2. Navigation items: Logo | [Examples] [Pricing] [API Docs] [What's Next] | [Login] [Dashboard]
3. "Examples" instead of "Scenarios" in navigation
4. "API Docs" prominent for developer audience

**Files to Update:** Navigation component
**Estimate:** 30 minutes

---

### Story 5.7: Meta/SEO Update

**As a** search engine,
**I want** meta tags to reflect the new positioning,
**so that** tellingCube ranks for "synthetic data" and "test data" rather than just "IBCS."

#### Acceptance Criteria

1. Meta title: "tellingCube - Realistic Business Data That Reconciles"
2. Meta description: "Generate consistent financial scenarios in minutes. Finance, Sales, HR—every number adds up. No more fake data. No more hours in Excel."
3. OG image updated to reflect new messaging (if time permits)
4. Keywords focus: synthetic data, test data, demo data, business data generator

**Files to Update:** Layout metadata, OG image
**Estimate:** 30 minutes

---

### Story 5.8: Features Section Update

**As a** visitor,
**I want** the features section to emphasize data quality over IBCS,
**so that** I understand the practical benefits.

#### Acceptance Criteria

1. Features section heading: "Everything You Need"
2. Feature list:
   - 1, 3, or 5 Year Scenarios
   - Finance + Sales + HR Domains
   - 9 Company Profiles (Startup → LargeCap)
   - 3 Industries (Consumer, Industrial, Financial)
   - CSV Export + JSON API
   - Mathematical Consistency Guaranteed
   - Preview Charts Included
   - Generate in ~3 Minutes
3. IBCS badge moved to bottom as bonus: "Bonus: Charts follow IBCS® visualization standards"

**Files to Update:** Features section component
**Estimate:** 1 hour

---

### Epic 5 Summary

| Story | Description | Estimate | Risk |
|-------|-------------|----------|------|
| 5.1 | Landing Page Hero Update | 1-2h | LOW |
| 5.2 | Problem Section Addition | 2-3h | LOW |
| 5.3 | Solution Section Update | 2-3h | LOW |
| 5.4 | Use Cases Section Addition | 2h | LOW |
| 5.5 | Chart Labels Update | 1h | LOW |
| 5.6 | Navigation Update | 0.5h | LOW |
| 5.7 | Meta/SEO Update | 0.5h | LOW |
| 5.8 | Features Section Update | 1h | LOW |

**Total Epic 5 Estimate:** 10-14 hours (1-2 days)
**Risk Level:** LOW - All copy changes, easily reversible

---

## Epic 6: Phase 2 - Auth & Dashboard

**Goal:** Implement Magic Link authentication and user dashboard.

**Timeline:** 1-2 weeks
**Dependencies:** Phase 1 complete

### Stories (High-Level)

- 6.1: Magic Link Authentication (Resend integration)
- 6.2: User Model (Prisma schema update)
- 6.3: Session Management (httpOnly cookies)
- 6.4: Dashboard Home Page
- 6.5: Scenario History Grid
- 6.6: Tier Badge Display
- 6.7: Active Generation Banner

**Detailed stories to be created after Phase 1 completion.**

---

## Epic 7: Phase 3 - Async Generation & Pricing

**Goal:** Implement QStash-based async generation and new pricing tiers.

**Timeline:** 1 week
**Dependencies:** Phase 2 complete

### Stories (High-Level)

- 7.1: QStash Integration
- 7.2: New Stripe Products (5 tiers)
- 7.3: Email Notifications (generation complete)
- 7.4: Progress Tracking
- 7.5: Webhook Handlers

**Detailed stories to be created after Phase 2 completion.**

---

## Epic 8: Phase 4 - New Features

**Goal:** Add HR domain, multi-year logic, and charity counter.

**Timeline:** 1-2 weeks
**Dependencies:** Phase 3 complete

### Stories (High-Level)

- 8.1: HR Domain Events & View
- 8.2: Multi-Year Logic (1/3/5 years)
- 8.3: hoki.help Donation Counter
- 8.4: Coupon System
- 8.5: Early Supporter Upgrade

**Detailed stories to be created after Phase 3 completion.**

---

## Checklist Results Report

### Executive Summary

**Overall PRD Completeness:** 92% (Excellent)

**MVP Scope Appropriateness:** Just Right - The scope is tightly focused on proving the core value proposition (event-sourced multi-view consistency) while remaining achievable within the 2-4 week timeline for a solo developer.

**Readiness for Architecture Phase:** ✅ **READY** - The PRD provides comprehensive technical guidance, clear requirements, and well-sequenced epics. The Architect can proceed immediately with system design.

**Most Critical Strengths:**
- Exceptionally detailed technical assumptions (frontend stack, backend, database, AI integration, monitoring)
- Well-structured epic breakdown with properly sized stories (2-4 hours each)
- Comprehensive acceptance criteria that are testable and specific
- Clear rationale for all major technical decisions (monorepo, monolith, pragmatic testing)
- Strong user experience vision with detailed branding guidelines

**Minor Gaps (Non-Blocking):**
- No visual diagrams (user journey flow, architecture diagram would enhance clarity)
- Out-of-scope items referenced in brief but not explicitly listed in PRD
- Support process details not defined (acceptable for MVP)

---

### Category Analysis

| Category                         | Status   | Critical Issues |
| -------------------------------- | -------- | --------------- |
| 1. Problem Definition & Context  | PASS     | None            |
| 2. MVP Scope Definition          | PASS     | None            |
| 3. User Experience Requirements  | PASS     | None            |
| 4. Functional Requirements       | PASS     | None            |
| 5. Non-Functional Requirements   | PASS     | None            |
| 6. Epic & Story Structure        | PASS     | None            |
| 7. Technical Guidance            | PASS     | None            |
| 8. Cross-Functional Requirements | PASS     | None            |
| 9. Clarity & Communication       | PARTIAL  | Minor: No diagrams; acceptable for handoff |

**Status Legend:**
- **PASS:** 90%+ complete, ready for next phase
- **PARTIAL:** 60-89% complete, minor improvements recommended
- **FAIL:** <60% complete, must address before proceeding

---

### Top Issues by Priority

#### BLOCKERS: None ✅

All critical elements are in place for the Architect to begin system design.

#### HIGH: None ✅

The PRD is comprehensive and well-structured.

#### MEDIUM: Recommended Enhancements

**M1: Add Visual Diagrams**
- **What:** User journey flow diagram (Landing → Generate → Results → Payment → Confirmation)
- **Why:** Visual representation would help Architect understand state transitions and routing structure
- **Impact if skipped:** Low - epic stories provide sufficient detail, but diagram would speed comprehension
- **Recommendation:** Add in architecture phase or defer to UX Expert

**M2: Explicit Out-of-Scope List**
- **What:** Add "Out of MVP Scope" section listing deferred features (custom inputs, data export, HR/Controlling views, user accounts)
- **Why:** Prevents scope creep during development; clarifies for future planning
- **Impact if skipped:** Low - implicitly clear from requirements and stories
- **Recommendation:** Add 1-paragraph section after Epic List referencing brief's detailed out-of-scope

#### LOW: Nice to Have

**L1: Support Process Definition**
- **What:** Define support channel (email address), response time expectations, escalation for founding members
- **Why:** Solo developer needs clear boundaries to manage support load
- **Impact if skipped:** Minimal - can be defined post-launch
- **Recommendation:** Document in operational playbook, not PRD

**L2: Architecture Diagram Placeholder**
- **What:** Add placeholder for system architecture diagram (to be created by Architect)
- **Why:** Standard practice for PRDs to include technical diagrams
- **Impact if skipped:** None - Architect will create during architecture phase
- **Recommendation:** Defer to architecture document

---

### MVP Scope Assessment

#### ✅ Scope is Appropriate

**Strengths:**
1. **True MVP:** 3 preset scenarios (no custom inputs) minimizes prompt engineering complexity
2. **2 views only:** Sales + Finance sufficient to prove multi-view consistency without overbuilding
3. **Ephemeral mode:** No auth system saves 2-3 weeks of development time
4. **Pragmatic testing:** Unit tests for critical paths, manual validation for quality - appropriate for MVP velocity
5. **Monolith architecture:** Right choice for <1,000 users; avoids premature microservices complexity

**No Features Should Be Cut:**
All 14 functional requirements are essential to deliver the core value proposition and enable founding member conversions.

**No Missing Essential Features:**
The PRD correctly omits features that would delay MVP without validating core assumptions (user accounts, data export, additional views, custom scenarios).

#### Timeline Realism: Achievable

**Total Estimate:** 58-90 hours (approximately 3-4.5 weeks at 20 hours/week)

**Breakdown:**
- Epic 1: 20-30 hours (Foundation & Event Generation) - **Most risk here due to AI prompt engineering**
- Epic 2: 18-28 hours (Multi-View Dashboards) - Lower risk, standard frontend work
- Epic 3: 20-32 hours (User Journey & Monetization) - Lower risk, integrating existing services

**Risk Assessment:**
- **Highest Risk:** Story 1.4 (Bakery Scenario Prompt Engineering) - AI quality unpredictable, may need iteration
- **Mitigation:** Brother validation after Epic 1 provides early quality gate; Week 1-2 go/no-go decision
- **Confidence:** 85% - timeline is achievable if prompt engineering succeeds in first 2-3 iterations

---

### Technical Readiness

#### ✅ Excellent Technical Clarity

**Strengths:**
1. **Technology Stack Fully Specified:** Next.js 14, TypeScript, TailwindCSS, Prisma, NeonDB, Claude API, Stripe, Resend, Vercel - no ambiguity
2. **Architectural Decisions Documented:** Monorepo, monolith, single wide events table - rationale provided for each
3. **Performance Budgets Defined:** <2s landing page load, <5min generation, <200KB JS bundle, <200ms queries
4. **Testing Strategy Clear:** 100% coverage for validation logic, 80%+ for queries, manual testing for UI/quality
5. **Security Requirements Explicit:** Rate limiting, input validation, circuit breakers, PCI delegation to Stripe

**Identified Technical Risks:**

**TR1: AI Data Quality Uncertainty (HIGH)**
- **Risk:** Claude may generate unrealistic data or fail consistency validation
- **Mitigation in PRD:** 3-stage prompt pipeline (context → trends → events), validation function, brother review, manual iteration
- **Architect Action:** Design prompt structure to be easily tunable; consider prompt version control

**TR2: Generation Time Variability (MEDIUM)**
- **Risk:** Claude API calls may exceed 5-minute timeout, especially for complex scenarios
- **Mitigation in PRD:** Circuit breaker, timeout handling, async generation with polling
- **Architect Action:** Design async generation flow with status tracking; consider background job queue

**TR3: Database Query Performance (LOW)**
- **Risk:** Aggregating 150-300 events for views may be slow with naive queries
- **Mitigation in PRD:** Indexes on scenario_id, event_type, event_date; Prisma query optimization
- **Architect Action:** Test query performance early; add indexes as needed

**Areas Requiring Architect Investigation:**

**AI1: Prompt Engineering Approach**
- **Question:** Should prompts be code (TypeScript) or configuration (YAML/JSON) for easier iteration?
- **Decision Needed:** Architect should recommend structure in architecture document
- **Impact:** Affects Story 1.4 implementation

**AI2: Async Generation Flow**
- **Question:** Should generation be synchronous (user waits) or async (polling/webhook)?
- **Decision Needed:** Architect should design state machine for generation status
- **Impact:** Affects Stories 1.6, 3.2 implementation

**AI3: Session Management Strategy**
- **Question:** How to track ephemeral sessions for 24-hour cleanup without user accounts?
- **Decision Needed:** Architect should design session ID strategy (cookie vs. URL param)
- **Impact:** Affects Story 1.5, FR13 implementation

---

### Recommendations

#### For Immediate PRD Improvement (Optional)

**R1: Add Out-of-Scope Section (5 minutes)**
Add the following section after Epic List:

```markdown
### Out of MVP Scope

The following features are explicitly deferred to post-MVP phases:

**Authentication & User Management (Alpha Phase):**
- User accounts, login/logout, password reset
- "My Scenarios" saved library

**Custom Scenario Inputs (Beta Phase):**
- User-defined prompts, industry selection, parameter customization

**Data Export (Alpha Phase):**
- CSV download, Excel export, JSON API access

**Additional Views (Alpha Phase):**
- HR View, Controlling View (2 views sufficient to prove MVP)

**Advanced Features (Post-MVP):**
- Multi-year scenarios, AI-generated narratives, scenario comparison

See Project Brief (docs/brief.md) for complete list and rationale.
```

**R2: Add User Journey Diagram Placeholder (Optional)**
Consider adding a simple text-based flow diagram in the UX section:

```
Landing Page → [Click Scenario] → Loading State → [Generation Complete] → Results View
                                                                             ↓
                                                                    [Become Member]
                                                                             ↓
                                                           Stripe Checkout → Confirmation → Email
```

#### For Architect Handoff (Recommended)

**R3: Create Architecture Document Outline**
The Architect should create `docs/architecture.md` covering:
1. System Architecture Diagram (components, data flow)
2. Database Schema (detailed Prisma model with relationships)
3. API Contract Specifications (request/response schemas for all endpoints)
4. Async Generation State Machine (if async approach chosen)
5. Deployment Architecture (Vercel configuration, environment variables)
6. Security Implementation Details (rate limiting, validation schemas)

**R4: Schedule Technical Spike for Prompt Engineering**
Before starting Story 1.4, allocate 2-4 hours for:
- Testing Claude API with sample prompts
- Validating JSON parsing reliability
- Assessing output quality with different prompt structures
- Documenting findings to inform implementation

---

### Final Decision

✅ **READY FOR ARCHITECT**

The PRD is comprehensive, properly structured, and provides sufficient detail for architectural design to begin immediately.

**Next Actions:**
1. **Architect:** Review PRD Technical Assumptions and Requirements; create architecture document
2. **PM (Mario):** Optional - Add out-of-scope section and user journey diagram (5-10 minutes)
3. **Team:** Schedule Epic 1 kickoff after architecture review (target: Week 1 Day 1)

**Confidence Level:** 95% - This PRD is well above the bar for MVP quality. The Architect has everything needed to design the system and begin implementation planning.

---

## Next Steps

### UX Expert Prompt

Ready to bring tellingCube's user experience to life? Review the **User Interface Design Goals** section (pages 7-13) and the **Epic structure** (pages 20-38) in this PRD.

Your mission: Create wireframes and detailed UI specifications for the five core screens (Landing Page, Loading State, Results View, Confirmation Page, FAQ Section) that bring our "zero-friction experimentation" vision to reality.

**Start here:** `/ux create-wireframes --source=docs/prd.md`

### Architect Prompt

Ready to architect tellingCube's technical foundation? Review the **Technical Assumptions** (pages 14-19), **Requirements** (pages 4-6), and **Epic structure** (pages 20-38) in this PRD.

Your mission: Design the system architecture, database schema, API contracts, and deployment strategy that will deliver our event-sourced multi-view consistency guarantee.

**Start here:** `/arch create-architecture --source=docs/prd.md`
