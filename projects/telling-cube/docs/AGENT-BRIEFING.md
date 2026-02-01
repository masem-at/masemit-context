# Agent Briefing - tellingCube Onboarding
**Purpose:** Quick onboarding checklist for all BMad agents
**Target:** New agents, new sessions, or agents switching roles

---

## 🚀 Quick Start (2-Minute Read)

**Project:** tellingCube - Realistic business data generator for demos/training
**Status:** MVP complete, awaiting validation, launch pending
**Your Role:** [Will vary - check with River/user]

**Before you start ANY work, read this entire document.**

---

## ✅ MANDATORY Reading (Before ANY Work)

### 1. Current Project State
📖 **READ FIRST:** `docs/PROJECT-CONTEXT.md`

**What you'll learn:**
- What tellingCube is and does
- Current status (MVP complete, pending validation)
- Technical architecture (Next.js, NeonDB, Claude, Stripe)
- Business model (€29-€999 lifetime pricing)
- Key features (Finance view, Sales view, IBCS-inspired charts)

**Time:** 5 minutes

---

### 2. Company & Agent Family Context
📖 **READ SECOND:** `docs/_masemIT/readme.md`

**What you'll learn:**
- Who Mario Semper (Sempre) is
- masemIT e.U. company details
- The full agent family roster (River, Mary, Winston, James, etc.)
- Ring of Fire (ROF) sessions innovation
- Human team (validation partner)

**Time:** 3 minutes

**⚠️ CRITICAL:** This file contains personal information you MUST NOT mention publicly:
- Day job: Product Owner at insurance company
- Family details
- See CRITICAL-GUIDELINES.md for what's forbidden

---

### 3. Mandatory Rules (If Doing Public-Facing Work)
📖 **READ BEFORE MARKETING/LEGAL/PUBLIC WORK:** `docs/marketing/CRITICAL-GUIDELINES.md`

**What you'll learn:**
- ❌ What to NEVER mention (day job, family, personal info)
- ✅ How to handle IBCS© trademark ("inspired by", not "compliant")
- ✅ Where to verify pricing (code, not assumptions)
- ✅ Pre-flight protocol requirements

**Time:** 5 minutes

**⚠️ FAILURE TO READ THIS HAS CAUSED CRITICAL ERRORS:**
- Sophie created LinkedIn posts with wrong pricing, privacy violations, trademark violations
- All fixed now, but this was painful and avoidable

---

## 📚 Role-Specific Reading

### For Technical Work (Dev, Architect, QA)

**READ:**
- `docs/architecture.md` - Full technical architecture (event-sourcing model)
- `docs/architecture/tech-stack.md` - Technologies used (Next.js 15, React 19, etc.)
- `docs/architecture/coding-standards.md` - Code style guide
- `docs/architecture/source-tree.md` - Project structure

**DON'T assume anything about implementation - READ THE CODE:**
- Pricing: `components/landing/PricingSection.tsx`
- Stripe: `app/api/stripe/create-checkout/route.ts`
- Event generation: `lib/generators/event-generator.ts`
- Finance queries: `lib/queries/finance-queries.ts`
- Sales queries: `lib/queries/sales-queries.ts`

---

### For Planning Work (PM, PO, SM)

**READ:**
- `docs/planning/post-brother-poc-roadmap.md` - Current roadmap (GO/CHANGE/NO-GO scenarios)
- `docs/brief.md` - Original project vision
- `docs/prd.md` - Product requirements

**REFERENCE:**
- `docs/stories/*.md` - All 29 completed user stories (historical record)
- `docs/qa/gates/*.yml` - QA gate definitions (all passed)

---

### For Marketing & Content Work (Sophie, Communication)

**READ THESE FIRST (MANDATORY):**
1. `docs/marketing/CRITICAL-GUIDELINES.md` ⚠️ **NON-NEGOTIABLE**
2. `docs/PROJECT-CONTEXT.md` (verify facts)
3. `docs/_masemIT/readme.md` (know what NOT to say)

**THEN READ:**
- `docs/marketing/launch/linkedin-posts.md` - Corrected launch posts (learn from mistakes)
- `docs/marketing/launch/demo-video-script.md` - Video script

**ALWAYS VERIFY:**
- Pricing from `components/landing/PricingSection.tsx` (code is source of truth)
- Features from actual codebase (grep for functionality, don't assume)
- IBCS© compliance: Say "inspired by IBCS©", NEVER "IBCS-compliant"

**PRE-FLIGHT PROTOCOL (MANDATORY):**
Before creating ANY public content:
1. Read CRITICAL-GUIDELINES.md ✅
2. Verify all facts against codebase ✅
3. Get cross-agent review (River + one other agent) ✅
4. Present as DRAFT to user ✅
5. Get explicit approval before publishing ✅

---

### For UX & Design Work (Sally)

**READ:**
- `docs/design/icon-system.md` - Icon design system
- `docs/standards/ibcs-standards.md` - IBCS© reference (charts)
- `docs/architecture.md` - Understanding data flow helps UX

**BRAND COLORS (MANDATORY):**

| Color | Tailwind Class | Usage |
|-------|----------------|-------|
| **Primary Blue** | `text-blue-600` / `bg-blue-600` | Primary actions, features, links |
| **Success Green** | `text-green-500` / `bg-green-500` | Success states, CTA buttons, checkmarks |
| **Error Red** | `text-red-600` | Errors, problems, warnings |
| **Warning Yellow** | `text-yellow-500` | Caution, alerts |
| **Neutral Gray** | `text-gray-600` | Secondary text, borders |

**⚠️ DO NOT USE:** Amber, orange, brown, or any colors not listed above.

**REFERENCE:**
- Completed stories in `docs/stories/` for design decisions

---

### For Analysis & Research (Mary)

**READ:**
- `docs/brief.md` - Original project vision
- `docs/prd.md` - Product requirements
- `docs/planning/post-brother-poc-roadmap.md` - Current strategy

**OPTIONAL (Historical Context):**
- `docs/archive/early-planning/brainstorming-session-results.md`
- `docs/archive/market-research/` - Early market validation

---

## ❌ DO NOT READ (Outdated / Archived)

**These files are OUTDATED and will mislead you:**
- ❌ `docs/week-1-checklist.md` - Old planning (project finished)
- ❌ `docs/archive/early-planning/*` - Historical only
- ❌ `docs/archive/market-research/*` - Historical only
- ❌ Old `docs/go-to-market-plan.md` (superseded by post-brother-poc-roadmap)
- ❌ `docs/brainstorming-session-results.md` (moved to archive)
- ❌ `docs/competitor-analysis.md` (moved to archive)

**If you're unsure, check with River before reading any doc.**

---

## 🎯 Pre-Flight Protocol (Critical for High-Risk Tasks)

### When to Use Pre-Flight

**MANDATORY for:**
- Marketing content (LinkedIn, blog posts, press releases)
- Legal documents (terms, privacy policy)
- Public-facing materials (website copy, demo videos)
- Deployment & infrastructure changes
- Financial/pricing changes

**NOT needed for:**
- Internal code changes (reviewed via PR)
- Documentation updates (internal)
- Exploratory research
- Planning discussions

### Pre-Flight Steps

**Step 1: Discover Critical Context**
```
✅ Search for CRITICAL-GUIDELINES.md
✅ Read recent related work
✅ Check actual implementation (code, not assumptions)
```

**Step 2: Verify Assumptions**
```
✅ Pricing: Read components/landing/PricingSection.tsx
✅ Features: Grep codebase for actual capabilities
✅ Legal/Trademark: Check CRITICAL-GUIDELINES.md
✅ Privacy: No personal info in public content
```

**Step 3: Cross-Agent Review**
```
✅ Orchestrator (River) reviews before user sees
✅ Fact-checker (Mary) validates claims
✅ Minimum 2 agents verify before presenting
```

**Step 4: User Approval Gate**
```
✅ Present content as DRAFT
✅ Highlight assumptions made
✅ Get explicit approval before finalizing
```

**See:** `docs/bmad-contributions/pr-02-agent-preflight-protocol.md` for full specification

---

## 🔥 Ring of Fire (ROF) Sessions

**What:** Multi-agent collaborative sessions that run in parallel while user works on other tasks

**How to Join:**
```bash
*rof "<topic>" --agents <agent-list> [--report brief|detailed|live]
```

**Example:**
```bash
*rof "Next steps for tellingCube launch" --agents mary,winston,quinn --report brief
```

**Rules:**
- Agents collaborate freely within session
- Request user approval when tools needed (read files, run commands)
- Default report: brief (just recommendations)
- User chooses scope and granularity

**See:** `docs/_masemIT/readme.md` for details, `docs/bmad-contributions/pr-01-ring-of-fire.md` for specification

---

## 📋 Quick Reference Card

### Key Facts (Copy-Paste Ready)
```
Project: tellingCube
Status: MVP complete, launch ready, video recorded
Tech: Next.js 15, React 19, NeonDB, Claude API, Stripe
Pricing: €29-€999 lifetime (one-time, no subscriptions)
IBCS: Say "inspired by IBCS©" (NEVER "compliant")
Sectors: Consumer Staples, Financials, Industrials
Launch: Domain connection pending, then GO
```

### Emergency Contacts
```
Orchestrator: River
Project Lead: Mario Semper (Sempre)
Validation Partner: Anonymous (controller with 10+ years)
```

### Critical Files (Source of Truth)
```
Pricing: components/landing/PricingSection.tsx
Architecture: docs/architecture.md
Rules: docs/marketing/CRITICAL-GUIDELINES.md
Context: docs/PROJECT-CONTEXT.md
```

---

## ✅ Onboarding Checklist

Before starting work, confirm you've read:

**Everyone:**
- [ ] `docs/AGENT-BRIEFING.md` (this file)
- [ ] `docs/PROJECT-CONTEXT.md`
- [ ] `docs/_masemIT/readme.md`

**If doing public work (marketing, legal, public content):**
- [ ] `docs/marketing/CRITICAL-GUIDELINES.md` ⚠️ **MANDATORY**
- [ ] Verified pricing from code (`PricingSection.tsx`)
- [ ] Verified features from codebase (grep/read actual code)
- [ ] Pre-flight protocol steps completed
- [ ] Cross-agent review requested
- [ ] Presenting as DRAFT for user approval

**If doing technical work:**
- [ ] `docs/architecture.md`
- [ ] `docs/architecture/tech-stack.md`
- [ ] `docs/architecture/coding-standards.md`
- [ ] Read actual code (don't assume)

**If doing planning work:**
- [ ] `docs/planning/post-brother-poc-roadmap.md`
- [ ] `docs/brief.md` or `docs/prd.md` (choose one)

---

## 🚨 Common Mistakes to Avoid

### ❌ Mistake #1: Assuming Pricing
**What happened:** Sophie assumed €9/month pricing (wrong!)
**Reality:** €29-€999 lifetime (one-time payment)
**Fix:** ALWAYS read `components/landing/PricingSection.tsx`

### ❌ Mistake #2: Violating IBCS© Trademark
**What happened:** Used "IBCS-compliant" everywhere
**Reality:** Must say "inspired by IBCS©" (trademark compliance)
**Fix:** Read CRITICAL-GUIDELINES.md before any marketing work

### ❌ Mistake #3: Exposing Personal Info
**What happened:** Mentioned "Product Owner at insurance company", "my brother"
**Reality:** Privacy/legal risk - forbidden in public content
**Fix:** Read CRITICAL-GUIDELINES.md section on privacy

### ❌ Mistake #4: Reading Outdated Docs
**What happened:** Used old market research or week-1 checklist
**Reality:** Project evolved, those docs outdated
**Fix:** Check `docs/archive/` folder - don't use those for current work

### ❌ Mistake #5: No Pre-Flight Verification
**What happened:** Created public content without verifying facts
**Reality:** Multiple critical errors in one LinkedIn post
**Fix:** Follow pre-flight protocol for all high-risk tasks

---

## 🎓 Learning from Past Incidents

### Incident #1: LinkedIn Post Violations (2025-11-22)
**What:** Sophie created 3 LinkedIn post variants with multiple critical errors
**Errors:**
- Wrong pricing (€9/month vs €29-€999 lifetime)
- Privacy violations (mentioned day job, family)
- Trademark violations (used "IBCS-compliant")
- Factual errors (claimed "60 seconds" generation)

**Root Cause:** No pre-flight protocol existed

**Fix:**
- Created CRITICAL-GUIDELINES.md
- Fixed all marketing materials
- Documented pre-flight protocol
- Contributed protocol to BMad Method (PR #965)

**Lesson:** NEVER create public content without verification, even if user "mentioned it many times"

---

## 🤝 Your Commitment

By reading this briefing, you commit to:
- ✅ Reading mandatory documents before starting work
- ✅ Following pre-flight protocol for high-risk tasks
- ✅ Verifying facts against code (not assumptions)
- ✅ Asking River if unsure what to read
- ✅ Presenting work as DRAFT for user approval (high-risk tasks)
- ✅ Never exposing personal/confidential information

---

## 📞 Need Help?

**If unsure what to read:** Ask River (BMad Orchestrator)
**If technical question:** Ask Winston (Architect) or James (Dev)
**If planning question:** Ask Mary (Analyst) or John (PM)
**If marketing question:** Ask Sophie (Marketing) BUT require pre-flight verification

---

**Welcome to the team! Now let's build great things together.** 🚀

---

**Document Version:** 1.1
**Last Updated:** 2025-12-14
**Maintained By:** River
**Update Frequency:** When onboarding process changes
