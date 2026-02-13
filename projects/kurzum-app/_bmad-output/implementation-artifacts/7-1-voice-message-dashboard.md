# Story 7.1: Voice Message Dashboard

Status: done

<!-- Note: Validation is optional. Run validate-create-story for quality check before dev-story. -->

## Story

As a **user (Stefan)**,
I want to see all voice messages with their AI summaries on a clean dashboard,
So that I have a morning overview of all field reports without making a single phone call.

**FFG Relevanz:** Dashboard visualisiert die AI-Pipeline-Ergebnisse (STT + LLM) und dient als Nachweis der Forschungsfrage #2 ("Semantische Transformation unstrukturierter Sprachnachrichten").

## Acceptance Criteria

1. **Given** the user is authenticated and voice messages exist, **When** the dashboard page at `/app` is loaded, **Then** voice messages are displayed as summary cards sorted newest first, messages are grouped by day ("Heute", "Gestern", or date like "Mo, 24. Feb"), and relative timestamps are used ("vor 2 Std", "gestern 16:30").

2. **Given** a voice message has a summary (status `"done"`), **When** the summary card is rendered, **Then** it shows: username + relative time (header), `summary.status` with checkmark, `summary.material` with package icon (if not empty array), `summary.problems` with warning icon (if not null, amber background tint), `summary.nextSteps` with arrow (if not empty), `summary.project` with pin icon. Only fields with content are shown — no "Material: —" for empty fields. If `summary.project` is "Unbekannt", show "Nicht zugeordnet" in muted color.

3. **Given** a summary card is tapped, **When** the card expands, **Then** the full transcript is shown (collapsed by default, toggled by tap), an audio player (waveform + play/pause) appears if audio is still available (`audioUrl` is set and `audioDeletedAt` is null), and metadata is shown in muted small text: recording duration, STT provider, processing time (sttDurationMs + llmDurationMs).

4. **Given** a voice message is still processing (status `"uploaded"`, `"transcribed"`, or `"pending"`), **When** it appears in the dashboard, **Then** it shows: username + "gerade eben" + hourglass "KI verarbeitet..." with a subtle animated indicator.

5. **Given** no voice messages exist (first login), **When** the dashboard loads, **Then** an empty state is shown: microphone icon, "Noch keine Nachrichten.", encouraging text "Sprich deine erste Nachricht und schau was die KI daraus macht.", and a CTA button "Erste Aufnahme starten" (links to `/app/record`). No onboarding wizard or multi-step tutorial.

6. **Given** the dashboard is viewed on mobile, **When** rendered, **Then** cards are full-width with appropriate padding, all interactive elements have ≥48px touch targets, and the page is scrollable. **Given** desktop (≥768px), **Then** content is centered with max-width 720px (already handled by app layout) and cards have comfortable spacing.

7. **Given** a voice message has status `"partial"` (transcript exists but no summary), **When** the card is rendered, **Then** the transcript is shown directly (no summary fields), the status indicates "Transkript vorhanden — Zusammenfassung ausstehend", and the card is still expandable to show full transcript + audio player.

8. **Given** a voice message has status `"error"` (STT failed, no transcript), **When** the card is rendered, **Then** it shows a muted card with "Spracherkennung fehlgeschlagen" and an audio player (audio is still available).

## Tasks / Subtasks

- [x] **Task 1: Create dashboard data fetching in page.tsx** (AC: #1)
  - [x] 1.1 Convert `app/app/page.tsx` from placeholder to Server Component that queries `voice_messages` for the authenticated user
  - [x] 1.2 Query: `db.select().from(voiceMessages).where(eq(voiceMessages.userId, userId)).orderBy(desc(voiceMessages.createdAt))`
  - [x] 1.3 Get `userId` from `headers().get("x-user-id")` (middleware-injected, same pattern as `layout.tsx`)
  - [x] 1.4 Also fetch user names for display: query `users` table (for Sprint 0, only one user exists, so `userName` from layout context suffices)
  - [x] 1.5 Pass messages to client component for interactive rendering

- [x] **Task 2: Create VoiceMessageCard component** (AC: #2, #7, #8)
  - [x] 2.1 Create `components/app/voice-message-card.tsx` as Client Component (`"use client"`)
  - [x] 2.2 Accept `DashboardMessage` type + `userName: string` (custom serialized type to avoid hydration issues)
  - [x] 2.3 Render summary fields conditionally: only show material if array has items, only show problems if not null, only show nextSteps if non-empty, only show project if not "Unbekannt" (show "Nicht zugeordnet" in muted color instead)
  - [x] 2.4 Problems section: amber/orange background tint (`bg-amber-50 border-l-2 border-amber-400`)
  - [x] 2.5 Handle `"partial"` status: show transcript directly, muted indicator "Transkript vorhanden — Zusammenfassung ausstehend"
  - [x] 2.6 Handle `"error"` status: muted card with "Spracherkennung fehlgeschlagen", only audio player
  - [x] 2.7 Handle processing states (`"uploaded"`, `"transcribed"`, `"pending"`): show "KI verarbeitet..." with animated indicator

- [x] **Task 3: Implement card expansion with transcript + audio** (AC: #3)
  - [x] 3.1 Add expandable section (click/tap to toggle) showing full transcript text
  - [x] 3.2 Show audio player only if `audioUrl` exists AND `audioDeletedAt` is null
  - [x] 3.3 Reuse waveform + Web Audio API playback pattern from `RecordingPreview` — standalone AudioPlayer with fetch-from-URL and lazy loading
  - [x] 3.4 Show metadata below audio: recording duration (`audioDurationSeconds`), STT provider (`sttProvider`), processing time (`sttDurationMs` + `llmDurationMs` in seconds)
  - [x] 3.5 Use `toFixed(1)` for processing time formatting (e.g., "4.2s Verarbeitung")

- [x] **Task 4: Implement day grouping and relative timestamps** (AC: #1)
  - [x] 4.1 Create helper function to group messages by day: "Heute", "Gestern", or formatted date (e.g., "Mo, 24. Feb")
  - [x] 4.2 Create relative timestamp formatter: "vor 2 Std", "vor 30 Min", "gestern 16:30", "Mo 09:15"
  - [x] 4.3 Use `Intl.DateTimeFormat('de-AT', ...)` for date formatting — NO date-fns, NO moment
  - [x] 4.4 Group messages in Server Component before passing to client (avoids hydration mismatch with date formatting)

- [x] **Task 5: Implement empty state** (AC: #5)
  - [x] 5.1 Create empty state UI: microphone icon, "Noch keine Nachrichten.", encouraging text, orange CTA button
  - [x] 5.2 CTA links to `/app/record` via `next/link`
  - [x] 5.3 Render inline in page.tsx when messages array is empty (no separate component needed)

- [x] **Task 6: Responsive design and accessibility** (AC: #6)
  - [x] 6.1 Cards full-width on mobile, comfortable spacing on desktop
  - [x] 6.2 All interactive elements ≥48px touch targets (expandable cards, audio player, CTA)
  - [x] 6.3 WCAG AA contrast (4.5:1) for all text
  - [x] 6.4 Semantic HTML: `<article>` for cards, `<section>` for day groups, `<time>` for timestamps
  - [x] 6.5 `aria-expanded` on expandable cards, screen reader labels for icons

## Dev Notes

### Architecture Context — This Story's Scope

This is the **only story** in Epic 7 (Dashboard & Research Documentation). It replaces the placeholder `app/app/page.tsx` with a functional dashboard. The pipeline is complete from Epic 6:

```
Epic 5: Auth (Magic Link) + App Shell + DB Schema                    DONE
Epic 6: Record → Upload → STT → LLM → DB (status: "done")           DONE
Epic 7: Dashboard — Display all voice messages with AI summaries      THIS STORY
```

After this story, the Sprint 0 prototype is feature-complete: Record → AI Process → View Results.

### Existing App Layout — Already Handles Max-Width

`app/app/layout.tsx` (Server Component) already provides:
- Header with userName (fetched from DB)
- `<main>` with `max-w-[720px] mx-auto px-4 pb-20 md:pb-4 pt-4`
- BottomNav (fixed, 2 tabs)

The dashboard page.tsx renders INSIDE this layout. **Do NOT add another max-width wrapper or header.**

### Server vs Client Component Strategy

**Critical decision:** The page MUST be a Server Component for data fetching (no `"use client"` at page level). Pass data to client components for interactivity.

```
app/app/page.tsx          → Server Component (data fetching, grouping)
  ├── (empty state)       → Inline JSX (no client interactivity needed)
  └── VoiceMessageCard    → Client Component (expansion, audio playback)
```

### Data Fetching — Drizzle ORM Pattern

```typescript
// app/app/page.tsx (Server Component)
import { headers } from "next/headers";
import { eq, desc } from "drizzle-orm";
import { db } from "@/lib/db";
import { voiceMessages } from "@/lib/db/schema";

const headersList = await headers();
const userId = headersList.get("x-user-id")!;

const messages = await db
  .select()
  .from(voiceMessages)
  .where(eq(voiceMessages.userId, userId))
  .orderBy(desc(voiceMessages.createdAt));
```

**Return type:** `VoiceMessage[]` (inferred from schema, includes all fields).

### VoiceMessage Status States

| Status | Meaning | Card Rendering |
|--------|---------|---------------|
| `"pending"` | Just created, no processing yet | "KI verarbeitet..." |
| `"uploaded"` | Audio in Blob, waiting for STT | "KI verarbeitet..." |
| `"transcribed"` | STT done, waiting for LLM (empty transcript) | "KI verarbeitet..." |
| `"done"` | Full pipeline complete | Summary card with all fields |
| `"partial"` | STT succeeded, LLM failed | Transcript shown, no summary fields |
| `"error"` | STT failed | "Spracherkennung fehlgeschlagen" + audio player |

### Day Grouping — Helper Pattern

```typescript
function groupMessagesByDay(messages: VoiceMessage[]): Map<string, VoiceMessage[]> {
  const groups = new Map<string, VoiceMessage[]>();
  const now = new Date();
  const today = new Date(now.getFullYear(), now.getMonth(), now.getDate());
  const yesterday = new Date(today.getTime() - 86400000);

  for (const msg of messages) {
    const msgDate = new Date(msg.createdAt);
    const msgDay = new Date(msgDate.getFullYear(), msgDate.getMonth(), msgDate.getDate());

    let label: string;
    if (msgDay.getTime() === today.getTime()) {
      label = "Heute";
    } else if (msgDay.getTime() === yesterday.getTime()) {
      label = "Gestern";
    } else {
      label = new Intl.DateTimeFormat("de-AT", {
        weekday: "short",
        day: "numeric",
        month: "short",
      }).format(msgDate);
    }

    if (!groups.has(label)) groups.set(label, []);
    groups.get(label)!.push(msg);
  }
  return groups;
}
```

**CRITICAL:** Do day grouping in the Server Component to avoid hydration mismatch. Timezone differences between server and client can cause "Heute"/"Gestern" to differ. Pass pre-grouped data as `{ label: string, messages: VoiceMessage[] }[]` to the client.

### Relative Timestamps — Intl.RelativeTimeFormat

```typescript
function formatRelativeTime(date: Date): string {
  const now = new Date();
  const diffMs = now.getTime() - date.getTime();
  const diffMin = Math.floor(diffMs / 60000);
  const diffHrs = Math.floor(diffMs / 3600000);

  if (diffMin < 1) return "gerade eben";
  if (diffMin < 60) return `vor ${diffMin} Min`;
  if (diffHrs < 24) return `vor ${diffHrs} Std`;

  // For older messages: "gestern 16:30" or "Mo 09:15"
  return new Intl.DateTimeFormat("de-AT", {
    hour: "2-digit",
    minute: "2-digit",
  }).format(date);
}
```

**Note:** Architecture mandates `Intl.DateTimeFormat('de-AT', ...)` for UI formatting — no date-fns or moment.

### Audio Player in Expanded Card

**CRITICAL: Reuse Web Audio API pattern from RecordingPreview**

The `components/app/recording-preview.tsx` already implements:
- `AudioContext.decodeAudioData()` for decoding
- `BufferSourceNode` for playback
- Canvas waveform rendering
- `getComputedStyle()` for CSS variable colors

**Do NOT create a new audio implementation.** Extract or import the shared pattern. Options:
1. **Extract `AudioPlayer` component** from RecordingPreview's playback logic — recommended
2. Import `generateWaveformData` from RecordingPreview (currently not exported)

The expanded card needs to fetch audio from `audioUrl` (Vercel Blob URL), decode it, and render a waveform + play/pause button. Same pattern as RecordingPreview but with a remote URL instead of a local Blob.

**Key difference:** RecordingPreview receives a `Blob` directly. Dashboard cards need to `fetch(audioUrl)` first, then decode. Add loading state for audio fetch.

### Card Expansion — Click/Tap Toggle

```typescript
// In VoiceMessageCard (Client Component)
const [isExpanded, setIsExpanded] = useState(false);

// Toggle on card click
<article
  onClick={() => setIsExpanded((prev) => !prev)}
  aria-expanded={isExpanded}
  role="button"
  tabIndex={0}
  onKeyDown={(e) => { if (e.key === "Enter" || e.key === " ") setIsExpanded((prev) => !prev); }}
>
  {/* Summary fields always visible */}
  {isExpanded && (
    <div>
      {/* Transcript */}
      {/* Audio player (lazy load) */}
      {/* Metadata */}
    </div>
  )}
</article>
```

**Lazy-load audio:** Only fetch and decode audio when the card is expanded AND `audioUrl` is present. Don't pre-fetch all audio files on page load.

### Summary Card Styling — UX Spec

From `docs/ux-sprint0-app-specs.md#5`:

```
Header:    🎙️ {userName} · {relativeTime}
Status:    {summary.status} ✅
Material:  📦 {summary.material.join(", ")}     ← only if array.length > 0
Problems:  ⚠️ {summary.problems}                 ← only if not null, amber tint
NextSteps: → {summary.nextSteps}                 ← only if non-empty
Project:   📍 {summary.project}                  ← "Nicht zugeordnet" if "Unbekannt"
```

**Conditional rendering rules:**
- Empty `material` array → hide entire block
- `problems === null` → hide entire block
- `problems !== null` → amber background: `bg-amber-50 border-l-2 border-amber-400 p-2 rounded`
- `nextSteps === ""` → hide entire block (shouldn't happen with current prompt, but defensive)
- `project === "Unbekannt"` → show "Nicht zugeordnet" in `text-muted-foreground`

### Processing Card — Animated Indicator

For messages still being processed (status: `"uploaded"`, `"transcribed"`, `"pending"`):

```
🎙️ {userName} · gerade eben
⏳ KI verarbeitet...
```

Use a subtle `animate-pulse` on the text or a small spinner. The `ProcessingIndicator` component from `components/app/processing-indicator.tsx` is designed for the recording page flow — **do NOT reuse it here**. The dashboard needs a simpler inline indicator, not a multi-step display.

### Metadata Display — FFG Research Fields

Show in expanded card, muted small text:
```
Aufnahme: {audioDurationSeconds}s · {sttProvider} · {(sttDurationMs + llmDurationMs) / 1000}s
```

Format: `Aufnahme: 23s · Voxtral · 4.2s Verarbeitung`

These fields serve the FFG audit trail requirement — they prove AI processing is happening with EU providers.

### Project Structure Notes

**New files:**
```
components/app/
  voice-message-card.tsx    # NEW — Summary card component (Client Component)
```

**Modified files:**
```
app/app/page.tsx            # Replace placeholder with data-fetching Server Component
```

**Potentially modified (if extracting audio player):**
```
components/app/recording-preview.tsx  # Extract shared audio utilities (optional)
```

**NOT touched:**
- `lib/db/schema.ts` — no schema changes
- `lib/ai/mistral.ts` — no AI changes
- `app/api/voice/route.ts` — no API changes
- `components/app/processing-indicator.tsx` — not reused here

### Previous Story Intelligence

**From Story 6.1 Code Review:**
- Canvas colors need `getComputedStyle()` to resolve CSS variables — apply same in dashboard audio player
- `redirectTimerRef` with cleanup on unmount — establish same cleanup patterns for audio in dashboard cards
- Web Audio API playback required for Safari WebM compatibility — same applies to dashboard audio player

**From Story 6.2/6.3:**
- `voiceMessages` table has all fields needed (summary JSONB, sttProvider, llmProvider, durations)
- Drizzle `db.select().from()` + `eq()` + `desc()` pattern established
- `VoiceMessage` type auto-inferred from schema — use directly, no manual interface needed

**From Story 6.3 Code Review (just completed):**
- `LLMValidationError` class added for structured error logging — not relevant to dashboard
- All logging now structured objects — follow same pattern if any logging needed
- Retry creates duplicate entries — dashboard may show duplicate messages for the same recording. Not a concern for Sprint 0 (manual cleanup), but be aware.

### Server vs Client Component Map

| File | Type | Reason |
|------|------|--------|
| `app/app/page.tsx` | Server Component | Data fetching with `headers()` + Drizzle query |
| `components/app/voice-message-card.tsx` | Client Component (`"use client"`) | Click expansion, audio playback, useState |

### References

- [UX Dashboard Spec: docs/ux-sprint0-app-specs.md#5 — Dashboard/Übersicht]
- [UX Empty State: docs/ux-sprint0-app-specs.md#6 — Empty State (Erster Login)]
- [UX Summary Card Anatomy: docs/ux-sprint0-app-specs.md#5 — Summary Card — Anatomie]
- [UX Card Expansion: docs/ux-sprint0-app-specs.md#5 — Card Expansion (Tap)]
- [UX Processing State: docs/ux-sprint0-app-specs.md#5 — Processing State in Dashboard]
- [Architecture Supplement: docs/architecture-sprint0-supplement.md#4 — Voice Processing Pipeline]
- [Architecture API Patterns: _bmad-output/planning-artifacts/architecture.md — Implementation Patterns]
- [DB Schema: lib/db/schema.ts — voiceMessages table (lines 100-127)]
- [VoiceSummary Interface: lib/db/schema.ts#L35-41]
- [App Layout: app/app/layout.tsx — max-w-[720px], header, bottomNav]
- [Middleware: middleware.ts — x-user-id header injection]
- [Recording Preview Audio Pattern: components/app/recording-preview.tsx]
- [Epics: _bmad-output/planning-artifacts/epics.md#story-71-voice-message-dashboard]
- [S0-FR8: Dashboard — list of voice messages with AI summaries]
- [S0-FR9: Summary card — expandable, shows transcript + audio player]
- [S0-NFR4: Mobile-first, 48px touch targets, WCAG AA contrast]
- [FFG Rules: CLAUDE.md + docs/_masemIT/kurzum-bmad-do-dont-liste.md]

## Dev Agent Record

### Agent Model Used

Claude Opus 4.6

### Debug Log References

### Completion Notes List

- Task 1: Converted `app/app/page.tsx` from placeholder to async Server Component. Fetches `voiceMessages` via Drizzle ORM with `eq(userId)` + `orderBy(desc(createdAt))`. Also queries `users` table for `userName` display.
- Task 2: Created `components/app/voice-message-card.tsx` as Client Component with `DashboardMessage` interface (serialized type for RSC→CC boundary). Renders summary fields conditionally with emoji icons. Handles all 6 status states: pending/uploaded/transcribed (processing), done (full summary), partial (transcript only), error (failed STT).
- Task 3: Implemented card expansion via click/tap toggle with `isExpanded` state. ExpandedContent shows transcript, AudioPlayer (Web Audio API with canvas waveform, fetches from URL with loading state), and metadata (duration, STT provider, processing time). Audio lazy-loaded only when card expands. `stopPropagation` on expanded content prevents accidental collapse.
- Task 4: Server-side `groupMessagesByDay()` and `formatRelativeTime()` using `Intl.DateTimeFormat('de-AT')`. Groups: "Heute", "Gestern", or "Mo, 24. Feb". Relative times: "gerade eben", "vor X Min", "vor X Std", "gestern HH:MM", "Do HH:MM". All formatting server-side to avoid hydration mismatch.
- Task 5: Empty state renders inline when `messages.length === 0`: Mic icon, "Noch keine Nachrichten.", encouraging text, orange CTA button linking to `/app/record` via `next/link`.
- Task 6: Semantic HTML (`<article>`, `<section>`, `<time>`), `aria-expanded`, `role="button"`, `tabIndex={0}`, keyboard navigation (Enter/Space), `aria-label` on audio buttons, `aria-hidden` on decorative icons. Cards full-width on mobile, comfortable gap-3 spacing. CTA button h-12 (48px), audio play button h-12 w-12 (48px).
- Code Review Fixes (H1+M1+M2): [H1] Error-state cards now show audio player directly without requiring expansion (AC #8). [M1] Audio play button increased from 40px to 48px touch target (AC #6). [M2] Extracted `generateWaveformData` into shared `lib/audio-utils.ts` — imported by both `voice-message-card.tsx` and `recording-preview.tsx`.

### File List

**New files:**
- `components/app/voice-message-card.tsx` — Client Component: VoiceMessageCard with DashboardMessage type, SummaryFields, ExpandedContent, AudioPlayer (Web Audio API waveform)
- `lib/audio-utils.ts` — Shared utility: `generateWaveformData()` used by both RecordingPreview and AudioPlayer

**Modified files:**
- `app/app/page.tsx` — Replaced placeholder with Server Component: data fetching, day grouping, relative timestamps, empty state, renders VoiceMessageCard
- `components/app/recording-preview.tsx` — Replaced local `generateWaveformData` with import from `lib/audio-utils.ts`

### Change Log

- 2026-02-13: Story 7.1 implementation complete — Voice message dashboard with summary cards, day grouping, card expansion, audio player, empty state (Claude Opus 4.6)
- 2026-02-13: Code review fixes applied — H1: error-state audio player visible without expansion, M1: 48px touch target for play button, M2: shared audio utility extracted (Claude Opus 4.6)
