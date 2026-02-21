# Story 6.4: Global Payments

Status: done

<!-- Note: Validation is optional. Run validate-create-story for quality check before dev-story. -->

## Story

As a Admin (Mario),
I want alle Payments ueber alle Tenants sehen und nach Tenant filtern koennen,
so that ich das Gesamtvolumen ueberblicke und bei Problemen schnell den betroffenen Tenant identifizieren kann.

## Acceptance Criteria

1. **Given** ich bin als Admin eingeloggt und oeffne `/admin/payments` (FR43)
   **When** die Payments-Seite laedt
   **Then** sehe ich eine Tabelle aller Payments ueber alle Tenants mit: Payment ID (monospace), Tenant (tenant_slug), Amount (monospace, USDC), Status (PaymentStatusBadge), txHash (monospace, truncated, Copy-Button), Created (Timestamp)
   **And** die Daten kommen von GET /v1/admin/paywatcher/payments via Proxy

2. **Given** ich den Tenant-Filter verwende
   **When** ich einen Tenant aus dem Dropdown waehle
   **Then** werden nur Payments dieses Tenants angezeigt (Query-Param `tenant_slug`)
   **And** der Filter ist als URL Query-Param gespeichert (`?tenant_slug=slug`)

3. **Given** die Admin Payments-Tabelle angezeigt wird
   **When** ich die gleichen Filter wie im User Dashboard nutze
   **Then** kann ich nach Status filtern (6 States), nach Zeitraum filtern (7d/30d/90d/All), und nach Created At sortieren
   **And** Pagination funktioniert identisch zum User Dashboard

4. **Given** ich auf ein Payment in der Tabelle klicke
   **When** die Detail-Ansicht angezeigt wird
   **Then** sehe ich die gleiche Payment-Detail-View wie im User Dashboard (Story 3.5) plus den Tenant-Slug als zusaetzliches Feld

5. **Given** die Payments-Daten laden
   **When** die API-Response noch aussteht
   **Then** sehe ich Skeleton-Zeilen als Loading-State

## Tasks / Subtasks

- [x] Task 1: Admin Payment Types definieren (AC: #1, #2)
  - [x] 1.1: `src/types/admin.ts` erweitern mit `AdminPayment` Interface — identisch zu `Payment` aus `src/types/payment.ts` plus zusaetzliches Feld `tenantSlug: string`
  - [x] 1.2: `AdminPaymentFilters` Interface — identisch zu `PaymentFilters` plus `tenantSlug?: string`
  - [x] 1.3: HINWEIS: MMS Admin Payments Response enthaelt `tenant_slug` pro Payment — `snakeToCamel` Transformation wird im Route Handler bereits angewendet (Route existiert bereits in `src/app/api/admin/payments/route.ts`)
  - [x] 1.4: HINWEIS: Pruefen ob MMS Admin Payments Response tatsaechlich `tenant_slug` pro Payment-Objekt enthaelt oder nur als Filter-Parameter dient. Types ggf. defensiv mit `tenantSlug?: string` (optional) definieren bis MMS Response verifiziert ist.

- [x] Task 2: `useAdminPayments` Hook erstellen (AC: #1, #3, #5)
  - [x] 2.1: `src/hooks/use-admin-payments.ts` erstellen — Pattern identisch zu `use-payments.ts`:
    ```typescript
    "use client"
    import { useQuery } from "@tanstack/react-query"
    import { parseApiError } from "@/lib/api/client-error"
    import type { AdminPayment, AdminPaymentFilters } from "@/types/admin"

    export interface AdminPaymentsResponse {
      data: AdminPayment[]
      meta: { page: number; limit: number; total: number }
    }

    async function fetchAdminPayments(filters?: AdminPaymentFilters): Promise<AdminPaymentsResponse> {
      const params = new URLSearchParams()
      if (filters?.status) {
        const statuses = Array.isArray(filters.status) ? filters.status : [filters.status]
        params.set("status", statuses.join(","))
      }
      if (filters?.from) params.set("from", filters.from)
      if (filters?.to) params.set("to", filters.to)
      if (filters?.page) params.set("page", String(filters.page))
      if (filters?.limit) params.set("limit", String(filters.limit))
      if (filters?.sort) params.set("sort", filters.sort)
      if (filters?.order) params.set("order", filters.order)
      if (filters?.tenantSlug) params.set("tenant_slug", filters.tenantSlug)

      const query = params.toString()
      const res = await fetch(`/api/admin/payments${query ? `?${query}` : ""}`)
      if (!res.ok) throw await parseApiError(res)
      return res.json()
    }

    export function useAdminPayments(filters?: AdminPaymentFilters) {
      return useQuery({
        queryKey: ["admin-payments", filters],
        queryFn: () => fetchAdminPayments(filters),
        staleTime: 30_000,
      })
    }
    ```
  - [x] 2.2: Kein dynamisches `refetchInterval` — Admin Payments aendern sich nicht so haeufig wie User Payments
  - [x] 2.3: Query Key `["admin-payments", filters]` — getrennt vom User Dashboard `["payments", filters]`

- [x] Task 3: Admin Payments API Route pruefen und ggf. anpassen (AC: #1)
  - [x] 3.1: **Route existiert bereits:** `src/app/api/admin/payments/route.ts` — GET Handler mit `requireAdminAuth()` + `mmsApi()` + Query-Param Weiterleitung (status, from, to, page, limit, sort, order, tenant_slug)
  - [x] 3.2: **Pruefen:** Ob `snakeToCamel()` Transformation auf Response angewendet wird. Payment-Responses von MMS sind typischerweise bereits camelCase (wie beim User Payments Endpoint). Falls nicht, Transformation hinzufuegen.
  - [x] 3.3: **WICHTIG:** KEINE Aenderung noetig wenn Response bereits korrekt ist. Route Handler NICHT unnoetig modifizieren.
  - [x] 3.4: HINWEIS: Der Route Handler leitet die Response direkt von MMS weiter (`NextResponse.json(data)`). Wenn MMS `tenant_slug` als snake_case im Payment-Objekt liefert, muss ggf. `snakeToCamel()` hinzugefuegt werden. Dev muss MMS Response inspizieren.

- [x] Task 4: `useAdminPaymentFilters` Hook erstellen (AC: #2, #3)
  - [x] 4.1: `src/hooks/use-admin-payment-filters.ts` erstellen — basierend auf bestehendem `use-payment-filters.ts` Pattern:
    - Identisches URL-State-Management (searchParams)
    - Gleiche Period/Status/Sort/Order/Page Logik
    - ZUSAETZLICH: `tenant_slug` als URL-Param lesen und setzen
    - HINWEIS: Tenant-Slug kommt oft ueber den Link aus der Tenants-Tabelle (`/admin/payments?tenant_slug=acme`), muss also beim Laden der Seite aus den URL-Params gelesen werden
  - [x] 4.2: `setFilter` Funktion erweitern um `tenant_slug` Handling:
    ```typescript
    // Wenn tenant_slug gesetzt wird:
    setFilter({ tenant_slug: "acme" }) // setzt URL-Param
    setFilter({ tenant_slug: null })   // entfernt URL-Param (zeigt alle)
    ```
  - [x] 4.3: ALTERNATIV-OPTION: Bestehenden `usePaymentFilters()` Hook wiederverwenden und nur `tenant_slug` Handling hinzufuegen (WENIGER DUPLIZIERUNG). Abwaegung: Aenderungen am bestehenden Hook koennten User Dashboard beeinflussen. Sicherere Option: SEPARATER Hook fuer Admin. Dev soll bestehenden Hook LESEN und entscheiden welche Option weniger Risiko birgt.

- [x] Task 5: Tenant-Selector-Dropdown erstellen (AC: #2)
  - [x] 5.1: `src/components/admin/admin-tenant-filter.tsx` erstellen:
    - Dropdown/Select mit Tenant-Liste (aus `useAdminTenants()` Hook, bereits vorhanden)
    - "All Tenants" als Default-Option (kein Filter)
    - Tenant-Name + Slug im Dropdown anzeigen
    - Bei Auswahl: `setFilter({ tenant_slug: slug })` aufrufen
    - Bei "All Tenants": `setFilter({ tenant_slug: null })` aufrufen
  - [x] 5.2: Dropdown soll Tenant-Liste beim Oeffnen laden (oder aus bestehendem Query Cache nutzen da `useAdminTenants()` bereits auf der Tenants-Seite verwendet wird)
  - [x] 5.3: Aktiver Tenant-Filter visuell erkennbar (z.B. Badge mit Tenant-Name oder gefuellter Select-State)
  - [x] 5.4: Wenn `tenant_slug` als URL-Param vorhanden ist (z.B. via Link von Tenants-Tabelle), Dropdown auf diesen Wert vorselektieren

- [x] Task 6: Admin Payments-Tabelle erstellen (AC: #1, #4)
  - [x] 6.1: `src/components/admin/admin-payments-table.tsx` erstellen:
    - **NICHT** aus `src/components/dashboard/payments-table.tsx` importieren (Boundary-Regel aus Story 6.1/6.3 Learnings: Admin Components NIE aus dashboard/ importieren)
    - Tabelle mit Spalten: Payment ID (monospace, truncated), **Tenant** (tenant_slug, monospace), Amount (monospace, USDC), Status (PaymentStatusBadge), txHash (monospace, truncated, Copy-Button), Created (Timestamp, sortierbar)
    - Sortierung: Click auf "Created" Header togglet asc/desc, Pfeil-Indikator
    - Klickbare Zeilen: Link zu `/admin/payments/[id]` (Payment Detail)
    - **PaymentStatusBadge** darf aus `src/components/dashboard/payment-status-badge.tsx` importiert werden — es ist eine shared/reusable Utility-Komponente (Farb+Label Mapping). ABER: Wenn Boundary-Regel strikt eingehalten werden soll, inline Badge-Logik duplizieren (identisches Pattern wie TenantStatusBadge in Story 6.3). **Dev soll bestehende Imports in admin-Komponenten pruefen** — wenn `PaymentStatusBadge` bereits in admin/ importiert wird, kann es wiederverwendet werden.
    - ALTERNATIVE: `PaymentStatusBadge` nach `src/components/shared/` verschieben (refactor). Abwaegung: Mehr Arbeit, aber korrekter Ort fuer Cross-Boundary Component. **NUR WENN EINFACH und ohne Regression moeglich.**
  - [x] 6.2: `AdminPaymentsTableSkeleton` mit 8 Skeleton-Zeilen (mehr als User Dashboard weil Admin typischerweise mehr Daten sieht)
  - [x] 6.3: Empty State: "No payments found." (dezent, kein grosses Illustration)
  - [x] 6.4: txHash als Link zu Basescan (oeffnet in neuem Tab, NFR14) + InlineCopyButton (shared Component, darf importiert werden)
  - [x] 6.5: Amounts in Geist Mono (`font-mono`), formatiert mit `formatAmount()` aus `src/lib/utils.ts`
  - [x] 6.6: Tenant-Slug-Spalte als Link zu `/admin/tenants` (optional, oder nur als Text) — Dev entscheidet ob sinnvoll

- [x] Task 7: Admin Payments Page implementieren (AC: #1, #2, #3, #5)
  - [x] 7.1: `src/app/(admin)/admin/payments/page.tsx` — Stub ("coming soon") ersetzen durch vollstaendige Payments Page:
    ```
    ┌─ Global Payments ──────────────────────────────────────────┐
    │                                                             │
    │  [h1: Payments]                                             │
    │                                                             │
    │  ┌─ Filters ──────────────────────────────────────────────┐│
    │  │ [Tenant: All ▼]  [Period: 30d ▼]  [Status: All ▼]    ││
    │  └────────────────────────────────────────────────────────┘│
    │                                                             │
    │  ┌──────────────────────────────────────────────────────┐  │
    │  │ ID     │ Tenant │ Amount │ Status │ txHash  │Created │  │
    │  │────────┼────────┼────────┼────────┼─────────┼────────│  │
    │  │ pay_7f │ acme   │ 49 USDC│ ●conf  │ 0x7a3f..│ Feb 21 │  │
    │  │ pay_3a │ test   │ 10 USDC│ ○pend  │ —       │ Feb 20 │  │
    │  └──────────────────────────────────────────────────────┘  │
    │                                                             │
    │  [← Prev]  Page 1 of 5  [Next →]                          │
    └─────────────────────────────────────────────────────────────┘
    ```
  - [x] 7.2: Page Structure:
    ```typescript
    "use client"
    import { Suspense } from "react"

    function AdminPaymentsContent() {
      const { filters, period, setFilter } = useAdminPaymentFilters()
      const { data, isLoading, error } = useAdminPayments(filters)
      // ... render Filter Bar + Tenant Selector + Table + Pagination
    }

    export default function AdminPaymentsPage() {
      return (
        <Suspense fallback={<AdminPaymentsPageFallback />}>
          <AdminPaymentsContent />
        </Suspense>
      )
    }
    ```
  - [x] 7.3: Filter-Leiste: Tenant-Selector (links) + PaymentFilterBar (Period + Status) + Sort-Toggle
  - [x] 7.4: Loading State: `AdminPaymentsTableSkeleton`
  - [x] 7.5: Error State: Fehlermeldung mit `text-destructive` (Pattern wie andere Admin-Seiten)
  - [x] 7.6: Admin ist Desktop-Only — kein Mobile-Layout, keine Card-Ansicht noetig

- [x] Task 8: Admin Payment Detail Page implementieren (AC: #4)
  - [x] 8.1: `src/app/(admin)/admin/payments/[id]/page.tsx` erstellen:
    - Payment Detail via `/api/admin/payments/[id]` laden — **ACHTUNG:** Dieser Route Handler existiert NICHT! Muss erstellt werden (siehe Task 8.2)
    - ALTERNATIVE: Bestehenden User Payments Detail Proxy nutzen (`/api/payments/[id]`). PROBLEM: Der User Endpoint nutzt den Tenant API Key aus der Session, nicht den Admin Key. Admin braucht eigenen Proxy.
  - [x] 8.2: `src/app/api/admin/payments/[id]/route.ts` erstellen:
    - GET Handler mit `requireAdminAuth()` + `mmsApi("/v1/admin/paywatcher/payments/${id}")`
    - HINWEIS: Pruefen ob MMS einen Admin Payment Detail Endpoint hat. Falls nicht (nur GET /v1/admin/paywatcher/payments als Liste), dann Payment via User Endpoint `/v1/payments/${id}` mit Admin API Key laden.
    - Defensive Implementierung: Erst `/v1/admin/paywatcher/payments/${id}` versuchen, bei 404 auf `/v1/payments/${id}` mit Admin Key zurueckfallen
  - [x] 8.3: `useAdminPayment(id)` Hook in `src/hooks/use-admin-payments.ts` hinzufuegen:
    ```typescript
    export function useAdminPayment(id: string) {
      return useQuery({
        queryKey: ["admin-payments", id],
        queryFn: async () => {
          const res = await fetch(`/api/admin/payments/${id}`)
          if (!res.ok) throw await parseApiError(res)
          const json = await res.json()
          return json.data ?? json
        },
      })
    }
    ```
  - [x] 8.4: Detail-Seite zeigt alle Payment-Felder (identisch zu User Dashboard Story 3.5):
    - id, status, amount, exactAmount, currency, chain, depositAddress, confirmations, confirmationsRequired, txHash, metadata, expiresAt, createdAt, updatedAt
    - PLUS: `tenantSlug` als zusaetzliches Feld (prominent oben)
  - [x] 8.5: `src/components/admin/admin-payment-detail.tsx` erstellen:
    - Key-Value Layout (dl Pattern wie AdminTenantDetail)
    - Tenant-Slug als erstes Feld mit Link zu `/admin/tenants` (oder inline-Anzeige)
    - Basescan Links fuer txHash und depositAddress
    - Copy-Buttons fuer IDs und Adressen (InlineCopyButton, shared)
    - PaymentStatusBadge fuer Status (size="md")
    - Amounts in Geist Mono
  - [x] 8.6: Zurueck-Navigation: "← Back to Payments" Link oben (zurueck zur Liste mit erhaltenen Filter-Params)
  - [x] 8.7: Webhook History Section — OPTIONAL/NICE-TO-HAVE:
    - Pruefen ob GET /v1/admin/paywatcher/payments/:id/webhooks existiert oder ob `/v1/payments/:id/webhooks` mit Admin Key funktioniert
    - Falls verfuegbar: Webhook History Tabelle unter Payment Detail (Event Name, Status Code, Timestamp)
    - Falls nicht verfuegbar: Section weglassen, kein Stub/Placeholder

- [x] Task 9: Bestehende Admin Route Header Titel aktualisieren (AC: #1)
  - [x] 9.1: `src/components/admin/admin-header.tsx` pruefen — der `routeTitles` Map sollte bereits `/admin/payments` enthalten. Falls nicht, hinzufuegen:
    ```typescript
    const routeTitles: Record<string, string> = {
      "/admin": "Overview",
      "/admin/tenants": "Tenants",
      "/admin/payments": "Payments",
    }
    ```
  - [x] 9.2: Fuer Payment Detail Route ggf. Fallback-Titel "Payment Detail" hinzufuegen wenn Pattern `/admin/payments/[id]` erkannt wird

## Dev Notes

### Kritische Architektur-Erkenntnisse

**API Route existiert bereits:**
- `src/app/api/admin/payments/route.ts` ist bereits implementiert (Story 6.1)
- Akzeptiert `tenant_slug` als Query-Param (neben status, from, to, page, limit, sort, order)
- Nutzt `requireAdminAuth()` + `mmsApi()` mit `MMS_API_KEY` (ist der Admin Key, kein separates `MMS_ADMIN_API_KEY`)
- **Kein neuer Route Handler fuer die Liste noetig** — nur neuer Hook und Page Component

**Boundary-Regel (KRITISCH):**
- Alle Admin Components in `src/components/admin/` — NIE aus `src/components/dashboard/` importieren
- UI Components aus `src/components/ui/` sind shared und duerfen verwendet werden
- Shared Components aus `src/components/shared/` (InlineCopyButton, etc.) duerfen verwendet werden
- `PaymentStatusBadge` ist in `src/components/dashboard/` — wenn Boundary strikt: inline duplizieren ODER nach shared/ verschieben
- **Dev soll bestehende Imports in admin-Komponenten pruefen** und konsistent bleiben

**MMS Admin Payments Response — Shape unbekannt:**
- Die exakte JSON-Struktur von `GET /v1/admin/paywatcher/payments` ist nicht vollstaendig spezifiziert
- Ob `tenant_slug` pro Payment-Objekt enthalten ist, muss verifiziert werden
- **ERSTER SCHRITT:** Dev-Server starten, als Admin einloggen, `/api/admin/payments` aufrufen und Response inspizieren
- Falls `tenant_slug` nicht im Payment-Objekt enthalten ist: Alternative Anzeige-Strategie (z.B. nur als Filter verwenden, nicht als Spalte)

**Admin Payment Detail Endpoint — Existenz unklar:**
- `GET /v1/admin/paywatcher/payments/:id` ist in der MMS API Docs nicht explizit erwaehnt
- Der User Endpoint `GET /v1/payments/:id` existiert, braucht aber den Tenant API Key
- **Strategie:** Admin Payment Detail Route versucht erst `/v1/payments/${id}` mit Admin API Key. Falls 403/404, Fehlermeldung anzeigen.
- ALTERNATIVE: Kein Detail-Page, nur Inline-Expansion in der Tabelle (wie TenantDetail in Story 6.3). Einfacher, weniger API-Abhaengigkeit.

**snake_case Transformation:**
- `snakeToCamel` existiert in `src/lib/api/transform.ts`
- User Payment Responses sind bereits camelCase vom MMS Backend
- Admin Payment Responses KOENNTEN anders sein — Dev muss pruefen
- Im Zweifelsfall `snakeToCamel()` anwenden (ist idempotent bei camelCase Input)

**Admin ist Desktop-Only:**
- Kein Mobile-Layout, kein Responsive-Handling, keine Card-Ansicht
- Sidebar ist immer sichtbar (200px), nicht collapsible
- Tabelle kann volle Breite nutzen

### Pattern-Uebernahme aus bestehender Codebase

| Aspekt | Vorlage | Admin Payments |
|--------|---------|----------------|
| Page Component | `admin/tenants/page.tsx` | `admin/payments/page.tsx` |
| Loading | `AdminTenantsTableSkeleton` | `AdminPaymentsTableSkeleton` |
| Error | `text-destructive` | `text-destructive` |
| Table Pattern | `AdminTenantsTable` | `AdminPaymentsTable` |
| Filter Pattern | `PaymentFilterBar` (Dashboard) | Eigene Admin-Version + TenantSelector |
| Badge Pattern | `PaymentStatusBadge` | Wiederverwendung oder Inline-Duplikat |
| Detail Pattern | `AdminTenantDetail` (dl key-value) | `AdminPaymentDetail` |
| Pagination | `PaymentPagination` (Dashboard) | Eigene Admin-Version oder Shared-Import |
| Hook Pattern | `use-admin-tenants.ts` | `use-admin-payments.ts` |
| URL State | `use-payment-filters.ts` | `use-admin-payment-filters.ts` |

### Hook Pattern (identisch zu bestehenden Hooks)

```typescript
// src/hooks/use-admin-payments.ts
"use client"
import { useQuery } from "@tanstack/react-query"
import { parseApiError } from "@/lib/api/client-error"
import type { AdminPayment } from "@/types/admin"

export interface AdminPaymentsResponse {
  data: AdminPayment[]
  meta: { page: number; limit: number; total: number }
}

async function fetchAdminPayments(filters?: AdminPaymentFilters): Promise<AdminPaymentsResponse> {
  const params = new URLSearchParams()
  // ... build params from filters
  const query = params.toString()
  const res = await fetch(`/api/admin/payments${query ? `?${query}` : ""}`)
  if (!res.ok) throw await parseApiError(res)
  return res.json()
}

export function useAdminPayments(filters?: AdminPaymentFilters) {
  return useQuery({
    queryKey: ["admin-payments", filters],
    queryFn: () => fetchAdminPayments(filters),
    staleTime: 30_000,
  })
}
```

### Bestehende wiederverwendbare UI Components

| Komponente | Pfad | Wiederverwendung |
|-----------|------|------------------|
| `Table` / `TableHeader` etc. | `src/components/ui/table.tsx` | Fuer AdminPaymentsTable |
| `Badge` | `src/components/ui/badge.tsx` | Fuer PaymentStatusBadge |
| `Button` | `src/components/ui/button.tsx` | Pagination, Links |
| `Select` | `src/components/ui/select.tsx` | Tenant-Selector Dropdown |
| `Skeleton` | `src/components/ui/skeleton.tsx` | Loading States |
| `InlineCopyButton` | `src/components/shared/inline-copy-button.tsx` | Copy-Buttons fuer txHash, Payment ID |
| `cn()` | `src/lib/utils.ts` | Conditional classNames |
| `formatAmount()` | `src/lib/utils.ts` | Amount Formatting (49.000042 USDC) |
| `truncateId()` | `src/lib/utils.ts` | Payment ID Truncation |
| `truncateTxHash()` | `src/lib/utils.ts` | txHash Truncation |
| `formatTimestamp()` | `src/lib/utils.ts` | Timestamp Formatting |
| `PAYMENT_STATUSES` | `src/lib/utils.ts` | Status Config Mapping |
| `snakeToCamel()` | `src/lib/api/transform.ts` | Response Transformation |
| `parseApiError()` | `src/lib/api/client-error.ts` | Client Error Handling |
| `requireAdminAuth()` | `src/lib/admin.ts` | Admin Auth Check |
| `mmsApi()` | `src/lib/api/client.ts` | MMS API Client |

### Bestehende Dateien die referenziert werden

| Datei | Zweck |
|-------|-------|
| `src/app/api/admin/payments/route.ts` | **EXISTIERT BEREITS** — Admin Payments List Proxy (GET) |
| `src/app/(admin)/admin/payments/page.tsx` | **EXISTIERT ALS STUB** — "coming soon", wird ersetzt |
| `src/hooks/use-payment-filters.ts` | Pattern-Vorlage fuer URL-State-Management |
| `src/hooks/use-payments.ts` | Pattern-Vorlage fuer Payments Hook |
| `src/hooks/use-admin-tenants.ts` | Pattern-Vorlage fuer Admin Hook |
| `src/components/dashboard/payments-table.tsx` | Referenz-Pattern fuer Tabelle (NICHT importieren) |
| `src/components/dashboard/payment-status-badge.tsx` | Badge Pattern (Boundary-Entscheidung noetig) |
| `src/components/dashboard/payment-filter-bar.tsx` | Filter Pattern (Boundary-Entscheidung noetig) |
| `src/components/dashboard/payment-pagination.tsx` | Pagination Pattern (Boundary-Entscheidung noetig) |
| `src/components/admin/admin-tenants-table.tsx` | "View Payments" Link zeigt bereits auf `/admin/payments?tenant_slug=...` |
| `src/components/admin/admin-tenant-detail.tsx` | "View Payments" Link Pattern |
| `src/types/payment.ts` | Payment, PaymentStatus, PaymentFilters Types |
| `src/types/admin.ts` | AdminTenant, AdminHealthData (erweitern mit AdminPayment) |

### Neue Dateien die erstellt werden

| Datei | Zweck |
|-------|-------|
| `src/hooks/use-admin-payments.ts` | TanStack Query Hook fuer Admin Payments (useAdminPayments, useAdminPayment) |
| `src/hooks/use-admin-payment-filters.ts` | URL-State-Management fuer Admin Payment Filter (Period, Status, Tenant, Sort, Page) |
| `src/components/admin/admin-payments-table.tsx` | Payments-Tabelle mit Tenant-Spalte, Skeleton, Empty State |
| `src/components/admin/admin-tenant-filter.tsx` | Tenant-Selector Dropdown (nutzt useAdminTenants) |
| `src/components/admin/admin-payment-detail.tsx` | Payment Detail Ansicht mit Tenant-Slug (dl Key-Value Layout) |
| `src/app/(admin)/admin/payments/[id]/page.tsx` | Payment Detail Page |
| `src/app/api/admin/payments/[id]/route.ts` | Admin Payment Detail Proxy Route (GET) |

### Bestehende Dateien die modifiziert werden

| Datei | Aenderung |
|-------|----------|
| `src/types/admin.ts` | AdminPayment, AdminPaymentFilters Types hinzufuegen |
| `src/app/(admin)/admin/payments/page.tsx` | Stub ersetzen durch vollstaendige Payments Page |
| `src/app/api/admin/payments/route.ts` | ggf. `snakeToCamel()` hinzufuegen falls Response snake_case enthaelt |
| `src/components/admin/admin-header.tsx` | ggf. Payment Detail Route Titel hinzufuegen |

### Learnings aus Story 6.1, 6.2 und 6.3

- `MMS_API_KEY` ist der Admin Key — kein separates `MMS_ADMIN_API_KEY` noetig
- Admin Layout prueft Auth auf Layout-Level (kein middleware.ts)
- MMS Health Response hatte komplett andere Shape als angenommen — Types MUESSEN verifiziert werden
- `snakeToCamel` Transformation immer anwenden (idempotent bei camelCase)
- Defensive Programmierung: Optional Chaining + Fallback-Werte ueberall
- Error Messages auf English (nicht Deutsch)
- Boundary-Regel strikt einhalten: Admin Components NIE aus dashboard/ importieren
- `parseApiError()` aus `@/lib/api/client-error` verwenden (nicht raw error handling)
- Admin Tenants Page nutzt: useQuery + Inline Skeleton + text-destructive Error
- Nach Code Review Story 6.3: Form Reset nach Success, Server-side Zod Validation, PATCH Body Whitelist
- "View Payments" Link in AdminTenantDetail setzt bereits `tenant_slug` als URL-Param

### Git Intelligence

Letzte Commits zeigen:
- `71fbd19` Add admin dashboard with tenant management (Epic 6: Stories 6.1-6.3)
- `3b67da2` Improve onboarding UX for new tenants without config
- `63ddb15` Handle MMS 404 gracefully for new tenants without config

Patterns:
- Grosse Commits pro Story (alle Tasks in einem Commit)
- Error Handling und Graceful Degradation sind wichtig
- MMS 404 bei fehlendem Config muss abgefangen werden
- Defensive Programmierung durchgehend

### Project Structure Notes

- Alignment mit Admin Route Group: `app/(admin)/admin/payments/` (bestehende Konvention)
- Admin Payments Stub existiert bereits — wird durch echte Page ersetzt
- API Route fuer Payments-Liste existiert bereits — ggf. minimale Anpassung
- Neue API Route fuer Payment Detail muss erstellt werden
- Alle neuen Components in `src/components/admin/` (Boundary-Regel)
- Types in `src/types/admin.ts` erweitern (nicht neue Datei)

### References

- [Source: _bmad-output/planning-artifacts/epics.md#Epic 6 - Story 6.4]
- [Source: _bmad-output/planning-artifacts/architecture.md#API Communication Pattern - Data Flow (Admin)]
- [Source: _bmad-output/planning-artifacts/architecture.md#Component Patterns]
- [Source: _bmad-output/planning-artifacts/architecture.md#TanStack Query Patterns]
- [Source: _bmad-output/implementation-artifacts/6-3-tenant-management.md - Boundary-Regel, Pattern-Uebernahme]
- [Source: _bmad-output/implementation-artifacts/6-3-tenant-management.md - Learnings und Code Review Fixes]
- [Source: src/app/api/admin/payments/route.ts - Admin Payments Proxy Route (GET, bereits implementiert)]
- [Source: src/app/(admin)/admin/payments/page.tsx - Stub Page (wird ersetzt)]
- [Source: src/hooks/use-payments.ts - Hook Pattern Template]
- [Source: src/hooks/use-payment-filters.ts - URL State Management Pattern]
- [Source: src/hooks/use-admin-tenants.ts - Admin Hook Pattern]
- [Source: src/components/dashboard/payments-table.tsx - Table Pattern (Referenz, nicht importieren)]
- [Source: src/components/dashboard/payment-status-badge.tsx - Badge Pattern (Boundary-Entscheidung)]
- [Source: src/components/dashboard/payment-filter-bar.tsx - Filter Pattern (Referenz)]
- [Source: src/components/dashboard/payment-pagination.tsx - Pagination Pattern (Referenz)]
- [Source: src/components/admin/admin-tenants-table.tsx - Table Pattern + "View Payments" Link]
- [Source: src/components/admin/admin-tenant-detail.tsx - Detail dl Pattern + Payments Link]
- [Source: src/lib/api/client.ts - mmsApi() mit MMS_API_KEY Fallback]
- [Source: src/lib/api/transform.ts - snakeToCamel Utility]
- [Source: src/lib/api/client-error.ts - parseApiError Pattern]
- [Source: src/lib/admin.ts - requireAdminAuth Pattern]
- [Source: src/lib/utils.ts - formatAmount, truncateId, truncateTxHash, formatTimestamp, PAYMENT_STATUSES]
- [Source: src/types/payment.ts - Payment, PaymentStatus, PaymentFilters Types]
- [Source: src/types/admin.ts - AdminTenant, AdminHealthData Types]

## Dev Agent Record

### Agent Model Used

Claude Opus 4.6 (claude-opus-4-6)

### Debug Log References

- TypeScript check: PASS (0 errors)
- ESLint: Only pre-existing errors in welcome-screen.tsx, mdx-code-tabs.tsx, auth.ts — no new errors
- IDE Diagnostics: 0 errors on all new/modified files
- No test framework configured in project (vitest found 0 test files)

### Completion Notes List

- Task 1: Added `AdminPayment` and `AdminPaymentFilters` interfaces to `src/types/admin.ts`. `tenantSlug` defined as optional (`tenantSlug?: string`) until MMS response shape is verified.
- Task 2: Created `useAdminPayments` and `useAdminPayment` hooks in `src/hooks/use-admin-payments.ts` following existing `use-payments.ts` pattern. Defensive fallback for meta (`{ page: 1, limit: 20, total: 0 }`).
- Task 3: Added `snakeToCamel()` transformation to admin payments route response. Import added, applied idempotently to MMS response.
- Task 4: Created separate `useAdminPaymentFilters` hook (safer option — no changes to user hook). Reads `tenant_slug` from URL params, supports all filter types from user hook plus tenant filtering.
- Task 5: Created `AdminTenantFilter` dropdown component using `useAdminTenants()` hook. Supports "All Tenants" default, pre-selects from URL param.
- Task 6: Created `AdminPaymentsTable` with Tenant column, inline `AdminPaymentStatusBadge` (boundary-compliant — no dashboard imports), Basescan links, InlineCopyButton, sort toggle, skeleton (8 rows), empty state.
- Task 7: Replaced stub page with full Payments page: Suspense wrapper, filter bar (tenant + period + status), table, pagination. Desktop-only layout.
- Task 8: Created admin payment detail API route (`/api/admin/payments/[id]`) with ID validation, `snakeToCamel`, requireAdminAuth. Created `AdminPaymentDetail` component (dl key-value layout, tenant slug as first field). Created detail page with back navigation.
- Task 9: Admin header already had `/admin/payments` title. Added dynamic fallback for `/admin/payments/[id]` → "Payment Detail".

### Change Log

- 2026-02-21: Implemented Story 6.4 — Global Payments (Admin Dashboard). Full payments list with tenant filtering, status/period filters, sort, pagination. Payment detail page with all fields plus tenant slug. All 9 tasks completed.
- 2026-02-21: Code Review Fixes (6 issues fixed):
  - H1: Added regex validation (`/^[\w-]+$/`) to payment detail API route to prevent path traversal
  - H2: `AdminPayment` now extends `Payment` instead of duplicating fields (drift prevention)
  - M1: Back-navigation in detail page uses `router.back()` to preserve filter state
  - M2: Table rows are now fully clickable with `cursor-pointer` + `onClick` (stopPropagation on interactive elements)
  - M3: Extracted shared `AdminPaymentStatusBadge` component (was duplicated in table + detail)
  - M4: Extracted shared filter constants (`VALID_STATUSES`, `PERIOD_PRESETS`, `getFromDate`) to `src/lib/payment-filter-constants.ts`

### File List

**New files:**
- src/hooks/use-admin-payments.ts
- src/hooks/use-admin-payment-filters.ts
- src/components/admin/admin-tenant-filter.tsx
- src/components/admin/admin-payments-table.tsx
- src/components/admin/admin-payment-detail.tsx
- src/components/admin/admin-payment-status-badge.tsx (extracted from table + detail)
- src/lib/payment-filter-constants.ts (shared filter constants)
- src/app/(admin)/admin/payments/[id]/page.tsx
- src/app/api/admin/payments/[id]/route.ts

**Modified files:**
- src/types/admin.ts (AdminPayment extends Payment, proper import)
- src/app/(admin)/admin/payments/page.tsx (replaced stub with full implementation)
- src/app/api/admin/payments/route.ts (added snakeToCamel transformation)
- src/app/api/admin/payments/[id]/route.ts (added regex ID validation)
- src/components/admin/admin-header.tsx (added Payment Detail title fallback)
- src/hooks/use-payment-filters.ts (uses shared constants from payment-filter-constants.ts)
