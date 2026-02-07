# ChainSights Testing Strategy

> Last updated: 2026-02-07

This document defines the testing strategy, conventions, tooling, and coverage goals for the ChainSights project. It is the reference for all developers and AI agents writing or maintaining tests.

---

## Table of Contents

1. [Test Pyramid](#test-pyramid)
2. [Tools and Frameworks](#tools-and-frameworks)
3. [Directory Structure and Conventions](#directory-structure-and-conventions)
4. [Running Tests](#running-tests)
5. [Writing Unit Tests](#writing-unit-tests)
6. [Writing Integration Tests](#writing-integration-tests)
7. [Writing E2E Tests](#writing-e2e-tests)
8. [Test Fixtures and Utilities](#test-fixtures-and-utilities)
9. [Mocking Patterns](#mocking-patterns)
10. [Coverage Goals](#coverage-goals)
11. [CI Integration Guidance](#ci-integration-guidance)
12. [Current Coverage Gaps and Recommendations](#current-coverage-gaps-and-recommendations)

---

## Test Pyramid

ChainSights follows the standard test pyramid with three tiers:

```
        /  E2E  \          ~5%   Playwright (full user flows, accessibility)
       /----------\
      / Integration \      ~15%  Vitest + real DB (API routes, query performance)
     /----------------\
    /    Unit Tests     \  ~80%  Vitest + jsdom (lib/, components, API handlers)
   /____________________\
```

**Principle:** The majority of tests should be fast, isolated unit tests. Integration tests validate database queries and cross-module behavior. E2E tests cover critical user journeys and accessibility compliance.

| Tier | Count | Runner | Speed | Dependencies |
|------|-------|--------|-------|--------------|
| Unit | ~72 files | Vitest | <30s total | None (all mocked) |
| Integration | 3 files | Vitest | ~10s (requires DB) | PostgreSQL (Neon) |
| E2E | 1 file | Playwright | ~60s (requires running app) | Next.js dev server, Chromium |

---

## Tools and Frameworks

### Unit + Integration Testing

| Tool | Version | Purpose |
|------|---------|---------|
| **Vitest** | 4.0.16 | Test runner, assertion library, mocking |
| **@testing-library/react** | 16.3.1 | React component testing (render, queries) |
| **@testing-library/jest-dom** | 6.9.1 | Custom DOM matchers (toBeInTheDocument, toHaveClass) |
| **@vitejs/plugin-react** | 5.1.2 | JSX/TSX compilation in tests |
| **@vitest/coverage-v8** | 4.0.16 | Code coverage via V8 |
| **jsdom** | 27.3.0 | Browser environment simulation |

### E2E Testing

| Tool | Version | Purpose |
|------|---------|---------|
| **Playwright** | 1.57.0 | Browser automation, E2E flows |
| **@axe-core/playwright** | 4.11.0 | Accessibility auditing (WCAG 2.1 AA) |

### Additional

| Tool | Purpose |
|------|---------|
| **Lighthouse CI** (`@lhci/cli` 0.15.1) | Performance auditing via `npm run lighthouse` |

---

## Directory Structure and Conventions

```
tests/
  setup.ts                        # Global test setup (imports jest-dom matchers)
  fixtures/
    orders.ts                     # Factory: createTestOrder()
    reports.ts                    # Factory: createTestReport()
  utils/
    accessibility.ts              # checkAccessibility() helper for Playwright
  unit/
    gvs/
      dei.test.ts                 # Mirrors src/lib/gvs/dei.ts
      hpr.test.ts                 # Mirrors src/lib/gvs/hpr.ts
      pdi.test.ts                 # Mirrors src/lib/gvs/pdi.ts
      gpi.test.ts                 # Mirrors src/lib/gvs/gpi.ts
      aggregate.test.ts           # Mirrors src/lib/gvs/aggregate.ts
      storage.test.ts             # Mirrors src/lib/gvs/storage.ts
      anomaly-detection.test.ts   # Mirrors src/lib/gvs/anomaly-detection.ts
    components/
      change-indicator.test.tsx   # Component rendering tests
      rankings-table.test.tsx     # Component rendering tests
      skip-link.test.tsx          # Accessibility component test
      ...
    lib/
      validation/
        opt-out.test.ts           # Mirrors src/lib/validation/opt-out.ts
        sanitize.test.ts          # Mirrors src/lib/validation/sanitize.ts
      dao/
        featured.test.ts          # Mirrors src/lib/dao/featured.ts
        opt-out.test.ts           # Mirrors src/lib/dao/opt-out.ts
      gdpr/
        retention-cleanup.test.ts # Mirrors src/lib/gdpr/retention-cleanup.ts
      analytics.test.ts           # Mirrors src/lib/analytics.ts
      pricing-config.test.ts      # Mirrors src/lib/config/pricing.ts
      ...
    api/
      rankings/
        opt-out.test.ts           # Tests POST /api/rankings/opt-out handler
      admin/
        job-logs.test.ts          # Tests admin API routes
        opt-out-process.test.ts
        recalculate.test.ts
        daos.test.ts
        featured.test.ts
        opt-out-all.test.ts
      ...
    middleware/
      rate-limit.test.ts          # Mirrors src/lib/middleware/rate-limit.ts
      middleware-integration.test.ts
    monitoring/
      sentry-scrubbing.test.ts    # Mirrors src/lib/monitoring/sentry-scrubbing.ts
    jobs/
      job-logs.test.ts            # Mirrors src/lib/jobs/
      monthly-aggregation.test.ts
      snapshot-cleanup.test.ts
    app/
      rankings/methodology/page.test.tsx
      impact/page.test.tsx
      success/page.test.tsx
  integration/
    gvs/
      storage-performance.test.ts # NFR-PERF-9: <100ms for 50 DAOs
      aggregate-performance.test.ts
    rankings/
      page.test.ts                # Real DB query: leaderboard rankings
  e2e/
    accessibility.spec.ts         # WCAG 2.1 AA compliance via axe-core
```

### Naming Conventions

| Convention | Pattern | Example |
|------------|---------|---------|
| Unit test files | `{source-name}.test.ts` or `.test.tsx` | `dei.test.ts`, `change-indicator.test.tsx` |
| E2E test files | `{feature}.spec.ts` | `accessibility.spec.ts` |
| Test directory | Mirrors `src/` structure | `tests/unit/gvs/` mirrors `src/lib/gvs/` |
| Fixture files | Descriptive noun, in `tests/fixtures/` | `orders.ts`, `reports.ts` |
| Utility files | Descriptive function, in `tests/utils/` | `accessibility.ts` |

### Describe/It Naming

Use the **Given-When-Then** pattern for describe/it blocks in domain logic tests:

```typescript
describe('Given DAO with no proposals', () => {
  it('When calculateDEI is called, Then returns score 5 neutral with low confidence', async () => {
    // ...
  })
})
```

For component tests, use direct descriptive names:

```typescript
describe('ChangeIndicator', () => {
  describe('Positive Delta (Up)', () => {
    it('should display "+0.3" with up arrow for positive change', () => {
      // ...
    })
  })
})
```

---

## Running Tests

### NPM Scripts

```bash
# Unit tests in watch mode (default for development)
npm test

# Unit tests - single run
npm run test:unit

# Integration tests (requires DATABASE_URL)
npm run test:integration

# Coverage report (text + JSON + HTML)
npm run test:coverage

# E2E tests (requires running dev server or starts one automatically)
npm run test:e2e

# E2E tests with Playwright UI mode (interactive debugging)
npm run test:e2e:ui

# Performance audit
npm run lighthouse
```

### Running a Single Test File

```bash
# Run a specific unit test
npx vitest run tests/unit/gvs/dei.test.ts

# Run a specific E2E test
npx playwright test tests/e2e/accessibility.spec.ts

# Run tests matching a pattern
npx vitest run --reporter=verbose -t "calculateDEI"
```

---

## Writing Unit Tests

### Core Library Functions (src/lib/)

Unit tests for library functions mock all external dependencies and test pure logic.

**Example: GVS metric calculation (tests/unit/gvs/dei.test.ts)**

```typescript
import { describe, it, expect, vi, beforeEach } from 'vitest'
import { calculateDEI } from '@/lib/gvs/dei'
import type { SnapshotProposal, SnapshotVote, VoterAnalysis } from '@/lib/snapshot/types'

// Mock external dependencies
vi.mock('@/lib/snapshot/client', () => ({
  fetchProposals: vi.fn(),
  fetchVotesForProposal: vi.fn(),
  analyzeVoters: vi.fn(),
}))

import { fetchProposals, fetchVotesForProposal, analyzeVoters } from '@/lib/snapshot/client'

describe('calculateDEI', () => {
  beforeEach(() => {
    vi.clearAllMocks()
  })

  describe('Given DAO with no proposals', () => {
    it('When calculateDEI is called, Then returns score 5 neutral with low confidence', async () => {
      vi.mocked(fetchProposals).mockResolvedValue([])

      const result = await calculateDEI('test-dao.eth')

      expect(result.score).toBe(5)
      expect(result.weight).toBe(0.25)
      expect(result.confidence).toBe('Low')
      expect(result.metadata.proposalsAnalyzed).toBe(0)
    })
  })

  describe('Given score bounds validation', () => {
    it('When calculateDEI returns, Then score is always within 0-10 bounds', async () => {
      vi.mocked(fetchProposals).mockResolvedValue([])
      vi.mocked(fetchVotesForProposal).mockResolvedValue([])

      const result = await calculateDEI('test-dao.eth')

      expect(result.score).toBeGreaterThanOrEqual(0)
      expect(result.score).toBeLessThanOrEqual(10)
    })
  })
})
```

**Key patterns:**
- Import `describe`, `it`, `expect`, `vi`, `beforeEach` from `vitest`
- Use `vi.mock()` at module level to mock dependencies
- Use `vi.mocked()` to set return values on mocked functions
- Call `vi.clearAllMocks()` in `beforeEach` to reset state between tests
- Test boundary conditions (0, null, empty arrays, max values)
- Validate result structure with `toHaveProperty`

### React Component Tests

Component tests render components with `@testing-library/react` and assert on DOM output.

**Example: ChangeIndicator (tests/unit/components/change-indicator.test.tsx)**

```typescript
import { describe, it, expect } from 'vitest'
import { render, screen } from '@testing-library/react'
import ChangeIndicator from '@/components/ChangeIndicator'

describe('ChangeIndicator', () => {
  describe('Positive Delta (Up)', () => {
    it('should display "+0.3" with up arrow for positive change', () => {
      render(<ChangeIndicator delta={0.3} deltaDirection="up" />)

      expect(screen.getByText('+0.3')).toBeInTheDocument()
      expect(screen.getByText('\u2197')).toBeInTheDocument()
    })

    it('should have "Increased by X points" ARIA label', () => {
      render(<ChangeIndicator delta={0.3} deltaDirection="up" />)

      expect(screen.getByLabelText('Increased by 0.3 points')).toBeInTheDocument()
    })

    it('should use green text color for positive change', () => {
      const { container } = render(<ChangeIndicator delta={0.3} deltaDirection="up" />)
      const span = container.querySelector('span')

      expect(span).toHaveClass('text-green-700')
    })
  })

  describe('Accessibility', () => {
    it('should mark decorative arrows as aria-hidden', () => {
      render(<ChangeIndicator delta={0.3} deltaDirection="up" />)
      const arrow = screen.getByText('\u2197')

      expect(arrow).toHaveAttribute('aria-hidden', 'true')
    })
  })
})
```

**Key patterns:**
- Use `render()` from `@testing-library/react`
- Query elements with `screen.getByText()`, `screen.getByLabelText()`, `screen.getByRole()`
- Test accessibility attributes (ARIA labels, aria-hidden)
- Test CSS classes for visual styling
- Test edge cases (null props, empty strings, extreme values)
- Use `rerender()` for testing prop changes

### API Route Handler Tests

API route tests mock the database and external services, then call the route handler directly.

**Example: Opt-Out API (tests/unit/api/rankings/opt-out.test.ts)**

```typescript
import { describe, it, expect, vi, beforeEach } from 'vitest'
import { NextRequest } from 'next/server'

// Mock database
const mockInsert = vi.fn()
const mockValues = vi.fn()
const mockReturning = vi.fn()

vi.mock('@/lib/db', () => ({
  isDatabaseConfigured: vi.fn(() => true),
  getDb: vi.fn(() => ({
    insert: mockInsert,
  })),
}))

// Mock email
const mockSendEmail = vi.fn()
vi.mock('@/lib/email', () => ({
  isEmailConfigured: vi.fn(() => true),
  sendOptOutConfirmationEmail: (options: unknown) => mockSendEmail(options),
}))

import { POST } from '@/app/api/rankings/opt-out/route'

function createMockRequest(body: unknown): NextRequest {
  return new NextRequest('http://localhost:3000/api/rankings/opt-out', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify(body),
  })
}

describe('POST /api/rankings/opt-out', () => {
  beforeEach(() => {
    vi.clearAllMocks()
    mockInsert.mockReturnValue({ values: mockValues })
    mockValues.mockReturnValue({ returning: mockReturning })
    mockReturning.mockResolvedValue([{ id: 'test-request-id' }])
    mockSendEmail.mockResolvedValue({ id: 'test-email-id' })
  })

  it('should accept valid opt-out request', async () => {
    const request = createMockRequest({
      daoName: 'Uniswap',
      contactEmail: 'admin@uniswap.org',
      reason: 'We prefer not to be ranked.',
    })

    const response = await POST(request)
    const data = await response.json()

    expect(response.status).toBe(200)
    expect(data.success).toBe(true)
    expect(data.requestId).toBe('test-request-id')
  })

  it('should return 400 for invalid input', async () => {
    const request = createMockRequest({ daoName: '' })

    const response = await POST(request)
    expect(response.status).toBe(400)
  })

  it('should return 503 when database is not configured', async () => {
    const { isDatabaseConfigured } = await import('@/lib/db')
    vi.mocked(isDatabaseConfigured).mockReturnValue(false)

    const request = createMockRequest({
      daoName: 'TestDAO',
      contactEmail: 'test@example.com',
    })

    const response = await POST(request)
    expect(response.status).toBe(503)
  })
})
```

**Key patterns:**
- Use `NextRequest` constructor to create mock HTTP requests
- Mock the entire Drizzle ORM chain: `insert -> values -> returning`
- Test success path, validation errors, service unavailability, and database errors
- Verify email side effects with `toHaveBeenCalledWith`
- Test graceful degradation (email fails but request succeeds)

### Middleware and Security Tests

**Example: Rate Limiting (tests/unit/middleware/rate-limit.test.ts)**

```typescript
import { describe, it, expect, vi, beforeEach, afterEach } from 'vitest'
import { checkRateLimit, RATE_LIMIT, RATE_WINDOW_MS } from '@/lib/middleware/rate-limit'

describe('Rate Limiting', () => {
  let testStore: Map<string, { count: number; resetAt: number }>

  beforeEach(() => {
    testStore = new Map()
    vi.useFakeTimers()
  })

  afterEach(() => {
    testStore.clear()
    vi.useRealTimers()
  })

  it('should allow first request from new IP', () => {
    const result = checkRateLimit('192.168.1.1', testStore, Date.now())
    expect(result.allowed).toBe(true)
    expect(result.remaining).toBe(RATE_LIMIT - 1)
  })

  it('should block requests after limit is reached', () => {
    const now = Date.now()
    for (let i = 0; i < RATE_LIMIT; i++) {
      checkRateLimit('192.168.1.1', testStore, now)
    }
    const result = checkRateLimit('192.168.1.1', testStore, now)
    expect(result.allowed).toBe(false)
  })

  it('should reset counter after window expires', () => {
    const now = Date.now()
    for (let i = 0; i < RATE_LIMIT; i++) {
      checkRateLimit('192.168.1.1', testStore, now)
    }
    const result = checkRateLimit('192.168.1.1', testStore, now + RATE_WINDOW_MS + 1)
    expect(result.allowed).toBe(true)
  })
})
```

**Key patterns:**
- Use `vi.useFakeTimers()` / `vi.useRealTimers()` for time-dependent logic
- Inject test state (e.g., `testStore` Map) rather than relying on module-level singletons
- Test boundary conditions (just before expiry, exact expiry moment, after expiry)

---

## Writing Integration Tests

Integration tests run against a real PostgreSQL database (Neon). They require `DATABASE_URL` to be set.

**Example: Storage Performance (tests/integration/gvs/storage-performance.test.ts)**

```typescript
import { describe, it, expect, beforeAll } from 'vitest'
import { getLatestSnapshots } from '@/lib/gvs/storage'
import { getDb } from '@/lib/db'
import { gvsSnapshots } from '@/lib/db/schema'

describe('GVS Storage Performance (NFR-PERF-9)', () => {
  let testDaoIds: string[] = []

  beforeAll(async () => {
    try {
      const db = getDb()
      const snapshots = await db
        .select({ daoId: gvsSnapshots.daoId })
        .from(gvsSnapshots)
        .limit(50)
      testDaoIds = [...new Set(snapshots.map(s => s.daoId))]
    } catch (error) {
      console.warn('Database not configured - skipping')
    }
  })

  it('fetches latest snapshots for 50 DAOs in <100ms', async () => {
    if (testDaoIds.length === 0) return

    // Warm up
    await getLatestSnapshots(testDaoIds.slice(0, 5))

    // Measure
    const start = performance.now()
    const result = await getLatestSnapshots(testDaoIds)
    const duration = performance.now() - start

    expect(duration).toBeLessThan(100)
    expect(result.size).toBeGreaterThan(0)
  })
})
```

**Key patterns:**
- Gracefully skip tests when database is unavailable (no hard failures in CI without DB)
- Use `beforeAll` for expensive setup (DB connection, fetching test data)
- Performance tests: warm up the connection pool, then measure
- Use real Drizzle ORM queries -- do NOT mock the database in integration tests
- Verify data integrity with real DB assertions (DISTINCT ON returns latest, sort order is correct)

---

## Writing E2E Tests

E2E tests use Playwright with Chromium. The dev server starts automatically via the `webServer` config in `playwright.config.ts`.

**Example: Accessibility (tests/e2e/accessibility.spec.ts)**

```typescript
import { test, expect } from '@playwright/test'
import { checkAccessibility } from '../utils/accessibility'

test.describe('Accessibility - WCAG 2.1 Level AA', () => {
  test('homepage should be accessible', async ({ page }) => {
    await page.goto('/')
    await checkAccessibility(page, 'Homepage')
  })

  test('rankings leaderboard should be accessible', async ({ page }) => {
    await page.goto('/rankings')
    await page.waitForSelector('table', { timeout: 10000 })
    await checkAccessibility(page, 'Rankings Leaderboard')
  })
})

test.describe('Accessibility - Interactive Elements', () => {
  test('expandable DAO rows should be keyboard accessible', async ({ page }) => {
    await page.goto('/rankings')
    await page.waitForSelector('table tbody tr', { timeout: 10000 })

    await page.keyboard.press('Tab')
    await page.keyboard.press('Tab')

    const focusedElement = await page.evaluate(() => {
      const el = document.activeElement
      return { tagName: el?.tagName, tabIndex: el?.getAttribute('tabindex') }
    })

    expect(focusedElement.tagName).toMatch(/^(BUTTON|A|INPUT|SELECT|TEXTAREA|TR|TD|TH)$/)
  })
})
```

**Key patterns:**
- Use `page.goto()` with relative paths (baseURL configured in playwright.config.ts)
- Wait for dynamic content with `page.waitForSelector()` before assertions
- Use the shared `checkAccessibility()` utility for WCAG compliance checks
- Test keyboard navigation with `page.keyboard.press()`
- Test on Chromium by default; expand to Firefox/WebKit if needed

### Playwright Configuration Highlights

```typescript
// playwright.config.ts
export default defineConfig({
  testDir: './tests/e2e',
  fullyParallel: true,
  forbidOnly: !!process.env.CI,       // Fail CI if .only() is left in
  retries: process.env.CI ? 2 : 0,    // Retry flaky tests in CI
  workers: process.env.CI ? 1 : undefined,
  reporter: 'html',
  use: {
    baseURL: process.env.BASE_URL || 'http://localhost:3000',
    trace: 'on-first-retry',          // Capture trace on failure for debugging
  },
  projects: [
    { name: 'chromium', use: { ...devices['Desktop Chrome'] } },
  ],
  webServer: {
    command: 'npm run dev',
    url: 'http://localhost:3000',
    reuseExistingServer: !process.env.CI,
  },
})
```

---

## Test Fixtures and Utilities

### Test Factories (tests/fixtures/)

Factories produce test data with sensible defaults and allow overrides.

**orders.ts:**
```typescript
import { randomBytes } from 'crypto'

export interface TestOrder {
  stripeSessionId: string
  stripePaymentIntent: string | null
  stripeCustomerId: string | null
  customerEmail: string
  customerName: string | null
  tier: 'standard' | 'deep_dive'
  status: 'pending' | 'paid' | 'processing' | 'delivered' | 'failed'
  amountCents: number
  currency: string
}

export function createTestOrder(overrides: Partial<TestOrder> = {}): TestOrder {
  const id = randomBytes(4).toString('hex')
  return {
    stripeSessionId: `cs_test_${id}`,
    stripePaymentIntent: `pi_test_${id}`,
    stripeCustomerId: `cus_test_${id}`,
    customerEmail: 'test@example.com',
    customerName: 'Test Customer',
    tier: 'standard',
    status: 'paid',
    amountCents: 9900,
    currency: 'eur',
    ...overrides,
  }
}
```

**Usage pattern:** Always use factories instead of inline object literals so that test data stays consistent and easy to maintain.

### Accessibility Utility (tests/utils/accessibility.ts)

```typescript
import AxeBuilder from '@axe-core/playwright'
import { Page, expect } from '@playwright/test'

export async function checkAccessibility(page: Page, pageName: string) {
  const results = await new AxeBuilder({ page })
    .withTags(['wcag2a', 'wcag2aa', 'wcag21a', 'wcag21aa'])
    .analyze()

  expect(results.violations,
    `${pageName} should not have accessibility violations`
  ).toEqual([])

  return results
}
```

### Global Setup (tests/setup.ts)

```typescript
import { beforeAll, afterAll } from 'vitest'
import '@testing-library/jest-dom'

beforeAll(async () => {
  console.log('[Test] Global setup complete')
})

afterAll(async () => {
  console.log('[Test] Global cleanup complete')
})
```

The setup file is loaded via `setupFiles` in `vitest.config.ts` and imports `@testing-library/jest-dom` to make matchers like `toBeInTheDocument()` and `toHaveClass()` available globally.

---

## Mocking Patterns

### Module Mocking

```typescript
// Mock an entire module
vi.mock('@/lib/db', () => ({
  isDatabaseConfigured: vi.fn(() => true),
  getDb: vi.fn(() => ({ insert: vi.fn() })),
}))

// Mock with partial real implementation
vi.mock('@/lib/validation/opt-out', async () => {
  const actual = await vi.importActual('@/lib/validation/opt-out')
  return actual  // Use real validation, only mock if needed
})
```

### Function Mocking

```typescript
// Set return value
vi.mocked(fetchProposals).mockResolvedValue([])

// Dynamic implementation
vi.mocked(fetchVotesForProposal).mockImplementation(async (proposalId: string) => {
  return votes.filter(v => v.id.startsWith(proposalId))
})

// Verify calls
expect(mockSendEmail).toHaveBeenCalledWith({
  to: 'admin@uniswap.org',
  daoName: 'Uniswap',
  requestId: 'test-request-id',
})
```

### Timer Mocking

```typescript
beforeEach(() => {
  vi.useFakeTimers()
})

afterEach(() => {
  vi.useRealTimers()
})

// Advance time
vi.advanceTimersByTime(60000)
```

### NextRequest Mocking

```typescript
function createMockRequest(body: unknown): NextRequest {
  return new NextRequest('http://localhost:3000/api/endpoint', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify(body),
  })
}
```

---

## Coverage Goals

### Vitest Coverage Configuration

From `vitest.config.ts`:

```typescript
coverage: {
  provider: 'v8',
  reporter: ['text', 'json', 'html'],
  include: ['src/**/*.ts', 'src/**/*.tsx'],
  exclude: ['src/**/*.d.ts', 'src/app/**/page.tsx'],
}
```

### Target Coverage

| Scope | Target | Rationale |
|-------|--------|-----------|
| `src/lib/` | **80% minimum** | Core business logic, calculations, and utilities |
| `src/lib/gvs/` | **90%+** | GVS scoring is the core product -- highest priority |
| `src/lib/validation/` | **90%+** | Input validation protects against security issues |
| `src/lib/middleware/` | **80%+** | Rate limiting, auth guards |
| `src/components/` | **70%** | UI components with behavior/accessibility |
| `src/app/api/` | **70%** | API route handlers (tested via unit mocking) |
| `src/app/**/page.tsx` | **Excluded** | Server Components are excluded from Vitest coverage (tested via E2E) |

### Generating Coverage Reports

```bash
npm run test:coverage
```

This produces:
- **Console output** (text summary)
- **coverage/index.html** (interactive HTML report)
- **coverage/coverage-final.json** (machine-readable for CI)

---

## CI Integration Guidance

### Recommended CI Pipeline

ChainSights deploys on Vercel. There is currently no dedicated CI workflow for tests. The following is the recommended GitHub Actions configuration:

```yaml
# .github/workflows/test.yml
name: Test

on:
  push:
    branches: [master]
  pull_request:
    branches: [master]

jobs:
  unit-tests:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: 'npm'
      - run: npm ci
      - run: npm run test:unit
      - run: npm run test:coverage
      - name: Check coverage threshold
        run: |
          # Parse coverage JSON and fail if lib/ is below 80%
          node -e "
            const coverage = require('./coverage/coverage-final.json');
            // Add threshold check logic here
          "

  integration-tests:
    runs-on: ubuntu-latest
    env:
      DATABASE_URL: ${{ secrets.TEST_DATABASE_URL }}
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: 'npm'
      - run: npm ci
      - run: npm run test:integration

  e2e-tests:
    runs-on: ubuntu-latest
    env:
      DATABASE_URL: ${{ secrets.TEST_DATABASE_URL }}
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: 'npm'
      - run: npm ci
      - run: npx playwright install --with-deps chromium
      - run: npm run test:e2e
      - uses: actions/upload-artifact@v4
        if: failure()
        with:
          name: playwright-report
          path: playwright-report/
```

### Pre-Commit Testing

Per the project workflow rules in `docs/project_context.md`:

```bash
# Run before every commit
npm test && npm run test:e2e
```

### CI Environment Variables

| Variable | Required For | Description |
|----------|-------------|-------------|
| `DATABASE_URL` | Integration, E2E | Neon PostgreSQL connection string (test database) |
| `CI` | All | Set automatically by GitHub Actions; affects retries, workers, forbidOnly |
| `BASE_URL` | E2E (optional) | Overrides default `http://localhost:3000` |

---

## Current Coverage Gaps and Recommendations

### Well-Covered Areas (72 unit test files)

- **GVS Scoring Engine** (`src/lib/gvs/`): All 4 metric calculations (DEI, HPR, PDI, GPI) plus aggregate and storage -- thorough coverage with boundary tests
- **Validation** (`src/lib/validation/`): Opt-out and sanitization validation
- **Components** (`src/components/`): ~25 component test files covering rankings, admin, donations, sparklines, banners, and accessibility
- **API Routes**: Opt-out, admin CRUD, donations, webhooks, crypto checkout
- **Middleware**: Rate limiting with exhaustive boundary tests
- **Monitoring**: Sentry PII scrubbing
- **GDPR**: Data retention cleanup

### Coverage Gaps (Recommended Additions)

#### High Priority

| Area | Files Missing Tests | Impact |
|------|-------------------|--------|
| **Auth / Magic Link** | `src/lib/auth/` -- session creation, JWT verification, middleware guards | Security-critical: session handling, token expiry, role-based access |
| **Stripe Integration** | `src/lib/stripe.ts`, `src/app/api/webhooks/stripe/route.ts` (payment handlers) | Revenue-critical: payment processing, webhook signature verification |
| **Email System** | `src/lib/email/` -- report delivery, recovery emails, share reminders | Customer-facing: email content, conditional sending based on feature flags |
| **/check Flow** | `src/app/check/` components -- progressive disclosure, tier selection, form validation | Primary conversion funnel |
| **Quick Check API** | `src/app/api/quick-check/route.ts` -- three-path lookup (L3/L2/on-the-fly), rate limiting per email | Core free product; complex branching logic |

#### Medium Priority

| Area | Files Missing Tests | Impact |
|------|-------------------|--------|
| **PDF Generation** | `src/lib/pdf/report-template.tsx` -- tier-aware content, DGI benchmarks | Report quality; difficult to test but high customer-facing impact |
| **AI Analysis** | `src/lib/ai/` -- prompt construction, response parsing | Expensive API calls; mock Anthropic SDK and validate prompt structure |
| **DGI / Benchmark** | `src/lib/dgi/` -- index calculation, category averages | New feature (2026-02-06); needs coverage as methodology evolves |
| **Signal Scanner** | `src/lib/signals/` -- relevance scoring | New area; ensure scoring logic is correct |
| **Matrix Page** | `src/app/matrix/` -- table rendering, detail pages, chart display | Public-facing dashboard; now fully free |
| **Feature Flags** | `src/lib/feature-flags.ts` -- flag evaluation logic | Controls feature visibility across the app |

#### Low Priority (Nice to Have)

| Area | Recommendation |
|------|---------------|
| **E2E: User Flows** | Add E2E tests for: `/check` conversion flow, `/matrix` navigation, `/pricing` page display |
| **E2E: Mobile** | Add Playwright `iPhone` device profile for mobile hamburger menu testing |
| **Visual Regression** | Consider Playwright screenshot comparison for key pages |
| **Load Testing** | Validate API endpoints under concurrent load (k6 or Artillery) |
| **Database Migrations** | Test migration scripts in CI against a fresh database |

### Action Items

1. **Create `tests/unit/lib/auth/` directory** and add tests for JWT creation, verification, session middleware, and role-based guards. This is the single highest-impact gap.

2. **Add Stripe webhook handler tests** covering `payment_intent.succeeded`, `checkout.session.completed`, and refund events with signature verification.

3. **Add Quick Check API tests** covering the three-path lookup logic (daos table, reportedDaos cache, on-the-fly GVS), rate limiting per email, and confidence badge assignment.

4. **Expand E2E test suite** from 1 file (accessibility only) to cover the primary user journeys: homepage -> /check -> Free Check result, and homepage -> /pricing -> checkout.

5. **Set up CI test workflow** in `.github/workflows/test.yml` to run unit tests on every push and integration/E2E tests on pull requests to master.

---

## Vitest Configuration Reference

The complete Vitest configuration from `vitest.config.ts`:

```typescript
import { defineConfig } from 'vitest/config'
import react from '@vitejs/plugin-react'
import path from 'path'

export default defineConfig({
  plugins: [react()],
  test: {
    globals: true,                                    // No need to import describe/it/expect
    environment: 'jsdom',                             // Simulates browser DOM
    include: ['tests/**/*.test.ts', 'tests/**/*.test.tsx'],
    exclude: ['tests/e2e/**'],                        // E2E handled by Playwright
    coverage: {
      provider: 'v8',
      reporter: ['text', 'json', 'html'],
      include: ['src/**/*.ts', 'src/**/*.tsx'],
      exclude: ['src/**/*.d.ts', 'src/app/**/page.tsx'],
    },
    setupFiles: ['./tests/setup.ts'],                 // Global setup + jest-dom matchers
  },
  resolve: {
    alias: {
      '@': path.resolve(__dirname, './src'),           // Matches tsconfig paths
    },
  },
})
```

**Notable settings:**
- `globals: true` -- `describe`, `it`, `expect`, `vi` are available without import (though explicit imports are recommended for clarity and used throughout the codebase)
- `environment: 'jsdom'` -- All tests run in a simulated browser environment
- `exclude: ['tests/e2e/**']` -- E2E tests are excluded from Vitest and run via Playwright
- `setupFiles` -- Loads `@testing-library/jest-dom` matchers globally
- `alias: '@'` -- Matches the `@/*` path alias from `tsconfig.json`
