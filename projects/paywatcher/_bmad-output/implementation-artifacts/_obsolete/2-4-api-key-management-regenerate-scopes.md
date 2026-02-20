# Story 2.4: API Key Management (Regenerate & Scopes)

Status: done

<!-- Note: Validation is optional. Run validate-create-story for quality check before dev-story. -->

## Story

As a Tenant,
I want meinen API Key regenerieren und dessen Scopes einsehen koennen,
so that ich bei einem Key-Leak schnell handeln kann und weiss welche Berechtigungen mein Key hat.

## Acceptance Criteria

1. **Given** ich bin auf der Settings/API Keys Seite (FR13) **When** ich auf "Regenerate API Key" klicke **Then** erscheint ein Confirmation Dialog (Danger-Pattern): "This will immediately invalidate your current key. Any integrations using the old key will stop working." **And** der Dialog hat "Cancel" (Ghost) und "Regenerate" (Danger-Button)

2. **Given** ich bestaetige die Regenerierung **When** der neue Key generiert wird **Then** wird der alte Key sofort invalidiert **And** der neue Key wird einmalig im Klartext angezeigt (wie beim Erstlogin) **And** danach nur noch masked

3. **Given** ich bin auf der API Key Seite (FR14) **When** ich die Scopes-Anzeige sehe **Then** sehe ich die aktiven Scopes meines Keys (z.B. payments:read, payments:write) **And** die Scopes sind read-only (keine manuelle Aenderung)

## Tasks / Subtasks

- [x] Task 1: Settings API Keys Seite erstellen (AC: #1, #2, #3)
  - [x] 1.1 `src/app/(dashboard)/dashboard/settings/api-keys/page.tsx` erstellen — Client Component, zeigt ApiKeyCard im masked-Modus, Scopes-Anzeige (read-only Badges), "Regenerate API Key"-Button (Danger-Variante)
  - [x] 1.2 Settings-Navigation pruefen — Sidebar-Link "Settings" existiert bereits (navigiert zu `/dashboard/settings`), ggf. Settings-Seite als Redirect oder mit Sub-Navigation (API Keys, Webhooks, Account) vorbereiten. Fuer diese Story reicht die API Keys-Unterseite
  - [x] 1.3 `src/app/(dashboard)/dashboard/settings/page.tsx` aktualisieren — Placeholder durch Redirect zu `/dashboard/settings/api-keys` oder Settings-Overview mit Links zu Unterbereichen ersetzen

- [x] Task 2: API Key Regenerate Endpoint (AC: #1, #2)
  - [x] 2.1 `src/app/api/api-keys/route.ts` erweitern — POST Handler hinzufuegen: Auth-Check via `auth()`, Request an MMS Backend `POST /v1/paywatcher/api-keys/regenerate` (oder `POST /v1/paywatcher/config/regenerate-key`) mit tenant_id aus Session, Klartext-Key aus Response zurueckgeben, snakeToCamel Transformation, Envelope Format
  - [x] 2.2 Error Handling: MMS_ERROR bei Backend-Problemen, UNAUTHORIZED wenn keine Session, rate limiting (optionaler Guard gegen Mehrfach-Regenerierung)

- [x] Task 3: Regenerate Mutation Hook (AC: #1, #2)
  - [x] 3.1 `src/hooks/use-api-key.ts` erweitern — `useRegenerateApiKey()` Mutation hinzufuegen: POST `/api/api-keys`, bei Erfolg `['api-key']` und `['tenant']` Query Keys invalidieren, Klartext-Key aus Response zurueckgeben
  - [x] 3.2 Mutation State Management: Loading-State fuer Button, Error-State fuer Fehlermeldung, Success-State fuer Klartext-Key Anzeige

- [x] Task 4: Confirmation Dialog Komponente (AC: #1)
  - [x] 4.1 shadcn/ui Dialog (AlertDialog) installieren falls noch nicht vorhanden — `npx shadcn@latest add alert-dialog`
  - [x] 4.2 Confirmation Dialog in der Settings API Keys Seite implementieren: Trigger = "Regenerate API Key" Button (Danger-Variante, outline, rote Border), Dialog Headline: "Regenerate API Key?", Dialog Body: "This will immediately invalidate your current key. Any integrations using the old key will stop working.", Buttons: "Cancel" (Ghost) + "Regenerate" (Danger, filled rot)
  - [x] 4.3 Loading-State im Dialog: "Regenerate"-Button zeigt Spinner waehrend der Regenerierung, Dialog schliesst NICHT automatisch bei Loading

- [x] Task 5: Post-Regenerate Klartext-Anzeige (AC: #2)
  - [x] 5.1 Nach erfolgreicher Regenerierung: Dialog schliessen, ApiKeyCard in "first-reveal" Modus umschalten (neuer Klartext-Key), Klartext-Key in useState halten (NICHT im Query Cache)
  - [x] 5.2 Warning-Banner anzeigen: "Save this — you won't see it again in full." (gleicher Stil wie Story 2.3 First-Login)
  - [x] 5.3 Nach Navigation/Refresh: ApiKeyCard zurueck in "masked" Modus (Klartext-Key verloren — gewollt, NFR6)
  - [x] 5.4 Toast bei Erfolg: "API Key regenerated ✓" (rechts unten, 3s auto-dismiss)

- [x] Task 6: Scopes-Anzeige (AC: #3)
  - [x] 6.1 Scopes werden bereits im masked-Modus der ApiKeyCard als Badges angezeigt (aus Story 2.3) — sicherstellen dass sie auch auf der Settings-Seite korrekt erscheinen
  - [x] 6.2 Scopes sind read-only, kein Management-UI noetig — vorkonfiguriert via MMS Backend (payments:read, payments:write)
  - [x] 6.3 Falls keine Scopes vorhanden (Edge Case): dezenter Hinweis anzeigen

- [x] Task 7: Build-Validierung und Qualitaetspruefung (Alle ACs)
  - [x] 7.1 `npm run build` muss fehlerfrei durchlaufen
  - [x] 7.2 `npm run type-check` muss fehlerfrei durchlaufen
  - [x] 7.3 `npm run lint` muss fehlerfrei durchlaufen
  - [x] 7.4 Verifizieren: Landing Page bleibt `○ (Static)` im Build Output
  - [x] 7.5 Verifizieren: Confirmation Dialog oeffnet und schliesst korrekt
  - [x] 7.6 Verifizieren: Nach Regenerierung zeigt ApiKeyCard den neuen Key im Klartext
  - [x] 7.7 Verifizieren: Nach Refresh zeigt ApiKeyCard den masked Key
  - [x] 7.8 Verifizieren: Scopes-Badges werden korrekt angezeigt

## Dev Notes

### Kontext & Zweck

Dies ist die **vierte und letzte Story von Epic 2** (User Authentication & API Key Management). Story 2.1 lieferte Dashboard Shell, Auth.js v5, Route Protection, MMS API Client und TanStack Query. Story 2.2 implementierte den Magic Link Signup/Login Flow mit MMS Parties API Integration. Story 2.3 implementierte die automatische API Key Erstellung und Anzeige (first-reveal + masked Modi). Diese Story schliesst Epic 2 ab mit API Key Regenerierung und Scopes-Einsicht.

**Was PayWatcher ist:** Developer-First Verification API fuer Stablecoin-Zahlungen (USDC auf Base Network). Non-custodial, reiner Verification Layer. Das Frontend ist ein reiner API-Consumer — kein eigenes Backend, keine eigene Datenbank. Alle Daten kommen vom MMS Backend (api.masem.at).

**Was diese Story liefert:**
- Settings/API Keys Seite mit API Key Anzeige (masked + Scopes)
- "Regenerate API Key"-Button mit Confirmation Dialog (Danger-Pattern)
- API Key Regenerierung via POST Endpoint zum MMS Backend
- Einmalige Klartext-Anzeige des neuen Keys (wie beim Erstlogin)
- Scopes-Anzeige als read-only Badges
- Settings-Seitenstruktur als Basis fuer Epic 4 (Webhooks, Account)

**Was diese Story NICHT beinhaltet:**
- Webhook Configuration (Story 4.1)
- Account Settings / Confirmation Override (Story 4.3)
- Deposit Address Anzeige (Story 4.3)
- Onboarding Screen mit curl-Beispiel (Story 3.2)
- API Key Scope-Management (Scopes sind vorkonfiguriert, read-only)

**Abhaengigkeiten:**
- Story 2.3 komplett abgeschlossen (ApiKeyCard, use-api-key.ts, use-tenant.ts, /api/api-keys Route, /api/tenant Route, types/tenant.ts)
- Story 2.1/2.2 komplett abgeschlossen (Dashboard Shell, Auth.js v5, mmsApi(), snakeToCamel, TanStack Query)
- MMS Backend muss Regenerate-Endpoint bereitstellen (POST /v1/paywatcher/api-keys/regenerate oder aehnlich)

### Technische Anforderungen

**Zu installierende Packages/Komponenten:**
- `npx shadcn@latest add alert-dialog` — shadcn/ui AlertDialog fuer Confirmation Dialog (falls noch nicht installiert)
- Keine weiteren neuen Packages noetig

**Bereits installierte und verwendbare Packages:**
- `next-auth@beta` v5 (Auth.js v5) — Session mit partyId fuer Tenant-Identifikation
- `@tanstack/react-query` v5 — Hooks fuer API Key Daten + Mutations
- Alle shadcn/ui Komponenten (Card, Button, Badge, Skeleton, Dialog)
- `lucide-react` — Icons (Key, RefreshCw, Copy, AlertTriangle, Check, Shield)
- `sonner` — Toasts (fuer Regenerate-Erfolg/Fehler, NICHT fuer Copy)
- `next-themes` — Theme Support
- `geist` — Geist Mono fuer API Key Anzeige

**NICHT in dieser Story installieren:**
- `react-hook-form` / `zod` (als Form-Library) — kein Formular in dieser Story, nur Actions + Dialog
- `recharts` — keine Charts
- Keine Payments/Webhook-bezogenen Packages

### KRITISCH: MMS Backend API Vertrag fuer API Key Regenerierung

**API Key regenerieren:**
```
POST /v1/paywatcher/api-keys/regenerate
Headers: x-api-key: {MMS_API_KEY}, Content-Type: application/json
Body: { "tenant_id": "{party_id}" }

Response (200 — Neuer Key generiert, alter invalidiert):
{
  "data": {
    "api_key": "pw_live_n8yT2k5mQ7rJ9wX4...",   <- Klartext, einmalig!
    "api_key_masked": "pw_live_n8y...wX4q",
    "scopes": ["payments:read", "payments:write"],
    "created_at": "2026-02-16T14:30:00Z"
  }
}

Response (404 — Tenant nicht gefunden):
{ "error": { "code": "NOT_FOUND", "message": "No config found for tenant" } }
```

**WICHTIG:** Der Klartext API Key wird NUR in der Regenerate-Response zurueckgegeben. Danach nur noch masked via GET. Der alte Key ist sofort invalidiert — keine Grace Period.

**HINWEIS:** Falls der MMS Endpoint anders heisst (z.B. `POST /v1/paywatcher/config/regenerate-key` oder `PUT /v1/paywatcher/api-keys`), muessen die Route Handler Pfade angepasst werden — die Frontend-Logik bleibt gleich.

### Architektur-Konformitaet

**Dateinamen:** `kebab-case.ts` / `kebab-case.tsx` — IMMER.

**Neue/geaenderte Dateien nach Abschluss:**
```
src/
├── app/
│   ├── (dashboard)/
│   │   └── dashboard/
│   │       └── settings/
│   │           ├── page.tsx                        # AENDERN: Placeholder → Redirect/Overview
│   │           └── api-keys/
│   │               └── page.tsx                    # NEU: API Key Management Seite
│   └── api/
│       └── api-keys/
│           └── route.ts                            # AENDERN: POST Handler fuer Regenerate hinzufuegen
├── components/
│   ├── dashboard/
│   │   └── api-key-card.tsx                        # GGFS. AENDERN: Kleinere Anpassungen fuer Settings-Kontext
│   └── ui/
│       └── alert-dialog.tsx                        # NEU: shadcn/ui AlertDialog (via CLI)
├── hooks/
│   └── use-api-key.ts                              # AENDERN: useRegenerateApiKey() Mutation hinzufuegen
```

**Server vs Client Components:**
- Server Components: Erweiterung von `api-keys/route.ts` (Route Handler — POST)
- Client Components: `settings/api-keys/page.tsx` (braucht useState fuer Klartext-Key, Dialog-State, Mutation-Calls)

**ANTI-Patterns:**
- KEIN Klartext-Key im TanStack Query Cache speichern — nur in lokalem useState
- KEIN Klartext-Key in localStorage oder sessionStorage
- KEIN Toast fuer Copy-Aktionen — nur Inline-Checkmark
- KEIN Full-Page-Spinner — Skeleton im Content, Sidebar+Header stehen sofort
- KEIN direkter fetch zu api.masem.at aus dem Browser — alles ueber /api/ Route Handlers
- KEIN mmsApi() Import in Client Components — nur in Route Handlers
- KEINE PascalCase Dateinamen — immer kebab-case.tsx
- KEIN zweiter Primary Button auf der Seite — "Regenerate" ist Danger (Secondary-Ebene), kein Primary
- KEIN Scope-Management-UI — Scopes sind vorkonfiguriert, read-only
- KEIN "Regenerate" ohne Confirmation Dialog — immer Danger-Pattern mit AlertDialog

### Library & Framework Anforderungen

**Auth.js v5 Session (aus Story 2.1/2.2):**
- `auth()` in Route Handlers fuer Session-Check
- `session.user.partyId` als tenant_id fuer MMS API Calls
- Session ist bereits konfiguriert — keine Aenderungen noetig

**TanStack Query v5 (aus Story 2.1):**
- QueryProvider bereits im Dashboard Layout eingebunden
- `useQuery` fuer Reads (bestehend: useApiKey), `useMutation` fuer Writes (neu: useRegenerateApiKey)
- Query Key Convention: `['api-key']`, `['tenant']`
- staleTime: 5min fuer API Key Daten (aendert sich selten)
- Mutation `onSuccess`: `queryClient.invalidateQueries({ queryKey: ['api-key'] })` + `['tenant']`

**MMS API Client (aus Story 2.1):**
- `mmsApi<T>(path, options)` — einziger Weg MMS aufzurufen
- `snakeToCamel()` — Transformation im Route Handler
- `MmsApiError` — Error Handling mit Error Codes
- Envelope Format: `{ data }` / `{ error }` beibehalten

**shadcn/ui Komponenten:**
- `Card` — API Key Card Container (bestehend)
- `Button` — Regenerate (Danger-Variante), Copy (Icon-Only) (bestehend)
- `Skeleton` — Loading State (bestehend)
- `Badge` — Scopes-Anzeige (bestehend)
- `AlertDialog` — Confirmation Dialog (NEU — muss installiert werden)

**Bestehende Shared Components:**
- `src/components/shared/copy-button.tsx` — Copy-to-Clipboard mit Inline-Checkmark Pattern, VERWENDEN
- `src/components/dashboard/api-key-card.tsx` — Bestehende Komponente mit first-reveal + masked Modi, WIEDERVERWENDEN

### Datei-Struktur Anforderungen

**Settings-Seiten Architektur:**
Die Settings-Seite wird in dieser Story grundlegend aufgebaut. Die Architektur soll erweiterbar sein fuer Epic 4 (Webhooks + Account):
```
(dashboard)/dashboard/settings/
├── page.tsx                # Settings Overview oder Redirect
├── api-keys/
│   └── page.tsx            # Story 2.4: API Key Management
├── webhooks/
│   └── page.tsx            # Story 4.1/4.2: Webhook Config + Log (spaeter)
└── account/
    └── page.tsx            # Story 4.3: Confirmations, Address, Email (spaeter)
```

**Settings-Navigation (aus UX-Spec):**
- Content-Bereich max-w-3xl und zentriert
- Settings-Navigation mit den Bereichen: API Keys (diese Story), Webhooks (Epic 4), Account (Epic 4)
- Tabs-Komponente (shadcn/ui) fuer Bereichswechsel ODER einfache Link-Navigation

### Testing-Anforderungen

1. **Build-Validierung:** `npm run build` muss ohne Fehler durchlaufen
2. **Type-Check:** `npm run type-check` muss ohne Fehler durchlaufen
3. **Lint:** `npm run lint` muss ohne Fehler durchlaufen
4. **SSG-Validierung:** Landing Page muss `○ (Static)` bleiben
5. **AlertDialog:** Oeffnet bei Klick auf "Regenerate", schliesst bei "Cancel"
6. **Regenerate Flow:** POST /api/api-keys → neuer Klartext-Key → first-reveal Modus → nach Refresh: masked
7. **Route Handler:** Auth-Check via `auth()`, Envelope Format, snakeToCamel Transformation
8. **TanStack Query:** useRegenerateApiKey Mutation invalidiert ['api-key'] und ['tenant']
9. **Kein Key Leak:** Klartext-Key NUR in useState, verschwindet bei Refresh/Navigation
10. **Scopes-Anzeige:** Badges zeigen payments:read, payments:write (read-only)
11. **Toast:** Erfolg-Toast "API Key regenerated ✓" (3s), Fehler-Toast bleibt bis dismissed
12. **Danger-Button:** Rote Border/Outline, NICHT Primary (Secondary-Ebene)

### Vorherige Story Intelligence

**Story 2.3 (Automatische API Key Erstellung & Anzeige) Learnings — KRITISCH:**
- **ApiKeyCard** existiert bereits in `src/components/dashboard/api-key-card.tsx` — zwei Modi: "first-reveal" (Klartext + Warning-Banner + Primary Copy) und "masked" (masked Key + Ghost Copy + Scopes Badges). Dieses Pattern WIEDERVERWENDEN fuer Post-Regenerate Klartext-Anzeige.
- **use-api-key.ts** existiert in `src/hooks/use-api-key.ts` — `useApiKey()` Query Hook mit Query Key `['api-key']`, staleTime 5min, enabled-Option. ERWEITERN mit `useRegenerateApiKey()` Mutation.
- **Route Handler** `src/app/api/api-keys/route.ts` hat GET Handler — ERWEITERN mit POST Handler fuer Regenerate.
- **Klartext-Key Handling:** Key wird NUR in useState gehalten, NICHT im Query Cache. Bei Refresh/Navigation weg (gewollt, NFR6). Gleicher Ansatz fuer Regenerate.
- **Code Review Fixes aus 2.3:** encodeURIComponent fuer Query-Parameter, enabled-Option bei useApiKey (verhindert unnoetige Calls), Error-State Handling in Dashboard.
- **Copy-Button:** `src/components/shared/copy-button.tsx` — Inline-Checkmark Pattern (1.5s, kein Toast). VERWENDEN.
- **Skeleton Loading:** shadcn/ui Skeleton Komponente ist installiert und wird verwendet.
- **snakeToCamel in Route Handlers:** Bestehender Shared `MmsTenantConfig` in `lib/api/types.ts` — neuen Response-Type dort hinzufuegen.

**Story 2.1/2.2 Patterns — BEACHTEN:**
- **mmsApi()** in `lib/api/client.ts` — EINZIGER MMS Fetch Wrapper, nie raw fetch
- **Credentials Provider** mit id "magic-link" — irrelevant fuer diese Story, aber Session-Handling gleich
- **Rate Limiting:** In-Memory Map Pattern — GGFS. auch fuer Regenerate-Endpoint anwenden (verhindert Spam)
- **Zod** ist als Dependency verfuegbar — fuer Request Validation in Route Handler verwenden
- **sonner** Toast Pattern: `toast.success()` fuer Erfolg (3s auto), `toast.error()` fuer Fehler (bleibt)

**Bestehende Settings-Seite:**
- `src/app/(dashboard)/dashboard/settings/page.tsx` ist aktuell ein Placeholder ("Settings kommt in Epic 4")
- Muss in dieser Story aktualisiert werden (entweder Redirect oder Overview mit Links)
- Sidebar-Navigation hat bereits "Settings" Link der zu `/dashboard/settings` navigiert

### Aktuelle Technologie-Informationen

**Keine neuen Libraries noetig.** Einzige Ergaenzung: shadcn/ui AlertDialog Komponente via CLI installieren.

Alle verwendeten Technologien sind aus Story 2.1-2.3 bekannt und stabil:
- Auth.js v5 (next-auth@beta) — Session mit partyId
- TanStack Query v5 — Hooks + Mutations
- shadcn/ui — Card, Button, Skeleton, Badge, AlertDialog (neu)
- lucide-react — Icons
- sonner — Toasts

**MMS Backend Endpoint-Annahmen:**
Die exakten MMS Endpoints fuer API Key Regenerierung muessen mit dem Backend-Team abgestimmt werden. Die Story verwendet folgende Annahmen:
- `POST /v1/paywatcher/api-keys/regenerate` mit `{ "tenant_id": "{party_id}" }` — regeneriert Key, invalidiert alten sofort
- Falls der Endpoint anders heisst (z.B. `POST /v1/paywatcher/config/regenerate-key`), muessen die Route Handler Pfade angepasst werden — die Frontend-Logik bleibt gleich

### Project Structure Notes

- **Dashboard Route-Struktur:** Pages liegen unter `(dashboard)/dashboard/` (URL: `/dashboard`), bestehendes Pattern aus Story 2.1
- **Settings Sub-Routes:** `(dashboard)/dashboard/settings/api-keys/` (URL: `/dashboard/settings/api-keys`) — neue Unterseite
- **ApiKeyCard Wiederverwendung:** Die bestehende Komponente aus Story 2.3 wird auf der Settings-Seite wiederverwendet, NICHT dupliziert
- **Settings-Seiten Vorbereitung:** Die Settings-Struktur wird so angelegt, dass Story 4.1 (Webhooks) und 4.3 (Account) einfach als weitere Unterseiten hinzugefuegt werden koennen
- **Dashboard Overview:** Die bestehende Overview-Seite (`/dashboard`) zeigt weiterhin den API Key im masked-Modus — die Settings-Seite ist der dedizierte Ort fuer Management-Aktionen (Regenerate)

### References

- [Source: _bmad-output/planning-artifacts/epics.md#Story 2.4] — User Story und Acceptance Criteria
- [Source: _bmad-output/planning-artifacts/prd.md#FR13] — API Key regenerieren (alter Key sofort invalidiert)
- [Source: _bmad-output/planning-artifacts/prd.md#FR14] — API Key Scopes einsehen
- [Source: _bmad-output/planning-artifacts/prd.md#NFR6] — API Keys gehasht gespeichert, nur bei Erstellung/Regenerierung im Klartext
- [Source: _bmad-output/planning-artifacts/architecture.md#API Client Pattern] — mmsApi(), Route Handler Format, TanStack Query
- [Source: _bmad-output/planning-artifacts/architecture.md#Component Patterns] — Danger-Button + Confirmation Dialog, Copy-Button Inline-Checkmark, Skeleton Loading
- [Source: _bmad-output/planning-artifacts/architecture.md#Enforcement Guidelines] — 10 Regeln + Anti-Patterns
- [Source: _bmad-output/planning-artifacts/architecture.md#Structure Patterns] — Settings unter `(dashboard)/settings/api-keys/`
- [Source: _bmad-output/planning-artifacts/ux-design-specification.md#Button-Hierarchie] — Danger-Button: Outline rote Border, immer mit Confirmation Dialog
- [Source: _bmad-output/planning-artifacts/ux-design-specification.md#Feedback Patterns] — Toast-System: Save/Update 3s auto-dismiss, Fehler bleibt
- [Source: _bmad-output/planning-artifacts/ux-design-specification.md#Confirmation Dialog Pattern] — Minimalistisch: Headline + Kontext + Cancel (Ghost) + Confirm (Danger)
- [Source: _bmad-output/planning-artifacts/ux-design-specification.md#APIKeyDisplay] — Settings: Key masked, Regenerate Button
- [Source: _bmad-output/planning-artifacts/ux-design-specification.md#Effortless Interactions] — API Key Scopes vorkonfiguriert, kein manuelles Scope-Management
- [Source: _bmad-output/implementation-artifacts/2-3-automatische-api-key-erstellung-anzeige.md] — ApiKeyCard, use-api-key.ts, api-keys/route.ts, types/tenant.ts

## Dev Agent Record

### Agent Model Used

Claude Opus 4.6 (claude-opus-4-6)

### Debug Log References

- Build, type-check, lint alle fehlerfrei beim ersten Durchlauf
- Keine Regressionen im Build Output

### Completion Notes List

- Task 1: Settings API Keys Seite erstellt unter `(dashboard)/dashboard/settings/api-keys/page.tsx`. Client Component mit ApiKeyCard (masked/first-reveal Modi), Scopes-Anzeige als read-only Badges, Regenerate-Button (Danger-Variante outline mit roter Border). Settings-Hauptseite (`settings/page.tsx`) von Placeholder zu Redirect auf `/dashboard/settings/api-keys` geaendert. Sidebar-Navigation war bereits korrekt konfiguriert.
- Task 2: POST Handler in `api/api-keys/route.ts` hinzugefuegt. Auth-Check via `auth()`, MMS Backend Call zu `POST /v1/paywatcher/api-keys/regenerate` mit tenant_id aus Session, snakeToCamel Transformation, Envelope Format, Error Handling mit MmsApiError.
- Task 3: `useRegenerateApiKey()` Mutation Hook in `use-api-key.ts` hinzugefuegt. POST `/api/api-keys`, bei Erfolg invalidiert `['api-key']` und `['tenant']` Query Keys. Loading/Error/Success States via useMutation.
- Task 4: shadcn/ui AlertDialog installiert. Confirmation Dialog in Settings API Keys Seite: Trigger = Regenerate Button (outline, destructive), Dialog mit Headline "Regenerate API Key?", Body-Text gemaess AC#1, Cancel (AlertDialogCancel) + Regenerate (destructive AlertDialogAction). Loading-State: Spinner + "Regenerating..." Text, Dialog schliesst nicht bei Loading, Cancel disabled bei Loading.
- Task 5: Nach erfolgreicher Regenerierung: Dialog schliesst, plaintextKey in useState → ApiKeyCard wechselt zu "first-reveal" Modus mit Warning-Banner ("Save this — you won't see it again in full."). Klartext-Key NUR in useState (nicht im Query Cache, nicht in localStorage). Nach Refresh: useState leer → masked Modus. Toast "API Key regenerated ✓" via sonner.
- Task 6: Scopes werden in der Settings-Seite als read-only Badges angezeigt (Shield-Icon + Badge-Komponenten). Edge Case: "No scopes configured" Hinweis wenn leer. Kein Scope-Management-UI.
- Task 7: Build (`npm run build`) fehlerfrei, type-check (`tsc --noEmit`) fehlerfrei, lint (`eslint`) fehlerfrei. Landing Page bleibt `○ (Static)`. Alle neuen Routes korrekt als `ƒ (Dynamic)` gerendert.

### Change Log

- 2026-02-16: Story 2.4 implementiert — API Key Management (Regenerate & Scopes). Settings-Seitenstruktur aufgebaut, Regenerate-Endpoint, Mutation Hook, Confirmation Dialog, Post-Regenerate Klartext-Anzeige, Scopes-Badges. Alle ACs erfuellt, Build/Type/Lint fehlerfrei.
- 2026-02-16: Code Review (AI) — 4 Issues gefixt: Doppelte Scopes-Anzeige entfernt (H1), AlertDialogAction variant="destructive" statt className (H2), Import-Duplikation behoben (M1), ApiError Klasse statt Plain Object throws (M2). Build/Type/Lint weiterhin fehlerfrei.

### File List

- `src/app/(dashboard)/dashboard/settings/api-keys/page.tsx` — NEU: API Key Management Seite (Client Component)
- `src/app/(dashboard)/dashboard/settings/page.tsx` — GEAENDERT: Placeholder → Redirect zu /dashboard/settings/api-keys
- `src/app/api/api-keys/route.ts` — GEAENDERT: POST Handler fuer API Key Regenerierung hinzugefuegt
- `src/hooks/use-api-key.ts` — GEAENDERT: useRegenerateApiKey() Mutation Hook hinzugefuegt
- `src/components/ui/alert-dialog.tsx` — NEU: shadcn/ui AlertDialog (via CLI installiert)
