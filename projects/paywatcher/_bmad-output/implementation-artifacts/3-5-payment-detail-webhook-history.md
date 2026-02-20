# Story 3.5: Payment Detail & Webhook History

Status: done

<!-- Note: Validation is optional. Run validate-create-story for quality check before dev-story. -->

## Story

As a Tenant,
I want ein einzelnes Payment im Detail sehen können inklusive Webhook Delivery History,
So that ich den Status und die Webhook-Zustellung debuggen kann.

## Acceptance Criteria

1. **Given** ich klicke auf ein Payment in der Liste **When** die Detail-Seite lädt **Then** sehe ich alle Payment-Felder: id, status, amount, exactAmount, currency, chain, depositAddress, confirmations, confirmationsRequired, txHash, metadata, expiresAt, createdAt, updatedAt

2. **Given** ein txHash vorhanden ist **Then** ist er als Link zu Basescan klickbar (neuer Tab)

3. **Given** die Payment-Detail-Seite geladen ist **Then** sehe ich eine Webhook Delivery History via GET /v1/payments/:id/webhooks mit: Event Name, Attempt Number, Response Code (farbig: 2xx=grün, 4xx/5xx=rot), Error Message (falls vorhanden), Timestamp

4. **Given** keine Webhooks für dieses Payment existieren **Then** sehe ich: "No webhooks sent yet."

## Tasks / Subtasks

- [x] Task 1: Webhook History Proxy Route erstellen (AC: #3)
  - [x] 1.1 `src/app/api/payments/[id]/webhooks/route.ts` — GET Route Handler: Proxy zu `api.masem.at/v1/payments/:id/webhooks`, snake_case → camelCase Transformation via `snakeToCamel()`, Envelope-Format `{ data: WebhookDelivery[] }`
  - [x] 1.2 Auth-Check via `auth()` + `getApiKeyForTenant()` — gleiches Pattern wie bestehender `payments/[id]/route.ts`

- [x] Task 2: useWebhookDeliveries Hook erstellen (AC: #3, #4)
  - [x] 2.1 `src/hooks/use-webhook-deliveries.ts` — TanStack Query Hook: `useWebhookDeliveries(paymentId: string)`, Query Key `['payments', paymentId, 'webhooks']`, `staleTime: 30_000`
  - [x] 2.2 Fetch-Funktion: `GET /api/payments/${paymentId}/webhooks` → `WebhookDelivery[]`

- [x] Task 3: Payment Detail Page & Komponenten erstellen (AC: #1, #2, #3, #4)
  - [x] 3.1 `src/app/(dashboard)/dashboard/payments/[id]/page.tsx` — Detail-Seite mit `usePayment(id)` + `useWebhookDeliveries(id)`, Skeleton-Loading, Breadcrumb-Navigation zurück zu Payments List
  - [x] 3.2 `src/components/dashboard/payment-detail.tsx` — Hauptkomponente: Payment Header (PaymentStatusBadge md-Size + Payment ID + Zeitstempel), Key-Value-Liste aller Payment-Felder (id, amount, exactAmount, status, currency, chain, depositAddress, confirmations/confirmationsRequired, txHash, metadata als JSON, expiresAt, createdAt, updatedAt)
  - [x] 3.3 txHash als Basescan-Link (`https://basescan.org/tx/${txHash}`, neuer Tab, NFR14) + Copy-Button (Inline-Checkmark)
  - [x] 3.4 depositAddress als Basescan-Link (`https://basescan.org/address/${depositAddress}`, neuer Tab) + Copy-Button
  - [x] 3.5 Alle Hashes, Adressen und Amounts in Geist Mono (`font-mono`)
  - [x] 3.6 Metadata als formatiertes JSON (Geist Mono, collapsible wenn vorhanden, "No metadata" wenn null)

- [x] Task 4: Webhook History Komponente erstellen (AC: #3, #4)
  - [x] 4.1 `src/components/dashboard/webhook-history.tsx` — Webhook Delivery History Tabelle/Liste: Event Name, Attempt Number, Response Code (farbig: 2xx=success, 4xx/5xx=error, null=muted), Error Message, Timestamp
  - [x] 4.2 Empty-State: "No webhooks sent yet." (dezent, kein großes Illustration-Empty-State)
  - [x] 4.3 Skeleton-Loading für Webhook-History während Daten laden

- [x] Task 5: Mobile Card als klickbare Links erweitern
  - [x] 5.1 `src/components/dashboard/payment-card.tsx` — ERWEITERN: Jede Payment-Card in `<Link href="/dashboard/payments/${payment.id}">` wrappen, damit auch auf Mobile die Navigation zur Detail-Seite funktioniert

- [x] Task 6: Auto-Refresh für aktive Payments auf Detail-Seite (AC: #1)
  - [x] 6.1 `usePayment(id)` mit `refetchInterval: 10_000` wenn Payment-Status `pending`, `matched` oder `confirming` ist
  - [x] 6.2 `useWebhookDeliveries(id)` mit `refetchInterval: 10_000` wenn Payment aktiv ist

- [x] Task 7: Build-Validierung
  - [x] 7.1 `npm run build` fehlerfrei
  - [x] 7.2 Navigation von Payments-Liste (Desktop Tabelle + Mobile Cards) zur Detail-Seite funktioniert
  - [x] 7.3 Alle Payment-Felder korrekt dargestellt (Monospace für Hashes/Adressen/Amounts)
  - [x] 7.4 txHash → Basescan Link öffnet in neuem Tab
  - [x] 7.5 Webhook History zeigt korrekte Daten oder Empty-State
  - [x] 7.6 Skeleton-Loading auf Detail-Seite korrekt (Sidebar + Header stehen sofort)
  - [x] 7.7 Zurück-Navigation zur Payments-Liste funktioniert
  - [x] 7.8 Desktop: max-w-4xl zentriert (wie Basescan TX-Page); Mobile: einspaltig, volle Breite

## Dev Notes

### Kontext & Zweck

Dies ist **Story 3.5 von Epic 3** (Payment Monitoring Dashboard) — die **letzte Story des Epics**. Sie erstellt die Payment Detail Page, auf der ein Tenant ein einzelnes Payment mit allen Feldern, Basescan-Links und der Webhook Delivery History einsehen kann.

**Was diese Story beinhaltet:**
- Payment Detail Page (`/dashboard/payments/[id]`)
- Alle Payment-Felder als Key-Value-Liste
- txHash + depositAddress als Basescan-Links + Copy-Buttons
- Webhook Delivery History (Event, Status Code, Error, Timestamp, Attempt)
- Auto-Refresh für aktive Payments
- Mobile Card-Navigation zur Detail-Seite
- Proxy Route für Webhook History

**Was diese Story NICHT beinhaltet:**
- PaymentTimeline-Komponente (UX-Spec zeigt ein Timeline-Design, aber die MMS API liefert keine Timeline-Events — nur den aktuellen Status. Timeline ist ein P1-Enhancement, nicht in den ACs von Story 3.5)
- Expired Payment Hint (curl-Befehl zum Retry) — ebenfalls Enhancement, nicht in ACs
- Status-spezifische Detail-Views (confirmed zeigt X, failed zeigt Y) — die ACs fordern nur die Anzeige aller Felder + Webhook History

### KRITISCH: Webhook Proxy Route — snake_case → camelCase Transformation NÖTIG

Die Webhook History Response vom MMS Backend kommt in **snake_case** (anders als Payment-Responses die bereits camelCase sind). Die Transformation muss im Proxy-Layer passieren:

```typescript
// src/app/api/payments/[id]/webhooks/route.ts
import { snakeToCamel } from "@/lib/api/transform"
import type { WebhookDelivery } from "@/types/payment"

// MMS Response: { data: [{ event, attempt_number, response_code, error, created_at }] }
// Frontend Response: { data: [{ event, attemptNumber, responseCode, error, createdAt }] }
const webhooks = snakeToCamel<WebhookDelivery[]>(response.data)
return NextResponse.json({ data: webhooks })
```

**Bestätigt durch:** `src/lib/api/types.ts` enthält `MmsWebhookDelivery` (snake_case) und `src/types/payment.ts` enthält `WebhookDelivery` (camelCase). Die `snakeToCamel`-Utility in `src/lib/api/transform.ts` existiert bereits und ist generisch nutzbar.

### KRITISCH: Bestehende Proxy Route als Pattern — EXAKT kopieren

Die bestehende Route `src/app/api/payments/[id]/route.ts` ist das Referenz-Pattern:

```typescript
// Pattern aus bestehender Route:
import { auth } from "@/lib/auth"
import { mmsApi } from "@/lib/api/client"
import { getApiKeyForTenant } from "@/lib/api/tenant-keys"
import { NextResponse } from "next/server"

export async function GET(
  request: Request,
  { params }: { params: Promise<{ id: string }> }
) {
  const session = await auth()
  if (!session?.user?.tenantSlug) {
    return NextResponse.json(
      { error: { code: "UNAUTHORIZED", message: "Not authenticated" } },
      { status: 401 }
    )
  }

  const apiKey = await getApiKeyForTenant(session.user.tenantSlug)
  const { id } = await params

  const response = await mmsApi<{ data: ... }>(`/v1/payments/${id}`, {
    headers: { "x-api-key": apiKey },
  })

  return NextResponse.json({ data: response.data })
}
```

**WICHTIG:** `params` ist in Next.js 16 ein `Promise` — MUSS mit `await params` aufgelöst werden! Nicht direkt destructuren.

### KRITISCH: Basescan Links — Korrektes URL-Pattern

```
txHash Link:       https://basescan.org/tx/${txHash}
depositAddress:    https://basescan.org/address/${depositAddress}
```

- Öffnet in neuem Tab (`target="_blank"`, `rel="noopener noreferrer"`)
- txHash wird truncated angezeigt (`0x7a3f...8b2c`) aber der Link nutzt den vollen Hash
- Nur anzeigen wenn txHash nicht null ist

### KRITISCH: Layout — max-w-4xl zentriert (UX-Spec)

Die UX-Spec definiert für Payment Detail:
- **Desktop:** `max-w-4xl`, zentriert — fokussierte Einzelansicht wie Basescan TX-Page
- **Mobile:** Einspaltig, volle Breite, lange txHashes truncated mit Copy-Button

### KRITISCH: Response Code Farbcodierung (WebhookStatusRow)

| Response Code | Farbe | CSS |
|---|---|---|
| 2xx (200-299) | Grün (success) | `text-success` |
| 4xx (400-499) | Rot (error) | `text-error` |
| 5xx (500-599) | Rot (error) | `text-error` |
| null (pending/no response) | Grau (muted) | `text-muted-foreground` |

### Bestehender Code-Stand (aus Story 3.1 + 3.2 + 3.3 + 3.4)

**Bereits vorhanden — WIEDERVERWENDEN, nicht neu erstellen:**

| Datei | Was existiert | Aktion |
|-------|-------------|--------|
| `src/types/payment.ts` | `Payment`, `PaymentStatus`, `WebhookDelivery` Interfaces | **VERWENDEN** — keine Änderung nötig |
| `src/lib/api/types.ts` | `MmsWebhookDelivery` (snake_case Vertrag) | **VERWENDEN** — Referenz für Transformation |
| `src/hooks/use-payment.ts` | `usePayment(id)` mit Query Key `['payments', id]` | **VERWENDEN** — Auto-Refresh erweitern |
| `src/app/api/payments/[id]/route.ts` | GET single Payment Proxy | **VERWENDEN** — Pattern für Webhook Route |
| `src/lib/api/transform.ts` | `snakeToCamel<T>()` generische Transformation | **VERWENDEN** — für Webhook Response |
| `src/lib/api/client.ts` | `mmsApi()` Fetch-Wrapper | **VERWENDEN** |
| `src/lib/api/tenant-keys.ts` | `getApiKeyForTenant()` | **VERWENDEN** |
| `src/lib/auth.ts` | `auth()` Session-Check | **VERWENDEN** |
| `src/lib/api/client-error.ts` | `parseApiError()` | **VERWENDEN** — für Hook Error Handling |
| `src/components/dashboard/payment-status-badge.tsx` | PaymentStatusBadge (sm/md) | **VERWENDEN** — md-Size im Detail Header |
| `src/components/shared/inline-copy-button.tsx` | InlineCopyButton mit Checkmark-Feedback | **VERWENDEN** — für txHash, depositAddress, Payment ID Copy |
| `src/lib/utils.ts` | `PAYMENT_STATUSES`, `formatAmount()`, `truncateId()`, `truncateTxHash()`, `formatTimestamp()`, `cn()` | **VERWENDEN** |
| `src/components/dashboard/payments-table.tsx` | Desktop-Tabelle mit Link auf Payment ID | **VERWENDEN** — Link existiert bereits |
| `src/components/dashboard/payment-card.tsx` | Mobile Card | **ERWEITERN** — Link-Wrapper hinzufügen |

### Neue Dateien die erstellt werden:

```
src/
  app/
    api/payments/[id]/webhooks/
      route.ts                                    # NEU: Webhook History Proxy
    (dashboard)/dashboard/payments/[id]/
      page.tsx                                    # NEU: Payment Detail Page
  hooks/
    use-webhook-deliveries.ts                     # NEU: TanStack Query Hook
  components/dashboard/
    payment-detail.tsx                            # NEU: Payment Detail Hauptkomponente
    webhook-history.tsx                           # NEU: Webhook Delivery History
```

### Bestehende Dateien die geändert werden:

```
src/
  components/dashboard/
    payment-card.tsx                              # ERWEITERN: Link-Wrapper für Mobile
```

### Learnings aus Story 3.4 (Previous Story Intelligence)

- **camelCase Payments:** MMS Payment-Responses sind bereits camelCase — KEINE Transformation nötig. ABER Webhook History Responses sind snake_case — Transformation via `snakeToCamel()` NÖTIG!
- **Proxy Pattern:** Etabliert in `app/api/payments/route.ts` und `app/api/payments/[id]/route.ts` — exakt kopieren
- **Hook Pattern:** `usePayment(id)` existiert bereits mit Query Key `['payments', id]`, staleTime 30s
- **InlineCopyButton:** `e.stopPropagation()` verhindert Row-Click-Propagation — bereits gelöst
- **Skeleton Pattern:** Pro Komponente eigene Skeleton-Variante — für Detail-Page Skeleton erstellen
- **formatTimestamp:** In `lib/utils.ts` — wiederverwenden für alle Zeitangaben
- **truncateTxHash:** In `lib/utils.ts` — wiederverwenden für txHash-Anzeige
- **params als Promise:** Next.js 16 Route Handlers nutzen `await params` — MUSS beachtet werden

### Git Intelligence (letzte 5 Commits)

1. `d980a47` — Add Payment Filters, Sorting & Pagination with review fixes (Story 3.4)
2. `8321838` — Add Payments List Page with DataTable, Card-Layout & review fixes (Story 3.3)
3. `86f1437` — Add Overview Page with state-dependent first screen, StatCards, HealthBar (Story 3.2)
4. `4b37e77` — sync to masem context
5. `4f2b9c3` — Add payments data layer, proxy routes, hooks & PaymentStatusBadge (Story 3.1)

**Relevantes Pattern:** Neue Proxy-Routen in `app/api/`, neue Hooks in `hooks/`, neue Komponenten in `components/dashboard/`. Page-Dateien sind `"use client"` mit Hook-Integration.

### Project Structure Notes

- Alignment mit Projektstruktur: Alle neuen Dateien folgen dem etablierten Pattern
- `payments/[id]/page.tsx` — Standard Next.js Dynamic Route, konsistent mit Architecture Doc
- `api/payments/[id]/webhooks/route.ts` — Architecture Doc definiert diesen Pfad explizit
- Keine Konflikte mit bestehender Struktur

### Architektur-Compliance

- Dateien in `kebab-case.tsx` — `payment-detail.tsx`, `webhook-history.tsx`, `use-webhook-deliveries.ts`
- Types in `types/payment.ts` (WebhookDelivery bereits vorhanden — keine Änderung)
- Route Handler Response im Envelope-Format `{ data }` / `{ error }`
- snake_case → camelCase Transformation im Route Handler (für Webhooks)
- TanStack Query: `['payments', id, 'webhooks']` als Query Key (hierarchisch)
- Kein `any` Type
- Kein Full-Page-Spinner — Sidebar + Header stehen sofort, nur Content zeigt Skeleton
- Copy-Feedback als Inline-Checkmark — nie als Toast
- Max 1 Primary Button pro Screen (Detail-Page hat keinen Primary Button)
- Amounts und Hashes in Geist Mono (`font-mono`)
- Basescan Links in neuem Tab (`target="_blank"`, `rel="noopener noreferrer"`)

### HINWEIS: Payment Detail max-w-4xl vs. Payments List volle Breite

Die Payments-Liste (Story 3.3/3.4) nutzt volle Breite für die Tabelle. Die Payment Detail Page nutzt `max-w-4xl` zentriert — das ist KORREKT so (UX-Spec: "fokussierte Einzelansicht wie Basescan TX-Page").

### HINWEIS: Auto-Refresh auf Detail-Seite

Wie in der Payments-Liste: `refetchInterval: 10_000` wenn der Payment-Status `pending`, `matched` oder `confirming` ist. Das gilt sowohl für `usePayment(id)` als auch für `useWebhookDeliveries(id)` — neue Webhook-Deliveries können jederzeit eintreffen.

### HINWEIS: Metadata-Anzeige

`metadata` ist `Record<string, string> | null`. Wenn vorhanden: als formatiertes JSON in Geist Mono anzeigen (z.B. `JSON.stringify(metadata, null, 2)`). Wenn null: "No metadata" anzeigen. Das Metadata-Feld ist für Entwickler gedacht (sie speichern eigene Referenz-IDs wie `order_id`, `customer_id` etc.).

### References

- [Source: _bmad-output/planning-artifacts/epics.md#Story 3.5]
- [Source: _bmad-output/planning-artifacts/architecture.md#API Communication Pattern]
- [Source: _bmad-output/planning-artifacts/architecture.md#App Structure & Routing — payments/[id]/page.tsx]
- [Source: _bmad-output/planning-artifacts/architecture.md#Component Patterns]
- [Source: _bmad-output/planning-artifacts/architecture.md#TanStack Query Patterns]
- [Source: _bmad-output/planning-artifacts/architecture.md#Vollständige Projektverzeichnis-Struktur — api/payments/[id]/webhooks/route.ts]
- [Source: _bmad-output/planning-artifacts/ux-design-specification.md#PaymentTimeline]
- [Source: _bmad-output/planning-artifacts/ux-design-specification.md#KeyValueList]
- [Source: _bmad-output/planning-artifacts/ux-design-specification.md#WebhookStatusRow]
- [Source: _bmad-output/planning-artifacts/ux-design-specification.md#Layout Regeln — Payment Detail max-w-4xl]
- [Source: _bmad-output/planning-artifacts/ux-design-specification.md#Loading-Strategie Skeleton]
- [Source: _bmad-output/planning-artifacts/prd.md#FR15-FR19, FR38]
- [Source: _bmad-output/implementation-artifacts/3-4-payment-filters-sorting-pagination.md]
- [Source: _bmad-output/implementation-artifacts/3-3-payments-list-page-datatable.md]
- [Source: src/types/payment.ts — WebhookDelivery Interface]
- [Source: src/lib/api/types.ts — MmsWebhookDelivery (snake_case)]
- [Source: src/lib/api/transform.ts — snakeToCamel()]
- [Source: src/hooks/use-payment.ts — usePayment()]
- [Source: src/app/api/payments/[id]/route.ts — Proxy Pattern Referenz]

## Dev Agent Record

### Agent Model Used

Claude Opus 4.6

### Debug Log References

- Build erfolgreich: `npm run build` kompiliert fehlerfrei mit allen neuen Routen und Seiten
- Hooks erweitert: `usePayment` und `useWebhookDeliveries` akzeptieren jetzt `refetchInterval` Option für Auto-Refresh

### Completion Notes List

- Task 1: Webhook History Proxy Route erstellt — exakt das Pattern der bestehenden `payments/[id]/route.ts` kopiert, `snakeToCamel()` für snake_case → camelCase Transformation, Auth via `requireAuth()`, Envelope-Format `{ data: WebhookDelivery[] }`
- Task 2: `useWebhookDeliveries` Hook erstellt — TanStack Query mit Query Key `['payments', paymentId, 'webhooks']`, staleTime 30s, optionales `refetchInterval`
- Task 3: Payment Detail Page + Komponenten erstellt — alle Payment-Felder als Key-Value-Liste, PaymentStatusBadge (md), txHash + depositAddress als Basescan-Links mit Copy-Buttons, Metadata collapsible als JSON, Skeleton-Loading, Breadcrumb zurück zur Liste
- Task 4: Webhook History Komponente erstellt — Desktop-Tabelle + Mobile Cards, Response Code farbcodiert (2xx=success grün, 4xx/5xx=error rot, null=muted grau), Empty-State "No webhooks sent yet.", Skeleton-Loading
- Task 5: Payment Card um Link-Wrapper erweitert — Mobile Cards navigieren jetzt zur Detail-Seite
- Task 6: Auto-Refresh implementiert — `refetchInterval: 10_000` für aktive Payments (pending/matched/confirming) auf beiden Hooks
- Task 7: Build-Validierung bestanden — `npm run build` fehlerfrei, alle Routen korrekt registriert

### Change Log

- 2026-02-19: Story 3.5 implementiert — Payment Detail Page mit Webhook History, Auto-Refresh, Mobile Card Navigation, Basescan-Links
- 2026-02-19: Code Review Fixes — H1: Webhook Error-Handling auf Detail-Seite, M1: Webhook Skeleton Desktop-Tabelle, M2: Back-Link dedupliziert, M3: Doppelter Import zusammengeführt, M4: Redundante Payment ID entfernt, M5: "Payment not found" State, L1: staleTime-Notation vereinheitlicht

### File List

**Neue Dateien:**
- `src/app/api/payments/[id]/webhooks/route.ts` — Webhook History Proxy Route
- `src/hooks/use-webhook-deliveries.ts` — TanStack Query Hook für Webhook Deliveries
- `src/app/(dashboard)/dashboard/payments/[id]/page.tsx` — Payment Detail Page
- `src/components/dashboard/payment-detail.tsx` — Payment Detail Hauptkomponente + Skeleton
- `src/components/dashboard/webhook-history.tsx` — Webhook History Komponente + Skeleton

**Geänderte Dateien:**
- `src/hooks/use-payment.ts` — Erweitert um optionales `refetchInterval` für Auto-Refresh
- `src/components/dashboard/payment-card.tsx` — `<div>` → `<Link>` Wrapper für Mobile Navigation
