# Story 13.3: Migrate Overview Section Content

Status: done

## Story

As an **admin**,
I want the **Overview section** to show key stats and recent activity,
So that I can **get a quick summary of the system state**.

## Acceptance Criteria

### AC1: Stats Cards Display
- [x] Stats cards visible: Total Orders, Ready for Review, Delivered, Pending, Share Claims
- [x] Each stat shows accurate count from database
- [x] Cards use consistent styling with shadcn/ui

### AC2: Stats Cards Navigation
- [x] Each stats card is clickable
- [x] Clicking navigates to relevant section (Reports for orders)
- [x] Hover state indicates clickability

### AC3: Recent Activity Display
- [x] Recent GVS Jobs section shows last 5 jobs
- [x] Pending Opt-Out Requests section shows pending requests
- [x] Each section has "View All" link to detailed page

### AC4: Share Claims Highlighting
- [x] Share Claims card highlighted when count > 0
- [x] Visual distinction (border color) for pending claims

## Tasks / Subtasks

- [x] Task 1: Make Stats Cards Clickable (AC: #2)
  - [x] 1.1 Wrap stats cards with click handlers
  - [x] 1.2 Navigate to Reports section on click
  - [x] 1.3 Add hover states for clickability indication
  - [x] 1.4 Add cursor-pointer and transition styles

- [x] Task 2: Verify Existing Content (AC: #1, #3, #4)
  - [x] 2.1 Confirm stats cards show correct counts
  - [x] 2.2 Confirm GVS Jobs section displays properly
  - [x] 2.3 Confirm Opt-Out Requests section displays properly
  - [x] 2.4 Confirm Share Claims highlighting works

- [x] Task 3: Testing (AC: #1-4)
  - [x] 3.1 Test stats card click navigation
  - [x] 3.2 Test existing content displays
  - [x] 3.3 Test Share Claims highlighting

## Dev Notes

### Files to Modify

```
src/components/admin/admin-dashboard-client.tsx   # MODIFY - make cards clickable
```

### Stats Card Click Behavior

| Card | Navigation Target |
|------|------------------|
| Total Orders | Reports section |
| Ready for Review | Reports section |
| Delivered | Reports section |
| Pending | Reports section |
| Share Claims | Reports section |

### Implementation Pattern

```tsx
// Clickable stats card with StatsCard component
<StatsCard
  value={allOrders.length}
  label="Total Orders"
  colorClass="text-white"
  onClick={() => onSectionChange('reports')}
/>
```

### References

- [Source: src/components/admin/admin-dashboard-client.tsx] - Current Overview section
- [Design: Vercel Dashboard] - Reference for clickable stats cards

## Dev Agent Record

### Context Reference

Story created from Epic 13: Admin Dashboard Layout & Navigation.
Source: docs/epics-admin-dashboard.md (Epic 1, Story 1.3)

### Agent Model Used

Claude Opus 4.5 (claude-opus-4-5-20251101)

### Debug Log References

None

### Completion Notes List

- All 4 Acceptance Criteria implemented and verified
- Created StatsCard component for reusable clickable stats cards
- All 5 stats cards navigate to Reports section on click
- Keyboard accessible (role="button", tabIndex, Enter/Space handlers)
- Share Claims card highlighted with pink border when count > 0
- 14 unit tests all passing
- No code review issues found

### File List

- `src/components/admin/admin-dashboard-client.tsx` (MODIFIED) - Added StatsCard component, made cards clickable
- `tests/unit/components/overview-stats-cards.test.tsx` (NEW) - 14 unit tests
