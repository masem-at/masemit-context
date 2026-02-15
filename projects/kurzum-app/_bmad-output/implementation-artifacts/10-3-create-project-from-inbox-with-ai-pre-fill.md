# Story 10.3: Create Project from Inbox with AI Pre-fill

Status: done

<!-- Note: Validation is optional. Run validate-create-story for quality check before dev-story. -->

## Story

As a **Meister**,
I want to create a new project directly from the Inbox with fields pre-filled from the AI summary,
so that I can quickly set up a new construction site when a message doesn't match any existing project.

## Acceptance Criteria

1. **Given** a Meister viewing an unassigned message in the Inbox **When** clicking "Neues Projekt erstellen" **Then** the project creation form opens with fields pre-filled from the voice message summary **And** the AI extracts: possible project name, customer name, and address from the summary/transcript **And** the Meister can review and edit all pre-filled values before submitting.

2. **Given** the Meister submits the pre-filled project form **When** the project is created **Then** the new project is saved with the Meister's companyId **And** the voice message is automatically assigned to the new project **And** `assignedBy` is set to `"user"` **And** the Meister is redirected to the new project's detail page **And** the message disappears from the Inbox.

3. **Given** the AI cannot extract meaningful data from the summary **When** the "Neues Projekt erstellen" form opens **Then** the form fields are empty (not pre-filled with garbage) **And** the Meister can fill in all fields manually.

**FRs:** S1-FR18 | **NFRs:** S1-NFR12

## Tasks / Subtasks

- [x] Task 1: Create AI project data extraction function (AC: #1, #3)
  - [x] 1.1 Add `extractProjectData()` function to `lib/ai/assignment.ts` that uses Mistral to extract project name, customer name, and address from transcript + summary
  - [x] 1.2 Define `PROJECT_EXTRACTION_PROMPT` in `lib/ai/prompts.ts` with German Handwerker context
  - [x] 1.3 Add Zod schema for extraction response validation (`extractionResponseSchema`)
  - [x] 1.4 Handle graceful fallback: if extraction fails or returns empty, return `null` fields (AC: #3)

- [x] Task 2: Create server action for pre-fill data retrieval (AC: #1, #3)
  - [x] 2.1 Add `getProjectPreFillData(messageId: string)` to `lib/actions/inbox.ts`
  - [x] 2.2 Fetch message by ID with companyId tenant isolation
  - [x] 2.3 Call `extractProjectData()` with transcript + summary
  - [x] 2.4 Return `{ name, customerName, address, description }` or empty defaults

- [x] Task 3: Create "Create Project from Inbox" page and form (AC: #1, #2)
  - [x] 3.1 Create `app/app/inbox/new-project/page.tsx` — Server Component accepting `?messageId=xxx` searchParam
  - [x] 3.2 Create `app/app/inbox/new-project/inbox-project-create-form.tsx` — Client Component wrapping existing `ProjectForm` with pre-fill values
  - [x] 3.3 Reuse existing `components/app/project-form.tsx` with `defaultValues` prop
  - [x] 3.4 After successful project creation, call `assignMessageToProject(messageId, projectId)` to auto-assign
  - [x] 3.5 Redirect to `/app/projects/[newProjectId]` after both operations succeed

- [x] Task 4: Add "Neues Projekt erstellen" button to Inbox page (AC: #1)
  - [x] 4.1 Add a "Neues Projekt erstellen" link/button next to each `AssignmentDropdown` in `app/app/inbox/page.tsx`
  - [x] 4.2 Link navigates to `/app/inbox/new-project?messageId={message.id}`
  - [x] 4.3 Button uses ghost/secondary style to differentiate from primary "Zuordnen" action
  - [x] 4.4 Ensure 48px min touch target (S1-NFR12)

- [x] Task 5: Extend `createProject` server action for combined create+assign flow (AC: #2)
  - [x] 5.1 Create `createProjectAndAssignMessage(input, messageId)` in `lib/actions/projects.ts` or `lib/actions/inbox.ts`
  - [x] 5.2 Wrap project creation + message assignment in atomic-like pattern (sequential operations with rollback awareness)
  - [x] 5.3 Call `revalidatePath` for `/app/inbox`, `/app`, `/app/projects`
  - [x] 5.4 Return the new project's ID for redirect

## Dev Notes

### Architecture & Implementation Patterns

- **Server Actions pattern** (established in Stories 8.2, 10.2): All mutations use `"use server"` functions with `getAuthContext()` for tenant isolation
- **Existing reusable components**: `ProjectForm` (shared between create/edit), `AssignmentDropdown`, `VoiceMessageCard`
- **Existing server actions**: `createProject()` in `lib/actions/projects.ts`, `assignMessageToProject()` in `lib/actions/inbox.ts`
- **AI module**: `lib/ai/assignment.ts` already demonstrates the pattern for Mistral LLM calls with Zod validation
- **Prompt versioning**: Constants in `lib/ai/prompts.ts`, version stored in DB per message — follow same pattern for extraction prompt

### Critical Constraints

- **Tenant isolation**: Every DB query MUST filter by `companyId` from `getAuthContext()` — verify message belongs to user's company before extraction
- **AI keys server-side only**: `MISTRAL_API_KEY` never exposed to client (S1-NFR7)
- **Graceful degradation**: If Mistral extraction fails, the form MUST still open with empty fields — never block the user
- **Combined operation atomicity**: Project creation + message assignment are two DB operations; if assignment fails after project creation, the project still exists (acceptable) — log the error
- **No PII in prompts/logs**: Don't log customer names or addresses in error messages (S1-NFR8)

### AI Extraction Approach

The extraction function should use a **lightweight Mistral call** (mistral-small-latest) to parse the transcript + summary and extract:
- **Project name**: Often the Baustelle/Ort/Projekt mentioned in the transcript (e.g., "Wohnhaus Müller", "Baustelle Hauptstraße")
- **Customer name**: The Auftraggeber/Kunde/Bauherr mentioned (e.g., "Familie Müller", "Firma Huber")
- **Address**: The Adresse/Straße/Ort mentioned (e.g., "Hauptstraße 12, 3500 Krems")

The prompt should handle common Handwerker speech patterns:
- "Bin grad beim Müller in der Hauptstraße..."
- "Baustelle Neubau Schmied, Alleestraße 5..."
- "Beim Kunden in Kautzen..."

If no clear data is found, return `null` fields — never guess or hallucinate.

### Reuse Strategy

| Component | Source | Reuse Approach |
|---|---|---|
| `ProjectForm` | `components/app/project-form.tsx` | Direct reuse — already accepts `defaultValues` prop |
| `createProject()` | `lib/actions/projects.ts` | Call directly, then separately call `assignMessageToProject()` |
| `assignMessageToProject()` | `lib/actions/inbox.ts` | Call directly after project creation |
| `getClient()` | `lib/ai/mistral.ts` | Reuse singleton Mistral client |
| Zod validation | `lib/validations/project.ts` | Same `createProjectSchema` for form validation |

### Project Structure Notes

- New files:
  - `app/app/inbox/new-project/page.tsx` — Server Component (pre-fill data fetch)
  - `app/app/inbox/new-project/inbox-project-create-form.tsx` — Client Component (form + submit)
- Modified files:
  - `app/app/inbox/page.tsx` — Add "Neues Projekt erstellen" button per message
  - `lib/ai/assignment.ts` — Add `extractProjectData()` function
  - `lib/ai/prompts.ts` — Add `PROJECT_EXTRACTION_PROMPT` and version constant
  - `lib/actions/inbox.ts` — Add `getProjectPreFillData()` and `createProjectAndAssignMessage()` server actions
- No conflicts with existing structure detected — follows established patterns from Stories 8.2, 10.1, 10.2

### Testing Strategy

- **Manual test 1**: Navigate to Inbox → click "Neues Projekt erstellen" → verify form opens with AI pre-filled values → edit values → submit → verify redirect to project detail → verify message assigned → verify message gone from Inbox
- **Manual test 2**: Test with a message that has minimal/no extractable data → verify form opens empty → manual fill → submit works
- **Manual test 3**: Test with Mistral API unavailable → verify form opens empty (graceful degradation), no error shown to user
- **Manual test 4**: Verify Inbox badge count decrements after message is assigned to new project
- **Manual test 5**: Verify tenant isolation — cannot create project from another company's message

### References

- [Source: _bmad-output/planning-artifacts/epics.md#Story 10.3] — Acceptance criteria and FR mapping
- [Source: _bmad-output/planning-artifacts/architecture-sprint1.md#Server Actions for CRUD] — Server Action pattern
- [Source: _bmad-output/planning-artifacts/architecture-sprint1.md#companyId filtering pattern] — Tenant isolation requirement
- [Source: _bmad-output/planning-artifacts/prd-sprint1.md#AI-prefilled project creation from Inbox] — MVP feature requirement
- [Source: lib/ai/assignment.ts] — Existing AI module pattern for Mistral calls with Zod validation
- [Source: lib/actions/projects.ts#createProject] — Existing project creation server action
- [Source: lib/actions/inbox.ts#assignMessageToProject] — Existing message assignment server action
- [Source: components/app/project-form.tsx] — Reusable project form component
- [Source: app/app/inbox/page.tsx] — Current inbox implementation to extend
- [Source: lib/ai/prompts.ts] — Prompt versioning pattern

### Previous Story Intelligence (10.1 + 10.2)

- **Inbox data model**: `getInboxData()` returns messages + all projects + company users — reuse for fetching message data
- **Assignment pattern**: `assignMessageToProject()` already handles atomic update with tenant isolation — reuse directly
- **Revalidation paths**: Stories 10.1/10.2 established `revalidatePath("/app/inbox")`, `revalidatePath("/app")`, `revalidatePath("/app/projects")` — follow same pattern
- **Navigation badge**: Badge updates automatically via `getUnassignedCount()` in layout — no additional work needed
- **AssignmentDropdown**: Already placed per-message in inbox — the "Neues Projekt erstellen" button should be placed adjacent to it

### Git Intelligence

Recent commits show consistent pattern:
- Story implementation in single commits
- Clear commit messages reflecting feature scope
- No package dependency additions expected for this story (all libraries already installed)
- Mistral AI client, Server Actions, ProjectForm — all available from previous stories

## Dev Agent Record

### Agent Model Used

Claude Opus 4.6

### Debug Log References

- No blocking issues encountered during implementation

### Completion Notes List

- Task 1: Added `extractProjectData()` to `lib/ai/assignment.ts` using Mistral LLM (mistral-small-latest) with `PROJECT_EXTRACTION_PROMPT`. Zod validation via `extractionResponseSchema`. Graceful fallback: returns null fields on any failure (empty transcript, API error, invalid JSON, schema mismatch).
- Task 2: Added `getProjectPreFillData()` server action with companyId tenant isolation. Fetches message transcript+summary, calls extraction, maps to form-compatible format.
- Task 3: Created `app/app/inbox/new-project/page.tsx` (Server Component with searchParams Promise pattern for Next.js 16) and `inbox-project-create-form.tsx` (Client Component reusing existing `ProjectForm`).
- Task 4: Added "Neues Projekt" link with Plus icon next to AssignmentDropdown in inbox page. Uses ghost styling, 48px min touch target (min-h-12).
- Task 5: Added `createProjectAndAssignMessage()` server action with sequential create+assign pattern. Assignment failure after project creation is logged but non-blocking (project still exists). Revalidates all affected paths.

### Implementation Plan

- Reused existing `ProjectForm` component with `defaultValues` prop (no modifications needed)
- Combined server action `createProjectAndAssignMessage()` placed in `lib/actions/inbox.ts` (not `projects.ts`) since it's inbox-workflow-specific
- Static import of `createProjectSchema` (no circular dependency exists)
- Extraction prompt designed for Austrian/German Handwerker speech patterns with explicit null-over-hallucination rule

### File List

- `lib/ai/prompts.ts` (modified) — Added `PROJECT_EXTRACTION_PROMPT`
- `lib/validations/voice.ts` (modified) — Added `extractionResponseSchema` and `ExtractionResponse` type
- `lib/ai/assignment.ts` (modified) — Added `extractProjectData()` function and `ProjectExtractionResult` interface
- `lib/actions/inbox.ts` (modified) — Added `getProjectPreFillData()`, `createProjectAndAssignMessage()`, `ProjectPreFillData` interface
- `app/app/inbox/page.tsx` (modified) — Added "Neues Projekt" link per unassigned message
- `app/app/inbox/new-project/page.tsx` (new) — Server Component for inbox project creation with AI pre-fill
- `app/app/inbox/new-project/inbox-project-create-form.tsx` (new) — Client Component wrapping ProjectForm with inbox-specific submit handler
- `app/app/inbox/new-project/loading.tsx` (new) — Loading skeleton for AI pre-fill wait time

### Change Log

- 2026-02-14: Implemented Story 10.3 — AI-prefilled project creation from inbox with Mistral extraction, combined create+assign server action, and inbox UI integration
- 2026-02-15: Code review fixes — Added messageId UUID guard with redirect, message existence check before project creation, loading skeleton, removed dead EXTRACTION_PROMPT_VERSION export, replaced unnecessary dynamic import with static import
