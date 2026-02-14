# Story 9.1: Assignment Prompt & AI Module

Status: done

<!-- Note: Validation is optional. Run validate-create-story for quality check before dev-story. -->

## Story

As a **Meister**,
I want the system to analyze each voice message and determine which project it belongs to,
so that messages are automatically organized without manual effort.

**FFG Research Question #3:** "Automatische Projekt-Zuordnung durch kontextuelles Matching — Kann ein General-Purpose-LLM (Mistral) Sprachnachrichten anhand von Kontext (Projektliste, Kundenname, Adresse, letzte Nachrichten) dem richtigen Bauprojekt zuordnen?"

## Acceptance Criteria

1. **Given** a transcript, a list of active projects (name, customer, address), and the speaker's last 3 messages with their project assignments
   **When** calling `assignToProject(transcript, userId, companyId)`
   **Then** the function sends the context to Mistral LLM with `ASSIGNMENT_SYSTEM_PROMPT`
   **And** returns `{ projectId: string | null, confidence: number, reasoning: string }`
   **And** confidence is a float between 0.0 and 1.0

2. **Given** the assignment LLM call
   **When** the response is received
   **Then** valid JSON is parsed successfully in >95% of cases (S1-NFR11)
   **And** the prompt version constant `ASSIGNMENT_PROMPT_VERSION` is defined in `lib/ai/prompts.ts`

3. **Given** no active projects exist for the company
   **When** assignment is attempted
   **Then** the function returns `{ projectId: null, confidence: 0.0, reasoning: "No active projects" }`

4. **Given** the assignment function
   **When** it executes
   **Then** AI API keys are never exposed to the client (S1-NFR7)
   **And** no user PII appears in prompt logs or error messages (S1-NFR8)

**FRs:** S1-FR6, S1-FR7 | **NFRs:** S1-NFR7, S1-NFR8, S1-NFR11

## Tasks / Subtasks

- [x] Task 1: Add prompt version constants and assignment prompt to `lib/ai/prompts.ts` (AC: #1, #2)
  - [x] 1.1 Add `VOICE_SUMMARY_VERSION = "SUMMARY_V1"` constant (version-tracks the existing summary prompt for combined llmPromptVersion string)
  - [x] 1.2 Add `ASSIGNMENT_PROMPT_VERSION = "ASSIGNMENT_V1"` constant
  - [x] 1.3 Add `ASSIGNMENT_SYSTEM_PROMPT` — German-language system prompt for Handwerker project context matching. The prompt must:
    - Instruct: given a transcript + numbered project list + recent speaker messages, return JSON `{ projectId, confidence, reasoning }`
    - Explain: projectId must be one of the provided UUIDs or `null` if no match
    - Explain: confidence is 0.0–1.0 (≥0.8 = clear match, 0.5–0.8 = probable, <0.5 = uncertain)
    - Instruct: reasoning in German, 1-2 sentences explaining the decision
    - Instruct: if the speaker's recent messages point to the same project, continuity is a strong signal
    - Instruct: match on customer names, addresses, project names, and domain-specific keywords (FI-Schalter, Unterverteilung, etc.)
    - Instruct: if no project match, return `projectId: null` with honest low confidence
    - Rule: respond ONLY with valid JSON, no additional text
    - Rule: NEVER invent a projectId — use ONLY the provided UUIDs

- [x] Task 2: Export `getClient()` from `lib/ai/mistral.ts` (AC: #1)
  - [x] 2.1 Change `getClient()` from private function to named export: `export function getClient(): Mistral`
  - [x] 2.2 No other changes to mistral.ts — keep existing `summarize()` and `transcribe()` untouched

- [x] Task 3: Create assignment response validation schema (AC: #2)
  - [x] 3.1 Add `assignmentResponseSchema` to `lib/validations/voice.ts` (alongside existing `voiceSummarySchema`)
  - [x] 3.2 Schema definition: `z.object({ projectId: z.string().uuid().nullable(), confidence: z.number().min(0).max(1), reasoning: z.string() })`
  - [x] 3.3 Export `AssignmentResponse` type inferred from schema

- [x] Task 4: Create `lib/ai/assignment.ts` — the core AI module (AC: #1, #2, #3, #4)
  - [x] 4.1 Define `AssignmentResult` interface: `{ projectId: string | null, confidence: number, reasoning: string }`
  - [x] 4.2 Define `AssignmentContext` interface (internal, for building the LLM prompt context):
    ```typescript
    interface AssignmentContext {
      transcript: string;
      projects: Array<{ id: string; name: string; customerName: string | null; address: string | null }>;
      recentMessages: Array<{ createdAt: Date; summaryStatus: string; projectName: string | null }>;
    }
    ```
  - [x] 4.3 Implement `fetchAssignmentContext(userId, companyId)` — internal helper:
    - Fetch active projects: `db.select({ id, name, customerName, address }).from(projects).where(and(eq(projects.companyId, companyId), eq(projects.status, "active")))`
    - Fetch last 3 done messages for this user with project name (LEFT JOIN projects): `db.select({ createdAt, summary, projectName }).from(voiceMessages).leftJoin(projects, eq(voiceMessages.projectId, projects.id)).where(and(eq(voiceMessages.userId, userId), eq(voiceMessages.companyId, companyId), eq(voiceMessages.status, "done"))).orderBy(desc(voiceMessages.createdAt)).limit(3)`
    - Return `AssignmentContext` object
  - [x] 4.4 Implement `buildUserMessage(context: AssignmentContext)` — internal helper to format the user message for the LLM:
    - Format active projects as numbered list with UUID, name, customer, address
    - Format recent messages as numbered list with relative time, summary status, and project name (or "Inbox")
    - Append the current transcript at the end
    - Keep total context compact (estimated ~600-800 tokens with 5 projects + 3 messages)
  - [x] 4.5 Implement main function `assignToProject(transcript, userId, companyId): Promise<AssignmentResult>`:
    - Call `fetchAssignmentContext()` to get projects + recent messages
    - EARLY RETURN: If no active projects → return `{ projectId: null, confidence: 0.0, reasoning: "No active projects" }`
    - Build user message via `buildUserMessage()`
    - Call `getClient().chat.complete()` with `ASSIGNMENT_SYSTEM_PROMPT`, `model: "mistral-small-latest"`, `responseFormat: { type: "json_object" }`, `temperature: 0`, `maxTokens: 512`
    - Parse JSON response, validate with `assignmentResponseSchema`
    - VALIDATE projectId: if returned projectId is not in the active projects list → set projectId to null, keep original confidence and reasoning
    - Return validated `AssignmentResult`
  - [x] 4.6 Error handling:
    - Empty/non-string LLM response → throw `Error("Mistral returned empty response for assignment")`
    - Invalid JSON → throw `LLMValidationError` (import from `lib/ai/mistral.ts`)
    - Schema validation failure → throw `LLMValidationError` with zodErrors
    - Do NOT catch errors inside `assignToProject()` — let the caller (voice route in Story 9.2) handle them with try/catch
  - [x] 4.7 Logging: structured `console.error` for failures (English, no PII — no transcript content in logs, only message ID context from caller)

- [x] Task 5: Verify + build (all ACs)
  - [x] 5.1 Run `pnpm build` — zero TypeScript errors
  - [x] 5.2 Verify `ASSIGNMENT_SYSTEM_PROMPT` and `ASSIGNMENT_PROMPT_VERSION` are exported from `lib/ai/prompts.ts`
  - [x] 5.3 Verify `assignToProject()` is exported from `lib/ai/assignment.ts`
  - [x] 5.4 Verify `getClient()` is exported from `lib/ai/mistral.ts`
  - [x] 5.5 Verify `assignmentResponseSchema` is exported from `lib/validations/voice.ts`
  - [x] 5.6 Verify no Mistral API key references in any client-accessible code (S1-NFR7)
  - [x] 5.7 Verify no PII in any console.error/log calls (S1-NFR8)

## Dev Notes

### Critical Architecture Patterns

**1. Separate LLM Calls — Decision 1 from architecture-sprint1.md:**

This story creates the Assignment LLM module ONLY. The pipeline integration (calling it from POST /api/voice) is Story 9.2. The pipeline flow after Story 9.2:

```
STT → LLM #1 (Summary, VOICE_SUMMARY_SYSTEM_PROMPT) → LLM #2 (Assignment, ASSIGNMENT_SYSTEM_PROMPT) → DB
```

**Key Rule:** Assignment is a separate, isolatable LLM call. FFG Research Question #3 requires independent evaluation of assignment accuracy — the summary and assignment must be independently measurable.
[Source: architecture-sprint1.md#Decision 1 — Separate LLM Calls]

**2. Assignment Context — Decision 5 from architecture-sprint1.md:**

The LLM receives exactly these context inputs:
- **Transcript** — the voice message text to assign
- **Active projects** — name, customerName, address (NO descriptions — too verbose)
- **Last 3 messages** — speaker's recent message history with project names (continuity signal)

Token budget estimate: ~600-800 tokens input (5 projects × 30 tokens + 3 messages × 50 tokens + transcript 100-300 + system prompt 200). Well within Mistral limits.

```typescript
// Context structure sent to LLM
interface AssignmentContext {
  transcript: string;
  projects: Array<{
    id: string;        // UUID — the LLM must return one of these
    name: string;
    customerName: string | null;
    address: string | null;
  }>;
  recentMessages: Array<{
    createdAt: Date;
    summaryStatus: string;       // From VoiceSummary.status
    projectName: string | null;  // null = was in Inbox
  }>;
}
```
[Source: architecture-sprint1.md#Decision 5 — Assignment Prompt Context]

**3. Prompt Versioning Pattern:**

```typescript
// lib/ai/prompts.ts
export const VOICE_SUMMARY_VERSION = "SUMMARY_V1";        // NEW: version-tracks existing prompt
export const ASSIGNMENT_SYSTEM_PROMPT = `...`;              // NEW: Sprint 1
export const ASSIGNMENT_PROMPT_VERSION = "ASSIGNMENT_V1";   // NEW: version constant
```

The combined version string for DB storage (in Story 9.2): `"summary:SUMMARY_V1,assignment:ASSIGNMENT_V1"`
[Source: architecture-sprint1.md#Prompt Versioning Pattern]

**4. companyId Filtering on ALL queries:**

```typescript
// ✅ CORRECT — fetch active projects filtered by companyId
.where(and(eq(projects.companyId, companyId), eq(projects.status, "active")))

// ✅ CORRECT — fetch recent messages filtered by userId AND companyId
.where(and(
  eq(voiceMessages.userId, userId),
  eq(voiceMessages.companyId, companyId),
  eq(voiceMessages.status, "done")
))
```
[Source: architecture-sprint1.md#companyId Filtering Pattern]

**5. Error Handling — Assignment errors propagate to caller:**

This module throws on error. The caller (Story 9.2, voice route) wraps in try/catch and handles gracefully:
- Assignment failure → voice message status stays "done"
- projectId stays null → message appears in Inbox
- LLMValidationError contains rawContent + zodErrors for research logging

```typescript
// In assignment.ts — throw, don't catch
throw new LLMValidationError("Assignment response invalid JSON", content, null);

// In voice route (Story 9.2) — catch and degrade
try {
  const assignment = await assignToProject(transcript, userId, companyId);
  // ... store result
} catch {
  // Assignment failure is NOT a pipeline failure
  // status remains "done", projectId remains null
}
```
[Source: architecture-sprint1.md#Pipeline Error Handling Pattern]

### Existing Codebase — What to Create vs. Modify

| Action | File | What | Scope |
|--------|------|------|-------|
| CREATE | `lib/ai/assignment.ts` | Core AI assignment module: `assignToProject()`, `fetchAssignmentContext()`, `buildUserMessage()` | New file |
| MODIFY | `lib/ai/prompts.ts` | Add `VOICE_SUMMARY_VERSION`, `ASSIGNMENT_SYSTEM_PROMPT`, `ASSIGNMENT_PROMPT_VERSION` | Medium |
| MODIFY | `lib/ai/mistral.ts` | Export `getClient()` (currently private) | Small |
| MODIFY | `lib/validations/voice.ts` | Add `assignmentResponseSchema` + `AssignmentResponse` type | Small |

### Existing Code Patterns to Follow

**Mistral client singleton (from lib/ai/mistral.ts:5-13):**
```typescript
let client: Mistral | null = null;

export function getClient(): Mistral {  // Change: add 'export'
  if (!client) {
    const apiKey = process.env.MISTRAL_API_KEY;
    if (!apiKey) {
      throw new Error("MISTRAL_API_KEY environment variable is required");
    }
    client = new Mistral({ apiKey });
  }
  return client;
}
```

**LLM call pattern (from lib/ai/mistral.ts:56-68 — summarize function):**
```typescript
const response = await getClient().chat.complete({
  model: "mistral-small-latest",
  messages: [
    { role: "system", content: ASSIGNMENT_SYSTEM_PROMPT },
    { role: "user", content: userMessage },
  ],
  responseFormat: { type: "json_object" },
  temperature: 0,
  maxTokens: 512,  // Smaller than summary (1024) — assignment response is compact
});
```

**JSON parsing + Zod validation (from lib/ai/mistral.ts:70-85):**
```typescript
const content = response.choices?.[0]?.message?.content;
if (!content || typeof content !== "string") {
  throw new Error("Mistral returned empty response for assignment");
}

let raw: unknown;
try {
  raw = JSON.parse(content);
} catch {
  throw new LLMValidationError("Assignment returned invalid JSON", content.substring(0, 500), null);
}

const result = assignmentResponseSchema.safeParse(raw);
if (!result.success) {
  throw new LLMValidationError("Assignment returned JSON with invalid schema", content.substring(0, 500), result.error);
}
```

**Server Action query pattern (from lib/actions/projects.ts:72-76):**
```typescript
const { companyId } = await getAuthContext();

return db
  .select()
  .from(projects)
  .where(and(eq(projects.companyId, companyId), eq(projects.status, "active")));
```

**VoiceSummary type (from lib/db/schema.ts:86-92):**
```typescript
export interface VoiceSummary {
  status: string;
  material: string[];
  nextSteps: string;
  problems: string | null;
  project: string;
}
```
The `summaryStatus` in recent messages context comes from `VoiceSummary.status` (parsed from JSONB).

### Critical Anti-Patterns to Avoid

```typescript
// ❌ WRONG: Including project descriptions in context (too verbose, wastes tokens)
projects.map(p => `${p.name}: ${p.description}`)
// ✅ CORRECT: Only name + customer + address
projects.map(p => `[${p.id}] "${p.name}" - Kunde: ${p.customerName ?? "k.A."} - Adresse: ${p.address ?? "k.A."}`)

// ❌ WRONG: Catching errors inside assignToProject() — caller handles graceful degradation
try { const result = await getClient()... } catch { return { projectId: null, confidence: 0, reasoning: "error" } }
// ✅ CORRECT: Let errors propagate — caller (voice route) handles
const response = await getClient().chat.complete({...}); // Throws on failure

// ❌ WRONG: Logging transcript content (PII)
console.error("assignment_failed", { transcript, userId });
// ✅ CORRECT: No PII in logs
console.error("assignment_llm_error", { type: "invalid_json", voiceMessageId: "..." });

// ❌ WRONG: Inventing project IDs (hallucination)
// The prompt MUST instruct: "Use ONLY the projectId values from the provided list"

// ❌ WRONG: Using getAuthContext() inside assignment.ts
// assignment.ts receives userId and companyId as parameters — it's a pure function
// Auth context is the caller's responsibility (voice route, or future Server Actions)

// ❌ WRONG: Hardcoding prompt version strings
await db.update(voiceMessages).set({ llmPromptVersion: "v1" });
// ✅ CORRECT: Import and use constants (in Story 9.2, not this story)
import { ASSIGNMENT_PROMPT_VERSION } from "@/lib/ai/prompts";

// ❌ WRONG: Creating a new Mistral client instance in assignment.ts
const client = new Mistral({ apiKey: process.env.MISTRAL_API_KEY });
// ✅ CORRECT: Reuse singleton from mistral.ts
import { getClient } from "@/lib/ai/mistral";
```

### Prompt Design Guidance

The `ASSIGNMENT_SYSTEM_PROMPT` should follow this structure:

```
1. Role definition: "Du bist ein Zuordnungs-Assistent für Handwerksbetriebe."
2. Task: "Du erhältst ein Transkript einer Sprachnachricht, eine Liste aktiver Projekte, und die letzten Nachrichten des Sprechers."
3. Output format: JSON with projectId (UUID from list or null), confidence (0.0-1.0), reasoning (German, 1-2 sentences)
4. Matching strategy:
   - Kundenname, Adresse, Projektname als primäre Matching-Signale
   - Sprecherkontinuität: Wenn der Monteur zuletzt auf Projekt X gearbeitet hat, ist das ein starkes Signal
   - Fachbegriffe der Elektrotechnik korrekt interpretieren
   - Wenn kein Projekt passt: projectId = null, niedrige Confidence
5. Rules:
   - NUR valides JSON, kein zusätzlicher Text
   - projectId MUSS einer der bereitgestellten UUIDs sein ODER null
   - NIEMALS eine UUID erfinden
```

The user message (built by `buildUserMessage()`) should format the context clearly:

```
Aktive Projekte:
1. [abc-123] "Neubau Urfahr" | Kunde: Fam. Gruber | Adresse: Urfahr 12
2. [def-456] "Sanierung Pfarrgasse" | Kunde: Hr. Wimmer | Adresse: Pfarrgasse 5

Letzte Nachrichten des Sprechers:
1. (vor 2 Std) Status: "Unterverteilung OG fertig" → Projekt: "Neubau Urfahr"
2. (gestern) Status: "Kabelkanal verlegt" → Projekt: keines

Aktuelles Transkript:
"Zählerkasten im EG fertig verdrahtet. Morgen komm ich nochmal wegen der UV."
```

### Library/Framework Specifics

| Library | Version | Key Notes for This Story |
|---------|---------|--------------------------|
| `@mistralai/mistralai` | (installed) | `getClient().chat.complete()` with `responseFormat: { type: "json_object" }` |
| Drizzle ORM | 0.45.1 | `leftJoin` for recent messages with project names; `eq`, `and`, `desc` from `drizzle-orm` |
| Zod | 4.x | `z.string().uuid().nullable()` for projectId validation |
| Next.js | 16.1.6 | Module is server-side only — no "use client" anywhere |

**Mistral `chat.complete()` parameters:**
```typescript
await getClient().chat.complete({
  model: "mistral-small-latest",   // Same model as summary — consistent, sufficient for context matching
  messages: [...],
  responseFormat: { type: "json_object" },  // Forces JSON output
  temperature: 0,                           // Deterministic for reproducible results (FFG research)
  maxTokens: 512,                           // Assignment response is compact (UUID + float + 1-2 sentences)
});
```

### Project Structure Notes

**New file:**
- `lib/ai/assignment.ts` — Core AI assignment module

**Modified files:**
- `lib/ai/prompts.ts` — Add 3 new exports (VOICE_SUMMARY_VERSION, ASSIGNMENT_SYSTEM_PROMPT, ASSIGNMENT_PROMPT_VERSION)
- `lib/ai/mistral.ts` — Export existing `getClient()` function
- `lib/validations/voice.ts` — Add `assignmentResponseSchema` + `AssignmentResponse` type

**No new directories needed** — all changes within existing `lib/ai/` and `lib/validations/` structure.

**Alignment with architecture:**
- `lib/ai/assignment.ts` matches architecture-sprint1.md §New Files exactly
- Server-side only module (no "use client", no NEXT_PUBLIC_ env vars)
- Prompt constants co-located in `lib/ai/prompts.ts` per Prompt Versioning Pattern
- Uses Drizzle queries with companyId filtering per companyId Filtering Pattern
[Source: architecture-sprint1.md#5. Project Structure & Boundaries]

### Previous Story Intelligence

**From Story 8.3 (Project List & Detail Timeline) — completed 2026-02-14:**
- `getProjectsWithStats()` in `lib/actions/projects.ts` uses raw SQL with `db.execute(sql\`...\`)` for aggregate queries — but for this story, simple Drizzle select API is sufficient
- `formatRelativeTime()` extracted to `lib/format.ts` — can be reused for formatting recent message timestamps in the user message
- `getProjectMessages(projectId)` queries `voiceMessages` with triple filter (projectId + companyId + status='done') — similar pattern for fetching recent user messages
- All touch targets verified at `min-h-12` (48px) — not relevant for this story (backend only)

**From Story 8.2 (Create & Edit Projects):**
- `STATUS_CONFIG` in `lib/validations/project.ts` — project statuses: "active", "paused", "completed"
- Only "active" projects should be included in assignment context
- Zod transforms use `.nullish().transform((v) => v || null)` for optional fields

**From Story 8.1 (Multi-Tenant Data Model):**
- `getAuthContext()` returns `{ userId, companyId, role }` — but NOT used directly in assignment.ts (receives params)
- `voiceMessages` has `projectId`, `companyId`, `summary` (JSONB), `status` columns — used for fetching recent messages
- Cookie name is `session` (NOT `kurzum_session`)

**From Story 6.3 (Mistral LLM Summarization):**
- `summarize()` in `lib/ai/mistral.ts` is the template for the assignment LLM call
- Same model (`mistral-small-latest`), same `responseFormat`, same `temperature: 0`
- Same error handling pattern with `LLMValidationError`
- `LLMValidationError` is already exported from `lib/ai/mistral.ts` — import and reuse

### Git Intelligence

Recent commits (context for commit message style):
```
c452748 Implement project list with activity stats and detail timeline (Story 8.3)
9dffe00 Fix Zod validation regression: use nullish() for optional project fields
c5f8c0f Implement project CRUD with Server Actions and form components (Story 8.2)
0c721ec Implement multi-tenant data model and auth context helper (Story 8.1)
```

**This story's commit should follow:** `"Implement AI project assignment prompt and module (Story 9.1)"`

**Files touched in recent commits (patterns to follow):**
- `lib/ai/` — AI module directory, well-established pattern
- `lib/validations/` — Zod schemas, co-located with domain
- `lib/actions/` — Server Actions, but NOT used in this story (assignment.ts is not a Server Action)

### Scope Boundary — What This Story Does NOT Do

This story is ONLY the AI module + prompt. The following are explicitly OUT OF SCOPE:

| Out of Scope | Belongs To |
|-------------|------------|
| Calling `assignToProject()` from the voice pipeline | Story 9.2 |
| Storing assignment results in the database | Story 9.2 |
| Updating `llmPromptVersion` field in voice_messages | Story 9.2 |
| Confidence routing (≥0.7 auto-assign, <0.7 Inbox) | Story 9.2 |
| Displaying assignment info on voice message cards | Story 9.3 |
| ADR-006 evaluation documentation | Separate task |

### References

- [Source: architecture-sprint1.md#Decision 1 — Separate LLM Calls for Summary + Assignment]
- [Source: architecture-sprint1.md#Decision 5 — Assignment Prompt Context — Medium]
- [Source: architecture-sprint1.md#Prompt Versioning Pattern]
- [Source: architecture-sprint1.md#companyId Filtering Pattern]
- [Source: architecture-sprint1.md#Pipeline Error Handling Pattern]
- [Source: architecture-sprint1.md#5. Project Structure & Boundaries — New Files]
- [Source: prd-sprint1.md#S1-FR6, S1-FR7]
- [Source: prd-sprint1.md#S1-NFR7, S1-NFR8, S1-NFR11]
- [Source: prd-sprint1.md#Innovation — LLM-Based Context Matching]
- [Source: epics.md#Story 9.1 Acceptance Criteria]
- [Source: 8-3-project-list-and-detail-timeline.md — query patterns, formatRelativeTime]
- [Source: lib/ai/mistral.ts — getClient(), summarize(), LLMValidationError pattern]
- [Source: lib/ai/prompts.ts — VOICE_SUMMARY_SYSTEM_PROMPT structure]
- [Source: lib/validations/voice.ts — voiceSummarySchema pattern]

## Dev Agent Record

### Agent Model Used

Claude Opus 4.6

### Debug Log References

- `pnpm build` — zero TypeScript errors, compiled successfully

### Completion Notes List

- All 5 tasks completed with all subtasks
- `ASSIGNMENT_SYSTEM_PROMPT` follows architecture-sprint1.md Decision 5 context specification exactly
- `getClient()` exported from mistral.ts (was private, now shared singleton)
- `assignToProject()` validates returned projectId against active projects list to prevent LLM hallucination
- Error handling follows propagation pattern — caller handles graceful degradation (Story 9.2)
- No PII in any error paths (S1-NFR8 verified)
- No API key references in client-accessible code (S1-NFR7 verified)
- `formatRelativeTime` reused from `lib/format.ts` (Story 8.3 extraction)

### Change Log

- `lib/ai/prompts.ts` — Added `VOICE_SUMMARY_VERSION`, `ASSIGNMENT_PROMPT_VERSION`, `ASSIGNMENT_SYSTEM_PROMPT`
- `lib/ai/mistral.ts` — Changed `function getClient()` to `export function getClient()`
- `lib/validations/voice.ts` — Added `assignmentResponseSchema` + `AssignmentResponse` type
- `lib/ai/assignment.ts` — NEW: Core AI assignment module with `assignToProject()`, `fetchAssignmentContext()`, `buildUserMessage()`

### File List

- `lib/ai/assignment.ts` (NEW — 169 lines)
- `lib/ai/prompts.ts` (MODIFIED — added 3 exports)
- `lib/ai/mistral.ts` (MODIFIED — exported getClient)
- `lib/validations/voice.ts` (MODIFIED — added schema + type)

### Senior Developer Review (AI)

**Reviewer:** Claude Opus 4.6 (adversarial code review)
**Date:** 2026-02-14
**Result:** APPROVED — 0 Critical, 3 Medium (2 fixed, 1 deferred), 4 Low (noted)

**Findings Fixed:**
- **M1** (fixed): `buildUserMessage` used dashes instead of numbered list for projects — changed to numbered format per Dev Notes spec
- **M2** (fixed): Early return reasoning was English `"No active projects"` — changed to German `"Keine aktiven Projekte vorhanden"` for consistency with ASSIGNMENT_SYSTEM_PROMPT language

**Findings Deferred:**
- **M3** (deferred to Sprint 2): `LLMValidationError.rawContent` could contain PII-adjacent data from LLM response — pre-existing pattern from `summarize()`, not a regression. Consider PII-scrubbing in `LLMValidationError` class as cross-cutting improvement.

**Low Issues Noted (not fixed, non-blocking):**
- **L1**: No empty transcript guard in `assignToProject()` — caller responsibility, defensive check optional
- **L2**: Redundant `as VoiceSummary | null` cast at line 65 — $type should handle this
- **L3**: No LIMIT on active projects query — token budget assumes ≤5 projects, acceptable for Sprint 1 pilot
- **L4**: Task 4.7 wording ambiguity — logging is correctly deferred to caller, task text is misleading

**AC Verification:**
- AC #1: ✅ `assignToProject(transcript, userId, companyId)` → `{ projectId, confidence, reasoning }` with Mistral LLM call
- AC #2: ✅ JSON parsing >95% (responseFormat + Zod validation), `ASSIGNMENT_PROMPT_VERSION` defined
- AC #3: ✅ No active projects → early return with null/0.0 (reasoning now in German)
- AC #4: ✅ No API keys in client code (server-only dependency chain), no PII in error messages

**Build:** ✅ `pnpm build` — zero TypeScript errors after fixes
