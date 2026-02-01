# Story 3.9: Privacy Disclaimer & Synthetic Data Badge

**Status:** done
**Epic:** 3 - HR Data Domain
**Points:** 2
**Created:** 2026-01-22

---

## Story

As a **user viewing or exporting HR data**,
I want **clear indicators that data is synthetic and options for anonymization**,
so that **I can confidently use the data knowing it's GDPR-compliant and not real individuals**.

---

## Acceptance Criteria

1. **AC1: "Synthetic Data" Badge on HR View** - ALREADY IMPLEMENTED
   - [x] Amber banner with AlertTriangle icon at top of HR view
   - [x] Text: "Synthetic Data - All employee names and personal data are AI-generated"
   - Location: `components/results/HRView.tsx` lines 96-105

2. **AC2: Footer Disclaimer** - ALREADY IMPLEMENTED
   - [x] Gray footer text below all charts
   - [x] Text: "All workforce data shown is synthetically generated..."
   - Location: `components/results/HRView.tsx` lines 310-314

3. **AC3: Option to Use Anonymized Names**
   - [x] Add toggle/switch to use anonymized names (Employee_001, Employee_002...)
   - [x] Store preference in localStorage
   - [x] Apply to employee list in export
   - [x] Default: Show synthetic names (current behavior)

4. **AC4: No Real Birthdates Exposed** - ALREADY IMPLEMENTED
   - [x] Only birth year stored, not full DOB
   - [x] Design decision from Story 3.1

5. **AC5: Export Includes Synthetic Data Notice** - ALREADY IMPLEMENTED
   - [x] JSON export has `synthetic: true` field
   - [x] CSV export has comment header: "# SYNTHETIC DATA - This file contains..."
   - Location: `app/api/export/[scenarioId]/hr.json/route.ts` line 148
   - Location: `app/api/export/[scenarioId]/hr.csv/route.ts` line 54

---

## Tasks / Subtasks

### Task 1: Add Anonymization Toggle (AC: #3)
- [x] 1.1 Add toggle state and localStorage persistence
- [x] 1.2 Create anonymization function (`anonymizeName(index) => "Employee_${String(index).padStart(3, '0')}"`)
- [x] 1.3 Add toggle UI component in HR view (near synthetic data badge)
- [x] 1.4 Pass anonymization flag to export API
- [x] 1.5 Update CSV export to use anonymized names when flag is set

---

## Dev Notes

### Current Implementation Status

**Most of Story 3.9 is already implemented:**
- Synthetic Data badge: `components/results/HRView.tsx` lines 96-105
- Footer disclaimer: `components/results/HRView.tsx` lines 310-314
- JSON export `synthetic: true`: `app/api/export/[scenarioId]/hr.json/route.ts` line 148
- CSV export comment: `app/api/export/[scenarioId]/hr.csv/route.ts` line 54
- Birth year only (not DOB): Design from Story 3.1

**Only AC3 (anonymization toggle) needs implementation.**

### Anonymization Toggle Design

**Option A: URL Query Parameter**
- `?anonymize=true` added to export URLs
- Simple, stateless
- No UI needed in HR view

**Option B: localStorage Toggle + UI**
- Toggle switch in HR view header
- Persists preference
- More user-friendly

**Recommendation:** Option A is simpler and sufficient. Add `?anonymize=true` support to export APIs. No UI toggle needed - document in API docs instead.

**Alternative Recommendation:** Skip this AC entirely as the synthetic names ARE already anonymized (faker-generated, not real). The "Employee_001" format is arguably LESS useful for testing than synthetic names. Mark story as complete with existing implementation.

### Scope Decision Needed

**Question for User:** Is the anonymized names feature (Employee_001) actually needed, or is the current faker-generated synthetic name approach sufficient?

If synthetic names are sufficient (they're already not real people), this story is **100% complete** with existing implementation.

### Files Already Modified (Previous Stories)

- `components/results/HRView.tsx` - Badge and disclaimer
- `app/api/export/[scenarioId]/hr.json/route.ts` - synthetic:true
- `app/api/export/[scenarioId]/hr.csv/route.ts` - Comment header

### Files to Potentially Modify (if AC3 needed)

- `app/api/export/[scenarioId]/hr.json/route.ts` - Add anonymize param
- `app/api/export/[scenarioId]/hr.csv/route.ts` - Add anonymize param
- `lib/queries/hr-queries.ts` - Add anonymization option to getEmployeeExport

---

## Dev Agent Record

### Agent Model Used

Claude Opus 4.5 (claude-opus-4-5-20251101)

### Completion Notes List

- [x] Story created via create-story workflow
- [x] Analyzed existing implementation - 4 of 5 ACs already done
- [x] AC3 implemented: Anonymization toggle with localStorage persistence
- [x] Added custom hook `useAnonymizePreference()` for state management
- [x] Added toggle button UI near Synthetic Data badge in HRView
- [x] CSV export API supports `?anonymize=true` query parameter
- [x] Anonymized names use format `Employee_001`, `Employee_002`, etc.
- [x] Export filename includes `-anon` suffix when anonymized
- [x] TypeScript and lint checks pass

### File List

**Files modified:**
- `components/results/HRView.tsx` - Badge, disclaimer, and anonymization toggle (AC1, AC2, AC3)
- `app/api/export/[scenarioId]/hr.csv/route.ts` - Anonymize query param support (AC3)
- `app/api/export/[scenarioId]/hr.json/route.ts` - Anonymize query param support (AC3 - code review fix)
- `app/results/[scenarioId]/page.tsx` - HR export links read localStorage preference (AC3 - code review fix)
- `docs/sprint-artifacts/sprint-status.yaml` - Story status tracking
- `docs/sprint-artifacts/3-9-privacy-disclaimer-synthetic-data-badge.md` - This story file

### Change Log

- 2026-01-22: Story 3.9 created via create-story workflow
- 2026-01-22: Analysis shows 4/5 ACs already implemented in previous stories
- 2026-01-22: AC3 implemented - Anonymization toggle with localStorage, CSV export support
- 2026-01-22: Code review found HIGH issue - export links not passing anonymize flag
- 2026-01-22: Fixed: results page reads localStorage, passes ?anonymize=true to export URLs
- 2026-01-22: Fixed: hr.json endpoint now also supports ?anonymize=true parameter
