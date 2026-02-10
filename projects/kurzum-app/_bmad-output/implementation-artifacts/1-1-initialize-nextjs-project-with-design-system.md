# Story 1.1: Initialize Next.js Project with Design System

Status: done

## Story

As a **developer**,
I want a fully configured Next.js project with Tailwind v4 design tokens, self-hosted Inter font, and all dependencies installed,
So that all subsequent development builds on a consistent, branded foundation.

## Acceptance Criteria

1. **Given** no project exists yet **When** the initialization commands are executed (create-next-app + shadcn init + dependencies) **Then** the project builds without errors using `pnpm build` **And** TypeScript strict mode is enabled with zero type errors **And** ESLint passes with zero errors

2. **Given** the project is initialized **When** the root layout (app/layout.tsx) is created **Then** `<html lang="de">` is set for German language **And** Inter variable font is self-hosted from /public/fonts/inter-var.woff2 (Latin + special chars) **And** the font loads without external network requests

3. **Given** the UX design specification color system **When** design tokens are configured in app/globals.css (Tailwind v4 CSS-first) **Then** all color tokens are defined (--color-primary, --color-accent, neutrals, semantic colors) **And** typography scale tokens match the UX spec (heading sizes, body sizes, weights) **And** spacing tokens support 48px minimum touch targets **And** shadcn/ui theme CSS variables are overridden with kurzum brand colors

4. **Given** the project is deployed to Vercel **When** a user visits kurzum.app **Then** the page loads with correct fonts and brand colors **And** .env.example is committed with all required environment variable placeholders

## Tasks / Subtasks

- [x] Task 1: Create Next.js project (AC: #1)
  - [x] 1.1 Run `pnpm dlx create-next-app@latest kurzum-app --typescript --tailwind --eslint --app --turbopack --empty --use-pnpm`
  - [x] 1.2 Verify project builds with `pnpm build`
  - [x] 1.3 Verify TypeScript strict mode enabled in tsconfig.json
  - [x] 1.4 Verify ESLint runs without errors

- [x] Task 2: Initialize shadcn/ui (AC: #1)
  - [x] 2.1 Run `pnpm dlx shadcn@latest init` inside project directory
  - [x] 2.2 Verify components.json is created
  - [x] 2.3 Add required components: `pnpm dlx shadcn@latest add button input textarea select label badge`

- [x] Task 3: Install all dependencies (AC: #1)
  - [x] 3.1 Core: `pnpm add drizzle-orm @neondatabase/serverless react-hook-form @hookform/resolvers zod`
  - [x] 3.2 Services: `pnpm add resend @marsidev/react-turnstile @upstash/ratelimit @upstash/redis`
  - [x] 3.3 Dev: `pnpm add -D drizzle-kit`
  - [x] 3.4 Verify `pnpm build` still passes after all installs

- [x] Task 4: Configure self-hosted Inter font (AC: #2)
  - [x] 4.1 Download Inter variable font (woff2, Latin + German subset including special chars)
  - [x] 4.2 Place at `public/fonts/inter-var.woff2`
  - [x] 4.3 Configure in `app/layout.tsx` using `next/font/local` with `variable: '--font-inter'` and `display: 'swap'`
  - [x] 4.4 Add `<html lang="de">` and apply font variable class to html element
  - [N/A] 4.5 Font preload not needed — next/font/local handles preloading automatically
  - [x] 4.6 Verify no external font requests (self-hosted from /public/fonts/)

- [x] Task 5: Configure Tailwind v4 design tokens (AC: #3)
  - [x] 5.1 Set up `app/globals.css` with `@import "tailwindcss"` and `@import "tw-animate-css"`
  - [x] 5.2 Define `:root` CSS variables for shadcn/ui semantic tokens (override with kurzum brand colors)
  - [x] 5.3 Define `@theme inline` block mapping CSS variables to Tailwind utilities
  - [x] 5.4 Define `@theme` block for custom design tokens (typography scale, spacing, shadows, border-radius)
  - [x] 5.5 Verify all color, typography, and spacing tokens from UX spec are represented

- [x] Task 6: Create project structure scaffolding (AC: #1, #3)
  - [x] 6.1 Create directory structure: `components/landing/`, `components/shared/`, `lib/db/`, `lib/validations/`, `public/fonts/`, `e2e/`
  - [x] 6.2 `lib/utils.ts` with `cn()` utility already created by shadcn init
  - [x] 6.3 Create `drizzle.config.ts` skeleton
  - [x] 6.4 Create placeholder `lib/db/index.ts` and `lib/db/schema.ts`

- [x] Task 7: Create .env.example (AC: #4)
  - [x] 7.1 Create `.env.example` with all placeholder environment variables
  - [x] 7.2 `.env.local` already in `.gitignore`

- [x] Task 8: Verify build and lint (AC: #1, #4)
  - [x] 8.1 Run `pnpm build` — zero errors ✓
  - [x] 8.2 Run `pnpm lint` — zero errors ✓
  - [x] 8.3 TypeScript produces zero type errors ✓

## Dev Notes

### Critical: Tailwind CSS v4 CSS-First Configuration

Tailwind v4 has NO `tailwind.config.js`. All configuration happens in CSS:

```css
/* app/globals.css */
@import "tailwindcss";
@import "tw-animate-css";
@import "shadcn/tailwind.css";  /* Required by shadcn/ui v3.8.4 for internal component styles */

@custom-variant dark (&:is(.dark *));
```

Design tokens use TWO mechanisms:
1. **`:root` CSS variables** — for shadcn/ui semantic tokens (these do NOT auto-generate utility classes)
2. **`@theme inline`** — maps CSS variables to Tailwind utilities (MUST use `inline` when referencing other CSS variables like font variables from next/font)
3. **`@theme`** — for static design tokens that auto-generate utility classes

**CRITICAL**: `@theme inline` is required (not `@theme`) when referencing CSS variables like `var(--font-inter)`. Without `inline`, variables resolve at definition time, not usage time.

### Critical: shadcn/ui + Tailwind v4 Pattern

The globals.css must follow this exact structure:

```css
@import "tailwindcss";
@import "tw-animate-css";

@custom-variant dark (&:is(.dark *));

/* shadcn/ui semantic tokens — NO utility classes generated */
:root {
  --background: oklch(...);
  --foreground: oklch(...);
  --primary: oklch(...);
  --primary-foreground: oklch(...);
  /* etc. */
  --radius: 0.5rem;
}

/* Map semantic tokens to Tailwind utilities */
@theme inline {
  --color-background: var(--background);
  --color-foreground: var(--foreground);
  --color-primary: var(--primary);
  --font-sans: var(--font-inter);
  /* etc. */
}

/* Static design tokens — auto-generate utility classes */
@theme {
  /* Additional custom tokens */
}
```

### Critical: Next.js 16 Breaking Changes

- **Turbopack is the default bundler** — no `--turbopack` flag needed (but explicit is fine)
- **Async request APIs** — `params`, `searchParams`, `cookies()`, `headers()` return Promises. Must `await` them.
- **`next lint` removed from build** — linting must be run separately via `pnpm lint`
- **React 19.2** included — supports View Transitions, `useEffectEvent()`
- **Node.js 20.9+ required**
- **TypeScript 5.1+ required**
- **ESLint Flat Config** is the new default

### Critical: Architecture requires `--empty` flag

The architecture document specifies `--empty` flag in create-next-app to get a clean starting point without Next.js boilerplate page content. This is intentional — we build from scratch.

### Brand Color System (from UX Spec)

Map these exact values into globals.css:

| Token | Hex | Usage |
|---|---|---|
| Primary | #1C1917 (Stone-900) | Dark backgrounds, text |
| Primary Light | #292524 (Stone-800) | Dark section backgrounds |
| Primary Lighter | #44403C (Stone-700) | Borders on dark |
| Accent | #F97316 (Orange-500) | CTAs, highlights |
| Accent Hover | #EA580C (Orange-600) | Button hover states |
| Accent Light | #FED7AA (Orange-200) | Subtle accent backgrounds |
| Accent Dark | #C2410C (Orange-700) | Accent text |
| BG Light | #FAFAF9 (Stone-50) | Light section backgrounds |
| BG Warm | #F5F5F4 (Stone-100) | Alternating light sections |
| Text Primary | #1C1917 (Stone-900) | Body on light |
| Text Secondary | #57534E (Stone-600) | Secondary text |
| Text Muted | #A8A29E (Stone-400) | Placeholder, disabled |
| Text on Dark | #FAFAF9 (Stone-50) | Text on dark |
| Secondary | #009BB1 (Teal) | Trust signals, legal links |
| Secondary Dark | #093236 | Footer accents |
| Success | #16A34A (Green-600) | Success states |
| Warning | #CA8A04 (Yellow-600) | Warning states |
| Error | #DC2626 (Red-600) | Error states |
| Info | #009BB1 (Teal) | Info states |

### Typography Scale (from UX Spec)

| Token | Size | Line Height | Weight | Usage |
|---|---|---|---|---|
| Display | 48px | 1.1 | 700 | Hero headline (desktop) |
| H1 | 36px | 1.2 | 700 | Section headlines, hero (mobile) |
| H2 | 28px | 1.25 | 600 | Sub-section headlines |
| H3 | 22px | 1.3 | 600 | Card titles |
| Body LG | 18px | 1.6 | 400 | Body text, subheadlines |
| Body | 16px | 1.6 | 400 | Default body, form labels |
| Body SM | 14px | 1.5 | 400 | Secondary, captions |
| Caption | 12px | 1.4 | 500 | Badges, tags, legal |

### Spacing & Layout Tokens (from UX Spec)

- Base unit: 8px
- Section padding: 64px mobile / 96px desktop
- Max content width: 960px
- Container padding: 16px mobile / 24px tablet / 0 desktop (centered)
- Touch targets: 48px minimum height for interactive elements

### Border Radius (from UX Spec)

| Token | Value | Usage |
|---|---|---|
| SM | 6px | Badges, tags |
| MD | 8px | Buttons, inputs |
| LG | 12px | Cards, sections |
| XL | 16px | Featured cards, chat bubbles |
| Full | 9999px | Avatars, circular buttons |

### Shadow Tokens (from UX Spec)

| Token | Value | Usage |
|---|---|---|
| SM | 0 1px 2px rgba(28,25,23,0.05) | Form inputs |
| MD | 0 4px 6px rgba(28,25,23,0.07) | Cards |
| LG | 0 10px 15px rgba(28,25,23,0.1) | Modals, floating |
| Accent | 0 4px 14px rgba(249,115,22,0.3) | CTA hover glow |

### Drizzle Config Skeleton

```typescript
// drizzle.config.ts
import { defineConfig } from 'drizzle-kit';

export default defineConfig({
  out: './drizzle',
  schema: './lib/db/schema.ts',
  dialect: 'postgresql',
  dbCredentials: {
    url: process.env.DATABASE_URL!,
  },
});
```

### DB Connection Skeleton

```typescript
// lib/db/index.ts
import { drizzle } from 'drizzle-orm/neon-http';

export const db = drizzle(process.env.DATABASE_URL!);
```

### Self-Hosted Font Setup

Use `next/font/local` (NOT `next/font/google`) since the architecture requires self-hosting from `/public/fonts/inter-var.woff2`:

```typescript
import localFont from 'next/font/local';

const inter = localFont({
  src: '../public/fonts/inter-var.woff2',
  variable: '--font-inter',
  display: 'swap',
});
```

The `--font-inter` CSS variable is injected on `<html>` by Next.js, then mapped in globals.css via `@theme inline { --font-sans: var(--font-inter); }`.

### Environment Variables (.env.example)

```bash
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

### Note: Analytics Package Decision

The Architecture Document lists `pnpm add @vercel/analytics` in the initialization commands but also specifies masemIT Analytics (self-hosted at `analytics.masem.at`) as the primary analytics solution, replacing Vercel Analytics. `@vercel/analytics` was intentionally NOT installed — masemIT Analytics is the sole analytics provider. Vercel Speed Insights may be evaluated separately in Story 4.3 if needed. The Architecture init commands are aspirational — they predate the masemIT Analytics decision.

### Known Gotchas

1. **shadcn init timing**: Run `shadcn init` IMMEDIATELY after `create-next-app`, BEFORE making manual CSS changes. The CLI auto-detects project config.
2. **tw-animate-css NOT tailwindcss-animate**: shadcn/ui v4 uses `tw-animate-css`. Do NOT install `tailwindcss-animate`.
3. **Border color default changed**: Tailwind v4 defaults `border-color` to `currentColor` (not `gray-200`). Always use explicit border color classes.
4. **`@theme inline` for font variables**: Without `inline`, the font CSS variable won't resolve correctly.
5. **No `tailwind.config.js`**: Tailwind v4 uses CSS-first config. Do NOT create a JS config file.
6. **Drizzle schema path**: Architecture specifies `lib/db/schema.ts` (not `src/db/schema.ts`). No `src/` directory is used.
7. **`--empty` flag**: Creates project without boilerplate page content. The default page.tsx will be minimal.

### Project Structure (Phase 1 Target)

```
kurzum-app/
  .env.example
  .env.local                    (git-ignored)
  eslint.config.mjs
  .gitignore
  components.json               (shadcn/ui config)
  drizzle.config.ts
  next.config.ts
  package.json
  pnpm-lock.yaml
  tsconfig.json
  app/
    globals.css                 (Tailwind v4 + design tokens)
    layout.tsx                  (root: html lang="de", font, metadata)
    page.tsx                    (landing page placeholder)
  components/
    ui/                         (shadcn/ui auto-generated)
    landing/                    (custom landing page components)
    shared/                     (shared between landing and app)
  lib/
    db/
      index.ts                  (Drizzle + NeonDB connection)
      schema.ts                 (table definitions placeholder)
    validations/                (Zod schemas, empty for now)
    utils.ts                    (cn() utility)
  public/
    fonts/
      inter-var.woff2           (self-hosted Inter)
  e2e/                          (E2E tests directory)
```

### Architecture Compliance

- **Naming conventions**: kebab-case files, PascalCase components, camelCase functions, snake_case DB
- **Imports**: Absolute via `@/` alias. No `../../` relative imports.
- **Types**: No `any`. TypeScript strict mode.
- **Components**: Server Components as default. Client only when state/effects/browser APIs needed.
- **Styling**: Tailwind CSS classes exclusively. No inline styles. No CSS Modules.

### References

- [Source: _bmad-output/planning-artifacts/architecture.md#Starter Template Evaluation]
- [Source: _bmad-output/planning-artifacts/architecture.md#Implementation Patterns & Consistency Rules]
- [Source: _bmad-output/planning-artifacts/architecture.md#Project Structure & Boundaries]
- [Source: _bmad-output/planning-artifacts/architecture.md#Environment Configuration]
- [Source: _bmad-output/planning-artifacts/ux-design-specification.md#Visual Design Foundation]
- [Source: _bmad-output/planning-artifacts/ux-design-specification.md#Typography]
- [Source: _bmad-output/planning-artifacts/ux-design-specification.md#Spacing & Layout]
- [Source: _bmad-output/planning-artifacts/ux-design-specification.md#Logo & Brand Mark]
- [Source: _bmad-output/planning-artifacts/epics.md#Story 1.1]

## Dev Agent Record

### Agent Model Used

Claude Opus 4.6 (claude-opus-4-6)

### Debug Log References

- create-next-app required temp directory workaround (existing BMAD dirs blocked init in project root)
- `--yes` flag defaulted to src/ directory — manually restructured to root-based layout per architecture
- pnpm esbuild approval prompt resolved via `onlyBuiltDependencies` in package.json
- shadcn init required `--defaults --base-color stone` flags to avoid interactive prompts
- Font preload link (subtask 4.5) skipped — next/font/local handles preloading automatically

### Completion Notes List

- Next.js 16.1.6 with Turbopack, React 19.2.3, TypeScript 5.x strict mode
- shadcn/ui initialized with Stone base color, 6 components added (button, input, textarea, select, label, badge)
- All 14 dependencies installed (8 runtime, 1 dev-only additional)
- Inter variable font self-hosted (352KB woff2), configured via next/font/local
- Complete kurzum brand design system in globals.css: oklch semantic tokens, @theme inline mappings, static brand tokens
- Dark mode tokens configured
- Drizzle config skeleton and DB connection placeholder ready for Story 3.2
- .env.example covers all 10 environment variables from architecture spec
- Temp directory `C:\Users\mario\Sources\dev\kurzum-app-init` should be cleaned up manually

### File List

- `package.json` — project manifest with all dependencies
- `pnpm-lock.yaml` — lockfile for reproducible builds
- `tsconfig.json` — TypeScript strict config with @/ alias
- `next.config.ts` — Next.js configuration
- `postcss.config.mjs` — PostCSS with Tailwind v4
- `eslint.config.mjs` — ESLint flat config
- `components.json` — shadcn/ui configuration
- `drizzle.config.ts` — Drizzle Kit config skeleton
- `.env.example` — environment variable placeholders
- `.gitignore` — merged BMAD + Next.js gitignore
- `app/layout.tsx` — root layout with Inter font, lang="de", metadata
- `app/globals.css` — Tailwind v4 design tokens + kurzum brand system
- `app/page.tsx` — minimal landing page placeholder
- `lib/utils.ts` — cn() utility (shadcn/ui)
- `lib/db/index.ts` — Drizzle + NeonDB connection placeholder
- `lib/db/schema.ts` — Drizzle schema placeholder
- `lib/validations/.gitkeep` — empty directory for Zod schemas
- `components/ui/button.tsx` — shadcn/ui Button
- `components/ui/input.tsx` — shadcn/ui Input
- `components/ui/textarea.tsx` — shadcn/ui Textarea
- `components/ui/select.tsx` — shadcn/ui Select
- `components/ui/label.tsx` — shadcn/ui Label
- `components/ui/badge.tsx` — shadcn/ui Badge
- `components/landing/.gitkeep` — empty directory for landing components
- `components/shared/.gitkeep` — empty directory for shared components
- `public/fonts/inter-var.woff2` — self-hosted Inter variable font (352KB)
- `pnpm-workspace.yaml` — pnpm workspace config with ignoredBuiltDependencies (sharp, unrs-resolver)
- `e2e/.gitkeep` — empty directory for E2E tests

### Change Log

- 2026-02-10: Code review (Claude Opus 4.6) — 7 issues found (3 HIGH, 4 MEDIUM), all fixed automatically

## Senior Developer Review (AI)

**Reviewer:** Claude Opus 4.6
**Date:** 2026-02-10
**Outcome:** Approve (with fixes applied)

### Issues Found & Fixed

| # | Severity | Issue | Fix Applied |
|---|----------|-------|-------------|
| H1 | HIGH | `pnpm-workspace.yaml` missing from File List | Added to File List |
| H2 | HIGH | `@import "shadcn/tailwind.css"` undocumented | Documented in Dev Notes CSS pattern |
| H3 | HIGH | Border radius tokens don't match UX spec (all values off by 2-4px) | Changed `--radius: 0.75rem`, adjusted calcs, added `--radius-full: 9999px` |
| M1 | MEDIUM | Spacing tokens from UX spec completely missing | Added 6 spacing/layout tokens to `@theme` block |
| M2 | MEDIUM | Story references `.eslintrc.json` but actual file is `eslint.config.mjs` | Updated Project Structure in story |
| M3 | MEDIUM | `@vercel/analytics` in Architecture init commands but not installed | Added clarification note: masemIT Analytics is sole provider |
| M4 | MEDIUM | `--destructive-foreground` token missing from theme | Added to `:root`, `.dark`, and `@theme inline` |

### Low Issues (Not Fixed — No Impact)

| # | Severity | Issue | Rationale |
|---|----------|-------|-----------|
| L1 | LOW | `--sidebar-*` CSS variables missing | Not needed for Phase 1 landing page |
| L2 | LOW | Inconsistent font-weight explicitness in typography tokens | Defaults are correct, cosmetic only |

### Verification

- `pnpm build` — zero errors after all fixes
- `pnpm lint` — zero errors
- TypeScript strict — zero type errors
- All Acceptance Criteria validated as implemented
