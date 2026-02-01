# Story: IBCS Landing Page Implementation

**Story ID:** IBCS-001
**Type:** Enhancement
**Priority:** High
**Status:** Ready for Dev
**Sprint:** Pre-Product Hunt Launch (Dec 21-Jan 8, 2026)
**Estimate:** 4-6 hours

---

## Story Description

As a **IBCS certification alumni**
I want to **land on a dedicated page that speaks to my specific pain point**
So that **I can quickly understand how tellingCube solves the certification-to-implementation gap**

## Context

After feedback from IBCS Institute CEO (Jürgen Faisst), we're implementing dual market positioning:
- **Primary (80%):** Broad market via homepage
- **Secondary (20%):** IBCS practitioners via `/ibcs` landing page

This story implements the IBCS-specific landing page while maintaining brand consistency and legal compliance with IBCS© trademark usage.

---

## Acceptance Criteria

### AC1: New Route Created
- [x] Route `/ibcs` exists and renders IBCS landing page
- [x] Page uses Next.js App Router (`app/ibcs/page.tsx`)
- [x] Page loads without errors (TypeScript compilation clean)
- [x] Route is accessible via direct URL and internal navigation

### AC2: IBCS-Specific Hero Section
- [x] Hero headline targets IBCS practitioners: "From IBCS Certification to Implementation in Minutes"
- [x] Subheadline addresses pain: "Spent €2K on certification, still building Excel mockups?"
- [x] Hero includes IBCS-style visualization (grayscale, professional) - Placeholder ready for Maya's final image
- [x] CTA button: "Start Implementing Today" (not generic "Try for Free")

### AC3: Trademark Compliance
- [x] All references use "IBCS©-inspired" (not "IBCS-compliant")
- [x] Page title: "tellingCube for IBCS Practitioners"
- [x] Footer includes disclaimer:
  > "IBCS© is a registered trademark of the IBCS Institute. tellingCube is an independent tool providing IBCS©-inspired visualizations and is not affiliated with, endorsed by, or certified by the IBCS Institute."
- [x] No language suggesting official certification or endorsement

### AC4: Content Structure
- [x] **Pain Section:** Describes certification-to-implementation gap
- [x] **Solution Section:** Zero-config data + instant visualization
- [x] **Benefits:** Controller-validated, event-sourced consistency
- [x] **Social Proof:** Reference IBCS CEO feedback (placeholder - pending permission from Jürgen Faisst)
- [x] **Use Cases:** Consulting firms, corporate controllers, trainers
- [x] **Pricing:** Emphasize Team Plus (€599) and Department Partner (€999) tiers - Uses existing PricingSection

### AC5: Links to Existing Product Flow
- [x] CTA button routes to pricing section (#pricing anchor)
- [x] Pricing section uses existing PricingSection component
- [x] Page feels like natural part of tellingCube, not separate product

### AC6: UTM Tracking
- [x] Page structure supports `?utm_campaign=ibcs` parameter (Next.js automatic)
- [ ] **PENDING**: Analytics tracking configuration (requires Mario to set up analytics events)
- [x] Links preserve URL parameters through Next.js Link component

### AC7: Mobile Responsive
- [x] Page uses responsive Tailwind classes (sm:, md:, lg:)
- [x] Hero text uses responsive font sizes (text-4xl md:text-5xl lg:text-6xl)
- [x] CTA buttons use min-h-[44px] for touch-friendly sizing
- [x] Images use responsive containers with max-width

### AC8: AEO/LLMO Optimization
- [x] Structured data markup (JSON-LD) for SoftwareApplication
- [x] Semantic HTML5 tags (article, section, main)
- [x] Clear content hierarchy (H1 "From IBCS..." → H2 sections → H3 subsections)
- [x] Meta description optimized for AI understanding (160 chars) in layout.tsx
- [x] Schema.org markup for Organization and SoftwareApplication
- [x] Content uses natural language (not keyword-stuffed)
- [x] All placeholder text includes context for LLM understanding
- [ ] **PENDING**: FAQ section with schema markup (can be added post-launch if needed)

---

## Technical Implementation Notes

### Component Reuse
- Use existing `PricingSection` component
- Use existing `ScenarioSelector` routing
- Create new `IBCSHero` component
- Create new `IBCSPainPoint` component

### Content Source
- Copy to be provided by Maya (Marketing)
- Must follow `docs/marketing/CRITICAL-GUIDELINES.md` compliance rules
- Trademark language reviewed by Daniel (Legal)

### File Structure
```
app/
  ibcs/
    page.tsx          # Main landing page
    layout.tsx        # Optional: IBCS-specific metadata
components/
  ibcs/
    IBCSHero.tsx      # Hero section
    IBCSPainPoint.tsx # Pain/solution section
```

---

## Definition of Done

- [ ] All 8 acceptance criteria met
- [ ] Page deployed to production
- [ ] Mobile responsive tested
- [ ] UTM tracking verified in analytics
- [ ] Trademark disclaimer reviewed
- [ ] AEO/LLMO markup validated
- [ ] Page loads in <2 seconds
- [ ] No console errors
- [ ] Peer review completed

---

## Dependencies

- Copy from Maya (Marketing) - **BLOCKER**
- Legal review of trademark language (Daniel)
- Existing components: PricingSection, ScenarioSelector

---

## Testing Checklist

### Functional Testing
- [ ] Route accessible via browser
- [ ] All internal links work
- [ ] CTA routes to correct destination
- [ ] UTM parameters persist through flow

### Visual Testing
- [ ] Hero renders correctly
- [ ] IBCS visualizations display properly
- [ ] Responsive on mobile/tablet/desktop
- [ ] Brand consistency with main site

### Compliance Testing
- [ ] Trademark language correct ("inspired by")
- [ ] Disclaimer present in footer
- [ ] No false claims of certification

### Performance Testing
- [ ] Page loads <2s on 4G connection
- [ ] Images optimized (WebP/Next.js Image)
- [ ] No layout shift (CLS < 0.1)

### AEO/LLMO Testing
- [ ] Structured data validates (Google Rich Results Test)
- [ ] Meta tags present and accurate
- [ ] Semantic HTML validates (W3C)
- [ ] Content readable by screen readers

---

## Story Status

**Current Status:** ✅ IMPLEMENTED (Pending Content from Maya)
**Assigned To:** Barry (Quick Flow Solo Dev)
**Created:** 2025-12-21
**Target Completion:** 2025-12-23
**Actual Completion:** 2025-12-21 (Scaffolding Complete)

---

## Notes

- This is part of dual positioning strategy before Product Hunt launch (Jan 8, 2026)
- Coordinate with Story IBCS-002 (Product Hunt listing update)
- Maya will draft landing page copy during implementation
- Keep it simple - this is a marketing page, not a product feature
