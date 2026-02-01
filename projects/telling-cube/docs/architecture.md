# tellingCube Fullstack Architecture Document

**Document Version:** 1.0
**Created:** 2025-11-15
**Status:** In Progress

---

## Introduction

This document outlines the complete fullstack architecture for **tellingCube**, including backend systems, frontend implementation, and their integration. It serves as the single source of truth for AI-driven development, ensuring consistency across the entire technology stack.

This unified approach combines what would traditionally be separate backend and frontend architecture documents, streamlining the development process for a modern fullstack application where these concerns are tightly integrated through Next.js 14's App Router paradigm.

**Key Architectural Drivers:**
- **Event-sourced data generation:** The core innovation is generating business events (not views), ensuring mathematical consistency across Sales and Finance dashboards
- **Effortless data connectivity (v1.1):** Design all data interfaces for zero-configuration simplicity - the IBCS Institute couldn't use their own data model because "data connection was too complex"; our architecture must solve this "last mile" problem by making connectivity invisible to users
- **Solo developer productivity:** Monolithic Next.js architecture eliminates microservices complexity
- **MVP velocity:** 2-4 week timeline demands pragmatic technology choices ("boring technology" where possible)
- **Zero-friction UX:** Ephemeral sessions with no authentication reduce development scope by ~3 weeks
- **Vercel-first deployment:** Leveraging serverless auto-scaling for 100-300 founding member target
- **IBCS practitioner focus (v1.1):** Architecture decisions should support the primary target audience (CFOs, controllers, BI analysts implementing IBCS) rather than trainers

### Starter Template or Existing Project

**Assessment:** N/A - Greenfield project

The PRD specifies a greenfield Next.js 14 monorepo with explicit technology choices. No fullstack starter templates (T3 Stack, create-t3-app, etc.) are needed. The tech stack is well-defined and building from a clean Next.js initialization with manual configuration will provide better control and avoid unnecessary dependencies.

### Change Log

| Date | Version | Description | Author |
|------|---------|-------------|--------|
| 2025-11-15 | 1.0 | Initial architecture document for tellingCube MVP | Winston (Architect) |
| 2025-12-18 | 1.1 | Added "Effortless Data Connectivity" as core architectural driver based on Jürgen Faisst feedback; added roadmap considerations for FR15-FR20 | Winston (Architect) |

---

## Roadmap Architectural Considerations (v1.1)

**Source:** Jürgen Faisst (CEO, IBCS Institute) feedback - December 2025
**Reference:** PRD v1.1 (FR15-FR20), Product Brief Addendum

This section documents architectural implications of roadmap features to ensure MVP design decisions don't create technical debt for future enhancements.

### FR15: Multi-Year Support (3-5 Years)

**Current Design:** Event dates stored as `DATE` type, no year constraints
**Impact:** ✅ LOW - Current schema already supports multi-year data

**Considerations:**
- Increase default `date_range` in generation prompts (currently 12 months)
- Query performance: Ensure indexes on `event_date` remain efficient with 36-60 months of data
- UI pagination: Results views may need year-selector or date range filters
- Generation time: Expect 3-5x longer generation for multi-year scenarios

**Action Required:** None for MVP; monitor query performance during testing

---

### FR16: Plan vs. Actual Data

**Current Design:** Events have `event_type` but no plan/actual distinction
**Impact:** ⚠️ MEDIUM - Requires schema extension

**Schema Change Required:**
```prisma
model Event {
  // ... existing fields
  dataType    DataType @default(ACTUAL)  // NEW FIELD
}

enum DataType {
  ACTUAL
  PLAN
  FORECAST
}
```

**Considerations:**
- Add `dataType` field to Event model (can be added without breaking changes)
- Claude prompts will need to generate both plan and actual data in sequence
- Validation logic must compare PLAN vs ACTUAL totals for variance analysis
- IBCS notation requires visual distinction (solid vs. outlined bars)

**Action Required:** Reserve `dataType` in mental model; no schema change for MVP

---

### FR17: Dimension Hierarchies

**Current Design:** Flat string fields (`organizationUnit`, `productId`)
**Impact:** ⚠️ MEDIUM-HIGH - May require normalized dimension tables

**Option A: Embedded Hierarchy (Simpler)**
```prisma
model Event {
  organizationUnit  String   // "Region/Country/Site/Dept"
  productCategory   String   // "Category/Subcategory/Product"
}
```
- Parse hierarchy from delimited strings
- Pro: No schema change, backward compatible
- Con: Complex query logic, no referential integrity

**Option B: Normalized Dimensions (Cleaner)**
```prisma
model Organization {
  id        String  @id
  name      String
  parentId  String?
  level     Int
}

model Event {
  organizationId  String
  organization    Organization @relation(...)
}
```
- Pro: Proper hierarchy support, efficient roll-ups
- Con: Schema migration required, more complex joins

**Recommendation:** Use Option A for initial post-MVP release; migrate to Option B if hierarchy queries become frequent

**Action Required:** Keep flat strings for MVP; avoid hardcoding dimension handling

---

### FR18: HR Domain View

**Current Design:** Employee lifecycle events already generated
**Impact:** ✅ LOW - Mostly query/UI work

**Considerations:**
- HR queries can leverage existing `EMPLOYEE_HIRE`, `EMPLOYEE_TERMINATION`, `PAYROLL` events
- New queries needed: headcount by month, FTE calculations, turnover rate
- UI: New tab in results view (alongside Sales, Finance)

**Action Required:** None for MVP; HR view is query + UI addition

---

### FR19: ESG Domain View

**Current Design:** No ESG-related event types
**Impact:** ⚠️ MEDIUM - Requires new event types

**New Event Types Required:**
- `CARBON_EMISSION` - CO2 output by activity/period
- `ENERGY_CONSUMPTION` - kWh by facility/period
- `SUSTAINABILITY_METRIC` - Generic ESG KPIs

**Considerations:**
- ESG data should correlate with operational events (production → emissions)
- May require industry-specific prompts (manufacturing vs. services ESG profiles)
- Low priority per Jürgen feedback; defer until demand validated

**Action Required:** None for MVP; design event types when implementing

---

### FR20: Full IBCS Visualization Compliance

**Current Design:** Basic charting with brand colors
**Impact:** ⚠️ MEDIUM - Chart component refactoring

**IBCS Requirements:**
- Semantic notation: solid (actual), outlined (plan), hatched (forecast)
- Chart type rules: horizontal bars for time, vertical for structure
- Color restraint: black/gray primary, color only for highlighting
- Proper scaling, labels, and unification

**Considerations:**
- Chart components must support `dataType` prop for styling
- May need custom Recharts components or switch to more flexible library
- Reference: `docs/standards/ibcs-cheatsheet.md`

**Action Required:** Review current chart components; ensure they can be extended for IBCS compliance

---

### Effortless Data Connectivity Principle

**Core Insight:** The IBCS Institute built their own data model but couldn't use it because "data connection was too complex."

**Architectural Implications:**

1. **Zero-Config Data Import (Future)**
   - Design for drag-and-drop CSV/Excel upload
   - Auto-detect column mappings where possible
   - Provide clear error messages for incompatible data

2. **Pre-Built Integrations (Future)**
   - Consider connectors to common BI tools (Power BI, Tableau)
   - Export formats optimized for target tools

3. **API-First Design (Current)**
   - All data access through clean API endpoints
   - JSON responses with consistent schemas
   - Makes future integrations straightforward

4. **Progressive Disclosure**
   - Simple use cases require zero configuration
   - Advanced options available but hidden by default

**Action Required:** Maintain clean API contracts; avoid tightly coupling UI to data layer

---

## High Level Architecture

### Technical Summary

tellingCube employs a **serverless monolithic architecture** built on Next.js 14, deployed as a unified application on Vercel's Edge Network. The system generates synthetic business data through a three-stage AI pipeline (**Claude 3.5 Sonnet API**), stores events in a single PostgreSQL table (NeonDB Launch), and derives Sales and Finance views through aggregation queries executed server-side. Frontend and backend share TypeScript types through a monorepo structure, with React Server Components handling initial rendering and Client Components providing interactivity for charts and dashboards. The architecture achieves the PRD's event-sourcing goal by generating a canonical event stream first, then querying it for multiple departmental views, guaranteeing mathematical consistency. Vercel's serverless infrastructure auto-scales to handle the 100-300 founding member target without manual capacity planning, while session-based data cleanup (24-hour TTL) eliminates the need for user authentication systems.

**PoC Scope (Week 1-2):** Single scenario (Bakery), Finance view only, basic validation
**Post-PoC Expansion (Week 3-4):** Add Hotel/Tech Startup scenarios, Sales view, landing page, payments

### Platform and Infrastructure Choice

**Platform:** Vercel Pro
**Key Services:**
- **Hosting:** Vercel Edge Network (global CDN, serverless functions)
- **Database:** NeonDB Launch tier (serverless PostgreSQL with auto-scaling)
- **AI Generation:** Claude API (Anthropic) - Claude 3.5 Sonnet recommended
- **Email:** Resend (free tier for transactional emails) - **Post-PoC**
- **Payments:** Stripe (webhooks work seamlessly with Vercel functions) - **Post-PoC**
- **Rate Limiting:** Upstash (pay-as-you-go Redis) - **Post-PoC**
- **Monitoring:** Vercel Analytics + Sentry (error tracking) - **Post-PoC**

**PoC Infrastructure (Week 1-2):**
- ✅ Vercel Pro + NeonDB Launch + Claude API only
- ❌ Deferred: Upstash, Stripe, Resend, Sentry

**Deployment Host and Regions:** Vercel Edge Network (auto-deployed to 20+ global edge locations; primary region: Frankfurt or US-East for NeonDB)

**Rationale:**
- **Vercel Pro:** Zero DevOps overhead, optimized for Next.js, automatic deployments, edge caching, preview environments
- **NeonDB Launch:** Production-grade serverless PostgreSQL, scales to zero when idle, Prisma-compatible
- **Claude API (3.5 Sonnet):** Best balance of quality and cost ($3/$15 per 1M tokens); ~$0.165/scenario vs OpenAI's ~$0.45
- **Upstash (Post-PoC):** Serverless Redis for rate limiting (protects against API abuse and runaway costs)
- **Cost efficiency:** PoC ~$2-5 total; Full MVP ~$50-100/mo

### Repository Structure

**Structure:** Monorepo (single Git repository)

**Monorepo Tool:** Native npm/pnpm workspaces (no Nx/Turborepo needed for MVP simplicity)

**Package Organization:**
```
/telling-cube (root)
  /app                    # Next.js 14 App Router (frontend + API routes)
  /components             # React components (shared UI)
  /lib                    # Shared utilities, AI prompts, queries, validators
  /prisma                 # Database schema, migrations
  /types                  # TypeScript type definitions (shared across app)
  /public                 # Static assets
  package.json            # Root dependencies
  tsconfig.json           # TypeScript config
  tailwind.config.js      # TailwindCSS config (brand colors)
  next.config.js          # Next.js config
```

**Rationale:**
- **Monorepo benefits:** Shared types between frontend/backend, unified deployment, single codebase for solo developer
- **No monorepo tooling:** Native npm workspaces sufficient for small project (avoid Nx complexity)
- **Next.js App Router conventions:** `/app` directory contains both pages and API routes (co-location)

### High Level Architecture Diagram

```mermaid
graph TB
    User[User Browser] -->|HTTPS| Vercel[Vercel Edge Network]

    Vercel -->|Static Assets| CDN[Vercel CDN]
    Vercel -->|Dynamic Pages| NextApp[Next.js App Router]
    Vercel -->|API Requests| APIRoutes[API Routes]

    NextApp -->|Server Components| ReactSSR[React Server Components]
    NextApp -->|Client Components| ReactClient[React Client Components]

    APIRoutes -->|/api/generate| GenService[Generation Service]
    APIRoutes -->|/api/views/*| QueryService[View Query Service]
    APIRoutes -->|/api/validate| ValidationService[Validation Service]
    APIRoutes -->|/api/stripe/webhook| StripeWebhook[Stripe Webhook Handler]

    GenService -->|1. Prompt| Claude[Claude 3.5 Sonnet API<br/>Anthropic]
    GenService -->|2. Store Events| Prisma[Prisma ORM]

    QueryService -->|Aggregate| Prisma
    ValidationService -->|Check Consistency| Prisma

    Prisma -->|SQL| NeonDB[(NeonDB Launch<br/>PostgreSQL)]

    APIRoutes -.->|Post-PoC| Upstash[(Upstash Redis<br/>Rate Limiting)]

    StripeWebhook -.->|Post-PoC| Stripe[Stripe API]
    StripeWebhook -.->|Post-PoC| Resend[Resend Email API]

    GenService -.->|Circuit Breaker| Sentry[Sentry<br/>Error Tracking]

    style Vercel fill:#2563EB,color:#fff
    style NeonDB fill:#10B981,color:#fff
    style Claude fill:#E5EAF5
    style Upstash fill:#CCCCCC
    style Stripe fill:#CCCCCC
    style Resend fill:#CCCCCC
```

### Architectural Patterns

**Overall Architecture:**
- **Jamstack Hybrid (SSR + API Routes):** Static generation where possible (landing page), server-side rendering for dynamic content (results view), API routes for data operations - _Rationale:_ Balances performance (pre-rendered pages) with dynamic data generation needs
- **Event-Sourced Data Model:** Generate canonical event stream, derive views through queries - _Rationale:_ Core differentiator ensuring multi-view consistency; events are immutable, views are computed

**Frontend Patterns:**
- **Component-Based Architecture:** Reusable React components with TypeScript interfaces - _Rationale:_ Maintainability and type safety across the application
- **Server Components by Default:** Use React Server Components for non-interactive content, Client Components only when needed (charts, tabs) - _Rationale:_ Reduces JavaScript bundle size, improves initial page load
- **Atomic Design Pattern:** Organize components as Atoms (Button, Card) → Molecules (ChartCard) → Organisms (SalesView) → Pages - _Rationale:_ Clear component hierarchy aids reusability

**Backend Patterns:**
- **Repository Pattern:** Abstract data access through Prisma repositories - _Rationale:_ Enables testing with mock data, centralizes query logic
- **Service Layer Pattern:** Business logic separated from API routes (GenerationService, ValidationService) - _Rationale:_ API routes handle HTTP concerns, services handle business rules
- **Circuit Breaker Pattern (Post-PoC):** AI API calls stop after 3 consecutive failures - _Rationale:_ Prevents runaway costs if Claude API fails repeatedly

**Integration Patterns:**
- **API-First Design:** All data flows through REST API endpoints (even server-side calls use internal fetch) - _Rationale:_ Clear contracts, easier to test, future API access for white-label customers
- **Shared Type Definitions:** TypeScript interfaces in `/types` imported by both frontend and backend - _Rationale:_ Single source of truth for data structures, compile-time validation
- **Redis-Backed Rate Limiting (Post-PoC):** Upstash Redis tracks API call counts per IP/session - _Rationale:_ Prevents abuse, protects Claude API costs

---

---

## **Integration Context: Lovable Components → Next.js Backend**

### **Frontend Components Built (75% Complete via Lovable)**

**✅ Completed (Prompts 1-3):**
- **Landing Page** (`app/page.tsx`) - Hero, 3 scenario buttons, pricing, FAQ
  - Libraries: Tailwind, shadcn/ui
  - Brand colors: #2563EB (blue), #10B981 (green)
- **Loading State** (`app/generate/loading/page.tsx`) - Spinner, status messages, polling
- **Results View** (`app/results/[scenarioId]/page.tsx`) - Sales + Finance dashboards
  - **IBCS-compliant charts** (Recharts): Grayscale (#333333, #999999, #CCCCCC)
  - Line charts, bar charts, IBCS tables (no vertical lines)
  - Consistency badge, CTA banner

**❌ Remaining (Prompt 4):**
- **Confirmation Page** (`app/confirmation/page.tsx`) - Post-payment thank you

**Integration Requirements:**
- Export TSX components from Lovable → integrate into Next.js
- Connect frontend to backend via API contracts (detailed in API Specification section)
- PoC Focus: Finance view only, Bakery scenario only

### **IBCS Compliance Scope**

**CRITICAL:** IBCS grayscale charts apply **ONLY** to Results View (Sales/Finance dashboards).

**Results View (IBCS):**
- Charts: Grayscale (#333333, #999999, #CCCCCC)
- No vertical table lines, minimal grid lines
- Message-driven headlines

**Rest of App (tellingCube Brand):**
- Landing page, loading state, confirmation: Brand colors (#2563EB, #10B981)
- UI buttons, tabs, badges: Brand colors
- **Only data visualization** follows IBCS

---

---

## Tech Stack

This is the **DEFINITIVE technology selection** for tellingCube. All development must use these exact versions and tools. This table serves as the single source of truth.

### Tech Stack Table

| Category | Technology | Version | Purpose | Rationale |
|----------|-----------|---------|---------|-----------|
| **Frontend Language** | TypeScript | 5.3+ | Type-safe frontend/backend code | Compile-time type checking prevents runtime errors; shared types between frontend/backend |
| **Frontend Framework** | Next.js 14 | 14.0+ (App Router) | React framework with SSR, API routes, file-based routing | Unified frontend/backend in one deployment; Vercel-optimized; Server Components reduce JS bundle |
| **UI Component Library** | shadcn/ui | Latest | Unstyled, accessible React components | Already used in Lovable components; Radix UI primitives; fully customizable with Tailwind |
| **State Management** | React Hooks (useState, useContext) | Built-in | Local component state, minimal global state | MVP has simple state needs (no complex global state); avoid Redux overhead |
| **Backend Language** | TypeScript | 5.3+ | Type-safe API routes and services | Same language as frontend; shared types; excellent Node.js ecosystem |
| **Backend Framework** | Next.js 14 API Routes | 14.0+ | Serverless API endpoints in App Router | Co-located with frontend; auto-deployed to Vercel; no separate API server needed |
| **API Style** | REST JSON | HTTP/1.1 | Request/response API endpoints | Simple, well-understood; Stripe webhooks require REST; future white-label API access |
| **Database** | NeonDB (PostgreSQL) | 15+ (Serverless) | Serverless PostgreSQL with auto-scaling | Scales to zero when idle; Prisma-compatible; branching for dev/staging; ~$0 for PoC |
| **ORM** | Prisma | 5.7+ | Type-safe database access and migrations | Best TypeScript ORM; auto-generates types from schema; excellent DX; schema migrations built-in |
| **Cache/Rate Limiting** | Upstash Redis | Latest (Post-PoC) | Serverless Redis for rate limiting, session storage | Pay-per-request pricing; Vercel integration; REST API (no connection pooling issues) |
| **Authentication** | None (MVP) | N/A | No auth in MVP - ephemeral sessions | Saves 2-3 weeks dev time; session-based data only; add auth post-MVP |
| **AI Generation** | Claude API (Anthropic) | Claude 3.5 Sonnet | Generate synthetic business events via structured prompts | Best balance of quality/cost ($0.165/scenario vs OpenAI $0.45); 200K context window; excellent JSON output |
| **Email Service** | Resend | Latest (Post-PoC) | Transactional emails (payment confirmations) | Vercel-native; simple API; free tier (100 emails/day); deferred to post-PoC |
| **Payment Processing** | Stripe Checkout | Latest (Post-PoC) | Hosted payment pages for founding members | Handles all payment UI/security; webhooks for confirmations; PCI compliance delegated |
| **CSS Framework** | Tailwind CSS | 3.4+ | Utility-first CSS framework | Already used in Lovable components; mobile-first; small production bundles; JIT compiler |
| **Chart Library** | Recharts | 2.10+ | React charts for IBCS dashboards | Already used in Lovable; composable API; SVG-based (responsive); supports IBCS grayscale styling |
| **Frontend Testing** | Vitest + React Testing Library | Latest (Post-PoC) | Unit tests for components and utilities | Fast (Vite-powered); Jest-compatible; deferred to post-PoC for MVP velocity |
| **Backend Testing** | Vitest | Latest (Post-PoC) | Unit tests for API routes, services, validation | Same test runner as frontend; TypeScript native; mock Prisma/Claude API |
| **E2E Testing** | Playwright | Latest (Post-PoC) | End-to-end user journey tests | Vercel recommended; cross-browser; deferred to Alpha phase |
| **Build Tool** | Next.js (Turbopack) | Built-in | Fast development server and production builds | Built into Next.js 14; replaces Webpack; 700x faster cold starts |
| **Package Manager** | pnpm | 8.0+ | Fast, disk-efficient package management | 3x faster than npm; saves disk space; monorepo-friendly; Vercel supports pnpm |
| **Deployment Platform** | Vercel | Pro tier | Hosting, CDN, serverless functions, edge network | Zero-config Next.js deployment; preview URLs; automatic SSL; global CDN; $20/mo |
| **CI/CD** | Vercel (Git-based) | Built-in | Automatic deployments on git push | Push to `main` → auto-deploy; preview branches; no GitHub Actions needed for MVP |
| **Monitoring** | Vercel Analytics | Built-in (Post-PoC) | Web Vitals, page views, performance | Free tier included with Vercel Pro; basic metrics; upgrade to Sentry for errors |
| **Error Tracking** | Sentry | Latest (Post-PoC) | Error monitoring and alerting | Free tier: 5K events/month; source maps; performance monitoring; deferred to post-PoC |

### PoC Tech Stack (Week 1-2)

| Category | Technology | Notes |
|----------|-----------|-------|
| Frontend | Next.js 14 + TypeScript + Tailwind + Recharts | Lovable components exported as TSX |
| Backend | Next.js API Routes + Prisma | `/api/generate`, `/api/views/finance`, `/api/validate` |
| Database | NeonDB Launch | Single `events` table |
| AI | Claude 3.5 Sonnet API | Bakery scenario only |
| Deployment | Vercel Pro | Push to deploy |

**Deferred to Post-PoC:** Upstash, Stripe, Resend, Sentry, Testing frameworks

### Key Technology Decisions

**Why Next.js 14 App Router?** Server Components reduce JS bundle; co-located API routes; Vercel-optimized

**Why Prisma?** Best TypeScript support; auto-generated types; built-in migrations; NeonDB compatible

**Why Claude 3.5 Sonnet?** $0.165/scenario vs GPT-4's $0.45 (3x cheaper); 200K context; excellent JSON output

**Why Recharts?** Already in Lovable; React-native API; SVG-based; IBCS-friendly grayscale styling

**Why shadcn/ui?** Already in Lovable; copy/paste (no dependency); Radix UI (accessible); Tailwind-based

**Why Vercel?** Next.js optimization; zero DevOps; git-based deployments; $20/mo includes CDN/SSL

**Why NeonDB?** Serverless PostgreSQL; scales to zero; free tier for PoC; database branching for dev/staging

---

**Status:** Section 3 (Tech Stack) - Complete

---

## Data Models

The data model design is **central to tellingCube's core innovation**: event-sourced architecture where all departmental views derive from a single stream of business events.

### Core Concept: Event Sourcing

**Traditional approach (competitors):**
- Generate Sales data → separate table
- Generate Finance data → separate table
- **Problem:** Data doesn't reconcile (Sales revenue ≠ Finance revenue)

**tellingCube's approach:**
- Generate **Events** → single source of truth
- Derive **Views** → query events to compute Sales/Finance dashboards
- **Benefit:** Perfect reconciliation guaranteed (same data source)

### Model 1: Event (Core Data Model)

**Purpose:** Represents a single business event in the generated scenario (e.g., "Employee hired", "Sale completed", "Invoice paid"). All departmental views are computed by aggregating events.

**Key Design Decisions:**
- **Wide table schema:** One table with nullable columns for different event types (vs. separate tables per event type)
- **JSON metadata field:** Flexible storage for event-specific attributes
- **5 dimensions:** Time, Organization, Product, Counterparty, Asset (standardized across all events)

#### TypeScript Interface

```typescript
// types/event.ts

export enum EventType {
  EMPLOYEE_LIFECYCLE = 'employee_lifecycle',
  SALES_TRANSACTION = 'sales_transaction',
  CASH_MOVEMENT = 'cash_movement',
  OPERATIONAL_WORK = 'operational_work',
  PROCUREMENT = 'procurement',
  ASSET_LIFECYCLE = 'asset_lifecycle',
  PLANNING_BUDGETING = 'planning_budgeting'
}

export interface Event {
  id: string
  scenarioId: string
  eventType: EventType
  eventDate: Date

  // Dimensions
  timePeriod: string              // "2024-01", "Q1 2024"
  organizationUnit?: string
  productId?: string
  counterpartyId?: string
  assetId?: string

  // Financial attributes
  amountEur?: number
  quantity?: number
  unitPrice?: number

  // Metadata (flexible JSON)
  metadata: Record<string, any>

  createdAt: Date
}
```

#### Prisma Schema

```prisma
model Event {
  id            String   @id @default(uuid())
  scenarioId    String
  eventType     String
  eventDate     DateTime

  timePeriod        String
  organizationUnit  String?
  productId         String?
  counterpartyId    String?
  assetId           String?

  amountEur     Decimal?  @db.Decimal(12, 2)
  quantity      Decimal?  @db.Decimal(10, 2)
  unitPrice     Decimal?  @db.Decimal(10, 2)

  metadata      Json      @default("{}")
  createdAt     DateTime  @default(now())

  scenario      Scenario  @relation(fields: [scenarioId], references: [id], onDelete: Cascade)

  @@index([scenarioId])
  @@index([eventType])
  @@index([eventDate])
  @@index([timePeriod])
}
```

### Model 2: Scenario (Metadata)

**Purpose:** Tracks each generated scenario (metadata container). Enables 24-hour TTL cleanup and scenario retrieval.

#### TypeScript Interface

```typescript
// types/scenario.ts

export enum ScenarioStatus {
  PROCESSING = 'processing',
  COMPLETED = 'completed',
  ERROR = 'error'
}

export interface Scenario {
  id: string
  scenarioType: 'bakery' | 'hotel' | 'tech_startup'
  status: ScenarioStatus

  companyName?: string
  employeeCount?: number
  timePeriodStart: Date
  timePeriodEnd: Date

  validationPassed: boolean
  validationErrors?: string[]

  createdAt: Date
  completedAt?: Date
  expiresAt: Date                 // 24 hours from creation

  events: Event[]
}
```

#### Prisma Schema

```prisma
model Scenario {
  id                String          @id @default(uuid())
  scenarioType      String
  status            String          @default("processing")

  companyName       String?
  employeeCount     Int?
  timePeriodStart   DateTime
  timePeriodEnd     DateTime

  validationPassed  Boolean         @default(false)
  validationErrors  String[]        @default([])

  createdAt         DateTime        @default(now())
  completedAt       DateTime?
  expiresAt         DateTime

  events            Event[]

  @@index([status])
  @@index([expiresAt])
}
```

### Design Rationale

**Why single Events table?** Simpler queries (no UNIONs); easier AI generation; consistent schema; Prisma-friendly

**Why JSON metadata?** Flexibility for new event types; AI-friendly; self-documenting; no schema migrations needed

**Why 24-hour TTL?** MVP simplicity; cost control; privacy-friendly; matches ephemeral UX (no user accounts)

**Why Decimal for currency?** Precision (avoid floating-point errors); IBCS compliance; validation requires exact reconciliation

### PoC Simplifications (Week 1-2)

**Included:** Event model (full), Scenario model (full), Finance view event types

**Deferred:** Asset lifecycle events, planning/budgeting events, HR-specific attributes

---

**Status:** Section 4 (Data Models) - Complete

---

## API Specification

This section defines the **exact API contracts** that connect Lovable frontend components to the Next.js backend.

### API Design Principles

- **REST JSON:** Simple, stateless HTTP endpoints
- **Typed responses:** All schemas match TypeScript interfaces (shared types)
- **Error handling:** Consistent error format across all endpoints
- **Idempotency:** GET requests are idempotent; POST requests use unique IDs

### PoC API Endpoints (Week 1-2)

| Endpoint | Method | Purpose | PoC Status |
|----------|--------|---------|------------|
| `/api/generate` | POST | Start scenario generation | ✅ Required |
| `/api/generate/status` | GET | Poll generation progress | ✅ Required |
| `/api/views/finance` | GET | Get Finance view data | ✅ Required |
| `/api/validate` | GET | Get consistency validation | ✅ Required |
| `/api/views/sales` | GET | Get Sales view data | ❌ Post-PoC |
| `/api/stripe/*` | POST | Payment processing | ❌ Post-PoC |

### Key API Contracts

#### POST /api/generate

**Request:**
```json
{ "scenarioType": "bakery" | "hotel" | "tech_startup" }
```

**Response:**
```json
{
  "generationId": "550e8400-e29b-41d4-a716-446655440000",
  "status": "processing"
}
```

#### GET /api/generate/status?id={generationId}

**Response (Processing):**
```json
{
  "status": "processing",
  "progress": 65,
  "message": "Validating consistency..."
}
```

**Response (Completed):**
```json
{
  "status": "completed",
  "scenarioId": "a1b2c3d4-e5f6-4a5b-8c9d-0e1f2a3b4c5d"
}
```

#### GET /api/views/finance?scenarioId={id}

**Response:**
```json
{
  "plSummary": {
    "revenue": 500000,
    "payroll": 150000,
    "cogs": 200000,
    "operatingProfit": 75000,
    "netMargin": 0.15
  },
  "monthlyCashFlow": [
    { "month": "2024-01", "cashIn": 42000, "cashOut": 38000, "netCash": 4000 }
  ],
  "costBreakdown": {
    "payroll": 150000,
    "cogs": 200000,
    "rent": 36000
  },
  "marginAnalysis": {
    "grossMargin": 0.60,
    "operatingMargin": 0.15,
    "netMargin": 0.15
  }
}
```

#### GET /api/validate?scenarioId={id}

**Response:**
```json
{
  "valid": true,
  "errors": [],
  "checks": [
    {
      "name": "Revenue Reconciliation",
      "passed": true,
      "message": "Finance revenue matches Sales events: €500,000"
    }
  ]
}
```

### TypeScript Type Definitions (Shared)

```typescript
// types/api.ts

export interface GenerateRequest {
  scenarioType: 'bakery' | 'hotel' | 'tech_startup'
}

export interface GenerationStatusResponse {
  status: 'processing' | 'completed' | 'error'
  scenarioId?: string
  error?: string
  progress?: number
  message?: string
}

export interface FinanceViewResponse {
  plSummary: {
    revenue: number
    payroll: number
    cogs: number
    otherExpenses: number
    operatingProfit: number
    netMargin: number
  }
  monthlyCashFlow: Array<{
    month: string
    cashIn: number
    cashOut: number
    netCash: number
  }>
  costBreakdown: Record<string, number>
  marginAnalysis: {
    grossMargin: number
    operatingMargin: number
    netMargin: number
  }
}

export interface ValidationResultResponse {
  valid: boolean
  errors: string[]
  checks?: Array<{
    name: string
    passed: boolean
    message: string
  }>
}
```

### Integration with Lovable Components

**Lovable components expect these exact response shapes** (from `ai-ui-prompts.md`):

1. ✅ **Loading State** polls `/api/generate/status`
2. ✅ **Finance View** fetches `/api/views/finance`
3. ✅ **Consistency Badge** fetches `/api/validate`

**No changes needed to Lovable components** - they already expect these contracts.

### Design Decisions

**Why REST?** Lovable components built for REST; simple contracts; Stripe compatibility; future API access

**Why polling?** Simpler than WebSockets; Vercel serverless-friendly; 30-90s generation = polling acceptable

**Why UUID IDs?** Unpredictable (secure); no collisions; shareable links

---

**Status:** Section 5 (API Specification) - Complete

---

## Backend Architecture

The backend implements tellingCube's **event-sourcing innovation** through three core services: Generation, Validation, and View Query.

### Service Layer Architecture

**3-Tier Design:**
1. **API Routes** (thin HTTP handlers) → delegate to services
2. **Service Layer** (business logic) → GenerationService, ValidationService, ViewQueryService
3. **Repository Layer** (Prisma) → database access

### Service 1: Generation Service

**Purpose:** Orchestrates AI-powered scenario generation through a 3-stage Claude API pipeline.

**3-Stage Pipeline:**

**Stage 1: Generate Master Data (10-15s)**
- Prompt: "Generate company master data for a bakery..."
- Output: Company name, employees, products, customers
- Why: Master data must exist before events reference it

**Stage 2: Generate Events (45-60s)**
- Prompt: "Generate 1 year of business events using master data..."
- Output: 250-400 Event objects (JSON array)
- Why: Core AI task; requires consistency across all events

**Stage 3: Validate & Store (5-10s)**
- Action: Run ValidationService, batch insert via Prisma
- Output: Scenario marked `completed` or `error`

**Key Implementation Details:**
- **Async generation:** API returns immediately; generation runs in background
- **Polling status:** Frontend polls `/api/generate/status` every 2-3s
- **Claude 3.5 Sonnet:** Temperature 0.7 for master data, 0.5 for events (consistency)
- **Error handling:** Circuit breaker after 3 consecutive Claude failures
- **Cost:** ~$0.165/scenario (Stage 1: $0.02, Stage 2: $0.14, Stage 3: $0.005)

### Service 2: Validation Service

**Purpose:** Ensures multi-view consistency by checking event reconciliation.

**Validation Checks:**

1. **Revenue Reconciliation:** Sum(sales_transaction) = Sum(cash_movement inbound)
2. **Payroll Reconciliation:** Employee salaries × 12 = Sum(payroll cash movements)
3. **COGS Reconciliation:** Procurement amounts = Products sold × unit COGS
4. **Time Distribution:** Events spread across all 12 months

**Tolerance:** 1% for revenue/payroll (decimal rounding); 5% for COGS (inventory variance)

**Implementation:** Prisma aggregation queries; returns `{ valid: boolean, errors: [], checks: [] }`

### Service 3: View Query Service

**Purpose:** Aggregate events into Finance/Sales views (compute from event stream).

**Finance View Aggregations:**
- **P&L Summary:** Sum events by type → revenue, payroll, COGS, profit
- **Monthly Cash Flow:** Group cash_movement by timePeriod → cashIn, cashOut per month
- **Cost Breakdown:** Sum procurement + payroll + fixed costs
- **Margin Analysis:** Compute gross/operating/net margins from aggregates

**Performance:** Indexed queries on `scenarioId`, `eventType`, `timePeriod` → <200ms for 300 events

### Design Rationale

**Why 3-stage pipeline?** Master data must exist first; validation requires complete event set; clear separation of concerns

**Why background async?** 60-90s generation exceeds serverless timeout (30s); polling UX requires async; better error handling

**Why 1% validation tolerance?** Decimal rounding in Prisma aggregations; Claude may introduce minor variance acceptable for synthetic data

**Why embed prompts in code?** Version control; faster iteration; no external prompt management; easier to test

---

**Status:** Section 6 (Backend Architecture) - Complete

---

## Database Schema

This section provides the complete Prisma setup for NeonDB, including schema definition, migrations, and seeding strategy.

### Prisma Setup

**File:** `prisma/schema.prisma`

```prisma
// prisma/schema.prisma

generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

// Scenario metadata
model Scenario {
  id                String    @id @default(uuid())
  scenarioType      String    // 'bakery' | 'hotel' | 'tech_startup'
  status            String    @default("processing") // 'processing' | 'completed' | 'error'

  // Metadata
  companyName       String?
  employeeCount     Int?
  timePeriodStart   DateTime
  timePeriodEnd     DateTime

  // Validation
  validationPassed  Boolean   @default(false)
  validationErrors  String[]  @default([])

  // Timestamps
  createdAt         DateTime  @default(now())
  completedAt       DateTime?
  expiresAt         DateTime  // 24 hours from createdAt

  // Relationships
  events            Event[]

  // Indexes
  @@index([status])
  @@index([expiresAt])
  @@index([createdAt])

  @@map("scenarios")
}

// Business events (core event-sourcing model)
model Event {
  id                String    @id @default(uuid())
  scenarioId        String
  eventType         String    // EventType enum
  eventDate         DateTime

  // Dimensions (standardized across all events)
  timePeriod        String    // "2024-01" format for grouping
  organizationUnit  String?
  productId         String?
  counterpartyId    String?   // Customer, employee, or supplier ID
  assetId           String?

  // Financial attributes (nullable - only for financial events)
  amountEur         Decimal?  @db.Decimal(12, 2)
  quantity          Decimal?  @db.Decimal(10, 2)
  unitPrice         Decimal?  @db.Decimal(10, 2)

  // Metadata (flexible JSON for event-specific attributes)
  metadata          Json      @default("{}")

  createdAt         DateTime  @default(now())

  // Relationships
  scenario          Scenario  @relation(fields: [scenarioId], references: [id], onDelete: Cascade)

  // Indexes for query performance
  @@index([scenarioId])
  @@index([eventType])
  @@index([eventDate])
  @@index([timePeriod])
  @@index([scenarioId, eventType])
  @@index([scenarioId, timePeriod])

  @@map("events")
}
```

### Environment Variables

**File:** `.env.local`

```bash
# Database
DATABASE_URL="postgresql://user:password@ep-xxx.eu-central-1.aws.neon.tech/tellingcube?sslmode=require"

# AI
ANTHROPIC_API_KEY="sk-ant-xxx"

# Stripe (Post-PoC)
# STRIPE_SECRET_KEY="sk_test_xxx"
# STRIPE_WEBHOOK_SECRET="whsec_xxx"

# Resend (Post-PoC)
# RESEND_API_KEY="re_xxx"

# Upstash (Post-PoC)
# UPSTASH_REDIS_REST_URL="https://xxx.upstash.io"
# UPSTASH_REDIS_REST_TOKEN="xxx"
```

### Prisma Client Setup

**File:** `lib/prisma.ts`

```typescript
// lib/prisma.ts

import { PrismaClient } from '@prisma/client'

const globalForPrisma = globalThis as unknown as {
  prisma: PrismaClient | undefined
}

export const prisma =
  globalForPrisma.prisma ??
  new PrismaClient({
    log: process.env.NODE_ENV === 'development' ? ['query', 'error', 'warn'] : ['error'],
  })

if (process.env.NODE_ENV !== 'production') globalForPrisma.prisma = prisma
```

**Rationale:** Singleton pattern prevents multiple Prisma instances in development (Next.js hot reload issue)

### Migration Commands

```bash
# Initial setup
pnpm install prisma @prisma/client

# Initialize Prisma (creates prisma/ folder)
npx prisma init

# Create migration from schema
npx prisma migrate dev --name init

# Generate Prisma Client (auto-generates types)
npx prisma generate

# Open Prisma Studio (visual database browser)
npx prisma studio

# Reset database (WARNING: deletes all data)
npx prisma migrate reset

# Deploy migrations to production
npx prisma migrate deploy
```

### Database Seeding (Optional - for testing)

**File:** `prisma/seed.ts`

```typescript
// prisma/seed.ts

import { PrismaClient } from '@prisma/client'

const prisma = new PrismaClient()

async function main() {
  console.log('Seeding database...')

  // Create test scenario
  const scenario = await prisma.scenario.create({
    data: {
      scenarioType: 'bakery',
      status: 'completed',
      companyName: 'Test Bakery GmbH',
      employeeCount: 10,
      timePeriodStart: new Date('2024-01-01'),
      timePeriodEnd: new Date('2024-12-31'),
      expiresAt: new Date(Date.now() + 24 * 60 * 60 * 1000),
      validationPassed: true,
    },
  })

  // Create test events
  await prisma.event.createMany({
    data: [
      {
        scenarioId: scenario.id,
        eventType: 'sales_transaction',
        eventDate: new Date('2024-01-15'),
        timePeriod: '2024-01',
        productId: 'bread-001',
        counterpartyId: 'customer-001',
        amountEur: 50.00,
        quantity: 10,
        unitPrice: 5.00,
        metadata: { customerName: 'ABC Corp', paymentMethod: 'cash' },
      },
      // Add more test events...
    ],
  })

  console.log('Seeding completed!')
}

main()
  .catch(e => {
    console.error(e)
    process.exit(1)
  })
  .finally(async () => {
    await prisma.$disconnect()
  })
```

**Run seeding:**
```bash
npx prisma db seed
```

### NeonDB-Specific Configuration

**Branching Strategy:**
- **Main branch:** Production database (connected to Vercel production)
- **Dev branch:** Development database (local development)
- **Preview branches:** Automatic preview databases per git branch (Vercel preview deployments)

**Connection Pooling:**
NeonDB automatically handles connection pooling - no need for PgBouncer with serverless

**Backup & Recovery:**
NeonDB provides automatic daily backups (7-day retention on Launch tier)

### Cleanup Cron Job (24-hour TTL)

**File:** `app/api/cron/cleanup/route.ts`

```typescript
// app/api/cron/cleanup/route.ts

import { prisma } from '@/lib/prisma'

export async function GET(request: Request) {
  // Verify cron secret (Vercel Cron Jobs set this header)
  const authHeader = request.headers.get('authorization')
  if (authHeader !== `Bearer ${process.env.CRON_SECRET}`) {
    return Response.json({ error: 'Unauthorized' }, { status: 401 })
  }

  try {
    // Delete expired scenarios (cascade deletes events)
    const result = await prisma.scenario.deleteMany({
      where: {
        expiresAt: {
          lt: new Date(),
        },
      },
    })

    return Response.json({
      success: true,
      deletedScenarios: result.count,
      timestamp: new Date().toISOString(),
    })
  } catch (error) {
    return Response.json({ error: 'Cleanup failed' }, { status: 500 })
  }
}
```

**Vercel Cron Configuration:**

**File:** `vercel.json`

```json
{
  "crons": [{
    "path": "/api/cron/cleanup",
    "schedule": "0 * * * *"
  }]
}
```

**Runs every hour to delete expired scenarios**

---

**Status:** Section 7 (Database Schema) - Complete

---

## Frontend Architecture

This section covers Lovable component integration into Next.js and frontend architectural patterns.

### Lovable Component Integration

**Export Process:**

1. **Export from Lovable:**
   - Click "Export Code" in Lovable
   - Download ZIP file containing TSX components
   - Components already use Tailwind, shadcn/ui, Recharts

2. **Integration into Next.js:**
   - Copy components to `components/` folder
   - Install dependencies: `pnpm install recharts @heroicons/react`
   - Update imports to match Next.js structure
   - Ensure Server/Client component boundaries are correct

### Component Structure

```
/components
  /landing                    # Lovable Prompt 1
    Hero.tsx                  # Hero section
    ScenarioButtons.tsx       # 3 scenario CTA buttons
    ProblemSolution.tsx       # Problem/solution section
    PricingSection.tsx        # 5 pricing tiers
    FAQSection.tsx            # Accordion FAQ

  /views                      # Lovable Prompt 3
    FinanceView.tsx           # Finance dashboard (IBCS charts)
    SalesView.tsx             # Sales dashboard (Post-PoC)
    ConsistencyBadge.tsx      # Validation badge

  /charts                     # IBCS-compliant Recharts
    LineChart.tsx             # Time series (IBCS grayscale)
    BarChart.tsx              # Comparisons (IBCS grayscale)
    DataTable.tsx             # IBCS tables (no vertical lines)

  /ui                         # shadcn/ui base components
    button.tsx                # Button (copy from shadcn)
    card.tsx                  # Card wrapper
    accordion.tsx             # FAQ accordion
    tabs.tsx                  # Sales/Finance tabs (mobile)

/app
  page.tsx                    # Landing page (uses landing components)
  /generate/loading/page.tsx  # Loading state (Lovable Prompt 2)
  /results/[scenarioId]/page.tsx  # Results view (uses views components)
  /confirmation/page.tsx      # Confirmation (Lovable Prompt 4)
```

### Server vs Client Components

**Server Components (default):**
- Landing page sections (Hero, ProblemSolution, Pricing, FAQ)
- Static content (no interactivity)
- SEO-friendly (pre-rendered HTML)

**Client Components (use 'use client'):**
- ScenarioButtons (onClick handlers)
- Loading page (polling, state management)
- Results view (fetches data, interactive charts)
- Charts (Recharts requires client-side rendering)

**Example:**

```typescript
// components/landing/ScenarioButtons.tsx
'use client'

import { useRouter } from 'next/navigation'

export function ScenarioButtons() {
  const router = useRouter()

  return (
    <div className="flex gap-4">
      <button onClick={() => router.push('/generate/loading?type=bakery')}>
        Generate Bakery Scenario
      </button>
      {/* Other buttons */}
    </div>
  )
}
```

### IBCS Chart Implementation

**File:** `components/charts/LineChart.tsx`

```typescript
// components/charts/LineChart.tsx
'use client'

import { LineChart as RechartsLineChart, Line, XAxis, YAxis, CartesianGrid, Tooltip, ResponsiveContainer } from 'recharts'

interface Props {
  data: Array<{ month: string; value: number }>
  dataKey: string
  title: string
}

export function LineChart({ data, dataKey, title }: Props) {
  return (
    <div className="space-y-2">
      <h3 className="text-lg font-medium text-gray-900">{title}</h3>
      <ResponsiveContainer width="100%" height={300}>
        <RechartsLineChart data={data}>
          <CartesianGrid strokeDasharray="3 3" stroke="#CCCCCC" />
          <XAxis
            dataKey="month"
            tick={{ fill: '#666666' }}
            stroke="#CCCCCC"
          />
          <YAxis
            tick={{ fill: '#666666' }}
            stroke="#CCCCCC"
          />
          <Tooltip />
          <Line
            type="monotone"
            dataKey={dataKey}
            stroke="#333333"
            strokeWidth={2}
            dot={false}
          />
        </RechartsLineChart>
      </ResponsiveContainer>
    </div>
  )
}
```

**IBCS Compliance:**
- Grayscale colors (#333333, #999999, #CCCCCC)
- Horizontal grid lines only
- No vertical grid lines
- Clean axes (no heavy borders)
- White background

### Data Fetching Pattern

**Results View Example:**

```typescript
// app/results/[scenarioId]/page.tsx
'use client'

import { useEffect, useState } from 'react'
import { FinanceView } from '@/components/views/FinanceView'
import { ConsistencyBadge } from '@/components/views/ConsistencyBadge'

export default function ResultsPage({ params }: { params: { scenarioId: string } }) {
  const [financeData, setFinanceData] = useState(null)
  const [validation, setValidation] = useState(null)
  const [loading, setLoading] = useState(true)

  useEffect(() => {
    async function fetchData() {
      try {
        const [financeRes, validationRes] = await Promise.all([
          fetch(`/api/views/finance?scenarioId=${params.scenarioId}`),
          fetch(`/api/validate?scenarioId=${params.scenarioId}`),
        ])

        const finance = await financeRes.json()
        const valid = await validationRes.json()

        setFinanceData(finance)
        setValidation(valid)
      } catch (error) {
        console.error('Failed to fetch data:', error)
      } finally {
        setLoading(false)
      }
    }

    fetchData()
  }, [params.scenarioId])

  if (loading) return <LoadingSkeleton />
  if (!financeData) return <ErrorState />

  return (
    <main className="min-h-screen bg-white">
      <ConsistencyBadge validation={validation} />
      <FinanceView data={financeData} />
    </main>
  )
}
```

### Responsive Design

**Breakpoints (Tailwind):**
- `sm:` 640px - Small tablets
- `md:` 768px - Tablets
- `lg:` 1024px - Desktop
- `xl:` 1280px - Large desktop

**Mobile (<768px):**
- Single column layout
- Tabs for Sales/Finance views (sticky)
- Stacked pricing cards
- Simplified charts (fewer data points)

**Desktop (>1024px):**
- Side-by-side Sales/Finance views (50/50 split)
- 5-column pricing grid
- Full chart detail

---

**Status:** Section 8 (Frontend Architecture) - Complete

---

## Unified Project Structure

Complete file/folder organization for the monorepo.

```
/telling-cube
├── .env.local                 # Environment variables (gitignored)
├── .gitignore
├── package.json               # Dependencies
├── pnpm-lock.yaml
├── tsconfig.json              # TypeScript config
├── next.config.js             # Next.js config
├── tailwind.config.ts         # Tailwind + brand colors
├── vercel.json                # Vercel deployment config + cron jobs
│
├── /app                       # Next.js 14 App Router
│   ├── layout.tsx             # Root layout (Inter font, metadata)
│   ├── page.tsx               # Landing page
│   ├── globals.css            # Tailwind imports
│   │
│   ├── /generate
│   │   └── /loading
│   │       └── page.tsx       # Loading state
│   │
│   ├── /results
│   │   └── /[scenarioId]
│   │       └── page.tsx       # Results view
│   │
│   ├── /confirmation
│   │   └── page.tsx           # Post-payment confirmation
│   │
│   └── /api                   # API Routes
│       ├── /generate
│       │   ├── route.ts       # POST /api/generate
│       │   └── /status
│       │       └── route.ts   # GET /api/generate/status
│       ├── /views
│       │   ├── /finance
│       │   │   └── route.ts   # GET /api/views/finance
│       │   └── /sales
│       │       └── route.ts   # GET /api/views/sales (Post-PoC)
│       ├── /validate
│       │   └── route.ts       # GET /api/validate
│       ├── /stripe            # Post-PoC
│       │   ├── /create-checkout
│       │   │   └── route.ts
│       │   └── /webhook
│       │       └── route.ts
│       └── /cron
│           └── /cleanup
│               └── route.ts   # Cron job for 24h TTL
│
├── /components                # React components
│   ├── /landing               # Landing page components
│   │   ├── Hero.tsx
│   │   ├── ScenarioButtons.tsx
│   │   ├── ProblemSolution.tsx
│   │   ├── PricingSection.tsx
│   │   └── FAQSection.tsx
│   │
│   ├── /views                 # Dashboard views
│   │   ├── FinanceView.tsx
│   │   ├── SalesView.tsx
│   │   └── ConsistencyBadge.tsx
│   │
│   ├── /charts                # IBCS charts
│   │   ├── LineChart.tsx
│   │   ├── BarChart.tsx
│   │   └── DataTable.tsx
│   │
│   └── /ui                    # shadcn/ui components
│       ├── button.tsx
│       ├── card.tsx
│       ├── accordion.tsx
│       └── tabs.tsx
│
├── /lib                       # Shared utilities
│   ├── prisma.ts              # Prisma client singleton
│   ├── /services              # Business logic services
│   │   ├── generation.service.ts
│   │   ├── validation.service.ts
│   │   └── view-query.service.ts
│   ├── /prompts               # Claude API prompts
│   │   ├── master-data.ts
│   │   └── events.ts
│   └── /utils                 # Helper functions
│       ├── format-currency.ts
│       └── date-helpers.ts
│
├── /types                     # TypeScript types (shared)
│   ├── api.ts                 # API request/response types
│   ├── event.ts               # Event types
│   └── scenario.ts            # Scenario types
│
├── /prisma                    # Prisma ORM
│   ├── schema.prisma          # Database schema
│   ├── seed.ts                # Database seeding script
│   └── /migrations            # Auto-generated migrations
│
├── /public                    # Static assets
│   ├── favicon.ico
│   └── /images
│
└── /docs                      # Documentation
    ├── architecture.md        # This document
    ├── prd.md
    ├── front-end-spec.md
    └── /standards
        └── ibcs-standards.md
```

### File Naming Conventions

- **Components:** PascalCase (`FinanceView.tsx`)
- **API routes:** lowercase (`route.ts` in folder)
- **Utilities:** kebab-case (`format-currency.ts`)
- **Types:** kebab-case (`api.ts`, not `API.ts`)
- **Pages:** lowercase (`page.tsx`)

---

**Status:** Section 9 (Unified Project Structure) - Complete

---

## Development Workflow

Local development setup, git workflow, and deployment process.

### Initial Setup

**Prerequisites:**
- Node.js 18+ installed
- pnpm installed (`npm install -g pnpm`)
- **Ports 3000-3008 available** (close any processes using these ports before running `pnpm dev`)
  - Port 3000: Next.js development server
  - Ports 3001-3008: Reserved for concurrent services during development (HMR, DevTools, etc.)

```bash
# 1. Clone repository
git clone https://github.com/masemIT/telling-cube.git
cd telling-cube

# 2. Install dependencies
pnpm install

# 3. Set up environment variables
cp .env.example .env.local
# Edit .env.local with real API keys

# 4. Set up database
npx prisma migrate dev --name init
npx prisma generate

# 5. (Optional) Seed test data
npx prisma db seed

# 6. Run development server
pnpm dev
```

**Access app:** http://localhost:3000

**Troubleshooting port conflicts:**
```bash
# Windows: Find process using port 3000
netstat -ano | findstr :3000
taskkill /PID <process_id> /F

# macOS/Linux: Find and kill process using port 3000
lsof -ti:3000 | xargs kill -9
```

### Git Workflow

**Branch Strategy:**
- `main` - Production (auto-deploys to Vercel)
- `dev` - Development (preview deployments)
- `feature/*` - Feature branches (preview deployments)

**Example workflow:**

```bash
# Create feature branch
git checkout -b feature/finance-view-optimization

# Make changes
# ... edit files ...

# Commit
git add .
git commit -m "Optimize Finance view query performance"

# Push to GitHub
git push origin feature/finance-view-optimization

# Create PR on GitHub
# Vercel automatically creates preview deployment

# After review, merge to main
# Vercel automatically deploys to production
```

### Environment Variables

**Required for local development:**

```bash
# .env.local

DATABASE_URL="postgresql://..."  # NeonDB dev branch
ANTHROPIC_API_KEY="sk-ant-..."   # Claude API key
```

**Production (Vercel):**

Set in Vercel dashboard → Project Settings → Environment Variables

- `DATABASE_URL` - NeonDB production branch
- `ANTHROPIC_API_KEY` - Claude API key (production)
- `CRON_SECRET` - Random string for cron job auth
- (Post-PoC) `STRIPE_SECRET_KEY`, `STRIPE_WEBHOOK_SECRET`, `RESEND_API_KEY`

### Development Commands

```bash
# Start dev server
pnpm dev

# Build production bundle
pnpm build

# Run production build locally
pnpm start

# Lint code
pnpm lint

# Format code with Prettier
pnpm format

# Type check
pnpm type-check

# Database commands
npx prisma studio          # Open visual DB browser
npx prisma migrate dev     # Create new migration
npx prisma generate        # Regenerate Prisma Client
npx prisma db seed         # Seed test data
```

### Deployment Process

**Vercel (automatic):**

1. Push to `main` branch
2. Vercel detects push, starts build
3. Runs `pnpm build`
4. Runs `npx prisma migrate deploy` (production migrations)
5. Deploys to production URL
6. Health check succeeds → deployment complete

**Manual deployment (if needed):**

```bash
# Install Vercel CLI
pnpm install -g vercel

# Deploy to preview
vercel

# Deploy to production
vercel --prod
```

### Database Migrations (Workflow)

**Development:**

```bash
# 1. Modify schema.prisma
# 2. Create migration
npx prisma migrate dev --name add_asset_events

# 3. Prisma generates migration SQL in prisma/migrations/
# 4. Commit migration files to git
git add prisma/migrations
git commit -m "Add asset lifecycle events to schema"
```

**Production:**

Vercel automatically runs `prisma migrate deploy` during build

**Rollback (if needed):**

```bash
# Revert to previous migration
npx prisma migrate resolve --rolled-back <migration-name>
```

### Testing Workflow (Post-PoC)

```bash
# Run unit tests
pnpm test

# Run unit tests in watch mode
pnpm test:watch

# Run E2E tests
pnpm test:e2e

# Test coverage
pnpm test:coverage
```

---

**Status:** Section 10 (Development Workflow) - Complete

---

## Deployment Architecture

Vercel deployment configuration, environment setup, and production infrastructure.

### Vercel Configuration

**File:** `vercel.json`

```json
{
  "buildCommand": "pnpm build",
  "installCommand": "pnpm install",
  "framework": "nextjs",
  "regions": ["fra1"],
  "crons": [
    {
      "path": "/api/cron/cleanup",
      "schedule": "0 * * * *"
    }
  ],
  "env": {
    "DATABASE_URL": "@database-url",
    "ANTHROPIC_API_KEY": "@anthropic-api-key",
    "CRON_SECRET": "@cron-secret"
  }
}
```

**Region Selection:** `fra1` (Frankfurt) - closest to Austria for low latency

### Environment Variables (Production)

**Vercel Dashboard → Project Settings → Environment Variables:**

| Variable | Type | Value | Notes |
|----------|------|-------|-------|
| `DATABASE_URL` | Secret | `postgresql://...neon.tech/...` | NeonDB production branch |
| `ANTHROPIC_API_KEY` | Secret | `sk-ant-...` | Claude API key |
| `CRON_SECRET` | Secret | Random UUID | Protects cron endpoint |
| `NODE_ENV` | Plain | `production` | Auto-set by Vercel |

**Post-PoC additions:**
- `STRIPE_SECRET_KEY`
- `STRIPE_WEBHOOK_SECRET`
- `RESEND_API_KEY`
- `UPSTASH_REDIS_REST_URL`
- `UPSTASH_REDIS_REST_TOKEN`

### Build Configuration

**File:** `next.config.js`

```javascript
/** @type {import('next').NextConfig} */
const nextConfig = {
  reactStrictMode: true,
  swcMinify: true,

  // Prisma generate in build
  webpack: (config, { isServer }) => {
    if (isServer) {
      config.externals.push('@prisma/client')
    }
    return config
  },

  // Environment variables exposed to client
  env: {
    NEXT_PUBLIC_APP_URL: process.env.NEXT_PUBLIC_APP_URL || 'https://tellingcube.com',
  },

  // Headers for security
  async headers() {
    return [
      {
        source: '/:path*',
        headers: [
          { key: 'X-DNS-Prefetch-Control', value: 'on' },
          { key: 'X-Frame-Options', value: 'DENY' },
          { key: 'X-Content-Type-Options', value: 'nosniff' },
          { key: 'Referrer-Policy', value: 'origin-when-cross-origin' },
        ],
      },
    ]
  },
}

module.exports = nextConfig
```

### Database Connection (Production)

**NeonDB Production Branch:**
- Automatic connection pooling (serverless-optimized)
- Auto-scaling (0 to max based on load)
- Daily backups (7-day retention)

**Connection String Format:**
```
postgresql://user:password@ep-xxx.eu-central-1.aws.neon.tech/tellingcube?sslmode=require&pgbouncer=true
```

**Note:** `pgbouncer=true` enables connection pooling for serverless

### Deployment Flow

```mermaid
graph LR
    A[Push to main] --> B[GitHub webhook]
    B --> C[Vercel build starts]
    C --> D[pnpm install]
    D --> E[prisma generate]
    E --> F[pnpm build]
    F --> G[prisma migrate deploy]
    G --> H[Deploy to Edge]
    H --> I[Health check]
    I -->|Pass| J[Traffic routed to new deployment]
    I -->|Fail| K[Rollback to previous]

    style J fill:#10B981,color:#fff
    style K fill:#EF4444,color:#fff
```

### Rollback Strategy

**Automatic rollback:**
- Vercel keeps last 10 deployments
- Can instant rollback from dashboard
- Zero downtime rollback

**Manual rollback:**
```bash
# List deployments
vercel ls

# Promote specific deployment to production
vercel promote <deployment-url>
```

### CDN & Edge Network

**Vercel Edge Network:**
- 20+ global edge locations
- Automatic static asset caching
- Edge Functions for API routes (select endpoints)

**Cached by default:**
- Static pages (Landing page)
- Static assets (`/public`)
- Fonts (Inter)

**NOT cached (dynamic):**
- API routes
- Results view (user-specific data)
- Loading page

### Monitoring & Alerts

**Vercel Analytics (built-in):**
- Web Vitals (LCP, FID, CLS)
- Page views
- Top pages
- Deployment status

**Custom Alerts (Post-PoC with Sentry):**
- Error rate > 5% → Email alert
- API latency > 2s → Slack notification
- Failed deployments → Email + Slack

---

**Status:** Section 11 (Deployment Architecture) - Complete

---

## Security & Performance

Security best practices and performance optimization strategies.

### Security

#### API Route Protection

**Rate Limiting (Post-PoC with Upstash):**

```typescript
// lib/rate-limit.ts

import { Ratelimit } from '@upstash/ratelimit'
import { Redis } from '@upstash/redis'

const ratelimit = new Ratelimit({
  redis: Redis.fromEnv(),
  limiter: Ratelimit.slidingWindow(10, '1 h'),
  analytics: true,
})

export async function checkRateLimit(identifier: string) {
  const { success, limit, reset, remaining } = await ratelimit.limit(identifier)

  return {
    success,
    limit,
    reset,
    remaining,
  }
}
```

**Usage in API route:**

```typescript
// app/api/generate/route.ts

export async function POST(request: Request) {
  const ip = request.headers.get('x-forwarded-for') || 'unknown'
  const { success } = await checkRateLimit(ip)

  if (!success) {
    return Response.json(
      { error: { code: 'RATE_LIMIT_EXCEEDED', message: 'Too many requests' } },
      { status: 429 }
    )
  }

  // ... continue with generation
}
```

#### Environment Variables

**Never commit:**
- `.env.local` (gitignored)
- API keys
- Database passwords

**Use Vercel secrets:**
- Store sensitive values as encrypted secrets
- Reference with `@secret-name` in vercel.json

#### CORS Configuration (Post-PoC)

```typescript
// middleware.ts (if needed for external API access)

import { NextResponse } from 'next/server'

export function middleware(request: Request) {
  const response = NextResponse.next()

  response.headers.set('Access-Control-Allow-Origin', 'https://tellingcube.com')
  response.headers.set('Access-Control-Allow-Methods', 'GET, POST, OPTIONS')
  response.headers.set('Access-Control-Allow-Headers', 'Content-Type')

  return response
}
```

#### Input Validation

**Validate all API inputs:**

```typescript
// lib/validators.ts

import { z } from 'zod'

export const GenerateRequestSchema = z.object({
  scenarioType: z.enum(['bakery', 'hotel', 'tech_startup']),
})

export const ScenarioIdSchema = z.string().uuid()

// Usage
const result = GenerateRequestSchema.safeParse(body)
if (!result.success) {
  return Response.json(
    { error: { code: 'INVALID_INPUT', message: result.error.message } },
    { status: 400 }
  )
}
```

### Performance

#### Database Query Optimization

**Indexes (already in schema):**
- `scenarioId` - Foreign key lookups
- `eventType` - Filter events by type
- `timePeriod` - Group by month
- Composite: `(scenarioId, eventType)` - Common query pattern

**Query patterns:**

```typescript
// ✅ GOOD: Use indexes
const events = await prisma.event.findMany({
  where: {
    scenarioId: 'xxx',
    eventType: 'sales_transaction',
  },
  select: {
    amountEur: true,
    timePeriod: true,
  },
})

// ❌ BAD: Full table scan
const events = await prisma.event.findMany({
  where: {
    metadata: { path: ['customerName'], equals: 'ABC' }, // No index on JSON field
  },
})
```

#### API Response Caching

**Static scenarios (Post-PoC):**

```typescript
// app/api/views/finance/route.ts

export const revalidate = 3600 // Cache for 1 hour

export async function GET(request: Request) {
  // ... fetch data

  return Response.json(data, {
    headers: {
      'Cache-Control': 'public, s-maxage=3600, stale-while-revalidate=86400',
    },
  })
}
```

#### Frontend Performance

**Code splitting:**
- Use dynamic imports for charts: `const Chart = dynamic(() => import('@/components/charts/LineChart'))`
- Split Lovable components by route
- Lazy load Recharts (not needed on landing page)

**Image optimization:**
- Use Next.js `<Image>` component
- WebP format with fallbacks
- Lazy load images below fold

**Bundle analysis:**

```bash
# Analyze bundle size
ANALYZE=true pnpm build
```

**Target metrics:**
- Landing page: <100KB JS
- Results view: <200KB JS (includes Recharts)
- First Contentful Paint: <1.5s
- Time to Interactive: <3s

#### Serverless Function Optimization

**Cold start mitigation:**
- Keep functions warm with synthetic pings (Post-PoC)
- Minimize dependencies in API routes
- Use Edge Functions for simple endpoints

**Timeout handling:**

```typescript
// Set timeout for async generation
const GENERATION_TIMEOUT = 5 * 60 * 1000 // 5 minutes

const timeoutPromise = new Promise((_, reject) =>
  setTimeout(() => reject(new Error('Generation timeout')), GENERATION_TIMEOUT)
)

const result = await Promise.race([
  generateScenario(),
  timeoutPromise,
])
```

---

**Status:** Section 12 (Security & Performance) - Complete

---

## Testing Strategy

Testing approach for PoC and post-PoC phases.

### PoC Testing (Manual)

**Week 1-2 (Brother's validation):**

**Manual test checklist:**

1. ✅ Generate Bakery scenario
2. ✅ Wait for generation (30-90s)
3. ✅ View Finance dashboard
4. ✅ Verify IBCS chart styling (grayscale, no vertical lines)
5. ✅ Check consistency badge (green = pass)
6. ✅ Validate numbers reconcile:
   - Revenue in P&L = Sum of sales transactions
   - Payroll in P&L = Employee count × monthly salary × 12
7. ✅ Test on mobile (iPhone/Android)
8. ✅ Share scenario URL with colleague (24h expiry)
9. ✅ Test error handling (invalid scenario ID)

**Quality gates:**
- ✅ 100% validation pass rate (10/10 scenarios)
- ✅ IBCS compliance verified by brother (controller background)
- ✅ Generation time <90s average

### Post-PoC Testing (Automated)

#### Unit Tests (Vitest)

**File:** `lib/services/generation.service.test.ts`

```typescript
import { describe, it, expect, vi } from 'vitest'
import { GenerationService } from './generation.service'

describe('GenerationService', () => {
  it('should generate master data for bakery', async () => {
    const service = new GenerationService()
    const masterData = await service['generateMasterData']('bakery')

    expect(masterData).toHaveProperty('companyName')
    expect(masterData).toHaveProperty('employees')
    expect(masterData.employees).toHaveLength(10)
  })

  it('should validate events reconcile', async () => {
    const service = new GenerationService()
    const events = [/* mock events */]
    const validation = await service['validateEvents'](events)

    expect(validation.valid).toBe(true)
    expect(validation.errors).toHaveLength(0)
  })
})
```

**Run tests:**
```bash
pnpm test
```

#### Integration Tests

**File:** `app/api/generate/route.test.ts`

```typescript
import { POST } from './route'

describe('POST /api/generate', () => {
  it('should return generationId', async () => {
    const request = new Request('http://localhost:3000/api/generate', {
      method: 'POST',
      body: JSON.stringify({ scenarioType: 'bakery' }),
    })

    const response = await POST(request)
    const data = await response.json()

    expect(response.status).toBe(200)
    expect(data).toHaveProperty('generationId')
    expect(data.status).toBe('processing')
  })
})
```

#### E2E Tests (Playwright)

**File:** `tests/e2e/generation-flow.spec.ts`

```typescript
import { test, expect } from '@playwright/test'

test('complete generation flow', async ({ page }) => {
  // 1. Land on homepage
  await page.goto('/')

  // 2. Click Bakery button
  await page.click('text=Generate Bakery Scenario')

  // 3. Wait for loading page
  await expect(page).toHaveURL(/\/generate\/loading/)

  // 4. Wait for redirect to results (max 90s)
  await page.waitForURL(/\/results\/.*/, { timeout: 95000 })

  // 5. Verify consistency badge
  await expect(page.locator('text=Multi-view consistency verified')).toBeVisible()

  // 6. Verify Finance view renders
  await expect(page.locator('text=Finance View')).toBeVisible()

  // 7. Verify charts render
  await expect(page.locator('.recharts-wrapper')).toBeVisible()
})
```

**Run E2E tests:**
```bash
pnpm test:e2e
```

### Test Coverage Goals

**Post-PoC targets:**
- Unit tests: >80% coverage
- Integration tests: All API endpoints
- E2E tests: 3 critical user flows

---

**Status:** Section 13 (Testing Strategy) - Complete

---

## Coding Standards

Code style, linting, and formatting rules.

### TypeScript Configuration

**File:** `tsconfig.json`

```json
{
  "compilerOptions": {
    "target": "ES2022",
    "lib": ["ES2022", "dom", "dom.iterable"],
    "jsx": "preserve",
    "module": "esnext",
    "moduleResolution": "bundler",
    "resolveJsonModule": true,
    "allowJs": true,
    "strict": true,
    "noUncheckedIndexedAccess": true,
    "forceConsistentCasingInFileNames": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "incremental": true,
    "plugins": [{ "name": "next" }],
    "paths": {
      "@/*": ["./*"]
    }
  },
  "include": ["next-env.d.ts", "**/*.ts", "**/*.tsx", ".next/types/**/*.ts"],
  "exclude": ["node_modules"]
}
```

### ESLint Configuration

**File:** `.eslintrc.json`

```json
{
  "extends": [
    "next/core-web-vitals",
    "plugin:@typescript-eslint/recommended"
  ],
  "rules": {
    "@typescript-eslint/no-unused-vars": "error",
    "@typescript-eslint/no-explicit-any": "warn",
    "@typescript-eslint/explicit-function-return-type": "off",
    "react/prop-types": "off",
    "react/react-in-jsx-scope": "off"
  }
}
```

### Prettier Configuration

**File:** `.prettierrc`

```json
{
  "semi": false,
  "singleQuote": true,
  "trailingComma": "es5",
  "tabWidth": 2,
  "printWidth": 100,
  "arrowParens": "avoid"
}
```

**Format code:**
```bash
pnpm format
```

### Naming Conventions

**Files:**
- Components: `PascalCase.tsx` (e.g., `FinanceView.tsx`)
- Utilities: `kebab-case.ts` (e.g., `format-currency.ts`)
- Types: `kebab-case.ts` (e.g., `api.ts`)
- API routes: `route.ts` (Next.js convention)

**Variables:**
- `camelCase` for variables and functions
- `PascalCase` for components and types
- `SCREAMING_SNAKE_CASE` for constants

**Example:**

```typescript
// Constants
const MAX_EVENTS_PER_SCENARIO = 400
const GENERATION_TIMEOUT_MS = 90_000

// Functions
function calculateNetMargin(revenue: number, costs: number): number {
  return (revenue - costs) / revenue
}

// Components
export function FinanceView({ data }: FinanceViewProps) {
  return <div>...</div>
}

// Types
interface FinanceViewProps {
  data: FinanceViewResponse
}
```

### Code Organization

**Import order:**

```typescript
// 1. React/Next.js
import { useState } from 'react'
import { useRouter } from 'next/navigation'

// 2. External libraries
import { z } from 'zod'
import { prisma } from '@prisma/client'

// 3. Internal utilities
import { formatCurrency } from '@/lib/utils/format-currency'
import { GenerationService } from '@/lib/services/generation.service'

// 4. Components
import { FinanceView } from '@/components/views/FinanceView'

// 5. Types
import type { FinanceViewResponse } from '@/types/api'

// 6. Styles (if any)
import styles from './styles.module.css'
```

### Comments

**When to comment:**
- ✅ Complex business logic
- ✅ IBCS-specific requirements
- ✅ Event-sourcing validation rules
- ✅ Workarounds for known issues

**When NOT to comment:**
- ❌ Obvious code (`// Increment counter` for `count++`)
- ❌ Type definitions (types are self-documenting)

**Example:**

```typescript
// ✅ GOOD: Explains WHY
// Allow 1% tolerance for revenue reconciliation due to Decimal rounding in Prisma aggregations
const RECONCILIATION_TOLERANCE = 0.01

// ❌ BAD: Explains WHAT (obvious from code)
// Create a new scenario
const scenario = await prisma.scenario.create({ ... })
```

### Git Commit Messages

**Format:**
```
<type>: <subject>

<body>
```

**Types:**
- `feat:` New feature
- `fix:` Bug fix
- `refactor:` Code refactoring
- `docs:` Documentation
- `test:` Tests
- `chore:` Build/tooling

**Example:**

```
feat: Add revenue reconciliation validation

Implements ValidationService.checkRevenueReconciliation() which ensures
Sales transaction amounts match cash movement inbound totals within 1% tolerance.

Closes #12
```

---

**Status:** Section 14 (Coding Standards) - Complete

---

## Error Handling

Global error handling patterns and strategies.

### API Error Responses

**Standard error format:**

```typescript
// types/api.ts

export interface ApiError {
  error: {
    code: string
    message: string
    details?: Record<string, any>
  }
}
```

**Error codes:**

| Code | HTTP Status | Meaning |
|------|-------------|---------|
| `INVALID_INPUT` | 400 | Request validation failed |
| `MISSING_SCENARIO_ID` | 400 | Required parameter missing |
| `SCENARIO_NOT_FOUND` | 404 | Scenario doesn't exist or expired |
| `SCENARIO_EXPIRED` | 410 | Scenario >24 hours old |
| `RATE_LIMIT_EXCEEDED` | 429 | Too many requests |
| `GENERATION_FAILED` | 500 | AI generation error |
| `DATABASE_ERROR` | 500 | Prisma query failed |
| `VALIDATION_FAILED` | 500 | Consistency check failed |

### Error Handling Middleware

**File:** `lib/error-handler.ts`

```typescript
export function handleApiError(error: unknown): Response {
  console.error('API Error:', error)

  // Prisma errors
  if (error instanceof Prisma.PrismaClientKnownRequestError) {
    if (error.code === 'P2025') {
      return Response.json(
        { error: { code: 'SCENARIO_NOT_FOUND', message: 'Scenario not found' } },
        { status: 404 }
      )
    }
  }

  // Validation errors
  if (error instanceof z.ZodError) {
    return Response.json(
      { error: { code: 'INVALID_INPUT', message: error.message } },
      { status: 400 }
      )
  }

  // Claude API errors
  if (error instanceof AnthropicError) {
    return Response.json(
      { error: { code: 'GENERATION_FAILED', message: 'AI generation failed' } },
      { status: 500 }
    )
  }

  // Default error
  return Response.json(
    { error: { code: 'INTERNAL_ERROR', message: 'Something went wrong' } },
    { status: 500 }
  )
}
```

**Usage:**

```typescript
// app/api/generate/route.ts

export async function POST(request: Request) {
  try {
    // ... business logic
  } catch (error) {
    return handleApiError(error)
  }
}
```

### Frontend Error Handling

**Error boundary:**

```typescript
// components/ErrorBoundary.tsx
'use client'

import { Component, ReactNode } from 'react'

interface Props {
  children: ReactNode
}

interface State {
  hasError: boolean
}

export class ErrorBoundary extends Component<Props, State> {
  constructor(props: Props) {
    super(props)
    this.state = { hasError: false }
  }

  static getDerivedStateFromError() {
    return { hasError: true }
  }

  componentDidCatch(error: Error, errorInfo: any) {
    console.error('Error caught by boundary:', error, errorInfo)
    // Send to Sentry (Post-PoC)
  }

  render() {
    if (this.state.hasError) {
      return (
        <div className="min-h-screen flex items-center justify-center">
          <div className="text-center">
            <h1 className="text-2xl font-bold mb-4">Something went wrong</h1>
            <button onClick={() => window.location.reload()}>
              Reload page
            </button>
          </div>
        </div>
      )
    }

    return this.props.children
  }
}
```

### Logging

**Development:**
```typescript
console.log('Generation started for scenario:', scenarioId)
console.error('Failed to generate events:', error)
```

**Production (Post-PoC with Sentry):**

```typescript
import * as Sentry from '@sentry/nextjs'

Sentry.captureException(error, {
  tags: {
    scenarioId,
    scenarioType: 'bakery',
  },
})
```

---

**Status:** Section 15 (Error Handling) - Complete

---

## Monitoring & Observability

Monitoring setup and observability practices.

### PoC Monitoring (Vercel Built-in)

**Vercel Analytics Dashboard:**
- Deployment status
- Build logs
- Function logs
- Web Vitals (LCP, FID, CLS)
- Page views

**Access:** https://vercel.com/[team]/telling-cube/analytics

### Post-PoC Monitoring (Sentry)

**Setup:**

```bash
pnpm install @sentry/nextjs
npx @sentry/wizard -i nextjs
```

**Configuration:**

**File:** `sentry.client.config.ts`

```typescript
import * as Sentry from '@sentry/nextjs'

Sentry.init({
  dsn: process.env.NEXT_PUBLIC_SENTRY_DSN,
  tracesSampleRate: 0.1,
  environment: process.env.NODE_ENV,

  beforeSend(event, hint) {
    // Filter out development errors
    if (process.env.NODE_ENV === 'development') {
      return null
    }
    return event
  },
})
```

**File:** `sentry.server.config.ts`

```typescript
import * as Sentry from '@sentry/nextjs'

Sentry.init({
  dsn: process.env.SENTRY_DSN,
  tracesSampleRate: 0.1,

  integrations: [
    new Sentry.Integrations.Prisma({ client: prisma }),
  ],
})
```

### Key Metrics to Monitor

| Metric | Target | Alert Threshold |
|--------|--------|----------------|
| Generation success rate | >95% | <90% |
| Validation pass rate | 100% | <100% |
| Average generation time | <60s | >90s |
| API error rate | <1% | >5% |
| P95 API latency | <500ms | >2s |
| Database query time | <200ms | >1s |

### Custom Instrumentation

**Track generation metrics:**

```typescript
// lib/services/generation.service.ts

async generateInBackground(scenarioId: string, scenarioType: string) {
  const startTime = Date.now()

  try {
    await this.runGeneration(scenarioId, scenarioType)

    const duration = Date.now() - startTime
    console.log('Generation completed', { scenarioId, duration, scenarioType })

    // Send to analytics (Post-PoC)
    // analytics.track('generation_completed', { duration, scenarioType })

  } catch (error) {
    console.error('Generation failed', { scenarioId, error })
    Sentry.captureException(error)
  }
}
```

### Alerts (Post-PoC)

**Sentry Alerts:**
- Error rate > 5% → Email
- Generation timeout → Slack
- Database connection failures → PagerDuty

**Vercel Alerts:**
- Failed deployments → Email
- Function timeout → Email

---

**Status:** Section 16 (Monitoring & Observability) - Complete

---

## Summary & Next Steps

### Architecture Document Complete ✅

**Sections covered:**
1. ✅ Introduction
2. ✅ High Level Architecture (Lovable integration)
3. ✅ Tech Stack
4. ✅ Data Models
5. ✅ API Specification
6. ✅ Backend Architecture
7. ✅ Database Schema
8. ✅ Frontend Architecture
9. ✅ Unified Project Structure
10. ✅ Development Workflow
11. ✅ Deployment Architecture
12. ✅ Security & Performance
13. ✅ Testing Strategy
14. ✅ Coding Standards
15. ✅ Error Handling
16. ✅ Monitoring & Observability

### Questions & Decisions for Mario

**When you wake up, please review and answer:**

#### Q1: Claude API Setup
- [ ] Do you have an Anthropic API key already?
- [ ] What's your budget for API calls during PoC? (Estimate: €5-10 for 10-30 scenarios)

#### Q2: NeonDB Setup
- [ ] Have you created a NeonDB account?
- [ ] Which region for database: Frankfurt (eu-central-1) or US-East?

#### Q3: Lovable Components
- [ ] Are all Prompt 1-3 components fully complete in Lovable?
- [ ] Any issues with export/integration you foresee?

#### Q4: Brother's Validation Timeline
- [ ] When can your brother review the PoC? (Aiming for Week 1-2)
- [ ] Any specific IBCS requirements he'll check?

#### Q5: Repository
- [ ] Should I create the initial Next.js project structure?
- [ ] GitHub repo name: `telling-cube` or `tellingcube`?

#### Q6: Domain
- [ ] Do you own `tellingcube.com`?
- [ ] Or using Vercel subdomain for PoC: `telling-cube.vercel.app`?

### Immediate Next Steps (When You're Ready)

**Step 1: Environment Setup**
```bash
# Initialize Next.js project
npx create-next-app@latest telling-cube --typescript --tailwind --app --src-dir=false

cd telling-cube
pnpm install
```

**Step 2: Add Dependencies**
```bash
pnpm install prisma @prisma/client @anthropic-ai/sdk recharts zod
pnpm install -D @types/node
```

**Step 3: Set Up Prisma**
```bash
npx prisma init
# Copy schema from architecture doc to prisma/schema.prisma
npx prisma migrate dev --name init
npx prisma generate
```

**Step 4: Copy Lovable Components**
- Export from Lovable
- Copy to `components/` folder
- Update imports

**Step 5: Implement Backend Services**
- Create `lib/services/generation.service.ts`
- Create `lib/services/validation.service.ts`
- Create `lib/services/view-query.service.ts`

**Step 6: Implement API Routes**
- `app/api/generate/route.ts`
- `app/api/generate/status/route.ts`
- `app/api/views/finance/route.ts`
- `app/api/validate/route.ts`

**Step 7: Test Locally**
```bash
pnpm dev
# Open http://localhost:3000
# Test generation flow
```

**Step 8: Deploy to Vercel**
```bash
vercel
# Follow prompts
# Set environment variables in Vercel dashboard
```

### Estimated Timeline

**Week 1 (Days 1-3):**
- Project setup
- Lovable integration
- Backend services (Generation)

**Week 1 (Days 4-5):**
- API routes
- Validation service
- View query service

**Week 2 (Days 1-2):**
- Testing & debugging
- IBCS styling refinement

**Week 2 (Day 3):**
- Brother's validation session
- Feedback & adjustments

**Week 2 (Days 4-5):**
- PoC presentation to Jürgen Faisst (IBCS Institute CEO)?

---

## 🎉 Architecture Document Status: COMPLETE

**Total Sections:** 16/16 ✅
**Document Length:** ~2,000 lines
**Ready for:** Development kickoff

**Good night, Mario! Rest well - you have a solid foundation to build on when you wake up!** 🌙🇦🇹
