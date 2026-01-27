# Story 3.2: Add Legal Disclaimers and Opt-Out Policy

**Status:** done
**Epic:** 3 - Methodology Transparency & Legal Foundation
**Story ID:** 3.2
**Created:** 2025-12-22
**Ultimate Context Engine Analysis:** Complete

---

## Story

**As a** DAO representative or legal-conscious user,
**I want** to see legal disclaimers and understand opt-out rights,
**So that** I know the limitations of the scores and how to request removal.

---

## Acceptance Criteria

### Given-When-Then Format (from Epic 3)

**Given** I view the methodology page
**When** I scroll to the legal section
**Then** the page includes:
  - **Legal disclaimer**: "GVS scores represent ChainSights' opinion based on publicly available data, not statements of fact or investment advice"
  - **No endorsement**: "Rankings do not constitute endorsement or recommendation"
  - **Opt-out policy**: "DAOs may request removal from public rankings within 24 hours by submitting the opt-out form"
  - **Data accuracy**: "Scores calculated using best efforts; no guarantee of accuracy or completeness"
  - **GDPR compliance**: "We collect minimal data; see Privacy Policy for details"
  - **Contact information**: Email for questions/disputes (hello@chainsights.one)
**And** link to opt-out form: `/rankings/opt-out` (placeholder until Story 4.1 creates the form)
**And** link to privacy policy in footer (ALREADY EXISTS - verify only)
**And** disclaimers use plain language (avoid legalese where possible)
**And** section uses `<aside>` or similar semantic HTML

---

## Critical Developer Context

### EXISTING IMPLEMENTATION - REQUIRES ENHANCEMENT!

**Methodology page already has disclaimer sections** at `src/app/rankings/methodology/page.tsx`:
- Lines 367-388: Opt-Out Policy card
- Lines 390-428: Limitations & Disclaimers card

**CRITICAL GAPS TO FIX:**

| Gap | Current State | Required State |
|-----|---------------|----------------|
| "No endorsement" clause | NOT explicitly stated | Add: "Rankings do not constitute endorsement or recommendation" |
| GDPR reference | NOT included | Add: "We collect minimal data; see Privacy Policy for details" with link |
| Opt-out form link | Email only (hello@chainsights.one) | Add prominent link/button to `/rankings/opt-out` (placeholder until Story 4.1) |
| Reusable constants | Hard-coded in page | Move to `src/lib/constants/legal-disclaimers.ts` for reuse |
| Semantic HTML | Uses `<section>` | Already correct! |

### What ALREADY Works (No Changes Needed)

✅ **Footer Privacy Link:** `src/components/footer.tsx` lines 60-62 already has privacy link
✅ **Privacy Policy Page:** `/privacy` page exists with comprehensive GDPR content
✅ **Semantic HTML:** Methodology page uses `<section>` elements with aria-labels
✅ **Plain Language:** Current disclaimers avoid legalese
✅ **Contact Email:** hello@chainsights.one already displayed

### Previous Story Intelligence (Story 3.1)

**Key Learnings:**
- Methodology page uses Card components from shadcn/ui
- Server Component pattern (no 'use client')
- ISR caching added: `export const revalidate = 86400` (24 hours)
- Constants created for weights (GVS_WEIGHTS) - follow same pattern for disclaimers
- Semantic `<section>` elements with aria-labels for accessibility
- Code review emphasized: maintainability > duplication (use constants!)

**Established Patterns:**
- Constants at module level (e.g., GVS_WEIGHTS, METHODOLOGY_VERSION)
- Badge component for visual emphasis
- Card structure: CardHeader > CardTitle, CardContent
- Links styled with `text-aqua hover:underline`
- Warning colors: `border-yellow-500/30`, `text-yellow-400`

### Architecture Requirements

**File Locations:**
- **UPDATE:** `src/app/rankings/methodology/page.tsx` (enhance existing disclaimer sections)
- **CREATE:** `src/lib/constants/legal-disclaimers.ts` (new reusable constants file)

**Server Component Pattern:**
- No 'use client' directive
- Static content only
- ISR caching already configured (24 hours)

**shadcn/ui Components Used:**
- Card, CardContent, CardHeader, CardTitle from `@/components/ui/card`
- Badge from `@/components/ui/badge`
- AlertCircle icon from `lucide-react`

**Project Standards (from project_context.md):**
- File naming: kebab-case for utilities (`legal-disclaimers.ts`)
- Export pattern: Named exports from lib/ modules
- Constants: SCREAMING_SNAKE_CASE for module-level constants
- Comments: JSDoc for exported constants explaining purpose and usage

###Git Intelligence (Recent Patterns)

**Recent Commits:**
```
6c48058 chore: mark stories 2.7, 2.8, 2.10 as done
dad8d07 fix: code review fixes for stories 2.7, 2.8, 2.10
e6315f9 feat(story-2.10): add LastUpdatedBadge component with staleness warning
863b2a4 fix: correct GVSResult property names in seed-leaderboard script
```

**Commit Message Pattern:** `feat(story-X.Y): description` or `fix: description`

---

## Implementation Strategy

### Phase 1: Create Reusable Legal Constants

**File:** `src/lib/constants/legal-disclaimers.ts` (NEW)

Create comprehensive legal text constants that can be reused across the application:

```typescript
/**
 * Legal Disclaimers and Compliance Text
 *
 * Centralized legal text for consistency across the application.
 * Consult legal advisor before modifying.
 *
 * @module legal-disclaimers
 */

/**
 * Main legal disclaimer for GVS scores
 */
export const LEGAL_DISCLAIMER =
  "Rankings are educational tools based on publicly available data and should not be considered financial, legal, or investment advice. Scores reflect quantitative metrics and may not capture the full complexity of governance structures. Rankings are opinions, not statements of fact. We make no guarantees about accuracy or completeness."

/**
 * No endorsement clause
 */
export const NO_ENDORSEMENT_CLAUSE =
  "Rankings do not constitute endorsement or recommendation of any DAO, token, or governance practice."

/**
 * Data accuracy disclaimer
 */
export const DATA_ACCURACY_DISCLAIMER =
  "Scores are calculated using best efforts based on available data. We make no guarantees about accuracy, completeness, or timeliness of information."

/**
 * GDPR compliance statement
 */
export const GDPR_COMPLIANCE_STATEMENT =
  "We collect minimal data necessary for service delivery. For details on data collection, storage, and your rights, see our"
// Note: "Privacy Policy" link added in UI

/**
 * Opt-out policy text
 */
export const OPT_OUT_POLICY =
  "DAOs may request removal from public rankings at any time. Requests are processed within 24 hours, no questions asked."

/**
 * Contact information for legal inquiries
 */
export const LEGAL_CONTACT_EMAIL = "hello@chainsights.one"
```

### Phase 2: Update Methodology Page Disclaimer Sections

**File:** `src/app/rankings/methodology/page.tsx` (UPDATE)

**Changes Required:**

1. **Import legal constants at top:**
   ```typescript
   import {
     LEGAL_DISCLAIMER,
     NO_ENDORSEMENT_CLAUSE,
     GDPR_COMPLIANCE_STATEMENT,
     OPT_OUT_POLICY,
     LEGAL_CONTACT_EMAIL
   } from '@/lib/constants/legal-disclaimers'
   ```

2. **Update Opt-Out Policy section** (lines 367-388):
   - Keep existing structure
   - Add prominent button/link to `/rankings/opt-out`
   - Use OPT_OUT_POLICY constant for consistency
   - Add note: "Opt-out form coming soon" (until Story 4.1)

3. **Enhance Limitations & Disclaimers section** (lines 390-428):
   - Add NO_ENDORSEMENT_CLAUSE as new bullet point
   - Replace hard-coded disclaimer text with LEGAL_DISCLAIMER constant
   - Add GDPR_COMPLIANCE_STATEMENT with link to `/privacy`
   - Keep existing bullet points about perspective, scores, metrics

### Phase 3: Verify Footer Privacy Link

**File:** `src/components/footer.tsx` (VERIFY ONLY - NO CHANGES)

- Confirm privacy link exists (lines 60-62) ✅
- No changes needed

---

## Tasks/Subtasks

### Task 1: Create Legal Disclaimers Constants File
- [x] Create file `src/lib/constants/legal-disclaimers.ts`
- [x] Add JSDoc module comment explaining purpose
- [x] Export LEGAL_DISCLAIMER constant
- [x] Export NO_ENDORSEMENT_CLAUSE constant
- [x] Export DATA_ACCURACY_DISCLAIMER constant
- [x] Export GDPR_COMPLIANCE_STATEMENT constant
- [x] Export OPT_OUT_POLICY constant
- [x] Export LEGAL_CONTACT_EMAIL constant

### Task 2: Update Opt-Out Policy Section in Methodology Page
- [x] Import legal constants at top of methodology page
- [x] Update opt-out policy text to use OPT_OUT_POLICY constant
- [x] Add button/link to `/rankings/opt-out` (styled as primary CTA)
- [x] Add temporary note: "Opt-out form coming soon - for now, email hello@chainsights.one"
- [x] Keep existing email contact as fallback

### Task 3: Enhance Limitations & Disclaimers Section
- [x] Add NO_ENDORSEMENT_CLAUSE as new bullet point (after "ONE perspective")
- [x] Replace hard-coded disclaimer text with LEGAL_DISCLAIMER constant
- [x] Add GDPR compliance bullet with link to `/privacy`
- [x] Ensure all text uses plain language (avoid legalese)
- [x] Maintain semantic HTML (`<section>` with aria-labels)

### Task 4: Verify Footer Privacy Link
- [x] Confirm privacy link exists in footer component
- [x] Verify link points to `/privacy`
- [x] Test link navigation works

### Task 5: Build and Validate
- [x] TypeScript build succeeds
- [x] Page renders correctly at /rankings/methodology
- [x] All disclaimer text displays properly
- [x] Privacy policy link in footer works
- [x] Opt-out section has placeholder link
- [x] No console errors or warnings

---

## Testing Strategy

### Manual Testing Checklist

**Legal Content Verification:**
- [ ] Legal disclaimer displays complete text
- [ ] "No endorsement" clause visible
- [ ] GDPR compliance statement present with privacy link
- [ ] Opt-out policy clearly stated
- [ ] Contact email displayed (hello@chainsights.one)
- [ ] Text uses plain language (readable by non-lawyers)

**Navigation:**
- [ ] Privacy link in footer navigates to `/privacy`
- [ ] Opt-out button/link points to `/rankings/opt-out` (placeholder)
- [ ] Contact email is clickable (`mailto:` link)

**Code Quality:**
- [ ] Legal constants imported from centralized file
- [ ] No hard-coded legal text in components
- [ ] JSDoc comments present on constants
- [ ] TypeScript types correct

**Accessibility:**
- [ ] Semantic HTML maintained (`<section>` elements)
- [ ] Screen reader can navigate sections
- [ ] Links have appropriate aria-labels if needed
- [ ] Color contrast meets WCAG AA standards

---

## Technical Requirements

### Legal Constants File Structure

```typescript
// src/lib/constants/legal-disclaimers.ts

/**
 * Legal Disclaimers and Compliance Text
 *
 * Centralized legal text for consistency across the application.
 * Consult legal advisor before modifying disclaimer text.
 *
 * @module legal-disclaimers
 */

export const LEGAL_DISCLAIMER: string
export const NO_ENDORSEMENT_CLAUSE: string
export const DATA_ACCURACY_DISCLAIMER: string
export const GDPR_COMPLIANCE_STATEMENT: string
export const OPT_OUT_POLICY: string
export const LEGAL_CONTACT_EMAIL: string
```

### Updated Methodology Page Structure

```tsx
// Opt-Out Policy Section
<section aria-label="Opt-out policy and legal disclaimers">
  <Card className="border-border bg-card mb-8 border-aqua/30">
    <CardHeader>
      <CardTitle>Opt-Out Policy</CardTitle>
    </CardHeader>
    <CardContent>
      <p>{OPT_OUT_POLICY}</p>

      {/* Prominent CTA */}
      <Link
        href="/rankings/opt-out"
        className="inline-block px-6 py-3 bg-aqua text-navy font-semibold rounded-lg hover:bg-aqua/90"
      >
        Request Removal
      </Link>
      <p className="text-sm text-gray-400 mt-2">
        Form coming soon. For now, email: <a href="mailto:hello@chainsights.one">{LEGAL_CONTACT_EMAIL}</a>
      </p>
    </CardContent>
  </Card>

  {/* Limitations & Disclaimers Section */}
  <Card className="border-border bg-card mb-8 border-yellow-500/30">
    <CardHeader>
      <CardTitle>
        <AlertCircle className="w-5 h-5 text-yellow-400" />
        Limitations & Disclaimers
      </CardTitle>
    </CardHeader>
    <CardContent>
      <ul className="space-y-3">
        <li>This is ONE perspective on governance health</li>
        <li>{NO_ENDORSEMENT_CLAUSE}</li>
        <li>High score ≠ perfect DAO, Low score ≠ bad DAO</li>
        <li>Quantitative metrics can't capture everything</li>
        <li>Use as starting point for improvement, not gospel</li>
      </ul>

      <div className="mt-6 p-4 bg-yellow-500/10 border border-yellow-500/30 rounded-lg">
        <p><strong>DISCLAIMER:</strong> {LEGAL_DISCLAIMER}</p>
      </div>

      <p className="mt-4 text-sm">
        <strong>GDPR Compliance:</strong> {GDPR_COMPLIANCE_STATEMENT}{' '}
        <Link href="/privacy" className="text-aqua hover:underline">
          Privacy Policy
        </Link>.
      </p>
    </CardContent>
  </Card>
</section>
```

---

## Dev Agent Record

### Context Reference
<!-- Path(s) to story context XML will be added here by context workflow -->

### Agent Model Used
claude-sonnet-4-5-20250929

### Debug Log References
- None

### Completion Notes List
- 2025-12-22: Story created with Ultimate Context Engine analysis
- CRITICAL: Methodology page already has disclaimer sections - must ENHANCE, not replace
- Privacy policy page and footer link already exist - verify only
- Created reusable legal constants pattern (following GVS_WEIGHTS precedent from Story 3.1)
- Opt-out form `/rankings/opt-out` doesn't exist yet - Story 4.1 will create it
- Story 3.2 adds placeholder link with "coming soon" message
- 2025-12-22: Implementation completed successfully
  - Created `src/lib/constants/legal-disclaimers.ts` with all 6 constants
  - Updated methodology page to import and use legal constants
  - Added prominent "Request Removal" CTA button to opt-out section
  - Enhanced limitations section with NO_ENDORSEMENT_CLAUSE
  - Added GDPR compliance statement with privacy link
  - Verified footer privacy link exists
  - TypeScript build successful
  - All acceptance criteria met
- 2025-12-22: Code review completed - 8 issues found, 5 fixed
  - Issue #1 (HIGH): Added unit test file with 15 tests (all passing)
  - Issue #2 (MEDIUM): Removed unused DATA_ACCURACY_DISCLAIMER export
  - Issue #3 (MEDIUM): Redesigned GDPR constant to be self-contained
  - Issue #4 (MEDIUM): Created placeholder /rankings/opt-out page with email fallback
  - Issue #5 (MEDIUM): Bumped methodology version to v1.1, added changelog entry
  - Issues #6-8 (LOW): Accepted as-is (hard-coded text, missing types, minor inconsistencies)
  - Build verified successful after all fixes
  - Tests passing: 15/15 in legal-disclaimers.test.ts

### File List

**Files Created:**
- `src/lib/constants/legal-disclaimers.ts` (52 lines, 5 exported constants with JSDoc - removed unused constant)
- `tests/unit/lib/constants/legal-disclaimers.test.ts` (120 lines, 15 unit tests - code review fix #1)
- `src/app/rankings/opt-out/page.tsx` (130 lines, placeholder page - code review fix #4)

**Files Modified:**
- `src/app/rankings/methodology/page.tsx` (added imports, updated opt-out section, enhanced disclaimers, bumped version to v1.1, added changelog)

**Files Verified:**
- `src/components/footer.tsx` (privacy link confirmed at line 60-61)
- `src/app/privacy/page.tsx` (comprehensive GDPR content exists)

---

## Change Log

- **2025-12-22**: Story 3.2 created with Ultimate Context Engine analysis
- **2025-12-22**: Implementation completed - all tasks, acceptance criteria met
- **2025-12-22**: Code review completed - 8 issues found, 5 high/medium issues fixed, 3 low issues accepted

---

## Definition of Done

- [x] Legal constants file created with all required text
- [x] JSDoc comments added to constants file
- [x] Methodology page imports and uses legal constants
- [x] Opt-out section enhanced with prominent CTA button
- [x] "No endorsement" clause added to disclaimers
- [x] GDPR compliance statement added with privacy link
- [x] Footer privacy link verified (exists)
- [x] TypeScript builds without errors
- [x] Page renders correctly with all legal text
- [x] All links functional (privacy, email)
- [x] Plain language maintained (no legalese)
- [x] Semantic HTML preserved

---

## References

**Epic 3 Story:** `docs/epics.md` lines 1666-1692
**Previous Story:** `docs/sprint-artifacts/3-1-create-methodology-page-with-gvs-explanation.md`
**Architecture:** `docs/architecture.md`
**Project Context:** `docs/project_context.md`
**Existing Files:**
- `src/app/rankings/methodology/page.tsx` (lines 367-428)
- `src/components/footer.tsx` (lines 60-62)
- `src/app/privacy/page.tsx`
