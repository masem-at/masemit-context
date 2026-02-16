# Story 12.4: Route Protection & Monteur Data Filtering

Status: done

<!-- Note: Validation is optional. Run validate-create-story for quality check before dev-story. -->

## Story

As a **Monteur**,
I want to see only the projects I'm assigned to,
so that I'm not overwhelmed by projects that don't concern me.

As a **system**,
I want RBAC enforced on all routes,
so that unauthorized access is prevented at every level.

## Acceptance Criteria

1. **Given** a user with role "monteur" on `/app` **When** the dashboard loads **Then** only projects where the user is a member (via `project_members`) are shown **And** the Inbox nav item is hidden **And** the Team nav item is hidden.

2. **Given** a user with role "monteur" **When** trying to access `/app/inbox` or `/app/team` directly **Then** they are redirected to `/app` with a brief toast "Kein Zugriff".

3. **Given** a user with role "meister" or "buero" **When** accessing any page **Then** all navigation items are visible (Dashboard, Projekte, Inbox, Team) **And** all projects for the company are shown (not filtered by membership).

4. **Given** the RBAC middleware **When** any Server Action is called **Then** `getAuthContext()` returns the role **And** the action checks `hasPermission()` before executing mutations **And** list queries are filtered: Monteur -> own projects only, Meister/Buero -> all company projects.

5. **Given** a Monteur assigned to specific projects **When** viewing a project they're assigned to **Then** the full project timeline with all messages is visible **When** trying to access a project they're NOT assigned to **Then** a 404 page is shown.

**FRs:** S2-FR5, S2-FR17, S2-FR18 | **NFRs:** S2-NFR2, S2-NFR3

## Tasks / Subtasks

- [x] Task 1: Add Monteur-aware project filtering to query functions (AC: #1, #3, #4, #5)
  - [x] 1.1 Update `getDashboardProjects()` in `lib/actions/projects.ts`: destructure `userId` + `role` from `getAuthContext()`, add conditional SQL — if `role === 'monteur'` add `AND EXISTS (SELECT 1 FROM project_members pm WHERE pm.project_id = p.id AND pm.user_id = ${userId})`, else no additional filter
  - [x] 1.2 Update `getProjectsWithStats()` in `lib/actions/projects.ts`: same pattern — add `EXISTS` subquery for monteur role
  - [x] 1.3 Update `getProject(id)` in `lib/actions/projects.ts`: for monteur role, add member check — if monteur AND `NOT EXISTS (SELECT 1 FROM project_members WHERE project_id = ${id} AND user_id = ${userId})` then return `null` (triggers 404 in project detail page)
  - [x] 1.4 Update `getProjectMessages(projectId)` in `lib/actions/projects.ts`: for monteur role, verify membership before returning messages — same `EXISTS` check, return empty array if not a member
  - [x] 1.5 Update `getProjects()` in `lib/actions/projects.ts` (used for dropdowns): for monteur role, add member filter via Drizzle `inArray` subquery on `project_members`

- [x] Task 2: Add RBAC permission gates to inbox functions (AC: #2, #4)
  - [x] 2.1 Update `getUnassignedMessages()` in `lib/actions/inbox.ts`: add `const { role } = await getAuthContext()` and `requirePermission(role, 'inbox:view')` at the top
  - [x] 2.2 Update `getInboxData()` in `lib/actions/inbox.ts`: add `requirePermission(role, 'inbox:view')` at the top (defense-in-depth — page also checks)
  - [x] 2.3 Update `assignMessageToProject()` in `lib/actions/inbox.ts`: add `requirePermission(role, 'inbox:assign')` before executing the mutation

- [x] Task 3: Add RBAC permission gates to project mutation actions (AC: #4)
  - [x] 3.1 Update `createProject()` in `lib/actions/projects.ts`: add `requirePermission(role, 'project:create')` — monteur cannot create projects
  - [x] 3.2 Update `updateProject()` in `lib/actions/projects.ts`: add `requirePermission(role, 'project:edit')` — monteur cannot edit project details
  - [x] 3.3 Update `updateProjectStatus()` in `lib/actions/projects.ts`: add `requirePermission(role, 'project:edit')` — monteur cannot change project status

- [x] Task 4: Add page-level route protection with redirect (AC: #2)
  - [x] 4.1 Update `app/app/inbox/page.tsx`: add `const { role } = await getAuthContext()` and `if (!hasPermission(role, 'inbox:view')) redirect('/app?denied=1')` at the top (before data fetching)
  - [x] 4.2 Verify `app/app/team/page.tsx` already has `requirePermission(role, 'team:manage')` — changed to `hasPermission()` check with `redirect('/app?denied=1')` for clean UX
  - [x] 4.3 Update `app/app/projects/[id]/page.tsx`: for monteur role, verify membership via `getProject(id)` return value (Task 1.3 makes it return null if not member) — existing `notFound()` call handles this

- [x] Task 5: Hide nav items based on role (AC: #1, #3)
  - [x] 5.1 Update `components/app/app-header.tsx`: hide "Posteingang" (Inbox) link for monteur role — add `if (hasPermission(role, 'inbox:view'))` before adding the inbox link to `navLinks`
  - [x] 5.2 Update `components/app/bottom-nav.tsx`: same — hide inbox tab for monteur role
  - [x] 5.3 Verify Team link already hidden for monteur (done in Story 12.2)

- [x] Task 6: Add toast notification for denied access (AC: #2)
  - [x] 6.1 Update `app/app/page.tsx`: read `denied` query param from `searchParams`, if present show AccessDeniedBanner client component
  - [x] 6.2 Created minimal `components/app/access-denied-banner.tsx` — auto-dismissing banner with ShieldAlert icon, cleans URL via history.replaceState

- [x] Task 7: Verify build + types (AC: all)
  - [x] 7.1 Run `npx tsc --noEmit` — zero TypeScript errors
  - [x] 7.2 Run `npx next build` — all routes compile successfully
  - [ ] 7.3 Manual test: Monteur login -> /app -> verify only assigned projects shown, Inbox/Team nav hidden
  - [ ] 7.4 Manual test: Monteur navigates to /app/inbox directly -> redirected to /app with "Kein Zugriff"
  - [ ] 7.5 Manual test: Monteur navigates to /app/projects/[unassigned-id] -> 404 page
  - [ ] 7.6 Manual test: Meister login -> verify all pages accessible, all projects visible

## Dev Notes

### Architecture & Implementation Patterns

- **Role-conditional SQL filtering**: For raw SQL queries (`getDashboardProjects`, `getProjectsWithStats`), add a conditional `AND EXISTS (SELECT 1 FROM project_members pm WHERE pm.project_id = p.id AND pm.user_id = ${userId})` clause only when `role === 'monteur'`. Meister/Buero skip this clause and see all company projects. This pattern avoids code duplication while keeping the SQL efficient. [Source: lib/actions/projects.ts]
- **Drizzle subquery filtering**: For Drizzle builder queries (`getProjects`, `getProjectMessages`), use `inArray(projects.id, db.select({ id: projectMembers.projectId }).from(projectMembers).where(eq(projectMembers.userId, userId)))` for monteur filtering. [Source: drizzle-orm docs]
- **Defense-in-depth RBAC**: Apply permission checks at BOTH the page level (redirect) AND the Server Action level (requirePermission). Page-level prevents UI access, action-level prevents API abuse. The `team.ts` pattern from Story 12.3 is the reference implementation. [Source: lib/actions/team.ts]
- **Redirect over PermissionError**: For page-level access denial, use `redirect('/app?denied=1')` instead of throwing `PermissionError`. Throwing causes an unhandled error page. `redirect()` provides a clean UX with a "Kein Zugriff" toast on the dashboard. [Source: next/navigation]
- **Toast for access denial**: Use query parameter `?denied=1` to trigger a brief "Kein Zugriff" notification on the dashboard. This is a simple pattern that works with Server Components without needing a toast library. A minimal client component can read `searchParams` and show a dismissible banner. [Source: AC #2]
- **project_members as access control**: The `project_members` table was created in migration 0006 (Story 8.1) as the membership table for Monteur-project relationships. It has a composite primary key `(projectId, userId)`. Currently EMPTY but populated via AI assignment (Story 9.2) or manual assignment. Story 12.4 enforces filtering USING this table, but does NOT change how members are added. [Source: lib/db/schema.ts lines 106-115]

### Critical Constraints

- **project_members may be empty**: If no rows exist in `project_members` for a Monteur, the `EXISTS` subquery returns false and the Monteur sees ZERO projects. This is correct behavior — a newly invited Monteur with no project assignments should see an empty dashboard with a friendly "Noch keine Projekte zugewiesen" message.
- **Do NOT modify middleware.ts for RBAC**: Middleware runs on the Edge Runtime with limited access. Role-based checks belong in Server Components and Server Actions, not middleware. Middleware continues to validate only session existence + deactivation check. [Source: Next.js 16 middleware docs, middleware.ts]
- **Team page already protected**: `app/app/team/page.tsx` already has `requirePermission(role, 'team:manage')` from Story 12.3. But it THROWS `PermissionError` instead of redirecting. Change to `if (!hasPermission(role, 'team:manage')) redirect('/app?denied=1')` for consistent UX.
- **Inbox link must be conditionally rendered**: Currently `app-header.tsx` and `bottom-nav.tsx` show "Posteingang" to ALL roles. Per AC #1, it MUST be hidden for monteur. The Team link is already conditionally rendered — apply the same pattern with `hasPermission(role, 'inbox:view')`.
- **`getAuthContext()` already returns role**: No changes needed to the auth module. `role` is typed as `Role` (from Story 12.1). All existing queries already destructure `companyId` from `getAuthContext()` — just add `userId` and `role` to the destructured fields.
- **SQL EXISTS vs JOIN**: Use `EXISTS` subquery (not JOIN) for member filtering to avoid duplicating project rows when a user has multiple membership entries (shouldn't happen with PK constraint, but EXISTS is semantically cleaner and never causes row multiplication).
- **No changes to `project_members` schema**: The table is correct as-is. Do NOT add timestamps or other columns — the composite PK `(projectId, userId)` is the minimal correct schema for a membership junction table.
- **Handle `redirect()` in Server Components**: `redirect()` from `next/navigation` throws internally in Next.js 16 — it must NOT be inside a try/catch block (or the catch must re-throw `redirect` errors). When converting `requirePermission()` to `redirect()`, remove any surrounding try/catch.

### Existing Code to Reuse

| Component/Function | Source | Reuse Approach |
|---|---|---|
| `getAuthContext()` | `lib/auth/session.ts` | Returns `{ userId, companyId, role }` — add `userId` + `role` to existing destructuring |
| `hasPermission()` | `lib/auth/rbac.ts` | Use for page-level checks (returns boolean, use with `if + redirect`) |
| `requirePermission()` | `lib/auth/rbac.ts` | Use for Server Action checks (throws PermissionError on denial) |
| `projectMembers` | `lib/db/schema.ts` | Import table for subqueries in project filtering |
| `getDashboardProjects()` | `lib/actions/projects.ts:167` | **MODIFY**: add role-conditional member filter |
| `getProjectsWithStats()` | `lib/actions/projects.ts:105` | **MODIFY**: add role-conditional member filter |
| `getProject(id)` | `lib/actions/projects.ts:68` | **MODIFY**: add member access check for monteur |
| `getProjectMessages()` | `lib/actions/projects.ts:206` | **MODIFY**: add member access check for monteur |
| `getProjects()` | `lib/actions/projects.ts:80` | **MODIFY**: add member filter for monteur |
| `getUnassignedMessages()` | `lib/actions/inbox.ts:29` | **MODIFY**: add `requirePermission(role, 'inbox:view')` |
| `getInboxData()` | `lib/actions/inbox.ts:132` | **MODIFY**: add `requirePermission(role, 'inbox:view')` |
| `assignMessageToProject()` | `lib/actions/inbox.ts:34` | **MODIFY**: add `requirePermission(role, 'inbox:assign')` |
| `createProject()` | `lib/actions/projects.ts:14` | **MODIFY**: add `requirePermission(role, 'project:create')` |
| `updateProject()` | `lib/actions/projects.ts` | **MODIFY**: add `requirePermission(role, 'project:edit')` |
| `updateProjectStatus()` | `lib/actions/projects.ts` | **MODIFY**: add `requirePermission(role, 'project:edit')` |
| AppHeader | `components/app/app-header.tsx` | **MODIFY**: hide Inbox link for monteur |
| BottomNav | `components/app/bottom-nav.tsx` | **MODIFY**: hide Inbox tab for monteur |
| Team page | `app/app/team/page.tsx` | **MODIFY**: change PermissionError throw to redirect |
| Inbox page | `app/app/inbox/page.tsx` | **MODIFY**: add permission check + redirect |
| Project detail | `app/app/projects/[id]/page.tsx` | **MODIFY**: 404 for unassigned monteur (via getProject returning null) |
| Dashboard | `app/app/page.tsx` | **MODIFY**: read `denied` searchParam, show "Kein Zugriff" banner |

### SQL Pattern: Monteur Member Filter

```sql
-- For getDashboardProjects() and getProjectsWithStats():
-- Add this clause conditionally when role = 'monteur':
AND EXISTS (
  SELECT 1 FROM project_members pm
  WHERE pm.project_id = p.id
  AND pm.user_id = ${userId}
)

-- For getProject(id) — monteur access check:
-- After fetching project by id + companyId, verify membership:
AND (
  ${role} != 'monteur'
  OR EXISTS (
    SELECT 1 FROM project_members pm
    WHERE pm.project_id = p.id
    AND pm.user_id = ${userId}
  )
)
```

### Drizzle Pattern: Monteur Member Filter

```typescript
// For getProjects() using Drizzle builder:
import { projectMembers } from "@/lib/db/schema";
import { inArray } from "drizzle-orm";

const baseWhere = eq(projects.companyId, companyId);
const memberFilter = role === "monteur"
  ? and(
      baseWhere,
      inArray(
        projects.id,
        db.select({ id: projectMembers.projectId })
          .from(projectMembers)
          .where(eq(projectMembers.userId, userId))
      )
    )
  : baseWhere;

return db.select().from(projects).where(memberFilter);
```

### Empty State: Monteur with No Assignments

When a Monteur has no `project_members` rows (newly invited, no projects assigned yet), the dashboard and project list should show a friendly empty state:

```
Noch keine Projekte zugewiesen.
Dein Meister wird dir Projekte zuweisen.
```

This replaces the generic "Noch keine aktiven Projekte" message when the user is a monteur with zero results.

### Navigation Visibility Matrix

```
Nav Item      | monteur | meister | buero
--------------|---------|---------|------
Aufnahme      |    ✓    |    ✓    |   ✓
Ubersicht     |    ✓    |    ✓    |   ✓
Projekte      |    ✓    |    ✓    |   ✓
Posteingang   |         |    ✓    |   ✓
Team          |         |    ✓    |   ✓
```

### Route Protection Matrix

```
Route                  | monteur        | meister/buero | Protection Mechanism
-----------------------|----------------|---------------|---------------------
/app                   | Own projects   | All projects  | Query filter
/app/projects          | Own projects   | All projects  | Query filter
/app/projects/[id]     | If assigned    | Always        | Query returns null -> 404
/app/inbox             | Redirect /app  | Full access   | Page-level redirect
/app/team              | Redirect /app  | Full access   | Page-level redirect
/app/aufnahme          | Full access    | Full access   | No restriction
```

### Server Action Permission Matrix

```
Action                   | Permission Required     | monteur | meister | buero
-------------------------|-------------------------|---------|---------|------
createProject()          | project:create          |    ✗    |    ✓    |   ✓
updateProject()          | project:edit            |    ✗    |    ✓    |   ✓
updateProjectStatus()    | project:edit            |    ✗    |    ✓    |   ✓
assignMessageToProject() | inbox:assign            |    ✗    |    ✓    |   ✓
getUnassignedMessages()  | inbox:view              |    ✗    |    ✓    |   ✓
getInboxData()           | inbox:view              |    ✗    |    ✓    |   ✓
getTeamMembers()         | team:manage (already)   |    ✗    |    ✓    |   ✓
createInvitation()       | team:invite (already)   |    ✗    |    ✓    |   ✓
```

### Edge Cases

| Case | Behavior |
|---|---|
| Monteur with zero project_members rows | Empty dashboard: "Noch keine Projekte zugewiesen" |
| Monteur assigned to project, then removed from project_members | Dashboard updates immediately (query-time check, no cache) |
| Monteur guesses project UUID in URL | getProject() returns null -> notFound() -> 404 page |
| Monteur calls createProject() directly (API abuse) | requirePermission throws PermissionError -> action returns error |
| Monteur calls assignMessageToProject() directly | requirePermission throws PermissionError -> action returns error |
| Meister downgrades self to Monteur (Story 12.3 allows this) | Immediately sees filtered dashboard on next page load |
| Role change while session active | getAuthContext() reads fresh role from DB each time (no caching) |
| project_members row deleted while Monteur on project detail | Next page load/refresh shows 404 (no real-time WebSocket eviction) |
| Concurrent role changes | getAuthContext() queries DB per request — always current role |
| Team page: existing PermissionError behavior | Replace throw with redirect('/app?denied=1') for clean UX |

### Project Structure Notes

**No new files needed** (unless toast component required):
- Optional: `components/app/access-denied-toast.tsx` — Client Component for "Kein Zugriff" banner (only if no toast library exists)

**Modified files:**
- `lib/actions/projects.ts` — Add role-conditional member filtering to ALL query functions + requirePermission to mutation functions
- `lib/actions/inbox.ts` — Add requirePermission('inbox:view') to query functions + requirePermission('inbox:assign') to assignMessageToProject
- `components/app/app-header.tsx` — Hide Inbox link for monteur role
- `components/app/bottom-nav.tsx` — Hide Inbox tab for monteur role
- `app/app/inbox/page.tsx` — Add permission check with redirect
- `app/app/team/page.tsx` — Change PermissionError throw to redirect
- `app/app/page.tsx` — Add `denied` searchParam handling for "Kein Zugriff" toast, update empty state message for monteur
- `app/app/projects/[id]/page.tsx` — Monteur access check handled by getProject() returning null (existing notFound() handles it)

### Testing Strategy

- **Test 1**: Meister login -> `/app` -> verify ALL active projects visible (regression)
- **Test 2**: Meister login -> `/app/inbox` -> accessible, all unassigned messages shown (regression)
- **Test 3**: Meister login -> `/app/team` -> accessible (regression)
- **Test 4**: Monteur login -> `/app` -> only projects from project_members shown
- **Test 5**: Monteur login -> verify Inbox and Team nav items are NOT visible
- **Test 6**: Monteur navigates to `/app/inbox` directly (URL bar) -> redirected to `/app?denied=1` -> "Kein Zugriff" toast shown
- **Test 7**: Monteur navigates to `/app/team` directly -> redirected to `/app?denied=1` -> "Kein Zugriff" toast shown
- **Test 8**: Monteur navigates to `/app/projects/[assigned-id]` -> full project timeline visible
- **Test 9**: Monteur navigates to `/app/projects/[unassigned-id]` -> 404 page
- **Test 10**: Monteur with zero assignments -> empty dashboard with "Noch keine Projekte zugewiesen" message
- **Test 11**: `npx tsc --noEmit` passes with zero errors
- **Test 12**: `npx next build` succeeds with all routes compiling

### References

- [Source: _bmad-output/planning-artifacts/epics.md#Story 12.4] — Acceptance criteria (S2-FR5, S2-FR17, S2-FR18)
- [Source: _bmad-output/planning-artifacts/epics.md#S2-FR5] — "RBAC middleware enforces permissions: Monteur can only view assigned projects, Inbox + Team pages restricted to Meister/Buero"
- [Source: _bmad-output/planning-artifacts/epics.md#S2-FR17] — "Monteur dashboard shows only assigned projects (RBAC-filtered)"
- [Source: _bmad-output/planning-artifacts/epics.md#S2-FR18] — "Inbox badge visible only to Meister/Buero roles"
- [Source: _bmad-output/planning-artifacts/epics.md#S2-NFR2] — "Tenant isolation on all new endpoints (companyId filter)"
- [Source: _bmad-output/planning-artifacts/epics.md#S2-NFR3] — "RBAC middleware on all protected routes with role-based permission checks"
- [Source: lib/auth/rbac.ts] — RBAC module: hasPermission, requirePermission, Permission type, PERMISSIONS map
- [Source: lib/db/schema.ts#projectMembers] — project_members table: (projectId, userId) composite PK
- [Source: lib/actions/projects.ts] — All project query functions (getDashboardProjects, getProjectsWithStats, getProject, getProjectMessages, getProjects)
- [Source: lib/actions/inbox.ts] — Inbox functions (fetchUnassigned, getUnassignedMessages, getInboxData, assignMessageToProject)
- [Source: lib/actions/team.ts] — Reference implementation for requirePermission pattern
- [Source: lib/auth/session.ts#getAuthContext] — Auth context: returns { userId, companyId, role }
- [Source: components/app/app-header.tsx] — Desktop nav with conditional Team link (pattern to extend for Inbox)
- [Source: components/app/bottom-nav.tsx] — Mobile nav with conditional Team link (pattern to extend for Inbox)
- [Source: _bmad-output/implementation-artifacts/12-3-team-management-ui.md] — Previous story: team management, deactivation, soft-delete pattern
- [Source: _bmad-output/implementation-artifacts/12-2-invitation-system-create-send-redeem.md] — Invitation system, RBAC nav gating
- [Source: _bmad-output/implementation-artifacts/12-1-pgenum-role-migration-and-rbac-permission-module.md] — RBAC module, pgEnum, permission matrix

### Previous Story Intelligence (12.3)

- **Team page RBAC check exists but throws**: `requirePermission(role, 'team:manage')` at page level throws `PermissionError` which causes an ugly error page for Monteur. Story 12.4 should change this to a `hasPermission()` check with `redirect('/app?denied=1')` for clean UX.
- **Soft-deactivation pattern**: `deactivatedAt` column added, middleware + getAuthContext both check `isNull(users.deactivatedAt)`. No changes needed for Story 12.4.
- **Server Action pattern**: `getAuthContext()` + `requirePermission()` before any mutation. This is the reference pattern for inbox and project actions.
- **`getTeamMembers()` uses `requirePermission()`**: This is the only query function that has RBAC enforcement. All project and inbox queries are missing this pattern.
- **shadcn AlertDialog installed**: Available if needed for any confirmation dialogs (not expected for Story 12.4).
- **ROLE_LABELS and ROLE_OPTIONS**: Shared in `lib/validations/team.ts` — available for role display in empty states or access denied messages.

### Git Intelligence

Recent commits:
- `90fd25b` Add pgEnum for roles/status columns, RBAC permission module, and invitations schema (Story 12.1)
- Uncommitted changes include Stories 12.2 + 12.3 (invitation system + team management UI)

Pattern: Single commit per story, descriptive message referencing story number.
Suggested commit: `"Enforce RBAC route protection and Monteur data filtering across all pages (Story 12.4)"`

## Change Log

- 2026-02-16: Implemented all RBAC route protection and Monteur data filtering (Tasks 1-7)
- 2026-02-16: Code review fixes — added RBAC to getUnassignedCount/getProjectPreFillData/createProjectAndAssignMessage, conditional layout query, single-query membership checks, hid edit controls for monteur, removed dead import

## Senior Developer Review (AI)

**Reviewer:** Sempre on 2026-02-16
**Outcome:** Approved (after fixes)

### Findings (7 total: 2 High, 3 Medium, 2 Low — all fixed)

| # | Severity | Issue | Fix Applied |
|---|----------|-------|-------------|
| H1 | HIGH | `createProjectAndAssignMessage()` had no RBAC — monteur could bypass project:create + inbox:assign | Added `requirePermission(role, "project:create")` |
| H2 | HIGH | `getUnassignedCount()` had no RBAC + called unconditionally in layout for all users | Added `requirePermission(role, "inbox:view")` + conditional call in layout |
| M1 | MEDIUM | `getProjectPreFillData()` had no RBAC — monteur could extract transcript data | Added `requirePermission(role, "inbox:view")` |
| M2 | MEDIUM | Project detail page showed Bearbeiten/StatusToggle/AssignmentDropdown to monteur | Added `hasPermission()` conditional rendering for all controls |
| M3 | MEDIUM | `getProject()` + `getProjectMessages()` used separate membership queries (2 extra roundtrips) | Refactored to single-query with `EXISTS` in WHERE clause |
| L1 | LOW | Unused `useRouter` import in AccessDeniedBanner | Removed dead import |
| L2 | LOW | AssignmentDropdown on project detail visible to monteur | Wrapped in `hasPermission(role, "inbox:assign")` check |

### Verification

- `tsc --noEmit`: 0 errors
- `next build`: All routes compile successfully
- Manual tests (7.3-7.6): Pending user verification

## Dev Agent Record

### Agent Model Used

Claude Opus 4.6

### Debug Log References

- No debug issues encountered

### Completion Notes List

- Task 1: Added role-conditional member filtering to all 5 project query functions (`getDashboardProjects`, `getProjectsWithStats`, `getProject`, `getProjectMessages`, `getProjects`). Raw SQL functions use conditional `EXISTS` subquery; Drizzle builder uses `inArray` subquery on `project_members`. Meister/Buero skip the filter and see all company projects.
- Task 2: Added `requirePermission(role, 'inbox:view')` to `getUnassignedMessages()` and `getInboxData()`, and `requirePermission(role, 'inbox:assign')` to `assignMessageToProject()`. Defense-in-depth: page-level AND action-level checks.
- Task 3: Added `requirePermission(role, 'project:create')` to `createProject()` and `requirePermission(role, 'project:edit')` to `updateProject()` and `updateProjectStatus()`.
- Task 4: Added `hasPermission() + redirect('/app?denied=1')` to inbox page. Changed team page from `requirePermission()` (throws) to `hasPermission() + redirect()` for clean UX. Project detail page already handles via `getProject()` returning null → `notFound()`.
- Task 5: Moved Inbox link from `baseNavLinks` to conditional insertion using `hasPermission(role, 'inbox:view')` in both `AppHeader` and `BottomNav`. Team link was already conditional from Story 12.2.
- Task 6: Created `AccessDeniedBanner` client component with auto-dismiss (4s), ShieldAlert icon, and URL cleanup via `history.replaceState`. Dashboard page reads `?denied=1` searchParam.
- Task 7: `tsc --noEmit` passes with zero errors; `next build` succeeds with all routes compiling. Manual tests (7.3-7.6) require user verification.

### File List

- `lib/actions/projects.ts` — Modified: added role-conditional member filtering to all query functions + requirePermission to mutation functions
- `lib/actions/inbox.ts` — Modified: added requirePermission to getUnassignedMessages, getInboxData, assignMessageToProject, getUnassignedCount, getProjectPreFillData, createProjectAndAssignMessage
- `components/app/app-header.tsx` — Modified: conditional Inbox link based on role
- `components/app/bottom-nav.tsx` — Modified: conditional Inbox tab based on role
- `app/app/inbox/page.tsx` — Modified: added hasPermission check with redirect
- `app/app/team/page.tsx` — Modified: changed requirePermission throw to hasPermission + redirect
- `app/app/page.tsx` — Modified: added denied searchParam handling, role-specific empty state for monteur
- `components/app/access-denied-banner.tsx` — New: auto-dismissing "Kein Zugriff" banner client component (review: removed unused useRouter import)
- `app/app/projects/[id]/page.tsx` — Modified: conditionally hide Bearbeiten button, StatusToggle, AssignmentDropdown for monteur
- `app/app/layout.tsx` — Modified: conditional getUnassignedCount() call (skip for monteur)
- `_bmad-output/implementation-artifacts/sprint-status.yaml` — Modified: story status updated
- `_bmad-output/implementation-artifacts/12-4-route-protection-and-monteur-data-filtering.md` — Modified: task checkboxes, dev agent record
