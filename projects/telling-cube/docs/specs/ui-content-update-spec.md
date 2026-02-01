# UI Content Update Specification
## Two-Phase Architecture Transition

**Prepared by:** Sally (UX Architect) & Sophie (Marketing)
**Date:** 2025-12-12
**Status:** Approved

---

## 1. HERO SECTION (`components/landing/Hero.tsx`)

### Change 1.1 - Headline (Line 5-6)

**Current:**
```tsx
<h1 className="text-4xl md:text-5xl lg:text-6xl font-bold text-gray-900 mb-6 leading-tight">
  Realistic business data in minutes, not hours
</h1>
```

**Proposed:**
```tsx
<h1 className="text-4xl md:text-5xl lg:text-6xl font-bold text-gray-900 mb-6 leading-tight">
  Enterprise-quality business data in 2 minutes
</h1>
```

**Rationale:** "minutes not hours" was appropriate for 10+ min generation. Now with ~120s, we can make a bolder, more specific claim.

---

### Change 1.2 - Subheadline (Line 9-11)

**Current:**
```tsx
<p className="text-base md:text-lg lg:text-xl text-gray-600 max-w-2xl mx-auto leading-relaxed">
  Generate 3 years of realistic, multi-departmental business data for training, demos, and strategy sessions. Every view draws from the same event stream—guaranteed mathematical consistency.
</p>
```

**Proposed:**
```tsx
<p className="text-base md:text-lg lg:text-xl text-gray-600 max-w-2xl mx-auto leading-relaxed">
  Generate 12 months of realistic, multi-departmental business data for training, demos, and strategy sessions. Every view draws from the same event stream—guaranteed mathematical consistency.
</p>
```

**Rationale:** Data range is now 12 months, not 3 years.

---

## 2. LOADING PAGE (`app/generate/loading/page.tsx`)

### Change 2.1 - Stage Labels (Lines 20-64) - HIGH PRIORITY

**Current:**
```tsx
const STAGE_LABELS: Record<string, string[]> = {
  largecap: [
    'Setting up company structure...',           // stage1_generating (10%)
    'Generating business trends...',              // stage2_generating (30%)
    'Creating monthly transactions...',           // stage3 (50-90%)
    'Storing events in database...',              // database_insert (90%)
    'Running consistency validation...'           // validation (95%)
  ],
  // ... similar for midcap, startup
}
```

**Proposed:**
```tsx
const STAGE_LABELS: Record<string, string[]> = {
  largecap: [
    'Building company world...',                  // phase1 - World Building
    'Charting monthly business course...',        // phase2 - Monthly Course
    'Deriving financial events...',               // phase3 - Event Derivation
    'Storing events in database...',              // database_insert
    'Running consistency validation...'           // validation
  ],
  midcap: [
    'Building company world...',
    'Charting monthly business course...',
    'Deriving financial events...',
    'Storing events in database...',
    'Running consistency validation...'
  ],
  startup: [
    'Building company world...',
    'Charting monthly business course...',
    'Deriving financial events...',
    'Storing events in database...',
    'Running consistency validation...'
  ],
}
```

**Rationale:** Aligns with new 3-phase architecture. Old labels were generic; new ones reflect actual process.

---

### Change 2.2 - Generation Description (Line 297-299)

**Current:**
```tsx
<p className="text-gray-600">
  Creating 3 years of realistic, mathematically consistent business data...
</p>
```

**Proposed:**
```tsx
<p className="text-gray-600">
  Creating 12 months of realistic, mathematically consistent business data...
</p>
```

---

### Change 2.3 - Time Claim (Line 376-378)

**Current:**
```tsx
<p className="text-center text-sm text-gray-600 pt-3 border-t border-blue-200">
  Doing this in Excel would take 3-4 hours. We&apos;ll have it ready in ~2 minutes.
</p>
```

**Status:** KEEP AS IS - Now accurate with ~120s generation time.

---

### Change 2.4 - Educational Tips (Lines 76-81)

**Current:**
```tsx
const EDUCATIONAL_TIPS = [
  'Your scenario includes AC (Actual), PY (Previous Year), and PL (Plan) data',
  'All charts are inspired by IBCS© standards for professional business reporting',
  'Sales and Finance views draw from the same event stream—guaranteed consistency',
  'Every scenario includes 365 days of transactions across multiple departments',
]
```

**Proposed:**
```tsx
const EDUCATIONAL_TIPS = [
  'Your scenario includes AC (Actual), PY (Previous Year), and PL (Plan) data',
  'All charts are inspired by IBCS© standards for professional business reporting',
  'Sales and Finance views draw from the same event stream—guaranteed consistency',
  'Each scenario includes 12 months of data across multiple regions and cost centers',
  'AI builds a complete company world with employees, customers, vendors, and products',
]
```

**Rationale:** Updated timeframe and added tip highlighting the World Building phase.

---

## 3. FAQ SECTION (`components/landing/FAQSection.tsx`)

### Change 3.1 - Update FAQ #1 "What is tellingCube?" (Lines 9-11)

**Current:**
```tsx
{
  question: "What is tellingCube?",
  answer: "tellingCube is an AI-powered tool that generates realistic, multi-departmental business data in minutes. It's designed for business trainers, professors, and consultants who need demo datasets for workshops and presentations. Unlike traditional data generation tools, tellingCube ensures that all data views (Sales, Finance) reconcile perfectly through event-sourced architecture."
}
```

**Proposed:**
```tsx
{
  question: "What is tellingCube?",
  answer: "tellingCube is an AI-powered tool that generates realistic, multi-departmental business data in about 2 minutes. It's designed for business trainers, professors, and consultants who need demo datasets for workshops and presentations. Unlike traditional data generation tools, tellingCube ensures that all data views (Sales, Finance) reconcile perfectly through event-sourced architecture."
}
```

---

### Change 3.2 - Add New FAQ "How does generation work?" (Insert after FAQ #2)

**Proposed (NEW):**
```tsx
{
  question: "How does generation work?",
  answer: "tellingCube uses a unique two-phase AI architecture. First, AI builds a complete 'world' for your company—employees, customers, vendors, products, regions, and cost centers. Then AI charts a monthly business course with revenue targets, headcount changes, and narrative events. Finally, code-based derivation transforms this guidance into thousands of mathematically consistent financial events. This approach guarantees realistic ratios while keeping generation under 2 minutes."
}
```

**Rationale:** This differentiates us from competitors and explains our USP.

---

## 4. WHAT'S NEXT FEATURES (`lib/constants/whats-next-features.ts`)

### Change 4.1 - Add New "NOW" Feature (Insert after 'ibcs-charts')

**Proposed:**
```tsx
{
  id: 'regions-costcenters',
  title: 'Regions & Cost Centers',
  description: 'Multi-region operations and departmental cost allocation',
  icon: Building2,
  status: 'now',
},
```

**Rationale:** New architecture includes regions and cost centers in World Building.

---

## SUMMARY TABLE

| File | Priority | Change Type | Status |
|------|----------|-------------|--------|
| `Hero.tsx` | MEDIUM | Tagline update, timeframe fix | Approved |
| `loading/page.tsx` | HIGH | Stage labels, timeframe, tips | Approved |
| `FAQSection.tsx` | MEDIUM | Speed claim, new FAQ | Approved |
| `whats-next-features.ts` | LOW | New feature addition | Approved |

---

## IMPLEMENTATION NOTES

1. All changes should be implemented in a single commit
2. Run `pnpm build` after changes to verify no TypeScript errors
3. Visual QA on landing page and loading page after deployment

---

**Approved by:** Sempre
**Date:** 2025-12-12
