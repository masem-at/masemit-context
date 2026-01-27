# Story 11.9: Donations Total API Endpoint

Status: done

## Story

**As a** frontend
**I want** an API to get cumulative donation total
**So that** I can display it on the impact page

## Acceptance Criteria

1. - [x] `GET /api/donations/total` returns total and count
2. - [x] Filters by project (default: chainsights)
3. - [x] Returns currency
4. - [x] Caches result for performance (optional)

## Tasks / Subtasks

- [x] Task 1: Create API route (AC: #1, #2, #3)
  - [x] 1.1 Create `src/app/api/donations/total/route.ts`
  - [x] 1.2 Import `getDonationTotal()` from donations queries library
  - [x] 1.3 Parse `project` query param with default 'chainsights'
  - [x] 1.4 Call `getDonationTotal(project)`
  - [x] 1.5 Return JSON with total, count, currency, lastUpdated
- [x] Task 2: Add response formatting (AC: #1, #3)
  - [x] 2.1 Format total to 2 decimal places
  - [x] 2.2 Include `currency: "EUR"` in response
  - [x] 2.3 Add `lastUpdated` timestamp (current time)
- [x] Task 3: Add caching (AC: #4)
  - [x] 3.1 Set `Cache-Control` header for 5-minute cache
  - [x] 3.2 Consider Vercel edge caching for performance
- [x] Task 4: Write unit tests
  - [x] 4.1 Test successful response with correct shape
  - [x] 4.2 Test default project filter (chainsights)
  - [x] 4.3 Test custom project filter via query param
  - [x] 4.4 Test response includes currency
  - [x] 4.5 Test cache header is set
  - [x] 4.6 Test error handling for database failures

## Dev Notes

### CRITICAL CONSTRAINT

> **ALL CODE CHANGES GO TO `develop` BRANCH ONLY!**

### API Contract (from Epic)

```typescript
// Request
GET /api/donations/total?project=chainsights

// Response
{
  total: 1247.53,
  count: 127,
  currency: "EUR",
  lastUpdated: "2026-01-19T12:00:00Z"
}
```

### Existing Dependencies

The `getDonationTotal()` function already exists in `src/lib/donations/queries.ts`:

```typescript
export interface DonationTotalResult {
  total: number
  count: number
}

export async function getDonationTotal(project: string): Promise<DonationTotalResult> {
  const db = getDb()
  const result = await db
    .select({
      total: sql<string>`COALESCE(SUM(${donations.donationAmount}), 0)`,
      count: sql<number>`COUNT(*)`,
    })
    .from(donations)
    .where(eq(donations.project, project))

  return {
    total: Number(result[0].total),
    count: Number(result[0].count),
  }
}
```

### Implementation Pattern

Follow existing API route patterns in the codebase:

```typescript
// src/app/api/donations/total/route.ts
import { NextResponse } from 'next/server'
import { getDonationTotal } from '@/lib/donations/queries'

export async function GET(request: Request) {
  const { searchParams } = new URL(request.url)
  const project = searchParams.get('project') || 'chainsights'

  try {
    const result = await getDonationTotal(project)

    return NextResponse.json(
      {
        total: result.total,
        count: result.count,
        currency: 'EUR',
        lastUpdated: new Date().toISOString(),
      },
      {
        headers: {
          'Cache-Control': 'public, s-maxage=300, stale-while-revalidate=60',
        },
      }
    )
  } catch (error) {
    console.error('Error fetching donation total:', error)
    return NextResponse.json(
      { error: 'Failed to fetch donation total' },
      { status: 500 }
    )
  }
}
```

### Caching Strategy

- **s-maxage=300**: Cache at CDN/edge for 5 minutes
- **stale-while-revalidate=60**: Serve stale while revalidating for 1 minute
- This reduces database load while keeping data reasonably fresh
- The donation total doesn't need real-time accuracy for the impact page

### Testing Patterns

Follow the test patterns from Story 11-8 and existing API tests:

```typescript
// tests/unit/api/donations-total.test.ts
import { describe, it, expect, vi, beforeEach } from 'vitest'
import { GET } from '@/app/api/donations/total/route'

// Mock the donations queries
vi.mock('@/lib/donations/queries', () => ({
  getDonationTotal: vi.fn(),
}))

import { getDonationTotal } from '@/lib/donations/queries'

describe('GET /api/donations/total', () => {
  beforeEach(() => {
    vi.clearAllMocks()
  })

  it('returns donation total with correct shape', async () => {
    ;(getDonationTotal as ReturnType<typeof vi.fn>).mockResolvedValue({
      total: 1247.53,
      count: 127,
    })

    const request = new Request('http://localhost/api/donations/total')
    const response = await GET(request)
    const data = await response.json()

    expect(data.total).toBe(1247.53)
    expect(data.count).toBe(127)
    expect(data.currency).toBe('EUR')
    expect(data.lastUpdated).toBeDefined()
  })

  it('uses default project filter', async () => {
    ;(getDonationTotal as ReturnType<typeof vi.fn>).mockResolvedValue({
      total: 0,
      count: 0,
    })

    const request = new Request('http://localhost/api/donations/total')
    await GET(request)

    expect(getDonationTotal).toHaveBeenCalledWith('chainsights')
  })

  it('accepts custom project filter', async () => {
    ;(getDonationTotal as ReturnType<typeof vi.fn>).mockResolvedValue({
      total: 50,
      count: 5,
    })

    const request = new Request('http://localhost/api/donations/total?project=other')
    await GET(request)

    expect(getDonationTotal).toHaveBeenCalledWith('other')
  })

  it('sets cache headers', async () => {
    ;(getDonationTotal as ReturnType<typeof vi.fn>).mockResolvedValue({
      total: 0,
      count: 0,
    })

    const request = new Request('http://localhost/api/donations/total')
    const response = await GET(request)

    expect(response.headers.get('Cache-Control')).toContain('s-maxage=300')
  })

  it('handles database errors gracefully', async () => {
    ;(getDonationTotal as ReturnType<typeof vi.fn>).mockRejectedValue(
      new Error('Database error')
    )

    const request = new Request('http://localhost/api/donations/total')
    const response = await GET(request)

    expect(response.status).toBe(500)
    const data = await response.json()
    expect(data.error).toBeDefined()
  })
})
```

### Previous Story Learnings (Story 11-8)

1. **Shared constants:** Use `src/lib/constants.ts` for any shared values
2. **Integer math precision:** Use `Math.round()` for decimal calculations
3. **Aria-labels:** Add accessibility where applicable
4. **Error handling:** Distinguish network vs API errors
5. **Test for edge cases:** Include zero, invalid, and error scenarios

### Files to Create

- `src/app/api/donations/total/route.ts` - API endpoint
- `tests/unit/api/donations-total.test.ts` - Unit tests

### Files to Modify

None - this is a new isolated endpoint.

### Dependencies

- **Story 11-2:** Donation queries library - DONE (provides `getDonationTotal()`)
- **Story 11-1:** Database schema - DONE (provides `donations` table)

### References

- [Epic: Story 9](./epic-crypto-payments-donations.md#story-9-donations-total-api-endpoint)
- [Architecture Doc](../architecture/crypto-payments-donations-architecture.md)
- [Existing Queries](../../src/lib/donations/queries.ts)

## Dev Agent Record

### Context Reference

Story created by create-story workflow with context from:
- Epic file (docs/sprint-artifacts/epic-crypto-payments-donations.md)
- Architecture doc (docs/architecture/crypto-payments-donations-architecture.md)
- Previous story (11-8) completion notes
- Existing donations queries library (src/lib/donations/queries.ts)
- Existing unit tests pattern (tests/unit/lib/donations-queries.test.ts)

### Agent Model Used

Claude Opus 4.5

### Debug Log References

### Completion Notes List

1. **API Route Created**: `src/app/api/donations/total/route.ts` - Simple GET endpoint that calls existing `getDonationTotal()` query function
2. **Response Format**: Returns `{ total, count, currency: "EUR", lastUpdated: ISO timestamp }`
3. **Project Filter**: Accepts `?project=` query param, defaults to 'chainsights'
4. **Caching Implemented**: `Cache-Control: public, s-maxage=300, stale-while-revalidate=60` for 5-minute CDN caching
5. **Error Handling**: Returns 500 with `{ error: "Failed to fetch donation total" }` on database errors
6. **Unit Tests**: 8 tests covering response shape, project filtering, caching, and error handling

### File List

- `src/app/api/donations/total/route.ts` (new) - Donations total API endpoint
- `tests/unit/api/donations-total.test.ts` (new) - 8 unit tests
- `docs/sprint-artifacts/11-9-donations-total-api-endpoint.md` (modified) - Story file

## Senior Developer Review

**Reviewed by:** Code Review Workflow
**Date:** 2026-01-21
**Verdict:** APPROVED with fixes applied

### Issues Found & Fixed:
1. **MEDIUM - Task 2.1 decimal formatting not implemented:** Code returned raw `result.total` without formatting. Fixed by adding `Number(result.total.toFixed(2))` to ensure consistent 2 decimal places.
2. **MEDIUM - Stale comments in test file:** Removed obsolete comments about "test will fail until route created" that were left from TDD development.
3. **MEDIUM - Story file not in File List:** Added to File List for completeness.

### Issues Noted (Low - Not Fixed):
4. **LOW - Redundant test:** Test 4 "returns currency in response" duplicates assertion from Test 1. Minor redundancy, acceptable for clarity.
5. **LOW - No input validation on project param:** Internal API, low risk. Could add validation in future if endpoint becomes public.

### Tests After Review:
- 8 unit tests passing
- Build: ✅ Verified

## Change Log

- 2026-01-21: Story created by Bob (Scrum Master) via create-story workflow
- 2026-01-21: Implementation completed by Dev Agent (Claude Opus 4.5) - all 8 tests passing, build verified
- 2026-01-21: Code review completed - 3 MEDIUM issues found, all fixed automatically
