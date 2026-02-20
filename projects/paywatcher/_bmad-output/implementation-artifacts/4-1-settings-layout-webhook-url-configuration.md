# Story 4.1: Settings Layout & Webhook URL Configuration

Status: done

<!-- Note: Validation is optional. Run validate-create-story for quality check before dev-story. -->

## Story

As a Tenant,
I want meine Webhook-URL im Dashboard konfigurieren können,
So that PayWatcher mir automatisch Benachrichtigungen bei Payment-Statusänderungen sendet.

## Acceptance Criteria

1. **Given** ich klicke auf "Settings" in der Sidebar **When** die Settings-Seite lädt **Then** sehe ich eine Settings-Navigation mit den Bereichen: Webhooks, API Keys und Account **And** der Content-Bereich ist max-w-3xl und zentriert **And** ich werde standardmäßig zum Webhooks-Bereich geleitet

2. **Given** die Tenant-/Webhook-Typen definiert werden **When** die Types erstellt werden **Then** existiert `types/tenant.ts` mit `TenantConfig` (bereits vorhanden) **And** API-Proxy-Routen existieren: PATCH-Handler in `app/api/tenant/route.ts` (PATCH → PATCH /v1/paywatcher/config) **And** TanStack Query Hooks existieren: `useTenant()` mit `staleTime: 5min` (bereits vorhanden) + Mutation-Hook `useUpdateConfig()` **And** HINWEIS: Kein separater webhook-config Proxy-Endpoint — webhook_url ist Teil der Config (PATCH /v1/paywatcher/config)

3. **Given** keine Webhook-URL konfiguriert ist **When** die Webhook-Settings-Seite angezeigt wird **Then** sehe ich ein gelbes Info-Banner: "No webhook URL configured. Payments are processed normally, but you won't receive notifications." **And** ein leeres URL-Input-Feld mit Placeholder "https://your-app.com/webhook" **And** der "Send Test Webhook"-Button ist disabled

4. **Given** ich gebe eine Webhook-URL ein (FR20) **When** ich auf "Save" klicke **Then** wird die URL via React Hook Form + Zod validiert (muss HTTPS sein, gültige URL) **And** bei Validierungsfehler erscheint Inline-Error unter dem Feld: "URL must use HTTPS" (rot, caption-size) **And** der Fehler verschwindet bei erneutem Tippen

5. **Given** ich eine gültige URL eingebe und "Save" klicke **When** die URL gespeichert wird **Then** wird die URL via Proxy an PATCH /v1/paywatcher/config mit `{ webhook_url: "..." }` gesendet **And** ein Toast erscheint: "Webhook URL saved" (rechts unten, 3s auto-dismiss) **And** das Info-Banner verschwindet **And** der "Send Test Webhook"-Button wird aktiv

6. **Given** bereits eine Webhook-URL konfiguriert ist **When** die Settings-Seite lädt **Then** sehe ich die bestehende URL im Input-Feld (aus GET /v1/paywatcher/config Response via `useTenant()`) **And** ich kann die URL ändern und erneut speichern

## Tasks / Subtasks

- [x] Task 1: Dependencies installieren (AC: #4)
  - [x] 1.1 `react-hook-form` und `@hookform/resolvers` installieren — Architecture Doc fordert "React Hook Form + Zod" für alle Formulare
  - [x] 1.2 shadcn/ui `label` Komponente installieren (`npx shadcn@latest add label`) — nötig für Form-Labels

- [x] Task 2: PATCH Config Proxy Route erstellen (AC: #2, #5)
  - [x] 2.1 `src/app/api/tenant/route.ts` — ERWEITERN: PATCH-Handler hinzufügen neben bestehendem GET. Pattern exakt wie GET: `requireAuth(request)` → `mmsApi()` mit apiKey → Response. Body: `{ webhook_url: string }`. MMS-Endpoint: `PATCH /v1/paywatcher/config`
  - [x] 2.2 Request Body validieren: Nur erlaubte Felder durchlassen (`webhook_url`, `confirmation_override` — für Story 4.3 vorbereiten). Zod-Schema server-seitig für Validierung

- [x] Task 3: useUpdateConfig Mutation Hook erstellen (AC: #2, #5)
  - [x] 3.1 `src/hooks/use-update-config.ts` — TanStack Query `useMutation` Hook: PATCH `/api/tenant`, bei Erfolg `queryClient.invalidateQueries({ queryKey: ["tenant"] })` um `useTenant()` Cache zu refreshen
  - [x] 3.2 Error-Handling via `parseApiError()` — gleiches Pattern wie bestehende Hooks

- [x] Task 4: Settings Layout mit Tab-Navigation erstellen (AC: #1)
  - [x] 4.1 `src/app/(dashboard)/dashboard/settings/layout.tsx` — Settings-Layout: max-w-3xl zentriert, Tab-Navigation als Links (Webhooks, API Keys, Account), active Tab highlighten basierend auf Pathname
  - [x] 4.2 Settings-Navigation: `<nav>` mit Links als Tab-Stil — Webhooks (`/dashboard/settings/webhooks`), API Keys (`/dashboard/settings/api-keys`), Account (`/dashboard/settings/account`)
  - [x] 4.3 `src/app/(dashboard)/dashboard/settings/page.tsx` — ÄNDERN: Redirect zu `/dashboard/settings/webhooks` statt `/dashboard/settings/api-keys` (Webhooks ist jetzt Default)

- [x] Task 5: Webhook URL Formular-Seite erstellen (AC: #3, #4, #5, #6)
  - [x] 5.1 `src/app/(dashboard)/dashboard/settings/webhooks/page.tsx` — `"use client"`, verwendet `useTenant()` + `useUpdateConfig()`, React Hook Form + Zod für URL-Validierung
  - [x] 5.2 `src/components/dashboard/webhook-url-form.tsx` — Formular-Komponente:
    - Gelbes Info-Banner wenn `webhookUrl` null/leer: "No webhook URL configured. Payments are processed normally, but you won't receive notifications."
    - URL Input mit Placeholder `https://your-app.com/webhook`
    - Zod-Validierung: URL muss HTTPS sein, gültige URL
    - Inline-Error unter dem Feld bei Validierungsfehler (rot, caption-size)
    - "Save" Button (Primary, max 1 pro Screen)
    - "Send Test Webhook" Button (Secondary, disabled wenn keine URL) — Button existiert, aber Funktionalität ist Story 4.2
    - Toast via sonner: "Webhook URL saved" bei Erfolg (3s auto-dismiss)
  - [x] 5.3 Skeleton-Loading: Skeleton für URL-Input + Buttons während `useTenant()` lädt
  - [x] 5.4 Error-State: Toast bei Fehler: "Failed to save webhook URL" (rot, bleibt bis dismissed)

- [x] Task 6: Account Placeholder-Seite erstellen (AC: #1)
  - [x] 6.1 `src/app/(dashboard)/dashboard/settings/account/page.tsx` — Placeholder-Seite mit "Coming in Story 4.3" Hinweis, damit Settings-Navigation funktioniert

- [x] Task 7: Dashboard Header Route-Title erweitern
  - [x] 7.1 `src/components/dashboard/dashboard-header.tsx` — ERWEITERN: routeTitles Map um Settings Sub-Routes: `/dashboard/settings/webhooks` → "Settings", `/dashboard/settings/api-keys` → "Settings", `/dashboard/settings/account` → "Settings". Fallback für alle `/dashboard/settings/*` Pfade auf "Settings"

- [x] Task 8: Build-Validierung
  - [x] 8.1 `npm run build` fehlerfrei
  - [x] 8.2 Settings-Sidebar-Link navigiert zu `/dashboard/settings/webhooks`
  - [x] 8.3 Tab-Navigation zwischen Webhooks, API Keys, Account funktioniert
  - [x] 8.4 Webhook URL speichern: gültige HTTPS URL → Toast "Webhook URL saved"
  - [x] 8.5 Webhook URL Validierung: HTTP URL zeigt Inline-Error
  - [x] 8.6 Info-Banner erscheint/verschwindet korrekt basierend auf webhookUrl
  - [x] 8.7 "Send Test Webhook" Button ist disabled wenn keine URL konfiguriert
  - [x] 8.8 Bestehende API Keys Seite funktioniert weiterhin unter neuem Settings-Layout
  - [x] 8.9 Mobile: Settings-Seite einspaltig, volle Breite, Touch Targets min. 44x44px
  - [x] 8.10 Skeleton-Loading für alle Settings-Seiten korrekt

## Dev Notes

### Kontext & Zweck

Dies ist **Story 4.1 von Epic 4** (Webhook Configuration & Tenant Settings) — die **erste Story des Epics**. Sie etabliert die Settings-Infrastruktur (Layout, Navigation, PATCH-Route) und implementiert die Webhook-URL-Konfiguration.

**Was diese Story beinhaltet:**
- Settings-Layout mit Tab-Navigation (Webhooks, API Keys, Account)
- PATCH Proxy-Route für Config-Updates
- Webhook URL Formular mit React Hook Form + Zod
- Info-Banner für fehlende Webhook-URL
- "Send Test Webhook" Button (disabled, Funktionalität kommt in Story 4.2)

**Was diese Story NICHT beinhaltet:**
- Test-Webhook senden (Story 4.2)
- Webhook Delivery Log (Story 4.2)
- Confirmation Override (Story 4.3)
- Deposit Address Anzeige (Story 4.3)
- Account-Einstellungen (Story 4.3)

### KRITISCH: React Hook Form muss installiert werden

`react-hook-form` und `@hookform/resolvers` sind NICHT im Projekt installiert. Die bestehende `auth-form.tsx` (Landing Page) nutzt vanilla State + Zod. Für Dashboard-Formulare fordert die Architecture Doc explizit React Hook Form + Zod. Installation nötig:

```bash
npm install react-hook-form @hookform/resolvers
npx shadcn@latest add label
```

### KRITISCH: PATCH Route zum bestehenden /api/tenant hinzufügen — NICHT neuen Endpoint erstellen

Der Architecture Doc zeigt `/api/config/route.ts`, aber der bestehende Code nutzt `/api/tenant/route.ts` für GET /v1/paywatcher/config. Der `useTenant()` Hook fetcht von `/api/tenant`. **PATCH muss zum bestehenden `/api/tenant/route.ts` hinzugefügt werden**, damit der `useTenant()` Cache-Invalidierung korrekt funktioniert und kein doppelter Endpoint entsteht.

```typescript
// src/app/api/tenant/route.ts — PATCH hinzufügen
export async function PATCH(request: Request) {
  const result = await requireAuth(request)
  if ("error" in result) return result.error

  const body = await request.json()
  // Zod-Validierung server-seitig
  // Nur erlaubte Felder: webhook_url, confirmation_override

  const response = await mmsApi<MmsApiResponse<MmsTenantConfig>>(
    "/v1/paywatcher/config",
    {
      method: "PATCH",
      body: JSON.stringify(body), // MMS erwartet snake_case — body IST bereits snake_case
    },
    result.apiKey
  )
  return NextResponse.json({ data: snakeToCamel(response.data) })
}
```

**WICHTIG:** Der Request Body muss in **snake_case** an MMS gesendet werden (`webhook_url`, nicht `webhookUrl`). Die Frontend-Komponente sendet camelCase, der Route Handler transformiert zu snake_case vor dem MMS-Call. Oder einfacher: Das Frontend sendet direkt snake_case Keys, da es nur ein Feld ist.

### KRITISCH: Settings Layout als Shared Layout mit Tab-Navigation

Die Settings-Seite braucht ein Layout mit einer Tab-artigen Navigation. **Nicht** shadcn/ui `<Tabs>` verwenden (das ist für Client-seitige Tab-Wechsel), sondern Links die wie Tabs aussehen. Grund: Jeder Tab ist eine eigene Route (`/dashboard/settings/webhooks`, `/dashboard/settings/api-keys`, `/dashboard/settings/account`), damit:
- URL teilbar/bookmarkbar ist
- Browser Back/Forward funktioniert
- Jede Sub-Route eigenständig lazy-loaded wird

```typescript
// src/app/(dashboard)/dashboard/settings/layout.tsx
"use client"

import Link from "next/link"
import { usePathname } from "next/navigation"
import { cn } from "@/lib/utils"

const settingsTabs = [
  { href: "/dashboard/settings/webhooks", label: "Webhooks" },
  { href: "/dashboard/settings/api-keys", label: "API Keys" },
  { href: "/dashboard/settings/account", label: "Account" },
]

export default function SettingsLayout({ children }: { children: React.ReactNode }) {
  const pathname = usePathname()

  return (
    <div className="mx-auto flex max-w-3xl flex-col gap-6 py-8">
      <div>
        <h2 className="text-h2 font-semibold">Settings</h2>
        <p className="text-sm text-muted-foreground">
          Manage your webhook, API keys, and account settings.
        </p>
      </div>
      <nav className="flex gap-1 border-b border-border">
        {settingsTabs.map((tab) => (
          <Link
            key={tab.href}
            href={tab.href}
            className={cn(
              "px-4 py-2 text-sm font-medium -mb-px border-b-2 transition-colors",
              pathname === tab.href
                ? "border-accent text-foreground"
                : "border-transparent text-muted-foreground hover:text-foreground"
            )}
          >
            {tab.label}
          </Link>
        ))}
      </nav>
      {children}
    </div>
  )
}
```

**KONSEQUENZ:** Die bestehende `api-keys/page.tsx` muss angepasst werden — der `max-w-3xl` Container und der `<h2>` Header werden jetzt vom Settings-Layout bereitgestellt. Die API Keys Page darf diese NICHT doppeln.

### KRITISCH: Webhook URL Form — Zod Schema + React Hook Form Pattern

```typescript
// Zod Schema für Webhook URL
import { z } from "zod"

const webhookUrlSchema = z.object({
  webhookUrl: z
    .string()
    .url("Please enter a valid URL")
    .refine((url) => url.startsWith("https://"), {
      message: "URL must use HTTPS",
    }),
})
```

**React Hook Form Integration:**

```typescript
import { useForm } from "react-hook-form"
import { zodResolver } from "@hookform/resolvers/zod"

const form = useForm({
  resolver: zodResolver(webhookUrlSchema),
  defaultValues: { webhookUrl: tenant?.webhookUrl ?? "" },
})
```

**WICHTIG:** `defaultValues` müssen aktualisiert werden wenn `useTenant()` Daten lädt. Verwende `useEffect` + `form.reset()` oder `values` Prop von `useForm` (React Hook Form v7.43+):

```typescript
const form = useForm({
  resolver: zodResolver(webhookUrlSchema),
  values: { webhookUrl: tenant?.webhookUrl ?? "" }, // Syncs when tenant changes
})
```

### KRITISCH: snake_case Transformation für PATCH Body

Der PATCH Body muss in snake_case an MMS gesendet werden. Zwei Optionen:

**Option A (empfohlen — einfach):** Transformation im Route Handler:
```typescript
// Route Handler
import { camelToSnake } from "@/lib/api/transform" // Falls vorhanden
// ODER einfach manuell:
const mmsBody = { webhook_url: body.webhookUrl }
```

**Option B:** Frontend sendet direkt snake_case. Aber das widerspricht dem Pattern "camelCase im Frontend".

→ **Option A verwenden.** Route Handler transformiert camelCase Body zu snake_case für MMS.

**ACHTUNG:** Prüfen ob `camelToSnake()` in `lib/api/transform.ts` existiert. Falls nicht, eine einfache Funktion erstellen oder manuell mappen (es ist nur ein Feld).

### Bestehender Code-Stand (aus Epic 2 + Epic 3)

**Bereits vorhanden — WIEDERVERWENDEN, nicht neu erstellen:**

| Datei | Was existiert | Aktion |
|-------|-------------|--------|
| `src/types/tenant.ts` | `TenantConfig` mit `webhookUrl: string \| null` | **VERWENDEN** — keine Änderung nötig |
| `src/lib/api/types.ts` | `MmsTenantConfig` (snake_case Vertrag) | **VERWENDEN** — Referenz für PATCH Body |
| `src/hooks/use-tenant.ts` | `useTenant()` mit Query Key `['tenant']`, staleTime 5min | **VERWENDEN** — für Webhook URL lesen |
| `src/app/api/tenant/route.ts` | GET /v1/paywatcher/config Proxy | **ERWEITERN** — PATCH Handler hinzufügen |
| `src/lib/api/client.ts` | `mmsApi()` Fetch-Wrapper | **VERWENDEN** |
| `src/lib/api/transform.ts` | `snakeToCamel<T>()` generische Transformation | **VERWENDEN** — für PATCH Response |
| `src/lib/auth.ts` | `requireAuth()` Helper | **VERWENDEN** |
| `src/lib/api/errors.ts` | `MmsApiError` Klasse | **VERWENDEN** |
| `src/lib/api/client-error.ts` | `parseApiError()` | **VERWENDEN** — für Mutation Error Handling |
| `src/components/ui/input.tsx` | shadcn/ui Input | **VERWENDEN** |
| `src/components/ui/button.tsx` | shadcn/ui Button | **VERWENDEN** |
| `src/components/ui/card.tsx` | shadcn/ui Card | **VERWENDEN** |
| `src/components/ui/skeleton.tsx` | shadcn/ui Skeleton | **VERWENDEN** |
| `src/components/ui/sonner.tsx` | sonner Toast Integration | **VERWENDEN** — via `toast()` |
| `src/app/(dashboard)/dashboard/settings/api-keys/page.tsx` | API Keys Seite | **ANPASSEN** — Container/Header entfernen (kommt jetzt vom Layout) |
| `src/components/dashboard/nav-config.ts` | Sidebar Nav Items | **UNVERÄNDERT** — Settings Link zeigt auf `/dashboard/settings` |
| `src/components/dashboard/dashboard-header.tsx` | routeTitles Map | **ERWEITERN** — Settings Sub-Routes hinzufügen |

### Neue Dateien die erstellt werden:

```
src/
  app/
    (dashboard)/dashboard/settings/
      layout.tsx                                    # NEU: Settings Layout mit Tab-Navigation
      webhooks/
        page.tsx                                    # NEU: Webhook URL Formular Page
      account/
        page.tsx                                    # NEU: Placeholder für Story 4.3
  hooks/
    use-update-config.ts                            # NEU: TanStack Query Mutation Hook
  components/dashboard/
    webhook-url-form.tsx                            # NEU: Webhook URL Formular Komponente
```

### Bestehende Dateien die geändert werden:

```
src/
  app/
    api/tenant/
      route.ts                                      # ERWEITERN: PATCH Handler hinzufügen
    (dashboard)/dashboard/settings/
      page.tsx                                      # ÄNDERN: Redirect zu /webhooks statt /api-keys
      api-keys/
        page.tsx                                    # ANPASSEN: Container/Header entfernen (Settings-Layout stellt bereit)
  components/dashboard/
    dashboard-header.tsx                            # ERWEITERN: routeTitles für Settings Sub-Routes
```

### Learnings aus Story 3.5 (Previous Story Intelligence)

- **Proxy Pattern:** Etabliert in `app/api/tenant/route.ts` — `requireAuth(request)` + `mmsApi()` + `snakeToCamel()`. PATCH-Handler exakt diesem Pattern folgen.
- **Hook Pattern:** `useTenant()` existiert mit Query Key `['tenant']`, staleTime 5min. Mutation muss `invalidateQueries({ queryKey: ["tenant"] })` nach Erfolg aufrufen.
- **Skeleton Pattern:** Pro Komponente eigene Skeleton-Variante.
- **Toast Pattern:** `sonner` ist installiert, Import: `import { toast } from "sonner"`. Position rechts unten, 3s auto-dismiss für Erfolg, bleibt für Fehler.
- **params als Promise:** Next.js 16 Route Handlers — hier nicht relevant da PATCH keinen `[id]` Parameter hat.
- **`"use client"` Pages:** Dashboard-Seiten mit Hooks sind Client Components.

### Git Intelligence (letzte 5 Commits)

1. `d275e8e` — Add Payment Detail & Webhook History with review fixes (Story 3.5)
2. `d980a47` — Add Payment Filters, Sorting & Pagination with review fixes (Story 3.4)
3. `8321838` — Add Payments List Page with DataTable, Card-Layout & review fixes (Story 3.3)
4. `86f1437` — Add Overview Page with state-dependent first screen, StatCards, HealthBar (Story 3.2)
5. `4b37e77` — sync to masem context

**Relevantes Pattern:** Commit-Messages als "Add [Feature] with review fixes (Story X.Y)". Neue Proxy-Routen in `app/api/`, neue Hooks in `hooks/`, neue Komponenten in `components/dashboard/`. Page-Dateien sind `"use client"` mit Hook-Integration.

### Project Structure Notes

- Alignment mit Projektstruktur: Alle neuen Dateien folgen dem etablierten Pattern
- `settings/webhooks/page.tsx` — neue Sub-Route unter bestehendem Settings-Verzeichnis
- `settings/layout.tsx` — neues Settings-Layout für Tab-Navigation
- `/api/tenant/route.ts` — PATCH zum bestehenden GET hinzufügen (gleiche Route, gleicher MMS-Endpoint)
- HINWEIS: Architecture Doc zeigt `/api/config/route.ts`, aber Codebase nutzt `/api/tenant/route.ts`. Wir folgen dem bestehenden Code um Konsistenz zu wahren.

### Architektur-Compliance

- Dateien in `kebab-case.tsx` — `webhook-url-form.tsx`, `use-update-config.ts`
- Types in `types/tenant.ts` — TenantConfig bereits vorhanden, keine Änderung
- Route Handler Response im Envelope-Format `{ data }` / `{ error }`
- camelCase → snake_case Transformation im Route Handler (für PATCH Body an MMS)
- snake_case → camelCase Transformation im Route Handler (für Response von MMS)
- TanStack Query: Mutation invalidiert `['tenant']` Query Key
- Kein `any` Type
- Kein Full-Page-Spinner — Sidebar + Header stehen sofort, nur Content zeigt Skeleton
- Toast via sonner: Erfolg 3s auto-dismiss, Fehler bleibt
- Max 1 Primary Button pro Screen ("Save" = Primary, "Send Test Webhook" = Secondary)
- Inline-Error für Form-Validierung (rot, caption-size)
- Copy-Feedback als Inline-Checkmark — nie als Toast
- Settings Content: max-w-3xl zentriert (UX-Spec)

### HINWEIS: "Send Test Webhook" Button — Story 4.2 Vorbereitung

Der Button wird in dieser Story als **disabled** gerendert wenn keine URL konfiguriert ist, und als **aktiv** (aber ohne Funktionalität) wenn eine URL existiert. Die tatsächliche Test-Funktionalität kommt in Story 4.2. Der Button soll trotzdem sichtbar sein, damit das UX-Layout komplett ist.

```typescript
<Button
  variant="outline"
  disabled={!webhookUrl}
  onClick={() => {
    // Story 4.2: POST /api/config/test-webhook
    toast.info("Coming soon — Test Webhook (Story 4.2)")
  }}
>
  Send Test Webhook
</Button>
```

### HINWEIS: API Keys Page Anpassung

Die bestehende `api-keys/page.tsx` hat eigenen `max-w-3xl` Container und `<h2>` Header:

```typescript
// AKTUELL:
<div className="mx-auto flex max-w-3xl flex-col gap-6 py-8">
  <div>
    <h2 className="text-h2 font-semibold">API Keys</h2>
    ...
```

Das Settings-Layout stellt jetzt Container + Header bereit. Die API Keys Page muss den äußeren Container und den h2 Header entfernen, damit kein Doppel-Container/Doppel-Header entsteht:

```typescript
// NEU:
<div className="flex flex-col gap-6">
  <ApiKeyCard ... />
  <p className="text-caption text-muted-foreground">
    Need a new API key? Contact your account manager.
  </p>
</div>
```

### HINWEIS: Verfügbare shadcn/ui Komponenten

Installiert: Button, Input, Card, Badge, Skeleton, Table, Tabs, Select, DropdownMenu, AlertDialog, Sheet, sonner

**Noch nicht installiert (für diese Story nötig):**
- `label` — für Form-Labels (`npx shadcn@latest add label`)

**Nicht nötig für diese Story:**
- `form` (shadcn/ui Form) — React Hook Form wird direkt verwendet, nicht über shadcn/ui Form-Wrapper

### References

- [Source: _bmad-output/planning-artifacts/epics.md#Story 4.1]
- [Source: _bmad-output/planning-artifacts/architecture.md#API Communication Pattern]
- [Source: _bmad-output/planning-artifacts/architecture.md#App Structure & Routing — settings/webhooks/page.tsx]
- [Source: _bmad-output/planning-artifacts/architecture.md#Component Patterns — Button-Hierarchie, Toast Pattern]
- [Source: _bmad-output/planning-artifacts/architecture.md#TanStack Query Patterns — Mutations]
- [Source: _bmad-output/planning-artifacts/architecture.md#Vollständige Projektverzeichnis-Struktur — api/config/route.ts]
- [Source: _bmad-output/planning-artifacts/ux-design-specification.md#Flow 4: Webhook Configuration & Testing]
- [Source: _bmad-output/planning-artifacts/ux-design-specification.md#Layout Regeln — Settings max-w-3xl]
- [Source: _bmad-output/planning-artifacts/ux-design-specification.md#Webhook Settings States]
- [Source: _bmad-output/planning-artifacts/ux-design-specification.md#Button-Hierarchie — Max 1 Primary pro Screen]
- [Source: _bmad-output/planning-artifacts/ux-design-specification.md#Toast Pattern — Save/Update 3s auto-dismiss]
- [Source: _bmad-output/planning-artifacts/prd.md#FR20 — Webhook-URL konfigurieren]
- [Source: _bmad-output/implementation-artifacts/3-5-payment-detail-webhook-history.md — Proxy Pattern Referenz]
- [Source: src/types/tenant.ts — TenantConfig Interface (webhookUrl)]
- [Source: src/lib/api/types.ts — MmsTenantConfig (snake_case)]
- [Source: src/hooks/use-tenant.ts — useTenant() Hook]
- [Source: src/app/api/tenant/route.ts — GET Proxy Pattern Referenz]
- [Source: src/lib/api/transform.ts — snakeToCamel()]
- [Source: src/lib/auth.ts — requireAuth() Helper]
- [Source: src/components/dashboard/api-key-card.tsx — Card-Formular-Pattern]
- [Source: src/components/dashboard/nav-config.ts — Settings Sidebar Link]
- [Source: src/components/dashboard/dashboard-header.tsx — routeTitles Map]

## Dev Agent Record

### Agent Model Used

Claude Opus 4.6

### Debug Log References

- Build erfolgreich ohne Fehler (TypeScript + Next.js 16.1.6 Turbopack)

### Completion Notes List

- Task 1: `react-hook-form`, `@hookform/resolvers` und shadcn/ui `label` installiert
- Task 2: PATCH-Handler zu `/api/tenant/route.ts` hinzugefügt mit Zod-Validierung (strict schema: nur webhookUrl — confirmationOverride erst in Story 4.3). camelCase→snake_case Transformation manuell im Handler
- Task 3: `useUpdateConfig()` Mutation Hook erstellt — PATCH `/api/tenant`, invalidiert `["tenant"]` Query Key bei Erfolg
- Task 4: Settings Layout mit Tab-Navigation (Links statt shadcn Tabs für Route-basierte Navigation). Redirect von `/dashboard/settings` zu `/dashboard/settings/webhooks`. API Keys Page Container/Header entfernt (Layout stellt bereit)
- Task 5: Webhook URL Formular mit React Hook Form + Zod (HTTPS-Validierung), gelbes Info-Banner bei fehlender URL, Skeleton-Loading, Success/Error Toasts, "Send Test Webhook" Button (disabled ohne URL, toast.info placeholder für Story 4.2)
- Task 6: Account Placeholder-Seite mit "Coming in Story 4.3" Hinweis
- Task 7: Dashboard Header routeTitles um Settings Sub-Routes erweitert
- Task 8: Build fehlerfrei validiert

### Change Log

- 2026-02-20: Story 4.1 implementiert — Settings Layout, Tab-Navigation, PATCH Proxy Route, Webhook URL Formular, Account Placeholder
- 2026-02-20: Code Review Fixes — Error-Toast persistent (duration: Infinity), confirmationOverride aus PATCH-Schema entfernt (Story 4.3), aria-describedby für Formular-Accessibility, Error-Message statt generisch, Tab-Navigation prefix-match

### File List

**Neue Dateien:**
- `src/app/(dashboard)/dashboard/settings/layout.tsx` — Settings Layout mit Tab-Navigation
- `src/app/(dashboard)/dashboard/settings/webhooks/page.tsx` — Webhook URL Formular Page
- `src/app/(dashboard)/dashboard/settings/account/page.tsx` — Account Placeholder
- `src/hooks/use-update-config.ts` — TanStack Query Mutation Hook
- `src/components/dashboard/webhook-url-form.tsx` — Webhook URL Formular Komponente
- `src/components/ui/label.tsx` — shadcn/ui Label (via shadcn CLI)

**Geänderte Dateien:**
- `src/app/api/tenant/route.ts` — PATCH-Handler hinzugefügt
- `src/app/(dashboard)/dashboard/settings/page.tsx` — Redirect zu /webhooks
- `src/app/(dashboard)/dashboard/settings/api-keys/page.tsx` — Container/Header entfernt
- `src/components/dashboard/dashboard-header.tsx` — routeTitles erweitert
- `package.json` — react-hook-form, @hookform/resolvers hinzugefügt
- `package-lock.json` — aktualisiert
