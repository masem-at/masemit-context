# Story 3.3: Payments List Page & DataTable

Status: done

<!-- Note: Validation is optional. Run validate-create-story for quality check before dev-story. -->

## Story

As a Tenant,
I want eine vollständige Liste meiner Payments in einer Tabelle sehen können,
So that ich alle Payments im Überblick habe und schnell den Status jeder Zahlung erfasse.

## Acceptance Criteria

1. **Given** ich klicke auf "Payments" in der Sidebar (FR15) **When** die Payments-Seite lädt **Then** sehe ich eine Tabelle mit den Spalten: Payment ID (monospace, truncated), Amount (monospace, USDC), Status (PaymentStatusBadge), txHash (monospace, truncated, Copy-Button), Created (Timestamp) **And** die Tabelle nutzt volle Breite (minus Sidebar) — kein max-width

2. **Given** die Payments-Tabelle auf Desktop (>=1024px) gerendert wird **When** ein Payment in der Tabelle angezeigt wird **Then** sind Payment ID und txHash als monospace (Geist Mono) dargestellt und truncated (z.B. `pay_7f2a...` / `0x7a3f...8b2c`) **And** jeder txHash hat einen Copy-Button (Inline-Checkmark, kein Toast) **And** txHash ist als Link zu Basescan klickbar (öffnet in neuem Tab, NFR14)

3. **Given** die Payments-Tabelle auf Mobile (<1024px) gerendert wird **When** die Seite rendert **Then** wechselt die Darstellung zu einem Card-Layout: Pro Payment eine Card mit Status-Badge + Amount + Payment ID + Timestamp **And** kein horizontaler Scroll der Tabelle (Cards statt Tabelle)

4. **Given** die Payments-Daten laden **When** die API-Response noch aussteht **Then** sehe ich Skeleton-Zeilen (Desktop) bzw. Skeleton-Cards (Mobile) als Loading-State

5. **Given** keine Payments im gewählten Zeitraum existieren **When** die leere Liste angezeigt wird **Then** sehe ich einen dezenten Hinweis: "No payments in this period." (kein großes Illustration-Empty-State)

6. **Given** Payments mit Status `pending`, `matched` oder `confirming` in der Liste sind **When** die Seite aktiv ist **Then** werden die Daten via TanStack Query alle 10 Sekunden automatisch refreshed (`refetchInterval: 10_000`) **And** der Refresh blockiert nicht die UI und zeigt keinen Loading-State

## Tasks / Subtasks

- [x] Task 1: shadcn Table-Komponente installieren & Shared Helpers extrahieren (AC: #1, #2)
  - [x] 1.1 `npx shadcn@latest add table` — shadcn Table-Primitives installieren
  - [x] 1.2 `src/lib/utils.ts` — `truncateId()` und `formatTimestamp()` aus `overview-content.tsx` nach `lib/utils.ts` extrahieren (DRY)
  - [x] 1.3 `src/components/dashboard/overview-content.tsx` — Importe auf extrahierte Helpers umstellen
  - [x] 1.4 Inline-CopyButton: Kleine `InlineCopyButton` Komponente erstellen (`src/components/shared/inline-copy-button.tsx`) — kein absolute Positioning, inline-flex, für Tabellenzellen geeignet (die bestehende `CopyButton` ist für Code-Blocks mit absolute Positioning)

- [x] Task 2: Payments-Seite erstellen — Desktop DataTable (AC: #1, #2)
  - [x] 2.1 `src/app/(dashboard)/dashboard/payments/page.tsx` — Page-Komponente mit Titel "Payments", `usePayments()` Hook, Zustandslogik (Loading/Empty/Data)
  - [x] 2.2 `src/components/dashboard/payments-table.tsx` — Desktop-Tabelle mit shadcn Table: Spalten Payment ID (truncated, monospace), Amount (monospace, USDC), Status (PaymentStatusBadge sm), txHash (truncated, monospace, InlineCopyButton + Basescan-Link), Created (formatTimestamp)
  - [x] 2.3 txHash als externen Link zu Basescan: `https://basescan.org/tx/${txHash}` (neuer Tab, `rel="noopener noreferrer"`)
  - [x] 2.4 Tabelle hat volle Breite (kein max-width) — `w-full` auf Table-Container

- [x] Task 3: Mobile Card-Layout (AC: #3)
  - [x] 3.1 `src/components/dashboard/payment-card.tsx` — Mobile Card-Komponente: Status-Badge + Timestamp oben, Payment ID + Amount unten (gemäß UX-Spec Card-Layout)
  - [x] 3.2 `src/app/(dashboard)/dashboard/payments/page.tsx` — Responsive Switch: `lg:hidden` für Cards, `hidden lg:block` für Tabelle

- [x] Task 4: Skeleton Loading States (AC: #4)
  - [x] 4.1 `src/components/dashboard/payments-table.tsx` — Desktop Skeleton: 10 Zeilen mit Skeleton-Shapes pro Spalte
  - [x] 4.2 `src/components/dashboard/payment-card.tsx` — Mobile Skeleton: 5 Card-Shapes mit Skeleton-Inhalt

- [x] Task 5: Empty State & Auto-Refresh (AC: #5, #6)
  - [x] 5.1 Empty State: Dezenter Hinweis "No payments in this period." (text-sm, text-muted-foreground, kein Illustration)
  - [x] 5.2 Auto-Refresh: `refetchInterval: 10_000` wenn Payments mit Status `pending`, `matched` oder `confirming` in der Response vorhanden sind (dynamisch aktivieren/deaktivieren)
  - [x] 5.3 Background Refresh blockiert NICHT die UI — kein Loading-Indicator beim Refresh

- [x] Task 6: Build-Validierung
  - [x] 6.1 `npm run build` fehlerfrei
  - [x] 6.2 Bestehende Seiten (Landing, Dashboard Overview, Settings) funktionieren weiterhin
  - [x] 6.3 Sidebar "Payments" Link führt zur neuen Payments-Seite
  - [x] 6.4 Desktop-Tabelle und Mobile-Cards rendern korrekt
  - [x] 6.5 "View all payments →" Link in Overview führt zur Payments-Seite

## Dev Notes

### Kontext & Zweck

Dies ist **Story 3.3 von Epic 3** (Payment Monitoring Dashboard). Sie baut die **Payments List Page** — die zentrale Datentabelle des Dashboards. Alle Payment-Daten werden in einer professionellen DataTable (Desktop) bzw. als Card-Liste (Mobile) dargestellt.

**Was diese Story beinhaltet:**
- Neue Payments-Seite unter `/dashboard/payments`
- Desktop DataTable (shadcn Table) mit allen Spalten
- Mobile Card-Layout als responsive Alternative
- Skeleton Loading States
- Auto-Refresh bei aktiven Payments
- Shared Helper-Extraktion (truncateId, formatTimestamp)
- InlineCopyButton für Tabellenzellen

**Was diese Story NICHT beinhaltet:**
- Filter, Sorting, Pagination (Story 3.4)
- Payment Detail Page (Story 3.5)
- Webhook History (Story 3.5)

### KRITISCH: shadcn Table muss installiert werden

Die shadcn Table-Komponente existiert noch NICHT im Projekt. Sie muss via CLI installiert werden:

```bash
npx shadcn@latest add table
```

Dies erstellt `src/components/ui/table.tsx` mit den Primitives: `Table`, `TableBody`, `TableCell`, `TableHead`, `TableHeader`, `TableRow`.

### KRITISCH: Shared Helpers extrahieren — truncateId & formatTimestamp

In `src/components/dashboard/overview-content.tsx` existieren bereits `truncateId()` und `formatTimestamp()`. Diese MÜSSEN nach `src/lib/utils.ts` extrahiert werden, da sie auch in der Payments-Tabelle benötigt werden. **NICHT** duplizieren!

```typescript
// src/lib/utils.ts — NEU hinzufügen:
export function truncateId(id: string): string {
  if (id.length <= 12) return id
  return `${id.slice(0, 8)}...`
}

export function formatTimestamp(iso: string): string {
  return new Intl.DateTimeFormat("en-US", {
    month: "short",
    day: "numeric",
    hour: "2-digit",
    minute: "2-digit",
  }).format(new Date(iso))
}
```

Danach `overview-content.tsx` auf die importierten Funktionen umstellen.

### KRITISCH: Bestehende CopyButton ist NICHT geeignet für Tabellenzellen

Die bestehende `CopyButton` in `src/components/shared/copy-button.tsx` nutzt `absolute right-2 top-2` Positioning — das ist für Code-Blocks gedacht. Für Tabellenzellen brauchen wir eine **InlineCopyButton** die `inline-flex` ist und ohne absolute Positioning funktioniert.

```typescript
// src/components/shared/inline-copy-button.tsx
"use client"
import { useState } from "react"
import { Copy, Check } from "lucide-react"
import { cn } from "@/lib/utils"

interface InlineCopyButtonProps {
  text: string
  className?: string
}

export function InlineCopyButton({ text, className }: InlineCopyButtonProps) {
  const [copied, setCopied] = useState(false)
  return (
    <button
      onClick={async (e) => {
        e.preventDefault() // Prevent row click navigation
        e.stopPropagation()
        await navigator.clipboard.writeText(text)
        setCopied(true)
        setTimeout(() => setCopied(false), 1500)
      }}
      className={cn("inline-flex shrink-0 rounded p-1 hover:bg-muted transition-colors", className)}
      aria-label="Copy to clipboard"
    >
      {copied ? <Check className="size-3.5 text-success" /> : <Copy className="size-3.5 text-muted-foreground" />}
    </button>
  )
}
```

**Wichtig:** `e.preventDefault()` und `e.stopPropagation()` verhindern dass der Copy-Klick die Zeilen-Navigation triggert (falls Zeilen klickbar gemacht werden).

### Bestehender Code-Stand (aus Story 3.1 + 3.2)

**Bereits vorhanden — WIEDERVERWENDEN, nicht neu erstellen:**

| Datei | Was existiert |
|-------|-------------|
| `src/hooks/use-payments.ts` | `usePayments(filters?, options?)` — **VERWENDEN** mit refetchInterval Option |
| `src/types/payment.ts` | Payment, PaymentStatus, PaymentFilters, WebhookDelivery — **VERWENDEN** |
| `src/components/dashboard/payment-status-badge.tsx` | PaymentStatusBadge (sm/md) — **VERWENDEN** mit `size="sm"` |
| `src/lib/utils.ts` | `formatAmount()`, `PAYMENT_STATUSES`, `cn()` — **VERWENDEN** |
| `src/lib/api/client-error.ts` | `parseApiError()` — wird intern von usePayments verwendet |
| `src/app/api/payments/route.ts` | Proxy GET → MMS `/v1/payments` — **EXISTIERT**, keine Änderung nötig |
| `src/components/ui/skeleton.tsx` | shadcn Skeleton — **VERWENDEN** für Loading-States |
| `src/components/ui/card.tsx` | shadcn Card — **VERWENDEN** für Mobile Cards |
| `src/components/dashboard/overview-content.tsx` | `truncateId()`, `formatTimestamp()` — **EXTRAHIEREN** nach `lib/utils.ts` |
| `src/components/shared/copy-button.tsx` | CopyButton (für Code-Blocks) — NICHT verwenden, neue InlineCopyButton erstellen |

### usePayments Hook — bereits erweiterbar

Der Hook aus Story 3.1 unterstützt bereits `refetchInterval` und `enabled` als Options (erweitert in Story 3.2):

```typescript
export function usePayments(filters?: PaymentFilters, options?: UsePaymentsOptions) {
  return useQuery({
    queryKey: ["payments", filters] as const,
    queryFn: () => fetchPayments(filters),
    staleTime: 30 * 1000,
    refetchInterval: options?.refetchInterval,
    enabled: options?.enabled,
  })
}
```

**Für Auto-Refresh:** `refetchInterval: 10_000` übergeben wenn aktive Payments vorhanden sind. Die Bestimmung ob aktive Payments vorhanden sind, erfolgt client-seitig nach der Response:

```typescript
const hasActivePayments = payments?.data?.some(p =>
  ["pending", "matched", "confirming"].includes(p.status)
)
// Dann: usePayments(filters, { refetchInterval: hasActivePayments ? 10_000 : false })
```

**ACHTUNG:** Da `refetchInterval` sich basierend auf den Daten ändert, muss dies als State oder Memo implementiert werden, NICHT als bedingter Hook-Aufruf (Rules of Hooks).

### Tabellen-Spalten (Desktop)

| Spalte | Breite | Styling | Inhalt |
|--------|--------|---------|--------|
| Payment ID | ~120px | `font-mono text-sm text-muted-foreground` | truncateId(id) z.B. `pay_7f2a...` |
| Amount | ~160px | `font-mono text-sm` | formatAmount(amount, currency) z.B. `49.000042 USDC` |
| Status | ~100px | PaymentStatusBadge sm | Farbiger Badge |
| txHash | ~200px | `font-mono text-sm text-muted-foreground` | Truncated + Copy + Basescan-Link (oder "—" wenn null) |
| Created | ~140px | `text-sm text-muted-foreground` | formatTimestamp(createdAt) |

**Volle Breite:** Die Tabelle nutzt `w-full` — kein max-width Container. Das ist wichtig weil txHash (42 Zeichen) + Amount + Status + Timestamps Platz brauchen (UX-Spec: "Dashboard Payments (Tabelle) → Volle Breite minus Sidebar").

### Mobile Card-Layout (UX-Spec)

Gemäß UX-Spezifikation — kein horizontaler Scroll, Card statt Tabelle:

```
┌─────────────────────────────────┐
│ ● confirmed        14:01 UTC   │
│ #pay_7f2a    49.000042 USDC    │
├─────────────────────────────────┤
│ ◌ pending           14:32 UTC  │
│ #pay_3b1c    120.000187 USDC   │
└─────────────────────────────────┘
```

- Status-Badge + Timestamp oben (flex, justify-between)
- Payment ID (truncated, monospace) + Amount (monospace) unten
- Divider zwischen Cards
- Touch Targets min. 44x44px

### Basescan txHash Links

```typescript
const BASESCAN_TX_URL = "https://basescan.org/tx"

function basescanLink(txHash: string): string {
  return `${BASESCAN_TX_URL}/${txHash}`
}
```

- txHash als `<a href={basescanLink(txHash)} target="_blank" rel="noopener noreferrer">` — öffnet in neuem Tab (NFR14)
- Wenn `txHash === null` (pending/expired Payments): "—" anzeigen, kein Link
- Link-Styling: `text-primary hover:underline` (dezent, nicht überladen)

### Routing & Navigation

- **Neue Page:** `src/app/(dashboard)/dashboard/payments/page.tsx`
- **Sidebar-Link** existiert bereits: `/dashboard/payments` (nav-config.ts)
- **Overview "View all payments →"** Link existiert bereits: `/dashboard/payments` (overview-content.tsx)
- **Zeilen-Klick:** Vorerst NICHT implementieren (Payment Detail = Story 3.5). Aber Link in Payment ID vorbereiten: `<Link href={/dashboard/payments/${id}}>` für die Payment ID Spalte.

### Auto-Refresh Logik

**Dynamisches refetchInterval:**

```typescript
// In der Payments Page:
const [refetchEnabled, setRefetchEnabled] = useState(false)

const { data, isLoading } = usePayments(
  filters,
  { refetchInterval: refetchEnabled ? 10_000 : false }
)

// Nach Datenerhalt prüfen:
useEffect(() => {
  if (data?.data) {
    const hasActive = data.data.some(p =>
      ["pending", "matched", "confirming"].includes(p.status)
    )
    setRefetchEnabled(hasActive)
  }
}, [data])
```

**Wichtig:** Der Background-Refresh darf KEINEN Loading-State triggern. TanStack Query macht das automatisch richtig — `isLoading` ist nur `true` beim initialen Load, nicht bei refetches. `isFetching` kann für einen subtilen Indicator genutzt werden (optional, nicht required per AC).

### Learnings aus Story 3.2 (Previous Story Intelligence)

- **camelCase Payments:** MMS Payment-Responses sind bereits camelCase — KEINE Transformation nötig
- **Proxy Pattern:** Etabliert in `app/api/payments/route.ts` — requireAuth(), mmsApi(), Envelope Format
- **Hook Pattern:** `usePayments(filters?)` mit Query Key `['payments', filters]`, staleTime 30s
- **PaymentStatusBadge:** Funktioniert, nutzt STATUS_CONFIG/PAYMENT_STATUSES aus utils
- **formatAmount:** In `lib/utils.ts`, Eingabe String + Currency, keine Rundung
- **StatCards:** Nutzen `meta.total` — die Payments API Pagination-Meta ist zuverlässig
- **Zustandslogik:** Payment-Daten als primäre Quelle (nicht localStorage)
- **sevenDaysAgo Memo:** Query-Key-Stabilität ist wichtig — memoize Datum-Werte!
- **refetchInterval + enabled:** Bereits als Options im usePayments Hook implementiert (Story 3.2)

### Git Intelligence (letzte 5 Commits)

1. `86f1437` — Add Overview Page with state-dependent first screen, StatCards, HealthBar (Story 3.2)
2. `4b37e77` — sync to masem context
3. `4f2b9c3` — Add payments data layer, proxy routes, hooks & PaymentStatusBadge (Story 3.1)
4. `a783b6d` — Remove debug logging from proxy
5. `bdaeddb` — Fix session: move MMS tenant resolution from signIn to jwt callback

**Relevantes Pattern:** Story 3.2 Commit zeigt dass Komponenten in `components/dashboard/` liegen, Helpers in `lib/utils.ts`, Hooks in `hooks/`. Page-Dateien sind "use client" mit Hook-Integration.

### Architektur-Compliance

- Dateien in `kebab-case.tsx` — `payments-table.tsx`, `payment-card.tsx`, `inline-copy-button.tsx`
- Types in `types/` (nicht inline) — Payment, PaymentFilters bereits vorhanden
- Neue Komponenten in `components/dashboard/` (payments-table.tsx, payment-card.tsx)
- Shared InlineCopyButton in `components/shared/`
- TanStack Query im Browser ruft `/api/payments` auf — nie direkt MMS
- Query Keys: Array-basiert, hierarchisch: `['payments', filters]`
- Kein `any` Type
- Kein Full-Page-Spinner — Skeleton Loading für alle Dashboard-Inhalte
- Amounts immer in `font-mono` (Geist Mono)
- Copy-Feedback: Inline-Checkmark, kein Toast
- Tabelle: Volle Breite (kein max-width)
- txHash: Monospace, truncated, Basescan-Link (neuer Tab), Copy-Button
- Payment ID: Monospace, truncated
- Status: PaymentStatusBadge mit `size="sm"`

### HINWEIS: Pagination/Filter kommen in Story 3.4

Diese Story lädt Payments OHNE Filter/Pagination UI — die Defaults der API (`page=1, limit=50, order=desc`) werden verwendet. Filter-Bar, Zeitraum-Filter, Sort-Buttons und Pagination-Controls kommen in Story 3.4. Aber die `usePayments` Hook-Signatur unterstützt bereits alle Filter-Parameter.

### HINWEIS: Payment-Zeilen-Klick kommt in Story 3.5

Zeilen in der Tabelle sind vorerst NICHT klickbar (keine onClick/Link auf der Zeile). Payment ID kann als Link zu `/dashboard/payments/${id}` vorbereitet werden, die Detail-Seite existiert aber erst mit Story 3.5.

### Project Structure Notes

Neue Dateien die erstellt werden:
```
src/
  app/(dashboard)/dashboard/
    payments/
      page.tsx                          # NEU: Payments List Page
  components/dashboard/
    payments-table.tsx                  # NEU: Desktop DataTable
    payment-card.tsx                    # NEU: Mobile Card-Komponente
  components/shared/
    inline-copy-button.tsx              # NEU: Inline Copy-Button für Tabellenzellen
  components/ui/
    table.tsx                           # NEU (via shadcn CLI): Table Primitives
```

Bestehende Dateien die geändert werden:
```
src/
  lib/utils.ts                         # ERWEITERN: truncateId(), formatTimestamp() hinzufügen
  components/dashboard/overview-content.tsx  # ÄNDERN: Importe auf lib/utils.ts umstellen
```

### References

- [Source: _bmad-output/planning-artifacts/epics.md#Story 3.3]
- [Source: _bmad-output/planning-artifacts/architecture.md#API Communication Pattern]
- [Source: _bmad-output/planning-artifacts/architecture.md#Component Patterns]
- [Source: _bmad-output/planning-artifacts/architecture.md#TanStack Query Patterns]
- [Source: _bmad-output/planning-artifacts/ux-design-specification.md#Payments-Tabelle Responsive]
- [Source: _bmad-output/planning-artifacts/ux-design-specification.md#Loading States]
- [Source: _bmad-output/planning-artifacts/ux-design-specification.md#Component Strategy]
- [Source: _bmad-output/planning-artifacts/ux-design-specification.md#Content Width]
- [Source: _bmad-output/planning-artifacts/prd.md#FR15]
- [Source: _bmad-output/implementation-artifacts/3-1-payments-data-layer-paymentstatusbadge.md]
- [Source: _bmad-output/implementation-artifacts/3-2-overview-page-zustandsabhaengiger-first-screen.md]

## Dev Agent Record

### Agent Model Used

Claude Opus 4.6

### Debug Log References

Keine Debug-Probleme aufgetreten. Build war beim ersten Versuch fehlerfrei.

### Completion Notes List

- Task 1: shadcn Table via CLI installiert, `truncateId()` und `formatTimestamp()` aus `overview-content.tsx` nach `lib/utils.ts` extrahiert (DRY), Importe in overview-content.tsx aktualisiert, neue `InlineCopyButton` Komponente für Tabellenzellen erstellt (inline-flex, kein absolute Positioning)
- Task 2: Payments Page (`/dashboard/payments`) mit Desktop DataTable implementiert — alle 5 Spalten (Payment ID monospace/truncated, Amount monospace, Status via PaymentStatusBadge sm, txHash monospace/truncated mit Basescan-Link + InlineCopyButton, Created via formatTimestamp), volle Breite (w-full), Payment ID als Link vorbereitet für Story 3.5
- Task 3: Mobile Card-Layout mit PaymentCardList — Status-Badge + Timestamp oben, Payment ID + Amount unten, responsive Switch via `lg:hidden` / `hidden lg:block`
- Task 4: Skeleton Loading States — PaymentsTableSkeleton (10 Desktop-Zeilen) und PaymentCardListSkeleton (5 Mobile-Cards)
- Task 5: Empty State "No payments in this period." (dezent, text-sm), Auto-Refresh mit dynamischem `refetchInterval: 10_000` bei aktiven Payments (pending/matched/confirming), Background-Refresh blockiert nicht die UI (nutzt TanStack Query isLoading vs isFetching)
- Task 6: `npm run build` fehlerfrei, alle bestehenden Routes intakt, Sidebar-Link und Overview "View all payments" Link korrekt

### File List

- `src/components/ui/table.tsx` — NEU (via shadcn CLI): Table Primitives
- `src/lib/utils.ts` — GEÄNDERT: truncateId(), formatTimestamp(), truncateTxHash() hinzugefügt
- `src/components/dashboard/overview-content.tsx` — GEÄNDERT: Importe auf lib/utils.ts umgestellt
- `src/components/shared/inline-copy-button.tsx` — NEU: Inline Copy-Button für Tabellenzellen (mit Clipboard Error-Handling)
- `src/components/dashboard/payments-table.tsx` — NEU: Desktop DataTable + Skeleton (nutzt truncateTxHash für korrekte txHash-Darstellung)
- `src/components/dashboard/payment-card.tsx` — NEU: Mobile Card-Layout + Skeleton (min-h-[44px] Touch Targets)
- `src/app/(dashboard)/dashboard/payments/page.tsx` — GEÄNDERT: Vollständige Payments Page (war Placeholder), EmptyState-Komponente, refetchInterval Callback
- `src/hooks/use-payments.ts` — GEÄNDERT: refetchInterval als Callback-Funktion unterstützt, PaymentsResponse exportiert

## Change Log

- 2026-02-19: Story 3.3 implementiert — Payments List Page mit Desktop DataTable (shadcn Table), Mobile Card-Layout, Skeleton Loading States, Empty State, Auto-Refresh bei aktiven Payments, Shared Helper-Extraktion (truncateId, formatTimestamp → lib/utils.ts), InlineCopyButton für txHash Copy
- 2026-02-19: Code Review (Adversarial) — 5 Fixes applied: [H1] truncateTxHash() für korrekte txHash-Darstellung (0x7a3f...8b2c), [H2] try/catch um navigator.clipboard.writeText, [M1] min-h-[44px] Touch Targets auf Mobile Cards, [M2] EmptyState-Komponente statt dupliziertem Code, [M3] refetchInterval als TanStack Query Callback statt useState+useEffect
