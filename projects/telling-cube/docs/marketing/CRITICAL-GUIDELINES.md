# CRITICAL Marketing Guidelines

**Status**: MANDATORY - All marketing materials MUST follow these rules
**Last Updated**: 2025-12-14
**Severity**: CRITICAL - Legal/privacy implications

---

## ❌ NEVER MENTION

### 1. Day Job Information (LEGAL RISK)
- ❌ "Product Owner" title
- ❌ "Insurance company"
- ❌ "20,000+ employee company"
- ❌ Any specific employer details

**Why**: Legal separation between day job and personal business required

---

### 2. Family/Personal Information (PRIVACY)
- ❌ "My brother"
- ❌ "Hiasey" (brother's name)
- ❌ Specific family member names

**Why**: Privacy protection

**Alternative**: "Controller-validated" or "Expert-validated" (no personal identifiers)

---

## ❌ TRADEMARK COMPLIANCE - IBCS©

### CRITICAL: IBCS© Trademark Usage

**NEVER say**:
- ❌ "IBCS-compliant"
- ❌ "IBCS compliance"
- ❌ "IBCS standards"
- ❌ "IBCS Institute standards"
- ❌ "follows IBCS"

**ALWAYS say**:
- ✅ "inspired by IBCS©"
- ✅ "IBCS©-inspired visualizations"
- ✅ "charts inspired by IBCS© standards"

**Why**: Trademark compliance per discussions with IBCS Institute CEO. We don't have certification rights.

**Source**: docs/brainstorming-session-results.md:8

---

## ✅ CORRECT PRODUCT INFORMATION

### Generation Time

**Updated 2025-12-12: Two-Phase Architecture enables accurate timing!**

| Tier | Expected Duration |
|------|-------------------|
| Startup | ~2 minutes (120s) |
| Midcap | ~3 minutes (160s) |
| Largecap | ~3 minutes (170s) |

- ✅ "in minutes" (general, safe)
- ✅ "about 2 minutes" (for startup tier)
- ✅ "2-3 minutes" (general, covers all tiers)
- ❌ "60 seconds" (still too aggressive)
- ❌ "instantly" (misleading)

**Source**: Landing page Hero.tsx - "Enterprise-quality business data in minutes"

---

### Pricing Structure

**Model**: ONE-TIME LIFETIME PURCHASE (not subscription!)

| Tier | Price | Seats | Description |
|------|-------|-------|-------------|
| Supporter S | €29 | 1 | Individual |
| Supporter M | €99 | 1 | Individual |
| Supporter L | €299 | 1 | Professional |
| Team Plus | €599 | 5 | Small Team |
| Department Partner | €999 | 20 | Department |

**Key Points**:
- ✅ "Lifetime access - pay once, use forever"
- ✅ "Grandfathered pricing - no price increases"
- ✅ "Unlimited scenario generations"
- ❌ "€9/month" (WRONG - this doesn't exist!)
- ❌ "monthly subscription" (WRONG - it's one-time!)

**Source**: components/landing/PricingSection.tsx

---

## ✅ SAFE FOUNDER STORY ELEMENTS

**What you CAN say**:
- ✅ "Austrian entrepreneur"
- ✅ "Founded masemIT e.U."
- ✅ "Built in Austria 🇦🇹"
- ✅ "Solo founder"
- ✅ "8 weeks to build"

**What you CAN'T say**:
- ❌ Specific day job title/employer
- ❌ Family member names

---

## ✅ VALIDATION LANGUAGE

**WRONG**:
- ❌ "My brother (controller) validated it"
- ❌ "Tested by Hiasey"

**RIGHT**:
- ✅ "Controller-validated with 10+ years experience"
- ✅ "Expert-validated by finance professionals"
- ✅ "Validated by controllers in 250+ person companies"

---

## 🛡️ PRE-FLIGHT PROTOCOL (MANDATORY)

### When to Use Pre-Flight

**MANDATORY Before Creating:**
- 📣 Marketing content (LinkedIn posts, blog posts, press releases)
- ⚖️ Legal documents (terms of service, privacy policy, license agreements)
- 🌐 Public-facing materials (website copy, demo videos, social media)
- 💰 Financial/pricing communications
- 🚀 Deployment announcements

**NOT Required For:**
- Internal code changes (PR review sufficient)
- Documentation updates (internal)
- Exploratory research
- Private planning discussions

---

### Pre-Flight Steps (All Required ✅)

#### Step 1: Discover Critical Context
```
✅ Read this file (CRITICAL-GUIDELINES.md)
✅ Read recent related work in project
✅ Check actual implementation (code, configs, NOT assumptions)
```

#### Step 2: Verify Assumptions Against Codebase
```
✅ Pricing: Read components/landing/PricingSection.tsx
✅ Features: Grep codebase for actual capabilities
✅ Legal/Trademark: Check documented compliance rules (IBCS©)
✅ Privacy: Verify no personal identifiers in content
✅ Timing claims: Don't make specific time promises ("60 seconds")
```

**Why this matters:**
- **Real failure case**: Sophie created LinkedIn posts with €9/month pricing (doesn't exist - actual: €29-€999 lifetime)
- **Real failure case**: Used "IBCS-compliant" (trademark violation - should be "inspired by IBCS©")
- **Real failure case**: Mentioned "Product Owner" title (privacy/legal risk)

#### Step 3: Cross-Agent Review (Minimum 2 Agents)
```
✅ Orchestrator (River) reviews before user sees
✅ Fact-checker (Mary) validates claims against codebase
✅ Minimum 2 agents verify before presenting to user
```

**Review Checklist:**
- [ ] No privacy violations (day job, family names)
- [ ] No trademark violations (IBCS© compliance)
- [ ] Pricing correct (verified from code)
- [ ] Features accurate (verified from codebase)
- [ ] No impossible claims (timing, generation quality)

#### Step 4: User Approval Gate
```
✅ Present content as DRAFT (not final)
✅ Highlight assumptions made (if any)
✅ Get explicit user approval before publishing
```

**Never assume approval** - always present and wait for GO signal.

---

### Pre-Flight Verification Template

**Use this template before presenting content:**

```markdown
## Pre-Flight Verification Report

**Content Type:** [LinkedIn post / Demo video / etc.]
**Agent:** [Your name]
**Date:** [Today's date]

### Verification Checklist ✅

**Step 1: Context Discovery**
- [x] Read CRITICAL-GUIDELINES.md
- [x] Reviewed [list files read]

**Step 2: Fact Verification**
- [x] Pricing verified from: components/landing/PricingSection.tsx
- [x] Features verified by: [grep command or file reads]
- [x] IBCS© language: "inspired by IBCS©" ✅
- [x] Privacy check: No personal identifiers ✅

**Step 3: Cross-Agent Review**
- [x] Reviewed by: [Agent names]
- [x] Issues found: [None / List issues]
- [x] Issues resolved: [Yes / N/A]

**Step 4: Presenting as DRAFT**
- [x] Content marked as DRAFT for user approval
- [x] Awaiting user GO signal

### Content Ready for Review ✅
[Present the content here]
```

---

## Approval Process

**ALL marketing materials must**:
1. ✅ Complete pre-flight protocol (all 4 steps)
2. ✅ Be reviewed by River + Mary before presenting to user
3. ✅ Have zero mentions of prohibited items
4. ✅ Use correct pricing/timing information verified from code
5. ✅ Be presented as DRAFT for user approval
6. ✅ Wait for explicit GO before publishing

**Files to check**:
- [ ] LinkedIn posts
- [ ] Demo video scripts
- [ ] Email templates
- [ ] Landing page copy (if changed)
- [ ] Social media posts
- [ ] Press releases
- [ ] Terms of service / Privacy policy
- [ ] Pricing announcements

---

## 🚨 Incident History (Learn From These)

### Incident #1: LinkedIn Post Violations (2025-11-22)
**Agent:** Sophie
**Violations Found:**
1. ❌ Used "€9/month" pricing (doesn't exist - actual: €29-€999 lifetime)
2. ❌ Mentioned "Product Owner" title (privacy/legal risk)
3. ❌ Mentioned "my brother" (privacy violation)
4. ❌ Used "IBCS-compliant" (trademark violation - should be "inspired by IBCS©")
5. ❌ Claimed "60 seconds" generation (inaccurate - should be "minutes")

**Root Cause:** No pre-flight verification - content created from assumptions, not facts

**Impact:** User caught all errors before publishing, but spent 30 minutes fixing

**Fix:** Created this pre-flight protocol, fixed all marketing materials

**Lesson:** NEVER create public content without verifying facts against actual codebase

---

## Emergency Contacts

**If unsure about ANYTHING**: Ask River or Mary before creating content!

**Better to ask and wait 5 minutes than publish wrong information.**

---

**Document Version:** 2.1 (Updated timing language)
**Last Updated:** 2025-12-14
**See Also:** `docs/bmad-contributions/pr-02-agent-preflight-protocol.md` (full specification)
