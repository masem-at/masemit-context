# Story 2.3: Welcome-Screen & Onboarding

Status: done

## Story

As a Tenant nach dem ersten Login,
I want einen Welcome-Screen sehen der mir die wichtigsten Infos zeigt,
So that ich sofort weiss wie ich starten kann.

## Acceptance Criteria

1. **Given** ich logge mich zum ersten Mal ein **When** das Dashboard laedt **Then** sehe ich den Welcome-Screen mit: API Key (masked, aus GET /v1/paywatcher/config/api-key), Deposit Address (aus Config), Webhook URL (aus Config, oder "not configured")

2. **Given** der Welcome-Screen angezeigt wird **Then** sehe ich CTAs: "Configure Webhook" → Settings, "View API Docs" → Docs

3. **Given** ich habe den Welcome-Screen gesehen **When** ich das Dashboard spaeter besuche **Then** sehe ich die regulaere Overview-Seite (localStorage-Flag "onboarding_complete")

- Kein auto-provisioning. Der Tenant existiert bereits (von Mario angelegt).

## Tasks / Subtasks

- [x] Task 1: Welcome-Screen Komponente erstellen (AC: #1, #2)
  - [x] 1.1 `src/components/dashboard/welcome-screen.tsx` erstellen — API Key (masked, ApiKeyCard), Deposit Address (monospace, copy-button), Webhook URL Status
  - [x] 1.2 CTAs: "Configure Webhook" → /dashboard/settings (Webhooks-Bereich), "View API Docs" → /docs
  - [x] 1.3 Deposit Address als Link zu Basescan (neuer Tab)

- [x] Task 2: First-Login Detection (AC: #3)
  - [x] 2.1 localStorage-Flag "onboarding_complete" pruefen
  - [x] 2.2 Wenn Flag nicht gesetzt → Welcome-Screen anzeigen
  - [x] 2.3 Nach Anzeige des Welcome-Screens → Flag setzen

- [x] Task 3: Dashboard Overview Page aktualisieren (AC: #1, #3)
  - [x] 3.1 `src/app/(dashboard)/dashboard/page.tsx` erweitern — Bedingung: onboarding_complete? → Overview : Welcome-Screen
  - [x] 3.2 Bestehende ApiKeyCard Anzeige beibehalten fuer Overview-Modus

- [x] Task 4: Build-Validierung (Alle ACs)
  - [x] 4.1 `npm run build` fehlerfrei
  - [x] 4.2 `npm run type-check` fehlerfrei
  - [x] 4.3 `npm run lint` fehlerfrei

## Dev Notes

### Kontext & Zweck

Dies ist die **dritte Story von Epic 2** im **Managed MVP** Ansatz. Der Tenant wurde von Mario manuell angelegt und hat seinen API Key per Email erhalten. Beim ersten Login im Dashboard soll er einen Welcome-Screen sehen der ihm die wichtigsten Infos zeigt.

**WICHTIG: Kein Auto-Provisioning.** Im alten Ansatz (Story 2.3 alt) wurde automatisch ein paywatcher_config Eintrag erstellt und ein API Key generiert. Das entfaellt komplett — der Tenant existiert bereits im MMS Backend.

**Was diese Story NICHT beinhaltet:**
- Auto-Provisioning (entfaellt — Managed MVP)
- API Key Klartext-Anzeige (Tenant hat Key bereits per Email erhalten)
- curl-Beispiel mit vorausgefuelltem API Key (kommt in Story 3.2)
- "Waiting for first payment..." Spinner (kommt in Story 3.2)

### Bestehender Code-Stand

Die Dashboard Overview Page (`dashboard/page.tsx`) zeigt aktuell nur eine ApiKeyCard im masked-Modus. Es gibt keinen Welcome-Screen, keine Deposit Address Anzeige, keine CTAs. Die Welcome-Screen Logik muss komplett neu implementiert werden.

**Bestehende Hooks die verwendet werden koennen:**
- `useTenant()` — holt TenantConfig (inkl. depositAddress, webhookUrl)
- `useApiKey()` — holt masked API Key + Scopes

### References

- [Source: _bmad-output/planning-artifacts/epics.md#Story 2.3]
- [Source: _bmad-output/planning-artifacts/prd.md#FR12, FR37]
- [Source: _bmad-output/planning-artifacts/ux-design-specification.md#Onboarding]

## Dev Agent Record

### Agent Model Used

Claude Opus 4.6

### Completion Notes List

- Welcome-Screen Komponente (`welcome-screen.tsx`) erstellt mit: masked API Key (via ApiKeyCard), Deposit Address (monospace + copy-button + Basescan-Link), Webhook URL Status (configured/not configured)
- CTAs implementiert: "Configure Webhook" → /dashboard/settings, "View API Docs" → /docs, "Skip to Dashboard"
- First-Login Detection via localStorage-Flag "onboarding_complete" implementiert (lazy useState initializer, konsistent mit sidebar-nav.tsx Pattern)
- Dashboard Overview Page aktualisiert: Bedingung showWelcome? → WelcomeScreen : regulaere Overview
- Bestehende ApiKeyCard Anzeige im Overview-Modus beibehalten
- Build, Type-Check und Lint alle fehlerfrei (pre-existing warning in auth.ts nicht relevant)

### Change Log

- 2026-02-18: Code Review (AI) — 2 HIGH, 2 MEDIUM, 2 LOW gefunden, 5 gefixt (H1, H2, M1, L1, L2), M2 als Known Limitation dokumentiert
- 2026-02-18: Story implementiert — Welcome-Screen Komponente, First-Login Detection, Dashboard Page Integration
- 2026-02-18: Story-File neu geschrieben fuer Managed MVP (alte Story beschrieb Auto-Provisioning Flow)
- 2026-02-16: Alte Story 2.3 (Automatische API Key Erstellung & Anzeige) implementiert und Code-Reviewed (obsolet, Code entfernt)

### File List

- src/app/(dashboard)/dashboard/page.tsx (GEAENDERT) — Welcome-Screen vs Overview Logik, localStorage-Flag Detection, Error States, Loading States
- src/components/dashboard/welcome-screen.tsx (NEU) — Welcome-Screen Komponente mit API Key, Deposit Address, Webhook URL, CTAs

### Senior Developer Review (AI)

**Reviewer:** Code Review Workflow (Claude Opus 4.6)
**Date:** 2026-02-18

**Findings (6 total: 2 HIGH, 2 MEDIUM, 2 LOW):**

- [x] [AI-Review][HIGH] H1: Onboarding-Flag nur bei "Skip to Dashboard" gesetzt, nicht bei CTA-Navigation. AC#3 verletzt. Fix: useEffect setzt Flag bei WelcomeScreen-Render. [page.tsx]
- [x] [AI-Review][HIGH] H2: Flash of wrong content ("Overview kommt in Story 3.2") waehrend apiKey-Loading zwischen tenant.isSuccess und apiKey.data. Fix: Erweiterte Loading-Bedingung. [page.tsx]
- [x] [AI-Review][MEDIUM] M1: Kein Error-State Handling — tenant.isError/apiKey.isError zeigten falschen Fallback. Fix: Separate Error-States hinzugefuegt. [page.tsx]
- [ ] [AI-Review][MEDIUM] M2: "Configure Webhook" CTA → /dashboard/settings redirected zu /dashboard/settings/api-keys (nicht Webhooks-Bereich). Known Limitation — Webhooks-Page existiert noch nicht (Epic 4).
- [x] [AI-Review][LOW] L1: Inkonsistente createdAt-Anzeige — WelcomeScreen zeigte createdAt, Overview nicht. Fix: createdAt an Overview ApiKeyCard uebergeben. [page.tsx]
- [x] [AI-Review][LOW] L2: Dead isLoading Prop — apiKey.isLoading immer false wenn apiKey.data truthy. Fix: Prop entfernt. [page.tsx]

**Result:** 5/6 Issues gefixt, 1 Known Limitation (M2). Build, Type-Check, Lint fehlerfrei nach Fixes.
