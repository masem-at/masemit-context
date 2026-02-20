# Story 2.2: Magic Link Signup & Login

Status: done

<!-- Note: Validation is optional. Run validate-create-story for quality check before dev-story. -->

## Story

As a Besucher,
I want mich mit meiner Email-Adresse registrieren und per Magic Link einloggen koennen,
so that ich in unter 2 Minuten einen Account habe ohne Passwort-Verwaltung.

## Acceptance Criteria

1. **Given** ich bin auf der Signup-Seite (FR8) **When** ich meine Email-Adresse eingebe und Submit klicke **Then** wird ein MMS Parties API Call ausgefuehrt (POST /v1/parties mit source_project: "paywatcher") **And** ein Magic Link wird per Resend an meine Email gesendet **And** ich sehe einen "Check your inbox"-Screen mit Hinweis auf Spam-Folder

2. **Given** ich klicke den Magic Link in meiner Email (FR9) **When** der Token verifiziert wird **Then** wird eine JWT Session erstellt (mit party_id, email, displayName) **And** ich werde zum Dashboard redirected

3. **Given** ich bin auf der Login-Seite **When** ich meine Email eingebe **Then** erhalte ich einen neuen Magic Link (gleicher Flow wie Signup — find-or-create)

4. **Given** der Magic Link aelter als 15 Minuten ist **When** ich ihn klicke **Then** sehe ich "Link expired. Enter your email again." mit Option einen neuen Link anzufordern

5. **Given** ich nach 30 Sekunden keine Email erhalten habe **When** ich den "Didn't receive it?"-Bereich sehe **Then** kann ich einen Resend-Button klicken um einen neuen Magic Link anzufordern

6. **Given** ich eingeloggt bin (FR10) **When** ich auf Logout klicke **Then** wird die Auth.js Session beendet und ich werde zur Landing Page redirected

7. **Given** der gesamte Signup-Flow gemessen wird (NFR3) **When** ein User sich registriert **Then** dauert der Flow von Email-Eingabe bis Dashboard unter 5 Sekunden (exkl. Email-Zustellung)

## Tasks / Subtasks

- [x] Task 1: Resend Package installieren und Magic Link Token System (AC: #1, #4)
  - [x] 1.1 `resend` Package installieren (`npm install resend`)
  - [x] 1.2 `src/lib/magic-link.ts` erstellen — Token-Generierung mit `jose` (SignJWT, 15min Expiry, HS256, email als Claim), Token-Verifizierung (jwtVerify), Email-Versand via Resend SDK
  - [x] 1.3 Email-Template: minimalistisch, PayWatcher-Branding, Magic Link Button, 15min Hinweis, "If you didn't request this"-Text
  - [x] 1.4 `RESEND_FROM_EMAIL` Variable in `.env.example` dokumentieren (z.B. `noreply@paywatcher.dev`)

- [x] Task 2: Auth.js Credentials Provider mit MMS Parties API (AC: #1, #2)
  - [x] 2.1 `src/lib/auth.ts` erweitern — bestehenden Provider-Stub ersetzen durch `Credentials` Provider mit id "magic-link"
  - [x] 2.2 `authorize()` Callback: Token verifizieren via `verifyMagicLinkToken()`, dann `mmsApi()` POST `/v1/parties` mit `{ email, source_project: "paywatcher" }` (find-or-create), User-Objekt mit party_id, email, displayName zurueckgeben
  - [x] 2.3 Bestehende JWT/Session Callbacks beibehalten (partyId, email, displayName)
  - [x] 2.4 `src/types/next-auth.d.ts` pruefen — Credentials authorize Rueckgabe muss mit User-Typ kompatibel sein

- [x] Task 3: API Routes fuer Magic Link Flow (AC: #1, #2, #4, #5)
  - [x] 3.1 `src/app/api/auth/request-magic-link/route.ts` erstellen — POST, Email validieren (Zod), Token generieren, MMS Party find-or-create, Magic Link Email senden via Resend, Rate Limiting (max 1 Request pro Email pro 60s via simple In-Memory Map)
  - [x] 3.2 `src/app/api/auth/verify/route.ts` erstellen — GET mit `?token=xxx`, Token verifizieren, bei Erfolg `signIn("magic-link", { token, redirect: true, redirectTo: "/dashboard" })`, bei Fehler Redirect zu `/login?error=ExpiredToken`
  - [x] 3.3 Error Handling: abgelaufene Tokens (15min), ungueltige Tokens, MMS API Fehler — alle mit klaren Fehlermeldungen

- [x] Task 4: Login-Seite erstellen (AC: #3, #4, #5)
  - [x] 4.1 `src/app/(landing)/login/page.tsx` erstellen — Server Component mit Metadata (title: "Login — PayWatcher")
  - [x] 4.2 `src/components/landing/auth-form.tsx` erstellen — Client Component, geteiltes Email-Formular fuer Login UND Signup (gleicher Flow), Email-Input + Submit-Button ("Send Magic Link"), "Check your inbox"-State nach Submit, "Didn't receive it?"-Resend-Button (30s Cooldown), Error-States (expired token via URL Query Param), Loading-State auf Submit-Button
  - [x] 4.3 Minimalistisches Layout: max-w-md zentriert, PayWatcher Logo oben, Dark-Mode kompatibel, kein ueberfluessiges UI

- [x] Task 5: Signup-Seite erstellen (AC: #1, #5)
  - [x] 5.1 `src/app/(landing)/signup/page.tsx` erstellen — Server Component mit Metadata (title: "Get API Key — PayWatcher")
  - [x] 5.2 Gleiche `auth-form.tsx` Komponente verwenden wie Login (mode Prop: "login" | "signup")
  - [x] 5.3 Signup-Variante: Headline "Get your API Key", Subline "Enter your email to create a free account. No credit card required."
  - [x] 5.4 Login-Variante: Headline "Welcome back", Subline "Enter your email to sign in."
  - [x] 5.5 Link zwischen Login und Signup ("Already have an account? Sign in" / "Don't have an account? Get started")

- [x] Task 6: Logout-Funktionalitaet (AC: #6)
  - [x] 6.1 Logout-Action in Dashboard Header/Sidebar integrieren — `signOut({ redirectTo: "/" })` aufrufen
  - [x] 6.2 Bestehende `sidebar-nav.tsx` oder `dashboard-header.tsx` erweitern um Logout-Button/-Option

- [x] Task 7: Build-Validierung und Qualitaetspruefung (Alle ACs)
  - [x] 7.1 `npm run build` muss fehlerfrei durchlaufen
  - [x] 7.2 `npm run type-check` muss fehlerfrei durchlaufen
  - [x] 7.3 `npm run lint` muss fehlerfrei durchlaufen
  - [x] 7.4 Verifizieren: Landing Page bleibt `○ (Static)` im Build Output
  - [x] 7.5 Verifizieren: Login/Signup-Seiten rendern korrekt
  - [x] 7.6 Verifizieren: Dashboard-Redirect bei fehlender Session funktioniert (-> /login)
  - [x] 7.7 Verifizieren: Magic Link Flow end-to-end (Token generieren, verifizieren, Session erstellen)

## Dev Notes

### Kontext & Zweck

Dies ist die **zweite Story von Epic 2** (User Authentication & API Key Management). Story 2.1 hat die Dashboard Shell, Auth.js v5 Stub, Route Protection, MMS API Client und TanStack Query geliefert. Diese Story macht das Auth-System funktional.

**Was PayWatcher ist:** Developer-First Verification API fuer Stablecoin-Zahlungen (USDC auf Base Network). Non-custodial, reiner Verification Layer. Das Frontend ist ein reiner API-Consumer — kein eigenes Backend, keine eigene Datenbank. Alle Daten kommen vom MMS Backend (api.masem.at).

**Was diese Story liefert:**
- Funktionalen Magic Link Signup/Login Flow
- Login-Seite (`/login`) und Signup-Seite (`/signup`)
- Custom Magic Link Token System (stateless JWT, kein DB-Adapter noetig)
- Resend Email Integration fuer Magic Link Versand
- MMS Parties API Integration (find-or-create bei Signup/Login)
- Logout-Funktionalitaet
- Geteilte Auth-Form-Komponente fuer Login und Signup

**Was diese Story NICHT beinhaltet:**
- API Key Erstellung und Anzeige (Story 2.3)
- API Key Regenerate und Scopes (Story 2.4)
- paywatcher_config Eintrag im MMS Backend (Story 2.3)
- Onboarding Screen nach erstem Login (Story 3.2)
- Form-Validation mit React Hook Form + Zod (nur einfache Zod-Validation in API Route)

**Abhaengigkeiten:**
- Story 2.1 komplett abgeschlossen (Dashboard Shell, Auth.js Stub, MMS API Client, proxy.ts)
- MMS Backend (api.masem.at) muss erreichbar sein mit gueltigem MMS_API_KEY
- Resend Account mit gueltigem API Key und verifizierter Domain (paywatcher.dev oder Resend-Subdomain)

### KRITISCH: Auth.js Email Provider NICHT verwendbar!

**Auth.js v5 Email Provider (Resend Provider) erfordert ZWINGEND einen Datenbank-Adapter:**
> "It is not possible to enable an email provider without using a database. Auth.js will throw an error."

PayWatcher hat KEINE eigene Datenbank. Alle Daten kommen vom MMS Backend.

**Loesung: Custom Magic Link mit Credentials Provider**

Statt `next-auth/providers/resend` verwenden wir `next-auth/providers/credentials` mit eigenem Magic Link Flow:

1. **Token-Generierung:** Stateless JWT via `jose` Library (bereits von next-auth als Dependency vorhanden)
2. **Token-Speicherung:** KEINE — der Token ist self-contained (Email + Expiry als JWT Claims)
3. **Email-Versand:** Resend SDK direkt (nicht via Auth.js Provider)
4. **Token-Verifikation:** JWT Verify im API Route Handler
5. **Session-Erstellung:** Credentials Provider `authorize()` → Auth.js JWT Session

**Warum stateless JWT statt Redis/Upstash:**
- Keine externe Dependency (kein Upstash Redis noetig)
- Skaliert ohne Limit auf Vercel Serverless
- 15-Minuten Expiry reicht als Sicherheit (Token kann nicht widerrufen werden, aber kurze Lebensdauer ist akzeptabel)
- `jose` Library ist bereits als Transitive Dependency von `next-auth` vorhanden
- Einziger Nachteil: Token ist nicht single-use (kann theoretisch innerhalb 15min mehrmals verwendet werden) — akzeptabler Tradeoff fuer MVP

### Technische Anforderungen

**Zu installierende Packages:**
- `resend` — Email Service SDK fuer Magic Link Versand

**Bereits installierte und verwendbare Packages:**
- `next-auth@beta` v5.0.0-beta.30 (Auth.js v5) — aus Story 2.1
- `jose` — Transitive Dependency von next-auth, fuer JWT Token-Generierung/-Verifikation
- `@tanstack/react-query` v5 — fuer spaetere Dashboard-Nutzung (nicht in Auth-Flow)
- Alle shadcn/ui Komponenten, Tailwind v4, next-themes, lucide-react, sonner

**NICHT in dieser Story installieren:**
- `@upstash/redis` — nicht noetig dank stateless JWT
- `react-hook-form` / `zod` (als Package) — einfache native Form + Zod nur in API Route (Zod ist via next bereits verfuegbar ODER als dev-dependency)
- `@auth/upstash-redis-adapter` — kein DB-Adapter noetig

**ACHTUNG:** Pruefen ob `zod` bereits installiert ist (oft Transitive Dependency). Falls nicht, `zod` installieren fuer Email-Validierung in der API Route.

### Auth Flow Architektur

```
┌─────────────────────────────────────────────────────┐
│  Browser                                             │
│                                                      │
│  /signup oder /login                                │
│  ┌──────────────────────┐                           │
│  │ AuthForm Component   │                           │
│  │ Email Input + Submit │                           │
│  └──────────┬───────────┘                           │
│             │ POST /api/auth/request-magic-link     │
└─────────────┼───────────────────────────────────────┘
              │
┌─────────────┼───────────────────────────────────────┐
│  Next.js Server (Vercel)                             │
│             ▼                                        │
│  ┌──────────────────────────┐                       │
│  │ request-magic-link Route │                       │
│  │ 1. Validate Email (Zod) │                       │
│  │ 2. Generate JWT Token   │ ← jose SignJWT        │
│  │ 3. Send Email           │ → Resend API          │
│  └──────────────────────────┘                       │
│                                                      │
│  User klickt Magic Link in Email                    │
│  GET /api/auth/verify?token=xxx                     │
│             ▼                                        │
│  ┌──────────────────────────┐                       │
│  │ verify Route             │                       │
│  │ 1. Verify JWT Token     │ ← jose jwtVerify      │
│  │ 2. signIn("magic-link") │ → Auth.js             │
│  └──────────┬───────────────┘                       │
│             │                                        │
│  ┌──────────▼───────────────┐                       │
│  │ Credentials Provider     │                       │
│  │ authorize() Callback     │                       │
│  │ 1. Verify Token (erneut) │                       │
│  │ 2. POST /v1/parties      │ → MMS Backend         │
│  │    (find-or-create)      │                       │
│  │ 3. Return User Object    │                       │
│  └──────────┬───────────────┘                       │
│             │                                        │
│  JWT Session erstellt (Cookie)                      │
│  Redirect → /dashboard                              │
└─────────────────────────────────────────────────────┘
```

### Magic Link Token Implementierung

```typescript
// src/lib/magic-link.ts
import { SignJWT, jwtVerify } from 'jose'
import { Resend } from 'resend'

const secret = new TextEncoder().encode(process.env.AUTH_SECRET)
const resend = new Resend(process.env.RESEND_API_KEY)

export async function generateMagicLinkToken(email: string): Promise<string> {
  return new SignJWT({ email })
    .setProtectedHeader({ alg: 'HS256' })
    .setIssuedAt()
    .setExpirationTime('15m')
    .setJti(crypto.randomUUID())
    .sign(secret)
}

export async function verifyMagicLinkToken(token: string): Promise<string | null> {
  try {
    const { payload } = await jwtVerify(token, secret)
    return payload.email as string
  } catch {
    return null // expired or invalid
  }
}

export async function sendMagicLinkEmail(email: string, token: string): Promise<void> {
  const magicLink = `${process.env.AUTH_URL}/api/auth/verify?token=${token}`

  await resend.emails.send({
    from: process.env.RESEND_FROM_EMAIL || 'PayWatcher <noreply@paywatcher.dev>',
    to: email,
    subject: 'Sign in to PayWatcher',
    html: `<!-- Minimalistisches Email Template -->`,
  })
}
```

### Auth.js Credentials Provider Pattern

```typescript
// src/lib/auth.ts — ERWEITERUNG des bestehenden Stubs
import NextAuth from "next-auth"
import Credentials from "next-auth/providers/credentials"
import { verifyMagicLinkToken } from "./magic-link"
import { mmsApi } from "./api/client"
import { snakeToCamel } from "./api/transform"

export const { auth, handlers, signIn, signOut } = NextAuth({
  providers: [
    Credentials({
      id: "magic-link",
      name: "Magic Link",
      credentials: {
        token: { type: "text" },
      },
      async authorize(credentials) {
        const token = credentials?.token as string | undefined
        if (!token) return null

        // 1. Verify magic link JWT
        const email = await verifyMagicLinkToken(token)
        if (!email) return null

        // 2. Find-or-create party in MMS Backend
        const response = await mmsApi<{ data: { party_id: string; email: string; display_name?: string } }>(
          '/v1/parties',
          {
            method: 'POST',
            body: JSON.stringify({
              email,
              source_project: 'paywatcher',
            }),
          }
        )

        const party = snakeToCamel(response.data)

        return {
          id: party.partyId,
          email: party.email,
          partyId: party.partyId,
          displayName: party.displayName,
        }
      },
    }),
  ],
  session: { strategy: "jwt" },
  callbacks: {
    // ... bestehende Callbacks beibehalten
  },
  pages: {
    signIn: "/login",
  },
})
```

### WICHTIG: Doppelte Token-Verifikation

Der Token wird ZWEIMAL verifiziert:
1. In `/api/auth/verify` Route — um bei ungueltigem Token sofort zu /login zu redirecten
2. In `authorize()` Callback — als Sicherheitsnetz, da Credentials Provider den Token erneut erhaelt

Das ist gewollt und sicher: JWT-Verifikation ist idempotent und schnell.

### MMS Parties API Vertrag

```
POST /v1/parties
Headers: x-api-key: {MMS_API_KEY}, Content-Type: application/json
Body: { "email": "user@example.com", "source_project": "paywatcher" }

Response (200 — existing party found):
{ "data": { "party_id": "uuid", "email": "user@example.com", "display_name": "..." } }

Response (201 — new party created):
{ "data": { "party_id": "uuid", "email": "user@example.com", "display_name": null } }
```

**find-or-create Semantik:** Gleicher Endpoint fuer Signup und Login. Wenn Party mit Email existiert, wird sie zurueckgegeben. Wenn nicht, wird sie erstellt. Kein separater Signup/Login-Unterschied im Backend.

### Login/Signup Seiten UX

**Design-Prinzipien (aus UX-Spec):**
- Minimalistisch — Email-Feld + Button, sonst nichts
- Kein Onboarding-Wizard, kein "Tell us about your company"
- Dark Mode kompatibel (erbt Theme vom Root Layout)
- max-w-md zentriert, PayWatcher Logo oben
- Developer-Audience: "Respekt vor meiner Zeit"

**Zustands-Maschine der AuthForm:**
```
idle → submitting → check-inbox → (User klickt Link)
                  ↓
                error (API Fehler)

check-inbox → resending → check-inbox (nach Resend)
            ↓
          (30s Cooldown fuer Resend-Button)
```

**URL Query Parameter:**
- `/login?error=ExpiredToken` → Zeigt "Link expired" Nachricht
- `/login?error=InvalidToken` → Zeigt "Invalid link" Nachricht
- `/signup` → Signup-Variante der gleichen Form

**Kein separater "Check your inbox" Screen** — der State wechselt inline in der Form:
1. Email-Input + "Send Magic Link" Button
2. Nach Submit: Gruenes Checkmark + "Check your inbox" + Email-Adresse + "Didn't receive it?" Link (nach 30s)

### Logout Integration

**Logout-Button Platzierung:**
- In der bestehenden `sidebar-nav.tsx` im Footer-Bereich (neben User Email)
- ODER im `dashboard-header.tsx` als Dropdown-Option
- `signOut({ redirectTo: "/" })` — redirect zur Landing Page nach Logout
- signOut ist eine Server Action (Auth.js v5) — muss als form action oder via fetch aufgerufen werden

### Email Template

Minimalistisch, kein HTML-Heavy Design:
```
Subject: Sign in to PayWatcher

---
PayWatcher

Click below to sign in:

[Sign in to PayWatcher]  ← Button/Link

This link expires in 15 minutes.
If you didn't request this, you can safely ignore this email.
---
```

- Kein Logo-Bild (erhoht Spam-Risiko)
- Kein Footer mit Social Links
- Plain und serioes — passend zur Developer-Audience

### Responsive Design

**Login/Signup-Seiten:**
- Mobile (<1024px): Volle Breite mit padding, zentrierter Content
- Desktop (>=1024px): max-w-md zentriert
- Touch Targets min. 44x44px (Submit Button, Resend Link)

### Rate Limiting Strategie

**Einfaches In-Memory Rate Limiting fuer request-magic-link:**
```typescript
const rateLimitMap = new Map<string, number>()

function isRateLimited(email: string): boolean {
  const lastRequest = rateLimitMap.get(email)
  if (lastRequest && Date.now() - lastRequest < 60_000) return true
  rateLimitMap.set(email, Date.now())
  return false
}
```

**ACHTUNG:** In-Memory Map wird bei Serverless Cold Start zurueckgesetzt. Fuer MVP akzeptabel — verhindert Rapid-Fire-Requests innerhalb einer Serverless Instance. Fuer Production spaeter durch Upstash Redis oder Vercel KV ersetzen.

### Architektur-Konformitaet

**Dateinamen:** `kebab-case.ts` / `kebab-case.tsx` — IMMER.

**Neue/geaenderte Dateien nach Abschluss:**
```
src/
├── app/
│   ├── (landing)/
│   │   ├── login/
│   │   │   └── page.tsx                    # NEU: Login Seite
│   │   └── signup/
│   │       └── page.tsx                    # NEU: Signup Seite
│   └── api/
│       └── auth/
│           ├── [...nextauth]/
│           │   └── route.ts                # BESTEHEND (unveraendert)
│           ├── request-magic-link/
│           │   └── route.ts                # NEU: Magic Link anfordern
│           └── verify/
│               └── route.ts                # NEU: Magic Link verifizieren
├── components/
│   └── landing/
│       └── auth-form.tsx                   # NEU: Geteilte Auth-Form Komponente
├── lib/
│   ├── auth.ts                             # AENDERN: Credentials Provider + MMS Integration
│   └── magic-link.ts                       # NEU: Token-Generierung, Verifikation, Email-Versand
.env.example                                 # AENDERN: RESEND_FROM_EMAIL hinzufuegen
```

**Server vs Client Components:**
- Server Components: `login/page.tsx`, `signup/page.tsx`, `request-magic-link/route.ts`, `verify/route.ts`, `magic-link.ts`, `auth.ts`
- Client Components: `auth-form.tsx` (braucht useState fuer Form-State, useSearchParams fuer Error Query Params)

**ANTI-Patterns:**
- KEIN `next-auth/providers/resend` — erfordert DB-Adapter, PayWatcher hat keine DB
- KEIN `@auth/upstash-redis-adapter` — unnoetige Komplexitaet fuer MVP
- KEIN `react-hook-form` in dieser Story — einfache native Form reicht fuer ein Email-Feld
- KEIN Full-Page-Spinner auf Login/Signup — Loading-State nur auf dem Button
- KEINE Passwort-Felder — rein Magic Link, kein Passwort
- KEIN OAuth (Google, GitHub) — nicht im Scope, Magic Link only
- KEINE Resend-Calls aus dem Browser — alles Server-Side via API Route

### Vorherige Story Intelligence (Story 2.1)

**Aus Story 2.1 uebernommene Patterns:**
- `mmsApi()` in `lib/api/client.ts` — fuer MMS Parties API Call
- `snakeToCamel()` in `lib/api/transform.ts` — fuer Party Response Transformation
- Auth.js v5 JWT Callbacks — partyId, email, displayName bereits definiert
- `proxy.ts` — schuetzt /dashboard Routen, redirect zu /login
- `sidebar-nav.tsx` — Footer mit User Email, hier Logout hinzufuegen
- shadcn/ui Sheet Pattern — Mobile Nav als Referenz

**Story 2.1 Learnings:**
- `AUTH_SECRET` und `AUTH_URL` Environment Variables (nicht NEXTAUTH_*)
- `auth()` ist die universelle Funktion (Server Components, Route Handlers, Proxy)
- `signIn` und `signOut` sind Server Actions
- proxy.ts Pattern (Next.js 16, nicht middleware.ts)
- Component Boundaries: Landing Components in `components/landing/`

**Story 2.1 Debug Learnings:**
- JWT Callback assigned `unknown` to typed fields → proper type narrowing noetig
- `setState` inside `useEffect` → lazy `useState` initializer bevorzugen
- `.env.local` hat `NEXTAUTH_SECRET` statt `AUTH_SECRET` — muss korrigiert werden fuer Auth.js v5

**Git Intelligence (letzte Commits):**
```
Story 2.1: Dashboard Shell, Auth.js v5, Route Protection, MMS API Client
Story 1.6: SEO, social sharing, analytics integration
Story 1.5: Pricing section, comparison callout, trust signals
```

### Aktuelle Technologie-Informationen (Web Research, Feb 2026)

**Auth.js v5 + Credentials Provider:**
- Credentials Provider funktioniert MIT JWT Strategy OHNE DB-Adapter ✅
- `authorize()` Callback erhaelt credentials Object und gibt User | null zurueck
- User-Object wird in JWT Callback als `user` Parameter verfuegbar
- `signIn("magic-link", { token, redirectTo: "/dashboard" })` fuer programmatischen Login
- WICHTIG: `signIn()` in Route Handlers muss ggf. als redirect ausgefuehrt werden

**jose Library (JWT):**
- Bereits als Transitive Dependency von next-auth vorhanden
- `SignJWT` fuer Token-Erstellung, `jwtVerify` fuer Verifikation
- Import: `import { SignJWT, jwtVerify } from 'jose'`
- Nutzt Web Crypto API (funktioniert in Node.js und Edge Runtime)

**Resend SDK:**
- Latest stable version installierbar via `npm install resend`
- Rate Limit: 2 Requests/Sekunde pro Team
- `resend.emails.send()` fuer Email-Versand
- Domain muss in Resend verifiziert sein (paywatcher.dev oder Resend-Subdomain fuer Dev)

**ACHTUNG: .env.local Korrektur noetig:**
Aus Story 2.1 Debug Log: `.env.local` verwendet noch `NEXTAUTH_SECRET` statt `AUTH_SECRET`. Auth.js v5 erwartet `AUTH_SECRET`. Pruefen und ggf. korrigieren.

### Testing-Anforderungen

1. **Build-Validierung:** `npm run build` muss ohne Fehler durchlaufen
2. **Type-Check:** `npm run type-check` muss ohne Fehler durchlaufen
3. **Lint:** `npm run lint` muss ohne Fehler durchlaufen
4. **SSG-Validierung:** Landing Page muss `○ (Static)` bleiben
5. **Login-Seite:** `/login` rendert Email-Form korrekt
6. **Signup-Seite:** `/signup` rendert Email-Form mit Signup-Variante korrekt
7. **Magic Link Token:** Token-Generierung und Verifikation funktioniert (Unit Test oder manuell)
8. **Expired Token:** Nach 15 Minuten wird Token abgelehnt
9. **MMS Party Integration:** `mmsApi()` Call in authorize() ist korrekt typisiert
10. **Logout:** signOut redirected zur Landing Page
11. **Dashboard-Redirect:** Unauthentifizierte User werden von /dashboard zu /login redirected (bestehendes Verhalten aus Story 2.1)

### Project Structure Notes

- Login/Signup-Seiten liegen unter `(landing)` Route Group — oeffentlich, kein Auth noetig
- Auth-Form ist `components/landing/auth-form.tsx` — gehoert zum Landing-Bereich, nicht Dashboard
- Magic Link Token System in `lib/magic-link.ts` — Server-Side only, nie im Client
- API Routes unter `api/auth/` neben den bestehenden `[...nextauth]` Routes
- Die bestehende Route `api/auth/[...nextauth]/route.ts` bleibt unveraendert — Auth.js Handler

### References

- [Source: _bmad-output/planning-artifacts/epics.md#Story 2.2] — User Story und Acceptance Criteria
- [Source: _bmad-output/planning-artifacts/prd.md#FR8-FR10] — Signup, Login, Logout Requirements
- [Source: _bmad-output/planning-artifacts/prd.md#NFR3] — Signup < 5s Performance
- [Source: _bmad-output/planning-artifacts/architecture.md#Auth Pattern] — Auth.js v5, JWT Strategy, Magic Link, MMS Parties API
- [Source: _bmad-output/planning-artifacts/architecture.md#API Client Pattern] — mmsApi(), Route Handler Format
- [Source: _bmad-output/planning-artifacts/architecture.md#Naming Patterns] — kebab-case Dateien, PascalCase Components
- [Source: _bmad-output/planning-artifacts/ux-design-specification.md#Onboarding] — "60s to API Key", kein Wizard
- [Source: _bmad-output/planning-artifacts/ux-design-specification.md#Emotional Journey] — Signup: "Respekt vor meiner Zeit"
- [Source: _bmad-output/planning-artifacts/ux-design-specification.md#Experience Principles] — "Null Friction bis zum Beweis"
- [Source: _bmad-output/implementation-artifacts/2-1-dashboard-shell-route-protection.md] — Vorherige Story Intelligence, Auth.js Stub, MMS Client
- [Source: authjs.dev/getting-started/authentication/email] — Email Provider erfordert DB-Adapter
- [Source: authjs.dev/getting-started/authentication/credentials] — Credentials Provider Pattern

## Dev Agent Record

### Agent Model Used

Claude Opus 4.6

### Debug Log References

- Lint-Fehler: `setState` in `useEffect` fuer Error-Query-Params → Refactored zu lazy `useState` Initializer
- Build-Fehler: `useSearchParams()` ohne Suspense Boundary → Login/Signup Seiten mit `<Suspense>` gewrapped

### Completion Notes List

- Task 1: `resend` Package installiert, `magic-link.ts` mit stateless JWT Token-System (jose SignJWT/jwtVerify), minimalistisches HTML Email-Template, `RESEND_FROM_EMAIL` in `.env.example` dokumentiert
- Task 2: Auth.js Credentials Provider mit id "magic-link" implementiert, `authorize()` verifiziert Token und ruft MMS Parties API (find-or-create) auf, bestehende JWT/Session Callbacks beibehalten, TypeScript-Typen kompatibel
- Task 3: `request-magic-link` Route mit Zod Email-Validierung und In-Memory Rate Limiting (60s pro Email), `verify` Route mit Token-Verifikation und signIn-Redirect, klare Error-Handling-Muster
- Task 4: Login-Seite mit Server Component + Metadata, AuthForm Client Component mit State Machine (idle/submitting/check-inbox/resending/error), 30s Resend-Cooldown, Error-States via URL Query Params
- Task 5: Signup-Seite nutzt gleiche AuthForm mit mode="signup", unterschiedliche Headlines/Sublines, Links zwischen Login und Signup
- Task 6: Logout via Server Action in `(dashboard)/actions.ts`, integriert in Desktop-Sidebar und Mobile-Sidebar mit LogOut-Icon
- Task 7: Build, Type-Check und Lint fehlerfrei, Landing Page bleibt Static, alle neuen Routes korrekt registriert

### Change Log

- 2026-02-16: Story 2.2 implementiert — Magic Link Signup/Login, Auth.js Credentials Provider, MMS Parties API Integration, Login/Signup Seiten, Logout-Funktionalitaet
- 2026-02-16: Code Review (AI) — 3 HIGH + 5 MEDIUM Issues gefixed: Error-Handling in verify/authorize, zod als explizite Dependency, Rate-Limit Memory Leak + Case-Normalisierung, Architecture Envelope Error Format, shadcn/ui Input, Brand-Farbe in Email-Template

### File List

- src/lib/magic-link.ts (NEU) — Token-Generierung, Verifikation, Email-Versand
- src/lib/auth.ts (GEAENDERT) — Credentials Provider mit MMS Parties API
- src/app/api/auth/request-magic-link/route.ts (NEU) — Magic Link anfordern
- src/app/api/auth/verify/route.ts (NEU) — Magic Link verifizieren
- src/app/(landing)/login/page.tsx (NEU) — Login Seite
- src/app/(landing)/signup/page.tsx (NEU) — Signup Seite
- src/components/landing/auth-form.tsx (NEU) — Geteilte Auth-Form Komponente
- src/app/(dashboard)/actions.ts (NEU) — Logout Server Action
- src/components/dashboard/sidebar-nav.tsx (GEAENDERT) — Logout-Button hinzugefuegt
- src/components/dashboard/mobile-sidebar.tsx (GEAENDERT) — Logout-Button hinzugefuegt
- .env.example (GEAENDERT) — RESEND_FROM_EMAIL hinzugefuegt
- package.json (GEAENDERT) — resend Dependency hinzugefuegt
- package-lock.json (GEAENDERT) — Lock-File aktualisiert
- src/components/ui/input.tsx (NEU) — shadcn/ui Input Komponente (via Code Review installiert)
