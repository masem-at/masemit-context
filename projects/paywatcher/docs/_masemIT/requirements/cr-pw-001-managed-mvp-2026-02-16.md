# Change Request: PayWatcher Frontend — Managed MVP

**CR-ID:** CR-PW-001
**Date:** 2026-02-16
**Author:** Mario Semper (PO)
**Status:** Approved
**Betrifft:** product-brief-paywatcher-phase1-frontend-2026-02-15.md
**Auslöser:** Team-Review des MMS Backend Requirements → Entscheidung: MVP = Managed, kein Self-Service Signup

---

## 1. Zusammenfassung

PayWatcher MVP wechselt von Self-Service Signup zu Managed Onboarding. Tenants werden vom Admin (Mario) manuell angelegt. Das hat Auswirkungen auf Landing Page CTAs, Dashboard Auth-Flow, und API-Pfade. Dashboard-Funktionalität bleibt unverändert.

---

## 2. Begründung

Self-Service Provisioning erfordert ein neues Session-Auth-System im MMS Backend (Magic Link), dynamische Tenant-Generierung und Slug-Kollisionserkennung — ein eigener Epic, nicht gerechtfertigt bei <10 erwarteten Tenants im MVP. Details siehe: `requirement-mms-paywatcher-frontend-endpoints-2026-02-16-v2.md`, Sektion 2 (AD-1, AD-2, AD-3).

---

## 3. Änderungen

### 3.1 Landing Page (paywatcher.dev)

#### Ä-01: Hero CTA

| Vorher | Nachher |
|--------|---------|
| "Get your API Key" → Instant Self-Service Signup | "Request Access" → Kontaktformular oder mailto:contact@masem.at |

Der CTA führt NICHT mehr zu einem Signup-Flow, sondern zu einem kurzen Request-Access-Formular (Name, Email, Kurzbeschreibung des Use Case). Das Formular sendet eine Email an contact@masem.at. Kein Account wird automatisch erstellt.

**Alternativ akzeptabel:** Calendly-Link für ein kurzes Onboarding-Gespräch.

#### Ä-02: Pricing Section CTAs

| Vorher | Nachher |
|--------|---------|
| Free Tier: "Start Free" → Instant Signup | Free Tier: "Request Access" → gleiches Formular wie Hero |
| Paid Tiers: "Contact Us" | Paid Tiers: "Contact Us" (unverändert) |

#### Ä-03: Signup Analytics Events anpassen

| Vorher | Nachher |
|--------|---------|
| `lp_signup_start` | `lp_request_access_start` |
| `lp_signup_complete` | `lp_request_access_submit` |

Restliche Analytics Events bleiben unverändert (lp_view, lp_hero_cta, lp_pricing_view, lp_code_copy, etc.).

#### Ä-04: Neuer Abschnitt "Request Access"

Statt einer Signup-Page ein eingebettetes Formular (oder eigene /request-access Seite):

**Felder:**
- Name (required)
- Email (required)
- Company / Project Name (optional)
- Use Case — kurze Beschreibung (optional, Textarea)
- Submit Button: "Request Access"

**Nach Submit:** Bestätigungsmeldung: "Thanks! We'll get back to you within 24 hours with your API credentials."

**Backend:** Formular sendet Email via Resend an contact@masem.at. Kein MMS-Call nötig. Kein Account wird erstellt.

---

### 3.2 User Dashboard (app.paywatcher.dev)

#### Ä-05: Auth-Flow vereinfacht

| Vorher | Nachher |
|--------|---------|
| Signup + Login als getrennte Flows, Magic Link Auth gegen MMS | Nur Login. User gibt API Key ein ODER loggt sich per Magic Link ein (NextAuth, Frontend-seitig). |

**Empfohlener Ansatz:** API-Key-basierter Login. Der Tenant hat seinen Key von Mario erhalten. Er gibt ihn im Dashboard ein, das Frontend validiert gegen MMS (GET /v1/paywatcher/config), und speichert den Key in der Session.

**Alternativ:** Magic Link via NextAuth (Frontend-seitig, NICHT im MMS Backend). Setzt voraus, dass das Frontend eine eigene User-Tabelle pflegt die den API Key mappt. Entscheidung beim Architecture Doc des Frontend-Projekts.

#### Ä-06: /signup entfällt

| Vorher | Nachher |
|--------|---------|
| /login und /signup als getrennte Seiten (zusammengelegt zu /login) | Nur /login. Keine /signup Seite. |

/signup → Redirect auf /login. Auf /login nur "Sign in to PayWatcher" mit API Key Eingabe oder Magic Link.

#### Ä-07: Onboarding-Screen nach erstem Login

Neuer optionaler Screen nach dem ersten erfolgreichen Login:

**"Welcome to PayWatcher"**
- Zeigt: API Key (masked), Deposit Address, Webhook URL (nicht konfiguriert)
- CTA: "Configure Webhook" → Settings Seite
- CTA: "View API Docs" → Docs

Dieser Screen ersetzt den ehemaligen "Your API Key"-Moment aus dem Self-Service Signup.

---

### 3.3 API-Pfade

#### Ä-08: Endpoint-Pfade aktualisiert

Die MMS Backend Endpoints haben sich gegenüber den impliziten Annahmen im Product Brief geändert:

| Funktion | Neuer Pfad | Auth |
|----------|-----------|------|
| Config lesen | GET /v1/paywatcher/config | API Key (payments:read) |
| Config updaten | PATCH /v1/paywatcher/config | API Key (payments:write) |
| Test Webhook | POST /v1/paywatcher/config/test-webhook | API Key (payments:write) |
| API Key anzeigen | GET /v1/paywatcher/api-key | API Key (payments:read) |
| Payment-Liste | GET /v1/payments | API Key (payments:read) |
| Payment Detail | GET /v1/payments/:id | API Key (payments:read) |
| Webhook-Historie | GET /v1/payments/:id/webhooks | API Key (payments:read) |

**Admin Endpoints:**

| Funktion | Pfad | Auth |
|----------|------|------|
| Tenant anlegen | POST /v1/admin/paywatcher/tenants | API Key (admin:write) |
| Tenants listen | GET /v1/admin/paywatcher/tenants | API Key (admin:read) |
| Tenant updaten | PATCH /v1/admin/paywatcher/tenants/:slug | API Key (admin:write) |
| Globale Payments | GET /v1/admin/paywatcher/payments | API Key (admin:read) |
| System Health | GET /v1/admin/paywatcher/health | API Key (admin:read) |

---

### 3.4 Admin Dashboard

#### Ä-09: Tenant-Anlage als Admin-Feature

Das Admin Dashboard bekommt eine "New Tenant" Funktion:

**Formular:**
- tenant_slug (required, auto-generated from name, editable)
- tenant_name (required)
- contact_email (required)
- tier (Dropdown: free, starter, pro, enterprise)
- deposit_address (pre-filled with shared address)
- webhook_url (optional)
- confirmation_override (optional, 2–20)

**Nach Erstellung:** Modal/Screen zeigt einmalig:
- API Key im Klartext (mit Copy-Button)
- Webhook Secret im Klartext (mit Copy-Button)
- Warnung: "This is the only time these credentials will be shown."

---

## 4. Keine Änderung

Folgende Aspekte aus dem Product Brief bleiben **unverändert:**

- Landing Page Sektionen 2–7 (Problem/Solution, How It Works, Code Examples, Pricing-Tabelle, Comparison, Trust Signals)
- Dashboard Pages: Overview, Payments, Settings (Funktionalität identisch)
- Admin Dashboard: System Overview, Global Payments, Webhooks, System/Config
- Tech Stack: Next.js 15, Tailwind, shadcn/ui, Vercel Pro
- Design: Dark Mode, Developer-Ästhetik, masemIT Brand Colors
- Analytics Events (außer Ä-03)
- Success Metrics (außer Signup-Conversion → Request-Access-Conversion)

---

## 5. Angepasste Success Metrics

| Metrik | Vorher | Nachher |
|--------|--------|---------|
| Landing Page → Signup Conversion | >5% | n/a (kein Self-Service) |
| **Landing Page → Request Access** | — | **>3% (neues Ziel)** |
| **Request → Onboarded** | — | **>50% (Mario onboardet manuell)** |
| Active Tenants (3 Monate) | 10 Free + 3 Paid | **5 Free + 2 Paid (realistischer bei Managed)** |
| Verified Payments | 100+ | 100+ (unverändert) |
| MRR | €87+ | €87+ (unverändert) |

---

## 6. Referenz-Dokumente

| Dokument | Relevanz |
|----------|---------|
| product-brief-paywatcher-phase1-frontend-2026-02-15.md | Basis-Dokument, das durch diesen CR aktualisiert wird |
| requirement-mms-paywatcher-frontend-endpoints-2026-02-16-v2.md | MMS Backend Requirement mit neuen Endpoint-Pfaden |
| paywatcher-architecture.md | Backend-Architektur (unverändert) |

---

*Dieser Change Request ist vom PO approved. Bitte in das Product Brief und die relevanten Epic/Story-Definitionen einarbeiten.*
