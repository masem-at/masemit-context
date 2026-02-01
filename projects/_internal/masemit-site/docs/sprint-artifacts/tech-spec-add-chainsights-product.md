# Tech Spec: Add ChainSights Product to Landing Page

**Status:** Ready for Implementation
**Created:** 2025-12-17
**Complexity:** Small (1-2 hours)

## Overview

Add ChainSights as a second product card in the Products section of the masemIT landing page, following the existing tellingCube card pattern.

## Product Information

- **Name:** ChainSights
- **Website:** https://chainsights.com
- **Tagline/Badge:** TBD (e.g., "DAO Analytics" or "Web3 Product")
- **Description:** ChainSights delivers identity-first governance analytics for DAOs. We cut through the noise of anonymous wallets to reveal who actually controls voting power — because wallets lie, but data doesn't.
- **Logo:** `public/chainsights-logo.svg` (already exists)

## Technical Notes

### Logo Visibility Issue
The ChainSights logo has wallet nodes with `fill="#F5F7FA"` (light gray) designed for dark backgrounds. On the light product card background, these will be invisible.

**SVG Structure:**
- Wallet nodes (outer circles): `#F5F7FA` - THE PROBLEM
- Connection lines: `#4A89F3` (blue) with 0.6 opacity
- Central identity node: gradient `#00E5C0` → `#4A89F3`

**Solutions (pick one):**
1. **Dark container** - Wrap logo in dark bg (e.g., `bg-slate-800 rounded-xl`)
2. **Modify SVG** - Change wallet node fill to darker color (e.g., `#64748B` slate-500)
3. **Create variant** - New `chainsights-logo-light.svg` for light backgrounds

**Recommendation:** Option 1 (dark container) preserves original brand design.

## Implementation Tasks

- [ ] **Task 1:** Add ChainSights i18n entries to `messages/en.json`
  - Badge text
  - Description
  - CTA button text
  - Feature highlights (3 features like tellingCube)

- [ ] **Task 2:** Add ChainSights i18n entries to `messages/de.json`
  - Same structure as English

- [ ] **Task 3:** Update `src/components/sections/products.tsx`
  - Add second product card following tellingCube pattern
  - Use `public/chainsights-logo.svg` via Next.js Image
  - Handle logo visibility issue (white circles)
  - Link to `https://chainsights.com` with UTM params

- [ ] **Task 4:** Update "Coming Soon" text
  - Remove ChainSights from `Products.comingSoon` since it's now listed
  - Keep tellingDash mention

- [ ] **Task 5:** Verify build and visual appearance
  - Run `npm run build`
  - Check both EN/DE locales
  - Verify logo displays correctly

## Files to Modify

| File | Changes |
|------|---------|
| `messages/en.json` | Add `Products.chainSights.*` entries |
| `messages/de.json` | Add `Products.chainSights.*` entries |
| `src/components/sections/products.tsx` | Add ChainSights card component |

## Acceptance Criteria

1. ChainSights card displays below tellingCube card
2. Card follows same visual pattern as tellingCube
3. Logo displays correctly (white circles visible)
4. External link opens chainsights.com with UTM tracking
5. Translations work for EN and DE
6. "Coming Soon" text updated to remove ChainSights
7. Build passes without errors

## Feature Highlights (Suggested)

Based on the product description, suggest 3 features:
1. **Identity Resolution** - See through anonymous wallets to real governance players
2. **Power Mapping** - Visualize who actually controls voting outcomes
3. **DAO Intelligence** - Data-driven insights for decentralized governance

*(Confirm with Mario during implementation)*

## UTM Parameters

```
https://chainsights.com?utm_source=masemit&utm_medium=site&utm_campaign=launch
```

## Reference

Current tellingCube implementation in `products.tsx:24-89` serves as the template.
