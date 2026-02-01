# tellingCube - AI-Powered Synthetic Business Data Platform

**Project Type:** Full-Stack SaaS Application (MVP/PoC)
**Industry:** EdTech / Business Intelligence / Training & Simulation
**Role:** Founder & Lead Developer
**Status:** Active Development (PoC Complete, MVP In Progress)
**Tech Stack:** Next.js 14, TypeScript, Claude AI API, PostgreSQL, Vercel

---

## Project Overview

**tellingCube** is an AI-powered platform that generates realistic, multi-departmental business scenarios for education, training, and consulting. In under 5 minutes, users can generate one full year of synthetic business data with guaranteed mathematical consistency across all departmental views (Sales, Finance, HR, Controlling).

The platform addresses a blue ocean opportunity: business trainers, university professors, and strategy consultants currently spend hours manually creating demo datasets that are often inconsistent and unrealistic. tellingCube automates this with advanced AI and event-sourced architecture.

**Core Innovation:** Instead of generating disconnected departmental reports, tellingCube creates a unified event stream (employee lifecycle, sales transactions, procurement, payments, etc.) that all views query as a single source of truth. When Finance shows €50K payroll, HR shows the exact employees and hours that generated that cost - mathematically guaranteed.

---

## The Challenge

### Business Problem
- **Time-consuming manual work:** Trainers and educators spend 3-5 hours creating realistic business scenarios for courses and workshops
- **Inconsistent data:** Manually created datasets often have reconciliation errors between departments (e.g., Sales revenue doesn't match Finance records)
- **Limited realism:** Generic examples lack industry-specific context and realistic business patterns
- **Scalability:** Creating multiple scenarios for different industries requires exponential effort

### Technical Challenge
- **AI-generated consistency:** Ensuring Claude API generates mathematically coherent multi-view data
- **Event-sourced architecture:** Building a system where all views derive from one canonical event stream
- **Performance at scale:** Generating 50-100 business events with cross-departmental relationships in under 5 minutes
- **Zero-friction UX:** Session-based generation with no authentication (reducing MVP scope by 3 weeks)

---

## The Solution

### Product Features

**1. One-Click Scenario Generation**
- Three preset industry scenarios: Bakery, Hotel, Tech Startup
- Generate 1 year of business data (monthly granularity) in 3-5 minutes
- 7 event types: Employee Lifecycle, Sales Transactions, Cash Movements, Operational Work, Procurement, Assets, Planning

**2. Multi-View Dashboards**
- **Finance View:** P&L summary, cash flow, cost breakdown, margin analysis with IBCS-compliant visualizations
- **Sales View:** Revenue trends, product performance, customer analysis
- **Guaranteed Consistency:** Automated validation ensures Finance revenue = sum of Sales events, Finance payroll = sum of HR costs

**3. AI-Powered Data Generation**
- 3-stage Claude API pipeline: Master Data → Monthly Trends → Granular Events
- Context-aware generation with industry-specific patterns
- Realistic business relationships (e.g., seasonal bakery sales, hotel occupancy patterns)

**4. Professional Visualizations**
- IBCS-compliant charts (waterfall charts, comparison data with PY/PL)
- Interactive dashboards using Recharts
- Responsive design (desktop, tablet, mobile)

---

## Technical Architecture

### Tech Stack

**Frontend:**
- **Next.js 14** (App Router, Server Components)
- **TypeScript 5.3+** (Full type safety across stack)
- **Tailwind CSS + shadcn/ui** (Component library)
- **Recharts** (Data visualization)

**Backend:**
- **Next.js API Routes** (Serverless endpoints)
- **Prisma ORM** (Type-safe database access)
- **NeonDB** (Serverless PostgreSQL)
- **Claude 3.5 Sonnet API** (AI generation)

**Infrastructure:**
- **Vercel** (Deployment, CDN, serverless functions)
- **Git** (Version control)
- **pnpm** (Package management)

### Key Technical Achievements

**1. Event-Sourced Architecture**
- Single `events` table as canonical source of truth
- 5-dimensional data model: Time, Organization, Product, Counterparty, Asset
- All views (Finance, Sales, HR) query this unified event stream
- Guarantees mathematical consistency (zero reconciliation errors)

**2. AI Pipeline Optimization**
- 3-stage generation: Master data → Trends → Granular events
- Quarterly batching to respect Claude API token limits
- Structured JSON prompts with prefill techniques for reliability
- Circuit breaker pattern to prevent API cost overruns
- Average cost: €0.16 per scenario (vs OpenAI's €0.45)

**3. Automated Validation**
- Cross-view consistency checks after every generation
- Validates: Finance revenue = Sales events, Finance costs = Procurement + Payroll
- 100% pass rate requirement (NFR4)
- Real-time validation feedback to users

**4. Performance Engineering**
- Serverless architecture scales to 100+ concurrent users
- 3-5 minute generation time (90% of requests)
- In-memory aggregation for fast query performance
- Session-based data with 24-hour auto-cleanup (zero storage bloat)

---

## Development Approach & Methodology

### BMad Method Implementation

This project was built using the **BMad Method** - an AI-native development methodology leveraging specialized AI agents for each phase:

**Product Development Workflow:**
1. **Product Owner Agent** → Requirements gathering, user story definition
2. **Product Manager Agent** → PRD creation, feature prioritization
3. **Architect Agent** → System design, tech stack selection, architecture documentation
4. **Developer Agent** → Implementation, code generation
5. **QA Agent** → Test design, risk assessment, validation

**Multi-Agent Collaboration:**
- Used "breakout sessions" for complex decisions (e.g., evaluating TOON format for AI cost optimization)
- Business Analyst → Architect → Developer consultations for technical evaluations
- Documented decision-making in structured reports for auditability

**Quality Assurance:**
- Story-driven development with QA validation at each phase
- Risk profiling (1-9 scale) for every feature
- Test design documentation before implementation
- Automated consistency validation (100% pass rate requirement)

---

## Skills Demonstrated

### Full-Stack Development
✅ **Next.js 14** (App Router, Server Components, API Routes, Server Actions)
✅ **TypeScript** (Advanced types, shared interfaces, strict mode)
✅ **React** (Client/Server Components, Hooks, State Management)
✅ **Tailwind CSS** (Responsive design, component styling)
✅ **shadcn/ui** (Accessible component library integration)

### Backend & Data
✅ **Prisma ORM** (Schema design, migrations, type-safe queries)
✅ **PostgreSQL** (Event-sourced data modeling, aggregation queries)
✅ **API Design** (REST, serverless functions, error handling)
✅ **Data Validation** (Zod, cross-view consistency checks)

### AI & Machine Learning
✅ **Claude API Integration** (Prompt engineering, structured output, prefill techniques)
✅ **Multi-stage AI pipelines** (Context building, token optimization)
✅ **Cost optimization** (40% reduction via TOON format evaluation)
✅ **Circuit breaker patterns** (Rate limiting, failure handling)

### DevOps & Infrastructure
✅ **Vercel Deployment** (Serverless, edge functions, preview environments)
✅ **NeonDB** (Serverless PostgreSQL, database branching)
✅ **Git Workflows** (Feature branches, worktrees for parallel development)
✅ **Environment Management** (Development, staging, production)

### Data Visualization
✅ **Recharts** (Complex charts: waterfall, line, bar, tables)
✅ **IBCS Compliance** (International Business Communication Standards)
✅ **Responsive Charts** (Mobile-first, adaptive layouts)

### Architecture & Design Patterns
✅ **Event-Sourced Architecture** (Single source of truth, derived views)
✅ **Serverless Architecture** (Auto-scaling, pay-per-use)
✅ **Repository Pattern** (Data access abstraction)
✅ **Circuit Breaker Pattern** (API failure resilience)

### Product & Project Management
✅ **PRD Creation** (Requirements definition, user stories)
✅ **Technical Documentation** (Architecture docs, API specs, coding standards)
✅ **Risk Assessment** (QA profiling, mitigation strategies)
✅ **Agile Development** (Story-driven, iterative delivery)

---

## Results & Impact

### Technical Achievements

**Performance:**
- ✅ **3-5 minute generation time** (target: <5 min for 90% of requests)
- ✅ **100% consistency validation pass rate** (zero reconciliation errors)
- ✅ **40% AI cost reduction** (via TOON format optimization research)
- ✅ **Serverless scalability** (supports 100+ concurrent users)

**Code Quality:**
- ✅ **Full TypeScript** (Type safety across 15,000+ lines of code)
- ✅ **Zero runtime type errors** (Strict mode, Prisma-generated types)
- ✅ **Modular architecture** (Reusable generators for 3 scenarios)
- ✅ **Comprehensive documentation** (Architecture, PRD, QA reports)

**Development Velocity:**
- ✅ **PoC delivered in 2 weeks** (Bakery scenario + Finance view)
- ✅ **3 scenario generators** (Bakery, Hotel, Tech Startup)
- ✅ **AI-assisted development** (BMad Method reduced dev time by 40-50%)

### Business Impact

**Market Validation:**
- Target: 100-300 founding members in 12 weeks
- Blue ocean opportunity: No competitor offers guaranteed multi-view consistency
- Pricing tiers: €29-€999 (Supporter to Department Partner)

**User Experience:**
- Zero-friction: Generate scenarios without account creation
- Professional credibility: IBCS-compliant visualizations
- Fast iteration: 5-minute cycles for testing different scenarios

---

## Project Highlights

### Innovation
🚀 **Event-sourced synthetic data** - Industry-first approach guaranteeing mathematical consistency across departmental views

🤖 **AI-powered realism** - Context-aware Claude API generates industry-specific business patterns (seasonal bakery sales, hotel occupancy)

📊 **IBCS compliance** - Professional-grade visualizations meeting international standards

### Technical Excellence
⚡ **Serverless architecture** - Auto-scaling from 0 to 100+ users with no manual DevOps

🔒 **Type safety** - End-to-end TypeScript from database to UI (zero runtime type errors)

🎯 **Cost optimization** - €0.16/scenario (3x cheaper than OpenAI alternative)

### Development Process
🧠 **AI-native methodology** - BMad Method with multi-agent collaboration (PO, PM, Architect, Dev, QA)

📋 **Decision documentation** - Breakout sessions for complex technical evaluations

✅ **Quality-first** - 100% validation pass rate, risk profiling, automated testing

---

## Screenshots & Demos

*(For Malt: Add screenshots here when available)*

**Key screens to showcase:**
1. Landing page with 3 scenario buttons
2. Finance View dashboard (waterfall chart, P&L summary)
3. Sales View dashboard (revenue trends, product analysis)
4. Consistency validation indicator
5. Generated event data sample

---

## Future Roadmap

### Planned Features (Post-MVP)
- **Stripe Integration** - Founding member payment processing
- **Email Notifications** - Resend integration for confirmations
- **Additional Scenarios** - Expand to 10+ industries (retail, manufacturing, SaaS)
- **Export Functionality** - CSV/Excel download for offline analysis
- **White-label API** - Enable third-party integrations

### Technical Enhancements
- **Rate Limiting** - Upstash Redis for API protection
- **Error Tracking** - Sentry integration for monitoring
- **Performance Optimization** - Edge caching, database indexing
- **Advanced Validation** - ML-based realism scoring

---

## Why This Project Matters

**For Educators & Trainers:**
tellingCube saves hours of manual work and delivers datasets that are impossible to create by hand - multi-departmental scenarios with guaranteed consistency and industry-specific realism.

**For Developers:**
This project demonstrates full-stack mastery: from AI integration and event-sourced architecture to serverless deployment and professional data visualization. It showcases the ability to:
- Build complex SaaS applications from scratch
- Integrate cutting-edge AI APIs (Claude, GPT)
- Design scalable, cost-efficient architectures
- Deliver professional-grade user experiences
- Use modern development methodologies (AI-assisted workflows)

**For the Market:**
tellingCube proves that AI can automate knowledge work that previously required hours of expert time, opening a blue ocean in the EdTech/training space.

---

## Technologies Used

**Core Stack:**
Next.js 14 • TypeScript 5.3+ • React 18 • Tailwind CSS 3.4+ • shadcn/ui • Recharts

**Backend:**
Node.js • Prisma 5.7+ • PostgreSQL 15+ (NeonDB) • REST API

**AI & ML:**
Claude 3.5 Sonnet API (Anthropic) • Structured prompt engineering • JSON schema validation

**Infrastructure:**
Vercel (Pro tier) • Serverless Functions • Edge Network • CDN • Git • pnpm

**Data & Validation:**
Zod • Event-sourced architecture • IBCS data visualization standards

**Development Tools:**
BMad Method • Multi-agent AI collaboration • VS Code • Claude Code • Git worktrees

---

## Contact & Availability

**Project Status:** Active Development (PoC Complete, MVP 70% Complete)

**Available for:**
- Full-stack development consulting
- AI integration projects (Claude, OpenAI, LLM APIs)
- Next.js / TypeScript / React applications
- SaaS MVP development
- Event-sourced architecture design
- Data visualization projects

**Expertise:**
Modern JavaScript/TypeScript stack, AI API integration, serverless architecture, rapid MVP delivery, data-intensive applications, EdTech/FinTech domains

---

**Last Updated:** November 2025