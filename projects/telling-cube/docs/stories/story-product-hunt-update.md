# Story: Product Hunt Listing IBCS Update

**Story ID:** IBCS-002
**Type:** Enhancement
**Priority:** High
**Status:** Ready for Dev
**Sprint:** Pre-Product Hunt Launch (Dec 21-Jan 8, 2026)
**Estimate:** 30 minutes

---

## Story Description

As the **Product Hunt launch manager**
I want to **add IBCS practitioner messaging to the Product Hunt listing**
So that **IBCS certification alumni discover tellingCube as a solution to their implementation challenges**

## Context

Product Hunt listing currently targets broad market (BI developers, data scientists, consultants). We're adding ONE LINE to signal IBCS practitioners without losing broad appeal.

This is a minimal addition to existing Product Hunt draft (docs/marketing/launch/product-hunt-listing.md).

---

## Acceptance Criteria

### AC1: Add IBCS Line to Description
- [ ] In the "Perfect For" section, add after "Professors & Trainers":
  ```
  ✅ **IBCS Practitioners** - Go from certification to implementation instantly—no custom data pipelines needed.
  ```
- [ ] Line appears in correct section (between use cases)
- [ ] Formatting matches existing bullet points
- [ ] Trademark language uses "IBCS Practitioners" (not "IBCS-certified users")

### AC2: Update First Comment Template
- [ ] Add to docs/marketing/launch/product-hunt-listing.md (or create separate file)
- [ ] First comment should include:
  ```
  🎯 **For IBCS Practitioners:** If you've completed IBCS certification and struggle with implementation, I built something specifically for you: [link to /ibcs page]

  The gap between certification and real-world implementation is huge. tellingCube gives you instant IBCS©-inspired visualizations with zero data pipeline setup.
  ```
- [ ] Link points to `/ibcs` page with UTM tracking: `tellingcube.com/ibcs?utm_source=producthunt&utm_campaign=ibcs`

### AC3: Verify Trademark Compliance
- [ ] All references use "IBCS©" with copyright symbol
- [ ] Language uses "inspired by" not "compliant with"
- [ ] No claims of certification or endorsement

---

## Technical Implementation Notes

### Files to Update
1. **docs/marketing/launch/product-hunt-listing.md**
   - Add IBCS line to "Perfect For" section (around line 128)
   - Add first comment template section

2. **Verify Product Hunt Draft**
   - Check that description on actual PH draft includes this line
   - Update PH listing if already submitted

### Copy Review
- Maya (Marketing) will review final wording
- Daniel (Legal) confirms trademark usage

---

## Definition of Done

- [ ] IBCS line added to Product Hunt description
- [ ] First comment template created/updated
- [ ] Trademark language verified
- [ ] UTM tracking link tested
- [ ] Mario approves final copy
- [ ] Product Hunt draft updated (if already submitted)

---

## Dependencies

- Story IBCS-001 (IBCS landing page) must be deployed before PH launch
- `/ibcs` URL must be live for first comment link to work

---

## Testing Checklist

### Content Testing
- [ ] IBCS line reads naturally in context
- [ ] Link in first comment works
- [ ] UTM parameters capture correctly

### Compliance Testing
- [ ] Trademark language correct
- [ ] No false claims or endorsements

---

## Story Status

**Current Status:** Ready for Dev
**Assigned To:** Maya (Marketing) or Paige (Tech Writer)
**Created:** 2025-12-21
**Target Completion:** 2025-12-22
**Actual Completion:** TBD

---

## Notes

- This is a documentation update, not code change
- Quick win: 30 minutes max
- Coordinate with Story IBCS-001 deployment
- First comment template will be used on PH launch day (Jan 8, 2026)
