# Story 1.6: Account-Löschung (DSGVO)

Status: done

<!-- Note: Validation is optional. Run validate-create-story for quality check before dev-story. -->
<!-- Generated: 2026-02-22 | Epic: 1 | Story: 6 | Key: 1-6-account-loeschung-dsgvo -->

## Story

As a authentifizierter User,
I want meinen Account und alle verknüpften Daten vollständig löschen können,
So that mein Recht auf Löschung (DSGVO Art. 17) gewährleistet ist.

## Acceptance Criteria

1. **Given** ein authentifizierter User auf der Settings-Page **When** er auf "Account löschen" klickt **Then** erscheint ein Bestätigungs-Modal (Destructive Confirmation) mit: Titel "Account unwiderruflich löschen?", Erklärung was gelöscht wird (Account, alle Wallets, alle Daten), Eingabefeld wo User "LÖSCHEN" tippen muss **And** der Confirm-Button ist im `status-negative` Destructive-Stil

2. **Given** ein User bestätigt die Löschung korrekt (tippt "LÖSCHEN") **When** die Löschung ausgeführt wird **Then** werden alle Daten des Users gelöscht: `users`-Eintrag (Cascade löscht `linked_wallets` automatisch), verwaiste `verification_tokens` (falls Email existiert) **And** die JWT-Session wird client-seitig via `signOut()` invalidiert **And** der User wird zur Landing Page weitergeleitet **And** ein sachlicher Hinweis erscheint: "Dein Account wurde gelöscht"

3. **Given** ein User tippt nicht das korrekte Bestätigungswort "LÖSCHEN" **When** er auf den Löschen-Button klickt **Then** wird die Löschung NICHT ausgeführt **And** das Eingabefeld zeigt einen Inline-Error: "Bitte tippe LÖSCHEN zur Bestätigung"

4. **Given** die Löschung schlägt technisch fehl (DB-Fehler) **When** ein Fehler auftritt **Then** zeigt die UI einen Error-Toast: "Löschung fehlgeschlagen. Bitte versuche es erneut." **And** der Account bleibt intakt **And** der Dialog bleibt offen für Retry

5. **Given** ein nicht-authentifizierter Request an den Delete-Endpoint **When** keine gültige Session vorhanden ist **Then** wird 401 zurückgegeben

6. **Given** ein User löscht seinen Account erfolgreich **When** er danach versucht eine geschützte Route aufzurufen **Then** wird er durch die Middleware zum Login weitergeleitet (da JWT-Cookie via `signOut()` gelöscht wurde)

## Tasks / Subtasks

- [x] Task 1: User-Repository erweitern — deleteUser Methode (AC: #2)
  - [x] 1.1: `deleteUser(userId: string): Promise<void>` in `src/infrastructure/db/repositories/user-repository.ts` hinzufügen — `db.delete(users).where(eq(users.id, userId))`
  - [x] 1.2: DB Cascade (`ON DELETE CASCADE` auf `linked_wallets.userId`) löscht verknüpfte Wallets automatisch — KEINE manuelle Löschung nötig
  - [x] 1.3: `deleteVerificationTokensByIdentifier(identifier: string): Promise<void>` hinzufügen — löscht verwaiste `verification_tokens` anhand der Email-Adresse (da kein FK auf `users` existiert)

- [x] Task 2: Account-Löschung API Endpoint (AC: #2, #4, #5)
  - [x] 2.1: `src/app/api/account/route.ts` erstellen — DELETE Endpoint
  - [x] 2.2: Session-Check: User muss authentifiziert sein (`getServerSession(authOptions)`)
  - [x] 2.3: Rate-Limiting: In-Memory Rate-Limit (1 req/user/60s) — konsistent mit bestehendem Pattern aus Story 1-5
  - [x] 2.4: User aus DB laden via `findUserById(userId)` — 404 falls nicht gefunden
  - [x] 2.5: Falls User eine Email hat → `deleteVerificationTokensByIdentifier(user.email)` aufrufen
  - [x] 2.6: `deleteUser(userId)` aufrufen — DB Cascade entfernt `linked_wallets` automatisch
  - [x] 2.7: Response: `{ data: { success: true } }` (200)
  - [x] 2.8: Error Response: `{ error: "...", code: "ACCOUNT_NOT_FOUND" }` (404), `{ error, code: "DELETE_FAILED" }` (500)
  - [x] 2.9: Logging: `[account-delete] Account deleted for user [userId-kurz]` — KEINE Wallet-Adressen oder Emails in Logs (NFR15)

- [x] Task 3: DeleteAccountSection Client Component (AC: #1, #3)
  - [x] 3.1: `src/components/settings/DeleteAccountSection.tsx` erstellen — Client Component (`"use client"`)
  - [x] 3.2: "Account löschen"-Button im `status-negative` Outline-Stil (Destructive Action gemäß UX-Spec)
  - [x] 3.3: Klick öffnet `DeleteAccountDialog`
  - [x] 3.4: Visuell klar abgetrennt von der Wallet-Sektion (eigene `<section>` mit Heading "Account")

- [x] Task 4: DeleteAccountDialog Component (AC: #1, #2, #3, #4)
  - [x] 4.1: `src/components/settings/DeleteAccountDialog.tsx` erstellen — Destructive Confirmation Modal
  - [x] 4.2: Natives `<dialog>` Element (konsistent mit UnlinkWalletDialog-Pattern aus Story 1-5)
  - [x] 4.3: Dialog-Layout gemäß UX-Spec (Destructive Confirmation):
    - Titel: "Account unwiderruflich löschen?"
    - Erklärungstext: "Alle deine Daten werden unwiderruflich gelöscht: Dein Account, alle verknüpften Wallets und alle zukünftigen Daten. Diese Aktion kann nicht rückgängig gemacht werden."
    - Eingabefeld: `<input>` mit Placeholder "Tippe LÖSCHEN zur Bestätigung"
    - Cancel-Button: Ghost-Style "Abbrechen"
    - Confirm-Button: `status-negative` Destructive-Stil "Account löschen" — DISABLED bis Eingabe === "LÖSCHEN"
  - [x] 4.4: Eingabe-Validierung: Confirm-Button wird nur enabled wenn `input.value.trim() === "LÖSCHEN"` (case-sensitive)
  - [x] 4.5: Bei Klick auf Confirm: `isLoading = true`, `fetch('/api/account', { method: 'DELETE' })`
  - [x] 4.6: Bei Erfolg (200): Dialog schließen, `signOut({ callbackUrl: '/' })` aufrufen (next-auth Client) — User wird zur Landing Page weitergeleitet, JWT-Cookie wird gelöscht
  - [x] 4.7: Bei Fehler: Error-Toast via `useToast()` — "Löschung fehlgeschlagen. Bitte versuche es erneut." — Dialog bleibt offen
  - [x] 4.8: `isLoading`-Guard: Während API-Call sind Buttons disabled, Spinner auf Confirm-Button, `onCancel` mit `preventDefault()` blockiert (konsistent mit UnlinkWalletDialog H2-Fix)
  - [x] 4.9: Accessibility: `role` wird durch natives `<dialog>` impliziert (kein redundantes `aria-modal`), `aria-labelledby` auf Titel, Focus-Trap, Escape schließt nur wenn nicht `isLoading`, Backdrop-Klick schließt nur wenn nicht `isLoading`

- [x] Task 5: Settings-Page Account-Sektion einbinden (AC: #1)
  - [x] 5.1: Bestehende Settings-Page (`src/app/(dashboard)/settings/page.tsx`) aktualisieren
  - [x] 5.2: Placeholder-Text "Account-Einstellungen werden in zukünftigen Updates verfügbar sein." durch `<DeleteAccountSection />` ersetzen
  - [x] 5.3: Visuelle Trennung: `border-t border-subtle` und `mt-8 pt-8` zwischen Wallet- und Account-Sektion
  - [x] 5.4: Heading "Account" als `<h2>` (konsistent mit "Verknüpfte Wallets"-Heading)

- [x] Task 6: Tests (AC: alle)
  - [x] 6.1: `src/infrastructure/db/repositories/user-repository.test.ts` erweitern — `deleteUser`: User löschen (Success), User nicht gefunden (kein Fehler, idempotent), `deleteVerificationTokensByIdentifier`: Tokens löschen (Success) — 3 Tests
  - [x] 6.2: `src/app/api/account/route.test.ts` erstellen — DELETE Endpoint: Nicht authentifiziert (401), User nicht gefunden (404), Account erfolgreich gelöscht (200), Account mit Email löscht auch verification_tokens, DB-Fehler (500), Rate-Limiting greift (429) — 6 Tests
  - [x] 6.3: `src/components/settings/DeleteAccountDialog.test.tsx` erstellen — Dialog öffnen, Confirm disabled ohne Eingabe, Confirm disabled bei falscher Eingabe, Confirm enabled bei "LÖSCHEN", Loading-State während API-Call, Error-State bei API-Fehler — 6 Tests (optional, falls Testing-Library-Infrastruktur vorhanden)
  - [x] 6.4: Regressions-Check: Alle bestehenden 108 Tests müssen weiterhin bestehen

## Dev Notes

### Kritische Architektur-Entscheidungen

- **DB Cascade übernimmt die schwere Arbeit:** `linked_wallets` hat bereits `ON DELETE CASCADE` auf `userId` FK → beim Löschen des `users`-Eintrags werden alle verknüpften Wallets automatisch entfernt. KEIN manueller `deleteAllLinkedWallets()`-Aufruf nötig (obwohl die Methode existiert).

- **`verification_tokens` hat KEINEN FK auf `users`:** Die Tabelle ist über `identifier` (Email-String) + `token` als Composite-PK identifiziert, nicht über `userId`. Daher muss explizit `deleteVerificationTokensByIdentifier(email)` aufgerufen werden, um verwaiste Tokens zu bereinigen. Dies ist nur nötig wenn der User eine Email hat (`users.email !== null`).

- **JWT-Session ist stateless — keine serverseitige Invalidierung möglich:** NextAuth nutzt `strategy: "jwt"` mit 7-Tage `maxAge`. Es gibt KEINEN serverseitigen Session-Store. Die einzige Möglichkeit die Session zu beenden ist `signOut()` auf dem Client aufzurufen, was den HTTP-only JWT-Cookie löscht. Nach DB-Löschung würde ein bestehender JWT bei API-Calls ohnehin fehlschlagen (User nicht in DB gefunden), aber der Cookie muss client-seitig gelöscht werden damit die Middleware-Redirects greifen.

- **Destructive Confirmation Pattern (UX-Spec):** Modals werden in StakeTrack AI NUR für Destructive Confirmations eingesetzt (nie für Upgrade-CTAs, Feature-Erklärungen etc.). Das Account-Lösch-Modal folgt exakt dem gleichen Pattern wie der UnlinkWalletDialog — natives `<dialog>`, Focus-Trap, Escape/Backdrop blockiert während Loading.

- **Bestätigungswort "LÖSCHEN" statt Checkbox:** Die Epics definieren explizit ein Eingabefeld wo der User "LÖSCHEN" tippen muss. Das ist bewusst stärker als eine Checkbox — es zwingt den User zum aktiven Nachdenken und verhindert versehentliches Klicken.

- **Kein Soft-Delete:** DSGVO Art. 17 (Recht auf Löschung) erfordert echte Datenlöschung, kein Soft-Delete mit `is_deleted` Flag. Die Daten werden physisch aus der DB entfernt.

- **Zukünftige Daten (Epic 2+):** Im aktuellen MVP-Stand (Epic 1 abgeschlossen) existieren nur `users`, `linked_wallets` und `verification_tokens`. In zukünftigen Epics kommen `staking_positions`, `alerts`, `alert_history`, `vrs_scores` etc. hinzu. Diese Tabellen MÜSSEN `ON DELETE CASCADE` auf `userId` haben (wird in den jeweiligen Stories sichergestellt). Die aktuelle Implementierung löscht nur die existierenden Daten.

### Bestehende Infrastruktur (aus Story 1-1 bis 1-5)

| Komponente | Datei | Status |
|---|---|---|
| DB Schema | `src/infrastructure/db/schema.ts` | Existiert (users + linked_wallets + verification_tokens) |
| User Repository | `src/infrastructure/db/repositories/user-repository.ts` | Existiert — MUSS um `deleteUser()` und `deleteVerificationTokensByIdentifier()` erweitert werden |
| Linked-Wallet Repository | `src/infrastructure/db/repositories/linked-wallet-repository.ts` | Existiert — `deleteAllLinkedWallets()` vorhanden, wird hier aber NICHT gebraucht (DB Cascade) |
| Settings Page | `src/app/(dashboard)/settings/page.tsx` | Existiert — Server Component, Account-Sektion ist Placeholder-Text |
| WalletList | `src/components/settings/WalletList.tsx` | Existiert — Client Component, Pattern-Referenz für DeleteAccountSection |
| UnlinkWalletDialog | `src/components/settings/UnlinkWalletDialog.tsx` | Existiert — Pattern-Referenz für DeleteAccountDialog (natives `<dialog>`, Loading-Guard, Error-Handling) |
| Toast-System | `src/components/ui/Toast.tsx` + `src/lib/hooks/use-toast.ts` | Existiert — `useToast()` Hook mit 4 Varianten |
| Auth Options | `src/lib/auth/auth-options.ts` | Existiert — JWT-Strategy, 7-Tage maxAge |
| Session Helper | `src/lib/auth/session.ts` | Existiert — `getServerSession()` Wrapper |
| Middleware | `src/middleware.ts` | Existiert — `/settings` im matcher, redirected zu `/auth/signin` ohne Token |
| Format Utils | `src/lib/utils/format.ts` | `truncateAddress()` vorhanden |
| next-auth Client | `next-auth/react` | `signOut()` verfügbar für client-seitige Session-Invalidierung |

### API Design: DELETE /api/account

```typescript
// DELETE /api/account
// Keine Route-Parameter nötig — User-ID kommt aus der Session

// Success Response (200):
{ data: { success: true } }

// Error Responses:
{ error: "Nicht authentifiziert", code: "UNAUTHORIZED" }           // 401
{ error: "Account nicht gefunden", code: "ACCOUNT_NOT_FOUND" }     // 404
{ error: "Löschung fehlgeschlagen", code: "DELETE_FAILED" }        // 500
{ error: "Zu viele Anfragen", code: "RATE_LIMITED" }               // 429
```

### Account-Löschung Flow

```
User klickt "Account löschen" auf Settings-Page
  → DeleteAccountDialog öffnet sich
  → User tippt "LÖSCHEN" in Eingabefeld
  → Confirm-Button wird enabled
  → User klickt "Account löschen"
  → DELETE /api/account
  → Server:
    1. Session prüfen → userId extrahieren
    2. Rate-Limit prüfen (1 req/60s)
    3. User aus DB laden (findUserById)
    4. Falls Email vorhanden → verification_tokens bereinigen
    5. User löschen (DB Cascade entfernt linked_wallets)
    6. Response: { data: { success: true } }
  → Client:
    1. signOut({ callbackUrl: '/' }) — JWT-Cookie löschen
    2. User landet auf Landing Page
```

### Settings-Page UI-Struktur (nach Story 1.6)

```
┌───────────────────────────────────────┐
│  Einstellungen                         │
│                                        │
│  ── Verknüpfte Wallets ──             │
│                                        │
│  ┌─────────────────────────────────┐  │
│  │ ETH  0x1234...5678  Primär      │  │
│  ├─────────────────────────────────┤  │
│  │ SOL  7xKX...4bPq    [Entfernen]│  │
│  └─────────────────────────────────┘  │
│                                        │
│  [ + Wallet hinzufügen ]              │
│                                        │
│  ────────────────────────────────── │  │  ← border-subtle Trenner
│                                        │
│  ── Account ──                        │
│                                        │
│  [ Account löschen ]                  │  ← status-negative Outline-Button
│  Lösche deinen Account und alle       │
│  verknüpften Daten unwiderruflich.    │
└───────────────────────────────────────┘
```

### DeleteAccountDialog Design (gemäß UX-Spec)

```
┌──────────────────────────────────────────┐
│  Account unwiderruflich löschen?          │
│                                           │
│  Alle deine Daten werden unwiderruflich   │
│  gelöscht:                                │
│  • Dein Account                           │
│  • Alle verknüpften Wallets              │
│  • Alle zukünftigen verknüpften Daten    │
│                                           │
│  Diese Aktion kann nicht rückgängig       │
│  gemacht werden.                          │
│                                           │
│  ┌──────────────────────────────────┐    │
│  │ Tippe LÖSCHEN zur Bestätigung   │    │
│  └──────────────────────────────────┘    │
│                                           │
│  [Abbrechen]        [Account löschen]    │
│   Ghost-Style       status-negative      │
│                     (disabled bis Input)  │
└──────────────────────────────────────────┘
```

### Learnings aus Story 1-5 (MUSS beachtet werden)

- **`--legacy-peer-deps` nötig:** Für ALLE npm installs wegen React 19 Peer-Dep-Konflikten
- **`serverEnv` statt `process.env`:** IMMER `serverEnv` aus `@/lib/env` verwenden (Review-Finding H2 aus Story 1-3)
- **Biome 2.2.0:** Alle neuen Dateien müssen Biome-Check bestehen
- **Next.js 15.5.12 (LTS), Tailwind v4:** Custom Tokens in `globals.css` via `@theme`
- **Drizzle `neon-http` Modus:** HTTP-basierter Treiber. Für Account-Delete reicht ein einzelner DELETE Query
- **In-Memory Rate-Limiting:** Upstash Rate Limit nicht installiert. Gleichen In-Memory-Fallback verwenden wie in Story 1-5
- **`signOut()` aus `next-auth/react`:** Client-seitige Funktion, muss in Client Component aufgerufen werden. Akzeptiert `{ callbackUrl: string }` für Redirect nach Logout
- **Provider-Hierarchy:** Web3Provider > AuthProvider > SolanaProvider > ThemeProvider — NICHT ändern
- **Natives `<dialog>` statt Custom Modal:** Konsistent mit UnlinkWalletDialog verwenden. Kein redundantes `aria-modal="true"` (Biome lint, Story 1-5 AC4.7)
- **`onCancel` Handler mit `preventDefault()`:** Verhindert Escape-Key-Schließen während Loading (Story 1-5 H2-Fix)
- **Dialog bleibt bei Fehler offen:** `onClose()` NICHT im catch-Block aufrufen (Story 1-5 H1-Fix)
- **Test-Pattern: Dynamische Imports in `it()` Blöcken:** Module innerhalb von Tests importieren (`const { DELETE } = await import("./route")`) für korrekte Mock-Isolation

### Sicherheitsaspekte

- **Server-seitige Auth-Prüfung (PFLICHT):** Der DELETE-Endpoint MUSS die Session prüfen. Ohne Session → 401.
- **Keine PII in Logs (NFR15):** Weder Wallet-Adressen noch Email-Adressen loggen. Nur gekürzte User-ID.
- **Rate-Limiting auf DELETE-Endpoint:** 1 Request pro User pro 60 Sekunden. Verhindert Brute-Force oder versehentliche Doppel-Löschungen.
- **CSRF-Schutz:** NextAuth Session-basierte Authentifizierung bietet CSRF-Schutz über den Session-Cookie.
- **Physische Löschung (DSGVO Art. 17):** Echtes DELETE, kein Soft-Delete. Nach der Löschung sind die Daten unwiederbringlich.
- **JWT-Cookie muss client-seitig gelöscht werden:** `signOut()` ist die einzige Möglichkeit bei JWT-Strategy. Ohne `signOut()` bleibt der Cookie bis zum Ablauf (7 Tage) gültig, aber API-Calls würden fehlschlagen (User nicht in DB).

### Abgrenzung: Was diese Story NICHT macht

- **Kein Soft-Delete:** Keine `is_deleted` Flags, keine Archivierung. Echte DSGVO-Löschung.
- **Kein Export-vor-Löschung:** Datenexport (DSGVO Art. 20, Recht auf Datenportabilität) ist nicht Teil dieser Story. Kann Post-MVP evaluiert werden.
- **Keine Staking-Daten-Löschung:** Staking-Positionen, VRS-Scores, Yield-Snapshots etc. existieren im aktuellen Epic-1-Stand noch nicht. Zukünftige Tabellen müssen `ON DELETE CASCADE` haben.
- **Kein Admin-Notification:** Admin (Sempre) wird nicht benachrichtigt wenn ein User seinen Account löscht. Kann über Vercel Logs nachverfolgt werden.
- **Keine Wartefrist:** Kein "30-Tage-Gnadenfrist" Pattern. Löschung ist sofort und endgültig.
- **Kein Email-Bestätigung:** Keine Bestätigungs-Email vor der Löschung. Die Eingabe von "LÖSCHEN" im Dialog ist die Bestätigung.

### Project Structure Notes

- `DeleteAccountDialog.tsx` gehört in `src/components/settings/` (neben UnlinkWalletDialog.tsx, AddWalletDialog.tsx)
- `DeleteAccountSection.tsx` gehört in `src/components/settings/`
- DELETE API Route gehört in `src/app/api/account/` (nicht `/api/users/` — der User löscht seinen eigenen Account)
- User-Repository-Erweiterung bleibt in `src/infrastructure/db/repositories/user-repository.ts`
- Domain Layer (`src/domain/`) wird in dieser Story NICHT berührt

### References

- [Source: _bmad-output/planning-artifacts/epics.md#Story 1.6 — Account-Löschung (DSGVO)]
- [Source: _bmad-output/planning-artifacts/epics.md#Epic 1 — FR7]
- [Source: _bmad-output/planning-artifacts/architecture.md#Authentication & Security]
- [Source: _bmad-output/planning-artifacts/architecture.md#Implementation Patterns & Consistency Rules]
- [Source: _bmad-output/planning-artifacts/architecture.md#Architectural Boundaries]
- [Source: _bmad-output/planning-artifacts/prd.md#FR7 — Account-Löschung (DSGVO)]
- [Source: _bmad-output/planning-artifacts/prd.md#NFR15 — Keine PII in Logs]
- [Source: _bmad-output/planning-artifacts/prd.md#NFR32 — DSGVO-konform: Recht auf Löschung]
- [Source: _bmad-output/planning-artifacts/ux-design-specification.md#Modal Pattern — Destructive Confirmations Only]
- [Source: _bmad-output/planning-artifacts/ux-design-specification.md#Button Hierarchy — Destructive Actions]
- [Source: _bmad-output/project-context.md#Security Rules — Read-Only Analytics]
- [Source: _bmad-output/project-context.md#DSGVO Compliance]
- [Source: _bmad-output/implementation-artifacts/1-5-wallet-verwaltung-einstellungen.md — Learnings, File List, Patterns]
- [Source: _bmad-output/implementation-artifacts/1-5-wallet-verwaltung-einstellungen.md#UnlinkWalletDialog Pattern — natives dialog, Loading-Guard, Error-Handling]
- [Source: _bmad-output/implementation-artifacts/1-5-wallet-verwaltung-einstellungen.md#Toast-System — useToast() Hook]
- [Source: _bmad-output/implementation-artifacts/1-5-wallet-verwaltung-einstellungen.md#Senior Developer Review H1+H2 Fixes — Dialog Error/Escape Handling]

## Dev Agent Record

### Agent Model Used

Claude Opus 4.6

### Debug Log References

Keine Debug-Issues aufgetreten.

### Completion Notes List

- Task 1: `deleteUser()` und `deleteVerificationTokensByIdentifier()` zum User-Repository hinzugefügt. DB Cascade auf `linked_wallets.userId` bestätigt (ON DELETE CASCADE im Schema vorhanden). 3 neue Unit-Tests.
- Task 2: DELETE `/api/account` Endpoint erstellt. Session-Auth, In-Memory Rate-Limiting (1 req/user/60s), User-Lookup, bedingte verification_tokens-Bereinigung, User-Löschung mit DB Cascade. Keine PII in Logs (NFR15). 6 Tests.
- Task 3: `DeleteAccountSection` Client Component erstellt. status-negative Outline-Button, eigene `<section>` mit "Account" Heading.
- Task 4: `DeleteAccountDialog` mit nativem `<dialog>` erstellt. Bestätigungswort "LÖSCHEN" Eingabe, isLoading-Guard, Escape/Backdrop blockiert während Loading, Error-Toast bei Fehler, `signOut({ callbackUrl: '/' })` bei Erfolg. 6 Tests.
- Task 5: Settings-Page aktualisiert. Placeholder-Text durch `<DeleteAccountSection />` ersetzt, visuelle Trennung mit `border-t border-border-subtle mt-8 pt-8`.
- Task 6: Alle 15 neuen Tests bestehen. Alle 123 Gesamttests bestehen (108 bestehende + 15 neue). Biome-Check bestanden.

### Change Log

- 2026-02-22: Story 1.6 Account-Löschung (DSGVO) implementiert — User-Repository erweitert, DELETE /api/account Endpoint, DeleteAccountSection + DeleteAccountDialog UI, Settings-Page Integration, 15 neue Tests
- 2026-02-22: Code Review (AI) — 3 HIGH, 3 MEDIUM behoben, 3 LOW akzeptiert. 126 Tests bestehen.

## Senior Developer Review (AI)

**Reviewer:** Sempre (via Claude Opus 4.6 adversarial review)
**Date:** 2026-02-22
**Outcome:** Approved (after fixes)

### Issues Found: 3 High, 3 Medium, 3 Low

**Fixed (6 issues):**

| # | Severity | Issue | Fix |
|---|----------|-------|-----|
| H1 | HIGH | Loeschreihenfolge falsch: verification_tokens VOR User geloescht — bei deleteUser-Fehler sind Tokens weg aber Account bleibt | Reihenfolge umgekehrt: deleteUser zuerst (Cascade loescht linked_wallets), dann Token-Cleanup als best-effort mit eigenem try/catch |
| H2 | HIGH | 3 nicht-existierende CSS-Token-Klassen im DeleteAccountDialog Input: bg-bg-primary, text-text-tertiary, ring-accent-primary | Ersetzt durch existierende Tokens: bg-bg-surface-raised, placeholder:text-text-secondary, focus:ring-brand-primary |
| H3 | HIGH | Kein Test fuer API-Fehler-Pfad im DeleteAccountDialog — showToast nie verifiziert, onClose-Verhalten bei Fehler ungetestet | Neuer Test: "should show error toast and keep dialog open when deletion fails" — verifiziert showToast und onClose-Verhalten |
| M1 | MEDIUM | sprint-status.yaml fehlt in File List | File List ergaenzt |
| M2 | MEDIUM | onClose() vor signOut() aufgerufen — Dialog schliesst sofort, Settings-Page mit geloeschten User-Daten kurz sichtbar | signOut() vor onClose(), isLoading bleibt true bis Redirect, setIsLoading(false) nur im catch |
| M3 | MEDIUM | findUserById hat keinen Unit-Test — kritischer Pfad in DELETE /api/account | 2 neue Tests: findUserById success + not found |

**Nicht gefixt (3 Low-Issues — akzeptabel fuer MVP):**

| # | Severity | Issue | Grund |
|---|----------|-------|-------|
| L1 | LOW | onKeyDown={() => {}} leerer Handler auf dialog | Konsistent mit UnlinkWalletDialog Pattern aus Story 1-5 |
| L2 | LOW | truncateAddress() fuer UUID userId semantisch falsch | Pre-existing (Story 1-4 L3), funktional korrekt |
| L3 | LOW | deleteVerificationTokensByIdentifier Fehler-Handling | Nach H1-Fix jetzt eigener try/catch — Fehler ist non-critical (15min Token-Expiry) |

### Verification

- 21/21 Vitest Test-Dateien bestanden
- 126 Tests total (123 bestehend + 3 neue: findUserById x2, DeleteAccountDialog error-path x1)
- 0 Regressionen
- Biome-Check: sauber

### File List

- `src/infrastructure/db/repositories/user-repository.ts` — Geändert (deleteUser, deleteVerificationTokensByIdentifier hinzugefügt)
- `src/infrastructure/db/repositories/user-repository.test.ts` — Geändert (3 neue Tests, delete Mock hinzugefügt)
- `src/app/api/account/route.ts` — Neu (DELETE Endpoint)
- `src/app/api/account/route.test.ts` — Neu (6 Tests)
- `src/components/settings/DeleteAccountSection.tsx` — Neu (Client Component)
- `src/components/settings/DeleteAccountDialog.tsx` — Neu (Destructive Confirmation Dialog)
- `src/components/settings/DeleteAccountDialog.test.tsx` — Neu (6 Tests)
- `src/app/(dashboard)/settings/page.tsx` — Geändert (DeleteAccountSection eingebunden, Placeholder entfernt)
- `_bmad-output/implementation-artifacts/sprint-status.yaml` — Geändert (Status-Update)
