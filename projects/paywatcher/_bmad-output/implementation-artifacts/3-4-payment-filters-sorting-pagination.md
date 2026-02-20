# Story 3.4: Payment Filters, Sorting & Pagination

Status: done

<!-- Note: Validation is optional. Run validate-create-story for quality check before dev-story. -->

## Story

As a Tenant,
I want Payments nach Status und Zeitraum filtern, sortieren und durch Seiten blättern können,
So that ich bei vielen Payments schnell die relevanten finde.

## Acceptance Criteria

1. **Given** ich bin auf der Payments-Seite (FR16) **When** ich den Status-Filter verwende **Then** kann ich nach einem oder mehreren der 6 Status filtern: pending, matched, confirming, confirmed, expired, failed **And** die Tabelle wird sofort aktualisiert **And** der aktive Filter ist visuell erkennbar

2. **Given** ich bin auf der Payments-Seite (FR17) **When** ich den Zeitraum-Filter verwende **Then** kann ich aus vordefinierten Zeiträumen wählen: 7d, 30d, 90d, All **And** die Tabelle wird sofort aktualisiert **And** der Default-Zeitraum ist "30d"

3. **Given** ich bin auf der Payments-Seite (FR18) **When** ich auf die Spaltenüberschrift "Created" klicke **Then** wird die Liste nach Created At sortiert **And** ein Klick togglet zwischen aufsteigend und absteigend **And** ein Sortier-Indikator (Pfeil) zeigt die aktive Sortierung und Richtung **And** HINWEIS: Die MMS API unterstützt NUR Sortierung nach `created_at` — keine Sortierung nach Amount oder Status möglich

4. **Given** mehr Payments existieren als auf einer Seite passen (FR19) **When** die Pagination angezeigt wird **Then** sehe ich Prev/Next-Buttons und die aktuelle Seitenzahl **And** die Navigation funktioniert via Ghost-Buttons

5. **Given** ich Filter, Sortierung oder Pagination ändere **When** die URL aktualisiert wird **Then** werden die Filter-Parameter als Query-Params in der URL gespeichert (z.B. `?status=confirmed&period=7d&order=desc&page=2`) **And** die URL ist teilbar/bookmarkbar — beim Öffnen werden die Filter wiederhergestellt

6. **Given** die gesamte Payments-Seite mit Filtern und Pagination geladen wird (NFR2) **When** die Daten vom MMS Backend geholt werden **Then** dauert der Datenabruf maximal 2 Sekunden

7. **Given** die Filter-Leiste auf Mobile (<1024px) angezeigt wird **When** die Seite rendert **Then** sind die Filter als kompakte Dropdowns/Selects dargestellt und Touch-freundlich (min. 44x44px)

## Tasks / Subtasks

- [x] Task 1: shadcn Select installieren & URL-State-Utility erstellen (AC: #5)
  - [x] 1.1 `npx shadcn@latest add select` — shadcn Select-Primitives installieren
  - [x] 1.2 `src/hooks/use-payment-filters.ts` — Custom Hook: liest Filter aus URL Search Params, schreibt Änderungen zurück via `useRouter().replace()`. Gibt typisiertes `PaymentFilters`-Objekt zurück + Setter-Funktionen. Default: `period=30d, order=desc, page=1`

- [x] Task 2: Payment Filter Bar Komponente erstellen (AC: #1, #2, #7)
  - [x] 2.1 `src/components/dashboard/payment-filter-bar.tsx` — Filter-Leiste mit:
    - Status-Filter: Multi-Select Dropdown (DropdownMenu mit Checkboxes), zeigt aktive Filter als Badges
    - Zeitraum-Filter: shadcn Select mit Optionen 7d, 30d, 90d, All
    - Active Filters visuell erkennbar (Badge-Count oder farbige Chips)
  - [x] 2.2 Mobile-Layout: Filter nebeneinander als kompakte Selects, Touch-freundlich (min-h-[44px])

- [x] Task 3: Sortierung auf "Created" Spaltenüberschrift (AC: #3)
  - [x] 3.1 `src/components/dashboard/payments-table.tsx` — ERWEITERN: "Created" Spaltenüberschrift klickbar machen mit Sort-Indikator (ArrowUp/ArrowDown Icon)
  - [x] 3.2 Klick toggled `order` zwischen `asc` und `desc` (Default: `desc`)
  - [x] 3.3 NUR "Created" Spalte ist sortierbar — andere Spalten haben KEINE Sortier-Funktion

- [x] Task 4: Pagination Controls erstellen (AC: #4)
  - [x] 4.1 `src/components/dashboard/payment-pagination.tsx` — Pagination-Komponente mit:
    - Prev/Next Ghost-Buttons (shadcn Button variant="ghost")
    - Aktuelle Seitenzahl / Gesamtseiten Anzeige
    - Buttons disabled wenn erste/letzte Seite
  - [x] 4.2 Page-Size: 20 Items pro Seite (Default der API: 50 ist zu viel für UI)

- [x] Task 5: Payments Page integrieren — Filter + Sort + Pagination (AC: #1-#7)
  - [x] 5.1 `src/app/(dashboard)/dashboard/payments/page.tsx` — UMBAUEN:
    - usePaymentFilters() Hook für URL-State
    - PaymentFilterBar oberhalb der Tabelle/Cards
    - PaymentPagination unterhalb der Tabelle/Cards
    - Filter-Objekt an usePayments(filters) übergeben
    - Zeitraum-Filter (period) in `from`/`to` ISO-Dates umrechnen
  - [x] 5.2 Auto-Refresh beibehalten: `refetchInterval: 10_000` bei aktiven Payments (bestehende Logik)
  - [x] 5.3 Loading-State: Skeleton-Zeilen beibehalten, Filter-Bar bleibt stabil (kein Skeleton für Filter)

- [x] Task 6: Build-Validierung
  - [x] 6.1 `npm run build` fehlerfrei
  - [x] 6.2 Bestehende Seiten (Landing, Dashboard Overview, Settings) funktionieren weiterhin
  - [x] 6.3 Filter-Parameter in URL teilbar/bookmarkbar — beim Öffnen der URL werden Filter wiederhergestellt
  - [x] 6.4 Desktop-Tabelle und Mobile-Cards mit Filtern korrekt
  - [x] 6.5 Pagination navigiert korrekt, Prev/Next disabled an Grenzen
  - [x] 6.6 Sort-Indikator auf "Created" Spalte wechselt korrekt

## Dev Notes

### Kontext & Zweck

Dies ist **Story 3.4 von Epic 3** (Payment Monitoring Dashboard). Sie erweitert die bestehende Payments List Page (Story 3.3) um **Filter, Sortierung und Pagination**. Das ist die "Power-User"-Funktionalität — die Tabelle wird damit voll nutzbar für Tenants mit vielen Payments.

**Was diese Story beinhaltet:**
- Status-Filter (Multi-Select, 6 Status-Optionen)
- Zeitraum-Filter (7d, 30d, 90d, All)
- Sortierung nach Created At (asc/desc Toggle)
- Pagination (Prev/Next mit Seitenzahl)
- URL-State Sync (Filter als Query-Params, teilbar/bookmarkbar)
- Mobile-optimierte Filter-Darstellung

**Was diese Story NICHT beinhaltet:**
- Payment Detail Page (Story 3.5)
- Webhook History (Story 3.5)
- Sortierung nach anderen Spalten als Created At (MMS API Limitierung)
- Erweiterte Suchfunktion (nicht in FRs)

### KRITISCH: shadcn Select muss installiert werden

Die shadcn Select-Komponente existiert noch NICHT im Projekt. Installation via CLI:

```bash
npx shadcn@latest add select
```

Dies erstellt `src/components/ui/select.tsx` mit den Primitives: `Select`, `SelectContent`, `SelectItem`, `SelectTrigger`, `SelectValue`.

**Bereits installiert und verwendbar:**
- `DropdownMenu` — für Status Multi-Select (Checkboxes)
- `Button` — für Pagination (variant="ghost")
- `Badge` — für aktive Filter-Anzeige

### KRITISCH: URL-State Management — KEIN Extra-Package

URL-State wird mit **Next.js nativen APIs** implementiert — KEIN `nuqs` oder andere Packages:

```typescript
// src/hooks/use-payment-filters.ts
"use client"
import { useSearchParams, useRouter, usePathname } from "next/navigation"
import { useMemo, useCallback } from "react"
import type { PaymentFilters } from "@/types/payment"

// Zeitraum-Presets → ISO-Dates
const PERIOD_PRESETS = {
  "7d": 7,
  "30d": 30,
  "90d": 90,
  all: null,
} as const

type PeriodKey = keyof typeof PERIOD_PRESETS

function getFromDate(period: PeriodKey): string | undefined {
  const days = PERIOD_PRESETS[period]
  if (!days) return undefined
  const d = new Date()
  d.setDate(d.getDate() - days)
  return d.toISOString()
}

export function usePaymentFilters() {
  const searchParams = useSearchParams()
  const router = useRouter()
  const pathname = usePathname()

  // Parse URL → Filters
  const filters = useMemo((): PaymentFilters => {
    const status = searchParams.get("status")
    const period = (searchParams.get("period") || "30d") as PeriodKey
    const order = (searchParams.get("order") || "desc") as "asc" | "desc"
    const page = Number(searchParams.get("page")) || 1

    return {
      status: status ? status.split(",") as PaymentStatus[] : undefined,
      from: getFromDate(period),
      sort: "created_at",
      order,
      page,
      limit: 20,
    }
  }, [searchParams])

  // Aktive Period für UI
  const period = (searchParams.get("period") || "30d") as PeriodKey

  // Update URL → Filters
  const setFilter = useCallback(
    (updates: Record<string, string | null>) => {
      const params = new URLSearchParams(searchParams.toString())
      for (const [key, value] of Object.entries(updates)) {
        if (value === null) {
          params.delete(key)
        } else {
          params.set(key, value)
        }
      }
      // Reset page when filters change (not when page itself changes)
      if (!("page" in updates)) {
        params.delete("page")
      }
      router.replace(`${pathname}?${params.toString()}`, { scroll: false })
    },
    [searchParams, router, pathname]
  )

  return { filters, period, setFilter }
}
```

**Wichtig:**
- `router.replace()` statt `router.push()` — kein History-Eintrag pro Filteränderung
- `{ scroll: false }` — kein Scroll-to-Top bei Filteränderung
- Page wird bei Filteränderung automatisch auf 1 zurückgesetzt
- `useMemo` für stabile Filter-Referenz (TanStack Query Key-Stabilität!)

### KRITISCH: Status-Filter als Multi-Select mit DropdownMenu

Für den Multi-Select Status-Filter die **bestehende DropdownMenu** Komponente nutzen (bereits installiert) — NICHT eine Custom Multi-Select bauen:

```typescript
// Konzept — Payment Filter Bar
import { DropdownMenu, DropdownMenuTrigger, DropdownMenuContent, DropdownMenuCheckboxItem } from "@/components/ui/dropdown-menu"
import { Button } from "@/components/ui/button"

// Status-Optionen aus PAYMENT_STATUSES Konstante
// Checkboxes in DropdownMenu — User kann mehrere Status an/abwählen
// Aktive Filter als Badge-Count auf dem Trigger-Button anzeigen
```

**Warum DropdownMenu statt Custom Multi-Select:**
- Bereits installiert (components/ui/dropdown-menu.tsx)
- `DropdownMenuCheckboxItem` ist perfekt für Multi-Select
- Touch-freundlich (shadcn/ui Standard)
- Kein Extra-Package nötig

### KRITISCH: Zeitraum-Filter als Period-Presets (NICHT als Date-Picker)

Die ACs definieren **vordefinierte Zeiträume** (7d, 30d, 90d, All) — KEIN Date-Range-Picker. Die `from`/`to` Werte werden client-seitig aus dem Period-Preset berechnet:

```typescript
// Period "30d" → from = new Date(now - 30 days).toISOString()
// Period "all" → from = undefined (kein from-Parameter → alle Payments)
// to wird nie gesetzt (immer bis jetzt)
```

**URL:** `?period=30d` (nicht `?from=2026-01-20T...`) — lesbarer, teilbar.

### KRITISCH: Sortierung NUR auf "Created" Spalte

Die MMS API unterstützt NUR Sortierung nach `created_at`. KEINE andere Spalte darf sortierbar sein:

```typescript
// payments-table.tsx — NUR die "Created" TableHead klickbar machen
<TableHead
  className="cursor-pointer select-none"
  onClick={() => onSortToggle()}
>
  Created
  {order === "desc" ? <ArrowDown className="ml-1 size-3.5" /> : <ArrowUp className="ml-1 size-3.5" />}
</TableHead>

// ALLE anderen TableHead: Kein onClick, kein Cursor-Pointer, kein Sort-Icon
```

### Bestehender Code-Stand (aus Story 3.1 + 3.2 + 3.3)

**Bereits vorhanden — WIEDERVERWENDEN, nicht neu erstellen:**

| Datei | Was existiert | Aktion |
|-------|-------------|--------|
| `src/hooks/use-payments.ts` | `usePayments(filters?, options?)` — nimmt bereits `PaymentFilters` | **VERWENDEN** — keine Änderung nötig |
| `src/types/payment.ts` | `PaymentFilters` mit status, from, to, page, limit, sort, order | **VERWENDEN** — keine Änderung nötig |
| `src/app/api/payments/route.ts` | Proxy leitet status, from, to, page, limit, sort, order weiter | **VERWENDEN** — keine Änderung nötig |
| `src/components/dashboard/payments-table.tsx` | Desktop DataTable (5 Spalten) | **ERWEITERN** — Sort auf Created-Head |
| `src/components/dashboard/payment-card.tsx` | Mobile Card-Layout | **VERWENDEN** — keine Änderung nötig |
| `src/app/(dashboard)/dashboard/payments/page.tsx` | Payments Page mit `usePayments(undefined)` | **UMBAUEN** — Filter integrieren |
| `src/components/ui/dropdown-menu.tsx` | DropdownMenu mit CheckboxItem | **VERWENDEN** — für Status Multi-Select |
| `src/components/ui/button.tsx` | Button mit variant="ghost" | **VERWENDEN** — für Pagination |
| `src/components/ui/badge.tsx` | Badge | **VERWENDEN** — für aktive Filter |
| `src/lib/utils.ts` | `PAYMENT_STATUSES`, `cn()`, `formatAmount()`, `truncateId()`, `formatTimestamp()` | **VERWENDEN** |

### Pagination — MMS API Meta Response

Die MMS API gibt Pagination-Meta in der Response zurück:

```json
{
  "data": [...],
  "meta": { "page": 1, "limit": 20, "total": 127 }
}
```

**Total Pages berechnen:** `Math.ceil(meta.total / meta.limit)`

Die `PaymentsResponse` in `use-payments.ts` hat bereits `meta` mit `page`, `limit`, `total` — perfekt nutzbar für die Pagination-Komponente.

### Payments Page Umbau — Integration aller Teile

```typescript
// src/app/(dashboard)/dashboard/payments/page.tsx — Struktur nach Umbau:
export default function PaymentsPage() {
  const { filters, period, setFilter } = usePaymentFilters()
  const { data, isLoading } = usePayments(filters, {
    refetchInterval: (query) => {
      const payments = query.state.data?.data
      if (!payments) return false
      return hasActivePayments(payments) ? 10_000 : false
    },
  })

  return (
    <div className="flex flex-col gap-6 py-8">
      <h2 className="text-h2 font-semibold">Payments</h2>

      {/* Filter Bar — IMMER sichtbar (kein Skeleton) */}
      <PaymentFilterBar
        period={period}
        statusFilter={filters.status}
        order={filters.order}
        onPeriodChange={(p) => setFilter({ period: p })}
        onStatusChange={(s) => setFilter({ status: s?.join(",") || null })}
        onSortToggle={() => setFilter({ order: filters.order === "desc" ? "asc" : "desc" })}
      />

      {/* Desktop Table / Mobile Cards — wie bisher, aber mit Sort-Callback */}
      {/* ... */}

      {/* Pagination — unterhalb der Tabelle/Cards */}
      {data?.meta && (
        <PaymentPagination
          page={data.meta.page}
          totalPages={Math.ceil(data.meta.total / data.meta.limit)}
          onPageChange={(p) => setFilter({ page: String(p) })}
        />
      )}
    </div>
  )
}
```

### Learnings aus Story 3.3 (Previous Story Intelligence)

- **camelCase Payments:** MMS Payment-Responses sind bereits camelCase — KEINE Transformation nötig
- **Proxy Pattern:** Etabliert in `app/api/payments/route.ts` — leitet alle erlaubten Params weiter
- **Hook Pattern:** `usePayments(filters?)` mit Query Key `['payments', filters]`, staleTime 30s — Filter als Teil des Keys = automatischer Refetch bei Änderung
- **refetchInterval als Callback:** Dynamisch basierend auf Daten — Pattern beibehalten
- **PaymentsResponse.meta:** `{ page, limit, total }` — zuverlässig für Pagination
- **InlineCopyButton:** `e.stopPropagation()` verhindert Row-Click-Propagation
- **Skeleton Pattern:** Pro Komponente eigene Skeleton-Variante
- **formatTimestamp:** In `lib/utils.ts` — wiederverwenden

### Git Intelligence (letzte 5 Commits)

1. `8321838` — Add Payments List Page with DataTable, Card-Layout & review fixes (Story 3.3)
2. `86f1437` — Add Overview Page with state-dependent first screen, StatCards, HealthBar (Story 3.2)
3. `4b37e77` — sync to masem context
4. `4f2b9c3` — Add payments data layer, proxy routes, hooks & PaymentStatusBadge (Story 3.1)
5. `a783b6d` — Remove debug logging from proxy

**Relevantes Pattern:** Komponenten in `components/dashboard/`, Hooks in `hooks/`, Helpers in `lib/utils.ts`. Page-Dateien sind `"use client"` mit Hook-Integration. Neue shadcn-Komponenten via CLI installieren (Table in 3.3, Select jetzt in 3.4).

### Project Structure Notes

Neue Dateien die erstellt werden:
```
src/
  hooks/
    use-payment-filters.ts              # NEU: URL-State Hook für Filter/Sort/Pagination
  components/dashboard/
    payment-filter-bar.tsx              # NEU: Filter-Leiste (Status + Zeitraum)
    payment-pagination.tsx              # NEU: Prev/Next Pagination Controls
  components/ui/
    select.tsx                          # NEU (via shadcn CLI): Select Primitives
```

Bestehende Dateien die geändert werden:
```
src/
  app/(dashboard)/dashboard/
    payments/page.tsx                   # UMBAUEN: Filter/Sort/Pagination integrieren
  components/dashboard/
    payments-table.tsx                  # ERWEITERN: Sort-Indikator auf "Created" Spalte
```

### Architektur-Compliance

- Dateien in `kebab-case.tsx` — `payment-filter-bar.tsx`, `payment-pagination.tsx`, `use-payment-filters.ts`
- Types in `types/` (PaymentFilters bereits vorhanden — keine Änderung)
- Neue Komponenten in `components/dashboard/` (payment-filter-bar, payment-pagination)
- URL-State via Next.js native APIs (useSearchParams, useRouter) — kein Extra-Package
- TanStack Query: Filter als Teil des queryKey → automatische Refetches
- Kein `any` Type
- Kein Full-Page-Spinner — Filter-Bar bleibt stabil, nur Content-Bereich zeigt Skeleton
- Ghost-Buttons für Pagination (UX-Spec)
- Select/Dropdown für Filter (UX-Spec)
- Touch-freundlich: min. 44x44px Touch Targets auf Mobile

### HINWEIS: Page-Size = 20 (nicht API-Default 50)

Die API-Default-Limit ist 50. Für die UI wird `limit: 20` gesetzt — bessere Übersichtlichkeit, schnellere Ladezeiten. Dies wird im `usePaymentFilters` Hook als Default gesetzt.

### HINWEIS: Period vs. From/To in URL

In der URL wird `period=30d` gespeichert (nicht das berechnete `from`-Datum). Der Hook berechnet `from` aus dem Period-Preset bei jedem Render. Das bedeutet:
- URL ist stabil und lesbar: `?period=30d&status=confirmed&order=desc`
- Beim Öffnen wird das aktuelle Datum minus 30 Tage berechnet
- `to` wird nie gesetzt (immer bis jetzt)

### References

- [Source: _bmad-output/planning-artifacts/epics.md#Story 3.4]
- [Source: _bmad-output/planning-artifacts/architecture.md#API Communication Pattern]
- [Source: _bmad-output/planning-artifacts/architecture.md#Component Patterns]
- [Source: _bmad-output/planning-artifacts/architecture.md#TanStack Query Patterns]
- [Source: _bmad-output/planning-artifacts/ux-design-specification.md#Select Component]
- [Source: _bmad-output/planning-artifacts/ux-design-specification.md#Button-Hierarchie Ghost]
- [Source: _bmad-output/planning-artifacts/ux-design-specification.md#Responsive Adaptionen]
- [Source: _bmad-output/planning-artifacts/prd.md#FR16-FR19]
- [Source: _bmad-output/implementation-artifacts/3-3-payments-list-page-datatable.md]

## Dev Agent Record

### Agent Model Used

Claude Opus 4.6 (claude-opus-4-6)

### Debug Log References

- Build passed on first attempt — no TypeScript or compilation errors

### Completion Notes List

- **Task 1:** Installed shadcn Select via CLI (`src/components/ui/select.tsx`). Created `usePaymentFilters` hook with URL-state management using Next.js native `useSearchParams`/`useRouter`. Defaults: `period=30d`, `order=desc`, `page=1`, `limit=20`. Page resets to 1 on filter change.
- **Task 2:** Created `PaymentFilterBar` with Status multi-select (DropdownMenu + CheckboxItems, Badge count for active filters) and Period select (shadcn Select, 7d/30d/90d/All). Both controls have `min-h-[44px]` for touch-friendliness. Flex-wrap layout adapts to mobile.
- **Task 3:** Extended `PaymentsTable` with `order` and `onSortToggle` props. "Created" column header is clickable with ArrowUp/ArrowDown indicator. Only "Created" column is sortable — all others remain static.
- **Task 4:** Created `PaymentPagination` with Prev/Next ghost buttons, page counter display, disabled states at boundaries. Component returns null when totalPages <= 1.
- **Task 5:** Rebuilt `PaymentsPage` — integrated `usePaymentFilters` hook, `PaymentFilterBar` above content, `PaymentPagination` below content. Filter bar stays stable during loading (no skeleton). Auto-refresh with `refetchInterval: 10_000` for active payments preserved. Wrapped in Suspense for `useSearchParams` compatibility.
- **Task 6:** `npm run build` successful. All routes compile correctly.

### Change Log

- 2026-02-19: Implemented payment filters, sorting & pagination (Story 3.4)
- 2026-02-19: Code Review — 4 Medium issues fixed: Suspense fallback added (page.tsx), trailing `?` in URL fixed (use-payment-filters.ts), status filter validation added (use-payment-filters.ts), skeleton rows matched to page size (payments-table.tsx). 3 Low issues noted (L1: conditional cursor-pointer, L2: no filter-reset button, L3: type assertion cleanup). Build verified.

### File List

- src/components/ui/select.tsx (NEW — via shadcn CLI)
- src/hooks/use-payment-filters.ts (NEW)
- src/components/dashboard/payment-filter-bar.tsx (NEW)
- src/components/dashboard/payment-pagination.tsx (NEW)
- src/app/(dashboard)/dashboard/payments/page.tsx (MODIFIED — integrated filters, sort, pagination)
- src/components/dashboard/payments-table.tsx (MODIFIED — sort indicator on Created column)
