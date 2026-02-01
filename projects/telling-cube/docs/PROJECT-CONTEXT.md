# tellingCube Project Context

**Version:** 2.0
**Last Updated:** 2026-01-21
**Author:** Paige (Technical Writer)
**Status:** ACTIVE - All agents must follow these guidelines

---

## CRITICAL: Repositioning Notice

**Effective January 21, 2026:** tellingCube has been repositioned. This document reflects the NEW positioning. All previous positioning guidance is superseded.

**OLD:** "IBCS tool for IBCS people"
**NEW:** "Realistic Business Data Generator for anyone who needs consistent test data"

---

## Purpose

This document provides critical context, rules, and patterns that all AI agents must follow when working on the tellingCube codebase. It ensures consistency across all implementations and prevents common mistakes.

---

## Project Overview

**tellingCube** is a Realistic Business Data Generator that produces mathematically consistent synthetic data for demos, testing, and training.

### One-Liner
> "Realistic business data that actually reconciles."

### Core Value Proposition
- Event-sourcing architecture ensures all views derive from the same transaction stream
- Finance, Sales, HR views always reconcile mathematically
- 3 minutes instead of 3 hours to create realistic business data

### The Problem We Solve
> "Synthetic data is either random garbage or takes weeks to build. AI hallucinates numbers that don't reconcile. Real data has compliance issues. You need realistic test data now."

### The Solution
> "tellingCube generates event-based business data where every view—Finance, Sales, HR—derives from the same transaction stream. No reconciliation errors. No data prep. Consistent, export-ready scenarios in minutes."

---

## Positioning Rules

### CRITICAL: What tellingCube IS

tellingCube is a **"Realistic Business Data Generator for anyone who needs consistent test data."**

### CRITICAL: What tellingCube is NOT

tellingCube is **NOT** an "IBCS tool for IBCS people."

### IBCS Guidelines

| Acceptable | NOT Acceptable |
|------------|----------------|
| "Charts follow IBCS® visualization standards" (small print) | "IBCS-ready business data" |
| Mentioning Jürgen Faisst for credibility (sparingly) | "You've mastered IBCS standards..." |
| IBCS as a bonus feature | IBCS in headlines or main selling points |
| | Leading with IBCS as the main topic |

### Key Messages (use consistently)

1. **"Realistic business data that actually reconciles"** - Primary message
2. **"3 minutes instead of 3 hours"** - Time savings
3. **"AI hallucinates. tellingCube doesn't."** - vs AI positioning
4. **"Every € is traceable to a source event"** - Technical credibility
5. **"3% goes to children's hospice"** - hoki.help charity angle

### Target Audiences (priority order)

| Priority | Segment | Use Case |
|----------|---------|----------|
| 1 | BI Consultants | Client demos, POCs, prototypes |
| 2 | Software Vendors | Product demos that look real |
| 3 | Trainers/Educators | Workshops with realistic scenarios |
| 4 | Developers/QA | Test data without production data risks |
| 5 | Data Scientists | Statistically valid synthetic data for ML |

### Competitive Positioning

| vs Competitor | Message |
|---------------|---------|
| vs AI (ChatGPT/Claude) | "AI hallucinates. Numbers don't reconcile." |
| vs Excel/Manual | "3-4 hours in Excel. 3 minutes with tellingCube." |
| vs Random Generators | "Not random garbage. Event-based logic." |
| vs Real Data | "No NDAs, no GDPR issues. Use it anywhere." |

### Social Proof Hierarchy

Use in this order:

1. **Brian Julius (55K followers):**
   > "The ivory-billed woodpecker of the data world. Specific use, but one distinctive click instantly, reliably meets the exact need."

2. **Jürgen Faisst (IBCS Institute):**
   > "Relevant for software vendors and BI consultants who want to create mockups based on real-world models."

3. **Early User Testimonials** (as they come in)

4. **hoki.help Charity Model** - 3% of revenue donated

---

## Technical Stack

### Core Technologies

| Layer | Technology |
|-------|------------|
| Framework | Next.js 14+ (App Router) |
| Language | TypeScript (strict mode) |
| Styling | TailwindCSS |
| Database | NeonDB (PostgreSQL) via Prisma |
| Auth | Magic Link (Resend) - Phase 2 |
| Payments | Stripe |
| Queue | QStash (Upstash) - Phase 3 |
| Hosting | Vercel |
| AI | Claude API (Anthropic) |

### Project Structure

```
/telling-cube
  /app              (Next.js App Router)
    /api            (API routes)
    /(routes)       (Page routes)
  /components       (React components)
  /lib              (Shared utilities)
  /prisma           (Database schema)
  /public           (Static assets)
  /types            (TypeScript definitions)
  /docs             (Documentation)
```

---

## Coding Standards

### TypeScript

- **Strict mode enabled** - No `any` types without explicit justification
- **Explicit return types** on all exported functions
- **Interface over type** for object shapes (unless union types needed)
- **Zod for runtime validation** on all API inputs

### React/Next.js

- **Server Components by default** - Use `'use client'` only when needed
- **App Router conventions** - Follow Next.js 14 patterns
- **Composition over inheritance** - Prefer small, composable components
- **Co-locate related files** - Tests, styles near components

### Naming Conventions

| Element | Convention | Example |
|---------|------------|---------|
| Components | PascalCase | `SalesView.tsx` |
| Utilities | camelCase | `calculateRevenue.ts` |
| API routes | kebab-case | `/api/generate-scenario` |
| Database tables | snake_case | `business_events` |
| Environment vars | SCREAMING_SNAKE | `DATABASE_URL` |

### Git Workflow

- **Default working branch:** `develop`
- **Production/release branch:** `master`
- All feature branches branch off `develop`
- PRs target `develop` unless explicitly releasing to production

---

## Architecture Patterns

### Event Sourcing (Core Pattern)

All business data derives from a single event stream:

```
Events (Source of Truth)
    │
    ├── Finance View (derived)
    ├── Sales View (derived)
    └── HR View (derived)
```

**Rule:** Never generate view data directly. Always generate events first, then derive views.

### API Design

- **RESTful JSON endpoints** - No GraphQL
- **Consistent error format:**
```typescript
{
  error: {
    message: string;
    code: string;
  }
}
```
- **Zod validation** on all inputs
- **Rate limiting** on expensive operations

### Database Access

- **Prisma only** - No raw SQL unless performance-critical
- **Repository pattern** for data access
- **Batch operations** for bulk inserts
- **Indexes** on frequently queried columns

---

## Security Requirements

### Input Validation

- All API inputs validated with Zod
- Never trust client-side data
- Sanitize all user-provided strings

### Authentication (Phase 2+)

- Magic Link only (no passwords)
- httpOnly cookies for session
- CSRF protection on mutations

### Data Protection

- No PII stored unnecessarily
- Ephemeral scenarios deleted after 24h (non-authenticated)
- GDPR compliant by design

### Payment Security

- All payment handling delegated to Stripe
- Never store card data
- Verify webhook signatures

---

## Testing Standards

### Unit Tests (Required)

- **100% coverage** for validation logic
- **80%+ coverage** for query utilities
- Jest as test framework

### Integration Tests (Required for APIs)

- Test all API endpoints
- Use test database instance
- Mock external services (Claude, Stripe)

---

## Performance Budgets

| Metric | Target |
|--------|--------|
| Landing page load | < 2 seconds |
| Generation time | < 5 minutes (90th percentile) |
| Chart rendering | < 500ms |
| JS bundle | < 200KB gzipped |
| API response | < 200ms for view queries |

---

## UI/UX Guidelines

### Brand Colors

| Name | Hex | Usage |
|------|-----|-------|
| Primary | #2563EB | CTAs, links, primary actions |
| Secondary | #E5EAF5 | Backgrounds, subtle elements |
| Accent | #10B981 | Success states, positive values |
| Error | #EF4444 | Errors, negative values |
| Text | #1F2937 | Body text |
| Muted | #6B7280 | Secondary text |

### Typography

- **Font:** Inter (with system fallbacks)
- **Headings:** 600-700 weight
- **Body:** 400 weight

---

## Pricing Model

| Tier | Price | Includes |
|------|-------|----------|
| Single | €9 | 1 generation, 1 year scenarios |
| Lifetime | €99 | Unlimited generations, 1-3 year scenarios |
| Lifetime+ | €299 | Unlimited, 1-5 years, HR domain |
| Pro | €19/mo | Unlimited, 1-5 years, HR domain, 1 seat |
| Team | €49/mo | Unlimited, 1-5 years, HR domain, 5 seats |

**Charity:** 3% of all revenue donated to hoki.help (Children's hospice Austria)

---

## Implementation Phases (Big Bang Launch)

### Phase 1: Copy/Messaging (This Week)
- Landing page new messaging
- Chart labels: "Data Preview" not "Inspired by IBCS©"
- Remove IBCS from main navigation
- Meta/SEO update

### Phase 2: Auth + Dashboard (1-2 weeks)
- Magic Link authentication
- User dashboard
- Scenario history

### Phase 3: Async + Pricing (1 week)
- QStash integration
- New Stripe products (5 tiers)
- Email notifications

### Phase 4: New Features (1-2 weeks)
- HR Domain
- Multi-year logic (1/3/5 years)
- hoki.help counter

---

## Common Pitfalls to Avoid

### DO NOT

- Generate view data directly (always use event stream)
- Store sensitive data in client-side state
- Use `any` type without justification
- Skip input validation on APIs
- Commit `.env` files or secrets
- **Use IBCS as primary positioning**
- Add features not in current sprint scope

### DO

- Follow event-sourcing pattern strictly
- Validate all inputs with Zod
- Write tests for validation logic
- Use TypeScript strict mode
- Keep components small and focused
- **Position product as "data that reconciles"**
- **Lead with Brian Julius quote for social proof**

---

## Quick Reference

### When writing copy/content:

1. Lead with "data that reconciles"
2. Use Brian Julius quote as primary social proof
3. Mention hoki.help charity angle
4. IBCS only as footnote/bonus feature
5. Target BI consultants and software vendors first

### When implementing features:

1. Check this context document first
2. Follow event-sourcing pattern
3. Validate inputs with Zod
4. Write tests for critical logic
5. Keep bundle size in mind

### When unsure:

1. Check `docs/product-brief.md` for positioning
2. Check `docs/prd.md` for requirements
3. Check `docs/tellingcube-strategic-briefing.md` for strategic context
4. Ask for clarification before implementing

---

## Document References

| Document | Purpose |
|----------|---------|
| `docs/product-brief.md` | Product positioning and messaging |
| `docs/prd.md` | Detailed requirements and stories |
| `docs/tellingcube-strategic-briefing.md` | Strategic decision context |
| `docs/tellingcube-repositioning-strategy.md` | Implementation details |

---

## Company Info

- **Company:** masemIT e.U.
- **Location:** Austria
- **Founder:** Mario Semper
- **Domain:** tellingcube.com

---

**Document Control**

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0-1.4 | 2025-12 | Various | Original MVP documentation |
| 2.0 | 2026-01-21 | Paige (TW) | Complete repositioning - IBCS to Data Reconciliation |

---

*This document is the bible for all agents working on tellingCube. Follow it strictly.*
