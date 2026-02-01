# Story 4.7: Scenario Cards with View/Download

**Epic:** 4 - Auth & User Dashboard (Phase 2)
**Status:** done
**Created:** 2026-01-21
**Depends On:** Story 4.6 (Dashboard Layout)

## User Story

As a **logged-in user**,
I want **to see my generated scenarios as cards with View and Download actions**,
So that **I can easily access my data**.

## Acceptance Criteria

### AC1: Scenario Cards Display
**Given** a user has generated scenarios
**When** the dashboard loads
**Then** scenario cards are displayed in a responsive grid (3 cols desktop, 2 tablet, 1 mobile)
**And** each card shows: sector icon, company name, size badge, generation date
**And** cards are sorted by creation date (newest first)

### AC2: View Action
**Given** a user clicks [View] on a scenario card
**When** the button is clicked
**Then** the user is navigated to the scenario detail page (/results/[scenarioId])

### AC3: Download Action
**Given** a user clicks [Download] on a scenario card
**When** the button is clicked
**Then** the CSV/JSON export is downloaded

### AC4: Empty State
**Given** a user has no scenarios
**When** the dashboard loads
**Then** an empty state is displayed with "Generate your first scenario" CTA

## Technical Details

### Source References
- Architecture: `docs/architecture/auth-dashboard-architecture.md`
- UX Spec: `docs/ux/dashboard-ux-spec.md`
- Epic Definition: `docs/epics-big-bang.md` (Story 4.7)

### Sector Icons
- consumer-staples: Sprout (green)
- industrials: Factory (blue)
- financials: Building2 (purple)

### Files to Create/Modify
1. `components/dashboard/ScenarioCard.tsx` - Individual scenario card component (NEW)
2. `components/dashboard/ScenarioGrid.tsx` - Grid container with empty state (NEW)
3. `app/dashboard/page.tsx` - Update to fetch and display scenarios (MODIFY)

## Tasks

- [x] Create ScenarioCard component with sector icon, company name, size, date
- [x] Create ScenarioGrid component with responsive grid layout
- [x] Add View button linking to /results/[scenarioId]
- [x] Add Download dropdown for CSV/JSON exports
- [x] Implement empty state with CTA
- [x] Update dashboard page to fetch user scenarios
- [x] Sort scenarios by creation date (newest first)

## Definition of Done

- [x] All acceptance criteria pass
- [x] Scenario cards display correctly in responsive grid
- [x] View button navigates to scenario detail
- [x] Download button triggers export
- [x] Empty state shows for users with no scenarios
