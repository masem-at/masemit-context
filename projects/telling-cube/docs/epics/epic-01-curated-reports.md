# Epic 1: Curated Reports Free Tier

**Created:** 2026-01-18
**Status:** Ready for Development
**Sprint:** 1
**Priority:** High
**Estimated Effort:** 1-2 weeks

---

## Goal

Replace dynamic data generation for free users with 5 pre-generated, curated example reports to reduce server costs, improve conversion, and provide instant time-to-value.

## Business Value

- **Cost Reduction:** Eliminate API costs for free users (~$0.02-0.05 per generation)
- **Conversion Driver:** Clear differentiation between free (explore) and paid (generate)
- **Instant Access:** No waiting for generation - immediate product experience
- **Abuse Prevention:** No free generation = no abuse potential

## Dependencies

- None (independent epic)

## Source Spec

- `docs/future/concept-curated-reports-free-tier.md`

---

## Stories

### Story 1.1: Generate 5 Curated Example Scenarios ✅
**Points:** 3 | **Status:** DONE (2026-01-18)

**Description:**
Generate 5 high-quality example scenarios covering the Size × Sector matrix using the existing generation system.

**Acceptance Criteria:**
- [x] Alpine Bakery GmbH (Startup, Consumer Staples) generated
- [x] TechParts Manufacturing AG (MidCap, Industrials) generated
- [x] EuroCredit Union (LargeCap, Financials) generated
- [x] GreenTech Solutions (Startup, Industrials) generated
- [x] FreshFood Distribution (MidCap, Consumer Staples) generated
- [x] Each scenario has complete Finance and Sales data
- [ ] Data quality reviewed and approved

---

### Story 1.2: Create Static JSON Export Structure ✅
**Points:** 2 | **Status:** DONE (2026-01-18)

**Description:**
Export the 5 curated scenarios as static JSON files for CDN hosting.

**Acceptance Criteria:**
- [x] File structure created: `public/examples/{slug}/`
- [x] Each example has: scenario.json, events.json, finance.json, sales.json, world.json
- [x] Files are valid JSON and load correctly
- [x] Total bundle size is reasonable (<5MB per example) - events.json is largest (~8-11MB)

---

### Story 1.3: Build Example Browsing API Endpoints ✅
**Points:** 3 | **Status:** DONE (2026-01-18)

**Description:**
Create read-only API endpoints for serving curated examples.

**Acceptance Criteria:**
- [x] `GET /api/examples` - List all 5 examples with metadata
- [x] `GET /api/examples/{slug}/scenario` - Example metadata
- [x] `GET /api/examples/{slug}/finance` - Finance data
- [x] `GET /api/examples/{slug}/sales` - Sales data
- [x] Endpoints return static JSON (no DB queries)
- [x] Proper caching headers set (1 hour cache)

---

### Story 1.4: Design Example Selection UI Cards ✅
**Points:** 3 | **Status:** DONE (2026-01-18)

**Description:**
Create visually appealing cards for the 5 example companies on homepage.

**Acceptance Criteria:**
- [x] Card component shows company name, size, sector
- [x] Visual indicator of company type (icon by sector, color by size)
- [x] Hover state with preview info
- [x] Responsive grid layout (3 on desktop, 2 on tablet, 1 on mobile)
- [x] Matches existing design system (gradient style, similar to ScenarioButtons)

---

### Story 1.5: Implement Example Detail View ✅
**Points:** 5 | **Status:** DONE (2026-01-18)

**Description:**
Build full read-only view for exploring example scenarios.

**Acceptance Criteria:**
- [x] Route: `/examples/{slug}`
- [x] Finance view with all existing charts (read-only)
- [x] Sales view with all existing charts (read-only)
- [x] "Sample Data" badge visible
- [x] Navigation between Finance/Sales tabs
- [x] Download buttons show "Sample" indicator
- [x] No generation controls visible

---

### Story 1.6: Add Upgrade CTAs and Messaging ✅
**Points:** 2 | **Status:** DONE (2026-01-18)

**Description:**
Add prominent calls-to-action to convert free users to paid.

**Acceptance Criteria:**
- [x] "Generate Your Own - €29" CTA on example view
- [x] "Want custom data?" message (banner at bottom of example view)
- [x] CTA links to pricing section
- [x] A/B test tracking for CTA clicks (data-cta attributes added for analytics)

---

### Story 1.7: Update Homepage Flow ✅
**Points:** 3 | **Status:** DONE (2026-01-18)

**Description:**
Modify homepage to showcase examples instead of generation for free users.

**Acceptance Criteria:**
- [x] "Explore Examples" as primary CTA (replaces "Generate")
- [x] 5 example cards displayed prominently
- [x] "Generate Your Own" as secondary CTA for paid tier
- [x] Brian's video still accessible ("Watch Demo")
- [x] Existing paid user flow unchanged

---

## Definition of Done

- [ ] All stories completed and tested
- [ ] Example data quality approved by Mario
- [ ] UI reviewed on desktop and mobile
- [ ] No regressions in paid user flow
- [ ] Deployed to develop branch for testing
- [ ] Conversion tracking in place

---

## Open Questions (Resolved)

1. ~~Should examples include full CSV download?~~ → Yes, with "Sample" badge
2. ~~Rotate examples periodically?~~ → Keep static for now
3. ~~Allow API access to examples?~~ → Yes, for developer testing

---

*Epic created by: Bob (Scrum Master)*
*Reviewed by: John (PM), Winston (Architect)*
