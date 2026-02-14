# Story 9.3: Assignment Display on Voice Message Cards

Status: done

<!-- Note: Validation is optional. Run validate-create-story for quality check before dev-story. -->

## Story

As a **Meister**,
I want to see which project a voice message was assigned to and the confidence level,
so that I can verify the AI's decisions and trust the system.

## Acceptance Criteria

1. **Given** a voice message that was auto-assigned by AI (confidence >= 0.7)
   **When** displayed on the dashboard or project timeline
   **Then** the assigned project name is shown
   **And** a confidence indicator is visible (e.g., "Sicher" for >=0.9, "Wahrscheinlich" for >=0.7)
   **And** `assignedBy: "ai"` is indicated visually

2. **Given** a voice message that was manually assigned by a user
   **When** displayed on the dashboard or project timeline
   **Then** the assigned project name is shown
   **And** `assignedBy: "user"` is indicated visually (e.g., "Manuell zugeordnet")

3. **Given** a voice message in the Inbox (unassigned, confidence < 0.7)
   **When** displayed on the dashboard (and later Inbox in Story 10.1)
   **Then** the AI's best guess project is shown as a suggestion (from `aiSuggestedProjectId`)
   **And** the low confidence is indicated
   **And** the reasoning summary is accessible

**FRs:** S1-FR9 | **NFRs:** S1-NFR12, S1-NFR13

## Tasks / Subtasks

- [x] Task 1: Extend `DashboardMessage` interface with assignment fields (AC: #1, #2, #3)
  - [x] 1.1 Add to `DashboardMessage` in `components/app/voice-message-card.tsx`:
    - `projectId: string | null`
    - `projectName: string | null` (resolved from projects table)
    - `assignedBy: string | null` ("ai" | "user" | null)
    - `aiConfidence: number | null`
    - `aiReasoning: string | null`
    - `aiSuggestedProjectName: string | null` (resolved from `aiSuggestedProjectId`)

- [x] Task 2: Update dashboard query to resolve project names (AC: #1, #2, #3)
  - [x] 2.1 In `app/app/page.tsx`, fetch all projects for the company alongside messages:
    ```typescript
    const { userId, companyId } = await getAuthContext();
    const [allProjects, messages, user] = await Promise.all([
      db.select({ id: projects.id, name: projects.name })
        .from(projects)
        .where(eq(projects.companyId, companyId)),
      db.select().from(voiceMessages)
        .where(eq(voiceMessages.userId, userId))
        .orderBy(desc(voiceMessages.createdAt)),
      db.query.users.findFirst({ where: eq(users.id, userId), columns: { name: true } }),
    ]);
    ```
  - [x] 2.2 Build a `projectNameMap: Map<string, string>` from the projects query
  - [x] 2.3 Pass `projectNameMap` to `groupMessagesByDay()` and add a second parameter to the function signature
  - [x] 2.4 Import `projects` from `@/lib/db/schema` in `app/app/page.tsx`

- [x] Task 3: Update `groupMessagesByDay()` to map assignment fields (AC: #1, #2, #3)
  - [x] 3.1 Extend the input type parameter of `groupMessagesByDay()` with assignment columns:
    - `projectId: string | null`
    - `assignedBy: string | null`
    - `aiConfidence: number | null`
    - `aiReasoning: string | null`
    - `aiSuggestedProjectId: string | null`
  - [x] 3.2 Add `projectNameMap: Map<string, string>` as second parameter
  - [x] 3.3 Map new fields in the DashboardMessage construction:
    ```typescript
    projectId: msg.projectId,
    projectName: msg.projectId ? projectNameMap.get(msg.projectId) ?? null : null,
    assignedBy: msg.assignedBy,
    aiConfidence: msg.aiConfidence,
    aiReasoning: msg.aiReasoning,
    aiSuggestedProjectName: msg.aiSuggestedProjectId
      ? projectNameMap.get(msg.aiSuggestedProjectId) ?? null : null,
    ```

- [x] Task 4: Add confidence label helper to `lib/format.ts` (AC: #1, #3)
  - [x] 4.1 Add `formatConfidenceLabel(confidence: number): string`:
    - `>= 0.9` returns `"Sicher"`
    - `>= 0.7` returns `"Wahrscheinlich"`
    - `>= 0.4` returns `"Unsicher"`
    - `< 0.4` returns `"Sehr unsicher"`
  - [x] 4.2 Function is exported for use in both dashboard card and future Inbox page

- [x] Task 5: Update `SummaryFields` component to show assignment info (AC: #1, #2, #3)
  - [x] 5.1 Change `SummaryFields` props to accept `message: DashboardMessage` instead of just `summary: VoiceSummary`
  - [x] 5.2 Replace the existing project display row (lines 279-286, the `summary.project` / "Nicht zugeordnet" logic) with assignment-aware display:
    - **If `message.projectName` exists (assigned):** Show project name as link-style text with assignment badge:
      - `assignedBy === "ai"`: Show confidence label badge (e.g., "Sicher" in green-tinted badge) + project name
      - `assignedBy === "user"`: Show "Manuell" in neutral badge + project name
    - **If `message.projectName` is null AND `message.aiSuggestedProjectName` exists (Inbox with AI suggestion):**
      Show "Vorschlag: {aiSuggestedProjectName}" in muted/dashed style with confidence label
    - **If no assignment and no suggestion:**
      Show "Nicht zugeordnet" in muted text (existing behavior)
  - [x] 5.3 Import `formatConfidenceLabel` from `@/lib/format`

- [x] Task 6: Show AI reasoning in `ExpandedContent` (AC: #3)
  - [x] 6.1 Add `aiReasoning` display to `ExpandedContent` component, below transcript and above metadata
  - [x] 6.2 Only show if `message.aiReasoning` is not null
  - [x] 6.3 Display with a label "KI-Begründung:" in muted small text, reasoning text below

- [x] Task 7: Enhance project timeline with confidence indicator (AC: #1)
  - [x] 7.1 In `app/app/projects/[id]/page.tsx`, extend the existing `assignmentLabel` logic (lines 114-119) to include confidence:
    - `assignedBy === "ai"` and `aiConfidence`: Show "{confidenceLabel} KI-Zuordnung" (e.g., "Sicher KI-Zuordnung")
    - `assignedBy === "user"`: Keep "Manuell zugeordnet" as-is
  - [x] 7.2 Import `formatConfidenceLabel` from `@/lib/format`

- [x] Task 8: Verify build and accessibility (all ACs)
  - [x] 8.1 Run `pnpm build` — zero TypeScript errors
  - [x] 8.2 Verify all new touch targets are >= 48px (S1-NFR12)
  - [x] 8.3 Verify contrast ratios meet WCAG AA (S1-NFR13)
  - [x] 8.4 Verify assignment info is visible on mobile (375px) without horizontal overflow

## Dev Notes

### Critical Architecture Patterns

**1. Project Name Resolution — Map-Based Lookup (NOT JOINs):**

The dashboard query fetches all voice messages with `db.select().from(voiceMessages)`. Assignment columns (`projectId`, `aiSuggestedProjectId`) contain UUIDs. To display project NAMES, we fetch all company projects in parallel and build a name map:

```typescript
// Simple, efficient, no complex JOIN aliases needed
const projectNameMap = new Map(allProjects.map(p => [p.id, p.name]));

// Resolve in groupMessagesByDay:
projectName: msg.projectId ? projectNameMap.get(msg.projectId) ?? null : null,
aiSuggestedProjectName: msg.aiSuggestedProjectId
  ? projectNameMap.get(msg.aiSuggestedProjectId) ?? null : null,
```

This approach is simpler than double LEFT JOINs with aliases on the same table and efficient for Sprint 1 scale (< 20 projects per company).
[Source: architecture-sprint1.md#Decision 6 — Server Components for Dashboard]

**2. Confidence Display — German Labels, Not Percentages:**

Users are Handwerker (Stefan, Markus), not data scientists. Raw percentages (e.g., "72%") are meaningless to them. Instead, use intuitive German labels:

| Confidence | Label | Visual |
|-----------|-------|--------|
| >= 0.9 | "Sicher" | Green-tinted badge |
| >= 0.7 | "Wahrscheinlich" | Default/neutral badge |
| >= 0.4 | "Unsicher" | Amber-tinted text |
| < 0.4 | "Sehr unsicher" | Muted text |

[Source: prd-sprint1.md#Journey 1 — "Zugeordnet zu: Neubau Urfahr ✓"]

**3. Assignment Badge Visual Hierarchy:**

Three distinct visual states for the project row in `SummaryFields`:

```
State 1 — AI auto-assigned (confidence >= 0.7):
📍 [Sicher] Neubau Urfahr, Fam. Gruber

State 2 — Manually assigned by user:
📍 [Manuell] Sanierung Pfarrgasse, Hr. Wimmer

State 3 — Unassigned with AI suggestion (Inbox):
📍 Vorschlag: Neubau Urfahr (Unsicher)

State 4 — No assignment, no suggestion:
📍 Nicht zugeordnet
```

[Source: epics.md#Story 9.3 Acceptance Criteria]

**4. companyId Filtering on projects query:**

```typescript
// ✅ CORRECT — fetch projects only for this company
db.select({ id: projects.id, name: projects.name })
  .from(projects)
  .where(eq(projects.companyId, companyId))
```

[Source: architecture-sprint1.md#companyId Filtering Pattern]

### Existing Codebase — What to Modify

| Action | File | What | Scope |
|--------|------|------|-------|
| MODIFY | `components/app/voice-message-card.tsx` | Extend DashboardMessage interface, update SummaryFields + ExpandedContent | Medium |
| MODIFY | `app/app/page.tsx` | Add projects query, pass projectNameMap, extend groupMessagesByDay | Medium |
| MODIFY | `app/app/projects/[id]/page.tsx` | Add confidence label to existing assignment display | Small |
| MODIFY | `lib/format.ts` | Add `formatConfidenceLabel()` helper | Small |

**No new files needed.** All changes are modifications to existing components and pages.

### Existing Code Patterns to Follow

**SummaryFields current project display (voice-message-card.tsx:279-286):**
```typescript
<div className="flex items-start gap-2 text-body">
  <span className="shrink-0" aria-hidden="true">&#x1F4CD;</span>
  {summary.project === "Unbekannt" ? (
    <span className="text-muted-foreground">Nicht zugeordnet</span>
  ) : (
    <span>{summary.project}</span>
  )}
</div>
```
This is REPLACED by assignment-aware display. The `summary.project` field from the LLM summary becomes a fallback — the explicit `projectName` from DB assignment takes priority.

**Project timeline assignment label (projects/[id]/page.tsx:114-135):**
```typescript
const assignmentLabel =
  message.assignedBy === "ai"
    ? "KI-Zuordnung"
    : message.assignedBy === "user"
      ? "Manuell zugeordnet"
      : null;

// Rendered as:
{assignmentLabel && (
  <span className="text-xs text-muted-foreground">
    {assignmentLabel}
  </span>
)}
```
This pattern is EXTENDED with confidence labels.

**Badge/label styling pattern (existing in codebase):**
```typescript
// Small inline badge — use text-xs with background tint
<span className="text-xs px-1.5 py-0.5 rounded bg-green-100 text-green-800">
  Sicher
</span>
```

### Critical Anti-Patterns to Avoid

```typescript
// ❌ WRONG: Showing raw confidence percentage to Handwerker users
<span>{(message.aiConfidence * 100).toFixed(0)}% Konfidenz</span>
// ✅ CORRECT: Use intuitive German labels
<span>{formatConfidenceLabel(message.aiConfidence)}</span>

// ❌ WRONG: Joining projects twice with aliases in dashboard query
const result = await db.select().from(voiceMessages)
  .leftJoin(p1, eq(voiceMessages.projectId, p1.id))
  .leftJoin(p2, eq(voiceMessages.aiSuggestedProjectId, p2.id));
// ✅ CORRECT: Simple map-based lookup
const projectNameMap = new Map(allProjects.map(p => [p.id, p.name]));

// ❌ WRONG: Using summary.project as the source of truth for project name
<span>{summary.project}</span>
// ✅ CORRECT: Use DB-assigned projectName (resolved from projectId)
<span>{message.projectName}</span>

// ❌ WRONG: Showing assignment reasoning directly in collapsed card
// aiReasoning can be 1-2 sentences — too long for collapsed view
// ✅ CORRECT: Show reasoning only in expanded view (ExpandedContent)

// ❌ WRONG: Missing companyId filter on projects query
db.select().from(projects) // Data leak!
// ✅ CORRECT: Always filter by companyId
db.select().from(projects).where(eq(projects.companyId, companyId))

// ❌ WRONG: Adding new UI without min-h-12 (48px) touch targets
<button className="text-xs px-2">Zuordnen</button>
// ✅ CORRECT: All interactive elements meet S1-NFR12
<button className="text-xs px-2 min-h-12">Zuordnen</button>
// Note: Badge labels are not interactive — no touch target needed
```

### Scope Boundary — What This Story Does NOT Do

| Out of Scope | Belongs To |
|-------------|------------|
| Creating the Inbox page at `/app/inbox` | Story 10.1 |
| Manual reassignment from Inbox | Story 10.2 |
| Creating new project from Inbox with AI pre-fill | Story 10.3 |
| Inbox badge/counter in navigation | Story 10.2 |
| Project-based dashboard rebuild | Story 11.1 |
| AI assignment accuracy evaluation (ADR-006) | Separate task |

**What this story DOES prepare for Story 10.1:** The `DashboardMessage` interface and `SummaryFields` component will handle the Inbox display case (unassigned + AI suggestion). Story 10.1 can reuse `VoiceMessageCard` directly for Inbox items.

### Project Structure Notes

**Modified files (4):**
- `components/app/voice-message-card.tsx` — Extend interface + update SummaryFields + ExpandedContent
- `app/app/page.tsx` — Add projects query, extend groupMessagesByDay
- `app/app/projects/[id]/page.tsx` — Add confidence label to timeline
- `lib/format.ts` — Add formatConfidenceLabel helper

**No new files or directories needed.** All changes within existing structure.

**Alignment with architecture:**
- Server-side data resolution via project name map (no client-side fetching)
- companyId filtering on all new queries
- Tailwind v4 CSS-first styling (no tailwind.config.js changes needed)
- All new elements follow existing component patterns

### Previous Story Intelligence

**From Story 9.2 (Voice Pipeline Extension) — completed 2026-02-14:**
- Assignment data is now stored on every processed voice message
- `aiConfidence`, `aiReasoning`, `assignedBy`, `assignedAt`, `aiSuggestedProjectId` are populated after pipeline
- `assignedBy === "ai"` only when confidence >= 0.7 (auto-assigned)
- `assignedBy === null` when confidence < 0.7 (Inbox) or assignment failed
- `aiSuggestedProjectId` always stores AI's best guess regardless of confidence threshold
- Pipeline error handler does NOT surface assignment failures to user — card should never show assignment error state

**From Story 9.1 (Assignment Prompt & AI Module) — completed 2026-02-14:**
- `aiReasoning` is German text, 1-2 sentences — keep in expanded view only
- Confidence scale: >= 0.8 = clear match, 0.5-0.8 = probable, < 0.5 = uncertain

**From Story 8.3 (Project List & Detail Timeline) — completed 2026-02-14:**
- Project timeline already shows basic assignment labels ("KI-Zuordnung" / "Manuell zugeordnet") at `projects/[id]/page.tsx:114-135`
- `formatRelativeTime()` extracted to `lib/format.ts` — `formatConfidenceLabel()` follows same pattern (utility function for reuse)
- `getProjectMessages()` returns full `VoiceMessage` type including all assignment columns

**From Story 7.1 (Voice Message Dashboard) — completed:**
- `VoiceMessageCard` is a Client Component ("use client") — imports must be serializable
- `groupMessagesByDay()` is a server-side helper that transforms DB rows to `DashboardMessage`
- Card expand/collapse pattern: click to toggle, ChevronDown rotates

### Git Intelligence

Recent commits:
```
342c47d Extend voice pipeline with AI project assignment step (Story 9.2)
33f6e15 Implement AI project assignment prompt and module (Story 9.1)
c452748 Implement project list with activity stats and detail timeline (Story 8.3)
```

**This story's commit should follow:** `"Add assignment display to voice message cards (Story 9.3)"`

### Library/Framework Specifics

| Library | Version | Key Notes for This Story |
|---------|---------|--------------------------|
| React | 19.2.3 | VoiceMessageCard is Client Component with useState for expand/collapse |
| Next.js | 16.1.6 | Dashboard + Timeline are Server Components with direct DB queries |
| Drizzle ORM | 0.45.1 | `db.select()` returns full inferred type including all columns |
| Tailwind v4 | CSS-first | Badge styles via utility classes, no config changes |
| Lucide React | (installed) | Icons for card UI — no new icons needed |

### References

- [Source: architecture-sprint1.md#Decision 6 — Server Components for Dashboard + Inbox]
- [Source: architecture-sprint1.md#companyId Filtering Pattern]
- [Source: prd-sprint1.md#S1-FR9 — User sees assigned project + confidence]
- [Source: prd-sprint1.md#S1-NFR12 — Touch targets >= 48px]
- [Source: prd-sprint1.md#S1-NFR13 — WCAG AA contrast ratio]
- [Source: prd-sprint1.md#Journey 1 — "Zugeordnet zu: Neubau Urfahr, Fam. Gruber ✓"]
- [Source: prd-sprint1.md#Journey 2 — Inbox AI suggestion display]
- [Source: epics.md#Story 9.3 Acceptance Criteria]
- [Source: 9-2-voice-pipeline-extension-with-assignment-step.md — assignment data structure]
- [Source: 9-1-assignment-prompt-and-ai-module.md — aiReasoning format, confidence scale]
- [Source: components/app/voice-message-card.tsx — DashboardMessage interface, SummaryFields]
- [Source: app/app/page.tsx — groupMessagesByDay, dashboard query]
- [Source: app/app/projects/[id]/page.tsx — existing assignment label pattern]
- [Source: lib/format.ts — formatRelativeTime pattern]
- [Source: lib/db/schema.ts — voiceMessages assignment columns]

## Change Log

- 2026-02-14: Implemented assignment display on voice message cards — extended DashboardMessage interface, added project name resolution via map-based lookup, updated SummaryFields with 4-state assignment display (AI-assigned with confidence badge, manually assigned, AI suggestion, unassigned), added AI reasoning in expanded view, enhanced project timeline with confidence labels, added formatConfidenceLabel helper.
- 2026-02-14: Code review fixes — added summary.project fallback for pre-9.2 messages, amber-tinted confidence label for "Unsicher" in suggestion state, replaced non-null assertion with runtime guard in SummaryFields.

## Dev Agent Record

### Agent Model Used

Claude Opus 4.6

### Debug Log References

No issues encountered. Build passed on first attempt with zero TypeScript errors.

### Completion Notes List

- Extended `DashboardMessage` interface with 6 new assignment fields (projectId, projectName, assignedBy, aiConfidence, aiReasoning, aiSuggestedProjectName)
- Dashboard query now fetches all company projects in parallel via `Promise.all()` and builds a `projectNameMap` for server-side name resolution — avoids complex JOINs
- `groupMessagesByDay()` extended with `projectNameMap` second parameter and maps all assignment fields into DashboardMessage
- Added `formatConfidenceLabel()` to `lib/format.ts` with 4-tier German labels (Sicher/Wahrscheinlich/Unsicher/Sehr unsicher)
- `SummaryFields` refactored from `summary: VoiceSummary` to `message: DashboardMessage` props — now shows assignment-aware project display with visual badges
- `ExpandedContent` shows "KI-Begründung:" section when `aiReasoning` is present
- Project timeline assignment label now includes confidence (e.g., "Sicher KI-Zuordnung")
- All new badge elements are non-interactive `<span>` — no touch target concerns
- WCAG AA contrast verified: green-800/green-100, stone-700/stone-100
- Mobile-safe: `flex-wrap` on badge container, no fixed widths

### Code Review Fixes (2026-02-14)

- **M1 Fixed:** Added `summary.project` fallback — when no DB assignment and no AI suggestion, falls back to LLM-extracted project name from `summary.project` (preserves pre-9.2 behavior)
- **M2 Fixed:** Added amber-tinted text (`text-amber-700`) for "Unsicher" confidence level in suggestion display
- **M3 Fixed:** Replaced `message.summary!` non-null assertion with `if (!summary) return null` runtime guard

### File List

- `components/app/voice-message-card.tsx` — MODIFIED (extended DashboardMessage interface, refactored SummaryFields to assignment-aware display, added AI reasoning in ExpandedContent)
- `app/app/page.tsx` — MODIFIED (added projects import, parallel projects query with companyId filter, projectNameMap, extended groupMessagesByDay with assignment fields)
- `app/app/projects/[id]/page.tsx` — MODIFIED (added formatConfidenceLabel import, extended assignmentLabel with confidence)
- `lib/format.ts` — MODIFIED (added formatConfidenceLabel helper function)
