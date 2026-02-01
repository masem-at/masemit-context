# Story 2.9: Fix CTA Pricing Ambiguity

**Status:** done
**Epic:** 2 - Public Rankings Leaderboard (MVP)
**Story ID:** 2.9
**Created:** 2025-12-22
**Ultimate Context Engine Analysis:** Complete

---

## Story

**As a** user considering purchasing a report,
**I want** to see accurate pricing on the CTA button,
**So that** I'm not confused when I reach the checkout page.

---

## Acceptance Criteria

### Given-When-Then Format (from Epic 2)

**Given** the expanded DAO row displays the "Get Report" CTA
**When** the CTA renders
**Then** the button text accurately reflects the actual pricing structure:
  - **Option A (Single Tier):** "Get Full Report — €49 →" if only one price exists
  - **Option B (Tiered Pricing):** "See Report Options → from €49" if multiple tiers exist
  - **Option C (Promotional):** "Get Report — €49 Launch Special (reg. €49) →" if promotional pricing
**And** CTA price matches the price shown on the checkout/order page
**And** no user confusion in user testing about pricing
**And** conversion tracking shows no price-mismatch bounces
**And** pricing configuration is stored in environment variable or database for easy updates

**Implementation Notes (from Epic):**
- Addresses UX-P0-2 (Critical pricing fix)
- Decision needed from Mario on actual pricing structure
- Default to €49 single tier if not specified
- Component: Update `src/components/ExpandableDAORow.tsx` CTA section

---

## Critical Developer Context

### Current Implementation Analysis

**Current CTA Locations:**
1. **`ExpandableDAORow.tsx` (line 317):** "Get Report" - NO pricing shown
2. **`MobileDAOCard.tsx` (line 259):** "Get Report" - NO pricing shown
3. **`header.tsx` (line 43):** "Get Report — €49" - Shows pricing but links to pricing section
4. **`pricing.tsx` (lines 81, 148):** Shows €49 (Deep Dive) and €149 (Governance Audit)
5. **`hero.tsx` (line 54):** "Get Report" - NO pricing shown

**Problem Identified:**
- Leaderboard CTAs ("Get Report") link to `/quiz?dao={slug}` - user doesn't see pricing
- Header CTA shows €49 but only scrolls to pricing section
- Mismatch: Users expect €49 from header but leaderboard doesn't confirm
- Two pricing tiers exist (€49, €149) but leaderboard CTA is ambiguous

**Current Pricing Structure (from `pricing.tsx`):**
- **Deep Dive:** €49 (one-time) - Recent governance snapshot
- **Governance Audit:** €149 (one-time) - Complete historical analysis

**User Flow Gap:**
1. User sees "Get Report — €49" in header
2. User expands DAO row → sees "Get Report" (no price)
3. User clicks → goes to quiz/order flow
4. User sees pricing options (€49 or €149)
5. **CONFUSION:** "Wait, I thought it was €49?"

---

### Previous Story Intelligence (Stories 2.5-2.8)

**Story 2.8 - Accessibility** (in-review):
- Both ExpandableDAORow and MobileDAOCard updated for keyboard/screen reader
- Focus rings, ARIA attributes all in place
- Any CTA text changes must maintain accessibility

**Story 2.7 - Mobile Responsive** (in-review):
- MobileDAOCard has identical CTA structure to ExpandableDAORow
- Both need synchronized updates

**Story 2.5 - Expandable Row** (done):
- CTA buttons styled with `bg-navy text-white hover:bg-navy-dark`
- Button text is hardcoded, not configurable

---

### Architectural Considerations

**Option A: Hardcoded Single Tier (Simplest)**
- Change button text to "Get Report — from €49 →"
- Pros: Simple, matches header
- Cons: Requires code change for price updates

**Option B: Environment Variable**
- Add `NEXT_PUBLIC_STARTING_PRICE=99` to env
- Button text: `Get Report — from €{process.env.NEXT_PUBLIC_STARTING_PRICE} →`
- Pros: Easy updates via Vercel dashboard
- Cons: Still static at build time

**Option C: Database-driven Pricing Config**
- Create `pricing_config` table or JSON in config
- Fetch at runtime or include in Server Component props
- Pros: Most flexible, real-time updates
- Cons: Adds complexity, may need caching

**Recommended Approach:** Option B (Environment Variable)
- Matches the lightweight MVP approach
- Can upgrade to Option C later if needed
- Vercel env vars allow runtime updates without redeploy (if using Edge runtime)

---

## Implementation Strategy

### Phase 1: Add Pricing Configuration

Create a centralized pricing config that can be used across components:

```typescript
// src/lib/config/pricing.ts
export const PRICING_CONFIG = {
  standardTier: {
    id: 'standard',
    name: 'Standard',
    price: parseInt(process.env.NEXT_PUBLIC_STANDARD_PRICE || '99', 10),
    currency: '€',
  },
  deepDiveTier: {
    id: 'deep_dive',
    name: 'Deep Dive',
    price: parseInt(process.env.NEXT_PUBLIC_DEEP_DIVE_PRICE || '199', 10),
    currency: '€',
  },
  // Computed value for "from €X" display
  get startingPrice() {
    return Math.min(this.standardTier.price, this.deepDiveTier.price)
  },
  get displayText() {
    return `from ${this.standardTier.currency}${this.startingPrice}`
  }
}
```

### Phase 2: Update ExpandableDAORow CTA

```tsx
// src/components/ExpandableDAORow.tsx
import { PRICING_CONFIG } from '@/lib/config/pricing'

// In CTA section:
<a
  href={`/quiz?dao=${ranking.slug}`}
  className={cn(/* existing styles */)}
>
  See Report Options — {PRICING_CONFIG.displayText}
</a>
```

### Phase 3: Update MobileDAOCard CTA

Same pattern as ExpandableDAORow for consistency.

### Phase 4: Update Header CTA

```tsx
// src/components/header.tsx
import { PRICING_CONFIG } from '@/lib/config/pricing'

<Button size="sm" onClick={scrollToPricing}>
  Get Report — {PRICING_CONFIG.displayText}
</Button>
```

### Phase 5: Update Hero CTA

```tsx
// src/components/hero.tsx
import { PRICING_CONFIG } from '@/lib/config/pricing'

<Button asChild>
  <Link href="/#pricing">
    Get Report — {PRICING_CONFIG.displayText}
  </Link>
</Button>
```

---

## Files to Create/Modify

### 1. Create: `src/lib/config/pricing.ts`

Centralized pricing configuration.

**Requirements:**
- Export `PRICING_CONFIG` object
- Read from environment variables with sensible defaults
- Provide computed `startingPrice` and `displayText` helpers
- Currency symbol centralized

---

### 2. Modify: `src/components/ExpandableDAORow.tsx`

Update CTA button text with pricing.

**Changes:**
- Import `PRICING_CONFIG`
- Update button text from "Get Report" to "See Report Options — from €49"
- Keep href as `/quiz?dao=${ranking.slug}`

---

### 3. Modify: `src/components/MobileDAOCard.tsx`

Mirror ExpandableDAORow changes.

**Changes:**
- Import `PRICING_CONFIG`
- Update button text from "Get Report" to "See Report Options — from €49"
- Ensure touch target size remains ≥48px

---

### 4. Modify: `src/components/header.tsx`

Update header CTA to use config.

**Changes:**
- Import `PRICING_CONFIG`
- Change "Get Report — €49" to use `PRICING_CONFIG.displayText`

---

### 5. Modify: `src/components/hero.tsx`

Update hero CTA to match header.

**Changes:**
- Import `PRICING_CONFIG`
- Add pricing text to "Get Report" button
- Ensure consistent messaging

---

### 6. Create: `.env.example` update

Document new environment variables.

**Add:**
```
# Pricing Configuration
NEXT_PUBLIC_DEEP_DIVE_PRICE=49
NEXT_PUBLIC_GOVERNANCE_AUDIT_PRICE=149
```

---

### 7. Create: `tests/unit/lib/pricing-config.test.ts`

Unit tests for pricing configuration.

**Test Coverage:**
- Default values work without env vars
- startingPrice returns minimum
- displayText formats correctly
- Currency symbol is correct

---

### 8. Update: Component Tests

Add tests for CTA text rendering:
- `tests/unit/components/expandable-dao-row.test.tsx` - CTA shows pricing
- `tests/unit/components/mobile-dao-card.test.tsx` - CTA shows pricing

---

## Technical Requirements

### Pricing Display Requirements (UX-P0-2)

| Requirement | Implementation |
|-------------|---------------|
| No ambiguity | "from €X" clearly indicates options |
| Accuracy | Uses same config as pricing page |
| Consistency | Same text in header, hero, leaderboard |
| Maintainability | Single config source |

### Environment Variables

| Variable | Type | Default | Purpose |
|----------|------|---------|---------|
| `NEXT_PUBLIC_DEEP_DIVE_PRICE` | number | 49 | Deep Dive tier price |
| `NEXT_PUBLIC_GOVERNANCE_AUDIT_PRICE` | number | 149 | Governance Audit tier price |

### Component Updates Summary

| Component | Current CTA | New CTA |
|-----------|-------------|---------|
| ExpandableDAORow | "Get Report" | "See Report Options — from €49" |
| MobileDAOCard | "Get Report" | "See Report Options — from €49" |
| header.tsx | "Get Report — €49" | "Get Report — from €49" |
| hero.tsx | "Get Report" | "Get Report — from €49" |

---

## Testing Strategy

### Unit Tests (Vitest)

**Pricing Config:**
- Returns correct defaults
- Computes startingPrice correctly
- Formats displayText correctly
- Handles missing env vars gracefully

**Component Tests:**
- CTA button contains pricing text
- CTA links to correct destination
- Text is accessible (readable by screen readers)

### Manual Testing Checklist

**Visual Consistency:**
- [ ] Header CTA shows "from €49"
- [ ] Hero CTA shows "from €49"
- [ ] Expanded DAO row CTA shows "from €49"
- [ ] Mobile card CTA shows "from €49"
- [ ] Pricing page still shows correct tier prices

**User Flow:**
- [ ] Click leaderboard CTA → lands on quiz/pricing
- [ ] No "surprise" pricing at any point
- [ ] Click header CTA → scrolls to pricing section

**Accessibility:**
- [ ] CTA text readable by screen reader
- [ ] Focus indicator visible on all CTAs
- [ ] Contrast ratio ≥4.5:1 for CTA text

---

## Tasks/Subtasks

### Task 1: Create Pricing Configuration
- [ ] Create `src/lib/config/pricing.ts`
- [ ] Define PRICING_CONFIG with tiers
- [ ] Add computed startingPrice and displayText
- [ ] Write unit tests

### Task 2: Update ExpandableDAORow
- [ ] Import PRICING_CONFIG
- [ ] Update CTA button text
- [ ] Update unit tests

### Task 3: Update MobileDAOCard
- [ ] Import PRICING_CONFIG
- [ ] Update CTA button text
- [ ] Update unit tests

### Task 4: Update Header
- [ ] Import PRICING_CONFIG
- [ ] Update CTA to use displayText
- [ ] Verify scrollToPricing still works

### Task 5: Update Hero
- [ ] Import PRICING_CONFIG
- [ ] Update CTA button text
- [ ] Verify link destination

### Task 6: Documentation and Environment
- [ ] Update `.env.example` with new variables
- [ ] Add env vars to Vercel dashboard (if not using defaults)

### Task 7: Validate and Test
- [ ] TypeScript build succeeds
- [ ] All unit tests pass
- [ ] Manual visual inspection
- [ ] User flow testing

---

## Dev Agent Record

### Context Reference
<!-- Path(s) to story context XML will be added here by context workflow -->

### Agent Model Used
claude-opus-4-5-20251101

### Debug Log References
- None

### Completion Notes List
- 2025-12-22: Story drafted with Ultimate Context Engine analysis

### File List

**Files to Create:**
- `src/lib/config/pricing.ts`
- `tests/unit/lib/pricing-config.test.ts`

**Files to Modify:**
- `src/components/ExpandableDAORow.tsx`
- `src/components/MobileDAOCard.tsx`
- `src/components/header.tsx`
- `src/components/hero.tsx`
- `tests/unit/components/expandable-dao-row.test.tsx`
- `tests/unit/components/mobile-dao-card.test.tsx`
- `.env.example`

**Files Referenced:**
- `docs/epics.md` (Story 2.9 requirements, lines 1570-1595)
- `docs/sprint-artifacts/2-8-*.md` (previous story context)
- `src/components/pricing.tsx` (current pricing structure)

---

## Change Log

- **2025-12-22**: Story 2.9 created with Ultimate Context Engine analysis

---

## Definition of Done

- [ ] Pricing configuration module created and tested
- [ ] ExpandableDAORow CTA shows "See Report Options — from €49"
- [ ] MobileDAOCard CTA shows "See Report Options — from €49"
- [ ] Header CTA shows "Get Report — from €49"
- [ ] Hero CTA shows "Get Report — from €49"
- [ ] All CTA prices match pricing page
- [ ] Unit tests pass for pricing config
- [ ] Component tests updated for new CTA text
- [ ] TypeScript builds without errors (strict mode)
- [ ] Manual visual inspection complete
- [ ] User flow validated (no pricing confusion)
- [ ] All acceptance criteria validated

---

## Open Questions / Decisions Needed

### Question 1: CTA Button Text Style

**Options:**
1. "See Report Options — from €49" (emphasizes multiple options)
2. "Get Report — from €49" (consistent with header)
3. "View Pricing — from €49" (most explicit)

**Recommendation:** Option 2 - "Get Report — from €49"
- Matches existing header language
- "from" indicates there are options
- More action-oriented than "View Pricing"

### Question 2: Future Promotional Pricing

**If launch special pricing is added:**
- Text could be: "Get Report — €49 Launch Special"
- Would need additional env var: `NEXT_PUBLIC_PROMO_ACTIVE=true`
- Can be added in future story if needed

---

## References

**Epic 2 Story:** `docs/epics.md` lines 1570-1595
**Architecture:** `docs/architecture.md`
**Project Context:** `docs/project_context.md`
**Previous Stories:**
- `docs/sprint-artifacts/2-5-build-expandable-row-with-component-breakdown.md`
- `docs/sprint-artifacts/2-7-implement-mobile-responsive-layout.md`
- `docs/sprint-artifacts/2-8-add-keyboard-navigation-and-screen-reader-support.md`
**Related Components:**
- `src/components/ExpandableDAORow.tsx`
- `src/components/MobileDAOCard.tsx`
- `src/components/header.tsx`
- `src/components/hero.tsx`
- `src/components/pricing.tsx`
