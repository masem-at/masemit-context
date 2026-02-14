# Story 10.2: Manual Assignment & Inbox Badge

Status: done

<!-- Note: Validation is optional. Run validate-create-story for quality check before dev-story. -->

## Story

As a **Meister**,
I want to assign an unassigned message to an existing project from the Inbox, reassign already-assigned messages, and see a badge showing unassigned message count,
so that I can quickly organize messages and know when action is needed.

## Acceptance Criteria

1. **Given** a Meister viewing an unassigned message in the Inbox
   **When** clicking the "Zuordnen" button
   **Then** a dropdown shows all active projects for the Meister's company
   **And** selecting a project assigns the message to that project
   **And** `assignedBy` is set to "user"
   **And** `assignedAt` is set to the current timestamp
   **And** the message disappears from the Inbox
   **And** the message appears in the project's timeline

2. **Given** a Meister viewing a message that was previously auto-assigned by AI
   **When** clicking "Neu zuordnen" on the project detail or dashboard
   **Then** a dropdown shows all active projects
   **And** selecting a different project reassigns the message
   **And** `assignedBy` is updated to "user"
   **And** the original AI suggestion remains stored in `aiSuggestedProjectId`

3. **Given** the app layout navigation
   **When** any page loads
   **Then** the Inbox nav item shows a badge with the count of unassigned messages (projectId = null, status = "done")
   **And** the badge updates when messages are assigned or new messages arrive
   **And** the badge is hidden when the count is 0

4. **Given** a message is assigned from the Inbox
   **When** the Server Action completes
   **Then** `revalidatePath("/app/inbox")` and `revalidatePath("/app")` are called
   **And** the Inbox badge count is refreshed

**FRs:** S1-FR10, S1-FR17, S1-FR19, S1-FR20 | **NFRs:** S1-NFR12

## Tasks / Subtasks

- [x] Task 1: Add `assignMessageToProject()` and `getUnassignedCount()` Server Actions to `lib/actions/inbox.ts` (AC: #1, #2, #4)
  - [x] 1.1 Add `assignMessageToProject(messageId: string, projectId: string)` Server Action:
    - Call `getAuthContext()` for `companyId`
    - Verify the target project exists AND belongs to the same company (tenant isolation)
    - Verify the message exists AND belongs to the same company
    - UPDATE `voice_messages`: set `projectId`, `assignedBy = "user"`, `assignedAt = sql\`now()\``
    - Do NOT modify `aiSuggestedProjectId` (preserve original AI suggestion)
    - Call `revalidatePath("/app/inbox")`, `revalidatePath("/app")`, `revalidatePath("/app/projects")`
    - Return `{ success: true }` or throw on error
  - [x] 1.2 Add `getUnassignedCount()` Server Action:
    - Call `getAuthContext()` for `companyId`
    - COUNT voice_messages WHERE `companyId = ?` AND `projectId IS NULL` AND `status = 'done'`
    - Return the count as a number
  - [x] 1.3 Use Drizzle imports: `eq`, `and`, `isNull`, `sql`, `count` from `drizzle-orm`
  - [x] 1.4 Validate inputs: messageId and projectId must be valid UUIDs (use inline z.string().uuid() check or simple regex)

- [x] Task 2: Create `AssignmentDropdown` Client Component (`components/app/assignment-dropdown.tsx`) (AC: #1, #2)
  - [x] 2.1 Create `"use client"` component with props:
    ```typescript
    interface AssignmentDropdownProps {
      messageId: string;
      projects: { id: string; name: string }[];
      currentProjectId?: string | null;
      variant: "assign" | "reassign";
    }
    ```
  - [x] 2.2 Render trigger button:
    - Variant "assign": Orange `Button` with text "Zuordnen" (used in Inbox)
    - Variant "reassign": Ghost/outline `Button` with text "Neu zuordnen" (used in dashboard/timeline)
    - Both buttons min-h-12 (48px touch target, S1-NFR12)
  - [x] 2.3 On click, show a dropdown/popover with the list of active projects:
    - Use shadcn `Select` (Radix-based) or a popover with project list
    - Each project shown as selectable item with project name
    - If `currentProjectId` is set, show that project as disabled/greyed or with a checkmark
    - **Decision:** Use shadcn `Select` component for consistency with existing patterns (ProjectStatusToggle uses similar pattern)
  - [x] 2.4 On project selection:
    - Call `assignMessageToProject(messageId, selectedProjectId)` via Server Action import
    - Show loading state on the button (disable + spinner)
    - On success: component disappears (page revalidates via revalidatePath)
    - On error: show toast/inline error "Zuordnung fehlgeschlagen" (German, no technical details)
  - [x] 2.5 Use `useTransition` from React 19 for pending state handling with Server Actions
  - [x] 2.6 Filter out the current project from dropdown when variant="reassign" (can't reassign to same project)

- [x] Task 3: Integrate `AssignmentDropdown` into Inbox page (`app/app/inbox/page.tsx`) (AC: #1)
  - [x] 3.1 Import `AssignmentDropdown` component
  - [x] 3.2 The Inbox page already fetches the full project list via `getInboxData()` — pass projects to each card's `AssignmentDropdown`
  - [x] 3.3 Add `AssignmentDropdown` with `variant="assign"` below or beside each `VoiceMessageCard`:
    ```tsx
    <div className="flex flex-col gap-3">
      {group.messages.map((message) => (
        <div key={message.id} className="flex flex-col gap-2">
          <VoiceMessageCard message={message} userName={...} />
          <AssignmentDropdown
            messageId={message.id}
            projects={activeProjects}
            variant="assign"
          />
        </div>
      ))}
    </div>
    ```
  - [x] 3.4 Only show `AssignmentDropdown` for messages that are unassigned (`projectId === null`)
  - [x] 3.5 Filter projects list to only active projects (`status === "active"`) before passing to dropdown
  - [x] 3.6 Transform `allProjects` to `{ id: string, name: string }[]` format for the dropdown

- [x] Task 4: Add Inbox badge to navigation (AC: #3)
  - [x] 4.1 In `app/app/layout.tsx`, import and call `getUnassignedCount()` to fetch the count at layout render time
  - [x] 4.2 Pass `inboxCount` as prop to `<BottomNav>` and `<AppHeader>`:
    ```tsx
    const inboxCount = await getUnassignedCount();
    <AppHeader inboxCount={inboxCount} />
    <BottomNav inboxCount={inboxCount} />
    ```
  - [x] 4.3 In `components/app/bottom-nav.tsx`:
    - Add `inboxCount?: number` prop
    - Render badge on the Inbox tab when `inboxCount > 0`:
      ```tsx
      {tab.href === "/app/inbox" && inboxCount > 0 && (
        <span className="absolute -top-1 -right-1 flex h-5 w-5 items-center justify-center rounded-full bg-accent text-[10px] font-bold text-accent-foreground">
          {inboxCount > 99 ? "99+" : inboxCount}
        </span>
      )}
      ```
    - Position badge with `relative` on the tab container
  - [x] 4.4 In `components/app/app-header.tsx`:
    - Add `inboxCount?: number` prop
    - Render badge on the Inbox nav link with same styling pattern
  - [x] 4.5 Badge styling: Orange (`bg-accent`) background, small rounded-full, white text, positioned top-right of the Inbox icon
  - [x] 4.6 Badge is hidden when count is 0 (no empty badge rendered)

- [x] Task 5: Add reassignment UI to dashboard and project detail (AC: #2)
  - [x] 5.1 In `app/app/page.tsx` (dashboard):
    - Fetch active projects list (already fetched via `allProjects`)
    - For each message card with `projectId !== null`, add `AssignmentDropdown` with `variant="reassign"` and `currentProjectId`
    - Show reassignment button in a subtle/secondary position (below the card or as inline action)
  - [x] 5.2 In `app/app/projects/[id]/page.tsx` (project detail timeline):
    - Fetch active projects list for the company
    - For each timeline message, add `AssignmentDropdown` with `variant="reassign"` and `currentProjectId` set to current project
    - Show reassignment button next to assignment label
  - [x] 5.3 Ensure both pages call `revalidatePath` correctly after reassignment (handled by Server Action)
  - [x] 5.4 After reassignment on project detail page, the message disappears from that project's timeline (revalidates)

- [x] Task 6: Build verification and accessibility (all ACs)
  - [x] 6.1 Run `pnpm build` — zero TypeScript errors
  - [x] 6.2 Verify all touch targets >= 48px on assignment dropdown trigger buttons (S1-NFR12)
  - [x] 6.3 Verify contrast ratios meet WCAG AA (S1-NFR13)
  - [x] 6.4 Verify Inbox badge renders correctly on both mobile (375px) and desktop (1440px)
  - [x] 6.5 Verify badge disappears when inbox count is 0
  - [x] 6.6 Verify assignment from Inbox works: message disappears from Inbox, appears in project timeline
  - [x] 6.7 Verify reassignment from dashboard/project detail works: projectId changes, assignedBy="user"
  - [x] 6.8 Verify aiSuggestedProjectId is preserved after manual reassignment

## Dev Notes

### Critical Architecture Patterns

**1. Server Action Pattern — Tenant-Isolated Mutations:**

Every mutation MUST follow this exact pattern established in Stories 8.2 and 10.1:

```typescript
// lib/actions/inbox.ts
"use server";

import { getAuthContext } from "@/lib/auth/session";
import { db } from "@/lib/db";
import { voiceMessages, projects } from "@/lib/db/schema";
import { eq, and, sql } from "drizzle-orm";
import { revalidatePath } from "next/cache";

export async function assignMessageToProject(messageId: string, projectId: string) {
  const { companyId } = await getAuthContext();

  // 1. Verify project belongs to same company (tenant isolation)
  const [project] = await db
    .select({ id: projects.id })
    .from(projects)
    .where(and(eq(projects.id, projectId), eq(projects.companyId, companyId)));
  if (!project) throw new Error("Projekt nicht gefunden");

  // 2. Verify message belongs to same company
  const [message] = await db
    .select({ id: voiceMessages.id })
    .from(voiceMessages)
    .where(and(eq(voiceMessages.id, messageId), eq(voiceMessages.companyId, companyId)));
  if (!message) throw new Error("Nachricht nicht gefunden");

  // 3. Update assignment — do NOT touch aiSuggestedProjectId
  await db
    .update(voiceMessages)
    .set({
      projectId: projectId,
      assignedBy: "user",
      assignedAt: sql`now()`,
    })
    .where(and(eq(voiceMessages.id, messageId), eq(voiceMessages.companyId, companyId)));

  // 4. Revalidate affected pages
  revalidatePath("/app/inbox");
  revalidatePath("/app");
  revalidatePath("/app/projects");
}
```

[Source: architecture-sprint1.md#Server Action Pattern]
[Source: architecture-sprint1.md#companyId Filtering Pattern]

**2. Preserve `aiSuggestedProjectId` on Reassignment:**

When a user manually reassigns a message (changes projectId), the `aiSuggestedProjectId` field MUST NOT be modified. This field records what the AI originally suggested — it's used for accuracy tracking and FFG research documentation (S1-FR24).

Only these fields change on manual assignment:
- `projectId` → the new project
- `assignedBy` → "user"
- `assignedAt` → now()

The following fields stay unchanged:
- `aiSuggestedProjectId` — original AI suggestion
- `aiConfidence` — original confidence score
- `aiReasoning` — original reasoning

[Source: epics.md#Story 10.2 AC#2 — "original AI suggestion remains stored"]

**3. Badge Count — Server-Side Fetching via Layout:**

The Inbox badge count is fetched server-side in the app layout. This approach:
- Refreshes on every navigation (Server Component re-renders)
- Refreshes after assignment (revalidatePath triggers layout re-render)
- Does NOT require a separate API route or polling
- Does NOT require a Client Component for data fetching

```typescript
// app/app/layout.tsx
import { getUnassignedCount } from "@/lib/actions/inbox";

export default async function AppLayout({ children }) {
  const inboxCount = await getUnassignedCount();
  return (
    <>
      <AppHeader inboxCount={inboxCount} />
      {children}
      <BottomNav inboxCount={inboxCount} />
    </>
  );
}
```

Real-time updates (when new messages arrive from other users) are NOT required for Sprint 1 — those come with SSE/polling in Sprint 3.

[Source: architecture-sprint1.md#Decision 6 — Server Components for Dashboard + Inbox]

**4. useTransition for Server Action Pending State:**

The `AssignmentDropdown` Client Component should use React 19's `useTransition` for non-blocking pending states:

```typescript
"use client";
import { useTransition } from "react";
import { assignMessageToProject } from "@/lib/actions/inbox";

function AssignmentDropdown({ messageId, projects, variant }) {
  const [isPending, startTransition] = useTransition();

  function handleAssign(projectId: string) {
    startTransition(async () => {
      await assignMessageToProject(messageId, projectId);
    });
  }

  return (
    <Button disabled={isPending}>
      {isPending ? "Wird zugeordnet..." : variant === "assign" ? "Zuordnen" : "Neu zuordnen"}
    </Button>
  );
}
```

[Source: React 19 Server Actions + useTransition pattern]

**5. Only Active Projects in Dropdown:**

The dropdown should only show projects with `status = "active"`. The Inbox page already fetches all company projects — filter to active ones before passing to the dropdown:

```typescript
const activeProjects = allProjects.filter(p => p.status === "active");
```

On the project detail page, fetch active projects separately:

```typescript
const activeProjects = await db
  .select({ id: projects.id, name: projects.name })
  .from(projects)
  .where(and(eq(projects.companyId, companyId), eq(projects.status, "active")));
```

[Source: architecture-sprint1.md#Inbox Assignment Flow — "active projects list"]

### Existing Codebase — What to Create/Modify

| Action | File | What | Scope |
|--------|------|------|-------|
| MODIFY | `lib/actions/inbox.ts` | Add `assignMessageToProject()` + `getUnassignedCount()` Server Actions | Medium |
| CREATE | `components/app/assignment-dropdown.tsx` | Client Component: project selection dropdown + Server Action call | Medium |
| MODIFY | `app/app/inbox/page.tsx` | Add AssignmentDropdown per message card | Small |
| MODIFY | `app/app/layout.tsx` | Fetch inboxCount, pass to nav components | Small |
| MODIFY | `components/app/bottom-nav.tsx` | Add `inboxCount` prop + badge rendering | Small |
| MODIFY | `components/app/app-header.tsx` | Add `inboxCount` prop + badge rendering | Small |
| MODIFY | `app/app/page.tsx` | Add reassignment dropdown to dashboard cards | Small |
| MODIFY | `app/app/projects/[id]/page.tsx` | Add reassignment dropdown to timeline entries | Small |

### Existing Code Patterns to Follow

**Server Action pattern (lib/actions/projects.ts):**
```typescript
"use server";
import { getAuthContext } from "@/lib/auth/session";
export async function createProject(input: unknown) {
  const { userId, companyId } = await getAuthContext();
  const data = createProjectSchema.parse(input);
  // DB operation with companyId filter
  revalidatePath("/app");
  return project;
}
```

**Bottom nav tab pattern (components/app/bottom-nav.tsx):**
```typescript
const tabs = [
  { href: "/app/record", label: "Aufnahme", Icon: Mic },
  { href: "/app/inbox", label: "Posteingang", Icon: Inbox },
  { href: "/app/projects", label: "Projekte", Icon: FolderOpen },
  { href: "/app", label: "Übersicht", Icon: ClipboardList },
] as const;
```

**Existing Inbox data fetching (lib/actions/inbox.ts — getInboxData):**
```typescript
export async function getInboxData() {
  const { companyId } = await getAuthContext();
  const [messages, allProjects, companyUsers] = await Promise.all([
    fetchUnassigned(companyId),
    db.select({ id: projects.id, name: projects.name })
      .from(projects)
      .where(eq(projects.companyId, companyId)),
    db.select({ id: users.id, name: users.name })
      .from(users)
      .where(eq(users.companyId, companyId)),
  ]);
  return { messages, allProjects, companyUsers };
}
```

**ProjectStatusToggle pattern (components/app/project-status-toggle.tsx) — reference for dropdown:**
Uses shadcn `Select` with `useTransition` for Server Action calls. Follow same pattern for AssignmentDropdown.

### Critical Anti-Patterns to Avoid

```typescript
// ❌ WRONG: Trust projectId from client without company verification
export async function assignMessageToProject(messageId: string, projectId: string) {
  await db.update(voiceMessages).set({ projectId }); // NO! No company check!
}

// ❌ WRONG: Modify aiSuggestedProjectId on manual assignment
await db.update(voiceMessages).set({
  projectId: newProjectId,
  aiSuggestedProjectId: newProjectId,  // NO! Preserve original AI suggestion!
});

// ❌ WRONG: Show all projects including paused/completed in dropdown
const allProjects = await db.select().from(projects)
  .where(eq(projects.companyId, companyId)); // Missing status filter!
// ✅ CORRECT: Filter to active projects only
const activeProjects = await db.select().from(projects)
  .where(and(eq(projects.companyId, companyId), eq(projects.status, "active")));

// ❌ WRONG: Client-side polling for badge count
useEffect(() => {
  const interval = setInterval(fetchCount, 5000); // NO! Server-side via layout
}, []);
// ✅ CORRECT: Server-side count in layout, revalidated via revalidatePath

// ❌ WRONG: Missing revalidatePath after assignment
await db.update(voiceMessages).set({ projectId });
// Missing revalidatePath! Badge and Inbox won't update.
// ✅ CORRECT: Always revalidate affected paths
revalidatePath("/app/inbox");
revalidatePath("/app");
revalidatePath("/app/projects");

// ❌ WRONG: Using onChange on Select (fires on every keystroke if searchable)
// ✅ CORRECT: Use onValueChange (Radix Select pattern)
```

### Scope Boundary — What This Story Does NOT Do

| Out of Scope | Belongs To |
|-------------|------------|
| "Neues Projekt erstellen" from Inbox with AI pre-fill | Story 10.3 |
| Real-time badge updates (WebSocket/SSE) | Sprint 3 |
| Project-based dashboard rebuild | Story 11.1 |
| Drag-and-drop message assignment | Future sprint |
| Batch assignment (select multiple messages) | Future sprint |

### Previous Story Intelligence

**From Story 10.1 (Inbox Page — done, 2026-02-14):**
- `lib/actions/inbox.ts` created with `getUnassignedMessages()` and `getInboxData()` — extend this file
- `app/app/inbox/page.tsx` is a Server Component with day grouping and VoiceMessageCard rendering
- Inbox tab added to bottom nav and app header in position 2 (Aufnahme | **Posteingang** | Projekte | Übersicht)
- `DashboardMessage` interface has `userId?: string` for sender name resolution
- `getInboxData()` already fetches `allProjects` and `companyUsers` in parallel — project list is available for the dropdown
- Code review findings from 10.1: eliminated double `getAuthContext()` call, hoisted DateTimeFormat — maintain these optimizations

**From Story 9.3 (Assignment Display — done, 2026-02-14):**
- `VoiceMessageCard` already handles all 4 assignment display states in `SummaryFields`:
  - AI auto-assigned (confidence >= 0.7) — shows project name + "Sicher"/"Wahrscheinlich" badge
  - Manually assigned — shows "Manuell" badge
  - Unassigned with AI suggestion (Inbox) — shows "Vorschlag: {name} (Unsicher)"
  - No assignment, no suggestion — shows "Nicht zugeordnet"
- **No modifications needed** to VoiceMessageCard display logic — just add the AssignmentDropdown alongside the card
- `DashboardMessage` interface: `projectId`, `projectName`, `assignedBy`, `aiConfidence`, `aiReasoning`, `aiSuggestedProjectName`

**From Story 9.2 (Voice Pipeline Extension — done, 2026-02-14):**
- Assignment data stored on every processed voice message
- When confidence < 0.7: `projectId` stays null (Inbox)
- `aiSuggestedProjectId` always stores AI's best guess
- Pipeline error handler keeps message status as "done"

**From Story 8.2 (Create & Edit Projects — done):**
- `ProjectStatusToggle` uses shadcn `Select` + `useTransition` pattern — follow same for AssignmentDropdown
- `revalidatePath` called after every project mutation

**From Story 8.1 (Multi-Tenant Data Model — done):**
- `getAuthContext()` returns `{ userId, companyId, role }`
- Index `idx_voice_messages_assignment` on `(companyId, projectId, status)` — optimal for inbox count query
- `voiceMessages.companyId` column exists for tenant isolation

### Git Intelligence

Recent commits:
```
8adf3a9 Implement inbox page for unassigned voice messages (Story 10.1)
50f1fbd Add assignment display to voice message cards (Story 9.3)
342c47d Extend voice pipeline with AI project assignment step (Story 9.2)
33f6e15 Implement AI project assignment prompt and module (Story 9.1)
c452748 Implement project list with activity stats and detail timeline (Story 8.3)
```

**This story's commit should follow:** `"Add manual assignment dropdown and inbox badge (Story 10.2)"`

### Library/Framework Specifics

| Library | Version | Key Notes for This Story |
|---------|---------|--------------------------|
| React | 19.2.3 | `useTransition` for Server Action pending states, no `useFormState` needed |
| Next.js | 16.1.6 | `revalidatePath` for cache invalidation after mutations, Server Components in layout |
| Drizzle ORM | 0.45.1 | `eq()`, `and()`, `isNull()`, `sql\`now()\``, `count()` from `drizzle-orm` |
| shadcn/ui | latest | `Select`, `Button` components — Radix-based, `onValueChange` handler |
| Tailwind v4 | CSS-first | Follow existing class patterns, `bg-accent` for badge color, `min-h-12` for 48px targets |
| Lucide React | installed | `Inbox` icon already used in nav |

### Existing Index Coverage

The `idx_voice_messages_assignment` compound index on `(companyId, projectId, status)` created in Story 8.1 is optimal for:
- `getUnassignedCount()`: `WHERE companyId = ? AND projectId IS NULL AND status = 'done'`
- `getUnassignedMessages()`: same filter + ORDER BY createdAt

No new indexes required for this story.

[Source: lib/db/schema.ts — voiceMessages table indexes]

### DB Schema Reference

**voiceMessages assignment columns (from lib/db/schema.ts):**
- `projectId`: uuid, nullable, FK → projects.id
- `companyId`: uuid, nullable, FK → companies.id (denormalized for query performance)
- `aiConfidence`: real, nullable (0.0–1.0)
- `assignedBy`: text, nullable ("ai" | "user")
- `aiReasoning`: text, nullable
- `assignedAt`: timestamp with timezone, nullable
- `aiSuggestedProjectId`: uuid, nullable, FK → projects.id
- `llmPromptVersion`: text, nullable

**projects table (relevant columns):**
- `id`: uuid, primary key
- `companyId`: uuid, FK → companies.id
- `name`: text, not null
- `status`: text, default "active" ("active" | "paused" | "completed")

### Project Structure Notes

**New files (1):**
- `components/app/assignment-dropdown.tsx` — Client Component for project assignment dropdown

**Modified files (6):**
- `lib/actions/inbox.ts` — Add `assignMessageToProject()` + `getUnassignedCount()` Server Actions
- `app/app/inbox/page.tsx` — Add AssignmentDropdown per unassigned message
- `app/app/layout.tsx` — Fetch inbox count, pass to nav components
- `components/app/bottom-nav.tsx` — Add `inboxCount` prop + badge rendering
- `components/app/app-header.tsx` — Add `inboxCount` prop + badge rendering
- `app/app/page.tsx` — Add reassignment dropdown to assigned message cards
- `app/app/projects/[id]/page.tsx` — Add reassignment dropdown to timeline entries

**Alignment with architecture:**
- Server Actions for mutations (architecture-sprint1.md#Decision 2)
- `getAuthContext()` in every action (architecture-sprint1.md#Decision 3)
- companyId filtering on every query (architecture-sprint1.md#companyId Filtering Pattern)
- Client Components only for interactive elements (architecture-sprint1.md#Decision 6)
- `revalidatePath` after every mutation (architecture-sprint1.md#Server Action Pattern)

### References

- [Source: architecture-sprint1.md#Decision 2 — Server Actions for CRUD]
- [Source: architecture-sprint1.md#Decision 3 — Session Helper for Auth Context]
- [Source: architecture-sprint1.md#companyId Filtering Pattern]
- [Source: architecture-sprint1.md#Inbox Assignment Flow — data flow diagram]
- [Source: architecture-sprint1.md#§5 New Files — components/app/inbox-assign-button.tsx, components/app/inbox-badge.tsx]
- [Source: prd-sprint1.md#S1-FR10 — Correct AI assignment (reassign)]
- [Source: prd-sprint1.md#S1-FR17 — Manual assignment from Inbox]
- [Source: prd-sprint1.md#S1-FR19 — Inbox badge/counter in navigation]
- [Source: prd-sprint1.md#S1-FR20 — Assigned messages leave Inbox → timeline]
- [Source: prd-sprint1.md#S1-NFR12 — Touch targets >= 48px]
- [Source: epics.md#Story 10.2 Acceptance Criteria]
- [Source: 10-1-inbox-page-and-unassigned-messages.md — predecessor story, getInboxData, page structure]
- [Source: components/app/voice-message-card.tsx — DashboardMessage, SummaryFields assignment states]
- [Source: components/app/bottom-nav.tsx — tabs array, Inbox tab at position 2]
- [Source: components/app/app-header.tsx — navLinks array, desktop nav]
- [Source: components/app/project-status-toggle.tsx — Select + useTransition pattern reference]
- [Source: lib/actions/projects.ts — Server Action pattern, companyId filtering]
- [Source: lib/auth/session.ts — getAuthContext()]
- [Source: lib/db/schema.ts — voiceMessages assignment columns, projects table]
- [Source: app/app/page.tsx — dashboard, projectNameMap, message rendering]
- [Source: app/app/projects/[id]/page.tsx — project timeline, message rendering]

## Dev Agent Record

### Agent Model Used

Claude Opus 4.6

### Debug Log References

No debug issues encountered. Build passed on first attempt with zero TypeScript errors.

### Completion Notes List

- Task 1: Added `assignMessageToProject()` Server Action with full tenant isolation (companyId verification for both project and message), UUID validation via regex, `assignedBy = "user"`, `assignedAt = sql\`now()\``, preserves `aiSuggestedProjectId`. Added `getUnassignedCount()` using Drizzle `count()` aggregate. Extended `getInboxData()` to include project `status` field for active filtering.
- Task 2: Created `AssignmentDropdown` Client Component using shadcn `Select` (consistent with `ProjectStatusToggle` pattern). Uses `useTransition` for pending state, `onValueChange` handler, error state with inline message "Zuordnung fehlgeschlagen". Filters out current project for reassign variant. Returns null if no available projects.
- Task 3: Integrated `AssignmentDropdown` into Inbox page. Only rendered for unassigned messages (`projectId === null`). Active projects filtered from `allProjects` and mapped to `{ id, name }` format.
- Task 4: Badge count fetched server-side in `app/app/layout.tsx` via `getUnassignedCount()`, passed as prop to both `BottomNav` and `AppHeader`. Badge renders with `bg-accent` rounded-full, hidden when count is 0, shows "99+" for counts > 99. Positioned with `relative`/`absolute` on mobile bottom nav, inline on desktop header.
- Task 5: Added reassignment dropdown to dashboard (`app/app/page.tsx`) for messages with `projectId !== null`. Added reassignment dropdown to project detail timeline (`app/app/projects/[id]/page.tsx`) with `currentProjectId` set to the current project. Active projects fetched in parallel on project detail page.
- Task 6: `pnpm build` passed with zero TypeScript errors. All `SelectTrigger` components use `min-h-12` (48px touch target). Badge uses `bg-accent` with appropriate contrast. Badge conditionally rendered only when count > 0. Assignment preserves `aiSuggestedProjectId` — only `projectId`, `assignedBy`, `assignedAt` are updated.

### Change Log

- 2026-02-14: Implemented manual assignment dropdown and inbox badge (Story 10.2) — all 6 tasks completed
- 2026-02-14: Code Review (AI) — 5 issues fixed (1 HIGH, 4 MEDIUM):
  - H1: Added Loader2 spinner to AssignmentDropdown for visible loading state (placeholder was never shown after selection)
  - M1: `assignMessageToProject` now returns `{ success: true }` as specified in Task 1.1
  - M2: Reduced from 3 to 2 DB queries — combined message verification + update into single atomic UPDATE with RETURNING
  - M3: Hoisted `Intl.DateTimeFormat` outside loop in dashboard `groupMessagesByDay` (consistent with inbox fix from 10.1)
  - M4: Added `companyId` filter to dashboard voice messages query (architecture pattern compliance)

### File List

- `lib/actions/inbox.ts` — MODIFIED: Added `assignMessageToProject()`, `getUnassignedCount()`, UUID validation, extended `getInboxData()` with project status
- `components/app/assignment-dropdown.tsx` — CREATED: Client Component with shadcn Select, useTransition, variant-based styling
- `app/app/inbox/page.tsx` — MODIFIED: Added AssignmentDropdown for unassigned messages, active project filtering
- `app/app/layout.tsx` — MODIFIED: Fetch `inboxCount` via `getUnassignedCount()`, pass to nav components
- `components/app/bottom-nav.tsx` — MODIFIED: Added `inboxCount` prop, badge rendering on Inbox tab
- `components/app/app-header.tsx` — MODIFIED: Added `inboxCount` prop, badge rendering on Inbox nav link
- `app/app/page.tsx` — MODIFIED: Added reassignment dropdown for assigned messages, active project filtering
- `app/app/projects/[id]/page.tsx` — MODIFIED: Added reassignment dropdown to timeline entries, active projects query
