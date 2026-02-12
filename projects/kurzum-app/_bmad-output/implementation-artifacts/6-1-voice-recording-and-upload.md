# Story 6.1: Voice Recording & Upload

Status: review

## Story

As a **user (Markus)**,
I want to record a voice message with one tap and send it for AI processing,
So that I can document my work on the construction site in 30 seconds without typing.

## Acceptance Criteria

1. **Given** the user is authenticated and on `/app` or `/app/record`, **When** the recording page is displayed, **Then** a large microphone button (80x80px, Orange, centered) is the primary action, with helper text "Zum Aufnehmen tippen" (disappears after first use), and first-time visitors see: "Sprich einfach los — die KI kümmert sich um den Rest."

2. **Given** the user taps the record button, **When** microphone permission is granted (or already granted), **Then** recording starts using MediaRecorder API (WebM/Opus preferred, WAV fallback), the button changes to a stop icon, color changes to red, a pulsing red indicator + timer (0:00, counting up) appear, a live audio level bar reacts to volume (AnalyserNode), and recording automatically stops at 5 minutes.

3. **Given** the user taps stop, **When** recording ends, **Then** a preview screen shows: waveform visualization + play/pause button + recording duration, with two buttons: "Verwerfen" (ghost/outline) and "Senden" (Orange, primary), and a "Nochmal aufnehmen" text link as tertiary action.

4. **Given** the user taps "Senden", **When** the audio is uploaded, **Then** POST `/api/voice` sends the audio as multipart/form-data, the API uploads to Vercel Blob (EU region), creates a `voice_messages` DB entry with status `uploaded`, and a processing indicator appears showing: checkmark "Hochgeladen" (steps 2-3 will activate in Stories 6.2/6.3).

5. **Given** microphone permission is denied or not available, **When** the error occurs, **Then** a warm amber message explains the issue in German with instructions to enable the microphone. No technical error codes shown.

6. **Given** upload or processing fails, **When** an error occurs, **Then** a user-friendly German message appears with a retry option, and the recording is preserved locally for retry.

## Tasks / Subtasks

- [x] **Task 1: Install @vercel/blob dependency** (AC: #4)
  - [x] 1.1 `pnpm add @vercel/blob`
  - [x] 1.2 Add `BLOB_READ_WRITE_TOKEN` to `.env.local` (obtain from Vercel Dashboard → Storage → Blob → Create Store in EU Frankfurt region)
  - [x] 1.3 Verify Blob store is configured for EU (Frankfurt) region in Vercel Dashboard

- [x] **Task 2: Create storage helper** (AC: #4)
  - [x] 2.1 Create `lib/storage.ts` — `uploadAudio(userId, file)` and `deleteAudio(url)` functions using @vercel/blob

- [x] **Task 3: Create Zod validation for voice upload** (AC: #4, #6)
  - [x] 3.1 Create `lib/validations/voice.ts` — schema for validating uploaded audio (file size ≤ 25MB, accepted MIME types)

- [x] **Task 4: Create POST /api/voice route** (AC: #4)
  - [x] 4.1 Create `app/api/voice/route.ts` — multipart/form-data handler: parse file, validate, upload to Blob, create DB entry, return result
  - [x] 4.2 Extract userId from `x-user-id` header (injected by middleware)
  - [x] 4.3 Measure and store `audioDurationSeconds` from client-provided metadata or file analysis

- [x] **Task 5: Create VoiceRecorder component (recording logic)** (AC: #1, #2)
  - [x] 5.1 Create `components/app/voice-recorder.tsx` — Client Component: MediaRecorder management, permission handling, recording state machine
  - [x] 5.2 Implement `getSupportedMimeType()` helper for cross-browser format detection
  - [x] 5.3 Implement 5-minute auto-stop timer
  - [x] 5.4 Implement live audio level visualization using AnalyserNode + requestAnimationFrame

- [x] **Task 6: Create RecordingPreview component** (AC: #3)
  - [x] 6.1 Create `components/app/recording-preview.tsx` — Client Component: waveform visualization, play/pause, duration display, Senden/Verwerfen/Nochmal buttons
  - [x] 6.2 Implement static waveform generation from AudioBuffer (Canvas-based, ~70 bars)
  - [x] 6.3 Implement audio playback via Web Audio API (avoids Safari WebM playback issues)

- [x] **Task 7: Create ProcessingIndicator component** (AC: #4)
  - [x] 7.1 Create `components/app/processing-indicator.tsx` — 3-step progress display: Hochgeladen → Wird transkribiert → Zusammenfassung
  - [x] 7.2 For Story 6.1: only step 1 (Hochgeladen) activates. Steps 2-3 remain as pending stubs, activated in Stories 6.2/6.3

- [x] **Task 8: Replace recording page placeholder** (AC: #1, #2, #3, #4, #5, #6)
  - [x] 8.1 Replace `app/app/record/page.tsx` with functional recording page integrating VoiceRecorder, RecordingPreview, ProcessingIndicator
  - [x] 8.2 Implement state machine: idle → recording → preview → uploading → processing → done/error
  - [x] 8.3 Implement error states with warm amber messages (German, no tech jargon)

- [x] **Task 9: Add ErrorCode for voice-specific errors** (AC: #6)
  - [x] 9.1 Add `"UPLOAD_FAILED"` and `"FILE_TOO_LARGE"` to `ErrorCode` type in `lib/api-response.ts`

## Dev Notes

### Architecture Context — This Story's Scope

This is the **first story** in the Voice Capture & AI Pipeline epic. It implements **recording + upload ONLY**. The processing pipeline is:

```
Story 6.1: Record → Upload to Blob → DB entry (status: "uploaded")
Story 6.2: + STT via Mistral Voxtral (status: "transcribed")
Story 6.3: + LLM Summarization (status: "done")
```

The `/api/voice` route is designed for extension — Stories 6.2/6.3 will add STT and LLM steps after the upload. For 6.1, the API returns immediately after upload. The client shows "Hochgeladen ✅" and navigates to dashboard.

### Vercel Blob — EU Region Setup

**Critical FFG requirement: All data must stay in EU (Frankfurt).**

```typescript
// lib/storage.ts
import { put, del } from "@vercel/blob";

export async function uploadAudio(
  userId: string,
  file: Blob,
  contentType: string
): Promise<string> {
  const blob = await put(
    `voice/${userId}/${crypto.randomUUID()}.webm`,
    file,
    {
      access: "public",
      contentType,
      addRandomSuffix: false,
      multipart: true, // Required: audio files can exceed 4.5MB
    }
  );
  return blob.url;
}

export async function deleteAudio(url: string): Promise<void> {
  await del(url);
}
```

**EU region is configured in Vercel Dashboard**, NOT in code:
1. Vercel Dashboard → Storage → Blob → Create Store
2. **Select Region: EU (Frankfurt)**
3. Copy `BLOB_READ_WRITE_TOKEN` to `.env.local` and Vercel env vars
4. The token determines which region is used

**Environment variable:**
```
BLOB_READ_WRITE_TOKEN=vercel_blob_xxx  # Server-side only, never NEXT_PUBLIC_*
```

### API Route: POST /api/voice

**Protected by middleware** — already configured in `middleware.ts`:
```typescript
export const config = {
  matcher: ["/app/:path*", "/api/voice/:path*"],
};
```

**Route pattern follows existing `app/api/auth/login/route.ts`:**

```typescript
// app/api/voice/route.ts
import { headers } from "next/headers";
import { uploadAudio } from "@/lib/storage";
import { db } from "@/lib/db";
import { voiceMessages } from "@/lib/db/schema";
import { successResponse, errorResponse } from "@/lib/api-response";

const MAX_FILE_SIZE = 25 * 1024 * 1024; // 25MB
const ACCEPTED_TYPES = ["audio/webm", "audio/wav", "audio/mp4", "audio/ogg"];

export async function POST(request: Request) {
  try {
    const headersList = await headers();
    const userId = headersList.get("x-user-id");
    if (!userId) {
      return errorResponse("UNAUTHORIZED", "Nicht authentifiziert.");
    }

    const formData = await request.formData();
    const audioFile = formData.get("audio");
    if (!(audioFile instanceof File)) {
      return errorResponse("VALIDATION_ERROR", "Keine Audiodatei gefunden.");
    }

    // Validate size
    if (audioFile.size > MAX_FILE_SIZE) {
      return errorResponse("FILE_TOO_LARGE", "Die Aufnahme ist zu groß (max. 25 MB).");
    }

    // Validate type
    const mimeBase = audioFile.type.split(";")[0]; // Strip codecs param
    if (!ACCEPTED_TYPES.includes(mimeBase)) {
      return errorResponse("VALIDATION_ERROR", "Ungültiges Audioformat.");
    }

    // Duration from client metadata (FormData field)
    const durationStr = formData.get("duration");
    const audioDurationSeconds = durationStr ? parseFloat(String(durationStr)) : null;

    // Upload to Vercel Blob
    const audioUrl = await uploadAudio(userId, audioFile, audioFile.type);

    // Create DB entry
    const [entry] = await db
      .insert(voiceMessages)
      .values({
        userId,
        audioUrl,
        audioDurationSeconds,
        status: "uploaded",
      })
      .returning();

    return successResponse({
      id: entry.id,
      status: entry.status,
      audioUrl: entry.audioUrl,
      audioDurationSeconds: entry.audioDurationSeconds,
    });
  } catch (error) {
    console.error(
      "Voice upload API error:",
      error instanceof Error ? `${error.name}: ${error.message}` : error
    );
    return errorResponse(
      "INTERNAL_ERROR",
      "Upload fehlgeschlagen. Bitte versuche es erneut."
    );
  }
}
```

**Important:** Use `request.formData()` (NOT `request.json()`) for multipart uploads.

### MediaRecorder — Cross-Browser Format Strategy

```typescript
export function getSupportedMimeType(): string {
  const types = [
    "audio/webm;codecs=opus", // Chrome, Firefox, modern Safari
    "audio/webm",              // Older WebM support
    "audio/mp4",               // Safari fallback
  ];
  return types.find((t) => MediaRecorder.isTypeSupported(t)) ?? "";
}
```

- **At 128kbps Opus, 5 minutes ≈ 4.8MB** — well within 25MB limit
- Use `mediaRecorder.start(10000)` — request data every 10s for memory safety
- **Do NOT use audio/wav** as primary — it's uncompressed (5 min ≈ 50MB, exceeds limit)

### Audio Level Meter — AnalyserNode Pattern

```typescript
const audioContext = new AudioContext();
const analyser = audioContext.createAnalyser();
analyser.fftSize = 2048;
const source = audioContext.createMediaStreamSource(stream);
source.connect(analyser);
// Do NOT connect analyser to destination (prevents echo)

const dataArray = new Uint8Array(analyser.fftSize);

function getVolume(): number {
  analyser.getByteTimeDomainData(dataArray);
  let sum = 0;
  for (let i = 0; i < dataArray.length; i++) {
    const v = (dataArray[i] - 128) / 128;
    sum += v * v;
  }
  return Math.sqrt(sum / dataArray.length); // RMS 0-1
}

// Update visual at 60fps via requestAnimationFrame — never from audio callback
```

**Critical:** `source.connect(analyser)` only — do NOT call `analyser.connect(audioContext.destination)` as this would play back the audio through speakers creating echo.

### Waveform Preview — Canvas from AudioBuffer

```typescript
async function generateWaveformData(blob: Blob, bars = 70): Promise<number[]> {
  const ctx = new AudioContext();
  const buffer = await ctx.decodeAudioData(await blob.arrayBuffer());
  const raw = buffer.getChannelData(0);
  const blockSize = Math.floor(raw.length / bars);
  const data: number[] = [];

  for (let i = 0; i < bars; i++) {
    let sum = 0;
    for (let j = 0; j < blockSize; j++) {
      sum += Math.abs(raw[blockSize * i + j]);
    }
    data.push(sum / blockSize);
  }

  const max = Math.max(...data);
  return data.map((v) => v / max); // Normalize to 0-1
}
```

**Audio playback for preview:** Use `AudioContext.decodeAudioData()` + `AudioBufferSourceNode` instead of `<audio>` element. Safari can't play WebM files via `<audio>` but Web Audio API decodes them fine.

### iOS Safari Specifics

- **Records `audio/webm;codecs=opus`** since iOS 14.3 — same as Chrome
- **Cannot play `.webm` via `<audio>`** — MUST use Web Audio API for preview playback
- **2.5ms frame duration** vs 20ms in Chrome — invisible to our code but affects internal encoding
- **Always use `MediaRecorder.isTypeSupported()`** — never assume formats

### Recording Page — State Machine

```
idle → (tap record) → requesting-permission → recording → (tap stop) → preview
                                                    ↓ (5 min)         ↓ (Senden)
                                              auto-stop → preview → uploading → done
                                                                      ↓ (error)
                                                                    error (retry available)
preview → (Verwerfen) → idle
preview → (Nochmal aufnehmen) → recording
```

### UX Specifications (from `docs/ux-sprint0-app-specs.md` Section 4)

**State A — Idle:**
- Mic button: 80x80px, round, Orange (`--accent`), centered, shadow for "pressable" feel
- Helper: "Zum Aufnehmen tippen" — use localStorage to track first use
- First visit: "Sprich einfach los — die KI kümmert sich um den Rest."
- Screen deliberately empty — reduction, invitation

**State C — Recording:**
- Button: 80x80px, stop icon (Square), Red (`--destructive`)
- Pulsing red dot: CSS animation (`animate-pulse`)
- Timer: counting up, monospace font (tabular-nums)
- Audio level bar: simple bar reacting to volume (CSS width transition)
- "Max. 5 Min" text: muted, bottom
- Subtle background tint to signal recording mode
- Bottom nav stays visible

**State D — Preview:**
- "Aufnahme fertig (0:23)" title with duration
- Waveform: Canvas-based, ~70 bars, play/pause button
- "Senden": Orange, primary button
- "Verwerfen": Ghost/outline, secondary
- "Nochmal aufnehmen": Text link, tertiary

**State E — Processing Indicator:**
- 3 steps: ✅ Hochgeladen → ⏳ Wird transkribiert → ○ Zusammenfassung
- ✅ = done (green), ⏳ = active (animated/orange), ○ = pending (muted)
- Indeterminate progress bar
- "Das dauert wenige Sekunden."
- User can navigate away

**State F — Error States:**

| Error | Message | Action |
|-------|---------|--------|
| No microphone | "Kein Mikrofon gefunden. Bitte prüfe deine Geräte-Einstellungen." | — |
| Permission denied | "Mikrofon-Zugriff nicht erlaubt. Aktiviere ihn in den Browser-Einstellungen." | — |
| Upload failed | "Upload fehlgeschlagen. Deine Aufnahme ist gespeichert." | [Nochmal versuchen] |

Amber banner design, NOT aggressive red. Warm tone, always offer a way out.

### Color Tokens (from UX spec Section 8)

| Element | Token |
|---------|-------|
| Record button (idle) | `--accent` (Orange) |
| Record button (recording) | `--destructive` (Red) |
| Success checkmarks | Use green (e.g., `text-green-600`) |
| Processing/Loading | `--accent` |
| Error banners | Amber (warm, not aggressive red) |

### Accessibility Requirements (from UX spec Section 9)

- Touch targets: ≥ 48x48px, Record button 80x80px
- WCAG AA contrast (4.5:1) on all text
- ARIA labels on all interactive elements
- Recording state as ARIA live region (`aria-live="assertive"`)
- `prefers-reduced-motion`: disable pulsing animation, use static indicator instead
- **Tap to Record** (NOT hold-to-record) — works with gloves

### Responsive Breakpoints

| Breakpoint | Changes |
|------------|---------|
| < 640px (Mobile) | Full width, 80px record button |
| 640-768px (Tablet) | More padding, same layout |
| ≥ 768px (Desktop) | Record button 64px, content centered max-width 720px |

### Existing Code Patterns to Follow

**API Route Pipeline** (from `app/api/auth/login/route.ts`):
1. Extract userId from headers (middleware-injected)
2. Parse request data (formData instead of JSON for this route)
3. Validate input
4. Business logic (upload + DB)
5. Return `successResponse()` or `errorResponse()`

**DB Insert** (from `lib/auth/session.ts`):
```typescript
const [entry] = await db
  .insert(voiceMessages)
  .values({ userId, audioUrl, status: "uploaded" })
  .returning();
```

**Error Response Pattern** (from `lib/api-response.ts`):
- Add `"UPLOAD_FAILED"` and `"FILE_TOO_LARGE"` to `ErrorCode` union type
- Add to `errorCodeToStatus` map: `UPLOAD_FAILED: 500`, `FILE_TOO_LARGE: 413`

### New Dependency: @vercel/blob

```bash
pnpm add @vercel/blob
```

Version: latest stable (v2.x). Only import `put` and `del` — no other features needed.

**IMPORTANT:** `@vercel/blob` is a server-side package. Never import in Client Components.

### Environment Variables

| Variable | Side | Description |
|----------|------|-------------|
| `BLOB_READ_WRITE_TOKEN` | Server | Vercel Blob token (EU region store) |

Add to `.env.local` and Vercel project environment variables.

### Project Structure Notes

**New files:**
```
lib/
├── storage.ts                         # NEW — uploadAudio, deleteAudio (Vercel Blob)
├── validations/
│   └── voice.ts                       # NEW — voice upload validation schema

app/
├── api/voice/
│   └── route.ts                       # NEW — POST handler for voice upload

components/
├── app/
│   ├── voice-recorder.tsx             # NEW — Recording logic + UI (Client Component)
│   ├── recording-preview.tsx          # NEW — Waveform + play/pause (Client Component)
│   └── processing-indicator.tsx       # NEW — 3-step progress (Client Component)
```

**Modified files:**
- `app/app/record/page.tsx` — Replace placeholder with functional recording page
- `lib/api-response.ts` — Add UPLOAD_FAILED, FILE_TOO_LARGE to ErrorCode

**No schema changes** — `voice_messages` table already exists from Story 5.1.

### Server vs Client Component Map

| File | Type | Reason |
|------|------|--------|
| `app/app/record/page.tsx` | Server Component (wrapper) or Client Component | Depends on whether we need server data; likely just imports VoiceRecorder |
| `components/app/voice-recorder.tsx` | Client Component | MediaRecorder, getUserMedia, state |
| `components/app/recording-preview.tsx` | Client Component | AudioContext, Canvas, playback |
| `components/app/processing-indicator.tsx` | Client Component | Animated state transitions |
| `app/api/voice/route.ts` | API Route (server) | Blob upload, DB insert |
| `lib/storage.ts` | Server utility | @vercel/blob operations |

### Previous Story Intelligence

**From Story 5.1 (Schema + App Shell):**
- Route group `(app)` does NOT work for URL paths → use `app/app/` instead
- `middleware.ts` at project root — already protects `/api/voice/:path*`
- `x-user-id` header injected by middleware for authenticated requests
- Logo + WaveformIcon in `components/shared/`
- Drizzle ORM: `.returning()` for getting auto-generated UUIDs back
- Index syntax: 3rd arg returns array

**From Story 5.2 (Magic Link Auth):**
- Turnstile: use `size: "normal"` with Managed widget (NOT invisible)
- Zod v4: `z.email()` top-level, NOT `z.string().email()`
- React Hook Form: fields managed via separate React state MUST NOT be in Zod schema
- Fire-and-forget pattern: `.catch(console.error)` for non-blocking side effects
- `errorCodeToStatus` map needs entries for all new ErrorCode values
- Keep German error messages warm/non-technical ("Etwas ist schiefgelaufen", not "500 Error")

**From Code Review (5.2):**
- Race conditions: use atomic UPDATE patterns where applicable
- Always wrap DB operations in try/catch in API routes
- Keep logout/cleanup operations safe — clear client state even if server call fails

### FFG Compliance Reminders

- **Audio files in EU ONLY** — Vercel Blob store must be Frankfurt region
- **No API keys in frontend** — BLOB_READ_WRITE_TOKEN is server-side only
- **DSGVO: 90-day audio retention** — `audioDeletedAt` field exists, cleanup cron in future story
- **Research documentation** — `audioDurationSeconds` tracked for FFG audit trail
- **Commit messages** — R&D character: "Implement voice recording with MediaRecorder API evaluation"

### References

- [Voice Pipeline: docs/architecture-sprint0-supplement.md#4-voice-processing-pipeline-synchronous]
- [Vercel Blob: docs/architecture-sprint0-supplement.md#5-vercel-blob-integration]
- [Storage Helper: docs/architecture-sprint0-supplement.md#5 → lib/storage.ts]
- [API Route: docs/architecture-sprint0-supplement.md#4 → POST /api/voice]
- [Recording UX: docs/ux-sprint0-app-specs.md#4-voice-recording-page]
- [Processing Indicator UX: docs/ux-sprint0-app-specs.md#4 → State E]
- [Error States UX: docs/ux-sprint0-app-specs.md#4 → State F]
- [Color Tokens: docs/ux-sprint0-app-specs.md#8-color-usage-im-app-bereich]
- [Accessibility: docs/ux-sprint0-app-specs.md#9-accessibility-field-conditions]
- [Responsive: docs/ux-sprint0-app-specs.md#7-responsive-breakpoints]
- [Schema: lib/db/schema.ts → voiceMessages table]
- [Middleware: middleware.ts → /api/voice/:path* protected]
- [API Response: lib/api-response.ts → successResponse, errorResponse]
- [Epics: _bmad-output/planning-artifacts/epics.md#story-61-voice-recording--upload]
- [Previous Story 5.1: _bmad-output/implementation-artifacts/5-1-database-schema-extension-and-app-shell.md]
- [Previous Story 5.2: _bmad-output/implementation-artifacts/5-2-magic-link-authentication.md]
- [FFG Rules: CLAUDE.md + docs/_masemIT/kurzum-bmad-do-dont-liste.md]

## Dev Agent Record

### Agent Model Used

Claude Opus 4.6

### Debug Log References

- React 19 lint: `react-hooks/refs` rule forbids ref access during render — converted `blobRef`/`durationRef` to state in page.tsx
- React 19 lint: `react-hooks` rule forbids synchronous setState in effects — used `useSyncExternalStore` for localStorage check, Tailwind `motion-safe:` variant for reduced motion
- Canvas `roundRect` used for waveform bars — supported in all modern browsers

### Completion Notes List

- ✅ Task 1: @vercel/blob@2.2.0 installed, BLOB_READ_WRITE_TOKEN added to .env.example and .env (placeholder — user must create Blob store in Vercel Dashboard EU Frankfurt)
- ✅ Task 2: lib/storage.ts created with uploadAudio() and deleteAudio() using @vercel/blob put/del
- ✅ Task 3: lib/validations/voice.ts created with Zod schema for file size (25MB) and MIME type validation
- ✅ Task 4: app/api/voice/route.ts created — POST handler with multipart/form-data parsing, validation, Blob upload, DB insert, error handling. UPLOAD_FAILED and FILE_TOO_LARGE added to ErrorCode type and errorCodeToStatus map.
- ✅ Task 5: VoiceRecorder component — MediaRecorder with WebM/Opus (MP4 fallback), getUserMedia permission handling, 5-min auto-stop, AnalyserNode + requestAnimationFrame level meter, useSyncExternalStore for first-visit detection
- ✅ Task 6: RecordingPreview component — Canvas-based waveform (~70 bars), Web Audio API playback (Safari WebM compatible), play/pause with progress tracking, Senden/Verwerfen/Nochmal buttons
- ✅ Task 7: ProcessingIndicator component — 3-step progress (Hochgeladen/Wird transkribiert/Zusammenfassung), only step 1 active for Story 6.1
- ✅ Task 8: Recording page replaced — full state machine (idle → recording → preview → uploading → done/error), warm amber error banners in German, retry for upload failures, auto-redirect to dashboard after success
- ✅ Task 9: ErrorCodes added in Task 4 (UPLOAD_FAILED: 500, FILE_TOO_LARGE: 413)

### File List

**New files:**
- lib/storage.ts — Vercel Blob upload/delete helpers
- lib/validations/voice.ts — Zod schema for voice upload validation
- app/api/voice/route.ts — POST /api/voice multipart upload handler
- components/app/voice-recorder.tsx — MediaRecorder + AnalyserNode recording component
- components/app/recording-preview.tsx — Waveform + Web Audio playback preview
- components/app/processing-indicator.tsx — 3-step processing progress display

**Modified files:**
- lib/api-response.ts — Added UPLOAD_FAILED, FILE_TOO_LARGE to ErrorCode type + errorCodeToStatus map
- app/app/record/page.tsx — Replaced placeholder with functional recording page
- .env.example — Added BLOB_READ_WRITE_TOKEN placeholder
- .env — Added BLOB_READ_WRITE_TOKEN placeholder
- package.json — Added @vercel/blob dependency
- pnpm-lock.yaml — Updated lockfile

### Change Log

- 2026-02-12: Implemented voice recording and upload (Story 6.1) — MediaRecorder API with WebM/Opus, Canvas waveform preview, Web Audio API playback, Vercel Blob upload, API route with validation
