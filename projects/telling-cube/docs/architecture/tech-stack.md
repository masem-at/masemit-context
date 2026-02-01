# Tech Stack

**Document Version:** 1.0
**Extracted From:** architecture.md
**Purpose:** Definitive technology selection for tellingCube

---

This is the **DEFINITIVE technology selection** for tellingCube. All development must use these exact versions and tools. This table serves as the single source of truth.

## Tech Stack Table

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

## PoC Tech Stack (Week 1-2)

| Category | Technology | Notes |
|----------|-----------|-------|
| Frontend | Next.js 14 + TypeScript + Tailwind + Recharts | Lovable components exported as TSX |
| Backend | Next.js API Routes + Prisma | `/api/generate`, `/api/views/finance`, `/api/validate` |
| Database | NeonDB Launch | Single `events` table |
| AI | Claude 3.5 Sonnet API | Bakery scenario only |
| Deployment | Vercel Pro | Push to deploy |

**Deferred to Post-PoC:** Upstash, Stripe, Resend, Sentry, Testing frameworks

## Key Technology Decisions

**Why Next.js 14 App Router?** Server Components reduce JS bundle; co-located API routes; Vercel-optimized

**Why Prisma?** Best TypeScript support; auto-generated types; built-in migrations; NeonDB compatible

**Why Claude 3.5 Sonnet?** $0.165/scenario vs GPT-4's $0.45 (3x cheaper); 200K context; excellent JSON output

**Why Recharts?** Already in Lovable; React-native API; SVG-based; IBCS-friendly grayscale styling

**Why shadcn/ui?** Already in Lovable; copy/paste (no dependency); Radix UI (accessible); Tailwind-based

**Why Vercel?** Next.js optimization; zero DevOps; git-based deployments; $20/mo includes CDN/SSL

**Why NeonDB?** Serverless PostgreSQL; scales to zero; free tier for PoC; database branching for dev/staging
