---
stepsCompleted: ['step-01-init', 'step-02-discovery', 'step-03-success', 'step-04-journeys', 'step-05-domain', 'step-06-innovation', 'step-07-project-type', 'step-08-scoping', 'step-09-functional', 'step-10-nonfunctional', 'step-11-polish', 'step-12-complete']
status: 'complete'
completedAt: '2026-02-13'
lastEdited: '2026-02-13'
classification:
  projectType: 'SaaS B2B (PWA)'
  domain: 'Communication/Collaboration — Handwerk KMU'
  complexity: 'medium-high'
  projectContext: 'brownfield (Sprint 0 complete)'
inputDocuments:
  - 'docs/_masemIT/bmad-briefing-kurzum-sprint1.md'
  - '_bmad-output/planning-artifacts/prd.md'
  - '_bmad-output/planning-artifacts/product-brief-kurzum-app-2026-02-08.md'
  - 'docs/project-context.md'
  - 'docs/architecture-sprint0-supplement.md'
  - 'docs/ux-sprint0-app-specs.md'
  - 'docs/_masemIT/bmad-briefing-kurzum-sprint0.md'
  - 'docs/_masemIT/kurzum-bmad-do-dont-liste.md'
  - 'docs/decisions.md'
  - 'docs/research/stt-evaluation-voxtral-2026-02.md'
  - 'docs/research/llm-summarization-evaluation-2026-02.md'
documentCounts:
  productBriefs: 1
  research: 2
  brainstorming: 0
  projectDocs: 8
workflowType: 'prd'
---

# Product Requirements Document - kurzum.app Sprint 1

**Author:** Sempre
**Date:** 2026-02-13
**Parent PRD:** `_bmad-output/planning-artifacts/prd.md` (product-level, 2026-02-08)
**Sprint Briefing:** `docs/_masemIT/bmad-briefing-kurzum-sprint1.md`

## Executive Summary

Sprint 0 proved: Speak → Transcribe → Summarize works. Sprint 1 makes it useful: voice messages are automatically assigned to the right construction project by AI.

**Sprint 1 Core Hypothesis (FFG Research Question #3):** A general-purpose LLM (Mistral) can assign voice messages to construction projects using context matching — project list, customer names, addresses, and recent message history. No custom-trained model, no NLP classifier.

**Sprint 1 Scope:** Projects/construction sites CRUD, AI project assignment with confidence scoring, "No Assignment" Inbox for manual correction, project-based dashboard. Single-user mode (developer as Meister).

**Explicitly NOT in Sprint 1:** Team management, photo capture, offline sync, RBAC enforcement, subscription/billing. These are deferred to Sprint 2+.

**Demo pitch after Sprint 1:** "Dein Monteur spricht rein. Die KI ordnet es dem richtigen Projekt zu. Du siehst im Dashboard was auf welcher Baustelle passiert ist. Wenn die KI unsicher ist, landet es in deiner Inbox."

## Success Criteria

### User Success

**Stefan (Meister/Inhaber) — Primary Target:**
- "Aha moment" at first demo: field worker speaks → AI summary appears → auto-assigned to correct project — "Das brauche ich."
- Morning overview: opens dashboard, sees per-project activity — no phone calls needed
- Correction feels natural: when AI is wrong, one click in Inbox → assigned to project → done

**Markus (Monteur) — Secondary:**
- Workflow unchanged from Sprint 0: Speak → Send → Done. Project assignment happens invisibly in the background
- Confirmation feedback: "Zugeordnet zu: Neubau Urfahr ✓" — builds trust in AI

### Business Success

| Timeframe | Goal | Metric |
|-----------|------|--------|
| **Sprint 1 complete** | Demo convinces pilot companies | ≥3 of 5 say "Ja, will ich testen" |
| **FFG R&D evidence** | Accuracy metrics tracked from day 1 | Learning progress documentable for final report |
| **Scope discipline** | Focus on core differentiator | Team + Photos deferred → Sprint 2 |
| **ADR documentation** | ADR-006 Project Assignment | Evaluation methodology + results documented |

### Technical Success

| Metric | Sprint 1 Target | FFG End Goal |
|--------|-----------------|--------------|
| **AI assignment accuracy** | ≥60% (test projects) | ≥75% (2 weeks real data) |
| **Accuracy with user corrections** | Tracking implemented | >85% (PRD FR12) |
| **Pipeline latency (STT → Summary → Assignment)** | <45 seconds | <30 seconds |
| **Confidence routing** | Threshold 0.7 correctly routes Auto-Assign vs. Inbox | Optimized from field data |
| **Tenant isolation** | 0 cross-company data leaks | 0 |
| **Correction tracking** | assignedBy (ai/user), confidence, reasoning stored | Feedback loop for model improvement |

### Measurable Outcomes

**Sprint 1 Go/No-Go for Pilot Preparation:**

| Gate | Criterion | Threshold |
|------|-----------|-----------|
| **Assignment works** | AI assigns correctly (test corpus) | ≥60% accuracy |
| **Correction works** | Every inbox item assignable | 100% completion |
| **Pipeline stable** | Extended pipeline without regression | 0 new error types vs. Sprint 0 |
| **Demo-ready** | Happy path end-to-end demonstrable | Speak → Summary → Project → Dashboard |
| **R&D documented** | ADR-006 + evaluation matrix present | Yes/No |

Decision point: 4 of 5 gates passed → GO for pilot preparation (Sprint 2). <4 → Reassess scope or extend sprint.

## User Journeys

### Journey 1: Markus — "Die KI weiß, wo ich bin" (Happy Path)

**Opening Scene:** Markus, Elektrotechniker bei Elektro Maier, steht im Obergeschoss eines Neubaus in Urfahr. Er hat gerade die Unterverteilung fertig verdrahtet — 24 Sicherungsautomaten, ein FI-Schalter 40A. Normalerweise würde er das abends in irgendeinen Zettel kritzeln oder seinem Chef eine WhatsApp schicken. Stattdessen drückt er den orangenen Aufnahme-Button.

**Rising Action:** "Unterverteilung OG fertig. 24 B16-Automaten und ein FI 40er eingebaut. Morgen komm ich zum Zählerkasten runter." — 12 Sekunden. Er tippt "Senden". Ein Spinner zeigt drei Schritte: Upload, Spracherkennung, Zusammenfassung.

**Climax:** Nach wenigen Sekunden erscheint die KI-Zusammenfassung:
- Status: Abgeschlossen — Unterverteilung OG
- Material: 24x B16, 1x FI 40A
- Nächster Schritt: Zählerkasten EG
- **Zugeordnet zu: Neubau Urfahr, Fam. Gruber ✓**

Markus hat weder ein Projekt ausgewählt noch irgendetwas getippt. Die KI hat aus dem Kontext — "Unterverteilung OG", seine letzten Nachrichten waren alle zum gleichen Projekt — die Zuordnung erkannt.

**Resolution:** Markus steckt das Handy weg. Für ihn hat sich gegenüber Sprint 0 nichts geändert: Sprechen → Senden → Fertig. Aber im Hintergrund ist seine Nachricht jetzt automatisch dem Projekt zugeordnet und erscheint in der Timeline. Stefan sieht sie sofort im Dashboard unter "Neubau Urfahr".

**Requirements revealed:** S1-FR6–10 (AI assignment), S1-FR13 (project timeline).

### Journey 2: Stefan — "Die KI ist unsicher, ich nicht" (Correction Flow)

**Opening Scene:** Stefan, Inhaber von Elektro Maier, öffnet morgens sein Dashboard. Im Navigationselement sieht er einen Badge: "2 nicht zugeordnet". Ein neuer Lehrling, Thomas, hat gestern zwei Sprachnachrichten aufgenommen — aber er hat weder einen Kunden noch ein Projekt erwähnt, nur gesagt: "Kabelkanal verlegt, 12 Meter, alles sauber."

**Rising Action:** Stefan tippt auf den Inbox-Badge. Er sieht zwei Karten:

Karte 1: Thomas, gestern 15:30 — "Kabelkanal verlegt, 12 Meter, alles sauber." KI-Vermutung: Keines (Confidence 0.4). Stefan weiß sofort: Thomas war gestern bei der Sanierung Pfarrgasse. Er tippt "Zuordnen" → Dropdown → "Sanierung Pfarrgasse, Hr. Wimmer" → Fertig. Die Nachricht verschwindet aus der Inbox und erscheint in der Projekt-Timeline.

Karte 2: Thomas, gestern 16:45 — "Steckdosen gesetzt in der Küche, 5 Stück doppelt." KI-Vermutung: Keines (Confidence 0.3). Stefan erkennt: Das ist ein neuer Auftrag, den er noch nicht angelegt hat. Er tippt "Neues Projekt". Die KI hat aus der Zusammenfassung vorbefüllt: Kundenname leer, Beschreibung "Steckdosen-Installation Küche". Stefan ergänzt: "Fam. Berger, Mozartstraße 12" → Speichern. Nachricht zugeordnet, Projekt erstellt.

**Climax:** Die Inbox ist leer. Zwei Klicks, ein neues Projekt. Kein Telefonat mit Thomas nötig. Stefan denkt: "Der Lehrling soll einfach nächstes Mal den Kunden erwähnen — dann macht die KI das automatisch."

**Resolution:** Stefan hat in 30 Sekunden zwei offene Nachrichten verarbeitet. Die Korrektur-Daten (assignedBy: 'user') werden gespeichert — über Zeit lernt das System, dass Thomas-Nachrichten mit "Kabelkanal" zum Pfarrgasse-Projekt gehören. Der Badge zeigt "0".

**Requirements revealed:** S1-FR16–20 (Inbox), S1-FR11 (project creation), S1-FR24 (correction tracking).

### Journey 3: Stefan — "Was lief gestern auf meinen Baustellen?" (Dashboard Overview)

**Opening Scene:** Dienstagmorgen, 6:45 Uhr. Stefan sitzt mit Kaffee am Küchentisch. Früher hätte er jetzt 4-5 WhatsApp-Chats durchgescrollt und 2 Monteure angerufen, um zu wissen, was gestern passiert ist. Heute öffnet er das kurzum-Dashboard.

**Rising Action:** Das Dashboard zeigt eine Projekt-Karte pro aktiver Baustelle:

| Projekt | Letzte Aktivität | Nachrichten gestern |
|---------|-----------------|---------------------|
| Neubau Urfahr, Fam. Gruber | Gestern, 16:30 | 3 |
| Sanierung Pfarrgasse, Hr. Wimmer | Gestern, 15:30 | 2 |
| Büroumbau Industriezeile | Montag, 14:00 | 0 |

Stefan tippt auf "Neubau Urfahr". Die Projekt-Timeline zeigt chronologisch:
- 16:30 Markus: "Unterverteilung OG fertig. 24x B16, 1x FI 40A. Morgen Zählerkasten."
- 14:15 Markus: "EG Dosen fertig, 18 Stück. Ring hat gepasst."
- 08:20 Markus: "Angefangen EG, Dosen setzen. Material passt."

**Climax:** In 3 Minuten hat Stefan ein vollständiges Bild aller drei Baustellen. Keine Telefonate, kein Scrollen durch Chatverläufe.

**Resolution:** Stefan schickt Markus keine Nachricht — er weiß alles. Beim Büroumbau ruft er kurz an, weil da seit Montag nichts passiert ist. 3 Minuten statt 30. Der Kaffee ist noch warm.

**Requirements revealed:** S1-FR21–22 (dashboard), S1-FR13 (project timeline), S1-FR15 (project list).

### Journey 4: Mario — "Forschungs-Modus" (R&D Evaluation)

**Opening Scene:** Mario (Entwickler, einziger Sprint-1-User) hat die KI-Zuordnungs-Pipeline implementiert. Die FFG erwartet im Endbericht eine dokumentierte Evaluierung mit Accuracy-Metriken und Lernfortschritt.

**Rising Action:** Mario legt 5 Testprojekte an, die den STT-Evaluierungs-Szenarien aus ADR-004 entsprechen. Er spielt die 8 Beispiel-Transkripte über die File-Upload-Funktion (Story 7.2) ein und dokumentiert pro Nachricht: erwartetes Projekt, KI-zugeordnetes Projekt, Confidence-Score, korrekt Ja/Nein.

**Climax:** Die Ergebnismatrix zeigt: 5 von 8 korrekt (62.5%) — über dem Sprint-1-Ziel von ≥60%. Die Fehler werden analysiert: Dialekt-schwere Aufnahme ohne Projektbezug (→ Inbox, korrekt als "unsicher"), mehrdeutiger Fall mit zwei passenden Projekten. Alles dokumentiert in ADR-006.

**Resolution:** Die Accuracy-Baseline steht. Bei jeder Prompt-Verbesserung kann der gleiche Testkorpus erneut durchlaufen werden — genau was die FFG im Endbericht sehen will.

**Requirements revealed:** S1-FR23–26 (R&D traceability), S1-FR11 (project CRUD for test data).

### Journey Requirements Summary

| Capability | Journeys | Priority |
|------------|----------|----------|
| AI Project Assignment (auto at ≥0.7 confidence) | J1, J2, J4 | MVP-Critical |
| Confidence display + confirmation UI | J1, J2 | MVP-Critical |
| Inbox page (projectId=null filter) | J2 | MVP-Critical |
| Manual assignment from Inbox | J2 | MVP-Critical |
| AI-prefilled project creation from Inbox | J2 | MVP |
| Inbox badge/counter in navigation | J2, J3 | MVP |
| Project CRUD (create, edit, list) | J2, J3, J4 | MVP-Critical |
| Project detail page with timeline | J1, J3 | MVP-Critical |
| Project-based dashboard view | J3 | MVP |
| Correction tracking (assignedBy, confidence) | J2, J4 | MVP-Critical (FFG) |
| File upload for test audio | J4 | Done (Story 7.2) |
| ADR-006 evaluation documentation | J4 | MVP (FFG requirement) |

## Project Scoping & Phased Development

### MVP Strategy

**Approach:** Problem-Solving MVP — validate one core hypothesis: "Can an LLM assign voice messages to the correct construction project using context matching?"

**Resource model:** Solo developer with AI-assisted development (Claude Code). Traditional time estimates do not apply.

### MVP Feature Set (Sprint 1)

**Core Journeys Supported:** J1 (Happy Path), J2 (Correction), J3 (Dashboard), J4 (Research)

**Must-Have Capabilities:**

| Capability | Rationale |
|------------|-----------|
| Project CRUD | Without projects, nothing to assign to |
| AI Project Assignment | Core hypothesis — the entire sprint exists for this |
| Confidence-based routing (≥0.7 auto, <0.7 Inbox) | Separates confident matches from uncertain ones |
| "No Assignment" Inbox | **The fallback IS the feature** — handles 100% of messages at any accuracy level |
| Manual assignment from Inbox | Meister must be able to fix what AI got wrong |
| AI-prefilled project creation | Inbox → "New Project" must be fast (1-click + confirm) |
| Project-based dashboard | Stefan's morning overview — the business value demonstration |
| Project detail timeline | Chronological view of all messages per project |
| Correction tracking | assignedBy, confidence, aiSuggestedProjectId — FFG requirement |
| Prompt versioning | llmPromptVersion in DB — FFG traceability |
| ADR-006 | Evaluation methodology and baseline results |

### Graceful Degradation Strategy

| AI Accuracy | User Experience | Sprint Impact |
|-------------|-----------------|---------------|
| **≥60%** | Most messages auto-assigned, few in Inbox | Sprint goal met |
| **40–59%** | Mixed — some auto, many in Inbox | Still functional. FFG: "honest baseline, improvement plan documented" |
| **20–39%** | Almost everything in Inbox | Product works as manual assignment tool |
| **<20% or JSON failures** | Pipeline broken | **Blocker.** Not expected given ADR-005 (96.3% JSON validity) |

**Key insight:** The Inbox is not an error state — it's the safety net that makes the system viable at any accuracy level.

### Post-MVP Roadmap

**Sprint 2 (Growth):**
- Team Management (company creation, invite flow, role assignment)
- RBAC enforcement (middleware guards for Monteur/Meister/Büro)
- Photo Capture (camera, attach to message/project)
- Tenant isolation enforcement (multi-user reality)

**Sprint 3+ (Expansion):**
- Offline-first with Service Worker sync
- Learning from corrections (improved accuracy over time)
- Subscription/billing
- Multi-trade support, DACH expansion

### Risk Mitigation

| Risk | Likelihood | Impact | Mitigation |
|------|-----------|--------|------------|
| AI accuracy <60% | Medium | Low | Inbox handles all messages. Document honest results for FFG |
| AI accuracy <20% / JSON failures | Low | High | Prompt redesign. ADR-005 baseline (96.3% JSON) makes this unlikely |
| Confidence scores not discriminative | Low | Medium | Manual threshold adjustment, user feedback signal |
| Pipeline latency >45s | Low | Medium | Reduce context window (fewer projects, shorter history) |
| Schema migration breaks Sprint 0 | Low | High | Drizzle migration, test on branch before main |
| Prompt changes invalidate results | Medium | Medium | Prompt versioning with version constants in code |
| Scope creep (Team/Photos sneak in) | Medium | Medium | Clear Sprint 2 boundary enforced |

## Domain-Specific Requirements

### FFG Research Methodology — AI Assignment as Research Object

The AI project assignment is not just a feature but **FFG Research Question #3**: "Automatische Projekt-Zuordnung durch kontextuelles Matching." This requires scientific rigor beyond normal feature development.

- **Baseline measurement** with the 8 test transcripts from STT evaluation (ADR-004) before any prompt optimization
- **Documented accuracy per prompt version** — every prompt change produces a new evaluation run against the same test corpus
- **Cross-sprint comparability** — test corpus and methodology remain stable, enabling FFG final report to show learning progress
- **ADR-006** documents: evaluation methodology, test corpus description, prompt version, accuracy matrix, error analysis
- **Reproducible tests** — file upload (Story 7.2) enables re-running the same audio files at any time

**Risk:** Accuracy target not met → research question not answered → FFG report weakened.
**Mitigation:** Track from day 1, document every attempt. Even failed experiments have R&D value.

### Prompt Versioning — Traceable AI Pipeline Results

- All prompts in `lib/ai/prompts.ts` with version constants (e.g., `ASSIGNMENT_PROMPT_V1`)
- Version string stored in DB: `llmPromptVersion` field in `voice_messages` table
- Every voice message records which prompt version produced its assignment
- Not required for Sprint 1: prompt management UI, A/B testing, dynamic prompt loading

### Tenant Isolation — Middleware-Based Company Filtering

Sprint 0 was single-user. Sprint 1 introduces `companyId` as the tenant boundary.

- Extract `companyId` from authenticated session
- Inject `companyId` filter into every database query
- **Every query without companyId filter is a bug** — no exceptions
- DB-level RLS deferred to later sprint when real multi-tenant load exists
- Decision rationale: middleware is simpler to implement, debug, and test in single-developer context

## Innovation & Novel Patterns

### Sprint 1 Innovation Focus: LLM-Based Context Matching (Research Question #3)

**Core hypothesis:** A general-purpose LLM (Mistral) can assign voice messages to the correct construction project using only contextual data — project list, customer names, addresses, and the speaker's recent message history. No custom-trained model, no NLP classifier, no embeddings database.

**Why this is novel:** Existing trades communication tools (WhatsApp, Benetics AI) require manual project tagging or pre-selection before recording. kurzum.app reverses the paradigm: speak first, let AI figure out the context. This transforms "communication with manual filing" into "incidental documentation."

**What Sprint 1 validates:**
- Can context-matching achieve ≥60% accuracy with 5 test projects and 8 test transcripts?
- Does confidence scoring (0.0–1.0) reliably separate correct assignments from uncertain ones?
- Is the 0.7 threshold appropriate, or does it need tuning?
- How does accuracy change with ambiguous inputs (no project mention, multiple possible projects)?

### Validation Approach

| Step | Method | Output |
|------|--------|--------|
| **Baseline** | 8 test transcripts × 5 test projects, ASSIGNMENT_PROMPT_V1 | Accuracy matrix in ADR-006 |
| **Error analysis** | Classify failures (ambiguous, no context, wrong match) | Error taxonomy |
| **Prompt iteration** | Modify prompt, re-run same corpus, compare | Accuracy delta per version |
| **Threshold tuning** | Analyze confidence distribution across correct/incorrect | Optimal threshold recommendation |

## SaaS B2B Specific Requirements

### Project-Type Overview

kurzum.app is a SaaS B2B product (PWA) targeting trades SMEs. Sprint 1 operates in **single-tenant, single-user mode** (developer as Meister). Multi-tenant architecture prepared in data model, not enforced at runtime until Sprint 2.

### RBAC Model

**Sprint 1 approach:** Schema-ready, enforcement-deferred.

- `role` field on `users` table: `monteur` | `meister` | `buero`
- Developer account set to `meister` (full access)
- No middleware enforcement — all routes accessible
- Sprint 2 adds role-based guards when users with different roles exist

**RBAC matrix (for reference, enforced in Sprint 2):**

| Action | Monteur | Meister | Büro |
|--------|---------|---------|------|
| Record voice message | Yes | Yes | Yes |
| View projects | Assigned only | All | All |
| Create/edit projects | No | Yes | Yes |
| View Inbox | No | Yes | Yes |
| Correct assignment | No | Yes | Yes |
| Dashboard (all projects) | No | Yes | Yes |

**Preparation for Sprint 2:** API routes and DB queries structured so adding a role-based guard is a one-line addition, not a refactor.

### Implementation Considerations

- **Session includes `companyId` and `role`** from Sprint 1 — data flows through the system even if not enforced
- **All new DB tables include `companyId`** as required field — pattern established now prevents migration pain
- **API routes accept but don't enforce role context** — `getSession()` returns `{ userId, companyId, role }`, Sprint 2 adds authorization

## Functional Requirements

### Voice Recording & AI Processing (Baseline)

- **S1-FR1:** User can record a voice message via single tap (Sprint 0, ref PRD FR1)
- **S1-FR2:** System transcribes voice messages using Mistral Voxtral STT, EU-hosted (Sprint 0, ref PRD FR8)
- **S1-FR3:** System generates structured JSON summary from transcription using Mistral LLM (Sprint 0, ref PRD FR9)
- **S1-FR4:** System displays processing state with step indicators during pipeline execution (Sprint 0, ref PRD FR14)
- **S1-FR5:** User can play back recorded audio before sending (Sprint 0, ref PRD FR7)

### AI Project Assignment

- **S1-FR6:** System automatically assigns a voice message to the most likely project based on transcript content, active project list, and speaker's recent message history (Neu, ref PRD FR10)
- **S1-FR7:** System produces a confidence score (0.0–1.0) for each assignment decision (Neu)
- **S1-FR8:** System auto-assigns messages with confidence ≥0.7 and routes messages with confidence <0.7 to the Inbox (Neu)
- **S1-FR9:** User can see which project a message was assigned to and the confidence level after processing (Neu, ref PRD FR11)
- **S1-FR10:** User can correct an AI assignment by reassigning a message to a different project (Neu, ref PRD FR11)

### Project Management

- **S1-FR11:** Meister can create a new project with name, customer name, address, and description (Neu, ref PRD FR15)
- **S1-FR12:** Meister can edit project details (Neu, ref PRD FR16)
- **S1-FR13:** Meister can view a project's detail page with chronological timeline of all assigned messages (Neu, ref PRD FR17)
- **S1-FR14:** Meister can change a project's status (active, paused, completed) (Neu)
- **S1-FR15:** Meister can view a list of all projects with last activity and message count (Neu)

### Assignment Inbox

- **S1-FR16:** Meister can access an Inbox showing all voice messages without project assignment (Neu, ref PRD FR19)
- **S1-FR17:** Meister can manually assign an unassigned message to an existing project from the Inbox (Neu, ref PRD FR21)
- **S1-FR18:** Meister can create a new project from the Inbox with fields pre-filled by AI from the message summary (Neu, ref PRD FR20)
- **S1-FR19:** System displays an Inbox badge/counter in the navigation showing unassigned message count (Neu)
- **S1-FR20:** Assigned messages disappear from Inbox and appear in the project timeline (Neu)

### Dashboard

- **S1-FR21:** Meister can view a project-based dashboard showing all active projects with latest activity and daily message count (Neu, ref PRD FR31)
- **S1-FR22:** Meister can navigate from a dashboard project card to the project detail timeline (Neu)

### R&D Evaluation & Traceability

- **S1-FR23:** System stores prompt version identifier with each voice message's AI results (Neu, FFG requirement)
- **S1-FR24:** System stores assignment metadata per message: assignedBy (ai/user), confidence score, AI-suggested project ID, reasoning (Neu, FFG requirement)
- **S1-FR25:** User can upload existing audio files for pipeline processing via file picker (Sprint 0 — Story 7.2, done)
- **S1-FR26:** System accepts audio/mpeg (.mp3) files alongside existing audio formats (Sprint 0 — Story 7.2, done)

### Data Architecture Preparation

- **S1-FR27:** All new database tables include companyId as required foreign key (Neu, Sprint 2 preparation)
- **S1-FR28:** User model includes role field (monteur/meister/buero) populated at account level (Neu, Sprint 2 preparation)
- **S1-FR29:** Session context includes userId, companyId, and role for all authenticated requests (Neu, Sprint 2 preparation)

## Non-Functional Requirements

### Performance

| ID | Requirement | Target | Context |
|----|-------------|--------|---------|
| **S1-NFR1** | Extended pipeline latency (STT → Summary → Assignment) | <45 seconds | Sprint 0 was <30s for STT+Summary. Assignment adds one LLM call with project context |
| **S1-NFR2** | Dashboard load with project cards | <2 seconds | Stefan's morning overview must feel instant |
| **S1-NFR3** | Inbox page load | <2 seconds | Few items expected (<20 unassigned messages) |
| **S1-NFR4** | Project timeline load | <3 seconds | Chronological query with message joins |

### Security & Data

| ID | Requirement | Target | Context |
|----|-------------|--------|---------|
| **S1-NFR5** | Data residency | 100% EU for all new data (projects, assignments) | Extends Sprint 0 EU-only principle to new tables |
| **S1-NFR6** | Company data isolation | Every DB query on new tables filtered by companyId | Middleware-enforced, no cross-tenant leakage |
| **S1-NFR7** | AI API keys | Server-side only, never exposed to client | Extends Sprint 0 principle to assignment LLM call |
| **S1-NFR8** | Prompt content security | No user PII in prompt logs or error messages | Transcripts may contain personal mentions |

### Reliability

| ID | Requirement | Target | Context |
|----|-------------|--------|---------|
| **S1-NFR9** | Pipeline regression | 0 new failure modes vs. Sprint 0 | Assignment step must not break existing STT+Summary flow |
| **S1-NFR10** | Graceful degradation | Assignment failure → message saved with projectId=null (Inbox) | Assignment is never a blocking failure |
| **S1-NFR11** | JSON parse reliability | >95% valid JSON from assignment LLM response | ADR-005 baseline: 96.3% for summary |

### Accessibility (Carried from Sprint 0)

| ID | Requirement | Target | Context |
|----|-------------|--------|---------|
| **S1-NFR12** | Touch targets | ≥48px for all new interactive elements | Inbox buttons, project cards, assignment dropdown |
| **S1-NFR13** | Contrast ratio | WCAG AA (4.5:1) for all new UI | Dashboard cards, Inbox, project timeline |
