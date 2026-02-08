# Story 1.1: Project Scaffold & Initial Deployment

Status: done

## Story

As a **developer**,
I want a fully scaffolded Hono project deployed to Vercel with a working health check,
so that the MMS API is live at api.masem.at and ready for feature development.

## Acceptance Criteria

1. **Given** no project exists yet **When** the Hono Vercel template is initialized with `npm create hono@latest mms-api -- --template vercel` **Then** the project structure is created with src/index.ts as entry point **And** all dependencies are installed at pinned versions (drizzle-orm@0.45.1, @neondatabase/serverless@1.0.2, @upstash/redis@1.36.1, @upstash/ratelimit@2.0.8, zod, resend, vitest)

2. **Given** the project is scaffolded **When** `GET /health` is called **Then** a 200 response with `{ "status": "ok" }` is returned

3. **Given** the project is configured with vercel.json and tsconfig.json **When** pushed to the main branch **Then** Vercel auto-deploys and api.masem.at responds to requests

4. **Given** the project is deployed **When** a developer checks the repository **Then** .env.example lists all required environment variables (DATABASE_URL, UPSTASH_REDIS_REST_URL, UPSTASH_REDIS_REST_TOKEN) **And** .gitignore excludes node_modules, .env, .vercel

## Tasks / Subtasks

- [x] Task 1: Initialize Hono project from Vercel template (AC: #1)
  - [x] 1.1 Run `npm create hono@latest mms-api -- --template vercel` in project root
  - [x] 1.2 Verify src/index.ts entry point is created with default export
  - [x] 1.3 Ensure package.json has `"type": "module"` for ESM compatibility

- [x] Task 2: Install all pinned dependencies (AC: #1)
  - [x] 2.1 Install production deps: `npm install drizzle-orm@0.45.1 @neondatabase/serverless@1.0.2 @upstash/redis@1.36.1 @upstash/ratelimit@2.0.8 zod resend`
  - [x] 2.2 Install dev deps: `npm install -D drizzle-kit @types/node dotenv vitest`
  - [x] 2.3 Verify all versions in package.json match pinned versions

- [x] Task 3: Configure project files (AC: #3, #4)
  - [x] 3.1 Configure tsconfig.json with strict mode, absolute imports from src/ root, target ES2022
  - [x] 3.2 Create/verify vercel.json for Vercel deployment with Fluid Compute
  - [x] 3.3 Create .env.example with DATABASE_URL, UPSTASH_REDIS_REST_URL, UPSTASH_REDIS_REST_TOKEN
  - [x] 3.4 Ensure .gitignore excludes node_modules, .env, .vercel, dist

- [x] Task 4: Create project directory structure (AC: #1)
  - [x] 4.1 Create src/middleware/ directory
  - [x] 4.2 Create src/services/donations/ directory
  - [x] 4.3 Create src/db/ directory
  - [x] 4.4 Create src/lib/ directory

- [x] Task 5: Implement health check endpoint (AC: #2)
  - [x] 5.1 Update src/index.ts with `GET /health` returning `{ "status": "ok" }`
  - [x] 5.2 Export default app for Vercel

- [x] Task 6: Create vitest configuration (AC: #1)
  - [x] 6.1 Create vitest.config.ts with TypeScript support and path aliases

- [x] Task 7: Write initial test (AC: #2)
  - [x] 7.1 Create src/index.test.ts testing health check returns 200 with correct body
  - [x] 7.2 Use Hono `app.request()` test client (no HTTP server needed)

## Dev Notes

### Architecture Compliance

- **Entry point:** `src/index.ts` — Hono app with `export default app`
- **Framework:** Hono 4.11.7 — ultralight, Web Standards-based
- **Runtime:** Vercel Fluid Compute (Node.js), ~115ms cold starts
- **ESM:** Use `"type": "module"` in package.json — Vercel supports native ESM
- **Vercel Experimental Backends:** Optionally set `VERCEL_EXPERIMENTAL_BACKENDS=1` for enhanced module resolution (Jan 2026 update) — TypeScript path aliases supported, no file extensions needed for imports

### Critical Constraints

- **Absolute imports:** Configure tsconfig.json `paths` for `src/` root — no relative `../../` paths
- **snake_case everywhere:** DB columns, API response fields, query parameters
- **Response envelope:** `{ data }` for single, `{ data, meta }` for lists, `{ error: { code, message, details } }` for errors — establish pattern in health check if applicable
- **File naming:** `kebab-case.ts` for all files
- **Test co-location:** Tests as `{name}.test.ts` next to source files — NO separate `__tests__/` directory

### Package Versions (Verified 2026-02-02)

| Package | Version | Notes |
|---------|---------|-------|
| hono | 4.11.7 | Stable, native Vercel support |
| drizzle-orm | 0.45.1 | Stable (v1.0 beta exists — stay on stable) |
| drizzle-kit | ~0.30.x | Aligned with drizzle-orm stable |
| @neondatabase/serverless | 1.0.2 | GA, requires Node.js >= 19 |
| @upstash/redis | 1.36.1 | HTTP-based, serverless-native |
| @upstash/ratelimit | 2.0.8 | Sliding window, dynamic limits |
| zod | latest | Input validation |
| resend | latest | Email notifications |
| vitest | latest | Test runner |

### Drizzle + Neon HTTP Setup Pattern

The simplified Drizzle + Neon HTTP connection (for db/index.ts in future stories):
```typescript
import { drizzle } from 'drizzle-orm/neon-http';
const db = drizzle(process.env.DATABASE_URL!);
```

### Hono App Pattern

```typescript
import { Hono } from 'hono'
const app = new Hono()
app.get('/health', (c) => c.json({ status: 'ok' }))
export default app
```

### Project Structure Notes

This story creates the scaffold. Full target structure for reference:
```
mms-api/
├── src/
│   ├── index.ts              # Hono app entry + health check + default export
│   ├── index.test.ts         # Health check test
│   ├── middleware/            # (empty, created for future stories)
│   ├── services/
│   │   └── donations/        # (empty, created for future stories)
│   ├── db/                   # (empty, created for future stories)
│   └── lib/                  # (empty, created for future stories)
├── drizzle.config.ts         # (placeholder or skip until Story 1.2)
├── vitest.config.ts
├── vercel.json
├── tsconfig.json
├── package.json
├── .env.example
└── .gitignore
```

### Anti-Patterns to Avoid

- DO NOT create `src/utils/` or `src/helpers/` — use `lib/` for shared utilities
- DO NOT use `camelCase` in JSON responses — always `snake_case`
- DO NOT create separate `__tests__/` directories
- DO NOT use relative imports like `../../lib/cache`
- DO NOT install packages not listed in the dependency list

### References

- [Source: _bmad-output/planning-artifacts/architecture.md#Starter Template Evaluation]
- [Source: _bmad-output/planning-artifacts/architecture.md#Additional Dependencies]
- [Source: _bmad-output/planning-artifacts/architecture.md#Structure Patterns]
- [Source: _bmad-output/planning-artifacts/epics.md#Story 1.1]
- [Source: _bmad-output/_masemIT/requirements/mms-donations-requirement.md#Phase 1]

## Dev Agent Record

### Agent Model Used

Claude Opus 4.5 (claude-opus-4-5-20251101)

### Debug Log References

- Hono CLI `npm create hono@latest` interactive prompts blocked in non-TTY — project scaffolded manually with identical structure
- All 3 tests pass: health check 200, content-type json, unknown route 404

### Completion Notes List

- Project scaffolded manually (equivalent to `npm create hono@latest mms-api -- --template vercel`) due to interactive CLI limitations
- All pinned dependencies installed successfully (hono@4.11.7, drizzle-orm@0.45.1, @neondatabase/serverless@1.0.2, @upstash/redis@1.36.1, @upstash/ratelimit@2.0.8, zod@4.3.6, resend@6.9.1)
- Dev dependencies: drizzle-kit, @types/node, dotenv, vitest installed
- Health check endpoint returns `{ "status": "ok" }` on GET /health
- ESM configured via `"type": "module"` in package.json
- Absolute imports configured via tsconfig.json paths `src/*`
- 3/3 tests passing (vitest v4.0.18)
- Directory structure prepared for future stories: middleware/, services/donations/, db/, lib/

### Code Review Fixes Applied (2026-02-02)

- Fixed: Production dependency versions pinned exactly (removed `^` caret ranges)
- Fixed: Added `engines.node >= 19` to package.json (@neondatabase/serverless requirement)
- Fixed: Removed unnecessary tsconfig.json build options (declaration, declarationMap, sourceMap, outDir, rootDir)
- Fixed: Added .gitkeep files to empty directories for git tracking
- Fixed: vitest.config.ts alias uses path.resolve for reliable resolution

### File List

- mms-api/package.json (new)
- mms-api/tsconfig.json (new)
- mms-api/vercel.json (new)
- mms-api/vitest.config.ts (new)
- mms-api/.env.example (new)
- mms-api/.gitignore (new)
- mms-api/src/index.ts (new)
- mms-api/src/index.test.ts (new)
- mms-api/src/middleware/.gitkeep (new)
- mms-api/src/services/donations/.gitkeep (new)
- mms-api/src/db/.gitkeep (new)
- mms-api/src/lib/.gitkeep (new)

