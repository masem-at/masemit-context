---
project_name: 'chainsights'
user_name: 'Mario'
date: '2026-01-28'
sections_completed: ['technology_stack', 'language_rules', 'framework_rules', 'testing_rules', 'code_quality', 'workflow_rules', 'critical_rules', 'pricing', 'naming', 'feature_flags', 'seo_aeo_llmo']
status: 'complete'
rule_count: 120
optimized_for_llm: true
---

# Project Context for AI Agents

_This file contains critical rules and patterns that AI agents must follow when implementing code in this project. Focus on unobvious details that agents might otherwise miss._

> **CRITICAL:** This document is the single source of truth. Load this file at the start of every session before making any changes.

---

## Official Pricing (READ FIRST)

| Tier | Internal ID | User-Facing Name | Price | Stripe Price ID Env |
|------|-------------|------------------|-------|---------------------|
| Free | `quick_check` | **Free Check** | €0 (email required) | N/A |
| Mid | `deep_dive` | **Deep Dive** | **€49** | `STRIPE_PRICE_DEEP_DIVE` |
| Premium | `governance_audit` | **Governance Audit** | **€149** | `STRIPE_PRICE_GOVERNANCE_AUDIT` |
| Legacy | `standard` | (deprecated) | €99 | `STRIPE_PRICE_STANDARD` |

### Pricing Rules
- ❌ **NEVER** show €99 or €199 to users (legacy/deprecated)
- ✅ Starting price displayed: "from €49"
- ✅ Deep Dive badge: "Most Popular"
- ✅ Governance Audit badge: "Complete Analysis"

---

## Official Naming (READ FIRST)

| Internal Code | User-Facing Name | CTA Text |
|---------------|------------------|----------|
| `quick_check` | **Free Check** | "Get Free Check" |
| `deep_dive` | **Deep Dive** | "Get Deep Dive – €49" |
| `governance_audit` | **Governance Audit** | "Get Governance Audit – €149" |

### Naming Rules
- ✅ Internal code (e.g., `quick_check`) stays in code/APIs
- ✅ User-facing text MUST use the official names above
- ❌ **NEVER** use: "Quick Check" (user-facing), "Order a report", "Get your report", "Standard Report"
- ✅ CTAs should include price (except Free Check)

---

## Feature Flags

| Feature | Status | Notes |
|---------|--------|-------|
| Free Check | **ENABLED** | Live, working |
| Deep Dive (€49) | **ENABLED** | Live, working |
| Governance Audit (€149) | **ENABLED** | Live, working |
| Share & Save (€20 cashback) | **DISABLED** | Hide everywhere, do not delete code |
| DAO Matrix | **NOT BUILT** | Phase 2, deferred |
| API Access | **NOT BUILT** | Future |

### Share & Save Hiding Checklist
When `SHARE_REWARDS_ENABLED = false`, hide in:
- `src/app/success/page.tsx`
- `src/lib/email/index.ts` (report delivery + share reminder emails)
- `src/lib/pdf/report-template.tsx` (PDF page 4)
- `src/app/api/jobs/share-reminder/route.ts` (skip sending)
- `src/app/share/page.tsx` (redirect or hide)
- `src/app/layout.tsx` (FAQ text)
- `src/app/llms.txt/route.ts`

---

## SEO / AEO / LLMO Content

These files contain machine-readable content that MUST stay in sync with pricing and naming:

| File | Purpose | Content to Sync |
|------|---------|-----------------|
| `src/app/layout.tsx` | Schema.org structured data (SEO/AEO) | Product offers, prices, FAQ answers |
| `src/app/llms.txt/route.ts` | LLM-readable summary (LLMO) | Pricing, features, Share & Save |

### Schema.org Offers (layout.tsx)
Must reflect current tiers:
```json
{ "@type": "Offer", "name": "Deep Dive", "price": "49", "priceCurrency": "EUR" },
{ "@type": "Offer", "name": "Governance Audit", "price": "149", "priceCurrency": "EUR" }
```

### FAQ Schema & llms.txt Rules
- ✅ Use "Free Check" (not "Quick Check")
- ✅ Use "€49 Deep Dive" (not €99)
- ✅ Use "€149 Governance Audit" (not €199)
- ❌ NO mention of Share & Save cashback (while disabled)

---

## Known Bugs & Issues (2026-01-28)

| Bug | Status | Root Cause | Fix |
|-----|--------|------------|-----|
| Donations not recording | **FIXED** | `payment_intent.succeeded` webhook was not enabled | Enabled in Stripe Dashboard |
| Quick Checks not recording | **FIXED** | Was recording correctly, just in `quick_checks` table | Verified working |

### Content Inconsistencies

✅ **All fixed as of 2026-01-28** - pricing, naming, and CTAs are now consistent across all files.

---

## Decision Log

| Date | Decision | Rationale |
|------|----------|-----------|
| 2026-01-28 | €99 → €49 for Deep Dive | Lower friction, impulse buy territory |
| 2026-01-28 | "Quick Check" → "Free Check" (user-facing) | Clearer value proposition |
| 2026-01-28 | Hide Share & Save temporarily | Validate purchases first, reactivate later |
| 2026-01-28 | Quiz CTA → Report Modal | Don't send engaged users back to landing page |
| 2026-01-28 | Matrix deferred to Phase 2 | Validate report demand first |
| 2026-01-28 | Unify DAO data architecture | Paid reports auto-populate rankings + gvsSnapshots for Quick Check consistency |
| 2026-01-28 | Stripe → Analytics integration | Send `checkout_complete` events to analytics.masem.at for conversion funnel |
| 2026-01-28 | Fix metric naming | DEI=Decentralization Index, PDI=Proposal Deliberation Index, GPI=Governance Process Index |
| 2026-01-28 | Mandatory project_context.md | All 13 agents now load this file as Step 3 before any work |

---

## Technology Stack & Versions

**Core Framework:**
- Next.js 14.2.0 (App Router) - Use Server Components by default
- React 18.3.0 (strict mode enabled)
- TypeScript 5.3.3 (**strict mode** - no implicit any allowed)

**Database:**
- Drizzle ORM 0.45.1 + drizzle-kit 0.31.8
- PostgreSQL via @neondatabase/serverless 1.0.2
- **All database identifiers MUST use snake_case** (tables, columns, enums)

**Styling:**
- Tailwind CSS 3.4.1 with custom theme (navy, aqua colors)
- Radix UI primitives (@radix-ui/*)
- shadcn/ui pattern (ui/ components)

**Web3:**
- wagmi 3.1.0 + viem 2.43.2
- WalletConnect Ethereum Provider 2.21.10
- Coinbase Wallet SDK 4.3.7

**External Services:**
- Stripe 14.14.0 (payments)
- Anthropic SDK 0.71.2 (AI analysis)
- Resend 6.6.0 (email)
- QStash 2.8.4 (job scheduling)
- Vercel Analytics 1.6.1

**Testing:**
- Vitest 4.0.16 (unit + integration)
- Playwright 1.57.0 (E2E)

---

## Critical Implementation Rules

### Language-Specific Rules (TypeScript)

**TypeScript Configuration:**
- ✅ **Strict mode is ENABLED** - all code must pass strict type checking
- ✅ Use `@/*` import alias for all src/ imports (configured in tsconfig paths)
- ❌ **NEVER use `any` type** - use `unknown` and type guards if needed
- ✅ All functions in src/lib/ MUST have explicit return types
- ✅ API route handlers MUST use NextRequest/NextResponse types

**Import/Export Patterns:**
- ✅ Use named exports from lib/ modules, default export only for React components
- ✅ Import order: React/Next → external packages → @/* internal → types
- ❌ **DO NOT use barrel exports (index.ts re-exports)** in lib/ - import directly from specific files

**Error Handling:**
- ✅ Use try-catch in all API routes with proper error responses
- ✅ External API calls MUST implement retry logic (3 attempts with exponential backoff)
- ✅ Database errors MUST be caught and return generic error messages (never expose DB structure)
- ❌ **NEVER throw errors in Server Components** - return error states instead

---

### Framework-Specific Rules (Next.js 14 + React)

**Server vs Client Components:**
- ✅ **USE SERVER COMPONENTS BY DEFAULT** - no 'use client' directive
- ✅ Add `'use client'` ONLY when you need:
  - Event handlers (onClick, onChange, etc.)
  - React hooks (useState, useEffect, etc.)
  - Browser APIs (localStorage, window, document)
- ❌ **NEVER fetch data in Client Components** - do it in Server Components and pass as props
- ✅ Server Components can import and use Client Components (but not vice versa)

**Data Fetching Patterns:**
- ✅ Use async Server Components for data fetching (no useEffect needed)
- ✅ Use Drizzle ORM for ALL database queries (never raw SQL strings)
- ✅ Database queries in src/lib/db/ must use prepared statements
- ✅ ISR caching: Use `export const revalidate = 3600` for 1-hour cache

**API Routes (App Router):**
- ✅ API routes in src/app/api/ MUST export route handlers: GET, POST, PUT, DELETE
- ✅ Use `NextRequest` for request, return `NextResponse.json()` for responses
- ✅ Validate ALL request bodies with Zod schemas (defined in src/lib/validation/)
- ❌ **DO NOT use req.body directly** - await request.json() first

**React Hooks Rules:**
- ✅ Custom hooks in src/lib/hooks/ - prefix with `use`
- ✅ useEffect ONLY in Client Components - prefer Server Components for data
- ❌ **NEVER call hooks conditionally** - must be at top level
- ✅ Use React Query (@tanstack/react-query) for client-side data fetching if needed

**Performance Rules:**
- ✅ Use Next.js Image component for all images (automatic optimization)
- ✅ Dynamic imports for heavy components: `const Heavy = dynamic(() => import('@/components/Heavy'))`
- ✅ Streaming with Suspense boundaries for slow data fetching
- ❌ **DO NOT fetch in loops** - batch with Promise.all()

---

### Database Rules (Drizzle ORM)

**Schema Definitions (src/lib/db/schema.ts):**
- ✅ Use `pgTable` for table definitions
- ✅ Use `pgEnum` for enum types (export for reuse in code)
- ✅ **ALL identifiers snake_case**: `dao_scores`, `created_at`, `stripe_session_id`
- ✅ Primary keys: `uuid('id').defaultRandom().primaryKey()`
- ✅ Timestamps: `timestamp('created_at').defaultNow().notNull()`
- ✅ Add inline comments for complex fields: `// Stripe payment intent ID`

**Query Patterns:**
- ✅ Import db from `@/lib/db`
- ✅ Use Drizzle query builder: `db.select().from(orders).where(eq(orders.id, id))`
- ✅ Use transactions for multi-step operations: `db.transaction(async (tx) => { ... })`
- ❌ **NEVER use raw SQL** - use Drizzle query builder for type safety
- ✅ Use Drizzle operators: `eq()`, `and()`, `or()`, `desc()`, `asc()`

**Migrations:**
- ✅ Generate migrations: `npm run db:generate` after schema changes
- ✅ Push to DB: `npm run db:push` (dev) or `npm run db:migrate` (prod)
- ❌ **DO NOT manually edit migration files** - regenerate if needed

---

### Testing Rules

**Test Organization:**
- ✅ Unit tests: `tests/unit/` - mirror src/ structure
- ✅ Integration tests: `tests/integration/` - test API routes, DB interactions
- ✅ E2E tests: `tests/e2e/` - test full user flows with Playwright
- ✅ Test files: `{filename}.test.ts` or `{filename}.spec.ts`

**Unit Testing (Vitest):**
- ✅ Import from `vitest`: `import { describe, it, expect, vi } from 'vitest'`
- ✅ Test lib/ functions in isolation with mocked dependencies
- ✅ Mock external services: `vi.mock('@/lib/snapshot')`
- ✅ Coverage requirement: 80% minimum for lib/ code

**Integration Testing:**
- ✅ Test API routes with real DB (use test database)
- ✅ Test Drizzle ORM queries with actual PostgreSQL
- ✅ Use beforeEach/afterEach for DB cleanup
- ❌ **DO NOT mock Drizzle ORM** in integration tests - use real DB

**E2E Testing (Playwright):**
- ✅ Test full user journeys: order flow, opt-out form, admin dashboard
- ✅ Use Playwright fixtures for auth, DB setup
- ✅ Test accessibility with @axe-core/playwright
- ✅ Run E2E in CI before deploy: `npm run test:e2e`

**Running Tests:**
```bash
npm test                  # Unit tests (watch mode)
npm run test:unit         # Unit tests (single run)
npm run test:integration  # Integration tests
npm run test:coverage     # Coverage report
npm run test:e2e          # E2E tests
```

---

### Code Quality & Style Rules

**File Naming:**
- ✅ Components: PascalCase `Header.tsx`, `IntakeForm.tsx`
- ✅ Utilities: kebab-case `generate-rankings.ts`, `verify-transaction.ts`
- ✅ API routes: kebab-case folders `api/opt-out-requests/route.ts`
- ✅ Test files: match source name `Header.test.tsx`

**Component Naming:**
- ✅ React components: PascalCase export `export function Header() { ... }`
- ✅ Custom hooks: camelCase with `use` prefix `export function useDAO() { ... }`
- ✅ Utility functions: camelCase `export function calculateGVS() { ... }`

**Code Organization:**
- ✅ Keep files focused: 1 component per file, max 300 lines
- ✅ Extract logic to lib/ if used by 2+ components
- ✅ Group related utilities in folders: `lib/snapshot/`, `lib/ai/`, `lib/validation/`
- ❌ **DO NOT create utils.ts dumping grounds** - organize by domain

**Comments & Documentation:**
- ✅ Add JSDoc comments to exported functions in lib/
- ✅ Inline comments for complex business logic (GVS calculation, wallet clustering)
- ❌ **DO NOT comment obvious code** - code should be self-documenting
- ✅ Add TODO comments with ticket numbers: `// TODO(CHAIN-123): Implement retry logic`

---

### Development Workflow Rules

**Git Workflow:**
- ✅ Branch naming: `feature/dao-rankings`, `fix/stripe-webhook`, `refactor/gvs-engine`
- ✅ Commit messages: Conventional Commits format
  - `feat: add weekly GVS calculation job`
  - `fix: resolve Stripe webhook signature verification`
  - `refactor: extract wallet clustering to separate service`
- ✅ Run tests before commit: `npm test && npm run test:e2e`

**Environment Variables:**
- ✅ Required vars in `.env.example` with dummy values
- ✅ Load with `process.env.VARIABLE_NAME` in server code
- ❌ **NEVER commit .env file** - use .env.local for local dev
- ✅ Vercel deployment: Set env vars in Vercel dashboard

**Scripts:**
- ✅ Database: `npm run db:push` (dev), `npm run db:generate` (migrations)
- ✅ Dev server: `npm run dev` (port 3000)
- ✅ Build: `npm run build` (checks TypeScript, builds Next.js)
- ✅ Custom scripts: Run with `tsx scripts/{script-name}.ts`

**Deployment (Vercel):**
- ✅ Every PR gets preview deployment automatically
- ✅ Merge to master deploys to production
- ✅ Check build logs in Vercel dashboard
- ❌ **DO NOT use Vercel CLI for production deploys** - use Git integration

---

### Critical Don't-Miss Rules

**Security:**
- ❌ **NEVER log sensitive data** (stripe keys, user emails, tokens)
- ❌ **NEVER expose DB errors to client** - return generic "Internal server error"
- ✅ Validate ALL user inputs with Zod before DB operations
- ✅ Rate limit API routes: 100 requests/minute per IP
- ✅ Use HTTPS only (enforced by Vercel)
- ✅ Stripe webhook signature verification REQUIRED: `stripe.webhooks.constructEvent()`

**Next.js 14 Gotchas:**
- ❌ **DO NOT use useEffect for data fetching** - use async Server Components
- ❌ **DO NOT use getServerSideProps or getStaticProps** - deprecated in App Router
- ✅ Use `export const dynamic = 'force-dynamic'` for no-cache routes
- ✅ Use `export const revalidate = 3600` for ISR caching
- ❌ **NEVER import server-only code in Client Components** - will fail build

**Drizzle ORM Gotchas:**
- ❌ **DO NOT use .get() on queries** - use .execute() and check array length
- ✅ Use `eq()` for equality, not `===`: `where(eq(orders.id, orderId))`
- ✅ NULL checks: `isNull()` and `isNotNull()` operators
- ❌ **DO NOT mutate query objects** - queries are immutable, chain methods

**Stripe Integration:**
- ✅ Use Stripe SDK, never raw API calls
- ✅ Webhook endpoint: `api/webhooks/stripe/route.ts`
- ✅ Test webhooks with Stripe CLI: `stripe listen --forward-to localhost:3000/api/webhooks/stripe`
- ❌ **NEVER trust client-side payment confirmation** - verify in webhook

**Web3 Integration (wagmi + viem):**
- ✅ Wrap app with `<WagmiProvider>` in providers/web3-provider.tsx
- ✅ Use wagmi hooks only in Client Components: `useAccount()`, `useConnect()`, `useSignMessage()`
- ✅ Chain config in `lib/crypto/wagmi-config.ts`
- ❌ **DO NOT use window.ethereum directly** - use wagmi abstractions

**Performance Anti-Patterns:**
- ❌ **NEVER fetch in loops** - batch with `Promise.all([...])` or `Promise.allSettled([...])`
- ❌ **NEVER use Client Components for data-heavy pages** - use Server Components
- ❌ **DO NOT load large images without Next/Image** - always use `next/image`
- ✅ Use dynamic imports for heavy dependencies: `dynamic(() => import(...))`

**Common Edge Cases:**
- ✅ Handle Snapshot API rate limits (429 status) with retry + backoff
- ✅ Handle DAO not found in Snapshot (return 404, not 500)
- ✅ Handle empty proposal history (calculate GVS = 0 with low confidence)
- ✅ Handle Stripe webhook retries (idempotency - check if order already processed)
- ✅ Handle crypto transaction confirmations (wait for N blocks before marking paid)

---

## Architecture Reference

**Full architecture documented in:** `docs/architecture.md`

**Key architectural patterns:**
- Server Components by default (Next.js 14 App Router)
- Drizzle ORM with snake_case schema
- Multi-layer caching (ISR 1-hour, API cache 15-min)
- Retry + circuit breaker for external APIs
- Type safety end-to-end (TypeScript strict + Drizzle + Zod)

**When implementing features, always:**
1. Read the architecture doc first to understand decisions
2. Follow the project structure defined in architecture
3. Use the patterns documented in architecture
4. Maintain consistency with existing code


---

## Usage Guidelines

**For AI Agents:**

- Read this file before implementing any code
- Follow ALL rules exactly as documented
- When in doubt, prefer the more restrictive option
- Refer to docs/architecture.md for architectural context
- Update this file if new patterns emerge during implementation

**For Humans:**

- Keep this file lean and focused on agent needs
- Update when technology stack changes
- Review quarterly for outdated rules
- Remove rules that become obvious over time
- Add new edge cases as they're discovered

**Maintenance:**

- Last Updated: 2026-01-28
- Review frequency: Quarterly or after major tech stack changes
- Target: <500 lines, focused on non-obvious critical rules

