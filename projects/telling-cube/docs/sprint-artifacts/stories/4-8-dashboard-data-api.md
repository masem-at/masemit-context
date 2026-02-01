# Story 4.8: Dashboard Data API

**Epic:** 4 - Auth & User Dashboard (Phase 2)
**Status:** done
**Created:** 2026-01-21
**Depends On:** Story 4.4 (Session Management APIs)

## User Story

As a **frontend**,
I want **API endpoints to fetch dashboard data**,
So that **the dashboard can display user scenarios and stats**.

## Acceptance Criteria

### AC1: Dashboard Summary Endpoint
**Given** a logged-in user calls GET /api/dashboard
**When** the request is authenticated
**Then** the response contains { stats: { totalGenerated, totalDownloads, memberSince, tier }, recentScenarios: [...] }

### AC2: Paginated Scenarios Endpoint
**Given** a logged-in user calls GET /api/dashboard/scenarios
**When** the request includes pagination params (?page=1&limit=10)
**Then** the response contains paginated scenario list with { data: [...], total, page, limit }

### AC3: Authentication Required
**Given** an unauthenticated user calls dashboard endpoints
**When** no valid session exists
**Then** a 401 response is returned

## Technical Details

### Source References
- Architecture: `docs/architecture/auth-dashboard-architecture.md`
- Epic Definition: `docs/epics-big-bang.md` (Story 4.8)

### API Endpoints
1. `GET /api/dashboard` - Summary endpoint with stats and recent scenarios
2. `GET /api/dashboard/scenarios` - Paginated scenarios list

### Files to Create
1. `app/api/dashboard/route.ts` - Dashboard summary endpoint (NEW)
2. `app/api/dashboard/scenarios/route.ts` - Paginated scenarios endpoint (NEW)

## Tasks

- [x] Create GET /api/dashboard endpoint
- [x] Create GET /api/dashboard/scenarios endpoint with pagination
- [x] Require authentication on both endpoints (401 for unauthenticated)
- [x] Include proper error handling

## Definition of Done

- [x] All acceptance criteria pass
- [x] GET /api/dashboard returns stats and recent scenarios
- [x] GET /api/dashboard/scenarios returns paginated list
- [x] Both endpoints return 401 for unauthenticated users
