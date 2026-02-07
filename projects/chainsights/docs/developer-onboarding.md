# ChainSights Developer Onboarding Guide

This guide covers everything you need to clone, configure, run, and contribute to the ChainSights codebase. Read it end-to-end before writing your first line of code.

---

## Table of Contents

1. [Prerequisites](#1-prerequisites)
2. [Clone and Initial Setup](#2-clone-and-initial-setup)
3. [Environment Variables](#3-environment-variables)
4. [Database Setup (Neon + Drizzle)](#4-database-setup-neon--drizzle)
5. [Running the Dev Server](#5-running-the-dev-server)
6. [Project Structure Overview](#6-project-structure-overview)
7. [Key Architectural Patterns](#7-key-architectural-patterns)
8. [Coding Conventions](#8-coding-conventions)
9. [Common Development Tasks](#9-common-development-tasks)
10. [Testing](#10-testing)
11. [Deployment Flow](#11-deployment-flow)
12. [Essential Reference Documents](#12-essential-reference-documents)

---

## 1. Prerequisites

| Tool | Required Version | Notes |
|------|-----------------|-------|
| **Node.js** | v20.x LTS or later | `@types/node` is pinned to `^20.11.0` |
| **npm** | v10+ (ships with Node 20) | Project uses npm, not pnpm/yarn |
| **Git** | Latest stable | |
| **Stripe CLI** | Latest | For testing webhooks locally |
| **Playwright browsers** | Installed via npx | Required for E2E tests |

**Optional but recommended:**

- **VS Code** with the following extensions: ESLint, Tailwind CSS IntelliSense, Prettier
- **Neon CLI** for direct database management
- **Drizzle Studio** (ships with drizzle-kit) for visual DB browsing

---

## 2. Clone and Initial Setup

```bash
# 1. Clone the repository
git clone <repo-url> chainsights
cd chainsights

# 2. Install dependencies
npm install

# 3. Copy the environment variable template
cp .env.example .env.local
# Then fill in the values (see Section 3 below)

# 4. Install Playwright browsers (for E2E tests)
npx playwright install

# 5. Push the Drizzle schema to your Neon database
npm run db:push

# 6. Start the dev server
npm run dev
```

The application runs at **http://localhost:3000** by default.

---

## 3. Environment Variables

Create `.env.local` in the project root. The file `.env.example` contains all variables with documentation comments. Below is a categorized reference.

### Required for Core Functionality

| Variable | Where to Get It | Purpose |
|----------|----------------|---------|
| `DATABASE_URL` | [Neon Console](https://console.neon.tech) | PostgreSQL connection string. Format: `postgresql://user:password@host/database?sslmode=require` |
| `STRIPE_SECRET_KEY` | [Stripe Dashboard > API keys](https://dashboard.stripe.com/apikeys) | Server-side Stripe API access (use `sk_test_...` for dev) |
| `STRIPE_WEBHOOK_SECRET` | Stripe CLI or Dashboard > Webhooks | Webhook signature verification (`whsec_...`) |
| `NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY` | Stripe Dashboard > API keys | Client-side Stripe.js (`pk_test_...`) |

### Required for the Data Pipeline

| Variable | Where to Get It | Purpose |
|----------|----------------|---------|
| `ANTHROPIC_API_KEY` | [Anthropic Console](https://console.anthropic.com) | Powers AI-generated governance analysis in reports |
| `QSTASH_TOKEN` | [Upstash QStash Console](https://console.upstash.com/qstash) | Background job scheduling (weekly recalculation, cron jobs) |
| `QSTASH_CURRENT_SIGNING_KEY` | Same Upstash console | Request signature verification for incoming QStash messages |
| `QSTASH_NEXT_SIGNING_KEY` | Same Upstash console | Key rotation support |

### Required for Email Delivery

| Variable | Where to Get It | Purpose |
|----------|----------------|---------|
| `RESEND_API_KEY` | [Resend Dashboard](https://resend.com/api-keys) | Transactional email (report delivery, magic links, reminders) |
| `FEEDBACK_EMAIL_ENABLED` | Set to `true` or `false` | Toggles email notification on feedback submissions |

### Authentication

| Variable | Where to Get It | Purpose |
|----------|----------------|---------|
| `AUTH_SECRET` | Generate with `openssl rand -hex 32` | Signs JWT tokens for magic-link sessions (`cs_session` cookie) |
| `MATRIX_ADMIN_EMAILS` | Comma-separated list | Emails with admin-level access |

### Optional / Development

| Variable | Default | Purpose |
|----------|---------|---------|
| `NEXT_PUBLIC_BASE_URL` | `http://localhost:3000` | Base URL for links in emails; auto-detected in production |
| `EMAIL_FROM` | `ChainSights <hello@chainsights.one>` | Sender address for outgoing emails |
| `CRON_SECRET` | Generate with `openssl rand -hex 32` | Authenticates scheduled cron job requests |
| `REVALIDATE_SECRET` | Generate with `openssl rand -hex 32` | ISR cache revalidation trigger |
| `SLACK_WEBHOOK_URL` | [Slack Incoming Webhooks](https://api.slack.com/messaging/webhooks) | Alerts for job failures |
| `GVS_DAILY_RETENTION_DAYS` | `30` | Daily GVS snapshot retention before cleanup |
| `GVS_ANOMALY_THRESHOLD` | `0.5` | Score change threshold for anomaly alerts |
| `SHARE_REWARDS_ENABLED` | `true` | Feature flag for the Share & Save cashback program |
| `DGI_PUBLIC` | `false` | Feature flag for public DGI (Decentralization Governance Index) pages |
| `NEXT_PUBLIC_DGI_PUBLIC` | `false` | Client-side mirror of `DGI_PUBLIC` |

### Crypto Payments (Legacy/Optional)

| Variable | Purpose |
|----------|---------|
| `TREASURY_WALLET_ADDRESS` | Ethereum wallet for USDC payments |
| `NEXT_PUBLIC_WALLETCONNECT_PROJECT_ID` | WalletConnect cloud project ID |
| `COINBASE_COMMERCE_API_KEY` | Coinbase Commerce API key |
| `COINBASE_COMMERCE_WEBHOOK_SECRET` | Coinbase Commerce webhook verification |

> **Security reminder:** Never commit `.env.local`. It is already in `.gitignore`.

---

## 4. Database Setup (Neon + Drizzle)

### 4.1 Create a Neon Project

1. Go to [console.neon.tech](https://console.neon.tech) and create a new project.
2. Copy the connection string into `DATABASE_URL` in your `.env.local`.
3. The connection string looks like: `postgresql://user:password@ep-xxxxx.region.aws.neon.tech/neondb?sslmode=require`

### 4.2 Schema and Migrations

The database schema lives in `src/lib/db/schema.ts`. Drizzle ORM generates typed SQL migrations from this file.

**Key npm scripts:**

```bash
# Push schema directly to database (development — no migration files)
npm run db:push

# Generate migration SQL files from schema changes
npm run db:generate

# Apply pending migrations (production)
npm run db:migrate

# Open Drizzle Studio (visual DB browser at https://local.drizzle.studio)
npm run db:studio
```

**Typical development workflow:**

1. Edit `src/lib/db/schema.ts` (add/modify tables or columns).
2. Run `npm run db:push` to apply changes to your Neon dev database.
3. When ready for production, run `npm run db:generate` to create a migration file in `drizzle/`.
4. The migration is applied in production via `npm run db:migrate`.

### 4.3 Database Access in Code

```typescript
import { getDb } from '@/lib/db'
import { eq } from 'drizzle-orm'
import { orders } from '@/lib/db/schema'

const db = getDb()
const result = await db.select().from(orders).where(eq(orders.id, orderId))
```

The `getDb()` helper throws a descriptive error if `DATABASE_URL` is not set, preventing silent failures.

### 4.4 Seeding Test Data

```bash
# Seed leaderboard with sample DAO data
npm run seed:leaderboard

# Seed general test data
npm run seed:test
```

Both scripts load `.env.local` automatically via `tsx --env-file=.env.local`.

---

## 5. Running the Dev Server

```bash
npm run dev
```

This starts the Next.js 14 development server on **http://localhost:3000** with Turbopack (fast refresh).

### Other Useful Commands

| Command | Purpose |
|---------|---------|
| `npm run build` | Production build (also runs `scripts/indexnow.mjs` for search engine notification) |
| `npm run start` | Start the production server locally (requires `npm run build` first) |
| `npm run lint` | Run ESLint checks |
| `npm run build:analyze` | Production build with bundle analyzer (`ANALYZE=true`) |
| `npm run lighthouse` | Run Lighthouse CI for performance auditing |

### Testing Stripe Webhooks Locally

```bash
# In a separate terminal, forward Stripe events to your local server
stripe listen --forward-to localhost:3000/api/webhooks/stripe
```

The CLI prints a webhook signing secret (`whsec_...`). Use it as `STRIPE_WEBHOOK_SECRET` in `.env.local`.

---

## 6. Project Structure Overview

```
chainsights/
├── drizzle/                    # Generated SQL migration files (do not edit manually)
├── docs/                       # Project documentation
│   ├── project_context.md      # SINGLE SOURCE OF TRUTH for decisions, pricing, naming
│   ├── architecture.md         # Full architecture decision document
│   └── ...
├── public/                     # Static assets (images, fonts)
├── scripts/                    # Standalone scripts (rankings, seeding, etc.)
├── tests/                      # All tests
│   ├── unit/                   # Vitest unit tests (mirrors src/ structure)
│   ├── integration/            # Vitest integration tests (real DB)
│   ├── e2e/                    # Playwright end-to-end tests
│   ├── fixtures/               # Shared test fixtures
│   ├── utils/                  # Test utility helpers
│   └── setup.ts                # Global test setup (Vitest)
├── src/
│   ├── app/                    # Next.js App Router (pages, layouts, API routes)
│   │   ├── layout.tsx          # Root layout (fonts, metadata, Schema.org, providers)
│   │   ├── page.tsx            # Landing page
│   │   ├── globals.css         # Tailwind imports + global styles
│   │   ├── api/                # API route handlers (serverless functions)
│   │   │   ├── auth/           # Magic link auth endpoints
│   │   │   ├── checkout/       # Stripe checkout session creation
│   │   │   ├── webhooks/       # Stripe + Coinbase webhook handlers
│   │   │   ├── quick-check/    # Free Check API (open-universe GVS)
│   │   │   ├── jobs/           # Background job endpoints (QStash-triggered)
│   │   │   ├── matrix/         # DAO Matrix data API
│   │   │   ├── dgi/            # Decentralization Governance Index API
│   │   │   └── ...
│   │   ├── matrix/             # DAO Matrix pages (list + detail)
│   │   ├── top-daos/           # Public DAO rankings (formerly /rankings)
│   │   ├── check/              # Free Check flow page
│   │   ├── pricing/            # Pricing page (3 tiers)
│   │   ├── admin/              # Admin dashboard (auth-gated)
│   │   ├── dgi/                # DGI public pages
│   │   └── ...                 # Other pages (methodology, privacy, terms, etc.)
│   ├── components/             # React components
│   │   ├── ui/                 # Base UI primitives (shadcn/ui pattern)
│   │   ├── hero/               # Landing page hero section variants
│   │   ├── auth/               # SignInModal, UserMenu
│   │   ├── providers/          # Context providers (Web3Provider)
│   │   ├── admin/              # Admin-specific components
│   │   ├── matrix-detail/      # Matrix detail page components
│   │   ├── quick-check/        # Free Check result components
│   │   └── ...                 # Feature components (rankings, pricing, etc.)
│   ├── lib/                    # Business logic and utilities
│   │   ├── db/                 # Drizzle ORM setup, schema, queries
│   │   │   ├── index.ts        # DB connection (Neon serverless)
│   │   │   ├── schema.ts       # Table definitions (pgTable, pgEnum)
│   │   │   └── queries/        # Reusable query functions
│   │   ├── ai/                 # Anthropic SDK integration (report generation)
│   │   ├── gvs/                # GVS calculation engine
│   │   ├── snapshot/            # Snapshot.org API client
│   │   ├── stripe.ts           # Stripe configuration and helpers
│   │   ├── email/              # Resend email templates and sending
│   │   ├── pdf/                # PDF report generation (@react-pdf/renderer)
│   │   ├── auth/               # Magic link auth utilities (JWT, session)
│   │   ├── validation/         # Zod schemas + security headers
│   │   ├── analytics.ts        # Analytics event tracking
│   │   ├── hooks/              # Custom React hooks
│   │   ├── middleware/         # Rate limiting logic
│   │   ├── crypto/             # wagmi config, wallet utilities
│   │   ├── dgi/                # DGI calculation and data
│   │   ├── jobs/               # Background job logic
│   │   └── ...
│   ├── middleware.ts           # Edge middleware (auth, rate limiting, security headers, vanity URLs)
│   └── types/                  # Global TypeScript type declarations
├── .env.example                # Full environment variable template with comments
├── .env.local.example          # Minimal environment variable template
├── drizzle.config.ts           # Drizzle Kit configuration
├── next.config.js              # Next.js configuration (Sentry, redirects, caching)
├── tailwind.config.ts          # Tailwind CSS theme (navy, aqua colors)
├── tsconfig.json               # TypeScript config (strict mode, @/* alias)
├── vitest.config.ts            # Vitest configuration
├── playwright.config.ts        # Playwright E2E configuration
└── package.json                # Dependencies and scripts
```

---

## 7. Key Architectural Patterns

### 7.1 Server Components by Default

Every component is a **React Server Component** unless it explicitly declares `'use client'` at the top. Add the client directive only when you need:

- Event handlers (`onClick`, `onChange`)
- React hooks (`useState`, `useEffect`)
- Browser APIs (`localStorage`, `window`)

Data fetching happens in Server Components; results are passed to Client Components as props.

### 7.2 Drizzle ORM (Type-Safe Database Access)

- Schema defined in `src/lib/db/schema.ts` using `pgTable` and `pgEnum`.
- All identifiers use **snake_case** (tables, columns, enums).
- Queries use the Drizzle query builder -- never raw SQL strings.
- Import the database via `getDb()` from `@/lib/db`.
- Transactions for multi-step operations: `db.transaction(async (tx) => { ... })`.

### 7.3 Neon Serverless Postgres

The database connection uses `@neondatabase/serverless` with HTTP-based queries (not WebSocket). This is optimized for serverless environments where connections are short-lived.

### 7.4 Authentication (Magic Link)

- Passwordless auth via magic links sent to email.
- Sessions stored as JWT in `cs_session` cookie (30-day expiry).
- JWT signed/verified with `jose` (Edge-compatible, no Node.js crypto dependency).
- Middleware (`src/middleware.ts`) resolves sessions on every request and injects `x-user-email` and `x-user-role` headers.
- Roles: `admin`, `free`, `subscriber`.

### 7.5 Multi-Layer Caching

| Layer | Mechanism | TTL |
|-------|-----------|-----|
| ISR | `export const revalidate = 3600` on pages | 1 hour |
| API Cache | Response caching in API routes | 15 minutes |
| CDN | Vercel Edge Network for static assets | 1 year (immutable) |

### 7.6 External API Resilience

- All external API calls implement retry logic (3 attempts with exponential backoff).
- Circuit breaker pattern: after 5 consecutive failures, skip calls for 5 minutes.
- Graceful degradation: show cached data with staleness indicators when APIs fail.

### 7.7 Edge Middleware

`src/middleware.ts` runs on every request and handles:

1. **Session resolution** from `cs_session` JWT cookie.
2. **Admin route protection** (redirects non-admins to login).
3. **Rate limiting** on API routes (100 req/min/IP).
4. **Security headers** (CSP, HSTS, etc.).
5. **Vanity URL redirects** (`/ens` redirects to `/matrix/ens`).
6. **CORS** (same-origin only).

### 7.8 Payments (Stripe)

- Stripe Hosted Checkout for paid tiers (Deep Dive at EUR 49, Governance Audit at EUR 149).
- Webhook handler at `src/app/api/webhooks/stripe/route.ts` for payment confirmation.
- Always verify webhook signatures with `stripe.webhooks.constructEvent()`.
- Never trust client-side payment confirmation.

---

## 8. Coding Conventions

These rules come from `docs/project_context.md`. Read that file for the complete set.

### TypeScript

- **Strict mode is enabled.** No `any` types -- use `unknown` with type guards.
- All functions in `src/lib/` must have **explicit return types**.
- API route handlers must use `NextRequest`/`NextResponse` types.
- Use `@/*` import alias for all `src/` imports.

### Import Order

1. React / Next.js imports
2. External packages
3. `@/*` internal imports
4. Type imports

### File Naming

| Kind | Convention | Example |
|------|-----------|---------|
| Components | PascalCase | `Header.tsx`, `IntakeForm.tsx` |
| Utilities | kebab-case | `generate-rankings.ts` |
| API routes | kebab-case folders | `api/opt-out-requests/route.ts` |
| Test files | Match source name | `Header.test.tsx` |

### Component Rules

- One component per file, max 300 lines.
- Named exports from `lib/` modules; default exports only for React components.
- No barrel exports (`index.ts` re-exports) in `lib/` -- import from specific files.
- Extract logic to `lib/` if used by 2 or more components.

### Database Naming

All Drizzle schema identifiers use **snake_case**: `dao_scores`, `created_at`, `stripe_session_id`.

### Error Handling

- `try-catch` in all API routes with proper error responses.
- Never expose database errors to clients -- return generic messages.
- Never throw errors in Server Components -- return error states instead.

### Validation

- Validate all request bodies with **Zod** schemas (defined in `src/lib/validation/`).
- Never use `req.body` directly -- always `await request.json()` first.

### Comments

- JSDoc comments on exported functions in `lib/`.
- Inline comments for complex business logic only.
- TODO comments with ticket numbers: `// TODO(CHAIN-123): Implement retry logic`.

---

## 9. Common Development Tasks

### 9.1 Adding a New Page

1. Create a folder under `src/app/` matching the URL path (e.g., `src/app/reports/page.tsx`).
2. Use a Server Component by default (async function, no `'use client'`).
3. Export metadata for SEO:

```typescript
import type { Metadata } from 'next'

export const metadata: Metadata = {
  title: 'Page Title | ChainSights',
  description: 'Page description for search engines.',
}

export default async function ReportsPage() {
  // Fetch data directly in the component
  const data = await fetchSomeData()
  return <div>{/* render */}</div>
}
```

4. If the page needs interactivity, create a separate Client Component in `src/components/` and import it.

### 9.2 Adding an API Route

1. Create a folder under `src/app/api/` (e.g., `src/app/api/my-endpoint/route.ts`).
2. Export named handler functions (`GET`, `POST`, etc.):

```typescript
import { NextRequest, NextResponse } from 'next/server'
import { z } from 'zod'
import { getDb } from '@/lib/db'

const requestSchema = z.object({
  name: z.string().min(1),
})

export async function POST(request: NextRequest): Promise<NextResponse> {
  try {
    const body = await request.json()
    const parsed = requestSchema.parse(body)
    const db = getDb()
    // ... do work
    return NextResponse.json({ success: true })
  } catch (error) {
    if (error instanceof z.ZodError) {
      return NextResponse.json({ error: 'Invalid input' }, { status: 400 })
    }
    return NextResponse.json({ error: 'Internal server error' }, { status: 500 })
  }
}
```

### 9.3 Adding a Database Table or Column

1. Edit `src/lib/db/schema.ts` -- add or modify the table definition using `pgTable`.
2. Run `npm run db:push` to apply changes to your dev database.
3. When done, generate a migration: `npm run db:generate`.
4. Commit the generated migration file in `drizzle/`.
5. Never manually edit migration files -- regenerate if needed.

### 9.4 Adding a Reusable Query

1. Create a file in `src/lib/db/queries/` (e.g., `reports.ts`).
2. Export typed query functions:

```typescript
import { getDb } from '@/lib/db'
import { eq, desc } from 'drizzle-orm'
import { reports } from '@/lib/db/schema'

export async function getReportsByDao(daoId: string) {
  const db = getDb()
  return db.select().from(reports).where(eq(reports.dao_id, daoId)).orderBy(desc(reports.created_at))
}
```

### 9.5 Adding a New Component

1. Create a `.tsx` file in `src/components/` (PascalCase name).
2. If it is a base UI primitive, place it in `src/components/ui/`.
3. If it is feature-specific, consider a subfolder (e.g., `src/components/matrix-detail/`).
4. Export with a named export for `lib/` utilities, default export for React components.

### 9.6 Working with Stripe

- Stripe configuration: `src/lib/stripe.ts`.
- Checkout session creation: `src/app/api/checkout/route.ts`.
- Webhook processing: `src/app/api/webhooks/stripe/route.ts`.
- Test locally with the Stripe CLI: `stripe listen --forward-to localhost:3000/api/webhooks/stripe`.

---

## 10. Testing

### 10.1 Test Structure

```
tests/
├── unit/           # Fast, isolated tests (mocked dependencies)
├── integration/    # Tests with real database
├── e2e/            # Browser-based tests (Playwright)
├── fixtures/       # Shared test data
├── utils/          # Test helpers
└── setup.ts        # Global Vitest setup (@testing-library/jest-dom)
```

### 10.2 Running Tests

| Command | What It Does |
|---------|-------------|
| `npm test` | Unit tests in watch mode (Vitest) |
| `npm run test:unit` | Unit tests, single run |
| `npm run test:integration` | Integration tests (requires `DATABASE_URL`) |
| `npm run test:coverage` | Unit tests with V8 coverage report |
| `npm run test:e2e` | E2E tests in headless Chromium (Playwright) |
| `npm run test:e2e:ui` | E2E tests with interactive Playwright UI |

### 10.3 Writing Unit Tests

Unit tests live in `tests/unit/` and mirror the `src/` structure. Use Vitest with mocked dependencies:

```typescript
import { describe, it, expect, vi } from 'vitest'
import { calculateGVS } from '@/lib/gvs/calculate'

vi.mock('@/lib/snapshot', () => ({
  fetchProposals: vi.fn().mockResolvedValue([]),
}))

describe('calculateGVS', () => {
  it('returns 0 for empty proposal history', async () => {
    const result = await calculateGVS('test-dao')
    expect(result.score).toBe(0)
  })
})
```

### 10.4 Writing Integration Tests

Integration tests use a real database. Do not mock Drizzle ORM:

```typescript
import { describe, it, expect, beforeEach } from 'vitest'
import { getDb } from '@/lib/db'

describe('orders query', () => {
  beforeEach(async () => {
    const db = getDb()
    // Clean up test data
  })

  it('creates and retrieves an order', async () => {
    // Test with real DB operations
  })
})
```

### 10.5 Writing E2E Tests

E2E tests live in `tests/e2e/` and use Playwright. The dev server starts automatically:

```typescript
import { test, expect } from '@playwright/test'

test('homepage loads and displays hero', async ({ page }) => {
  await page.goto('/')
  await expect(page.getByRole('heading', { level: 1 })).toBeVisible()
})
```

Accessibility testing is available via `@axe-core/playwright`:

```typescript
import AxeBuilder from '@axe-core/playwright'

test('homepage passes accessibility checks', async ({ page }) => {
  await page.goto('/')
  const results = await new AxeBuilder({ page }).analyze()
  expect(results.violations).toEqual([])
})
```

### 10.6 Coverage Target

- 80% minimum coverage for `src/lib/` code.
- Coverage reports are generated in text, JSON, and HTML formats.
- Page components (`src/app/**/page.tsx`) are excluded from coverage.

---

## 11. Deployment Flow

### 11.1 Vercel Git Integration

ChainSights deploys to **Vercel** via Git integration:

- **Every pull request** gets an automatic preview deployment with a unique URL.
- **Merging to `master`** triggers a production deployment to [chainsights.one](https://chainsights.one).
- Do not use the Vercel CLI for production deploys -- always go through Git.

### 11.2 Pre-Merge Checklist

Before merging a PR:

1. Verify the TypeScript build passes: `npm run build`
2. Run linting: `npm run lint`
3. Run unit tests: `npm run test:unit`
4. Run E2E tests: `npm run test:e2e`
5. Review the Vercel preview deployment for visual correctness.

### 11.3 Environment Variables in Production

Production environment variables are set in the **Vercel Dashboard** (Settings > Environment Variables). Never commit secrets to the repository.

### 11.4 Database Migrations in Production

1. Generate migration files locally: `npm run db:generate`
2. Commit the migration files in the `drizzle/` directory.
3. Migrations are applied via `npm run db:migrate` during the build process or manually before deploy.

### 11.5 Monitoring

| Service | Purpose |
|---------|---------|
| **Sentry** | Error tracking with source maps (configured in `next.config.js`) |
| **Vercel Analytics** | Core Web Vitals (LCP, FCP, TTI) |
| **masemIT Analytics** | Self-hosted traffic tracking (`analytics.masem.at`) |
| **UptimeRobot** | 5-minute uptime pings |
| **Slack Webhooks** | Alerts for job failures and anomalies |

### 11.6 Git Workflow

- **Branch naming:** `feature/dao-rankings`, `fix/stripe-webhook`, `refactor/gvs-engine`
- **Commit messages:** Conventional Commits format
  - `feat: add weekly GVS calculation job`
  - `fix: resolve Stripe webhook signature verification`
  - `refactor: extract wallet clustering to separate service`

---

## 12. Essential Reference Documents

| Document | Path | What It Contains |
|----------|------|-----------------|
| **Project Context** | `docs/project_context.md` | Single source of truth: pricing, naming, feature flags, decision log, all coding rules |
| **Architecture** | `docs/architecture.md` | Full architecture decision document with component diagrams and patterns |
| **PRD** | `docs/prd.md` | Product requirements document (93 functional requirements) |
| **UX Design** | `docs/ux-design-specification.md` | UX design specification |
| **Phase 2 Design** | `docs/phase-2-design.md` | Governance Audit and DAO Matrix design decisions |
| **MMS Analytics Integration** | `docs/integrations/mms-analytics-api.md` | Self-hosted analytics API documentation |

> **Critical reminder:** Always read `docs/project_context.md` at the start of every working session. It contains pricing, naming, and feature flag decisions that override any assumptions.

---

## Quick Reference Card

```bash
# Daily development
npm run dev                     # Start dev server (port 3000)
npm test                        # Run unit tests (watch mode)
npm run db:studio               # Browse database visually

# Database changes
npm run db:push                 # Apply schema to dev DB
npm run db:generate             # Generate migration files

# Before committing
npm run lint                    # ESLint check
npm run test:unit               # Unit tests
npm run build                   # Full production build

# Stripe webhook testing
stripe listen --forward-to localhost:3000/api/webhooks/stripe

# Seeding data
npm run seed:leaderboard        # Populate leaderboard
npm run seed:test               # Populate test data
```
