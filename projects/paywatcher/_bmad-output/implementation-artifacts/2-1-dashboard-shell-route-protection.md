# Story 2.1: Dashboard Shell & Route Protection

Status: done

<!-- Note: Validation is optional. Run validate-create-story for quality check before dev-story. -->

## Story

As a authentifizierter Tenant,
I want ein Dashboard-Layout mit Sidebar-Navigation und geschuetzten Routen,
so that ich nach dem Login eine konsistente Oberflaeche fuer alle Dashboard-Features habe.

## Acceptance Criteria

1. **Given** ein authentifizierter User das Dashboard aufruft **When** das Layout rendert **Then** sehe ich eine linke Sidebar (~200px, text-basiert) mit den Items: Overview, Payments, Settings, Docs **And** einen Content-Bereich rechts der Sidebar **And** einen Header mit Theme Toggle und User-Info (Email)

2. **Given** die Sidebar auf Desktop (>=1024px) sichtbar ist **When** ich den Collapse-Chevron klicke **Then** kollabiert die Sidebar auf ~60px (Icon-Only)

3. **Given** das Dashboard auf Mobile (<1024px) geladen wird **When** die Seite rendert **Then** ist die Sidebar versteckt und ein Hamburger-Icon oben links oeffnet sie als Sheet/Overlay

4. **Given** ein nicht-authentifizierter User eine Dashboard-Route aufruft **When** die Proxy den Auth-Check durchfuehrt (NFR8) **Then** wird der User zur Login-Seite redirected

5. **Given** das API Proxy Pattern implementiert ist **When** ein Route Handler aufgerufen wird **Then** kommuniziert er via mmsApi() mit dem MMS Backend (api.masem.at) unter Verwendung des Server-Side MMS_API_KEY **And** Responses werden von snake_case zu camelCase transformiert **And** das Envelope-Format { data } / { error } wird eingehalten

## Tasks / Subtasks

- [x] Task 1: MMS API Client Layer aufbauen (AC: #5)
  - [x] 1.1 `src/lib/api/client.ts` erstellen — `mmsApi<T>(path, options?)` Fetch-Wrapper mit `MMS_API_URL` + `MMS_API_KEY` aus `process.env`, Content-Type JSON, Error Handling
  - [x] 1.2 `src/lib/api/transform.ts` erstellen — `snakeToCamel()` generische Transformation fuer API Responses (snake_case → camelCase Keys)
  - [x] 1.3 `src/lib/api/errors.ts` erstellen — Error Code Mapping (UNAUTHORIZED, FORBIDDEN, NOT_FOUND, VALIDATION_ERROR, MMS_ERROR, INTERNAL_ERROR) mit HTTP Status Codes
  - [x] 1.4 `src/lib/api/types.ts` erstellen — MMS API Response Types (snake_case Originale): `MmsApiResponse<T>`, `MmsListResponse<T>`, `MmsErrorResponse`
  - [x] 1.5 Bestehende `.gitkeep` in `src/lib/api/` entfernen

- [x] Task 2: Auth.js v5 Konfiguration mit Magic Link Stub (AC: #4)
  - [x] 2.1 `next-auth@beta` (Auth.js v5) und `@auth/core` installieren
  - [x] 2.2 `src/lib/auth.ts` erstellen — NextAuth Config mit JWT Strategy, Session Callbacks (party_id, email, displayName ins JWT/Session), Provider-Stub fuer Resend Magic Link (wird in Story 2.2 vollstaendig implementiert)
  - [x] 2.3 `src/app/api/auth/[...nextauth]/route.ts` erstellen — Auth.js Route Handler (GET + POST exportieren via `handlers` aus auth.ts)
  - [x] 2.4 `src/types/auth.ts` erstellen — AuthSession, User, JwtPayload Types (party_id: string, email: string, displayName?: string)
  - [x] 2.5 `.env.example` aktualisieren — `AUTH_SECRET` (war NEXTAUTH_SECRET), `AUTH_URL` (war NEXTAUTH_URL) gemaess Auth.js v5 Naming Convention
  - [x] 2.6 `.env.local` Anpassung dokumentieren (AUTH_SECRET generieren via `npx auth secret`)

- [x] Task 3: Route Protection via proxy.ts (AC: #4)
  - [x] 3.1 `src/proxy.ts` erstellen (NICHT middleware.ts — Next.js 16 Rename!) — Auth-Check fuer Dashboard-Routen, Redirect zu /login wenn keine Session
  - [x] 3.2 Matcher Config: `['/dashboard/:path*', '/admin/:path*']` — nur geschuetzte Routen pruefen
  - [x] 3.3 Public Routes (Landing Page, /login, /signup, /docs, /api/auth, /api/health) bleiben OHNE Auth-Check
  - [x] 3.4 Auth.js v5 Integration: `auth()` Wrapper-Funktion als proxy exportieren

- [x] Task 4: Dashboard Layout mit Sidebar (AC: #1, #2, #3)
  - [x] 4.1 `src/app/(dashboard)/layout.tsx` erstellen — Dashboard Layout mit Auth-Check via `auth()` in Server Component, Redirect wenn keine Session
  - [x] 4.2 `src/components/dashboard/sidebar-nav.tsx` erstellen — Sidebar Navigation Component (~200px, text-basiert), 4 Items: Overview, Payments, Settings, Docs, Active State mit Accent-Farbe, Collapse-Chevron am unteren Rand
  - [x] 4.3 `src/components/dashboard/dashboard-header.tsx` erstellen — Top Bar mit Theme Toggle (bestehend), User Email aus Session, Mobile Hamburger-Icon (nur <1024px)
  - [x] 4.4 `src/components/dashboard/mobile-sidebar.tsx` erstellen — shadcn/ui Sheet Component, side="left", oeffnet von links, Tap-Outside schliesst
  - [x] 4.5 Route-Struktur beibehalten: `src/app/(dashboard)/dashboard/page.tsx` fuer Overview (URL: `/dashboard`), Placeholder-Content "Overview kommt in Story 3.2" — Route-Kollision mit Landing Page `(landing)/page.tsx` (URL: `/`) vermieden
  - [x] 4.6 `src/app/(dashboard)/payments/page.tsx` erstellen — Placeholder "Payments kommt in Epic 3"
  - [x] 4.7 `src/app/(dashboard)/settings/page.tsx` erstellen — Placeholder "Settings kommt in Epic 4"

- [x] Task 5: TanStack Query Provider Setup (AC: #5)
  - [x] 5.1 `@tanstack/react-query` und `@tanstack/react-query-devtools` installieren
  - [x] 5.2 `src/lib/query-client.ts` erstellen — QueryClient mit Default-Options (staleTime: 30s fuer Payments, refetchOnWindowFocus: false)
  - [x] 5.3 `src/components/shared/query-provider.tsx` erstellen — Client Component mit QueryClientProvider + ReactQueryDevtools
  - [x] 5.4 QueryProvider in Dashboard Layout einbinden (NICHT im Root Layout — nur Dashboard braucht TanStack Query)

- [x] Task 6: Build-Validierung und Qualitaetspruefung (Alle ACs)
  - [x] 6.1 `npm run build` muss fehlerfrei durchlaufen
  - [x] 6.2 `npm run type-check` muss fehlerfrei durchlaufen
  - [x] 6.3 `npm run lint` muss fehlerfrei durchlaufen
  - [x] 6.4 Verifizieren: Landing Page bleibt `○ (Static)` im Build Output (SSG nicht gebrochen)
  - [x] 6.5 Verifizieren: Dashboard-Routen sind geschuetzt (Redirect zu /login ohne Session)
  - [x] 6.6 Verifizieren: mmsApi() Client funktioniert (Type-Check, kein Runtime-Test noetig — MMS Integration kommt in Story 2.2)
  - [x] 6.7 Verifizieren: Sidebar rendert korrekt auf Desktop und Mobile

## Dev Notes

### Kontext & Zweck

Dies ist die **erste Story von Epic 2** (User Authentication & API Key Management). Epic 1 hat die gesamte Landing Page geliefert (6 Stories: Scaffold, Theme, Hero/Problem-Solution, How It Works/Code Examples, Pricing/Comparison/Trust, SEO/Analytics). Epic 2 baut jetzt das authentifizierte Dashboard auf.

**Was PayWatcher ist:** Developer-First Verification API fuer Stablecoin-Zahlungen (USDC auf Base Network). Non-custodial, reiner Verification Layer. Das Frontend ist ein reiner API-Consumer — kein eigenes Backend, keine eigene Datenbank. Alle Daten kommen vom MMS Backend (api.masem.at).

**Was diese Story liefert:**
- MMS API Client Layer (`mmsApi()` Wrapper, Transformation, Error Handling)
- Auth.js v5 Konfiguration (JWT Strategy, Session Callbacks) mit Magic Link Stub
- Route Protection via `proxy.ts` (Next.js 16 Pattern)
- Dashboard Layout mit Sidebar Navigation (4 Items: Overview, Payments, Settings, Docs)
- Dashboard Header mit Theme Toggle und User-Info
- Mobile Sidebar als Sheet/Overlay (shadcn/ui)
- Sidebar Collapse auf Desktop
- TanStack Query Provider fuer Dashboard-Datenfetching
- Placeholder-Pages fuer Overview, Payments, Settings

**Was diese Story NICHT beinhaltet:**
- Magic Link Email-Versand (Story 2.2)
- MMS Parties API Integration / Signup Flow (Story 2.2)
- API Key Erstellung und Anzeige (Story 2.3)
- API Key Regenerate und Scopes (Story 2.4)
- Payment-Daten, Overview-Content, Settings-Content (Epic 3+4)
- Onboarding Screen (Story 3.2)

**Abhaengigkeiten:**
- Epic 1 komplett abgeschlossen (Scaffold, Theme System, Landing Page, SEO/Analytics)
- Auth wird hier als Stub angelegt — Story 2.2 macht den Magic Link Flow komplett
- Dashboard Layout ist Voraussetzung fuer alle weiteren Dashboard-Stories

### Technische Anforderungen

**Zu installierende Packages:**
- `next-auth@beta` (Auth.js v5) — Auth System, JWT Strategy, Route Protection
- `@tanstack/react-query` (v5) — Client-Side Data Fetching + Caching
- `@tanstack/react-query-devtools` — DevTools fuer Development (optional, empfohlen)

**Bereits installierte und verwendbare Packages:**
- Next.js 16.1.6 mit App Router
- React 19.2.3
- Tailwind CSS v4 mit @theme Directive
- shadcn/ui (Button, Badge, Card, Sheet, Tabs, etc.)
- next-themes 0.4.6 (Theme Toggle)
- lucide-react 0.564.0 (Icons)
- sonner 2.0.7 (Toasts)
- geist 1.7.0 (Fonts)

**NICHT in dieser Story installieren:**
- `resend` — Magic Link Email wird in Story 2.2 implementiert
- `react-hook-form` / `zod` — Formulare kommen erst in Epic 4 (Settings)
- `recharts` — Dashboard-Charts kommen in Phase 1b
- `@auth/prisma-adapter` oder aehnliche DB-Adapter — PayWatcher hat KEINE eigene DB

### KRITISCH: Next.js 16 Proxy (nicht Middleware!)

**Next.js 16 hat `middleware.ts` in `proxy.ts` umbenannt!**
- Dateiname: `proxy.ts` (NICHT `middleware.ts`)
- Export: `export function proxy(request)` ODER `export { auth as proxy }` mit Auth.js
- Codemod verfuegbar: `npx @next/codemod@canary middleware-to-proxy`
- Das Architecture Doc referenziert noch `middleware.ts` — dies ist veraltet, `proxy.ts` verwenden!
- Die `proxy.ts` Datei liegt in `src/` (neben `app/`)

```typescript
// src/proxy.ts — KORREKT fuer Next.js 16
import { auth } from "@/lib/auth"

export const proxy = auth((req) => {
  const isAuthenticated = !!req.auth
  const { pathname } = req.nextUrl

  // Dashboard-Routen schuetzen
  if (pathname.startsWith('/dashboard') && !isAuthenticated) {
    return Response.redirect(new URL('/login', req.url))
  }
})

export const config = {
  matcher: ['/dashboard/:path*', '/admin/:path*']
}
```

### Auth.js v5 Konfiguration (aus Architecture Doc + Web Research)

**Environment Variables (Auth.js v5 Naming):**
- `AUTH_SECRET` (war: `NEXTAUTH_SECRET`) — Session Encryption Key
- `AUTH_URL` (war: `NEXTAUTH_URL`) — Base URL der App

**JWT Strategy (kein DB-Adapter noetig):**
```typescript
// src/lib/auth.ts
import NextAuth from "next-auth"

export const { auth, handlers, signIn, signOut } = NextAuth({
  providers: [
    // Magic Link Provider — Stub fuer Story 2.1
    // Vollstaendige Resend-Integration in Story 2.2
  ],
  session: { strategy: "jwt" },
  callbacks: {
    jwt({ token, user }) {
      if (user) {
        token.partyId = user.partyId
        token.email = user.email
        token.displayName = user.displayName
      }
      return token
    },
    session({ session, token }) {
      session.user.partyId = token.partyId as string
      session.user.email = token.email as string
      session.user.displayName = token.displayName as string | undefined
      return session
    },
  },
  pages: {
    signIn: '/login',
    // signOut redirects to Landing Page
  },
})
```

**WICHTIG — Auth.js v5 Gotchas:**
- `next-auth@beta` Package Name (nicht `@auth/nextjs`)
- `AUTH_*` Environment Variable Prefix (nicht `NEXTAUTH_*`)
- `auth()` ist die universelle Funktion fuer Server Components, Route Handlers, und Proxy
- Kein `getServerSession` oder `getToken` Import mehr — alles ueber `auth()`
- Magic Link Provider benoetigt eigentlich einen DB-Adapter fuer Verification Tokens — in Story 2.1 als Stub, Story 2.2 loest das (evtl. mit Custom Token Storage via MMS oder temporaerer Loesung)

### MMS API Client Pattern (aus Architecture Doc)

```typescript
// src/lib/api/client.ts
const MMS_API_URL = process.env.MMS_API_URL
const MMS_API_KEY = process.env.MMS_API_KEY

export async function mmsApi<T>(
  path: string,
  options?: RequestInit
): Promise<T> {
  if (!MMS_API_URL || !MMS_API_KEY) {
    throw new Error('MMS_API_URL and MMS_API_KEY must be configured')
  }

  const res = await fetch(`${MMS_API_URL}${path}`, {
    ...options,
    headers: {
      'x-api-key': MMS_API_KEY,
      'Content-Type': 'application/json',
      ...options?.headers,
    },
  })

  if (!res.ok) {
    // Error handling mit MMS Error Codes
    const error = await res.json().catch(() => ({}))
    throw new MmsApiError(res.status, error)
  }

  return res.json()
}
```

**Regeln (STRIKT):**
- `mmsApi()` ist die EINZIGE Funktion die MMS aufruft — nie raw `fetch` zu api.masem.at
- API Key kommt aus `process.env.MMS_API_KEY` — nie hardcoded, nie im Client
- Transformation (snake → camel) passiert im Route Handler NACH dem mmsApi-Call
- TanStack Query im Browser ruft IMMER `/api/...` Route Handlers auf — nie direkt MMS
- `mmsApi()` wird NUR in Route Handlers (`app/api/`) und Server Components importiert — NIE in Client Components

### Dashboard Sidebar Spezifikation (aus UX-Spec)

**Layout:**
- Sidebar: ~200px breit, text-basiert (nicht Icon-Only im Default)
- Collapsible: Chevron am unteren Rand, collapsed = ~60px Icon-Only
- Surface-Farbe als Hintergrund (1-2 Stufen heller als Background)
- 4 Nav-Items: Overview, Payments, Settings, Docs

**Active State:**
- Aktiver Item: Accent-Farbe (Text + subtiler Background-Highlight)
- Hover: Leichter Background-Shift
- Keine verschachtelten Menues, keine Subnavigation

**Icons (lucide-react):**
- Overview: `LayoutDashboard` oder `Home`
- Payments: `CreditCard` oder `Wallet`
- Settings: `Settings`
- Docs: `BookOpen` oder `FileText`

**Mobile (<1024px):**
- Sidebar versteckt, Hamburger-Icon oben links
- shadcn/ui Sheet Component, `side="left"`, von links einsliden
- Tap-Outside schliesst die Sidebar

**Footer:**
- User Email (aus Session) + Avatar-Placeholder
- Collapse-Chevron

**Docs-Link:**
- Linkt auf `/docs` (oeffentliche Docs-Seiten, gleich fuer eingeloggte und nicht-eingeloggte User)

### Dashboard Header Spezifikation

**Desktop (>=1024px):**
- Linke Seite: Page Title (dynamisch je nach Route)
- Rechte Seite: Theme Toggle (bestehende Komponente) + User Email

**Mobile (<1024px):**
- Linke Seite: Hamburger-Icon (oeffnet Mobile Sidebar)
- Mitte: PayWatcher Logo/Text (optional)
- Rechte Seite: Theme Toggle

### TanStack Query Setup

**QueryClient Konfiguration:**
```typescript
// src/lib/query-client.ts
import { QueryClient } from "@tanstack/react-query"

export function makeQueryClient() {
  return new QueryClient({
    defaultOptions: {
      queries: {
        staleTime: 30 * 1000,      // 30s — Payments
        refetchOnWindowFocus: false, // Kein Auto-Refetch bei Tab-Wechsel
      },
    },
  })
}
```

**Provider als Client Component:**
```typescript
// src/components/shared/query-provider.tsx
"use client"

import { QueryClientProvider } from "@tanstack/react-query"
import { ReactQueryDevtools } from "@tanstack/react-query-devtools"
import { makeQueryClient } from "@/lib/query-client"
import { useState } from "react"

export function QueryProvider({ children }: { children: React.ReactNode }) {
  const [queryClient] = useState(() => makeQueryClient())

  return (
    <QueryClientProvider client={queryClient}>
      {children}
      <ReactQueryDevtools initialIsOpen={false} />
    </QueryClientProvider>
  )
}
```

**Einbindung NUR im Dashboard Layout** — Landing Page braucht kein TanStack Query:
```typescript
// src/app/(dashboard)/layout.tsx
import { QueryProvider } from "@/components/shared/query-provider"

export default async function DashboardLayout({ children }) {
  const session = await auth()
  if (!session) redirect('/login')

  return (
    <QueryProvider>
      <div className="flex h-screen">
        <SidebarNav />
        <main className="flex-1 overflow-auto">
          <DashboardHeader session={session} />
          {children}
        </main>
      </div>
    </QueryProvider>
  )
}
```

### Route Handler Response Format (aus Architecture Doc)

**Success (single):**
```json
{ "data": { "id": "uuid", "exactAmount": "49.000042" } }
```

**Success (list):**
```json
{
  "data": [{ "id": "uuid", "exactAmount": "49.000042" }],
  "meta": { "page": 1, "limit": 50, "total": 127 }
}
```

**Error:**
```json
{
  "error": { "code": "UNAUTHORIZED", "message": "Invalid or expired session" }
}
```

### Responsive Strategy

**Zwei Modi, nicht drei:** Mobile (<1024px) und Desktop (>=1024px).

| Element | Desktop (>=1024px) | Mobile (<1024px) |
|---------|-------------------|------------------|
| Sidebar | Permanent sichtbar, ~200px, collapsible | Versteckt, Sheet/Overlay via Hamburger |
| Header | Page Title links, User/Theme rechts | Hamburger links, Theme rechts |
| Content | Neben Sidebar, flex-1 | Volle Breite |

**Tailwind Breakpoint:** `lg:` fuer Desktop-Overrides. Base-Styles = Mobile.

### Architektur-Konformitaet

**Dateinamen:** `kebab-case.ts` / `kebab-case.tsx` — IMMER.

**Neue/geaenderte Dateien nach Abschluss:**
```
src/
├── proxy.ts                                    # NEU: Route Protection (Next.js 16)
├── app/
│   ├── (dashboard)/
│   │   ├── layout.tsx                          # NEU: Dashboard Layout (Auth + Sidebar + QueryProvider)
│   │   └── dashboard/
│   │       ├── page.tsx                        # AENDERN: Overview Placeholder (URL: /dashboard)
│   │       ├── payments/
│   │       │   └── page.tsx                    # NEU: Payments Placeholder (URL: /dashboard/payments)
│   │       └── settings/
│   │           └── page.tsx                    # NEU: Settings Placeholder (URL: /dashboard/settings)
│   └── api/
│       └── auth/
│           └── [...nextauth]/
│               └── route.ts                    # NEU: Auth.js Route Handler
├── components/
│   ├── dashboard/
│   │   ├── sidebar-nav.tsx                     # NEU: Sidebar Navigation
│   │   ├── dashboard-header.tsx                # NEU: Dashboard Top Bar
│   │   └── mobile-sidebar.tsx                  # NEU: Mobile Sheet Sidebar
│   └── shared/
│       └── query-provider.tsx                  # NEU: TanStack Query Provider
├── hooks/                                       # (noch leer, Hooks kommen in Epic 3)
├── lib/
│   ├── api/
│   │   ├── client.ts                           # NEU: mmsApi() Wrapper
│   │   ├── transform.ts                        # NEU: snake_case → camelCase
│   │   ├── errors.ts                           # NEU: Error Code Mapping
│   │   └── types.ts                            # NEU: MMS API Response Types
│   ├── auth.ts                                 # NEU: Auth.js v5 Config
│   └── query-client.ts                         # NEU: QueryClient Factory
├── types/
│   └── auth.ts                                 # NEU: Auth Types
.env.example                                     # AENDERN: AUTH_SECRET, AUTH_URL
```

**Zu entfernende Dateien:**
- `src/lib/api/.gitkeep` — ersetzt durch echte Dateien
- `src/app/(dashboard)/dashboard/page.tsx` — falsche Route, wird zu `src/app/(dashboard)/page.tsx`

**Server vs Client Components:**
- Server Components (DEFAULT): `layout.tsx` (Dashboard), `proxy.ts`, `auth.ts`, `api/auth/[...nextauth]/route.ts`, alle `lib/api/` Files
- Client Components: `sidebar-nav.tsx` (braucht useState fuer Collapse), `dashboard-header.tsx` (braucht usePathname), `mobile-sidebar.tsx` (braucht Sheet State), `query-provider.tsx`

**ANTI-Patterns:**
- KEIN `middleware.ts` — Next.js 16 nutzt `proxy.ts`
- KEIN `getServerSession` Import — Auth.js v5 nutzt `auth()`
- KEIN `NEXTAUTH_*` Environment Variable — Auth.js v5 nutzt `AUTH_*`
- KEIN `@auth/prisma-adapter` — PayWatcher hat keine eigene DB
- KEIN TanStack Query im Root Layout — nur im Dashboard Layout
- KEIN `mmsApi()` Import in Client Components — nur in Route Handlers und Server Components
- KEINE direkte `fetch` Calls zu `api.masem.at` aus dem Browser
- KEINE verschachtelten Sidebar-Menues oder Subnavigation
- KEIN Full-Page-Spinner — Sidebar + Header stehen sofort, Skeleton nur im Content
- KEINE PascalCase Dateinamen — immer `kebab-case.tsx`

### Library & Framework Anforderungen

**Auth.js v5 (next-auth@beta):**
- JWT Strategy ohne DB-Adapter
- `auth()` universelle Funktion (Server Components, Route Handlers, Proxy)
- Custom Callbacks fuer party_id, email, displayName im JWT/Session
- Magic Link Provider als Stub (vollstaendige Implementation in Story 2.2)
- `AUTH_SECRET` und `AUTH_URL` Environment Variables

**TanStack Query v5 (~5.90):**
- `@tanstack/react-query` fuer Hooks
- `@tanstack/react-query-devtools` fuer Development
- `useState(() => makeQueryClient())` Pattern fuer Client-Side Singleton
- staleTime: 30s Default, individuell pro Hook ueberschreibbar

**shadcn/ui Komponenten (bereits installiert):**
- `Sheet` — Mobile Sidebar
- `Button` — Nav Items, Collapse Toggle
- `Badge` — (nicht in dieser Story, aber vorbereitet)

**shadcn/ui Komponenten (evtl. nachinstallieren):**
- `Tooltip` — fuer collapsed Sidebar Icon-Only Labels
- `Separator` — visuelle Trennung in Sidebar
- `Avatar` — User-Avatar Placeholder im Sidebar-Footer

### Testing-Anforderungen

1. **Build-Validierung:** `npm run build` muss ohne Fehler durchlaufen
2. **Type-Check:** `npm run type-check` muss ohne Fehler durchlaufen
3. **Lint:** `npm run lint` muss ohne Fehler durchlaufen
4. **SSG-Validierung:** Landing Page muss als `○ (Static)` im Build Output erscheinen — Dashboard-Aenderungen duerfen SSG NICHT brechen
5. **Route Protection:** Dashboard-Routen (`/`, `/payments`, `/settings` unter (dashboard)) redirecten zu `/login` ohne Session
6. **Sidebar:** 4 Nav-Items sichtbar auf Desktop, Sheet auf Mobile
7. **Sidebar Collapse:** Chevron toggle zwischen 200px und ~60px
8. **Header:** Theme Toggle + User Email auf Desktop, Hamburger + Theme auf Mobile
9. **mmsApi():** TypeScript Types korrekt, Error Handling vorhanden (Runtime-Test nicht noetig)
10. **TanStack Query:** QueryProvider rendert ohne Fehler im Dashboard

### Vorherige Story Intelligence

**Story 1.6 (letzte Story in Epic 1) Learnings — KRITISCH:**
- **Analytics-Provider** als Client Component in `(landing)/layout.tsx` eingebunden — Dashboard braucht einen EIGENEN Layout, kein Sharing mit Landing
- **SSG bestaetigt funktionierend** — Landing Page ist `○ (Static)` im Build Output, NICHT brechen
- **`'use client'` nur in Component-Dateien** — nie in page.tsx oder layout.tsx (ausser Layout ist Client Component)
- **Theme Toggle** (`src/components/shared/theme-toggle.tsx`) existiert und funktioniert — wiederverwenden im Dashboard Header
- **kebab-case Dateinamen** strikt eingehalten — beibehalten
- **`.env.example`** hat `NEXT_PUBLIC_ANALYTICS_PROJECT`, `MMS_API_URL`, `MMS_API_KEY`, `NEXTAUTH_SECRET`, `NEXTAUTH_URL`, `RESEND_API_KEY`

**Story 1.1 Learnings:**
- **Route Group `(dashboard)`** existiert bereits mit `dashboard/page.tsx` — Route-Struktur muss korrigiert werden (page.tsx direkt unter (dashboard), nicht unter dashboard/)
- **Route Group `(admin)`** existiert bereits mit `admin/page.tsx`
- **`geist` Package** fuer Fonts — bewaehrt
- **shadcn/ui `Sheet` Component** bereits installiert — fuer Mobile Sidebar verwenden

**Story 1.2 Learnings:**
- **Design Tokens** via OKLch in globals.css — Surface-Farbe fuer Sidebar verwenden
- **`next-themes`** mit `attribute="class"` und `defaultTheme="dark"` — Dashboard erbt Theme
- **Payment Status Types** in `src/types/payment.ts` definiert — Patterns fuer neue Types uebernehmen

**Story 1.3 Learnings:**
- **Landing Header** (`src/components/landing/landing-header.tsx`) hat Mobile Nav Pattern — als Referenz fuer Dashboard Mobile Nav
- **Mobile Nav** (`src/components/landing/mobile-nav.tsx`) nutzt Sheet Component — Pattern uebernehmen

**Git Intelligence:**
```
260ae0a Story 1.6: SEO, social sharing, analytics integration with code review fixes
95b7f01 Story 1.5: Pricing section, comparison callout, trust signals with code review fixes
ef7f25e Code review fixes for Story 1.4: CopyButton timeout, scroll indicator, FOUC, grid alignment
3b8016a Story 1.4: How It Works section, interactive code examples with tabs
36a83f8 Story 1.3: Landing page layout, hero section, and problem/solution
5a45117 Story 1.2: Theme system, design tokens, and payment status colors
48d5ba9 Story 1.1: Project scaffold with Next.js 16, Tailwind v4, shadcn/ui
```

**Patterns aus vorherigen Stories:**
- Server Components als Default — bewaehrt, beibehalten
- Client Components nur wenn noetig (`'use client'` Directive)
- `npm run build && npm run type-check && npm run lint` als Validierung
- Component Boundaries: `landing/` vs `dashboard/` vs `shared/` strikt trennen
- shadcn/ui Sheet Pattern aus `mobile-nav.tsx` als Vorlage fuer Mobile Sidebar

### Aktuelle Technologie-Informationen (Web Research, Feb 2026)

**Auth.js v5 (next-auth@beta):**
- Noch in Beta, aber production-ready fuer die meisten Use Cases
- Package: `next-auth@beta` (nicht `@auth/nextjs`)
- Environment Variables: `AUTH_SECRET`, `AUTH_URL` (NICHT `NEXTAUTH_*`)
- Universelle `auth()` Funktion ersetzt `getServerSession()` und `getToken()`
- JWT Strategy benoetigt KEINEN DB-Adapter
- Magic Link Provider benoetigt normalerweise DB-Adapter fuer Verification Tokens — Workaround noetig fuer PayWatcher (kein eigenes DB)
- `signIn` und `signOut` Server Actions exportiert
- Proxy-Integration: `export { auth as proxy }` oder Wrapper-Funktion

**Next.js 16 Proxy (ehemals Middleware):**
- `middleware.ts` → `proxy.ts` Rename
- Funktion heisst `proxy`, nicht `middleware`
- Matcher-Syntax identisch zu bisheriger Middleware
- Node.js Runtime (nicht mehr Edge-only)
- Codemod: `npx @next/codemod@canary middleware-to-proxy`

**TanStack Query v5 (~5.90.21, Feb 2026):**
- Volle React 19 Kompatibilitaet
- `useSuspenseQuery` fuer React 19 `use` Hook
- `useState(() => makeQueryClient())` Pattern fuer App Router
- SSR/Hydration Support via `dehydrate()` + `HydrationBoundary` (nicht noetig fuer diese Story)

**shadcn/ui Sheet:**
- `side="left"` fuer linke Sidebar
- Vollstaendige Accessibility (Focus Trapping, Keyboard Navigation)
- `SheetContent`, `SheetHeader`, `SheetTitle`, `SheetTrigger`

### Project Structure Notes

- Die **(dashboard) Route Group** hat aktuell eine falsche Struktur: `(dashboard)/dashboard/page.tsx`. Das muss zu `(dashboard)/page.tsx` korrigiert werden, damit die URL `/` (nicht `/dashboard`) unter dem Dashboard-Layout ist. Alternativ: wenn `/dashboard` als URL gewuenscht ist, dann `(dashboard)/dashboard/page.tsx` beibehalten, aber das widerspricht dem Architecture Doc das `/` als Overview-Seite definiert
- **ACHTUNG:** Im Architecture Doc ist die Dashboard-Route als `app/(dashboard)/page.tsx` (URL: `/`) definiert, aber das kollidiert mit der Landing Page `app/(landing)/page.tsx` (URL: `/`). Loesung: Dashboard-Routen unter `/dashboard/*` — d.h. URL ist `/dashboard` fuer Overview, `/dashboard/payments` fuer Payments, etc.
- **Component Boundaries beachten:** Dashboard-Components in `components/dashboard/`, NICHT in `components/landing/` oder `components/shared/` (ausser sie werden in beiden Bereichen verwendet)
- **Sidebar State:** `collapsed` State als `useState` in der Sidebar-Komponente. Persistent via `localStorage` (optional, nice-to-have)

### References

- [Source: _bmad-output/planning-artifacts/epics.md#Story 2.1] — User Story und Acceptance Criteria
- [Source: _bmad-output/planning-artifacts/prd.md#Functional Requirements] — FR8-FR14 (Auth & API Key), NFR8 (Protected Routes), NFR17 (No Secrets)
- [Source: _bmad-output/planning-artifacts/architecture.md#Auth Pattern] — Auth.js v5, JWT Strategy, Magic Link, Route Protection
- [Source: _bmad-output/planning-artifacts/architecture.md#API Client Pattern] — mmsApi(), TanStack Query, Route Handler Response Format
- [Source: _bmad-output/planning-artifacts/architecture.md#App Structure] — Route Groups, Dashboard Layout
- [Source: _bmad-output/planning-artifacts/architecture.md#Structure Patterns] — Dateistruktur, Naming Conventions
- [Source: _bmad-output/planning-artifacts/architecture.md#Enforcement Guidelines] — 10 Regeln + Anti-Patterns
- [Source: _bmad-output/planning-artifacts/ux-design-specification.md#Sidebar] — ~200px, text-basiert, collapsible, 4 Items
- [Source: _bmad-output/planning-artifacts/ux-design-specification.md#Responsive Strategy] — Desktop-First Dashboard, Mobile Sheet
- [Source: _bmad-output/planning-artifacts/ux-design-specification.md#Loading States] — Skeleton, kein Full-Page-Spinner
- [Source: _bmad-output/planning-artifacts/ux-design-specification.md#Navigation Patterns] — Active State, keine Subnavigation
- [Source: _bmad-output/implementation-artifacts/1-6-seo-social-sharing-analytics-integration.md] — Letzte Story Intelligence

## Dev Agent Record

### Agent Model Used

Claude Opus 4.6 (claude-opus-4-6)

### Debug Log References

- Type error in auth.ts: JWT callback assigned `unknown` to typed fields — fixed with proper type narrowing via augmented User interface
- Lint error in sidebar-nav.tsx: `setState` inside `useEffect` — fixed with lazy `useState` initializer for localStorage read

### Completion Notes List

- **Task 1:** MMS API Client Layer — `mmsApi<T>()` fetch wrapper, `snakeToCamel()` transformer, `MmsApiError` class with error code mapping, typed response interfaces
- **Task 2:** Auth.js v5 (next-auth@beta) — JWT strategy config, session callbacks for partyId/email/displayName, route handler, type augmentations for next-auth module, .env.example updated to AUTH_* naming
- **Task 3:** Route Protection via `src/proxy.ts` (Next.js 16 pattern) — protects /dashboard/* and /admin/*, redirects to /login
- **Task 4:** Dashboard Layout — Server Component layout with auth check, collapsible sidebar (200px/60px with localStorage persistence), responsive header with theme toggle, mobile Sheet sidebar, placeholder pages for Overview/Payments/Settings
- **Task 5:** TanStack Query v5 — QueryClient factory with 30s staleTime, QueryProvider client component with DevTools, integrated only in Dashboard layout
- **Task 6:** Build validation — build, type-check, lint all pass, landing page remains static (SSG), dashboard routes are dynamic (server-rendered)

### Change Log

- 2026-02-16: Story 2.1 implemented — Dashboard Shell, Auth.js v5, Route Protection, MMS API Client, TanStack Query
- 2026-02-16: Code Review — 3 HIGH, 4 MEDIUM, 3 LOW findings. Fixed: ERROR_STATUS_MAP 400 mapping, snakeToCamel regex, duplicate h1, navItems extraction, story documentation corrections

### File List

New files (added by code review):
- src/components/dashboard/nav-config.ts

New files:
- src/proxy.ts
- src/lib/auth.ts
- src/lib/query-client.ts
- src/lib/api/client.ts
- src/lib/api/transform.ts
- src/lib/api/errors.ts
- src/lib/api/types.ts
- src/types/auth.ts
- src/types/next-auth.d.ts
- src/app/api/auth/[...nextauth]/route.ts
- src/app/(dashboard)/layout.tsx
- src/app/(dashboard)/dashboard/payments/page.tsx
- src/app/(dashboard)/dashboard/settings/page.tsx
- src/components/dashboard/sidebar-nav.tsx
- src/components/dashboard/dashboard-header.tsx
- src/components/dashboard/mobile-sidebar.tsx
- src/components/shared/query-provider.tsx

Modified files:
- src/app/(dashboard)/dashboard/page.tsx (updated to Overview placeholder)
- .env.example (AUTH_SECRET/AUTH_URL statt NEXTAUTH_*)
- package.json (next-auth@beta, @tanstack/react-query, @tanstack/react-query-devtools)

Deleted files:
- src/lib/api/.gitkeep
- src/components/dashboard/.gitkeep
