# Story 6.3: Mistral LLM Summarization

Status: done

<!-- Note: Validation is optional. Run validate-create-story for quality check before dev-story. -->

## Story

As a **developer + researcher**,
I want voice transcripts summarized into structured JSON by Mistral LLM with documented prompt engineering evaluation,
So that unstructured voice messages become actionable project updates and FFG research requirements are met.

**FFG Forschungsfrage #2:** "Wie können unstrukturierte Sprachnachrichten semantisch in strukturierte Zusammenfassungen transformiert werden?"

## Acceptance Criteria

1. **Given** a voice message has been transcribed (Story 6.2, status `"transcribed"`), **When** the `/api/voice` route processes it, **Then** `lib/ai/mistral.ts` calls Mistral chat/completions API with the transcript + system prompt, the resulting `VoiceSummary` JSON is stored in `voice_messages.summary` (JSONB), `llmProvider` is set to `"mistral-small"`, `llmDurationMs` records processing time, `status` is set to `"done"`, and `processedAt` is set to `now()`.

2. **Given** the LLM returns a response, **When** the summary is parsed, **Then** the output conforms to the `VoiceSummary` interface: `{ status: string, material: string[], nextSteps: string, problems: string | null, project: string }`, validated by Zod before DB storage.

3. **Given** the system prompt is designed for Handwerker context, **When** multiple prompt variants are tested against real transcripts, **Then** the winning prompt achieves >=85% semantic correctness across test scenarios (clear, dialect, domain terms), the winning prompt is stored in `lib/ai/prompts.ts` (version-controlled), and results are documented in `docs/research/llm-summarization-evaluation-2026-02.md`.

4. **Given** the LLM API is unavailable or returns invalid JSON, **When** the failure occurs, **Then** the transcript is preserved (never lost), `voice_messages.status` is set to `"partial"` (transcript exists but no summary), the error is logged (English, structured, no PII), and the user sees step 2 complete + "Zusammenfassung konnte nicht erstellt werden" (German, warm tone).

5. **Given** the LLM returns valid JSON but with unexpected structure, **When** Zod validation fails, **Then** the raw LLM response is logged for research analysis, the status is set to `"partial"`, and the transcript is still accessible.

## Tasks / Subtasks

- [x] **Task 1: Create summarize() function in lib/ai/mistral.ts** (AC: #1, #2)
  - [x] 1.1 Add `summarize(transcript: string): Promise<VoiceSummary>` function to existing `lib/ai/mistral.ts`
  - [x] 1.2 Use `getClient().chat.complete()` with `model: "mistral-small-latest"` and `responseFormat: { type: "json_object" }`
  - [x] 1.3 Parse response: `JSON.parse(response.choices[0].message.content)` then validate with Zod v4 schema
  - [x] 1.4 Return typed `VoiceSummary` matching the interface in `lib/db/schema.ts`
  - [x] 1.5 Set `temperature: 0` for deterministic output, `maxTokens: 1024`

- [x] **Task 2: Create system prompt in lib/ai/prompts.ts** (AC: #1, #3)
  - [x] 2.1 Create new file `lib/ai/prompts.ts` with `VOICE_SUMMARY_SYSTEM_PROMPT` constant
  - [x] 2.2 Prompt must: instruct Handwerker context, require JSON output matching VoiceSummary schema, handle German/Austrian dialect input
  - [x] 2.3 Include field descriptions: `status` = short progress summary, `material` = materials mentioned (empty array if none), `nextSteps` = next actions described, `problems` = issues/blockers (null if none), `project` = project/customer reference (infer from context)
  - [x] 2.4 Add prompt version comment for traceability (e.g., `// v1 — 2026-02`)

- [x] **Task 3: Create Zod validation schema for VoiceSummary** (AC: #2, #5)
  - [x] 3.1 Create validation schema in `lib/validations/voice.ts` (file already exists) using Zod v4
  - [x] 3.2 Schema fields: `status: z.string()`, `material: z.array(z.string())`, `nextSteps: z.string()`, `problems: z.string().nullable()`, `project: z.string()`
  - [x] 3.3 Export as `voiceSummarySchema` for use in `summarize()` function

- [x] **Task 4: Add LLM_FAILED error code to API response types** (AC: #4)
  - [x] 4.1 Add `"LLM_FAILED"` to `ErrorCode` type in `lib/api-response.ts`
  - [x] 4.2 Add `LLM_FAILED: 502` to `errorCodeToStatus` map

- [x] **Task 5: Extend POST /api/voice route with LLM step** (AC: #1, #4, #5)
  - [x] 5.1 After STT success (status "transcribed"), add LLM summarization step
  - [x] 5.2 Call `summarize(transcript)` from `lib/ai/mistral.ts`
  - [x] 5.3 Measure LLM duration: `const llmStart = Date.now()` / `Date.now() - llmStart`
  - [x] 5.4 On LLM success: UPDATE `voice_messages` SET `summary`, `llmProvider: "mistral-small"`, `llmDurationMs`, `status: "done"`, `processedAt: sql\`now()\``
  - [x] 5.5 On LLM failure: UPDATE `voice_messages` SET `status: "partial"` (transcript preserved), return `errorResponse("LLM_FAILED", ...)`
  - [x] 5.6 Wrap LLM DB update in its own try/catch (pattern from Story 6.2 code review)
  - [x] 5.7 Return full response including `summary`, `llmProvider`, `llmDurationMs`

- [x] **Task 6: Update client recording page for LLM step** (AC: #1, #4)
  - [x] 6.1 Update `app/app/record/page.tsx` — after API response, set `processingStep(3)` if summary is present
  - [x] 6.2 Add `"llm-failed"` to `ErrorType` union and `ERROR_MESSAGES` map
  - [x] 6.3 Handle LLM_FAILED: show step 2 complete + amber error "Zusammenfassung konnte nicht erstellt werden. Dein Transkript ist gespeichert."
  - [x] 6.4 Update response type to expect `summary`, `llmProvider`, `llmDurationMs` fields

- [x] **Task 7: LLM summarization evaluation — prompt engineering** (AC: #3)
  - [x] 7.1 Create evaluation script `scripts/llm-evaluation.ts` (standalone, self-loads .env.local)
  - [x] 7.2 Test minimum 3 prompt variants against transcripts from STT evaluation (reuse `docs/research/stt-evaluation-raw-results.json`)
  - [x] 7.3 Evaluate: semantic correctness (manual), field completeness, JSON validity rate, latency, cost
  - [x] 7.4 Create `docs/research/llm-summarization-evaluation-2026-02.md` with results matrix
  - [x] 7.5 Create ADR-005 for prompt engineering approach decision
  - [x] 7.6 Commit with R&D message: "Evaluate Mistral LLM summarization: prompt engineering for Handwerker context"

## Dev Notes

### Architecture Context — This Story's Scope

This is the **third and final story** in the Voice Capture & AI Pipeline epic. It adds **LLM summarization** to complete the pipeline. The processing progression:

```
Story 6.1: Record -> Upload to Blob -> DB entry (status: "uploaded")     DONE
Story 6.2: + STT via Mistral Voxtral (status: "transcribed")            DONE
Story 6.3: + LLM Summarization (status: "done")                         THIS STORY
```

After this story, the full synchronous pipeline is complete:
```
Browser -> POST /api/voice -> Blob Upload -> STT -> LLM -> DB -> Response
```

### Mistral Chat API — Structured JSON Output

**Model:** `mistral-small-latest` (Mistral Small 3.2, 24B parameters)
- Native German fluency, 128k context window
- Structured output support (`json_object` and `json_schema` modes)
- Pricing: $0.06/M input, $0.18/M output -> ~$0.00008 per summarization
- EU-hosted (French company, DSGVO-compliant)

**SDK Method: `client.chat.complete()`**

Use `chat.complete()` with `responseFormat: { type: "json_object" }`. Do **NOT** use `chat.parse()` — it depends on `zod-to-json-schema` which is not installed and has Zod v4 compatibility issues.

```typescript
const response = await getClient().chat.complete({
  model: "mistral-small-latest",
  messages: [
    { role: "system", content: VOICE_SUMMARY_SYSTEM_PROMPT },
    { role: "user", content: transcript },
  ],
  responseFormat: { type: "json_object" },
  temperature: 0,
  maxTokens: 1024,
});

const content = response.choices?.[0]?.message?.content;
if (!content || typeof content !== "string") {
  throw new Error("Mistral returned empty response");
}

const raw = JSON.parse(content);
const parsed = voiceSummarySchema.parse(raw); // Zod v4 validation
return parsed;
```

**Response structure:**
```json
{
  "id": "cmpl-xxx",
  "object": "chat.completion",
  "model": "mistral-small-latest",
  "choices": [{
    "index": 0,
    "message": {
      "role": "assistant",
      "content": "{\"status\":\"Verteiler montiert...\",\"material\":[...],\"nextSteps\":\"...\",\"problems\":null,\"project\":\"Müller\"}"
    },
    "finish_reason": "stop"
  }],
  "usage": { "prompt_tokens": 180, "completion_tokens": 95, "total_tokens": 275 }
}
```

**CRITICAL: `chat.parse()` Compatibility Issue**

The SDK's `chat.parse()` method uses `zod-to-json-schema` (line 6 in `node_modules/@mistralai/mistralai/extra/structChat.js`) which:
1. Is NOT installed as a project dependency (only listed in Mistral SDK's `package.json`)
2. `zod-to-json-schema` v3.24.x is designed for Zod v3, NOT our Zod v4.3.6

**Safe approach:** Use `chat.complete()` + manual `JSON.parse()` + Zod v4 `.parse()`. This gives us full control.

### VoiceSummary Schema — Already Defined

The `VoiceSummary` interface already exists in `lib/db/schema.ts` (line 35-41):

```typescript
export interface VoiceSummary {
  status: string;
  material: string[];
  nextSteps: string;
  problems: string | null;
  project: string;
}
```

The `voice_messages.summary` column is `jsonb("summary").$type<VoiceSummary | null>()` (line 109). No schema changes needed.

### Zod v4 Validation Schema

Create in `lib/validations/voice.ts` (file already exists with `MAX_FILE_SIZE`, `ACCEPTED_AUDIO_TYPES`, etc.):

```typescript
import { z } from "zod";

export const voiceSummarySchema = z.object({
  status: z.string(),
  material: z.array(z.string()),
  nextSteps: z.string(),
  problems: z.string().nullable(),
  project: z.string(),
});
```

**Zod v4 note:** `z.string()` is correct (not `z.string().min(1)` — empty strings are valid for some fields). `z.string().nullable()` is correct for `problems` which can be `null`.

### System Prompt Design — lib/ai/prompts.ts

Create a new file `lib/ai/prompts.ts`:

```typescript
// Prompt v1 — 2026-02
// FFG Forschungsfrage #2: Semantische Transformation unstrukturierter Sprachnachrichten

export const VOICE_SUMMARY_SYSTEM_PROMPT = `Du bist ein Assistent für Elektro-Handwerksbetriebe. Du erhältst das Transkript einer Sprachnachricht von einem Monteur auf der Baustelle.

Erstelle eine strukturierte Zusammenfassung als JSON mit genau diesen Feldern:

- "status": Kurze Statusbeschreibung der Arbeit (1-2 Sätze)
- "material": Array von erwähnten Materialien/Werkzeugen (leeres Array [] wenn keine genannt)
- "nextSteps": Nächste geplante Schritte oder Aufgaben (1-2 Sätze)
- "problems": Probleme, Störungen oder Verzögerungen (null wenn keine)
- "project": Projekt- oder Kundenname (aus Kontext erschließen, "Unbekannt" wenn nicht erkennbar)

Regeln:
- Antworte NUR mit validem JSON, kein zusätzlicher Text
- Verwende deutsche Sprache in den Werten
- Interpretiere österreichische Dialektausdrücke korrekt
- Fachbegriffe der Elektrotechnik korrekt übernehmen (z.B. FI-Schalter, Sicherungsautomat, Zählerkasten)
- Wenn ein Feld nicht aus dem Transkript ableitbar ist, verwende sinnvolle Defaults (leeres Array, null, "Unbekannt")`;
```

**Prompt engineering evaluation (Task 7):** Test at least 3 variants — minimal prompt, detailed prompt, few-shot prompt. Use transcripts from STT evaluation as test inputs.

### Extending the Voice Route — After STT Step

Current flow ends at STT (returns status "transcribed"). Extend to add LLM after STT success:

```typescript
// After successful STT update (line ~70 in current route.ts):
// ... existing STT code that sets status to "transcribed" ...

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
    })
    .where(eq(voiceMessages.id, entry.id))
    .returning();

  return successResponse({
    id: final.id,
    status: final.status,
    audioUrl: final.audioUrl,
    audioDurationSeconds: final.audioDurationSeconds,
    transcript: final.transcript,
    summary: final.summary,
    sttProvider: final.sttProvider,
    sttDurationMs: final.sttDurationMs,
    llmProvider: final.llmProvider,
    llmDurationMs: final.llmDurationMs,
  });
} catch (llmError) {
  console.error(
    "Mistral LLM error:",
    llmError instanceof Error ? `${llmError.name}: ${llmError.message}` : llmError
  );

  try {
    await db
      .update(voiceMessages)
      .set({ status: "partial" })
      .where(eq(voiceMessages.id, entry.id));
  } catch (dbError) {
    console.error(
      "Failed to update voice message status to partial:",
      dbError instanceof Error ? dbError.message : dbError
    );
  }

  return errorResponse(
    "LLM_FAILED",
    "Zusammenfassung konnte nicht erstellt werden. Dein Transkript ist gespeichert."
  );
}
```

**IMPORTANT structural change:** The STT success path currently returns early (line 72-80 in `app/api/voice/route.ts`). The LLM step must be added INSIDE the STT try block, BEFORE the return statement. The STT try block's success path should flow into LLM instead of returning immediately.

### Client-Side Updates — Recording Page

**After Story 6.3, the flow becomes:**
```
uploading -> processingStep(1) -> processingStep(2) -> processingStep(3) -> done -> redirect
```

Changes to `app/app/record/page.tsx`:
1. Add `"llm-failed"` to `ErrorType` union
2. Add to `ERROR_MESSAGES`: `"llm-failed": "Zusammenfassung konnte nicht erstellt werden. Dein Transkript ist gespeichert."`
3. In `handleSend` success path: check `data.data?.summary` to set `processingStep(3)`
4. In error handling: check `data?.error?.code === "LLM_FAILED"` -> set step 2 complete + error

```typescript
// Success path update:
setProcessingStep(1);
if (data.data?.transcript !== undefined) {
  setProcessingStep(2);
}
if (data.data?.summary !== undefined) {
  setProcessingStep(3);
}
setPageState("done");

// Error handling update (add before existing STT_FAILED check):
if (data?.error?.code === "LLM_FAILED") {
  setProcessingStep(2); // STT + Upload succeeded
  setErrorType("llm-failed");
  setPageState("error");
  return;
}
```

### Error Handling Strategy

| Error Type | DB Status | User Message (German) | Log (English) |
|---|---|---|---|
| LLM API timeout | `"partial"` | "Zusammenfassung konnte nicht erstellt werden." | `Mistral LLM timeout after ${ms}ms` |
| LLM API error (4xx/5xx) | `"partial"` | "Zusammenfassung konnte nicht erstellt werden." | `Mistral LLM error: ${status}` |
| Invalid JSON response | `"partial"` | "Zusammenfassung konnte nicht erstellt werden." | `Mistral LLM returned invalid JSON: ${content.substring(0,200)}` |
| Zod validation failed | `"partial"` | "Zusammenfassung konnte nicht erstellt werden." | `Mistral LLM schema validation failed: ${zodError}` |
| Empty transcript input | skip LLM | (no error — empty transcript = no summary) | `Skipping LLM for empty transcript: ${id}` |

**Design principle:** Status `"partial"` = transcript exists but no summary. The transcript is NEVER lost. Status `"done"` = both transcript AND summary are present.

**Empty transcript handling:** If `sttResult.transcript` is empty string, skip the LLM call entirely. Set status to `"transcribed"` (no summary needed for empty content).

### Timeout Budget

```
Upload:  ~1-2s
STT:     ~3-8s  (30s audio)
LLM:     ~2-5s  (summary generation)
Total:   ~6-15s (well within Vercel Pro 60s timeout)
```

### Processing Indicator — Already Supports Step 3

`components/app/processing-indicator.tsx` already defines step 3:
```typescript
const steps = [
  { label: "Hochgeladen", key: "uploaded" },      // step 1
  { label: "Wird transkribiert", key: "transcribed" }, // step 2
  { label: "Zusammenfassung", key: "summarized" },   // step 3
];
```

No changes needed to this component.

### Drizzle ORM Patterns — From Previous Stories

```typescript
import { eq, sql } from "drizzle-orm";

// Update with sql`now()` for server-side timestamp:
await db
  .update(voiceMessages)
  .set({
    summary: parsedSummary,       // VoiceSummary object -> JSONB
    llmProvider: "mistral-small",
    llmDurationMs,
    status: "done",
    processedAt: sql`now()`,      // DB-side timestamp
  })
  .where(eq(voiceMessages.id, entry.id))
  .returning();
```

**JSONB storage:** Drizzle handles `VoiceSummary` object -> JSONB serialization automatically via the `.$type<VoiceSummary | null>()` column definition.

### Evaluation Script Pattern — Reuse from Story 6.2

Follow the same pattern as `scripts/stt-evaluation.ts`:
- Self-load `.env.local` at script start
- Use `tsx` to run: `npx tsx scripts/llm-evaluation.ts`
- Reuse transcripts from `docs/research/stt-evaluation-raw-results.json` as inputs
- Output raw results to `docs/research/llm-evaluation-raw-results.json`

### Project Structure Notes

**New files:**
```
lib/
  ai/
    prompts.ts               # NEW — System prompt constants (version-controlled)

docs/
  research/
    llm-summarization-evaluation-2026-02.md  # NEW — Evaluation results

scripts/
  llm-evaluation.ts          # NEW — Evaluation script (standalone)
```

**Modified files:**
- `lib/ai/mistral.ts` — Add `summarize()` function
- `app/api/voice/route.ts` — Add LLM step after STT, restructure success path
- `app/app/record/page.tsx` — Add LLM error type, processingStep(3)
- `lib/api-response.ts` — Add `LLM_FAILED` error code
- `lib/validations/voice.ts` — Add `voiceSummarySchema`
- `docs/decisions.md` — Add ADR-005 link
- `docs/decisions/ADR-005-*.md` — NEW: Prompt engineering decision

### Server vs Client Component Map

| File | Type | Reason |
|------|------|--------|
| `lib/ai/mistral.ts` | Server utility | API key, network calls |
| `lib/ai/prompts.ts` | Server utility | Imported by mistral.ts only |
| `lib/validations/voice.ts` | Shared utility | Zod schema, used server-side |
| `app/api/voice/route.ts` | API Route (server) | Orchestrates upload + STT + LLM |
| `app/app/record/page.tsx` | Client Component (`"use client"`) | State management, UI updates |
| `components/app/processing-indicator.tsx` | Client Component | No changes needed |

### Previous Story Intelligence

**From Story 6.2 (Mistral Voxtral STT Integration):**
- `lib/ai/mistral.ts` — lazy `getClient()` pattern established. Reuse for `summarize()`.
- `app/api/voice/route.ts` — STT try/catch with inner DB try/catch for error handling. Apply same pattern to LLM step.
- `sttDurationMs` tracking pattern -> replicate for `llmDurationMs`
- Code review finding M1: lazy client init prevents module-level crash -> already fixed, `summarize()` reuses same pattern
- Code review finding M2: DB update in catch wrapped in try/catch -> apply same for LLM catch block

**From Story 6.2 Evaluation:**
- Transcripts from STT evaluation available in `docs/research/stt-evaluation-raw-results.json` -> reuse as LLM evaluation inputs
- `scripts/stt-evaluation.ts` pattern: self-load .env.local, tsx runner, raw JSON output -> replicate for LLM evaluation

**From Story 3.4 (Double Opt-In Email):**
- `sql\`now()\`` for DB-side timestamps (import from `drizzle-orm`)
- Drizzle `.update().set().where()` — set BEFORE where

**From Git History (recent commits):**
- `f51034a` — Story 6.2 done
- `9675512` — STT pipeline with contextBias
- `f1939b9` — STT evaluation: WER comparison Voxtral vs Whisper

### FFG Compliance Reminders

- **Mistral AI is EU PRIMARY** — French company, EU-default hosting. FFG-compliant.
- **Research documentation required** — `llmProvider`, `llmDurationMs` tracked in DB for FFG audit trail
- **Prompt engineering is core R&D** — document all variants tested, evaluation criteria, rationale for winning prompt
- **Commit messages** — R&D character: "Evaluate Mistral LLM summarization for Handwerker domain"
- **No API keys in frontend** — `MISTRAL_API_KEY` already server-side only
- **Decision documentation** — ADR-005 for prompt engineering approach

### Cost Projection

| Metric | Value |
|---|---|
| Avg input per summary | ~750 tokens ($0.000045) |
| Avg output per summary | ~200 tokens ($0.000036) |
| Cost per summarization | ~$0.00008 |
| 100 summaries/day | ~$0.008/day |
| Monthly (3000 summaries) | ~$0.24/month |
| Combined STT + LLM monthly | ~$45 (STT) + $0.24 (LLM) = ~$45.24/month |

### References

- [Mistral Structured Output: docs.mistral.ai/capabilities/structured_output]
- [Mistral JSON Mode: docs.mistral.ai/capabilities/structured_output/json_mode]
- [Mistral Models: docs.mistral.ai/getting-started/models]
- [Mistral Pricing: mistral.ai/pricing]
- [@mistralai/mistralai SDK v1.14.0: npmjs.com/package/@mistralai/mistralai]
- [VoiceSummary interface: lib/db/schema.ts#L35-41]
- [Voice Processing Pipeline: docs/architecture-sprint0-supplement.md#4]
- [Dashboard Summary Cards: docs/ux-sprint0-app-specs.md#5 -> Card Anatomy]
- [Processing Indicator UX: docs/ux-sprint0-app-specs.md#4 -> State E]
- [Error States UX: docs/ux-sprint0-app-specs.md#4 -> State F]
- [API Route: app/api/voice/route.ts]
- [API Response Types: lib/api-response.ts]
- [Validation Schemas: lib/validations/voice.ts]
- [STT Evaluation Raw Results: docs/research/stt-evaluation-raw-results.json]
- [Story 6.2: _bmad-output/implementation-artifacts/6-2-mistral-voxtral-stt-integration-and-evaluation.md]
- [Epics: _bmad-output/planning-artifacts/epics.md#story-63-mistral-llm-summarization]
- [FFG Rules: CLAUDE.md + docs/_masemIT/kurzum-bmad-do-dont-liste.md]

## Dev Agent Record

### Agent Model Used

Claude Opus 4.6

### Debug Log References

### Completion Notes List

- Task 1: Added `summarize()` function to `lib/ai/mistral.ts` — uses `chat.complete()` with `json_object` mode, Zod v4 validation, typed `VoiceSummary` return
- Task 2: Created `lib/ai/prompts.ts` with `VOICE_SUMMARY_SYSTEM_PROMPT` (v1) — Handwerker context, dialect handling, technical term preservation, field descriptions
- Task 3: Added `voiceSummarySchema` to `lib/validations/voice.ts` — Zod v4 object schema matching `VoiceSummary` interface
- Task 4: Added `LLM_FAILED` error code (502) to `lib/api-response.ts`
- Task 5: Extended `/api/voice` route — LLM step after STT, empty transcript skip, separate try/catch for LLM, `"partial"` status on failure, `"done"` on success with `processedAt`
- Task 6: Updated recording page — `"llm-failed"` error type, `processingStep(3)` on summary, `LLM_FAILED` error handling shows step 2 complete + amber message, retry support
- Task 7: LLM evaluation complete — 3 prompt variants (minimal, detailed, few-shot) tested against 8 transcripts (24 API calls). Detailed prompt wins: 100% JSON validity, 96.3% semantic correctness, $0.00004/summary. ADR-005 created.

### Code Review Fixes (AI)

- **H1/M1 fixed:** `summarize()` now separates JSON parse vs Zod validation errors via `LLMValidationError` class. Raw LLM response content (truncated to 500 chars) is preserved for research analysis (AC #5). Uses `safeParse()` instead of `parse()` for controlled error handling.
- **M2 documented:** Added comment on `handleRetry` documenting that retry re-sends full pipeline, creating duplicate DB entries. Accepted limitation for research prototype.
- **M3 fixed:** All `console.error` calls in `app/api/voice/route.ts` converted to structured object format per architecture standard (`event_name`, `{ key: value }`).

### File List

**New files:**
- `lib/ai/prompts.ts` — System prompt constant (VOICE_SUMMARY_SYSTEM_PROMPT v1)
- `scripts/llm-evaluation.ts` — Standalone evaluation script (3 prompt variants, 8 transcripts)
- `docs/research/llm-summarization-evaluation-2026-02.md` — Evaluation results and analysis
- `docs/research/llm-evaluation-raw-results.json` — Raw evaluation data (24 API call results)
- `docs/decisions/ADR-005 - Prompt Engineering for Handwerker LLM Summarization.md` — Architecture decision record

**Modified files:**
- `lib/ai/mistral.ts` — Added `summarize()` function with imports for prompts, Zod schema, VoiceSummary type
- `lib/validations/voice.ts` — Added `voiceSummarySchema` Zod v4 object schema + `z` import
- `lib/api-response.ts` — Added `LLM_FAILED` to ErrorCode union and errorCodeToStatus map
- `app/api/voice/route.ts` — Added LLM step after STT, empty transcript skip, `sql` import, error handling with `"partial"` status
- `app/app/record/page.tsx` — Added `"llm-failed"` error type, `processingStep(3)`, LLM_FAILED handling, retry support
- `docs/decisions.md` — Added ADR-005 to index

### Change Log

- 2026-02-13: Story 6.3 implementation complete — Mistral LLM summarization pipeline with prompt engineering evaluation (Claude Opus 4.6)
- 2026-02-13: Code review fixes — LLMValidationError for AC #5 compliance, structured logging, retry limitation documented (Claude Opus 4.6)
