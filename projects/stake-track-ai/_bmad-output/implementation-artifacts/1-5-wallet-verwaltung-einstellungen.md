# Story 1.5: Wallet-Verwaltung & Einstellungen

Status: done

<!-- Note: Validation is optional. Run validate-create-story for quality check before dev-story. -->
<!-- Generated: 2026-02-21 | Epic: 1 | Story: 5 | Key: 1-5-wallet-verwaltung-einstellungen -->

## Story

As a authentifizierter User,
I want meine verknüpften Wallets einsehen und einzeln entfernen können,
So that ich die Kontrolle über meine Daten behalte.

## Acceptance Criteria

1. **Given** ein authentifizierter User auf der Settings-Page (`/settings`) **When** die Seite geladen wird **Then** sieht er eine Liste aller verknüpften Wallets mit: Chain-Badge (ETH/SOL/ATOM), gekürzter Adresse (0x1234...5678), Verknüpfungsdatum **And** die primäre Auth-Wallet ist als "Primär" markiert

2. **Given** ein User klickt "Entfernen" bei einer verknüpften Wallet **When** es sich um eine sekundäre (nicht-primäre) Wallet handelt **Then** erscheint ein Bestätigungs-Dialog: "Wallet [gekürzte Adresse] auf [Chain] trennen?" **And** bei Bestätigung wird die Wallet aus `linked_wallets` entfernt **And** ein Success-Toast erscheint: "Wallet getrennt" **And** die Liste aktualisiert sich

3. **Given** ein User versucht seine einzige/primäre Auth-Wallet zu entfernen **When** keine alternative Auth-Methode existiert (kein Email-Login) **Then** zeigt die UI einen Hinweis: "Deine primäre Wallet kann nicht entfernt werden, solange keine alternative Login-Methode existiert"

4. **Given** ein User hat sowohl Wallet als auch Email verknüpft **When** er die primäre Wallet entfernen will **Then** wird die Email zum primären Auth-Mechanismus **And** die Wallet wird entfernt

5. **Given** das Wallet-Entfernen fehlschlägt (DB-Fehler, Netzwerkfehler) **When** der Fehler erkannt wird **Then** zeigt die UI einen Error-Toast: "Wallet konnte nicht getrennt werden. Nochmal versuchen?" **And** die Wallet bleibt unverändert in der Liste

6. **Given** ein User entfernt eine verknüpfte Wallet **When** die Wallet erfolgreich entfernt wurde **Then** wird der TanStack Query Cache für linked Wallets invalidiert **And** der ChainFilter im Dashboard aktualisiert sich (entfernt den Chain-Tab falls keine Wallets mehr auf dieser Chain)

## Tasks / Subtasks

- [x] Task 1: Wallet-Unlink API Endpoint (AC: #2, #3, #4, #5)
  - [x] 1.1: `src/app/api/wallets/[walletId]/route.ts` erstellen — DELETE Endpoint
  - [x] 1.2: Session-Check: User muss authentifiziert sein
  - [x] 1.3: Ownership-Check: Wallet muss dem authentifizierten User gehören (`userId` Match)
  - [x] 1.4: Primär-Wallet-Schutz: Prüfe ob die Wallet die primäre Auth-Wallet ist (`users.wallet_address`)
  - [x] 1.5: Fallback-Auth-Check: Falls primäre Wallet → prüfe ob `users.email` existiert (alternative Auth-Methode)
  - [x] 1.6: Falls primäre Wallet UND Email existiert → `users.wallet_address` auf NULL setzen, Wallet aus `linked_wallets` entfernen (falls dort), Email wird primärer Auth
  - [x] 1.7: Falls primäre Wallet OHNE Email → 403 mit `{ error: "Primäre Wallet kann nicht entfernt werden ohne alternative Login-Methode", code: "PRIMARY_WALLET_PROTECTED" }`
  - [x] 1.8: Falls sekundäre Wallet → `unlinkWallet(walletId, userId)` aus linked-wallet-repository aufrufen
  - [x] 1.9: Zod-Schema für Route-Params: walletId (uuid)
  - [x] 1.10: Response Format: `{ data: { success: true } }` (Success 200), `{ error, code }` (Error)
  - [x] 1.11: Error Codes: `PRIMARY_WALLET_PROTECTED` (403), `WALLET_NOT_FOUND` (404), `UNAUTHORIZED` (401)
  - [x] 1.12: Logging: `[wallet-unlink] Wallet unlinked for user [userId-kurz]` — KEINE Wallet-Adressen in Logs (NFR15)

- [x] Task 2: User-Repository erweitern (AC: #4)
  - [x] 2.1: `clearWalletAddress(userId: string): Promise<void>` in `src/infrastructure/db/repositories/user-repository.ts` hinzufügen — setzt `wallet_address` auf NULL
  - [x] 2.2: `hasAlternativeAuth(userId: string): Promise<{ hasEmail: boolean, hasWallet: boolean }>` hinzufügen — prüft ob User Email und/oder Wallet hat

- [x] Task 3: Settings-Page erweitern — Wallet-Entfernen UI (AC: #1, #2, #3)
  - [x] 3.1: Bestehende Settings-Page (`src/app/(dashboard)/settings/page.tsx`) erweitern
  - [x] 3.2: Pro Wallet (NICHT primäre ohne Alternative): "Entfernen"-Button (Text-Button, `text-secondary`, hover: `status-negative`)
  - [x] 3.3: Primäre Wallet ohne Alternative: Kein Entfernen-Button, stattdessen Tooltip/Hinweis "Primäre Wallet"
  - [x] 3.4: Primäre Wallet MIT Email-Alternative: "Entfernen"-Button mit Zusatzhinweis "Email wird zum primären Login"

- [x] Task 4: Bestätigungs-Dialog für Wallet-Entfernen (AC: #2, #5)
  - [x] 4.1: `src/components/settings/UnlinkWalletDialog.tsx` erstellen — Destructive Confirmation Modal
  - [x] 4.2: Modal-Layout laut UX-Spec: Titel "Wallet trennen?", Erklärung "Wallet [gekürzte Adresse] auf [Chain] wird von deinem Account getrennt.", Cancel-Button (Ghost-Style), Confirm-Button (`status-negative` Outline-Style: "Trennen")
  - [x] 4.3: Falls primäre Wallet mit Email-Fallback: Zusätzlicher Hinweis im Dialog "Dein Email-Login wird zum primären Authentifizierungsmechanismus."
  - [x] 4.4: Loading-State auf Confirm-Button während API-Call
  - [x] 4.5: Bei Fehler: Error-Toast "Wallet konnte nicht getrennt werden. Nochmal versuchen?"
  - [x] 4.6: Bei Erfolg: Dialog schließen, Success-Toast "Wallet getrennt", TanStack Query Cache invalidieren (`queryClient.invalidateQueries({ queryKey: ['linked-wallets'] })`)
  - [x] 4.7: Accessibility: `aria-modal="true"`, `aria-labelledby` auf Titel, Focus-Trap innerhalb Dialog (native `<dialog>`), Escape schließt Dialog, Backdrop-Klick schließt Dialog

- [x] Task 5: Toast-Notification-System (AC: #2, #5)
  - [x] 5.1: Prüfe ob ein Toast-System bereits existiert — falls nicht: `src/components/ui/Toast.tsx` erstellen
  - [x] 5.2: Toast-Varianten laut UX-Spec: success (`status-positive`, ✓, 3s Auto-Dismiss), error (`status-negative`, ✕, persistent), warning (`status-warning`, ⚠, persistent), info (`status-info`, ℹ, persistent)
  - [x] 5.3: Positionierung: Top-right (Desktop), Top-center (Mobile)
  - [x] 5.4: Maximal 1 Toast gleichzeitig sichtbar — neuer Toast ersetzt alten
  - [x] 5.5: `useToast()` Hook erstellen in `src/lib/hooks/use-toast.ts` — `{ showToast(type, message), dismissToast() }`
  - [x] 5.6: ToastProvider in Layout einbinden (nur falls neues System erstellt wird) — Toast wird direkt in WalletList gerendert, kein separater Provider nötig

- [x] Task 6: Settings-Page Wallet-Liste UX-Polish (AC: #1)
  - [x] 6.1: Wallet-Karten mit hover-State (`bg-surface-raised`) auf Desktop
  - [x] 6.2: Mobile: Swipe-to-reveal "Entfernen"-Action (optional, falls technisch einfach — sonst sichtbarer Button) — sichtbarer Button gewählt (simpler, konsistenter)
  - [x] 6.3: Leere-Wallets-State: Falls nur primäre Wallet und keine linked Wallets → "Füge weitere Wallets hinzu, um dein Multi-Chain-Portfolio zu sehen"
  - [x] 6.4: Chain-Badge Farben konsistent mit ChainFilter: ETH `bg-blue-600`, SOL `bg-purple-600`, ATOM `bg-indigo-600`

- [x] Task 7: Tests (AC: alle)
  - [x] 7.1: `src/app/api/wallets/[walletId]/route.test.ts` — DELETE Endpoint: Auth-Check, Ownership-Check, sekundäre Wallet entfernen (Success), primäre Wallet ohne Email (403), primäre Wallet mit Email (Success + wallet_address NULL), Wallet nicht gefunden (404) — 6 Tests
  - [x] 7.2: `src/infrastructure/db/repositories/user-repository.test.ts` erweitern — clearWalletAddress, hasAlternativeAuth — 4 Tests
  - [x] 7.3: `src/components/settings/UnlinkWalletDialog.test.tsx` — Dialog öffnen, Cancel schließt, Confirm ruft API auf, Loading-State, Error-State — 5 Tests (optional, falls Testing-Library-Infrastruktur vorhanden)
  - [x] 7.4: Regressions-Check: Alle bestehenden 92 Tests müssen weiterhin bestehen — 108 Tests bestehen (92 + 16 neue)

## Dev Notes

### Kritische Architektur-Entscheidungen

- **Wallet-Entfernen ist ein DELETE auf linked_wallets ODER ein UPDATE auf users:** Sekundäre Wallets (SOL/ATOM, zusätzliche ETH) stehen in `linked_wallets` und werden per `unlinkWallet()` gelöscht. Die primäre ETH-Wallet steht in `users.wallet_address` — beim Entfernen wird diese auf NULL gesetzt (nur wenn Email als Fallback existiert).

- **Primäre-Wallet-Schutz ist SERVERSEITIG:** Die UI zeigt/versteckt den Entfernen-Button, aber der Server MUSS unabhängig prüfen ob die Wallet entfernt werden darf. Defense in Depth — Client-Side ist nur UX, nicht Security.

- **Wallet-ID-Routing:** Der DELETE-Endpoint nutzt `[walletId]` als Route-Parameter. Für sekundäre Wallets ist das die UUID aus `linked_wallets.id`. Für die primäre Wallet wird ein spezieller Identifier verwendet (z.B. `primary` als Konvention oder die Wallet-Adresse selbst). **Empfehlung:** Verwende `linked_wallets.id` für sekundäre Wallets und ein separates Flag/Endpoint-Logik für die primäre Wallet, da die primäre Wallet NICHT in `linked_wallets` steht.

- **ETH-Wallet als primäre Auth-Quelle:** Die primäre ETH-Wallet steht in `users.wallet_address`. In der Wallet-Liste auf der Settings-Page wird sie als "virtuelle" LinkedWallet dargestellt (GET `/api/wallets` macht das bereits seit Story 1-4, Task 8.3). Beim Entfernen der primären Wallet muss der DELETE-Endpoint erkennen, dass es sich um die primäre handelt (z.B. über einen Query-Parameter `?type=primary` oder indem die Wallet-Adresse statt der UUID gesendet wird).

- **Email als Fallback-Auth nach Wallet-Entfernung:** Wenn die primäre Wallet entfernt wird und Email existiert, wird Email zum alleinigen Auth-Mechanismus. Die NextAuth-Session bleibt gültig (JWT-based), aber beim nächsten Login muss der User Email nutzen. Der JWT `sub` Claim ändert sich NICHT sofort — erst beim nächsten Login wird er auf die User-ID (statt Wallet-Adresse) gesetzt.

- **Kein Cascading-Daten-Delete bei Wallet-Entfernen:** Beim Entfernen einer Wallet werden KEINE Staking-Daten gelöscht (die kommen erst in Epic 2). Nur der `linked_wallets`-Eintrag wird gelöscht. In Zukunft (Epic 2+) muss hier evaluiert werden, ob zugehörige Staking-Positionen ebenfalls gelöscht werden sollen.

### Bestehende Infrastruktur (aus Story 1-1 bis 1-4)

| Komponente | Datei | Status |
|---|---|---|
| DB Schema | `src/infrastructure/db/schema.ts` | Existiert (users + verification_tokens + linked_wallets) |
| Linked-Wallet Repository | `src/infrastructure/db/repositories/linked-wallet-repository.ts` | Existiert — `unlinkWallet(id, userId)` bereits implementiert (Story 1-4, Task 2.5) |
| User Repository | `src/infrastructure/db/repositories/user-repository.ts` | Existiert — MUSS um `clearWalletAddress()` und `hasAlternativeAuth()` erweitert werden |
| Settings Page | `src/app/(dashboard)/settings/page.tsx` | Existiert — zeigt Wallet-Liste, "Wallet hinzufügen"-Button, Placeholder für Account-Sektion |
| AddWalletDialog | `src/components/settings/AddWalletDialog.tsx` | Existiert — SOL+ATOM Linking (Story 1-4) |
| ChainFilter | `src/components/dashboard/ChainFilter.tsx` | Existiert — dynamische Tabs basierend auf verknüpften Wallets |
| useLinkedWallets Hook | `src/lib/hooks/use-linked-wallets.ts` | Existiert — TanStack Query, fetcht GET /api/wallets |
| GET /api/wallets | `src/app/api/wallets/route.ts` | Existiert — gibt alle Wallets zurück (primäre + linked) |
| POST /api/wallets/link | `src/app/api/wallets/link/route.ts` | Existiert — Wallet-Linking (Story 1-4) |
| Wallet Verification | `src/lib/auth/wallet-verification.ts` | Existiert — NICHT relevant für diese Story |
| Format Utils | `src/lib/utils/format.ts` | `truncateAddress()` vorhanden |
| Middleware | `src/middleware.ts` | `/settings` bereits im matcher (Story 1-4) |

### API Design: DELETE /api/wallets/[walletId]

```typescript
// DELETE /api/wallets/[walletId]
// walletId = linked_wallets.id (UUID) für sekundäre Wallets
// walletId = "primary" für die primäre ETH-Wallet

// Success Response (200):
{ data: { success: true } }

// Error Responses:
{ error: "Nicht authentifiziert", code: "UNAUTHORIZED" }                                          // 401
{ error: "Wallet nicht gefunden", code: "WALLET_NOT_FOUND" }                                     // 404
{ error: "Primäre Wallet kann nicht entfernt werden ohne alternative Login-Methode",
  code: "PRIMARY_WALLET_PROTECTED" }                                                             // 403
```

### Primäre-Wallet-Entfernen Flow

```
User klickt "Entfernen" bei primärer ETH-Wallet
  → DELETE /api/wallets/primary
  → Server prüft: Hat User eine Email? (users.email !== null)
    → JA: users.wallet_address = NULL, Response 200
    → NEIN: Response 403 PRIMARY_WALLET_PROTECTED
```

### Settings-Page UI-Struktur (nach Story 1.5)

```
┌───────────────────────────────────────┐
│  Einstellungen                         │
│                                        │
│  ── Verknüpfte Wallets ──             │
│                                        │
│  ┌─────────────────────────────────┐  │
│  │ ETH  0x1234...5678  Primär      │  │  ← Kein Entfernen (keine Email)
│  │      Verknüpft am 18.02.26     │  │     ODER Entfernen (wenn Email existiert)
│  ├─────────────────────────────────┤  │
│  │ SOL  7xKX...4bPq    [Entfernen]│  │  ← Entfernen-Button
│  │      Verknüpft am 21.02.26     │  │
│  ├─────────────────────────────────┤  │
│  │ ATOM cosmos1...abc4  [Entfernen]│  │  ← Entfernen-Button
│  │      Verknüpft am 21.02.26     │  │
│  └─────────────────────────────────┘  │
│                                        │
│  [ + Wallet hinzufügen ]              │
│                                        │
│  ── Account ──                        │  ← Vorbereitet für Story 1.6
│  [ Account löschen ]                  │  ← Destructive, kommt in Story 1.6
└───────────────────────────────────────┘
```

### Bestätigungs-Dialog Design (laut UX-Spec)

```
┌─────────────────────────────────────┐
│  Wallet trennen?                     │
│                                      │
│  Wallet 7xKX...4bPq auf Solana      │
│  wird von deinem Account getrennt.   │
│                                      │
│  [Abbrechen]        [Trennen]        │
│   Ghost-Style     status-negative    │
└─────────────────────────────────────┘
```

### Learnings aus Story 1-4 (MUSS beachtet werden)

- **`--legacy-peer-deps` nötig:** Für ALLE npm installs wegen React 19 Peer-Dep-Konflikten
- **`serverEnv` statt `process.env`:** IMMER `serverEnv` aus `@/lib/env` verwenden (Review-Finding H2 aus Story 1-3)
- **Biome 2.2.0:** Alle neuen Dateien müssen Biome-Check bestehen
- **Next.js 15.5.12 (LTS), Tailwind v4:** Custom Tokens in `globals.css` via `@theme`
- **`truncateAddress()` in `src/lib/utils/format.ts`:** Bereits vorhanden — für Wallet-Adressen-Kürzung verwenden
- **Drizzle `neon-http` Modus:** HTTP-basierter Treiber. Für Wallet-Unlink reichen einzelne DELETE/UPDATE Queries
- **In-Memory Rate-Limiting:** Upstash Rate Limit nicht installiert. Falls Rate-Limiting auf DELETE-Endpoint nötig, gleichen In-Memory-Fallback verwenden
- **`unlinkWallet(id, userId)` bereits implementiert:** Repository-Methode aus Story 1-4 Task 2.5 ist ready-to-use
- **`deleteAllLinkedWallets(userId)` bereits implementiert:** Für Story 1.6 vorbereitet, NICHT in dieser Story verwenden
- **Provider-Hierarchy:** Web3Provider > AuthProvider > SolanaProvider > ThemeProvider — NICHT ändern

### Sicherheitsaspekte

- **Server-seitige Ownership-Prüfung (PFLICHT):** Der DELETE-Endpoint MUSS prüfen, dass die Wallet dem authentifizierten User gehört. Ohne diese Prüfung könnte ein authentifizierter User fremde Wallets löschen.
- **Primäre-Wallet-Schutz serverseitig:** Die Logik "keine alternative Auth-Methode vorhanden → Entfernen verbieten" MUSS auf dem Server geprüft werden, nicht nur in der UI.
- **Keine PII in Logs (NFR15):** Wallet-Adressen nur gekürzt loggen, User-IDs nur gekürzt.
- **CSRF-Schutz:** NextAuth Session-basierte Authentifizierung bietet CSRF-Schutz über den Session-Cookie. Kein zusätzlicher CSRF-Token nötig für DELETE-Requests.
- **Rate-Limiting auf DELETE-Endpoint:** Optional aber empfohlen — verhindert Mass-Unlink-Angriffe. In-Memory-Fallback reicht für MVP.

### Abgrenzung: Was diese Story NICHT macht

- **Kein Account-Löschen:** Das ist Story 1.6 (Account-Löschung DSGVO). Die `deleteAllLinkedWallets()`-Methode existiert bereits, wird aber hier NICHT aufgerufen.
- **Keine Staking-Daten-Löschung:** Beim Wallet-Entfernen werden KEINE Staking-Positionen gelöscht (kommen erst in Epic 2).
- **Kein Tier-Gating:** Free-User-Limits für Chain-Anzahl kommen in Epic 6. In dieser Story kann jeder User beliebig Wallets entfernen.
- **Keine Email-Verwaltung:** Email hinzufügen/ändern ist nicht Teil dieser Story. Nur die Prüfung ob Email als Alternative existiert.
- **Kein Theme-Toggle:** Dark/Light-Mode-Toggle auf der Settings-Page ist nicht Teil dieser Story.

### Project Structure Notes

- `UnlinkWalletDialog.tsx` gehört in `src/components/settings/` (neben AddWalletDialog.tsx)
- DELETE API Route gehört in `src/app/api/wallets/[walletId]/` (App Layer)
- Toast-System (falls neu erstellt) gehört in `src/components/ui/` (UI Layer)
- `useToast` Hook gehört in `src/lib/hooks/` (Shared Utilities)
- User-Repository-Erweiterung bleibt in `src/infrastructure/db/repositories/user-repository.ts`
- Domain Layer (`src/domain/`) wird in dieser Story NICHT berührt

### References

- [Source: _bmad-output/planning-artifacts/epics.md#Story 1.5 — Wallet-Verwaltung & Einstellungen]
- [Source: _bmad-output/planning-artifacts/epics.md#Epic 1 — FR6]
- [Source: _bmad-output/planning-artifacts/architecture.md#Authentication & Security]
- [Source: _bmad-output/planning-artifacts/architecture.md#Implementation Patterns & Consistency Rules]
- [Source: _bmad-output/planning-artifacts/architecture.md#Complete Project Directory Structure]
- [Source: _bmad-output/planning-artifacts/architecture.md#Architectural Boundaries]
- [Source: _bmad-output/planning-artifacts/prd.md#FR6 — Wallet-Verwaltung (einsehen/entfernen)]
- [Source: _bmad-output/planning-artifacts/prd.md#NFR15 — Keine PII in Logs]
- [Source: _bmad-output/planning-artifacts/ux-design-specification.md#Modal Pattern — Destructive Confirmations Only]
- [Source: _bmad-output/planning-artifacts/ux-design-specification.md#Toast Feedback Patterns]
- [Source: _bmad-output/planning-artifacts/ux-design-specification.md#Bottom Sheet Anatomy]
- [Source: _bmad-output/project-context.md#Authentication Rules — Unified User Profile]
- [Source: _bmad-output/project-context.md#Security Rules — Read-Only Analytics]
- [Source: _bmad-output/implementation-artifacts/1-4-multi-chain-wallet-linking-sol-atom.md — Learnings, File List, Repository-Methoden]
- [Source: _bmad-output/implementation-artifacts/1-4-multi-chain-wallet-linking-sol-atom.md#Task 2.5 — unlinkWallet() bereits implementiert]
- [Source: _bmad-output/implementation-artifacts/1-4-multi-chain-wallet-linking-sol-atom.md#Task 8.3 — GET /api/wallets inkludiert primäre ETH als virtuelle LinkedWallet]

## Dev Agent Record

### Agent Model Used

Claude Opus 4.6

### Debug Log References

Keine Debug-Probleme während der Implementierung.

### Completion Notes List

- Task 2: `clearWalletAddress()` und `hasAlternativeAuth()` im User-Repository implementiert mit 4 neuen Tests
- Task 1: DELETE `/api/wallets/[walletId]` Endpoint implementiert mit Zod-Validierung (UUID oder "primary"), Session-Check, Ownership-Check, Primär-Wallet-Schutz serverseitig, 6 Tests
- Task 5: Toast-System neu erstellt (`Toast.tsx` + `useToast` Hook) mit 4 Varianten (success/error/warning/info), success auto-dismiss nach 3s, max 1 Toast gleichzeitig
- Task 4: `UnlinkWalletDialog` mit nativem `<dialog>` Element, Bestätigungs-Flow, Loading-State, Error-Handling, primäre Wallet Email-Fallback-Hinweis, 5 Tests
- Task 3: Settings-Page refactored — Server Component lädt initial-Daten, neue `WalletList` Client Component für Interaktivität (Unlink-Button, Dialog, Toast)
- Task 6: Hover-State auf Wallet-Karten, Chain-Badge-Farben konsistent, Leere-Wallets-Hinweis, sichtbarer Entfernen-Button (kein Swipe)
- Task 7: 16 neue Tests (6 API + 4 Repository + 5 Dialog + 1 Regression), alle 108 Tests bestehen (vorher 92)
- Biome-Check bestanden auf allen neuen/geänderten Dateien
- AC4.7: `role="dialog"` entfernt da redundant auf nativem `<dialog>` Element (Biome lint), restliche a11y-Attribute beibehalten

### Senior Developer Review (AI) — 2026-02-21

**Reviewer:** Code Review Workflow (adversarial)

**Findings Fixed (7):**
- **H1:** Dialog schloss bei API-Fehler — `onClose()` aus catch-Block entfernt, Dialog bleibt für Retry offen (AC5)
- **H2:** Native `<dialog>` cancel-Event umging isLoading-Guard — `onCancel` Handler mit `preventDefault()` hinzugefügt, `onKeyDown` durch `onCancel` ersetzt
- **H3:** TOCTOU Race Condition bei Primary-Wallet-Unlink — Re-Validierung nach `clearWalletAddress()` als Defense-in-Depth
- **M1:** Rate-Limiting auf DELETE-Endpoint hinzugefügt (5 req/user/15min, In-Memory, konsistent mit bestehendem Pattern)
- **M2:** Windows `nul`-Artefakt gelöscht, `.gitignore` ergänzt
- **M3:** Task 7.4 Testanzahl korrigiert (103 → 108)
- **M4:** Settings-Page DB-Queries optimiert: `findUserById()` statt `findUserByWalletAddress()`, 3 Queries parallelisiert via `Promise.all()`
- **L1 (Bonus):** Redundantes `aria-modal="true"` von nativem `<dialog>` entfernt

**Low Issues (nicht gefixt, akzeptabel):**
- L2: `truncateAddress()` semantisch für UUIDs verwendet — funktional korrekt, kosmetisch

### Implementation Plan

- Verwendet bestehende `unlinkWallet()` Repository-Methode für sekundäre Wallets
- Primäre Wallet wird über `walletId="primary"` Konvention identifiziert
- Settings-Page-Architektur: Server Component (Daten laden) → Client Component (Interaktion) für optimale Performance
- Toast-System ohne globalen Provider — direkt in WalletList gerendert, da nur dort aktuell benötigt
- Native `<dialog>` Element statt custom Modal für bessere Accessibility und Browser-Support

### File List

- `src/app/api/wallets/[walletId]/route.ts` — NEU: DELETE Endpoint für Wallet-Unlink
- `src/app/api/wallets/[walletId]/route.test.ts` — NEU: 6 API-Tests
- `src/infrastructure/db/repositories/user-repository.ts` — GEÄNDERT: +findUserById, +clearWalletAddress, +hasAlternativeAuth
- `src/infrastructure/db/repositories/user-repository.test.ts` — GEÄNDERT: +4 Tests
- `src/components/settings/UnlinkWalletDialog.tsx` — NEU: Bestätigungs-Dialog
- `src/components/settings/UnlinkWalletDialog.test.tsx` — NEU: 5 Component-Tests
- `src/components/settings/WalletList.tsx` — NEU: Client Component für interaktive Wallet-Liste
- `src/components/ui/Toast.tsx` — NEU: Toast-Notification Komponente
- `src/lib/hooks/use-toast.ts` — NEU: Toast-State-Management Hook
- `src/app/(dashboard)/settings/page.tsx` — GEÄNDERT: Refactored mit WalletList + hasAlternativeAuth
