# Story 3.2: Overview Page & Zustandsabhängiger First Screen

Status: done

<!-- Note: Validation is optional. Run validate-create-story for quality check before dev-story. -->

## Story

As a Tenant,
I want beim Dashboard-Einstieg entweder einen Onboarding-Screen (wenn ich neu bin) oder einen 5-Sekunden Health Check (wenn ich Payments habe) sehen,
So that ich als neuer User sofort loslegen kann und als wiederkehrender User in Sekunden weiß ob alles läuft.

## Acceptance Criteria

1. **Given** ein neu ongeboardeter Tenant das Dashboard öffnet (0 Payments) **When** die Overview-Seite lädt **Then** sehe ich den OnboardingScreen mit: API Key (masked, mit Copy-Button, aus GET /v1/paywatcher/config/api-key), curl-Beispiel zum Erstellen eines Payment Intent (mit vorausgefülltem API Key), Deposit Address + erwarteter Betrag (Platzhalter bis Payment erstellt), "Waiting for first payment..." mit Spinner, Links zu "Full Docs" und "Webhook Settings"

2. **Given** der Onboarding-Screen aktiv ist **When** der Tenant über seine API ein Payment erstellt und es den Status `confirmed` erreicht **Then** wechselt der Spinner zu einem grünen Haken ("First payment verified!") **And** nach kurzer Verzögerung wechselt der Screen automatisch zur Overview-Ansicht **And** kein "Dismiss"- oder "Skip"-Button existiert — der Wechsel ist systemgesteuert

3. **Given** ein Tenant mit bestehenden Payments das Dashboard öffnet **When** die Overview-Seite lädt **Then** sehe ich StatCards: Confirmed (success), Expired (neutral), Failed (error) — Zahlen der letzten 7 Tage **And** eine "Recent Payments"-Liste mit den letzten ~5 Payments (PaymentStatusBadge + Amount + ID + Timestamp) **And** eine HealthBar am unteren Rand (Webhook Health %, API Calls, Avg Confirmation Time)

4. **Given** die Overview-Daten laden **When** die API-Response noch aussteht **Then** sehe ich Skeleton-Placeholders: Card-Shapes für StatCards, Zeilen-Shapes für Recent Payments **And** Sidebar + Header stehen sofort (kein Full-Page-Spinner)

5. **Given** der Onboarding-Screen auf Mobile (<1024px) angezeigt wird **When** der Screen rendert **Then** ist das Layout einspaltig, Code-Block mit horizontal Scroll, alle Touch Targets min. 44x44px

## Tasks / Subtasks

- [x] Task 1: Bestehenden WelcomeScreen zum vollständigen OnboardingScreen erweitern (AC: #1, #2)
  - [x] 1.1 `src/components/dashboard/welcome-screen.tsx` — Erweitern: curl-Beispiel mit vorausgefülltem API Key (masked Key aus useApiKey), Copy-Button, Shiki/compact CodeBlock
  - [x] 1.2 `src/components/dashboard/welcome-screen.tsx` — "Waiting for first payment..." Spinner hinzufügen, Polling auf usePayments für confirmed-Status
  - [x] 1.3 `src/components/dashboard/welcome-screen.tsx` — Success-State: Spinner → grüner Haken ("First payment verified!"), automatischer Wechsel zur Overview nach 2-3s Delay
  - [x] 1.4 `src/components/dashboard/welcome-screen.tsx` — "Skip"-Button entfernen (falls vorhanden) — Wechsel nur systemgesteuert über confirmed Payment ODER localStorage-Flag `onboarding_complete`

- [x] Task 2: StatCard Komponente erstellen (AC: #3)
  - [x] 2.1 `src/components/dashboard/stat-card.tsx` — Komponente mit Props: label (string), value (number | string), variant (success | neutral | error | muted), subtext? (string)
  - [x] 2.2 Farbvarianten: success=grün, error=rot, neutral=grau/muted — CSS Custom Properties aus globals.css

- [x] Task 3: HealthBar Komponente erstellen (AC: #3)
  - [x] 3.1 `src/components/dashboard/health-bar.tsx` — Kompakte Leiste: Webhook Health %, API Calls, Avg Confirmation Time
  - [x] 3.2 Status-Dot: grün (alles OK), gelb (Warnungen), rot (Probleme)

- [x] Task 4: Overview-Seite bauen (AC: #3, #4)
  - [x] 4.1 `src/app/(dashboard)/dashboard/page.tsx` — Zustandslogik: 0 Payments → OnboardingScreen, >0 Payments → OverviewContent
  - [x] 4.2 `src/components/dashboard/overview-content.tsx` — Layout: StatCards (3er Grid), Recent Payments Liste, HealthBar am unteren Rand
  - [x] 4.3 Recent Payments Liste: Letzte ~5 Payments via usePayments({limit: 5, order: 'desc'}), PaymentStatusBadge + formatAmount + Payment ID (truncated, monospace) + Timestamp
  - [x] 4.4 StatCards: Confirmed/Expired/Failed Counts der letzten 7 Tage — via usePayments mit status-Filter + meta.total (Option B)
  - [x] 4.5 Skeleton-Loading: Card-Shapes für StatCards, Zeilen-Shapes für Recent Payments (Skeleton aus shadcn/ui)

- [x] Task 5: Responsive Design (AC: #5)
  - [x] 5.1 OnboardingScreen: Einspaltig auf Mobile, Code-Block mit `overflow-x-auto`, Touch Targets min. 44x44px
  - [x] 5.2 Overview: StatCards Stack vertikal auf Mobile, Recent Payments als kompakte Liste

- [x] Task 6: Build-Validierung
  - [x] 6.1 `npm run build` fehlerfrei
  - [x] 6.2 Bestehende Seiten (Landing, Dashboard Shell, Settings, Payments) funktionieren weiterhin
  - [x] 6.3 Onboarding-Screen wird korrekt bei 0 Payments angezeigt
  - [x] 6.4 Overview wird korrekt bei vorhandenen Payments angezeigt

## Dev Notes

### Kontext & Zweck

Dies ist **Story 3.2 von Epic 3** (Payment Monitoring Dashboard). Sie baut die zentrale Overview-Seite des Dashboards — den First Screen nach dem Login. Das Kernkonzept ist der **zustandsabhängige First Screen**: Neue User sehen Onboarding, wiederkehrende User sehen den 5-Sekunden Health Check.

**Was diese Story beinhaltet:**
- Erweiterung des bestehenden WelcomeScreen zum vollständigen OnboardingScreen (curl-Beispiel, Polling, Auto-Wechsel)
- Neue StatCard Komponente
- Neue HealthBar Komponente
- Overview-Seite mit StatCards + Recent Payments + HealthBar
- Zustandslogik (Onboarding vs. Overview)

**Was diese Story NICHT beinhaltet:**
- Payments List Page (Story 3.3)
- Filter/Sorting/Pagination (Story 3.4)
- Payment Detail Page (Story 3.5)

### KRITISCH: Bestehender WelcomeScreen erweitern, NICHT neu erstellen

In Story 2.3 wurde bereits `src/components/dashboard/welcome-screen.tsx` implementiert. Diese Komponente ERWEITERN, nicht ersetzen. Der WelcomeScreen zeigt bereits:
- API Key (masked) mit Copy-Button
- Deposit Address mit Copy-Button und Basescan-Link
- Webhook URL Status
- CTAs: Configure Webhook, View Docs

**Was fehlt und hinzugefügt werden muss:**
- curl-Beispiel mit vorausgefülltem (masked) API Key
- "Waiting for first payment..." Spinner mit Polling
- Success-State mit grünem Haken und Auto-Wechsel
- Entfernung des "Skip to Dashboard"-Buttons (Wechsel nur systemgesteuert)

### KRITISCH: Zustandslogik auf der Dashboard-Page

Die aktuelle `src/app/(dashboard)/page.tsx` (Story 2.3 Implementierung) nutzt bereits localStorage-Flag `onboarding_complete` um zwischen WelcomeScreen und Platzhalter zu wechseln. Für Story 3.2:

**Zustandslogik (ERSETZEN der bestehenden Platzhalter-Logik):**
```typescript
// Pseudo-Code für die Dashboard-Page
const { data: payments, isLoading } = usePayments({ limit: 1 })
const hasPayments = payments && payments.length > 0

// Bestimmung:
// 1. Wenn isLoading → Skeleton anzeigen
// 2. Wenn !hasPayments → OnboardingScreen (WelcomeScreen)
// 3. Wenn hasPayments → OverviewContent
```

**WICHTIG:** Die Zustandslogik soll NICHT mehr nur auf localStorage basieren, sondern auf echten Payment-Daten. Der localStorage-Flag `onboarding_complete` kann als Fallback/Cache dienen um Flackern beim Laden zu vermeiden, aber die primäre Quelle der Wahrheit ist: "Hat der Tenant Payments?"

### Bestehender Code-Stand (aus Story 3.1 + Epic 2)

**Bereits vorhanden — WIEDERVERWENDEN, nicht neu erstellen:**

| Datei | Was existiert |
|-------|-------------|
| `src/components/dashboard/welcome-screen.tsx` | WelcomeScreen — **ERWEITERN** |
| `src/components/dashboard/payment-status-badge.tsx` | PaymentStatusBadge (sm/md) — **VERWENDEN** in Recent Payments |
| `src/hooks/use-payments.ts` | usePayments(filters?) — **VERWENDEN** für Recent Payments + Stats |
| `src/hooks/use-payment.ts` | usePayment(id) — nicht direkt benötigt |
| `src/hooks/use-api-key.ts` | useApiKey() — **VERWENDEN** im OnboardingScreen (curl-Beispiel) |
| `src/hooks/use-tenant.ts` | useTenant() — **VERWENDEN** für Config-Daten |
| `src/lib/utils.ts` | formatAmount(), PAYMENT_STATUSES, cn() — **VERWENDEN** |
| `src/types/payment.ts` | Payment, PaymentStatus, PaymentFilters — **VERWENDEN** |
| `src/types/tenant.ts` | TenantConfig, ApiKeyInfo — **VERWENDEN** |
| `src/components/ui/skeleton.tsx` | shadcn Skeleton — **VERWENDEN** für Loading-States |
| `src/components/ui/card.tsx` | shadcn Card — **VERWENDEN** als Basis für StatCard |
| `src/components/ui/badge.tsx` | shadcn Badge — bereits in PaymentStatusBadge genutzt |
| `src/app/(dashboard)/page.tsx` | Dashboard-Page mit WelcomeScreen-Integration — **ERWEITERN** |
| `src/app/(dashboard)/layout.tsx` | Dashboard-Layout mit Auth-Check, Sidebar, Header — **NICHT ÄNDERN** |

**Etabliertes Hook-Pattern (aus Story 3.1 — use-payments.ts):**
```typescript
"use client"
import { useQuery } from "@tanstack/react-query"
import { parseApiError } from "@/lib/api/client-error"

async function fetchPayments(filters?: PaymentFilters): Promise<{ data: Payment[]; meta: PaginationMeta }> {
  const params = new URLSearchParams()
  // ... filter params
  const res = await fetch(`/api/payments?${params}`)
  if (!res.ok) throw await parseApiError(res)
  return res.json()
}

export function usePayments(filters?: PaymentFilters) {
  return useQuery({
    queryKey: ["payments", filters],
    queryFn: () => fetchPayments(filters),
    staleTime: 30 * 1000,
  })
}
```

### StatCard Spezifikation

Basierend auf UX-Spec — kompakte Kennzahl-Darstellung:

```
[Label]          (caption, text-muted-foreground)
[Zahl]           (text-stat size, farbig nach variant)
[Subtext]        (caption, text-muted-foreground, optional)
```

**Varianten:**
- `success` — grün (bestätigte Payments)
- `neutral` / `muted` — grau (abgelaufene Payments)
- `error` — rot (fehlgeschlagene Payments)

**Implementierung:** shadcn Card als Basis, Farben über CSS Custom Properties (`--success`, `--error`, `--muted-foreground` aus globals.css).

### HealthBar Spezifikation

Basierend auf UX-Spec — kompakte Status-Leiste:

```
[●] Webhook Health: 100%  |  API Calls: 847  |  Avg Confirmation: 18s
```

**HINWEIS:** Es gibt keinen dedizierten MMS-Endpoint für diese Metriken. Die Daten müssen aus vorhandenen Quellen abgeleitet werden:
- **Webhook Health %:** Aus den Payments + deren Webhook-Histories berechnen (pro Payment: hat Webhook → success/fail ratio). Alternativ: Vereinfacht als Platzhalter "—" anzeigen wenn keine Webhook-Daten auf der Overview verfügbar sind (da GET /v1/payments/:id/webhooks einzeln pro Payment aufgerufen werden müsste). **Empfehlung: In Story 3.2 als vereinfachte Version ohne Webhook Health implementieren, oder mit Platzhalter-Wert.**
- **API Calls:** Kein MMS-Endpoint. Platzhalter oder weglassen.
- **Avg Confirmation Time:** Aus den Payments berechnen (Differenz createdAt → updatedAt für confirmed Payments). Oder weglassen.

**Pragmatische Empfehlung:** Die HealthBar in Story 3.2 mit den Daten befüllen die aus der Payments-Liste ableitbar sind (z.B. Confirmed Count, Failed Count, ggf. Avg Time). Metriken die eigene Endpoints benötigen können als "—" angezeigt oder in einer späteren Story ergänzt werden.

### Overview-Layout (Desktop)

```
┌─────────────────────────────────────────────────┐
│  Last 7 days                                    │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐      │
│  │ Confirmed │  │ Expired  │  │  Failed  │      │
│  │    12     │  │    2     │  │    0     │      │
│  └──────────┘  └──────────┘  └──────────┘      │
│                                                 │
│  Recent Payments                                │
│  ┌─────────────────────────────────────────┐    │
│  │ #pay_7f2a  49.000042 USDC   ● confirmed │    │
│  │ #pay_3b1c  120.000187 USDC  ● confirmed │    │
│  │ #pay_9e4d  25.000523 USDC   ◌ pending   │    │
│  │ ...                                     │    │
│  └─────────────────────────────────────────┘    │
│  [View all payments →]                          │
│                                                 │
│  ───────────────────────────────────────────    │
│  Webhook Health: —  │  Avg Confirmation: 18s    │
└─────────────────────────────────────────────────┘
```

**Content-Breite:** `max-w-6xl` zentriert (UX-Spec: "Statistik-Cards brauchen keinen Fullscreen").

### StatCards Berechnung (aus Payments-Daten)

Da kein separater Statistik-Endpoint existiert, werden die StatCard-Zahlen aus der Payments-Response berechnet:

```typescript
// Option A: Client-seitig filtern (einfach, aber limitiert bei vielen Payments)
const sevenDaysAgo = new Date(Date.now() - 7 * 24 * 60 * 60 * 1000).toISOString()
const { data } = usePayments({ from: sevenDaysAgo, limit: 100, order: "desc" })
const confirmed = data?.data.filter(p => p.status === "confirmed").length ?? 0
const expired = data?.data.filter(p => p.status === "expired").length ?? 0
const failed = data?.data.filter(p => p.status === "failed").length ?? 0

// Option B: Drei separate API Calls mit Status-Filter (genauer, nutzt MMS-Pagination meta.total)
const { data: confirmedData } = usePayments({ from: sevenDaysAgo, status: "confirmed", limit: 1 })
const confirmedCount = confirmedData?.meta?.total ?? 0
// ... analog für expired, failed
```

**Empfehlung:** Option B ist genauer (nutzt `meta.total` statt client-seitiges Zählen) aber macht 3 API Calls. Option A ist einfacher. **Wähle basierend auf erwarteter Datenmenge** — bei wenigen Payments (<100 pro 7 Tage) reicht Option A.

### Polling für Onboarding (Warten auf ersten confirmed Payment)

```typescript
// Im OnboardingScreen: Polling auf Payments alle 10s
const { data: payments } = usePayments({ limit: 1, order: "desc" })
const hasConfirmedPayment = payments?.data?.some(p => p.status === "confirmed")

// Wenn confirmed → Success-State anzeigen, nach 2-3s Delay localStorage setzen und Overview laden
```

**refetchInterval:** 10_000 (10 Sekunden, konsistent mit Story 3.3 Spezifikation für aktive Payments).

### Payment-Liste in Recent Payments

Die Recent Payments Liste zeigt die letzten ~5 Payments als kompakte Zeilen (KEINE volle DataTable):

```
[Payment ID (truncated, monospace)] [Amount (monospace)] [PaymentStatusBadge sm] [Timestamp]
```

- Payment ID: `pay_7f2a...` (truncated, `font-mono`)
- Amount: `49.000042 USDC` (via formatAmount, `font-mono`)
- Status: `PaymentStatusBadge` mit `size="sm"`
- Timestamp: Relative oder absolute (UX-Spec sagt absolute Timestamps, kein relative)
- Klick auf Zeile → navigiert zu `/payments/${id}` (Payment Detail, Story 3.5) — kann als Link vorbereitet werden

### Learnings aus Story 3.1 (Previous Story Intelligence)

- **camelCase Payments:** MMS Payment-Responses sind bereits camelCase — KEINE Transformation nötig
- **Proxy Pattern:** Etabliert in `app/api/payments/route.ts` — requireAuth(), mmsApi(), Envelope Format
- **Hook Pattern:** `usePayments(filters?)` mit Query Key `['payments', filters]`, staleTime 30s
- **PaymentStatusBadge:** Funktioniert, nutzt STATUS_CONFIG/PAYMENT_STATUSES aus utils
- **formatAmount:** In `lib/utils.ts`, Eingabe String + Currency, keine Rundung
- **Code Review Learnings:** Payment-ID Validierung im Route Handler hinzugefügt (M4 Fix), PAYMENT_STATUSES als offizieller Export-Name (M3)

### Git Intelligence (letzte 5 Commits)

1. `4f2b9c3` — Add payments data layer, proxy routes, hooks & PaymentStatusBadge (Story 3.1)
2. `a783b6d` — Remove debug logging from proxy
3. `bdaeddb` — Fix session: move MMS tenant resolution from signIn to jwt callback
4. `36189d5` — Add proxy debug logging to diagnose session issue
5. `063f5a7` — Add verbose MMS response logging to signIn callback

**Relevantes Pattern:** Story 3.1 Commit zeigt dass alle Dateien in einem Commit zusammengefasst werden. Auth-Session-Fix (Commit 3) betrifft JWT Callback — Session-Daten (tenantSlug etc.) sind jetzt stabil verfügbar.

### Architektur-Compliance

- Dateien in `kebab-case.tsx`
- Types in `types/` (nicht inline)
- Neue Komponenten in `components/dashboard/` (stat-card.tsx, health-bar.tsx, overview-content.tsx)
- TanStack Query im Browser ruft `/api/...` auf — nie direkt MMS
- Query Keys: Array-basiert, hierarchisch: `['payments', filters]`
- Kein `any` Type
- Kein Full-Page-Spinner — Skeleton Loading für alle Dashboard-Inhalte
- Amounts immer in `font-mono` (Geist Mono)
- Max 1 Primary Button pro Screen
- Copy-Feedback: Inline-Checkmark, kein Toast
- Content-Breite: `max-w-6xl` zentriert für Overview

### Project Structure Notes

Neue Dateien die erstellt werden:
```
src/
  components/dashboard/
    stat-card.tsx               # NEU: StatCard Komponente
    health-bar.tsx              # NEU: HealthBar Komponente
    overview-content.tsx        # NEU: Overview-Ansicht (StatCards + Recent Payments + HealthBar)
```

Bestehende Dateien die erweitert werden:
```
src/
  app/(dashboard)/page.tsx             # ERWEITERN: Zustandslogik (Onboarding vs. Overview) basierend auf Payment-Daten
  components/dashboard/welcome-screen.tsx  # ERWEITERN: curl-Beispiel, Polling, Success-State, Skip-Button entfernen
```

### References

- [Source: _bmad-output/planning-artifacts/epics.md#Story 3.2]
- [Source: _bmad-output/planning-artifacts/architecture.md#Component Patterns]
- [Source: _bmad-output/planning-artifacts/architecture.md#TanStack Query Patterns]
- [Source: _bmad-output/planning-artifacts/ux-design-specification.md#Experience Mechanics]
- [Source: _bmad-output/planning-artifacts/ux-design-specification.md#StatCard]
- [Source: _bmad-output/planning-artifacts/ux-design-specification.md#OnboardingScreen]
- [Source: _bmad-output/planning-artifacts/ux-design-specification.md#HealthBar]
- [Source: _bmad-output/planning-artifacts/prd.md#FR15-FR19]
- [Source: _bmad-output/implementation-artifacts/3-1-payments-data-layer-paymentstatusbadge.md]

## Dev Agent Record

### Agent Model Used

Claude Opus 4.6 (claude-opus-4-6)

### Debug Log References

### Completion Notes List

- Task 1: WelcomeScreen erweitert mit curl-Beispiel (masked API Key, Copy-Button), Polling alle 10s via usePayments mit refetchInterval, Success-State (CheckCircle2 → auto-switch nach 2.5s), Skip-Button entfernt. Zustandswechsel ist systemgesteuert über confirmed Payment-Erkennung.
- Task 2: StatCard Komponente erstellt mit Varianten success/neutral/error/muted, tabular-nums für Zahlenausrichtung, basiert auf shadcn Card.
- Task 3: HealthBar Komponente erstellt mit Status-Dot (grün/gelb/rot), Webhook Health als Platzhalter "—" (kein MMS-Endpoint), Avg Confirmation Time aus Payments berechnet.
- Task 4: Dashboard-Page komplett umgebaut — Zustandslogik basiert auf Payment-Daten statt localStorage. OverviewContent mit 3 StatCards (Confirmed/Expired/Failed, letzte 7 Tage via meta.total), Recent Payments Liste (letzte 5, mit PaymentStatusBadge, truncated ID, formatAmount, Timestamp), HealthBar. Skeleton-Loading für alle Bereiche.
- Task 5: Responsive Design — Grid-Breakpoints (md:grid-cols-2 für Onboarding, sm:grid-cols-3 für Stats), overflow-x-auto auf Code-Block, min-h/w 44px auf Copy-Buttons.
- Task 6: Build erfolgreich, alle Routes intakt.
- usePayments Hook erweitert um optionale refetchInterval und enabled Parameter.

### Implementation Plan

- Zustandslogik: Payment-Daten als primäre Quelle (usePayments limit:1), kein localStorage mehr
- StatCards: Option B gewählt (3 separate API-Calls mit status-Filter, meta.total für genaue Counts)
- HealthBar: Webhook Health als "—" (kein dedizierter Endpoint), Avg Confirmation Time berechnet aus confirmed Payments (updatedAt - createdAt)
- Polling: refetchInterval 10s im OnboardingScreen, stoppt bei Success-State

### File List

**Neue Dateien:**
- src/components/dashboard/stat-card.tsx
- src/components/dashboard/health-bar.tsx
- src/components/dashboard/overview-content.tsx

**Geänderte Dateien:**
- src/app/(dashboard)/dashboard/page.tsx — Zustandslogik komplett überarbeitet
- src/components/dashboard/welcome-screen.tsx — curl-Beispiel, Polling, Success-State, Skip entfernt
- src/hooks/use-payments.ts — refetchInterval + enabled Options hinzugefügt

### Change Log

- 2026-02-19: Story 3.2 implementiert — Overview Page mit zustandsabhängigem First Screen (Onboarding vs. Overview), StatCards, HealthBar, Polling, Skeleton-Loading
- 2026-02-19: Code Review (AI) — 6 Issues gefixed: H1 falsche API-URL im curl (api.masempay.com→api.masem.at), H2 sevenDaysAgo() memoized (Query-Key-Thrashing verhindert), H3 erwarteter Betrag Platzhalter hinzugefügt, M1 payments.isError-Behandlung, M2 Polling mit status:confirmed Filter, M3 useCallback-Konsistenz
