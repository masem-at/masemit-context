# Story 2.2: API-Key-basierter Dashboard Login

Status: done

## Story

As a Tenant,
I want mich mit meinem API Key im Dashboard einloggen koennen,
So that ich ohne Passwort-Verwaltung auf mein Dashboard zugreifen kann.

## Acceptance Criteria

1. **Given** ich bin auf der Login-Seite (/login) **When** die Seite laedt **Then** sehe ich ein Eingabefeld fuer den API Key mit Placeholder "mms_paywatcher_..." und einen "Sign In" Button

2. **Given** ich gebe meinen API Key ein und klicke Sign In **When** der Key validiert wird **Then** ruft ein Route Handler GET /v1/paywatcher/config mit dem Key als x-api-key Header auf

3. **Given** der Key valide ist (200 Response) **Then** wird eine Session erstellt (JWT mit tenant_slug, tenant_name, contact_email aus der Config Response, verschluesselter API Key) **And** ich werde zum Dashboard redirected

4. **Given** der Key ungueltig ist (401/403) **Then** sehe ich: "Invalid API key. Please check your credentials."

5. **Given** ich nach dem Login eine Dashboard-Route aufrufe **Then** enthaelt die Session den API Key (verschluesselt) fuer MMS-Calls

6. **Given** ich /signup aufrufe **Then** werde ich auf /login redirected

7. **Given** ich eingeloggt bin **When** ich auf Logout klicke **Then** wird die Session beendet und ich werde zur Landing Page redirected

## Tasks / Subtasks

- [x] Task 1: Auth.js Credentials Provider mit API-Key-Validierung (AC: #2, #3, #4)
  - [x] 1.1 `src/lib/auth.ts` — Credentials Provider mit id "api-key", authorize() validiert Key via GET /v1/paywatcher/config
  - [x] 1.2 API Key Verschluesselung — EncryptJWT/jwtDecrypt mit AUTH_SECRET, Key wird verschluesselt im JWT gespeichert
  - [x] 1.3 JWT/Session Callbacks — tenantSlug, tenantName, contactEmail in Session verfuegbar
  - [x] 1.4 requireAuth() Helper — extrahiert Session + decrypted API Key fuer Route Handlers
  - [x] 1.5 mmsApi() erweitert — optionaler apiKey Parameter fuer Tenant-spezifische Calls

- [x] Task 2: Login-Seite erstellen (AC: #1, #4)
  - [x] 2.1 `src/app/(landing)/login/page.tsx` — Server Component mit Metadata
  - [x] 2.2 `src/components/landing/auth-form.tsx` — Client Component, API Key Input (type="password"), KeyRound Icon, Placeholder "mms_paywatcher_...", Sign In Button, Error-State, Loading-State
  - [x] 2.3 Link "Don't have an API key? Request access" → /#request-access

- [x] Task 3: Signup-Redirect (AC: #6)
  - [x] 3.1 `src/app/(landing)/signup/page.tsx` — redirect("/login")

- [x] Task 4: Logout-Funktionalitaet (AC: #7)
  - [x] 4.1 `src/app/(dashboard)/actions.ts` — logout Server Action mit signOut({ redirectTo: "/" })
  - [x] 4.2 Logout-Button in sidebar-nav.tsx (Desktop) und mobile-sidebar.tsx (Mobile)

- [x] Task 5: Dashboard Layout Auth-Check (AC: #5)
  - [x] 5.1 `src/app/(dashboard)/layout.tsx` — auth() Check, redirect zu /login wenn kein tenantSlug
  - [x] 5.2 Session-Daten (tenantName, contactEmail) an Header und Sidebar weitergeben

- [x] Task 6: TypeScript Types aktualisieren (AC: #3, #5)
  - [x] 6.1 `src/types/next-auth.d.ts` — Session/JWT Augmentation mit tenantSlug, tenantName, contactEmail, encryptedApiKey
  - [x] 6.2 `src/types/auth.ts` — AuthUser, AuthSession, JwtPayload Interfaces
  - [x] 6.3 `src/lib/api/types.ts` — MmsTenantConfig, MmsApiKeyInfo (snake_case MMS Response Types)

- [x] Task 7: Alte Magic-Link-Artefakte entfernen
  - [x] 7.1 `src/lib/magic-link.ts` — GELOESCHT
  - [x] 7.2 `src/app/api/auth/request-magic-link/route.ts` — GELOESCHT
  - [x] 7.3 `src/app/api/auth/verify/route.ts` — GELOESCHT
  - [x] 7.4 `src/hooks/use-first-login.ts` — GELOESCHT

- [x] Task 8: Build-Validierung (Alle ACs)
  - [x] 8.1 `npm run build` fehlerfrei
  - [x] 8.2 Landing Page bleibt Static
  - [x] 8.3 /login und /signup Routen korrekt registriert

## Dev Notes

### Kontext & Zweck

Dies ist die **zweite Story von Epic 2** (Authentication & API Key Management) im **Managed MVP** Ansatz. Statt Self-Service Signup mit Magic Link (alter Ansatz) loggen sich Tenants jetzt mit dem API Key ein, den sie von Mario (Operator) per Email erhalten haben.

**Was PayWatcher ist:** Developer-First Verification API fuer Stablecoin-Zahlungen (USDC auf Base Network). Non-custodial, reiner Verification Layer. Das Frontend ist ein reiner API-Consumer gegen das MMS Backend (api.masem.at).

**Managed MVP Ansatz (CR-PW-001):**
- Kein Self-Service Signup — Mario legt Tenants manuell im MMS Backend an
- Kein Magic Link — Tenant gibt seinen API Key direkt ein
- Kein MMS Parties API — keine User-Registrierung im Frontend
- API Key wird in der Session verschluesselt gespeichert und fuer alle MMS-Calls verwendet

**Abhaengigkeiten:**
- Story 2.1 komplett (Dashboard Shell, Auth.js v5 Stub, Route Protection, MMS API Client, TanStack Query)
- MMS Backend (api.masem.at) muss GET /v1/paywatcher/config mit Tenant-API-Key unterstuetzen

### API-Key Login Flow

```
Browser                          Next.js Server                    MMS Backend
   │                                  │                                │
   │  API Key eingeben + Sign In      │                                │
   │─────────────────────────────────>│                                │
   │  signIn("api-key", { apiKey })   │                                │
   │                                  │  GET /v1/paywatcher/config     │
   │                                  │  x-api-key: {tenant-api-key}   │
   │                                  │───────────────────────────────>│
   │                                  │  200: { data: TenantConfig }   │
   │                                  │<───────────────────────────────│
   │                                  │                                │
   │                                  │  Encrypt API Key (EncryptJWT)  │
   │                                  │  Create JWT Session            │
   │  Set Cookie + Redirect /dashboard│                                │
   │<─────────────────────────────────│                                │
```

### API Key Verschluesselung

Der Tenant-API-Key wird mit `EncryptJWT` (jose) verschluesselt im JWT-Cookie gespeichert. Route Handler entschluesseln ihn via `requireAuth()` und verwenden ihn fuer MMS-Calls. So muss kein globaler MMS_API_KEY fuer Tenant-Operationen verwendet werden — jeder Tenant nutzt seinen eigenen Key.

### References

- [Source: _bmad-output/planning-artifacts/epics.md#Story 2.2]
- [Source: _bmad-output/planning-artifacts/prd.md#FR9, FR10, FR37]
- [Source: _bmad-output/planning-artifacts/architecture.md]
- [Source: docs/_masemIT/requirements/cr-pw-001-managed-mvp-2026-02-16.md]

## Dev Agent Record

### Agent Model Used

Unknown (vorherige Session, nicht dokumentiert)

### Completion Notes List

- Migration von Magic Link zu API-Key-basiertem Login in einer vorherigen Session durchgefuehrt
- Alle Magic-Link-Artefakte entfernt (magic-link.ts, request-magic-link route, verify route, use-first-login hook)
- Auth.js Provider von "magic-link" zu "api-key" gewechselt
- requireAuth() Helper fuer Route Handlers erstellt
- mmsApi() um optionalen apiKey Parameter erweitert
- Session-Struktur von partyId/email/displayName zu tenantSlug/tenantName/contactEmail geaendert
- Build fehlerfrei

### Change Log

- 2026-02-18: Code Review (AI) — 7 Fixes (H1: Encryption Secret Null-Fallback, H2: TenantConfig confirmationOverride, M1+M2: Shared Client ApiError, M3+M4: File List, L1: Login Metadata)
- 2026-02-18: Story-File neu geschrieben fuer Managed MVP (alte Story beschrieb Magic Link Flow)
- ~2026-02-17: Code-Migration von Magic Link zu API-Key Login (nicht committed, nachtraeglich committed am 2026-02-18)
- 2026-02-16: Code Review (AI) der alten Magic Link Implementierung (8 Fixes)
- 2026-02-16: Alte Story 2.2 (Magic Link Signup & Login) implementiert (obsolet)

### File List

- src/lib/auth.ts (GEAENDERT) — Credentials Provider "api-key", EncryptJWT, requireAuth()
- src/lib/api/client.ts (GEAENDERT) — optionaler apiKey Parameter
- src/lib/api/client-error.ts (NEU) — Shared ApiError Klasse fuer Client-Side Hooks
- src/lib/api/types.ts (GEAENDERT) — MmsTenantConfig, MmsApiKeyInfo Interfaces
- src/components/landing/auth-form.tsx (GEAENDERT) — API Key Input statt Email + Magic Link
- src/app/(landing)/login/page.tsx (GEAENDERT) — Login Seite, Metadata Beschreibung korrigiert
- src/app/(landing)/signup/page.tsx (BESTEHEND) — redirect("/login"), bereits in cd9c9d6 geaendert
- src/app/(dashboard)/layout.tsx (GEAENDERT) — Auth-Check mit tenantSlug
- src/app/(dashboard)/dashboard/page.tsx (GEAENDERT) — Overview Page mit ApiKeyCard
- src/app/(dashboard)/dashboard/settings/api-keys/page.tsx (GEAENDERT) — API Key Anzeige
- src/app/(dashboard)/actions.ts (BESTEHEND) — logout Server Action
- src/app/api/api-keys/route.ts (GEAENDERT) — API Key Route Handler mit requireAuth
- src/app/api/tenant/route.ts (GEAENDERT) — Tenant Config Route Handler mit requireAuth
- src/components/dashboard/sidebar-nav.tsx (BESTEHEND) — Logout-Button
- src/components/dashboard/mobile-sidebar.tsx (BESTEHEND) — Logout-Button
- src/components/dashboard/dashboard-header.tsx (GEAENDERT) — tenantName Anzeige
- src/hooks/use-api-key.ts (GEAENDERT) — TanStack Query Hook, nutzt shared ApiError
- src/hooks/use-tenant.ts (GEAENDERT) — TanStack Query Hook, nutzt shared ApiError
- src/types/next-auth.d.ts (GEAENDERT) — tenantSlug/tenantName/contactEmail/encryptedApiKey
- src/types/auth.ts (GEAENDERT) — AuthUser, AuthSession, JwtPayload
- src/types/tenant.ts (GEAENDERT) — TenantConfig (inkl. confirmationOverride), ApiKeyInfo
- .env.example (GEAENDERT) — Kommentare fuer Managed MVP aktualisiert
- src/lib/magic-link.ts (GELOESCHT)
- src/app/api/auth/request-magic-link/route.ts (GELOESCHT)
- src/app/api/auth/verify/route.ts (GELOESCHT)
- src/hooks/use-first-login.ts (GELOESCHT)
