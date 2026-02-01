# Story 9.1: Fix Paid User Scenario Retention

Status: done

## Story

As a paid user (Lifetime+, Pro, Team),
I want my generated scenarios to persist indefinitely,
So that I don't lose my work after 24 hours.

## Bug Report

**Reporter:** Mario (Lifetime+ user)
**Severity:** CRITICAL - Data loss for paying customers
**Date:** 2026-01-25

**Symptoms:**
- User created scenario while logged in with Lifetime+ account
- Scenario disappeared from dashboard after ~24 hours
- No warning or notification about deletion

**Root Cause Analysis:**

1. `app/api/generate/route.ts:295` - ALL scenarios get hardcoded 24h expiry:
   ```typescript
   expiresAt: new Date(Date.now() + 24 * 60 * 60 * 1000), // 24 hours from now
   ```

2. `app/api/cron/cleanup/route.ts:25` - Cleanup deletes ALL expired scenarios without checking user tier:
   ```typescript
   await prisma.scenario.deleteMany({
     where: { expiresAt: { lt: new Date() } },
   })
   ```

## Acceptance Criteria

1. - [x] AC1: Cleanup job checks user tier before deleting scenarios
2. - [x] AC2: Paid tiers (lifetime, lifetime_plus, pro, team) have unlimited retention
3. - [x] AC3: Free/anonymous scenarios still expire after 24h
4. - [x] AC4: Existing paid user scenarios are protected (retroactive fix)

## Tasks / Subtasks

- [x] Task 1: Update cleanup job to check tier (AC: 1, 2, 3, 4)
  - [x] 1.1: Modify `app/api/cron/cleanup/route.ts`
  - [x] 1.2: Join through `UserScenario` → `User` to get tier
  - [x] 1.3: Only delete scenarios where:
    - No linked user (anonymous), OR
    - Linked user has `tier = 'free'` or `tier = 'single'`
  - [x] 1.4: Paid tiers skip deletion entirely (expiresAt ignored)
- [x] Task 2: Add tests for cleanup logic (AC: 1, 2, 3)
  - [x] 2.1: Test anonymous scenario deleted after expiry
  - [x] 2.2: Test free user scenario deleted after expiry
  - [x] 2.3: Test paid user scenario NOT deleted after expiry
- [x] Task 3: Verify fix (AC: 4)
  - [x] 3.1: Implementation verified via unit tests (12 tests pass)
  - [x] 3.2: Retroactive fix confirmed - existing scenarios protected by tier check

## Dev Notes

### Architecture Decision

**Option 2 (Winston):** Cleanup job joins on UserScenario → User to check tier.

Benefits:
- Tier upgrades protect existing scenarios retroactively
- Subscription lapse eventually allows cleanup
- Anonymous scenarios still cleaned up
- Single source of truth for retention policy

### Paid Tiers (from lib/auth/entitlements.ts)

| Tier | Retention |
|------|-----------|
| `free` | 24 hours |
| `single` | 24 hours (one-time purchase, no account) |
| `lifetime` | Unlimited |
| `lifetime_plus` | Unlimited |
| `pro` | Unlimited (while subscription active) |
| `team` | Unlimited (while subscription active) |

### Key Files

| File | Change |
|------|--------|
| `app/api/cron/cleanup/route.ts` | Add tier check before deletion |

## References

- [Source: app/api/cron/cleanup/route.ts] - Current cleanup logic
- [Source: app/api/generate/route.ts:295] - Hardcoded 24h expiry
- [Source: lib/auth/entitlements.ts] - Tier definitions

## Dev Agent Record

### Context Reference

Story context from Epic 9 Bug Fixes - Party Mode bug triage session 2026-01-25

### Agent Model Used

Claude Opus 4.5 (claude-opus-4-5-20251101)

### Debug Log References

N/A - Clean implementation

### Completion Notes List

- Modified `app/api/cron/cleanup/route.ts` to implement tier-aware cleanup
- Cleanup now joins `UserScenario` → `User` to check tier AND subscriptionStatus
- Only deletes scenarios where: no linked user OR user.tier is 'free' or 'single'
- Lifetime tiers (lifetime, lifetime_plus) always protected (one-time purchase)
- Subscription tiers (pro, team) only protected when subscriptionStatus='active'
- Canceled/past_due subscriptions are NOT protected (churned customers)
- Created 14 comprehensive unit tests covering all tier and subscription scenarios
- All tests pass (14/14)

### Code Review Fixes Applied

| Issue | Severity | Fix |
|-------|----------|-----|
| H1: Subscription status not checked | HIGH | Added subscriptionStatus check for pro/team |
| M2: Missing canceled subscription test | MEDIUM | Added 2 new tests for canceled/past_due |
| L2: Magic string for tiers | LOW | Imported Tier type from entitlements.ts |

### File List

| File | Change |
|------|--------|
| `app/api/cron/cleanup/route.ts` | MODIFIED - Tier-based retention with subscription check |
| `app/api/cron/cleanup/__tests__/cleanup.test.ts` | NEW - 14 unit tests for cleanup logic |
