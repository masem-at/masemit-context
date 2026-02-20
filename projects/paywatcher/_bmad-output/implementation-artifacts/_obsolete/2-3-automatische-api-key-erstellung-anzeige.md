# Story 2.3: Automatische API Key Erstellung & Anzeige

Status: done

<!-- Note: Validation is optional. Run validate-create-story for quality check before dev-story. -->

## Story

As a neu registrierter Tenant,
I want bei meinem ersten Login automatisch einen API Key erhalten und einmalig im Klartext sehen,
so that ich sofort mit der PayWatcher API arbeiten kann.

## Acceptance Criteria

1. **Given** ich logge mich zum ersten Mal ein (FR11) **When** das Dashboard laedt **Then** wird automatisch ein paywatcher_config Eintrag im MMS Backend erstellt (tenant_id = party_id) **And** ein API Key wird generiert und einmalig im Klartext angezeigt

2. **Given** mein API Key zum ersten Mal angezeigt wird **When** ich den Key sehe **Then** wird er prominent dargestellt mit einem fetten Copy-Button **And** eine Warnung zeigt: "Save this — you won't see it again in full." **And** der Key wird in Geist Mono angezeigt

3. **Given** ich den API Key kopiert habe (FR12) **When** ich das Dashboard spaeter erneut besuche **Then** sehe ich den Key nur noch masked (z.B. pw_live_k7x...vT4q) **And** der Copy-Button kopiert die masked Version **And** der Key ist im MMS Backend gehasht gespeichert (NFR6)

## Tasks / Subtasks

- [x] Task 1: API Key Types und Tenant Types definieren (AC: #1, #2, #3)
  - [x] 1.1 `src/types/tenant.ts` erstellen — `TenantConfig` Interface (tenantId, apiKeyMasked, apiKeyPrefix, scopes, depositAddress, confirmations, webhookUrl, createdAt), `ApiKeyResponse` Interface (apiKey: string — Klartext bei Erstellung), `ApiKeyInfo` Interface (masked: string, prefix: string, scopes: string[], createdAt: string)
  - [x] 1.2 `src/types/auth.ts` erweitern — `isNewUser` Flag zum AuthUser/Session hinzufuegen (oder alternativ: Erkennung via fehlenden paywatcher_config)
  - [x] 1.3 `src/types/next-auth.d.ts` erweitern — Session/JWT um `tenantConfigured?: boolean` augmentieren

- [x] Task 2: API Proxy Routes fuer paywatcher_config und API Keys (AC: #1, #3)
  - [x] 2.1 `src/app/api/tenant/route.ts` erstellen — GET: Tenant Config vom MMS Backend holen (GET /v1/paywatcher/config mit tenant_id aus Session), POST: paywatcher_config erstellen (POST /v1/paywatcher/config mit tenant_id = party_id). Beide via `mmsApi()`, snake_case → camelCase Transformation, Envelope Format
  - [x] 2.2 `src/app/api/api-keys/route.ts` erstellen — GET: aktuellen API Key Info holen (masked), POST: neuen API Key generieren (gibt Klartext zurueck). Auth-Check via `auth()` in jedem Handler
  - [x] 2.3 Error Handling: 404 bei nicht existierendem Config = "neuer User" (kein Fehler, Trigger fuer Erstellung), MMS_ERROR bei Backend-Problemen

- [x] Task 3: TanStack Query Hooks fuer Tenant und API Key (AC: #1, #3)
  - [x] 3.1 `src/hooks/use-tenant.ts` erstellen — `useTenant()` Hook mit Query Key `['tenant']`, staleTime 5min, GET `/api/tenant`
  - [x] 3.2 `src/hooks/use-api-key.ts` erstellen — `useApiKey()` Hook mit Query Key `['api-key']`, staleTime 5min, GET `/api/api-keys`. `useCreateApiKey()` Mutation mit POST `/api/api-keys`, bei Erfolg `['api-key']` und `['tenant']` invalidieren
  - [x] 3.3 `useProvisionTenant()` Mutation — POST `/api/tenant` fuer Ersteinrichtung, bei Erfolg `['tenant']` invalidieren

- [x] Task 4: First-Login Detection und Auto-Provisioning (AC: #1)
  - [x] 4.1 `src/hooks/use-first-login.ts` erstellen — Hook der prueft ob Tenant bereits konfiguriert ist (via `useTenant()` — 404 = neuer User), steuert den Auto-Provisioning Flow
  - [x] 4.2 Auto-Provisioning Logik: Wenn kein Tenant Config existiert → automatisch POST `/api/tenant` (paywatcher_config erstellen) → dann POST `/api/api-keys` (API Key generieren) → Klartext-Key im lokalen State halten (useState, NICHT in Query Cache)
  - [x] 4.3 Klartext-Key wird NUR im Component-State gehalten — bei Page Refresh oder Navigation ist er weg (gewollt — NFR6)

- [x] Task 5: API Key Card Komponente (AC: #2, #3)
  - [x] 5.1 `src/components/dashboard/api-key-card.tsx` erstellen — Zwei Modi: "first-reveal" (Klartext + Warning) und "masked" (masked Key + Copy)
  - [x] 5.2 First-Reveal Modus: Key in Geist Mono, prominenter Copy-Button (Primary-Stil), Warning-Banner "Save this — you won't see it again in full." mit Warn-Icon, Card mit Accent-Border
  - [x] 5.3 Masked Modus: Key masked anzeigen (z.B. `pw_live_k7x...vT4q`) in Geist Mono, Copy-Button (Icon-Only, Inline-Checkmark — kein Toast), Read-Only
  - [x] 5.4 Copy-Feedback: Bestehende `copy-button.tsx` Pattern aus `components/shared/` verwenden (Inline-Checkmark, 1.5s, kein Toast)
  - [x] 5.5 Skeleton Loading State waehrend API Key geladen wird

- [x] Task 6: Dashboard Overview Integration (AC: #1, #2)
  - [x] 6.1 `src/app/(dashboard)/dashboard/page.tsx` erweitern — Bestehenden Placeholder durch API Key Card ersetzen fuer neue User (First-Login-Flow), bestehende User sehen weiterhin Overview-Placeholder (wird in Story 3.2 zum Onboarding/Stats Screen)
  - [x] 6.2 Flow: Dashboard laedt → `useFirstLogin()` prueft Status → neuer User: Auto-Provisioning → API Key Card im "first-reveal" Modus → nach Navigation/Refresh: masked Modus
  - [x] 6.3 Loading State: Skeleton waehrend Provisioning laeuft (Sidebar + Header stehen sofort)

- [x] Task 7: Build-Validierung und Qualitaetspruefung (Alle ACs)
  - [x] 7.1 `npm run build` muss fehlerfrei durchlaufen
  - [x] 7.2 `npm run type-check` muss fehlerfrei durchlaufen
  - [x] 7.3 `npm run lint` muss fehlerfrei durchlaufen
  - [x] 7.4 Verifizieren: Landing Page bleibt `○ (Static)` im Build Output
  - [x] 7.5 Verifizieren: API Key Card rendert korrekt in beiden Modi (first-reveal + masked)
  - [x] 7.6 Verifizieren: Auto-Provisioning Flow funktioniert (Type-Check, kein Runtime-Test ohne MMS)
  - [x] 7.7 Verifizieren: Copy-Button funktioniert mit Inline-Checkmark (kein Toast)

## Dev Notes

### Kontext & Zweck

Dies ist die **dritte Story von Epic 2** (User Authentication & API Key Management). Story 2.1 hat die Dashboard Shell, Auth.js v5, Route Protection, MMS API Client und TanStack Query geliefert. Story 2.2 hat den Magic Link Signup/Login Flow mit MMS Parties API Integration implementiert. Diese Story schliesst die Luecke: Nach dem Login bekommt der neue Tenant automatisch seinen API Key.

**Was PayWatcher ist:** Developer-First Verification API fuer Stablecoin-Zahlungen (USDC auf Base Network). Non-custodial, reiner Verification Layer. Das Frontend ist ein reiner API-Consumer — kein eigenes Backend, keine eigene Datenbank. Alle Daten kommen vom MMS Backend (api.masem.at).

**Was diese Story liefert:**
- Automatische paywatcher_config Erstellung im MMS Backend beim ersten Login
- API Key Generierung und einmalige Klartext-Anzeige
- API Key Card Komponente (first-reveal + masked Modi)
- API Proxy Routes fuer Tenant Config und API Keys
- TanStack Query Hooks fuer Tenant und API Key Daten
- First-Login Detection und Auto-Provisioning Flow
- Dashboard Overview Integration mit API Key Display

**Was diese Story NICHT beinhaltet:**
- API Key Regenerate und Scopes (Story 2.4)
- Onboarding Screen mit curl-Beispiel, Deposit Address, "Waiting for first payment" (Story 3.2)
- Webhook Configuration (Epic 4)
- Settings-Seiten Layout (Epic 4)

**Abhaengigkeiten:**
- Story 2.1 komplett abgeschlossen (Dashboard Shell, mmsApi(), Auth.js v5, TanStack Query, proxy.ts)
- Story 2.2 komplett abgeschlossen (Magic Link, Credentials Provider, MMS Parties API, Login/Signup Seiten)
- MMS Backend muss paywatcher_config Endpoints bereitstellen (POST/GET /v1/paywatcher/config)
- MMS Backend muss API Key Endpoints bereitstellen (POST/GET /v1/paywatcher/api-keys)

### Technische Anforderungen

**Zu installierende Packages:**
- Keine neuen Packages noetig — alle Abhaengigkeiten sind bereits vorhanden

**Bereits installierte und verwendbare Packages:**
- `next-auth@beta` v5 (Auth.js v5) — Session mit partyId fuer Tenant-Identifikation
- `@tanstack/react-query` v5 — Hooks fuer Tenant und API Key Daten
- `@tanstack/react-query-devtools` — DevTools fuer Development
- Alle shadcn/ui Komponenten (Card, Button, Badge, Skeleton, etc.)
- `lucide-react` — Icons (Key, Copy, AlertTriangle, Check, Eye, EyeOff)
- `sonner` — Toasts (NUR fuer Fehler, NICHT fuer Copy-Aktionen)
- `next-themes` — Theme Support
- `geist` — Geist Mono fuer API Key Anzeige

**NICHT in dieser Story installieren:**
- `react-hook-form` / `zod` (als Form-Library) — kein Formular in dieser Story, nur Display
- `recharts` — keine Charts
- Keine neuen shadcn/ui Komponenten noetig (Card, Button, Skeleton, Badge bereits vorhanden)

### KRITISCH: MMS Backend API Vertrag fuer paywatcher_config

**Tenant Config erstellen (First Login):**
```
POST /v1/paywatcher/config
Headers: x-api-key: {MMS_API_KEY}, Content-Type: application/json
Body: { "tenant_id": "{party_id}" }

Response (201 — Config erstellt):
{
  "data": {
    "tenant_id": "uuid",
    "api_key": "pw_live_k7xR9m2nP5qL8wJ3...",   ← Klartext, einmalig!
    "api_key_masked": "pw_live_k7x...vT4q",
    "scopes": ["payments:read", "payments:write"],
    "deposit_address": "0x...",
    "confirmations": 6,
    "webhook_url": null,
    "created_at": "2026-02-16T10:00:00Z"
  }
}
```

**Tenant Config abrufen (nachfolgende Besuche):**
```
GET /v1/paywatcher/config?tenant_id={party_id}
Headers: x-api-key: {MMS_API_KEY}

Response (200 — Config existiert):
{
  "data": {
    "tenant_id": "uuid",
    "api_key_masked": "pw_live_k7x...vT4q",
    "scopes": ["payments:read", "payments:write"],
    "deposit_address": "0x...",
    "confirmations": 6,
    "webhook_url": null,
    "created_at": "2026-02-16T10:00:00Z"
  }
}

Response (404 — Config existiert nicht):
{ "error": { "code": "NOT_FOUND", "message": "No config found for tenant" } }
```

**WICHTIG:** Der Klartext API Key wird NUR in der POST Response (Erstellung) zurueckgegeben. Bei GET kommt nur `api_key_masked`. Das ist by design (NFR6: Keys gehasht gespeichert).

**HINWEIS:** Falls die MMS Endpoints noch nicht exakt so existieren, muss der Dev die Route Handler als Stub implementieren mit TODO-Kommentaren und die korrekte Endpoint-Struktur mit dem MMS Backend-Team abstimmen. Die Frontend-Logik (Types, Hooks, Components) sollte unabhaengig davon korrekt implementiert werden.

### Auto-Provisioning Flow Architektur

```
┌─────────────────────────────────────────────────────────────┐
│  Browser — Dashboard Overview Page                           │
│                                                              │
│  useFirstLogin() Hook                                       │
│  ┌──────────────────────┐                                   │
│  │ 1. useTenant() Query │ → GET /api/tenant                 │
│  │    → 404 = neuer User│                                   │
│  │    → 200 = bestehend │                                   │
│  └──────────┬───────────┘                                   │
│             │ (404: neuer User)                              │
│  ┌──────────▼───────────┐                                   │
│  │ 2. useProvisionTenant│ → POST /api/tenant                │
│  │    Mutation           │   (erstellt Config + API Key)     │
│  └──────────┬───────────┘                                   │
│             │                                                │
│  ┌──────────▼───────────┐                                   │
│  │ 3. Klartext-Key in   │                                   │
│  │    useState() halten  │ ← aus POST Response              │
│  └──────────┬───────────┘                                   │
│             │                                                │
│  ┌──────────▼───────────┐                                   │
│  │ 4. ApiKeyCard rendern│                                   │
│  │    mode="first-reveal"│ → Key im Klartext + Warning      │
│  └──────────────────────┘                                   │
│                                                              │
│  Nach Refresh/Navigation:                                   │
│  ┌──────────────────────┐                                   │
│  │ useTenant() → 200    │                                   │
│  │ useApiKey() → masked │                                   │
│  │ ApiKeyCard            │                                   │
│  │ mode="masked"         │ → Key masked + Copy              │
│  └──────────────────────┘                                   │
└─────────────────────────────────────────────────────────────┘
              │                    │
              ▼                    ▼
┌─────────────────────────────────────────────────────────────┐
│  Next.js Route Handlers                                      │
│                                                              │
│  GET /api/tenant                                            │
│  → auth() Session Check                                     │
│  → mmsApi('/v1/paywatcher/config?tenant_id={partyId}')     │
│  → snakeToCamel() Transformation                            │
│  → { data: TenantConfig } oder 404                          │
│                                                              │
│  POST /api/tenant                                           │
│  → auth() Session Check                                     │
│  → mmsApi('/v1/paywatcher/config', { method: 'POST',       │
│      body: { tenant_id: partyId } })                        │
│  → snakeToCamel() Transformation                            │
│  → { data: TenantConfig + apiKey (Klartext) }              │
│                                                              │
│  GET /api/api-keys                                          │
│  → auth() Session Check                                     │
│  → Liest api_key_masked aus Tenant Config                   │
│  → { data: ApiKeyInfo }                                     │
└─────────────────────────────────────────────────────────────┘
              │
              ▼
┌─────────────────────────────────────────────────────────────┐
│  MMS Backend (api.masem.at)                                  │
│  /v1/paywatcher/config (POST, GET)                          │
│  API Key wird server-side generiert und gehasht gespeichert │
└─────────────────────────────────────────────────────────────┘
```

### API Key Card Komponente — Spezifikation

**Zwei Modi:**

| Modus | Wann | Darstellung |
|-------|------|-------------|
| `first-reveal` | Einmalig nach Erstanmeldung (Key im Component-State) | Key im Klartext, Geist Mono, prominenter Copy-Button (Primary), Warning-Banner mit AlertTriangle Icon |
| `masked` | Alle nachfolgenden Besuche | Key masked (`pw_live_k7x...vT4q`), Geist Mono, Copy-Button (Icon-Only, Inline-Checkmark) |

**First-Reveal Modus Layout:**
```
┌──────────────────────────────────────────────┐
│  ⚠️  Save this — you won't see it again     │
│      in full.                                │
├──────────────────────────────────────────────┤
│                                              │
│  Your API Key                                │
│                                              │
│  ┌────────────────────────────────────────┐  │
│  │ pw_live_k7xR9m2nP5qL8wJ3dF6gH...     │  │
│  │                              [Copy] ■  │  │
│  └────────────────────────────────────────┘  │
│                                              │
│  Use this key in your x-api-key header.     │
│                                              │
└──────────────────────────────────────────────┘
```

**Masked Modus Layout:**
```
┌──────────────────────────────────────────────┐
│  API Key                                     │
│                                              │
│  pw_live_k7x...vT4q              [📋]       │
│                                              │
│  Scopes: payments:read, payments:write       │
└──────────────────────────────────────────────┘
```

**UX-Regeln (aus UX-Spec):**
- Kein Toast fuer Copy — nur Inline-Checkmark am Button (1.5s)
- Key IMMER in Geist Mono (`font-mono`)
- Warning-Banner im first-reveal: gelb/amber Hintergrund, AlertTriangle Icon
- Max 1 Primary Button pro Screen — der Copy-Button im first-reveal ist der Primary
- Skeleton Loading waehrend Daten geladen werden

### Architektur-Konformitaet

**Dateinamen:** `kebab-case.ts` / `kebab-case.tsx` — IMMER.

**Neue/geaenderte Dateien nach Abschluss:**
```
src/
├── app/
│   ├── (dashboard)/
│   │   └── dashboard/
│   │       └── page.tsx                        # AENDERN: Overview mit First-Login Flow + API Key Card
│   └── api/
│       ├── tenant/
│       │   └── route.ts                        # NEU: GET + POST Proxy zu MMS /v1/paywatcher/config
│       └── api-keys/
│           └── route.ts                        # NEU: GET Proxy fuer API Key Info
├── components/
│   └── dashboard/
│       └── api-key-card.tsx                    # NEU: API Key Display (first-reveal + masked)
├── hooks/
│   ├── use-tenant.ts                           # NEU: TanStack Query Hook fuer Tenant Config
│   ├── use-api-key.ts                          # NEU: TanStack Query Hook fuer API Key
│   └── use-first-login.ts                      # NEU: First-Login Detection + Auto-Provisioning
├── types/
│   ├── tenant.ts                               # NEU: TenantConfig, ApiKeyResponse, ApiKeyInfo
│   └── auth.ts                                 # GGFS. AENDERN: tenantConfigured Flag (optional)
```

**Server vs Client Components:**
- Server Components: `tenant/route.ts`, `api-keys/route.ts` (Route Handlers)
- Client Components: `api-key-card.tsx` (braucht useState, Copy-Interaktion), `use-first-login.ts` (Hook mit Mutations)

**ANTI-Patterns:**
- KEIN Klartext-Key im TanStack Query Cache speichern — nur in lokalem useState
- KEIN Klartext-Key in localStorage oder sessionStorage — bei Refresh ist er weg (gewollt)
- KEIN Klartext-Key im JWT/Session Token — zu sensitiv fuer Cookie
- KEIN Toast fuer Copy-Aktionen — nur Inline-Checkmark
- KEIN Full-Page-Spinner waehrend Provisioning — Skeleton im Content, Sidebar+Header stehen
- KEIN direkter fetch zu api.masem.at aus dem Browser — alles ueber /api/ Route Handlers
- KEIN mmsApi() Import in Client Components — nur in Route Handlers
- KEINE PascalCase Dateinamen — immer kebab-case.tsx
- KEIN separater "Onboarding Wizard" — API Key Card wird direkt auf der Overview-Seite gezeigt

### Library & Framework Anforderungen

**Auth.js v5 Session (aus Story 2.1/2.2):**
- `auth()` in Route Handlers fuer Session-Check
- `session.user.partyId` als tenant_id fuer MMS API Calls
- Session ist bereits konfiguriert — keine Aenderungen noetig

**TanStack Query v5 (aus Story 2.1):**
- QueryProvider bereits im Dashboard Layout eingebunden
- `useQuery` fuer Reads, `useMutation` fuer Writes
- Query Key Convention: `['tenant']`, `['api-key']`
- staleTime: 5min fuer Tenant/API Key Daten (aendert sich selten)
- Mutation `onSuccess`: `queryClient.invalidateQueries({ queryKey: ['tenant'] })`

**MMS API Client (aus Story 2.1):**
- `mmsApi<T>(path, options)` — einziger Weg MMS aufzurufen
- `snakeToCamel()` — Transformation im Route Handler
- `MmsApiError` — Error Handling mit Error Codes
- Envelope Format: `{ data }` / `{ error }` beibehalten

**shadcn/ui Komponenten (bereits installiert):**
- `Card` — API Key Card Container
- `Button` — Copy-Button, Actions
- `Skeleton` — Loading State
- `Badge` — Optional fuer Scopes-Anzeige

**Bestehende Shared Components:**
- `src/components/shared/copy-button.tsx` — Copy-to-Clipboard mit Inline-Checkmark Pattern, VERWENDEN
- `src/components/shared/theme-provider.tsx` — Theme Support

### Testing-Anforderungen

1. **Build-Validierung:** `npm run build` muss ohne Fehler durchlaufen
2. **Type-Check:** `npm run type-check` muss ohne Fehler durchlaufen
3. **Lint:** `npm run lint` muss ohne Fehler durchlaufen
4. **SSG-Validierung:** Landing Page muss `○ (Static)` bleiben
5. **API Key Card (first-reveal):** Key wird in Geist Mono angezeigt, Copy-Button funktioniert, Warning-Banner sichtbar
6. **API Key Card (masked):** Masked Key angezeigt, Copy kopiert masked Version, Inline-Checkmark
7. **Route Handlers:** Auth-Check via `auth()`, Envelope Format, snakeToCamel Transformation
8. **TanStack Query Hooks:** Korrekte Query Keys, staleTime, Mutation Invalidation
9. **First-Login Flow:** useTenant 404 → Auto-Provisioning → first-reveal → nach Refresh: masked
10. **Kein Key Leak:** Klartext-Key NUR in useState, verschwindet bei Refresh/Navigation

### Vorherige Story Intelligence

**Story 2.2 (Magic Link Signup/Login) Learnings — KRITISCH:**
- **Credentials Provider** mit id "magic-link" funktioniert — authorize() ruft MMS Parties API auf, gibt User mit partyId zurueck
- **Stateless JWT Tokens** (jose) fuer Magic Link — gleiche Technik koennte theoretisch fuer API Key Tokens verwendet werden, aber hier nicht noetig (Key kommt vom MMS Backend)
- **Rate Limiting:** In-Memory Map Pattern fuer API Routes — koennte auch fuer /api/tenant POST relevant sein (verhindert doppelte Provisioning Requests)
- **signOut** als Server Action in `(dashboard)/actions.ts` — Pattern fuer zukuenftige Server Actions
- **auth-form.tsx** ist Client Component unter `components/landing/` — Component Boundary beachten (API Key Card gehoert in `components/dashboard/`)
- **Zod** ist als Dependency verfuegbar — fuer API Route Request Validation verwenden

**Story 2.1 (Dashboard Shell) Learnings — RELEVANT:**
- **mmsApi()** in `lib/api/client.ts` — EINZIGER MMS Fetch Wrapper, nie raw fetch
- **snakeToCamel()** in `lib/api/transform.ts` — Transformation im Route Handler
- **MmsApiError** in `lib/api/errors.ts` — Error Handling mit Status Code Mapping
- **QueryProvider** nur im Dashboard Layout — korrekt, nicht im Root
- **Dashboard Layout** mit `auth()` Check — Session mit partyId verfuegbar
- **Sidebar Collapse** mit localStorage Persistence — bewaehrtes Pattern
- **Placeholder Pages** existieren: dashboard/page.tsx, payments/page.tsx, settings/page.tsx

**Story 2.1 Debug Learnings:**
- JWT Callback assigned `unknown` to typed fields → proper type narrowing mit `as string` Cast
- `setState` inside `useEffect` → lazy `useState` initializer bevorzugen
- snakeToCamel regex wurde in Code Review gefixt — bestehende Funktion verwenden

**Git Intelligence (aktuelle Commits):**
```
edafda0 Story 2.1: Dashboard shell, Auth.js v5, route protection, MMS API client with code review fixes
260ae0a Story 1.6: SEO, social sharing, analytics integration
95b7f01 Story 1.5: Pricing section, comparison callout, trust signals
```
(Story 2.2 implementiert aber noch nicht committed — alle Story 2.2 Dateien existieren im Working Tree)

**Patterns aus vorherigen Stories:**
- Server Components als Default — Client Components nur wenn noetig (`'use client'`)
- Route Handler Pattern: `auth()` Check, `mmsApi()` Call, `snakeToCamel()`, Envelope Response
- TanStack Query: `useState(() => makeQueryClient())` Pattern, Query Keys hierarchisch
- Component Boundaries: `components/dashboard/` fuer Dashboard-spezifische Components
- Copy-Button: Inline-Checkmark Pattern (`copy-button.tsx`), KEIN Toast
- Skeleton Loading: shadcn/ui `<Skeleton />`, nur im Content-Bereich
- kebab-case Dateinamen: strikt eingehalten

### Aktuelle Technologie-Informationen

**Keine neuen Libraries noetig.** Alle verwendeten Technologien sind aus Story 2.1/2.2 bekannt und stabil:
- Auth.js v5 (next-auth@beta) — Session mit partyId
- TanStack Query v5 — Hooks + Mutations
- shadcn/ui — Card, Button, Skeleton, Badge
- lucide-react — Key, Copy, AlertTriangle, Check Icons

**MMS Backend Endpoint-Annahmen:**
Die exakten MMS Endpoints fuer paywatcher_config muessen mit dem Backend-Team abgestimmt werden. Die Story verwendet folgende Annahmen basierend auf dem Architecture Doc:
- `POST /v1/paywatcher/config` — Config + API Key erstellen
- `GET /v1/paywatcher/config?tenant_id={id}` — Config abrufen
- Falls die Endpoints anders heissen (z.B. `/v1/tenants/config` oder `/v1/api-keys`), muessen die Route Handler Pfade angepasst werden — die Frontend-Logik bleibt gleich

### Project Structure Notes

- **Dashboard Route-Struktur:** Pages liegen unter `(dashboard)/dashboard/` (URL: `/dashboard`), NICHT direkt unter `(dashboard)/` — bestehendes Pattern aus Story 2.1 beibehalten
- **API Key Card** wird auf der Overview-Seite (`/dashboard`) gezeigt, NICHT auf einer separaten Settings-Seite — die Settings-Seite mit Regenerate kommt in Story 2.4
- **Hooks-Verzeichnis** (`src/hooks/`) ist aktuell leer (nur .gitkeep) — diese Story erstellt die ersten echten Hooks
- **Types-Verzeichnis** hat bereits `payment.ts` und `auth.ts` — `tenant.ts` wird neu erstellt
- **Bestehende copy-button.tsx** in `components/shared/` — Pattern VERWENDEN, nicht neu implementieren

### References

- [Source: _bmad-output/planning-artifacts/epics.md#Story 2.3] — User Story und Acceptance Criteria
- [Source: _bmad-output/planning-artifacts/prd.md#FR11-FR12] — Auto API Key, Key Display (masked)
- [Source: _bmad-output/planning-artifacts/prd.md#NFR6] — API Keys gehasht gespeichert, nur bei Erstellung im Klartext
- [Source: _bmad-output/planning-artifacts/architecture.md#Auth & Data Architecture] — paywatcher_config mit tenant_id = party_id, API Key Generierung ueber MMS
- [Source: _bmad-output/planning-artifacts/architecture.md#Signup-Flow] — Schritt 5+6: Config erstellen, Key generieren
- [Source: _bmad-output/planning-artifacts/architecture.md#API Client Pattern] — mmsApi(), Route Handler Format, TanStack Query
- [Source: _bmad-output/planning-artifacts/architecture.md#Structure Patterns] — Dateistruktur, Component Boundaries
- [Source: _bmad-output/planning-artifacts/architecture.md#Component Patterns] — Copy-Button Inline-Checkmark, Skeleton Loading, kein Full-Page-Spinner
- [Source: _bmad-output/planning-artifacts/architecture.md#Enforcement Guidelines] — 10 Regeln + Anti-Patterns
- [Source: _bmad-output/planning-artifacts/ux-design-specification.md#ApiKeyDisplay] — First Login Klartext, danach masked, Copy-Button
- [Source: _bmad-output/planning-artifacts/ux-design-specification.md#Effortless Interactions] — "API Key bekommen: Email eingeben → bestaetigen → Key da"
- [Source: _bmad-output/planning-artifacts/ux-design-specification.md#Emotional Journey] — "Erster API Key: Bestaetigung: Die meinen Developer-First ernst"
- [Source: _bmad-output/implementation-artifacts/2-1-dashboard-shell-route-protection.md] — Dashboard Shell, mmsApi(), Auth.js, TanStack Query, Sidebar
- [Source: _bmad-output/implementation-artifacts/2-2-magic-link-signup-login.md] — Magic Link, Credentials Provider, MMS Parties API, Logout

## Dev Agent Record

### Agent Model Used

Claude Opus 4.6

### Debug Log References

- Keine Fehler waehrend der Implementierung aufgetreten
- Type-Check, Lint und Build alle beim ersten Versuch bestanden

### Completion Notes List

- Task 1: `TenantConfig`, `ApiKeyResponse`, `ApiKeyInfo` Interfaces in `src/types/tenant.ts` erstellt. auth.ts und next-auth.d.ts nicht geaendert — Erkennung neuer User laeuft via fehlenden paywatcher_config (404 Response), nicht via Session-Flag. Dies ist die bevorzugte Architektur laut Story (Alternative).
- Task 2: Route Handler `GET/POST /api/tenant` und `GET /api/api-keys` implementiert. Beide nutzen `auth()` Session Check, `mmsApi()` Client, `snakeToCamel()` Transformation, Envelope Format. Error Handling: 404 wird korrekt durchgereicht, MmsApiError mit Status Code Mapping.
- Task 3: TanStack Query Hooks `useTenant()`, `useApiKey()`, `useProvisionTenant()` erstellt. Query Keys: `['tenant']`, `['api-key']`. staleTime: 5min. Mutation invalidiert beide Query Keys bei Erfolg. 404-Errors werden nicht retried.
- Task 4: `useFirstLogin()` Hook implementiert Auto-Provisioning: useTenant() 404 → Auto POST /api/tenant → Klartext-Key in useState → bei Refresh/Navigation weg (NFR6). Ref-Guard verhindert doppelte Provisioning-Requests.
- Task 5: `ApiKeyCard` Komponente mit zwei Modi: "first-reveal" (Klartext + Warning-Banner + Copy-Button Primary) und "masked" (masked Key + Ghost Icon-Only Copy + Scopes Badges). Inline-Checkmark Copy-Pattern (1.5s, kein Toast). Skeleton Loading State. Geist Mono via `font-mono`.
- Task 6: Dashboard Overview Page als Client Component umgebaut mit First-Login Flow Integration. Bestehende User sehen masked Key + Scopes, neue User sehen first-reveal, Fallback auf Placeholder.
- Task 7: Build, Type-Check und Lint alle bestanden. Landing Page bleibt Static. shadcn/ui Skeleton Komponente nachinstalliert.

### File List

- `src/types/tenant.ts` — NEU: TenantConfig, ApiKeyResponse, ApiKeyInfo Interfaces
- `src/lib/api/types.ts` — GEAENDERT: Shared MmsTenantConfig Interface hinzugefuegt
- `src/app/api/tenant/route.ts` — NEU: GET + POST Proxy zu MMS /v1/paywatcher/config
- `src/app/api/api-keys/route.ts` — NEU: GET Proxy fuer API Key Info (masked)
- `src/hooks/use-tenant.ts` — NEU: useTenant() Query + useProvisionTenant() Mutation
- `src/hooks/use-api-key.ts` — NEU: useApiKey() Query mit enabled-Option
- `src/hooks/use-first-login.ts` — NEU: First-Login Detection + Auto-Provisioning Hook mit Error-State
- `src/components/dashboard/api-key-card.tsx` — NEU: API Key Display (first-reveal + masked Modi)
- `src/components/ui/skeleton.tsx` — NEU: shadcn/ui Skeleton Komponente (via shadcn add)
- `src/app/(dashboard)/dashboard/page.tsx` — GEAENDERT: Placeholder → Client Component mit First-Login Flow + API Key Card + Error-State
- `package.json` — GEAENDERT: shadcn/ui Skeleton Abhängigkeit
- `package-lock.json` — GEAENDERT: Lock-Datei aktualisiert

### Senior Developer Review (AI)

**Reviewer:** Sempre | **Date:** 2026-02-16 | **Outcome:** Approved (after fixes)

**Findings (10 total: 4 High, 4 Medium, 2 Low) — ALL HIGH + MEDIUM fixed:**

- [x] [H1] TenantConfig.apiKeyPrefix existierte nicht in MMS API Response — Feld entfernt aus types/tenant.ts
- [x] [H2] Dashboard OverviewPage zeigte keinen Error-State — Error-Card mit AlertTriangle Icon hinzugefuegt
- [x] [H3] useFirstLogin gab keinen Fehler-State zurueck — error-Feld hinzugefuegt (Provisioning + Tenant-Fehler)
- [x] [H4] Query-Parameter-Injection in tenant/route.ts — encodeURIComponent() in beiden Route Handlers
- [x] [M1] useEffect Dependency-Instabilitaet in use-first-login.ts — provision → mutate (stabile Referenz)
- [x] [M2] Duplizierte MmsTenantConfig in Route Handlers — Shared Interface in lib/api/types.ts extrahiert
- [x] [M3] useApiKey feuerte unnoetig fuer neue User — enabled-Option hinzugefuegt, Dashboard nutzt { enabled: isConfigured }
- [x] [M4] package.json/package-lock.json fehlten in File List — File List aktualisiert
- [ ] [L1] Prefix-Parsing fragil (api-keys/route.ts:36) — Akzeptiert, Fallback vorhanden
- [ ] [L2] Stille Clipboard-Fehler (api-key-card.tsx:29) — Akzeptiert, Clipboard API Limitierung

**Build-Validierung nach Fixes:** type-check ✓, lint ✓, build ✓, Landing Page Static ✓

### Change Log

- 2026-02-16: Code Review — 8 von 10 Issues gefixt (4 High, 4 Medium). TenantConfig Typ korrigiert, Error-States hinzugefuegt, Security-Fix (encodeURIComponent), Hook-Stabilitaet verbessert, DRY-Verletzung behoben, unnoetige API-Calls eliminiert, File List vervollstaendigt
- 2026-02-16: Story 2.3 implementiert — Automatische API Key Erstellung & Anzeige mit Auto-Provisioning Flow, API Proxy Routes, TanStack Query Hooks und API Key Card Komponente
