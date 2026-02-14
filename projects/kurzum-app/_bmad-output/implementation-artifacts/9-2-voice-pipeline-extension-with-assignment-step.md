# Story 9.2: Voice Pipeline Extension with Assignment Step

Status: done

<!-- Note: Validation is optional. Run validate-create-story for quality check before dev-story. -->

## Story

As a **Meister**,
I want voice messages to be automatically assigned to projects after transcription and summary,
so that most messages land directly in the correct project timeline.

**FFG Research Question #3:** "Automatische Projekt-Zuordnung durch kontextuelles Matching — Kann ein General-Purpose-LLM (Mistral) Sprachnachrichten anhand von Kontext (Projektliste, Kundenname, Adresse, letzte Nachrichten) dem richtigen Bauprojekt zuordnen?"

## Acceptance Criteria

1. **Given** a voice message has been successfully transcribed and summarized (status="done")
   **When** the assignment step runs in the pipeline
   **Then** `assignToProject()` is called with transcript, userId, companyId
   **And** the result (confidence, reasoning, suggestedProjectId) is stored in the voice_messages record
   **And** the `llmPromptVersion` field stores the combined version string (e.g., "summary:SUMMARY_V1,assignment:ASSIGNMENT_V1")

2. **Given** the assignment returns confidence >= 0.7
   **When** the pipeline completes
   **Then** the voice message's `projectId` is set to the suggested project
   **And** `assignedBy` is set to "ai"
   **And** `assignedAt` is set to the current timestamp

3. **Given** the assignment returns confidence < 0.7
   **When** the pipeline completes
   **Then** the voice message's `projectId` remains null (appears in Inbox)
   **And** `aiSuggestedProjectId` still stores the AI's best guess
   **And** `assignedBy` remains null

4. **Given** the assignment step fails (LLM error, timeout, JSON parse failure)
   **When** the pipeline handles the error
   **Then** the voice message status remains "done" (NOT changed to "error")
   **And** `projectId` remains null (message appears in Inbox)
   **And** the existing STT + Summary results are preserved (S1-NFR9)
   **And** the error is logged server-side but does not surface to the user

5. **Given** the extended pipeline
   **When** measuring total latency (STT + Summary + Assignment)
   **Then** total pipeline time is <45 seconds (S1-NFR1)

6. **Given** the voice route handler `POST /api/voice`
   **When** processing a voice message
   **Then** it uses `getAuthContext()` (already implemented — verify unchanged)
   **And** `companyId` is stored on the voice message record (already implemented — verify unchanged)

**FRs:** S1-FR8, S1-FR23, S1-FR24 | **NFRs:** S1-NFR1, S1-NFR9, S1-NFR10

## Tasks / Subtasks

- [x] Task 1: Add assignment step to voice pipeline after summary (AC: #1, #2, #3, #4)
  - [x] 1.1 Import `assignToProject` from `@/lib/ai/assignment` in `app/api/voice/route.ts`
  - [x] 1.2 Import `VOICE_SUMMARY_VERSION` and `ASSIGNMENT_PROMPT_VERSION` from `@/lib/ai/prompts`
  - [x] 1.3 After the successful summary DB update (line 88-98, where status is set to "done"), add a new try/catch block for the assignment step
  - [x] 1.4 Inside the try block:
    - Call `const assignment = await assignToProject(sttResult.transcript, userId, companyId);`
    - Determine auto-assign: `const autoAssign = assignment.confidence >= 0.7;`
    - Build the combined prompt version string: `` `summary:${VOICE_SUMMARY_VERSION},assignment:${ASSIGNMENT_PROMPT_VERSION}` ``
    - Update voice message with assignment metadata via `db.update(voiceMessages).set({...}).where(eq(voiceMessages.id, entry.id))`
    - Set fields: `aiConfidence`, `aiReasoning`, `aiSuggestedProjectId` (always the AI's suggestion), `llmPromptVersion` (combined string)
    - If `autoAssign`: additionally set `projectId`, `assignedBy: "ai"`, `assignedAt: sql\`now()\``
    - If NOT `autoAssign`: `projectId` stays null, `assignedBy` stays null — message goes to Inbox
  - [x] 1.5 Inside the catch block:
    - Log the error with structured `console.error` (no PII — only voiceMessageId and error type/message)
    - Distinguish `LLMValidationError` for research logging (include rawContent, zodErrors)
    - Do NOT change the voice message status — it stays "done"
    - Do NOT rethrow — the response should still be a success (transcript + summary are preserved)
  - [x] 1.6 Update the `llmPromptVersion` in the summary step's DB update as well: set it to `` `summary:${VOICE_SUMMARY_VERSION}` `` so that messages where assignment fails still have partial version tracking

- [x] Task 2: Extend the success response payload (AC: #1, #2, #3)
  - [x] 2.1 After the assignment try/catch block, re-fetch the final voice message state (or build from the update result)
  - [x] 2.2 Include assignment fields in the success response: `projectId`, `aiConfidence`, `assignedBy`, `aiReasoning`
  - [x] 2.3 Include the project name if assigned (optional — query projects table or return projectId only for simplicity)

- [x] Task 3: Verify build and existing behavior preservation (AC: #4, #5, #6)
  - [x] 3.1 Run `pnpm build` — zero TypeScript errors
  - [x] 3.2 Verify that STT failure still returns `STT_FAILED` error response (unchanged)
  - [x] 3.3 Verify that Summary failure still sets status to "partial" (unchanged)
  - [x] 3.4 Verify that Assignment failure does NOT change status (stays "done") and response is still successful
  - [x] 3.5 Verify that `getAuthContext()` is still called at the top of the handler (unchanged from current code)
  - [x] 3.6 Verify no API keys in client-accessible code (S1-NFR7)

## Dev Notes

### Critical Architecture Patterns

**1. Pipeline Extension Point — Decision 1 from architecture-sprint1.md:**

The pipeline is extended sequentially AFTER the successful summary step. The flow becomes:

```
STT → LLM #1 (Summary, VOICE_SUMMARY_SYSTEM_PROMPT) → LLM #2 (Assignment, ASSIGNMENT_SYSTEM_PROMPT) → DB
```

**Key Rule:** Assignment failure MUST NOT change the voice message status. The message is "done" after successful summary. Assignment is a best-effort enrichment step.
[Source: architecture-sprint1.md#Decision 1 — Separate LLM Calls]

**2. Pipeline Error Handling — Architecture Pattern:**

```
Upload → STT (fail = "error") → Summary (fail = "partial") → Assignment (fail = silent, Inbox)
```

| Stage | On Failure | Status | User Sees |
|-------|-----------|--------|-----------|
| Upload | Abort pipeline | no entry | "Upload fehlgeschlagen" |
| STT | Stop, audio saved | "error" | "Spracherkennung hat nicht funktioniert" |
| Summary | Stop, transcript saved | "partial" | "Zusammenfassung konnte nicht erstellt werden" |
| **Assignment** | **Continue**, projectId=null | **"done"** | Message appears in Inbox |

[Source: architecture-sprint1.md#Pipeline Error Handling Pattern]

**3. Confidence Routing Logic:**

```typescript
// Confidence >= 0.7 → auto-assign
if (assignment.confidence >= 0.7) {
  // SET: projectId, assignedBy="ai", assignedAt=now()
}
// Confidence < 0.7 → Inbox
else {
  // projectId stays null → message appears in Inbox
  // aiSuggestedProjectId still stores the AI's best guess
}
```

The threshold 0.7 is from the PRD (S1-FR8) and epics. The ASSIGNMENT_SYSTEM_PROMPT confidence scale has >=0.8 as "clear match" and 0.5-0.8 as "probable", so 0.7 sits correctly in the "probable" range as the auto-assign cutoff.
[Source: prd-sprint1.md#S1-FR8, epics.md#Story 9.2]

**4. Prompt Version String Format:**

Combined version string stored in `llmPromptVersion` DB field:

```typescript
// After summary only (if assignment fails or is skipped):
`summary:${VOICE_SUMMARY_VERSION}`
// → "summary:SUMMARY_V1"

// After both summary + assignment:
`summary:${VOICE_SUMMARY_VERSION},assignment:${ASSIGNMENT_PROMPT_VERSION}`
// → "summary:SUMMARY_V1,assignment:ASSIGNMENT_V1"
```

[Source: architecture-sprint1.md#Prompt Versioning Pattern]

### Existing Code — Exact Insertion Point

The assignment step is inserted in `app/api/voice/route.ts` AFTER the successful summary update (currently lines 88-111). Here's the exact structure:

```typescript
// CURRENT CODE (lines 82-111):
// LLM: Mistral summarization
const llmStart = Date.now();
try {
  const summary = await summarize(sttResult.transcript);
  const llmDurationMs = Date.now() - llmStart;

  const [final] = await db
    .update(voiceMessages)
    .set({
      summary,
      llmProvider: "mistral-small",
      llmDurationMs,
      status: "done",
      processedAt: sql`now()`,
      llmPromptVersion: `summary:${VOICE_SUMMARY_VERSION}`, // ← ADD THIS (Task 1.6)
    })
    .where(eq(voiceMessages.id, entry.id))
    .returning();

  // ═══════════════════════════════════════════════════════
  // NEW: Assignment step (Task 1.3-1.5) — INSERT HERE
  // ═══════════════════════════════════════════════════════
  // ... assignment try/catch ...

  return successResponse({
    // ... extended response with assignment fields (Task 2)
  });
}
```

### Existing Code Patterns to Follow

**Assignment function signature (from lib/ai/assignment.ts:108-112):**
```typescript
export async function assignToProject(
  transcript: string,
  userId: string,
  companyId: string,
): Promise<AssignmentResult>
// Returns: { projectId: string | null, confidence: number, reasoning: string }
```

**Error handling pattern for LLMValidationError (from current route.ts:112-127):**
```typescript
if (llmError instanceof LLMValidationError) {
  console.error("mistral_llm_validation_error", {
    type: llmError.zodErrors ? "schema_mismatch" : "invalid_json",
    message: llmError.message,
    rawContent: llmError.rawContent,
    zodErrors: llmError.zodErrors?.issues.map((i) => ({ path: i.path, code: i.code, message: i.message })),
    voiceMessageId: entry.id,
  });
} else {
  console.error("mistral_llm_error", {
    name: llmError instanceof Error ? llmError.name : "unknown",
    message: llmError instanceof Error ? llmError.message : String(llmError),
    voiceMessageId: entry.id,
  });
}
```

Use the SAME pattern for assignment errors, but with different log keys (e.g., `"assignment_llm_validation_error"`, `"assignment_llm_error"`).

**DB update pattern (from current route.ts:88-98):**
```typescript
const [final] = await db
  .update(voiceMessages)
  .set({
    // fields to update
  })
  .where(eq(voiceMessages.id, entry.id))
  .returning();
```

### Critical Anti-Patterns to Avoid

```typescript
// ❌ WRONG: Assignment failure changes status
} catch (assignmentError) {
  await db.update(voiceMessages).set({ status: "error" }).where(...);
}
// ✅ CORRECT: Status stays "done" — assignment failure is silent
} catch (assignmentError) {
  console.error("assignment_failed", { voiceMessageId: entry.id, ... });
  // No status change. Message goes to Inbox (projectId=null).
}

// ❌ WRONG: Assignment failure returns error response to user
} catch (assignmentError) {
  return errorResponse("ASSIGNMENT_FAILED", "Zuordnung fehlgeschlagen");
}
// ✅ CORRECT: Still return success — transcript + summary are the valuable result
} catch (assignmentError) {
  // Continue to return successResponse with transcript + summary
}

// ❌ WRONG: Using a different confidence threshold than 0.7
if (assignment.confidence >= 0.5) { // NO — architecture says 0.7
// ✅ CORRECT: Use 0.7 as the auto-assign threshold
if (assignment.confidence >= 0.7) {

// ❌ WRONG: Logging PII (transcript content) in error handler
console.error("assignment_failed", { transcript, userId });
// ✅ CORRECT: Only voiceMessageId and error type
console.error("assignment_failed", { voiceMessageId: entry.id, type: "..." });

// ❌ WRONG: Setting assignedBy for low-confidence (Inbox) messages
if (assignment.confidence < 0.7) {
  set({ assignedBy: "ai" });  // NO — assignedBy is null for Inbox messages
}
// ✅ CORRECT: assignedBy="ai" only when actually auto-assigned
if (assignment.confidence >= 0.7) {
  set({ projectId: assignment.projectId, assignedBy: "ai", assignedAt: sql`now()` });
}

// ❌ WRONG: Forgetting aiSuggestedProjectId for low-confidence messages
// The AI's best guess should ALWAYS be stored, even when confidence < 0.7
// ✅ CORRECT: Always set aiSuggestedProjectId to assignment.projectId
set({ aiSuggestedProjectId: assignment.projectId });  // Even if projectId stays null in the main field

// ❌ WRONG: Hardcoding prompt version strings
set({ llmPromptVersion: "v1" });
// ✅ CORRECT: Use imported constants
import { VOICE_SUMMARY_VERSION, ASSIGNMENT_PROMPT_VERSION } from "@/lib/ai/prompts";
set({ llmPromptVersion: `summary:${VOICE_SUMMARY_VERSION},assignment:${ASSIGNMENT_PROMPT_VERSION}` });

// ❌ WRONG: Calling assignToProject when transcript is empty
// The current code already returns early for empty transcripts (lines 70-80)
// So this is naturally prevented — assignment only runs after successful summary
```

### Scope Boundary — What This Story Does NOT Do

| Out of Scope | Belongs To |
|-------------|------------|
| Creating the `assignToProject()` function | Story 9.1 (DONE) |
| Creating the `ASSIGNMENT_SYSTEM_PROMPT` | Story 9.1 (DONE) |
| Displaying assignment info on voice message cards | Story 9.3 |
| Manual reassignment from Inbox | Story 10.2 |
| Inbox page and unassigned messages list | Story 10.1 |
| Client-side processing indicator for assignment step | Future enhancement |
| Retry logic for failed assignments | Future enhancement |

### Project Structure Notes

**Modified files (ONLY ONE):**
- `app/api/voice/route.ts` — Add assignment step after summary, extend response

**New imports needed:**
```typescript
import { assignToProject } from "@/lib/ai/assignment";
import { VOICE_SUMMARY_VERSION, ASSIGNMENT_PROMPT_VERSION } from "@/lib/ai/prompts";
```

Note: `LLMValidationError` is already imported from `@/lib/ai/mistral` (line 6 of current route.ts).

**No new files needed.** All dependencies (assignment module, prompts, schema columns) are already in place from Story 9.1 and Story 8.1.

**Alignment with architecture:**
- Pipeline extension follows architecture-sprint1.md Decision 1 exactly
- Error handling follows Pipeline Error Handling Pattern exactly
- Prompt versioning follows Prompt Versioning Pattern exactly
- companyId already flows from getAuthContext() (Story 8.1)
[Source: architecture-sprint1.md#5. Project Structure & Boundaries — Modified Files]

### Previous Story Intelligence

**From Story 9.1 (Assignment Prompt & AI Module) — completed 2026-02-14:**
- `assignToProject(transcript, userId, companyId)` is exported from `lib/ai/assignment.ts`
- Returns `{ projectId: string | null, confidence: number, reasoning: string }`
- Throws `LLMValidationError` on invalid JSON or schema mismatch — catch block should handle this
- Throws plain `Error` on empty Mistral response
- Uses `mistral-small-latest` model with `temperature: 0` and `responseFormat: { type: "json_object" }`
- Validates returned projectId against active projects list (prevents hallucinated UUIDs)
- Early returns `{ projectId: null, confidence: 0.0, reasoning: "Keine aktiven Projekte vorhanden" }` if no active projects
- `getClient()` was exported from `lib/ai/mistral.ts`
- `assignmentResponseSchema` added to `lib/validations/voice.ts`
- `VOICE_SUMMARY_VERSION = "SUMMARY_V1"` and `ASSIGNMENT_PROMPT_VERSION = "ASSIGNMENT_V1"` in `lib/ai/prompts.ts`
- `formatRelativeTime` reused from `lib/format.ts` inside assignment context builder

**From Story 8.1 (Multi-Tenant Data Model) — completed:**
- voiceMessages schema already has all assignment columns (projectId, companyId, aiConfidence, assignedBy, aiReasoning, llmPromptVersion, assignedAt, aiSuggestedProjectId)
- `getAuthContext()` returns `{ userId, companyId, role }`
- Voice route already uses `getAuthContext()` and stores `companyId` on insert (line 12, 45)

**From Story 6.3 (Mistral LLM Summarization) — completed:**
- `summarize()` returns `VoiceSummary` object
- LLM error handling pattern with `LLMValidationError` established
- Status flow: "uploaded" → "transcribed" → "done" (or "error"/"partial" on failure)

### Git Intelligence

Recent commits (context for commit message style):
```
33f6e15 Implement AI project assignment prompt and module (Story 9.1)
c452748 Implement project list with activity stats and detail timeline (Story 8.3)
9dffe00 Fix Zod validation regression: use nullish() for optional project fields
c5f8c0f Implement project CRUD with Server Actions and form components (Story 8.2)
0c721ec Implement multi-tenant data model and auth context helper (Story 8.1)
```

**This story's commit should follow:** `"Extend voice pipeline with AI project assignment step (Story 9.2)"`

### Library/Framework Specifics

| Library | Version | Key Notes for This Story |
|---------|---------|--------------------------|
| Drizzle ORM | 0.45.1 | `db.update().set().where().returning()` — already used in route.ts |
| `drizzle-orm` | | `eq`, `sql` already imported in route.ts |
| Next.js | 16.1.6 | Route Handler pattern — already in place |
| `@mistralai/mistralai` | (installed) | Used via `assignToProject()` — no direct Mistral usage in this story |

### References

- [Source: architecture-sprint1.md#Decision 1 — Separate LLM Calls for Summary + Assignment]
- [Source: architecture-sprint1.md#Pipeline Error Handling Pattern]
- [Source: architecture-sprint1.md#Prompt Versioning Pattern]
- [Source: architecture-sprint1.md#5. Project Structure & Boundaries — Modified Files]
- [Source: prd-sprint1.md#S1-FR8 — Confidence routing >=0.7 auto, <0.7 Inbox]
- [Source: prd-sprint1.md#S1-FR23 — Prompt version tracking]
- [Source: prd-sprint1.md#S1-FR24 — Assignment metadata storage]
- [Source: prd-sprint1.md#S1-NFR1 — Pipeline <45s total latency]
- [Source: prd-sprint1.md#S1-NFR9 — Pipeline regression prevention]
- [Source: prd-sprint1.md#S1-NFR10 — Graceful degradation → Inbox]
- [Source: epics.md#Story 9.2 Acceptance Criteria]
- [Source: 9-1-assignment-prompt-and-ai-module.md — assignToProject() API, error handling, LLMValidationError]
- [Source: app/api/voice/route.ts — Current pipeline code (insertion point)]
- [Source: lib/ai/prompts.ts — VOICE_SUMMARY_VERSION, ASSIGNMENT_PROMPT_VERSION constants]
- [Source: lib/db/schema.ts — voiceMessages assignment columns]

## Dev Agent Record

### Agent Model Used

Claude Opus 4.6

### Debug Log References

- `pnpm build` passed with zero TypeScript errors (Turbopack, 9.1s compile)

### Completion Notes List

- Extended voice pipeline with AI project assignment step after successful summary
- Assignment step calls `assignToProject(transcript, userId, companyId)` from `lib/ai/assignment.ts` (Story 9.1)
- Confidence routing: >= 0.7 auto-assigns to project (`assignedBy: "ai"`, `assignedAt: now()`), < 0.7 stays in Inbox (`projectId: null`)
- `aiSuggestedProjectId` always stores AI's best guess regardless of confidence
- `llmPromptVersion` tracks both prompt versions: `"summary:SUMMARY_V1,assignment:ASSIGNMENT_V1"` (or just `"summary:SUMMARY_V1"` if assignment fails)
- Assignment failure is silent: status stays "done", no error response to user, STT + Summary results preserved (S1-NFR9)
- Error logging follows established pattern with `LLMValidationError` distinction for research analysis
- Success response extended with `projectId`, `aiConfidence`, `assignedBy`, `aiReasoning`
- No test framework configured in project; validation via TypeScript build + code review
- All existing pipeline behavior preserved: STT failure → "error", Summary failure → "partial", unchanged error responses

### Senior Developer Review (AI)

**Reviewer:** Sempre (via adversarial code review workflow)
**Date:** 2026-02-14
**Outcome:** Approved (after fixes applied)

**Issues Found:** 0 High, 2 Medium, 3 Low
**Issues Fixed:** 2 Medium, 2 Low (auto-fixed)

**Fixes Applied:**
- M1: Replaced `Record<string, unknown>` with typed inline object in assignment DB update — Drizzle now validates column names at compile time
- M2: Added `.returning()` to assignment DB update for consistency with summary update pattern
- L1: Added `// S1-FR8 threshold` comment on confidence threshold magic number
- L3: Removed unused `aiSuggestedProjectId` from local `assignmentResult` variable (only stored in DB, not returned in response)

**Remaining (accepted):**
- L2: `rawContent` in assignment error log may contain business context — consistent with existing summary error handler pattern, accepted as-is

**AC Validation:** All 6 Acceptance Criteria verified as implemented.
**Task Audit:** All 15 subtasks verified — no false `[x]` markings.
**Build:** `pnpm build` passed with zero TypeScript errors after fixes.

### Change Log

- 2026-02-14: Implemented assignment step in voice pipeline (Story 9.2)
- 2026-02-14: Code review fixes — type safety, .returning(), threshold comment (Review)

### File List

- `app/api/voice/route.ts` — Modified: added assignment step after summary, extended response payload
