# Story 4.1: Legal Pages — Impressum & Datenschutzerklärung

Status: done

## Story

As a **visitor (Stefan)**,
I want to find Impressum and Datenschutzerklärung with one click from the footer,
so that I can verify who runs this service and how my data is handled — which builds the trust I need before submitting the form.

## Acceptance Criteria

1. **Given** the footer with legal links exists (Epic 2, Story 2.5) **When** the `/impressum` page is implemented **Then** it displays the complete legal information for masemIT e.U.: Firmenname (masemIT e.U.), Firmenbuchnummer (FN 661236g), Adresse (Alleegasse 26, 3851 Kautzen, Austria), Kontakt (contact@masem.at), UID-Nummer (ATU82330407), Unternehmensgegenstand, Gerichtsstand **And** the page uses the kurzum brand design (layout, colors, typography) **And** a link back to the landing page is available (logo in header or explicit link)

2. **Given** the impressum page exists **When** the `/datenschutz` page is implemented **Then** it explains in clear, non-legal German (duzen where appropriate): Which data is collected via the waitlist form, how data is processed (NeonDB Frankfurt, MMS API EU), purpose of processing (pilot program recruitment), legal basis (consent via form submission + double-opt-in), data retention period, right to deletion/correction/data portability, contact for data protection inquiries (contact@masem.at), that no external tracking scripts are used, masemIT Analytics usage (self-hosted, no cookies requiring consent), Cloudflare Turnstile usage for bot protection **And** the page uses the kurzum brand design

3. **Given** both legal pages exist **When** rendered on mobile (375px) **Then** text is readable with appropriate font size and line length **And** both pages are Server Components (static content, no client-side JS needed)

4. **Given** the route is deployed **When** `pnpm build` is run **Then** zero errors, zero type errors, zero lint errors

## Tasks / Subtasks

- [x] Task 1: Create Impressum page (AC: #1, #3, #4)
  - [x] 1.1 Create `app/impressum/page.tsx` as Server Component — uses PageHeader, PageFooter, content in between
  - [x] 1.2 Add `Metadata` export: `{ title: "kurzum. — Impressum" }` (indexable, follow)
  - [x] 1.3 Implement company info content (from Dev Notes below): Firmenname, FN, UID, Adresse, Kontakt, Unternehmensgegenstand, Gerichtsstand
  - [x] 1.4 Use brand typography: `text-h1` for page title, `text-h2` for section headers, `text-body` for body text
  - [x] 1.5 Layout: `max-w-[960px]` centered, `px-4 md:px-6`, `pt-14 md:pt-16` (header offset)

- [x] Task 2: Create Datenschutzerklärung page (AC: #2, #3, #4)
  - [x] 2.1 Create `app/datenschutz/page.tsx` as Server Component — uses PageHeader, PageFooter
  - [x] 2.2 Add `Metadata` export: `{ title: "kurzum. — Datenschutzerklärung" }` (indexable, follow)
  - [x] 2.3 Implement privacy content sections (see Dev Notes below): Verantwortlicher, Zweck, Rechtsgrundlage, Datenkategorien, Empfänger/Auftragsverarbeiter, Speicherdauer, Betroffenenrechte, Beschwerderecht, Cookies/Tracking, Turnstile, Analytics
  - [x] 2.4 Use same brand typography and layout as Impressum
  - [x] 2.5 Include specific data processing details for: Waitlist form, MMS API, Resend email, Cloudflare Turnstile, masemIT Analytics

- [x] Task 3: Fix PageHeader for sub-pages (AC: #1, #2)
  - [x] 3.1 The current `PageHeader` logo links to `#hero` — on legal pages it should link to `/` (home). Check if the existing implementation handles this correctly or if an adjustment is needed (e.g., `href="/"` instead of `href="#hero"`)

- [x] Task 4: Verify Build & Lint (AC: #4)
  - [x] 4.1 Run `pnpm build` — zero errors
  - [x] 4.2 Run `pnpm lint` — zero errors
  - [x] 4.3 TypeScript produces zero type errors

## Dev Notes

### Critical: Existing Files — USE, DO NOT RECREATE

| File | Exports | How to Use |
|------|---------|------------|
| `components/landing/page-header.tsx` | `PageHeader` | REUSE — header with kurzum logo (⚠️ check `#hero` link) |
| `components/landing/page-footer.tsx` | `PageFooter` | REUSE — footer already has `/impressum` and `/datenschutz` links |
| `components/landing/section-wrapper.tsx` | `SectionWrapper` | Optional — use for consistent section padding/width |
| `components/landing/logo.tsx` | `Logo` | Already used by PageHeader |
| `app/confirmed/page.tsx` | ConfirmedPage | REFERENCE PATTERN — standalone page with Metadata, Server Component |

### Critical: PageHeader `#hero` Link Issue

The current `PageHeader` has `<a href="#hero">` which only works on the landing page. On `/impressum` and `/datenschutz`, this won't scroll anywhere — it should link to `/` instead.

**Two options:**
1. **Simplest:** Change `href="#hero"` to `href="/"` in `page-header.tsx` — navigating back to home from any page (landing page scroll-to-top is acceptable)
2. **Alternative:** Accept `#hero` as-is — on sub-pages it effectively does nothing, the logo still acts as a visual brand element

**Recommended:** Option 1 — change `href="#hero"` to `href="/"` so the logo acts as home navigation everywhere. This is standard web convention.

### Critical: Company Information (Impressum Content)

Source: `docs/company-info.md`

```
masemIT e.U.
Inhaber: Mario Semper
Einzelunternehmer

Firmenbuchnummer: FN 661236g
UID-Nummer: ATU82330407

Alleegasse 26
3851 Kautzen
Österreich

E-Mail: contact@masem.at
Web: https://masem.at

Unternehmensgegenstand: IT-Dienstleistungen
Firmenbuchgericht: Landesgericht Krems an der Donau
```

**Austrian legal requirements (ECG §5, UGB §14):**
- Firma (masemIT e.U.)
- Rechtsform (Einzelunternehmer)
- Firmenbuchnummer + Firmenbuchgericht
- Firmensitz (Alleegasse 26, 3851 Kautzen)
- UID-Nummer
- Kontakt (E-Mail reicht, keine Telefonnummer nötig)
- Unternehmensgegenstand

### Critical: Datenschutzerklärung Content Sections

Write in clear, accessible German. Duzen (informal "du") where addressing the user. Formal for legal terminology.

**Section 1 — Verantwortlicher**
```
masemIT e.U.
Mario Semper
Alleegasse 26, 3851 Kautzen, Österreich
contact@masem.at
```

**Section 2 — Welche Daten wir erheben**

Waitlist form data:
- Name (Pflichtfeld)
- E-Mail-Adresse (Pflichtfeld)
- Firmenname (Pflichtfeld)
- Fachbereich (Pflichtfeld)
- Betriebsgröße (Pflichtfeld)
- Telefonnummer (optional)
- Nachricht (optional)
- UTM-Parameter (automatisch aus URL)
- IP-Adresse (für Rate Limiting und Bot-Schutz)

**Section 3 — Zweck der Verarbeitung**
- Rekrutierung von Pilotbetrieben für die kurzum.app
- Persönliche Kontaktaufnahme zu qualifizierten Interessenten
- Double-Opt-In zur Verifizierung der Anmeldung

**Section 4 — Rechtsgrundlage**
- Art. 6 Abs. 1 lit. a DSGVO (Einwilligung via Formular + Double-Opt-In)
- Art. 6 Abs. 1 lit. f DSGVO (berechtigtes Interesse: Bot-Schutz, Sicherheit)

**Section 5 — Empfänger und Auftragsverarbeiter**

| Dienst | Anbieter | Zweck | Standort |
|--------|----------|-------|----------|
| Datenbank | NeonDB (Neon Inc.) | Speicherung der Wartelisten-Daten | Frankfurt, EU |
| E-Mail-Versand | Resend Inc. | Double-Opt-In + Bestätigungs-E-Mails | EU-Infrastruktur |
| Kontakt-Management | MMS API (masemIT e.U.) | Zentrales Party/Consent-Management | EU |
| Bot-Schutz | Cloudflare Turnstile (Cloudflare Inc.) | Schutz vor automatisierten Anmeldungen | Global (EU-Verarbeitung) |
| Hosting | Vercel Inc. | Webseiten-Hosting und Auslieferung | EU (Frankfurt) |
| Web-Analyse | masemIT Analytics (self-hosted) | Anonyme Nutzungsstatistiken | EU (self-hosted) |

**Section 6 — Speicherdauer**
- Wartelisten-Daten: bis Ende der Pilotphase (voraussichtlich April 2027) oder bis zum Widerruf der Einwilligung
- Consent-Audit-Trail (MMS): gesetzliche Aufbewahrungspflicht

**Section 7 — Deine Rechte**
- Auskunft (Art. 15 DSGVO)
- Berichtigung (Art. 16 DSGVO)
- Löschung (Art. 17 DSGVO)
- Einschränkung der Verarbeitung (Art. 18 DSGVO)
- Datenübertragbarkeit (Art. 20 DSGVO)
- Widerspruch (Art. 21 DSGVO)
- Widerruf der Einwilligung (jederzeit, ohne Angabe von Gründen)
- Kontakt: contact@masem.at

**Section 8 — Beschwerderecht**
- Österreichische Datenschutzbehörde (dsb.gv.at)
- Barichgasse 40-42, 1030 Wien
- dsb@dsb.gv.at

**Section 9 — Cookies und Tracking**
- Diese Website verwendet KEINE Cookies, die einer Einwilligung bedürfen
- masemIT Analytics: selbst-gehostet, cookiefrei, keine Weitergabe an Dritte
- Cloudflare Turnstile: setzt ein technisch notwendiges Cookie zur Bot-Erkennung (kein Tracking)
- KEINE externen Tracking-Scripts (kein Google Analytics, kein Facebook Pixel, kein LinkedIn Insight Tag)

**Section 10 — Änderungen**
- Hinweis: Diese Datenschutzerklärung kann aktualisiert werden. Stand: Februar 2026

### Critical: Page Layout Pattern

Both pages should use the SAME layout structure:

```tsx
import { PageHeader } from "@/components/landing/page-header";
import { PageFooter } from "@/components/landing/page-footer";
import type { Metadata } from "next";

export const metadata: Metadata = {
  title: "kurzum. — Impressum",  // or "kurzum. — Datenschutzerklärung"
};

export default function ImpressumPage() {
  return (
    <>
      <PageHeader />
      <main className="pt-14 md:pt-16">
        <article className="mx-auto max-w-[960px] px-4 md:px-6 py-16 md:py-24">
          <h1 className="text-h1 font-bold text-foreground mb-8">Impressum</h1>
          {/* Content with text-h2, text-body, etc. */}
        </article>
      </main>
      <PageFooter />
    </>
  );
}
```

**Key layout decisions:**
- `pt-14 md:pt-16` — offsets the fixed header height
- `max-w-[960px]` — matches landing page content width
- `py-16 md:py-24` — standard section padding
- `article` tag — semantic HTML for content pages
- No `SectionWrapper` needed — simpler to apply classes directly for one-section pages

### Critical: Typography for Content Pages

Use the project's existing text utility classes. **NO** `@tailwindcss/typography` or prose classes — they don't exist in this project.

```tsx
{/* Page title */}
<h1 className="text-h1 font-bold text-foreground mb-8">Impressum</h1>

{/* Section headings */}
<h2 className="text-h2 font-semibold text-foreground mt-10 mb-4">Firmeninformation</h2>

{/* Body text */}
<p className="text-body text-muted-foreground mb-4">Lorem ipsum...</p>

{/* Lists */}
<ul className="text-body text-muted-foreground space-y-2 mb-6 list-disc list-inside">
  <li>Item one</li>
  <li>Item two</li>
</ul>

{/* Links in text */}
<a href="mailto:contact@masem.at" className="text-accent hover:text-accent/80 underline underline-offset-4">
  contact@masem.at
</a>

{/* Muted/small text */}
<p className="text-body-sm text-muted-foreground">Stand: Februar 2026</p>
```

### Critical: NO @tailwindcss/typography

This project does NOT have `@tailwindcss/typography` installed. Do NOT use `prose` classes. Style all text manually with the typography utilities defined in `globals.css` (`text-h1`, `text-h2`, `text-body`, etc.).

### Critical: NO MDX

Content is plain JSX — no `.mdx` files, no markdown rendering. Write content directly as JSX elements.

### Previous Story Learnings (Story 3.1–3.4)

1. **Next.js 16 `searchParams`** — is a `Promise`, MUST `await` it (not relevant for static pages but be aware)
2. **Server Components** — default for pages without interactivity. No `"use client"` needed.
3. **Imports** — use `@/` alias for cross-directory imports
4. **Metadata** — export `const metadata: Metadata = { ... }` for SEO
5. **Confirmed page pattern** — Logo + content + back link. Legal pages extend this with header + footer.
6. **Brand design** — use `text-foreground`, `text-muted-foreground`, `bg-background`, `text-accent` from globals.css tokens
7. **Mobile-first** — single column layout, `px-4 md:px-6`, readable font sizes
8. **WCAG AA** — semantic HTML, heading hierarchy (`h1` → `h2`), focus visible, sufficient contrast

### Installed Package Versions

| Package | Version | Notes |
|---------|---------|-------|
| `next` | 16.1.6 | Server Components default, `Metadata` export |
| `react` | 19.2.3 | Not directly needed (no client components) |
| `tailwindcss` | ^4 | CSS-first config, utility classes from globals.css |
| `lucide-react` | ^0.563.0 | Not needed for legal pages |

### Project Structure (After This Story)

```
app/
├── impressum/
│   └── page.tsx                  # NEW: Impressum page (Server Component)
├── datenschutz/
│   └── page.tsx                  # NEW: Datenschutzerklärung page (Server Component)
components/
└── landing/
    └── page-header.tsx           # MODIFIED: href="#hero" → href="/" (if approved)
```

### Architecture Compliance Checklist

- [ ] Impressum page: `app/impressum/page.tsx` (kebab-case directory)
- [ ] Datenschutz page: `app/datenschutz/page.tsx` (kebab-case directory)
- [ ] Both Server Components — no `"use client"`
- [ ] Both use PageHeader + PageFooter for consistent navigation
- [ ] Content in German, accessible language
- [ ] No `any` types — full TypeScript strict mode
- [ ] Imports use `@/` alias for cross-directory
- [ ] Semantic HTML: `<main>`, `<article>`, `<h1>`→`<h2>` hierarchy
- [ ] Mobile-responsive: single column, readable at 375px
- [ ] No external dependencies needed
- [ ] Metadata export for SEO per page

### References

- [Source: _bmad-output/planning-artifacts/epics.md#Story 4.1]
- [Source: _bmad-output/planning-artifacts/architecture.md#Project Structure]
- [Source: _bmad-output/planning-artifacts/ux-design-specification.md#Footer]
- [Source: _bmad-output/planning-artifacts/ux-design-specification.md#Confirmation Page Navigation]
- [Source: docs/company-info.md#Legal Entity]
- [Source: components/landing/page-header.tsx — #hero link needs update]
- [Source: components/landing/page-footer.tsx — links already point to /impressum and /datenschutz]

## Dev Agent Record

### Agent Model Used

Claude Opus 4.6

### Debug Log References

- ESLint caught `<a href="/">` in PageHeader — must use Next.js `<Link>` for internal navigation. Fixed by converting `<a>` to `<Link>` from `next/link`.

### Completion Notes List

- Created `/impressum` page with complete Austrian legal information (ECG §5, UGB §14): Firmenname, Rechtsform, FN, UID, Firmensitz, Kontakt, Unternehmensgegenstand, Firmenbuchgericht
- Created `/datenschutz` page with all 10 required sections: Verantwortlicher, Datenkategorien, Zweck, Rechtsgrundlage (Art. 6 DSGVO), Empfänger/Auftragsverarbeiter (6 Dienste tabellarisch), Speicherdauer, Betroffenenrechte (7 Rechte), Beschwerderecht (DSB Wien), Cookies/Tracking, Änderungen
- Datenschutz written in clear German with "du" form where addressing user, formal for legal terminology
- Both pages: Server Components, no `"use client"`, semantic HTML (`<main>`, `<article>`, `<h1>`→`<h2>`), brand typography, mobile-responsive layout
- PageHeader: Changed `<a href="#hero">` to `<Link href="/">` — logo now navigates home from any page (standard web convention)
- Build: 0 errors, both pages generated as static content (○)
- Lint: 0 errors after Link fix
- TypeScript: 0 type errors

### Implementation Plan

1. Create `app/impressum/page.tsx` — Server Component with PageHeader, PageFooter, company info content per ECG §5/UGB §14
2. Create `app/datenschutz/page.tsx` — Server Component with all 10 privacy sections from Dev Notes
3. Fix PageHeader `href="#hero"` → `<Link href="/">` for proper home navigation on sub-pages
4. Verify build, lint, and TypeScript — zero errors

### File List

- `app/impressum/page.tsx` — NEW: Impressum page (Server Component)
- `app/datenschutz/page.tsx` — NEW: Datenschutzerklärung page (Server Component)
- `components/landing/page-header.tsx` — MODIFIED: `<a href="#hero">` → `<Link href="/">`, added `import Link from "next/link"`, updated aria-label
- `_bmad-output/implementation-artifacts/sprint-status.yaml` — MODIFIED: story status transitions

### Change Log

- 2026-02-11: Implemented Story 4.1 — Created Impressum and Datenschutzerklärung pages, fixed PageHeader home navigation
- 2026-02-11: Code review fixes — Impressum refactored to `<dl>` semantic markup, Datenschutz Auftragsverarbeiter table replaced with stacked mobile layout, File List completed

## Senior Developer Review (AI)

### Review Date
2026-02-11

### Review Outcome
Approve (with fixes applied)

### Action Items

- [x] [M1] Auftragsverarbeiter-Tabelle mobil-optimiert: Desktop-Tabelle beibehalten, Mobile-Ansicht als gestapelte `<dl>` Karten (`app/datenschutz/page.tsx`)
- [x] [M2] File List vervollständigt: `sprint-status.yaml` hinzugefügt
- [x] [M3] Impressum Firmendaten auf `<dl>/<dt>/<dd>` Semantik umgestellt (`app/impressum/page.tsx`)
- [ ] [M4] ESLint-Regel-Dokumentation: `mailto:` und externe Links als native `<a>` sind korrekt — kein Code-Fix nötig, nur Awareness
- [ ] [L1] `font-bold` auf `<h1>` redundant zu `--text-h1--font-weight: 700` — kosmetisch, kein Fix
- [ ] [L2] JSX-Section-Kommentare in Datenschutz-Seite — bewusst belassen für Wartbarkeit
- [ ] [L3] Kein explizites `lang` auf Legal-Seiten — Root-Layout `lang="de"` vererbt korrekt
