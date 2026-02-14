---
stepsCompleted: ['step-01-init', 'step-02-context', 'step-03-starter-skipped', 'step-04-decisions', 'step-05-patterns', 'step-06-structure', 'step-07-validation', 'step-08-complete']
status: 'complete'
completedAt: '2026-02-13'
lastEdited: '2026-02-13'
inputDocuments:
  - '_bmad-output/planning-artifacts/prd-sprint1.md'
  - '_bmad-output/planning-artifacts/architecture.md'
  - 'docs/architecture-sprint0-supplement.md'
  - 'docs/project-context.md'
  - 'docs/decisions.md'
  - 'docs/_masemIT/bmad-briefing-kurzum-sprint1.md'
  - 'docs/ux-sprint0-app-specs.md'
  - 'lib/db/schema.ts'
  - 'lib/ai/prompts.ts'
  - 'middleware.ts'
  - 'app/api/voice/route.ts'
workflowType: 'architecture'
project_name: 'kurzum-app'
user_name: 'Sempre'
date: '2026-02-13'
baseDocument: '_bmad-output/planning-artifacts/architecture.md'
sprintSupplementFor: 'Sprint 0 supplement: docs/architecture-sprint0-supplement.md'
---

# Architecture Supplement: Sprint 1 — Project Assignment & Inbox

**Author:** Winston (Architect), collaborative with Sempre
**Date:** 2026-02-13
**Status:** In Progress
**Base Document:** `_bmad-output/planning-artifacts/architecture.md` (2026-02-10)
**Sprint 0 Supplement:** `docs/architecture-sprint0-supplement.md` (2026-02-12)
**Sprint 1 PRD:** `_bmad-output/planning-artifacts/prd-sprint1.md` (2026-02-13)

---

_This document covers Sprint 1-specific architecture decisions as a supplement to the base architecture. It follows the same pattern as the Sprint 0 supplement. Sections are appended as we work through each decision together._

## 1. Scope & Relationship to Base Architecture

The base architecture document covers the overall product vision and landing page (Phase 1 Detail). The Sprint 0 supplement covers the auth-protected prototype (voice → transcription → summary).

This supplement covers **Sprint 1: Project Assignment & Inbox** — the transition from "voice recorder with AI" to "project-aware communication tool."

### Key Deviations from Sprint 0

| Topic | Sprint 0 | Sprint 1 Decision Needed | Rationale |
|-------|----------|-------------------------|-----------|
| Schema | 4 tables (users, sessions, magic_links, voice_messages) | +3 tables (companies, projects, project_members) + extensions | Multi-tenant foundation + project domain |
| Voice Pipeline | Synchronous: Upload → STT → LLM Summary | Extended: + LLM Assignment step | Core Sprint 1 feature |
| Middleware | Auth only (userId injection) | + companyId + role injection | Tenant isolation |
| AI Prompts | 1 prompt (VOICE_SUMMARY) | +1 prompt (ASSIGNMENT_PROMPT_V1) | Pipeline extension |
| API Routes | `/api/voice` only | + `/api/projects/*` CRUD | New domain |
| App Routes | `/app` (dashboard), `/app/record` | + `/app/inbox`, `/app/projects/*` | Inbox + project pages |
| User model | `role: "user"` | `role: "monteur"/"meister"/"buero"` + `companyId` | RBAC prep + tenant |

---

## 2. Project Context Analysis

### Requirements Overview

**Functional Requirements (29 FRs, 7 categories):**

| Category | FRs | Architectural Significance |
|----------|-----|---------------------------|
| **Voice & AI Baseline** | S1-FR1–5 | Carried from Sprint 0, no changes. Existing pipeline untouched. |
| **AI Project Assignment** | S1-FR6–10 | **Core Sprint 1.** New LLM call after summary. Needs: project context fetching, confidence scoring, routing logic. Extends `POST /api/voice` pipeline. |
| **Project Management** | S1-FR11–15 | New CRUD domain. `companies` + `projects` tables. API routes for create/edit/list. Project detail page with timeline (join voice_messages by projectId). |
| **Assignment Inbox** | S1-FR16–20 | Filter: `voice_messages WHERE projectId IS NULL`. Manual assignment = UPDATE projectId. AI-prefilled project creation. Badge counter (aggregation query). |
| **Dashboard** | S1-FR21–22 | Rebuild existing dashboard. Group by project instead of flat list. Project cards with activity summary. |
| **R&D Traceability** | S1-FR23–26 | New DB fields: `llmPromptVersion`, `assignedBy`, `aiConfidence`, `aiSuggestedProjectId`, `aiReasoning`. No new routes — metadata stored during pipeline. |
| **Data Architecture Prep** | S1-FR27–29 | `companyId` on all new tables. `role` on users. Session includes companyId + role. Foundation for Sprint 2 RBAC enforcement. |

**Non-Functional Requirements (13 NFRs):**

| Category | NFRs | Architectural Driver |
|----------|------|---------------------|
| **Performance** | S1-NFR1–4 | Pipeline +1 LLM call → <45s target (vs <30s Sprint 0). Dashboard <2s, Inbox <2s, Timeline <3s. All read-heavy queries — indexes critical. |
| **Security** | S1-NFR5–8 | EU data residency (extends to projects table). **Tenant isolation** via companyId middleware — every query filtered. No PII in prompt logs. |
| **Reliability** | S1-NFR9–11 | Assignment failure → graceful degradation (projectId=null → Inbox). Must NOT break existing STT+Summary. >95% JSON validity for assignment response. |
| **Accessibility** | S1-NFR12–13 | 48px touch targets, WCAG AA. Applies to new UI: Inbox buttons, project cards, assignment dropdown. |

### Scale & Complexity

- **Primary domain:** Full-stack brownfield extension (existing Next.js + NeonDB)
- **Complexity level:** Medium — new domain (projects) + pipeline extension + tenant prep
- **Architectural components affected:** ~6 (Schema, Voice Pipeline, Middleware, AI Module, new API routes, new pages)
- **New vs. modified:** 3 new tables, 2 extended tables, 1 extended pipeline, 1 extended middleware, ~4 new pages

### Technical Constraints

| Constraint | Source | Impact |
|------------|--------|--------|
| **Existing schema** | Sprint 0 live in NeonDB | Must use Drizzle migrations, not `push`. Additive only — no breaking changes to users/voice_messages. |
| **Synchronous pipeline** | Sprint 0 ADR (no QStash) | Assignment LLM call adds ~3-8s to pipeline. Total <45s target. If exceeded → consider parallel LLM calls or combined prompt. |
| **Vercel 60s timeout** | Vercel Pro plan | Pipeline: STT (~5s) + Summary (~3s) + Assignment (~5s) = ~13-20s. Well within limits. |
| **Single developer** | Resource reality | No complex abstractions. Middleware-based tenant isolation, not RLS. |
| **FFG research** | Grant requirement | Every AI decision must be traceable: prompt version, confidence, reasoning. |

### Cross-Cutting Concerns

| Concern | Affects | Sprint 1 Approach |
|---------|---------|-------------------|
| **Tenant Isolation** | All new DB queries, API routes, middleware | `companyId` injected via middleware into request headers. Every query on companies/projects/voice_messages must include companyId filter. |
| **Pipeline Extension** | voice route, AI module, DB schema | Assignment step appended to existing pipeline. Failure = graceful degradation (Inbox). Never blocks existing STT+Summary flow. |
| **Prompt Versioning** | AI module, DB schema | Constants in `lib/ai/prompts.ts`, version string in `llmPromptVersion` DB field. |
| **RBAC Preparation** | User model, session, middleware | `role` stored and propagated but NOT enforced. Sprint 2 adds guards. |
| **Dashboard Rebuild** | Dashboard page, API queries | Current flat voice_messages list → project-grouped cards. Breaking UI change, not data change. |

---

## 3. Core Architectural Decisions

All decisions below are Sprint 1-specific. Base architecture decisions (tech stack, naming conventions, API response format, styling, deployment) remain unchanged from `architecture.md`.

### Decision 1: Separate LLM Calls for Summary + Assignment

**Decision:** Sequential, separate LLM calls — NOT a combined prompt.

```
STT → LLM #1 (Summary, VOICE_SUMMARY_SYSTEM_PROMPT) → LLM #2 (Assignment, ASSIGNMENT_PROMPT_V1) → DB
```

**Rationale:**
- FFG Research Question #3 requires isolated evaluation of assignment accuracy
- Existing summary prompt (ADR-005, 96.3% validity) remains untouched — zero regression risk
- Separate `llmPromptVersion` tracking per step enables independent optimization
- Assignment failure → graceful degradation (Inbox), summary still saved as "partial" success
- Latency acceptable: ~13-20s total, well under 45s target and 60s Vercel timeout
- Cost optimization (combined prompt) deferred to Sprint 2 after baselines are established

**Pipeline Extension Point:**

```typescript
// In app/api/voice/route.ts — after existing LLM summary step:
// Step 3 (NEW): Project Assignment
try {
  const assignment = await assignToProject(transcript, userId, companyId);
  // assignment = { projectId: string | null, confidence: number, reasoning: string }
  // if confidence >= 0.7 → auto-assign
  // if confidence < 0.7 → projectId stays null (Inbox)
} catch {
  // Assignment failure is NOT a pipeline failure
  // voice message saved with projectId=null → appears in Inbox
  // status remains "done" (not "error" or "partial")
}
```

**Key Rule:** Assignment failure MUST NOT change the voice message status. The message is "done" after successful summary. Assignment is a best-effort enrichment.

### Decision 2: Server Actions for CRUD, Route Handlers for Upload

**Decision:** Server Actions for all project/inbox mutations. Route Handler only for `POST /api/voice` (multipart upload).

**Rationale:**
- Base architecture recommends "Server Actions for app CRUD" (architecture.md §API Patterns)
- Server Actions = less boilerplate, direct invocation from Client Components
- `POST /api/voice` stays as Route Handler — multipart/formData requires it
- Inbox assignment (UPDATE projectId) = Server Action
- Project CRUD = Server Actions

**File Structure:**

```
lib/actions/
├── projects.ts         # createProject, updateProject, listProjects
├── inbox.ts            # assignToProject, getUnassignedCount
└── voice-messages.ts   # getProjectTimeline, getRecentMessages
```

**Server Action Pattern:**

```typescript
// lib/actions/projects.ts
"use server";

import { getAuthContext } from "@/lib/auth/session";

export async function createProject(data: CreateProjectInput) {
  const { userId, companyId } = await getAuthContext();
  // Zod validation → DB insert with companyId → revalidatePath
}
```

### Decision 3: Session Helper for Auth Context

**Decision:** `getAuthContext()` helper function for unified auth context access in Server Actions AND Route Handlers. Middleware remains for route protection only.

**Rationale:**
- Server Actions can't read middleware-injected headers (they run server-side, not via HTTP)
- Helper reads session cookie directly → validates → returns typed context
- Single source of truth for `{ userId, companyId, role }`
- Middleware continues to protect `/app/*` and `/api/voice/*` routes (redirect if no session)

**Implementation:**

```typescript
// lib/auth/session.ts — EXTEND existing module

interface AuthContext {
  userId: string;
  companyId: string;
  role: "monteur" | "meister" | "buero";
}

export async function getAuthContext(): Promise<AuthContext> {
  // 1. Read session cookie (next/headers)
  // 2. Validate session against DB (existing validateSession)
  // 3. Join users table for companyId + role
  // 4. Throw if no valid session (caught by error boundary)
  // Returns: { userId, companyId, role }
}
```

**Migration Impact:** Existing `POST /api/voice/route.ts` currently reads `x-user-id` from middleware header. Sprint 1 refactors to use `getAuthContext()` for `userId` + `companyId`. The middleware header injection (`x-user-id`) is removed — all routes use the helper.

### Decision 4: Schema Migration Strategy

**Decision:** Drizzle `generate` + `migrate` for production. NOT `push`. Multi-step migration with data backfill.

**Migration Sequence:**

```
Migration 001: Create companies table
Migration 002: Add companyId (nullable) + role to users
Migration 003: Seed company "masemIT Testbetrieb", backfill Mario user
Migration 004: ALTER users SET companyId NOT NULL
Migration 005: Create projects table (with companyId NOT NULL)
Migration 006: Create project_members table
Migration 007: Add assignment columns to voice_messages (all nullable)
```

**Key Schema Decisions:**

| Table | Column | Type | Nullable | Rationale |
|-------|--------|------|----------|-----------|
| `users` | `companyId` | uuid FK | NOT NULL (after backfill) | Every user belongs to a company. Enforced from day 1. |
| `users` | `role` | text | NOT NULL, default "monteur" | Sprint 0 users get default role. Mario manually set to "meister". |
| `voice_messages` | `projectId` | uuid FK | YES | null = Inbox (unassigned). This is intentional, not missing data. |
| `voice_messages` | `aiSuggestedProjectId` | uuid FK | YES | What AI suggested, even if user reassigned. |
| `voice_messages` | `aiConfidence` | real | YES | 0.0–1.0. null for Sprint 0 messages (no assignment). |
| `voice_messages` | `assignedBy` | text | YES | "ai" or "user". null for Sprint 0 messages. |
| `voice_messages` | `aiReasoning` | text | YES | LLM reasoning string. FFG traceability. |
| `voice_messages` | `llmPromptVersion` | text | YES | e.g., "VOICE_SUMMARY_V1" or "ASSIGNMENT_V1". |
| `voice_messages` | `assignedAt` | timestamptz | YES | When assignment happened. |
| `voice_messages` | `companyId` | uuid FK | YES | Denormalized from user for query performance. Backfilled for existing messages. |

**Existing Sprint 0 Data:** All existing voice_messages get `projectId=null`, `companyId` backfilled from user. They appear in the Inbox until manually assigned — this is correct behavior.

### Decision 5: Assignment Prompt Context — Medium

**Decision:** Transcript + active projects (name, customer, address) + last 3 messages of the speaker with their project assignment.

**Context Data Fetched:**

```typescript
// lib/ai/assignment.ts
interface AssignmentContext {
  transcript: string;
  projects: Array<{
    id: string;
    name: string;
    customerName: string | null;
    address: string | null;
  }>;
  recentMessages: Array<{
    createdAt: string;
    summaryStatus: string;
    projectName: string | null;
  }>;
}
```

**Rationale:**
- Sprint 1 briefing proposes exactly this context window
- Projects: name + customer + address gives the LLM enough to match location/customer references
- Recent messages: "speaker was on project X yesterday" → strong signal for continuity
- No project descriptions (too verbose, risks token limit with many projects)
- No messages from other speakers (privacy within tenant, complexity)
- Limit: 3 recent messages is enough for continuity signal without bloating context

**Token Budget Estimate:**
- 5 projects × ~30 tokens each = ~150 tokens
- 3 recent messages × ~50 tokens each = ~150 tokens
- Transcript: ~100-300 tokens
- System prompt: ~200 tokens
- Total input: ~600-800 tokens — well within Mistral limits

### Decision 6: Server Components for Dashboard + Inbox

**Decision:** Dashboard, Inbox, and Project Timeline are Server Components with direct Drizzle queries. No client-side data fetching.

**Rationale:**
- No real-time requirements in Sprint 1
- Server Components = fastest initial load, no client JS bundle for data fetching
- Inbox badge count in navigation layout (Server Component) — revalidated on every navigation
- Project timeline = simple chronological query, no pagination needed for Sprint 1 scale
- Real-time updates deferred to Sprint 3 (PWA with SSE/polling)

**Page Architecture:**

```
app/app/page.tsx              → Server Component: project-based dashboard
app/app/layout.tsx            → Server Component: nav with Inbox badge count
app/app/inbox/page.tsx        → Server Component: unassigned messages list
app/app/projects/page.tsx     → Server Component: all projects list
app/app/projects/[id]/page.tsx → Server Component: project timeline
```

**Client Components (interactive elements only):**

```
components/app/inbox-assign-button.tsx   → "use client": dropdown + Server Action call
components/app/project-form.tsx          → "use client": RHF form for create/edit
components/app/project-status-toggle.tsx → "use client": status change dropdown
```

### Decision Impact Analysis

**Implementation Sequence (dependency order):**

1. Schema migrations (001-007) — foundation for everything
2. `getAuthContext()` helper — required by all Server Actions
3. Middleware extension (companyId validation) — required for route protection
4. Projects CRUD (Server Actions + pages) — required before assignment
5. `ASSIGNMENT_PROMPT_V1` + `assignToProject()` — core Sprint 1 feature
6. Voice pipeline extension (add assignment step to `POST /api/voice`)
7. Inbox page + assignment Server Actions
8. Dashboard rebuild (project-based view)

**Cross-Component Dependencies:**

```
Schema → getAuthContext → Middleware
                       → Server Actions → Pages
                       → Voice Pipeline Extension
Projects CRUD → Assignment (needs projects to assign to)
Assignment → Inbox (unassigned messages)
Dashboard depends on all of the above
```

---

## 4. Sprint 1 Implementation Patterns

_Base patterns from `architecture.md` §Implementation Patterns remain in effect. This section adds Sprint 1-specific patterns only._

### Server Action Pattern

All CRUD mutations as Server Actions. Every Server Action follows this template:

```typescript
// lib/actions/projects.ts
"use server";

import { revalidatePath } from "next/cache";
import { getAuthContext } from "@/lib/auth/session";
import { db } from "@/lib/db";
import { projects } from "@/lib/db/schema";
import { createProjectSchema } from "@/lib/validations/project";

export async function createProject(input: unknown) {
  const { userId, companyId } = await getAuthContext();
  const data = createProjectSchema.parse(input);    // Zod validation

  const [project] = await db
    .insert(projects)
    .values({ ...data, companyId })                  // ALWAYS include companyId
    .returning();

  revalidatePath("/app");                             // Revalidate affected pages
  return project;
}
```

**Rules:**
- Every Action starts with `getAuthContext()` — no exceptions
- Zod validation BEFORE DB access
- `companyId` is ALWAYS injected from AuthContext, NEVER accepted from client input
- `revalidatePath()` after every mutation
- Return type is the result, not `ApiResponse<T>` (that's for Route Handlers)

### companyId Filtering Pattern

**The most important Sprint 1 rule:** Every DB read on tenant-scoped tables MUST filter by companyId.

```typescript
// ✅ CORRECT — companyId filter always present
const myProjects = await db
  .select()
  .from(projects)
  .where(eq(projects.companyId, companyId));

// ❌ WRONG — missing companyId filter = data leak
const allProjects = await db.select().from(projects);
```

**Tenant-scoped tables:**

| Table | companyId Source | Filter Required |
|-------|----------------|-----------------|
| `companies` | `id` (IS the tenant) | `eq(companies.id, companyId)` |
| `users` | `companyId` column | `eq(users.companyId, companyId)` |
| `projects` | `companyId` column | `eq(projects.companyId, companyId)` |
| `project_members` | via projects join | Join through projects |
| `voice_messages` | `companyId` column (denormalized) | `eq(voiceMessages.companyId, companyId)` |
| `waitlist_entries` | NOT tenant-scoped | No filter needed |
| `sessions` | NOT tenant-scoped (auth) | No filter needed |
| `magic_links` | NOT tenant-scoped (auth) | No filter needed |

### Pipeline Error Handling Pattern

The voice pipeline has 3 stages. Each stage has its own error handling:

```
Upload → STT (fail = "error") → Summary (fail = "partial") → Assignment (fail = silent, Inbox)
```

| Stage | On Failure | Status | User Sees |
|-------|-----------|--------|-----------|
| Upload | Abort pipeline | no entry | "Upload fehlgeschlagen" |
| STT | Stop, audio saved | "error" | "Spracherkennung hat nicht funktioniert" |
| Summary | Stop, transcript saved | "partial" | "Zusammenfassung konnte nicht erstellt werden" |
| **Assignment** | **Continue**, projectId=null | **"done"** | Message appears in Inbox |

**Key Rule:** Assignment failure MUST NOT change the status. The message is "done" after successful summary. Assignment is best-effort enrichment.

### Prompt Versioning Pattern

```typescript
// lib/ai/prompts.ts
export const VOICE_SUMMARY_SYSTEM_PROMPT = `...`;        // Existing, unchanged
export const VOICE_SUMMARY_VERSION = "SUMMARY_V1";       // NEW: version constant

export const ASSIGNMENT_SYSTEM_PROMPT = `...`;            // NEW: Sprint 1
export const ASSIGNMENT_PROMPT_VERSION = "ASSIGNMENT_V1"; // NEW: version constant
```

**DB storage:** `llmPromptVersion` field in voice_messages stores a combined version string, format: `"summary:V1,assignment:V1"`. Simple text, no JSON parsing needed, still traceable.

### Inbox Query Pattern

Inbox = `voice_messages WHERE projectId IS NULL AND status = 'done' AND companyId = ?`

```typescript
// lib/actions/inbox.ts
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

**Important:** Only `status = "done"` — no "error" or "partial" messages in Inbox.

### Anti-Patterns (Sprint 1-specific)

```typescript
// ❌ WRONG: companyId from client input
export async function createProject(data: { name: string; companyId: string }) {
  await db.insert(projects).values(data);  // Client could send any companyId!
}

// ❌ WRONG: Assignment failure changes status
if (!assignment) {
  await db.update(voiceMessages).set({ status: "error" });  // NO! Status stays "done"
}

// ❌ WRONG: Missing getAuthContext in Server Action
export async function listProjects() {
  return db.select().from(projects);  // No auth check, no companyId filter!
}

// ❌ WRONG: Hardcoded prompt version
await db.update(voiceMessages).set({ llmPromptVersion: "v1" });
// ✅ CORRECT: Use constant
await db.update(voiceMessages).set({ llmPromptVersion: ASSIGNMENT_PROMPT_VERSION });
```

---

## 5. Project Structure & Boundaries

_This section documents Sprint 1 file additions and modifications relative to the Sprint 0 codebase. The base project structure is defined in `architecture.md`._

### New Files (Sprint 1)

```
lib/
├── actions/
│   ├── projects.ts              # createProject, updateProject, listProjects, getProject
│   ├── inbox.ts                 # getUnassignedMessages, assignToProject, getUnassignedCount
│   └── voice-messages.ts        # getProjectTimeline, getRecentMessages
├── ai/
│   └── assignment.ts            # assignToProject(transcript, userId, companyId) → AssignmentResult
├── validations/
│   └── project.ts               # createProjectSchema, updateProjectSchema (Zod)
└── db/
    └── migrations/
        ├── 0001_create_companies.sql
        ├── 0002_add_company_role_to_users.sql
        ├── 0003_seed_test_company.sql
        ├── 0004_users_company_not_null.sql
        ├── 0005_create_projects.sql
        ├── 0006_create_project_members.sql
        └── 0007_voice_messages_assignment_columns.sql

app/app/
├── inbox/
│   └── page.tsx                 # Server Component: unassigned voice messages list
├── projects/
│   ├── page.tsx                 # Server Component: all projects list
│   └── [id]/
│       └── page.tsx             # Server Component: project detail + timeline

components/app/
├── inbox-assign-button.tsx      # "use client": project assignment dropdown + Server Action
├── project-form.tsx             # "use client": RHF form for create/edit project
├── project-card.tsx             # Server Component: project summary card for dashboard
├── project-status-toggle.tsx    # "use client": active/archived toggle
└── inbox-badge.tsx              # "use client": badge counter in nav (polls revalidation)
```

### Modified Files (Sprint 1)

| File | Change | Scope |
|------|--------|-------|
| `lib/db/schema.ts` | Add `companies`, `projects`, `project_members` tables. Extend `users` (+companyId, +role). Extend `voiceMessages` (+projectId, +aiConfidence, +assignedBy, +aiReasoning, +llmPromptVersion, +assignedAt, +aiSuggestedProjectId, +companyId) | Major |
| `lib/ai/prompts.ts` | Add `VOICE_SUMMARY_VERSION`, `ASSIGNMENT_SYSTEM_PROMPT`, `ASSIGNMENT_PROMPT_VERSION` constants | Medium |
| `lib/ai/mistral.ts` | Add `chatWithMistral()` or extend existing for assignment call (structured JSON output) | Medium |
| `lib/auth/session.ts` | Add `getAuthContext()` → `{ userId, companyId, role }` | Medium |
| `middleware.ts` | Add companyId validation (check user has companyId), remove `x-user-id` header injection | Medium |
| `app/api/voice/route.ts` | Add Step 3: Assignment LLM call after summary. Use `getAuthContext()` instead of header. Add companyId to DB insert. | Major |
| `app/app/layout.tsx` | Add nav links for Inbox (with badge), Projects. Restructure navigation. | Medium |
| `app/app/page.tsx` | Rebuild dashboard: project-based grouping instead of flat voice message list | Major |

### Requirements → File Mapping

| FR Category | Primary Files | Supporting Files |
|------------|--------------|-----------------|
| **S1-FR6–10: AI Project Assignment** | `lib/ai/assignment.ts`, `lib/ai/prompts.ts`, `app/api/voice/route.ts` | `lib/ai/mistral.ts`, `lib/db/schema.ts` |
| **S1-FR11–15: Project Management** | `lib/actions/projects.ts`, `app/app/projects/` | `lib/validations/project.ts`, `components/app/project-form.tsx`, `components/app/project-card.tsx` |
| **S1-FR16–20: Assignment Inbox** | `lib/actions/inbox.ts`, `app/app/inbox/page.tsx` | `components/app/inbox-assign-button.tsx`, `components/app/inbox-badge.tsx` |
| **S1-FR21–22: Dashboard** | `app/app/page.tsx` | `components/app/project-card.tsx`, `lib/actions/voice-messages.ts` |
| **S1-FR23–26: R&D Traceability** | `lib/db/schema.ts` (new columns), `lib/ai/prompts.ts` (versions) | `app/api/voice/route.ts` (stores metadata) |
| **S1-FR27–29: Data Architecture** | `lib/db/schema.ts`, `lib/db/migrations/`, `lib/auth/session.ts` | `middleware.ts` |

### Data Flow Diagrams

**Voice Pipeline (Extended — Sprint 1):**

```
User speaks → MediaRecorder → Blob
  → POST /api/voice (multipart)
    → getAuthContext() → { userId, companyId }
    → Validate (Zod) → Upload Blob (Vercel Blob)
    → INSERT voice_messages (userId, companyId, blobUrl, status="processing")
    → STT: Mistral Voxtral (transcript)
      → UPDATE voice_messages (transcript, status still "processing")
    → LLM #1: Summary (VOICE_SUMMARY_SYSTEM_PROMPT)
      → UPDATE voice_messages (summary, summaryStatus, status="done")
    → LLM #2: Assignment (ASSIGNMENT_SYSTEM_PROMPT)          ← NEW
      → Fetch: active projects (companyId) + last 3 messages  ← NEW
      → Result: { projectId | null, confidence, reasoning }    ← NEW
      → If confidence ≥ 0.7: UPDATE projectId + assignedBy="ai" ← NEW
      → If confidence < 0.7: projectId stays null (→ Inbox)    ← NEW
      → UPDATE aiConfidence, aiReasoning, llmPromptVersion     ← NEW
      → CATCH: Assignment failure → no status change, Inbox    ← NEW
    → Return ApiResponse<{ id, transcript, summary, projectName? }>
```

**Inbox Assignment Flow:**

```
Inbox Page (Server Component)
  → getUnassignedMessages() → voice_messages WHERE projectId IS NULL AND status="done" AND companyId=?
  → Render list: summary, timestamp, AI suggestion (if aiSuggestedProjectId)

User clicks "Zuordnen" (InboxAssignButton — Client Component)
  → Dropdown: active projects list
  → User selects project
  → Server Action: assignToProject(messageId, projectId)
    → getAuthContext() → companyId
    → UPDATE voice_messages SET projectId=?, assignedBy="user", assignedAt=now()
      WHERE id=? AND companyId=?
    → revalidatePath("/app/inbox")
    → revalidatePath("/app")
```

### Architectural Boundaries

**API Boundary:**
- `POST /api/voice` — only Route Handler (multipart upload). All other mutations via Server Actions.
- Server Actions are NOT callable from external clients — only from React Server/Client Components.

**Tenant Boundary:**
- `companyId` is the tenant isolation key. Every DB query on tenant-scoped tables includes companyId filter.
- `companyId` flows: Session cookie → `getAuthContext()` → DB query filter. Never from URL params or client input.

**AI Module Boundary:**
- `lib/ai/` is the single location for all AI logic (prompts, Mistral calls, assignment logic).
- Voice route (`app/api/voice/route.ts`) orchestrates but delegates to `lib/ai/` for AI operations.
- Prompt constants and version strings are co-located in `lib/ai/prompts.ts`.

**Component Boundary:**
- Server Components: pages, layouts, data-fetching containers. Direct Drizzle queries via Server Actions.
- Client Components: interactive elements only (forms, dropdowns, toggles). Call Server Actions for mutations.
- No `useEffect` + `fetch` for data loading in Sprint 1. All data fetching is server-side.

---

## 6. Architecture Validation Results

### Coherence Validation ✅

**Decision Compatibility:**
All 6 decisions validated pairwise — no contradictions. Key validations:
- D1 (Separate LLM Calls) + D5 (Medium Context): Assignment-Call erhält die in D5 definierten Context-Daten. Kein Konflikt mit Summary-Call.
- D2 (Server Actions) + D3 (getAuthContext): Server Actions rufen `getAuthContext()` direkt auf — genau dafür wurde der Helper entworfen.
- D3 (getAuthContext) + D4 (Schema Migration): Migration fügt `companyId` + `role` zu users hinzu → getAuthContext() kann sie ab Migration 002 lesen.

**Pattern Consistency:**
- Server Action Pattern + companyId Filtering Pattern → jede Action startet mit `getAuthContext()`, jede Query filtert `companyId`. ✅
- Pipeline Error Handling + Inbox Query Pattern → Assignment-Fehler → `projectId=null` → Inbox-Query findet sie. ✅
- Prompt Versioning + R&D Traceability Columns → Versions-Konstanten in `prompts.ts`, gespeichert in `llmPromptVersion` DB-Feld. ✅

**Structure Alignment:**
- Neue Dateien folgen existierender Struktur (kein `src/`, `@/` Imports, `app/app/` für geschützte Routen). ✅
- Client Components nur für interaktive Elemente — minimal Client JS. ✅

### Requirements Coverage ✅

**Functional Requirements: 29/29 abgedeckt.**

| FR-Block | FRs | Architektonische Abdeckung |
|----------|-----|---------------------------|
| Voice Baseline | S1-FR1–5 | Sprint 0 carried, keine Änderung |
| AI Assignment | S1-FR6–10 | `lib/ai/assignment.ts`, Pipeline Extension (D1), Schema (D4) |
| Project Management | S1-FR11–15 | `lib/actions/projects.ts`, `app/app/projects/`, `project-form.tsx` |
| Inbox | S1-FR16–20 | `lib/actions/inbox.ts`, `app/app/inbox/`, Inbox Query Pattern |
| Dashboard | S1-FR21–22 | `app/app/page.tsx` rebuild, `project-card.tsx` |
| R&D Traceability | S1-FR23–26 | Prompt Versioning Pattern, Schema assignment columns, Story 7.2 done |
| Data Architecture | S1-FR27–29 | Schema Migrations (D4), `getAuthContext()` (D3) |

**Non-Functional Requirements: 13/13 adressiert.**

| NFR-Block | NFRs | Architektonische Abdeckung |
|-----------|------|---------------------------|
| Performance | S1-NFR1–4 | Sequential pipeline ~13-20s (<45s), Server Components for read pages |
| Security | S1-NFR5–8 | EU residency extended, companyId filtering, server-side keys, no PII in logs |
| Reliability | S1-NFR9–11 | Assignment failure → silent Inbox, no pipeline regression, JSON validity baseline |
| Accessibility | S1-NFR12–13 | Extends Sprint 0 design system (48px targets, WCAG AA) |

### Implementation Readiness ✅

**Decision Completeness:** All 6 decisions with code examples, rationale, and explicit "Key Rules." ✅
**Pattern Completeness:** 5 patterns + anti-patterns with complete code templates. ✅
**Structure Completeness:** All new files listed, modified files with scope, FR → File mapping, data flow diagrams. ✅

### Gap Analysis

**Critical Gaps:** None.

**Important Gaps (non-blocking, addressed in Stories):**

| Gap | Recommendation |
|-----|---------------|
| Index strategy for performance NFRs | Define compound indexes `(companyId, projectId, status)` on `voice_messages` in migration stories |
| Assignment prompt text | Defined in implementation story, analogous to ADR-005 for summary |
| Error boundary for Server Actions | `error.tsx` in `app/app/` sufficient for Sprint 1 |

### Architecture Readiness Assessment

**Overall Status:** READY FOR IMPLEMENTATION

**Confidence Level:** High — brownfield extension with clear boundaries, all requirements covered, no critical gaps.

### Implementation Handoff

**AI Agent Guidelines:**
- Follow all 6 architectural decisions exactly as documented
- Use implementation patterns (Server Action template, companyId filter) consistently
- Respect tenant boundary — every query filtered by companyId from getAuthContext()
- Assignment failure MUST NOT change voice message status
- Refer to this document + base architecture for all technical decisions

**Implementation Sequence (dependency order):**
1. Schema migrations (001-007) — foundation
2. `getAuthContext()` helper — required by everything
3. Middleware extension — companyId validation
4. Projects CRUD — required before assignment
5. Assignment prompt + `assignToProject()` — core feature
6. Voice pipeline extension — add assignment step
7. Inbox page + assignment actions
8. Dashboard rebuild — project-based view
