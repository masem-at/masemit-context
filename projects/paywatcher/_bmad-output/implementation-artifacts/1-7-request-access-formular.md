# Story 1.7: Request Access Formular

Status: done

<!-- Note: Validation is optional. Run validate-create-story for quality check before dev-story. -->

## Story

As a Besucher,
I want ein Request-Access-Formular ausfuellen koennen,
so that ich Zugang zu PayWatcher beantragen kann.

## Acceptance Criteria

1. **Given** ich bin auf der Landing Page **When** ich auf "Request Access" klicke **Then** scrolle ich zur Request-Access-Section (`id="request-access"`) und sehe ein Formular mit: Name (required), Email (required), Company/Project Name (optional), Use Case (optional, Textarea)

2. **Given** ich fuelle das Formular korrekt aus und klicke Submit **When** der POST Request an `/api/request-access` gesendet wird **Then** wird eine Email via Resend an contact@masem.at gesendet mit allen Formulardaten **And** ich sehe eine Erfolgsmeldung: "Thanks! We'll get back to you within 24 hours with your API credentials."

3. **Given** das Formular hat Validierungsfehler (Name oder Email fehlt, Email ungueltig) **When** ich Submit klicke **Then** sehe ich Inline-Fehler direkt unter den betroffenen Feldern (rot, caption-size) **And** die Fehler verschwinden bei erneutem Tippen

4. **Given** der Server-Request fehlschlaegt (Resend API Error, Netzwerk-Fehler) **When** die Fehler-Response zurueckkommt **Then** sehe ich einen Toast (sonner, rechts unten, bleibt bis dismissed): "Could not send request. Please try again."

5. **Given** die Landing Page CTAs **When** ich "Request Access" klicke in Hero, Pricing oder Trust Signals **Then** scrolle ich smooth zur Request-Access-Section

6. **Given** ein Besucher das Formular absendet **When** Analytics aktiv ist **Then** werden die Events `lp_request_access_start` (bei Focus auf erstes Feld) und `lp_request_access_submit` (bei erfolgreichem Submit) an masemIT Analytics gesendet

## Tasks / Subtasks

- [x] Task 1: Request Access API Route Handler erstellen (AC: #2, #4)
  - [x] 1.1 `src/app/api/request-access/route.ts` erstellen — POST Handler mit Zod v4 Validierung
  - [x] 1.2 Resend SDK Integration: Email an contact@masem.at mit Formulardaten (Name, Email, Company, Use Case)
  - [x] 1.3 Envelope Response Format: `{ data: { success: true } }` / `{ error: { code, message } }`
  - [x] 1.4 Error Handling: Zod Validation Error (400), Resend API Error (502), Generic Error (500)

- [x] Task 2: Request Access Section Component erstellen (AC: #1, #3, #5)
  - [x] 2.1 `src/components/landing/request-access-section.tsx` erstellen — Client Component ("use client")
  - [x] 2.2 Formular mit 4 Feldern: Name (required), Email (required), Company/Project (optional), Use Case (optional, Textarea)
  - [x] 2.3 Client-Side Validierung mit Zod v4 Schema (z.string().min(1) fuer Name, z.email() fuer Email)
  - [x] 2.4 Inline-Fehleranzeige: rot, caption-size, direkt unter dem Feld, verschwindet bei Tippen
  - [x] 2.5 Submit-Button mit Loading-State (disabled + Spinner waehrend Request)
  - [x] 2.6 Erfolgs-State: Formular wird durch Erfolgsmeldung ersetzt
  - [x] 2.7 Fehler-State: Toast via sonner bei Server-Fehler

- [x] Task 3: Landing Page CTAs auf Request Access Section verlinken (AC: #5)
  - [x] 3.1 `src/app/(landing)/page.tsx` erweitern — `<RequestAccessSection />` nach `<TrustSignals />` einfuegen
  - [x] 3.2 CTA-Buttons in hero-section.tsx, pricing-section.tsx, comparison-callout.tsx, trust-signals.tsx von `href="/login"` auf `href="/#request-access"` aendern
  - [x] 3.3 landing-header.tsx und mobile-nav.tsx CTA auf `href="/#request-access"` aendern
  - [x] 3.4 Smooth Scroll Verhalten sicherstellen (CSS `scroll-behavior: smooth` oder JS)

- [x] Task 4: Analytics Events integrieren (AC: #6)
  - [x] 4.1 `trackEvent("lp_request_access_start")` bei Focus auf erstes Formularfeld (einmalig pro Session)
  - [x] 4.2 `trackEvent("lp_request_access_submit", { status: "success" })` bei erfolgreichem Submit
  - [x] 4.3 `analytics-provider.tsx` CTA-Tracking aktualisieren: Selector von `a[href='/login']` auf `a[href='/#request-access']` anpassen (fuer Landing Page CTAs)

- [x] Task 5: Build-Validierung und Qualitaetspruefung (Alle ACs)
  - [x] 5.1 `npm run build` muss fehlerfrei durchlaufen
  - [x] 5.2 `npm run type-check` muss fehlerfrei durchlaufen
  - [x] 5.3 `npm run lint` muss fehlerfrei durchlaufen
  - [x] 5.4 Landing Page bleibt als Static im Build Output (SSG) — Request Access Section als Client Component darf SSG nicht brechen
  - [ ] 5.5 Formular-Submit auf localhost testen (Resend API Key in .env.local noetig)
  - [ ] 5.6 Mobile Responsive pruefen: Formular auf 375px (iPhone SE) funktional

## Dev Notes

### Kontext & Zweck

Dies ist die **siebte und letzte Story** von Epic 1 (Project Foundation & Landing Page). Stories 1.1–1.6 haben das Projekt-Scaffold, Design System, alle Landing Page Sektionen (Hero, Problem/Solution, How It Works, Code Examples, Pricing, Comparison, Trust Signals), SEO, Analytics und Social Sharing geliefert. Diese Story schliesst Epic 1 ab mit dem **Request Access Formular** — dem einzigen Conversion-Punkt der Landing Page im Managed MVP.

**Was PayWatcher ist:** Developer-First Verification API fuer Stablecoin-Zahlungen (USDC auf Base Network). Non-custodial, reiner Verification Layer. Im Managed MVP gibt es keinen Self-Service Signup — Besucher beantragen Zugang ueber ein Formular, Mario legt den Tenant manuell an.

**Was diese Story liefert:**
- Request Access Section als neue Landing Page Sektion (nach Trust Signals, vor Footer)
- POST `/api/request-access` Route Handler der via Resend eine Email an contact@masem.at sendet
- Alle Landing Page CTAs zeigen auf `/#request-access` statt `/login`
- Analytics Events fuer Request Access Start und Submit
- Formular mit Client-Side Validierung (Zod v4) und Inline-Fehleranzeige

**Was diese Story NICHT beinhaltet:**
- Kein MMS-Call — kein Account wird erstellt
- Kein Signup-Flow — nur Email an Mario
- Kein Magic Link — kein Resend Magic Link Integration
- Keine Aenderung am Login-Flow (Login bleibt unter `/login` fuer bestehende Tenants)
- Keine Datenbank — Formulardaten werden nur per Email uebermittelt
- Kein Rate Limiting (kann spaeter ergaenzt werden)
- Kein CAPTCHA (kann spaeter ergaenzt werden falls Spam-Problem)

**Kritische Design-Entscheidung:** Die `auth-form.tsx` (Login-Seite) enthaelt bereits einen Link `href="/#request-access"` mit Text "Request access" — dieser Backlink zeigt dass das Zusammenspiel Login ↔ Request Access bereits geplant war. Die Section MUSS `id="request-access"` verwenden.

### Technische Anforderungen

**Zu installierende Packages:** KEINE — alles ist bereits installiert.

**Bereits installierte und verwendbare Packages:**
- `resend` v6.9.2 — Email-Versand an contact@masem.at
- `zod` v4.3.6 — Server-Side und Client-Side Validierung
- `sonner` v2.0.7 — Toast-Benachrichtigungen (bereits im Root Layout gemountet: `<Toaster position="bottom-right" />`)
- `lucide-react` v0.564.0 — Icons (Send, Loader2, CheckCircle)
- Next.js 16.1.6 — Route Handlers, App Router
- React 19.2.3 — Client Components mit useState

**NICHT in dieser Story installieren:**
- `react-hook-form` — Ist NICHT installiert. Bestehende Forms (auth-form.tsx) nutzen `useState` + native `onSubmit`. Fuer ein 4-Feld-Formular reicht `useState` + Zod voellig aus. Konsistenz mit bestehendem Pattern beibehalten.
- `@hookform/resolvers` — Nicht noetig ohne RHF
- `nodemailer` — Resend SDK ist bereits da
- Kein CAPTCHA-Package (hCaptcha, reCAPTCHA) — nicht im Scope

**Zod v4 API-Aenderungen (KRITISCH — v4 ist anders als v3!):**
- `z.email()` statt `z.string().email()` fuer Top-Level Email-Validierung
- `error.issues` statt `error.errors` bei ZodError
- `z.string().min(1)` funktioniert weiterhin fuer Required-Felder
- `z.string().optional()` funktioniert weiterhin
- Schema-Definition bleibt aehnlich, aber Error-Handling anpassen

**Resend v6 API Pattern:**
```typescript
import { Resend } from "resend"

const resend = new Resend(process.env.RESEND_API_KEY)

await resend.emails.send({
  from: process.env.RESEND_FROM_EMAIL!,  // "PayWatcher <noreply@paywatcher.dev>"
  to: "contact@masem.at",
  subject: "New Request Access: {name}",
  html: "..." // oder text: "..."
})
```

**Form State Pattern (konsistent mit auth-form.tsx):**
```typescript
type FormState = "idle" | "submitting" | "success" | "error"
const [formState, setFormState] = useState<FormState>("idle")
```
Das bestehende auth-form.tsx Pattern nutzt `"idle" | "submitting" | "error"`. Fuer Request Access kommt `"success"` hinzu, da das Formular nach Erfolg durch eine Erfolgsmeldung ersetzt wird (im Gegensatz zum Login, der redirected).

**Envelope Response Format (konsistent mit bestehenden Route Handlers):**
```typescript
// Erfolg:
{ data: { success: true } }

// Fehler:
{ error: { code: "VALIDATION_ERROR", message: "..." } }
{ error: { code: "EMAIL_ERROR", message: "Failed to send email" } }
{ error: { code: "INTERNAL_ERROR", message: "..." } }
```

### Architektur-Konformitaet

**Dateinamen:** `kebab-case.ts` / `kebab-case.tsx` — IMMER.

**Komponenten-Pfade:**
- `src/app/api/request-access/route.ts` — NEU: POST Route Handler (Resend Email)
- `src/components/landing/request-access-section.tsx` — NEU: Client Component mit Formular
- `src/app/(landing)/page.tsx` — AENDERN: RequestAccessSection einfuegen
- `src/components/landing/hero-section.tsx` — AENDERN: CTA href auf `/#request-access`
- `src/components/landing/pricing-section.tsx` — AENDERN: CTA hrefs auf `/#request-access`
- `src/components/landing/comparison-callout.tsx` — AENDERN: CTA href auf `/#request-access`
- `src/components/landing/trust-signals.tsx` — AENDERN: CTA href auf `/#request-access`
- `src/components/landing/landing-header.tsx` — AENDERN: CTA href auf `/#request-access`
- `src/components/landing/mobile-nav.tsx` — AENDERN: CTA href auf `/#request-access`
- `src/components/landing/analytics-provider.tsx` — AENDERN: CTA-Selector anpassen

**Server vs Client Components:**
- Server Component: `app/api/request-access/route.ts` (Route Handler)
- Client Component: `request-access-section.tsx` (braucht useState, onSubmit, trackEvent)
- `page.tsx` bleibt Server Component — RequestAccessSection wird als Client Component importiert (wie andere Landing Sections)

**Bestehende Patterns EXAKT befolgen:**
- Section-Wrapper: `<section id="request-access" className="px-4 py-16">` mit `<div className="mx-auto max-w-5xl ...">` (wie pricing-section.tsx)
- Error Display: `text-body-sm text-destructive` unter Feldern (wie auth-form.tsx)
- Button Loading: `<Loader2 className="mr-2 size-4 animate-spin" />` (wie auth-form.tsx)
- Input Style: `className="h-11"` (wie auth-form.tsx)
- Toast: `toast.error("...")` von sonner, Position rechts unten (bereits konfiguriert)

**ANTI-Patterns:**
- KEIN `react-hook-form` — nicht installiert, useState-Pattern beibehalten
- KEIN MMS-Call — nur Resend Email
- KEIN Full-Page-Spinner — Skeleton oder Inline-Loading nur
- KEIN Toast fuer Erfolg — Erfolgs-State wird im Formular selbst angezeigt (Formular → Erfolgsmeldung)
- KEINE neuen Packages installieren
- KEIN `<form action=...>` Server Action — Client-Side fetch an `/api/request-access`

### Dateistruktur-Anforderungen

**Neue/geaenderte Dateien nach Abschluss:**

```
src/
├── app/
│   ├── (landing)/
│   │   └── page.tsx                              # AENDERN: RequestAccessSection einfuegen
│   └── api/
│       └── request-access/
│           └── route.ts                           # NEU: POST Handler (Resend Email)
│
├── components/
│   └── landing/
│       ├── request-access-section.tsx             # NEU: Client Component mit Formular
│       ├── hero-section.tsx                       # AENDERN: CTA href
│       ├── pricing-section.tsx                    # AENDERN: CTA hrefs
│       ├── comparison-callout.tsx                 # AENDERN: CTA href
│       ├── trust-signals.tsx                      # AENDERN: CTA href
│       ├── landing-header.tsx                     # AENDERN: CTA href
│       ├── mobile-nav.tsx                         # AENDERN: CTA href
│       └── analytics-provider.tsx                 # AENDERN: CTA-Selector
```

### Testing-Anforderungen

1. **Build-Validierung:** `npm run build` fehlerfrei — Landing Page bleibt `○ (Static)` im Build Output
2. **Type-Check:** `npm run type-check` fehlerfrei
3. **Lint:** `npm run lint` fehlerfrei
4. **SSG-Validierung:** `request-access-section.tsx` als Client Component darf SSG der Landing Page NICHT brechen (gleich wie analytics-provider.tsx, copy-button.tsx)
5. **API Route Test:** `curl -X POST http://localhost:3000/api/request-access -H "Content-Type: application/json" -d '{"name":"Test","email":"test@example.com"}'` muss `{ data: { success: true } }` liefern (mit gueltigem RESEND_API_KEY in .env.local)
6. **Validierungs-Test:** POST ohne Name/Email muss 400 mit `VALIDATION_ERROR` zurueckgeben
7. **CTA-Navigation:** Alle "Request Access" CTAs auf der Landing Page scrollen smooth zu `#request-access`
8. **Mobile Responsive:** Formular auf 375px (iPhone SE) muss vollstaendig funktional und bedienbar sein (Touch Targets min. 44x44px)
9. **Analytics:** In Dev zeigt `console.debug('[analytics]', 'lp_request_access_start')` und `'lp_request_access_submit'` in der Browser-Console
10. **Login-Backlink:** `auth-form.tsx` Link `href="/#request-access"` navigiert korrekt zur Request Access Section

### Vorherige Story Intelligence

**Story 1.6 Learnings (direkte Vorgaenger-Story):**
- **Analytics-Infrastruktur komplett:** `lib/analytics.ts` mit `trackEvent()`, `analytics-provider.tsx` mit IntersectionObserver und CTA-Click-Tracking. Pattern WIEDERVERWENDEN fuer `lp_request_access_start` und `lp_request_access_submit`.
- **CTA-Tracking aktuell auf `a[href='/login']`:** Muss auf `a[href='/#request-access']` geaendert werden. Der CTA-Click-Handler in analytics-provider.tsx verwendet `querySelectorAll` mit dem Selector.
- **Section-IDs konsistent:** hero, problem-solution, how-it-works, code-examples, pricing, comparison, trust-signals. NEUE Section: `request-access`.
- **SSG bestaetigt funktionierend:** Client Components (analytics-provider, copy-button, code-tabs, mobile-nav, theme-toggle) brechen SSG nicht. request-access-section.tsx wird gleich behandelt.
- **`.env.example` bereits aktualisiert:** `RESEND_API_KEY` und `RESEND_FROM_EMAIL` sind schon definiert.

**Story 1.5 Learnings:**
- **CTA-Buttons in ALLEN Sections** verlinken auf `/login` — muessen auf `/#request-access` geaendert werden.
- **Section-Spacing:** `py-16` fuer alle Sections — beibehalten.
- **Enterprise-CTA** linkt auf `mailto:enterprise@paywatcher.dev` — NICHT aendern (Enterprise ist Sonderfall).

**Epic 2 Learnings (Stories 2.1–2.4):**
- **auth-form.tsx Pattern etabliert:** `useState` fuer FormState, native `onSubmit`, Inline-Fehler mit `text-destructive`, `Loader2` Spinner. EXAKT dieses Pattern fuer request-access-section.tsx wiederverwenden.
- **Route Handler Pattern etabliert:** `app/api/api-keys/route.ts` und `app/api/tenant/route.ts` zeigen Envelope Format `{ data }` / `{ error: { code, message } }`. Gleicher Pattern fuer `/api/request-access`.
- **Zod v4 bereits im Einsatz:** v4 API (`z.email()`, `error.issues`) ist bestaetigt — keine v3-Syntax verwenden.

### Git Intelligence

**Letzte relevante Commits:**
```
0bbd31a Story 2.4 code review: add createdAt display, fix prefix operator, accessibility
f183fd7 Story 2.2 code review: fix encryption fallback, types, error handling
9d062a0 Migrate auth from Magic Link to API-Key-based login (Managed MVP)
```

**Erkenntnisse:**
- Auth-Migration auf API-Key-Login ist abgeschlossen — Login bleibt unter `/login`, Request Access ist ein separater Flow
- auth-form.tsx wurde kuerzlich refactored (Magic Link → API Key) — Code ist frisch und sauber, Pattern ist etabliert
- Code Review Fixes zeigen: Team achtet auf Accessibility, korrekte Error Handling, Type Safety
- Keine offenen Konflikte oder instabile Bereiche in Landing Page Code

### Project Structure Notes

- Story 1.7 ist die **letzte Story in Epic 1** — nach Abschluss ist Epic 1 vollstaendig
- **Request Access ersetzt Signup im Managed MVP:** Besucher beantragen Zugang, Mario onboardet manuell. Self-Service Signup erst Phase 2.
- **auth-form.tsx Backlink bereits vorhanden:** `href="/#request-access"` → Section muss `id="request-access"` haben
- **Keine Login-Page-Aenderungen noetig** — Login bleibt fuer bestehende Tenants, Request Access ist fuer neue Interessenten
- **Component Boundaries beachten:** request-access-section.tsx ist Landing-spezifisch → `components/landing/`, NICHT `components/shared/`
- **Email geht NUR an contact@masem.at** — die Formulardaten werden NICHT gespeichert, keine DB, kein MMS-Call

### References

- [Source: _bmad-output/planning-artifacts/epics.md#Story 1.7] — User Story und Acceptance Criteria (FR36)
- [Source: _bmad-output/planning-artifacts/prd.md#Request Access & Onboarding] — FR36 (Request Access Formular)
- [Source: _bmad-output/planning-artifacts/prd.md#Landing Page & Marketing] — FR7 (CTA zum Request Access)
- [Source: _bmad-output/planning-artifacts/prd.md#Analytics & Tracking] — FR32 (Request Access Start/Submit Events)
- [Source: _bmad-output/planning-artifacts/architecture.md#Route Structure] — `app/api/request-access/route.ts` Platzierung
- [Source: _bmad-output/planning-artifacts/architecture.md#Auth & Data Architecture] — Request Access Flow Beschreibung
- [Source: _bmad-output/planning-artifacts/architecture.md#Infrastructure & Deployment] — Resend fuer Email
- [Source: _bmad-output/planning-artifacts/architecture.md#Implementation Patterns] — Naming, Component, API Client Patterns
- [Source: _bmad-output/planning-artifacts/ux-design-specification.md#Core User Experience] — Conversion Flow, Button Hierarchie
- [Source: _bmad-output/implementation-artifacts/1-6-seo-social-sharing-analytics-integration.md] — Analytics Pattern, Section-IDs, CTA-Tracking
- [Source: src/components/landing/auth-form.tsx] — Form State Pattern, Inline-Fehler, Loader Spinner, `/#request-access` Backlink

## Change Log

- 2026-02-18: Story 1.7 implementiert — Request Access Formular mit API Route, Landing Page CTAs, Analytics Events
- 2026-02-18: Code Review Fixes — Toast duration:Infinity (AC#4), aria-describedby auf Formular-Inputs, scroll-behavior prefers-reduced-motion, Success-State role="status", auth-form.tsx in File List ergaenzt

## Dev Agent Record

### Agent Model Used

Claude Opus 4.6

### Debug Log References

- Build validated: `npm run build` — Landing Page remains `○ (Static)`, `/api/request-access` registered as `ƒ (Dynamic)`
- Type-check: `npm run type-check` — clean
- Lint: `npm run lint` — clean

### Completion Notes List

- **Task 1:** Created `src/app/api/request-access/route.ts` — POST handler with Zod v4 validation (`z.email()`, `error.issues`), Resend SDK email to contact@masem.at, envelope response format `{ data }` / `{ error: { code, message } }`, error handling for validation (400), Resend API (502), generic (500)
- **Task 2:** Created `src/components/landing/request-access-section.tsx` — Client Component with 4 fields (Name required, Email required, Company optional, Use Case textarea optional), Zod v4 client-side validation, inline error display (`text-body-sm text-destructive`, clears on typing), submit button with Loader2 spinner, success state replaces form with CheckCircle message, error state shows toast via sonner
- **Task 3:** Added `<RequestAccessSection />` to landing page after `<TrustSignals />`. Changed all CTA hrefs from `/login` to `/#request-access` in hero-section, pricing-section (Free/Starter/Pro tiers, Enterprise mailto unchanged), comparison-callout, trust-signals, landing-header, mobile-nav. Added `scroll-behavior: smooth` to globals.css
- **Task 4:** Added `trackEvent("lp_request_access_start")` on first field focus (once per session via useRef), `trackEvent("lp_request_access_submit", { status: "success" })` on successful submit. Updated analytics-provider.tsx CTA selector from `a[href='/login']` to `a[href='/#request-access']`. Added `request-access` to TRACKED_SECTIONS for IntersectionObserver
- **Task 5:** Build, type-check, lint all pass. SSG preserved. Subtasks 5.5 (localhost Resend test) and 5.6 (mobile responsive) require manual testing

### File List

- `src/app/api/request-access/route.ts` — NEU: POST Route Handler (Zod v4 + Resend)
- `src/components/landing/request-access-section.tsx` — NEU: Client Component mit Formular
- `src/app/(landing)/page.tsx` — GEAENDERT: RequestAccessSection import und Einbindung
- `src/components/landing/hero-section.tsx` — GEAENDERT: CTA href `/login` → `/#request-access`
- `src/components/landing/pricing-section.tsx` — GEAENDERT: CTA hrefs `/login` → `/#request-access` (3 Tiers)
- `src/components/landing/comparison-callout.tsx` — GEAENDERT: CTA href `/login` → `/#request-access`
- `src/components/landing/trust-signals.tsx` — GEAENDERT: CTA href `/login` → `/#request-access`
- `src/components/landing/landing-header.tsx` — GEAENDERT: CTA href `/login` → `/#request-access`
- `src/components/landing/mobile-nav.tsx` — GEAENDERT: CTA href `/login` → `/#request-access`
- `src/components/landing/analytics-provider.tsx` — GEAENDERT: CTA-Selector + TRACKED_SECTIONS
- `src/components/landing/auth-form.tsx` — GEAENDERT: Request-Access-Link Styling (text-accent + hover)
- `src/app/globals.css` — GEAENDERT: `scroll-behavior: smooth` hinzugefuegt
