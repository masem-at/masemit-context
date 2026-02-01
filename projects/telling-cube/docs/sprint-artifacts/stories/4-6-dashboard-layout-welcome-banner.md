# Story 4.6: Dashboard Layout & Welcome Banner

**Epic:** 4 - Auth & User Dashboard (Phase 2)
**Status:** ready-for-dev
**Created:** 2026-01-21
**Depends On:** Story 4.4 (Session Management APIs)

## User Story

As a **logged-in user**,
I want **to see a personalized dashboard with my account information**,
So that **I feel welcomed and know my account status**.

## Acceptance Criteria

### AC1: Dashboard Display
**Given** a logged-in user visits /dashboard
**When** the page loads
**Then** the welcome banner displays "Welcome back, {email}!"
**And** the tier badge is displayed (e.g., "Lifetime Member") with correct color
**And** quick stats show total scenarios generated

### AC2: Authentication Redirect
**Given** an unauthenticated user visits /dashboard
**When** the page loads
**Then** the user is redirected to /login

### AC3: Responsive Layout
**Given** the dashboard is viewed on different screen sizes
**When** desktop (1024px+)
**Then** full layout is displayed with 3-column grid
**When** tablet (768-1023px)
**Then** 2-column grid with collapsed stats
**When** mobile (<768px)
**Then** 1-column stacked layout

## Technical Details

### Source References
- Architecture: `docs/architecture/auth-dashboard-architecture.md`
- UX Spec: `docs/ux/dashboard-ux-spec.md`
- Epic Definition: `docs/epics-big-bang.md` (Story 4.6)

### Tier Badge Colors (from UX spec)
- Free: Gray (bg-gray-100, text-gray-700)
- Single: Blue (bg-blue-100, text-blue-700)
- Lifetime: Emerald (bg-emerald-100, text-emerald-700)
- Lifetime+: Purple (bg-purple-100, text-purple-700)
- Pro: Teal (bg-teal-100, text-teal-700)
- Team: Indigo (bg-indigo-100, text-indigo-700)

### Files to Create
1. `app/dashboard/page.tsx` - Dashboard page (NEW)
2. `components/dashboard/WelcomeBanner.tsx` - Welcome banner component (NEW)
3. `components/dashboard/TierBadge.tsx` - Tier badge component (NEW)
4. `components/dashboard/QuickStats.tsx` - Stats footer component (NEW)

## Tasks

- [x] Create app/dashboard/page.tsx with auth check
- [x] Create WelcomeBanner component
- [x] Create TierBadge component using entitlements colors
- [x] Create QuickStats component
- [x] Add responsive grid layout

## Definition of Done

- [x] All acceptance criteria pass
- [x] Dashboard renders at /dashboard for logged-in users
- [x] Unauthenticated users redirected to /login
- [x] Tier badge displays correct color
- [x] Responsive on desktop, tablet, mobile
