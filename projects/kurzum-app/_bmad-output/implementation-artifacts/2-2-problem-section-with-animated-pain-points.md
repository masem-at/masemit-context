# Story 2.2: Problem Section with Animated Pain Points

Status: done

## Story

As a **visitor (Stefan)**,
I want to see my daily frustrations named ("Kommt dir das bekannt vor?"),
so that I feel validated and think "Die kennen mein Problem."

## Acceptance Criteria

1. **Given** the SectionWrapper exists (Story 2.1) **When** a ScrollFadeIn Client Component is implemented **Then** it uses IntersectionObserver to detect when elements enter the viewport **And** elements animate from opacity 0 + translateY 20px to opacity 1 + translateY 0 **And** animation duration is 0.4s with ease-out timing **And** stagger delay between sequential elements is 100-150ms **And** `prefers-reduced-motion` is respected — animations disabled when user prefers it **And** all animations use CSS only — no JavaScript animation libraries

2. **Given** the ScrollFadeIn component exists **When** the Problem section is implemented as a Server Component (with ScrollFadeIn wrapper) **Then** it displays the headline "Kommt dir das bekannt vor?" **And** 3 ProblemCard components are rendered with icon + headline + text:
   - "WhatsApp-Chaos" — "30 Nachrichten, 5 Gruppen, nichts zugeordnet..."
   - "Abend-Papierkram" — "Nach 10 Stunden Baustelle noch Stundenzettel ausfüllen..."
   - "Der Chef als Flaschenhals" — "Alles läuft über dich..."
   **And** cards use border-left accent in --color-accent (Orange) **And** cards fade in sequentially via ScrollFadeIn as user scrolls into view

3. **Given** the section is viewed on mobile **When** cards are rendered **Then** they stack vertically with consistent spacing
   **Given** the section is viewed on desktop **When** cards are rendered **Then** they display in a 3-column grid

## Tasks / Subtasks

- [x] Task 1: Create ScrollFadeIn Client Component (AC: #1)
  - [x] 1.1 Create `components/landing/scroll-fade-in.tsx` as Client Component ("use client")
  - [x] 1.2 Implement IntersectionObserver with threshold 0.5 (50% visible trigger)
  - [x] 1.3 Use a single `useRef` + `useState` for visibility tracking
  - [x] 1.4 Initial state: `opacity-0 translate-y-5` (opacity 0, translateY 20px)
  - [x] 1.5 Visible state: `opacity-100 translate-y-0 transition-all duration-[400ms] ease-out`
  - [x] 1.6 Implement `delay` prop (number in ms) for stagger: apply via `style={{ transitionDelay: '${delay}ms' }}`
  - [x] 1.7 Respect `prefers-reduced-motion`: if reduced motion preferred, render children immediately visible (no animation)
  - [x] 1.8 Export `ScrollFadeInProps` interface
  - [x] 1.9 Content MUST be in DOM and accessible regardless of animation state (use CSS visibility, not conditional rendering)

- [x] Task 2: Create ProblemCard Server Component (AC: #2)
  - [x] 2.1 Create `components/landing/problem-card.tsx` as Server Component (NO "use client")
  - [x] 2.2 Define and export `ProblemCardProps` interface: `{ icon: string; title: string; description: string }`
  - [x] 2.3 Implement card container: `border border-border rounded-lg p-6 border-l-[3px] border-l-accent`
  - [x] 2.4 Render icon (emoji string) at top: `text-xl mb-3`
  - [x] 2.5 Render title: `text-base font-semibold text-foreground`
  - [x] 2.6 Render description: `text-sm text-muted-foreground leading-relaxed`
  - [x] 2.7 Use design tokens (Tailwind semantic classes) — no hardcoded hex values

- [x] Task 3: Create ProblemSection Server Component (AC: #2, #3)
  - [x] 3.1 Create `components/landing/problem-section.tsx` as Server Component
  - [x] 3.2 Use SectionWrapper with `variant="light"` and `id="problem"`
  - [x] 3.3 Render headline "Kommt dir das bekannt vor?" using `text-h2 md:text-h1 font-bold mb-8 md:mb-12 text-center`
  - [x] 3.4 Create responsive grid: `grid grid-cols-1 md:grid-cols-3 gap-4 md:gap-6`
  - [x] 3.5 Wrap each ProblemCard in ScrollFadeIn with stagger delays: 0ms, 100ms, 200ms
  - [x] 3.6 Define pain point data array with 3 items:
    - `{ icon: "📱", title: "WhatsApp-Chaos", description: "30 Nachrichten, 5 Gruppen, nichts zugeordnet. Wichtige Infos gehen unter – und DSGVO-konform ist es auch nicht." }`
    - `{ icon: "📝", title: "Abend-Papierkram", description: "Nach 10 Stunden Baustelle noch Stundenzettel ausfüllen. Rapporte schreiben. E-Mails beantworten. Kein Wunder, dass keiner Bock hat." }`
    - `{ icon: "🔄", title: "Der Chef als Flaschenhals", description: "Alles läuft über dich. Jede Info, jede Rückfrage, jede Entscheidung. Und am Ende weißt du trotzdem nicht, was auf der Baustelle los war." }`

- [x] Task 4: Integrate Problem Section into Landing Page (AC: #2)
  - [x] 4.1 Update `app/page.tsx` to import ProblemSection
  - [x] 4.2 Add ProblemSection between HeroSection and waitlist placeholder
  - [x] 4.3 Ensure section order: PageHeader → Hero → Problem → (waitlist placeholder)

- [x] Task 5: Verify Build & Lint (AC: all)
  - [x] 5.1 Run `npm run build` — zero errors
  - [x] 5.2 Run `npm run lint` — zero errors
  - [x] 5.3 TypeScript produces zero type errors
  - [x] 5.4 Verify ScrollFadeIn animation works (manual visual check at different scroll positions)
  - [x] 5.5 Verify prefers-reduced-motion disables animations

## Dev Notes

### Critical: Pain Point Copy (German)

From PRD and Epics (LP-FR3):

| # | Icon | Title | Description |
|---|------|-------|-------------|
| 1 | 📱 | WhatsApp-Chaos | 30 Nachrichten, 5 Gruppen, nichts zugeordnet. Wichtige Infos gehen unter – und DSGVO-konform ist es auch nicht. |
| 2 | 📝 | Abend-Papierkram | Nach 10 Stunden Baustelle noch Stundenzettel ausfüllen. Rapporte schreiben. E-Mails beantworten. Kein Wunder, dass keiner Bock hat. |
| 3 | 🔄 | Der Chef als Flaschenhals | Alles läuft über dich. Jede Info, jede Rückfrage, jede Entscheidung. Und am Ende weißt du trotzdem nicht, was auf der Baustelle los war. |

**Tone:** Informal German (duzen), Handwerker-Sprache, zero tech jargon. Stefan should feel "Die kennen mein Problem."

### Critical: ScrollFadeIn Implementation Pattern

```tsx
// components/landing/scroll-fade-in.tsx
"use client";

import { useEffect, useRef, useState } from "react";
import { cn } from "@/lib/utils";

export interface ScrollFadeInProps {
  children: React.ReactNode;
  delay?: number;
  className?: string;
}

export function ScrollFadeIn({ children, delay = 0, className }: ScrollFadeInProps) {
  const ref = useRef<HTMLDivElement>(null);
  const [isVisible, setIsVisible] = useState(false);

  useEffect(() => {
    const el = ref.current;
    if (!el) return;

    // Check prefers-reduced-motion
    const prefersReducedMotion = window.matchMedia("(prefers-reduced-motion: reduce)").matches;
    if (prefersReducedMotion) {
      setIsVisible(true);
      return;
    }

    const observer = new IntersectionObserver(
      ([entry]) => {
        if (entry.isIntersecting) {
          setIsVisible(true);
          observer.unobserve(el);
        }
      },
      { threshold: 0.5 }
    );

    observer.observe(el);
    return () => observer.disconnect();
  }, []);

  return (
    <div
      ref={ref}
      className={cn(
        "transition-all duration-[400ms] ease-out",
        isVisible ? "opacity-100 translate-y-0" : "opacity-0 translate-y-5",
        className
      )}
      style={{ transitionDelay: isVisible ? `${delay}ms` : "0ms" }}
    >
      {children}
    </div>
  );
}
```

### Critical: ProblemCard Implementation Pattern

```tsx
// components/landing/problem-card.tsx
export interface ProblemCardProps {
  icon: string;
  title: string;
  description: string;
}

export function ProblemCard({ icon, title, description }: ProblemCardProps) {
  return (
    <div className="border border-border rounded-lg p-6 border-l-[3px] border-l-accent">
      <span className="text-xl mb-3 block">{icon}</span>
      <h3 className="text-base font-semibold text-foreground mb-2">{title}</h3>
      <p className="text-sm text-muted-foreground leading-relaxed">{description}</p>
    </div>
  );
}
```

### Critical: ProblemSection Implementation Pattern

```tsx
// components/landing/problem-section.tsx
import { SectionWrapper } from "./section-wrapper";
import { ProblemCard } from "./problem-card";
import { ScrollFadeIn } from "./scroll-fade-in";

const painPoints = [
  {
    icon: "📱",
    title: "WhatsApp-Chaos",
    description: "30 Nachrichten, 5 Gruppen, nichts zugeordnet. Wichtige Infos gehen unter – und DSGVO-konform ist es auch nicht.",
  },
  {
    icon: "📝",
    title: "Abend-Papierkram",
    description: "Nach 10 Stunden Baustelle noch Stundenzettel ausfüllen. Rapporte schreiben. E-Mails beantworten. Kein Wunder, dass keiner Bock hat.",
  },
  {
    icon: "🔄",
    title: "Der Chef als Flaschenhals",
    description: "Alles läuft über dich. Jede Info, jede Rückfrage, jede Entscheidung. Und am Ende weißt du trotzdem nicht, was auf der Baustelle los war.",
  },
];

export function ProblemSection() {
  return (
    <SectionWrapper variant="light" id="problem">
      <h2 className="text-h2 md:text-h1 font-bold mb-8 md:mb-12 text-center">
        Kommt dir das bekannt vor?
      </h2>
      <div className="grid grid-cols-1 md:grid-cols-3 gap-4 md:gap-6">
        {painPoints.map((point, index) => (
          <ScrollFadeIn key={point.title} delay={index * 100}>
            <ProblemCard {...point} />
          </ScrollFadeIn>
        ))}
      </div>
    </SectionWrapper>
  );
}
```

### Critical: Section Background Pattern

From UX Design Specification (Landing Page Section Color Map):

| Section | Background Token | SectionWrapper variant |
|---------|------------------|-----------------------|
| Hero | `--color-primary` (dark) | `variant="dark"` |
| **Problem** | **`--color-bg-light`** | **`variant="light"`** |
| Example | `--color-primary` (dark) | `variant="dark"` |
| Solution | `--color-bg-warm` | `variant="warm"` |
| USPs | `--color-bg-light` | `variant="light"` |
| Pilot | `--color-primary` (dark) | `variant="dark"` |
| Form | `--color-bg-light` | `variant="light"` |
| Footer | `--color-primary` (dark) | `variant="dark"` |

**Pattern:** Dark-Light-Dark-Warm-Light-Dark-Light-Dark alternating rhythm creates visual "rooms" as you scroll.

### Critical: Animation Specifications

| Property | Value |
|----------|-------|
| Initial opacity | 0 |
| Initial translateY | 20px (`translate-y-5`) |
| Final opacity | 1 |
| Final translateY | 0 |
| Duration | 400ms |
| Easing | ease-out |
| Stagger delay | 100ms between cards |
| Trigger | IntersectionObserver at 50% visible |
| Reduced motion | Skip animation, show immediately |
| Animation type | CSS transitions only — NO JS animation libraries |

### Critical: Responsive Grid

| Breakpoint | Grid | Gap | Card Layout |
|------------|------|-----|-------------|
| Mobile (<768px) | `grid-cols-1` | `gap-4` (16px) | Full-width stack |
| Desktop (768px+) | `grid-cols-3` | `gap-6` (24px) | 3 equal columns |

### Previous Story Learnings (Story 2.1 + Review)

From Senior Developer Code Review (2026-02-11):

1. **DO NOT use `text-muted-foreground` on dark backgrounds** — contrast ratio is only ~3.7:1 (fails WCAG AA). Use `text-primary-foreground/70` instead (~8.4:1). This was a code review finding (M1).
2. **Use design token hover colors** — Use `hover:bg-brand-accent-hover` (#EA580C) not `hover:bg-accent/90`. Code review finding (H1).
3. **Export component prop interfaces** — Always `export interface XxxProps`. Code review finding (L1).
4. **Use SectionWrapper for ALL sections** — Including placeholders. Code review finding (M3).
5. **Server Components by default** — No "use client" unless state/effects/browser APIs needed. ScrollFadeIn NEEDS "use client" (IntersectionObserver). ProblemCard and ProblemSection do NOT.
6. **Test at build** — Run `npm run build` and `npm run lint` before marking complete.

**Existing Components to Reuse:**
- `@/lib/utils` — cn() utility for className merging
- `@/components/ui/button` — shadcn Button (not needed here)
- `@/components/landing/section-wrapper` — SectionWrapper with variant/padding/maxWidth/id props
- `@/components/landing/page-header` — Fixed header (already integrated)

### Project Structure (After This Story)

```
components/
├── landing/
│   ├── logo.tsx                  # From Story 1.2
│   ├── waveform-icon.tsx         # From Story 1.2
│   ├── page-header.tsx           # From Story 1.2
│   ├── section-wrapper.tsx       # From Story 2.1
│   ├── hero-section.tsx          # From Story 2.1
│   ├── scroll-fade-in.tsx        # NEW: CSS scroll animation wrapper (Client Component)
│   ├── problem-card.tsx          # NEW: Pain point card with border-left accent
│   └── problem-section.tsx       # NEW: Problem section with 3 cards + animation
app/
├── globals.css                   # Existing (no changes needed)
└── page.tsx                      # UPDATED: Add ProblemSection after HeroSection
```

### Architecture Compliance Checklist

- [x] File naming: kebab-case (`scroll-fade-in.tsx`, `problem-card.tsx`, `problem-section.tsx`)
- [x] Component naming: PascalCase (`ScrollFadeIn`, `ProblemCard`, `ProblemSection`)
- [x] Server Components: Default for ProblemCard and ProblemSection
- [x] Client Component: Only ScrollFadeIn (needs IntersectionObserver)
- [x] Imports: Absolute via `@/` alias
- [x] Styling: Tailwind classes only, design tokens from CSS variables (exception: inline `style` for dynamic transitionDelay — documented)
- [x] Types: No `any`, exported interfaces for all component props
- [x] Accessibility: prefers-reduced-motion respected, content always in DOM, noscript fallback added

### References

- [Source: _bmad-output/planning-artifacts/ux-design-specification.md#Experience Mechanics]
- [Source: _bmad-output/planning-artifacts/ux-design-specification.md#Animation & Motion Specifications]
- [Source: _bmad-output/planning-artifacts/ux-design-specification.md#Component Strategy]
- [Source: _bmad-output/planning-artifacts/ux-design-specification.md#Landing Page Section Color Map]
- [Source: _bmad-output/planning-artifacts/ux-design-specification.md#Spacing & Layout Foundation]
- [Source: _bmad-output/planning-artifacts/architecture.md#Implementation Patterns & Consistency Rules]
- [Source: _bmad-output/planning-artifacts/architecture.md#Frontend Architecture]
- [Source: _bmad-output/planning-artifacts/epics.md#Story 2.2]
- [Source: _bmad-output/implementation-artifacts/2-1-section-layout-system-and-hero-section.md#Senior Developer Review]

## Dev Agent Record

### Agent Model Used

Claude Opus 4.6

### Debug Log References

- Initial lint failure: ESLint `react-hooks/set-state-in-effect` flagged synchronous `setIsVisible(true)` in useEffect for prefers-reduced-motion check. Resolved by using Tailwind `motion-safe:` variants for CSS-only reduced-motion handling instead of JS state.

### Completion Notes List

- ScrollFadeIn uses `motion-safe:` Tailwind variants instead of JS setState for prefers-reduced-motion. When reduced motion is preferred, `motion-safe:opacity-0` and `motion-safe:translate-y-5` don't apply, so content is immediately visible without animation. Observer setup is skipped entirely.
- All three components follow story patterns exactly: ScrollFadeIn (Client), ProblemCard (Server), ProblemSection (Server).
- Pain point copy matches PRD/Epics exactly (German, informal du-form).
- Responsive grid: single column on mobile, 3 columns on md+ breakpoint.
- Stagger delays: 0ms, 100ms, 200ms via `delay` prop on ScrollFadeIn.
- All design tokens used (no hardcoded hex values). Border-left accent uses `border-l-accent`.
- Build passes (Next.js 16.1.6 Turbopack), lint passes (ESLint), zero TypeScript errors.

### File List

- `components/landing/scroll-fade-in.tsx` — NEW: Client Component, IntersectionObserver + CSS transition scroll animation wrapper
- `components/landing/problem-card.tsx` — NEW: Server Component, pain point card with orange border-left accent
- `components/landing/problem-section.tsx` — NEW: Server Component, Problem section with headline + 3-card grid + scroll animations
- `app/page.tsx` — MODIFIED: Added ProblemSection import and placement between HeroSection and waitlist placeholder
- `app/layout.tsx` — MODIFIED: Added noscript fallback for scroll animation visibility without JS (review fix)

### Change Log

- 2026-02-11: Implemented Story 2.2 — Problem Section with Animated Pain Points. Created ScrollFadeIn, ProblemCard, ProblemSection components. Integrated into landing page. Fixed ESLint react-hooks/set-state-in-effect by using motion-safe: CSS variants for reduced-motion handling.
- 2026-02-11: Senior Developer Code Review (AI) — 6 findings (2H, 2M, 2L), all fixed:
  - H1: Added noscript fallback (data-scroll-fade + layout.tsx) for no-JS content visibility
  - H2: Documented inline style exception for dynamic transitionDelay in ScrollFadeIn
  - M1: Added tablet 2-column breakpoint (sm:grid-cols-2 lg:grid-cols-3) per UX spec grid system
  - M2: Added lg:gap-8 for desktop gutter size per UX spec (32px)
  - L1: Added aria-hidden="true" to decorative emoji icons in ProblemCard
  - L2: Checked off Architecture Compliance Checklist items in story
