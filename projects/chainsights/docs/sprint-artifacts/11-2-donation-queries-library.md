# Story 11.2: Donation Queries Library

Status: Done

## Story

**As a** developer
**I want** reusable functions for donation data operations
**So that** I can easily create and query donation records

## Acceptance Criteria

1. - [x] `createDonation()` function creates donation record
2. - [x] `getDonationTotal()` returns sum for a project
3. - [x] `getLastWebhookTimestamp()` returns last donation timestamp by provider
4. - [x] All functions have TypeScript types
5. - [x] Unit tests pass (9 tests)

## Tasks / Subtasks

- [x] Task 1: Create donation queries module (AC: #1, #2, #3, #4)
  - [x] 1.1 Create `src/lib/donations/queries.ts`
  - [x] 1.2 Implement `createDonation()`
  - [x] 1.3 Implement `getDonationTotal()`
  - [x] 1.4 Implement `getLastWebhookTimestamp()`
  - [x] 1.5 Create index.ts export
- [x] Task 2: Write unit tests (AC: #5)
  - [x] 2.1 Create `tests/unit/lib/donations-queries.test.ts`
  - [x] 2.2 Test createDonation
  - [x] 2.3 Test getDonationTotal
  - [x] 2.4 Test getLastWebhookTimestamp
  - [x] 2.5 Run tests - All 9 tests pass

## Dev Notes

### Critical Constraint
> **ALL CODE CHANGES GO TO `develop` BRANCH ONLY!**

### Function Specifications (from Architecture)

```typescript
// src/lib/donations/queries.ts

import { db } from '@/lib/db'
import { donations, DonationInsert } from '@/lib/db/schema'
import { eq, sql } from 'drizzle-orm'

/**
 * Create a new donation record
 */
export async function createDonation(data: DonationInsert): Promise<string> {
  const [result] = await db.insert(donations).values(data).returning({ id: donations.id })
  return result.id
}

/**
 * Get total donation amount for a project
 */
export async function getDonationTotal(project: string): Promise<{
  total: number
  count: number
}> {
  const result = await db
    .select({
      total: sql<string>`COALESCE(SUM(donation_amount), 0)`,
      count: sql<number>`COUNT(*)`,
    })
    .from(donations)
    .where(eq(donations.project, project))

  return {
    total: Number(result[0].total),
    count: Number(result[0].count),
  }
}

/**
 * Get last webhook timestamp for a payment provider
 * Used for monitoring webhook health
 */
export async function getLastWebhookTimestamp(provider: string): Promise<Date | null> {
  const result = await db
    .select({ lastCreated: sql<Date>`MAX(created_at)` })
    .from(donations)
    .where(eq(donations.paymentProvider, provider))

  return result[0]?.lastCreated || null
}
```

### Project Structure Notes

- **New directory:** `src/lib/donations/`
- **Files to create:**
  - `src/lib/donations/queries.ts` - Query functions
  - `src/lib/donations/index.ts` - Re-exports
  - `tests/unit/lib/donations-queries.test.ts` - Unit tests

### References

- [Source: docs/architecture/crypto-payments-donations-architecture.md#Impact Page Implementation]
- [Source: docs/sprint-artifacts/epic-crypto-payments-donations.md#Story 2]

## Dev Agent Record

### Agent Model Used

Claude Opus 4.5

### Completion Notes List

- ✅ Created `src/lib/donations/queries.ts` with 3 functions
- ✅ Created `src/lib/donations/index.ts` for clean exports
- ✅ All functions have full TypeScript typing with `DonationTotalResult` interface
- ✅ 9 unit tests covering all functions and edge cases
- ✅ Tests pass: `npx vitest run tests/unit/lib/donations-queries.test.ts`

### File List

- `src/lib/donations/queries.ts` (new)
- `src/lib/donations/index.ts` (new)
- `tests/unit/lib/donations-queries.test.ts` (new)

## Change Log

- 2026-01-20: Story created and implementation started by Amelia (Developer)
- 2026-01-20: Implementation completed - all tests pass
