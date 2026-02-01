# QStash Implementation Plan

**Author:** Winston (Solutions Architect)
**Date:** 2025-12-09
**Status:** APPROVED - Ready for Implementation

---

## Decision: Upstash QStash for Async Job Processing

### Why QStash

**Architectural Requirements:**
- Long-running operations (7-8 minutes for Tech Startup)
- Real-time progress feedback to user
- Browser timeout constraints (~60 seconds)
- Vercel serverless limitations (no fire-and-forget)

**QStash Benefits:**
1. ✅ Designed for serverless environments
2. ✅ Guaranteed job execution
3. ✅ Same vendor as existing Upstash Redis
4. ✅ Free tier sufficient (1,000 messages/day)
5. ✅ Simple integration (~3-4 hours)

**Rejected Alternatives:**
- ❌ Synchronous: Browser timeout (current problem)
- ❌ Fire-and-forget fetch: Vercel kills background functions
- ❌ SSE: Complex, still has connection limits
- ❌ Reduce generation time: Degrades product quality

---

## Architecture Design

### Flow Diagram

```
User Click
    ↓
[POST /api/generate]
    ↓
Create DB record (status='queued')
    ↓
Publish to QStash → [/api/generate/worker]
    ↓                         ↓
Return scenarioId      (runs async 7-8 min)
    ↓                         ↓
Frontend polls         Updates DB progress
/api/status            (0% → 25% → 50% → 100%)
    ↓                         ↓
Shows progress         Stores events
    ↓                         ↓
Redirects when         Marks complete
status='completed'
```

### Components

#### 1. `/api/generate` (Enqueue Route)
```typescript
// maxDuration: 10s (short)
export async function POST(request: NextRequest) {
  // 1. Create scenario record (status='queued')
  const scenarioId = generateScenarioId()
  await prisma.scenario.create({ id: scenarioId, status: 'queued', ... })

  // 2. Publish to QStash
  await qstash.publishJSON({
    url: `${baseUrl}/api/generate/worker`,
    body: { scenarioId, scenarioType }
  })

  // 3. Return immediately
  return { scenarioId, status: 'queued' }
}
```

#### 2. `/api/generate/worker` (Background Worker)
```typescript
// maxDuration: 800s (long-running)
export async function POST(request: NextRequest) {
  const { scenarioId, scenarioType } = await request.json()

  // Update status to processing
  await updateProgress(scenarioId, 0, 'processing')

  // Generate (with progress updates)
  const result = await generateTechStartupScenario()
  await updateProgress(scenarioId, 80, 'storing')

  // Store events
  await insertEvents(scenarioId, result.events)
  await updateProgress(scenarioId, 95, 'validating')

  // Validate
  const validation = await validateScenario(scenarioId)

  // Mark complete
  await prisma.scenario.update({
    where: { id: scenarioId },
    data: { status: 'completed', progress: 100 }
  })
}
```

#### 3. `/api/status/[scenarioId]` (Status Endpoint)
- Unchanged - already exists
- Frontend polls every 2-3 seconds
- Returns current progress and status

#### 4. Frontend Loading Page
- Unchanged - already polls correctly
- Shows real progress from database
- Redirects on completion

---

## Implementation Steps

### Step 1: Setup QStash (30 min)
```bash
# Install SDK
pnpm add @upstash/qstash

# Add env vars to Vercel
QSTASH_URL=https://qstash.upstash.io
QSTASH_TOKEN=<from upstash dashboard>
QSTASH_CURRENT_SIGNING_KEY=<from upstash dashboard>
QSTASH_NEXT_SIGNING_KEY=<from upstash dashboard>
```

### Step 2: Update `/api/generate/route.ts` (1 hour)
- Import QStash client
- Replace fetch() with qstash.publishJSON()
- Keep enqueue pattern (status='queued')
- Remove synchronous generation code

### Step 3: Rename `/api/generate/background` → `/api/generate/worker` (15 min)
- Rename route file
- Add QStash signature verification
- Keep existing generation logic
- Update progress tracking

### Step 4: Test End-to-End (1 hour)
- Test Bakery (should still work)
- Test Hotel (should still work)
- Test Tech Startup (should complete in 7-8 min with progress)
- Verify error handling

### Step 5: Documentation (30 min)
- Update architecture docs
- Add QStash to tech stack
- Document environment variables
- Update deployment guide

**Total Time:** ~3-4 hours

---

## Progress Tracking Strategy

### Database Progress Mapping

| Stage | Progress % | currentStage | Duration |
|-------|-----------|--------------|----------|
| Enqueued | 0% | 'queued' | 0s |
| Starting | 5% | 'starting' | 5s |
| Stage 1 | 10-30% | 'stage1_generating' | 60-90s |
| Stage 2 | 35-45% | 'stage2_trends' | 15-20s |
| Stage 3 | 50-80% | 'stage3_month_X' | 300-400s |
| Database | 85-90% | 'storing' | 1-2s |
| Validation | 95% | 'validating' | 1s |
| Complete | 100% | 'completed' | - |

### Frontend Display

```typescript
// Map backend stages to user-friendly messages
const STAGE_MESSAGES = {
  'stage1_generating': 'Generating company data and structure...',
  'stage2_trends': 'Creating monthly business trends...',
  'stage3_month_1': 'Generating January events...',
  'stage3_month_2': 'Generating February events...',
  // ... etc
  'storing': 'Storing events in database...',
  'validating': 'Running consistency validation...',
  'completed': 'Complete!'
}
```

---

## Error Handling

### QStash Failures
- QStash has automatic retries (3x by default)
- If all retries fail → mark scenario as 'failed' with error message
- Frontend shows error and "Try Again" button

### Generation Failures
- Caught in worker route
- Update scenario: `status='failed', errorMessage=<error>`
- Frontend polling detects failed status → shows error UI

### Timeout Protection
- Worker has 800s max (13.3 minutes)
- Generation typically takes 441s (7.4 minutes)
- 359s buffer for spikes

---

## Rollback Plan

If QStash has unexpected issues:
1. Keep synchronous code in git history
2. Can revert with `git revert <commit>`
3. Fall back to "email when done" pattern temporarily
4. ~15 minutes to rollback

---

## Success Metrics

✅ **Technical:**
- Tech Startup completes in <800s
- Progress updates every 2-3 seconds
- All 3 scenarios work reliably
- No frontend timeouts

✅ **User Experience:**
- Real-time progress feedback
- Clear stage messaging
- Smooth redirect on completion
- Professional error handling

✅ **Code Quality:**
- Clean architecture
- Proper error handling
- Comprehensive logging
- Updated documentation

---

## Next Steps

1. **Immediate:** Get QStash credentials from Upstash
2. **Implementation:** James follows this plan (Steps 1-5)
3. **Review:** Winston reviews code before deployment
4. **Testing:** Quinn executes test plan
5. **Sign-off:** Mario approves for brother's validation

---

**Winston's Commitment:** This is the right architecture. No more quick fixes - we do this properly.
