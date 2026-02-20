# Story 2.4: API Key Anzeige & Scopes

Status: done

## Story

As a Tenant,
I want meinen API Key (masked) und dessen Scopes im Dashboard einsehen koennen,
So that ich weiss welchen Key ich verwende und welche Berechtigungen er hat.

## Acceptance Criteria

1. **Given** ich bin auf der Settings/API Keys Seite **When** die Seite laedt **Then** sehe ich meinen API Key masked (z.B. "mms_paywat****...****") via GET /v1/paywatcher/config/api-key

2. **Given** der API Key angezeigt wird **Then** sehe ich die Scopes als Badges (z.B. payments:read, payments:write)

3. **Given** der API Key angezeigt wird **Then** sehe ich das Erstellungsdatum (created_at)

4. **Given** der Copy-Button geklickt wird **Then** wird der masked Key kopiert (Inline-Checkmark, kein Toast)

- KEIN Regenerate-Button — kein MMS-Endpoint dafuer. Regeneration erfolgt manuell durch Mario.
- Proxy Route: GET /api/api-keys → GET /v1/paywatcher/config/api-key (Response: { masked_key, scopes, created_at })

## Tasks / Subtasks

- [x] Task 1: API Key Proxy Route (AC: #1)
  - [x] 1.1 `src/app/api/api-keys/route.ts` — GET Handler: requireAuth(), mmsApi GET /v1/paywatcher/config/api-key, snakeToCamel, Envelope Format
  - [x] 1.2 Alte POST Handler (Regenerate) entfernt

- [x] Task 2: TanStack Query Hook (AC: #1)
  - [x] 2.1 `src/hooks/use-api-key.ts` — useApiKey() mit Query Key ['api-key'], staleTime 5min, enabled-Option
  - [x] 2.2 Alte useRegenerateApiKey() Mutation entfernt

- [x] Task 3: Settings API Keys Seite (AC: #1, #2, #4)
  - [x] 3.1 `src/app/(dashboard)/dashboard/settings/api-keys/page.tsx` — ApiKeyCard im masked-Modus, Scopes
  - [x] 3.2 Alte Regenerate-UI entfernt (AlertDialog, Confirmation Dialog, etc.)
  - [x] 3.3 Hinweis "Need a new API key? Contact your account manager."

- [x] Task 4: Erstellungsdatum anzeigen (AC: #3)
  - [x] 4.1 createdAt aus useApiKey() Response im UI anzeigen (wird bereits geholt aber nicht gerendert)

- [x] Task 5: ApiKeyCard Komponente (AC: #1, #2, #4)
  - [x] 5.1 `src/components/dashboard/api-key-card.tsx` — masked Modus mit Copy-Button (Inline-Checkmark), Scopes Badges, Skeleton Loading

- [x] Task 6: Settings-Seitenstruktur (AC: #1)
  - [x] 6.1 `src/app/(dashboard)/dashboard/settings/page.tsx` — Redirect zu /dashboard/settings/api-keys
  - [x] 6.2 Sidebar-Link "Settings" navigiert zu /dashboard/settings

- [x] Task 7: Build-Validierung (Alle ACs)
  - [x] 7.1 `npm run build` fehlerfrei
  - [x] 7.2 Landing Page bleibt Static

## Dev Notes

### Kontext & Zweck

Dies ist die **vierte und letzte Story von Epic 2** im **Managed MVP** Ansatz. Der Tenant kann seinen masked API Key und die Scopes einsehen. Es gibt **keinen Regenerate-Button** — FR13 wurde gestrichen, da kein MMS-Endpoint dafuer existiert. Key-Regeneration erfolgt manuell durch Mario.

**Was diese Story NICHT beinhaltet:**
- API Key Regenerate (FR13 entfaellt — kein MMS-Endpoint)
- Webhook Configuration (Epic 4)
- Account Settings (Epic 4)

### Bestehender Code-Stand

Die Settings/API-Keys Seite und die ApiKeyCard funktionieren korrekt im masked-Modus. Alle Regenerate-Artefakte wurden entfernt. Die Scopes werden als Badges angezeigt. Der Copy-Button nutzt Inline-Checkmark (kein Toast).

### References

- [Source: _bmad-output/planning-artifacts/epics.md#Story 2.4]
- [Source: _bmad-output/planning-artifacts/prd.md#FR12, FR14]
- [Source: _bmad-output/planning-artifacts/architecture.md#Component Patterns]

## Dev Agent Record

### Agent Model Used

Unknown (vorherige Session, nicht dokumentiert)

### Completion Notes List

- Migration von Regenerate-Flow zu Read-Only in einer vorherigen Session durchgefuehrt
- POST Handler in api-keys/route.ts entfernt
- useRegenerateApiKey() Mutation entfernt
- AlertDialog und Confirmation Dialog UI entfernt
- Settings-Seite vereinfacht auf masked Key + Scopes + "Contact your account manager"
- createdAt wird jetzt im UI angezeigt (Code Review Fix)

### Change Log

- 2026-02-18: Code Review (AI) — 5 Fixes: createdAt im UI (H1), prefix-Operator (H2), Copy-Fehlerbehandlung (M1), aria-label (M2), Architecture-Hinweis (M3)
- 2026-02-18: Story-File neu geschrieben fuer Managed MVP (alte Story beschrieb Regenerate Flow)
- ~2026-02-17: Code-Migration — Regenerate-UI und Endpoints entfernt
- 2026-02-16: Code Review (AI) der alten Regenerate-Implementierung (4 Fixes, obsolet)
- 2026-02-16: Alte Story 2.4 (API Key Management — Regenerate & Scopes) implementiert (obsolet, Code entfernt)

### File List

- src/app/api/api-keys/route.ts (GEAENDERT) — nur GET, POST entfernt
- src/hooks/use-api-key.ts (GEAENDERT) — nur useApiKey(), useRegenerateApiKey entfernt
- src/app/(dashboard)/dashboard/settings/api-keys/page.tsx (GEAENDERT) — vereinfacht, kein Regenerate
- src/app/(dashboard)/dashboard/settings/page.tsx (BESTEHEND) — Redirect zu api-keys
- src/components/dashboard/api-key-card.tsx (BESTEHEND) — masked + first-reveal Modi, Scopes, Copy
- src/components/ui/alert-dialog.tsx (BESTEHEND) — nicht mehr verwendet, kann entfernt werden
