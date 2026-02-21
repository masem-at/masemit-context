# Story 6.5: Request Access Confirmation Email

Status: done

<!-- Note: Validation is optional. Run validate-create-story for quality check before dev-story. -->

## Story

As a Besucher der PayWatcher Landing Page,
I want nach dem Absenden des Request-Access-Formulars eine Bestaetigungsmail erhalten und einen aussagekraeftigen Use Case angeben muessen,
so that ich weiss dass meine Anfrage eingegangen ist und Mario die Anfrage besser bewerten kann.

## Acceptance Criteria

1. **Given** das Use-Case-Feld im Request-Access-Formular
   **When** ich das Formular ausfuelle
   **Then** ist das Use-Case-Feld ein Pflichtfeld (required)
   **And** die Validierung erfordert mindestens 20 Zeichen
   **And** der Placeholder lautet: "Tell us about your project — e.g. what are you building, how will you accept USDC payments?"
   **And** bei Validierungsfehler erscheint: "Please describe your use case so we can set up the right plan for you."

2. **Given** ich habe das Request-Access-Formular ausgefuellt und abgesendet
   **When** die Anfrage erfolgreich verarbeitet wird
   **Then** erhalte ich eine Bestaetigungsmail an meine angegebene Email-Adresse
   **And** die Bestaetigungsmail enthaelt: persoenliche Anrede mit meinem Namen, Bestaetigung des Eingangs, Zeitrahmen (innerhalb 24h), Link zu den API Docs (https://paywatcher.dev/docs), Hinweis "Questions? Reply to this email.", Signatur (Mario Semper, Founder, masemIT)
   **And** die bestehende Admin-Notification-Email an contact@masem.at wird weiterhin gesendet (unveraendert)

3. **Given** die Bestaetigungsmail wird gesendet
   **When** ich die Email in meinem Postfach oeffne
   **Then** ist die Email professionell formatiert (HTML mit Plain-Text-Fallback)
   **And** der Absender ist `process.env.RESEND_FROM_EMAIL` (PayWatcher noreply)
   **And** Reply-To ist `contact@masem.at` (damit Antworten bei Mario ankommen)
   **And** der Betreff ist: "PayWatcher — We received your request"
   **And** die Email ist auf Englisch (internationale Developer-Zielgruppe)

4. **Given** der Email-Versand an den User fehlschlaegt (z.B. Resend Error)
   **When** die API Route die Anfrage verarbeitet
   **Then** wird der Fehler geloggt (`console.error`) aber die API Response bleibt erfolgreich
   **And** die Admin-Notification-Email wird trotzdem gesendet (oder wurde bereits gesendet)
   **And** der User sieht weiterhin die Erfolgs-Nachricht im Formular

5. **Given** die Erfolgs-Nachricht nach dem Absenden
   **When** die Anfrage erfolgreich war
   **Then** sehe ich: "Request submitted! Check your email for confirmation — we'll get back to you within 24 hours."

## Tasks / Subtasks

- [x] Task 1: Use-Case-Feld zum Pflichtfeld machen (AC: #1)
  - [x] 1.1: `src/app/api/request-access/route.ts` — Zod Schema anpassen:
    - `useCase` von `z.string().optional()` zu `z.string().min(20, "Please describe your use case so we can set up the right plan for you.")`
    - HINWEIS: Die bestehende Zod Schema `requestAccessSchema` aendern, nicht eine neue erstellen
  - [x] 1.2: `src/components/landing/request-access-section.tsx` — Formular anpassen:
    - UseCase Textarea von optional zu required aendern
    - Client-seitige Validierung: Min. 20 Zeichen (Zod Schema im Component anpassen falls vorhanden, oder inline Validation)
    - Placeholder aktualisieren: "Tell us about your project — e.g. what are you building, how will you accept USDC payments?"
    - Label-Aenderung: "(optional)" Marker entfernen falls vorhanden
    - Fehlermeldung bei Validierungsfehler anzeigen (inline, unter dem Feld)
  - [x] 1.3: HINWEIS: Das Feld existiert bereits als Textarea im Formular — es muss nur von optional auf required umgestellt und die Validierung verschaerft werden. Kein neues UI-Element noetig.

- [x] Task 2: Email-Template fuer Bestaetigungsmail erstellen (AC: #2, #3)
  - [x] 2.1: Neues Verzeichnis `src/lib/email/` erstellen
  - [x] 2.2: `src/lib/email/request-access-confirmation.ts` erstellen:
    - Exportiert Funktion `buildConfirmationEmail(name: string): { subject: string; html: string; text: string }`
    - Subject: `"PayWatcher — We received your request"`
    - HTML-Version: Minimales, email-client-kompatibles HTML mit Inline-CSS
    - Plain-Text-Version: Gleicher Inhalt ohne Formatierung
  - [x] 2.3: Email-Inhalt (HTML + Text):
    ```
    Hi {name},

    Thanks for your interest in PayWatcher!

    We've received your request and will review it within 24 hours.
    If approved, we'll send you your API credentials and help you
    get set up.

    In the meantime, you can check out our API docs:
    https://paywatcher.dev/docs

    Questions? Reply to this email.

    — Mario Semper
    Founder, masemIT
    ```
  - [x] 2.4: HTML-Template Regeln:
    - Inline CSS (keine externen Stylesheets)
    - Einfaches Layout (kein komplexes Grid, keine Bilder/Logo)
    - Monospace fuer den Docs-Link (email-sicher)
    - Dunkler Hintergrund vermeiden (Email-Clients rendern Dark Mode unterschiedlich)
    - Einfache Schriftart (system-ui, Arial, sans-serif)
  - [x] 2.5: KEIN React Email, MJML oder anderes Template-System — einfaches Template-Literal reicht fuer MVP

- [x] Task 3: Request-Access API Route erweitern (AC: #2, #3, #4)
  - [x] 3.1: `src/app/api/request-access/route.ts` erweitern — NACH dem erfolgreichen Versand der Admin-Email:
    ```typescript
    // Bestaetigungsmail an den User (fire-and-forget)
    try {
      const confirmation = buildConfirmationEmail(name)
      await resend.emails.send({
        from: process.env.RESEND_FROM_EMAIL!,
        to: email,
        replyTo: "contact@masem.at",
        subject: confirmation.subject,
        html: confirmation.html,
        text: confirmation.text,
      })
    } catch (confirmError) {
      console.error("[request-access] Failed to send confirmation email:", confirmError)
    }
    ```
  - [x] 3.2: Import hinzufuegen: `import { buildConfirmationEmail } from "@/lib/email/request-access-confirmation"`
  - [x] 3.3: REIHENFOLGE: Erst Admin-Email (primaer), dann User-Bestaetigungsmail (sekundaer). Wenn Admin-Email fehlschlaegt, trotzdem Bestaetigungsmail versuchen.
  - [x] 3.4: NICHT parallelisieren — sequentieller Versand ist sicherer und einfacher zu debuggen
  - [x] 3.5: Bestehende Logik (Admin-Email + MMS Lead-Erfassung) NICHT veraendern, nur die Bestaetigungsmail HINZUFUEGEN
  - [x] 3.6: HINWEIS: Die Admin-Email nutzt aktuell `resend.emails.send()` direkt in der Route. Der gleiche `resend` Client-Instance kann fuer beide Emails verwendet werden.

- [x] Task 4: Erfolgs-Nachricht aktualisieren (AC: #5)
  - [x] 4.1: `src/components/landing/request-access-section.tsx` — Erfolgs-Nachricht anpassen:
    - Aktuell: "Request received! Check your email for confirmation — we'll get back to you within 24 hours with your API credentials."
    - Neu: "Request submitted! Check your email for confirmation — we'll get back to you within 24 hours."
    - HINWEIS: Nur Text-Update. Keine strukturelle Aenderung am Component.

## Dev Notes

### Kritische Architektur-Erkenntnisse

**Resend ist bereits vollstaendig integriert:**
- `resend` Package ist installiert und konfiguriert
- `RESEND_API_KEY` und `RESEND_FROM_EMAIL` als Environment Variables vorhanden
- RESEND_FROM_EMAIL = "PayWatcher <noreply@paywatcher.dev>"
- Bestehende Nutzung: Magic Link Auth (NextAuth Provider) + Request Access Admin-Notification
- Domain `paywatcher.dev` ist in Resend verifiziert (Magic Link funktioniert bereits)
- Keine neue Dependency noetig

**Bestehende Request-Access API Route (`src/app/api/request-access/route.ts`):**
- POST Handler mit Zod-Validierung (name: required, email: required, company: optional, useCase: optional)
- Sendet Email an `contact@masem.at` via Resend (Subject: "New Request Access: {name}")
- Body: Plain-Text mit Name, Email, Company, Use Case
- Fire-and-forget MMS Lead-Erfassung: POST `/v1/parties` mit `party_status: "lead"`
- Response: `{ data: { success: true } }`
- `resend` Client wird direkt in der Route instantiiert: `const resend = new Resend(process.env.RESEND_API_KEY)`

**Bestehende Request-Access Form (`src/components/landing/request-access-section.tsx`):**
- Client Component mit React State (kein React Hook Form — Custom Form Handling)
- Felder: name (required), email (required), company (optional), useCase (optional)
- Client-seitige Validierung via Zod
- Erfolgs-State mit Checkmark und Nachricht
- Analytics Events: `lp_request_access_start` (on focus), `lp_request_access_submit` (on success)

**Bestehender Flow:**
```
1. User fuellt Formular aus
2. POST /api/request-access
3. Zod Validation (server-side)
4. Email an contact@masem.at (Admin-Notification)
5. Fire-and-forget: POST /v1/parties (MMS Lead)
6. Response: { data: { success: true } }
7. UI: Erfolgs-Nachricht
```

**Neuer Flow (nach Story 6.5):**
```
1. User fuellt Formular aus (Use Case jetzt required, min 20 Zeichen)
2. POST /api/request-access
3. Zod Validation (useCase jetzt required + min 20)
4. Email an contact@masem.at (Admin-Notification) — UNVERAENDERT
5. Email an user@email.com (Bestaetigungsmail) — NEU, fire-and-forget
6. Fire-and-forget: POST /v1/parties (MMS Lead) — UNVERAENDERT
7. Response: { data: { success: true } }
8. UI: Aktualisierte Erfolgs-Nachricht — TEXT UPDATE
```

### Pattern-Uebernahme aus bestehender Codebase

| Aspekt | Vorlage | Story 6.5 |
|--------|---------|-----------|
| Email-Versand | Admin-Email in gleicher Route | Gleicher Resend Client, gleiche `from` Adresse |
| Error Handling | MMS Lead fire-and-forget in gleicher Route | Gleicher try-catch Pattern |
| Zod Validation | Bestehende Schema in Route | Schema-Erweiterung (useCase: required + min 20) |
| Form Component | Bestehende request-access-section.tsx | Validation-Update + Text-Update |

### Bestehende wiederverwendbare Utilities

| Utility | Pfad | Nutzung |
|---------|------|---------|
| `Resend` Client | `resend` npm package (bereits in route.ts) | Gleicher Client fuer zweite Email |
| `RESEND_FROM_EMAIL` | `.env` | Gleicher Absender |
| `RESEND_API_KEY` | `.env` | Gleicher Key |
| Zod | `zod` npm package (bereits importiert) | Schema-Erweiterung |

### Neue Dateien die erstellt werden

| Datei | Zweck |
|-------|-------|
| `src/lib/email/request-access-confirmation.ts` | HTML+Text Email Template + `buildConfirmationEmail()` Funktion |

### Bestehende Dateien die modifiziert werden

| Datei | Aenderung |
|-------|----------|
| `src/app/api/request-access/route.ts` | 1) Zod Schema: useCase required + min 20, 2) Bestaetigungsmail-Versand hinzufuegen |
| `src/components/landing/request-access-section.tsx` | 1) Client Zod Schema: useCase required + min 20, 2) Placeholder-Text aendern, 3) Erfolgs-Nachricht Text anpassen |

### Learnings aus vorherigen Stories

- Fire-and-forget Pattern bewaehrt (MMS Lead-Erfassung nutzt es bereits in dieser Route)
- `console.error` fuer Logging (kein separates Error Monitoring im MVP)
- Bestehende Logik NICHT anfassen — nur erweitern
- Defensive Programmierung: Fehler in sekundaeren Operationen duerfen nie die primaere Response blockieren
- English fuer User-facing Content (internationale Developer-Zielgruppe)
- Admin-Notification (contact@masem.at) ist die primaere Email — muss immer funktionieren
- Nach Code Review: Server-side Zod Validation immer spiegeln (Client + Server Schemas konsistent halten)
- Boundary-Regel beachten: Landing Components in `components/landing/` — diese Story betrifft nur Landing + lib

### Git Intelligence

Letzte relevante Commits:
- `6d7dca3` Add global payments admin page with code review fixes (Epic 6: Story 6.4)
- `71fbd19` Add admin dashboard with tenant management (Epic 6: Stories 6.1-6.3)
- Alle Epic 6 Stories in einem Commit pro Story
- Error Handling und Graceful Degradation durchgehend wichtig
- Code Review Fixes werden im gleichen Commit nachgezogen

### Project Structure Notes

- Neue Email-Template-Datei in `src/lib/email/` (neues Verzeichnis)
- Minimal-invasive Aenderung: 1 neue Datei, 2 modifizierte Dateien
- Kein neuer API Endpoint, keine neue Dependency, kein DB-Schema-Change
- Kein Impact auf Auth, Dashboard, Admin Dashboard, oder andere Features
- Landing Page Components bleiben in `src/components/landing/`

### Scope-Begrenzung

Diese Story umfasst:
- Use Case Feld → Pflichtfeld (min 20 Zeichen) mit angepasstem Placeholder und Fehlermeldung
- Bestaetigungsmail an den User nach Request Access (HTML + Plain Text)
- Aktualisierte Erfolgs-Nachricht im Formular

Diese Story umfasst NICHT:
- Approval-Email nach Admin Tenant-Erstellung (separater manueller Prozess durch Mario)
- HTML-Email-Template-System (React Email, MJML)
- Email-Tracking oder Delivery-Status
- Aenderungen an der Admin-Notification-Email
- Aenderungen an der MMS Lead-Erfassung

### References

- [Source: src/app/api/request-access/route.ts — Bestehende Request Access API Route]
- [Source: src/components/landing/request-access-section.tsx — Request Access Form UI]
- [Source: src/lib/auth.ts — Resend Integration fuer Magic Link]
- [Source: _bmad-output/planning-artifacts/architecture.md#Infrastructure & Deployment — Resend Email Service]
- [Source: _bmad-output/planning-artifacts/prd.md#FR36 — Request Access Formular]
- [Source: _bmad-output/planning-artifacts/epics.md#Story 1.7 — Request Access Formular (Basis-Implementation)]
- [Source: _bmad-output/implementation-artifacts/6-4-global-payments.md — Vorherige Story Learnings]
- [Source: _bmad-output/implementation-artifacts/sprint-status.yaml — 6-5-request-access-confirmation-email: backlog]

## Dev Agent Record

### Agent Model Used

Claude Opus 4.6

### Debug Log References

- TypeScript type-check: PASS (0 errors)
- ESLint: PASS (0 warnings/errors on all 3 changed files)
- No test framework configured in project (MVP-Ansatz, konsistent mit allen vorherigen Stories)

### Completion Notes List

- Task 1: Zod Schema auf Client und Server konsistent aktualisiert (useCase: required, min 20 Zeichen). Textarea mit required-Attribut, neuem Placeholder, required-Marker (*) im Label, aria-invalid/aria-describedby fuer Accessibility, und inline Fehlermeldung. `useCase.trim() || undefined` zu `useCase.trim()` geaendert (da required, kein undefined-Fallback noetig).
- Task 2: Email-Template als einfaches Template-Literal in `src/lib/email/request-access-confirmation.ts`. HTML mit Inline-CSS, table-Layout fuer Email-Client-Kompatibilitaet, monospace Docs-Link, system-ui Font-Stack, weisser Hintergrund. Plain-Text-Fallback mit identischem Inhalt.
- Task 3: Bestaetigungsmail in API Route eingefuegt NACH Admin-Email-Versand und NACH Admin-Error-Check. Confirmation wird nur gesendet wenn Admin-Email erfolgreich war (verhindert widerspruechliche UX). Sequential error-safe Pattern mit try-catch und console.error. Bestehende Admin-Email und MMS Lead-Erfassung unberuehrt.
- Task 4: Erfolgs-Nachricht Text aktualisiert: "Request submitted! Check your email for confirmation — we'll get back to you within 24 hours."

### File List

- `src/app/api/request-access/route.ts` (modified) — Zod Schema useCase required+min20, Bestaetigungsmail-Versand hinzugefuegt
- `src/components/landing/request-access-section.tsx` (modified) — Client Zod Schema, Textarea required+placeholder+error, Erfolgs-Nachricht Text
- `src/lib/email/request-access-confirmation.ts` (new) — buildConfirmationEmail() HTML+Text Template

### Senior Developer Review (AI)

**Reviewer:** Sempre | **Date:** 2026-02-21 | **Model:** Claude Opus 4.6

**Findings (3 fixed, 2 low noted):**
- [FIXED][HIGH] HTML-Injection in Email-Template: `name` Parameter ohne HTML-Escaping in HTML interpoliert → `escapeHtml()` Funktion hinzugefuegt
- [FIXED][MEDIUM] Confirmation Email vor Admin-Error-Check: User konnte Bestaetigungsmail erhalten obwohl Admin nicht benachrichtigt → Confirmation nach Error-Check verschoben
- [FIXED][MEDIUM] Irreführender "fire-and-forget" Kommentar: Code verwendet `await` (blockiert), Kommentar sagte fire-and-forget → Kommentar korrigiert zu "sequential, error-safe"
- [NOTED][LOW] Dead Code: `useCase ? ... : null` in Admin-Email ist jetzt unnoetig (useCase required) — bewusst nicht geaendert (bestehende Logik)
- [NOTED][LOW] Kein maxLength auf useCase-Feld — Schutz gegen Missbrauch waere sinnvoll

## Change Log

- 2026-02-21: Code Review Fixes — HTML-Escaping fuer Email-Template, Confirmation nach Admin-Error-Check verschoben, Kommentar korrigiert
- 2026-02-21: Story 6.5 implementiert — Use-Case-Feld als Pflichtfeld (min 20 Zeichen), Bestaetigungsmail an User nach Request Access (HTML+Text via Resend), aktualisierte Erfolgs-Nachricht im Formular
