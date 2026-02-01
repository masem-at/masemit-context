# Phase 3 Test Verification Checklist

**Author:** Quinn (QA & Test Architect)
**Date:** 2025-12-09
**Purpose:** Verify QStash async architecture works end-to-end

---

## Test Environment

**Deployment:** Production (Vercel)
**QStash:** Configured (all 3 credentials)
**Database:** NeonDB (production)
**First Test Started:** [In Progress]

---

## Critical Success Criteria

### ✅ Must Pass (Production Blocking)

1. **Generation Completes Successfully**
   - [ ] Tech Startup generation completes (no timeout)
   - [ ] Progress updates visible in real-time
   - [ ] Redirect to results page after completion
   - [ ] Data visible in results page

2. **Progress Tracking Works**
   - [ ] Progress bar updates every 2-3 seconds
   - [ ] Progress percentages accurate (0% → 100%)
   - [ ] Stage indicators update correctly
   - [ ] No stuck progress (frozen at one value)

3. **Frontend Experience**
   - [ ] No browser timeout errors
   - [ ] Loading page displays correctly
   - [ ] Educational tips rotate
   - [ ] Smooth animations

4. **Backend Logs**
   - [ ] QStash publish successful
   - [ ] Worker receives job
   - [ ] Generation stages logged
   - [ ] No errors in Vercel logs

### ⚠️ Should Pass (Non-Blocking)

5. **All 3 Scenarios Work**
   - [ ] Bakery: Completes in ~2-3 minutes
   - [ ] Hotel: Completes in ~3-4 minutes
   - [ ] Tech Startup: Completes in ~7-8 minutes

6. **Validation Passes**
   - [ ] No critical validation errors
   - [ ] Acceptable validation warnings (document any)
   - [ ] Multi-view consistency verified

7. **Error Handling**
   - [ ] Graceful error display if generation fails
   - [ ] Clear error messages
   - [ ] "Try Again" button works

---

## Test Execution Log

### Test 1: [Scenario Type] - [Timestamp]

**Start Time:** _____
**End Time:** _____
**Duration:** _____
**Result:** ⬜ PASS / ⬜ FAIL / ⬜ IN PROGRESS

**Observations:**
- Progress updates: _____
- Final redirect: _____
- Data quality: _____
- Any errors: _____

**Logs:**
```
[Paste relevant Vercel logs here]
```

**Screenshots:**
- Loading page: _____
- Results page: _____
- Any errors: _____

---

### Test 2: [Scenario Type] - [Timestamp]

**Start Time:** _____
**End Time:** _____
**Duration:** _____
**Result:** ⬜ PASS / ⬜ FAIL / ⬜ IN PROGRESS

**Observations:**
_____

---

### Test 3: [Scenario Type] - [Timestamp]

**Start Time:** _____
**End Time:** _____
**Duration:** _____
**Result:** ⬜ PASS / ⬜ FAIL / ⬜ IN PROGRESS

**Observations:**
_____

---

## What to Check in Vercel Logs

### Enqueue Route (`/api/generate`)
```
✅ Expected:
[timestamp] Enqueueing generation for scenarioId: [id]
[timestamp] Publishing to QStash: https://[domain]/api/generate/worker
[timestamp] ✅ Enqueued successfully

❌ Red Flags:
- "Failed to publish to QStash"
- "QSTASH_TOKEN not found"
- Timeout errors
```

### Worker Route (`/api/generate/worker`)
```
✅ Expected:
[timestamp] 🔄 Worker Generation Started (QStash)
[timestamp] Progress: 10% (stage1_generating)
[timestamp] Progress: 50% (stage1_complete)
[timestamp] Progress: 90% (database_insert)
[timestamp] Progress: 95% (validation)
[timestamp] ✅ Worker Generation Complete
Duration Breakdown: AI Generation: Xs, DB Insert: Xs, Validation: Xs

❌ Red Flags:
- "Signature verification failed"
- Generation errors
- Validation failures
- Timeout errors
```

### Status Endpoint (`/api/status/[scenarioId]`)
```
✅ Expected:
Multiple GET requests every 2 seconds
Returns progress updates: 0% → 15% → 30% → ... → 100%

❌ Red Flags:
- 404 Not Found
- Stuck at same progress value
- No requests (polling not working)
```

---

## Data Quality Checks

### After Generation Completes

1. **Results Page Loads**
   - [ ] Charts render correctly
   - [ ] Data displays properly
   - [ ] No "undefined" or null values
   - [ ] IBCS colors applied

2. **Database Record**
   - [ ] Scenario status = 'completed'
   - [ ] Progress = 100
   - [ ] Events stored (count > 0)
   - [ ] Validation results logged

3. **Validation Review**
   - [ ] Check `validationPassed` field
   - [ ] Review `validationErrors` array
   - [ ] Acceptable warnings documented
   - [ ] Critical errors = 0

---

## Known Issues to Monitor

### From Bug Inventory (docs/qa/bug-inventory.md)

**BUG-002: Validation Warning - Invalid Sales Amounts**
- Expected: May see "X sales events have invalid amounts"
- Action: Document count, investigate if >20 events affected
- Non-blocking if <20 events

**ISSUE-005: Database State Confusion**
- Check: Are old test scenarios visible?
- Action: Document if present, plan cleanup

---

## Performance Benchmarks

### Target Durations (from Winston's plan)

| Scenario | Target | Acceptable Range |
|----------|--------|------------------|
| Bakery | 2 min | 1.5-3 min |
| Hotel | 3 min | 2.5-4 min |
| Tech Startup | 7.5 min | 6-9 min |

### Actual Results

| Scenario | Duration | Status | Notes |
|----------|----------|--------|-------|
| Test 1 | _____ | _____ | _____ |
| Test 2 | _____ | _____ | _____ |
| Test 3 | _____ | _____ | _____ |

---

## Browser Compatibility

**Primary Test:** Chrome (latest)

**If time permits:**
- [ ] Firefox
- [ ] Safari (macOS)
- [ ] Mobile Chrome (iOS/Android)

---

## Regression Tests

**Verify existing features still work:**
- [ ] Home page loads
- [ ] Scenario selection buttons work
- [ ] Stripe checkout flow (spot check)
- [ ] Email verification (if applicable)

---

## Sign-Off Criteria

### ✅ **PASS - Production Ready**

All must be true:
- [ ] At least 1 Tech Startup completes successfully (no timeout)
- [ ] Progress tracking works (real-time updates)
- [ ] Results page displays correctly
- [ ] No critical errors in logs
- [ ] Validation passes or has acceptable warnings only

### ⚠️ **PARTIAL PASS - Minor Issues**

Criteria:
- Tech Startup works BUT has minor issues (e.g., validation warnings)
- Document issues and plan fixes for next iteration
- Mario approves launch with known issues

### ❌ **FAIL - Blocks Production**

Any of these:
- Tech Startup times out (same as before)
- Progress tracking broken (stuck at 0%)
- Critical errors in generation
- Data quality unacceptable
- Frontend crashes or major UX issues

---

## Post-Test Actions

### If PASS ✅
1. Run Bakery & Hotel scenarios (quick verification)
2. Document any validation warnings
3. Proceed to Phase 4: Documentation cleanup
4. Prepare for brother's re-validation

### If PARTIAL PASS ⚠️
1. Document all issues clearly
2. Triage: Fix now vs. fix later
3. Get Mario's approval to proceed
4. Create follow-up tasks

### If FAIL ❌
1. Capture all error logs
2. Emergency team meeting (ROF)
3. Debug root cause
4. Implement fix
5. Re-test

---

## Notes Section

**Observations during testing:**
_____

**Unexpected behaviors:**
_____

**Performance notes:**
_____

**Recommendations:**
_____

---

**Quinn's Commitment:** Every test result thoroughly documented, issues tracked, quality gates enforced.
