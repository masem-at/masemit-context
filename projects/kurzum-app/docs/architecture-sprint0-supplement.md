# Architecture Supplement: Sprint 0 — Prototype

**Author:** Winston (Architect), BMAD Party Mode Review
**Date:** 2026-02-12
**Status:** Draft — pending team approval
**Base Document:** `_bmad-output/planning-artifacts/architecture.md` (2026-02-10)
**Briefing:** `docs/_masemIT/bmad-briefing-kurzum-sprint0.md`

---

## 1. Scope & Relationship to Base Architecture

The base architecture document covers:
- **Phase 1 (Detail):** Landing Page — COMPLETE, SHIPPED
- **Phase 2 (High-Level):** Full App — deferred until FFG approval

This supplement covers the **gap between Phase 1 and Phase 2:**
- **Sprint 0 (Detail):** Auth-protected prototype for voice → transcription → summary
- Timeline: KW 8–12 (17.02. – 16.03.2026)
- Goal: Demonstrable E2E prototype, research documentation for FFG

### Key Deviations from Base Architecture

| Topic | Base Architecture (Phase 2) | Sprint 0 Decision | Rationale |
|-------|---------------------------|-------------------|-----------|
| Auth | Auth.js v5 (recommended) | Eigenbau Magic Link | Simpler for E-Mail-only, fewer deps, full control. Port from ChainSights/TellingCube |
| Queue | QStash for async AI pipeline | Synchronous (await) | Prototype scale, <30s expected latency, YAGNI |
| Audio Storage | TBD (file storage provider) | Vercel Blob (EU) | Simplest in Vercel ecosystem, EU-region available |
| NeonDB | Separate project possible | Extend existing project | Waitlist data already there, cost savings, shared billing |
| Testing | Vitest + Playwright | Manual + curl smoke tests | Research focus, not production-readiness |
| Branch Strategy | develop → Preview | Trunk-based (main + feature branches) | Solo/small team, short iterations |

---

## 2. Authentication Flow: Magic Link

### Design Principles
- **No external auth library** — custom implementation, ported from existing masemIT project
- **E-Mail only** — no social logins, no passwords
- **httpOnly Cookie** — secure session management
- **Middleware-enforced** — `/app/*` routes protected at edge

### Sequence Diagram

```
User                    Browser              Server                Resend           NeonDB
  │                        │                    │                    │                 │
  ├─ Enter email ─────────►│                    │                    │                 │
  │                        ├─ POST /api/auth/login ───────────────►│                 │
  │                        │  { email, turnstileToken }             │                 │
  │                        │                    ├─ Verify Turnstile  │                 │
  │                        │                    ├─ Rate limit check  │                 │
  │                        │                    ├─ Find/create user ─┼────────────────►│
  │                        │                    ├─ Generate token ───┼────────────────►│
  │                        │                    │  (UUID v4, 15min TTL)               │
  │                        │                    ├─ Send magic link ──►│                │
  │                        │                    │                    ├─ Email delivered │
  │                        │◄─ 200 OK ──────────┤                    │                 │
  │                        │  "Prüfe dein Postfach!"                │                 │
  │                        │                    │                    │                 │
  ├─ Click email link ────►│                    │                    │                 │
  │  /api/auth/verify?token=xxx                 │                    │                 │
  │                        ├─ GET /api/auth/verify?token=xxx ──────►│                 │
  │                        │                    ├─ Lookup token ─────┼────────────────►│
  │                        │                    ├─ Validate: not expired, not used     │
  │                        │                    ├─ Mark token used ──┼────────────────►│
  │                        │                    ├─ Create session ───┼────────────────►│
  │                        │                    │  (UUID, 7d expiry)                   │
  │                        │◄─ Set-Cookie: session=xxx (httpOnly, Secure, SameSite=Lax)│
  │                        │◄─ 302 Redirect → /app                  │                 │
  │                        │                    │                    │                 │
  ├─ Access /app ─────────►│                    │                    │                 │
  │                        ├─ Request with cookie ─────────────────►│                 │
  │                        │                    ├─ Middleware: validate session ──────►│
  │                        │                    ├─ Inject userId into request headers  │
  │                        │◄─ 200 OK (app page) ──────────────────┤                 │
```

### Session Schema

```typescript
// Addition to lib/db/schema.ts
export const sessions = pgTable('sessions', {
  id: uuid('id').primaryKey().defaultRandom(),
  userId: uuid('user_id').references(() => users.id).notNull(),
  token: varchar('token', { length: 255 }).notNull().unique(),
  expiresAt: timestamp('expires_at', { withTimezone: true }).notNull(),
  createdAt: timestamp('created_at', { withTimezone: true }).defaultNow(),
});
```

### Security Constraints

| Aspect | Implementation |
|--------|---------------|
| Token entropy | UUID v4 (122 bits) |
| Token expiry | 15 minutes (magic link), 7 days (session) |
| Token reuse | Single-use, marked `used_at` on verification |
| Cookie flags | `httpOnly`, `Secure`, `SameSite=Lax`, `Path=/` |
| Rate limiting | 5 magic links per email per hour (Upstash) |
| Brute force | Turnstile on login form |
| Session storage | DB-backed (not JWT) — enables server-side revocation |

### Why DB Sessions, Not JWT

For the prototype: DB sessions are simpler to implement, support instant revocation (logout = delete row), and we already have NeonDB. JWT would require refresh token rotation and can't be revoked without a blocklist (which is just a DB session by another name).

---

## 3. Route Structure

### Public vs. Protected Routes

```
app/
├── page.tsx                    # Landing page (PUBLIC)
├── confirmed/page.tsx          # Double-opt-in confirmation (PUBLIC)
├── datenschutz/page.tsx        # Privacy policy (PUBLIC)
├── impressum/page.tsx          # Legal notice (PUBLIC)
├── login/                      # Login page (PUBLIC, redirects if already authed)
│   └── page.tsx
├── login/check-email/          # "Check your inbox" page (PUBLIC)
│   └── page.tsx
│
├── (app)/                      # PROTECTED route group — all behind auth middleware
│   ├── layout.tsx              # App shell: header with logo + logout, bottom nav (mobile)
│   ├── page.tsx                # Dashboard: recent voice messages + summaries
│   └── record/
│       └── page.tsx            # Voice recording page (could also be modal on dashboard)
│
├── api/
│   ├── waitlist/route.ts       # Waitlist signup (PUBLIC, Turnstile-protected)
│   ├── unsubscribe/route.ts    # Unsubscribe (PUBLIC, token-based)
│   ├── auth/
│   │   ├── login/route.ts      # POST: send magic link (PUBLIC, rate-limited)
│   │   ├── verify/route.ts     # GET: verify magic link token, set session cookie
│   │   └── logout/route.ts     # POST: destroy session
│   └── voice/
│       └── route.ts            # POST: upload + process voice message (PROTECTED)
```

### Middleware

```typescript
// middleware.ts
// Protects all /app/* routes
// Reads session cookie → validates against DB → injects userId header
// Redirects to /login if no valid session

export const config = {
  matcher: ['/(app)/:path*', '/api/voice/:path*'],
};
```

**Important:** API routes under `/api/voice/*` are also protected by middleware. Auth API routes (`/api/auth/*`) and waitlist routes are public.

---

## 4. Voice Processing Pipeline (Synchronous)

### Flow

```
┌──────────┐     ┌──────────────┐     ┌───────────┐     ┌───────────┐     ┌────────┐
│  Browser  │────►│ POST /api/   │────►│  Vercel   │────►│  Mistral  │────►│ NeonDB │
│ Recording │     │    voice     │     │   Blob    │     │ Voxtral + │     │  Save  │
│ (WebM/    │     │              │     │  Upload   │     │   LLM     │     │ Result │
│  Opus)    │     │  ~2-5 MB     │     │  (EU)     │     │  (EU)     │     │        │
└──────────┘     └──────────────┘     └───────────┘     └───────────┘     └────────┘
                        │                                                       │
                        │◄──────────── JSON Response with transcript + summary ─┘
```

### API: POST /api/voice

```typescript
// Request: multipart/form-data with audio file
// Response: ApiResponse<VoiceMessageResult>

interface VoiceMessageResult {
  id: string;            // UUID
  status: 'done' | 'error';
  transcript: string | null;
  summary: VoiceSummary | null;
  sttProvider: string;
  sttDurationMs: number;
  llmProvider: string;
  llmDurationMs: number;
}

interface VoiceSummary {
  status: string;         // "FI-Schalter im OG getauscht"
  material: string[];     // ["3x Sicherungsautomat B16"]
  nextSteps: string;      // "Morgen Zählerkasten im EG"
  problems: string | null;
  project: string;        // "Müller, Hauptstraße 5"
}
```

### Processing Steps (Server-Side)

```typescript
// Pseudocode — app/api/voice/route.ts
export async function POST(req: Request) {
  // 1. Auth: userId from middleware
  const userId = req.headers.get('x-user-id');

  // 2. Parse multipart form data
  const formData = await req.formData();
  const audioFile = formData.get('audio') as File;

  // 3. Validate: size ≤ 25MB, type audio/webm or audio/wav
  // 4. Upload to Vercel Blob (EU region)
  const blob = await put(`voice/${userId}/${uuid()}.webm`, audioFile, {
    access: 'public', // signed URL alternative for private access
    addRandomSuffix: false,
  });

  // 5. Create DB entry with status 'processing'
  const [entry] = await db.insert(voiceMessages).values({
    userId,
    audioUrl: blob.url,
    status: 'processing',
  }).returning();

  // 6. STT: Mistral Voxtral
  const sttStart = Date.now();
  const transcript = await mistralSTT(blob.url);
  const sttDurationMs = Date.now() - sttStart;

  // 7. LLM: Mistral summarization
  const llmStart = Date.now();
  const summary = await mistralSummarize(transcript);
  const llmDurationMs = Date.now() - llmStart;

  // 8. Update DB with results
  await db.update(voiceMessages)
    .set({
      transcript,
      summary,
      sttProvider: 'voxtral',
      sttDurationMs,
      llmProvider: 'mistral-large',
      llmDurationMs,
      status: 'done',
      processedAt: sql`now()`,
    })
    .where(eq(voiceMessages.id, entry.id));

  // 9. Return result
  return Response.json({ success: true, data: { id: entry.id, status: 'done', transcript, summary, ... } });
}
```

### Timeout Consideration

Vercel serverless functions have a **60-second timeout** on the Pro plan. Expected processing:
- Upload: ~1-2s (5MB file)
- STT: ~3-8s (30s audio)
- LLM: ~2-5s (summarization)
- Total: ~6-15s — well within limits

If STT latency exceeds expectations, the fallback is splitting into two requests (upload → poll), not QStash. QStash only if we need background processing beyond 60s.

---

## 5. Vercel Blob Integration

### Configuration

```typescript
// lib/storage.ts
import { put, del, list } from '@vercel/blob';

// Upload audio
export async function uploadAudio(userId: string, file: File): Promise<string> {
  const blob = await put(
    `voice/${userId}/${crypto.randomUUID()}.webm`,
    file,
    { access: 'public', addRandomSuffix: false }
  );
  return blob.url;
}

// Delete audio (DSGVO retention)
export async function deleteAudio(url: string): Promise<void> {
  await del(url);
}
```

### EU Region

Vercel Blob supports EU region via the `BLOB_READ_WRITE_TOKEN` environment variable. The token determines the region. Ensure the Blob store is created in **EU (Frankfurt)** via Vercel Dashboard → Storage → Blob → Create Store → Region: EU.

### Retention Policy (DSGVO)

- Audio files are deleted after 90 days
- Sprint 0: Manual deletion or simple cron job (Vercel Cron)
- `audioDeletedAt` timestamp tracked in `voice_messages` table
- Transcript and summary persist (text only, no PII audio data)

```typescript
// Future: app/api/cron/cleanup-audio/route.ts
// Vercel Cron: runs daily
// SELECT * FROM voice_messages WHERE audio_deleted_at IS NULL AND created_at < now() - interval '90 days'
// For each: delete from Vercel Blob, set audio_deleted_at
```

---

## 6. Schema Additions for Sprint 0

All new tables extend the existing NeonDB project alongside `waitlist_entries`.

```typescript
// lib/db/schema.ts — Sprint 0 additions

import { pgTable, uuid, varchar, text, integer, timestamp, jsonb } from 'drizzle-orm/pg-core';

// ── Users ──────────────────────────────────────────────
export const users = pgTable('users', {
  id: uuid('id').primaryKey().defaultRandom(),
  email: varchar('email', { length: 255 }).notNull().unique(),
  name: varchar('name', { length: 255 }),
  role: varchar('role', { length: 50 }).default('user'),
  createdAt: timestamp('created_at', { withTimezone: true }).defaultNow(),
  lastLoginAt: timestamp('last_login_at', { withTimezone: true }),
});

// ── Magic Links ────────────────────────────────────────
export const magicLinks = pgTable('magic_links', {
  id: uuid('id').primaryKey().defaultRandom(),
  userId: uuid('user_id').references(() => users.id).notNull(),
  token: varchar('token', { length: 255 }).notNull().unique(),
  expiresAt: timestamp('expires_at', { withTimezone: true }).notNull(),
  usedAt: timestamp('used_at', { withTimezone: true }),
  createdAt: timestamp('created_at', { withTimezone: true }).defaultNow(),
});

// ── Sessions ───────────────────────────────────────────
export const sessions = pgTable('sessions', {
  id: uuid('id').primaryKey().defaultRandom(),
  userId: uuid('user_id').references(() => users.id).notNull(),
  token: varchar('token', { length: 255 }).notNull().unique(),
  expiresAt: timestamp('expires_at', { withTimezone: true }).notNull(),
  createdAt: timestamp('created_at', { withTimezone: true }).defaultNow(),
});

// ── Voice Messages ─────────────────────────────────────
export const voiceMessages = pgTable('voice_messages', {
  id: uuid('id').primaryKey().defaultRandom(),
  userId: uuid('user_id').references(() => users.id).notNull(),
  audioUrl: varchar('audio_url', { length: 500 }),
  audioDurationSeconds: integer('audio_duration_seconds'),
  transcript: text('transcript'),
  summary: jsonb('summary').$type<VoiceSummary | null>(),
  sttProvider: varchar('stt_provider', { length: 50 }),
  sttDurationMs: integer('stt_duration_ms'),
  llmProvider: varchar('llm_provider', { length: 50 }),
  llmDurationMs: integer('llm_duration_ms'),
  status: varchar('status', { length: 20 }).default('pending'),
  createdAt: timestamp('created_at', { withTimezone: true }).defaultNow(),
  processedAt: timestamp('processed_at', { withTimezone: true }),
  audioDeletedAt: timestamp('audio_deleted_at', { withTimezone: true }),
});

interface VoiceSummary {
  status: string;
  material: string[];
  nextSteps: string;
  problems: string | null;
  project: string;
}
```

### Migration Strategy

- Use `drizzle-kit push` for dev iteration
- Use `drizzle-kit generate` + `drizzle-kit migrate` for production
- Existing `waitlist_entries` table remains untouched
- New tables added incrementally per AP

---

## 7. Service Integration: Mistral AI

### Client Setup

```typescript
// lib/ai/mistral.ts
const MISTRAL_API_KEY = process.env.MISTRAL_API_KEY!;
const MISTRAL_BASE_URL = 'https://api.mistral.ai/v1';

// STT via Voxtral
export async function transcribe(audioUrl: string): Promise<string> {
  // Mistral Voxtral API call
  // Returns: plain text transcript
}

// LLM Summarization
export async function summarize(transcript: string): Promise<VoiceSummary> {
  // Mistral chat/completions API
  // System prompt: Handwerker-context, JSON output
  // Returns: parsed VoiceSummary
}
```

### Environment Variable

```bash
MISTRAL_API_KEY=xxx  # Server-side only, NEVER in NEXT_PUBLIC_*
```

### Research Documentation Pattern

Every Mistral API call should log:
- Provider + model used
- Input size (audio duration or transcript length)
- Processing duration (ms)
- Output quality assessment (when manually reviewed)

This data feeds into the FFG research documentation (Forschungstagebuch).

---

## 8. Open Items for Sally (UX)

The following UX decisions need Sally's input before implementation:

1. **Login Flow States:** Login form → "Check email" → Magic link click → Redirect to /app. What happens on expired/invalid token?
2. **Recording UI States:** Idle → Permission prompt → Recording (with timer + waveform) → Preview (play/pause) → Uploading → Processing → Done. Mobile-first, one-thumb operation.
3. **Dashboard Card Design:** How does a voice message summary card look? Expandable? Swipeable? What info is visible at a glance vs. expanded?
4. **Error States:** No microphone, permission denied, upload failed, STT failed, LLM failed — each needs a distinct, user-friendly message in German.
5. **Empty State:** First login, no messages yet — what does the user see?
6. **Processing Feedback:** While STT + LLM runs (~6-15s), what does the user see? Progress bar? Animated spinner? Steps indicator?
