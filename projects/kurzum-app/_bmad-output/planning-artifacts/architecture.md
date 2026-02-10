---
stepsCompleted: [1, 2, 3, 4, 5, 6, 7, 8]
status: 'complete'
completedAt: '2026-02-10'
inputDocuments:
  - 'prd.md'
  - 'prd-validation-report.md'
  - 'product-brief-kurzum-app-2026-02-08.md'
  - 'ux-design-specification.md'
  - 'research/domain-interne-kommunikation-kmu-handwerk-research-2026-02-08.md'
  - 'research/market-ki-kommunikation-kmu-handwerk-research-2026-02-08.md'
  - 'docs/project-context.md'
  - 'docs/company-info.md'
  - 'docs/_masemIT/foerderungen/ffg.md'
  - 'docs/_masemIT/requirements/requirement-kurzum-landing-page-2026-02-10.md'
workflowType: 'architecture'
project_name: 'kurzum-app'
user_name: 'Sempre'
date: '2026-02-10'
scopeDecision: 'Gesamtarchitektur high-level, Landing Page Architektur im Detail'
---

# Architecture Decision Document

_This document builds collaboratively through step-by-step discovery. Sections are appended as we work through each architectural decision together._

## Project Context Analysis

### Requirements Overview

**Functional Requirements:**
43 FRs organized into 6 domains:

| Domain | FRs | Architectural Significance |
|---|---|---|
| **Voice & Media Capture** | FR1-FR7 | Core input layer. Audio recording API, camera API, offline local storage (IndexedDB), media file management. Must work with 2 taps max, offline-capable. |
| **AI Processing** | FR8-FR14 | Core processing layer. External AI service integration (Mistral), async queue processing (QStash), auto-assignment ML logic, processing state management. <30s latency target. |
| **Project Management** | FR15-FR22 | Domain data layer. CRUD operations, timeline views, "No Assignment" inbox pattern (AI-generated project suggestions), project-worker assignment. |
| **Team & Account Management** | FR23-FR30 | Multi-tenant administration. Registration, invite flow (SMS/email), RBAC enforcement, subscription tier management, trial logic, Stripe billing integration. |
| **Dashboard & Notifications** | FR31-FR34 | Real-time presentation layer. Role-filtered views, push notifications (Web Push API), voice reply capability from dashboard. |
| **Data & Compliance** | FR35-FR39 | Compliance infrastructure. Automated 90-day audio retention/deletion, audit trail logging, GDPR data export, transparency notices, right-to-deletion processing. |
| **Offline & Sync** | FR40-FR43 | PWA infrastructure. Service Worker lifecycle, background sync protocol, local storage management, connectivity state detection, zero-data-loss guarantee. |

**Non-Functional Requirements:**
27 NFRs across 5 categories:

| Category | NFRs | Key Architectural Drivers |
|---|---|---|
| **Performance** | NFR1-NFR6 | Voice→AI <30s, PWA load <3s on 3G, dashboard <2s, sync <10s non-blocking. Drives caching strategy, SSR/SSG decisions, API optimization. |
| **Security** | NFR7-NFR13 | TLS 1.3, AES-256 at rest, 100% EU data residency, Argon2 hashing, RLS tenant isolation, complete audit logging, automated audio retention. Drives database design, storage architecture, provider selection. |
| **Scalability** | NFR14-NFR18 | MVP: 10 companies/120 users → Year 1: 100 companies/1,200 users. Burst: 3x avg at shift boundaries. AI queue: 50+ messages in 5 min. ~50MB/company/month storage. Drives queue architecture, auto-scaling strategy. |
| **Accessibility (Field Conditions)** | NFR19-NFR23 | 48px touch targets, WCAG AA contrast, single-button recording, full offline capability, Android 10+/iOS 15+ support. Drives UI component constraints, PWA compatibility targets. |
| **Reliability** | NFR24-NFR27 | 99.5% uptime (6-18h CET), zero data loss, graceful AI degradation (queue-not-block), <30 min recovery. Drives queue architecture, fallback patterns, monitoring strategy. |

**Scale & Complexity:**

- Primary domain: Full-stack PWA with AI backend services
- Complexity level: Medium-High (offline-first + voice AI pipeline + GDPR compliance + multi-tenancy in combination is architecturally demanding, tempered by solo-founder MVP scope and existing infrastructure)
- Estimated architectural components: ~12 major (Auth, API Gateway, Voice Capture, Media Storage, AI Pipeline, Queue Processing, Project Service, Team Service, Dashboard/Notifications, Offline Sync Engine, Compliance Engine, Billing)

### Technical Constraints & Dependencies

**Existing Infrastructure (masemIT e.U.):**

| Service | Constraint | Implication |
|---|---|---|
| Vercel Pro | Existing account, proven deployment pipeline | Next.js App Router, Edge Functions, serverless API routes |
| NeonDB Launch | Existing PostgreSQL, supports RLS | Shared-database multi-tenancy, Row-Level Security policies, branching for dev/staging |
| Upstash | Redis-compatible cache, free tier | Session caching, rate limiting, real-time state |
| QStash | Message queue, free tier | AI processing queue, burst handling, retry logic |
| Resend Pro | Transactional email | Double-opt-in, team invitations, notifications |
| Cloudflare Turnstile | Bot protection | Form protection, API rate limiting |
| Vercel Analytics | Privacy-first analytics | No external tracking scripts, GDPR-compliant |

**External Dependencies:**

| Dependency | Risk Level | Constraint |
|---|---|---|
| Mistral AI (EU-hosted) | Medium | Must be EU-only processing. Queue-based to handle unavailability. API rate limits unknown at scale. |
| Stripe | Low | Subscription billing. Well-documented, proven. Not needed for MVP pilot (free). |
| Web Push API | Low | Browser-dependent. Not available in all iOS Safari versions (iOS 16.4+). |
| Service Worker API | Medium | PWA offline capability. IndexedDB for local storage. Background Sync API support varies. |

**Solo-Founder Resource Constraint:**
Architecture must be pragmatic — no microservices, no Kubernetes, no custom infrastructure. Serverless-first (Vercel), managed services (NeonDB, Upstash), queue-based decoupling (QStash). BMAD method for AI-assisted development.

**Development Constraint (FFG Funding):**
FFG Basisprogramm Kleinprojekt (€98,800, Antragsnr. 69168884) submitted 10.02.2026, approval expected May/June 2026. Before approval: only design, planning, and the recruiting landing page (as Vorbereitungsmaßnahme) may be developed. Full app development blocked until FFG-Freigabe.

### Cross-Cutting Concerns Identified

| Concern | Affects | Architectural Impact |
|---|---|---|
| **Tenant Isolation** | Database, API, Auth, File Storage, AI Processing | Every data path must be tenant-scoped. RLS policies on all tables. API middleware enforces company_id. File storage organized by tenant. AI context includes only tenant data. |
| **GDPR Compliance** | Audio Storage, AI Processing, Audit Trail, Data Retention, User Data | 90-day audio auto-deletion (scheduled job), complete audit logging (every read/write), data export API, right-to-deletion workflow, EU-only data residency enforced at infrastructure level. |
| **Offline Capability** | Voice Recording, Photo Capture, UI State, Sync Protocol | Service Worker intercepts all requests. IndexedDB stores recordings + photos locally. Background Sync API uploads when online. Conflict resolution strategy needed. UI must reflect online/offline state. |
| **AI Processing Pipeline** | Voice Messages, Project Assignment, Summaries | Async queue-based: recording → upload → queue → Mistral transcription → summary generation → auto-assignment → notification. Must handle failures gracefully (retry, fallback, user notification). Learning loop for assignment accuracy improvement. |
| **Authentication & Authorization** | All API routes, UI navigation, Data access | Long-lived mobile sessions, invite-based onboarding (link via SMS/email), 3-role RBAC (Monteur/Meister/Büro), subscription-tier enforcement (user limits, trial logic). |
| **Real-time Updates** | Dashboard, Notifications, Sync Status | Push notifications for new messages, dashboard activity feed updates, sync completion feedback. Consider SSE or polling — WebSockets may be over-engineered for MVP. |
| **Observability** | All services | Error tracking, performance monitoring, AI processing metrics, queue health, audit trail integrity. Essential for solo founder to maintain production system. |

## Starter Template Evaluation

### Primary Technology Domain

Full-stack web application (Next.js PWA) based on project requirements analysis. Two delivery contexts: Static/SSR landing page (Phase 1) and offline-first PWA with AI backend (Phase 2+).

### Starter Options Considered

| Option | Stack | Verdict |
|---|---|---|
| `create-next-app` + `shadcn init` + Drizzle | Next.js 16 + Tailwind v4 + shadcn/ui + Drizzle ORM | **Selected** — clean foundation, maximum control, no unnecessary dependencies |
| `shadcn create` | Next.js + shadcn/ui (visual style system) | Too new (Dec 2025), less customization documentation, same manual DB/Auth work |
| `create-t3-app` | Next.js + tRPC + Drizzle/Prisma + NextAuth | tRPC unnecessary with App Router Server Actions, declining maintenance, no shadcn/ui |
| Vercel Postgres Auth Starter | Next.js + Drizzle + NeonDB + NextAuth | Good reference implementation, but auth patterns may not fit 3-role RBAC model, no shadcn/ui |

### Selected Starter: create-next-app + shadcn/ui init + Drizzle ORM

**Rationale for Selection:**
Solo-founder project requiring full understanding of every dependency. The modular approach (base framework → UI library → ORM) ensures each architectural layer is consciously added and understood. No framework lock-in (tRPC), no premature auth decisions, no maintenance risk from declining starters. Each piece is independently maintained by strong teams (Vercel, shadcn, Drizzle Team).

**Initialization Command:**

```bash
# Step 1: Create Next.js project
npx create-next-app@latest kurzum-app --typescript --tailwind --eslint --app --turbopack --empty --use-pnpm

# Step 2: Add shadcn/ui
cd kurzum-app
npx shadcn@latest init

# Step 3: Add Drizzle ORM + NeonDB driver
pnpm add drizzle-orm @neondatabase/serverless
pnpm add -D drizzle-kit

# Step 4: Add landing page dependencies
pnpm add resend @marsidev/react-turnstile
pnpm add @vercel/analytics
```

### Architectural Decisions Provided by Starter

**Language & Runtime:**
TypeScript strict mode, React 19.2 (via Next.js 16), Node.js runtime for API routes, Edge runtime available for specific routes.

**Styling Solution:**
Tailwind CSS v4 with CSS-first configuration (no `tailwind.config.js`). CSS variables for theming, compatible with shadcn/ui design tokens. Note: UX spec references v3 patterns — design tokens transfer directly, only configuration syntax changes.

**Build Tooling:**
Turbopack for development (fast HMR), Next.js built-in bundler for production. pnpm as package manager for efficient dependency management.

**Testing Framework:**
Not included by starter — to be added as needed (Vitest recommended for unit tests, Playwright for E2E).

**Code Organization:**
Next.js App Router conventions (`/app` directory), shadcn/ui components in `/components/ui/`, custom components in `/components/landing/` and `/components/shared/` per UX spec component strategy.

**Development Experience:**
Turbopack HMR, TypeScript IntelliSense, ESLint for code quality, Vercel CLI for deployment previews.

**ORM Decision: Drizzle ORM over Prisma**

| Factor | Drizzle | Prisma |
|---|---|---|
| NeonDB integration | First-class `@neondatabase/serverless` driver | Via adapter, historically edge issues |
| Bundle size | Minimal, no binary | Larger client engine |
| Serverless cold starts | Negligible | Noticeable (improved in v7) |
| Type safety | Code-first, inferred from TS | Generated via `prisma generate` |
| SQL proximity | Very close to raw SQL | Abstracted query API |
| Vercel ecosystem | Used in official Neon templates | Also supported |
| Migration tooling | `drizzle-kit` (automatic SQL generation) | `prisma migrate` (mature) |

Selected Drizzle for: serverless-optimized bundle, first-class Neon support, no codegen step, alignment with Vercel's own template choices.

**Current Versions (verified Feb 2026):**

| Technology | Version |
|---|---|
| Next.js | 16.1.6 |
| Tailwind CSS | v4.1.18 |
| shadcn/ui | 3.8.4 |
| React | 19.2 |

**Note:** Project initialization using these commands should be the first implementation story.

## Core Architectural Decisions

### Decision Priority Analysis

**Critical Decisions (Block Implementation):**
- Data validation: Zod (enables form handling, API validation, DB schema sync)
- Form handling: React Hook Form + Zod Resolver
- API pattern: Route Handler for landing page form, Server Actions for app CRUD
- Analytics: masemIT Analytics (self-hosted, GDPR-compliant)
- API security: Turnstile + Upstash Rate Limiting + Zod validation

**Important Decisions (Shape Architecture):**
- Multi-tenancy: Shared DB with RLS (company_id on all tables)
- Auth strategy: Auth.js v5 (Phase 2)
- State management: React useState for landing page, Zustand/TanStack Query for app
- Error handling: Unified response format across all API routes
- Environment strategy: dev/preview/production with NeonDB branching

**Deferred Decisions (Post-MVP / Phase 2+):**
- Auth library final selection (Auth.js v5 recommended, deferred until FFG approval)
- Real-time update pattern (SSE vs. polling vs. WebSocket)
- PWA Service Worker strategy (Serwist)
- File storage provider for audio/photos
- Stripe billing integration details

### Data Architecture

**Validation: Zod**
- Schema-first TypeScript-native validation, works client- and server-side
- `drizzle-zod` generates Zod schemas from Drizzle table definitions — single source of truth
- Used in: API route request validation, form validation (via React Hook Form resolver), Server Action inputs

**Form Handling: React Hook Form + @hookform/resolvers/zod**
- Performant (minimal re-renders), industry standard
- Landing page: WaitlistForm with 7 fields (5 required, 2 optional)
- App (Phase 2): Project creation, team invite, settings forms

**Migration Strategy: drizzle-kit**
- `drizzle-kit generate` for production (SQL migration files, version-controlled)
- `drizzle-kit push` for development iteration (direct schema push to dev DB)
- Migration files committed to git for audit trail

**Multi-Tenancy (High-Level, Phase 2):**
- Shared database with Row-Level Security (RLS) — pragmatic for MVP scale (<100 companies)
- Every table gets `company_id` column (FK to companies table)
- PostgreSQL RLS policies enforce tenant isolation at DB level
- Application middleware sets `company_id` from authenticated session
- Drizzle queries always filtered by `where(eq(table.companyId, ctx.companyId))`

**Caching Strategy:**
- Landing page: SSG/ISR via Next.js (static pages, no runtime caching needed)
- App (Phase 2): Upstash Redis for session data, rate limiting, and frequently accessed data (project lists, team members)

### Authentication & Security

**Landing Page: No Authentication**
- Form submissions are anonymous (no user accounts)
- Bot protection: Cloudflare Turnstile (invisible mode preferred)
- Rate limiting: Upstash `@upstash/ratelimit` — 5 submissions per IP per hour

**App Authentication (High-Level, Phase 2):**
- Recommended: Auth.js v5 (NextAuth) — Next.js-native, Drizzle adapter available, self-hosted (GDPR)
- Long-lived mobile sessions (field workers don't want daily login)
- Invite-based onboarding: SMS/email link → set password → done
- 3-role RBAC: Monteur (restricted), Meister (admin), Büro (admin)
- Final selection deferred to Phase 2 development start

**API Route Security (Landing Page):**

| Layer | Implementation |
|---|---|
| Bot Protection | Cloudflare Turnstile token verification (server-side) |
| Rate Limiting | `@upstash/ratelimit` — 5 requests/IP/hour on `/api/waitlist` |
| Input Validation | Zod schema validation of request body |
| CORS | Only `kurzum.app` origin allowed |
| Content-Type | Reject non-JSON requests |

### API & Communication Patterns

**Landing Page: Route Handler**
- `POST /api/waitlist` — single API endpoint
- Accepts JSON body with Turnstile token + form data
- Server-side: Turnstile verification → Rate limit check → Zod validation → DB insert → Resend double-opt-in email
- Returns unified response format

**App (High-Level, Phase 2):**
- Server Actions for CRUD operations (projects, team, settings)
- Route Handlers for external webhooks (Stripe callbacks, QStash AI processing callbacks)
- QStash for async AI processing queue (voice → transcription → summary → assignment)

**Unified Error Response Format:**

```typescript
type ApiResponse<T> =
  | { success: true; data: T }
  | { success: false; error: { code: string; message: string } }
```

Standard error codes: `VALIDATION_ERROR`, `BOT_DETECTED`, `RATE_LIMITED`, `INTERNAL_ERROR`, `NOT_FOUND`, `UNAUTHORIZED`

### Frontend Architecture

**Server vs. Client Components (Landing Page):**

| Type | Components |
|---|---|
| Server Components | HeroSection, ProblemCard, SolutionStep, USPItem, PilotBenefitList, PageFooter, SectionWrapper |
| Client Components | WaitlistForm (form state + API call), ScrollFadeIn (IntersectionObserver), ChatMockup (animation timing) |

**State Management:**
- Landing Page: React `useState` in WaitlistForm (form state, submission state, confirmation state). No global state needed.
- App (High-Level, Phase 2): Zustand for lightweight global state (auth, offline status). TanStack Query or `useSWR` for server data caching.

### Infrastructure & Deployment

**Environment Configuration:**

| Environment | Vercel | NeonDB | Purpose |
|---|---|---|---|
| `development` | Local (`next dev`) | Dev branch | Local development |
| `preview` | Vercel Preview Deployment | Preview branch (auto per PR) | PR review, testing |
| `production` | Vercel Production | Main branch | kurzum.app live |

NeonDB branching integrated with Vercel: each preview deployment gets its own database branch.

**CI/CD: Vercel Auto-Deploy**
- GitHub push → Vercel build → Deploy (automatic)
- Build gates: ESLint (no errors), TypeScript strict (no type errors)
- Optional: Lighthouse CI via Vercel Web Vitals monitoring
- No custom CI/CD pipeline needed for landing page

**Analytics: masemIT Analytics (self-hosted)**

Replaces Vercel Analytics. Self-hosted at `analytics.masem.at` (v1.3.1), GDPR-compliant, supports all required tracking events.

Integration:
```tsx
// app/layout.tsx
<Script
  src="https://analytics.masem.at/tracker.js"
  strategy="afterInteractive"
  data-project="kurzum-app"
/>
```

Event mapping:

| Requirement Event | Implementation |
|---|---|
| `page_view` | Automatic via tracker |
| `scroll_depth` (25/50/75/100%) | `data-engagement-pages` attribute |
| `form_start` | `masemit.track('form_start')` |
| `form_submit` | `masemit.track('form_submit', { gewerk, size })` |
| `form_error` | `masemit.track('form_error', { field })` |
| `cta_click` | `masemit.track('cta_click', { location })` |

UTM tracking: Automatic via page URL parameters (`utm_source`, `utm_medium`, `utm_campaign`).

**Performance Monitoring:**
- Vercel Speed Insights (Web Vitals: FCP, LCP, CLS, TBT) — parallel to masemIT Analytics
- Vercel built-in function logs for API route monitoring

**Error Tracking:**
- Vercel Log Drain for serverless function errors
- Optional: Sentry Free Tier for structured error tracking (evaluate if needed)
- NeonDB query monitoring via Neon dashboard

**Uptime Monitoring:**
- Vercel built-in monitoring
- Optional: BetterStack Free Tier for external uptime checks

### Decision Impact Analysis

**Implementation Sequence (Landing Page):**
1. Project init (`create-next-app` + `shadcn init` + Drizzle + dependencies)
2. NeonDB schema + Drizzle config + migration
3. Design tokens (Tailwind CSS v4 variables from UX spec)
4. Static sections (Server Components: Hero, Problem, Example, Solution, USP, Pilot, Footer)
5. Interactive components (Client Components: ChatMockup, ScrollFadeIn)
6. WaitlistForm (React Hook Form + Zod + Turnstile)
7. API route (`/api/waitlist` with full security stack)
8. Email integration (Resend double-opt-in)
9. Analytics integration (masemIT Analytics)
10. Confirmation page (`/confirmed`)
11. SEO + OG tags + meta
12. Testing + Lighthouse audit

**Cross-Component Dependencies:**
- Zod schemas shared between form validation (client) and API validation (server) — single source of truth via `drizzle-zod`
- Turnstile token flows from client form → API route → Cloudflare verification
- masemIT Analytics script must load before user interactions (but `afterInteractive` is correct)
- Design tokens consumed by all components — must be configured before component development

## Implementation Patterns & Consistency Rules

### Pattern Categories Defined

**Critical Conflict Points Identified:** 5 categories where AI agents could make different choices — naming, structure, format, communication, and process patterns.

### Naming Patterns

**Database Naming (Drizzle + NeonDB):**

| Element | Convention | Example |
|---|---|---|
| Tables | `snake_case`, **plural** | `waitlist_entries`, `companies`, `voice_messages` |
| Columns | `snake_case` | `company_id`, `created_at`, `email_confirmed` |
| Primary Keys | Always `id` | `id` (serial or uuid) |
| Foreign Keys | `{referenced_table_singular}_id` | `company_id`, `user_id`, `project_id` |
| Indexes | `idx_{table}_{column}` | `idx_waitlist_entries_email` |
| Timestamps | `created_at`, `updated_at` on every table | Automatic via Drizzle defaults |
| Booleans | `is_` or `has_` prefix | `is_confirmed`, `has_synced` |

**API Naming (Next.js App Router):**

| Element | Convention | Example |
|---|---|---|
| Route Handlers | `kebab-case` directories | `/api/waitlist/route.ts` |
| Query Params | `camelCase` | `?companyId=123&sortBy=date` |
| JSON Request Body | `camelCase` | `{ "firstName": "Stefan", "companyName": "..." }` |
| JSON Response Body | `camelCase` | `{ "success": true, "data": { "entryId": 42 } }` |

**Code Naming (TypeScript/React):**

| Element | Convention | Example |
|---|---|---|
| Files (Components) | `kebab-case.tsx` | `hero-section.tsx`, `waitlist-form.tsx` |
| Files (Utilities) | `kebab-case.ts` | `api-response.ts`, `validation.ts` |
| React Components | `PascalCase` | `HeroSection`, `WaitlistForm` |
| Functions | `camelCase` | `validateForm()`, `submitWaitlistEntry()` |
| Variables/Consts | `camelCase` | `formData`, `isSubmitting` |
| Env Variables | `SCREAMING_SNAKE_CASE` | `DATABASE_URL`, `RESEND_API_KEY` |
| Types/Interfaces | `PascalCase` | `WaitlistEntry`, `ApiResponse<T>` |
| Zod Schemas | `camelCase` + `Schema` suffix | `waitlistEntrySchema`, `projectSchema` |
| CSS Variables | `kebab-case` with `--` prefix | `--color-accent`, `--radius-lg` |

### Structure Patterns

**Test Organization: Co-located**
- Tests live next to source: `waitlist-form.test.tsx` beside `waitlist-form.tsx`
- No separate `__tests__/` directory
- Naming: `{filename}.test.ts(x)` for unit tests, `{feature}.e2e.ts` for E2E

**Component Organization:**

Landing Page (Phase 1):
```
components/
├── ui/           # shadcn/ui (auto-generated, don't manually edit)
├── landing/      # Landing page components
└── shared/       # Shared between landing and app (Phase 2)
```

App (High-Level, Phase 2):
```
components/
├── ui/           # shadcn/ui
├── landing/      # Landing page
├── app/          # App components by feature
│   ├── voice/    # Voice recording, playback
│   ├── project/  # Project management
│   ├── team/     # Team management
│   └── dashboard/# Dashboard views
└── shared/       # Shared components
```

**Utility/Service Organization:**
```
lib/
├── db/           # Drizzle config, schema, migrations
│   ├── schema.ts # Drizzle table definitions
│   ├── index.ts  # DB connection
│   └── migrations/
├── validations/  # Zod schemas (exported, shared client/server)
├── actions/      # Server Actions (Phase 2)
├── utils.ts      # General utilities (cn() from shadcn, etc.)
└── constants.ts  # App-wide constants
```

### Format Patterns

**API Response Format:**

```typescript
// lib/api-response.ts
type ApiResponse<T> =
  | { success: true; data: T }
  | { success: false; error: { code: ErrorCode; message: string } }

type ErrorCode =
  | 'VALIDATION_ERROR'
  | 'BOT_DETECTED'
  | 'RATE_LIMITED'
  | 'INTERNAL_ERROR'
  | 'NOT_FOUND'
  | 'UNAUTHORIZED'
  | 'FORBIDDEN'
```

**Date/Time:**
- DB: `timestamp with time zone` (PostgreSQL)
- API/JSON: ISO 8601 strings (`"2026-02-10T14:30:00.000Z"`)
- UI: Formatted with `Intl.DateTimeFormat('de-AT', ...)` — no date-fns/moment
- Timezone: Always UTC in DB and API, local formatting only in UI

**Null Handling:**
- DB: `null` for optional fields
- API: Fields with `null` value are included (not omitted)
- UI: Optional fields show fallback text or are hidden

### Communication Patterns

**Analytics Events (masemIT Analytics):**

| Convention | Rule | Example |
|---|---|---|
| Event Names | `snake_case` | `form_submit`, `cta_click`, `scroll_depth` |
| Event Properties | `snake_case` | `{ gewerk: '...', company_size: '...' }` |

**Logging:**

| Level | When | Example |
|---|---|---|
| `error` | Unexpected errors, failed external calls | Resend API error, DB connection error |
| `warn` | Expected but unusual situations | Rate limit reached, Turnstile suspicious score |
| `info` | Business events | Waitlist entry created, double-opt-in confirmed |

Format: `console.error/warn/info` with structured object — no string concatenation.

```typescript
console.info('waitlist_entry_created', { email: '***@***.com', gewerk: 'Elektrotechnik' })
```

### Process Patterns

**Error Handling — Three Layers:**

| Layer | Responsibility | Pattern |
|---|---|---|
| **API Route** | Zod validation → 400, Turnstile → 403, Rate limit → 429, Everything else → 500 | try/catch wrapper, always `ApiResponse` format |
| **Client Component** | Form errors inline (per-field), network errors as toast | React state for error tracking, no `window.alert` |
| **Server Component** | `error.tsx` Error Boundary per route segment | Next.js Error Boundary convention |

**Rule:** No `throw` in Client Components. Errors are handled as state. Server errors are communicated via `ApiResponse`.

**Loading States:**

| Context | Pattern |
|---|---|
| Landing Page (static) | No loading state needed (SSG) |
| Form Submission | Button spinner + `"Wird gesendet..."` + fields disabled |
| App Dashboard (Phase 2) | shadcn/ui `Skeleton` matching card layout |
| App AI Processing (Phase 2) | Inline spinner below message, never full-screen loader |

**Validation Timing:**
- Client: `onBlur` (not `onChange` — avoids frustration while typing)
- Server: Always full Zod validation, even if client already validated (Defense in Depth)

### Enforcement Guidelines

**All AI Agents MUST:**

1. **Naming:** Follow exactly the conventions defined above. No deviations on DB naming (snake_case), component naming (kebab-case files, PascalCase exports), API naming (camelCase JSON).
2. **Imports:** Absolute imports via `@/` alias (`import { cn } from '@/lib/utils'`). No relative imports across more than one level (`../../` forbidden).
3. **Types:** No `any`. Strictly typed. Zod schemas as single source of truth for validation.
4. **API Responses:** Always `ApiResponse<T>` format. No direct JSON responses without wrapper.
5. **Error Messages:** User-facing errors in German. Log errors in English. No technical details exposed to users.
6. **Components:** Server Components as default. Only `'use client'` when state, effects, or browser APIs are needed.
7. **Styling:** Exclusively Tailwind CSS classes. No inline `style={}`. No CSS Modules. Design tokens from CSS variables.

### Anti-Patterns (Forbidden)

```typescript
// ❌ WRONG: camelCase table name
export const waitlistEntries = pgTable('waitlistEntries', ...)
// ✅ CORRECT: snake_case table name
export const waitlistEntries = pgTable('waitlist_entries', ...)

// ❌ WRONG: Direct JSON response
return Response.json({ id: 1, name: 'test' })
// ✅ CORRECT: ApiResponse wrapper
return Response.json({ success: true, data: { id: 1, name: 'test' } })

// ❌ WRONG: Relative imports across multiple levels
import { schema } from '../../../lib/db/schema'
// ✅ CORRECT: Absolute import
import { schema } from '@/lib/db/schema'

// ❌ WRONG: any type
const handleSubmit = (data: any) => ...
// ✅ CORRECT: Zod-inferred type
const handleSubmit = (data: z.infer<typeof waitlistFormSchema>) => ...

// ❌ WRONG: Technical error message to user
error: { message: 'PostgreSQL unique constraint violation on email' }
// ✅ CORRECT: User-friendly in German
error: { code: 'VALIDATION_ERROR', message: 'Diese E-Mail-Adresse ist bereits registriert.' }
```

## Project Structure & Boundaries

### Complete Project Directory Structure

```
kurzum-app/
├── .env.example                      # Environment variable template (committed)
├── .env.local                        # Local env variables (git-ignored)
├── .eslintrc.json                    # ESLint configuration
├── .gitignore
├── components.json                   # shadcn/ui configuration
├── drizzle.config.ts                 # Drizzle Kit configuration
├── next.config.ts                    # Next.js configuration
├── package.json
├── pnpm-lock.yaml
├── postcss.config.mjs                # PostCSS (Tailwind v4)
├── tsconfig.json                     # TypeScript strict mode
│
├── app/                              # Next.js App Router
│   ├── globals.css                   # Tailwind v4 imports + design tokens (CSS variables)
│   ├── layout.tsx                    # Root layout: <html lang="de">, fonts, analytics script
│   ├── page.tsx                      # Landing page (kurzum.app/)
│   ├── error.tsx                     # Root error boundary
│   ├── not-found.tsx                 # 404 page
│   ├── robots.ts                     # robots.txt generation
│   ├── sitemap.ts                    # sitemap.xml generation
│   ├── opengraph-image.tsx           # OG image for LinkedIn sharing
│   │
│   ├── confirmed/                    # Double-opt-in confirmation
│   │   └── page.tsx                  # /confirmed — "Du bist dabei!"
│   │
│   ├── impressum/                    # Legal: Impressum
│   │   └── page.tsx
│   │
│   ├── datenschutz/                  # Legal: Datenschutzerklärung
│   │   └── page.tsx
│   │
│   └── api/                          # API Route Handlers
│       └── waitlist/
│           └── route.ts              # POST /api/waitlist
│
├── components/                       # React components
│   ├── ui/                           # shadcn/ui (auto-generated, don't edit)
│   │   ├── button.tsx
│   │   ├── input.tsx
│   │   ├── textarea.tsx
│   │   ├── select.tsx
│   │   ├── label.tsx
│   │   └── badge.tsx
│   │
│   ├── landing/                      # Custom landing page components
│   │   ├── hero-section.tsx          # Full-viewport hero, headline, CTA
│   │   ├── problem-card.tsx          # Pain point card (border-left orange)
│   │   ├── chat-mockup.tsx           # THE conversion element (voice → AI)
│   │   ├── voice-bubble.tsx          # Sub-component: voice input bubble
│   │   ├── ai-result-card.tsx        # Sub-component: AI output card
│   │   ├── solution-step.tsx         # Numbered step (1-2-3)
│   │   ├── usp-item.tsx             # USP list item
│   │   ├── pilot-benefit-list.tsx    # Pilot program benefits
│   │   ├── waitlist-form.tsx         # Form: RHF + Zod + Turnstile (Client)
│   │   ├── form-confirmation.tsx     # Replaces form on success
│   │   ├── section-wrapper.tsx       # Consistent section container
│   │   ├── scroll-fade-in.tsx        # IntersectionObserver animation (Client)
│   │   └── page-footer.tsx           # Legal footer
│   │
│   └── shared/                       # Shared (empty Phase 1)
│
├── lib/                              # Utilities, services, config
│   ├── db/                           # Database layer
│   │   ├── index.ts                  # Drizzle instance + NeonDB connection
│   │   ├── schema.ts                 # Table definitions (waitlist_entries)
│   │   └── migrations/               # drizzle-kit generated SQL
│   │
│   ├── validations/                  # Zod schemas (shared client/server)
│   │   └── waitlist.ts               # waitlistFormSchema
│   │
│   ├── mms.ts                        # MMS API client (api.masem.at)
│   ├── api-response.ts              # ApiResponse<T> type + helper functions
│   ├── turnstile.ts                  # Server-side Turnstile verification
│   ├── rate-limit.ts                 # Upstash @upstash/ratelimit config
│   ├── email.ts                      # Resend: double-opt-in + confirmation
│   └── utils.ts                      # cn() + general utilities
│
├── public/                           # Static assets
│   ├── fonts/
│   │   └── inter-var.woff2           # Self-hosted Inter (Latin + ä/ö/ü/ß/€)
│   ├── favicon.ico
│   └── apple-touch-icon.png          # 180x180
│
└── e2e/                              # End-to-end tests (Playwright)
    └── waitlist-form.e2e.ts
```

**Phase 2 Extensions (High-Level, after FFG approval):**

```
app/
├── (app)/                            # App route group (after auth)
│   ├── layout.tsx                    # App layout with bottom nav
│   ├── dashboard/                    # Stefan/Andrea: Overview
│   ├── projects/                     # Project management
│   │   ├── page.tsx                  # Project list
│   │   └── [projectId]/             # Project detail + timeline
│   ├── record/                       # Markus: Voice recording
│   ├── inbox/                        # "No Assignment" inbox
│   ├── team/                         # Team management
│   └── settings/                     # Account + billing
├── (auth)/                           # Auth route group
│   ├── login/
│   ├── register/
│   └── invite/[token]/
└── api/
    ├── voice/                        # Voice upload + processing
    ├── projects/                     # Project CRUD
    ├── team/                         # Team management
    ├── webhooks/
    │   ├── qstash/                   # AI processing callbacks
    │   └── stripe/                   # Billing webhooks
    └── export/                       # GDPR data export

components/app/
├── voice/                            # RecordButton, WaveformDisplay, etc.
├── project/                          # ProjectCard, Timeline, etc.
├── team/                             # MemberList, InviteForm, etc.
└── dashboard/                        # DashboardCard, ActivityFeed, etc.

lib/
├── auth/                             # Auth.js v5 config
├── ai/                               # Mistral API integration
├── storage/                          # Audio/photo file storage
├── sync/                             # Offline sync logic
└── queue/                            # QStash job definitions
```

### Architectural Boundaries

**API Boundaries:**

| Boundary | Landing Page (Phase 1) | App (Phase 2, High-Level) |
|---|---|---|
| **Public API** | `POST /api/waitlist` (anonymous, Turnstile-protected) | — |
| **Authenticated API** | — | All `/api/voice/*`, `/api/projects/*`, `/api/team/*` |
| **Webhook API** | — | `/api/webhooks/qstash/*`, `/api/webhooks/stripe/*` |
| **Export API** | — | `GET /api/export` (GDPR data export, Meister/Büro only) |
| **External: MMS API** | `POST /v1/parties`, consent tracking | Party management, consent, email preferences |

**Component Boundaries:**

| Boundary | Rule |
|---|---|
| `components/ui/` | shadcn/ui generated — NEVER manually edit. Add via `npx shadcn@latest add`. |
| `components/landing/` | Landing page only. Server Components default, Client only where marked. |
| `components/shared/` | Must work in BOTH landing page and app context. No landing-specific or app-specific logic. |
| `components/app/` (Phase 2) | App only. Feature-organized. Always behind auth. |

**Data Boundaries:**

| Layer | Responsibility | Access Rule |
|---|---|---|
| `lib/db/schema.ts` | Table definitions only | No business logic |
| `lib/validations/` | Zod schemas only | Importable by both client and server |
| `lib/mms.ts` | MMS API client only | Called from API routes, never from client |
| `app/api/*/route.ts` | HTTP handling + orchestration | Calls lib/ functions, never DB directly in complex cases |
| `lib/*.ts` (services) | Business logic + DB queries | Only called from API routes or Server Actions |

**External Service Boundaries:**

| Service | Base URL | Auth | Purpose in kurzum |
|---|---|---|---|
| **MMS API** | `https://api.masem.at/v1` | `x-api-key: mms_kurzum_app_xxx` | Party/contact management, GDPR consent tracking, email preferences |
| **Resend** | API SDK | `RESEND_API_KEY` | Sending actual emails (double-opt-in, confirmation) |
| **Cloudflare Turnstile** | Verification API | `TURNSTILE_SECRET_KEY` | Bot protection on form submit |
| **Upstash Redis** | REST API | `UPSTASH_REDIS_REST_TOKEN` | Rate limiting |
| **masemIT Analytics** | `analytics.masem.at` | `data-project` attribute | User behavior tracking |
| **NeonDB** | Connection string | `DATABASE_URL` | Local waitlist data storage |

### MMS Integration Architecture

**MMS Parties & Consents API** (`api.masem.at`) provides centralized lead/contact management across all masemIT projects. kurzum-app integrates as a source project.

**Party Lifecycle:**
- Waitlist signup → `POST /v1/parties` with `source_project: "kurzum_app"`, `marketing_consent: true`
- MMS automatically creates: Party record + consent record (immutable audit trail) + email preference + unsubscribe token
- Party deduplication: If email exists from another masemIT project (hoki, tellingcube), MMS returns existing party
- Party status progression: `lead` (signup) → `prospect` (confirmed) → `customer` (pilot start)

**What lives WHERE:**

| Data | Location | Rationale |
|---|---|---|
| Email, Name, Consent | MMS API (`api.masem.at`) | Centralized across masemIT, GDPR audit trail, deduplication |
| Company name, Gewerk, Size, Phone, Message, UTM | Local NeonDB (`waitlist_entries`) | kurzum-specific business data |
| Unsubscribe token | MMS API (returned on party creation) | Centralized email preference management |
| Double-opt-in status | Both (MMS consent + local `is_confirmed`) | MMS for audit, local for query performance |

**lib/mms.ts client interface:**

```typescript
// MMS API client for kurzum-app
const MMS_BASE_URL = 'https://api.masem.at/v1'

// Create or find party + record consent
async function createParty(data: {
  email: string
  displayName: string
  marketingConsent: boolean
  consentIp: string
  consentUserAgent: string
}): Promise<{
  partyId: string
  contactPointId: string
  created: boolean
  unsubscribeToken: string | null
}>

// Look up party by email
async function lookupParty(email: string): Promise<Party | null>

// Update party status (e.g., lead → prospect on double-opt-in confirm)
async function updatePartyStatus(
  partyId: string,
  status: 'lead' | 'prospect' | 'customer'
): Promise<Party>
```

### Requirements to Structure Mapping

**FR Category → File Mapping (Landing Page, Detail):**

| Requirement | Files |
|---|---|
| Waitlist Form (§4.7) | `components/landing/waitlist-form.tsx`, `lib/validations/waitlist.ts`, `app/api/waitlist/route.ts`, `lib/db/schema.ts`, `lib/mms.ts` |
| Double-Opt-In (§4.7) | `lib/email.ts`, `lib/mms.ts` (consent tracking), `app/confirmed/page.tsx` |
| Bot Protection (§6.1) | `lib/turnstile.ts`, `app/api/waitlist/route.ts` |
| Analytics Events (§7.1) | `app/layout.tsx` (script tag), `components/landing/waitlist-form.tsx` (event calls) |
| SEO + OG Tags (§6.3) | `app/layout.tsx` (metadata), `app/opengraph-image.tsx` |
| Legal Pages (§4.8) | `app/impressum/page.tsx`, `app/datenschutz/page.tsx` |
| UTM Tracking (§7.2) | `app/api/waitlist/route.ts` (extract from referrer/headers), `lib/db/schema.ts` (utm columns) |

**FR Category → Directory Mapping (App, High-Level):**

| FR Category | Primary Directory |
|---|---|
| Voice & Media Capture (FR1-FR7) | `components/app/voice/`, `app/(app)/record/`, `lib/storage/` |
| AI Processing (FR8-FR14) | `lib/ai/`, `lib/queue/`, `app/api/webhooks/qstash/` |
| Project Management (FR15-FR22) | `components/app/project/`, `app/(app)/projects/`, `app/(app)/inbox/` |
| Team & Account (FR23-FR30) | `components/app/team/`, `app/(app)/team/`, `app/(auth)/`, `lib/auth/` |
| Dashboard & Notifications (FR31-FR34) | `components/app/dashboard/`, `app/(app)/dashboard/` |
| Data & Compliance (FR35-FR39) | `lib/db/` (retention jobs), `app/api/export/`, `lib/mms.ts` (consent queries) |
| Offline & Sync (FR40-FR43) | `lib/sync/`, Service Worker (root level) |

**Cross-Cutting Concerns → Location:**

| Concern | Files |
|---|---|
| Tenant Isolation | `lib/db/schema.ts` (RLS policies), `middleware.ts` (company_id injection) |
| GDPR Compliance | MMS API (consent audit trail), `lib/db/schema.ts` (audit columns), scheduled jobs (audio retention) |
| Error Handling | `lib/api-response.ts`, `app/error.tsx` |
| Rate Limiting | `lib/rate-limit.ts` (used in API routes) |
| Analytics | `app/layout.tsx` (global script), component-level `masemit.track()` calls |
| Party/Contact Management | `lib/mms.ts` (centralized via MMS API) |

### Data Flow

**Landing Page Waitlist Flow (with MMS Integration):**

```
User fills form → WaitlistForm (Client Component)
  → Client-side Zod validation (onBlur)
  → Turnstile token generated
  → POST /api/waitlist (Route Handler)
    → Server-side Turnstile verification (lib/turnstile.ts)
    → Upstash rate limit check (lib/rate-limit.ts)
    → Server-side Zod validation (lib/validations/waitlist.ts)
    → MMS: POST /v1/parties (lib/mms.ts)
      → email, display_name, source_project: "kurzum_app"
      → marketing_consent: true, consent_ip, consent_user_agent
      → Returns: party_id, contact_point_id, unsubscribe_token, created
    → Local DB: INSERT waitlist_entries (lib/db/)
      → mms_party_id, company_name, gewerk, company_size, phone, message, utm_*
    → Resend: Double-opt-in email (lib/email.ts)
      → Includes unsubscribe link with MMS token
    → masemit.track('form_submit') via response header or client callback
    → Return ApiResponse<{ entryId }>
  → WaitlistForm shows FormConfirmation

User clicks email confirmation link → /confirmed?token=xxx
  → Lookup waitlist_entry by confirmation_token
  → Local DB: UPDATE waitlist_entries SET confirmed_at = now()
  → MMS: PATCH /v1/parties/:id (status: "prospect")
  → Resend: Notification to Mario
  → Show "Du bist dabei!" page
```

**App Voice Pipeline (High-Level, Phase 2):**

```
Markus records → IndexedDB (offline storage)
  → Background Sync → POST /api/voice/upload
    → Store audio in file storage
    → QStash: enqueue AI processing job
      → Mistral: transcription
      → Mistral: summarization
      → AI: project assignment
      → DB: save results
      → Web Push: notify relevant users
  → Dashboard shows new entry
```

### Database Schema (Landing Page — Detail)

**waitlist_entries table:**

| Column | Type | Nullable | Source |
|---|---|---|---|
| `id` | `serial` PRIMARY KEY | No | Auto |
| `mms_party_id` | `uuid` | No | MMS API response |
| `mms_contact_point_id` | `uuid` | No | MMS API response |
| `company_name` | `text` | No | Form field |
| `gewerk` | `text` | No | Form field (Elektrotechnik, Sanitär/Heizung/Lüftung, Sonstige) |
| `company_size` | `text` | No | Form field (1-5, 6-10, 11-20, 21-50, 50+) |
| `phone` | `text` | Yes | Form field (optional) |
| `message` | `text` | Yes | Form field (optional) |
| `utm_source` | `text` | Yes | URL parameter |
| `utm_medium` | `text` | Yes | URL parameter |
| `utm_campaign` | `text` | Yes | URL parameter |
| `confirmation_token` | `uuid` UNIQUE DEFAULT gen_random_uuid() | No | Auto (for double-opt-in link) |
| `confirmed_at` | `timestamptz` | Yes | Double-opt-in click timestamp |
| `created_at` | `timestamptz` DEFAULT now() | No | Auto |
| `updated_at` | `timestamptz` DEFAULT now() | No | Auto |

Indexes: `idx_waitlist_entries_mms_party_id` on `mms_party_id`, `idx_waitlist_entries_confirmation_token` on `confirmation_token`

### Environment Configuration

```bash
# .env.example

# Database (NeonDB)
DATABASE_URL=postgresql://user:pass@host/neondb?sslmode=require

# MMS API (api.masem.at)
MMS_API_KEY=mms_kurzum_app_xxx

# Email (Resend)
RESEND_API_KEY=re_xxx
RESEND_FROM_EMAIL=mario@kurzum.app

# Bot Protection (Cloudflare Turnstile)
NEXT_PUBLIC_TURNSTILE_SITE_KEY=0x4xxx
TURNSTILE_SECRET_KEY=0x4xxx

# Rate Limiting (Upstash)
UPSTASH_REDIS_REST_URL=https://xxx.upstash.io
UPSTASH_REDIS_REST_TOKEN=xxx

# Analytics (masemIT)
NEXT_PUBLIC_ANALYTICS_PROJECT=kurzum-app

# App URL
NEXT_PUBLIC_APP_URL=https://kurzum.app
```

## Architecture Validation Results

### Coherence Validation ✅

**Decision Compatibility:** All technology choices are compatible. Next.js 16 + Tailwind v4 + shadcn/ui 3.8.4 + Drizzle + NeonDB form a coherent, well-integrated stack. MMS API integration adds centralized party management without conflicting with local architecture. No version conflicts detected.

**Pattern Consistency:** Naming conventions (snake_case DB → camelCase API → PascalCase Components), API response formats (ApiResponse<T>), error handling (3-layer model), and component boundaries are internally consistent. No contradictory patterns found.

**Structure Alignment:** Project directory structure directly supports all architectural decisions. Each component maps to a specific location. Integration boundaries (MMS, Resend, Turnstile, Upstash, NeonDB) are clear and enforceable.

### Requirements Coverage ✅

**Landing Page Requirements:** All 12 acceptance criteria from the requirement document are architecturally supported — waitlist form, double-opt-in (with confirmation_token), bot protection, analytics, SEO/OG, legal pages, performance budget, GDPR compliance.

**App Requirements (High-Level):** All 43 FRs across 7 categories have directory-level mapping. All 27 NFRs addressed by infrastructure and technology choices.

**Cross-Cutting Concerns:** All 7 identified concerns (tenant isolation, GDPR, offline, AI pipeline, auth/RBAC, real-time, observability) mapped to specific architectural locations and patterns.

### Implementation Readiness ✅

**Decision Completeness:** All critical and important decisions documented with verified versions (Feb 2026). Deferred decisions (Auth, PWA, real-time) explicitly marked as Phase 2.

**Structure Completeness:** Complete file tree for Phase 1 (landing page) with every file specified. High-level structure for Phase 2 (app) with directory-level mapping.

**Pattern Completeness:** All 5 pattern categories (naming, structure, format, communication, process) fully specified with concrete examples and anti-patterns.

### Gaps Addressed

**Critical — Schema Fix (Applied):**
Added `confirmation_token` (uuid, unique, auto-generated) and `confirmed_at` (timestamptz, nullable) to `waitlist_entries` schema. Replaced `is_confirmed` boolean with `confirmed_at` timestamp for double-opt-in verification flow.

**Important — Pre-Launch Configuration Checklist:**

1. Register `kurzum.app` domain
2. Configure DNS: Vercel (A/CNAME) + Resend (SPF, DKIM, MX)
3. Register `kurzum_app` as `source_project` in MMS
4. Create MMS API key: scopes `parties:write`, `consents:write`, `consents:read`
5. Create NeonDB production database + run initial migration
6. Configure Cloudflare Turnstile site for `kurzum.app`
7. Set up Upstash Redis instance for rate limiting
8. Configure masemIT Analytics project `kurzum-app`
9. Create Vercel project + set environment variables
10. Verify Resend domain (`kurzum.app`) for email sending

### Architecture Completeness Checklist

**✅ Requirements Analysis**
- [x] Project context thoroughly analyzed (43 FRs, 27 NFRs)
- [x] Scale and complexity assessed (Medium-High)
- [x] Technical constraints identified (FFG, solo founder, existing infra)
- [x] Cross-cutting concerns mapped (7 concerns)

**✅ Starter Template**
- [x] Technology domain identified (Full-stack Next.js PWA)
- [x] Starter options researched with current versions (Feb 2026)
- [x] Starter selected with clear rationale
- [x] Initialization commands documented

**✅ Architectural Decisions**
- [x] Critical decisions documented with versions
- [x] Technology stack fully specified
- [x] Integration patterns defined (MMS, Resend, Turnstile, Upstash, masemIT Analytics)
- [x] Performance considerations addressed
- [x] Phase 1 vs Phase 2 clearly separated

**✅ Implementation Patterns**
- [x] Naming conventions established (DB, API, Code)
- [x] Structure patterns defined (co-located tests, layer organization)
- [x] Communication patterns specified (analytics events, logging)
- [x] Process patterns documented (error handling, loading, validation)
- [x] Enforcement guidelines with examples and anti-patterns

**✅ Project Structure**
- [x] Complete directory structure defined
- [x] Component boundaries established
- [x] Integration points mapped (5 external services)
- [x] Requirements to structure mapping complete
- [x] Data flow documented (waitlist + double-opt-in with MMS)
- [x] Database schema defined with indexes
- [x] Environment configuration specified

### Architecture Readiness Assessment

**Overall Status:** READY FOR IMPLEMENTATION

**Confidence Level:** High

**Key Strengths:**
- Clean Phase 1/Phase 2 separation driven by FFG funding constraint
- MMS integration provides centralized GDPR-compliant party/consent management with immutable audit trail
- Modular starter approach — no unnecessary dependencies
- Comprehensive patterns prevent AI agent implementation conflicts
- Every file has a clear purpose, every component a clear boundary

**Areas for Future Enhancement (Phase 2):**
- Auth strategy finalization (Auth.js v5 recommended)
- Real-time update pattern (SSE vs polling)
- PWA Service Worker strategy (Serwist)
- File storage provider for audio/photos
- Automated testing expansion
- Stripe billing integration

### Implementation Handoff

**AI Agent Guidelines:**
- Follow all architectural decisions exactly as documented
- Use implementation patterns consistently across all components
- Respect project structure and component boundaries
- Refer to this document for all architectural questions
- When in doubt: Server Components over Client, Zod for all validation, ApiResponse<T> for all API responses, MMS for party/consent, local DB for business data

**First Implementation Priority:**
```bash
# 1. Create project
npx create-next-app@latest kurzum-app --typescript --tailwind --eslint --app --turbopack --empty --use-pnpm

# 2. Add UI framework
cd kurzum-app && npx shadcn@latest init

# 3. Add core dependencies
pnpm add drizzle-orm @neondatabase/serverless react-hook-form @hookform/resolvers zod
pnpm add -D drizzle-kit

# 4. Add service integrations
pnpm add resend @marsidev/react-turnstile @upstash/ratelimit @upstash/redis
```
