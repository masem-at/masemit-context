# Story 8.2: Create & Edit Projects

Status: done

<!-- Note: Validation is optional. Run validate-create-story for quality check before dev-story. -->

## Story

As a **Meister**,
I want to create and edit construction projects with name, customer, address, and description,
so that I can organize my construction sites in the system.

## Acceptance Criteria

1. **Given** a logged-in Meister on `/app/projects/new`
   **When** filling out name (required), customer name, address, description and submitting
   **Then** a new project is created with status "active" and the Meister's companyId
   **And** the Meister is redirected to the project detail page

2. **Given** a logged-in Meister on a project's detail page
   **When** clicking "Bearbeiten" and updating project fields
   **Then** the project details are saved
   **And** the page shows the updated values

3. **Given** a logged-in Meister viewing a project
   **When** changing the project status via a dropdown (active, paused, completed)
   **Then** the status is updated immediately
   **And** the change is reflected on the project list and detail pages

4. **Given** a Server Action for project mutations
   **When** creating or updating a project
   **Then** `getAuthContext()` is called to get the companyId
   **And** Zod validates the input
   **And** the companyId is injected from AuthContext, never from client input
   **And** `revalidatePath` is called after mutation

**FRs:** S1-FR11, S1-FR12, S1-FR14 | **NFRs:** S1-NFR12, S1-NFR13

## Tasks / Subtasks

- [x] Task 1: Create Zod validation schemas (AC: #4)
  - [x] 1.1 Create `lib/validations/project.ts` with `createProjectSchema`: name (required, min 2, max 100), customerName (optional, max 100), address (optional, max 200), description (optional, max 1000)
  - [x] 1.2 Add `updateProjectSchema` extending create schema
  - [x] 1.3 Add `updateProjectStatusSchema`: id (uuid), status (enum: "active", "paused", "completed")
  - [x] 1.4 Export types: `CreateProjectData`, `UpdateProjectData`, `UpdateProjectStatusData`
  - [x] 1.5 All error messages in German (e.g., "Projektname muss mindestens 2 Zeichen lang sein.")

- [x] Task 2: Create Server Actions for project CRUD (AC: #1, #2, #3, #4)
  - [x] 2.1 Create `lib/actions/projects.ts` with `"use server"` directive
  - [x] 2.2 Implement `createProject(input: unknown)`: getAuthContext() → Zod validate → db.insert with companyId from auth → revalidatePath → return project
  - [x] 2.3 Implement `updateProject(id: string, input: unknown)`: getAuthContext() → verify project belongs to companyId → Zod validate → db.update with `updatedAt: sql\`now()\`` → revalidatePath → return project
  - [x] 2.4 Implement `updateProjectStatus(id: string, status: string)`: getAuthContext() → verify ownership → Zod validate status enum → db.update → revalidatePath
  - [x] 2.5 Implement `getProject(id: string)`: getAuthContext() → db.select WHERE id AND companyId → return project or null
  - [x] 2.6 CRITICAL: Every query MUST include `eq(projects.companyId, companyId)` — never trust client input for companyId
  - [x] 2.7 Use `revalidatePath("/app/projects")` and `revalidatePath("/app")` after mutations

- [x] Task 3: Create ProjectForm client component (AC: #1, #2)
  - [x] 3.1 Create `components/app/project-form.tsx` as `"use client"` component
  - [x] 3.2 Use React Hook Form with `zodResolver(createProjectSchema as any)` — Zod v4 type quirk
  - [x] 3.3 Render form fields with shadcn/ui components (Input, Textarea, Label, Button)
  - [x] 3.4 Fields: name (Input, required, autofocus), customerName (Input, optional), address (Input, optional), description (Textarea, optional)
  - [x] 3.5 Accept `mode: "create" | "edit"` prop and optional `defaultValues` for edit mode
  - [x] 3.6 Accept `onSubmit` async callback prop that calls the appropriate Server Action
  - [x] 3.7 Submit button: "Projekt erstellen" (create) or "Änderungen speichern" (edit), Orange bg, full-width on mobile
  - [x] 3.8 Loading state with disabled button and "Wird gespeichert..." text during submission
  - [x] 3.9 Error display: inline field errors (German) + top-level error banner for server errors
  - [x] 3.10 Validation mode: `onBlur` (matching waitlist form pattern from Story 3.1)
  - [x] 3.11 All touch targets ≥48px (S1-NFR12), WCAG AA contrast (S1-NFR13)

- [x] Task 4: Create project pages (AC: #1, #2, #3)
  - [x] 4.1 Create `app/app/projects/new/page.tsx` — Server Component wrapper that renders ProjectForm in create mode (NOTE: codebase uses `app/app/` not `app/(app)/`)
  - [x] 4.2 Implement create submit handler: call `createProject` Server Action → `router.push(\`/app/projects/\${project.id}\`)`
  - [x] 4.3 Create `app/app/projects/[id]/page.tsx` — Server Component: loads project via `getProject(id)`, displays project details in view mode, "Bearbeiten" button, ProjectStatusToggle
  - [x] 4.4 Show 404-style message if project not found or doesn't belong to company (uses Next.js `notFound()`)
  - [x] 4.5 Create `app/app/projects/[id]/edit/page.tsx` — Server Component wrapper: loads project, renders ProjectForm in edit mode with defaultValues
  - [x] 4.6 Implement edit submit handler: call `updateProject` → `router.push(\`/app/projects/\${id}\`)`
  - [x] 4.7 Add "Zurück" navigation on create/edit pages

- [x] Task 5: Create ProjectStatusToggle component (AC: #3)
  - [x] 5.1 Create `components/app/project-status-toggle.tsx` as `"use client"` component
  - [x] 5.2 Render shadcn Select with 3 options: "Aktiv" (active), "Pausiert" (paused), "Abgeschlossen" (completed)
  - [x] 5.3 On change: call `updateProjectStatus` Server Action → show loading state → router.refresh()
  - [x] 5.4 Display current status with colored badge: green (active), amber (paused), gray (completed)
  - [x] 5.5 Touch target ≥48px for the Select trigger

- [x] Task 6: Update navigation (AC: #1)
  - [x] 6.1 Add "Projekte" link to `components/app/app-header.tsx` navLinks array: `{ href: "/app/projects", label: "Projekte", Icon: FolderOpen }`
  - [x] 6.2 Add "Projekte" tab to `components/app/bottom-nav.tsx` for mobile: 3rd tab between Aufnahme and Übersicht
  - [x] 6.3 Reorder tabs: Aufnahme | Projekte | Übersicht
  - [x] 6.4 Ensure nav still works cleanly with 3 items (equal spacing, 48px touch targets)

- [x] Task 7: Create minimal project list page placeholder (AC: #1)
  - [x] 7.1 Create `app/app/projects/page.tsx` — Server Component: getProjects() (uses getAuthContext internally) → render simple list with project name + status + link to detail
  - [x] 7.2 Add "Neues Projekt" button linking to `/app/projects/new`
  - [x] 7.3 Empty state: "Noch keine Projekte. Erstelle dein erstes Projekt." + CTA button
  - [x] 7.4 NOTE: This is a minimal placeholder — Story 8.3 enhances with last activity, message count, and full design

- [x] Task 8: Verify + test (all ACs)
  - [x] 8.1 Run `pnpm build` — zero TypeScript errors
  - [x] 8.2 Test create project: fill form, submit, verify redirect to detail page, verify companyId is set correctly
  - [x] 8.3 Test edit project: navigate to detail, click "Bearbeiten", update fields, verify changes saved
  - [x] 8.4 Test status change: toggle status on detail page, verify reflected immediately
  - [x] 8.5 Test tenant isolation: verify project queries always include companyId filter
  - [x] 8.6 Test validation: submit empty name, verify German error message
  - [x] 8.7 Test navigation: verify "Projekte" link works in header and bottom nav

## Dev Notes

### Critical Architecture Patterns

**1. Server Actions — NOT API Routes (Architecture Decision 2):**
All project mutations MUST be Server Actions in `lib/actions/projects.ts`. Route Handlers are ONLY for `POST /api/voice` (multipart). Server Actions = direct invocation from Components, automatic `revalidatePath`, no fetch boilerplate.
[Source: architecture-sprint1.md#Decision 2]

**2. Server Action Template — EVERY action follows this exact pattern:**
```typescript
"use server";
import { revalidatePath } from "next/cache";
import { getAuthContext } from "@/lib/auth/session";
import { db } from "@/lib/db";
import { projects } from "@/lib/db/schema";
import { createProjectSchema } from "@/lib/validations/project";

export async function createProject(input: unknown) {
  const { userId, companyId } = await getAuthContext();
  const data = createProjectSchema.parse(input);    // Zod BEFORE DB
  const [project] = await db
    .insert(projects)
    .values({ ...data, companyId })                  // companyId from auth, NEVER client
    .returning();
  revalidatePath("/app/projects");
  revalidatePath("/app");
  return project;
}
```
[Source: architecture-sprint1.md#Server Action Pattern]

**3. companyId Filtering — THE most critical Sprint 1 rule:**
```typescript
// ✅ CORRECT — ALWAYS filter by companyId
const project = await db.select().from(projects)
  .where(and(eq(projects.id, id), eq(projects.companyId, companyId)));

// ❌ WRONG — missing companyId = cross-tenant data leak
const project = await db.select().from(projects).where(eq(projects.id, id));
```
[Source: architecture-sprint1.md#companyId Filtering Pattern]

**4. companyId NEVER from client input:**
```typescript
// ❌ WRONG — client could send any companyId
export async function createProject(data: { name: string; companyId: string }) { ... }

// ✅ CORRECT — companyId from getAuthContext()
export async function createProject(input: unknown) {
  const { companyId } = await getAuthContext();
  // inject companyId from auth context
}
```
[Source: architecture-sprint1.md#Anti-Patterns]

### Existing Codebase — What to Create vs. Modify

| Action | File | What | Scope |
|--------|------|------|-------|
| CREATE | `lib/validations/project.ts` | Zod schemas: createProjectSchema, updateProjectSchema, updateProjectStatusSchema | New file |
| CREATE | `lib/actions/projects.ts` | Server Actions: createProject, updateProject, updateProjectStatus, getProject | New file, new directory |
| CREATE | `components/app/project-form.tsx` | RHF form, "use client", shadcn Input/Textarea/Label/Button | New file |
| CREATE | `components/app/project-status-toggle.tsx` | shadcn Select, "use client", Server Action call | New file |
| CREATE | `app/(app)/projects/page.tsx` | Minimal project list (placeholder for Story 8.3) | New file, new directory |
| CREATE | `app/(app)/projects/new/page.tsx` | Create project page | New file |
| CREATE | `app/(app)/projects/[id]/page.tsx` | Project detail page (basic, timeline added in 8.3) | New file |
| CREATE | `app/(app)/projects/[id]/edit/page.tsx` | Edit project page | New file |
| MODIFY | `components/app/app-header.tsx` | Add "Projekte" nav link | Small change |
| MODIFY | `components/app/bottom-nav.tsx` | Add "Projekte" tab (3rd tab) | Small change |

### Existing Code Patterns to Follow

**Zod v4 pattern (from lib/validations/waitlist.ts):**
```typescript
import { z } from "zod";
export const createProjectSchema = z.object({
  name: z.string().min(2, { message: "Projektname muss mindestens 2 Zeichen lang sein." }),
  // ...
});
export type CreateProjectData = z.infer<typeof createProjectSchema>;
```
- `z.email()` is top-level in Zod v4 (not relevant here, but remember for future)
- Error messages in German
- Export inferred types for form components

**React Hook Form + Zod (from components/app/login-form.tsx):**
```typescript
const { register, handleSubmit, formState: { errors, isValid } } = useForm<CreateProjectData>({
  resolver: zodResolver(createProjectSchema as any),  // Zod v4 type quirk — use `as any`
  mode: "onBlur",
  defaultValues: { name: "", customerName: "", address: "", description: "" },
});
```
- `zodResolver(schema as any)` — required due to @hookform/resolvers v5.2.x + Zod 4.3.x type mismatch
- mode: `"onBlur"` for reduced frustration (matching Story 3.1 pattern)
- `register()` for simple inputs, `Controller` for shadcn Select (Radix primitives)

**Server Component page pattern (from app/(app)/page.tsx):**
```typescript
import { getAuthContext } from "@/lib/auth/session";
import { db } from "@/lib/db";
import { projects } from "@/lib/db/schema";
import { eq, desc } from "drizzle-orm";

export default async function ProjectsPage() {
  const { companyId } = await getAuthContext();
  const myProjects = await db.select().from(projects)
    .where(eq(projects.companyId, companyId))
    .orderBy(desc(projects.createdAt));
  // render...
}
```

**Drizzle query patterns established in this project:**
```typescript
// INSERT with returning
const [created] = await db.insert(projects).values({ ...data, companyId }).returning();

// UPDATE — set BEFORE where (Drizzle v0.45.1 API)
const [updated] = await db.update(projects)
  .set({ name: data.name, updatedAt: sql`now()` })
  .where(and(eq(projects.id, id), eq(projects.companyId, companyId)))
  .returning();

// SELECT single
const [project] = await db.select().from(projects)
  .where(and(eq(projects.id, id), eq(projects.companyId, companyId)))
  .limit(1);
```

**Navigation pattern (from components/app/app-header.tsx):**
```typescript
const navLinks = [
  { href: "/app/record", label: "Aufnahme", Icon: Mic },
  { href: "/app", label: "Übersicht", Icon: ClipboardList },
] as const;
// Add: { href: "/app/projects", label: "Projekte", Icon: FolderOpen }
```

**Bottom nav pattern (from components/app/bottom-nav.tsx):**
```typescript
// Currently 2 tabs — same structure, add 3rd tab
// Uses usePathname() for active state detection
// Active: Orange underline + accent color, Inactive: muted
```

### Project Schema — Already Exists (from Story 8.1)

The `projects` table was created in Story 8.1 migration. No new migrations needed for this story.

```typescript
// lib/db/schema.ts — projects table (already exists)
export const projects = pgTable("projects", {
  id: uuid("id").primaryKey().defaultRandom(),
  companyId: uuid("company_id").references(() => companies.id).notNull(),
  name: text("name").notNull(),
  customerName: text("customer_name"),    // nullable
  address: text("address"),               // nullable
  description: text("description"),       // nullable
  status: text("status").notNull().default("active"),
  createdAt: timestamp("created_at", { withTimezone: true }).defaultNow().notNull(),
  updatedAt: timestamp("updated_at", { withTimezone: true }).defaultNow().notNull(),
}, (table) => [
  index("idx_projects_company_id").on(table.companyId),
  index("idx_projects_status").on(table.companyId, table.status),
]);

export type Project = typeof projects.$inferSelect;
export type NewProject = typeof projects.$inferInsert;
```

**Indexes already in place:** `idx_projects_company_id`, `idx_projects_status` (compound: companyId + status).

### Critical Anti-Patterns to Avoid

```typescript
// ❌ WRONG: Using API Route Handler for project CRUD
// Story 3.3 used Route Handler — DO NOT follow that pattern for projects
// Server Actions are the Sprint 1 standard for CRUD mutations

// ❌ WRONG: Creating a new migration for projects table
// The projects table already exists from Story 8.1 — DO NOT run drizzle-kit generate

// ❌ WRONG: Using useEffect + fetch for data loading
// All data loading is server-side in Server Components
// Client Components only for interactive forms/dropdowns

// ❌ WRONG: Accepting status values without validation
// Status must be one of: "active", "paused", "completed"
// Use Zod enum validation on both client AND server

// ❌ WRONG: router.push without revalidation
// Server Actions handle revalidatePath — the redirect in the client component uses router.push
// Do NOT call revalidatePath in the client component

// ❌ WRONG: Inline edit toggle without URL change
// Use separate /edit page — cleaner, bookmarkable, better browser back button behavior
```

### Library/Framework Specifics

| Library | Version | Key Notes for This Story |
|---------|---------|--------------------------|
| Drizzle ORM | 0.45.1 | `sql\`now()\`` for DB-side timestamps on update |
| React Hook Form | latest | `register()` for inputs, `Controller` for shadcn Select |
| @hookform/resolvers | 5.2.x | `zodResolver(schema as any)` — Zod v4 type cast required |
| Zod | 4.3.x | `z.string().min()`, `z.enum()`, error messages in `{ message: "" }` |
| Next.js | 16.1.6 | `cookies()` is async, `searchParams` is Promise, Server Actions supported |
| shadcn/ui | latest | Input, Textarea, Label, Button, Select, Badge available |
| Lucide React | latest | FolderOpen icon for Projekte nav, ArrowLeft for back nav |

**shadcn Select + React Hook Form:**
shadcn's Select is built on Radix primitives — it does NOT work with `register()`. Use `Controller` from react-hook-form:
```typescript
<Controller
  name="status"
  control={control}
  render={({ field }) => (
    <Select onValueChange={field.onChange} value={field.value}>
      <SelectTrigger><SelectValue /></SelectTrigger>
      <SelectContent>
        <SelectItem value="active">Aktiv</SelectItem>
        ...
      </SelectContent>
    </Select>
  )}
/>
```
[Source: Memory — Story 3.1 learnings]

**Next.js 16 params pattern:**
```typescript
// app/(app)/projects/[id]/page.tsx
export default async function ProjectDetailPage({
  params,
}: {
  params: Promise<{ id: string }>;
}) {
  const { id } = await params;  // params is a Promise in Next.js 16
  // ...
}
```

### Project Structure Notes

**New directories created by this story:**
- `lib/actions/` — Server Actions directory (first time created)
- `app/(app)/projects/` — Project pages route group
- `app/(app)/projects/new/` — Create project page
- `app/(app)/projects/[id]/` — Project detail (dynamic route)
- `app/(app)/projects/[id]/edit/` — Edit project page

**Alignment with architecture:**
- File structure matches architecture-sprint1.md §5 exactly
- Server Actions in `lib/actions/` as specified in Decision 2
- Validation schemas in `lib/validations/` following existing pattern
- Client Components in `components/app/` with `"use client"` directive
- Server Components for pages (direct Drizzle queries)
[Source: architecture-sprint1.md#New Files (Sprint 1)]

### Previous Story Intelligence

**From Story 8.1 (Multi-Tenant Data Model):**
- `getAuthContext()` is fully implemented and tested — returns `{ userId, companyId, role }`
- Cookie name is `session` (NOT `kurzum_session` — was incorrectly documented initially)
- VALID_ROLES runtime validation prevents unsafe type casts
- Single JOIN query for auth (optimized in code review)
- `projects` table already created with all columns and indexes
- `Project` and `NewProject` types already exported from schema.ts
- Middleware validates session + companyId existence (no header injection)
- `DEFAULT_COMPANY_ID`: `00000000-0000-0000-0000-000000000001` (masemIT Testbetrieb)

**From Story 8.1 Debug Log — lessons to avoid:**
- Drizzle journal timestamps must be ascending — but NO new migrations needed for this story
- Use `drizzle-orm` imports for `eq`, `and`, `sql`, `desc` — NOT from `drizzle-kit`

**From Story 3.1 (Waitlist Form) — form patterns:**
- `zodResolver(schema as any)` cast is needed and works fine at runtime
- mode: `"onBlur"` for validation — NOT `"onChange"` (reduced frustration)
- shadcn Select requires `Controller`, not `register` (Radix primitive)

### Git Intelligence

Recent commit patterns:
```
0c721ec Implement multi-tenant data model and auth context helper (Story 8.1)
fbcd1f2 Implement audio file upload for research pipeline testing (Story 7.2)
0b2dcc0 Implement voice message dashboard with AI summary cards (Story 7.1)
```

**This story's commit should follow:** `"Implement project CRUD with Server Actions and form components (Story 8.2)"`

### References

- [Source: architecture-sprint1.md#Decision 2 — Server Actions for CRUD]
- [Source: architecture-sprint1.md#Decision 3 — Session Helper for Auth Context]
- [Source: architecture-sprint1.md#Server Action Pattern]
- [Source: architecture-sprint1.md#companyId Filtering Pattern]
- [Source: architecture-sprint1.md#Anti-Patterns]
- [Source: architecture-sprint1.md#New Files (Sprint 1)]
- [Source: prd-sprint1.md#S1-FR11, S1-FR12, S1-FR14]
- [Source: prd-sprint1.md#S1-NFR12, S1-NFR13]
- [Source: epics.md#Story 8.2 Acceptance Criteria]
- [Source: 8-1-multi-tenant-data-model-and-auth-context.md — getAuthContext, schema, patterns]

## Dev Agent Record

### Agent Model Used

Claude Opus 4.6 (claude-opus-4-6)

### Debug Log References

- Codebase uses `app/app/` route group (NOT `app/(app)/` as documented in story Dev Notes). Adapted all page paths accordingly.
- No test framework (jest/vitest) configured in project. Validation via `pnpm build` (zero TS errors) + `pnpm lint`.
- ProjectStatusToggle uses `useTransition` for optimistic UI updates with rollback on error.
- `getProjects()` added as convenience Server Action wrapping auth + query (not in original story spec but needed for project list page).

### Completion Notes List

- Task 1: Created `lib/validations/project.ts` with createProjectSchema, updateProjectSchema, updateProjectStatusSchema. All error messages in German. Types exported.
- Task 2: Created `lib/actions/projects.ts` with 5 Server Actions (createProject, updateProject, updateProjectStatus, getProject, getProjects). All queries filter by companyId from getAuthContext(). revalidatePath called after mutations.
- Task 3: Created `components/app/project-form.tsx` — RHF + Zod with mode prop, shadcn components, onBlur validation, min-h-12 touch targets, loading state, error banner.
- Task 4: Created 5 page files across `app/app/projects/` — new, [id], [id]/edit. Separate client wrapper components for create/edit form handlers. Next.js 16 async params pattern. notFound() for missing projects.
- Task 5: Created `components/app/project-status-toggle.tsx` — shadcn Select + Badge with color coding (green/amber/gray). useTransition for non-blocking updates.
- Task 6: Added "Projekte" (FolderOpen icon) to both app-header.tsx and bottom-nav.tsx. Tab order: Aufnahme | Projekte | Übersicht.
- Task 7: Created `app/app/projects/page.tsx` — project list with status badges, empty state with CTA, "Neues Projekt" button.
- Task 8: `pnpm build` passes with zero errors. `pnpm lint` clean (only pre-existing warning in scripts/stt-evaluation.ts). All routes registered correctly.

### File List

**New files:**
- `lib/validations/project.ts` — Zod schemas + types for project CRUD
- `lib/actions/projects.ts` — Server Actions for project CRUD (createProject, updateProject, updateProjectStatus, getProject, getProjects)
- `components/app/project-form.tsx` — Reusable project form (create/edit modes)
- `components/app/project-status-toggle.tsx` — Status dropdown with colored badge
- `app/app/projects/page.tsx` — Project list page (placeholder for Story 8.3)
- `app/app/projects/new/page.tsx` — Create project page (Server Component)
- `app/app/projects/new/project-create-form.tsx` — Create form client wrapper
- `app/app/projects/[id]/page.tsx` — Project detail page
- `app/app/projects/[id]/edit/page.tsx` — Edit project page (Server Component)
- `app/app/projects/[id]/edit/project-edit-form.tsx` — Edit form client wrapper

**Modified files:**
- `components/app/app-header.tsx` — Added "Projekte" nav link with FolderOpen icon
- `components/app/bottom-nav.tsx` — Added "Projekte" tab (3-tab layout)

### Senior Developer Review (AI)

**Reviewer:** Claude Opus 4.6 | **Date:** 2026-02-14

**Findings (1 High, 5 Medium, 3 Low):**

- **[H1][FIXED]** Zod transform `v || undefined` prevented clearing optional fields on update — changed to `v || null` (lib/validations/project.ts)
- **[M1][FIXED]** STATUS_CONFIG duplicated in project-status-toggle.tsx and projects/page.tsx — extracted to lib/validations/project.ts as shared export
- **[M2][FIXED]** `<dt>`/`<dd>` without `<dl>` wrapper on detail page — changed container to `<dl>` (projects/[id]/page.tsx)
- **[M3][FIXED]** Silent rollback on status update failure — added error state and user-facing error message (project-status-toggle.tsx)
- **[M4][FIXED]** Invalid UUID in route params caused 500 instead of 404 — added UUID regex validation with notFound() (projects/[id]/page.tsx, projects/[id]/edit/page.tsx)
- **[M5][FIXED]** Back button touch targets 36px (h-9 w-9) below 48px NFR requirement — changed to h-12 w-12 (new, [id], [id]/edit pages)
- **[L1]** Error banner uses amber colors instead of destructive — accepted (design choice)
- **[L2]** Redundant Badge + Select in ProjectStatusToggle — accepted (visual clarity)
- **[L3]** updateProject doesn't validate id as UUID (inconsistent with updateProjectStatusSchema) — accepted (caught by M4 UUID validation at route level)

**Verdict:** All HIGH and MEDIUM issues fixed. Build passes. Status → done.

### Change Log

- 2026-02-14: Implemented project CRUD with Server Actions, form components, and navigation (Story 8.2)
- 2026-02-14: Code review fixes — Zod null transform, shared STATUS_CONFIG, semantic HTML, error feedback, UUID validation, touch targets
