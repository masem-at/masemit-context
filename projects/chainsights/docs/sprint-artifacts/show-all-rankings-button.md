# Story: Add "Show All" Button to Rankings Page

**Epic:** Rankings UX Improvements
**Priority:** Medium
**Estimate:** Small (2-3 hours)

---

## User Story

**As a** visitor to the rankings page
**I want** to see a manageable initial list with option to expand
**So that** I can quickly scan top DAOs while having access to the full list

---

## Background

Currently all featured DAOs are displayed. As the list grows beyond 10, we want to:
1. Show first 10 by default for clean UX
2. Provide "Show All" button to reveal remaining DAOs
3. Ensure filtering/sorting always operates on complete dataset

---

## Acceptance Criteria

### AC1: Default Display Limit
- [ ] Rankings page shows maximum 10 DAOs by default
- [ ] DAOs are sorted/ranked before the display limit is applied
- [ ] Count indicator shows "Showing 10 of N DAOs" when truncated

### AC2: Show All Button
- [ ] "Show All (N total)" button appears below table when more than 10 DAOs exist
- [ ] Clicking button reveals all DAOs
- [ ] Button text changes to "Show Less" after expansion
- [ ] Clicking "Show Less" returns to 10-item view

### AC3: Filter/Search Bypass
- [ ] When ANY filter is active (category !== 'all'), display ALL matching DAOs
- [ ] When sorting is changed from default, display ALL DAOs
- [ ] "Show All" button hidden when filter/sort is active (not needed)

### AC4: Mobile Support
- [ ] Show All/Show Less button works on mobile card layout
- [ ] Button has adequate touch target (min 44px height)
- [ ] Smooth transition when expanding/collapsing

### AC5: State Persistence
- [ ] "Show All" state resets when filter/sort changes
- [ ] Page refresh returns to default (10 items) view

---

## Technical Approach

### Files to Modify

1. **`src/app/rankings/page.tsx`**
   - Pass `hasActiveFilter` boolean to client
   - No server-side limit needed (already fetches all)

2. **`src/components/RankingsTable.tsx`** (convert to client component or create wrapper)
   - Add `showAll` state
   - Implement display slicing logic:
     ```typescript
     const hasActiveFilter = category !== 'all' || sortBy !== 'rank'
     const displayRankings = hasActiveFilter || showAll
       ? rankings
       : rankings.slice(0, 10)
     const showButton = !hasActiveFilter && rankings.length > 10
     ```
   - Render "Show All / Show Less" button

3. **`src/components/ShowAllButton.tsx`** (new - optional)
   - Reusable button component
   - Props: `total`, `isExpanded`, `onToggle`

### Display Logic

```
┌─────────────────────────────────────────┐
│ IF hasActiveFilter (category/sort)      │
│   → Show ALL matching DAOs              │
│   → Hide Show All button                │
│                                         │
│ ELSE (default view)                     │
│   → Show first 10 DAOs                  │
│   → Show "Show All (N)" button if N>10  │
│   → On click: show all, button→"Less"   │
└─────────────────────────────────────────┘
```

---

## Out of Scope

- Pagination with page numbers
- Infinite scroll
- URL persistence of "showAll" state
- Server-side pagination (LIMIT/OFFSET)

---

## Testing Requirements

### Unit Tests
- [ ] Display slicing logic with various ranking counts
- [ ] Button visibility conditions
- [ ] Filter active detection

### Integration Tests
- [ ] Show All → Show Less toggle cycle
- [ ] Filter applied → all results shown
- [ ] Mobile card layout with button

---

## Design Notes

**Button Styling:**
```
Default state:  [▼ Show All (15 total)]
Expanded state: [▲ Show Less]
```

- Centered below table
- Subtle styling (outline button, not primary)
- Chevron indicates expand/collapse direction

---

## Dependencies

- None (self-contained UI change)

---

## Definition of Done

- [ ] All ACs pass manual testing
- [ ] Unit tests written and passing
- [ ] Works on desktop table and mobile cards
- [ ] No regression in existing filter/sort functionality
- [ ] Build passes, no TypeScript errors
