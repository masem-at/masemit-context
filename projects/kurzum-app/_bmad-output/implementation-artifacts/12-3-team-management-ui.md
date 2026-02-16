# Story 12.3: Team Management UI

Status: done

<!-- Note: Validation is optional. Run validate-create-story for quality check before dev-story. -->

## Story

As a **Meister**,
I want to see all team members, change their roles, and deactivate members,
so that I can manage who has access to my company's data.

## Acceptance Criteria

1. **Given** a Meister/Buero on `/app/team` **When** the page loads **Then** all team members for the company are listed: name, email, role, joined date **And** pending invitations are shown separately with: email, role, invited by, expires at **And** the page is only accessible with `team:manage` permission (Meister/Buero).

2. **Given** a Meister viewing a team member **When** clicking the role dropdown **Then** the role can be changed to Monteur/Meister/Buero **And** a confirmation prompt appears for role changes **And** the change is saved immediately via Server Action.

3. **Given** a Meister viewing a team member **When** clicking "Entfernen" **Then** the user is soft-deactivated (not deleted from DB) **And** their sessions are invalidated **And** they can no longer log in **And** a confirmation dialog warns about the consequences.

4. **Given** a pending invitation **When** a Meister clicks "Zuruckziehen" **Then** the invitation is deleted and can no longer be redeemed.

**FRs:** S2-FR4 | **NFRs:** S2-NFR7

## Tasks / Subtasks

- [x] Task 1: Database migration — add `deactivatedAt` column to users table (AC: #3)
  - [x] 1.1 Add `deactivatedAt: timestamp("deactivated_at", { withTimezone: true })` to users table in `lib/db/schema.ts` (nullable, null = active)
  - [x] 1.2 Run `drizzle-kit generate` to create migration SQL
  - [x] 1.3 Run `drizzle-kit migrate` to apply migration to NeonDB
  - [x] 1.4 Verify migration success: `SELECT column_name FROM information_schema.columns WHERE table_name = 'users' AND column_name = 'deactivated_at'`

- [x] Task 2: Update middleware + auth to block deactivated users (AC: #3)
  - [x] 2.1 Update `middleware.ts`: add `users.deactivatedAt IS NULL` check to the session+user join query — deactivated users get redirected to `/login` with session cookie deleted
  - [x] 2.2 Update `getAuthContext()` in `lib/auth/session.ts`: add `isNull(users.deactivatedAt)` to the WHERE clause — deactivated users get "Invalid or expired session" error
  - [x] 2.3 Import `isNull` from `drizzle-orm` in both files

- [x] Task 3: Create team management Server Actions (AC: #1, #2, #3, #4)
  - [x] 3.1 Add `getTeamMembers(companyId: string)` to `lib/actions/team.ts`: query users table WHERE companyId matches AND deactivatedAt IS NULL, return id, name, email, role, createdAt, ordered by createdAt ASC
  - [x] 3.2 Add `changeUserRole(userId: string, newRole: Role)` Server Action: `getAuthContext()` + `requirePermission(role, 'team:manage')`, validate newRole is valid pgEnum value, prevent self-role-change, UPDATE users SET role WHERE id AND companyId matches, revalidatePath('/app/team')
  - [x] 3.3 Add `deactivateUser(userId: string)` Server Action: `getAuthContext()` + `requirePermission(role, 'team:manage')`, prevent self-deactivation, UPDATE users SET deactivatedAt = now() WHERE id AND companyId matches AND deactivatedAt IS NULL, DELETE all sessions for that userId, revalidatePath('/app/team')
  - [x] 3.4 Add `revokeInvitation(invitationId: string)` Server Action: `getAuthContext()` + `requirePermission(role, 'team:manage')`, DELETE FROM invitations WHERE id AND companyId matches AND usedAt IS NULL, revalidatePath('/app/team')
  - [x] 3.5 Update `getPendingInvitations(companyId)`: add `invitedBy` user name via JOIN to return inviter name alongside existing fields, ALSO return invitation `id` (required for `revokeInvitation()`)
  - [x] 3.6 Extract shared `ROLE_LABELS` map to `lib/validations/team.ts` (currently duplicated in `app/app/team/page.tsx` and `app/invite/[token]/page.tsx`) — import from there in all team-related components. InviteForm uses separate `ROLE_OPTIONS` array (value+label pairs) — keep that for Select component, derive from `ROLE_LABELS` or keep separate.

- [x] Task 4: Create Team Members List UI component (AC: #1, #2, #3)
  - [x] 4.0 Install shadcn AlertDialog: `npx shadcn@latest add alert-dialog` (not yet installed — needed for confirmation dialogs)
  - [x] 4.1 Create `components/app/team-members-list.tsx` — Client Component ("use client") accepting `members` array + `currentUserId` as props
  - [x] 4.2 List each member: name, email, role dropdown (shadcn Select), joined date (German format "dd.MM.yyyy"), "Entfernen" button (destructive variant)
  - [x] 4.3 Role dropdown: use shadcn `Select` + `Controller` pattern (same as invite form), disabled for current user (cannot change own role), calls `changeUserRole()` on value change
  - [x] 4.4 Before role change: show confirmation dialog (shadcn `AlertDialog`): "Rolle von {name} zu {newRole} andern?" with "Abbrechen" and "Rolle andern" buttons
  - [x] 4.5 "Entfernen" button: show confirmation dialog: "Mitarbeiter entfernen — {name} wird deaktiviert und kann sich nicht mehr einloggen. Diese Aktion kann nicht ruckgangig gemacht werden." with "Abbrechen" and "Entfernen" (destructive) buttons
  - [x] 4.6 Call `deactivateUser()` on confirm, show success feedback
  - [x] 4.7 Current user row: hide "Entfernen" button, disable role dropdown, show "(Du)" label next to name
  - [x] 4.8 Touch targets >= 48px for all interactive elements (S2-NFR7)

- [x] Task 5: Create Pending Invitations List with revoke action (AC: #4)
  - [x] 5.1 Create `components/app/pending-invitations-list.tsx` — Client Component accepting `invitations` array as props
  - [x] 5.2 List each invitation: email, role (German label), invited by (inviter name), expires date (German format)
  - [x] 5.3 Add "Zuruckziehen" button per invitation (ghost variant, destructive text)
  - [x] 5.4 Confirmation dialog: "Einladung zuruckziehen — Die Einladung an {email} wird geloscht und kann nicht mehr eingelost werden."
  - [x] 5.5 Call `revokeInvitation()` on confirm
  - [x] 5.6 Touch targets >= 48px (S2-NFR7)

- [x] Task 6: Update Team Page to compose all sections (AC: #1)
  - [x] 6.1 Update `app/app/team/page.tsx`: add call to `getTeamMembers(companyId)`, pass `currentUserId` from auth context
  - [x] 6.2 Page layout: (1) Header "Team" + subtitle, (2) Team Members section with TeamMembersList component, (3) Invite Form section (existing), (4) Pending Invitations section with PendingInvitationsList component
  - [x] 6.3 Pass `members` and `currentUserId` to TeamMembersList
  - [x] 6.4 Pass `invitations` to PendingInvitationsList
  - [x] 6.5 Ensure sections are visually separated with consistent card styling (rounded-lg border bg-card p-6 pattern)

- [x] Task 7: Verify build + types (AC: all)
  - [x] 7.1 Run `npx tsc --noEmit` — zero TypeScript errors
  - [x] 7.2 Run `npx next build` — all routes compile successfully
  - [ ] 7.3 Manual test: Meister login → /app/team → verify team members list, role change, deactivation, invitation revoke

## Dev Notes

### Architecture & Implementation Patterns

- **Server Actions for mutations**: All mutations use Server Actions consistent with the existing codebase. `getAuthContext()` provides auth context, `requirePermission()` enforces RBAC before any DB writes. [Source: lib/auth/session.ts, lib/auth/rbac.ts]
- **Server Components for pages**: `/app/team` is an async Server Component with direct Drizzle queries. Client Components only for interactive lists (TeamMembersList, PendingInvitationsList). [Source: app/app/team/page.tsx]
- **Soft-deactivation pattern**: Add `deactivatedAt` timestamp column (nullable). `NULL` = active, non-null = deactivated timestamp. Do NOT delete user records — voice messages, projects, audit trail must be preserved. [Source: docs/_masemIT/bmad-briefing-kurzum-sprint2.md line 158]
- **Session invalidation**: DELETE all rows from `sessions` WHERE `userId = deactivatedUserId`. Combined with middleware `deactivatedAt IS NULL` check, deactivated users are immediately locked out. [Source: lib/auth/session.ts, middleware.ts]
- **shadcn AlertDialog for confirmations**: Use `AlertDialog` from shadcn/ui for all destructive actions (role change, deactivation, invitation revoke). Pattern: trigger button → dialog with warning text → cancel/confirm buttons. [Source: shadcn/ui AlertDialog docs]
- **shadcn Select + Controller (RHF)**: For role dropdowns in the team member list, use the same pattern established in the InviteForm: `Controller` wrapper around `Select`. [Source: components/app/invite-form.tsx]
- **Drizzle migration workflow**: `drizzle-kit generate` → `drizzle-kit migrate` (NOT push). Journal timestamps must be ascending. [Source: MEMORY.md#Story 8.1]

### Critical Constraints

- **MUST add `deactivatedAt` column via migration**: The users table currently has NO deactivation mechanism. A Drizzle migration adding `deactivatedAt timestamp(with timezone) NULL` is required BEFORE any deactivation logic works.
- **MUST update middleware.ts**: Current middleware only checks session validity + companyId. It does NOT check if a user is deactivated. Without adding `isNull(users.deactivatedAt)` to the WHERE clause, deactivated users can still access the app until their session expires.
- **MUST update getAuthContext()**: Same as middleware — the auth context query must exclude deactivated users so Server Actions also reject them.
- **Prevent self-deactivation**: The `deactivateUser()` action MUST reject if `targetUserId === authContext.userId`. A Meister should not lock themselves out.
- **Prevent self-role-change**: The `changeUserRole()` action MUST reject if `targetUserId === authContext.userId`. This prevents a Meister from accidentally downgrading themselves to Monteur.
- **CompanyId scoping on ALL mutations**: Every Server Action must verify the target user/invitation belongs to the same `companyId` as the authenticated user. This is multi-tenant isolation — a Meister in Company A cannot manage users in Company B.
- **No Drizzle `onConflict` needed**: Role changes and deactivations are simple UPDATEs with WHERE clauses, not upserts.
- **No undo for deactivation**: Per AC, deactivation is one-way in this story. If needed later, a reactivation feature can be added in a future story.
- **Pending invitations include inviter name**: The `getPendingInvitations()` query needs a JOIN with users table to get inviter name. Currently it only returns email, role, expiresAt, createdAt.

### Existing Code to Reuse

| Component/Function | Source | Reuse Approach |
|---|---|---|
| `getAuthContext()` | `lib/auth/session.ts` | Auth context for Server Actions — **MODIFY**: add deactivatedAt check |
| `hasPermission()` / `requirePermission()` | `lib/auth/rbac.ts` | Check `team:manage` before all team mutations |
| `createInvitation()` | `lib/actions/team.ts` | Existing — no changes |
| `getPendingInvitations()` | `lib/actions/team.ts` | **MODIFY**: add inviter name via JOIN |
| `InviteForm` | `components/app/invite-form.tsx` | Existing component — no changes |
| Team page | `app/app/team/page.tsx` | **MODIFY**: add team members list + updated invitations list |
| `deleteSession()` | `lib/auth/session.ts` | Not directly reusable — need bulk delete by userId (new query) |
| Middleware | `middleware.ts` | **MODIFY**: add deactivatedAt IS NULL check |
| ROLE_LABELS map | `app/app/team/page.tsx` | **EXTRACT** to shared constant or keep in page (currently inline) |
| `revalidatePath()` | Next.js | Same pattern for cache invalidation after mutations |
| `AlertDialog` | shadcn/ui | **NEW**: needs to be added via `npx shadcn@latest add alert-dialog` |
| `Select` | shadcn/ui | Already installed — reuse for role dropdown in team member rows |

### Schema Change Reference

```typescript
// lib/db/schema.ts — ADD to users table:
deactivatedAt: timestamp("deactivated_at", { withTimezone: true }),
// nullable: null = active, non-null = deactivated at this timestamp
```

### Middleware Change Reference

```typescript
// middleware.ts — ADD to WHERE clause:
import { isNull } from "drizzle-orm";
// ...
.where(and(
  eq(sessions.token, sessionToken),
  gt(sessions.expiresAt, new Date()),
  isNull(users.deactivatedAt)  // <-- NEW: block deactivated users
))
```

### Server Action Signatures

```typescript
// lib/actions/team.ts — NEW exports:
export async function getTeamMembers(companyId: string): Promise<TeamMember[]>
export async function changeUserRole(userId: string, newRole: Role): Promise<ActionResult>
export async function deactivateUser(userId: string): Promise<ActionResult>
export async function revokeInvitation(invitationId: string): Promise<ActionResult>

// Shared result type (extend existing InviteResult pattern):
export type ActionResult =
  | { success: true }
  | { success: false; error: string };

// Team member type:
export interface TeamMember {
  id: string;
  name: string;
  email: string;
  role: string;
  createdAt: Date;
}
```

### Confirmation Dialog Copy (German)

| Action | Dialog Title | Dialog Description | Cancel | Confirm |
|---|---|---|---|---|
| Role change | "Rolle andern" | "{name} wird die Rolle {newRoleLabel} zugewiesen." | "Abbrechen" | "Rolle andern" |
| Deactivate user | "Mitarbeiter entfernen" | "{name} wird deaktiviert und kann sich nicht mehr einloggen. Bestehende Daten bleiben erhalten." | "Abbrechen" | "Entfernen" (destructive) |
| Revoke invitation | "Einladung zuruckziehen" | "Die Einladung an {email} wird geloscht und kann nicht mehr eingelost werden." | "Abbrechen" | "Zuruckziehen" |

### Edge Cases

| Case | Behavior |
|---|---|
| Meister tries to deactivate themselves | Reject: "Du kannst dich nicht selbst deaktivieren." |
| Meister tries to change own role | Reject: show role dropdown as disabled, "(Du)" label |
| Last Meister in company tries to downgrade to Monteur | Allowed — no minimum-role enforcement in Sprint 2 scope |
| Deactivated user tries to log in | Magic link still works for email, but session creation in `findOrCreateUser()` checks user exists. Middleware blocks on `deactivatedAt IS NOT NULL`. User sees login page. |
| Race condition: two Meisters deactivate same user | First succeeds (deactivatedAt set + sessions deleted). Second sees user already has deactivatedAt set — UPDATE affects 0 rows. No error, idempotent. |
| Deactivated user's data | Voice messages, projects, project_members remain intact. Historical data preserved for company timeline. |
| Invitation for deactivated user's email | Allowed — invitation targets email, not userId. If redeemed, creates/updates user (but user has deactivatedAt set). Need to handle: if email matches deactivated user, invitation redemption should clear deactivatedAt (DEFER to future story — for now, deactivated user with pending invite is an edge case). |
| Company with only 1 active user, deactivation attempted | Allowed — no minimum-member enforcement in Sprint 2 scope |

### Project Structure Notes

**New files:**
- `components/app/team-members-list.tsx` — Client Component: team members list with role change + deactivation
- `components/app/pending-invitations-list.tsx` — Client Component: pending invitations with revoke action

**Modified files:**
- `lib/db/schema.ts` — Add `deactivatedAt` column to users table
- `middleware.ts` — Add `isNull(users.deactivatedAt)` check
- `lib/auth/session.ts` — Add `isNull(users.deactivatedAt)` to `getAuthContext()` WHERE clause
- `lib/actions/team.ts` — Add `getTeamMembers()`, `changeUserRole()`, `deactivateUser()`, `revokeInvitation()` Server Actions, modify `getPendingInvitations()` to include inviter name
- `app/app/team/page.tsx` — Compose team members list + pending invitations list + invite form

**New shadcn component (install required):**
- `alert-dialog` — for confirmation dialogs (`npx shadcn@latest add alert-dialog`)

**Migration file (auto-generated):**
- `drizzle/XXXX_*.sql` — ALTER TABLE users ADD COLUMN deactivated_at TIMESTAMPTZ

### Testing Strategy

- **Test 1**: Meister logs in → `/app/team` → verify team members listed with name, email, role, joined date
- **Test 2**: Change a team member's role from Monteur to Buero → confirm dialog → verify role updated in DB
- **Test 3**: Click "Entfernen" on a team member → confirm dialog → verify `deactivatedAt` is set, sessions deleted
- **Test 4**: Deactivated user tries to visit `/app` → redirected to `/login` (middleware blocks)
- **Test 5**: Click "Zuruckziehen" on a pending invitation → confirm dialog → verify invitation deleted from DB
- **Test 6**: Verify current user row shows "(Du)", disabled role dropdown, no "Entfernen" button
- **Test 7**: Monteur tries to access `/app/team` → permission denied / redirect (unchanged from Story 12.2)
- **Test 8**: `npx tsc --noEmit` passes with zero errors
- **Test 9**: `npx next build` succeeds with all routes compiling

### References

- [Source: _bmad-output/planning-artifacts/epics.md#Story 12.3] — Acceptance criteria (S2-FR4)
- [Source: docs/_masemIT/bmad-briefing-kurzum-sprint2.md#AP 2.2 line 158] — "Mitarbeiter entfernen (Soft-Delete: deaktivieren, nicht loschen)"
- [Source: _bmad-output/planning-artifacts/epics.md#S2-FR4] — "Meister/Buro can view, change roles, and deactivate team members on /app/team"
- [Source: _bmad-output/planning-artifacts/epics.md#S2-NFR7] — "All new UI elements with >=48px touch targets and WCAG AA contrast"
- [Source: lib/db/schema.ts#users] — Users table (needs deactivatedAt column)
- [Source: lib/auth/session.ts#getAuthContext] — Auth context query (needs deactivatedAt check)
- [Source: middleware.ts] — Route protection (needs deactivatedAt check)
- [Source: lib/actions/team.ts] — Existing Server Actions (needs new team management actions)
- [Source: app/app/team/page.tsx] — Existing team page (needs team members list + invitations revoke)
- [Source: lib/auth/rbac.ts] — RBAC module: team:manage permission for all team management actions
- [Source: components/app/invite-form.tsx] — Select + Controller pattern to reuse for role dropdown
- [Source: _bmad-output/implementation-artifacts/12-2-invitation-system-create-send-redeem.md] — Previous story: invitation system, team page foundation

### Previous Story Intelligence (12.2)

- **Team page already exists with invite form + pending list**: `/app/team` has InviteForm component and pending invitations display. Story 12.3 extends this page with team members list and management actions.
- **RBAC gating is already in place**: Page checks `requirePermission(role, 'team:manage')` — no changes needed for access control.
- **InviteForm uses Controller + Select pattern**: Same pattern for role dropdowns in team member rows.
- **Fire-and-forget email pattern**: Not directly needed for Story 12.3 (no emails sent on role change or deactivation).
- **Code review findings from 12.2**: NavLink array type fix (use `NavLink[]` not `as const`), Select reset with `value ?? ""`, truncate CSS for BottomNav labels — all already applied.
- **ROLE_LABELS map**: Currently defined inline in `app/app/team/page.tsx` — can be extracted to a shared location or kept inline since it's only used in team-related components.

### Git Intelligence

Recent commits:
- `90fd25b` Add pgEnum for roles/status columns, RBAC permission module, and invitations schema (Story 12.1)
- Uncommitted changes include Story 12.2 implementation (invitation system: create, send & redeem)

Pattern: Single commit per story, descriptive message referencing story number.
Suggested commit: `"Implement team management UI: members list, role changes, deactivation (Story 12.3)"`

## Dev Agent Record

### Agent Model Used

Claude Opus 4.6

### Debug Log References

- No debug issues encountered. All tasks completed cleanly.

### Completion Notes List

- Task 1: Added `deactivatedAt` column to users table via Drizzle migration (0010_sharp_lady_bullseye.sql). Verified in NeonDB.
- Task 2: Updated `middleware.ts` and `getAuthContext()` with `isNull(users.deactivatedAt)` — deactivated users are immediately blocked from both middleware and Server Action access.
- Task 3: Implemented 4 new Server Actions (`getTeamMembers`, `changeUserRole`, `deactivateUser`, `revokeInvitation`), updated `getPendingInvitations` with inviter name JOIN and invitation id. All mutations scoped to `companyId` for multi-tenant isolation. Self-deactivation and self-role-change prevented.
- Task 3.6: Extracted `ROLE_LABELS` to `lib/validations/team.ts` — eliminated duplication in `app/app/team/page.tsx` and `app/invite/[token]/page.tsx`.
- Task 4: Created `TeamMembersList` client component with Select role dropdown, AlertDialog confirmations for role change and deactivation, current user protection ("(Du)" label, disabled controls), 48px touch targets.
- Task 5: Created `PendingInvitationsList` client component with inviter name display and AlertDialog revoke confirmation, 48px touch targets.
- Task 6: Recomposed `app/app/team/page.tsx` with 4 sections: header, team members list, invite form, pending invitations. Parallel data fetching with `Promise.all`.
- Task 7: `tsc --noEmit` — zero errors. `next build` — all routes compile successfully. Task 7.3 (manual testing) deferred to reviewer.
- No automated tests added: project has vitest configured but no established test patterns for Server Actions/Components. Story testing strategy specifies manual verification.

### Change Log

- 2026-02-16: Implement team management UI — members list, role changes, soft-deactivation, invitation revoke (Story 12.3)
- 2026-02-16: Code review fixes — secure query functions with auth context (cross-tenant fix), shared ROLE_OPTIONS, typed Role fields, error color consistency, limit on team query

### File List

**New files:**
- `components/app/team-members-list.tsx` — Client Component: team members list with role change + deactivation
- `components/app/pending-invitations-list.tsx` — Client Component: pending invitations with revoke action
- `components/ui/alert-dialog.tsx` — shadcn AlertDialog component (auto-installed)
- `drizzle/0010_sharp_lady_bullseye.sql` — Migration: ALTER TABLE users ADD COLUMN deactivated_at
- `drizzle/meta/0010_snapshot.json` — Drizzle migration snapshot (auto-generated by drizzle-kit generate)

**Modified files:**
- `lib/db/schema.ts` — Added `deactivatedAt` column to users table
- `middleware.ts` — Added `isNull(users.deactivatedAt)` check to block deactivated users
- `lib/auth/session.ts` — Added `isNull(users.deactivatedAt)` to `getAuthContext()` WHERE clause
- `lib/actions/team.ts` — Added `getTeamMembers()`, `changeUserRole()`, `deactivateUser()`, `revokeInvitation()`, updated `getPendingInvitations()` with inviter name + id
- `lib/validations/team.ts` — Added shared `ROLE_LABELS` map, `ROLE_OPTIONS` array, typed `Record<Role, string>`
- `app/app/team/page.tsx` — Recomposed with TeamMembersList + PendingInvitationsList components
- `components/app/invite-form.tsx` — Import shared `ROLE_OPTIONS`, error color fix
- `app/invite/[token]/page.tsx` — Import ROLE_LABELS from shared location
- `drizzle/meta/_journal.json` — Auto-updated by drizzle-kit generate
