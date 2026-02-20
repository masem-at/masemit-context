# Story 4.3: Tenant Configuration & Account Settings

Status: done

<!-- Note: Validation is optional. Run validate-create-story for quality check before dev-story. -->

## Story

As a Tenant,
I want meine Confirmation-Anzahl anpassen, meine Deposit Address sehen, und meine Account-Einstellungen verwalten,
So that ich PayWatcher an meine Bedürfnisse anpassen und meine Kontodaten einsehen kann.

## Acceptance Criteria

1. **Given** ich bin im Account-Bereich der Settings (FR23) **When** die Confirmation-Override-Einstellung angezeigt wird **Then** sehe ich die aktuelle Default-Confirmation-Anzahl (Standard: 6, aus GET /v1/paywatcher/config → confirmationOverride Feld) **And** ich kann den Wert über ein Zahlenfeld zwischen 2 und 20 anpassen **And** eine kurze Erklärung ist sichtbar: "Number of block confirmations required before a payment is marked as confirmed."

2. **Given** ich ändere die Confirmation-Anzahl **When** ich auf "Save" klicke **Then** wird der Wert via Proxy an PATCH /v1/paywatcher/config mit `{ confirmation_override: value }` gesendet **And** Zod validiert: Ganzzahl, min 2, max 20 (oder null für Default) **And** ein Toast erscheint: "Settings updated" (3s auto-dismiss)

3. **Given** ich bin im Account-Bereich der Settings (FR24) **When** die Deposit Address angezeigt wird **Then** sehe ich meine Deposit Address in Geist Mono (read-only, nicht editierbar, aus GET /v1/paywatcher/config → depositAddress Feld) **And** ein Copy-Button neben der Adresse kopiert sie in die Zwischenablage (Inline-Checkmark, kein Toast) **And** die Adresse ist als Link zu Basescan klickbar (öffnet in neuem Tab, NFR14)

4. **Given** ich bin im Account-Bereich der Settings (FR25) **When** meine Account-Einstellungen angezeigt werden **Then** sehe ich meine Email-Adresse (contactEmail aus Config, read-only) **And** ein Logout-Button ist verfügbar (konsistent mit bestehendem Logout in Dashboard Header)

5. **Given** die Settings-Daten laden **When** die API-Response noch aussteht **Then** sehe ich Skeleton-Placeholders für die Felder **And** Sidebar + Header stehen sofort

6. **Given** die Settings auf Mobile (<1024px) angezeigt werden **When** die Seite rendert **Then** ist das Layout einspaltig, Formulare nutzen volle Breite, alle Touch Targets min. 44x44px

## Tasks / Subtasks

- [x] Task 1: PATCH Config Proxy Route erweitern — confirmationOverride zum Zod-Schema hinzufügen (AC: #2)
  - [x] 1.1 `src/app/api/tenant/route.ts` — ÄNDERN: `patchConfigSchema` erweitern um `confirmationOverride: z.number().int().min(2).max(20).nullable().optional()`. Aktuell akzeptiert das Schema nur `webhookUrl`
  - [x] 1.2 Transformation im PATCH-Handler erweitern: `if (parsed.data.confirmationOverride !== undefined) { mmsBody.confirmation_override = parsed.data.confirmationOverride }`. Behandlung von `null` (Reset auf Default) sicherstellen

- [x] Task 2: Account Settings Formular-Komponente erstellen (AC: #1, #2, #5, #6)
  - [x] 2.1 `src/components/dashboard/account-settings-form.tsx` — NEU: React Hook Form + Zod Komponente:
    - **Confirmation Override Section:** Label "Block Confirmations", Number Input (min=2, max=20, step=1), Hilfetext "Number of block confirmations required before a payment is marked as confirmed.", Default-Anzeige wenn confirmationOverride null ist (zeigt `confirmations` Wert als Platzhalter). "Save" Button (Primary, max 1 pro Screen)
    - **Zod Schema:** `confirmationOverrideSchema = z.object({ confirmationOverride: z.coerce.number().int().min(2).max(20) })`. HINWEIS: `z.coerce.number()` nötig weil HTML number inputs String-Werte liefern
    - Bei Erfolg: `toast.success("Settings updated")` (3s auto-dismiss). Bei Fehler: `toast.error(error.message, { duration: Infinity })`
    - Mutation via `useUpdateConfig()` (bestehend, unterstützt bereits `confirmationOverride`)
  - [x] 2.2 `AccountSettingsFormSkeleton` — Skeleton-Variante: Skeleton-Blocks für Input-Felder + Buttons

- [x] Task 3: Deposit Address Anzeige-Komponente erstellen (AC: #3, #5)
  - [x] 3.1 `src/components/dashboard/deposit-address-card.tsx` — NEU: Read-only Card:
    - Label "Deposit Address"
    - Adresse in `font-mono` (Geist Mono) dargestellt
    - Copy-Button (Inline-Checkmark, kein Toast) — bestehenden `InlineCopyButton` Komponente verwenden (`src/components/shared/inline-copy-button.tsx`)
    - Basescan-Link: `https://basescan.org/address/{depositAddress}` → öffnet in neuem Tab (NFR14)
    - Hilfetext: "Your shared USDC deposit address on Base network. Payments are identified by unique amounts."
  - [x] 3.2 `DepositAddressCardSkeleton` — Skeleton-Variante

- [x] Task 4: Account Info Section erstellen (AC: #4)
  - [x] 4.1 Innerhalb der Account-Seite: Email-Anzeige (read-only) mit Label "Email" und Wert `contactEmail` aus `useTenant()`. Darstellung als text-muted-foreground
  - [x] 4.2 Logout-Button: `variant="outline"` (Secondary, nicht Primary — max 1 Primary pro Screen für Save). onClick: `signOut({ callbackUrl: "/" })` (Auth.js v5). Import: `import { signOut } from "next-auth/react"`

- [x] Task 5: Account Settings Page zusammenbauen (AC: #1-#6)
  - [x] 5.1 `src/app/(dashboard)/dashboard/settings/account/page.tsx` — ERSETZEN: Placeholder durch echte Page. `"use client"`, verwendet `useTenant()` für Config-Daten. Layout: Drei Sections vertikal gestackt mit `gap-8`:
    1. **Block Confirmations** — `AccountSettingsForm` Komponente
    2. **Deposit Address** — `DepositAddressCard` Komponente
    3. **Account** — Email-Anzeige + Logout-Button
  - [x] 5.2 Loading State: Wenn `useTenant()` lädt → `AccountSettingsFormSkeleton` + `DepositAddressCardSkeleton` + Skeleton für Email
  - [x] 5.3 Error State: Wenn `useTenant()` Error → Error-Anzeige mit Retry-Möglichkeit

- [x] Task 6: Build-Validierung
  - [x] 6.1 `npm run build` fehlerfrei
  - [x] 6.2 Account Settings Tab zeigt Confirmation Override mit aktuellem Wert
  - [x] 6.3 Confirmation Override ändern (2-20) → Toast "Settings updated"
  - [x] 6.4 Confirmation Override Validierung: < 2 oder > 20 zeigt Inline-Error
  - [x] 6.5 Deposit Address in Geist Mono angezeigt, Copy-Button funktioniert (Inline-Checkmark)
  - [x] 6.6 Basescan-Link öffnet in neuem Tab
  - [x] 6.7 Email (read-only) korrekt angezeigt
  - [x] 6.8 Logout-Button funktioniert (Session löschen, Redirect zur Landing Page)
  - [x] 6.9 Skeleton-Loading für alle Sections korrekt
  - [x] 6.10 Mobile: Layout einspaltig, volle Breite, Touch Targets min. 44x44px
  - [x] 6.11 Bestehende Webhooks + API Keys Pages funktionieren weiterhin

## Dev Notes

### Kontext & Zweck

Dies ist **Story 4.3 von Epic 4** (Webhook Configuration & Tenant Settings) — die **dritte und letzte Story des Epics**. Sie vervollständigt die Settings-Seite mit den Account-bezogenen Einstellungen. Stories 4.1 und 4.2 haben bereits das Settings-Layout, die Tab-Navigation, das Webhook-URL-Formular, den Test-Webhook-Button und den Delivery Log etabliert.

**Was diese Story beinhaltet:**
- Confirmation Override (FR23): Zahlenfeld 2-20 mit PATCH /v1/paywatcher/config
- Deposit Address (FR24): Read-only Anzeige mit Copy + Basescan-Link
- Account-Einstellungen (FR25): Email-Anzeige (read-only) + Logout-Button

**Was diese Story NICHT beinhaltet:**
- Webhook URL Konfiguration (Story 4.1 — fertig)
- Test Webhook & Delivery Log (Story 4.2 — fertig)
- Tenant-Erstellung oder -Verwaltung (Admin-Funktion)

### KRITISCH: PATCH Route Schema erweitern — confirmationOverride hinzufügen

Die PATCH Route (`src/app/api/tenant/route.ts`, Zeile 9-11) hat aktuell ein striktes Zod-Schema das NUR `webhookUrl` akzeptiert:

```typescript
// AKTUELL (Zeile 9-11):
const patchConfigSchema = z.object({
  webhookUrl: z.string().url().optional(),
}).strict()
```

Muss erweitert werden zu:

```typescript
// NEU:
const patchConfigSchema = z.object({
  webhookUrl: z.string().url().optional(),
  confirmationOverride: z.number().int().min(2).max(20).nullable().optional(),
}).strict()
```

Und die Transformation im PATCH-Handler (Zeile 54-56) muss erweitert werden:

```typescript
// AKTUELL (Zeile 54-56):
if (parsed.data.webhookUrl !== undefined) {
  mmsBody.webhook_url = parsed.data.webhookUrl
}

// NEU — HINZUFÜGEN:
if (parsed.data.confirmationOverride !== undefined) {
  mmsBody.confirmation_override = parsed.data.confirmationOverride
}
```

**HINWEIS:** `confirmationOverride: null` setzt den Override zurück auf den System-Default (6 Confirmations). Der MMS-Endpoint akzeptiert `null` als Wert für `confirmation_override`.

### KRITISCH: useUpdateConfig bereits vorbereitet

Der `useUpdateConfig()` Hook (`src/hooks/use-update-config.ts`) unterstützt BEREITS `confirmationOverride` im `UpdateConfigPayload` Interface (Zeile 8-9):

```typescript
interface UpdateConfigPayload {
  webhookUrl?: string
  confirmationOverride?: number | null
}
```

**Keine Änderung am Hook nötig** — nur die PATCH Route (Server-seitig) muss das Feld akzeptieren.

### KRITISCH: TenantConfig hat bereits alle nötigen Felder

Das `TenantConfig` Interface (`src/types/tenant.ts`) enthält BEREITS:
- `confirmationOverride: number | null` — für Confirmation Override
- `confirmations: number` — System-Default (6), als Fallback wenn Override null
- `depositAddress: string` — für Deposit Address Anzeige
- `contactEmail: string` — für Email-Anzeige
- `tenantName: string` — für Account-Name

**Keine Änderung an Types nötig.**

### KRITISCH: Number Input Validierung — z.coerce.number()

HTML `<input type="number">` liefert String-Werte in onChange Events. React Hook Form gibt den Wert als String weiter. Daher MUSS das Zod-Schema `z.coerce.number()` verwenden statt `z.number()`:

```typescript
const confirmationOverrideSchema = z.object({
  confirmationOverride: z.coerce.number().int().min(2, "Minimum 2 confirmations").max(20, "Maximum 20 confirmations"),
})
```

**OHNE `z.coerce`:** Zod rejectet den String-Wert und zeigt "Expected number, received string".

### KRITISCH: Deposit Address — Basescan Link Format

Base Network Explorer: `https://basescan.org/address/{address}`

```typescript
const basescanUrl = `https://basescan.org/address/${depositAddress}`
// Link öffnet in neuem Tab:
<a href={basescanUrl} target="_blank" rel="noopener noreferrer">
```

### KRITISCH: Logout Pattern — Auth.js v5

```typescript
import { signOut } from "next-auth/react"

// onClick Handler:
onClick={() => signOut({ callbackUrl: "/" })}
```

Der bestehende Dashboard-Header (`src/components/dashboard/dashboard-header.tsx`) hat bereits einen Logout-Mechanismus — prüfe ob `signOut` dort schon verwendet wird und folge dem gleichen Pattern.

### KRITISCH: Copy Button — useCopy Hook verwenden

Der bestehende `useCopy()` Hook (`src/hooks/use-copy.ts`) bietet Clipboard-Copy mit Inline-Checkmark-Feedback. Bereits verwendet in:
- `src/components/dashboard/api-key-card.tsx`
- `src/components/shared/copy-button.tsx`
- `src/components/dashboard/payment-detail-view.tsx`

Pattern:
```typescript
import { useCopy } from "@/hooks/use-copy"

const { copied, copy } = useCopy()

<Button variant="ghost" size="icon" onClick={() => copy(depositAddress)}>
  {copied ? <Check className="h-4 w-4" /> : <Copy className="h-4 w-4" />}
</Button>
```

### Bestehender Code-Stand (aus Stories 4.1 + 4.2)

**Bereits vorhanden — WIEDERVERWENDEN, nicht neu erstellen:**

| Datei | Was existiert | Aktion |
|-------|-------------|--------|
| `src/app/(dashboard)/dashboard/settings/layout.tsx` | Settings Layout mit Tab-Navigation (Webhooks, API Keys, Account) | **UNVERÄNDERT** |
| `src/app/(dashboard)/dashboard/settings/account/page.tsx` | Placeholder "Coming in Story 4.3" | **ERSETZEN** — durch echte Account Page |
| `src/app/api/tenant/route.ts` | GET + PATCH /v1/paywatcher/config (Zod: nur webhookUrl) | **ÄNDERN** — confirmationOverride zu Zod-Schema + Transformation |
| `src/hooks/use-tenant.ts` | `useTenant()` mit staleTime 5min | **VERWENDEN** — für Config-Daten |
| `src/hooks/use-update-config.ts` | `useUpdateConfig()` mit confirmationOverride Support | **VERWENDEN** — für Save-Mutation |
| `src/hooks/use-copy.ts` | `useCopy()` für Clipboard + Checkmark | **VERWENDEN** — für Deposit Address Copy |
| `src/types/tenant.ts` | `TenantConfig` (alle Felder vorhanden) | **UNVERÄNDERT** |
| `src/lib/api/types.ts` | `MmsTenantConfig` (snake_case) | **UNVERÄNDERT** |
| `src/lib/api/client.ts` | `mmsApi()` | **UNVERÄNDERT** |
| `src/lib/api/transform.ts` | `snakeToCamel()` | **UNVERÄNDERT** |
| `src/lib/auth.ts` | `requireAuth()` | **UNVERÄNDERT** |
| `src/lib/api/errors.ts` | `MmsApiError` | **UNVERÄNDERT** |
| `src/lib/api/client-error.ts` | `parseApiError()` | **UNVERÄNDERT** |
| `src/components/ui/*` | shadcn/ui: Button, Input, Card, Skeleton, Label | **VERWENDEN** |
| `src/components/dashboard/webhook-url-form.tsx` | React Hook Form + Zod Pattern Referenz | **REFERENZ** — für Account Settings Form |

### Neue Dateien die erstellt werden:

```
src/
  components/dashboard/
    account-settings-form.tsx                    # NEU: Confirmation Override Formular
    deposit-address-card.tsx                     # NEU: Deposit Address Anzeige
```

### Bestehende Dateien die geändert werden:

```
src/
  app/
    api/tenant/
      route.ts                                   # ÄNDERN: confirmationOverride zu Zod + Transformation
    (dashboard)/dashboard/settings/account/
      page.tsx                                   # ERSETZEN: Placeholder → echte Account Page
```

### Learnings aus Stories 4.1 + 4.2 (Previous Story Intelligence)

- **React Hook Form Pattern:** `useForm({ resolver: zodResolver(schema), values: { ... } })` — `values` Prop synchronisiert wenn tenant-Daten laden (kein useEffect + reset nötig)
- **Mutation Pattern:** `useUpdateConfig()` mit `toast.success()` (3s) / `toast.error()` (Infinity). Invalidiert `["tenant"]` Query Key automatisch
- **PATCH Route Pattern:** `requireAuth(request)` → Zod-Validierung → camelCase → snake_case Transformation → `mmsApi()` → `snakeToCamel()` → Envelope Response
- **Settings Layout:** max-w-3xl zentriert, Tab-Navigation als Links. Account-Page braucht keinen eigenen Container/Header — Settings-Layout stellt bereit
- **Form-Button-Hierarchie:** "Save" = Primary Button, alle anderen = outline/Secondary. Max 1 Primary pro Screen
- **Toast-Pattern:** Erfolg = `toast.success()` 3s auto-dismiss. Fehler = `toast.error()` mit `duration: Infinity`
- **Skeleton-Pattern:** Pro Komponente eigene Skeleton-Variante (Suffix `Skeleton`)
- **Commit-Message Pattern:** "Add [Feature] with review fixes (Story X.Y)"
- **Tab-Navigation Prefix-Match:** Settings-Layout nutzt `pathname.startsWith(tab.href)` — Account-Inhalte sind automatisch korrekt hervorgehoben

### Git Intelligence (letzte 5 Commits)

1. `5890372` — Add Test Webhook & Delivery Log with review fixes (Story 4.2)
2. `661de6f` — Add Settings Layout & Webhook URL Configuration with review fixes (Story 4.1)
3. `d275e8e` — Add Payment Detail & Webhook History with review fixes (Story 3.5)
4. `d980a47` — Add Payment Filters, Sorting & Pagination with review fixes (Story 3.4)
5. `8321838` — Add Payments List Page with DataTable, Card-Layout & review fixes (Story 3.3)

**Relevantes Pattern:** Proxy-Routen in `app/api/`, Hooks in `hooks/`, Komponenten in `components/dashboard/`. PATCH Route wird erweitert (nicht neu erstellt). Alle Dashboard-Pages sind `"use client"` mit Hook-Integration.

### Project Structure Notes

- `/dashboard/settings/account/page.tsx` — bestehende Placeholder-Seite wird ersetzt
- `/api/tenant/route.ts` — PATCH Zod-Schema wird erweitert, gleicher Endpoint, gleiche Route
- Neue Komponenten folgen dem Dashboard-Component-Pattern: `account-settings-form.tsx`, `deposit-address-card.tsx`
- Settings-Layout bleibt unverändert — Account-Tab ist bereits konfiguriert
- Keine neuen API-Endpoints nötig — alle Daten kommen aus GET/PATCH /v1/paywatcher/config (bestehend)

### Architektur-Compliance

- Dateien in `kebab-case.tsx` — `account-settings-form.tsx`, `deposit-address-card.tsx`
- Types in `types/tenant.ts` — bereits vorhanden, keine Änderung
- Route Handler Response im Envelope-Format `{ data }` / `{ error }`
- camelCase → snake_case Transformation im Route Handler (für PATCH Body an MMS)
- snake_case → camelCase Transformation im Route Handler (für Response von MMS)
- TanStack Query: Mutation via `useUpdateConfig()`, Query via `useTenant()`
- Kein `any` Type
- Kein Full-Page-Spinner — Skeleton-Loading für alle Sections
- Toast via sonner: Erfolg 3s auto-dismiss, Fehler bleibt (Infinity)
- Max 1 Primary Button pro Screen ("Save" = Primary, "Logout" = outline/Secondary)
- Copy-Feedback als Inline-Checkmark — nie als Toast
- Amounts/Adressen in Geist Mono (`font-mono`)
- Basescan-Link öffnet in neuem Tab (NFR14)
- MMS-Calls ausschließlich über `mmsApi()` in Route Handlers
- KEIN separater Tenant-Proxy-Endpoint — alle Daten kommen aus GET/PATCH /v1/paywatcher/config

### Library/Framework Requirements

- **react-hook-form** — bereits installiert (Story 4.1)
- **@hookform/resolvers** — bereits installiert (Story 4.1)
- **zod** — bereits installiert (Projektbasis)
- **lucide-react** — bereits installiert (Icons: Copy, Check, ExternalLink)
- **next-auth** — bereits installiert (signOut)
- **sonner** — bereits installiert (Toasts)
- **shadcn/ui** — Button, Input, Card, Skeleton, Label bereits installiert

**Keine neuen Dependencies nötig.**

### References

- [Source: _bmad-output/planning-artifacts/epics.md#Story 4.3]
- [Source: _bmad-output/planning-artifacts/architecture.md#FR-Kategorie: Tenant Settings (FR23-FR25)]
- [Source: _bmad-output/planning-artifacts/architecture.md#Component Patterns — Button-Hierarchie, Toast Pattern]
- [Source: _bmad-output/planning-artifacts/architecture.md#TanStack Query Patterns — Mutations]
- [Source: _bmad-output/planning-artifacts/architecture.md#Vollständige Projektverzeichnis-Struktur — settings/account/page.tsx]
- [Source: _bmad-output/planning-artifacts/architecture.md#API Communication Pattern — PATCH /v1/paywatcher/config]
- [Source: _bmad-output/planning-artifacts/epics.md#FR23 — Confirmation Override 2-20]
- [Source: _bmad-output/planning-artifacts/epics.md#FR24 — Deposit Address read-only]
- [Source: _bmad-output/planning-artifacts/epics.md#FR25 — Account-Einstellungen Email]
- [Source: _bmad-output/implementation-artifacts/4-1-settings-layout-webhook-url-configuration.md — Previous Story Pattern]
- [Source: _bmad-output/implementation-artifacts/4-2-test-webhook-delivery-log.md — Previous Story Pattern]
- [Source: src/app/api/tenant/route.ts — PATCH Route Zeile 9-11 (Zod Schema) + Zeile 54-56 (Transformation)]
- [Source: src/hooks/use-update-config.ts — UpdateConfigPayload mit confirmationOverride]
- [Source: src/hooks/use-tenant.ts — useTenant() Hook]
- [Source: src/hooks/use-copy.ts — useCopy() Hook]
- [Source: src/types/tenant.ts — TenantConfig Interface]
- [Source: src/components/dashboard/webhook-url-form.tsx — React Hook Form + Zod Pattern]
- [Source: src/app/(dashboard)/dashboard/settings/account/page.tsx — Aktueller Placeholder]

## Dev Agent Record

### Agent Model Used

Claude Opus 4.6

### Debug Log References

- Build-Fehler mit `z.coerce.number()` und `zodResolver` TypeScript-Inkompatibilität — gelöst durch `z.number()` + `valueAsNumber: true` in `register()` Options
- Story referenzierte nicht-existierenden `useCopy()` Hook — stattdessen bestehende `InlineCopyButton` Komponente verwendet

### Completion Notes List

- Task 1: PATCH Route `patchConfigSchema` um `confirmationOverride` erweitert (nullable, int, min 2, max 20) + snake_case Transformation hinzugefügt
- Task 2: `AccountSettingsForm` mit React Hook Form + Zod erstellt, `AccountSettingsFormSkeleton` für Loading State
- Task 3: `DepositAddressCard` mit Basescan-Link, `InlineCopyButton` und `font-mono` erstellt, plus `DepositAddressCardSkeleton`
- Task 4: Email-Anzeige (read-only, text-muted-foreground) + Logout-Button (variant="outline", signOut mit callbackUrl "/") inline in Page
- Task 5: Account Page zusammengebaut mit 3 Sections (Block Confirmations, Deposit Address, Account), Loading/Error States
- Task 6: `npm run build` erfolgreich, alle Pages generiert

### Senior Developer Review (AI)

**Reviewer:** Sempre | **Datum:** 2026-02-20 | **Ergebnis:** Approved (3 Fixes applied)

**AC-Validierung:** 6/6 ACs implementiert (2 davon nach Fix vollständig)
**Task-Audit:** 6/6 Tasks korrekt als [x] — keine falschen Claims
**Git vs Story:** 0 Diskrepanzen bei Source-Dateien

**Fixes Applied:**
1. **M1 — Touch Targets (AC6):** `h-11 lg:h-9` auf Save/Logout/Reset Buttons, `min-h-11 min-w-11 lg:min-h-0 lg:min-w-0` auf InlineCopyButton → 44px Touch Targets auf Mobile
2. **M2 — Reset to Default (AC2):** "Reset to Default" Button hinzugefügt (variant="ghost"), sendet `{ confirmationOverride: null }`, nur sichtbar wenn Override aktiv
3. **M3 — Dirty State:** Save Button disabled wenn keine Änderung (`!form.formState.isDirty`)

**Verbleibende LOW Issues (akzeptiert):**
- L1: Leeres Input → NaN → unfreundliche Zod-Fehlermeldung (Edge Case)
- L2: Inkonsistentes aria-describedby Pattern vs webhook-url-form
- L3: Kein Guard für leeren depositAddress (TypeScript-Typ erzwingt string)
- L4: Fehlende aria-labelledby auf Section-Elementen

### Change Log

- 2026-02-20: Code Review — 3 MEDIUM Fixes applied (Touch Targets, Reset to Default, Dirty State)
- 2026-02-20: Story 4.3 implementiert — Tenant Configuration & Account Settings (Confirmation Override, Deposit Address, Email + Logout)

### File List

- `src/app/api/tenant/route.ts` — GEÄNDERT: confirmationOverride zu Zod-Schema + Transformation
- `src/components/dashboard/account-settings-form.tsx` — NEU: Confirmation Override Formular + Skeleton
- `src/components/dashboard/deposit-address-card.tsx` — NEU: Deposit Address Anzeige + Skeleton
- `src/app/(dashboard)/dashboard/settings/account/page.tsx` — ERSETZT: Placeholder → vollständige Account Page
