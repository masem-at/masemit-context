# Story 6.1: Admin Shell, Auth & Route Protection

Status: done

<!-- Note: Validation is optional. Run validate-create-story for quality check before dev-story. -->

## Story

As a Admin (Mario),
I want ein Admin-Dashboard-Layout mit eigenem Auth-Check und geschützten Routen,
so that nur ich als Admin auf die Admin-Seiten zugreifen kann.

## Acceptance Criteria

1. **Given** ich bin per Magic Link eingeloggt und meine Email ist als Admin konfiguriert (z.B. mario@masem.at)
   **When** ich `/admin` aufrufe
   **Then** sehe ich das Admin-Dashboard-Layout mit einer Sidebar mit den Items: Overview, Tenants, Payments
   **And** der Header zeigt "Admin" als Kennzeichnung

2. **Given** ein eingeloggter User, dessen Email NICHT als Admin konfiguriert ist
   **When** er `/admin` aufruft
   **Then** wird er auf das User Dashboard redirected (oder sieht eine 403-Seite)

3. **Given** ein nicht-authentifizierter User
   **When** er `/admin` aufruft
   **Then** wird er auf `/login` redirected

4. **Given** das Admin-Layout implementiert wird
   **When** die Route Group erstellt wird
   **Then** existiert `app/(admin)/layout.tsx` mit `isAdmin` Check (Email-basiert, konfigurierbar via ENV `ADMIN_EMAILS`)
   **And** die Admin-Sidebar ist separat vom User-Dashboard-Sidebar

5. **Given** Admin API Proxy Routes erstellt werden
   **When** ein Admin Route Handler aufgerufen wird
   **Then** nutzt er `process.env.MMS_API_KEY` für MMS-Calls (nicht die tenant_keys DB)
   **And** existieren: `app/api/admin/health/route.ts`, `app/api/admin/tenants/route.ts`, `app/api/admin/tenants/[slug]/route.ts`, `app/api/admin/payments/route.ts`
   **And** jeder Route Handler prüft `isAdmin` bevor er den Request weiterleitet

## Tasks / Subtasks

- [x] Task 1: `isAdmin` Helper + ADMIN_EMAILS ENV (AC: #2, #4)
  - [x] 1.1: `ADMIN_EMAILS` zu `.env.example` hinzufügen (kommaseparierte Email-Liste)
  - [x] 1.2: `isAdmin(email: string)` Helper in `src/lib/admin.ts` erstellen — prüft ob Email in `ADMIN_EMAILS` ENV enthalten ist
  - [x] 1.3: `requireAdminAuth()` Helper erstellen — ruft `auth()`, prüft Session + `isAdmin(contactEmail)`, gibt `{ session }` oder `{ error: NextResponse(401/403) }` zurück

- [x] Task 2: Admin Layout + Route Protection (AC: #1, #3, #4)
  - [x] 2.1: `src/app/(admin)/layout.tsx` erstellt — Server Component, `auth()` Check → redirect `/login` wenn nicht eingeloggt, `isAdmin()` Check → redirect `/dashboard` wenn kein Admin
  - [x] 2.2: `QueryProvider` eingebunden (wie Dashboard Layout)
  - [x] 2.3: Admin Shell mit AdminSidebar + AdminHeader rendert

- [x] Task 3: Admin Sidebar Navigation (AC: #1, #4)
  - [x] 3.1: `src/components/admin/admin-nav-config.ts` — Nav Items: Overview (`/admin`), Tenants (`/admin/tenants`), Payments (`/admin/payments`)
  - [x] 3.2: `src/components/admin/admin-sidebar.tsx` — Desktop-Only Sidebar (kein Mobile-Layout), Pattern von dashboard `sidebar-nav.tsx` übernommen, ohne Collapse und ohne Mobile-Sheet
  - [x] 3.3: Admin-Sidebar zeigt "Admin" Label prominent an (Shield Icon + "Admin" Text)

- [x] Task 4: Admin Header (AC: #1)
  - [x] 4.1: `src/components/admin/admin-header.tsx` — Einfacher Header mit Route-Title-Mapping, "Admin" Badge-Kennzeichnung, ThemeToggle, User Email-Anzeige
  - [x] 4.2: Keine MobileSidebar (Desktop-Only)

- [x] Task 5: Admin API Proxy Routes (AC: #5)
  - [x] 5.1: `src/app/api/admin/health/route.ts` — GET → `mmsApi('/v1/admin/paywatcher/health')` (Fallback auf `MMS_API_KEY`)
  - [x] 5.2: `src/app/api/admin/tenants/route.ts` — GET + POST → `mmsApi('/v1/admin/paywatcher/tenants')`
  - [x] 5.3: `src/app/api/admin/tenants/[slug]/route.ts` — PATCH → `mmsApi('/v1/admin/paywatcher/tenants/${slug}')`
  - [x] 5.4: `src/app/api/admin/payments/route.ts` — GET → `mmsApi('/v1/admin/paywatcher/payments')` mit Query-Params Weiterleitung
  - [x] 5.5: Jeder Route Handler nutzt `requireAdminAuth()` vor dem MMS-Call

- [x] Task 6: Admin Stub Pages (AC: #1)
  - [x] 6.1: `src/app/(admin)/admin/page.tsx` überarbeitet → Admin Overview Platzhalter ("System Health kommt in Story 6.2")
  - [x] 6.2: `src/app/(admin)/admin/tenants/page.tsx` → Platzhalter ("Tenant Management kommt in Story 6.3")
  - [x] 6.3: `src/app/(admin)/admin/payments/page.tsx` → Platzhalter ("Global Payments kommt in Story 6.4")

## Dev Notes

### Kritische Architektur-Erkenntnisse

**`MMS_API_KEY` IST der Admin Key:**
- Die Codebase hat KEIN separates `MMS_ADMIN_API_KEY` — das bestehende `MMS_API_KEY` in `.env.example` hat bereits die Scopes `parties:read`, `admin:read`, `admin:write`
- `mmsApi()` in `src/lib/api/client.ts` fällt automatisch auf `process.env.MMS_API_KEY` zurück wenn kein `apiKey` Argument übergeben wird
- Admin Route Handlers rufen `mmsApi(path)` OHNE drittes Argument auf → automatisch Admin Key
- KEIN `getTenantApiKey()` in Admin Routes nötig — das ist nur für Tenant Dashboard Routes

**Auth Pattern — Layout-Level, KEINE Middleware:**
- Es gibt KEIN `middleware.ts` im Projekt. Auth-Checks laufen über `auth()` im Layout.
- Dashboard Layout Pattern: `const session = await auth(); if (!session?.user) redirect("/login")`
- Admin Layout muss zusätzlich `isAdmin(session.user.email)` prüfen → redirect `/dashboard` bei Nicht-Admin

**Admin ist Desktop-Only:**
- Kein MobileSidebar, kein Sheet, kein Hamburger-Menü
- Sidebar ist immer sichtbar, nicht collapsible
- Kein `lg:hidden` / `lg:block` Responsive-Handling nötig

**Bestehende Route-Struktur:**
- `src/app/(admin)/admin/page.tsx` existiert als Stub — MUSS überarbeitet werden
- `src/app/(admin)/layout.tsx` existiert NICHT — MUSS erstellt werden
- `src/app/api/admin/` existiert NICHT — alle 4 Route-Dateien müssen erstellt werden

### Pattern-Übernahme vom Dashboard

Die Admin Shell folgt dem gleichen Architektur-Pattern wie das Dashboard:

| Aspekt | Dashboard | Admin |
|--------|-----------|-------|
| Layout | `(dashboard)/layout.tsx` | `(admin)/layout.tsx` |
| Auth Check | `auth()` + `tenantSlug` Check | `auth()` + `isAdmin(email)` Check |
| Sidebar | `components/dashboard/sidebar-nav.tsx` | `components/admin/admin-sidebar.tsx` |
| Header | `components/dashboard/dashboard-header.tsx` | `components/admin/admin-header.tsx` |
| Nav Config | `components/dashboard/nav-config.ts` | `components/admin/admin-nav-config.ts` |
| API Key | Tenant Key aus DB (`getTenantApiKey`) | `MMS_API_KEY` ENV (Fallback in `mmsApi()`) |
| Mobile | Responsive + MobileSidebar | Desktop-Only, keine Mobile-Variante |
| QueryProvider | Ja | Ja |

### Route Handler Pattern (Admin)

```typescript
// Beispiel: src/app/api/admin/health/route.ts
import { requireAdminAuth } from "@/lib/admin"
import { mmsApi } from "@/lib/api/client"

export async function GET() {
  const authResult = await requireAdminAuth()
  if ("error" in authResult) return authResult.error

  // Kein apiKey-Argument → mmsApi fällt auf MMS_API_KEY zurück
  const data = await mmsApi("/v1/admin/paywatcher/health")
  return Response.json(data)
}
```

### MMS Admin Endpoints (alle live, benötigen `admin:read` bzw. `admin:write` Scope)

| Proxy Route | MMS Endpoint | Method | Scope |
|-------------|--------------|--------|-------|
| `/api/admin/health` | `GET /v1/admin/paywatcher/health` | GET | `admin:read` |
| `/api/admin/tenants` | `GET /v1/admin/paywatcher/tenants` | GET | `admin:read` |
| `/api/admin/tenants` | `POST /v1/admin/paywatcher/tenants` | POST | `admin:write` |
| `/api/admin/tenants/[slug]` | `PATCH /v1/admin/paywatcher/tenants/:slug` | PATCH | `admin:write` |
| `/api/admin/payments` | `GET /v1/admin/paywatcher/payments` | GET | `admin:read` |

### `isAdmin` Design-Entscheidung

```typescript
// src/lib/admin.ts
export function isAdmin(email: string | null | undefined): boolean {
  if (!email) return false
  const adminEmails = (process.env.ADMIN_EMAILS ?? "")
    .split(",")
    .map((e) => e.trim().toLowerCase())
    .filter(Boolean)
  return adminEmails.includes(email.toLowerCase())
}

export async function requireAdminAuth() {
  const session = await auth()
  if (!session?.user?.email) {
    return { error: NextResponse.json({ error: { code: "UNAUTHORIZED", message: "Not authenticated" } }, { status: 401 }) }
  }
  if (!isAdmin(session.user.email)) {
    return { error: NextResponse.json({ error: { code: "FORBIDDEN", message: "Admin access required" } }, { status: 403 }) }
  }
  return { session }
}
```

### Project Structure Notes

- Alle neuen Dateien unter `src/components/admin/` — NIE in `components/dashboard/` (Boundary-Regel)
- Admin Routes unter `src/app/(admin)/admin/` — das doppelte `admin` ist korrekt (Route Group `(admin)` + URL-Segment `admin`)
- API Routes unter `src/app/api/admin/` — getrennt von Tenant API Routes
- Types für Admin-Entitäten erst in Story 6.2/6.3 — hier nur Shell + Auth + Proxy-Skelett

### Bestehende Dateien die modifiziert werden

| Datei | Änderung |
|-------|----------|
| `.env.example` | `ADMIN_EMAILS=mario@masem.at` hinzufügen |
| `src/app/(admin)/admin/page.tsx` | Stub durch echten Platzhalter mit Admin-Layout ersetzen |

### Neue Dateien die erstellt werden

| Datei | Zweck |
|-------|-------|
| `src/lib/admin.ts` | `isAdmin()` + `requireAdminAuth()` |
| `src/app/(admin)/layout.tsx` | Admin Layout (Auth + Admin Check + Shell) |
| `src/components/admin/admin-nav-config.ts` | Admin Navigation Items |
| `src/components/admin/admin-sidebar.tsx` | Desktop-Only Admin Sidebar |
| `src/components/admin/admin-header.tsx` | Admin Header mit "Admin" Label |
| `src/app/(admin)/admin/tenants/page.tsx` | Stub für Story 6.3 |
| `src/app/(admin)/admin/payments/page.tsx` | Stub für Story 6.4 |
| `src/app/api/admin/health/route.ts` | Proxy → MMS Health |
| `src/app/api/admin/tenants/route.ts` | Proxy → MMS Tenants (GET, POST) |
| `src/app/api/admin/tenants/[slug]/route.ts` | Proxy → MMS Tenant (PATCH) |
| `src/app/api/admin/payments/route.ts` | Proxy → MMS Global Payments |

### References

- [Source: _bmad-output/planning-artifacts/epics.md#Epic 6 - Story 6.1]
- [Source: _bmad-output/planning-artifacts/architecture.md#App Structure & Routing]
- [Source: _bmad-output/planning-artifacts/architecture.md#Auth Pattern - Route Protection]
- [Source: _bmad-output/planning-artifacts/architecture.md#API Communication Pattern - Data Flow (Admin)]
- [Source: src/app/(dashboard)/layout.tsx - Dashboard Layout Pattern]
- [Source: src/components/dashboard/sidebar-nav.tsx - Sidebar Pattern]
- [Source: src/components/dashboard/nav-config.ts - Nav Config Pattern]
- [Source: src/components/dashboard/dashboard-header.tsx - Header Pattern]
- [Source: src/lib/api/client.ts - mmsApi() Fallback auf MMS_API_KEY]
- [Source: src/lib/auth.ts - requireAuth() Pattern]
- [Source: .env.example - MMS_API_KEY ist Admin Key]

## Dev Agent Record

### Agent Model Used

Claude Opus 4.6

### Debug Log References

- TypeScript type-check: Session User Type nutzt `contactEmail` statt `email` — Dev Notes Code-Beispiel war inkorrekt, korrigiert zu `session.user.contactEmail`

### Completion Notes List

- Task 1: `isAdmin()` und `requireAdminAuth()` in `src/lib/admin.ts` implementiert. `ADMIN_EMAILS` ENV zu `.env.example` hinzugefügt. Nutzt `contactEmail` statt `email` (Session Type Anpassung).
- Task 2: Admin Layout mit auth() + isAdmin() Guard erstellt. Nicht-eingeloggte → /login, Nicht-Admin → /dashboard redirect.
- Task 3: Desktop-Only Admin Sidebar mit Shield-Icon "Admin" Label, Logo, Navigation (Overview/Tenants/Payments), User Email, Logout.
- Task 4: Admin Header mit Route-Title-Mapping, "Admin" Badge (Shield + Text), ThemeToggle, User Email.
- Task 5: Alle 4 Admin API Proxy Routes implementiert. Jeder Handler nutzt `requireAdminAuth()` vor MMS-Call. Kein apiKey-Argument → automatischer Fallback auf `MMS_API_KEY`.
- Task 6: Drei Stub Pages (Overview, Tenants, Payments) mit Platzhalter-Text für kommende Stories.
- Separate `actions.ts` für Admin-Logout erstellt (Boundary-Regel: keine Dashboard-Imports).

### Review Notes

- **Architecture Doc Inkonsistenz (M3):** `architecture.md` referenziert `MMS_ADMIN_API_KEY` als ENV-Variable für Admin-Calls (Zeilen 294, 314, 620). Die Implementation nutzt `MMS_API_KEY` als Fallback von `mmsApi()` — pragmatisch korrekt (kein separater Admin Key nötig, `MMS_API_KEY` hat bereits `admin:read`, `admin:write` Scopes). Architecture Doc sollte in einem zukünftigen Architektur-Update angepasst werden.

### Change Log

- 2026-02-20: Story 6.1 implementiert — Admin Shell, Auth & Route Protection (alle 6 Tasks)
- 2026-02-20: Code Review — 7 Issues gefunden (2 HIGH, 3 MEDIUM, 2 LOW). HIGH + MEDIUM gefixt: Error Handling in allen Route Handlers, Slug Validation, Input Validation, File List korrigiert, Architecture Inkonsistenz dokumentiert.

### File List

**Neue Dateien:**
- `src/lib/admin.ts`
- `src/app/(admin)/layout.tsx`
- `src/app/(admin)/actions.ts`
- `src/components/admin/admin-nav-config.ts`
- `src/components/admin/admin-sidebar.tsx`
- `src/components/admin/admin-header.tsx`
- `src/app/(admin)/admin/tenants/page.tsx`
- `src/app/(admin)/admin/payments/page.tsx`
- `src/app/api/admin/health/route.ts`
- `src/app/api/admin/tenants/route.ts`
- `src/app/api/admin/tenants/[slug]/route.ts`
- `src/app/api/admin/payments/route.ts`

**Modifizierte Dateien:**
- `.env.example` (ADMIN_EMAILS hinzugefügt)
- `src/app/(admin)/admin/page.tsx` (Stub überarbeitet)
- `docs/_masemIT/api-key-architecture.md` (Admin/Tenant Key Architektur-Doku aktualisiert)
