# Story 13.2: Implement Status Overview Panel

Status: done

## Story

As an **admin**,
I want a **status overview panel at the top of the dashboard**,
So that I can **instantly see system health without scrolling**.

## Acceptance Criteria

### AC1: Status Panel Display
- [x] Status overview panel visible at top of dashboard
- [x] Panel shows: Last GVS run time and status (success/failed)
- [x] Panel shows: Number of failed jobs in last 7 days
- [x] Panel shows: Pending opt-out requests count
- [x] Panel shows: Total active DAOs count
- [x] Panel visible in <2 seconds without scrolling

### AC2: Color-Coded Health Indicators
- [x] Green indicator for healthy status
- [x] Yellow indicator for attention needed
- [x] Red indicator for critical issues
- [x] Clear visual distinction between states

### AC3: Clickable Indicators
- [x] Non-healthy indicators are clickable
- [x] Clicking navigates to relevant section
- [x] Navigation uses existing sidebar section switching

## Tasks / Subtasks

- [x] Task 1: Create StatusOverviewPanel Component (AC: #1, #2)
  - [x] 1.1 Create `src/components/admin/status-overview-panel.tsx`
  - [x] 1.2 Add 4 status indicator cards in horizontal layout
  - [x] 1.3 Implement color-coded status badges (green/yellow/red)
  - [x] 1.4 Style with shadcn/ui cards and Tailwind

- [x] Task 2: Integrate Data Sources (AC: #1)
  - [x] 2.1 Pass job logs data to determine last GVS run status
  - [x] 2.2 Calculate failed jobs count in last 7 days
  - [x] 2.3 Pass pending opt-out count
  - [x] 2.4 Calculate total active DAOs count

- [x] Task 3: Add Click Navigation (AC: #3)
  - [x] 3.1 Make non-healthy indicators clickable
  - [x] 3.2 Connect to section switching in admin-dashboard-client
  - [x] 3.3 Navigate to appropriate section on click

- [x] Task 4: Integration (AC: #1)
  - [x] 4.1 Add panel to Overview section in admin-dashboard-client
  - [x] 4.2 Ensure panel renders at top without scrolling
  - [x] 4.3 Test responsive layout on mobile

- [x] Task 5: Testing (AC: #1-3)
  - [x] 5.1 Unit test status panel component
  - [x] 5.2 Test color-coded indicators
  - [x] 5.3 Test click navigation

## Dev Notes

### Files to Create/Modify

```
src/components/admin/status-overview-panel.tsx    # NEW - status panel component
src/components/admin/admin-dashboard-client.tsx   # MODIFY - add panel to Overview
src/app/admin/page.tsx                            # MODIFY - pass active DAOs count
```

### Status Indicator Logic

```tsx
// Last GVS Run
- Green: last job succeeded within 24h
- Yellow: last job succeeded but >24h ago
- Red: last job failed

// Failed Jobs (7 days)
- Green: 0 failures
- Yellow: 1-2 failures
- Red: 3+ failures

// Pending Opt-Outs
- Green: 0 pending
- Yellow: 1-5 pending
- Red: 6+ pending

// Active DAOs
- Always neutral (informational only)
```

### Layout Pattern

```
┌──────────────────────────────────────────────────────────────────────────────┐
│ Status Overview                                                               │
├─────────────────┬─────────────────┬─────────────────┬─────────────────────────┤
│ Last GVS Run    │ Failed Jobs     │ Pending Opt-Outs│ Active DAOs             │
│ ● 2h ago        │ ● 0 in 7 days   │ ● 3 pending     │ 42 DAOs                 │
│   Success       │                 │                 │                         │
└─────────────────┴─────────────────┴─────────────────┴─────────────────────────┘
```

### References

- [Source: src/lib/jobs/job-logs.ts] - getRecentJobLogs() for job status
- [Source: src/lib/dao/opt-out.ts] - getPendingOptOutRequests() for opt-out count
- [Design: Vercel Dashboard] - Reference for status panel pattern
- [Component: shadcn/ui Card] - Card component for indicators

## Dev Agent Record

### Context Reference

Story created from Epic 13: Admin Dashboard Layout & Navigation.
Source: docs/epics-admin-dashboard.md (Epic 1, Story 1.2)

### Agent Model Used

Claude Opus 4.5 (claude-opus-4-5-20251101)

### Debug Log References

None

### Completion Notes List

- All 3 Acceptance Criteria implemented and verified
- StatusOverviewPanel component with 4 health indicators
- Color-coded status levels: healthy (green), attention (yellow), critical (red)
- Clickable indicators navigate to Actions section for non-healthy states
- Keyboard accessible (role="button", tabIndex, Enter/Space handlers)
- 18 unit tests all passing
- Pulse animation only on non-healthy indicators (code review fix)

### File List

- `src/components/admin/status-overview-panel.tsx` (NEW) - Status panel component
- `src/components/admin/admin-dashboard-client.tsx` (MODIFIED) - Added panel to Overview section
- `tests/unit/components/status-overview-panel.test.tsx` (NEW) - 18 unit tests
