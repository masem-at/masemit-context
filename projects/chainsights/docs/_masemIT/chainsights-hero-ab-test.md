# Product Requirement: Hero Section A/B Test

**Project:** ChainSights (chainsights.one)  
**Priority:** High  
**Author:** Mario Semper  
**Date:** 2026-01-31  
**Version:** 1.0  

---

## Problem Statement

The ChainSights homepage receives traffic (38 unique visitors on Jan 31 alone), but the conversion funnel shows a critical drop-off at the hero section:

- **60 CTA views** (hero_dao_matrix: 30, hero_free_check: 30)
- **0 CTA clicks** (0.0% CTR)
- **31 Landing → 4 Explore (13%) → 0 Tier Modal → 0 Quick Check → 0 Checkout**

The hero section currently offers too many competing options (5 clickable elements), and the primary CTA "See Your DAO's Score — Free" implies a process that may deter visitors who arrived via content marketing and want to see results, not start one.

LinkedIn is the primary traffic source (7 tracked visits via UTM). These visitors come from posts about specific DAO governance insights (Arbitrum, Optimism, ENS). They want to **see analysis**, not **run their own**.

---

## Objective

Implement a randomized client-side A/B test on the hero section to determine which CTA strategy converts better. Measure with distinct event names per variant to enable clear comparison.

---

## Scope

### In Scope

1. Client-side A/B test framework (cookie-based, 50/50 split)
2. Two hero variants with distinct CTAs and event tracking
3. Variant assignment persistence (same user always sees same variant)
4. Analytics events with variant prefix for comparison

### Out of Scope

- Server-side A/B testing
- Multivariate testing (only 2 variants)
- Statistical significance calculation (manual evaluation)
- Changes to pages other than the homepage hero

---

## Functional Requirements

### FR1: Variant Assignment

- On first visit, randomly assign the user to Variant A or Variant B (50/50 split)
- Store assignment in a cookie (`cs_hero_variant`, value: `a` or `b`)
- Cookie duration: 30 days
- Same user must always see the same variant across visits
- If cookie already exists, use stored variant

### FR2: Variant A — Current Hero (Control)

Keep the existing hero section unchanged:

**Primary CTA (Button):**  
`See Your DAO's Score — Free` → links to Quick Check flow

**Secondary Links:**  
- `See Sample Report` → /chainsights-sample-report.pdf  
- `Or compare all DAOs in the Matrix →` → /matrix  
- `test your governance IQ` → /quiz  

**Subtext:**  
`Full reports from €49 · Free check included · 3% donated to hoki.help`  
`✓ Instant results · No signup required`

**Events (prefix `a_`):**

| Event Name | Trigger |
|------------|---------|
| `a_hero_free_check_view` | Hero section enters viewport |
| `a_hero_free_check_click` | Primary CTA button clicked |
| `a_hero_sample_report_click` | Sample Report link clicked |
| `a_hero_matrix_link_click` | Matrix link clicked |
| `a_hero_quiz_click` | Quiz link clicked |

### FR3: Variant B — Exploration-First Hero (Treatment)

Redesign the hero section to prioritize browsing existing analysis over starting a personal check:

**Primary CTA (Button):**  
`See Which DAOs Are Failing →` → links to /matrix

**Secondary CTA (Button, outlined/ghost style):**  
`Read a Sample Report` → /chainsights-sample-report.pdf

**Tertiary Link (small, below buttons):**  
`Or check your own DAO for free` → Quick Check flow

**Subtext:**  
`47 DAOs analyzed · Updated weekly · 3% donated to hoki.help`  
`✓ No signup required`

**Remove from Variant B:**
- Quiz link (reduces decision paralysis)
- Lido ticker animation in hero area (optional — keep if removal is complex)

**Events (prefix `b_`):**

| Event Name | Trigger |
|------------|---------|
| `b_hero_matrix_cta_view` | Hero section enters viewport |
| `b_hero_matrix_cta_click` | Primary CTA button clicked |
| `b_hero_sample_report_click` | Sample Report button clicked |
| `b_hero_free_check_click` | Free check link clicked |

### FR4: Shared Tracking

In addition to variant-specific events, track:

| Event Name | Properties | Trigger |
|------------|------------|---------|
| `hero_variant_assigned` | `{ variant: "a" \| "b" }` | First visit, variant assigned |
| `hero_variant_loaded` | `{ variant: "a" \| "b" }` | Hero section rendered |

---

## Technical Requirements

### TR1: Implementation Approach

- **Client-side only** — no server-side changes required
- Use a React component wrapper or hook (e.g., `useABTest('hero_variant')`)
- Cookie-based persistence (no localStorage due to artifact restrictions in dev, but production can use cookies)
- Render correct variant based on cookie value

### TR2: Suggested Component Structure

```
components/
  hero/
    HeroSection.tsx          ← Wrapper: reads variant, renders A or B
    HeroVariantA.tsx         ← Current hero (extract as-is)
    HeroVariantB.tsx         ← New hero
    useABTest.ts             ← Hook: manages cookie + assignment
```

### TR3: Cookie Specification

| Property | Value |
|----------|-------|
| Name | `cs_hero_variant` |
| Values | `a` or `b` |
| Max-Age | 30 days (2592000 seconds) |
| Path | `/` |
| SameSite | `Lax` |
| Secure | `true` (production) |

### TR4: Event Integration

Use existing `trackEvent()` function from masemIT Analytics. Events must include the variant as a property for cross-referencing:

```typescript
trackEvent('a_hero_free_check_click', { variant: 'a', page: '/' })
trackEvent('b_hero_matrix_cta_click', { variant: 'b', page: '/' })
```

---

## Variant B — Copy Specification

### Hero Headline & Subheadline

**Keep unchanged:**
- Badge: `Identity-First Analytics`
- Headline: `Wallets lie. We don't.`
- Subheadline: `Free governance health check. Identity-first analytics for DAOs. Know who really controls your governance — not just which wallets voted.`

### CTA Area (changed)

```
[  See Which DAOs Are Failing →  ]        ← Primary, filled button
[  Read a Sample Report          ]        ← Secondary, outlined button

Or check your own DAO for free             ← Tertiary, text link

47 DAOs analyzed · Updated weekly · 3% donated to hoki.help
✓ No signup required
```

### Design Notes

- Primary button: same style as current CTA (green/accent color, full width on mobile)
- Secondary button: outlined/ghost variant of same button style
- Tertiary link: small, subtle, same style as current "Or compare all DAOs in the Matrix →"
- Mobile: stack buttons vertically, primary on top
- Keep the ENS quote below the CTA area unchanged

---

## Success Criteria

### Primary Metric
- **Hero CTA Click-Through Rate** per variant
- Target: Variant B achieves >3% CTR (any CTA) vs. Variant A's current 0%

### Secondary Metrics
- Explore page conversion rate per variant (Landing → any sub-page)
- Downstream funnel progress (Tier Modal, Quick Check, Checkout)
- Bounce rate comparison

### Evaluation Timeline
- **Minimum runtime:** 4 weeks (target: ~1,000 total visitors per variant)
- **Early stop:** If one variant shows >5% CTR with 200+ visitors, consider early winner
- **Decision:** After evaluation period, implement winning variant permanently

---

## Analytics Dashboard Requirements

Add a simple comparison view to masemIT Analytics (or evaluate manually via database):

| Metric | Variant A | Variant B |
|--------|-----------|-----------|
| Visitors (unique) | count | count |
| Hero Views | count | count |
| Primary CTA Clicks | count | count |
| Secondary CTA Clicks | count | count |
| CTR (Primary) | % | % |
| CTR (Any CTA) | % | % |
| Funnel: Explore | count | count |
| Funnel: Quick Check | count | count |

Manual SQL queries are acceptable for MVP evaluation. No dashboard UI required.

---

## Privacy & Compliance

- Cookie is **functional** (variant assignment), not tracking → no consent banner required under GDPR
- No PII is stored in the cookie
- Event tracking follows existing analytics setup (already GDPR-compliant)
- Mention A/B testing in Privacy Policy under "Analytics" section (optional but recommended)

---

## Risks & Mitigations

| Risk | Impact | Mitigation |
|------|--------|------------|
| Low traffic delays statistical significance | Medium | Accept directional signals, extend runtime if needed |
| Cookie deletion resets variant | Low | Accept minor noise, most users won't clear cookies |
| Bots inflate one variant | Low | Filter known bot user agents in analysis |
| Variant B performs worse | Low | Easy rollback — just remove B component and cookie logic |

---

## Open Questions for Team

1. **Winston:** Can the existing `trackEvent()` handle additional properties (`variant`, `page`), or does the schema need updating?
2. **Winston:** Is there a global layout component where the cookie check should happen, or per-page?
3. **Sally (if applicable):** Any design input on the Variant B button layout — horizontal vs. stacked on desktop?

---

## Future Considerations

- If A/B test framework proves useful, extract `useABTest` hook as reusable utility for other pages (pricing, matrix landing)
- Consider server-side variant assignment via middleware for SEO-sensitive pages (not needed for hero section)
- Multivariate testing (3+ variants) if traffic increases significantly
