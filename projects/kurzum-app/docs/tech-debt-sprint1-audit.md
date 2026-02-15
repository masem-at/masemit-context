# Tech Debt Audit — Sprint 1 (Post-Audit Priorisierung)

**Audit-Datum:** 2026-02-15
**Auditor:** Winston (Architekt), Party Mode
**Priorisierung:** Sempre (2026-02-15)

---

## Sofort fixen (vor/während Sprint 2)

| # | Issue | Severity | Warum jetzt | Datei(en) |
|---|-------|----------|-------------|-----------|
| 4 | **DEFAULT_COMPANY_ID ersetzen** | HIGH | Wird durch Sprint 2 Team-Feature ohnehin ersetzt — muss sauber sein bevor echte User kommen | `lib/auth/user.ts` |
| 3 | **Silent Assignment Failure** | HIGH | Fehler schlucken ist gefährlich — try/catch mit Logging/Fallback zu Inbox | `lib/actions/inbox.ts:177-196`, `app/api/voice/route.ts:116-160` |
| 2 | **Unbounded Queries** | HIGH | Schneller Fix: LIMIT 50 auf alle Listen-Endpoints | `lib/actions/projects.ts:80-88,205-219`, `lib/actions/inbox.ts:14-25` |
| 10 | **Confidence Threshold hardcoded** | MEDIUM | In env var oder config auslagern, Schwelle → 0.75 in Sprint 2 | `app/api/voice/route.ts:118` |
| 8 | **DSGVO Audio Retention Job** | HIGH | MUSS vor Pilot da sein — 90 Tage Auto-Delete ist PRD-Pflicht und FFG-relevant | Schema: `lib/db/schema.ts:169`, Job: noch zu erstellen |

### Details

**#4 — DEFAULT_COMPANY_ID:**
- Aktuell: Alle neuen User werden `00000000-0000-0000-0000-000000000001` (masemIT Testbetrieb) zugewiesen
- Sprint 2 bringt Team-Management (Einladungs-Flow, Company-Registrierung)
- DEFAULT_COMPANY_ID muss durch echte Company-Zuordnung ersetzt werden

**#3 — Silent Assignment Failure:**
- `createProjectAndAssignMessage()` in `inbox.ts` schluckt Assignment-Fehler nach erfolgreicher Projekt-Erstellung
- Voice Pipeline in `route.ts` loggt Assignment-Fehler, gibt aber Success-Response zurück
- Fix: Logging beibehalten, aber Return-Wert um `assignmentFailed: boolean` erweitern

**#2 — Unbounded Queries:**
- `getProjects()`, `getProjectMessages()`, `fetchUnassigned()` haben kein LIMIT
- Fix: `.limit(50)` auf alle Listen-Queries, Pagination in Sprint 2+ UI

**#10 — Confidence Threshold:**
- Hardcoded `0.7` in `app/api/voice/route.ts:118`
- ADR-006 zeigt: TC-03 wurde mit genau 0.70 falsch auto-zugeordnet
- Fix: `AI_AUTO_ASSIGN_CONFIDENCE_THRESHOLD` als Konstante in `lib/ai/prompts.ts` oder env var
- Sprint 2 Wert: 0.75 (eliminiert TC-03 False Positive)

**#8 — DSGVO Audio Retention:**
- Schema hat `audioDeletedAt` Column, aber kein Cron-Job existiert
- DSGVO + PRD verlangen 90-Tage Auto-Delete
- Fix: `scripts/retention-job.ts` + Vercel Cron, `audioDeletedAt` setzen nach Blob-Löschung

---

## Sprint 2 mitlaufen lassen

| # | Issue | Severity | Kontext | Datei(en) |
|---|-------|----------|---------|-----------|
| 5 | **Rate Limiting auf Voice-Endpoint** | MEDIUM | Upstash Rate Limit (z.B. 10 req/min pro User) auf `/api/voice` POST | `app/api/voice/route.ts` |
| 6 | **Timeouts auf Mistral API Calls** | MEDIUM | 30s Timeout per API Call (AbortController oder Promise.race) | `lib/ai/mistral.ts`, `lib/ai/assignment.ts` |
| 7 | **pgEnum statt freier Text** | MEDIUM | `projects.status` und `voiceMessages.status` als pgEnum, bei nächster Schema-Migration | `lib/db/schema.ts` |
| 9 | **getAuthContext Error-Handling** | MEDIUM | Custom `SessionError` Class statt generischem `Error`, unterscheidbare Error-Codes | `lib/auth/session.ts` |

### Details

**#5 — Rate Limiting:**
- Upstash `Ratelimit` existiert bereits für Auth (`lib/rate-limit.ts`)
- Gleiche Infrastruktur für Voice-Endpoint nutzen
- Empfehlung: `slidingWindow(10, "1m")` pro userId

**#6 — Timeouts:**
- Aktuell: Kein Timeout auf `transcribe()`, `summarize()`, `assignToProject()`
- Wenn Mistral hängt, hängt der gesamte Request bis Vercel-Timeout
- Fix: `AbortController` mit 30s Timeout per Call

**#7 — pgEnum:**
- `projects.status`: "active" | "paused" | "completed" — als freier Text, kein DB-Level Constraint
- `voiceMessages.status`: "pending" | "uploaded" | "transcribed" | "done" | "partial" | "error"
- Fix: Drizzle `pgEnum()` in nächster Migration

**#9 — getAuthContext:**
- Wirft generisches `Error("No session cookie found")`, `Error("Invalid or expired session")`
- Server Actions zeigen generische Fehlermeldung
- Fix: `SessionError` Class mit Codes (`MISSING_COOKIE`, `EXPIRED`, `NO_COMPANY`)

---

## Bewusst aufgeschoben

| # | Issue | Entscheidung | Begründung |
|---|-------|-------------|-----------|
| 1 | **Zero Tests** | Aufgeschoben bis Pilot-Start | Sprint 0 Entscheidung: kein Test-Framework für Prototyp. Kommt wenn der Code stabil sein muss und Pilot startet. Velocity nicht opfern. |

### Kontext zu #1:
- Keine `.test.ts` oder `.spec.ts` Dateien im Projekt
- Kein vitest/jest konfiguriert
- Bewusste Entscheidung für Prototyp-Phase: Velocity > Coverage
- Zeitpunkt: Vor Pilot-Start (Q4 2026) Test-Suite einrichten
- Priorität dann: Tenant Isolation, Zod Schemas, AI Response Parsing, Session Management

---

## Weitere Findings (LOW, kein Action nötig)

| Finding | Severity | Status |
|---------|----------|--------|
| Foreign Keys ohne ON DELETE Behavior | LOW | Akzeptiert für Prototyp |
| `role` gelesen aber nie geprüft (kein RBAC) | LOW | Kommt mit Sprint 2 Team-Feature |
| Doppelte Session-Validierung (Middleware + getAuthContext) | LOW | Performance akzeptabel für Prototyp |
| `VoiceSummary` Interface in schema.ts statt validations | LOW | Refactoring wenn nötig |
| `as any` Cast in Form Resolvers (Zod v4 + RHF) | LOW | Bekanntes Issue, Runtime funktioniert |
| Kein README.md | LOW | Paige-Audit Empfehlung, separater Track |
| Missing indexes auf projectMembers | LOW | Tabelle hat <100 Rows |

---

## Quellen

- Winston Tech Debt Audit (3 parallele Runs, konsolidiert)
- Paige FFG Documentation Audit (2 parallele Runs, konsolidiert)
- ADR-006 Evaluierungsergebnisse (Confidence-Threshold-Analyse)
- Sempre Priorisierung (2026-02-15)
