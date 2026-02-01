# CRITICAL: Tech Startup Scenario Generation Issues

**Document Date:** 2025-12-09
**Status:** 🔴 NOT PRODUCTION READY
**Severity:** BLOCKING - Prevents PoC validation by brother

---

## Executive Summary

The Tech Startup scenario generation fails due to **fundamental architectural incompatibility** between long-running AI generation (7-8 minutes) and browser HTTP timeout limits (~30-60 seconds). Multiple attempted fixes have failed. System requires proper async job queue architecture.

**Current State:**
- ✅ Bakery scenario: Works (shorter duration ~2-3 min)
- ✅ Hotel scenario: Works (shorter duration ~2-3 min)
- ❌ Tech Startup: Backend succeeds but frontend times out
- ❌ Cannot validate PoC with brother until fixed

---

## Timeline of Issues & Attempted Fixes

### Problem 1: JSON Truncation + Initial Timeouts (Dec 8, Early)

**Issue:** Tech Startup generation hit 300s Vercel timeout, JSON responses truncated at ~11,517 characters.

**Root Cause:**
- Stage 2 prompts generated verbose JSON (~21,600 chars)
- Quarterly batching (4 API calls) took too long
- 300s Vercel Hobby tier timeout exceeded

**Attempted Fixes:**
1. ✅ Reduced Stage 2 token limit: 4096 → 2048
2. ✅ Removed verbose arrays from Stage 2 output
3. ✅ Added retry logic with exponential backoff
4. ✅ Changed to monthly batching (12 API calls instead of 4)

**Result:** Still hit 300s timeout. Led to architectural redesign.

---

### Problem 2: Async Architecture - Background Function Not Triggering (Dec 8, Evening)

**Issue:** Winston's async background function architecture implemented but background worker never executes.

**Implementation:**
- Added database fields: `progress`, `currentStage`, `errorMessage`
- Created `/api/generate` (enqueue, 10s timeout)
- Created `/api/generate/background` (worker, 800s timeout)
- Created `/api/status/[scenarioId]` (polling)
- Frontend polls status every 2 seconds

**Attempted Fixes:**
1. ✅ Added diagnostic logging to background trigger
2. ✅ Made fetch() synchronous to capture errors
3. ✅ Added auto-migrations to build process

**Results:**
```
Logs showed:
- [22:55:54] Enqueueing generation for scenarioId: 42bb1605...
- [22:55:55] Triggering background function at https://telling-cube.vercel.app/api/generate/background
- [22:56:04] Vercel Runtime Timeout Error: Task timed out after 10 seconds

Frontend:
- Hundreds of successful GET /api/status/[scenarioId] requests (polling works)
- Progress stuck at 0% or 15%
- No backend generation logs
```

**Root Cause:** Vercel doesn't support fire-and-forget background functions via HTTP fetch:
- If you **await** the fetch: Enqueue route times out waiting for 800s job
- If you **don't await**: Vercel may terminate the function before completion

**Architectural Finding:** ⚠️ **Vercel serverless functions cannot trigger other long-running serverless functions on the same domain.**

---

### Problem 3: Synchronous Approach - Browser Timeout (Dec 9, Morning)

**Issue:** Winston recommended reverting to synchronous generation (Option A) for quick PoC validation.

**Implementation:**
- Changed `/api/generate` maxDuration: 10s → 800s
- Removed background trigger logic
- Direct synchronous generation
- Frontend waits for response

**Results:**
```
Backend Logs (SUCCESS):
- Started:  07:48:38
- Finished: 07:55:59
- Duration: 441.5 seconds (7.4 minutes)
- Status:   ✅ COMPLETED
- Events:   153 events stored
- Scenario:  f89153e1-663f-4ed9-9063-63f3875d5011

Frontend (FAILURE):
- Shows timeout error after ~30-60 seconds
- User sees error message
- Scenario actually exists in database but user doesn't know
```

**Root Cause:** Browser `fetch()` has built-in timeout (~30-60 seconds) that cannot be extended. Cannot wait 7+ minutes for HTTP response.

**Architectural Finding:** ⚠️ **Synchronous approach only works for server-to-server calls, NOT browser-to-server when duration >60 seconds.**

---

## Technical Root Cause Analysis

### Why Tech Startup Takes 7+ Minutes

```
Stage 1: Master data generation          ~66 seconds
Stage 2: Monthly trends (1 API call)     ~17 seconds
Stage 3: Events (12 monthly API calls)   ~360 seconds (30s each)
Database insert:                         ~1.2 seconds
Validation:                              ~0.4 seconds
─────────────────────────────────────────────────────
Total:                                   ~445 seconds (7.4 minutes)
```

**Why it's slower than Bakery/Hotel:**
- More complex data model (subscriptions, MRR, churn, employees)
- 12 API calls vs 4 (monthly vs quarterly batching)
- Larger prompts with more context
- More events generated per month

### Why Current Approaches Fail

| Approach | Backend | Frontend | Vercel | Result |
|----------|---------|----------|--------|--------|
| **Synchronous** | ✅ Completes in 800s | ❌ Times out ~60s | ✅ Supports 800s | ❌ Frontend sees error |
| **Async (fire-and-forget)** | ❌ Never starts | ✅ Polls successfully | ❌ Kills background fn | ❌ Progress stuck at 0% |
| **Async (await trigger)** | ❌ Timeout in 10s | ✅ Polls successfully | ❌ Enqueue route timeout | ❌ Never starts generation |

**Fundamental Issue:** Vercel's serverless architecture doesn't support:
1. Long-running operations (>60s) called from browser
2. Fire-and-forget background jobs triggered by serverless functions
3. Server-to-server async job triggering on same domain

---

## What Actually Works Right Now

### Backend Generation (Isolated)
✅ Tech Startup generation **completes successfully** when called directly:
- Generates 153 events
- Takes 441.5 seconds
- Stores in database
- Runs validation (has warnings but completes)

**Proof:** Scenario `f89153e1-663f-4ed9-9063-63f3875d5011` exists in database with all data.

### What Doesn't Work
❌ User-facing flow from browser
❌ Progress feedback during generation
❌ Error handling when frontend times out
❌ User has no way to know generation succeeded

---

## Validation Issues Found (Secondary)

During successful backend run, validation found:
```
⚠️ Revenue: 18 sales events have invalid amounts
```

This is a **data quality issue** separate from the timeout problem, but needs investigation.

---

## Proposed Solutions

### Option 1: Upstash QStash (Recommended by James)
**What:** Serverless message queue designed for Vercel

**Pros:**
- Proper async job queue
- Guaranteed execution
- Same vendor as existing Redis
- Free tier: 1,000 messages/day (sufficient)

**Cons:**
- New dependency
- Requires setup and testing
- ~2-3 hours implementation

**Implementation:**
```typescript
// In /api/generate
await qstash.publishJSON({
  url: `${baseUrl}/api/generate/background`,
  body: { scenarioId, scenarioType }
})
```

**Estimated Timeline:** 2-3 hours

---

### Option 2: Fix Async Polling (Debug Why It Failed)
**What:** Investigate why background function wasn't triggered, fix root cause

**Pros:**
- No new dependencies
- Uses existing Vercel infrastructure
- Architecture already implemented

**Cons:**
- Unknown root cause
- May hit Vercel limitations again
- Already spent 4+ hours debugging with no success

**Estimated Timeline:** Unknown (2-8 hours)

---

### Option 3: Server-Sent Events (SSE)
**What:** Stream progress updates from long-running connection

**Pros:**
- Real-time progress updates
- No polling overhead
- No external dependencies

**Cons:**
- Complex implementation
- Still has browser connection limits (~5 min max)
- May not work on all browsers/proxies

**Estimated Timeline:** 4-6 hours

---

### Option 4: Reduce Generation Time
**What:** Optimize AI calls to complete in <60 seconds

**Pros:**
- Works with synchronous approach
- No architectural changes needed

**Cons:**
- **Significantly degrades data quality**
- Would need to:
  - Reduce from 12 months to 3 months (quarterly only)
  - Remove detailed event generation
  - Use smaller context windows
  - Skip validation
- Defeats purpose of "realistic business data"

**Estimated Timeline:** 2 hours, but **NOT RECOMMENDED** - compromises product value

---

### Option 5: Two-Step User Flow (Workaround)
**What:**
1. User clicks "Generate" → shows "Generation started" page
2. Send email when complete
3. User returns later to view results

**Pros:**
- Simple implementation
- No architectural changes
- Works with existing code

**Cons:**
- ❌ **Terrible UX** - user must wait for email
- ❌ Requires email integration
- ❌ User might never return
- ❌ Not suitable for demo/PoC

**Estimated Timeline:** 1 hour, but **NOT RECOMMENDED** - poor UX

---

## Cost-Benefit Analysis

| Solution | Implementation Time | Risk | Data Quality | UX | Production Ready |
|----------|-------------------|------|--------------|----|--------------------|
| **QStash** | 2-3 hours | Low | ✅ Full | ✅ Real-time progress | ✅ Yes |
| **Fix Async** | 2-8 hours | High | ✅ Full | ✅ Real-time progress | ⚠️ Maybe |
| **SSE** | 4-6 hours | Medium | ✅ Full | ✅ Real-time progress | ⚠️ Maybe |
| **Reduce Time** | 2 hours | Low | ❌ Degraded | ✅ Fast | ❌ No (poor data) |
| **Email Flow** | 1 hour | Low | ✅ Full | ❌ Poor | ❌ No (bad UX) |

---

## Team Recommendation

**Primary:** Implement **Option 1 (QStash)**
- Cleanest architecture
- Production-ready
- Same vendor as existing Redis
- 2-3 hours to working solution

**Fallback:** If QStash has issues, investigate **Option 2 (Fix Async)**
- Already partially implemented
- No new dependencies
- But unknown if Vercel supports this pattern

**Not Recommended:**
- ❌ Option 4: Degrades product value
- ❌ Option 5: Poor user experience

---

## Current Code State

### Files Modified (Not Yet Reverted):
- ✅ `app/api/generate/route.ts` - Synchronous generation (800s timeout)
- ✅ `app/generate/loading/page.tsx` - Simulated progress (not polling)
- ⚠️ `app/api/generate/background/route.ts` - Exists but unused
- ⚠️ `app/api/status/[scenarioId]/route.ts` - Exists but unused
- ✅ `prisma/schema.prisma` - Has async fields (harmless to keep)
- ✅ `package.json` - Auto-migrations in build (keep)

### Database State:
- ⚠️ Multiple test scenarios created during debugging
- ⚠️ Some marked as 'failed' or 'queued' but actually completed
- ⚠️ User sees old scenarios even after database cleanup (cache issue?)

---

## Impact on PoC Validation

**Brother's Validation Checklist:**
- ✅ Scenario 1 (Bakery): Works
- ✅ Scenario 2 (Hotel): Works
- ❌ **Scenario 3 (Tech Startup): BLOCKS VALIDATION**
- ✅ Stripe checkout: Works
- ✅ Email verification: Works

**Cannot proceed with launch until Tech Startup works.**

---

## Questions for Team Discussion

1. **Architecture:** QStash vs Fix Async vs Other approach?
2. **Timeline:** Can we afford 2-3 hours for QStash implementation?
3. **Scope:** Should we reduce Tech Startup complexity to make it faster? (Not recommended)
4. **Validation:** Should we fix the "18 sales events have invalid amounts" warning separately?
5. **Database Cleanup:** How do we handle test scenarios in production?
6. **User Feedback:** What should user see if generation takes >5 minutes?

---

## Next Steps (Pending Decision)

1. **Immediate:** Team discussion to choose solution approach
2. **Implementation:** 2-3 hours (if QStash chosen)
3. **Testing:** Full end-to-end test of Tech Startup generation
4. **Brother Re-validation:** All 3 scenarios
5. **Launch:** If validation passes

---

## Lessons Learned

1. ⚠️ **Vercel serverless functions have architectural limits** - can't do background jobs without external queue
2. ⚠️ **Browser fetch() timeouts are non-negotiable** - must use async pattern for >60s operations
3. ⚠️ **"Quick fixes" can waste more time than proper solutions** - should have gone with QStash initially
4. ⚠️ **Test with production constraints early** - local dev doesn't reveal serverless limitations
5. ✅ **Comprehensive logging helps diagnosis** - we now know exactly what fails and why

---

**Document prepared by:** James (Backend Developer)
**Reviewed by:** Winston (Solutions Architect - architectural analysis)
**For discussion with:** Mario (Product Owner), full team
