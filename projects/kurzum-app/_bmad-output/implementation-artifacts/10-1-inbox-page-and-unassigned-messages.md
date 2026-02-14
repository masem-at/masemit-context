# Story 10.1: Inbox Page & Unassigned Messages

Status: done

<!-- Note: Validation is optional. Run validate-create-story for quality check before dev-story. -->

## Story

As a **Meister**,
I want to access an Inbox showing all voice messages without project assignment,
so that I can review messages the AI was unsure about and assign them manually.

## Acceptance Criteria

1. **Given** a logged-in Meister navigating to `/app/inbox`
   **When** the page loads
   **Then** all voice messages with `projectId = null` and `status = "done"` for the Meister's company are listed
   **And** each message shows: summary, timestamp, and AI suggestion (project name from `aiSuggestedProjectId` if available)
   **And** messages are sorted by creation date (newest first)
   **And** only messages with the Meister's companyId are shown (tenant isolation)
   **And** the page loads within 2 seconds (S1-NFR3)

2. **Given** no unassigned messages exist
   **When** the Inbox page loads
   **Then** an empty state is displayed (e.g., "Keine unzugeordneten Nachrichten")

3. **Given** the Inbox page
   **When** rendered
   **Then** all touch targets are >=48px (S1-NFR12)
   **And** contrast ratios meet WCAG AA

4. **Given** the app navigation
   **When** the Inbox tab is added
   **Then** the bottom navigation (mobile) and header navigation (desktop) include an Inbox link at `/app/inbox`

**FRs:** S1-FR16 | **NFRs:** S1-NFR3, S1-NFR12

## Tasks / Subtasks

- [x] Task 1: Create `lib/actions/inbox.ts` with `getUnassignedMessages()` (AC: #1)
  - [x] 1.1 Create `lib/actions/inbox.ts` with `"use server"` directive
  - [x] 1.2 Implement `getUnassignedMessages()`:
    ```typescript
    export async function getUnassignedMessages() {
      const { companyId } = await getAuthContext();
      return db
        .select()
        .from(voiceMessages)
        .where(
          and(
            eq(voiceMessages.companyId, companyId),
            isNull(voiceMessages.projectId),
            eq(voiceMessages.status, "done")
          )
        )
        .orderBy(desc(voiceMessages.createdAt));
    }
    ```
  - [x] 1.3 Import `and`, `eq`, `isNull`, `desc` from `drizzle-orm`
  - [x] 1.4 Import `getAuthContext` from `@/lib/auth/session`
  - [x] 1.5 Import `db` from `@/lib/db` and `voiceMessages` from `@/lib/db/schema`
  - [x] 1.6 Note: query is company-scoped (NOT user-scoped) — Meister sees ALL team members' unassigned messages

- [x] Task 2: Create `app/app/inbox/page.tsx` as Server Component (AC: #1, #2, #3)
  - [x] 2.1 Create `app/app/inbox/` directory and `page.tsx`
  - [x] 2.2 Import `getUnassignedMessages` from `@/lib/actions/inbox`
  - [x] 2.3 Import `getAuthContext` from `@/lib/auth/session`
  - [x] 2.4 Import `db`, `projects`, `users` from schema for parallel queries
  - [x] 2.5 Fetch unassigned messages, all company projects (for name map), and all company users (for sender name resolution) in parallel:
    ```typescript
    const { companyId } = await getAuthContext();
    const [messages, allProjects, companyUsers] = await Promise.all([
      getUnassignedMessages(),
      db.select({ id: projects.id, name: projects.name })
        .from(projects)
        .where(eq(projects.companyId, companyId)),
      db.select({ id: users.id, name: users.name })
        .from(users)
        .where(eq(users.companyId, companyId)),
    ]);
    ```
  - [x] 2.6 Build `projectNameMap: Map<string, string>` and `userNameMap: Map<string, string>` from query results
  - [x] 2.7 Transform messages to `DashboardMessage[]` with `formatRelativeTime` for each message's `relativeTime`, and resolve `projectName`, `aiSuggestedProjectName` via `projectNameMap`
  - [x] 2.8 Group messages by day using the same `groupMessagesByDay` pattern from `app/app/page.tsx`:
    - Extract the `groupMessagesByDay` function or refactor into a shared utility? **Decision: Inline the grouping in the inbox page** — keep it simple, avoid premature abstraction. Story 11.1 (dashboard rebuild) may change the dashboard grouping, so a shared utility would be premature.
  - [x] 2.9 Render grouped messages using `VoiceMessageCard` with per-message `userName` from `userNameMap`:
    ```tsx
    {messageGroups.map((group) => (
      <section key={group.label}>
        <h2 className="text-body-sm font-semibold text-muted-foreground uppercase tracking-wider mb-3">
          {group.label}
        </h2>
        <div className="flex flex-col gap-3">
          {group.messages.map((message) => (
            <VoiceMessageCard
              key={message.id}
              message={message}
              userName={userNameMap.get(message.userId) ?? "Benutzer"}
            />
          ))}
        </div>
      </section>
    ))}
    ```
  - [x] 2.10 **Important:** `DashboardMessage` does NOT have a `userId` field — we need to preserve `userId` from the DB row for sender name resolution. Two options:
    - Option A: Add `userId` to `DashboardMessage` interface (simple, non-breaking)
    - Option B: Build a separate `messageUserMap: Map<string, string>` keyed by message ID
    - **Decision: Option A** — add `userId` to `DashboardMessage` as optional field. It's already in the DB row and is a natural extension. Update `VoiceMessageCard` interface but ignore the field in rendering.
  - [x] 2.11 Add page heading "Posteingang" with optional message count badge

- [x] Task 3: Implement empty state for Inbox (AC: #2)
  - [x] 3.1 When `messages.length === 0`, render empty state:
    ```tsx
    <div className="flex flex-col items-center justify-center py-16 text-center">
      <div className="flex h-16 w-16 items-center justify-center rounded-full bg-muted mb-4">
        <Inbox className="h-8 w-8 text-muted-foreground" />
      </div>
      <h1 className="text-h3 font-semibold mb-2">
        Keine unzugeordneten Nachrichten
      </h1>
      <p className="text-body text-muted-foreground max-w-sm">
        Alle Nachrichten wurden einem Projekt zugeordnet.
      </p>
    </div>
    ```
  - [x] 3.2 Use `Inbox` icon from `lucide-react` for empty state and page identity

- [x] Task 4: Add Inbox tab to bottom navigation (AC: #4)
  - [x] 4.1 In `components/app/bottom-nav.tsx`, import `Inbox` from `lucide-react`
  - [x] 4.2 Add new tab entry to the `tabs` array:
    ```typescript
    { href: "/app/inbox", label: "Posteingang", Icon: Inbox },
    ```
  - [x] 4.3 Position: Insert between "Projekte" and "Übersicht" (or after "Übersicht"):
    - **Decision:** Place Inbox as 2nd tab: Aufnahme | Posteingang | Projekte | Übersicht
    - Rationale: Natural workflow — record → inbox (review) → projects → overview
  - [x] 4.4 Verify `isActive` logic handles `/app/inbox` correctly (uses `pathname.startsWith(href)`)
  - [x] 4.5 Verify all 4 tabs fit on 375px viewport without text truncation — if labels are too wide, consider shorter label "Inbox" or icon-only with aria-label

- [x] Task 5: Add Inbox link to desktop header navigation (AC: #4)
  - [x] 5.1 In `components/app/app-header.tsx`, import `Inbox` from `lucide-react`
  - [x] 5.2 Add new nav link entry to the `navLinks` array in the same order as bottom nav:
    ```typescript
    { href: "/app/inbox", label: "Posteingang", Icon: Inbox },
    ```
  - [x] 5.3 Verify desktop nav handles 4 items without layout issues

- [x] Task 6: Extend `DashboardMessage` with `userId` field (AC: #1)
  - [x] 6.1 In `components/app/voice-message-card.tsx`, add `userId?: string` to `DashboardMessage` interface
  - [x] 6.2 This is an optional field — dashboard page does NOT need to pass it (all messages are same user)
  - [x] 6.3 Inbox page passes `userId` for sender name resolution in the page component (NOT in VoiceMessageCard)

- [x] Task 7: Verify build and accessibility (all ACs)
  - [x] 7.1 Run `pnpm build` — zero TypeScript errors
  - [x] 7.2 Verify all touch targets are >= 48px on new nav items (S1-NFR12)
  - [x] 7.3 Verify contrast ratios meet WCAG AA (S1-NFR13)
  - [x] 7.4 Verify Inbox page works on mobile (375px) and desktop (1440px)
  - [x] 7.5 Verify empty state renders correctly
  - [x] 7.6 Verify 4-tab bottom nav fits on 375px viewport

## Dev Notes

### Critical Architecture Patterns

**1. Company-Scoped Query — NOT User-Scoped:**

The Inbox is fundamentally different from the Dashboard:
- **Dashboard:** Shows `voiceMessages WHERE userId = ?` (one user's own messages)
- **Inbox:** Shows `voiceMessages WHERE companyId = ? AND projectId IS NULL AND status = 'done'` (all team members' unassigned messages)

This means the Inbox can show messages from ANY team member in the company. The Meister reviews all their team's unassigned work. This requires resolving sender names (userName) per message, unlike the dashboard which knows all messages belong to the logged-in user.

```typescript
// ✅ CORRECT: Company-scoped inbox query
const messages = await db.select().from(voiceMessages)
  .where(and(
    eq(voiceMessages.companyId, companyId),  // Tenant isolation
    isNull(voiceMessages.projectId),          // Unassigned only
    eq(voiceMessages.status, "done")          // Fully processed only
  ))
  .orderBy(desc(voiceMessages.createdAt));

// ❌ WRONG: User-scoped (misses other team members' messages)
const messages = await db.select().from(voiceMessages)
  .where(and(
    eq(voiceMessages.userId, userId),         // NO! This is dashboard behavior
    isNull(voiceMessages.projectId),
    eq(voiceMessages.status, "done")
  ));
```

[Source: architecture-sprint1.md#Inbox Query Pattern]
[Source: architecture-sprint1.md#companyId Filtering Pattern]

**2. Sender Name Resolution — userNameMap Pattern:**

Since Inbox messages come from different users, we need to resolve sender names. Follow the same map-based lookup pattern used for project names in Story 9.3:

```typescript
const companyUsers = await db
  .select({ id: users.id, name: users.name })
  .from(users)
  .where(eq(users.companyId, companyId));

const userNameMap = new Map(companyUsers.map(u => [u.id, u.name]));

// Per-card rendering:
<VoiceMessageCard
  message={msg}
  userName={userNameMap.get(msg.userId) ?? "Benutzer"}
/>
```

[Source: 9-3-assignment-display-on-voice-message-cards.md — projectNameMap pattern]

**3. Reuse VoiceMessageCard Directly:**

Story 9.3 already implemented all 4 assignment display states in `SummaryFields`:
- **State 1:** AI auto-assigned (confidence >= 0.7) — shows project name + confidence badge
- **State 2:** Manually assigned — shows "Manuell" badge
- **State 3:** Unassigned with AI suggestion (Inbox) — shows "Vorschlag: {name} (Unsicher)"
- **State 4:** No assignment, no suggestion — shows "Nicht zugeordnet"

For Inbox messages, State 3 and State 4 are the relevant ones. The component handles them natively — no modifications needed to the card rendering.

[Source: components/app/voice-message-card.tsx — SummaryFields]

**4. Only `status = "done"` in Inbox:**

Messages with `status = "error"` or `status = "partial"` are NOT shown in the Inbox. The Inbox is for fully processed messages that simply lack project assignment. Error/partial messages remain visible only on the Dashboard.

[Source: architecture-sprint1.md#Inbox Query Pattern — "Only status='done'"]

**5. No Assignment Actions in This Story:**

Story 10.1 is display-only. No "Zuordnen" button, no project dropdown, no reassignment. Those come in Story 10.2. The Inbox page in 10.1 is read-only.

### Existing Codebase — What to Create/Modify

| Action | File | What | Scope |
|--------|------|------|-------|
| CREATE | `lib/actions/inbox.ts` | Server Action: `getUnassignedMessages()` | Small |
| CREATE | `app/app/inbox/page.tsx` | Server Component: Inbox page with message list + empty state | Medium |
| MODIFY | `components/app/bottom-nav.tsx` | Add Inbox tab (4th tab) | Small |
| MODIFY | `components/app/app-header.tsx` | Add Inbox link to desktop nav | Small |
| MODIFY | `components/app/voice-message-card.tsx` | Add optional `userId` to `DashboardMessage` interface | Minimal |

### Existing Code Patterns to Follow

**Dashboard page query pattern (app/app/page.tsx):**
```typescript
const { userId, companyId } = await getAuthContext();
const [allProjects, messages, user] = await Promise.all([
  db.select({ id: projects.id, name: projects.name })
    .from(projects)
    .where(eq(projects.companyId, companyId)),
  db.select().from(voiceMessages)
    .where(eq(voiceMessages.userId, userId))
    .orderBy(desc(voiceMessages.createdAt)),
  db.query.users.findFirst({
    where: eq(users.id, userId),
    columns: { name: true },
  }),
]);
const projectNameMap = new Map(allProjects.map(p => [p.id, p.name]));
```

**Inbox page follows the same parallel-query + map-lookup pattern, but with company-scoped messages + user name resolution.**

**Bottom nav tab pattern (components/app/bottom-nav.tsx:8-12):**
```typescript
const tabs = [
  { href: "/app/record", label: "Aufnahme", Icon: Mic },
  { href: "/app/projects", label: "Projekte", Icon: FolderOpen },
  { href: "/app", label: "Übersicht", Icon: ClipboardList },
] as const;
```

**Add Inbox entry in the same pattern format.**

**App header desktop nav (components/app/app-header.tsx:9-13):**
```typescript
const navLinks = [
  { href: "/app/record", label: "Aufnahme", Icon: Mic },
  { href: "/app/projects", label: "Projekte", Icon: FolderOpen },
  { href: "/app", label: "Übersicht", Icon: ClipboardList },
] as const;
```

**Same array — add Inbox entry in same position as bottom nav.**

**Empty state pattern (app/app/page.tsx:68-83):**
```typescript
if (messages.length === 0) {
  return (
    <div className="flex flex-col items-center justify-center py-16 text-center">
      <div className="flex h-16 w-16 items-center justify-center rounded-full bg-muted mb-4">
        <Mic className="h-8 w-8 text-muted-foreground" />
      </div>
      <h1 className="text-h3 font-semibold mb-2">
        Noch keine Nachrichten.
      </h1>
      <p className="text-body text-muted-foreground max-w-sm mb-6">
        Sprich deine erste Nachricht und schau was die KI daraus macht.
      </p>
    </div>
  );
}
```

**Follow this exact empty state visual pattern — different icon (Inbox), different text.**

### Critical Anti-Patterns to Avoid

```typescript
// ❌ WRONG: User-scoped inbox query (misses team messages)
.where(eq(voiceMessages.userId, userId))
// ✅ CORRECT: Company-scoped
.where(eq(voiceMessages.companyId, companyId))

// ❌ WRONG: Including non-done messages in Inbox
.where(and(
  isNull(voiceMessages.projectId),
  eq(voiceMessages.companyId, companyId)
  // Missing status filter!
))
// ✅ CORRECT: Only fully processed messages
.where(and(
  eq(voiceMessages.companyId, companyId),
  isNull(voiceMessages.projectId),
  eq(voiceMessages.status, "done")
))

// ❌ WRONG: Adding "Zuordnen" button in Story 10.1
// Assignment actions belong to Story 10.2
// ✅ CORRECT: Display-only inbox page

// ❌ WRONG: Same userName for all cards (like dashboard)
<VoiceMessageCard message={msg} userName={currentUserName} />
// ✅ CORRECT: Per-message sender name resolution
<VoiceMessageCard message={msg} userName={userNameMap.get(msg.userId) ?? "Benutzer"} />

// ❌ WRONG: Missing companyId filter on project/user lookups
db.select().from(users)  // Data leak!
// ✅ CORRECT: Always filter by companyId
db.select().from(users).where(eq(users.companyId, companyId))

// ❌ WRONG: Creating a shared utility for groupMessagesByDay prematurely
// Story 11.1 (dashboard rebuild) may change grouping logic
// ✅ CORRECT: Inline the grouping in inbox page for now
```

### Scope Boundary — What This Story Does NOT Do

| Out of Scope | Belongs To |
|-------------|------------|
| "Zuordnen" button to assign message to project | Story 10.2 |
| Inbox badge/counter in navigation | Story 10.2 |
| "Neu zuordnen" for already-assigned messages | Story 10.2 |
| "Neues Projekt erstellen" from Inbox | Story 10.3 |
| Project-based dashboard rebuild | Story 11.1 |

**What this story DOES prepare for Story 10.2:**
- The `lib/actions/inbox.ts` file is created — Story 10.2 adds `assignToProject()` and `getUnassignedCount()` Server Actions to it
- The Inbox page is navigable — Story 10.2 adds the assignment dropdown per message
- The navigation already has the Inbox tab — Story 10.2 adds the badge with unassigned count

### Project Structure Notes

**New files (2):**
- `lib/actions/inbox.ts` — Server Action for fetching unassigned messages
- `app/app/inbox/page.tsx` — Server Component inbox page

**Modified files (3):**
- `components/app/bottom-nav.tsx` — Add 4th tab (Inbox)
- `components/app/app-header.tsx` — Add Inbox to desktop nav
- `components/app/voice-message-card.tsx` — Add optional `userId` to `DashboardMessage`

**Alignment with architecture:**
- `app/app/inbox/page.tsx` → Server Component (matches architecture-sprint1.md#Decision 6)
- `lib/actions/inbox.ts` → Server Actions pattern (matches architecture-sprint1.md#Decision 2)
- companyId filtering on all queries (matches architecture-sprint1.md#companyId Filtering Pattern)
- No client-side data fetching (Server Component + direct Drizzle queries)

### Previous Story Intelligence

**From Story 9.3 (Assignment Display) — completed 2026-02-14:**
- `VoiceMessageCard` now handles 4 assignment states including the Inbox case (unassigned + AI suggestion)
- `SummaryFields` shows "Vorschlag: {name} (Unsicher)" for messages with `aiSuggestedProjectName` but no `projectName`
- `DashboardMessage` interface has all assignment fields: `projectId`, `projectName`, `assignedBy`, `aiConfidence`, `aiReasoning`, `aiSuggestedProjectName`
- `formatConfidenceLabel()` exists in `lib/format.ts` with German labels
- Map-based project name resolution pattern established — follow same for user names
- `ExpandedContent` shows "KI-Begründung:" when `aiReasoning` is present

**From Story 9.2 (Voice Pipeline Extension) — completed 2026-02-14:**
- Assignment data is stored on every processed voice message
- When confidence < 0.7: `projectId` stays null (message goes to Inbox)
- `aiSuggestedProjectId` always stores AI's best guess regardless of confidence
- Pipeline error handler does NOT surface assignment failures — message status stays "done"

**From Story 8.1 (Multi-Tenant Data Model) — completed:**
- `getAuthContext()` returns `{ userId, companyId, role }`
- `voiceMessages.companyId` column exists and is backfilled for existing messages
- Index `idx_voice_messages_assignment` exists on `(companyId, projectId, status)` — optimal for inbox query

**From Story 8.3 (Project List & Detail Timeline) — completed:**
- `getProjects()` and `getProjectMessages()` in `lib/actions/projects.ts` use same `getAuthContext()` + companyId pattern
- Server Component data fetching with `Promise.all()` for parallel queries

### Git Intelligence

Recent commits:
```
50f1fbd Add assignment display to voice message cards (Story 9.3)
342c47d Extend voice pipeline with AI project assignment step (Story 9.2)
33f6e15 Implement AI project assignment prompt and module (Story 9.1)
c452748 Implement project list with activity stats and detail timeline (Story 8.3)
```

**This story's commit should follow:** `"Implement inbox page for unassigned voice messages (Story 10.1)"`

### Library/Framework Specifics

| Library | Version | Key Notes for This Story |
|---------|---------|--------------------------|
| React | 19.2.3 | Inbox page is Server Component — VoiceMessageCard remains Client Component |
| Next.js | 16.1.6 | Server Components with direct DB queries, `app/app/inbox/` route |
| Drizzle ORM | 0.45.1 | `and()`, `eq()`, `isNull()`, `desc()` from `drizzle-orm` for inbox query |
| Tailwind v4 | CSS-first | Follow existing class patterns, no config changes |
| Lucide React | (installed) | `Inbox` icon available for nav tab + empty state |

### Existing Index Coverage

The `idx_voice_messages_assignment` compound index on `(companyId, projectId, status)` — created in Story 8.1 migration — is optimal for the inbox query pattern:

```sql
WHERE company_id = ? AND project_id IS NULL AND status = 'done'
```

This index covers all three filter columns, ensuring the inbox query loads within 2 seconds (S1-NFR3) even at scale.

[Source: lib/db/schema.ts — voiceMessages table indexes]

### References

- [Source: architecture-sprint1.md#Decision 6 — Server Components for Dashboard + Inbox]
- [Source: architecture-sprint1.md#Decision 2 — Server Actions for CRUD, Route Handlers for Upload]
- [Source: architecture-sprint1.md#Decision 3 — Session Helper for Auth Context]
- [Source: architecture-sprint1.md#Inbox Query Pattern]
- [Source: architecture-sprint1.md#companyId Filtering Pattern]
- [Source: architecture-sprint1.md#§5 New Files — lib/actions/inbox.ts, app/app/inbox/page.tsx]
- [Source: prd-sprint1.md#S1-FR16 — Inbox page (unassigned messages)]
- [Source: prd-sprint1.md#S1-NFR3 — Inbox page load <2 seconds]
- [Source: prd-sprint1.md#S1-NFR12 — Touch targets >= 48px]
- [Source: epics.md#Story 10.1 Acceptance Criteria]
- [Source: 9-3-assignment-display-on-voice-message-cards.md — DashboardMessage interface, VoiceMessageCard reuse]
- [Source: components/app/voice-message-card.tsx — DashboardMessage, SummaryFields assignment states]
- [Source: components/app/bottom-nav.tsx — tabs array pattern]
- [Source: components/app/app-header.tsx — navLinks array pattern]
- [Source: app/app/page.tsx — groupMessagesByDay, parallel query, projectNameMap]
- [Source: lib/auth/session.ts — getAuthContext()]
- [Source: lib/db/schema.ts — voiceMessages assignment columns, idx_voice_messages_assignment]
- [Source: lib/format.ts — formatRelativeTime, formatConfidenceLabel]

## Dev Agent Record

### Agent Model Used

Claude Opus 4.6

### Debug Log References

No issues encountered during implementation.

### Completion Notes List

- Task 6 implemented first (DashboardMessage `userId` field) as prerequisite for Task 2
- Task 1: Created `lib/actions/inbox.ts` with company-scoped query (companyId + projectId IS NULL + status = "done")
- Task 2+3: Created `app/app/inbox/page.tsx` as Server Component with parallel queries (messages, projects, users), inline `groupMessagesByDay`, userNameMap for per-message sender resolution, empty state with Inbox icon
- Task 4+5: Added Inbox tab to both bottom-nav and app-header in position 2 (Aufnahme | Posteingang | Projekte | Übersicht)
- Task 6: Added optional `userId?: string` to `DashboardMessage` interface — non-breaking for dashboard
- Task 7: Build passes with zero TypeScript errors, `/app/inbox` route visible as Dynamic
- All ACs satisfied: company-scoped messages, sorted newest first, AI suggestion display, empty state, navigation links, touch targets >=48px (min-h-12 on nav items)

### Change Log

- 2026-02-14: Implemented inbox page for unassigned voice messages (Story 10.1)
- 2026-02-14: Code review fixes — eliminated double getAuthContext(), removed unused import, hoisted DateTimeFormat, removed dead code

### File List

- `lib/actions/inbox.ts` — NEW: Server Actions `getUnassignedMessages()` + `getInboxData()` with company-scoped queries
- `app/app/inbox/page.tsx` — NEW: Server Component inbox page with message list, empty state, and day grouping
- `components/app/voice-message-card.tsx` — MODIFIED: Added optional `userId` field to `DashboardMessage` interface
- `components/app/bottom-nav.tsx` — MODIFIED: Added Inbox tab (4th tab) with `Inbox` icon from lucide-react
- `components/app/app-header.tsx` — MODIFIED: Added Inbox link to desktop navigation

## Senior Developer Review (AI)

**Reviewer:** Sempre | **Date:** 2026-02-14 | **Outcome:** Approved with fixes applied

### Git vs Story Discrepancies: 0

All 5 files in File List match git reality. No undocumented changes.

### Issues Found: 0 High, 2 Medium, 5 Low

**MEDIUM (fixed):**
- M1: Unused `desc` import in page.tsx — removed
- M2: Double `getAuthContext()` call per page load (page + server action) — eliminated via `getInboxData()` bundled function with internal `fetchUnassigned()` helper

**LOW (fixed):**
- L1: Dead code `projectName` ternary always evaluating to null — simplified to `projectName: null`
- L2: `Intl.DateTimeFormat` created per-message — hoisted outside loop

**LOW (noted, not fixed — pre-existing patterns):**
- L3: "Posteingang" label (12 chars) could overflow on ≤320px viewports
- L4: Server timezone used for day grouping (pre-existing dashboard pattern)
- L5: Read function exported via `"use server"` (architectural smell, but follows project convention)

### AC Validation

| AC | Status |
|----|--------|
| AC#1: Company-scoped unassigned messages with summary/timestamp/AI suggestion | IMPLEMENTED |
| AC#2: Empty state | IMPLEMENTED |
| AC#3: Touch targets ≥48px | IMPLEMENTED |
| AC#4: Inbox in bottom nav + header nav | IMPLEMENTED |

### Build Verification

`pnpm build` — 0 errors, `/app/inbox` listed as Dynamic route.
