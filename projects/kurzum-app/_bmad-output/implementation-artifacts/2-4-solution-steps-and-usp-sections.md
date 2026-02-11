# Story 2.4: Solution Steps & USP Sections

Status: done

## Story

As a **visitor (Stefan)**,
I want to understand how kurzum works in 3 simple steps and why it's the right choice,
so that I move from "interesting" to "I want this."

## Acceptance Criteria

1. **Given** the section layout and ScrollFadeIn exist (Stories 2.1, 2.2) **When** the Solution section is implemented **Then** it displays the headline "So einfach funktioniert kurzum" on a warm background (`variant="warm"`) using SectionWrapper with `id="solution"` **And** 3 SolutionStep components are rendered with number + headline + text:
   - Step 1: "Sprechen" — "Monteur spricht 30 Sekunden ins Handy. Fertig."
   - Step 2: "KI verarbeitet" — "Zusammenfassung, Projekt-Zuordnung, Aktionspunkte. Automatisch."
   - Step 3: "Überblick" — "Chef und Büro sehen alles im Dashboard. Kein Anruf nötig."
   **And** steps fade in sequentially via ScrollFadeIn

2. **Given** steps are viewed on mobile **When** rendered **Then** steps stack vertically with consistent spacing **Given** steps are viewed on desktop **When** rendered **Then** steps display in a 3-column grid layout

3. **Given** the Solution section exists **When** the USP section is implemented **Then** it displays the headline "Warum kurzum?" on a light background (`variant="light"`) using SectionWrapper with `id="usps"` **And** 5 UspItem components are rendered with emoji + headline + one-liner:
   - 🎙️ Voice-First — "Für dreckige Hände gemacht. Sprechen statt tippen."
   - 🔒 DSGVO by Design — "EU-Server, kein Telefonbuchzugriff. Nicht aufgeklebt, eingebaut."
   - 💰 KMU-Preis — "Ab €10/User/Monat. Weniger als ein Strafzettel."
   - 🇦🇹 Made in Austria — "Kein Silicon-Valley-Startup. Einer der versteht, wie KMU ticken."
   - 🤝 Ergänzt statt ersetzt — "kurzum ersetzt nicht dein ERP – es macht Kommunikation intelligent."
   **And** USPs fade in sequentially via ScrollFadeIn

4. **Given** Solution and USP sections are purely presentational **When** rendered **Then** all text uses Handwerker-Sprache (informal German, no tech jargon) **And** `prefers-reduced-motion` is respected (all elements visible immediately) **And** all colors use design tokens — NO hardcoded hex values

## Tasks / Subtasks

- [x] Task 1: Create SolutionStep Component (AC: #1, #2)
  - [x] 1.1 Create `components/landing/solution-step.tsx` — NO "use client" (presentational Server Component)
  - [x] 1.2 Define and export `SolutionStepProps` interface: `{ number: number; title: string; description: string }`
  - [x] 1.3 Render step number: `text-accent text-h1 font-bold` — large orange number as visual anchor
  - [x] 1.4 Render title: `text-body-lg font-semibold text-foreground mt-2` as `<h3>`
  - [x] 1.5 Render description: `text-body-sm text-muted-foreground mt-1 leading-relaxed`
  - [x] 1.6 All colors via design tokens — NO `text-stone-*` or `text-orange-*` classes, use `text-accent`, `text-foreground`, `text-muted-foreground`

- [x] Task 2: Create UspItem Component (AC: #3)
  - [x] 2.1 Create `components/landing/usp-item.tsx` — NO "use client" (presentational Server Component)
  - [x] 2.2 Define and export `UspItemProps` interface: `{ emoji: string; title: string; description: string }`
  - [x] 2.3 Layout: `flex items-start gap-3` — emoji left, text right
  - [x] 2.4 Emoji: `text-2xl shrink-0` with `aria-hidden="true"` (decorative)
  - [x] 2.5 Title: `text-base font-semibold text-foreground` as `<h3>`
  - [x] 2.6 Description: `text-sm text-muted-foreground` on same line or below title
  - [x] 2.7 All colors via design tokens

- [x] Task 3: Create SolutionSection Component (AC: #1, #2)
  - [x] 3.1 Create `components/landing/solution-section.tsx` as Server Component
  - [x] 3.2 Use SectionWrapper with `variant="warm"`, `id="solution"`, `ariaLabel="So einfach funktioniert kurzum"`
  - [x] 3.3 Render headline "So einfach funktioniert kurzum" using `text-h2 md:text-h1 font-bold mb-8 md:mb-12 text-center`
  - [x] 3.4 Create responsive grid: `grid grid-cols-1 md:grid-cols-3 gap-4 md:gap-6 lg:gap-8`
  - [x] 3.5 Wrap each SolutionStep in ScrollFadeIn with stagger delays: 0ms, 100ms, 200ms
  - [x] 3.6 Hardcode step data as `const` array in component file (3 items from AC #1)

- [x] Task 4: Create UspSection Component (AC: #3)
  - [x] 4.1 Create `components/landing/usp-section.tsx` as Server Component
  - [x] 4.2 Use SectionWrapper with `variant="light"`, `id="usps"`, `ariaLabel="Warum kurzum?"`
  - [x] 4.3 Render headline "Warum kurzum?" using `text-h2 md:text-h1 font-bold mb-8 md:mb-12 text-center`
  - [x] 4.4 USP list container: `space-y-4 max-w-2xl mx-auto` (centered, constrained width)
  - [x] 4.5 Wrap each UspItem in ScrollFadeIn with stagger delays: 0ms, 80ms, 160ms, 240ms, 320ms
  - [x] 4.6 Hardcode USP data as `const` array in component file (5 items from AC #3)

- [x] Task 5: Integrate into Landing Page (AC: #1, #3)
  - [x] 5.1 Update `app/page.tsx` to import SolutionSection and UspSection
  - [x] 5.2 Add SolutionSection after ChatMockup
  - [x] 5.3 Add UspSection after SolutionSection
  - [x] 5.4 Remove or move the waitlist placeholder below UspSection
  - [x] 5.5 Ensure section order: PageHeader → Hero → Problem → ChatMockup → **SolutionSection** → **UspSection** → (waitlist placeholder)

- [x] Task 6: Verify Build & Lint (AC: all)
  - [x] 6.1 Run `npm run build` — zero errors
  - [x] 6.2 Run `npm run lint` — zero errors
  - [x] 6.3 TypeScript produces zero type errors
  - [x] 6.4 Verify ScrollFadeIn animation sequence works (manual visual check at different scroll positions)
  - [x] 6.5 Verify responsive layout: mobile stacked, desktop 3-column for steps
  - [x] 6.6 Verify prefers-reduced-motion disables all animations

## Dev Notes

### Critical: Solution Step Content (German — from Epics)

| # | Number Display | Title | Description |
|---|------|-------|-------------|
| 1 | 1 | Sprechen | Monteur spricht 30 Sekunden ins Handy. Fertig. |
| 2 | 2 | KI verarbeitet | Zusammenfassung, Projekt-Zuordnung, Aktionspunkte. Automatisch. |
| 3 | 3 | Überblick | Chef und Büro sehen alles im Dashboard. Kein Anruf nötig. |

### Critical: USP Content (German — from Epics)

| # | Emoji | Title | Description |
|---|-------|-------|-------------|
| 1 | 🎙️ | Voice-First | Für dreckige Hände gemacht. Sprechen statt tippen. |
| 2 | 🔒 | DSGVO by Design | EU-Server, kein Telefonbuchzugriff. Nicht aufgeklebt, eingebaut. |
| 3 | 💰 | KMU-Preis | Ab €10/User/Monat. Weniger als ein Strafzettel. |
| 4 | 🇦🇹 | Made in Austria | Kein Silicon-Valley-Startup. Einer der versteht, wie KMU ticken. |
| 5 | 🤝 | Ergänzt statt ersetzt | kurzum ersetzt nicht dein ERP – es macht Kommunikation intelligent. |

### Critical: Section Background Pattern

From UX Design Specification (Landing Page Section Color Map):

| Section | SectionWrapper variant | Background |
|---------|----------------------|------------|
| Hero | `variant="dark"` | bg-primary text-primary-foreground |
| Problem | `variant="light"` | bg-background text-foreground |
| Concrete Example | `variant="dark"` | bg-primary text-primary-foreground |
| **Solution** | **`variant="warm"`** | **bg-muted text-foreground** |
| **USPs** | **`variant="light"`** | **bg-background text-foreground** |
| Pilot (next story) | `variant="dark"` | bg-primary text-primary-foreground |

Pattern: Dark-Light-Dark-**Warm**-**Light**-Dark alternating rhythm.

### Critical: Color Approach for Light-Background Components

Both sections are on LIGHT backgrounds (warm/light variants). Color tokens:

| Element | Token | Usage |
|---------|-------|-------|
| Primary text (headings, titles) | `text-foreground` | Default from SectionWrapper variant |
| Secondary text (descriptions) | `text-muted-foreground` | OK on light backgrounds (~5.6:1 contrast) |
| Accent (step numbers, emphasis) | `text-accent` | Orange accent color |
| Borders/separators | `border-border` | Default border token |

**`text-muted-foreground` is SAFE on light backgrounds** (bg-background, bg-muted). It only fails on dark backgrounds (bg-primary). See Story 2.2/2.3 review findings.

### Critical: Animation Specifications

| Element | Duration | Delay | Via |
|---------|----------|-------|-----|
| SolutionStep 1 | 400ms (ScrollFadeIn default) | 0ms | `<ScrollFadeIn delay={0}>` |
| SolutionStep 2 | 400ms | 100ms | `<ScrollFadeIn delay={100}>` |
| SolutionStep 3 | 400ms | 200ms | `<ScrollFadeIn delay={200}>` |
| UspItem 1-5 | 400ms each | 0, 80, 160, 240, 320ms | `<ScrollFadeIn delay={index * 80}>` |

**prefers-reduced-motion:** Handled by ScrollFadeIn's `motion-safe:` Tailwind variants. No additional work needed.

### Critical: Responsive Layout

**Solution Steps Grid:**

| Breakpoint | Grid | Gap |
|------------|------|-----|
| Mobile (<768px) | `grid-cols-1` | `gap-4` (16px) |
| Desktop (768px+) | `md:grid-cols-3` | `md:gap-6 lg:gap-8` (24/32px) |

**USP List:**

| Breakpoint | Layout | Spacing |
|------------|--------|---------|
| All breakpoints | Vertical stack | `space-y-4` (16px) |
| Container | `max-w-2xl mx-auto` | Centered, ~672px max |

### Previous Story Learnings (Story 2.2 + 2.3 + Reviews)

From Senior Developer Code Reviews (2026-02-11):

1. **DO NOT use `text-muted-foreground` on dark backgrounds** — contrast ratio fails WCAG AA. Use `text-primary-foreground/70` instead. (Only relevant if you accidentally place text on dark bg)
2. **Use design token hover colors** — `hover:bg-brand-accent-hover` not `hover:bg-accent/90`.
3. **Export component prop interfaces** — Always `export interface XxxProps`.
4. **Use SectionWrapper for ALL sections** — With appropriate variant.
5. **Server Components by default** — No "use client" unless state/effects/browser APIs needed. SolutionStep, UspItem, SolutionSection, UspSection are ALL Server Components.
6. **Test at build** — Run `npm run build` and `npm run lint` before marking complete.
7. **Add `aria-hidden="true"` to decorative emojis** — Required for USP emoji icons.
8. **Use `ariaLabel` prop on SectionWrapper** — Added in Story 2.3 review for landmark accessibility.
9. **Relative imports within `components/landing/`** — Project convention for sibling imports (e.g., `import { SectionWrapper } from "./section-wrapper"`). Use `@/` only for cross-directory imports.
10. **Hardcode content data as `const` arrays** — In the section component file, not in separate data files.

**Existing Components to Reuse:**
- `./section-wrapper` — SectionWrapper with variant/padding/maxWidth/id/ariaLabel props
- `./scroll-fade-in` — ScrollFadeIn with delay/className props for stagger animations
- `@/lib/utils` — `cn()` utility for className merging (if needed)

### Critical: Typography Scale (from globals.css)

Use these custom Tailwind utilities (defined in globals.css `@theme`):

| Token | Size | Weight | Usage |
|-------|------|--------|-------|
| `text-h1` | 2.25rem (36px) | 700 | Section heading (desktop) |
| `text-h2` | 1.75rem (28px) | 600 | Section heading (mobile) |
| `text-body-lg` | 1.125rem (18px) | — | Step titles |
| `text-body-sm` | 0.875rem (14px) | — | Descriptions |
| `text-body` | 1rem (16px) | — | USP titles (via `text-base`) |

Section headings use: `text-h2 md:text-h1 font-bold mb-8 md:mb-12 text-center`

### Project Structure (After This Story)

```
components/
├── landing/
│   ├── logo.tsx                  # From Story 1.2
│   ├── waveform-icon.tsx         # From Story 1.2
│   ├── page-header.tsx           # From Story 1.2
│   ├── section-wrapper.tsx       # From Story 2.1 (ariaLabel added in 2.3)
│   ├── hero-section.tsx          # From Story 2.1
│   ├── scroll-fade-in.tsx        # From Story 2.2
│   ├── problem-card.tsx          # From Story 2.2
│   ├── problem-section.tsx       # From Story 2.2
│   ├── voice-bubble.tsx          # From Story 2.3
│   ├── ai-result-card.tsx        # From Story 2.3
│   ├── chat-mockup.tsx           # From Story 2.3
│   ├── solution-step.tsx         # NEW: Step number + title + description
│   ├── usp-item.tsx              # NEW: Emoji + title + description
│   ├── solution-section.tsx      # NEW: "So einfach funktioniert kurzum" section
│   └── usp-section.tsx           # NEW: "Warum kurzum?" section
app/
├── globals.css                   # Existing (no changes needed)
├── layout.tsx                    # Existing (no changes needed)
└── page.tsx                      # MODIFIED: Add SolutionSection + UspSection
```

### Architecture Compliance Checklist

- [x] File naming: kebab-case (`solution-step.tsx`, `usp-item.tsx`, `solution-section.tsx`, `usp-section.tsx`)
- [x] Component naming: PascalCase (`SolutionStep`, `UspItem`, `SolutionSection`, `UspSection`)
- [x] Server Components: Default for ALL four new components
- [x] Client Component: None needed — ScrollFadeIn handles client boundary
- [x] Imports: Relative within `components/landing/` (project convention), absolute `@/` for cross-directory
- [x] Styling: Tailwind classes only, design tokens from CSS variables
- [x] Types: No `any`, exported interfaces for all component props
- [x] Accessibility: `aria-hidden="true"` on decorative emojis, `ariaLabel` on SectionWrapper, prefers-reduced-motion via ScrollFadeIn, WCAG AA contrast on all backgrounds

### References

- [Source: _bmad-output/planning-artifacts/epics.md#Story 2.4]
- [Source: _bmad-output/planning-artifacts/ux-design-specification.md#Solution Steps Section]
- [Source: _bmad-output/planning-artifacts/ux-design-specification.md#USP Section]
- [Source: _bmad-output/planning-artifacts/ux-design-specification.md#Landing Page Section Color Map]
- [Source: _bmad-output/planning-artifacts/ux-design-specification.md#Animation & Motion Specifications]
- [Source: _bmad-output/planning-artifacts/architecture.md#Frontend Architecture — Server vs Client Components]
- [Source: _bmad-output/planning-artifacts/architecture.md#Implementation Patterns & Consistency Rules]
- [Source: _bmad-output/implementation-artifacts/2-2-problem-section-with-animated-pain-points.md#Previous Story Learnings]
- [Source: _bmad-output/implementation-artifacts/2-3-concrete-example-chat-mockup.md#Previous Story Learnings]

## Dev Agent Record

### Agent Model Used

Claude Opus 4.6

### Debug Log References

None — zero errors during implementation.

### Completion Notes List

- All 4 new components created as Server Components (no "use client")
- All content hardcoded as const arrays per project convention
- SolutionSection uses warm variant, UspSection uses light variant per Section Color Map
- ScrollFadeIn stagger delays applied: 100ms for steps, 80ms for USPs
- ariaLabel added to both SectionWrappers for landmark accessibility
- aria-hidden="true" on decorative emojis in UspItem
- Responsive grid (1-col → 3-col) for Solution, vertical stack for USPs
- Build passed: "Compiled successfully", 6/6 static pages generated, zero errors
- Lint passed: clean run, zero warnings
- Waitlist placeholder preserved below UspSection for Story 3.1

### File List

- `components/landing/solution-step.tsx` — NEW: Presentational component for numbered solution steps
- `components/landing/usp-item.tsx` — NEW: Presentational component for USP items with emoji
- `components/landing/solution-section.tsx` — NEW: Solution section with 3-step grid
- `components/landing/usp-section.tsx` — NEW: USP section with 5 items
- `app/page.tsx` — MODIFIED: Added SolutionSection and UspSection imports and placement

### Change Log

- 2026-02-11: Initial implementation of all 6 tasks — 4 new components, page integration, build/lint verified
- 2026-02-11: Code Review fixes — M1: UspItem typography tokens aligned to custom design tokens (text-body, text-body-sm), M2: Semantic list markup (ol/ul) for Solution and USP sections, L1: Removed redundant font-bold from step number, L3: Added leading-relaxed to UspItem description
