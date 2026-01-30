# Story 1.4: Dashboard - Custom Events Display

**Epic:** 001 - Self-Hosted Analytics Tracking
**Status:** Ready for Development
**Priority:** P1
**Estimate:** M (Medium)
**Assignee:** Amelia (Dev Agent)

---

## User Story

**As a** project owner
**I want to** see custom events alongside page views in the dashboard
**So that** I can understand user interactions beyond just page visits

---

## Description

Update the analytics dashboard to display custom events. This includes:
1. Adding a "Custom Events" count card to the main dashboard
2. Showing event type (pageView/customEvent) in the events table
3. Displaying event_name and event_data for custom events
4. Adding a filter to show only page views, only custom events, or all

---

## Acceptance Criteria

- [ ] **AC1:** Main dashboard shows "Custom Events" count card
- [ ] **AC2:** "All Events" table shows event type with visual indicator (icon/badge)
- [ ] **AC3:** Custom events show `event_name` in the table
- [ ] **AC4:** Custom events show truncated `event_data` in the table
- [ ] **AC5:** Filter by event type (All / Page Views / Custom Events)
- [ ] **AC6:** Project dashboard events table also updated

---

## Technical Specification

### Main Dashboard Changes

1. **Add Custom Events Count Card:**
   - Query: `COUNT(*) FILTER (WHERE type = 'customEvent')`
   - Add to the stats grid

2. **Update Events Query:**
   - Add `type`, `event_name`, `event_data` to SELECT

3. **Update Events Table:**
   - Add "Type" column with badge
   - Add "Event" column (shows event_name for custom events)
   - Show truncated event_data on hover or in tooltip

4. **Add Event Type Filter:**
   - URL param: `?type=all|pageView|customEvent`
   - Filter events accordingly

### Visual Design

**Type Badges:**
```
[Page View] - Blue/cyan badge
[Event: cta_click] - Purple/violet badge with event name
```

**Event Data Display:**
- Truncate to ~50 chars
- Show full JSON on hover (title attribute)

---

## Tasks

- [ ] **Task 1:** Add custom events count to main dashboard stats query
- [ ] **Task 2:** Add type/event_name/event_data to events query
- [ ] **Task 3:** Update events table with type column and badges
- [ ] **Task 4:** Add event type filter component
- [ ] **Task 5:** Update project dashboard events table
- [ ] **Task 6:** Test with sample data

---

## Files to Modify

| File | Action |
|------|--------|
| `app/dashboard/page.tsx` | Modify |
| `app/dashboard/[project]/page.tsx` | Modify |

---

## Dependencies

- Story 1.1 (Events API Endpoint) ✅ Complete

---

## Definition of Done

- [ ] Custom Events count visible on main dashboard
- [ ] Event type badges displayed in table
- [ ] Event name and data visible for custom events
- [ ] Filter works correctly
- [ ] Code reviewed
- [ ] Committed and pushed
