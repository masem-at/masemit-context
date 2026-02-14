# Story 8.3: Project List & Detail Timeline

Status: done

<!-- Note: Validation is optional. Run validate-create-story for quality check before dev-story. -->

## Story

As a **Meister**,
I want to see all my projects with last activity and message count, and view a project's chronological timeline,
so that I can quickly find and monitor my construction sites.

## Acceptance Criteria

1. **Given** a logged-in Meister on `/app/projects`
   **When** the page loads
   **Then** all projects for the Meister's company are listed
   **And** each project shows: name, customer name, status, last activity timestamp, and count of assigned voice messages
   **And** projects are sorted by last activity (most recent first)
   **And** only projects with the Meister's companyId are shown (tenant isolation)

2. **Given** a logged-in Meister clicking on a project in the list
   **When** navigating to `/app/projects/[id]`
   **Then** the project detail page shows the project name, customer, address, description, and status
   **And** a chronological timeline of all voice messages assigned to this project is displayed
   **And** each timeline entry shows the voice message summary, timestamp, and assignment info
   **And** the timeline is empty for new projects (no assigned messages yet)
   **And** only messages with the Meister's companyId are shown

3. **Given** the project list and timeline pages
   **When** rendered
   **Then** all touch targets are >=48px (S1-NFR12)
   **And** contrast ratios meet WCAG AA (S1-NFR13)
   **And** project list loads within 2s (S1-NFR2)
   **And** project timeline loads within 3s (S1-NFR4)

**FRs:** S1-FR13, S1-FR15 | **NFRs:** S1-NFR2, S1-NFR4, S1-NFR12, S1-NFR13

## Tasks / Subtasks

- [x] Task 1: Create enhanced project list query with aggregates (AC: #1)
  - [x] 1.1 Add `getProjectsWithStats()` Server Action in `lib/actions/projects.ts` — query projects with LEFT JOIN on voice_messages to get message count + last activity (MAX createdAt)
  - [x] 1.2 Query pattern: `SELECT projects.*, COUNT(voice_messages.id) AS messageCount, MAX(voice_messages.created_at) AS lastActivity FROM projects LEFT JOIN voice_messages ON voice_messages.project_id = projects.id AND voice_messages.company_id = projects.company_id WHERE projects.company_id = ? GROUP BY projects.id ORDER BY lastActivity DESC NULLS LAST, projects.created_at DESC`
  - [x] 1.3 Use Drizzle's `sql` template for the aggregate query since Drizzle's built-in aggregate support requires raw SQL for LEFT JOIN + GROUP BY
  - [x] 1.4 Return type: `Array<Project & { messageCount: number; lastActivity: Date | null }>`
  - [x] 1.5 CRITICAL: Always filter by `companyId` from `getAuthContext()` — never trust client input

- [x] Task 2: Enhance project list page (AC: #1, #3)
  - [x] 2.1 Update `app/app/projects/page.tsx` — replace `getProjects()` call with `getProjectsWithStats()`
  - [x] 2.2 Each project card shows: name, customerName (if present), status badge (via shared STATUS_CONFIG), message count (e.g., "3 Nachrichten"), last activity relative time (e.g., "vor 2 Std")
  - [x] 2.3 Use `formatRelativeTime()` — extract from `app/app/page.tsx` into shared utility `lib/format.ts` (DRY — dashboard also uses it)
  - [x] 2.4 Sort order: last activity (most recent first), projects without messages at the bottom
  - [x] 2.5 Keep existing empty state pattern (no projects at all)
  - [x] 2.6 Project cards: `min-h-12` touch targets on the link, `p-4` padding, existing card styling
  - [x] 2.7 Message count display: `MessageSquare` icon + count, muted text, skip if 0
  - [x] 2.8 Last activity display: `Clock` icon + relative time, muted text, show "Keine Aktivität" if null

- [x] Task 3: Create project timeline query (AC: #2)
  - [x] 3.1 Add `getProjectMessages(projectId: string)` Server Action in `lib/actions/projects.ts`
  - [x] 3.2 Query: `SELECT voice_messages.* FROM voice_messages WHERE project_id = ? AND company_id = ? AND status = 'done' ORDER BY created_at DESC`
  - [x] 3.3 Return type: `VoiceMessage[]` (from schema types)
  - [x] 3.4 CRITICAL: Filter by both `projectId` AND `companyId` — double tenant isolation

- [x] Task 4: Add timeline to project detail page (AC: #2, #3)
  - [x] 4.1 Update `app/app/projects/[id]/page.tsx` — call `getProjectMessages(id)` alongside existing `getProject(id)`
  - [x] 4.2 Render timeline section below project details `<dl>` block
  - [x] 4.3 Section header: "Nachrichten" with message count (e.g., "Nachrichten (5)")
  - [x] 4.4 Each timeline entry: compact card showing summary fields (status, material, problems, nextSteps), relative timestamp, assignment info (assignedBy: "ai" → "KI-Zuordnung", "user" → "Manuell zugeordnet")
  - [x] 4.5 Reuse `SummaryFields` rendering pattern from `components/app/voice-message-card.tsx` — extract into shared component or inline (keep simple for now)
  - [x] 4.6 Empty timeline: "Noch keine Nachrichten zugeordnet." with subdued text
  - [x] 4.7 All cards: `min-h-12` touch targets, rounded-lg border, bg-card styling
  - [x] 4.8 Group messages by day: "Heute", "Gestern", or date (e.g., "Mo, 24. Feb") — reuse `groupMessagesByDay` pattern from dashboard — DEFERRED: flat list per Task 5.3 note (day groups deferred to Story 11.1)

- [x] Task 5: Extract shared formatting utilities (AC: #1, #2)
  - [x] 5.1 Create `lib/format.ts` with `formatRelativeTime(date: Date): string` — move from `app/app/page.tsx`
  - [x] 5.2 Update `app/app/page.tsx` (dashboard) to import from `lib/format.ts` instead of inline function
  - [x] 5.3 Also extract `groupMessagesByDay()` if timeline uses day grouping — OR keep it simple with flat list for Story 8.3 (day groups deferred to dashboard rebuild Story 11.1)

- [x] Task 6: Verify + test (all ACs)
  - [x] 6.1 Run `pnpm build` — zero TypeScript errors
  - [x] 6.2 Test project list: verify message counts are correct, last activity shows correct relative time
  - [x] 6.3 Test project list sorting: project with most recent voice message appears first
  - [x] 6.4 Test project detail timeline: verify messages for this project are shown, ordered newest first
  - [x] 6.5 Test tenant isolation: verify queries always include companyId filter
  - [x] 6.6 Test empty states: project with no messages shows "Keine Aktivität" and "Noch keine Nachrichten zugeordnet."
  - [x] 6.7 Test navigation: clicking project in list → detail page → timeline visible

## Dev Notes

### Critical Architecture Patterns

**1. Server Actions for Queries — same pattern as Story 8.2:**
```typescript
"use server";
import { getAuthContext } from "@/lib/auth/session";
import { db } from "@/lib/db";
import { projects, voiceMessages } from "@/lib/db/schema";

export async function getProjectsWithStats() {
  const { companyId } = await getAuthContext();
  // Aggregate query with LEFT JOIN
}
```
[Source: architecture-sprint1.md#Decision 2 — Server Actions for CRUD]

**2. companyId Filtering — THE most critical Sprint 1 rule:**
```typescript
// ✅ CORRECT — ALWAYS filter by companyId on BOTH tables
.where(and(
  eq(voiceMessages.projectId, projectId),
  eq(voiceMessages.companyId, companyId)
))

// ❌ WRONG — missing companyId = cross-tenant data leak
.where(eq(voiceMessages.projectId, projectId))
```
[Source: architecture-sprint1.md#companyId Filtering Pattern]

**3. Aggregate Query Strategy — Drizzle + raw SQL:**
Drizzle ORM v0.45.1 supports `sql` template literals for aggregates. Two approaches:

**Option A: Single query with LEFT JOIN + GROUP BY (recommended for list page):**
```typescript
import { sql, eq, desc } from "drizzle-orm";

const result = await db.execute(sql`
  SELECT
    p.*,
    COUNT(vm.id)::int AS message_count,
    MAX(vm.created_at) AS last_activity
  FROM projects p
  LEFT JOIN voice_messages vm
    ON vm.project_id = p.id
    AND vm.company_id = p.company_id
    AND vm.status = 'done'
  WHERE p.company_id = ${companyId}
  GROUP BY p.id
  ORDER BY last_activity DESC NULLS LAST, p.created_at DESC
`);
```

**Option B: Drizzle's select API (for timeline — simple WHERE query):**
```typescript
const messages = await db
  .select()
  .from(voiceMessages)
  .where(and(
    eq(voiceMessages.projectId, projectId),
    eq(voiceMessages.companyId, companyId),
    eq(voiceMessages.status, "done")
  ))
  .orderBy(desc(voiceMessages.createdAt));
```

**Choose Option A for the aggregate list query (performance — single DB round-trip), Option B for the timeline (simple, type-safe).**

**4. Index coverage — Already in place from Story 8.1:**
- `idx_voice_messages_project_id` on `projectId` — covers timeline query
- `idx_voice_messages_assignment` compound on `(companyId, projectId, status)` — covers filtered timeline + aggregate queries
- `idx_projects_company_id` on `companyId` — covers project list filter
[Source: lib/db/schema.ts — voiceMessages indexes]

### Existing Codebase — What to Create vs. Modify

| Action | File | What | Scope |
|--------|------|------|-------|
| CREATE | `lib/format.ts` | Extract `formatRelativeTime()` from dashboard, shared utility | New file |
| MODIFY | `lib/actions/projects.ts` | Add `getProjectsWithStats()` and `getProjectMessages()` Server Actions | Medium |
| MODIFY | `app/app/projects/page.tsx` | Replace placeholder list with enhanced version (message count + last activity) | Major |
| MODIFY | `app/app/projects/[id]/page.tsx` | Add voice message timeline section | Major |
| MODIFY | `app/app/page.tsx` | Import `formatRelativeTime` from `lib/format.ts` instead of inline | Small refactor |

### Existing Code Patterns to Follow

**Relative time formatting (from app/app/page.tsx:14–46):**
```typescript
function formatRelativeTime(date: Date): string {
  const now = new Date();
  const diffMs = now.getTime() - date.getTime();
  const diffMin = Math.floor(diffMs / 60000);
  const diffHrs = Math.floor(diffMs / 3600000);

  if (diffMin < 1) return "gerade eben";
  if (diffMin < 60) return `vor ${diffMin} Min`;
  if (diffHrs < 24) return `vor ${diffHrs} Std`;

  const timeStr = new Intl.DateTimeFormat("de-AT", {
    hour: "2-digit",
    minute: "2-digit",
  }).format(date);

  const today = new Date(now.getFullYear(), now.getMonth(), now.getDate());
  const yesterday = new Date(today.getTime() - 86400000);
  const msgDay = new Date(date.getFullYear(), date.getMonth(), date.getDate());

  if (msgDay.getTime() === yesterday.getTime()) return `gestern ${timeStr}`;

  const dayStr = new Intl.DateTimeFormat("de-AT", { weekday: "short" }).format(date);
  return `${dayStr} ${timeStr}`;
}
```
Move this to `lib/format.ts` and import from there. Dashboard + project pages both use it.

**Voice message summary rendering (from components/app/voice-message-card.tsx:244–289):**
```typescript
// Summary field pattern with emoji icons
<div className="flex items-start gap-2 text-body">
  <span className="shrink-0" aria-hidden="true">✅</span>
  <span>{summary.status}</span>
</div>
// Material, problems (amber bg), nextSteps, project — conditional rendering
```
For the timeline, render a simplified version: status + material (if any) + problems (if any). Skip full expandable card for now — Story 8.3 is about the timeline structure, not full card interactivity.

**VoiceSummary type (from lib/db/schema.ts:86–92):**
```typescript
export interface VoiceSummary {
  status: string;
  material: string[];
  nextSteps: string;
  problems: string | null;
  project: string;
}
```

**Server-side date formatting — CRITICAL:**
Format dates server-side (in Server Components) to avoid hydration mismatches. Pass pre-formatted strings to the rendered HTML. Do NOT format in client-side `useEffect`.
[Source: app/app/page.tsx — server-side relativeTime calculation]

### Critical Anti-Patterns to Avoid

```typescript
// ❌ WRONG: Formatting dates on client side → hydration mismatch
"use client";
const relativeTime = formatRelativeTime(new Date(message.createdAt)); // WRONG

// ✅ CORRECT: Format server-side in Server Component
// In the page.tsx Server Component:
const formattedMessages = messages.map(m => ({
  ...m,
  relativeTime: formatRelativeTime(new Date(m.createdAt)),
}));

// ❌ WRONG: N+1 query — fetching messages per project in a loop
for (const project of projects) {
  const messages = await db.select()... // N+1 queries!
}

// ✅ CORRECT: Single aggregate query with LEFT JOIN
const projectsWithStats = await db.execute(sql`
  SELECT p.*, COUNT(vm.id)::int AS message_count ...
`);

// ❌ WRONG: Missing status filter — showing "error" or "partial" messages in timeline
.where(eq(voiceMessages.projectId, projectId))

// ✅ CORRECT: Only show "done" messages (transcript + summary both complete)
.where(and(
  eq(voiceMessages.projectId, projectId),
  eq(voiceMessages.companyId, companyId),
  eq(voiceMessages.status, "done")
))

// ❌ WRONG: Creating a new client component for the timeline
// Timeline is read-only, no interactivity needed → keep as Server Component

// ❌ WRONG: Using a new migration for this story
// All tables and columns already exist from Story 8.1. NO new migrations needed.
```

### Library/Framework Specifics

| Library | Version | Key Notes for This Story |
|---------|---------|--------------------------|
| Drizzle ORM | 0.45.1 | `db.execute(sql\`...\`)` for aggregate queries with raw SQL; `sql` template for parameterized values |
| Next.js | 16.1.6 | `params` is a Promise in dynamic routes; Server Components for all data fetching |
| Lucide React | latest | `MessageSquare` for message count, `Clock` for last activity, existing icons |
| shadcn/ui | latest | Badge for status, Button for actions — already available |

**Drizzle `db.execute()` for raw SQL:**
```typescript
import { sql } from "drizzle-orm";

// Returns rows as plain objects (not typed by Drizzle schema)
const result = await db.execute(sql`
  SELECT p.id, p.name, COUNT(vm.id)::int AS message_count
  FROM projects p
  LEFT JOIN voice_messages vm ON vm.project_id = p.id
  WHERE p.company_id = ${companyId}
  GROUP BY p.id
`);
// result.rows is Array<Record<string, unknown>>
// Cast to your expected type
```

**Alternative: Drizzle's aggregate with `leftJoin`:**
```typescript
// Drizzle v0.45.1 supports:
import { count, max } from "drizzle-orm";

const result = await db
  .select({
    id: projects.id,
    name: projects.name,
    // ... all project fields
    messageCount: count(voiceMessages.id),
    lastActivity: max(voiceMessages.createdAt),
  })
  .from(projects)
  .leftJoin(voiceMessages, and(
    eq(voiceMessages.projectId, projects.id),
    eq(voiceMessages.status, "done")
  ))
  .where(eq(projects.companyId, companyId))
  .groupBy(projects.id)
  .orderBy(desc(max(voiceMessages.createdAt)));
```
This approach is type-safe but requires listing all project fields in the select. Choose based on what's cleaner.

### Project Structure Notes

**No new directories needed** — all changes within existing structure.

**New file:**
- `lib/format.ts` — Shared formatting utilities (extracted from dashboard)

**Modified files:**
- `lib/actions/projects.ts` — Add 2 new Server Actions
- `app/app/projects/page.tsx` — Enhanced list with stats
- `app/app/projects/[id]/page.tsx` — Add timeline section
- `app/app/page.tsx` — Import refactor for formatRelativeTime

**Alignment with architecture:**
- All data fetching in Server Components (Decision 6)
- Server Actions for reusable query logic (Decision 2)
- companyId filtering on every query (companyId Filtering Pattern)
- No client components needed — timeline is read-only
[Source: architecture-sprint1.md#Decision 6 — Server Components for Dashboard + Inbox]

### Previous Story Intelligence

**From Story 8.2 (Create & Edit Projects) — Code Review Fixes:**
- `STATUS_CONFIG` extracted to `lib/validations/project.ts` as shared export — reuse in list page
- `ProjectStatus` type exported — use for type-safe status access
- Zod transforms use `.nullish().transform((v) => v || null)` — not `.optional()`
- UUID validation added to route params with `UUID_RE` regex — already in `[id]/page.tsx`
- Back button touch targets corrected to `h-12 w-12` (48px)
- `<dl>` used for definition lists (semantic HTML fix)
- Error feedback added to ProjectStatusToggle

**From Story 8.1 (Multi-Tenant Data Model):**
- `getAuthContext()` returns `{ userId, companyId, role }` — use in all Server Actions
- `voiceMessages` table has `projectId`, `companyId`, `status`, `summary` columns — query for timeline
- Compound index `idx_voice_messages_assignment(companyId, projectId, status)` — covers filtered timeline queries efficiently
- Cookie name is `session` (NOT `kurzum_session`)
- `VALID_ROLES` runtime validation in getAuthContext()

**From Dashboard (app/app/page.tsx):**
- `formatRelativeTime()` server-side function for German locale timestamps
- `groupMessagesByDay()` for day-based grouping
- `DashboardMessage` interface with `relativeTime` pre-calculated field
- Server-side formatting to avoid hydration mismatches

### Git Intelligence

Recent commits:
```
9dffe00 Fix Zod validation regression: use nullish() for optional project fields
c5f8c0f Implement project CRUD with Server Actions and form components (Story 8.2)
0c721ec Implement multi-tenant data model and auth context helper (Story 8.1)
```

**This story's commit should follow:** `"Implement project list with activity stats and detail timeline (Story 8.3)"`

### References

- [Source: architecture-sprint1.md#Decision 2 — Server Actions for CRUD]
- [Source: architecture-sprint1.md#Decision 6 — Server Components for Dashboard + Inbox]
- [Source: architecture-sprint1.md#companyId Filtering Pattern]
- [Source: architecture-sprint1.md#Anti-Patterns]
- [Source: prd-sprint1.md#S1-FR13, S1-FR15]
- [Source: prd-sprint1.md#S1-NFR2, S1-NFR4, S1-NFR12, S1-NFR13]
- [Source: epics.md#Story 8.3 Acceptance Criteria]
- [Source: 8-1-multi-tenant-data-model-and-auth-context.md — schema, indexes, getAuthContext]
- [Source: 8-2-create-and-edit-projects.md — project CRUD patterns, STATUS_CONFIG, code review fixes]

## Dev Agent Record

### Agent Model Used

Claude Opus 4.6 (claude-opus-4-6)

### Debug Log References

No debug issues encountered.

### Completion Notes List

- **Task 1:** Implemented `getProjectsWithStats()` using raw SQL with `db.execute(sql\`...\`)` for single-query aggregate (LEFT JOIN + GROUP BY). Also added `getProjectMessages()` using Drizzle typed select API with triple-filter (projectId + companyId + status='done'). Exported `ProjectWithStats` interface for type safety.
- **Task 2:** Enhanced project list page to use `getProjectsWithStats()`. Each card now shows project name, customer name, status badge (via shared STATUS_CONFIG), message count with MessageSquare icon (hidden when 0), and last activity with Clock icon (shows "Keine Aktivität" if null). Added `min-h-12` touch target on card links.
- **Task 3:** `getProjectMessages()` implemented in Task 1 — uses Drizzle typed select with double tenant isolation (projectId AND companyId), filtered to status='done', ordered DESC by createdAt.
- **Task 4:** Added voice message timeline section to project detail page below the `<dl>` details block. Uses `Promise.all()` for parallel data fetching. Each timeline entry renders summary fields (status, material, problems, nextSteps) with emoji icons following voice-message-card pattern. Shows assignment info (KI-Zuordnung / Manuell zugeordnet). Empty state text when no messages. Day grouping deferred to Story 11.1 per Task 5.3 note — flat list used.
- **Task 5:** Extracted `formatRelativeTime()` from `app/app/page.tsx` into new shared utility `lib/format.ts`. Updated dashboard to import from shared module. `groupMessagesByDay()` kept in dashboard for now (deferred to Story 11.1).
- **Task 6:** `pnpm build` passes with zero TypeScript errors. All Server Components use server-side date formatting. Tenant isolation verified — all queries filter by companyId from getAuthContext().

### Change Log

- 2026-02-14: Implemented project list with activity stats and detail timeline (Story 8.3)
- 2026-02-14: Code review fixes — removed unused imports (count, max), fixed touch targets on "Neues Projekt" and "Bearbeiten" buttons to min-h-12 (48px, S1-NFR12)

### File List

- `lib/format.ts` — NEW: Shared `formatRelativeTime()` utility extracted from dashboard
- `lib/actions/projects.ts` — MODIFIED: Added `getProjectsWithStats()` (aggregate query), `getProjectMessages()` (timeline query), `ProjectWithStats` interface
- `app/app/projects/page.tsx` — MODIFIED: Enhanced list with message count, last activity, improved card layout
- `app/app/projects/[id]/page.tsx` — MODIFIED: Added voice message timeline section with summary rendering
- `app/app/page.tsx` — MODIFIED: Import `formatRelativeTime` from `lib/format.ts` instead of inline definition
- `_bmad-output/implementation-artifacts/sprint-status.yaml` — MODIFIED: Story status updated
- `_bmad-output/implementation-artifacts/8-3-project-list-and-detail-timeline.md` — MODIFIED: Story file updated
