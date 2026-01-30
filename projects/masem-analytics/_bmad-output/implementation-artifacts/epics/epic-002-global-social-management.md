# Epic 002: Global Social Post Management

**Status:** Ready for Development
**Priority:** High
**Created:** 2026-01-30
**Owner:** Sempre

---

## Epic Summary

Decouple social post management from individual projects. Create a dedicated global Social page (`/dashboard/social`) where posts are imported, managed, and assigned to projects. Add language detection, auto-title fetching, and a shared table component used across global and project views.

---

## Business Value

- **Faster Import Workflow:** Import posts without selecting a project upfront -- assign later
- **Central Overview:** One place to see all social posts across all projects
- **Better Data Quality:** Auto-title and language detection reduce manual data entry
- **Consistent UX:** Shared table component ensures feature parity across views

---

## Technical Context

### DB Changes
- `social_posts.project_id` → nullable (currently NOT NULL)
- New column `social_posts.language` VARCHAR(5) DEFAULT NULL (NULL renders as "N/A" in UI)

### API Changes
- `/api/social/import` → `projectId` becomes optional
- `/api/social/[postId]` PATCH → add `project_id`, `language` fields
- New query function for global (unfiltered) post listing with optional date range
- New server-side utility: fetch `og:title` from URL (best effort, timeout 5s)

### New Route
- `/dashboard/social` → Server Component with own date filter (Von/Bis, date only)

### Shared Component
- Refactor `SocialTabClient` into shared component with `mode: 'project' | 'global'`
- Global mode: shows project column (inline editable), no external filter dependency
- Project mode: filtered by project, as currently

---

## Stories

### Story 2.1: DB Migration -- Nullable project_id + language column

**Priority:** P0 (Blocker for all other stories)

#### Description
Make `social_posts.project_id` nullable and add `language` column.

#### Acceptance Criteria
- [ ] AC-2.1.1: `project_id` column on `social_posts` allows NULL values
- [ ] AC-2.1.2: New column `language` VARCHAR(5) DEFAULT NULL exists on `social_posts`
- [ ] AC-2.1.3: Existing rows retain their current `project_id` values unchanged
- [ ] AC-2.1.4: Migration SQL file created at `lib/migrations/004-social-global.sql`
- [ ] AC-2.1.5: Migration applied successfully to Neon DB

---

### Story 2.2: Auto-Title Fetch via og:title

**Priority:** P1

#### Description
Create a server-side utility that fetches `og:title` from a given URL. Used during import and manual post creation as the default title (supersedes filename fallback for import when og:title is available).

#### Acceptance Criteria
- [ ] AC-2.2.1: New utility `lib/social/fetch-og-title.ts` exports `fetchOgTitle(url: string): Promise<string | null>`
- [ ] AC-2.2.2: HTTP GET request with 5-second timeout, follows redirects
- [ ] AC-2.2.3: Parses `<meta property="og:title" content="...">` from response HTML
- [ ] AC-2.2.4: Returns `null` on any failure (403, timeout, parse error, no og:title)
- [ ] AC-2.2.5: Also extracts `lang` attribute from `<html lang="...">` or `og:locale` and returns it as secondary value
- [ ] AC-2.2.6: Return type: `{ title: string | null; language: string | null }`

---

### Story 2.3: Update Import API -- Optional project, auto-title, language detection

**Priority:** P1

#### Description
Update `/api/social/import` so `projectId` is optional. Use og:title fetch for auto-title (fallback to filename). Detect language from page metadata.

#### Acceptance Criteria
- [ ] AC-2.3.1: `projectId` is optional in the import request -- posts can be created with `project_id = NULL`
- [ ] AC-2.3.2: For each imported file, call `fetchOgTitle(postUrl)` to get title and language
- [ ] AC-2.3.3: Title priority: og:title > filename (without .xlsx) > NULL
- [ ] AC-2.3.4: Language priority: detected from HTML lang/og:locale > NULL (shown as "N/A" in UI)
- [ ] AC-2.3.5: Language values normalized to "DE" or "EN" (case-insensitive match on `de`, `en`, `de-AT`, `de-DE`, `en-US`, etc.)
- [ ] AC-2.3.6: Unrecognized language codes stored as NULL (shown as "N/A")
- [ ] AC-2.3.7: Import response unchanged in structure (imported/updated/errors)

---

### Story 2.4: Update PATCH API -- project_id + language editable

**Priority:** P1

#### Description
Extend `/api/social/[postId]` PATCH to accept `project_id` and `language` fields for inline editing.

#### Acceptance Criteria
- [ ] AC-2.4.1: PATCH accepts optional `project_id` (string or null) -- updates `social_posts.project_id`
- [ ] AC-2.4.2: PATCH accepts optional `language` (string or null) -- updates `social_posts.language`
- [ ] AC-2.4.3: `project_id` validated against existing projects (returns 400 if invalid)
- [ ] AC-2.4.4: `language` validated: only "DE", "EN", or null accepted
- [ ] AC-2.4.5: Existing fields (title, ad_spend) continue to work unchanged

---

### Story 2.5: Shared Social Table Component

**Priority:** P1 (Depends on 2.1, 2.4)

#### Description
Refactor `SocialTabClient` into a shared component that works in two modes: `project` (filtered, as today) and `global` (unfiltered, with project assignment column).

#### Acceptance Criteria
- [ ] AC-2.5.1: New component `SocialTable` with prop `mode: 'project' | 'global'`
- [ ] AC-2.5.2: Global mode shows additional "Project" column with inline-editable project assignment (dropdown of all projects)
- [ ] AC-2.5.3: Global mode shows all posts regardless of project assignment (including unassigned)
- [ ] AC-2.5.4: Project mode behaves identical to current `SocialTabClient` (filtered by project)
- [ ] AC-2.5.5: Language badge displayed: "DE" (blue), "EN" (green), "N/A" (gray/amber)
- [ ] AC-2.5.6: Post type badge displayed as icon: person icon for personal, building icon for company
- [ ] AC-2.5.7: Language inline editable (click badge → select DE/EN)
- [ ] AC-2.5.8: Mobile card view works in both modes (responsive, no horizontal scroll)
- [ ] AC-2.5.9: Existing features preserved: inline title edit, inline ad spend, post link icon, platform icon, snapshot count

---

### Story 2.6: Global Social Page with Date Filter

**Priority:** P2 (Depends on 2.3, 2.5)

#### Description
Create `/dashboard/social` page with import area, date filter (Von/Bis, date only), and the shared social table in global mode.

#### Acceptance Criteria
- [ ] AC-2.6.1: Route `/dashboard/social` renders a server component page
- [ ] AC-2.6.2: Header with "Social Posts" title and back link to `/dashboard`
- [ ] AC-2.6.3: Import section: platform + post type selects, drag-and-drop XLSX upload (no project select required)
- [ ] AC-2.6.4: Manual post form available (adapted without required project)
- [ ] AC-2.6.5: Date filter with Von/Bis date inputs (no time), applied via URL search params `start` and `end`
- [ ] AC-2.6.6: Date filter only affects this page, not the global dashboard filter
- [ ] AC-2.6.7: Posts listed DESC by `published_at`
- [ ] AC-2.6.8: SocialTable in `global` mode with all features (project assignment, badges, inline editing)
- [ ] AC-2.6.9: Posts with assigned project also appear on respective project's Social tab

---

### Story 2.7: Global Social Query Function

**Priority:** P1 (Depends on 2.1)

#### Description
Add query function for fetching all social posts globally with optional date range filter, independent of project.

#### Acceptance Criteria
- [ ] AC-2.7.1: New function `getGlobalSocialPosts(startDate?: string, endDate?: string): Promise<SocialPost[]>` in `lib/queries/social.ts`
- [ ] AC-2.7.2: Returns all posts with latest metrics via LATERAL JOIN (same pattern as `getSocialPosts`)
- [ ] AC-2.7.3: Includes `project_id` and project name in result (LEFT JOIN on projects)
- [ ] AC-2.7.4: Optional date range filters on `published_at`
- [ ] AC-2.7.5: Ordered by `published_at DESC`
- [ ] AC-2.7.6: No project filter applied

---

### Story 2.8: Navigation + Dashboard Link

**Priority:** P2 (Depends on 2.6)

#### Description
Add navigation link to the global Social page from the main dashboard.

#### Acceptance Criteria
- [ ] AC-2.8.1: Link to `/dashboard/social` visible on main dashboard page (e.g., in header or sidebar)
- [ ] AC-2.8.2: Current page highlighted in navigation when active
- [ ] AC-2.8.3: Consistent styling with existing dashboard navigation

---

## Implementation Order

```
2.1 (DB Migration)
 ├── 2.2 (og:title Fetch)
 ├── 2.4 (PATCH API Update)
 └── 2.7 (Global Query)
      │
      ├── 2.3 (Import API Update) ← depends on 2.2
      └── 2.5 (Shared Table) ← depends on 2.4
           │
           └── 2.6 (Global Page) ← depends on 2.3, 2.5, 2.7
                │
                └── 2.8 (Navigation)
```

---

## Out of Scope

- Bulk operations (mass assign project, mass delete)
- Social post scheduling or publishing
- OAuth-based LinkedIn/X API integration
- Advanced analytics on the global social page
