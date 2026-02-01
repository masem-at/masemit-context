# Technical Debt Inventory

**Author:** James (Full Stack Developer)
**Date:** 2025-12-09
**Purpose:** Honest accounting of shortcuts, hacks, and issues before clean rebuild

---

## 🔴 Critical Technical Debt (Fix During Rebuild)

### DEBT-001: Unused Background Function Routes
**Location:**
- `app/api/generate/background/route.ts`
- `app/api/status/[scenarioId]/route.ts`

**Issue:**
- Created for async architecture that didn't work
- Currently unused in synchronous approach
- Will be repurposed for QStash but need cleanup

**Action:**
- Rename `background` → `worker`
- Add QStash signature verification
- Remove or repurpose status endpoint (might keep for polling)
- Delete dead code

**Technical Debt Cost:** Low (2 files to clean up)

---

### DEBT-002: Inconsistent Progress Tracking
**Location:**
- Frontend: `app/generate/loading/page.tsx`
- Backend: `app/api/generate/route.ts`
- Database: `prisma/schema.prisma`

**Issue:**
- Database has `progress`, `currentStage`, `errorMessage` fields
- Frontend simulates progress (not using DB values correctly)
- Backend synchronous approach doesn't update progress during run
- Mismatch between what's stored and what's displayed

**Action:**
- Use real backend progress with QStash
- Update frontend to trust DB values
- Remove simulated progress logic
- Align progress percentages with actual work stages

**Technical Debt Cost:** Medium (3 files to align)

---

### DEBT-003: Error Handling Inconsistency
**Location:** All API routes

**Issue:**
- Some routes use `ApiError` class
- Some routes use plain `Error`
- Some routes return `{ error: string }`
- Some routes return `{ status: 'error', message: string }`
- Inconsistent HTTP status codes

**Action:**
- Standardize on `ApiError` class everywhere
- Consistent error response shape
- Document error codes
- Proper HTTP status codes (400, 404, 429, 500)

**Technical Debt Cost:** Medium (5+ routes to standardize)

---

### DEBT-004: Rate Limiting Disabled
**Location:** `lib/middleware/rate-limit.ts`

**Issue:**
```typescript
Rate limiting disabled: Redis not configured
```

**Root Cause:**
- Upstash Redis requires env vars
- Not configured in development
- Not tested in production

**Action:**
- Add Redis env vars to Vercel
- Test rate limiting works
- Document rate limits per endpoint
- Verify limits are reasonable

**Technical Debt Cost:** Low (just config, code exists)

---

### DEBT-005: Validation Logic Unclear
**Location:** `lib/validators/consistency-validator.ts`

**Issue:**
- Validation finds "18 sales events have invalid amounts"
- Not clear what makes an amount invalid
- Might be false positive
- Might be real generation bug

**Action:**
- Review validation rules
- Add comments explaining each rule
- Test against known-good data
- Fix if validation is wrong, OR fix generation if data is wrong

**Technical Debt Cost:** Medium (requires investigation)

---

## ⚠️ High Priority Technical Debt

### DEBT-006: No TypeScript Strict Mode
**Location:** `tsconfig.json`

**Issue:**
```json
{
  "compilerOptions": {
    "strict": false  // Should be true
  }
}
```

**Impact:**
- Missing null checks
- Implicit any types
- Potential runtime errors

**Action:**
- Enable `"strict": true`
- Fix all TypeScript errors (might be many)
- Add explicit types where needed

**Technical Debt Cost:** High (could be 50+ errors to fix)

**Note:** Defer until after QStash implementation to avoid scope creep

---

### DEBT-007: Database Cleanup Strategy Missing
**Location:** Database management

**Issue:**
- Test scenarios accumulating
- No deletion mechanism
- Users see old scenarios (even after "cleanup")
- Possible caching issue

**Questions:**
- Should scenarios auto-expire after 24 hours?
- Should users be able to delete?
- Is there a cache layer we're not aware of?

**Action:**
- Implement scenario deletion endpoint
- Add "Delete" button in UI
- Respect `expiresAt` field in database
- Add cleanup cron job for expired scenarios

**Technical Debt Cost:** Medium (new endpoint + UI + cron)

---

### DEBT-008: Environment Variables Not Documented
**Location:** `.env.example` (doesn't exist!)

**Issue:**
- Required env vars scattered across code
- No single source of truth
- Hard to onboard new developers
- Easy to miss required vars in deployment

**Current Required Vars:**
```bash
# Database
DATABASE_URL=

# Anthropic
ANTHROPIC_API_KEY=

# Stripe
STRIPE_SECRET_KEY=
STRIPE_WEBHOOK_SECRET=
STRIPE_PRICE_ID_ONE_TIME=
STRIPE_PRICE_ID_PRO=
STRIPE_PRICE_ID_UNLIMITED=

# Resend (Email)
RESEND_API_KEY=

# Upstash Redis (for rate limiting)
UPSTASH_REDIS_REST_URL=
UPSTASH_REDIS_REST_TOKEN=

# Upstash QStash (NEW - for async jobs)
QSTASH_URL=
QSTASH_TOKEN=
QSTASH_CURRENT_SIGNING_KEY=
QSTASH_NEXT_SIGNING_KEY=

# Next.js
NEXT_PUBLIC_BASE_URL=
```

**Action:**
- Create `.env.example` with all vars
- Add comments explaining each
- Document in README
- Verify Vercel has all vars

**Technical Debt Cost:** Low (documentation task)

---

### DEBT-009: No Logging Strategy
**Location:** Everywhere (console.log scattered)

**Issue:**
- Using `console.log` everywhere
- No log levels (debug, info, warn, error)
- No structured logging
- Hard to filter in production
- No log aggregation

**Action (Future):**
- Consider Pino or Winston for structured logging
- Add log levels
- Consistent log format
- Integrate with Vercel's logging

**Technical Debt Cost:** High (many files to change)

**Note:** Defer to post-MVP - current logging is adequate for now

---

## 🟡 Medium Priority Technical Debt

### DEBT-010: Generators Have Duplicate Code
**Location:**
- `lib/generators/bakery-generator.ts`
- `lib/generators/hotel-generator.ts`
- `lib/generators/tech-startup-generator.ts`

**Issue:**
- Similar retry logic in all 3
- Similar prompt handling
- Similar error handling
- Should be abstracted

**Action:**
- Create `lib/generators/base-generator.ts` with shared logic
- Extract common patterns
- DRY up the code

**Technical Debt Cost:** Medium (refactoring 3 large files)

**Note:** Works fine now, can defer to post-MVP

---

### DEBT-011: No Database Migrations Strategy
**Location:** `prisma/migrations/`

**Issue:**
- Migrations created ad-hoc
- No rollback strategy
- Not tested on production clone first
- Auto-migrate on build (good) but risky

**Action:**
- Document migration process
- Test migrations on staging DB first
- Have rollback plan for each migration
- Consider seeding strategy for dev/test

**Technical Debt Cost:** Low (process documentation)

---

### DEBT-012: Frontend Bundle Size Not Optimized
**Location:** Next.js build output

**Issue:**
- Haven't analyzed bundle size
- Might be importing large libraries unused
- No code splitting strategy
- No lazy loading

**Action:**
- Run `next build` and analyze bundle
- Use `@next/bundle-analyzer`
- Lazy load heavy components
- Tree-shake unused imports

**Technical Debt Cost:** Medium (requires analysis + optimization)

**Note:** Defer to post-MVP - performance seems fine

---

### DEBT-013: No Monitoring or Observability
**Location:** Production

**Issue:**
- No error tracking (Sentry, etc.)
- No performance monitoring
- No user analytics
- Hard to know if things break in production

**Action (Future):**
- Add Sentry for error tracking
- Add PostHog or Mixpanel for analytics
- Vercel's built-in monitoring
- Set up alerts for critical errors

**Technical Debt Cost:** Medium (integration work)

**Note:** Defer to post-launch

---

## 🟢 Low Priority / Nice-to-Have

### DEBT-014: No E2E Tests
**Location:** Tests

**Issue:**
- Only unit tests exist (if any)
- No Playwright/Cypress E2E tests
- No automated user flow testing

**Action (Future):**
- Set up Playwright
- Write critical path tests
- Run in CI/CD

**Technical Debt Cost:** High (test infrastructure + writing tests)

---

### DEBT-015: No CI/CD Pipeline
**Location:** GitHub / Vercel

**Issue:**
- Relying on Vercel auto-deploy only
- No GitHub Actions for tests
- No automated checks before merge
- No preview deployments being used

**Action (Future):**
- Set up GitHub Actions
- Run tests on PR
- Require passing tests before merge
- Use Vercel preview deployments more

**Technical Debt Cost:** Medium (CI/CD setup)

---

### DEBT-016: Magic Numbers and Strings
**Location:** Throughout codebase

**Examples:**
```typescript
// Should be constants
if (progress > 90) { ... }
setTimeout(() => { ... }, 800)
'2024-01-01'
'techStartup'
```

**Action:**
- Extract to constants
- Use enums for string literals
- Document magic numbers

**Technical Debt Cost:** Low (refactoring)

---

## Code Quality Issues

### Missing
- ❌ ESLint rules not strict enough
- ❌ Prettier not configured
- ❌ Pre-commit hooks missing
- ❌ Code review guidelines

### Existing (Good)
- ✅ TypeScript configured
- ✅ Prisma for type-safe DB
- ✅ Next.js best practices mostly followed

---

## Architecture Improvements (Future)

### Consider
1. **Separate backend API:** Move heavy logic out of Next.js API routes (future scale)
2. **Caching layer:** Redis for scenario results (future performance)
3. **CDN for results:** Store generated scenarios in S3 + CloudFront (future scale)
4. **WebSocket support:** Real-time updates instead of polling (future UX)
5. **Multi-tenancy:** Proper user isolation if we add auth (future feature)

---

## Security Audit Items

### To Review
1. SQL injection via Prisma (should be safe but verify)
2. XSS in results display (sanitize event data)
3. CSRF protection (Next.js handles this)
4. Rate limiting (once Redis configured)
5. API key rotation strategy
6. Webhook signature verification (Stripe)
7. QStash signature verification (NEW - must implement)

---

## Performance Optimization Opportunities

### Backend
1. Parallel Claude API calls where possible (risky, need ordering)
2. Cache master data generation (Stage 1 could be reused)
3. Database query optimization (add indexes if needed)
4. Reduce Stage 3 API calls (bi-monthly instead of monthly?)

### Frontend
1. Lazy load results charts (only when visible)
2. Virtualize long lists
3. Optimize images (if any)
4. Reduce JavaScript bundle size

**Note:** Don't optimize prematurely - measure first

---

## Documentation Debt

### Missing Docs
- ❌ API documentation (endpoint specs)
- ❌ Database schema docs (beyond Prisma)
- ❌ Deployment guide (step-by-step)
- ❌ Contributing guide
- ❌ Environment setup guide
- ❌ Troubleshooting guide

### Existing Docs (Good)
- ✅ PRD exists
- ✅ Architecture docs exist (being updated)
- ✅ This technical debt doc (meta!)

---

## Dependencies to Review

### Outdated or Risky
- Check `npm audit` for vulnerabilities
- Review major version updates available
- Consider alternatives to heavy dependencies

### Current Stack (Good)
- Next.js 14 (latest)
- React 18 (latest)
- Prisma 6 (latest)
- TypeScript 5 (latest)
- Tailwind 3 (latest)

---

## Post-QStash Implementation Cleanup

**Once QStash is working, immediately clean up:**

1. Remove synchronous generation code from `app/api/generate/route.ts`
2. Delete or repurpose unused routes
3. Update all documentation to reflect new architecture
4. Remove commented-out code
5. Remove debug logs (keep only meaningful logs)
6. Run `pnpm lint` and fix all warnings
7. Run `pnpm test` and ensure all pass

---

## Commitment to Code Quality

**Going Forward:**

1. ✅ **No more quick fixes** - Do it right or don't do it
2. ✅ **Documentation first** - Write the doc before the code
3. ✅ **Test as we go** - Don't accumulate test debt
4. ✅ **Review before merge** - At least one other agent reviews
5. ✅ **Refactor when adding features** - Leave code better than we found it

---

## Estimated Cleanup Time

**Critical Debt (During Rebuild):** ~2-3 hours
**High Priority (Post-Rebuild):** ~4-6 hours
**Medium Priority (Post-MVP):** ~8-12 hours
**Low Priority (Ongoing):** ~20+ hours

**Total:** ~34-41 hours of technical debt

**Note:** This is normal for a fast-moving MVP. The key is managing it, not eliminating it.

---

**James's Commitment:** I'll clean up as I implement QStash. No more duct tape and string.
