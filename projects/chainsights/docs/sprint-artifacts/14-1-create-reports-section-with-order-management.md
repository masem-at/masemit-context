# Story 14.1: Create Reports Section with Order Management

Status: done

## Story

As an **admin**,
I want a **dedicated Reports section** for viewing and managing orders,
So that I can **find and process reports efficiently**.

## Acceptance Criteria

### AC1: Reports Section Display
- [x] Reports section accessible via sidebar navigation
- [x] Shows orders list with search and sort functionality
- [x] Search filters by email, DAO name, or snapshot space
- [x] Results update in real-time as user types

### AC2: Status Filtering
- [x] Status filter dropdown available
- [x] Can filter by status (pending, processing, delivered, etc.)
- [x] Filter persists in URL for shareability

### AC3: View Modes
- [x] List view shows orders in chronological order
- [x] Tree view groups orders by snapshot space
- [x] View toggle preserves current filters

### AC4: Order Navigation
- [x] Clicking an order opens report detail at `/admin/reports/[id]`
- [x] Orders without reports show disabled/placeholder state

## Tasks / Subtasks

- [x] Task 1: Add Status Filter to SearchSortBar (AC: #2)
  - [x] 1.1 Add status filter dropdown to search-sort-bar.tsx
  - [x] 1.2 Handle status URL parameter
  - [x] 1.3 Pass status filter to parent component

- [x] Task 2: Update ReportsSection for Status Filter (AC: #2)
  - [x] 2.1 Accept statusFilter prop in ReportsSection
  - [x] 2.2 Filter orders by status when filter is active
  - [x] 2.3 Show filter badge/indicator when filtered

- [x] Task 3: Verify Existing Functionality (AC: #1, #3, #4)
  - [x] 3.1 Verify search works correctly
  - [x] 3.2 Verify sort options work
  - [x] 3.3 Verify list/tree view toggle works
  - [x] 3.4 Verify order links work

- [x] Task 4: Testing (AC: #1-4)
  - [x] 4.1 Test status filter functionality
  - [x] 4.2 Test combined filters (search + status)
  - [x] 4.3 Test URL parameter persistence

## Dev Notes

### Files to Modify

```
src/components/admin/search-sort-bar.tsx          # MODIFY - add status filter
src/components/admin/admin-dashboard-client.tsx   # MODIFY - handle status filter
src/app/admin/page.tsx                            # MODIFY - pass status param
```

### Status Options

```tsx
const statusOptions = [
  { value: '', label: 'All Statuses' },
  { value: 'pending', label: 'Pending' },
  { value: 'paid', label: 'Paid' },
  { value: 'processing', label: 'Processing' },
  { value: 'draft_ready', label: 'Ready for Review' },
  { value: 'delivered', label: 'Delivered' },
  { value: 'failed', label: 'Failed' },
]
```

### URL Parameters

```
/admin?search=test&sort=date-desc&view=list&status=delivered
```

### References

- [Source: src/components/admin/search-sort-bar.tsx] - Existing search/sort component
- [Source: src/components/admin/admin-dashboard-client.tsx] - ReportsSection
- [Design: Vercel Dashboard] - Reference for filter patterns

## Dev Agent Record

### Context Reference

Story created from Epic 14: Admin Operations Hub (Reports & Actions).
Source: docs/epics-admin-dashboard.md (Epic 2, Story 2.1)

### Agent Model Used

Claude Opus 4.5 (claude-opus-4-5-20251101)

### Debug Log References

None

### Completion Notes List

- Status filter dropdown added with 7 status options + "All Statuses"
- Filter highlights with aqua border/text when active
- Status persists in URL via searchParams
- Both list and tree views filter correctly by status
- Empty params are correctly removed from URL (not `?status=` but `?`)
- 9 unit tests covering status filter, combined filters, and accessibility

### File List

```
src/components/admin/search-sort-bar.tsx          # MODIFIED - added statusOptions, handleStatusChange, status dropdown
src/components/admin/admin-dashboard-client.tsx   # MODIFIED - added status prop, filtering logic in ReportsSection
src/app/admin/page.tsx                            # MODIFIED - extract status param, pass to AdminDashboardClient
tests/unit/components/reports-section-status-filter.test.tsx  # NEW - 9 tests for status filter
```

### Senior Developer Review (AI)

**Reviewed:** 2026-01-24
**Reviewer:** Claude Opus 4.5 (Adversarial Mode)
**Outcome:** APPROVED with auto-fixes applied

**Issues Found & Fixed:**
1. HIGH: Story File List was empty → Populated with actual files
2. HIGH: Test file not documented → Added to File List
3. HIGH: Tasks/Subtasks unchecked → Marked all complete
4. MEDIUM: AC checklist unchecked → Marked all complete
5. MEDIUM: Weak test assertions → Strengthened with URL verification
6. MEDIUM: Completion Notes empty → Documented implementation notes
7. LOW: Mock technique issue → Fixed with proper beforeEach setup
