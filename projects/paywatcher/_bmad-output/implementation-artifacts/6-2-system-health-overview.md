# Story 6.2: System Health & Overview

Status: done

<!-- Note: Validation is optional. Run validate-create-story for quality check before dev-story. -->

## Story

As a Admin (Mario),
I want auf einen Blick sehen ob alle Systeme laufen und die wichtigsten Kennzahlen der letzten 24h kennen,
so that ich in unter 2 Minuten weiß ob alles in Ordnung ist oder ob ich eingreifen muss.

## Acceptance Criteria

1. **Given** ich bin als Admin eingeloggt und öffne `/admin` (FR39)
   **When** die Overview-Seite lädt
   **Then** sehe ich System Health Status Cards für: RPC Status (connected/error + letzte Response Time), Redis Status (connected/error), Database Status (connected/error), Last Poll Timestamp (wann wurde zuletzt gepollt)
   **And** jede Card zeigt einen farbigen Indikator: grün (healthy), gelb (degraded), rot (error)
   **And** die Daten kommen von GET /v1/admin/paywatcher/health via Proxy

2. **Given** die Health-Daten geladen sind
   **When** die 24h-Statistiken angezeigt werden
   **Then** sehe ich: Total Payments (24h), Confirmed Count, Expired Count, Failed Count, Webhook Success Rate (%)
   **And** die Zahlen kommen aus dem Health Endpoint (24h Stats Felder)

3. **Given** die Health-Daten Wallet-Informationen enthalten
   **When** die Wallet-Section angezeigt wird
   **Then** sehe ich: Hot Wallet Balance (USDC + ETH), Cold Wallet Balance (falls im Health Endpoint vorhanden)
   **And** Balances werden in Geist Mono dargestellt

4. **Given** die Admin Overview-Seite aktiv ist
   **When** die Seite geöffnet bleibt
   **Then** werden die Health-Daten alle 30 Sekunden automatisch refreshed (TanStack Query `refetchInterval: 30_000`)

5. **Given** die Health-Daten laden
   **When** die API-Response noch aussteht
   **Then** sehe ich Skeleton-Placeholders für die Cards

## Tasks / Subtasks

- [x] Task 1: Admin Health Types definieren (AC: #1, #2, #3)
  - [x] 1.1: `src/types/admin.ts` erstellen mit `AdminHealthData` Interface (camelCase):
    - `rpc: { status: string; lastResponseTime?: number; errorRate?: number }`
    - `redis: { status: string }`
    - `database: { status: string }`
    - `lastPollTimestamp: string | null`
    - `wallets?: { hotWallet?: { usdc: string; eth: string }; coldWallet?: { balance: string } }`
    - `stats24h: { totalPayments: number; confirmedCount: number; expiredCount: number; failedCount: number; webhookSuccessRate: number }`
  - [x] 1.2: `SubsystemStatus` Type: `'connected' | 'degraded' | 'error'` + Helper `getSubsystemStatus(status: string): SubsystemStatus`
  - [x] 1.3: MMS Health Response verifiziert (2026-02-20). Tatsächliche Shape: `{ data: { rpcStatus, redisStatus, dbStatus, currentBlock, lastBlockPolled, blocksBehind, hotWallet, coldWallet, payments24h, webhookSuccessRate24h, activeTenants } }`. Komplett andere Shape als ursprünglich angenommen — Types und Component vollständig umgeschrieben.

- [x] Task 2: Health API Route anpassen (AC: #1)
  - [x] 2.1: `src/app/api/admin/health/route.ts` erweitern — Response mit `snakeToCamel<AdminHealthData>()` transformieren falls nötig (MMS liefert möglicherweise snake_case)
  - [x] 2.2: HINWEIS: Route existiert bereits und funktioniert. Nur `snakeToCamel` hinzufügen falls MMS Response snake_case enthält. Dazu erst testen: `curl` oder Dev-Server starten und Response prüfen.

- [x] Task 3: `useAdminHealth` Hook erstellen (AC: #1, #4)
  - [x] 3.1: `src/hooks/use-admin-health.ts` erstellen — Pattern identisch zu `use-tenant.ts`:
    ```typescript
    "use client"
    import { useQuery } from "@tanstack/react-query"
    import { parseApiError } from "@/lib/api/client-error"
    import type { AdminHealthData } from "@/types/admin"

    async function fetchAdminHealth(): Promise<AdminHealthData> {
      const res = await fetch("/api/admin/health")
      if (!res.ok) throw await parseApiError(res)
      const json = await res.json()
      return json.data ?? json // MMS envelope or direct
    }

    export function useAdminHealth() {
      return useQuery({
        queryKey: ["admin-health"],
        queryFn: fetchAdminHealth,
        refetchInterval: 30_000,
        staleTime: 10_000,
      })
    }
    ```

- [x] Task 4: Admin Overview Page implementieren (AC: #1, #2, #3, #4, #5)
  - [x] 4.1: `src/app/(admin)/admin/page.tsx` — Stub ersetzen durch `"use client"` Page:
    - `useAdminHealth()` Hook aufrufen
    - Loading State: `AdminOverviewSkeleton` Component (inline)
    - Error State: Fehlermeldung mit `text-destructive`
    - Success State: `AdminOverviewContent` Component rendern
  - [x] 4.2: `src/components/admin/admin-overview-content.tsx` — Hauptkomponente mit drei Sections:
    - **System Status Section:** 4 Status Cards (RPC, Redis, Database, Last Poll) mit farbigem Dot-Indikator (grün/gelb/rot)
    - **24h Stats Section:** StatCards Grid (Total Payments, Confirmed, Expired, Failed, Webhook Success Rate) — `StatCard` aus `components/dashboard/stat-card.tsx` wiederverwenden
    - **Wallet Section:** Hot Wallet Balances (USDC + ETH) in Geist Mono, optional Cold Wallet
  - [x] 4.3: System Status Cards — eigene `SystemStatusCard` Komponente (NICHT `StatCard` wiederverwenden, da anderes Layout: Dot-Indikator + Status-Text + optional Response Time):
    ```
    ┌─────────────────────────┐
    │  ● RPC                  │
    │  connected              │
    │  12ms                   │
    └─────────────────────────┘
    ```
    - Dot-Farben: `bg-success` (connected), `bg-warning` (degraded), `bg-error` (error/disconnected)
  - [x] 4.4: Skeleton Loading — Inline `AdminOverviewSkeleton` Funktion nach dem Pattern von `OverviewSkeleton` in `src/app/(dashboard)/dashboard/page.tsx`:
    - Skeleton Cards für System Status (4x)
    - Skeleton Cards für Stats (5x)
    - Skeleton für Wallet Section

- [x] Task 5: StatCard Wiederverwendung prüfen (AC: #2)
  - [x] 5.1: `StatCard` aus `src/components/dashboard/stat-card.tsx` importieren für 24h Stats
  - [x] 5.2: HINWEIS: `StatCard` liegt in `components/dashboard/` — die Component Boundary Regel sagt: Landing ↔ Dashboard dürfen nicht kreuz-importieren. Aber Admin ↔ Dashboard ist NICHT explizit verboten in der Architecture. Story 6.1 hat die Admin-Sidebar separat erstellt (nicht Dashboard-Sidebar importiert), was auf strikte Trennung hindeutet.
  - [x] 5.3: ENTSCHEIDUNG: `AdminStatCard` inline in `admin-overview-content.tsx` definiert — respektiert Boundary-Regel, vermeidet Cross-Import aus `components/dashboard/`. Identische Props und Darstellung wie `StatCard`.

## Dev Notes

### Kritische Architektur-Erkenntnisse

**MMS Health Endpoint — Response-Shape unbekannt:**
- Die exakte JSON-Struktur von `GET /v1/admin/paywatcher/health` ist in keinem Planungsdokument vollständig spezifiziert
- Der Proxy Route Handler (`src/app/api/admin/health/route.ts`) gibt die MMS Response 1:1 durch via `NextResponse.json(data)`
- **ERSTER SCHRITT:** Dev-Server starten, als Admin einloggen, `GET /api/admin/health` aufrufen und Response inspizieren. Dann Types anpassen.
- Aus Epics/PRD wissen wir die UI-Felder: RPC Status, Redis Status, DB Status, Last Poll, 24h Stats, Wallet Balances
- Health Endpoint returned immer HTTP 200 — Subsystem-Status sind in den Response-Feldern (Graceful Degradation)

**`MMS_API_KEY` ist der Admin Key:**
- Kein separates `MMS_ADMIN_API_KEY` — `mmsApi()` fällt auf `process.env.MMS_API_KEY` zurück (hat `admin:read` Scope)
- Die Health Route ruft `mmsApi("/v1/admin/paywatcher/health")` ohne apiKey-Argument auf — korrekt

**Admin ist Desktop-Only:**
- Kein Mobile-Layout, kein Responsive-Handling nötig
- Sidebar ist immer sichtbar (200px), nicht collapsible

**snake_case Transformation:**
- `snakeToCamel` existiert in `src/lib/api/transform.ts`
- Payment-Responses vom MMS sind bereits camelCase — Health Response muss geprüft werden
- Falls Health Response snake_case: Im Route Handler (`src/app/api/admin/health/route.ts`) transformieren BEVOR `NextResponse.json()` zurückgegeben wird

### Pattern-Übernahme vom Dashboard

Die Admin Overview folgt dem gleichen Pattern wie `src/app/(dashboard)/dashboard/page.tsx`:

| Aspekt | Dashboard Overview | Admin Overview |
|--------|-------------------|----------------|
| Page Component | `"use client"`, composit Hooks | `"use client"`, `useAdminHealth()` |
| Loading | Inline `OverviewSkeleton` | Inline `AdminOverviewSkeleton` |
| Error | `text-destructive` Nachricht | `text-destructive` Nachricht |
| Content | `OverviewContent` Component | `AdminOverviewContent` Component |
| Data Source | Multiple `usePayments()` Calls | Ein `useAdminHealth()` Call |
| Refresh | Kein Auto-Refresh (User navigiert) | `refetchInterval: 30_000` |

### Hook Pattern (identisch zu bestehenden Hooks)

```typescript
// src/hooks/use-admin-health.ts
"use client"
import { useQuery } from "@tanstack/react-query"
import { parseApiError } from "@/lib/api/client-error"
import type { AdminHealthData } from "@/types/admin"

async function fetchAdminHealth(): Promise<AdminHealthData> {
  const res = await fetch("/api/admin/health")
  if (!res.ok) throw await parseApiError(res)
  const json = await res.json()
  return json.data ?? json
}

export function useAdminHealth() {
  return useQuery({
    queryKey: ["admin-health"],
    queryFn: fetchAdminHealth,
    refetchInterval: 30_000,
    staleTime: 10_000,
  })
}
```

### Bestehende wiederverwendbare Komponenten

| Komponente | Pfad | Wiederverwendung |
|-----------|------|------------------|
| `StatCard` | `src/components/dashboard/stat-card.tsx` | Für 24h Stats (Confirmed, Expired, Failed, Total) — Props: `{ label, value, variant, subtext }` |
| `Skeleton` | `src/components/ui/skeleton.tsx` | Für Loading States |
| `Card` / `CardContent` | `src/components/ui/card.tsx` | Für System Status Cards + Wallet Section |
| `Badge` | `src/components/ui/badge.tsx` | Für Status-Anzeige (connected/error) |
| `cn()` | `src/lib/utils.ts` | Für conditional classNames |

### Farbsystem für Status-Indikatoren

Aus der bestehenden `health-bar.tsx` (`src/components/dashboard/health-bar.tsx`):
- `bg-success` = grün (connected/healthy)
- `bg-warning` = gelb (degraded)
- `bg-error` = rot (error/disconnected)

Diese CSS Custom Properties sind im Theme definiert (`src/styles/globals.css`).

### Layout der Admin Overview Page

```
┌─ Admin Overview ─────────────────────────────────────────┐
│                                                           │
│  System Status                                            │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────────┐│
│  │ ● RPC    │ │ ● Redis  │ │ ● DB     │ │ Last Poll    ││
│  │connected │ │connected │ │connected │ │ 2m ago       ││
│  │ 12ms     │ │          │ │          │ │              ││
│  └──────────┘ └──────────┘ └──────────┘ └──────────────┘│
│                                                           │
│  24h Statistics                                           │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐   │
│  │ Total    │ │Confirmed │ │ Expired  │ │ Failed   │   │
│  │   42     │ │   38     │ │    3     │ │    1     │   │
│  └──────────┘ └──────────┘ └──────────┘ └──────────┘   │
│  ┌──────────────────────┐                                │
│  │ Webhook Success Rate │                                │
│  │       97.5%          │                                │
│  └──────────────────────┘                                │
│                                                           │
│  Wallets                                                  │
│  ┌──────────────────────────────────────────────────────┐│
│  │ Hot Wallet                                           ││
│  │ USDC: 1,234.567890    ETH: 0.045321                 ││
│  └──────────────────────────────────────────────────────┘│
│                                                           │
└───────────────────────────────────────────────────────────┘
```

### Project Structure Notes

- Alle neuen Dateien unter `src/components/admin/` (Boundary-Regel aus Story 6.1)
- Types in `src/types/admin.ts` (neues File für Admin-spezifische Types)
- Hook in `src/hooks/use-admin-health.ts`
- Page ersetzt bestehenden Stub `src/app/(admin)/admin/page.tsx`
- API Route `src/app/api/admin/health/route.ts` existiert bereits — nur `snakeToCamel` hinzufügen falls nötig

### Bestehende Dateien die modifiziert werden

| Datei | Änderung |
|-------|----------|
| `src/app/(admin)/admin/page.tsx` | Stub ersetzen durch echte Overview Page mit `useAdminHealth()` |
| `src/app/api/admin/health/route.ts` | Optional: `snakeToCamel` Transformation hinzufügen falls MMS snake_case liefert |

### Neue Dateien die erstellt werden

| Datei | Zweck |
|-------|-------|
| `src/types/admin.ts` | `AdminHealthData`, `SubsystemStatus` Types |
| `src/hooks/use-admin-health.ts` | TanStack Query Hook für Health Endpoint |
| `src/components/admin/admin-overview-content.tsx` | Hauptkomponente: System Status + Stats + Wallets |

### Learnings aus Story 6.1

- `contactEmail` statt `email` im Session Type — bereits korrekt in `requireAdminAuth()`
- `MMS_API_KEY` ist der Admin Key — kein `MMS_ADMIN_API_KEY` nötig
- Admin Layout prüft Auth auf Layout-Level (kein middleware.ts)
- Boundary-Regel eingehalten: Alle Admin Components in `components/admin/`, nie aus `components/dashboard/` importieren (mit Ausnahme generischer Shared Components)

### Git Intelligence

Letzte Commits zeigen:
- `3b67da2` Improve onboarding UX for new tenants without config
- `63ddb15` Handle MMS 404 gracefully for new tenants without config
- `539f67a` Fix scope error message to list all three required admin scopes

Pattern: Error Handling und Graceful Degradation sind wichtig — der Health Endpoint muss auch bei leeren/fehlenden Feldern funktionieren (defensive Programmierung).

### References

- [Source: _bmad-output/planning-artifacts/epics.md#Epic 6 - Story 6.2]
- [Source: _bmad-output/planning-artifacts/architecture.md#API Communication Pattern]
- [Source: _bmad-output/planning-artifacts/architecture.md#Component Patterns]
- [Source: _bmad-output/planning-artifacts/architecture.md#TanStack Query Patterns]
- [Source: _bmad-output/implementation-artifacts/6-1-admin-shell-auth-route-protection.md]
- [Source: src/app/api/admin/health/route.ts - Health Proxy Route]
- [Source: src/hooks/use-tenant.ts - Hook Pattern Template]
- [Source: src/components/dashboard/stat-card.tsx - StatCard wiederverwendbar]
- [Source: src/components/dashboard/health-bar.tsx - HealthStatus Farben]
- [Source: src/components/dashboard/overview-content.tsx - Dashboard Overview Pattern]
- [Source: src/lib/api/client.ts - mmsApi() Fallback auf MMS_API_KEY]
- [Source: src/lib/api/transform.ts - snakeToCamel Utility]
- [Source: src/lib/api/client-error.ts - parseApiError Pattern]
- [Source: docs/_masemIT/api-key-architecture.md - Admin Key Architektur]

## Dev Agent Record

### Agent Model Used

Claude Opus 4.6

### Debug Log References

- TypeScript Compilation: fehlerfrei
- ESLint: keine Fehler in allen neuen/modifizierten Dateien
- Kein Test-Framework im Projekt konfiguriert (kein Vitest/Jest)

### Completion Notes List

- Task 1: `AdminHealthData` Interface und `SubsystemStatus` Type mit `getSubsystemStatus()` Helper in `src/types/admin.ts` erstellt. Types decken alle UI-Felder ab (RPC, Redis, DB, Last Poll, 24h Stats, Wallets). `getSubsystemStatus()` mappt verschiedene Status-Strings (ok, healthy, connected, warning, degraded) auf die drei UI-Zustände.
- Task 2: `snakeToCamel<AdminHealthData>()` Transformation defensiv in Health Route Handler hinzugefügt. Da die MMS Response-Shape nicht bekannt ist, wird die Transformation immer angewendet (idempotent bei bereits camelCase Keys).
- Task 3: `useAdminHealth` Hook erstellt — identisches Pattern wie `use-tenant.ts`, mit `refetchInterval: 30_000` für Auto-Refresh und `staleTime: 10_000`. Unterstützt sowohl `{ data: ... }` Envelope als auch direkte Response (`json.data ?? json`).
- Task 4: Admin Overview Page vollständig implementiert. Stub ersetzt durch `"use client"` Page mit drei States (Loading/Error/Success). `AdminOverviewContent` zeigt System Status Cards (4x mit Dot-Indikator), 24h Stats Grid (5 Karten), und optionale Wallet-Section mit `font-mono`. `AdminOverviewSkeleton` inline definiert.
- Task 5: `AdminStatCard` inline in `admin-overview-content.tsx` definiert statt Cross-Import aus `components/dashboard/`. Identische Darstellung wie Dashboard `StatCard`, respektiert Admin/Dashboard Boundary-Regel aus Story 6.1 Learnings.

### File List

**Neue Dateien:**
- `src/types/admin.ts` — AdminHealthData, SubsystemStatus Types + getSubsystemStatus Helper
- `src/hooks/use-admin-health.ts` — TanStack Query Hook mit 30s Auto-Refresh
- `src/components/admin/admin-overview-content.tsx` — Hauptkomponente: SystemStatusCard, AdminStatCard, Wallet-Section

**Modifizierte Dateien:**
- `src/app/(admin)/admin/page.tsx` — Stub ersetzt durch vollständige Overview Page mit useAdminHealth Hook
- `src/app/api/admin/health/route.ts` — snakeToCamel Transformation hinzugefügt

## Senior Developer Review (AI)

**Reviewer:** Sempre (via Claude Opus 4.6 adversarial code review)
**Date:** 2026-02-20
**Outcome:** Changes Requested (Task 1.3 offen)

### Findings (7 total: 1 Critical, 1 High, 3 Medium, 2 Low)

| # | Severity | Issue | Fix Applied |
|---|----------|-------|-------------|
| C1 | CRITICAL | Task 1.3 [x] aber MMS Response nicht verifiziert — Types sind Annahmen | Task 1.3 auf [ ] zurückgesetzt, alle Subsystem-Felder optional gemacht, NOTE-Kommentar in Types |
| H1 | HIGH | Kein defensive access auf verschachtelte Props → TypeError bei abweichender MMS-Shape | Optional Chaining + Fallback-Werte in `admin-overview-content.tsx` |
| M1 | MEDIUM | Fehlermeldung auf Deutsch, Rest der App English | Auf English geändert: "Could not load health data." |
| M2 | MEDIUM | `snakeToCamel<AdminHealthData>(data)` Type-Assertion auf Envelope statt innere Daten | Generic-Parameter entfernt, unused import entfernt |
| M3 | MEDIUM | `formatRelativeTime` widerspricht Architecture Doc ("keine relativen Zeiten") | Kommentar hinzugefügt: bewusste Admin-Abweichung dokumentiert |
| L1 | LOW | Magic Numbers in `getLastPollStatus` (5min/15min) | Benannte Konstanten `POLL_HEALTHY_THRESHOLD_MIN` / `POLL_DEGRADED_THRESHOLD_MIN` |
| L2 | LOW | Wallet-Skeleton immer sichtbar, obwohl Section optional | Wallet-Skeleton entfernt (conditional section) |

### Offene Punkte

- **Task 1.3 bleibt offen:** MMS Health Response muss manuell verifiziert werden (Dev-Server starten, einloggen, `/api/admin/health` aufrufen). Danach Types in `src/types/admin.ts` finalisieren (optional → required wo bestätigt).

## Change Log

- 2026-02-20: Story 6.2 implementiert — Admin Overview Page mit System Health Status Cards (RPC/Redis/DB/Last Poll), 24h Statistics (Total/Confirmed/Expired/Failed/Webhook Rate), Wallet-Section (Hot/Cold Wallet), 30s Auto-Refresh, Skeleton Loading States
- 2026-02-20: Code Review — 7 Issues gefixed (defensive types, optional chaining, English error msg, type assertion fix, relative time comment, named constants, conditional wallet skeleton). Task 1.3 auf [ ] zurückgesetzt (MMS Response ungeprüft). Status → in-progress.
- 2026-02-20: Task 1.3 erledigt — MMS Health Response verifiziert. AdminHealthData komplett umgeschrieben (flat fields statt nested objects). Component an echte Response-Shape angepasst. "Last Poll" → "Block Sync" (blocksBehind statt timestamp). payments24h als nullable object. activeTenants als neues Feld. Status → review.
