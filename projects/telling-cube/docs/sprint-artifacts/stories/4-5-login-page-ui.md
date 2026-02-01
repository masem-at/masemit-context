# Story 4.5: Login Page UI

**Epic:** 4 - Auth & User Dashboard (Phase 2)
**Status:** ready-for-dev
**Created:** 2026-01-21
**Depends On:** Story 4.2 (Magic Link Request API)

## User Story

As a **returning user**,
I want **a simple login page where I can enter my email**,
So that **I can request access to my dashboard**.

## Acceptance Criteria

### AC1: Login Page Display
**Given** a user visits /login
**When** the page loads
**Then** a form is displayed with an email input field and "Send Magic Link" button
**And** the tellingCube branding is visible
**And** the page is desktop-first responsive

### AC2: Form Submission Success
**Given** a user enters their email and submits the form
**When** the form is submitted
**Then** the /api/auth/request-link endpoint is called
**And** a success message is displayed: "Check your email for a login link"
**And** the email input is disabled to prevent resubmission

### AC3: Form Validation
**Given** a user enters an invalid email
**When** the form is submitted
**Then** a validation error is displayed
**And** no API call is made

### AC4: Error Display (from redirect)
**Given** a user is redirected to /login with error params
**When** the page loads with ?error=expired_token
**Then** the error message is displayed above the form
**And** the user can request a new link

## Technical Details

### Source References
- Epic Definition: `docs/epics-big-bang.md` (Story 4.5)
- UX Spec: `docs/ux/dashboard-ux-spec.md`

### Files to Create
1. `app/login/page.tsx` - Login page component (NEW)

### Implementation Notes
- Use existing tellingCube design system (blue-600 primary)
- Show loading state during API call
- Handle rate limit error (429) with user-friendly message
- Clear error params from URL after displaying

## Tasks

- [ ] Create app/login/page.tsx with email form
- [ ] Implement form validation
- [ ] Add API call with loading state
- [ ] Handle success/error states
- [ ] Style with existing design system

## Definition of Done

- [ ] All acceptance criteria pass
- [ ] Login page renders at /login
- [ ] Form submits to API and shows success message
- [ ] Errors from redirects are displayed
- [ ] Responsive on desktop and mobile
