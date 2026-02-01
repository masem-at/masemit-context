# Project Source Tree

**Document Version:** 1.0
**Extracted From:** architecture.md
**Purpose:** Project structure and file organization for tellingCube
**Status:** ⚠️ PARTIALLY OUTDATED - Structure below shows original plan. See actual codebase for current implementation.

> **Note (2025-12-14):** The architecture has evolved significantly since this document was written:
> - Old scenarios (bakery, hotel, tech-startup) replaced with Size × Sector matrix
> - Two-phase generation: World Building → Monthly Course → Events
> - New components: DemoVideo, CompanyDetailsPanel, Sparkline with Notable Events
> - Export APIs: /api/export/[scenarioId]/events.csv, events.json, finance.json, sales.json
> - Contact form, My Scenarios page, What's Next page added

---

## Repository Structure

**Structure:** Monorepo (single Git repository)

**Monorepo Tool:** Native npm/pnpm workspaces (no Nx/Turborepo needed for MVP simplicity)

## High-Level Package Organization

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

## Detailed Directory Structure

### `/app` - Next.js 14 App Router

```
/app
  /api                          # API Routes (serverless functions)
    /generate
      route.ts                  # POST /api/generate - Generate scenarios
    /views
      /finance
        route.ts                # GET /api/views/finance/:scenarioId
      /sales
        route.ts                # GET /api/views/sales/:scenarioId (Post-PoC)
    /validate
      route.ts                  # POST /api/validate/:scenarioId
    /test-bakery-generator
      route.ts                  # Test route for bakery scenario
  /scenario
    /[id]
      page.tsx                  # Dynamic route for scenario viewing
  page.tsx                      # Homepage / Landing page
  layout.tsx                    # Root layout (global providers, fonts)
  globals.css                   # Global styles
```

### `/lib` - Shared Utilities & Business Logic

```
/lib
  /ai                           # AI Generation
    claude-client.ts            # Claude API client wrapper
  /prompts                      # AI Prompt Templates
    bakery-scenario.ts          # 3-stage bakery prompts
    hotel-scenario.ts           # Hotel prompts (Post-PoC)
    tech-startup-scenario.ts    # Tech startup prompts (Post-PoC)
  /generators                   # Scenario Generators
    bakery-generator.ts         # Bakery orchestration
    hotel-generator.ts          # Hotel orchestration (Post-PoC)
  /data                         # Data Access Layer
    event-repository.ts         # Event CRUD operations
    scenario-repository.ts      # Scenario management (Post-PoC)
  /services                     # Business Logic Services
    generation.service.ts       # Generation orchestration
    validation.service.ts       # Event validation (Post-PoC)
  /queries                      # Database Query Functions
    finance-view.ts             # Finance dashboard aggregations
    sales-view.ts               # Sales dashboard aggregations (Post-PoC)
  /utils                        # General Utilities
    format-currency.ts          # Currency formatting
    date-utils.ts               # Date manipulation
  /validators                   # Validation Logic
    event-validator.ts          # Event schema validation
```

### `/components` - React Components

```
/components
  /ui                           # shadcn/ui Components
    button.tsx                  # Button component
    card.tsx                    # Card component
    accordion.tsx               # Accordion component
  /charts                       # Recharts Wrappers
    finance-chart.tsx           # Finance dashboard chart
    sales-chart.tsx             # Sales dashboard chart (Post-PoC)
  /views                        # Page-Level Components
    FinanceView.tsx             # Finance dashboard view
    SalesView.tsx               # Sales dashboard view (Post-PoC)
  /layout                       # Layout Components
    Header.tsx                  # Site header
    Footer.tsx                  # Site footer
```

### `/prisma` - Database Schema & Migrations

```
/prisma
  schema.prisma                 # Database schema (Event, Scenario models)
  /migrations                   # Prisma migration history
    /20240115_init
      migration.sql             # Initial migration
  seed.ts                       # Database seeding script (optional)
```

### `/types` - TypeScript Type Definitions

```
/types
  api.ts                        # API request/response types
  event.ts                      # Event model types
  scenario.ts                   # Scenario model types
  finance.ts                    # Finance view types
  sales.ts                      # Sales view types (Post-PoC)
```

### `/public` - Static Assets

```
/public
  /images
    logo.svg                    # tellingCube logo
    favicon.ico                 # Site favicon
```

## File Naming Conventions

- **React Components**: `PascalCase.tsx` (e.g., `FinanceView.tsx`)
- **Utilities/Services**: `kebab-case.ts` (e.g., `format-currency.ts`)
- **API Routes**: `route.ts` (Next.js 14 convention)
- **Types**: `kebab-case.ts` (e.g., `api.ts`)

## Import Alias

All imports use the `@/` alias to reference the root directory:

```typescript
import { formatCurrency } from '@/lib/utils/format-currency'
import { FinanceView } from '@/components/views/FinanceView'
import type { Event } from '@/types/event'
```

**Configuration:** Set in `tsconfig.json`:
```json
{
  "compilerOptions": {
    "paths": {
      "@/*": ["./*"]
    }
  }
}
```

## Current Implementation Status (After Story 1.4)

### Completed Files

```
/lib
  /ai
    claude-client.ts            # ✅ Claude API wrapper
  /prompts
    bakery-scenario.ts          # ✅ 3-stage bakery prompts
  /generators
    bakery-generator.ts         # ✅ Quarterly batched orchestration
/app
  /api
    /test-bakery-generator
      route.ts                  # ✅ Manual test endpoint
/prisma
  schema.prisma                 # ✅ Event & Scenario models defined
```

### Next Files (Story 1.5+)

```
/lib
  /data
    event-repository.ts         # 📋 Story 1.5 - Event CRUD functions
```
