# Story 12.2: Invitation System — Create, Send & Redeem

Status: done

<!-- Note: Validation is optional. Run validate-create-story for quality check before dev-story. -->

## Story

As a **Meister**,
I want to invite team members by entering their email and role,
so that they receive an invitation email and can join my company with one click.

## Acceptance Criteria

1. **Given** a Meister/Buero on `/app/team` **When** clicking "Mitarbeiter einladen" and entering email + role (Monteur/Meister/Buero) **Then** a Server Action creates an invitation record with a UUID token and 7-day expiry **And** an invitation email is sent via Resend with a prominent "Team beitreten" button linking to `/invite/[token]` **And** the email uses kurzum branding (same style as magic link email) **And** the email subject is "kurzum. — Du wurdest eingeladen" **And** `hasPermission(role, 'team:invite')` is checked before creating the invitation.

2. **Given** an invited user clicks the invitation link **When** they visit `/invite/[token]` **Then** the token is validated (exists, not expired, not used) **And** the page shows: company name, invited role, and a "Jetzt beitreten" button **And** clicking the button triggers the Magic Link login flow **And** after successful login/registration, the user is automatically assigned to the inviting company with the specified role **And** the invitation is marked as used (`usedAt = now()`) **And** `findOrCreateUser()` uses the invitation's companyId instead of DEFAULT_COMPANY_ID **And** the user is redirected to `/app`.

3. **Given** an expired or already-used invitation token **When** the user visits the invite link **Then** a friendly German message explains the issue ("Einladung abgelaufen" / "Einladung bereits verwendet") **And** a hint to contact the team admin is shown.

4. **Given** the invitation flow **When** measured end-to-end (email click to dashboard) **Then** completion takes < 30 seconds (S2-NFR6).

**FRs:** S2-FR1, S2-FR2, S2-FR3 | **NFRs:** S2-NFR2, S2-NFR6

## Tasks / Subtasks

- [x] Task 1: Create invitation Server Action (AC: #1)
  - [x] 1.1 Create `lib/actions/team.ts` with `createInvitation(formData: FormData)` Server Action
  - [x] 1.2 Validate auth context via `getAuthContext()` + `requirePermission(role, 'team:invite')`
  - [x] 1.3 Validate input: email (Zod via `inviteSchema`), role (must be valid `Role` value from pgEnum)
  - [x] 1.4 Check for duplicate active invitation: same email + companyId + not expired + not used → reject with "Einladung bereits aktiv"
  - [x] 1.5 Generate UUID token via `crypto.randomUUID()`
  - [x] 1.6 Insert into `invitations` table: companyId (from auth context), email (lowercased), role, token, invitedBy (userId), expiresAt (7 days from now)
  - [x] 1.7 Send invitation email fire-and-forget via `sendInvitationEmail()` — `.catch()` pattern, never block response
  - [x] 1.8 Return success/error response
  - [x] 1.9 Create `getPendingInvitations(companyId)` query for the team page (email, role, expiresAt, createdAt)

- [x] Task 2: Add invitation email function (AC: #1)
  - [x] 2.1 Add `sendInvitationEmail({ to, inviterName, companyName, role, token })` to `lib/email.ts`
  - [x] 2.2 Create `buildInvitationHtml()` with kurzum branding — same layout as `buildLoginHtml()`: kurzum. logo, white card on stone-50 background, orange CTA button, footer
  - [x] 2.3 Email subject: `"kurzum. — Du wurdest eingeladen"`
  - [x] 2.4 Email body text: `"Hallo, [inviterName] hat dich eingeladen, bei [companyName] als [role] beizutreten."` + "Team beitreten" button → link to `${APP_URL}/invite/[token]`
  - [x] 2.5 Use `escapeHtml()` for all dynamic values (XSS prevention)
  - [x] 2.6 Fallback link text below button (same pattern as login email)

- [x] Task 3: Create `/invite/[token]` page (AC: #2, #3)
  - [x] 3.1 Create `app/invite/[token]/page.tsx` as async Server Component (public — NOT behind auth middleware, middleware matcher `["/app/:path*", "/api/voice/:path*"]` does not match `/invite/*`)
  - [x] 3.2 Look up invitation by `params.token`: JOIN with `companies` table to get company name, JOIN with `users` (invitedBy) to get inviter name
  - [x] 3.3 If token not found → show "Einladung nicht gefunden" + hint "Bitte kontaktiere deinen Teamleiter"
  - [x] 3.4 If token expired (`expiresAt < now()`) → show "Einladung abgelaufen" + hint
  - [x] 3.5 If token already used (`usedAt IS NOT NULL`) → show "Einladung bereits verwendet" + "Du kannst dich direkt einloggen" link to `/login`
  - [x] 3.6 If valid → show: kurzum. logo, company name, invited role (German: Monteur/Meister/Buero), inviter name, "Jetzt beitreten" button
  - [x] 3.7 "Jetzt beitreten" button → `<Link href="/login?invite=[token]">` (passes token to login flow)
  - [x] 3.8 Layout: centered card, kurzum branding, minimal — similar to `/login` page aesthetic

- [x] Task 4: Modify login flow for invitation context (AC: #2)
  - [x] 4.1 Create `lib/validations/team.ts` with `inviteSchema` (Zod: email + role) for the invitation form
  - [x] 4.2 Update login page (`app/login/page.tsx`): read `invite` from `searchParams`, pass as hidden field to login API call body, show context text "Du wurdest eingeladen — bitte logge dich ein" when invite param present
  - [x] 4.3 Update `loginSchema` in `lib/validations/auth.ts`: add optional `inviteToken: z.string().uuid().optional()` field
  - [x] 4.4 Update `/api/auth/login` route (`app/api/auth/login/route.ts`): if `inviteToken` provided, look up invitation, validate (exists, not expired, not used, email matches lowercase), pass invitation's `companyId` + `role` to `findOrCreateUser()`
  - [x] 4.5 Update `findOrCreateUser()` in `lib/auth/user.ts`: add optional second parameter `options?: { companyId?: string; role?: Role }` — if provided, use instead of DEFAULT_COMPANY_ID
  - [x] 4.6 If invite token provided but invalid or email mismatch → return error "Ungueltige Einladung" (do not reveal details)

- [x] Task 5: Complete invitation in verify route (AC: #2)
  - [x] 5.1 Update `/api/auth/verify` route (`app/api/auth/verify/route.ts`): after session creation, look up the user's email
  - [x] 5.2 Find unused invitation for that email: `WHERE email = ? AND usedAt IS NULL AND expiresAt > now()` (most recent by createdAt)
  - [x] 5.3 Atomically mark invitation as used: `UPDATE invitations SET usedAt = now(), updatedAt = now() WHERE id = ? AND usedAt IS NULL RETURNING *`
  - [x] 5.4 If user already existed (not just created via invitation): UPDATE user's `companyId` + `role` to match invitation values
  - [x] 5.5 Redirect to `/app` as usual

- [x] Task 6: Team page with invitation form UI (AC: #1)
  - [x] 6.1 Create `app/app/team/page.tsx` — Server Component: `getAuthContext()` + `requirePermission(role, 'team:manage')`, fetch pending invitations
  - [x] 6.2 Create `components/app/invite-form.tsx` — Client Component: email input + role select (shadcn Select with `Controller` from RHF) + submit button
  - [x] 6.3 Role select options: Monteur, Meister, Buero (German labels)
  - [x] 6.4 Form validation via `useForm` + Zod resolver (same pattern as waitlist form)
  - [x] 6.5 Submit calls `createInvitation()` Server Action via `useTransition` or form action
  - [x] 6.6 Success toast/message: "Einladung an [email] gesendet"
  - [x] 6.7 Show pending invitations list below form: email, role, expires (simple table/list)
  - [x] 6.8 Page title: "Team" with subtitle "Mitarbeiter einladen und verwalten"

- [x] Task 7: Add Team nav link with RBAC gating (AC: #1)
  - [x] 7.1 Update `app/app/layout.tsx`: destructure `role` from `getAuthContext()`, pass to `AppHeader` and `BottomNav`
  - [x] 7.2 Update `components/app/app-header.tsx`: accept `role` prop, conditionally add `{ href: "/app/team", label: "Team", Icon: Users }` to navLinks only if `hasPermission(role, 'team:manage')`
  - [x] 7.3 Update `components/app/bottom-nav.tsx`: same RBAC-gated Team link
  - [x] 7.4 Import `Users` icon from lucide-react, `hasPermission` from `@/lib/auth/rbac`

## Dev Notes

### Architecture & Implementation Patterns

- **Server Actions for mutations**: All mutations use Server Actions consistent with the existing codebase. `getAuthContext()` provides auth context, `requirePermission()` enforces RBAC before any DB writes. [Source: lib/auth/session.ts, lib/auth/rbac.ts]
- **Fire-and-forget email**: Same pattern as `sendLoginEmail()` — `.catch()` on the promise, never block the HTTP response. [Source: lib/email.ts:76-85]
- **Atomic token operations**: Use `UPDATE ... WHERE ... AND usedAt IS NULL RETURNING *` to prevent TOCTOU race conditions. This is the exact same pattern as magic link verification. [Source: lib/auth/magic-link.ts#verifyMagicLink]
- **Server Components for pages**: All pages are async Server Components with direct Drizzle queries. Client Components only for interactive forms (invite form). [Source: app/app/layout.tsx]
- **Resend SDK pattern**: `new Resend(process.env.RESEND_API_KEY)`, from: `"Mario von kurzum <${RESEND_FROM_EMAIL}>"`, inline HTML with `escapeHtml()`. [Source: lib/email.ts:1-6]
- **Zod v4**: Use `z.email()` (top-level, NOT `z.string().email()`). [Source: MEMORY.md#Story 3.1]
- **shadcn Select + RHF**: Use `Controller` wrapper, not `register` (Radix primitives). [Source: MEMORY.md#Story 3.1]

### Critical Constraints

- **Invitations table already exists in DB**: Story 12.1 created the `invitations` table with all columns, FK constraints, and indexes (token, companyId, compound companyId+email). NO new Drizzle migration needed. [Source: drizzle/0008_next_sauron.sql, drizzle/0009_overconfident_the_order.sql]
- **`/invite/[token]` is public**: Middleware matcher is `["/app/:path*", "/api/voice/:path*"]` — `/invite/*` is NOT matched, so it's publicly accessible. Do NOT add it to middleware. [Source: middleware.ts]
- **Token must be UUID**: Use `crypto.randomUUID()` for token generation. The `token` column is `text` type with unique constraint (not uuid type). [Source: lib/db/schema.ts:51]
- **7-day expiry**: `expiresAt = new Date(Date.now() + 7 * 24 * 60 * 60 * 1000)` — same approach as session (7 days) and magic link (15 min). [Source: lib/auth/session.ts:69]
- **Email case normalization**: Always lowercase emails before comparing or storing. `findOrCreateUser()` already does this. [Source: lib/auth/user.ts:5]
- **Race-safe user creation**: `findOrCreateUser()` uses `onConflictDoNothing` + fallback query — preserve this when adding companyId override. [Source: lib/auth/user.ts:14-24]
- **No SMS invitations**: Email-only per Sprint 2 scope. [Source: docs/_masemIT/bmad-briefing-kurzum-sprint2.md#Section 6]
- **No Starter Plan limits**: 3-user limit enforcement is deferred to Sprint 3. [Source: docs/_masemIT/bmad-briefing-kurzum-sprint2.md#Section 6]
- **Existing user accepting invitation**: If an already-registered user accepts an invitation to a different company, their `companyId` and `role` are UPDATED. They leave their previous company. This is intentional per AC ("automatically assigned to the inviting company"). Note: their old voice messages/projects stay with the old company.

### Existing Code to Reuse

| Component/Function | Source | Reuse Approach |
|---|---|---|
| `getAuthContext()` | `lib/auth/session.ts` | Auth context for Server Actions — use as-is |
| `hasPermission()` / `requirePermission()` | `lib/auth/rbac.ts` | Check `team:invite` before creating invitation |
| `findOrCreateUser()` | `lib/auth/user.ts` | **MODIFY**: add optional `options: { companyId?, role? }` param |
| `DEFAULT_COMPANY_ID` | `lib/auth/user.ts` | Keep as fallback when no invitation context |
| `sendLoginEmail()` pattern | `lib/email.ts` | **COPY pattern** for `sendInvitationEmail()` |
| `buildLoginHtml()` layout | `lib/email.ts:172-202` | **COPY layout** for `buildInvitationHtml()` |
| `escapeHtml()` | `lib/email.ts:204-210` | Reuse for invitation email HTML |
| `loginSchema` | `lib/validations/auth.ts` | **MODIFY**: add optional `inviteToken` field |
| Login route | `app/api/auth/login/route.ts` | **MODIFY**: handle invitation context |
| Verify route | `app/api/auth/verify/route.ts` | **MODIFY**: mark invitation used after session |
| `createMagicLink()` | `lib/auth/magic-link.ts` | Use as-is in invitation login flow |
| `createSession()` | `lib/auth/session.ts` | Use as-is in verify route |
| `successResponse()` / `errorResponse()` | `lib/api-response.ts` | Use for form validation responses |
| `checkAuthRateLimit()` | `lib/rate-limit.ts` | Apply to invitation creation (prevent spam) |
| Invitation schema + types | `lib/db/schema.ts:43-66` | Use `Invitation`, `NewInvitation` types |
| `AppHeader` | `components/app/app-header.tsx` | **MODIFY**: add Team nav link + role prop |
| `BottomNav` | `components/app/bottom-nav.tsx` | **MODIFY**: add Team nav link + role prop |
| App layout | `app/app/layout.tsx` | **MODIFY**: pass role to nav components |

### Invitation Schema Reference (Already in DB)

```typescript
// lib/db/schema.ts lines 43-66 — created by Story 12.1
export const invitations = pgTable("invitations", {
  id: uuid("id").primaryKey().defaultRandom(),
  companyId: uuid("company_id").references(() => companies.id).notNull(),
  email: text("email").notNull(),
  role: userRoleEnum("role").notNull(),            // "monteur" | "meister" | "buero"
  token: text("token").notNull().unique(),
  invitedBy: uuid("invited_by").references(() => users.id).notNull(),
  expiresAt: timestamp("expires_at", { withTimezone: true }).notNull(),
  usedAt: timestamp("used_at", { withTimezone: true }),                   // null = pending
  createdAt: timestamp("created_at", { withTimezone: true }).defaultNow().notNull(),
  updatedAt: timestamp("updated_at", { withTimezone: true }).defaultNow().notNull(),
}, (table) => [
  index("idx_invitations_token").on(table.token),
  index("idx_invitations_company_id").on(table.companyId),
  index("idx_invitations_company_email").on(table.companyId, table.email),
]);
```

### Invitation Flow Diagram

```
CREATION FLOW:
Meister/Buero → /app/team → "Mitarbeiter einladen"
  → Enter email + role (Monteur/Meister/Buero)
  → Server Action: createInvitation()
    → requirePermission(role, 'team:invite')
    → Validate: email format, role, no duplicate active invitation
    → INSERT invitations (token=UUID, expiresAt=7d)
    → sendInvitationEmail() (fire-and-forget)
  → Success: "Einladung an [email] gesendet"

REDEMPTION FLOW:
Invited user clicks email → /invite/[token]
  → Server Component: validate token (exists + not expired + not used)
  → Show: company name, role, inviter, "Jetzt beitreten"
  → Click → /login?invite=[token]
  → Login page: email form (invite token as hidden field)
  → POST /api/auth/login
    → Validate invitation + email match
    → findOrCreateUser(email, { companyId, role })
    → Create magic link + send email
  → User clicks magic link → /api/auth/verify
    → Create session
    → Mark invitation used (usedAt = now())
    → If existing user: update companyId + role
    → Redirect → /app
```

### Edge Cases

| Case | Behavior |
|---|---|
| Duplicate active invitation (same email + company) | Reject: "Einladung bereits aktiv fuer diese E-Mail" |
| Existing user from DIFFERENT company | Accept: update companyId + role (user switches companies) |
| Existing user from SAME company | Accept: update role only |
| Email case mismatch | Normalize both to lowercase before comparison |
| Concurrent token redemption | Atomic UPDATE with `usedAt IS NULL` prevents double-use |
| Invitation email != login email | Reject: "Bitte verwende die eingeladene E-Mail-Adresse" |
| Expired invitation on invite page | Show: "Einladung abgelaufen — bitte kontaktiere deinen Teamleiter" |
| Used invitation on invite page | Show: "Einladung bereits verwendet" + link to `/login` |
| Monteur tries to invite | `requirePermission()` throws `PermissionError` — return 403 |
| Already logged in user visits invite page | Page renders normally (it's public). They can click "Jetzt beitreten" and go through the login flow again. |

### Project Structure Notes

**New files:**
- `app/invite/[token]/page.tsx` — Public invitation landing page (Server Component)
- `app/app/team/page.tsx` — Team management page (Server Component, RBAC-gated)
- `components/app/invite-form.tsx` — Client Component: invitation form (email + role + submit)
- `lib/actions/team.ts` — Server Actions: `createInvitation()`, `getPendingInvitations()`
- `lib/validations/team.ts` — Zod schemas: `inviteSchema` (email + role validation)

**Modified files:**
- `lib/auth/user.ts` — `findOrCreateUser()`: add optional `{ companyId, role }` override parameter
- `lib/email.ts` — Add `sendInvitationEmail()` + `buildInvitationHtml()`
- `lib/validations/auth.ts` — Add optional `inviteToken` field to `loginSchema`
- `app/api/auth/login/route.ts` — Handle invitation context: validate invite token, pass companyId to findOrCreateUser
- `app/api/auth/verify/route.ts` — Mark invitation used after session creation, update user if existing
- `app/login/page.tsx` — Read `invite` query param, pass as hidden field, show invitation context text
- `components/app/app-header.tsx` — Add `role` prop, conditionally show Team nav link
- `components/app/bottom-nav.tsx` — Add `role` prop, conditionally show Team nav link
- `app/app/layout.tsx` — Destructure `role` from `getAuthContext()`, pass to AppHeader + BottomNav

### Testing Strategy

- **Test 1**: Meister logs in → `/app/team` → enters email + Monteur role → "Einladen" → verify invitation in DB + email received with correct link
- **Test 2**: Click invitation email link → `/invite/[token]` shows company name, role, inviter name, "Jetzt beitreten" button
- **Test 3**: Click "Jetzt beitreten" → login page (with invite context) → enter email → magic link → click → user created with correct company + role → dashboard
- **Test 4**: Visit `/invite/[token]` with expired token → "Einladung abgelaufen" message
- **Test 5**: Visit `/invite/[token]` with already-used token → "Einladung bereits verwendet" message
- **Test 6**: Visit `/invite/[token]` with non-existent token → "Einladung nicht gefunden" message
- **Test 7**: Monteur tries to access `/app/team` → permission denied / redirect
- **Test 8**: Try to use invitation with different email than invited → error "Bitte verwende die eingeladene E-Mail-Adresse"
- **Test 9**: `npx tsc --noEmit` passes with zero errors
- **Test 10**: `npx next build` succeeds with all routes compiling

### References

- [Source: _bmad-output/planning-artifacts/epics.md#Story 12.2] — Acceptance criteria (S2-FR1, S2-FR2, S2-FR3)
- [Source: docs/_masemIT/bmad-briefing-kurzum-sprint2.md#AP 2.2] — Team invitation flow, RBAC middleware, schema reference
- [Source: lib/db/schema.ts#invitations] — Invitations table (created by Story 12.1, migrations 0008+0009)
- [Source: lib/auth/rbac.ts] — RBAC module: hasPermission, requirePermission, team:invite permission
- [Source: lib/auth/user.ts#findOrCreateUser] — User creation with DEFAULT_COMPANY_ID (must be modified)
- [Source: lib/auth/session.ts#getAuthContext] — Auth context for Server Actions
- [Source: lib/auth/magic-link.ts] — Magic link creation + atomic verification pattern
- [Source: lib/email.ts#sendLoginEmail] — Email sending pattern to replicate for invitations
- [Source: lib/email.ts#buildLoginHtml] — HTML email template layout to copy
- [Source: app/api/auth/login/route.ts] — Login route (must add invitation context handling)
- [Source: app/api/auth/verify/route.ts] — Verify route (must mark invitation used)
- [Source: middleware.ts#config] — Route matcher confirms `/invite/*` is public
- [Source: components/app/app-header.tsx] — Nav component (must add Team link with RBAC)
- [Source: components/app/bottom-nav.tsx] — Mobile nav (must add Team link with RBAC)
- [Source: app/app/layout.tsx] — App layout (must pass role to nav components)
- [Source: _bmad-output/implementation-artifacts/12-1-pgenum-role-migration-and-rbac-permission-module.md] — Previous story: RBAC module, invitations schema, Vitest infrastructure

### Previous Story Intelligence (12.1)

- **RBAC module complete and tested**: `hasPermission()` and `requirePermission()` are ready with 20 unit tests covering the full permission matrix. `team:invite` permission is already defined for meister + buero roles.
- **pgEnum enforces roles at DB level**: No runtime validation of role values needed — DB rejects invalid values.
- **`getAuthContext()` returns typed `Role`**: Can use directly with `requirePermission()` without casting.
- **Invitations table is schema-only**: Created in Story 12.1 with all columns, indexes, and FK constraints. Compound index `(companyId, email)` was specifically added for Story 12.2 lookup performance.
- **Vitest infrastructure exists**: `vitest.config.ts` with `@/` alias, test scripts in `package.json`. Can add invitation unit tests if needed.
- **Migration 0008 + 0009 applied**: 3 pgEnum types + invitations table + updatedAt column + compound index. No further schema changes needed.

### Git Intelligence

Recent commits:
- `90fd25b` Add pgEnum for roles/status columns, RBAC permission module, and invitations schema (Story 12.1)
- `35f118a` Plan Sprint 2: Team/RBAC, Foto-Capture, Assignment V2, Dashboard extension

Pattern: Single commit per story, descriptive message referencing story number.
Suggested commit: `"Implement invitation system: create, send & redeem team invitations (Story 12.2)"`

## Dev Agent Record

### Agent Model Used

Claude Opus 4.6

### Debug Log References

- TypeScript initial build failed: `as const` nav arrays incompatible with `.push()` — fixed by typing arrays as `NavLink[]`/`NavTab[]` with `LucideIcon` interface
- All 20 existing RBAC unit tests pass (no regressions)
- `npx tsc --noEmit` passes with zero errors
- `npx next build` succeeds with all routes compiling (27 routes)

### Completion Notes List

- **Task 1**: Created `createInvitation()` Server Action with full RBAC check (`team:invite`), Zod validation, duplicate detection, UUID token generation, 7-day expiry, fire-and-forget email, and `getPendingInvitations()` query
- **Task 2**: Added `sendInvitationEmail()` + `buildInvitationHtml()` to lib/email.ts — same kurzum branding layout as login email, with "Team beitreten" CTA button, escapeHtml for all dynamic values
- **Task 3**: Created `/invite/[token]` public Server Component page — handles all states: not found, expired, already used (with login link), and valid (shows company, role, inviter, "Jetzt beitreten" button)
- **Task 4**: Extended login flow — added `inviteToken` to loginSchema, login page reads `invite` searchParam and shows "Du wurdest eingeladen" context, login route validates invitation + email match before creating user, `findOrCreateUser()` accepts optional `{ companyId, role }` override
- **Task 5**: Updated verify route — after session creation, looks up pending invitation by email, atomically marks as used (`usedAt IS NULL` guard), updates user's companyId + role for both new and existing users
- **Task 6**: Created `/app/team` page (RBAC-gated via `team:manage`) with InviteForm client component (shadcn Select + Controller + RHF + Zod) and pending invitations list
- **Task 7**: Added RBAC-gated "Team" nav link (Users icon) to AppHeader + BottomNav — only visible for meister/buero roles

### Change Log

- 2026-02-15: Implemented invitation system — create, send & redeem team invitations (Story 12.2)
- 2026-02-15: Code Review (AI) — 7 fixes applied:
  - [H1] Pass invite token through magic link verify URL to prevent multi-invitation mismatch (verify route, email, login route)
  - [H2] Remove unused imports `gt`, `isNull`, `and`, `sql` from invite page
  - [M1] Add rate limiting (10/hour per inviter) to `createInvitation()` via Upstash
  - [M2] Combine two separate user UPDATE queries into one in verify route
  - [M3] Remove unnecessary `"use strict"` from `lib/validations/team.ts`
  - [M4] Add `truncate` CSS to BottomNav labels for 5-item safety
  - [M5] Fix Select reset: use `value ?? ""` for proper Radix placeholder after form reset

### File List

**New files:**
- `lib/validations/team.ts`
- `lib/actions/team.ts`
- `app/invite/[token]/page.tsx`
- `app/app/team/page.tsx`
- `components/app/invite-form.tsx`

**Modified files:**
- `lib/email.ts` — added `sendInvitationEmail()` + `buildInvitationHtml()`, added optional `inviteToken` param to `sendLoginEmail()` [review fix H1]
- `lib/validations/auth.ts` — added optional `inviteToken` to `loginSchema`
- `lib/auth/user.ts` — added optional `options: { companyId?, role? }` param to `findOrCreateUser()`
- `lib/rate-limit.ts` — added `checkInviteRateLimit()` (10/hour per user via Upstash) [review fix M1]
- `app/api/auth/login/route.ts` — invitation validation + `findOrCreateUser()` with invite context, pass `inviteToken` to `sendLoginEmail()` [review fix H1]
- `app/api/auth/verify/route.ts` — mark invitation used + update user company/role, prefer specific invite token from URL, single user UPDATE [review fix H1+M2]
- `app/login/page.tsx` — read `invite` param, pass to LoginForm, show context text
- `components/app/login-form.tsx` — accept `inviteToken` prop, send in API call
- `components/app/app-header.tsx` — accept `role` prop, conditionally add Team nav link
- `components/app/bottom-nav.tsx` — accept `role` prop, conditionally add Team nav link, truncate labels [review fix M4]
- `app/app/layout.tsx` — destructure `role` from `getAuthContext()`, pass to nav components
