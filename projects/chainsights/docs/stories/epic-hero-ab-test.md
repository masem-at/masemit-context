# Epic: Hero Section A/B Test

**Status:** Completed
**Priority:** High
**Created:** 2026-01-31
**Source:** docs/_masemIT/chainsights-hero-ab-test.md (full PRD by Mario)

## Problem Statement

Homepage hero has 0% CTR (60 CTA views, 0 clicks). Too many competing options (5 clickable elements). LinkedIn visitors from content marketing posts want to see analysis, not start their own. Need data to decide between current approach and exploration-first CTA.

## Solution

Client-side A/B test with cookie-based variant assignment (50/50 split). Variant A = current hero (control). Variant B = exploration-first hero pointing to Matrix. Distinct event names per variant for clean comparison.

---

## Story A: Create useABTest hook

**File:** `src/lib/hooks/useABTest.ts` (new)

### Acceptance Criteria

- [x] AC1: Export `useABTest(testName: string)` hook that returns `{ variant: 'a' | 'b' }`
- [x] AC2: On first call, check for cookie `cs_hero_variant`
- [x] AC3: If cookie exists, return stored variant
- [x] AC4: If no cookie, randomly assign 'a' or 'b' (50/50 split via `Math.random()`)
- [x] AC5: Set cookie: name=`cs_hero_variant`, max-age=2592000 (30 days), path=`/`, SameSite=Lax, Secure in production
- [x] AC6: Fire `hero_variant_assigned` event with `{ variant }` on first assignment only
- [x] AC7: Fire `hero_variant_loaded` event with `{ variant }` on every render

---

## Story B: Extract current hero as HeroVariantA

**File:** `src/components/hero/HeroVariantA.tsx` (new)

### Acceptance Criteria

- [x] AC1: Extract current `Hero` component content into `HeroVariantA`
- [x] AC2: Replace event names with `a_` prefix:
  - `cta_view` → `a_hero_free_check_view` (viewport entry)
  - `cta_click` (free check) → `a_hero_free_check_click`
  - `cta_click` (matrix link) → `a_hero_matrix_link_click`
- [x] AC3: Add new events: `a_hero_sample_report_click` (sample PDF link), `a_hero_quiz_click` (quiz link)
- [x] AC4: All events include `{ variant: 'a', page: '/' }` properties
- [x] AC5: Visual appearance and functionality identical to current hero

---

## Story C: Create HeroVariantB

**File:** `src/components/hero/HeroVariantB.tsx` (new)

### Acceptance Criteria

- [x] AC1: Same badge, headline ("Wallets lie. We don't."), and subheadline as Variant A
- [x] AC2: Primary CTA (filled button): "See Which DAOs Are Failing →" → links to `/matrix`
- [x] AC3: Secondary CTA (outlined button): "Read a Sample Report" → links to `/chainsights-sample-report.pdf` (target=_blank)
- [x] AC4: Tertiary link (small text): "Or check your own DAO for free" → opens ReportSelectionModal
- [x] AC5: Subtext: "47 DAOs analyzed · Updated weekly · 3% donated to hoki.help" + "✓ No signup required"
- [x] AC6: Remove quiz link (reduce decision paralysis)
- [x] AC7: Keep ENS quote, donation counter, scroll indicator
- [x] AC8: Events with `b_` prefix:
  - `b_hero_matrix_cta_view` (viewport entry)
  - `b_hero_matrix_cta_click` (primary CTA)
  - `b_hero_sample_report_click` (secondary CTA)
  - `b_hero_free_check_click` (tertiary link)
- [x] AC9: All events include `{ variant: 'b', page: '/' }` properties
- [x] AC10: Mobile: buttons stacked vertically, primary on top

---

## Story D: Create HeroSection wrapper and replace hero

**Files:** `src/components/hero/HeroSection.tsx` (new), `src/components/hero.tsx` (modify or redirect)

### Acceptance Criteria

- [x] AC1: `HeroSection` reads variant from `useABTest('hero_variant')`
- [x] AC2: Renders `HeroVariantA` if variant is 'a', `HeroVariantB` if 'b'
- [x] AC3: No layout shift — both variants occupy same space
- [x] AC4: Update homepage import to use new `HeroSection` instead of `Hero`
- [x] AC5: Old `hero.tsx` can be deleted or re-export from new structure

---

## Implementation Order

1. **Story A** — useABTest hook (no dependencies)
2. **Story B** — Extract Variant A (depends on A for events)
3. **Story C** — Create Variant B (depends on A for events)
4. **Story D** — Wrapper + integration (depends on B + C)

## Out of Scope

- Server-side A/B testing
- Statistical significance calculation
- Changes to pages other than homepage hero
- Analytics dashboard UI (manual SQL evaluation)
- Multivariate testing (only 2 variants)
