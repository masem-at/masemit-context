# Story 11.1: Project-Based Dashboard Rebuild

Status: done

<!-- Note: Validation is optional. Run validate-create-story for quality check before dev-story. -->

## Story

As a **Meister**,
I want to see a project-based dashboard showing all active construction sites with latest activity and daily message count,
so that I get a quick morning overview of what's happening across all my projects.

## Acceptance Criteria

1. **Given** a logged-in Meister on `/app` (dashboard) **When** the page loads **Then** the dashboard shows project cards instead of the flat voice message list from Sprint 0 **And** each card shows: project name, customer name, status, last activity timestamp, and count of today's voice messages **And** only active projects for the Meister's company are shown (tenant isolation) **And** cards are sorted by last activity (most recent first) **And** the dashboard loads within 2 seconds (S1-NFR2).

2. **Given** a Meister viewing a project card on the dashboard **When** clicking on the card **Then** the Meister navigates to `/app/projects/[id]` (project detail timeline) **And** the timeline loads within 3 seconds (S1-NFR4).

3. **Given** a Meister with no active projects **When** the dashboard loads **Then** an empty state is shown with a call-to-action to create a first project.

4. **Given** the dashboard **When** rendered **Then** all touch targets are >=48px (S1-NFR12) **And** contrast ratios meet WCAG AA (S1-NFR13) **And** project cards have comfortable spacing for mobile use.

**FRs:** S1-FR21, S1-FR22 | **NFRs:** S1-NFR2, S1-NFR4, S1-NFR12, S1-NFR13

## Tasks / Subtasks

- [x] Task 1: Create `getDashboardProjects()` server action (AC: #1)
  - [x] 1.1 Add `DashboardProject` interface to `lib/actions/projects.ts` with: id, name, customerName, status, lastActivity (Date | null), todayMessageCount (number)
  - [x] 1.2 Add `getDashboardProjects()` function using raw SQL query with LEFT JOIN on voice_messages
  - [x] 1.3 Filter: `projects.status = 'active'` AND `projects.company_id = companyId` (tenant isolation via `getAuthContext()`)
  - [x] 1.4 Aggregate today's message count: `COUNT(vm.id) FILTER (WHERE vm.created_at >= CURRENT_DATE AND vm.status = 'done')` (PostgreSQL FILTER syntax)
  - [x] 1.5 Aggregate last activity: `MAX(vm.created_at)` for done messages assigned to each project
  - [x] 1.6 Sort by `last_activity DESC NULLS LAST, p.created_at DESC` (projects with no activity sorted by creation date)

- [x] Task 2: Rebuild `app/app/page.tsx` as project-based dashboard (AC: #1, #2, #4)
  - [x] 2.1 Replace the entire current dashboard implementation (flat voice message list) with project card layout
  - [x] 2.2 Fetch data using `getDashboardProjects()` — single query, no N+1
  - [x] 2.3 Render page header: "Übersicht" heading (consistent with current nav label)
  - [x] 2.4 Render project cards as `<Link>` elements wrapping each card → navigates to `/app/projects/${project.id}`
  - [x] 2.5 Each card displays: project name (bold), customer name (muted, below name), status badge (from `STATUS_CONFIG`), last activity (relative time via `formatRelativeTime()`), today's message count with Mic icon
  - [x] 2.6 Cards stack vertically on mobile with full-width, comfortable padding (px-4, py-3)
  - [x] 2.7 Ensure all card touch targets >=48px (min-h-12 or equivalent) and contrast ratios meet WCAG AA
  - [x] 2.8 Use Server Component (async function) — no "use client", no client-side data fetching

- [x] Task 3: Create empty state for dashboard (AC: #3)
  - [x] 3.1 Show empty state when `getDashboardProjects()` returns empty array
  - [x] 3.2 Display: FolderOpen icon (lucide-react), heading "Noch keine aktiven Projekte", descriptive text, and CTA button "Erstes Projekt anlegen" linking to `/app/projects/new`
  - [x] 3.3 Follow existing empty state pattern from inbox page and projects page (consistent style)

- [x] Task 4: Clean up removed dashboard dependencies (AC: #1)
  - [x] 4.1 Remove voice message flat list imports from `app/app/page.tsx` (VoiceMessageCard, AssignmentDropdown, message grouping logic)
  - [x] 4.2 Remove unused imports: voiceMessages schema, message grouping helper functions
  - [x] 4.3 Verify no dead code left in page file after rebuild
  - [x] 4.4 Do NOT remove or modify any imported components/actions themselves — they are still used by other pages (inbox, project detail)

## Dev Notes

### Architecture & Implementation Patterns

- **Server Component dashboard** (Architecture Decision 6): Dashboard is a Server Component with direct Drizzle queries. No client-side data fetching, no "use client". This is the same pattern used by all existing pages (inbox, projects list, project detail).
- **Server Action for data query**: `getDashboardProjects()` follows the established pattern from `getProjectsWithStats()` — raw SQL with LEFT JOIN for aggregation. Cannot use Drizzle query builder for complex aggregation with FILTER clauses.
- **Tenant isolation**: Every query MUST filter by `companyId` from `getAuthContext()`. This is the most important Sprint 1 rule.
- **`revalidatePath`**: The dashboard page is already revalidated by existing Server Actions (`createProject`, `assignMessageToProject`, `createProjectAndAssignMessage`) via `revalidatePath("/app")`. No additional revalidation needed.

### Critical Constraints

- **This is a BREAKING UI change, not a data change**: The dashboard page (`app/app/page.tsx`) is completely rebuilt. The flat voice message list is removed and replaced with project cards. The underlying data (projects, voice_messages) is unchanged.
- **No new Client Components needed**: The dashboard is purely read-only with navigation links. No forms, dropdowns, or interactive state. Server Component is the correct choice.
- **Active projects ONLY**: Filter `projects.status = 'active'` — paused and completed projects are NOT shown on the dashboard. Users access them via `/app/projects` (all projects list).
- **Today's message count**: Use PostgreSQL `CURRENT_DATE` for the "today" boundary. This uses the database session timezone (UTC by default in NeonDB). For Sprint 1 single-user mode (Austria, UTC+1/+2), the 1-2 hour offset at midnight is acceptable.
- **Last activity = last DONE message**: Only count messages with `status = 'done'` for activity tracking. Processing/error messages are not "activity."

### Existing Code to Reuse

| Component/Function | Source | Reuse Approach |
|---|---|---|
| `getAuthContext()` | `lib/auth/session.ts` | Direct call for companyId (established pattern) |
| `formatRelativeTime()` | `lib/format.ts` | Format lastActivity timestamp (already used on all pages) |
| `STATUS_CONFIG` | `lib/validations/project.ts` | Status badge labels + colors (already used on projects list) |
| `getProjectsWithStats()` | `lib/actions/projects.ts` | **Reference pattern only** — the SQL query structure is the model for the new `getDashboardProjects()`, but with different filters and aggregations |
| Empty state pattern | `app/app/inbox/page.tsx`, `app/app/projects/page.tsx` | Follow same visual pattern (icon + heading + text + CTA) |
| Card link pattern | `app/app/projects/page.tsx` | Same `<Link href="/app/projects/${id}">` wrapping a card div |

### Code Being Replaced (Current Dashboard)

The **entire** current `app/app/page.tsx` is replaced. The current implementation:
- Fetches ALL voice messages for the user, grouped by date
- Uses `VoiceMessageCard` + `AssignmentDropdown` components
- Shows message grouping by day ("Heute", "Gestern", date)
- Has its own empty state ("Noch keine Nachrichten")

**None of this logic is needed** in the new dashboard. The developer MUST start fresh with the project-based query and card layout. Do NOT try to "adapt" the existing page — rewrite it from scratch while keeping the same file path.

### Query Design: `getDashboardProjects()`

The query follows the same raw SQL pattern as `getProjectsWithStats()` but with key differences:

```
Differences from getProjectsWithStats():
- FILTER: projects.status = 'active' only (getProjectsWithStats returns all statuses)
- COUNT: today's messages only (not total), using FILTER (WHERE created_at >= CURRENT_DATE)
- SAME: LEFT JOIN voice_messages, GROUP BY project, companyId filter
- SAME: ORDER BY last_activity DESC NULLS LAST
```

PostgreSQL `COUNT(...) FILTER (WHERE ...)` is the idiomatic way to do conditional aggregation:
```sql
COUNT(vm.id) FILTER (WHERE vm.created_at >= CURRENT_DATE AND vm.status = 'done') AS today_message_count
```

This avoids a subquery and is efficient. NeonDB (PostgreSQL 16) supports this syntax.

### Card Design Guidance

Each project card should show:
- **Row 1**: Project name (font-semibold, text-base) + Status badge (right-aligned, from STATUS_CONFIG)
- **Row 2**: Customer name if present (text-muted-foreground, text-sm) — "Fam. Gruber" or omit row if null
- **Row 3**: Stats row — Mic icon + today's count ("3 heute") | Clock icon + last activity (relative time) — OR "Keine Aktivität" if null

Visual reference from PRD Journey 3:
```
| Neubau Urfahr, Fam. Gruber | Gestern, 16:30 | 3 |
| Sanierung Pfarrgasse, Hr. Wimmer | Gestern, 15:30 | 2 |
| Büroumbau Industriezeile | Montag, 14:00 | 0 |
```

Follow the card styling from existing projects page (`app/app/projects/page.tsx`) for consistency — border, rounded, padding, hover state.

### Project Structure Notes

- **Modified files:**
  - `app/app/page.tsx` — Full rewrite (flat messages → project cards)
  - `lib/actions/projects.ts` — Add `getDashboardProjects()` function + `DashboardProject` interface
- **No new files needed** — all changes fit in existing files
- **No deleted files** — VoiceMessageCard, AssignmentDropdown, inbox actions remain used by other pages
- **No package changes** — all dependencies already installed (lucide-react, Drizzle, etc.)

### Testing Strategy

- **Manual test 1**: Log in → navigate to `/app` → verify project cards are shown (not flat message list) → verify only active projects visible → verify today's message count is correct → verify last activity is formatted correctly
- **Manual test 2**: Click on a project card → verify navigation to `/app/projects/[id]` → verify timeline loads correctly
- **Manual test 3**: Delete/pause all projects (or use account with none) → verify empty state shown with CTA → click CTA → verify navigation to `/app/projects/new`
- **Manual test 4**: Send a new voice message → wait for processing → navigate to dashboard → verify today's count incremented for the assigned project
- **Manual test 5**: Test on mobile viewport (375px) → verify cards are full-width, touch targets >=48px, text readable
- **Manual test 6**: Verify tenant isolation — only projects belonging to the user's company are shown (check companyId filter in query)

### References

- [Source: _bmad-output/planning-artifacts/epics.md#Story 11.1] — Acceptance criteria and FR mapping
- [Source: _bmad-output/planning-artifacts/architecture-sprint1.md#Decision 6] — Server Components for Dashboard, page architecture
- [Source: _bmad-output/planning-artifacts/architecture-sprint1.md#companyId filtering pattern] — Tenant isolation requirement
- [Source: _bmad-output/planning-artifacts/architecture-sprint1.md#Implementation Sequence] — Dashboard rebuild is step 8 (final)
- [Source: _bmad-output/planning-artifacts/prd-sprint1.md#Journey 3] — Stefan's morning overview use case with table mockup
- [Source: _bmad-output/planning-artifacts/prd-sprint1.md#S1-FR21] — Project-based dashboard with activity cards
- [Source: _bmad-output/planning-artifacts/prd-sprint1.md#S1-FR22] — Dashboard → project timeline navigation
- [Source: lib/actions/projects.ts#getProjectsWithStats] — Existing raw SQL query pattern for project aggregation
- [Source: lib/format.ts#formatRelativeTime] — German relative time formatting (reuse)
- [Source: lib/validations/project.ts#STATUS_CONFIG] — Status badge labels + colors (reuse)
- [Source: app/app/projects/page.tsx] — Existing project card rendering pattern (reference for consistency)
- [Source: app/app/page.tsx] — Current dashboard implementation (to be fully replaced)

### Previous Story Intelligence (10.1, 10.2, 10.3)

- **Revalidation paths established**: `revalidatePath("/app")` already called by `createProject`, `assignMessageToProject`, `createProjectAndAssignMessage`, `updateProject`, `updateProjectStatus` — dashboard will auto-refresh when data changes
- **`getAuthContext()` pattern**: Consistently used across all Stories 8.1–10.3 — call first, extract companyId, filter all queries
- **Raw SQL pattern**: `getProjectsWithStats()` demonstrates the exact JOIN + aggregation pattern needed — follow this for `getDashboardProjects()`
- **Empty state pattern**: All three app pages (dashboard, inbox, projects) have empty states — follow same visual structure (centered icon + heading + text + CTA button)
- **Status badge pattern**: `STATUS_CONFIG` from `lib/validations/project.ts` used in projects list page — reuse exact same styling

### Git Intelligence

Recent commits (Stories 8.1–10.3) show:
- Single-commit per story implementation pattern
- All libraries needed are already installed (no new dependencies)
- Server Component + Server Actions pattern used consistently
- `formatRelativeTime()` and `STATUS_CONFIG` already established and tested
- Commit message pattern: "Implement/Add/Rebuild [feature] (Story X.Y)"
- Suggested commit: "Rebuild dashboard with project-based overview cards (Story 11.1)"

## Dev Agent Record

### Agent Model Used

Claude Opus 4.6 (claude-opus-4-6)

### Debug Log References

No issues encountered. Build passed on first attempt.

### Completion Notes List

- Task 1: Added `DashboardProject` interface and `getDashboardProjects()` to `lib/actions/projects.ts`. Uses raw SQL with LEFT JOIN on voice_messages, PostgreSQL FILTER syntax for today's message count, tenant isolation via companyId from getAuthContext().
- Task 2: Complete rewrite of `app/app/page.tsx` from flat voice message list to project-based dashboard. Server Component with single query, project cards as Link elements, status badges from STATUS_CONFIG, relative time formatting, Mic + Clock icons for stats row.
- Task 3: Empty state with FolderOpen icon, "Noch keine aktiven Projekte" heading, descriptive text, and "Erstes Projekt anlegen" CTA button linking to /app/projects/new. Follows established pattern from inbox and projects pages.
- Task 4: All old imports removed (VoiceMessageCard, AssignmentDropdown, groupMessagesByDay, voiceMessages schema, users schema, message grouping interfaces). Components themselves preserved — still used by inbox and project detail pages. No dead code remains.

### Change Log

- 2026-02-15: Full dashboard rebuild from flat voice message list to project-based overview cards (Story 11.1)
- 2026-02-15: Code review — 2 MEDIUM fixes applied: (1) Removed redundant vm.status='done' from SQL FILTER clause (already in JOIN), (2) today's message count now hidden when 0 (consistent with projects page pattern). 3 LOW issues accepted as-is.

### File List

- `lib/actions/projects.ts` — Added DashboardProject interface + getDashboardProjects() function
- `app/app/page.tsx` — Full rewrite: flat message list → project-based dashboard with cards, empty state
- `_bmad-output/implementation-artifacts/sprint-status.yaml` — Status updated: ready-for-dev → in-progress → review
- `_bmad-output/implementation-artifacts/11-1-project-based-dashboard-rebuild.md` — Story file updated
