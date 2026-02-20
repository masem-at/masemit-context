# Story 4.2: Test Webhook & Delivery Log

Status: done

<!-- Note: Validation is optional. Run validate-create-story for quality check before dev-story. -->

## Story

As a Tenant,
I want einen Test-Webhook an meine URL senden und den Delivery Log einsehen können,
So that ich sicher sein kann dass meine Webhook-Integration funktioniert und Probleme debuggen kann.

## Acceptance Criteria

1. **Given** eine Webhook-URL ist konfiguriert (FR21) **When** ich auf "Send Test Webhook" klicke **Then** wird ein Test-Webhook via Proxy an POST /v1/paywatcher/config/test-webhook gesendet **And** der Button zeigt einen Loading-State während der Request läuft

2. **Given** der Test-Webhook erfolgreich zugestellt wird (success: true in Response) **When** die Response angezeigt wird **Then** sehe ich: "Test webhook delivered successfully", Response Status (z.B. "200 OK"), und die Webhook URL **And** ein Toast: "Test webhook sent — 200 OK" (grün, 3s auto-dismiss)

3. **Given** der Test-Webhook fehlschlägt (success: false in Response) **When** die Response angezeigt wird **Then** sehe ich: "Webhook delivery failed", Response Status (z.B. "500 Internal Server Error") oder Error Message (z.B. "Connection refused"), und einen hilfreichen Hint: "Check that your endpoint is reachable and returns 2xx." **And** ein Toast: "Test webhook failed" (rot, bleibt bis dismissed)

4. **Given** keine Webhook-URL konfiguriert ist **When** die Settings-Seite angezeigt wird **Then** ist der "Send Test Webhook"-Button disabled **And** kein Test kann gesendet werden

5. **Given** ein Delivery Log auf der Settings-Seite angezeigt werden soll (FR22) **When** der Delivery Log geladen wird **Then** werden die letzten N Payments abgerufen und deren Webhook-Histories via GET /v1/payments/:id/webhooks aggregiert **And** jede Zeile zeigt: Event Name (z.B. `payment.confirmed`), Status Code (farbig: 2xx=grün, 4xx/5xx=rot, null=grau), Timestamp, und Attempt-Nummer **And** HINWEIS: Es gibt keinen globalen Webhook-Delivery-Log-Endpoint — der Log wird pro-Payment via GET /v1/payments/:id/webhooks zusammengestellt

6. **Given** der Delivery Log leer ist **When** die Liste angezeigt wird **Then** sehe ich: "No webhooks sent yet."

7. **Given** die Delivery Log Daten laden **When** die API-Response noch aussteht **Then** sehe ich Skeleton-Zeilen als Loading-State

## Tasks / Subtasks

- [x] Task 1: Test-Webhook Proxy Route erstellen (AC: #1)
  - [x] 1.1 `src/app/api/webhooks/test/route.ts` — POST Handler: `requireAuth(request)` → `mmsApi("/v1/paywatcher/config/test-webhook", { method: "POST" }, apiKey)` → Response (success, response_code, error). snakeToCamel Transformation auf Response anwenden
  - [x] 1.2 MMS Response Type definieren: `MmsTestWebhookResponse` in `src/lib/api/types.ts` — `{ success: boolean, response_code: number | null, error: string | null }`. Frontend Type: `TestWebhookResult` in `src/types/tenant.ts` — `{ success: boolean, responseCode: number | null, error: string | null }`

- [x] Task 2: useTestWebhook Mutation Hook erstellen (AC: #1, #2, #3)
  - [x] 2.1 `src/hooks/use-test-webhook.ts` — TanStack Query `useMutation`: POST `/api/webhooks/test`, Return `TestWebhookResult`. Error-Handling via `parseApiError()`

- [x] Task 3: WebhookUrlForm um Test-Webhook-Funktionalität erweitern (AC: #1, #2, #3, #4)
  - [x] 3.1 `src/components/dashboard/webhook-url-form.tsx` — ÄNDERN: Den bestehenden placeholder `toast.info("Coming soon — Story 4.2")` durch echte `useTestWebhook()` Mutation ersetzen. Button zeigt Loading-State (`isPending`). Bei Erfolg: `toast.success("Test webhook sent — {responseCode}")`. Bei Fehler (success: false): `toast.error("Test webhook failed", { duration: Infinity })`. Bei Netzwerk-Fehler: `toast.error(error.message, { duration: Infinity })`
  - [x] 3.2 Test-Ergebnis-Anzeige unterhalb des Buttons: Erfolgreich → grüner Text "Test webhook delivered successfully" + Response Status + Webhook URL. Fehlgeschlagen → roter Text "Webhook delivery failed" + Error Message + Hint "Check that your endpoint is reachable and returns 2xx." Kein Ergebnis → nichts anzeigen. State wird als lokaler React State gehalten (`testResult: TestWebhookResult | null`)

- [x] Task 4: Webhook Delivery Log Aggregation implementieren (AC: #5, #6, #7)
  - [x] 4.1 `src/hooks/use-delivery-log.ts` — NEU: Custom Hook der die letzten Payments abruft (via `usePayments({ limit: 10, sort: "created_at", order: "desc" })`) und parallel deren Webhook-Histories lädt (via parallel fetches zu `/api/payments/{id}/webhooks`). Aggregiert alle Webhook-Deliveries, sortiert nach Timestamp (neueste zuerst). Return: `{ data: WebhookDelivery[], isLoading: boolean }`. HINWEIS: Es gibt keinen globalen Delivery-Log-Endpoint — wir aggregieren pro-Payment
  - [x] 4.2 Erweitere `WebhookDelivery` Type in `src/types/payment.ts`: Optionales Feld `paymentId?: string` hinzufügen, damit im aggregierten Log erkennbar ist zu welchem Payment ein Webhook gehört

- [x] Task 5: Delivery Log Section auf Webhooks-Seite hinzufügen (AC: #5, #6, #7)
  - [x] 5.1 `src/app/(dashboard)/dashboard/settings/webhooks/page.tsx` — ERWEITERN: Unterhalb des WebhookUrlForm eine "Delivery Log" Section mit Heading "Recent Webhook Deliveries" und der bestehenden `WebhookHistory` Komponente (aus Story 3.5, `src/components/dashboard/webhook-history.tsx`) anzeigen. Daten kommen aus dem neuen `useDeliveryLog()` Hook
  - [x] 5.2 Skeleton-Loading: `WebhookHistorySkeleton` (bereits vorhanden) während Daten laden
  - [x] 5.3 Empty State: "No webhooks sent yet." (via bestehende WebhookHistory Komponente, die diesen State bereits handelt)

- [x] Task 6: Build-Validierung
  - [x] 6.1 `npm run build` fehlerfrei
  - [x] 6.2 "Send Test Webhook" Button: disabled ohne Webhook-URL, aktiv mit URL
  - [x] 6.3 Test-Webhook senden: Erfolg → grüner Toast + Ergebnis-Anzeige
  - [x] 6.4 Test-Webhook senden: Fehler → roter Toast + Fehler-Anzeige mit Hint
  - [x] 6.5 Delivery Log zeigt aggregierte Webhook-Histories der letzten Payments
  - [x] 6.6 Delivery Log: Skeleton-Loading korrekt
  - [x] 6.7 Delivery Log: Empty State "No webhooks sent yet." wenn keine Webhooks
  - [x] 6.8 Mobile: Layout einspaltig, Touch Targets min. 44x44px

## Dev Notes

### Kontext & Zweck

Dies ist **Story 4.2 von Epic 4** (Webhook Configuration & Tenant Settings) — die **zweite Story des Epics**. Sie baut direkt auf Story 4.1 auf, die das Settings-Layout, die Tab-Navigation und das Webhook-URL-Formular etabliert hat. Story 4.2 aktiviert den "Send Test Webhook"-Button (bisher disabled/placeholder) und fügt den Webhook Delivery Log auf der Settings-Seite hinzu.

**Was diese Story beinhaltet:**
- Test-Webhook senden (POST /v1/paywatcher/config/test-webhook)
- Test-Ergebnis-Anzeige (Erfolg/Fehler)
- Webhook Delivery Log auf der Settings/Webhooks-Seite (aggregiert aus pro-Payment Webhook-Histories)

**Was diese Story NICHT beinhaltet:**
- Confirmation Override (Story 4.3)
- Deposit Address Anzeige (Story 4.3)
- Account-Einstellungen (Story 4.3)

### KRITISCH: Test-Webhook Proxy Route — NEUEN Endpoint erstellen

Die Architecture Doc zeigt `app/api/config/test-webhook/route.ts`, aber der bestehende Code nutzt `/api/tenant/route.ts` für Config. Für den Test-Webhook wird ein **separater Endpoint** erstellt unter `/api/webhooks/test/route.ts`, weil:
- POST /v1/paywatcher/config/test-webhook ist ein eigener MMS-Endpoint mit eigener Response-Struktur
- Der bestehende `/api/tenant/route.ts` handelt nur GET/PATCH für die Config
- Semantisch ist "Test Webhook senden" eine andere Aktion als "Config lesen/ändern"

```typescript
// src/app/api/webhooks/test/route.ts
export async function POST(request: Request) {
  const result = await requireAuth(request)
  if ("error" in result) return result.error

  const response = await mmsApi<MmsApiResponse<MmsTestWebhookResponse>>(
    "/v1/paywatcher/config/test-webhook",
    { method: "POST" },
    result.apiKey
  )
  return NextResponse.json({ data: snakeToCamel(response.data) })
}
```

### KRITISCH: MMS Test-Webhook Response-Struktur

Der MMS-Endpoint `POST /v1/paywatcher/config/test-webhook` gibt zurück:
```json
{
  "data": {
    "success": true,
    "response_code": 200,
    "error": null
  }
}
```

Bei Fehler:
```json
{
  "data": {
    "success": false,
    "response_code": 500,
    "error": "Internal Server Error"
  }
}
```

Oder bei Connection Error:
```json
{
  "data": {
    "success": false,
    "response_code": null,
    "error": "Connection refused"
  }
}
```

**WICHTIG:** Die Response ist IMMER im `{ data: ... }` Envelope — auch bei fehlgeschlagener Webhook-Zustellung. Das ist kein HTTP-Error des Proxy-Calls, sondern das Ergebnis der Webhook-Zustellung. HTTP 200 vom MMS mit `success: false` in der Response.

### KRITISCH: Webhook Delivery Log — Aggregation aus pro-Payment Webhook-Histories

Es gibt **keinen globalen Webhook-Delivery-Log-Endpoint** in der MMS API. Der Delivery Log wird zusammengestellt aus:

1. Die letzten N Payments abrufen: `GET /api/payments?limit=10&sort=created_at&order=desc`
2. Für jedes Payment die Webhook-History abrufen: `GET /api/payments/{id}/webhooks`
3. Alle Webhook-Deliveries zusammenführen und nach Timestamp sortieren (neueste zuerst)

**Implementation-Strategie:**
- `useDeliveryLog()` Hook orchestriert die Aggregation
- Verwendet `usePayments()` für die Payment-Liste
- Verwendet `Promise.all()` mit parallel fetches für die Webhook-Histories
- Caching via TanStack Query (staleTime 30s wie bestehende Webhook-Hooks)
- Jede `WebhookDelivery` wird um `paymentId` angereichert, damit im Log erkennbar ist zu welchem Payment ein Webhook gehört

**ACHTUNG: Performance-Überlegung:**
- Maximal 10 Payments × 1 API-Call pro Payment = 11 API-Calls total
- Die API-Calls sind parallel und gehen an den Proxy (nicht direkt MMS) — akzeptable Latenz
- `limit: 10` begrenzt die Anzahl der Payments — mehr ist für einen Settings-Log nicht nötig

### KRITISCH: Bestehende WebhookHistory Komponente wiederverwenden

Die `WebhookHistory` Komponente aus Story 3.5 (`src/components/dashboard/webhook-history.tsx`) kann 1:1 wiederverwendet werden für den Delivery Log. Sie zeigt bereits:
- Event Name, Attempt Number, Response Code (farbig), Error, Timestamp
- Desktop Table + Mobile Cards Layout
- Empty State: "No webhooks sent yet."
- Skeleton Loading via `WebhookHistorySkeleton`

**Keine Änderung an der bestehenden Komponente nötig** — sie nimmt ein `WebhookDelivery[]` Array als Props.

### KRITISCH: WebhookUrlForm Änderung — Placeholder ersetzen

Der bestehende Code in `webhook-url-form.tsx` (Zeile 93-102) hat einen Placeholder-Button:

```typescript
<Button
  type="button"
  variant="outline"
  disabled={!webhookUrl}
  onClick={() => {
    toast.info("Coming soon — Test Webhook (Story 4.2)")
  }}
>
  Send Test Webhook
</Button>
```

Dieser wird ersetzt durch:
```typescript
<Button
  type="button"
  variant="outline"
  disabled={!webhookUrl || testWebhook.isPending}
  onClick={() => testWebhook.mutate()}
>
  {testWebhook.isPending ? "Sending…" : "Send Test Webhook"}
</Button>
```

Plus: Test-Ergebnis-Anzeige unterhalb der Buttons (lokaler State `testResult`).

### Bestehender Code-Stand (aus Story 4.1)

**Bereits vorhanden — WIEDERVERWENDEN, nicht neu erstellen:**

| Datei | Was existiert | Aktion |
|-------|-------------|--------|
| `src/components/dashboard/webhook-url-form.tsx` | Webhook URL Formular mit Placeholder-Button | **ÄNDERN** — Placeholder durch echte Test-Funktionalität ersetzen |
| `src/components/dashboard/webhook-history.tsx` | WebhookHistory + WebhookHistorySkeleton | **WIEDERVERWENDEN** — für Delivery Log Section |
| `src/hooks/use-webhook-deliveries.ts` | `useWebhookDeliveries(paymentId)` | **REFERENZ** — Pattern für neue Hooks |
| `src/hooks/use-update-config.ts` | `useUpdateConfig()` Mutation | **REFERENZ** — Pattern für useTestWebhook |
| `src/hooks/use-payments.ts` | `usePayments(filters)` | **VERWENDEN** — für Delivery Log Aggregation |
| `src/hooks/use-tenant.ts` | `useTenant()` | **VERWENDEN** — für webhookUrl Check |
| `src/app/api/tenant/route.ts` | GET/PATCH /v1/paywatcher/config | **UNVERÄNDERT** |
| `src/app/api/payments/[id]/webhooks/route.ts` | GET /v1/payments/:id/webhooks Proxy | **VERWENDEN** — für Delivery Log |
| `src/app/(dashboard)/dashboard/settings/webhooks/page.tsx` | Webhooks Page mit WebhookUrlForm | **ERWEITERN** — Delivery Log Section hinzufügen |
| `src/types/payment.ts` | `WebhookDelivery` Interface | **ERWEITERN** — optionales `paymentId` Feld |
| `src/types/tenant.ts` | `TenantConfig` Interface | **ERWEITERN** — `TestWebhookResult` Type hinzufügen |
| `src/lib/api/types.ts` | MMS Response Types | **ERWEITERN** — `MmsTestWebhookResponse` hinzufügen |
| `src/lib/api/client.ts` | `mmsApi()` Fetch-Wrapper | **VERWENDEN** |
| `src/lib/api/transform.ts` | `snakeToCamel()` | **VERWENDEN** |
| `src/lib/auth.ts` | `requireAuth()` | **VERWENDEN** |
| `src/lib/api/errors.ts` | `MmsApiError` | **VERWENDEN** |
| `src/lib/api/client-error.ts` | `parseApiError()` | **VERWENDEN** |

### Neue Dateien die erstellt werden:

```
src/
  app/
    api/webhooks/test/
      route.ts                                    # NEU: POST Proxy für Test-Webhook
  hooks/
    use-test-webhook.ts                            # NEU: TanStack Query Mutation Hook
    use-delivery-log.ts                            # NEU: Aggregation Hook für Delivery Log
```

### Bestehende Dateien die geändert werden:

```
src/
  components/dashboard/
    webhook-url-form.tsx                            # ÄNDERN: Placeholder → echte Test-Funktionalität + Ergebnis-Anzeige
  app/(dashboard)/dashboard/settings/webhooks/
    page.tsx                                       # ERWEITERN: Delivery Log Section hinzufügen
  types/
    payment.ts                                     # ERWEITERN: paymentId zu WebhookDelivery
    tenant.ts                                      # ERWEITERN: TestWebhookResult Type
  lib/api/
    types.ts                                       # ERWEITERN: MmsTestWebhookResponse
```

### Learnings aus Story 4.1 (Previous Story Intelligence)

- **PATCH Route Pattern:** `requireAuth(request)` → Zod-Validierung → `mmsApi()` → `snakeToCamel()` → Envelope Response. POST Route für Test-Webhook folgt demselben Pattern (ohne Zod, da kein Body nötig).
- **Mutation Hook Pattern:** `useMutation` mit `parseApiError()` für Error-Handling. Test-Webhook Hook folgt exakt demselben Pattern wie `useUpdateConfig()`.
- **Form-Zustand:** React Hook Form für URL-Validierung. Test-Ergebnis wird als separater lokaler State gehalten (nicht Teil des Formulars).
- **Toast-Pattern:** Erfolg = `toast.success()` mit 3s auto-dismiss. Fehler = `toast.error()` mit `duration: Infinity`. Test-Webhook folgt demselben Pattern.
- **Settings Layout:** Bereits vorhanden mit max-w-3xl und Tab-Navigation — kein neues Layout nötig.
- **react-hook-form, @hookform/resolvers, label:** Bereits installiert (Story 4.1 Task 1).
- **Prefix-Match für Tab-Navigation:** Settings-Layout nutzt `pathname.startsWith(tab.href)` für active state — neue Sub-Inhalte auf der Webhooks-Seite sind automatisch korrekt hervorgehoben.

### Git Intelligence (letzte 5 Commits)

1. `661de6f` — Add Settings Layout & Webhook URL Configuration with review fixes (Story 4.1)
2. `d275e8e` — Add Payment Detail & Webhook History with review fixes (Story 3.5)
3. `d980a47` — Add Payment Filters, Sorting & Pagination with review fixes (Story 3.4)
4. `8321838` — Add Payments List Page with DataTable, Card-Layout & review fixes (Story 3.3)
5. `86f1437` — Add Overview Page with state-dependent first screen, StatCards, HealthBar (Story 3.2)

**Relevantes Pattern:** Commit-Messages als "Add [Feature] with review fixes (Story X.Y)". Neue Proxy-Routen in `app/api/`, neue Hooks in `hooks/`, Komponenten-Änderungen in `components/dashboard/`.

### Project Structure Notes

- `/api/webhooks/test/route.ts` — neuer Proxy-Endpoint, konsistent mit Architecture Doc Route `app/api/config/test-webhook/route.ts` aber unter `/api/webhooks/test/` für bessere semantische Trennung
- `hooks/use-test-webhook.ts` — Mutation Hook, folgt Naming-Convention `use-{action}.ts`
- `hooks/use-delivery-log.ts` — Aggregation Hook, orchestriert mehrere Queries
- Bestehende Webhooks-Seite wird erweitert, nicht neu erstellt
- Bestehende WebhookHistory Komponente wird wiederverwendet

### Architektur-Compliance

- Dateien in `kebab-case.tsx` — `use-test-webhook.ts`, `use-delivery-log.ts`
- Route Handler Response im Envelope-Format `{ data }` / `{ error }`
- snake_case → camelCase Transformation im Route Handler (für MMS Response)
- TanStack Query: Mutation für Test-Webhook, Query für Delivery Log Aggregation
- Kein `any` Type
- Kein Full-Page-Spinner — Skeleton-Loading für Delivery Log
- Toast via sonner: Erfolg 3s auto-dismiss, Fehler bleibt
- Max 1 Primary Button pro Screen ("Save" = Primary, "Send Test Webhook" = outline/Secondary)
- Webhook History Komponente wiederverwendet (DRY)
- MMS-Calls ausschließlich über `mmsApi()` in Route Handlers

### References

- [Source: _bmad-output/planning-artifacts/epics.md#Story 4.2]
- [Source: _bmad-output/planning-artifacts/architecture.md#API Communication Pattern]
- [Source: _bmad-output/planning-artifacts/architecture.md#App Structure & Routing — config/test-webhook/route.ts]
- [Source: _bmad-output/planning-artifacts/architecture.md#Component Patterns — Toast Pattern]
- [Source: _bmad-output/planning-artifacts/architecture.md#TanStack Query Patterns — Mutations]
- [Source: _bmad-output/planning-artifacts/architecture.md#FR-Kategorie: Webhook Management (FR20-FR22)]
- [Source: _bmad-output/implementation-artifacts/4-1-settings-layout-webhook-url-configuration.md — Previous Story]
- [Source: src/components/dashboard/webhook-url-form.tsx — Bestehender Placeholder-Button Zeile 93-102]
- [Source: src/components/dashboard/webhook-history.tsx — Wiederverwendbare Webhook-History Komponente]
- [Source: src/hooks/use-webhook-deliveries.ts — Pattern für Webhook-Fetch]
- [Source: src/hooks/use-update-config.ts — Pattern für Mutation Hook]
- [Source: src/hooks/use-payments.ts — Pattern für Payment-Liste]
- [Source: src/app/api/tenant/route.ts — Proxy Pattern Referenz]
- [Source: src/app/api/payments/[id]/webhooks/route.ts — Webhook Proxy Referenz]
- [Source: src/types/payment.ts — WebhookDelivery Interface]
- [Source: src/types/tenant.ts — TenantConfig Interface]
- [Source: src/lib/api/types.ts — MMS Response Types]

## Dev Agent Record

### Agent Model Used

Claude Opus 4.6

### Debug Log References

Keine Debug-Issues aufgetreten. Build erfolgreich beim ersten Versuch.

### Completion Notes List

- Task 1: POST Proxy Route `/api/webhooks/test` erstellt. Folgt exakt dem bestehenden Pattern aus `/api/tenant/route.ts` mit `requireAuth` → `mmsApi` → `snakeToCamel` → Envelope Response. `MmsTestWebhookResponse` und `TestWebhookResult` Types definiert.
- Task 2: `useTestWebhook` Mutation Hook erstellt. Folgt dem `useUpdateConfig` Pattern mit `useMutation` und `parseApiError`.
- Task 3: `WebhookUrlForm` Placeholder-Button ersetzt durch echte Test-Webhook-Funktionalität. Loading-State, Success/Error Toasts (3s bzw. Infinity), und Test-Ergebnis-Anzeige (`TestResultDisplay` Komponente) implementiert. Lokaler State `testResult` für Ergebnis-Anzeige.
- Task 4: `useDeliveryLog` Aggregation Hook erstellt. Lädt letzte 10 Payments via `usePayments`, fetcht parallel Webhook-Histories, sortiert nach Timestamp. `WebhookDelivery` Type um optionales `paymentId` erweitert.
- Task 5: Webhooks-Seite erweitert mit "Recent Webhook Deliveries" Section. Verwendet bestehende `WebhookHistory` und `WebhookHistorySkeleton` Komponenten — keine Änderungen an diesen nötig.
- Task 6: Build erfolgreich. Alle AC-Kriterien durch Code-Analyse verifiziert.

### File List

**Neue Dateien:**
- src/app/api/webhooks/test/route.ts
- src/hooks/use-test-webhook.ts
- src/hooks/use-delivery-log.ts

**Geänderte Dateien:**
- src/lib/api/types.ts (MmsTestWebhookResponse hinzugefügt)
- src/types/tenant.ts (TestWebhookResult hinzugefügt)
- src/types/payment.ts (paymentId zu WebhookDelivery hinzugefügt)
- src/components/dashboard/webhook-url-form.tsx (Placeholder → echte Test-Funktionalität + Ergebnis-Anzeige)
- src/app/(dashboard)/dashboard/settings/webhooks/page.tsx (Delivery Log Section hinzugefügt)

### Change Log

- 2026-02-20: Story 4.2 implementiert — Test-Webhook senden (Proxy Route + Mutation Hook + UI), Delivery Log Aggregation (Custom Hook + Settings-Page Section). Build fehlerfrei.
- 2026-02-20: Code Review — 5 Fixes applied (2 HIGH, 3 MEDIUM): H1 Error-Display zeigt jetzt responseCode UND error, H2 Promise.allSettled statt Promise.all für Resilienz, M1+M2 HTTP Status-Text Mapping für Toast/Display ("200 OK"), M3 useDeliveryLog exponiert error/isError State. Build fehlerfrei.
