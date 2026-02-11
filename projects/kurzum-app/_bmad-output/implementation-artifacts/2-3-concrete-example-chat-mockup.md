# Story 2.3: Concrete Example Chat-Mockup

Status: done

## Story

As a **visitor (Stefan)**,
I want to see a real field worker scenario where voice input becomes structured AI output,
so that I think "Das passiert bei mir jeden Tag" and understand the product's core value.

## Acceptance Criteria

1. **Given** the section layout and ScrollFadeIn exist (Stories 2.1, 2.2) **When** the ChatMockup section is implemented **Then** it displays the headline "Ein typischer Einsatz" on a dark background (`variant="dark"`) using SectionWrapper with `id="example"`

2. **Given** the ChatMockup is rendered **When** it enters the viewport **Then** the left side shows a VoiceBubble sub-component: avatar circle (orange, "M" initial) + name "Monteur Martin" + waveform graphic (CSS bars) + "0:28" duration + transcript text in speech bubble **And** the right side shows an AiResultCard sub-component: 3 structured action items with icons (✅ Status, 📦 Material, 📅 Nächster Schritt) **And** a transformation arrow separates VoiceBubble from AiResultCard **And** the VoiceBubble fades in first (0.3s via ScrollFadeIn delay=0), then the arrow (delay=150ms), then the AiResultCard (delay=300ms) — showing the transformation sequence

3. **Given** the mockup is viewed on desktop **When** content is rendered **Then** VoiceBubble appears on the left, arrow in the middle, AiResultCard on the right (side-by-side with `flex gap-8 items-start`) **Given** it is viewed on mobile **When** content is rendered **Then** VoiceBubble appears above, arrow rotated 90°, AiResultCard below (stacked with `flex flex-col gap-4`)

4. **Given** the ChatMockup is purely visual **When** rendered **Then** no audio playback functionality exists — this is a static visual mockup with scroll animation **And** text content uses Handwerker-Sprache (informal German, no tech jargon) **And** `prefers-reduced-motion` is respected (all elements visible immediately)

## Tasks / Subtasks

- [x] Task 1: Create VoiceBubble Component (AC: #2)
  - [x] 1.1 Create `components/landing/voice-bubble.tsx` — NO "use client" (presentational, receives props)
  - [x] 1.2 Define and export `VoiceBubbleProps` interface: `{ name: string; duration: string; transcript: string }`
  - [x] 1.3 Implement avatar: `div` with `bg-accent rounded-full w-9 h-9 flex items-center justify-center text-accent-foreground text-sm font-semibold` containing "M"
  - [x] 1.4 Implement waveform: CSS-only visualization with ~20 `div` bars, `bg-accent` with varying heights (3px wide, 2-3px gap), total height 32px — NOT the WaveformIcon component (that's a 5-bar brand mark, not an audio waveform)
  - [x] 1.5 Render duration: `text-primary-foreground/50 text-xs` showing "0:28"
  - [x] 1.6 Render transcript below separator: `border-t border-primary-foreground/10 pt-3 mt-3`, text in `text-primary-foreground/50 text-sm italic`
  - [x] 1.7 Container styling: `bg-primary-foreground/5 rounded-2xl p-5 max-w-[400px]`
  - [x] 1.8 All colors via design tokens — NO hardcoded hex values, NO `text-stone-*` classes

- [x] Task 2: Create AiResultCard Component (AC: #2)
  - [x] 2.1 Create `components/landing/ai-result-card.tsx` — NO "use client" (presentational)
  - [x] 2.2 Define and export `AiResultCardProps` interface: `{ items: Array<{ emoji: string; label: string; text: string }> }`
  - [x] 2.3 Container: `bg-primary-foreground/5 rounded-2xl p-5 border border-accent max-w-[400px]`
  - [x] 2.4 Each item: emoji `aria-hidden="true"` + label in `text-accent text-xs font-semibold uppercase tracking-wider` + text in `text-primary-foreground/70 text-sm`
  - [x] 2.5 Items separated by `border-b border-primary-foreground/10 py-2.5`, last item no border
  - [x] 2.6 All colors via design tokens — match VoiceBubble palette

- [x] Task 3: Create ChatMockup Section Component (AC: #1, #2, #3, #4)
  - [x] 3.1 Create `components/landing/chat-mockup.tsx` as Server Component
  - [x] 3.2 Use SectionWrapper with `variant="dark"` and `id="example"`
  - [x] 3.3 Render headline "Ein typischer Einsatz" using `text-h2 md:text-h1 font-bold mb-8 md:mb-12 text-center`
  - [x] 3.4 Layout container: `flex flex-col md:flex-row gap-4 md:gap-8 items-center md:items-start justify-center`
  - [x] 3.5 Wrap VoiceBubble in ScrollFadeIn with `delay={0}`
  - [x] 3.6 Transformation arrow: `text-accent text-2xl` — horizontal `→` on desktop, vertical `↓` on mobile via `md:rotate-0 rotate-90` — wrapped in ScrollFadeIn with `delay={150}`
  - [x] 3.7 Wrap AiResultCard in ScrollFadeIn with `delay={300}`
  - [x] 3.8 Hardcode content data (voice text + 3 AI result items) as const arrays in component file
  - [x] 3.9 Use Monteur Martin scenario from PRD Journey 1 (see Dev Notes for exact copy)

- [x] Task 4: Integrate ChatMockup into Landing Page (AC: #1)
  - [x] 4.1 Update `app/page.tsx` to import ChatMockup
  - [x] 4.2 Add ChatMockup between ProblemSection and waitlist placeholder
  - [x] 4.3 Ensure section order: PageHeader → Hero → Problem → **ChatMockup** → (waitlist placeholder)

- [x] Task 5: Verify Build & Lint (AC: all)
  - [x] 5.1 Run `npm run build` — zero errors
  - [x] 5.2 Run `npm run lint` — zero errors
  - [x] 5.3 TypeScript produces zero type errors
  - [x] 5.4 Verify ChatMockup animation sequence works (manual visual check at different scroll positions)
  - [x] 5.5 Verify responsive layout: mobile stacked, desktop side-by-side
  - [x] 5.6 Verify prefers-reduced-motion disables all animations

## Dev Notes

### Critical: Monteur Martin Voice Content (German — from PRD Journey 1)

**VoiceBubble Transcript:**
> "Unterverteilung dritte Etage links fertig. 24 Sicherungsautomaten eingebaut, alles nach Plan. FI-Schutzschalter muss noch vom Elektriker-Meister abgenommen werden. Material reicht noch für die rechte Wohnung."

**Duration:** 0:28 (28 seconds)

**AiResultCard Items:**

| # | Emoji | Label | Text |
|---|-------|-------|------|
| 1 | ✅ | STATUS | Unterverteilung 3. OG links fertig — 24 Sicherungsautomaten eingebaut |
| 2 | 📦 | MATERIAL | Reicht noch für rechte Wohnung |
| 3 | 📅 | NÄCHSTER SCHRITT | FI-Schutzschalter: Meister-Abnahme ausstehend |

**Tone:** Informal German (duzen), Handwerker-Sprache, zero tech jargon. Stefan reads this and thinks "Das passiert bei mir jeden Tag."

### Critical: WaveformIcon vs ChatMockup Waveform — DO NOT CONFUSE

| Component | Purpose | Bars | Usage |
|-----------|---------|------|-------|
| `WaveformIcon` (exists) | Brand mark icon (logo/favicon) | 5 SVG bars, fixed, 24x24px | Logo, favicon — **DO NOT USE for ChatMockup** |
| ChatMockup waveform (NEW) | Audio recording visualization | ~20 CSS `div` bars, varying heights, 32px tall | VoiceBubble only |

The ChatMockup waveform is a DIFFERENT visual element. Build it with `div` bars and Tailwind height utilities, NOT by reusing `WaveformIcon`.

### Critical: Color Approach for Dark-on-Dark Components

The ChatMockup sits inside `SectionWrapper variant="dark"` which sets `bg-primary text-primary-foreground`. Sub-components (VoiceBubble, AiResultCard) need subtle contrast:

| Element | Color Approach | Token |
|---------|---------------|-------|
| Bubble/card background | Slightly lighter than section bg | `bg-primary-foreground/5` (white at 5% opacity) |
| Primary text | Light on dark | `text-primary-foreground` (default from SectionWrapper) |
| Secondary text | Muted light on dark | `text-primary-foreground/50` or `text-primary-foreground/70` |
| Borders/separators | Subtle | `border-primary-foreground/10` |
| Accent (labels, avatar, waveform) | Orange | `bg-accent` / `text-accent` |
| AiResultCard border | Orange accent | `border-accent` |

**DO NOT use `text-muted-foreground` on dark backgrounds** — contrast ratio is only ~3.7:1 (fails WCAG AA). Use `text-primary-foreground/70` instead (~8.4:1). (Story 2.1 review finding M1)

### Critical: Animation Specifications

| Element | Duration | Delay | Via |
|---------|----------|-------|-----|
| VoiceBubble | 400ms (ScrollFadeIn default) | 0ms | `<ScrollFadeIn delay={0}>` |
| Arrow | 400ms | 150ms | `<ScrollFadeIn delay={150}>` |
| AiResultCard | 400ms | 300ms | `<ScrollFadeIn delay={300}>` |

**Note:** UX spec says 300ms duration for ChatMockup elements, but ScrollFadeIn uses 400ms. The 100ms difference is imperceptible. Use ScrollFadeIn as-is — DO NOT modify it or create a custom animation system. Consistency > spec-perfection.

**prefers-reduced-motion:** Handled by ScrollFadeIn's `motion-safe:` Tailwind variants. No additional work needed.

### Critical: Responsive Layout

| Breakpoint | Layout | Gap | Arrow |
|------------|--------|-----|-------|
| Mobile (<768px) | `flex-col` (stacked) | `gap-4` (16px) | `↓` vertical |
| Desktop (768px+) | `flex-row` (side-by-side) | `gap-8` (32px) | `→` horizontal |

Both VoiceBubble and AiResultCard: `max-w-[400px]` on desktop, full-width on mobile.

### Critical: Section Background Pattern

From UX Design Specification (Landing Page Section Color Map):

| Section | SectionWrapper variant |
|---------|----------------------|
| Hero | `variant="dark"` |
| Problem | `variant="light"` |
| **Concrete Example** | **`variant="dark"`** |
| Solution (next story) | `variant="warm"` |

Dark-Light-**Dark**-Warm alternating rhythm.

### Previous Story Learnings (Story 2.2 + Review)

From Senior Developer Code Review (2026-02-11):

1. **DO NOT use `text-muted-foreground` on dark backgrounds** — contrast ratio fails WCAG AA. Use `text-primary-foreground/70` instead.
2. **Use design token hover colors** — `hover:bg-brand-accent-hover` not `hover:bg-accent/90`.
3. **Export component prop interfaces** — Always `export interface XxxProps`.
4. **Use SectionWrapper for ALL sections** — Including this ChatMockup section.
5. **Server Components by default** — No "use client" unless state/effects/browser APIs needed. VoiceBubble and AiResultCard are presentational → Server Components. ChatMockup section wrapper is also Server (ScrollFadeIn handles the client boundary).
6. **Test at build** — Run `npm run build` and `npm run lint` before marking complete.
7. **Add `aria-hidden="true"` to decorative emojis** — Review finding L1.
8. **Add `data-scroll-fade` attribute** on ScrollFadeIn divs (already built into component) for noscript fallback.
9. **Use tablet breakpoints where UX spec requires** — `sm:` for tablet, `lg:` for desktop (review finding M1).
10. **Architecture exception for inline `style`** — Documented for dynamic `transitionDelay`. No other inline styles.

**Existing Components to Reuse:**
- `@/lib/utils` — `cn()` utility for className merging
- `@/components/landing/section-wrapper` — SectionWrapper with variant/padding/maxWidth/id props
- `@/components/landing/scroll-fade-in` — ScrollFadeIn with delay prop for stagger animations

**DO NOT reuse:**
- `@/components/landing/waveform-icon` — This is a 5-bar brand SVG mark, NOT an audio waveform visualization

### Project Structure (After This Story)

```
components/
├── landing/
│   ├── logo.tsx                  # From Story 1.2
│   ├── waveform-icon.tsx         # From Story 1.2 (brand mark — NOT for ChatMockup)
│   ├── page-header.tsx           # From Story 1.2
│   ├── section-wrapper.tsx       # From Story 2.1
│   ├── hero-section.tsx          # From Story 2.1
│   ├── scroll-fade-in.tsx        # From Story 2.2
│   ├── problem-card.tsx          # From Story 2.2
│   ├── problem-section.tsx       # From Story 2.2
│   ├── voice-bubble.tsx          # NEW: Monteur Martin voice message display
│   ├── ai-result-card.tsx        # NEW: AI structured output card
│   └── chat-mockup.tsx           # NEW: ChatMockup section with animation sequence
app/
├── globals.css                   # Existing (no changes needed)
├── layout.tsx                    # Existing (no changes needed)
└── page.tsx                      # MODIFIED: Add ChatMockup after ProblemSection
```

### Architecture Compliance Checklist

- [x] File naming: kebab-case (`voice-bubble.tsx`, `ai-result-card.tsx`, `chat-mockup.tsx`)
- [x] Component naming: PascalCase (`VoiceBubble`, `AiResultCard`, `ChatMockup`)
- [x] Server Components: Default for VoiceBubble, AiResultCard, and ChatMockup section
- [x] Client Component: None needed — ScrollFadeIn handles client boundary
- [x] Imports: Relative within `components/landing/` (project-wide convention), absolute `@/` for cross-directory
- [x] Styling: Tailwind classes only, design tokens from CSS variables
- [x] Types: No `any`, exported interfaces for all component props
- [x] Accessibility: `aria-hidden="true"` on decorative emojis, `aria-label` on section, prefers-reduced-motion via ScrollFadeIn, WCAG AA contrast on dark backgrounds (text-primary-foreground/70)

### References

- [Source: _bmad-output/planning-artifacts/prd.md#Journey 1: Markus — "Endlich kein Abend-Papierkram"]
- [Source: _bmad-output/planning-artifacts/ux-design-specification.md#Concrete Example (Chat-Mockup)]
- [Source: _bmad-output/planning-artifacts/ux-design-specification.md#ChatMockup Component Strategy]
- [Source: _bmad-output/planning-artifacts/ux-design-specification.md#Landing Page Section Color Map]
- [Source: _bmad-output/planning-artifacts/ux-design-specification.md#Animation & Motion Specifications]
- [Source: _bmad-output/planning-artifacts/architecture.md#Frontend Architecture — Server vs Client Components]
- [Source: _bmad-output/planning-artifacts/architecture.md#Implementation Patterns & Consistency Rules]
- [Source: _bmad-output/planning-artifacts/epics.md#Story 2.3]
- [Source: _bmad-output/implementation-artifacts/2-2-problem-section-with-animated-pain-points.md#Previous Story Learnings]

## Dev Agent Record

### Agent Model Used

Claude Opus 4.6

### Debug Log References

- No errors encountered during implementation. Build and lint passed on first attempt.

### Completion Notes List

- VoiceBubble uses 20 CSS `div` bars with Tailwind height utilities (h-2 through h-8) for varying waveform visualization. Not using WaveformIcon (5-bar brand SVG).
- All three components are Server Components (no "use client"). ScrollFadeIn handles the client boundary for animations.
- AiResultCard exports both `AiResultItem` type and `AiResultCardProps` interface. ChatMockup imports `AiResultItem` for type-safe data array.
- Colors use opacity modifiers on `primary-foreground` token (e.g., `/5`, `/50`, `/70`) for dark-on-dark contrast hierarchy per Story 2.2 review learnings.
- Monteur Martin voice text taken verbatim from PRD Journey 1 (Markus scenario, 28 seconds).
- AI result items derived from voice message content: Status (UV fertig), Material (reicht für rechte Wohnung), Nächster Schritt (Meister-Abnahme).
- Transformation arrow uses Unicode `→` with `rotate-90` on mobile for vertical orientation.
- Build passes (Next.js 16.1.6 Turbopack), lint passes (ESLint), zero TypeScript errors.

### File List

- `components/landing/voice-bubble.tsx` — NEW: Server Component, Monteur Martin voice message display with CSS waveform
- `components/landing/ai-result-card.tsx` — NEW: Server Component, AI structured output card with emoji + label + text items
- `components/landing/chat-mockup.tsx` — NEW: Server Component, ChatMockup section with headline + VoiceBubble + arrow + AiResultCard + ScrollFadeIn animation sequence
- `components/landing/section-wrapper.tsx` — MODIFIED: Added optional `ariaLabel` prop for section landmark accessibility
- `app/page.tsx` — MODIFIED: Added ChatMockup import and placement between ProblemSection and waitlist placeholder

### Change Log

- 2026-02-11: Implemented Story 2.3 — Concrete Example Chat-Mockup. Created VoiceBubble, AiResultCard, ChatMockup components. Integrated into landing page between Problem section and waitlist placeholder. All Server Components, CSS-only waveform, design tokens throughout.
- 2026-02-11: Code Review fixes — (M1) VoiceBubble contrast upgraded from /50 to /70 for WCAG AA compliance consistency. (M2) Architecture Compliance Checklist verified and checked off. (M3) Added aria-label to SectionWrapper for section landmark accessibility; applied in ChatMockup.
