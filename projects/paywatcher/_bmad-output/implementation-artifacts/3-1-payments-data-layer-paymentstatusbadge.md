# Story 3.1: Payments Data Layer & PaymentStatusBadge

Status: done

## Story

As a Developer (Sempre),
I want Payment-Typen, API-Proxy-Routen, TanStack Query Hooks und die PaymentStatusBadge-Komponente,
So that alle nachfolgenden Payment-Features eine stabile Datengrundlage und die zentrale Status-Darstellung haben.

## Acceptance Criteria

1. **Given** die Dashboard Shell aus Epic 2 existiert **When** die Payment-Typen definiert werden **Then** existiert `types/payment.ts` mit: `PaymentStatus` (Union Type), `Payment` Interface (id, amount, exactAmount, status, txHash, depositAddress, confirmations, confirmationsRequired, currency, chain, metadata, expiresAt, createdAt, updatedAt), `PaymentFilters` Interface **And** `PAYMENT_STATUSES` Konstante mit Farb- und Label-Mapping gemaess UX-Spec

2. **Given** der API Proxy Pattern aus Story 2.1 existiert (mmsApi, API Key aus Session) **When** die Payments-Proxy-Routen erstellt werden **Then** existiert `app/api/payments/route.ts` (GET -> `api.masem.at/v1/payments` mit Query-Params Weiterleitung) **And** existiert `app/api/payments/[id]/route.ts` (GET -> `api.masem.at/v1/payments/:id`) **And** das Envelope-Format `{ data }` / `{ data, meta }` wird eingehalten **And** Payment-Responses sind bereits camelCase vom MMS Backend -- keine Transformation noetig

3. **Given** die Proxy-Routen existieren **When** TanStack Query Hooks erstellt werden **Then** existiert `usePayments(filters?: PaymentFilters)` mit Query Key `['payments', filters]` **And** existiert `usePayment(id: string)` mit Query Key `['payments', id]` **And** `staleTime` ist 30 Sekunden fuer Payments

4. **Given** das STATUS_CONFIG Mapping definiert ist **When** eine PaymentStatusBadge gerendert wird **Then** zeigt sie den korrekten farbigen Badge: pending=muted, matched=accent, confirming=warning, confirmed=success, expired=neutral, failed=error **And** der Badge zeigt immer Farbe + Text-Label zusammen (nie nur Farbe -- Accessibility) **And** der Badge hat zwei Groessen: `sm` (Tabelle) und `md` (Detail Header)

5. **Given** ein Amount-Wert aus der API kommt (String, 6 Dezimalen) **When** der Betrag formatiert werden muss **Then** existiert ein `formatAmount()` Helper der "49.000042 USDC" korrekt formatiert **And** Amounts werden in Geist Mono dargestellt

## Tasks / Subtasks

- [x] Task 1: Payment Types erweitern (AC: #1)
  - [x] 1.1 `src/types/payment.ts` -- `Payment` Interface hinzufuegen (id, amount, exactAmount, status, txHash, depositAddress, confirmations, confirmationsRequired, currency, chain, metadata, expiresAt, createdAt, updatedAt)
  - [x] 1.2 `src/types/payment.ts` -- `PaymentFilters` Interface (status?, from?, to?, page?, limit?, sort?, order?)
  - [x] 1.3 `src/types/payment.ts` -- `WebhookDelivery` Interface (fuer Story 3.5 vorbereiten: event, attemptNumber, responseCode, error, createdAt)
  - [x] 1.4 `src/lib/api/types.ts` -- `MmsPayment` Interface (snake_case: Felder wie MMS sie liefert)
  - [x] 1.5 `src/lib/api/types.ts` -- `MmsWebhookDelivery` Interface (snake_case)

- [x] Task 2: Payments Proxy Routes (AC: #2)
  - [x] 2.1 `src/app/api/payments/route.ts` -- GET Handler: requireAuth(), Query-Params durchleiten (status, from, to, page, limit, sort, order), mmsApi GET /v1/payments, Envelope Format { data, meta }
  - [x] 2.2 `src/app/api/payments/[id]/route.ts` -- GET Handler: requireAuth(), mmsApi GET /v1/payments/:id, Envelope Format { data }

- [x] Task 3: TanStack Query Hooks (AC: #3)
  - [x] 3.1 `src/hooks/use-payments.ts` -- usePayments(filters?) mit Query Key ['payments', filters], staleTime 30s, fetchApi pattern
  - [x] 3.2 `src/hooks/use-payment.ts` -- usePayment(id) mit Query Key ['payments', id], staleTime 30s, enabled: !!id

- [x] Task 4: PaymentStatusBadge Komponente (AC: #4)
  - [x] 4.1 `src/components/dashboard/payment-status-badge.tsx` -- PaymentStatusBadge mit size prop (sm | md), nutzt STATUS_CONFIG aus lib/utils.ts, CSS Custom Properties fuer Farben

- [x] Task 5: formatAmount Helper (AC: #5)
  - [x] 5.1 `src/lib/utils.ts` -- formatAmount(amount: string, currency?: string) Funktion hinzufuegen

- [x] Task 6: Build-Validierung
  - [x] 6.1 `npm run build` fehlerfrei
  - [x] 6.2 Bestehende Seiten (Landing, Dashboard, Settings) funktionieren weiterhin

## Dev Notes

### Kontext & Zweck

Dies ist die **erste Story von Epic 3** (Payment Monitoring Dashboard). Sie legt die Datengrundlage fuer alle nachfolgenden Payment-Stories (3.2-3.5). Es werden **keine UI-Seiten** gebaut -- nur Types, Proxy Routes, Hooks und die Badge-Komponente.

**Was diese Story beinhaltet:**
- Payment TypeScript Types (Frontend camelCase + MMS snake_case)
- API Proxy Routes (GET /api/payments, GET /api/payments/[id])
- TanStack Query Hooks (usePayments, usePayment)
- PaymentStatusBadge Komponente
- formatAmount Helper

**Was diese Story NICHT beinhaltet:**
- Payments List Page (Story 3.3)
- Overview Page (Story 3.2)
- Filter/Sorting/Pagination UI (Story 3.4)
- Payment Detail Page (Story 3.5)

### KRITISCH: Payment-Responses sind bereits camelCase

Das MMS Backend liefert Payment-Responses **bereits in camelCase**. KEINE snakeToCamel-Transformation im Proxy noetig fuer Payment-Daten. Dies ist anders als bei Config-Endpoints (die snake_case liefern).

Die Proxy Routes muessen die Payment-Daten **direkt durchreichen** im Envelope-Format.

### Bestehender Code-Stand (aus Epic 2)

**Bereits vorhanden -- WIEDERVERWENDEN, nicht neu erstellen:**

| Datei | Was existiert |
|-------|-------------|
| `src/types/payment.ts` | PaymentStatus Union Type, StatusColor, StatusConfig -- **erweitern** um Payment Interface |
| `src/lib/utils.ts` | STATUS_CONFIG Record, cn() -- **erweitern** um formatAmount() |
| `src/lib/api/client.ts` | mmsApi() -- **verwenden** in Proxy Routes |
| `src/lib/api/transform.ts` | snakeToCamel() -- **NICHT verwenden** fuer Payments (bereits camelCase) |
| `src/lib/api/types.ts` | MmsApiResponse, MmsListResponse, MmsErrorResponse -- **erweitern** um MmsPayment |
| `src/lib/api/errors.ts` | MmsApiError -- **verwenden** in Proxy Routes |
| `src/lib/api/client-error.ts` | parseApiError() -- **verwenden** in Hooks |
| `src/lib/auth.ts` | requireAuth() -- **verwenden** in Proxy Routes |
| `src/components/ui/badge.tsx` | shadcn Badge -- **verwenden** als Basis fuer PaymentStatusBadge |

**Etabliertes Proxy-Route-Pattern (aus api-keys/route.ts):**
```typescript
import { NextResponse } from "next/server"
import { requireAuth } from "@/lib/auth"
import { mmsApi } from "@/lib/api/client"
import { MmsApiError } from "@/lib/api/errors"

export async function GET(request: Request) {
  const result = await requireAuth(request)
  if ("error" in result) return result.error

  try {
    const response = await mmsApi<...>(path, undefined, result.apiKey)
    return NextResponse.json({ data: response.data })
  } catch (error) {
    if (error instanceof MmsApiError) {
      return NextResponse.json(
        { error: { code: error.code, message: error.message } },
        { status: error.status }
      )
    }
    return NextResponse.json(
      { error: { code: "INTERNAL_ERROR", message: "..." } },
      { status: 500 }
    )
  }
}
```

**Etabliertes Hook-Pattern (aus use-api-key.ts):**
```typescript
"use client"
import { useQuery } from "@tanstack/react-query"
import { parseApiError } from "@/lib/api/client-error"

async function fetchX(): Promise<X> {
  const res = await fetch("/api/...")
  if (!res.ok) throw await parseApiError(res)
  const json = await res.json()
  return json.data
}

export function useX() {
  return useQuery({
    queryKey: ["x"],
    queryFn: fetchX,
    staleTime: 30 * 1000,
  })
}
```

### Payment Interface Definition

Basierend auf MMS API Docs und Epics-Spezifikation:

```typescript
// src/types/payment.ts -- hinzufuegen zu bestehendem File
export interface Payment {
  id: string
  amount: string          // User-friendly amount (z.B. "49.00")
  exactAmount: string     // Exact 6-decimal amount (z.B. "49.000042")
  status: PaymentStatus
  txHash: string | null
  depositAddress: string
  confirmations: number
  confirmationsRequired: number
  currency: string        // "USDC"
  chain: string           // "base"
  metadata: Record<string, string> | null
  expiresAt: string       // ISO 8601
  createdAt: string       // ISO 8601
  updatedAt: string       // ISO 8601
}

export interface PaymentFilters {
  status?: PaymentStatus | PaymentStatus[]
  from?: string           // ISO date
  to?: string             // ISO date
  page?: number
  limit?: number
  sort?: "created_at"     // MMS unterstuetzt NUR created_at
  order?: "asc" | "desc"
}
```

### MMS API Endpunkte

**GET /v1/payments** (List):
- Query-Params: status, from, to, amount_min, amount_max, page, limit, sort, order
- Response: `{ data: Payment[], meta: { page, limit, total } }`
- Responses sind **camelCase** -- KEINE Transformation noetig

**GET /v1/payments/:id** (Single):
- Response: `{ data: Payment }`
- Responses sind **camelCase** -- KEINE Transformation noetig

### PaymentStatusBadge Spezifikation

Zwei Groessen: `sm` (Tabelle, kompakt) und `md` (Detail Header, groesser).

Farb-Mapping ueber CSS Custom Properties (bereits in globals.css definiert):
- `pending` -> `bg-muted text-muted-foreground`
- `matched` -> `bg-accent text-accent-foreground`
- `confirming` -> `bg-warning text-warning-foreground`
- `confirmed` -> `bg-success text-success-foreground`
- `expired` -> `bg-neutral text-neutral-foreground`
- `failed` -> `bg-error text-error-foreground`

**KRITISCH:** Badge zeigt IMMER Farbe + Text-Label (Accessibility, UX-Spec). Nie nur Farbe.

### formatAmount Spezifikation

```typescript
// Eingabe: "49.000042", "USDC"
// Ausgabe: "49.000042 USDC"
// Keine Rundung, keine Locale-Formatierung
// Dezimalpunkt (nicht Komma)
// Trailing Zeros beibehalten
```

### Architektur-Compliance

- Dateien in `kebab-case.tsx`
- Types in `types/` (nicht inline)
- MMS-Calls ueber `mmsApi()` in Route Handlers
- Route Handler Responses im Envelope-Format
- TanStack Query im Browser ruft `/api/...` auf -- nie direkt MMS
- Query Keys: Array-basiert, hierarchisch: `['payments']`, `['payments', id]`, `['payments', filters]`
- Kein `any` Type
- Kein Full-Page-Spinner
- Amounts immer in Geist Mono (font-mono class)

### Project Structure Notes

Neue Dateien die erstellt werden:
```
src/
  app/api/payments/
    route.ts                    # NEU: GET /api/payments -> MMS
    [id]/
      route.ts                  # NEU: GET /api/payments/:id -> MMS
  components/dashboard/
    payment-status-badge.tsx    # NEU: Status Badge Komponente
  hooks/
    use-payments.ts             # NEU: usePayments Hook
    use-payment.ts              # NEU: usePayment Hook
```

Bestehende Dateien die erweitert werden:
```
src/
  types/payment.ts              # ERWEITERN: Payment, PaymentFilters, WebhookDelivery
  lib/api/types.ts              # ERWEITERN: MmsPayment, MmsWebhookDelivery
  lib/utils.ts                  # ERWEITERN: formatAmount()
```

### References

- [Source: _bmad-output/planning-artifacts/epics.md#Story 3.1]
- [Source: _bmad-output/planning-artifacts/architecture.md#API Communication Pattern]
- [Source: _bmad-output/planning-artifacts/architecture.md#Component Patterns]
- [Source: _bmad-output/planning-artifacts/architecture.md#TanStack Query Patterns]
- [Source: _bmad-output/planning-artifacts/ux-design-specification.md#Payment Status Farb-Mapping]
- [Source: _bmad-output/planning-artifacts/prd.md#FR15-FR19, FR38]

## Dev Agent Record

### Agent Model Used

Claude Opus 4.6

### Debug Log References

### Completion Notes List

- Task 1: Payment, PaymentFilters, WebhookDelivery Interfaces in types/payment.ts hinzugefuegt; MmsPayment, MmsWebhookDelivery in lib/api/types.ts hinzugefuegt. Alle Felder exakt wie in Story-Spec und MMS API Docs.
- Task 2: Proxy Routes erstellt nach bestehendem api-keys Pattern. GET /api/payments leitet Query-Params (status, from, to, page, limit, sort, order) an MMS weiter. GET /api/payments/[id] fuer Einzel-Payments. Keine snakeToCamel-Transformation (Payments bereits camelCase). Envelope-Format { data, meta } bzw. { data }.
- Task 3: usePayments(filters?) und usePayment(id) Hooks nach bestehendem use-api-key Pattern. Query Keys hierarchisch ['payments', filters] bzw. ['payments', id]. staleTime 30s. enabled: !!id fuer usePayment.
- Task 4: PaymentStatusBadge mit size prop (sm/md), nutzt STATUS_CONFIG und shadcn Badge als Basis. Farb-Klassen per CSS Custom Properties. Zeigt immer Farbe + Text-Label (Accessibility).
- Task 5: formatAmount(amount, currency?) in lib/utils.ts. Keine Rundung, keine Locale-Formatierung, Trailing Zeros beibehalten.
- Task 6: npm run build erfolgreich. Alle Seiten (Landing, Dashboard, Settings) kompilieren fehlerfrei. Neue Routes /api/payments und /api/payments/[id] registriert.

### Change Log

- 2026-02-19: Story 3.1 implementiert — Payment Types, Proxy Routes, TanStack Query Hooks, PaymentStatusBadge, formatAmount Helper
- 2026-02-19: Code Review (Adversarial) — 8 Issues gefunden (2H, 4M, 2L), 6 gefixt:
  - H1/H2: JSDoc-Kommentare zu MmsPayment/MmsWebhookDelivery (ungenutzten snake_case Types bei camelCase-Realitaet)
  - M1: Doppelter Import in PaymentStatusBadge zusammengefuehrt
  - M2: COLOR_CLASSES von Record<string,string> auf Record<StatusColor,string> typisiert
  - M3: PAYMENT_STATUSES Konstante eingefuehrt (AC#1 Naming), STATUS_CONFIG als deprecated Alias
  - M4: Payment-ID Validierung im [id] Route Handler hinzugefuegt
  - L1/L2: Akzeptiert (keine Aenderung noetig)

### File List

**Neue Dateien:**
- src/app/api/payments/route.ts
- src/app/api/payments/[id]/route.ts
- src/components/dashboard/payment-status-badge.tsx
- src/hooks/use-payments.ts
- src/hooks/use-payment.ts

**Erweiterte Dateien:**
- src/types/payment.ts
- src/lib/api/types.ts
- src/lib/utils.ts
