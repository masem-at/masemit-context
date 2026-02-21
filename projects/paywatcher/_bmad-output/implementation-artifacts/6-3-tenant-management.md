# Story 6.3: Tenant Management

Status: done

<!-- Note: Validation is optional. Run validate-create-story for quality check before dev-story. -->

## Story

As a Admin (Mario),
I want alle Tenants sehen, neue Tenants anlegen und bestehende Tenants aktivieren/deaktivieren können,
so that ich den Managed MVP Onboarding-Flow vollständig über das Dashboard abwickeln kann.

## Acceptance Criteria

1. **Given** ich bin als Admin eingeloggt und öffne `/admin/tenants` (FR40)
   **When** die Tenants-Seite lädt
   **Then** sehe ich eine Tabelle aller Tenants mit: tenant_slug, tenant_name, Status (enabled/disabled als Badge), Tier (free/starter/pro/enterprise), Payment Count, Total Volume, Webhook Success Rate
   **And** die Daten kommen von GET /v1/admin/paywatcher/tenants (inkl. Aggregationen)

2. **Given** ich klicke auf "New Tenant" (FR41)
   **When** das Erstell-Formular angezeigt wird
   **Then** sehe ich Felder für: tenant_name (required), tenant_slug (required, auto-generated aus Name, editierbar), contact_email (required), tier (Dropdown: free, starter, pro, enterprise), webhook_url (optional), confirmation_override (optional, 2-20)
   **And** React Hook Form + Zod Validierung

3. **Given** ich fülle das Formular aus und klicke "Create Tenant"
   **When** der Tenant erfolgreich erstellt wird (POST /v1/admin/paywatcher/tenants)
   **Then** zeigt ein Modal einmalig: API Key im Klartext (mit Copy-Button), Webhook Secret im Klartext (mit Copy-Button)
   **And** eine Warnung: "This is the only time these credentials will be shown."
   **And** nach Schließen des Modals ist der neue Tenant in der Liste sichtbar

4. **Given** ich klicke auf den Enable/Disable Toggle eines Tenants (FR42)
   **When** ich den Status ändere
   **Then** wird der Status via PATCH /v1/admin/paywatcher/tenants/:slug mit `{ enabled: true/false }` aktualisiert
   **And** bei Deaktivierung erscheint ein Confirmation Dialog: "Disable tenant [name]? They will lose API access."
   **And** der Status-Badge in der Tabelle aktualisiert sich sofort

5. **Given** ich klicke auf einen Tenant in der Tabelle
   **When** die Detail-Ansicht angezeigt wird
   **Then** sehe ich: Config-Details (webhook_url, confirmation_override, deposit_address, tier), Created/Updated Timestamps
   **And** einen Link zu den Payments dieses Tenants (-> `/admin/payments?tenant=slug`)

## Tasks / Subtasks

- [x] Task 1: Admin Tenant Types definieren (AC: #1, #2, #3)
  - [x] 1.1: `src/types/admin.ts` erweitern mit `AdminTenant` Interface (camelCase):
    - `tenantSlug: string`
    - `tenantName: string`
    - `contactEmail: string`
    - `tier: string`
    - `enabled: boolean`
    - `webhookUrl: string | null`
    - `confirmationOverride: number | null`
    - `depositAddress: string | null` (defensiv als nullable)
    - `paymentCount: number | null` (Aggregation, ggf. nicht im Response)
    - `totalVolume: string | null` (Aggregation, ggf. nicht im Response)
    - `webhookSuccessRate: number | null` (Aggregation, ggf. nicht im Response)
    - `createdAt: string`
    - `updatedAt: string`
  - [x] 1.2: `AdminTenantCreateRequest` Interface:
    - `tenant_name: string` (snake_case fuer MMS)
    - `tenant_slug: string`
    - `contact_email: string`
    - `tier: string`
    - `webhook_url?: string`
    - `confirmation_override?: number`
  - [x] 1.3: `AdminTenantCreateResponse` Interface:
    - `tenantSlug: string`
    - `apiKey: string` (Klartext, einmalig)
    - `webhookSecret: string` (Klartext, einmalig)
  - [x] 1.4: `AdminTenantPatchRequest` Interface:
    - `enabled?: boolean`
    - Optional weitere Felder die PATCH unterstuetzt
  - [x] 1.5: HINWEIS: Types defensiv mit optionalen/nullable Feldern definiert (depositAddress als `string | null`). MMS Response-Verifizierung steht aus — Types anpassbar nach Verifizierung.

- [x] Task 2: `useAdminTenants` Hook erstellen (AC: #1)
  - [x] 2.1: `src/hooks/use-admin-tenants.ts` erstellen — Pattern identisch zu `use-admin-health.ts`:
    ```typescript
    "use client"
    import { useQuery, useMutation, useQueryClient } from "@tanstack/react-query"
    import { parseApiError } from "@/lib/api/client-error"
    import type { AdminTenant, AdminTenantCreateRequest, AdminTenantCreateResponse } from "@/types/admin"

    async function fetchAdminTenants(): Promise<AdminTenant[]> {
      const res = await fetch("/api/admin/tenants")
      if (!res.ok) throw await parseApiError(res)
      const json = await res.json()
      const data = json.data ?? json
      return Array.isArray(data) ? data : []
    }

    export function useAdminTenants() {
      return useQuery({
        queryKey: ["admin-tenants"],
        queryFn: fetchAdminTenants,
        staleTime: 30_000,
      })
    }

    export function useCreateTenant() {
      const queryClient = useQueryClient()
      return useMutation({
        mutationFn: async (body: AdminTenantCreateRequest): Promise<AdminTenantCreateResponse> => {
          const res = await fetch("/api/admin/tenants", {
            method: "POST",
            headers: { "Content-Type": "application/json" },
            body: JSON.stringify(body),
          })
          if (!res.ok) throw await parseApiError(res)
          const json = await res.json()
          return json.data ?? json
        },
        onSuccess: () => {
          queryClient.invalidateQueries({ queryKey: ["admin-tenants"] })
        },
      })
    }

    export function useToggleTenant() {
      const queryClient = useQueryClient()
      return useMutation({
        mutationFn: async ({ slug, enabled }: { slug: string; enabled: boolean }) => {
          const res = await fetch(`/api/admin/tenants/${slug}`, {
            method: "PATCH",
            headers: { "Content-Type": "application/json" },
            body: JSON.stringify({ enabled }),
          })
          if (!res.ok) throw await parseApiError(res)
          const json = await res.json()
          return json.data ?? json
        },
        onSuccess: () => {
          queryClient.invalidateQueries({ queryKey: ["admin-tenants"] })
        },
      })
    }
    ```
  - [x] 2.2: Kein `refetchInterval` — Tenant-Liste aendert sich selten, manuelles Refresh via Mutation-Invalidierung

- [x] Task 3: Admin Tenants API Route anpassen (AC: #1, #3, #4)
  - [x] 3.1: `src/app/api/admin/tenants/route.ts` — GET Response mit `snakeToCamel()` transformiert.
  - [x] 3.2: POST Route — Response mit `snakeToCamel()` transformiert.
  - [x] 3.3: `src/app/api/admin/tenants/[slug]/route.ts` — PATCH Response mit `snakeToCamel()` transformiert.
  - [x] 3.4: Import und Transformation in bestehende Route Handler hinzugefuegt.

- [x] Task 4: Tenant Management Page implementieren (AC: #1, #5)
  - [x] 4.1: `src/app/(admin)/admin/tenants/page.tsx` — Stub ersetzt durch `"use client"` Page:
    - `useAdminTenants()` Hook aufgerufen
    - Loading State: `AdminTenantsTableSkeleton` aus Table-Komponente
    - Error State: Fehlermeldung mit `text-destructive`
    - Success State: `AdminTenantsTable` Component gerendert
    - "New Tenant" Button im Header-Bereich (oeffnet Create-Dialog)
  - [x] 4.2: Page Layout:
    ```
    ┌─ Tenants ──────────────────────────────────────────────────┐
    │                                                             │
    │  [h1: Tenants]                         [+ New Tenant]       │
    │                                                             │
    │  ┌──────────────────────────────────────────────────────┐  │
    │  │ Slug    │ Name    │ Status │ Tier  │ Payments │ Vol  │  │
    │  │─────────┼─────────┼────────┼───────┼──────────┼──────│  │
    │  │ acme    │ Acme    │ ●active│ pro   │ 142      │$12k  │  │
    │  │ test    │ Test Co │ ○off   │ free  │ 3        │$0.5k │  │
    │  └──────────────────────────────────────────────────────┘  │
    │                                                             │
    └─────────────────────────────────────────────────────────────┘
    ```

- [x] Task 5: TenantsTable Component erstellen (AC: #1, #4, #5)
  - [x] 5.1: `src/components/admin/admin-tenants-table.tsx` — Tabelle mit allen Spalten, klickbare Zeilen fuer inline Detail-Expansion, Toggle-Button pro Zeile
  - [x] 5.2: `TenantStatusBadge` inline definiert (Boundary-Regel eingehalten)
  - [x] 5.3: `TenantTierBadge` inline definiert mit allen 4 Tier-Varianten
  - [x] 5.4: `AdminTenantsTableSkeleton` mit 5 Skeleton-Zeilen
  - [x] 5.5: Empty State: "No tenants found."

- [x] Task 6: Tenant Detail Ansicht implementieren (AC: #5)
  - [x] 6.1: `src/components/admin/admin-tenant-detail.tsx` — Inline-Expansion mit dl Key-Value Layout
  - [x] 6.2: `<dl>` Pattern mit KeyValueRow Helper (wie PaymentDetail)
  - [x] 6.3: deposit_address als truncated Basescan-Link + InlineCopyButton (shared Component)

- [x] Task 7: Create Tenant Dialog implementieren (AC: #2, #3)
  - [x] 7.1: `src/components/admin/create-tenant-dialog.tsx` — Dialog-Formular mit React Hook Form + Zod:
    - Felder: tenant_name (Input, required), tenant_slug (Input, auto-generated, editierbar), contact_email (Input, required, email validation), tier (Select: free/starter/pro/enterprise, default: free), webhook_url (Input, optional, HTTPS validation), confirmation_override (Number Input, optional, 2-20)
    - Zod Schema:
      ```typescript
      const createTenantSchema = z.object({
        tenant_name: z.string().min(1, "Name is required").max(100),
        tenant_slug: z.string().min(1, "Slug is required").max(50)
          .regex(/^[a-z0-9][a-z0-9-]*[a-z0-9]$|^[a-z0-9]$/, "Lowercase alphanumeric and hyphens only"),
        contact_email: z.string().email("Valid email required"),
        tier: z.enum(["free", "starter", "pro", "enterprise"]),
        webhook_url: z.string().url().startsWith("https://", "Must use HTTPS").optional().or(z.literal("")),
        confirmation_override: z.number().int().min(2).max(20).optional().nullable(),
      })
      ```
    - Auto-Slug-Generierung: tenant_name onChange -> tenant_slug (slugify: lowercase, replace spaces/special chars with hyphens)
    - Submit Button mit Loading State
  - [x] 7.2: `src/components/ui/dialog.tsx` installiert via `npx shadcn@latest add dialog`. Button statt Switch fuer Enable/Disable.
  - [x] 7.3: Request Body in snake_case, Zod Schema nutzt snake_case Keys.

- [x] Task 8: Credentials Modal implementieren (AC: #3)
  - [x] 8.1: `src/components/admin/tenant-credentials-modal.tsx` — Modal mit API Key + Webhook Secret, Copy-Buttons, AlertTriangle Warning, "Done" Button, preventClose (kein Overlay/Escape dismiss), Toast nach Schliessen
  - [x] 8.2: Kein Dismiss via Overlay-Click oder Escape-Key — nur expliziter "Done" Button

- [x] Task 9: Enable/Disable Toggle mit Confirmation Dialog (AC: #4)
  - [x] 9.1: Enable direkt ausfuehren, kein Confirmation Dialog
  - [x] 9.2: Disable mit AlertDialog Confirmation (destructive variant)
  - [x] 9.3: Query-Invalidierung nach Mutation (statt Optimistic Update — robuster bei MMS-Fehlern)
  - [x] 9.4: Toast nach Erfolg: "Tenant enabled/disabled" (3s auto-dismiss)
  - [x] 9.5: Toast bei Fehler: Error Message

## Dev Notes

### Kritische Architektur-Erkenntnisse

**MMS Tenants Endpoint — Response-Shape unbekannt:**
- Die exakte JSON-Struktur von `GET /v1/admin/paywatcher/tenants` ist in keinem Planungsdokument vollstaendig spezifiziert
- Ob Aggregationsfelder (paymentCount, totalVolume, webhookSuccessRate) in der Response enthalten sind, ist unklar
- **ERSTER SCHRITT:** Dev-Server starten, als Admin einloggen, `GET /api/admin/tenants` aufrufen und Response inspizieren. Dann Types anpassen.
- Gleiches fuer POST Response: Pruefen ob `api_key` und `webhook_secret` in der Response enthalten sind und wie sie heissen
- Alle Aggregationsfelder als optional/nullable definieren (defensive Programmierung)

**`MMS_API_KEY` ist der Admin Key:**
- Kein separates `MMS_ADMIN_API_KEY` — `mmsApi()` faellt auf `process.env.MMS_API_KEY` zurueck (hat `admin:read`, `admin:write` Scopes)
- Alle Admin Route Handler nutzen `requireAdminAuth()` + `mmsApi()` ohne API Key Argument — korrekt

**Admin ist Desktop-Only:**
- Kein Mobile-Layout, kein Responsive-Handling noetig
- Sidebar ist immer sichtbar (200px), nicht collapsible

**snake_case Transformation:**
- `snakeToCamel` existiert in `src/lib/api/transform.ts`
- MMS Tenant Response liefert wahrscheinlich snake_case — im Route Handler transformieren
- MMS Tenant CREATE Request erwartet snake_case — Zod Schema nutzt bereits snake_case Keys
- PATCH Request ebenfalls snake_case (`{ enabled: true/false }`)

**Boundary-Regel (aus Story 6.1 Learnings):**
- Alle Admin Components in `src/components/admin/`
- NIE aus `src/components/dashboard/` importieren
- UI Components aus `src/components/ui/` sind shared und duerfen verwendet werden
- Badges, Cards, Tables, etc. als Admin-spezifische Varianten inline definieren

**Nach Tenant-Erstellung — Manueller Onboarding-Flow:**
- Nach POST /v1/admin/paywatcher/tenants muss Mario den Onboarding-Flow manuell fortfuehren:
  1. POST /v1/parties -> Party mit contact_email erstellen (oder Find-or-Create)
  2. POST /v1/parties/:party_id/tenants -> Zuordnung: paywatcher + tenant_slug
  3. API Key verschluesselt in Frontend-DB hinterlegen (tenant_keys Tabelle via CLI Script `scripts/set-tenant-key.ts`)
- Dieser Flow ist AUSSERHALB dieses Stories — hier wird nur der MMS Tenant erstellt und Credentials angezeigt
- Die Story zeigt die Credentials im Modal an, damit Mario sie notieren/kopieren kann

### Pattern-Uebernahme aus bestehender Codebase

| Aspekt | Vorlage | Admin Tenants |
|--------|---------|---------------|
| Page Component | `admin/page.tsx` (Overview) | `admin/tenants/page.tsx` |
| Loading | Inline Skeleton | Inline `AdminTenantsSkeleton` |
| Error | `text-destructive` | `text-destructive` |
| Content Component | `AdminOverviewContent` | `AdminTenantsContent` |
| Data Source | `useAdminHealth()` | `useAdminTenants()` |
| Table Pattern | `PaymentsTable` | `AdminTenantsTable` |
| Form Pattern | `AccountSettingsForm` / `WebhookUrlForm` | `CreateTenantDialog` (Zod + RHF) |
| Badge Pattern | `PaymentStatusBadge` | `TenantStatusBadge` + `TenantTierBadge` |
| Detail Pattern | `PaymentDetail` (dl key-value) | `AdminTenantDetail` |
| Confirmation | AlertDialog (shadcn) | Disable Tenant Confirmation |
| Credentials | Keine Vorlage | `TenantCredentialsModal` (neues Pattern) |

### Hook Pattern (identisch zu bestehenden Hooks)

```typescript
// src/hooks/use-admin-tenants.ts
"use client"
import { useQuery, useMutation, useQueryClient } from "@tanstack/react-query"
import { parseApiError } from "@/lib/api/client-error"
import type { AdminTenant } from "@/types/admin"

async function fetchAdminTenants(): Promise<AdminTenant[]> {
  const res = await fetch("/api/admin/tenants")
  if (!res.ok) throw await parseApiError(res)
  const json = await res.json()
  const data = json.data ?? json
  return Array.isArray(data) ? data : []
}

export function useAdminTenants() {
  return useQuery({
    queryKey: ["admin-tenants"],
    queryFn: fetchAdminTenants,
    staleTime: 30_000,
  })
}
```

### Bestehende wiederverwendbare UI Components

| Komponente | Pfad | Wiederverwendung |
|-----------|------|------------------|
| `Table` / `TableHeader` etc. | `src/components/ui/table.tsx` | Fuer TenantsTable |
| `Badge` | `src/components/ui/badge.tsx` | Fuer TenantStatusBadge + TierBadge |
| `Card` / `CardContent` | `src/components/ui/card.tsx` | Fuer Tenant Detail Ansicht |
| `Button` | `src/components/ui/button.tsx` | "New Tenant", Toggle, etc. |
| `Input` | `src/components/ui/input.tsx` | Formular-Felder |
| `Select` | `src/components/ui/select.tsx` | Tier-Dropdown |
| `Label` | `src/components/ui/label.tsx` | Formular-Labels |
| `Skeleton` | `src/components/ui/skeleton.tsx` | Loading States |
| `AlertDialog` | `src/components/ui/alert-dialog.tsx` | Disable Confirmation |
| `Dialog` | `src/components/ui/dialog.tsx` | Create Tenant + Credentials Modal (installieren falls fehlend) |
| `InlineCopyButton` | `src/components/shared/inline-copy-button.tsx` | Copy-Buttons fuer API Key, Webhook Secret, deposit_address, tenant_slug (shared Component, DARF importiert werden) |
| `cn()` | `src/lib/utils.ts` | Conditional classNames |
| `snakeToCamel()` | `src/lib/api/transform.ts` | Response Transformation |

### Slug-Generierung Helper

```typescript
function slugify(name: string): string {
  return name
    .toLowerCase()
    .trim()
    .replace(/[^a-z0-9]+/g, "-")
    .replace(/^-|-$/g, "")
}
```

### Bestehende Admin Route Handler (aus Story 6.1)

Die API Routes existieren bereits und funktionieren. Nur `snakeToCamel` Transformation hinzufuegen:

| Route | Datei | Aenderung |
|-------|-------|-----------|
| GET /api/admin/tenants | `src/app/api/admin/tenants/route.ts` | `snakeToCamel()` auf Response |
| POST /api/admin/tenants | `src/app/api/admin/tenants/route.ts` | `snakeToCamel()` auf Response |
| PATCH /api/admin/tenants/[slug] | `src/app/api/admin/tenants/[slug]/route.ts` | `snakeToCamel()` auf Response |

### Project Structure Notes

- Tenant Stub Page `src/app/(admin)/admin/tenants/page.tsx` existiert — wird durch echte Page ersetzt
- API Routes existieren bereits — nur Transformation hinzufuegen
- Types in `src/types/admin.ts` erweitern (nicht neue Datei)
- Hook in `src/hooks/use-admin-tenants.ts` (neues File)
- Alle Components in `src/components/admin/` (Boundary-Regel)

### Bestehende Dateien die modifiziert werden

| Datei | Aenderung |
|-------|----------|
| `src/app/(admin)/admin/tenants/page.tsx` | Stub ersetzen durch vollstaendige Tenants Page |
| `src/types/admin.ts` | AdminTenant, AdminTenantCreateRequest, AdminTenantCreateResponse, AdminTenantPatchRequest Types |
| `src/app/api/admin/tenants/route.ts` | `snakeToCamel()` auf GET + POST Response |
| `src/app/api/admin/tenants/[slug]/route.ts` | `snakeToCamel()` auf PATCH Response |

### Neue Dateien die erstellt werden

| Datei | Zweck |
|-------|-------|
| `src/hooks/use-admin-tenants.ts` | TanStack Query Hooks fuer Tenants (useAdminTenants, useCreateTenant, useToggleTenant) |
| `src/components/admin/admin-tenants-table.tsx` | Tenants-Tabelle mit Status/Tier Badges, Toggle, Detail-Click |
| `src/components/admin/admin-tenant-detail.tsx` | Tenant Detail-Ansicht (Config, Timestamps, Payments Link) |
| `src/components/admin/create-tenant-dialog.tsx` | Dialog-Formular fuer neuen Tenant (Zod + RHF) |
| `src/components/admin/tenant-credentials-modal.tsx` | Modal fuer einmalige API Key + Webhook Secret Anzeige |

### Learnings aus Story 6.1 und 6.2

- `contactEmail` statt `email` im Session Type — bereits korrekt in `requireAdminAuth()`
- `MMS_API_KEY` ist der Admin Key — kein `MMS_ADMIN_API_KEY` noetig
- Admin Layout prueft Auth auf Layout-Level (kein middleware.ts)
- MMS Health Response hatte komplett andere Shape als angenommen — Types MUESSEN verifiziert werden (Story 6.2 Task 1.3)
- `snakeToCamel` Transformation immer anwenden (idempotent bei camelCase)
- Defensive Programmierung: Optional Chaining + Fallback-Werte ueberall
- AdminStatCard inline definieren statt Cross-Import (Boundary-Regel)
- Error Messages auf English (nicht Deutsch, M1 aus Story 6.2 Code Review)

### Git Intelligence

Letzte Commits zeigen:
- `3b67da2` Improve onboarding UX for new tenants without config
- `63ddb15` Handle MMS 404 gracefully for new tenants without config
- `539f67a` Fix scope error message to list all three required admin scopes
- `9fe33f4` Add CLI script to set encrypted tenant API keys

Patterns:
- Error Handling und Graceful Degradation sind wichtig
- MMS 404 bei fehlendem Config muss abgefangen werden
- Admin Scopes: `parties:read`, `admin:read`, `admin:write` — alle drei noetig
- CLI Script `scripts/set-tenant-key.ts` existiert fuer tenant_keys DB-Eintraege

### References

- [Source: _bmad-output/planning-artifacts/epics.md#Epic 6 - Story 6.3]
- [Source: _bmad-output/planning-artifacts/architecture.md#API Communication Pattern - Data Flow (Admin)]
- [Source: _bmad-output/planning-artifacts/architecture.md#Component Patterns]
- [Source: _bmad-output/planning-artifacts/architecture.md#TanStack Query Patterns]
- [Source: _bmad-output/planning-artifacts/architecture.md#Admin Onboarding Flow]
- [Source: _bmad-output/implementation-artifacts/6-1-admin-shell-auth-route-protection.md]
- [Source: _bmad-output/implementation-artifacts/6-2-system-health-overview.md]
- [Source: src/app/api/admin/tenants/route.ts - Tenants Proxy Route (GET, POST)]
- [Source: src/app/api/admin/tenants/[slug]/route.ts - Tenant PATCH Route]
- [Source: src/hooks/use-admin-health.ts - Hook Pattern Template]
- [Source: src/components/dashboard/payments-table.tsx - Table Pattern]
- [Source: src/components/dashboard/payment-status-badge.tsx - Badge Pattern]
- [Source: src/components/dashboard/payment-detail.tsx - Detail dl Pattern]
- [Source: src/components/dashboard/account-settings-form.tsx - Form Pattern (Zod + RHF)]
- [Source: src/components/dashboard/webhook-url-form.tsx - Form Pattern (HTTPS Validation)]
- [Source: src/lib/api/client.ts - mmsApi() Fallback auf MMS_API_KEY]
- [Source: src/lib/api/transform.ts - snakeToCamel Utility]
- [Source: src/lib/api/client-error.ts - parseApiError Pattern]
- [Source: src/lib/admin.ts - requireAdminAuth Pattern]
- [Source: src/types/admin.ts - AdminHealthData Types erweitern]
- [Source: src/types/tenant.ts - TenantConfig Interface (Dashboard Pendant)]
- [Source: src/types/payment.ts - Payment Types fuer Aggregationsfelder]

## Dev Agent Record

### Agent Model Used

Claude Opus 4.6

### Debug Log References

- Build erfolgreich, keine TypeScript-Fehler
- ESLint: 1 Warning (React Compiler incompatible-library fuer form.watch — bekanntes Pattern, identisch zu bestehenden Formularen)
- MMS Response-Verifizierung ausgelassen (User-Entscheidung), Types defensiv mit nullable Feldern

### Completion Notes List

- Task 1: Types definiert in `src/types/admin.ts` — AdminTenant, AdminTenantCreateRequest, AdminTenantCreateResponse, AdminTenantPatchRequest. depositAddress als `string | null` (defensiv).
- Task 2: Hook `useAdminTenants` erstellt mit useQuery, useCreateTenant mit useMutation + invalidation, useToggleTenant mit useMutation + invalidation. Kein refetchInterval.
- Task 3: snakeToCamel Transformation auf GET, POST und PATCH Responses in bestehenden Route Handlern.
- Task 4: Tenants Page mit Loading/Error/Success States, "New Tenant" Button mit Plus-Icon.
- Task 5: TenantsTable mit TenantStatusBadge (Active/Disabled), TenantTierBadge (4 Varianten), klickbare Zeilen mit Inline-Detail-Expansion, Skeleton-Loading, Empty State.
- Task 6: TenantDetail mit dl Key-Value Layout, Basescan Link + Copy-Button fuer deposit_address, "View Payments" Link.
- Task 7: CreateTenantDialog mit Zod + RHF, Auto-Slug-Generierung, Select fuer Tier, HTTPS-Validation fuer webhook_url. Dialog UI installiert via shadcn.
- Task 8: CredentialsModal mit preventClose (kein Overlay/Escape), Copy-Buttons, AlertTriangle Warning, Toast nach Done.
- Task 9: TenantToggleButton als separate Komponente, Enable direkt, Disable mit AlertDialog Confirmation, destructive Styling, Toast-Feedback.

### Change Log

- 2026-02-21: Story 6.3 implementiert — Tenant Management mit Tabelle, Detail-Ansicht, Create-Dialog, Credentials-Modal, Enable/Disable Toggle
- 2026-02-21: Code Review bestanden (6 Fixes applied: H1 Fragment Key, H2 Form Reset, M1 Server-side Zod Validation, M2 PATCH Body Whitelist, M3 TierBadge Enterprise Styling, M4 Volume Monospace)

### File List

**Neue Dateien:**
- src/components/ui/dialog.tsx (shadcn installiert)
- src/hooks/use-admin-tenants.ts
- src/components/admin/admin-tenants-table.tsx
- src/components/admin/admin-tenant-detail.tsx
- src/components/admin/create-tenant-dialog.tsx
- src/components/admin/tenant-credentials-modal.tsx
- src/components/admin/tenant-toggle-button.tsx

**Modifizierte Dateien:**
- src/types/admin.ts (AdminTenant, AdminTenantCreateRequest, AdminTenantCreateResponse, AdminTenantPatchRequest)
- src/app/(admin)/admin/tenants/page.tsx (Stub ersetzt durch vollstaendige Page)
- src/app/api/admin/tenants/route.ts (snakeToCamel auf GET + POST Response)
- src/app/api/admin/tenants/[slug]/route.ts (snakeToCamel auf PATCH Response)
