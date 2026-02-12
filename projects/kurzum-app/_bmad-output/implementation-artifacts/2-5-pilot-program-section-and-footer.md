# Story 2.5: Pilot Program Section & Footer

Status: done

## Story

As a **visitor (Stefan)**,
I want to see clear benefits of becoming a pilot company and find legal/contact information,
so that I feel trusted enough to scroll to the form and the page feels complete and professional.

## Acceptance Criteria

1. **Given** the section layout and ScrollFadeIn exist (Stories 2.1, 2.2) **When** the Pilot Program section is implemented **Then** it displays the headline "Werde einer der ersten 10 Pilotbetriebe" on a dark background (`variant="dark"`) using SectionWrapper with `id="pilot"` **And** introductory text explains the pilot search for 5-10 Elektro/Haustechnik-Betriebe **And** 4 PilotBenefitList items are rendered with checkmark icons:
   - Kostenlos während der Pilotphase (6 Monate)
   - Mitgestalten – dein Feedback formt das Produkt
   - Bevorzugter Zugang nach dem Pilottest
   - Persönliches Onboarding durch den Gründer
   **And** benefits fade in via ScrollFadeIn

2. **Given** the content sections are complete **When** the PageFooter is implemented as a Server Component **Then** it renders on a dark background (bg-primary) matching the hero (visual bookend) **And** it displays 4 elements:
   - Link to /impressum (text: "Impressum")
   - Link to /datenschutz (text: "Datenschutz")
   - Contact: servus@kurzum.app (mailto link)
   - Link to masem.at (external, opens in new tab)
   **And** links use text-secondary (masemIT Teal #009BB1) on dark background **And** footer is responsive and readable at 375px

3. **Given** Pilot and Footer are rendered **When** viewed on mobile (375px) **Then** footer links stack vertically with consistent spacing **Given** viewed on desktop (640px+) **Then** footer links display inline/horizontally

4. **Given** all new components are purely presentational **When** rendered **Then** all text uses Handwerker-Sprache (informal German) **And** `prefers-reduced-motion` is respected **And** all colors use design tokens — NO hardcoded hex values **And** footer links have minimum 48px touch targets on mobile (LP-NFR14) **And** WCAG AA contrast is met for all text on dark backgrounds

## Tasks / Subtasks

- [x] Task 1: Create PilotBenefitList Component (AC: #1)
  - [x] 1.1 Create `components/landing/pilot-benefit-list.tsx` — NO "use client" (Server Component)
  - [x] 1.2 Define and export `PilotBenefitListProps` interface: `{ items: string[] }`
  - [x] 1.3 Render as `<ul className="space-y-3 list-none">` for semantic markup (review learning from Story 2.4)
  - [x] 1.4 Each `<li>`: flex layout with checkmark icon + text
  - [x] 1.5 Checkmark: `<span className="text-accent shrink-0" aria-hidden="true">` with SVG checkmark or "✓" character, sized appropriately
  - [x] 1.6 Benefit text: `text-body text-primary-foreground/70 leading-relaxed` — readable on dark background
  - [x] 1.7 All colors via design tokens — on dark bg use `text-primary-foreground` and `text-primary-foreground/70`, NOT `text-muted-foreground`

- [x] Task 2: Create PilotSection Component (AC: #1)
  - [x] 2.1 Create `components/landing/pilot-section.tsx` as Server Component
  - [x] 2.2 Use SectionWrapper with `variant="dark"`, `id="pilot"`, `ariaLabel="Werde einer der ersten 10 Pilotbetriebe"`
  - [x] 2.3 Render headline "Werde einer der ersten 10 Pilotbetriebe" using `text-h2 md:text-h1 font-bold mb-8 md:mb-12 text-center`
  - [x] 2.4 Render intro paragraph below headline: "Wir suchen 5–10 Elektro- und Haustechnik-Betriebe, die kurzum als Erste im echten Einsatz testen. Keine Verpflichtung, kein Risiko." — `text-body-lg text-primary-foreground/70 text-center mb-8 max-w-2xl mx-auto`
  - [x] 2.5 Wrap PilotBenefitList in ScrollFadeIn — `<ScrollFadeIn delay={0}>` for the entire list
  - [x] 2.6 Hardcode benefit strings as `const` array in component file (4 items from AC #1)
  - [x] 2.7 Center benefit list: `max-w-lg mx-auto`

- [x] Task 3: Create PageFooter Component (AC: #2, #3)
  - [x] 3.1 Create `components/landing/page-footer.tsx` as Server Component
  - [x] 3.2 Use `<footer>` HTML element — NOT SectionWrapper (footer is semantically different from section)
  - [x] 3.3 Dark background: `bg-primary text-primary-foreground` (matching hero = visual bookend)
  - [x] 3.4 Inner container: `mx-auto max-w-[960px] px-4 md:px-6 py-8 md:py-12`
  - [x] 3.5 Render 4 link elements:
    - Next.js `<Link href="/impressum">` for Impressum (internal route, Story 4.1)
    - Next.js `<Link href="/datenschutz">` for Datenschutz (internal route, Story 4.1)
    - `<a href="mailto:servus@kurzum.app">` for email contact
    - `<a href="https://masem.at" target="_blank" rel="noopener noreferrer">` for masem.at (external)
  - [x] 3.6 Link styling: `text-secondary hover:text-secondary/80 transition-colors` — masemIT Teal on dark background
  - [x] 3.7 Responsive layout: `flex flex-col sm:flex-row items-center justify-center gap-4 sm:gap-6` — stacked on mobile, inline on desktop
  - [x] 3.8 Touch targets: Links wrapped with sufficient padding for 48px minimum tap area (`py-2 px-1` minimum)
  - [x] 3.9 Import `Link` from `next/link` for internal routes
  - [x] 3.10 Optional: Small "kurzum.app" brand text above links in `text-primary-foreground/50 text-body-sm`

- [x] Task 4: Integrate into Landing Page (AC: #1, #2)
  - [x] 4.1 Update `app/page.tsx` to import PilotSection and PageFooter
  - [x] 4.2 Add PilotSection after UspSection (before waitlist placeholder)
  - [x] 4.3 Add PageFooter OUTSIDE `<main>` (after closing `</main>` tag) — footer is a standalone landmark
  - [x] 4.4 Ensure section order: ... → UspSection → **PilotSection** → (waitlist placeholder) → `</main>` → **PageFooter**
  - [x] 4.5 Waitlist placeholder: Change `variant="light"` to `variant="dark"` since it now sits between two dark sections (Pilot and Footer) — OR keep as-is since Story 3.1 will redesign it

- [x] Task 5: Verify Build & Lint (AC: all)
  - [x] 5.1 Run `npm run build` — zero errors
  - [x] 5.2 Run `npm run lint` — zero errors
  - [x] 5.3 TypeScript produces zero type errors
  - [x] 5.4 Verify footer links are clickable (href values correct)
  - [x] 5.5 Verify responsive: mobile stacked footer links, desktop inline
  - [x] 5.6 Verify ScrollFadeIn animation on Pilot benefits

## Dev Notes

### Critical: Pilot Section Content (German — from Epics)

**Headline:** "Werde einer der ersten 10 Pilotbetriebe"

**Intro Paragraph:** "Wir suchen 5–10 Elektro- und Haustechnik-Betriebe, die kurzum als Erste im echten Einsatz testen. Keine Verpflichtung, kein Risiko."

**4 Benefits:**

| # | Text |
|---|------|
| 1 | Kostenlos während der Pilotphase (6 Monate) |
| 2 | Mitgestalten – dein Feedback formt das Produkt |
| 3 | Bevorzugter Zugang nach dem Pilottest |
| 4 | Persönliches Onboarding durch den Gründer |

### Critical: Footer Content

| Element | Type | Value |
|---------|------|-------|
| Impressum | Internal Link | `/impressum` (Story 4.1 — page doesn't exist yet, Link still works) |
| Datenschutz | Internal Link | `/datenschutz` (Story 4.1 — page doesn't exist yet, Link still works) |
| Email | mailto Link | `mailto:servus@kurzum.app` |
| masem.at | External Link | `https://masem.at` (target="_blank", rel="noopener noreferrer") |

### Critical: Dark Background Color Rules

Both Pilot section and Footer use dark backgrounds. On dark backgrounds:

| Element | Token | DO NOT USE |
|---------|-------|-----------|
| Headings | `text-primary-foreground` (or inherit from variant="dark") | `text-foreground` |
| Body text / descriptions | `text-primary-foreground/70` | `text-muted-foreground` (fails WCAG AA on dark) |
| Accent elements (checkmarks) | `text-accent` | `text-orange-*` |
| Footer links | `text-secondary` | `text-stone-*` or hardcoded hex |
| Subtle/muted text | `text-primary-foreground/50` | `text-muted-foreground` |
| Borders/dividers | `border-primary-foreground/10` | `border-border` |

**Contrast Verification (WCAG AA 4.5:1):**
- `text-primary-foreground` (#FAFAF9) on `bg-primary` (#1C1917): ~18:1 ratio ✓
- `text-primary-foreground/70` on `bg-primary`: ~12:1 ratio ✓
- `text-secondary` (#009BB1, Teal) on `bg-primary`: ~5.5:1 ratio ✓
- `text-accent` (#F97316, Orange) on `bg-primary`: ~5.1:1 ratio ✓

### Critical: Section Background Pattern

| Section | SectionWrapper variant | Background |
|---------|----------------------|------------|
| Hero | `variant="dark"` | bg-primary text-primary-foreground |
| Problem | `variant="light"` | bg-background text-foreground |
| Concrete Example | `variant="dark"` | bg-primary text-primary-foreground |
| Solution | `variant="warm"` | bg-muted text-foreground |
| USPs | `variant="light"` | bg-background text-foreground |
| **Pilot** | **`variant="dark"`** | **bg-primary text-primary-foreground** |
| Waitlist (Story 3.1) | TBD | TBD (will be redesigned) |
| **Footer** | **N/A (uses `<footer>`)** | **bg-primary text-primary-foreground** |

### Critical: Footer is NOT a SectionWrapper

The `<footer>` MUST use the HTML `<footer>` element, NOT `<section>` (SectionWrapper). Reasons:
1. Semantic HTML: `<footer>` has a specific ARIA role (`contentinfo`) that assistive technology uses for navigation
2. Landmark navigation: Screen readers expose `<footer>` as a separate landmark from content sections
3. Placement: Footer goes OUTSIDE `<main>`, at the same level as `<PageHeader />`
4. Styling: Footer has its own padding/spacing separate from section rhythm

```tsx
// app/page.tsx structure:
<>
  <PageHeader />
  <main className="pt-14 md:pt-16">
    {/* All content sections */}
  </main>
  <PageFooter />    {/* OUTSIDE main, sibling to header */}
</>
```

### Critical: Footer Link Implementation

**Internal links** (pages don't exist yet — that's OK, Next.js `<Link>` works with any route):
```tsx
import Link from "next/link";
// Use Next.js Link for internal routes
<Link href="/impressum">Impressum</Link>
<Link href="/datenschutz">Datenschutz</Link>
```

**External links** (security attributes required):
```tsx
// External link needs target + rel
<a href="https://masem.at" target="_blank" rel="noopener noreferrer">masem.at</a>
```

**mailto link:**
```tsx
<a href="mailto:servus@kurzum.app">servus@kurzum.app</a>
```

### Critical: Touch Target Compliance (LP-NFR14)

Footer links MUST meet 48px minimum touch target on mobile:
- Apply `py-2 px-1` (or `min-h-[48px] flex items-center`) to ensure sufficient tap area
- `gap-4` between stacked links on mobile provides visual separation
- Do NOT reduce link padding below 48px effective touch area

### Critical: Animation Specifications

| Element | Duration | Delay | Via |
|---------|----------|-------|-----|
| PilotBenefitList (all 4 items) | 400ms (ScrollFadeIn default) | 0ms | `<ScrollFadeIn delay={0}>` |
| Footer | No animation | N/A | Static — always visible at bottom |

**Note:** Benefits animate as ONE block (single ScrollFadeIn), not individually staggered. This is simpler and appropriate since there are only 4 short text items that should appear together.

### Critical: Responsive Layout

**Pilot Section:**

| Breakpoint | Layout |
|------------|--------|
| All sizes | Centered content, `max-w-lg mx-auto` for benefits |
| All sizes | Vertical stack for headline → intro → benefits |

**Footer Links:**

| Breakpoint | Layout | Gap |
|------------|--------|-----|
| Mobile (<640px) | `flex-col` (stacked) | `gap-4` (16px) |
| Desktop (640px+) | `sm:flex-row` (inline) | `sm:gap-6` (24px) |

### Previous Story Learnings (Story 2.2 + 2.3 + 2.4 + Reviews)

From Senior Developer Code Reviews:

1. **DO NOT use `text-muted-foreground` on dark backgrounds** — Use `text-primary-foreground/70` instead
2. **Use design token hover colors** — Not opacity-based hacks (e.g., `hover:bg-accent/90`)
3. **Export component prop interfaces** — Always `export interface XxxProps`
4. **Use SectionWrapper for content sections** — But NOT for footer (use `<footer>` element)
5. **Server Components by default** — All three new components are Server Components
6. **Test at build** — `npm run build` and `npm run lint` before marking complete
7. **`aria-hidden="true"` on decorative elements** — Checkmark icons in benefit list
8. **`ariaLabel` on SectionWrapper** — Pilot section needs ariaLabel
9. **Relative imports within `components/landing/`** — Use `./` for sibling imports
10. **Hardcode content data as `const` arrays** — In component files
11. **Semantic list markup** — Use `<ul>` for benefit list (Story 2.4 review fix)
12. **Use custom typography tokens consistently** — `text-body`, `text-body-sm`, `text-h1`, `text-h2` (not Tailwind defaults `text-base`, `text-sm`)
13. **Add `leading-relaxed` to description text** — For consistent line-height
14. **Import `Link` from `next/link`** for internal routes — NOT plain `<a>` tags

**Existing Components to Reuse:**
- `./section-wrapper` — SectionWrapper with variant/padding/maxWidth/id/ariaLabel props
- `./scroll-fade-in` — ScrollFadeIn with delay/className props
- `next/link` — Next.js Link for internal routes (/impressum, /datenschutz)

### Critical: Typography Scale (from globals.css)

| Token | Size | Usage in This Story |
|-------|------|---------------------|
| `text-h1` | 2.25rem (36px) | Pilot headline (desktop) |
| `text-h2` | 1.75rem (28px) | Pilot headline (mobile) |
| `text-body-lg` | 1.125rem (18px) | Pilot intro paragraph |
| `text-body` | 1rem (16px) | Benefit text |
| `text-body-sm` | 0.875rem (14px) | Footer links, footer brand text |

Section headings: `text-h2 md:text-h1 font-bold mb-8 md:mb-12 text-center`

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
│   ├── solution-step.tsx         # From Story 2.4
│   ├── solution-section.tsx      # From Story 2.4
│   ├── usp-item.tsx              # From Story 2.4
│   ├── usp-section.tsx           # From Story 2.4
│   ├── pilot-benefit-list.tsx    # NEW: Checkmark list of pilot benefits
│   ├── pilot-section.tsx         # NEW: "Werde einer der ersten 10 Pilotbetriebe" section
│   └── page-footer.tsx           # NEW: Minimal legal footer with links
app/
├── globals.css                   # Existing (no changes needed)
├── layout.tsx                    # Existing (no changes needed)
└── page.tsx                      # MODIFIED: Add PilotSection + PageFooter
```

### Architecture Compliance Checklist

- [x] File naming: kebab-case (`pilot-benefit-list.tsx`, `pilot-section.tsx`, `page-footer.tsx`)
- [x] Component naming: PascalCase (`PilotBenefitList`, `PilotSection`, `PageFooter`)
- [x] Server Components: Default for ALL three new components (no "use client")
- [x] Client Component: None needed — ScrollFadeIn handles client boundary
- [x] Imports: Relative within `components/landing/` (project convention), `next/link` for Link, `@/` for cross-directory
- [x] Styling: Tailwind classes only, design tokens from CSS variables, NO hardcoded hex
- [x] Types: No `any`, exported interfaces for component props
- [x] Accessibility: `aria-hidden="true"` on decorative checkmarks, `ariaLabel` on SectionWrapper, 48px touch targets on footer links, prefers-reduced-motion via ScrollFadeIn, WCAG AA contrast on all dark backgrounds
- [x] Semantic HTML: `<footer>` for footer (not `<section>`), `<ul>` for benefit list, `<nav>` optional for footer links

### References

- [Source: _bmad-output/planning-artifacts/epics.md#Story 2.5]
- [Source: _bmad-output/planning-artifacts/ux-design-specification.md#Pilot Program Section]
- [Source: _bmad-output/planning-artifacts/ux-design-specification.md#Footer]
- [Source: _bmad-output/planning-artifacts/ux-design-specification.md#Landing Page Section Color Map]
- [Source: _bmad-output/planning-artifacts/ux-design-specification.md#Animation & Motion Specifications]
- [Source: _bmad-output/planning-artifacts/architecture.md#Frontend Architecture — Server vs Client Components]
- [Source: _bmad-output/planning-artifacts/architecture.md#Implementation Patterns & Consistency Rules]
- [Source: _bmad-output/implementation-artifacts/2-4-solution-steps-and-usp-sections.md#Previous Story Learnings]

## Dev Agent Record

### Agent Model Used

Claude Opus 4.6

### Debug Log References

None — clean build on first attempt.

### Completion Notes List

- All 3 components created as Server Components (no "use client")
- PilotBenefitList uses SVG checkmark icon with aria-hidden="true"
- Footer uses `<footer>` element with `<nav aria-label="Footer">`, NOT SectionWrapper
- Dark bg color rules applied: text-primary-foreground/70 for body text, text-secondary for footer links, text-accent for checkmarks
- Footer links use min-h-12 inline-flex items-center for proper 48px touch targets (review fix)
- Internal routes use next/link Link, external link has target="_blank" rel="noopener noreferrer"
- PilotSection placed after UspSection, PageFooter placed outside `<main>`
- Waitlist placeholder kept as-is (Story 3.1 will redesign)

### File List

- components/landing/pilot-benefit-list.tsx (NEW)
- components/landing/pilot-section.tsx (NEW)
- components/landing/page-footer.tsx (NEW)
- app/page.tsx (MODIFIED — added PilotSection + PageFooter imports and usage)

### Change Log

- Task 1: Created PilotBenefitList — semantic `<ul>`, SVG checkmark, text-body + text-primary-foreground/70
- Task 2: Created PilotSection — SectionWrapper variant="dark", headline, intro paragraph, ScrollFadeIn-wrapped benefit list
- Task 3: Created PageFooter — `<footer>` element, 4 links (2 internal via next/link, 1 mailto, 1 external), text-secondary links, responsive flex layout
- Task 4: Updated page.tsx — PilotSection after UspSection, PageFooter outside `<main>`
- Task 5: Build and lint verified — zero errors
- Review: [H1] Fixed footer link touch targets — replaced py-2 with min-h-12 inline-flex items-center (was ~37px, now 48px)
- Review: [M1] Marked all Tasks/Subtasks and Architecture Compliance Checklist as [x]
- Review: [M2] Fixed footer hover — replaced opacity-based hover:text-secondary/80 with underline-offset-4 hover:underline (per UX spec Ghost link pattern)
- Review: [M3] Removed redundant leading-relaxed from PilotBenefitList — text-body token already sets line-height 1.6
