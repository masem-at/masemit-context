# Story 3.12: API Docs Bug Fixes

**Status:** done
**Epic:** 3 - HR Data Domain (Maintenance)
**Points:** 1
**Created:** 2026-01-21

---

## Story

As a **user browsing the tellingCube website**,
I want **working navigation links and proper dark mode support**,
so that **I can navigate reliably and use my preferred color scheme**.

---

## Acceptance Criteria

1. **AC1: Header Examples Link Fixed**
   - [x] "Examples" link in header points to `/#examples` (not `#examples`)
   - [x] Link works from any page (API docs, What's Next, etc.)
   - [x] Scrolls to examples section on landing page

2. **AC2: Header Pricing Link Fixed**
   - [x] "Pricing" link in header points to `/#pricing` (not `#pricing`)
   - [x] Link works from any page
   - [x] Scrolls to pricing section on landing page

3. **AC3: API Docs Dark Mode Support**
   - [x] Main background uses `dark:` classes (not hardcoded light colors)
   - [x] All sections readable in dark mode
   - [x] Code blocks maintain contrast in both modes
   - [x] Tables and cards styled for dark mode

---

## Tasks / Subtasks

### Task 1: Fix Header Navigation Links (AC: #1, #2)
- [x] 1.1 Change `href="#examples"` to `href="/#examples"`
- [x] 1.2 Change `href="#pricing"` to `href="/#pricing"`
- [x] 1.3 Test from multiple pages

### Task 2: Fix API Docs Dark Mode (AC: #3)
- [x] 2.1 Change `bg-gradient-to-b from-gray-50 to-white` to include dark variants
- [x] 2.2 Update section backgrounds with dark mode classes
- [x] 2.3 Ensure tables have dark mode styling
- [x] 2.4 Verify code blocks work in both modes
- [x] 2.5 Test full page in dark mode

---

## Dev Notes

### Header Link Fix

In `components/layout/Header.tsx` (lines 55 and 61):

**Before:**
```tsx
<Link href="#examples">Examples</Link>
<Link href="#pricing">Pricing</Link>
```

**After:**
```tsx
<Link href="/#examples">Examples</Link>
<Link href="/#pricing">Pricing</Link>
```

### Dark Mode Pattern

In `app/api-docs/page.tsx` (line 55):

**Before:**
```tsx
<main className="min-h-screen bg-gradient-to-b from-gray-50 to-white">
```

**After:**
```tsx
<main className="min-h-screen bg-gradient-to-b from-gray-50 to-white dark:from-gray-900 dark:to-gray-950">
```

### Section Backgrounds to Update

Several sections use light-only colors:
- `bg-white` → `bg-white dark:bg-gray-900`
- `bg-gray-50` → `bg-gray-50 dark:bg-gray-800`
- `from-blue-50` → `from-blue-50 dark:from-blue-900/20`
- `border-gray-200` → `border-gray-200 dark:border-gray-700`
- `text-gray-900` → `text-gray-900 dark:text-gray-100`
- `text-gray-600` → `text-gray-600 dark:text-gray-400`

### Files to Modify

- `components/layout/Header.tsx` - Fix anchor links (lines 55, 61)
- `app/api-docs/page.tsx` - Add dark mode classes throughout

---

## Dev Agent Record

### Agent Model Used

Claude Opus 4.5 (claude-opus-4-5-20251101)

### Completion Notes List

- [x] Story created by Bob (SM)
- [x] Header links fixed (`#examples` → `/#examples`, `#pricing` → `/#pricing`)
- [x] API docs dark mode implemented (all sections, tables, cards, badges)
- [x] IBCSLink and CodeBlock components updated for dark mode
- [x] TypeScript passes
- [x] Code review passed

### File List

**Files modified:**
- `components/layout/Header.tsx` - Fixed Examples and Pricing links
- `app/api-docs/page.tsx` - Added comprehensive dark mode support
- `docs/sprint-artifacts/3-12-api-docs-bug-fixes.md` - This story file

### Change Log

- 2026-01-21: Story 3.12 created via Party Mode (bug fixes identified by Winston & Sally)
- 2026-01-22: All tasks implemented - Header links fixed, API docs dark mode complete
