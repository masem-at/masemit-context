# Bug & Issue Inventory

**Author:** Quinn (QA & Test Architect)
**Date:** 2025-12-09
**Purpose:** Complete inventory of known issues before RESET & REBUILD

---

## 🔴 Critical Issues (BLOCKS PRODUCTION)

### BUG-001: Tech Startup Frontend Timeout
**Status:** IN PROGRESS (QStash implementation)
**Severity:** CRITICAL
**Impact:** Users cannot generate Tech Startup scenarios

**Description:**
- Backend completes successfully (441s)
- Frontend fetch() times out after ~60s
- User sees error despite successful generation
- Scenario exists in DB but user doesn't know

**Root Cause:** Browser timeout limits + synchronous approach incompatible

**Resolution:** QStash async architecture (Winston's plan)

**Test Plan:**
1. Generate Tech Startup scenario
2. Verify progress updates every 2-3 seconds
3. Confirm completion after 7-8 minutes
4. Validate redirect to results page

---

### BUG-002: Validation Warning - Invalid Sales Amounts
**Status:** OPEN (needs investigation)
**Severity:** HIGH (data quality)
**Impact:** 18 sales events have invalid amounts in Tech Startup

**Evidence from logs:**
```
⚠️ Revenue: 18 sales events have invalid amounts
Scenario: f89153e1-663f-4ed9-9063-63f3875d5011
```

**Questions:**
- What makes an amount "invalid"?
- Is this a validation rule issue or generation issue?
- Does it affect Bakery/Hotel scenarios?
- Does it impact financial calculations?

**Action Required:** James to investigate validation logic

**Test Plan:**
1. Review validation rules in `lib/validators/consistency-validator.ts`
2. Check sales events for scenario f89153e1-663f-4ed9-9063-63f3875d5011
3. Determine if amounts are truly invalid or validation is too strict
4. Fix either generation or validation logic
5. Re-run validation on all test scenarios

---

## ⚠️ High Priority Issues

### ISSUE-003: Frontend/Backend Out of Sync
**Status:** OPEN
**Severity:** HIGH
**Impact:** User experience degraded, development friction

**Description:**
- Loading page has polling logic but backend had synchronous approach
- Progress tracking in DB but not used correctly
- UX design doesn't match implementation
- Sally (UX) not consulted on recent changes

**Resolution:** Sally + James alignment session (Phase 2 of rebuild)

**Action Items:**
1. Sally documents desired UX flow
2. James reviews current implementation
3. Align on progress tracking strategy
4. Update frontend to match backend reality

---

### ISSUE-004: Documentation Chaos
**Status:** OPEN
**Severity:** MEDIUM (developer productivity)
**Impact:** Hard to onboard, find info, maintain system

**Problems:**
- Outdated docs not archived
- No clear structure for different doc types
- Some docs contradict current implementation
- Changes not documented as they happen

**Resolution:** Quinn creates folder structure + cleanup plan

**Action Items:**
1. Create `/docs/archive/DEPRECATED/` structure
2. Move irrelevant docs to archive
3. Establish documentation standards
4. All agents commit to updating docs with changes

---

### ISSUE-005: Database State Confusion
**Status:** OPEN
**Severity:** MEDIUM
**Impact:** Test scenarios polluting production DB

**Description:**
- Multiple test scenarios with status 'queued' or 'failed'
- Some completed scenarios not shown correctly
- User reported seeing old scenarios after DB cleanup
- Possible caching issue

**Questions:**
- Are we clearing test data properly?
- Is there a frontend cache issue?
- Do we need a "delete scenario" function?
- Should test scenarios be marked differently?

**Action Required:** James to investigate + propose solution

---

## 🟡 Medium Priority Issues

### ISSUE-006: Agent Names Not Loaded
**Status:** OPEN
**Severity:** LOW (UX issue for Mario)
**Impact:** Generic agent names instead of personalized ones

**Description:**
- River doesn't auto-load `docs/_masemIT/readme.md`
- Agent names should be: Winston, James, Mary, Quinn, Sally, etc.
- Currently showing generic "Solutions Architect", "Developer", etc.

**Resolution:** River updates orchestrator activation instructions

**Action Items:**
1. Update `.bmad-core/agents/bmad-orchestrator.md`
2. Add `docs/_masemIT/readme.md` to auto-load files
3. Test that names appear correctly on next activation

---

### ISSUE-007: No Bug Tracking System
**Status:** OPEN
**Severity:** MEDIUM
**Impact:** Issues discovered but not tracked systematically

**Description:**
- Currently tracking bugs in ad-hoc documents
- No priority system
- No assignment tracking
- No status workflow (Open → In Progress → Resolved → Verified)

**Proposal:** Use this document as lightweight tracker OR implement proper system

**Options:**
1. Continue with markdown files (simple, lightweight)
2. Use GitHub Issues (free, integrated)
3. Use Linear/Jira (overkill for now?)

**Decision Required:** Mario's preference?

---

## 🟢 Low Priority / Enhancement Requests

### ENHANCEMENT-001: Reduce Tech Startup Generation Time
**Status:** BACKLOG
**Severity:** LOW
**Impact:** User experience (7+ minute wait)

**Ideas:**
- Parallel API calls where possible (requires careful ordering)
- Reduce from 12 monthly calls to 6 bi-monthly (less granularity)
- Optimize prompts for faster Claude responses
- Cache certain generation steps

**Note:** Only pursue if QStash doesn't solve UX issue adequately

---

### ENHANCEMENT-002: Progress Estimation Accuracy
**Status:** BACKLOG
**Severity:** LOW
**Impact:** User doesn't know accurate time remaining

**Description:**
- Current progress is linear (1% every 4 seconds)
- Actual generation is non-linear (Stage 3 takes 80% of time)
- Could provide more accurate estimates based on stage

**Proposal:** Use historical timing data to improve estimates

---

## Test Coverage Gaps

### Missing Tests
1. ❌ **End-to-end generation tests** (all 3 scenarios)
2. ❌ **Error handling tests** (network failures, API errors)
3. ❌ **Validation logic tests** (unit tests for consistency-validator)
4. ❌ **Frontend timeout tests** (verify error states)
5. ❌ **Database constraint tests** (uniqueness, relationships)

### Existing Tests (Good)
1. ✅ Unit tests exist (vitest configured)
2. ✅ Test scripts in package.json

**Action Required:** Create comprehensive test plan (separate document)

---

## Browser Compatibility

**Not Tested:**
- Safari (macOS/iOS)
- Firefox
- Mobile browsers
- Different network conditions (slow 3G, etc.)

**Assumption:** Next.js handles most compatibility, but should verify

---

## Security Concerns

### Potential Issues
1. ⚠️ **Rate limiting:** Currently disabled (Redis not configured)
2. ⚠️ **API key exposure:** Verify env vars not leaked
3. ⚠️ **QStash signature verification:** Must implement in worker route
4. ⚠️ **Database injection:** Using Prisma (should be safe) but verify

**Action Required:** Security audit after QStash implementation

---

## Performance Concerns

### Database
- ✅ Indexes exist on scenarioId (primary key)
- ⚠️ No index on status (used for polling) - add if performance degrades
- ✅ Connection pooling via Prisma

### API Routes
- ✅ Rate limiting architecture exists (just needs Redis)
- ⚠️ No caching strategy for results
- ⚠️ Status endpoint polled every 2s (could be optimized with SSE later)

---

## Lessons Learned (For Future Prevention)

1. ❌ **Quick fixes cascade** - Spent 8+ hours on patches instead of 3 hours on proper solution
2. ❌ **Architecture assumptions not verified** - Assumed Vercel could do fire-and-forget
3. ❌ **UX not consulted** - Sally should have designed loading experience first
4. ❌ **Testing after implementation** - Should have test plan before coding
5. ✅ **Good logging** - Helped diagnose issues quickly
6. ✅ **Documentation of problems** - CRITICAL-TECH-STARTUP-TIMEOUT-ISSUES.md was valuable

---

## Sign-Off Checklist (After Rebuild)

Before declaring "production ready":

**Functional:**
- [ ] All 3 scenarios generate successfully
- [ ] Progress tracking works accurately
- [ ] Error states display correctly
- [ ] Validation passes (or warnings understood)
- [ ] Results display correctly

**Technical:**
- [ ] QStash implemented and tested
- [ ] Error handling comprehensive
- [ ] Logs clean and informative
- [ ] Database state clean
- [ ] Security verified

**Documentation:**
- [ ] Architecture docs updated
- [ ] API docs current
- [ ] Environment variables documented
- [ ] Deployment guide accurate

**Quality:**
- [ ] Test plan executed
- [ ] Code reviewed
- [ ] Brother validation completed
- [ ] Mario sign-off obtained

---

**Quinn's Commitment:** Every bug gets tracked, tested, and verified before close.
