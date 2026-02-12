# Story 5.1: Database Schema Extension & App Shell

Status: done

## Story

As a **developer**,
I want the database extended with users, sessions, magic_links, and voice_messages tables, plus a protected app shell with 2-tab navigation,
So that all Sprint 0 features have a foundation to build on.

## Acceptance Criteria

1. **Given** the existing NeonDB project with `waitlist_entries`, **When** the Sprint 0 schema is added to `lib/db/schema.ts`, **Then** four new tables are defined: `users`, `magic_links`, `sessions`, `voice_messages` — all with UUID primary keys (`defaultRandom()`), all timestamps with `withTimezone: true`.

2. **Given** the `voice_messages` table, **When** it is defined, **Then** it includes `sttProvider`, `sttDurationMs`, `llmProvider`, `llmDurationMs` fields for FFG research documentation, and `audioDeletedAt` for DSGVO retention tracking.

3. **Given** the new schema, **When** `drizzle-kit generate` is run, **Then** a migration file is generated and committed to git in `drizzle/`.

4. **Given** the schema exists, **When** the app route group is created at `app/(app)/`, **Then** a `layout.tsx` renders the app shell: header (logo + user name + logout) + page content + bottom navigation.

5. **Given** the app shell layout, **When** the bottom navigation renders, **Then** it has 2 tabs: Aufnahme | Übersicht, with active tab showing Orange underline + filled icon, inactive showing muted outline. Bottom nav is sticky, min 48px height per tab.

6. **Given** the app shell, **When** viewed on desktop (≥768px), **Then** bottom nav is replaced by header navigation.

7. **Given** the app shell exists, **When** Next.js middleware is configured, **Then** all routes matching `/(app)/:path*` and `/api/voice/:path*` require a valid session cookie. Unauthenticated requests redirect to `/login`.

8. **Given** middleware validates sessions, **When** a request has a valid session cookie, **Then** the `userId` is injected into request headers (`x-user-id`) for downstream use.

9. **Given** the login page route exists at `/login`, **When** a user with a valid session visits `/login`, **Then** they are automatically redirected to `/app`.

10. **Given** the login page route, **When** an unauthenticated user visits, **Then** they see a placeholder login page (full implementation in Story 5.2).

## Tasks / Subtasks

- [x] **Task 1: Extend Database Schema** (AC: #1, #2)
  - [x] 1.1 Add `users` table to `lib/db/schema.ts` with UUID PK, email (unique), name, role (default 'user'), createdAt, lastLoginAt
  - [x] 1.2 Add `magicLinks` table with UUID PK, userId FK → users.id, token (unique), expiresAt, usedAt, createdAt
  - [x] 1.3 Add `sessions` table with UUID PK, userId FK → users.id, token (unique), expiresAt, createdAt
  - [x] 1.4 Add `voiceMessages` table with UUID PK, userId FK → users.id, audioUrl, audioDurationSeconds, transcript, summary (jsonb with VoiceSummary type), sttProvider, sttDurationMs, llmProvider, llmDurationMs, status (default 'pending'), createdAt, processedAt, audioDeletedAt
  - [x] 1.5 Add indexes: `idx_users_email`, `idx_magic_links_token`, `idx_sessions_token`, `idx_sessions_user_id`, `idx_voice_messages_user_id`, `idx_voice_messages_status`
  - [x] 1.6 Export TypeScript types: `User`, `NewUser`, `MagicLink`, `NewMagicLink`, `Session`, `NewSession`, `VoiceMessage`, `NewVoiceMessage`, `VoiceSummary` (interface)

- [x] **Task 2: Generate and Apply Migration** (AC: #3)
  - [x] 2.1 Run `npx drizzle-kit generate` to create migration SQL
  - [x] 2.2 Review generated SQL for correctness
  - [x] 2.3 Run `npx drizzle-kit push` to apply to dev database (or `drizzle-kit migrate` for production)
  - [x] 2.4 Verify tables created in NeonDB

- [x] **Task 3: Create App Route Group & Shell Layout** (AC: #4, #5, #6)
  - [x] 3.1 Create `app/app/layout.tsx` — app shell with header + mobile bottom nav + desktop header nav
  - [x] 3.2 Create `components/app/app-header.tsx` — logo (links to /app) + user name + logout button
  - [x] 3.3 Create `components/app/bottom-nav.tsx` — 2-tab sticky navigation (Client Component)
  - [x] 3.4 Create `app/app/page.tsx` — placeholder dashboard page ("Übersicht" tab target)
  - [x] 3.5 Create `app/app/record/page.tsx` — placeholder recording page ("Aufnahme" tab target)

- [x] **Task 4: Implement Auth Middleware** (AC: #7, #8)
  - [x] 4.1 Create `middleware.ts` at project root
  - [x] 4.2 Configure `matcher` for `/app/:path*` and `/api/voice/:path*`
  - [x] 4.3 Implement session cookie reading (`session` cookie name)
  - [x] 4.4 Implement session validation against `sessions` table (check token exists, not expired)
  - [x] 4.5 Inject `x-user-id` header on valid session
  - [x] 4.6 Redirect to `/login` on invalid/missing session
  - [x] 4.7 Create `lib/auth/session.ts` — session validation helper for middleware

- [x] **Task 5: Create Login Page Placeholder** (AC: #9, #10)
  - [x] 5.1 Create `app/login/page.tsx` — placeholder with kurzum logo, tagline, email input (disabled), hint text
  - [x] 5.2 Create `app/login/check-email/page.tsx` — placeholder "Prüfe dein Postfach!" page
  - [x] 5.3 Implement session check in login page — redirect to `/app` if already authenticated
  - [x] 5.4 Add `/login` route to `app/sitemap.ts` exclusion and `app/robots.ts` disallow

- [x] **Task 6: Add New Environment Variables** (AC: all)
  - [x] 6.1 No new env vars needed — SESSION_COOKIE_NAME hardcoded as "session", DATABASE_URL unchanged
  - [x] 6.2 Verify existing DATABASE_URL works for new tables — confirmed via drizzle-kit push

## Dev Notes

### Schema Design — Exact Specifications

The schema is defined in `docs/architecture-sprint0-supplement.md` Section 6. Follow it exactly:

```typescript
// VoiceSummary interface — define BEFORE the table
export interface VoiceSummary {
  status: string;         // "FI-Schalter im OG getauscht"
  material: string[];     // ["3x Sicherungsautomat B16"]
  nextSteps: string;      // "Morgen Zählerkasten im EG"
  problems: string | null;
  project: string;        // "Müller, Hauptstraße 5"
}
```

**Critical schema patterns from existing codebase:**
- Existing `waitlistEntries` uses `serial("id").primaryKey()` — new tables use `uuid("id").primaryKey().defaultRandom()`
- Both patterns coexist fine in the same schema file
- Index syntax: 3rd arg returns array — `(table) => [index(...).on(table.col)]`
- Timestamps: always `{ withTimezone: true }`
- Drizzle ORM version: 0.45.1

**JSONB column pattern:**
```typescript
summary: jsonb("summary").$type<VoiceSummary | null>(),
```

**Foreign key pattern:**
```typescript
userId: uuid("user_id").references(() => users.id).notNull(),
```

### App Shell — UX Specifications

Source: `docs/ux-sprint0-app-specs.md` Sections 3, 7, 8.

**Header (sticky, ~56px):**
- Left: `kurzum.` logo (link to `/app`)
- Right: user's `name` (from session/user lookup) + logout icon button
- No hamburger menu, no dropdown

**Bottom Navigation (mobile < 768px):**
- Sticky at bottom
- 2 tabs, full width per tab, min 48px height
- Active: Orange underline + filled icon (`--accent`)
- Inactive: muted color + outline icon (`--muted-foreground`)
- Tab 1: 🎙️ Aufnahme → `/app/record`
- Tab 2: 📋 Übersicht → `/app`

**Desktop (≥768px):**
- Bottom nav hidden
- 2 navigation links in header (between logo and user info)
- Content centered, max-width ~720px

**Color tokens to use:**
- Record icon: `--accent` (Orange)
- Active tab underline: `--accent`
- Inactive tab: `--muted-foreground`
- Header background: `--card` (white)
- Page background: `--background` (Stone 50)
- Logout icon: `--muted-foreground`

### Middleware Architecture

Source: `docs/architecture-sprint0-supplement.md` Section 3.

**File: `middleware.ts` at project root**

```typescript
export const config = {
  matcher: ['/(app)/:path*', '/api/voice/:path*'],
};
```

**Session validation flow:**
1. Read `session` cookie from request
2. If no cookie → redirect to `/login`
3. If cookie → look up session token in `sessions` table
4. If session not found or expired → redirect to `/login`
5. If valid → set `x-user-id` header, continue

**Important: Middleware DB access.**
The middleware runs at the edge. We use `drizzle-orm/neon-http` which works in serverless/edge environments. The existing `lib/db/index.ts` uses this driver already. Create a lightweight session validation function in `lib/auth/session.ts` that queries the sessions table directly.

**Note on Next.js 16:** Verify whether `middleware.ts` still works or needs to be renamed to `proxy.ts` (reported change in Next.js 16). Start with `middleware.ts` — if build fails, rename to `proxy.ts` and change export from `middleware()` to `proxy()`.

### Login Page — Placeholder Only

Story 5.2 implements the full login flow. For 5.1, the login page needs:
- kurzum logo + "Sprich statt tipp." tagline
- Email input field (visually present but not functional yet)
- "Login-Link senden" button (disabled, Orange)
- Hint text: "Kein Passwort nötig. Wir senden dir einen Einmal-Link per E-Mail."
- Session check: if valid session cookie → redirect to `/app`

Layout: vertically centered, max-width 400px, same visual style as landing page.

### Existing Code Patterns to Follow

**DB connection (lib/db/index.ts):**
```typescript
import { drizzle } from "drizzle-orm/neon-http";
import * as schema from "./schema";
export const db = drizzle(process.env.DATABASE_URL!, { schema });
```
→ No changes needed. The `* as schema` import automatically picks up new tables.

**File naming:** `kebab-case.tsx` for components, `PascalCase` exports.

**Component organization:**
- `components/app/` — new directory for app-area components (parallel to `components/landing/`)
- `components/shared/` — for Logo component (already shared between landing and app)

**Logo component reuse:** The Logo component is at `components/landing/logo.tsx` (with `WaveformIcon` at `components/landing/waveform-icon.tsx`). For Sprint 0, either:
- **Option A (recommended):** Move both to `components/shared/` so they're importable from both landing and app contexts
- **Option B:** Import directly from `@/components/landing/logo` (works but semantically wrong per architecture boundaries)

Logo props: `variant="light"` for app header (white bg), `size="sm"` or `"md"`, `showWaveform={true}`. It uses `@/lib/utils` for `cn()`.

**Server vs Client components:**
- `app/(app)/layout.tsx` — Server Component (fetches user data)
- `components/app/app-header.tsx` — Server Component (receives user as prop)
- `components/app/bottom-nav.tsx` — Client Component (uses `usePathname()` for active tab)
- `app/(app)/page.tsx` — Server Component (placeholder)
- `app/(app)/record/page.tsx` — Server Component (placeholder)
- `app/login/page.tsx` — Server Component (session check + redirect)
- `middleware.ts` — Edge middleware

### Migration Strategy

- Use `npx drizzle-kit generate` to create SQL migration file
- Migration goes to `drizzle/` directory (configured in `drizzle.config.ts`)
- Apply to NeonDB dev: `npx drizzle-kit push` (fast iteration)
- For production: `npx drizzle-kit migrate`
- **Existing `waitlist_entries` table remains untouched**

### FFG Research Fields in voice_messages

These fields are required for FFG audit documentation:
- `sttProvider` — which STT model processed this (e.g., "voxtral", "whisper")
- `sttDurationMs` — processing time for transcription
- `llmProvider` — which LLM summarized (e.g., "mistral-large")
- `llmDurationMs` — processing time for summarization
- `audioDurationSeconds` — length of original recording

This data feeds into `docs/research/` evaluation documents (Stories 6.2, 6.3).

### DSGVO Retention in voice_messages

`audioDeletedAt` tracks when the audio file was deleted from Vercel Blob:
- Audio files are deleted after 90 days (future cron job)
- Transcript and summary persist (text only, no PII audio data)
- When audio is deleted: set `audioDeletedAt = now()`, `audioUrl = null`

### Project Structure Notes

**New files to create:**
```
app/
├── (app)/                      # PROTECTED route group (NEW)
│   ├── layout.tsx              # App shell layout (NEW)
│   ├── page.tsx                # Dashboard placeholder (NEW)
│   └── record/
│       └── page.tsx            # Recording placeholder (NEW)
├── login/                      # Login page (NEW)
│   ├── page.tsx                # Login form placeholder (NEW)
│   └── check-email/
│       └── page.tsx            # Check email placeholder (NEW)
│
middleware.ts                   # Auth middleware (NEW, project root)

components/
├── app/                        # App-area components (NEW directory)
│   ├── app-header.tsx          # Header with logo + user + logout (NEW)
│   └── bottom-nav.tsx          # 2-tab mobile navigation (NEW)

lib/
├── auth/                       # Auth utilities (NEW directory)
│   └── session.ts              # Session validation for middleware (NEW)
├── db/
│   └── schema.ts               # EXTEND with 4 new tables + VoiceSummary interface
```

**Files to modify:**
- `lib/db/schema.ts` — add 4 tables + types + interface
- `app/robots.ts` — disallow `/app/*`, `/login`
- `app/sitemap.ts` — exclude `/app/*`, `/login`

**Alignment with base architecture:**
- `components/app/` directory per architecture doc Phase 2 structure
- `lib/auth/` per architecture doc Phase 2 structure
- Route group `(app)` matches architecture doc Section 3

### References

- [Schema: docs/architecture-sprint0-supplement.md#6-schema-additions-for-sprint-0]
- [Auth Flow: docs/architecture-sprint0-supplement.md#2-authentication-flow-magic-link]
- [Route Structure: docs/architecture-sprint0-supplement.md#3-route-structure]
- [Middleware: docs/architecture-sprint0-supplement.md#3-route-structure → Middleware]
- [App Shell UX: docs/ux-sprint0-app-specs.md#3-app-shell--navigation]
- [Recording Page UX: docs/ux-sprint0-app-specs.md#4-voice-recording-page]
- [Dashboard UX: docs/ux-sprint0-app-specs.md#5-dashboard--ubersicht]
- [Empty State UX: docs/ux-sprint0-app-specs.md#6-empty-state-erster-login]
- [Responsive: docs/ux-sprint0-app-specs.md#7-responsive-breakpoints]
- [Colors: docs/ux-sprint0-app-specs.md#8-color-usage-im-app-bereich]
- [Accessibility: docs/ux-sprint0-app-specs.md#9-accessibility-field-conditions]
- [Epics: _bmad-output/planning-artifacts/epics.md#story-51-database-schema-extension--app-shell]
- [FFG Rules: CLAUDE.md + docs/_masemIT/kurzum-bmad-do-dont-liste.md]
- [Existing Schema Pattern: lib/db/schema.ts]
- [Existing DB Connection: lib/db/index.ts]
- [Briefing: docs/_masemIT/bmad-briefing-kurzum-sprint0.md]

## Dev Agent Record

### Agent Model Used

Claude Opus 4.6 (claude-opus-4-6)

### Debug Log References

- Route group `(app)` does NOT create `/app` URL prefix in Next.js — it's a layout-only grouping. Changed to `app/app/` to get actual `/app` URL paths.
- Middleware matcher updated from `/(app)/:path*` to `/app/:path*` accordingly.
- Next.js 16.1.6 shows deprecation warning for `middleware.ts` → recommends `proxy.ts`. Build succeeds with `middleware.ts`; no rename needed per Dev Notes ("start with middleware.ts — if build fails, rename").
- Logo + WaveformIcon moved to `components/shared/` (Option A); landing originals re-export for backward compatibility.

### Completion Notes List

- **Task 1:** Extended `lib/db/schema.ts` with 4 new tables (users, magic_links, sessions, voice_messages), VoiceSummary interface, 6 indexes, and all TypeScript types exported
- **Task 2:** Migration generated (`drizzle/0001_parallel_nighthawk.sql`) and pushed to NeonDB. All 5 tables verified via Neon MCP
- **Task 3:** App shell created with sticky header (logo + user name + logout), 2-tab bottom nav (mobile), header nav (desktop), content max-width 720px. Placeholder pages for Übersicht and Aufnahme
- **Task 4:** Middleware at project root validates session cookie, redirects to /login on invalid/missing, injects x-user-id header. Session helper in lib/auth/session.ts
- **Task 5:** Login placeholder with logo, tagline, disabled email/button, hint text. Check-email placeholder. Session redirect to /app. robots.ts updated with /app and /login disallow
- **Task 6:** No new env vars needed; DATABASE_URL covers all new tables

### Change Log

- 2026-02-12: Story 5.1 implementation — database schema extension (4 tables), app shell with auth middleware, login placeholder
- 2026-02-12: Code review (Opus 4.6) — 5 MEDIUM issues fixed: middleware error handling, desktop nav active state, filled/outline icon toggle, logout stub route, stray nul file deleted

### File List

**New files:**
- `lib/db/schema.ts` (modified — 4 new tables, VoiceSummary interface, types)
- `drizzle/0001_parallel_nighthawk.sql` (generated migration)
- `app/app/layout.tsx` (app shell layout — Server Component)
- `app/app/page.tsx` (dashboard placeholder)
- `app/app/record/page.tsx` (recording placeholder)
- `components/app/app-header.tsx` (header with logo + user + logout)
- `components/app/bottom-nav.tsx` (2-tab mobile nav — Client Component)
- `components/shared/logo.tsx` (moved from landing)
- `components/shared/waveform-icon.tsx` (moved from landing)
- `middleware.ts` (auth middleware)
- `lib/auth/session.ts` (session validation helper)
- `app/login/page.tsx` (login placeholder with session redirect)
- `app/login/check-email/page.tsx` (check email placeholder)

**Modified files:**
- `components/landing/logo.tsx` (re-export from shared)
- `components/landing/waveform-icon.tsx` (re-export from shared)
- `app/robots.ts` (added /app, /login to disallow)

**Added during code review:**
- `app/api/auth/logout/route.ts` (stub logout route — clears cookie, redirects to /login)

## Senior Developer Review (AI)

**Reviewer:** Claude Opus 4.6
**Date:** 2026-02-12
**Outcome:** Approved (after fixes)

### Findings (5 MEDIUM, 3 LOW)

**Fixed (MEDIUM):**
1. **M1:** Middleware error handling — wrapped `validateSession()` in try/catch to prevent 500 on DB errors
2. **M2:** Desktop header nav missing active state — converted `AppHeader` to Client Component with `usePathname()`, active links show accent color + font-medium
3. **M3:** Bottom nav filled/outline icons — added `fill={isActive ? "currentColor" : "none"}` to Lucide icons for visual active/inactive distinction (matching AC#5)
4. **M4:** Logout button 404 — created stub `app/api/auth/logout/route.ts` that clears session cookie and redirects to /login
5. **M5:** Stray `nul` file — deleted Windows artifact from project root

**Accepted (LOW — no action needed for Sprint 0):**
6. **L1:** Schema uses `text` instead of `varchar` from architecture spec — reasonable Postgres choice, no functional difference
7. **L2:** `users.name` is notNull (spec had it nullable) — Story 5.2 must provide name on user creation (email fallback)
8. **L3:** No ON DELETE CASCADE on FKs — acceptable for Sprint 0 (no user deletion feature)
