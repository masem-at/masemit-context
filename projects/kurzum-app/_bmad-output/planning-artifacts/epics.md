---
stepsCompleted: [1, 2, 3, 4]
status: 'complete'
completedAt: '2026-02-10'
inputDocuments:
  - 'prd.md'
  - 'architecture.md'
  - 'ux-design-specification.md'
  - 'docs/project-context.md'
  - 'docs/_masemIT/requirements/requirement-kurzum-landing-page-2026-02-10.md'
scopeConstraint: 'ONLY Recruiting Landing Page — FFG funding constraint, no app development before grant approval (expected May/June 2026)'
---

# kurzum-app - Epic Breakdown

## Overview

This document provides the complete epic and story breakdown for kurzum-app, decomposing the requirements from the PRD, UX Design, Architecture, and Landing Page Requirement into implementable stories.

**SCOPE CONSTRAINT:** Due to FFG funding regulations (Basisprogramm Kleinprojekt, €98,800, Antragsnr. 69168884, submitted 10.02.2026), only the Recruiting Landing Page may be developed before grant approval. The full app (FR1-FR43, NFR1-NFR27 from PRD) is deferred to Phase 2 after FFG-Freigabe. This epic breakdown covers ONLY the landing page and its supporting infrastructure.

## Requirements Inventory

### Functional Requirements

LP-FR1: Landing page accessible as single-page under kurzum.app domain
LP-FR2: Hero section with headline "Sprich statt tipp – kurzum erledigt den Rest.", subheadline, CTA button that scrolls to form
LP-FR3: Problem section with 3 pain points (WhatsApp-Chaos, Abend-Papierkram, Chef als Flaschenhals) displayed as icon + short text
LP-FR4: Concrete example section with chat-mockup showing voice input → AI structured output (Monteur Martin scenario)
LP-FR5: Solution section with 3-step explanation (Sprechen → KI verarbeitet → Überblick)
LP-FR6: USP section with 5 unique selling points (Voice-First, DSGVO, KMU-Preis, Made in Austria, Ergänzt statt ersetzt)
LP-FR7: Pilot program section with 4 benefits (kostenlos, mitgestalten, bevorzugter Zugang, persönliches Onboarding)
LP-FR8: Waitlist form with 7 fields — 5 required (Name, E-Mail, Firmenname, Gewerk dropdown, Anzahl Mitarbeiter dropdown) + 2 optional (Telefon, Nachricht)
LP-FR9: Form submission creates party in MMS API (api.masem.at) and stores kurzum-specific business data in local NeonDB (waitlist_entries table)
LP-FR10: Double-opt-in email sent via Resend after successful form submission, containing confirmation link with unique token
LP-FR11: Email confirmation page (/confirmed) validates token, updates waitlist_entries.confirmed_at, updates MMS party status to "prospect", sends notification to founder
LP-FR12: Footer with Impressum link, Datenschutzerklärung link, servus@kurzum.app, masem.at link
LP-FR13: Impressum page with masemIT e.U. legal information (FN 661236g, Alleegasse 26, 3851 Kautzen)
LP-FR14: Datenschutzerklärung page with details about form data processing, AI processing transparency, data retention
LP-FR15: SEO meta tags (Title, Description) + OG tags for LinkedIn sharing with preview image
LP-FR16: Structured data markup (Organization, Product) for search engine visibility
LP-FR17: Analytics events: page_view (auto), scroll_depth (25/50/75/100%), form_start, form_submit, form_error, cta_click — via masemIT Analytics
LP-FR18: UTM parameter tracking (utm_source, utm_medium, utm_campaign) stored with waitlist entries
LP-FR19: Bot protection via Cloudflare Turnstile on form submission
LP-FR20: Rate limiting on POST /api/waitlist — 5 submissions per IP per hour via Upstash
LP-FR21: Inline form confirmation replaces form on success — "Danke! Wir melden uns persönlich bei dir. Voraussichtlicher Start: Herbst 2026."
LP-FR22: robots.txt and sitemap.xml generation for search engine indexing
LP-FR23: Logo implementation — wordmark "kurzum." (Inter Bold) + audio waveform icon element in navigation header, as favicon (16x16, 32x32), as OG-image element, and as apple-touch-icon

### NonFunctional Requirements

LP-NFR1: Mobile-first responsive design (375px base – 1440px max)
LP-NFR2: Lighthouse Score ≥90 for Performance, Accessibility, and SEO
LP-NFR3: First Contentful Paint <1.5 seconds
LP-NFR4: No external tracking scripts — GDPR-compliant by default
LP-NFR5: Form data stored only in own infrastructure (NeonDB Region Frankfurt + MMS API EU)
LP-NFR6: Cookie banner only if technically necessary (masemIT Analytics = no consent needed)
LP-NFR7: All text in informal German (duzen), no tech jargon — Handwerker-Sprache
LP-NFR8: WCAG AA contrast ratio (4.5:1 minimum) for outdoor/sunlight readability
LP-NFR9: CSS-only animations with prefers-reduced-motion support — no JavaScript animation libraries
LP-NFR10: Self-hosted fonts (Inter variable, Latin + German special chars ä/ö/ü/ß/€)
LP-NFR11: Server Components as default — Client Components only where state/effects/browser APIs needed
LP-NFR12: TypeScript strict mode — no `any` types
LP-NFR13: Dark/light alternating section backgrounds for visual rhythm
LP-NFR14: 48px minimum touch targets for all interactive elements

### Additional Requirements

**From Architecture — Starter Template & Infrastructure:**
- Project initialized with: `create-next-app@latest` + TypeScript + Tailwind + ESLint + App Router + Turbopack + pnpm
- shadcn/ui initialized for component library (Button, Input, Textarea, Select, Label, Badge)
- Drizzle ORM + @neondatabase/serverless for database access
- Tailwind CSS v4 with CSS-first configuration (no tailwind.config.js — design tokens as CSS variables in globals.css)
- NeonDB branching integrated with Vercel: preview deployments get own DB branches

**From Architecture — Integration Stack:**
- MMS API client (lib/mms.ts) for party creation, consent tracking, status updates
- Resend for transactional emails (double-opt-in, confirmation notification)
- Cloudflare Turnstile server-side verification (lib/turnstile.ts)
- Upstash @upstash/ratelimit for API rate limiting (lib/rate-limit.ts)
- masemIT Analytics tracker script (analytics.masem.at/tracker.js) with data-project="kurzum-app"

**From Architecture — Code Patterns:**
- Zod for shared client/server validation (drizzle-zod for schema generation)
- React Hook Form + @hookform/resolvers/zod for form handling
- ApiResponse<T> unified response format for all API routes
- 3-layer error handling (API Route → Client Component → Error Boundary)
- Naming: snake_case DB, camelCase API/JSON, PascalCase Components, kebab-case files
- Absolute imports via @/ alias

**From UX — Visual & Interaction Design:**
- Color system: Anthracite primary (#1C1917), Orange accent (#F97316), warm neutrals
- ScrollFadeIn component (IntersectionObserver) for section entrance animations
- ChatMockup interactive component (voice bubble left → AI result card right)
- Section-by-section scroll mechanics ("PowerPoint slides you scroll through")
- Emotional arc: Recognition → Desire → Trust → Action
- Form validation: onBlur (not onChange) for reduced frustration
- Success state: inline confirmation replaces form, no redirect

**From UX — Logo & Brand Mark:**
- Wordmark "kurzum." in Inter Bold + simplified audio waveform icon (3-5 flat bars)
- Color variants: Orange (#F97316) on Anthracite (#1C1917) for dark mode, Anthracite on White for light mode
- Orange "." period as brand accent when wordmark is in neutral color
- Favicon: waveform icon only (16x16, 32x32) — wordmark too small at this size
- App Icon: waveform icon + "k." or waveform only (192x192, 512x512)
- Navigation: full wordmark + waveform (~32px height)
- OG Image: full wordmark + waveform, centered
- No gradients, no shadows, no rounded corners on waveform bars — flat and purist

**From Architecture — Pre-Launch Configuration:**
1. Register kurzum.app domain
2. Configure DNS (Vercel + Resend SPF/DKIM/MX)
3. Register kurzum_app as source_project in MMS
4. Create MMS API key (parties:write, consents:write, consents:read)
5. Create NeonDB production database + initial migration
6. Configure Cloudflare Turnstile for kurzum.app
7. Set up Upstash Redis for rate limiting
8. Configure masemIT Analytics project "kurzum-app"
9. Create Vercel project + environment variables
10. Verify Resend domain (kurzum.app)

### FR Coverage Map

| FR | Epic | Description |
|---|---|---|
| LP-FR1 | Epic 1 | Page accessible under kurzum.app domain |
| LP-FR2 | Epic 2 | Hero section (headline, subheadline, CTA) |
| LP-FR3 | Epic 2 | Problem section (3 pain points) |
| LP-FR4 | Epic 2 | Concrete example chat-mockup (conversion trigger) |
| LP-FR5 | Epic 2 | Solution section (3 steps) |
| LP-FR6 | Epic 2 | USP section (5 unique selling points) |
| LP-FR7 | Epic 2 | Pilot program section (4 benefits) |
| LP-FR8 | Epic 3 | Waitlist form (7 fields) |
| LP-FR9 | Epic 3 (3.2 + 3.3) | MMS API + NeonDB data storage |
| LP-FR10 | Epic 3 (3.4) | Double-opt-in email via Resend |
| LP-FR11 | Epic 3 (3.4) | Confirmation page (/confirmed) |
| LP-FR12 | Epic 2 | Footer (legal links, contact) |
| LP-FR13 | Epic 4 | Impressum page |
| LP-FR14 | Epic 4 | Datenschutzerklärung page |
| LP-FR15 | Epic 4 | SEO meta tags + OG tags for LinkedIn |
| LP-FR16 | Epic 4 | Structured data (Organization, Product) |
| LP-FR17 | Epic 4 | Analytics events via masemIT Analytics |
| LP-FR18 | Epic 4 | UTM parameter tracking |
| LP-FR19 | Epic 3 (3.2 + 3.3) | Cloudflare Turnstile bot protection |
| LP-FR20 | Epic 3 (3.2 + 3.3) | Upstash rate limiting (5/IP/hour) |
| LP-FR21 | Epic 3 | Inline form confirmation message |
| LP-FR22 | Epic 4 | robots.txt + sitemap.xml generation |
| LP-FR23 | Epic 1 | Logo (wordmark + waveform icon, all sizes) |

## Epic List

### Epic 1: Project Foundation & Brand Identity
After this epic, kurzum.app is a deployable, branded Next.js project with the correct design system, typography, color tokens, and logo assets — the canvas on which everything else is built.
**FRs covered:** LP-FR1, LP-FR23
**NFRs covered:** LP-NFR10, LP-NFR11, LP-NFR12

### Epic 2: Landing Page Content & Storytelling
After this epic, Stefan (visitor) understands kurzum's value proposition within 30 seconds — the complete emotional scroll narrative from Hero to Footer is live, responsive, and animated.
**FRs covered:** LP-FR2, LP-FR3, LP-FR4, LP-FR5, LP-FR6, LP-FR7, LP-FR12
**NFRs covered:** LP-NFR1, LP-NFR7, LP-NFR8, LP-NFR9, LP-NFR13, LP-NFR14

### Epic 3: Waitlist Registration & Lead Pipeline
After this epic, Stefan can sign up for the pilot waitlist, receive a double-opt-in email, confirm his interest, and Mario gets notified — the complete lead capture funnel works end-to-end with bot protection and rate limiting.
**FRs covered:** LP-FR8, LP-FR9, LP-FR10, LP-FR11, LP-FR19, LP-FR20, LP-FR21
**NFRs covered:** LP-NFR2, LP-NFR3, LP-NFR4, LP-NFR5

### Epic 4: Launch Readiness — SEO, Analytics & Legal
After this epic, kurzum.app is ready for LinkedIn Ads: findable via Google, trackable via masemIT Analytics with UTM support, and legally compliant with Impressum and Datenschutzerklärung.
**FRs covered:** LP-FR13, LP-FR14, LP-FR15, LP-FR16, LP-FR17, LP-FR18, LP-FR22
**NFRs covered:** LP-NFR2, LP-NFR4, LP-NFR6

---

## Epic 1: Project Foundation & Brand Identity

After this epic, kurzum.app is a deployable, branded Next.js project with the correct design system, typography, color tokens, and logo assets — the canvas on which everything else is built.

### Story 1.1: Initialize Next.js Project with Design System

As a **developer**,
I want a fully configured Next.js project with Tailwind v4 design tokens, self-hosted Inter font, and all dependencies installed,
So that all subsequent development builds on a consistent, branded foundation.

**Acceptance Criteria:**

**Given** no project exists yet
**When** the initialization commands are executed (create-next-app + shadcn init + dependencies)
**Then** the project builds without errors using `pnpm build`
**And** TypeScript strict mode is enabled with zero type errors
**And** ESLint passes with zero errors

**Given** the project is initialized
**When** the root layout (app/layout.tsx) is created
**Then** `<html lang="de">` is set for German language
**And** Inter variable font is self-hosted from /public/fonts/inter-var.woff2 (Latin + ä/ö/ü/ß/€)
**And** the font loads without external network requests

**Given** the UX design specification color system
**When** design tokens are configured in app/globals.css (Tailwind v4 CSS-first)
**Then** all color tokens are defined (--color-primary, --color-accent, neutrals, semantic colors)
**And** typography scale tokens match the UX spec (heading sizes, body sizes, weights)
**And** spacing tokens support 48px minimum touch targets
**And** shadcn/ui theme CSS variables are overridden with kurzum brand colors

**Given** the project is deployed to Vercel
**When** a user visits kurzum.app
**Then** the page loads with correct fonts and brand colors
**And** .env.example is committed with all required environment variable placeholders

### Story 1.2: Implement Logo & Brand Assets

As a **visitor (Stefan)**,
I want to see the kurzum brand mark in the browser tab and page header,
So that I immediately recognize this as a professional, trustworthy product.

**Acceptance Criteria:**

**Given** the design system is configured (Story 1.1)
**When** the logo components are implemented
**Then** a reusable Logo component renders the wordmark "kurzum." in Inter Bold
**And** the trailing "." uses --color-accent (Orange) when the wordmark is in neutral color
**And** a reusable WaveformIcon component renders 3-5 flat vertical bars as SVG

**Given** the logo color variants are defined
**When** the Logo component is rendered on a dark background
**Then** the wordmark and waveform use --color-accent (#F97316 Orange)
**When** rendered on a light background
**Then** the wordmark and waveform use --color-primary (#1C1917 Anthracite)

**Given** favicon assets are needed for browser tabs
**When** favicon files are generated
**Then** favicon.ico exists at 16x16 and 32x32 with the waveform icon only
**And** apple-touch-icon.png exists at 180x180

**Given** the page is loaded
**When** the navigation area is visible
**Then** the full logo (wordmark + waveform, ~32px height) is displayed
**And** the logo links to the page top (anchor to hero section)

---

## Epic 2: Landing Page Content & Storytelling

After this epic, Stefan (visitor) understands kurzum's value proposition within 30 seconds — the complete emotional scroll narrative from Hero to Footer is live, responsive, and animated.

### Story 2.1: Section Layout System & Hero Section

As a **visitor (Stefan)**,
I want to immediately understand what kurzum is and feel "Die reden wie ich" within 3 seconds of landing,
So that I'm compelled to scroll further instead of bouncing.

**Acceptance Criteria:**

**Given** the design system is configured (Epic 1)
**When** a SectionWrapper component is implemented
**Then** it accepts props for background variant (dark/light/warm), padding, and max-width
**And** it renders consistent section spacing across all breakpoints (375px–1440px)
**And** dark sections use --color-primary, light sections alternate --color-bg-light and --color-bg-warm

**Given** the SectionWrapper exists
**When** the HeroSection is implemented as a Server Component
**Then** it displays the headline "Sprich statt tipp – kurzum erledigt den Rest."
**And** it displays the subheadline with product description and "Ab €10/User"
**And** a CTA button "Jetzt Pilotbetrieb werden" smooth-scrolls to the form section
**And** a visual placeholder for the Monteur illustration is rendered

**Given** the hero is viewed on mobile (375px)
**When** the viewport is fully visible
**Then** headline + subheadline + CTA fit within one screen without scrolling
**And** the CTA button has minimum 48px height touch target
**And** contrast ratio meets WCAG AA (4.5:1) for text on dark background

**Given** the hero is viewed on desktop (1440px)
**When** the viewport is fully visible
**Then** content is centered with appropriate max-width and whitespace

### Story 2.2: Problem Section with Animated Pain Points

As a **visitor (Stefan)**,
I want to see my daily frustrations named ("Kommt dir das bekannt vor?"),
So that I feel validated and think "Die kennen mein Problem."

**Acceptance Criteria:**

**Given** the SectionWrapper exists (Story 2.1)
**When** a ScrollFadeIn Client Component is implemented
**Then** it uses IntersectionObserver to detect when elements enter the viewport
**And** elements animate from opacity 0 + translateY 20px → opacity 1 + translateY 0
**And** animation duration is 0.4s with ease-out timing
**And** stagger delay between sequential elements is 100-150ms
**And** `prefers-reduced-motion` is respected — animations disabled when user prefers it
**And** all animations use CSS only — no JavaScript animation libraries

**Given** the ScrollFadeIn component exists
**When** the Problem section is implemented as a Server Component (with ScrollFadeIn wrapper)
**Then** it displays the headline "Kommt dir das bekannt vor?"
**And** 3 ProblemCard components are rendered with icon + headline + text:
  - "WhatsApp-Chaos" — "30 Nachrichten, 5 Gruppen, nichts zugeordnet..."
  - "Abend-Papierkram" — "Nach 10 Stunden Baustelle noch Stundenzettel ausfüllen..."
  - "Der Chef als Flaschenhals" — "Alles läuft über dich..."
**And** cards use border-left accent in --color-accent (Orange)
**And** cards fade in sequentially via ScrollFadeIn as user scrolls into view

**Given** the section is viewed on mobile
**When** cards are rendered
**Then** they stack vertically with consistent spacing
**Given** the section is viewed on desktop
**When** cards are rendered
**Then** they display in a 3-column grid

### Story 2.3: Concrete Example Chat-Mockup

As a **visitor (Stefan)**,
I want to see a real field worker scenario where voice input becomes structured AI output,
So that I think "Das passiert bei mir jeden Tag" and understand the product's core value.

**Acceptance Criteria:**

**Given** the section layout and ScrollFadeIn exist (Stories 2.1, 2.2)
**When** the ChatMockup Client Component is implemented
**Then** it displays the headline "Ein typischer Einsatz" on a dark background

**Given** the ChatMockup is rendered
**When** it enters the viewport
**Then** the left side shows a VoiceBubble sub-component: avatar + waveform graphic + "0:28" duration + Monteur Martin's message text in a speech bubble
**And** the right side shows an AiResultCard sub-component: 3 structured action items with icons (✅ Status, 📦 Material, 📅 Terminänderung)
**And** the VoiceBubble fades in first (0.3s), then the AiResultCard fades in with 0.3s delay — showing the transformation sequence

**Given** the mockup is viewed on desktop
**When** content is rendered
**Then** VoiceBubble appears on the left, AiResultCard on the right (side-by-side)
**Given** it is viewed on mobile
**When** content is rendered
**Then** VoiceBubble appears above, AiResultCard below (stacked vertically)

**Given** the ChatMockup is purely visual
**When** rendered
**Then** no audio playback functionality exists — this is a static visual mockup with animation
**And** text content uses Handwerker-Sprache (informal German, no tech jargon)

### Story 2.4: Solution Steps & USP Sections

As a **visitor (Stefan)**,
I want to understand how kurzum works in 3 simple steps and why it's the right choice,
So that I move from "interesting" to "I want this."

**Acceptance Criteria:**

**Given** the section layout and ScrollFadeIn exist
**When** the Solution section is implemented
**Then** it displays the headline "So einfach funktioniert kurzum"
**And** 3 SolutionStep components are rendered with number + icon + headline + text:
  - Step 1: "Sprechen" — "Monteur spricht 30 Sekunden ins Handy. Fertig."
  - Step 2: "KI verarbeitet" — "Zusammenfassung, Projekt-Zuordnung, Aktionspunkte. Automatisch."
  - Step 3: "Überblick" — "Chef und Büro sehen alles im Dashboard. Kein Anruf nötig."
**And** steps fade in sequentially via ScrollFadeIn

**Given** steps are viewed on mobile
**When** rendered
**Then** steps stack vertically with numbered indicators
**Given** steps are viewed on desktop
**When** rendered
**Then** steps display horizontally in a 3-column layout

**Given** the Solution section exists
**When** the USP section is implemented
**Then** it displays the headline "Warum kurzum?"
**And** 5 UspItem components are rendered with emoji/icon + headline + one-liner:
  - 🎙️ Voice-First — "Für dreckige Hände gemacht. Sprechen statt tippen."
  - 🔒 DSGVO by Design — "EU-Server, kein Telefonbuchzugriff. Nicht aufgeklebt, eingebaut."
  - 💰 KMU-Preis — "Ab €10/User/Monat. Weniger als ein Strafzettel."
  - 🇦🇹 Made in Austria — "Kein Silicon-Valley-Startup. Einer der versteht, wie KMU ticken."
  - 🤝 Ergänzt statt ersetzt — "kurzum ersetzt nicht dein ERP – es macht Kommunikation intelligent."
**And** USPs use a vertical stack layout with consistent spacing

### Story 2.5: Pilot Program Section & Footer

As a **visitor (Stefan)**,
I want to see clear benefits of becoming a pilot company and find legal/contact information,
So that I feel trusted enough to scroll to the form and the page feels complete and professional.

**Acceptance Criteria:**

**Given** the section layout and ScrollFadeIn exist
**When** the Pilot Program section is implemented
**Then** it displays the headline "Werde einer der ersten 10 Pilotbetriebe" on a dark background
**And** introductory text explains the pilot search for 5-10 Elektro/Haustechnik-Betriebe
**And** 4 PilotBenefitList items are rendered with checkmark icons:
  - ✅ Kostenlos während der Pilotphase (6 Monate)
  - ✅ Mitgestalten – dein Feedback formt das Produkt
  - ✅ Bevorzugter Zugang nach dem Pilottest
  - ✅ Persönliches Onboarding durch den Gründer
**And** benefits fade in via ScrollFadeIn

**Given** the content sections are complete
**When** the PageFooter is implemented as a Server Component
**Then** it renders on a dark background (--color-primary) matching the hero (visual bookend)
**And** it displays 4 elements:
  - Link to /impressum (text: "Impressum")
  - Link to /datenschutz (text: "Datenschutz")
  - Contact: servus@kurzum.app (mailto link)
  - Link to masem.at (external, opens in new tab)
**And** links use --color-secondary (masemIT Teal) on dark background
**And** footer is responsive and readable at 375px

---

## Epic 3: Waitlist Registration & Lead Pipeline

After this epic, Stefan can sign up for the pilot waitlist, receive a double-opt-in email, confirm his interest, and Mario gets notified — the complete lead capture funnel works end-to-end with bot protection and rate limiting.

### Story 3.1: Waitlist Form with Client-Side Validation

As a **visitor (Stefan)**,
I want to fill out a simple waitlist form with instant field-level feedback,
So that I can sign up for the pilot program in under 2 minutes without confusion.

**Acceptance Criteria:**

**Given** the landing page content sections exist (Epic 2)
**When** a shared Zod validation schema is created (lib/validations/waitlist.ts)
**Then** it validates all 7 fields:
  - Name: required, min 2 chars
  - E-Mail: required, valid email format
  - Firmenname: required, min 2 chars
  - Gewerk: required, enum (Elektrotechnik, Sanitär/Heizung/Lüftung, Sonstige)
  - Anzahl Mitarbeiter: required, enum (1–5, 6–10, 11–20, 21–50, 50+)
  - Telefon: optional, valid phone format if provided
  - Nachricht: optional, max 1000 chars
**And** the schema is importable by both client and server code

**Given** the Zod schema exists
**When** the WaitlistForm Client Component is implemented
**Then** it uses React Hook Form with @hookform/resolvers/zod
**And** all 5 required fields and 2 optional fields are rendered using shadcn/ui components (Input, Select, Textarea, Label, Button)
**And** the Nachricht field displays placeholder text "Was nervt dich am meisten an der aktuellen Kommunikation?"
**And** Cloudflare Turnstile widget is visually integrated via @marsidev/react-turnstile

**Given** a user is filling out the form
**When** they leave a required field empty and blur away
**Then** an inline error message appears below that field in German (e.g., "Bitte gib deinen Namen ein.")
**And** validation triggers onBlur, NOT onChange
**And** error messages disappear when the field is corrected

**Given** the form is submitted successfully
**When** the API returns success
**Then** the WaitlistForm is replaced by a FormConfirmation component (inline, no page redirect)
**And** the confirmation displays "Danke! Wir melden uns persönlich bei dir. Voraussichtlicher Start: Herbst 2026."

**Given** the form is submitted
**When** a network error or API error occurs
**Then** a user-friendly error message appears in German (no technical details)
**And** the form remains editable for retry
**And** masemit.track('form_error', { field }) is NOT called yet (analytics is Epic 4)

**Given** the form is viewed on mobile (375px)
**When** rendered
**Then** all fields stack vertically with minimum 48px touch targets
**And** the CTA button "Pilotplatz sichern" uses --color-accent (Orange), full width on mobile

### Story 3.2: Database Schema & Integration Clients

As a **founder (Mario)**,
I want the database schema and external service clients ready,
So that the waitlist API route can store leads and sync them with my CRM.

**Acceptance Criteria:**

**Given** no database schema exists yet
**When** the database layer is set up
**Then** lib/db/index.ts creates a Drizzle instance connected to NeonDB via @neondatabase/serverless
**And** lib/db/schema.ts defines the waitlist_entries table with all columns per architecture spec (id, mms_party_id, mms_contact_point_id, company_name, gewerk, company_size, phone, message, utm_source, utm_medium, utm_campaign, confirmation_token, confirmed_at, created_at, updated_at)
**And** indexes are created: idx_waitlist_entries_mms_party_id, idx_waitlist_entries_confirmation_token
**And** drizzle-kit generates a migration file committed to git
**And** drizzle.config.ts is configured for NeonDB

**Given** the database is ready
**When** the MMS API client is implemented (lib/mms.ts)
**Then** it calls POST /v1/parties with: email, displayName, source_project "kurzum_app", marketing_consent true, consent IP and user agent
**And** it returns partyId, contactPointId, created boolean, and unsubscribeToken
**And** it authenticates via x-api-key header using MMS_API_KEY env variable
**And** MMS errors are caught and logged without exposing details to the user

**Given** security utilities are needed
**When** the Turnstile verification helper is implemented (lib/turnstile.ts)
**Then** it verifies Cloudflare Turnstile tokens server-side via the siteverify API
**And** it returns a boolean indicating valid/invalid
**And** it uses TURNSTILE_SECRET_KEY env variable

**Given** rate limiting is needed
**When** the rate limiter is implemented (lib/rate-limit.ts)
**Then** it uses @upstash/ratelimit with a sliding window of 5 requests per IP per hour
**And** it uses UPSTASH_REDIS_REST_URL and UPSTASH_REDIS_REST_TOKEN env variables

**Given** a unified API response format is needed
**When** lib/api-response.ts is implemented
**Then** it exports the ApiResponse<T> type and helper functions for success/error responses
**And** it defines standard error codes: VALIDATION_ERROR, BOT_DETECTED, RATE_LIMITED, INTERNAL_ERROR

### Story 3.3: Waitlist API Route

As a **founder (Mario)**,
I want every waitlist signup securely validated, bot-protected, rate-limited, and stored in both MMS and local DB,
So that I receive only legitimate leads with GDPR-compliant consent tracking.

**Acceptance Criteria:**

**Given** the database, MMS client, Turnstile, and rate limiter are ready (Story 3.2)
**When** POST /api/waitlist receives a request
**Then** processing follows this exact sequence:
  1. Cloudflare Turnstile token verification (lib/turnstile.ts) → 403 if invalid
  2. Upstash rate limit check (lib/rate-limit.ts, 5 per IP per hour) → 429 if exceeded
  3. Server-side Zod validation of request body → 400 with VALIDATION_ERROR if invalid
  4. MMS API: create/find party with consent
  5. Local DB: INSERT waitlist_entries with mms_party_id + business data + UTM params
  6. Return ApiResponse<{ entryId }> with success: true

**Given** UTM parameters are present in the request
**When** the entry is stored
**Then** utm_source, utm_medium, utm_campaign are extracted from the request body or referrer headers
**And** stored in the corresponding columns of waitlist_entries

**Given** any step in the pipeline fails
**When** an error occurs
**Then** the API returns the appropriate ApiResponse error format with correct HTTP status code
**And** errors are logged with structured console.error (English, no PII)
**And** user-facing error messages are in German
**And** CORS only allows kurzum.app origin
**And** non-JSON requests are rejected

### Story 3.4: Double-Opt-In Email & Confirmation Flow

As a **visitor (Stefan)**,
I want to receive a confirmation email after signing up and verify my interest with one click,
So that I know my registration was received and I'm officially on the waitlist.

As a **founder (Mario)**,
I want to be notified when a lead confirms via double-opt-in,
So that I can personally reach out to qualified pilot companies.

**Acceptance Criteria:**

**Given** a waitlist entry was successfully created (Story 3.3)
**When** the API route completes the DB insert
**Then** a double-opt-in email is sent via Resend (lib/email.ts)
**And** the email is sent from the address configured in RESEND_FROM_EMAIL (mario@kurzum.app)
**And** the email contains a confirmation link: {NEXT_PUBLIC_APP_URL}/confirmed?token={confirmation_token}
**And** the email includes an unsubscribe link using the MMS unsubscribe token
**And** the email text is in informal German (duzen), matching the brand tonality
**And** the email subject clearly identifies the purpose (e.g., "kurzum. — Bitte bestätige deine Anmeldung")

**Given** a visitor clicks the confirmation link
**When** /confirmed?token=xxx is loaded
**Then** the page looks up the waitlist_entry by confirmation_token
**And** if the token is valid and not yet confirmed:
  - waitlist_entries.confirmed_at is set to now()
  - MMS API: party status is updated to "prospect" via PATCH /v1/parties/:id
  - A notification email is sent to Mario (founder) with the lead details
  - The page displays "Du bist dabei!" with a friendly confirmation message
**And** if the token is already confirmed:
  - The page displays "Du bist bereits bestätigt." (no error state, calm tone)
**And** if the token is invalid or not found:
  - The page displays a friendly "Link ungültig" message (no technical details)

**Given** the confirmation page is rendered
**When** viewed on any device
**Then** it uses the kurzum brand design (logo, colors, typography)
**And** it is a standalone page (no form, no navigation to other sections needed)

**Given** the Resend API is unavailable
**When** email sending fails
**Then** the waitlist entry is still saved (email failure doesn't block registration)
**And** the error is logged for manual follow-up
**And** the user sees the form confirmation (not an error)

---

## Epic 4: Launch Readiness — SEO, Analytics & Legal

After this epic, kurzum.app is ready for LinkedIn Ads: findable via Google, trackable via masemIT Analytics with UTM support, and legally compliant with Impressum and Datenschutzerklärung.

### Story 4.1: Legal Pages — Impressum & Datenschutzerklärung

As a **visitor (Stefan)**,
I want to find Impressum and Datenschutzerklärung with one click from the footer,
So that I can verify who runs this service and how my data is handled — which builds the trust I need before submitting the form.

**Acceptance Criteria:**

**Given** the footer with legal links exists (Epic 2, Story 2.5)
**When** the /impressum page is implemented
**Then** it displays the complete legal information for masemIT e.U.:
  - Firmenname: masemIT e.U.
  - Firmenbuchnummer: FN 661236g
  - Adresse: Alleegasse 26, 3851 Kautzen
  - Kontakt: servus@kurzum.app
  - Unternehmensgegenstand, Gerichtsstand, UID-Nummer as legally required
**And** the page uses the kurzum brand design (layout, colors, typography)
**And** a link back to the landing page is available (logo in header or explicit link)

**Given** the impressum page exists
**When** the /datenschutz page is implemented
**Then** it explains in clear, non-legal German (duzen where appropriate):
  - Which data is collected via the waitlist form (name, email, company, etc.)
  - How data is processed (NeonDB Frankfurt, MMS API EU)
  - Purpose of processing (pilot program recruitment)
  - Legal basis (consent via form submission + double-opt-in)
  - Data retention period
  - Right to deletion, correction, and data portability
  - Contact for data protection inquiries (servus@kurzum.app)
  - That no external tracking scripts are used (no Google Analytics, no Facebook Pixel)
  - masemIT Analytics usage (self-hosted, no cookies requiring consent)
  - Cloudflare Turnstile usage for bot protection
**And** the page uses the kurzum brand design

**Given** both legal pages exist
**When** rendered on mobile (375px)
**Then** text is readable with appropriate font size and line length
**And** both pages are Server Components (static content, no client-side JS needed)

### Story 4.2: SEO, Meta Tags & Social Sharing

As a **founder (Mario)**,
I want kurzum.app to be discoverable via Google search and look professional when shared on LinkedIn,
So that organic traffic and LinkedIn Ads campaigns drive qualified visitors to the page.

**Acceptance Criteria:**

**Given** the landing page content exists (Epic 2)
**When** metadata is configured in app/layout.tsx
**Then** the page title is set (e.g., "kurzum. — Sprich statt tipp. Die KI-App für Handwerksbetriebe.")
**And** meta description targets the primary keywords and summarizes the value proposition
**And** `<html lang="de">` is set (already from Epic 1, verify)
**And** canonical URL points to https://kurzum.app

**Given** metadata is configured
**When** the OG tags are implemented
**Then** og:title, og:description, og:image, og:url, og:type are set
**And** twitter:card is set to "summary_large_image"
**And** og:locale is set to "de_AT"

**Given** OG tags exist
**When** app/opengraph-image.tsx is implemented
**Then** it generates a 1200x630 OG image dynamically using Next.js ImageResponse
**And** the image includes the kurzum logo (wordmark + waveform) centered
**And** the image uses brand colors (Orange on Anthracite or as appropriate)
**And** the image renders correctly in LinkedIn post preview

**Given** SEO metadata is configured
**When** structured data is added to the page
**Then** JSON-LD for Organization is embedded (masemIT e.U., address, contact)
**And** JSON-LD for Product is embedded (kurzum.app, description, offers)

**Given** search engine indexing is needed
**When** app/robots.ts is implemented
**Then** it allows indexing of /, /impressum, /datenschutz
**And** it disallows /api/*, /confirmed
**When** app/sitemap.ts is implemented
**Then** it lists all public pages with lastModified dates

**Given** the page is deployed
**When** a Lighthouse audit is run
**Then** the SEO score is ≥90

### Story 4.3: Analytics Integration & Event Tracking

As a **founder (Mario)**,
I want to track visitor behavior, scroll engagement, form interactions, and traffic sources,
So that I can measure conversion rate, optimize messaging, and prove market interest (KPI: ≥5% conversion, 30-50 signups in 4 months).

**Acceptance Criteria:**

**Given** the landing page and form exist (Epics 2+3)
**When** masemIT Analytics is integrated in app/layout.tsx
**Then** the tracker script loads from https://analytics.masem.at/tracker.js
**And** the script tag uses strategy="afterInteractive" and data-project="kurzum-app"
**And** no external tracking scripts are loaded (no Google Analytics, no Facebook Pixel)
**And** no cookie consent banner is required (masemIT Analytics is cookieless)

**Given** the tracker is loaded
**When** a visitor views the page
**Then** page_view is tracked automatically by the tracker
**When** a visitor scrolls through the page
**Then** scroll_depth is tracked at 25%, 50%, 75%, 100% thresholds via data-engagement-pages attribute

**Given** the tracker is loaded
**When** a visitor clicks the first form field
**Then** masemit.track('form_start') is fired
**When** the form is successfully submitted
**Then** masemit.track('form_submit', { gewerk, company_size }) is fired
**When** a form validation error occurs
**Then** masemit.track('form_error', { field }) is fired for the specific field

**Given** CTA buttons exist on the page
**When** a visitor clicks a CTA button ("Jetzt Pilotbetrieb werden" in hero, "Pilotplatz sichern" in form)
**Then** masemit.track('cta_click', { location }) is fired with the button's section as location

**Given** a visitor arrives via LinkedIn Ads or other campaigns
**When** the URL contains utm_source, utm_medium, or utm_campaign parameters
**Then** these are automatically tracked by masemIT Analytics via the page URL
**And** the UTM values are also stored with the waitlist entry in NeonDB (already implemented in Story 3.2)

**Given** all analytics events are configured
**When** a Lighthouse audit is run
**Then** the Performance score is ≥90 (tracker script does not degrade performance)
**And** no third-party cookie warnings appear
