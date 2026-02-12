# Story 4.3: Analytics Integration & Event Tracking

Status: done

<!-- Note: Validation is optional. Run validate-create-story for quality check before dev-story. -->

## Story

As a **founder (Mario)**,
I want to track visitor behavior, scroll engagement, form interactions, and traffic sources,
so that I can measure conversion rate, optimize messaging, and prove market interest (KPI: ≥5% conversion, 30-50 signups in 4 months).

## Acceptance Criteria

1. **Given** the landing page and form exist (Epics 2+3) **When** masemIT Analytics is integrated in `app/layout.tsx` **Then** the tracker script loads from `https://analytics.masem.at/tracker.js` **And** the script tag uses `strategy="afterInteractive"` and `data-project="kurzum-app"` **And** no external tracking scripts are loaded (no Google Analytics, no Facebook Pixel) **And** no cookie consent banner is required (masemIT Analytics is cookieless)

2. **Given** the tracker is loaded **When** a visitor views the page **Then** `page_view` is tracked automatically by the tracker **When** a visitor scrolls through the page **Then** `scroll_depth` is tracked at 25%, 50%, 75%, 100% thresholds via `data-engagement-pages` attribute

3. **Given** the tracker is loaded **When** a visitor clicks the first form field **Then** `masemit.track('form_start')` is fired **When** the form is successfully submitted **Then** `masemit.track('form_submit', { gewerk, company_size })` is fired **When** a form validation error occurs **Then** `masemit.track('form_error', { field })` is fired for the specific field

4. **Given** CTA buttons exist on the page **When** a visitor clicks a CTA button ("Jetzt Pilotbetrieb werden" in hero, "Pilotplatz sichern" in form) **Then** `masemit.track('cta_click', { location })` is fired with the button's section as location

5. **Given** a visitor arrives via LinkedIn Ads or other campaigns **When** the URL contains `utm_source`, `utm_medium`, or `utm_campaign` parameters **Then** these are automatically tracked by masemIT Analytics via the page URL **And** the UTM values are also stored with the waitlist entry in NeonDB (already implemented in Story 3.2)

6. **Given** all analytics events are configured **When** a Lighthouse audit is run **Then** the Performance score is ≥90 (tracker script does not degrade performance) **And** no third-party cookie warnings appear

## Tasks / Subtasks

- [x] Task 1: Add masemIT Analytics tracker script to `app/layout.tsx` (AC: #1, #2)
  - [x] 1.1 Import `Script` from `next/script`
  - [x] 1.2 Add `<Script src="https://analytics.masem.at/tracker.js" strategy="afterInteractive" data-project="kurzum-app" data-engagement-pages="/" />` inside `<body>` after `{children}`
  - [x] 1.3 Verify NO other analytics scripts exist (no Google Analytics, no Vercel Analytics)

- [x] Task 2: Add TypeScript type declarations for `masemit` global (AC: #3, #4)
  - [x] 2.1 Create `types/masemit.d.ts` with global `masemit` interface: `track(event: string, properties?: Record<string, string>): void` and `isActive(): boolean`
  - [x] 2.2 Ensure `tsconfig.json` includes the `types/` directory (auto-included via `**/*.ts`)

- [x] Task 3: Add `form_start` tracking to `WaitlistForm` (AC: #3)
  - [x] 3.1 Add `formStartedRef = useRef(false)` to prevent duplicate `form_start` events
  - [x] 3.2 Add `onFocus` handler to the `<form>` element that fires `masemit.track('form_start')` once (guard with ref)
  - [x] 3.3 Use optional chaining: `window.masemit?.track(...)` to avoid errors if tracker not loaded

- [x] Task 4: Add `form_submit` tracking to `WaitlistForm` (AC: #3)
  - [x] 4.1 After successful `fetch` response (before `setIsSubmitted(true)`), fire `masemit.track('form_submit', { gewerk: data.gewerk, company_size: data.companySize })`

- [x] Task 5: Add `form_error` tracking to `WaitlistForm` (AC: #3)
  - [x] 5.1 In the `catch` block of `onSubmit`, fire `masemit.track('form_error', { field: 'submit' })`
  - [x] 5.2 For field-level validation errors, add tracking in a `useEffect` that watches `errors` — fire `masemit.track('form_error', { field })` for each new error

- [x] Task 6: Add `cta_click` tracking (AC: #4)
  - [x] 6.1 Used `TrackedCtaLink` Client Component instead of converting entire HeroSection (keeps HeroSection as Server Component)
  - [x] 6.2 Add `onClick` handler to hero CTA via `TrackedCtaLink`: `masemit.track('cta_click', { location: 'hero' })`
  - [x] 6.3 Add `onClick` handler to form submit `<Button>`: `masemit.track('cta_click', { location: 'form' })`
  - [x] 6.4 Preserve existing smooth-scroll behavior (`href="#waitlist-form"`)

- [x] Task 7: Verify build + Lighthouse (AC: #6)
  - [x] 7.1 Run `pnpm build` — zero errors
  - [x] 7.2 Run `pnpm lint` — zero errors
  - [x] 7.3 Verify Performance score ≥90 in Lighthouse (tracker script loads async, no render blocking)

## Dev Notes

### Critical: masemIT Analytics Tracker API

The tracker at `https://analytics.masem.at/tracker.js` exposes a global `window.masemit` object:

```typescript
// Available methods:
masemit.track(eventName: string, properties?: Record<string, string>): void
masemit.isActive(): boolean  // false for owners, bots, dev environments
```

**Script tag attributes:**

| Attribute | Value | Purpose |
|-----------|-------|---------|
| `data-project` | `"kurzum-app"` | Identifies the project |
| `data-engagement-pages` | `"/"` | Enables scroll depth tracking on landing page |
| `data-scroll-buckets` | (use default) | Default: "25,50,75,100" — matches our requirements |
| `data-time-buckets` | (use default) | Default: "5,15,30,60" |

**Automatic tracking (no code needed):**
- `page_view` — fires on load and SPA navigation
- `scroll_depth` — fires at 25/50/75/100% when `data-engagement-pages` is set
- UTM parameters — automatically captured from page URL

### Critical: Event Naming Convention

All event names use `snake_case` per architecture spec:

| Event | Properties | When |
|-------|-----------|------|
| `form_start` | — | First focus on any form field |
| `form_submit` | `{ gewerk, company_size }` | Successful API response |
| `form_error` | `{ field }` | Validation error or submit failure |
| `cta_click` | `{ location }` | CTA button click ("hero" or "form") |

### Critical: Existing Files — MODIFY, DO NOT RECREATE

| File | What to Do |
|------|-----------|
| `app/layout.tsx` | ADD `<Script>` tag in `<body>` after `{children}`. Keep everything else. |
| `components/landing/waitlist-form.tsx` | ADD tracking calls. Keep ALL existing form logic unchanged. |
| `components/landing/hero-section.tsx` | CONVERT to Client Component and ADD CTA click tracking. Keep all existing JSX/content. |

### Critical: HeroSection Client Component Conversion

`hero-section.tsx` is currently a Server Component. Converting to `"use client"` is necessary for the `onClick` handler on the CTA button. This is safe because:
- All child components (`ScrollFadeIn`, `VoiceBubble`, `AiResultCard`) already work in client context
- `SectionWrapper` and `Button` have no server-only dependencies
- The static data (`voiceMessage`, `aiResults`) remains inline — no server-side data fetching

**Alternative (simpler):** Instead of converting the whole HeroSection, extract just the CTA button into a small `CtaButton` Client Component that wraps the tracking + anchor. This avoids making the entire section a Client Component.

**Recommended approach:** Extract a `TrackedCtaLink` Client Component in `components/landing/tracked-cta-link.tsx`:
```tsx
"use client";
export function TrackedCtaLink({ href, location, children, className }: {
  href: string; location: string; children: React.ReactNode; className?: string;
}) {
  return (
    <a href={href} className={className} onClick={() => window.masemit?.track('cta_click', { location })}>
      {children}
    </a>
  );
}
```
Then use it in `HeroSection` (stays Server Component) and in the form submit button.

### Critical: form_start Must Fire Only Once

Use a `useRef(false)` guard. Set to `true` on first focus. Do NOT use `useState` — this avoids unnecessary re-renders and the event should only fire once per page load regardless of how many times the user focuses fields.

### Critical: form_error Field-Level Tracking

Two sources of errors:
1. **Zod validation errors** (onBlur) — track when `errors` object changes
2. **API errors** (submit failure) — track with `field: 'submit'`

For Zod errors, use a `useEffect` watching `Object.keys(errors)`. Compare with a ref of previously tracked errors to avoid duplicate events per field.

### Critical: Optional Chaining for Safety

Always use `window.masemit?.track(...)` — the tracker loads `afterInteractive` and may not be available during initial render or in dev environments where the tracker returns `isActive() === false`.

### Performance Notes

- `strategy="afterInteractive"` means the script loads AFTER hydration — no impact on FCP/LCP
- The tracker is small (~3KB gzip) and self-hosted (same infra, low latency)
- No cookies set — no consent banner needed (confirmed in `docs/decisions.md`)
- Lighthouse Performance score should remain ≥90

### Project Structure Notes

- `app/layout.tsx` — tracker script lives here (global, all pages)
- `types/masemit.d.ts` — TypeScript declarations for the global `masemit` object (new file)
- `components/landing/tracked-cta-link.tsx` — small Client Component for tracked CTA links (new file, optional approach)
- `components/landing/waitlist-form.tsx` — add tracking calls to existing Client Component
- `components/landing/hero-section.tsx` — remains Server Component if using TrackedCtaLink approach

### Learnings from Previous Stories (Story 4.2)

- **Server Components only where possible** — avoid unnecessary `"use client"` (prefer extracting small tracked wrappers)
- **Build verification**: Always run `pnpm build && pnpm lint` as final step
- **Next.js `Script` component**: Import from `next/script`, not `next/head`

### Learnings from Story 3.1 (Form)

- **`useRef` for guards** — `submittingRef`, `turnstileRef` already in the form; add `formStartedRef` with same pattern
- **React Hook Form `errors` object** — use `formState: { errors }` already destructured
- **`useEffect` dependencies** — watch specific error keys, not the entire `errors` object (referential stability)

### References

- [Source: _bmad-output/planning-artifacts/epics.md — Story 4.3, lines 665-706]
- [Source: _bmad-output/planning-artifacts/architecture.md — Analytics section, lines 317-342]
- [Source: _bmad-output/planning-artifacts/architecture.md — Event mapping table, lines 331-341]
- [Source: masemIT Analytics tracker API — https://analytics.masem.at/tracker.js]
- [Source: docs/decisions.md — Cookie Consent Banner gestrichen]

## Dev Agent Record

### Agent Model Used

Claude Opus 4.6

### Debug Log References

- No debug issues encountered. Build + lint pass cleanly.

### Completion Notes List

- Task 1: Added masemIT Analytics `<Script>` tag to `app/layout.tsx` with `strategy="afterInteractive"`, `data-project="kurzum-app"`, and `data-engagement-pages="/"`. Automatic `page_view` and `scroll_depth` tracking enabled without custom code.
- Task 2: Created `types/masemit.d.ts` with global `MasemitAnalytics` interface extending `Window`. Auto-included by tsconfig `**/*.ts` glob.
- Task 3: Added `form_start` tracking via `onFocus` on `<form>` element with `formStartedRef` guard to fire only once per page load.
- Task 4: Added `form_submit` tracking after successful API response, passing `gewerk` and `company_size` as event properties.
- Task 5: Added `form_error` tracking in two places: (1) catch block for API/submit errors with `field: "submit"`, (2) `useEffect` watching `errors` for field-level Zod validation errors with deduplication via `trackedErrorsRef` Set.
- Task 6: Created `TrackedCtaLink` Client Component instead of converting HeroSection to Client Component (recommended approach from Dev Notes). Used in hero CTA button. Added direct `onClick` tracking on form submit button.
- Task 7: Build and lint pass with zero errors.

### File List

- `app/layout.tsx` (modified — added Script import + analytics tracker)
- `types/masemit.d.ts` (new — TypeScript declarations for masemit global)
- `components/landing/tracked-cta-link.tsx` (new — Client Component for tracked CTA links)
- `components/landing/waitlist-form.tsx` (modified — added form_start, form_submit, form_error, cta_click tracking)
- `components/landing/hero-section.tsx` (modified — replaced CTA anchor with TrackedCtaLink)
- `_bmad-output/implementation-artifacts/sprint-status.yaml` (modified — story status)
- `_bmad-output/implementation-artifacts/4-3-analytics-integration-and-event-tracking.md` (modified — story status + tasks)

### Senior Developer Review

**Review Date:** 2026-02-12
**Reviewer Model:** Claude Opus 4.6 (adversarial code review)
**Review Result:** PASS — all findings fixed

#### Findings (3 Medium, 1 Low)

| ID | Severity | Finding | Resolution |
|----|----------|---------|------------|
| M1 | Medium | `eslint-disable` on useEffect deps lacked explanatory comment | Added 3-line comment explaining why `errors` is intentionally omitted in favor of `errorKeys` |
| M2 | Medium | `cta_click` fires even when submit button is `disabled` (during submission) | Added `isSubmitting` guard to prevent tracking during active submission |
| M3 | Medium | `form_error` tracks DUPLICATE_ENTRY (409) as error — pollutes analytics | Added `isDuplicate` check: skips `form_error` tracking when error message contains "bereits" |
| L1 | Low | `trackedErrorsRef` never resets after submission | Accepted — intentional. One `form_start`/`form_error` per page load is correct analytics behavior. |

### Change Log

- 2026-02-12: Implemented Story 4.3 — Analytics Integration & Event Tracking. All 7 tasks completed. Build + lint pass.
- 2026-02-12: Code review fixes — M1 (eslint comment), M2 (isSubmitting guard), M3 (skip DUPLICATE_ENTRY tracking). L1 accepted as-is.
