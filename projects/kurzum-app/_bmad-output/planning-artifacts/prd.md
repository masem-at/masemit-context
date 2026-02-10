---
stepsCompleted: ['step-01-init', 'step-02-discovery', 'step-03-success', 'step-04-journeys', 'step-05-domain', 'step-06-innovation', 'step-07-project-type', 'step-08-scoping', 'step-09-functional', 'step-10-nonfunctional', 'step-11-polish', 'step-12-complete']
status: 'complete'
lastEdited: '2026-02-08'
editHistory:
  - date: '2026-02-08'
    changes: 'Validation-driven fixes: FR8 implementation leakage removed, FR12 measurability added, Made in Austria positioning added'
classification:
  projectType: 'SaaS B2B (PWA)'
  domain: 'Communication/Collaboration — Handwerk KMU'
  complexity: 'medium'
  projectContext: 'greenfield'
inputDocuments:
  - '_bmad-output/planning-artifacts/product-brief-kurzum-app-2026-02-08.md'
  - '_bmad-output/planning-artifacts/research/domain-interne-kommunikation-kmu-handwerk-research-2026-02-08.md'
  - '_bmad-output/planning-artifacts/research/market-ki-kommunikation-kmu-handwerk-research-2026-02-08.md'
  - '_bmad-output/planning-artifacts/research/foerderungs-leitfaden-kurzum-app-2026.md'
  - 'docs/project-context.md'
  - 'docs/company-info.md'
documentCounts:
  productBriefs: 1
  research: 3
  brainstorming: 0
  projectDocs: 2
workflowType: 'prd'
---

# Product Requirements Document - kurzum.app

**Author:** Sempre
**Date:** 2026-02-08

## Executive Summary

kurzum.app is an AI-powered, GDPR-compliant communication app for trades companies (Handwerksbetriebe) with field workers in the DACH region. It replaces illegal WhatsApp usage and after-work paperwork with a single principle: **"Task done → voice message → done."**

**Core Differentiator:** "Incidental Documentation" — communication IS the documentation. Field workers speak, AI transcribes, summarizes, and auto-assigns to projects. No typing, no forms, no evening paperwork.

**Target Users:** Electrical and HVAC/plumbing trades companies (5-50 employees) in Austria, expanding to DE/CH. Three personas: Markus (field worker, daily user), Stefan (owner/master craftsman, buyer & power user), Andrea (office admin, data consumer).

**Market Opportunity:** TAM ~€425M/year, SAM ~€99M/year, 87% of ~708,000 DACH trades companies have no dedicated communication solution. Sole direct competitor (Benetics AI) targets large construction at 2x the price.

**Business Model:** SaaS — Free (3 users) / €10/user/month (Team) / €8/user/month (Business). Year 1 target: €120k ARR. Year 3: €1.8M ARR.

**Technology:** PWA (Vercel), NeonDB, Mistral AI (EU-hosted for GDPR), Upstash/QStash for queuing. Offline-first architecture for construction site conditions.

**Positioning:** Made in Austria — built by a solo founder who understands the trades SME reality. Regional identity and DACH trust as competitive advantage vs. US-cloud competitors.

**MVP:** 6 core features — Voice Recording & AI Processing, AI Project Assignment with "No Assignment" inbox, Team Dashboard, Basic Photo Capture, Team Management, PWA with offline sync. Validated with 5-10 pilot companies in Austria.

## Success Criteria

### User Success

**Markus (Monteur/Techniker):**
- Zero after-work documentation: 0 minutes paperwork after end of shift (from ~30-60 min)
- Voice message duration <60 seconds per task
- Voluntary daily adoption without reminders from management
- "Aha moment" within first use: records voice → sees AI summary → "That's it?"

**Stefan (Meister/Inhaber):**
- Morning overview of all job sites in <5 minutes (from 30+ min phone rounds)
- 50% reduction in incoming WhatsApp/calls from field workers
- Can be absent 2 days without business standstill
- No longer the "human database" bottleneck

**Andrea (Büro):**
- Billing/accounting time halved
- Zero follow-up calls to field workers for missing information
- Complete time tracking data without manual rework

### Business Success

| Timeframe | Goal | Metric |
|-----------|------|--------|
| **3 months** (Pilot) | Product works, first companies use it | 5-10 pilot companies active, >70% daily usage by field workers |
| **6 months** | Product-Market Fit confirmed | NPS >40, <5% monthly churn, first paying customers |
| **12 months** (Year 1) | €120k ARR, market position AT | ~100 paying companies, guild partnership |
| **36 months** (Year 3) | €1.8M ARR, DACH expansion | Elektro + SHK covered, DE market entry |

**Key KPIs:**
- DAU/MAU (field workers): >60%
- Voice messages per worker per day: >3
- Time-to-first-value: <5 minutes
- Free→Paid conversion: >25%
- Monthly churn (companies): <3%
- Net Revenue Retention: >110%
- CAC: <€200/company, LTV:CAC >3:1

### Technical Success

| Metric | Target | Rationale |
|--------|--------|-----------|
| **Voice→AI Summary Latency** | <30 seconds | Field workers expect near-instant feedback |
| **Transcription Error Rate** | <2% | Trust in AI requires high accuracy |
| **Offline Sync Reliability** | 0 lost messages | Construction sites have poor connectivity |
| **PWA Load Time** | <3 seconds on 3G | Field conditions, older devices |
| **Time-to-Record** | <2 taps from app open | Dirty hands, gloves, ladders |
| **Platform Uptime** | 99.5% | Business-critical during work hours (6-18h) |
| **Data Residency** | 100% EU | GDPR-by-design, zero third-country transfers |
| **Audit Trail** | Complete | Every data access logged for GDPR compliance |

### Measurable Outcomes

**Pilot Go/No-Go Decision (after 3 months):**

| Gate | Criterion | Threshold |
|------|-----------|-----------|
| **Adoption** | Field workers use it daily without reminders | >70% DAU in pilot companies |
| **Value** | Owners say "I have oversight now" | >80% of owners use dashboard daily |
| **Retention** | No pilot company goes back to WhatsApp | 0 churn during pilot |
| **Willingness to Pay** | "Yes, I'd pay €10/user for this" | >60% of pilot companies |
| **Technical** | Voice→AI pipeline stable | <2% transcription error rate |

Decision point: 3 of 5 gates passed → GO for scale. <3 → Pivot or kill.

## User Journeys

### Journey 1: Markus — "Endlich kein Abend-Papierkram" (Primary User, Happy Path)

**Opening Scene:**
Montag, 7:15. Markus steht auf einer Baustelle in Linz-Urfahr, dritte Etage, Kabelschächte. Gerade hat er die Unterverteilung für die neue Wohnung fertig verdrahtet. Früher hätte er jetzt auf einen Zettel gekritzelt "UV fertig, 3.OG links" — oder es vergessen und abends daheim versucht sich zu erinnern.

**Rising Action:**
Markus zieht den Handschuh aus, öffnet kurzum auf dem Handy — ein Tap. Drückt den großen Aufnahme-Button — zweiter Tap. Spricht: "Unterverteilung dritte Etage links fertig. 24 Sicherungsautomaten eingebaut, alles nach Plan. FI-Schutzschalter muss noch vom Elektriker-Meister abgenommen werden. Material reicht noch für die rechte Wohnung." 28 Sekunden.

Wenige Sekunden später sieht er die KI-Zusammenfassung: drei Zeilen, alles drin. Die KI hat's dem Projekt "Neubau Urfahr, Fam. Gruber" zugeordnet — richtig. Ein Swipe bestätigt. Handschuh wieder an, weiter geht's.

**Climax:**
17:30, Markus ist daheim. Seine Frau fragt: "Musst du noch was schreiben?" — "Nein, ist alles erledigt." Er hat heute 6 Sprachnachrichten aufgenommen, insgesamt keine 4 Minuten. Null Papierkram. Stefan hat trotzdem den kompletten Überblick.

**Resolution:**
Nach zwei Wochen ist es Routine. Markus denkt nicht mehr darüber nach — Aufgabe erledigt, kurz reinsprechen, fertig. Seine Kollegen machen es genauso. "Wie haben wir das früher ohne gemacht?"

---

### Journey 2: Markus — "Kein Netz im Keller" (Primary User, Edge Case)

**Opening Scene:**
Mittwoch, ein Altbau-Keller in einer Funkloch-Gegend. Markus hat gerade Leerrohre verlegt, will es dokumentieren. Kein Empfang.

**Rising Action:**
Markus öffnet kurzum — die App zeigt ein kleines Offline-Symbol, funktioniert aber normal. Er drückt Aufnahme und spricht seine Nachricht ein. Die App speichert lokal und zeigt: "Wird gesendet sobald du wieder online bist."

**Climax:**
20 Minuten später im Auto — Netz ist da. Kurze Vibration: Nachricht synchronisiert, KI-Zusammenfassung verfügbar. Die KI konnte das Projekt nicht eindeutig zuordnen — es ist ein neuer Kunde. Die Nachricht landet in der **"Keine Zuordnung"-Inbox**.

Stefan sieht die Nachricht dort, tippt "Neues Projekt" — die Felder sind von der KI vorbefüllt: "Altbau-Sanierung, Kellner GmbH, Kellergasse 8". Stefan korrigiert den Kundennamen ("Kellner" → "Kelnner"), erstellt das Projekt mit zwei Klicks. Ab jetzt ordnet die KI Nachrichten zu diesem Projekt richtig zu.

**Resolution:**
Keine Nachricht ging verloren. Der Offline-Fall ist für Markus unsichtbar — es funktioniert einfach.

---

### Journey 3: Stefan — "Vom Flaschenhals zum Überblick" (Decision Maker, Happy Path + Edge Cases)

**Opening Scene:**
Stefan sitzt Sonntagabend am Küchentisch und überlegt ob er kurzum.app für seinen Betrieb testen soll. Er hat den Flyer von der Innungsversammlung. €10 pro Kopf, 10 Leute — €100/Monat. Ein Mitarbeiter-Stundensatz. Wenn es nur eine Stunde Chaos pro Woche spart, lohnt es sich.

**Rising Action — Setup:**
Stefan registriert sich. Legt seinen Betrieb an: "Elektro Maier GmbH". Erstellt drei Projekte: "Neubau Urfahr", "Sanierung Schule Graz", "Kleinaufträge". Dauert 5 Minuten. Dann lädt er seine Leute ein — eine SMS mit Link, die Monteure installieren die PWA auf dem Handy. Kein App Store, kein Genehmigungsprozess.

**Rising Action — Neuer Mitarbeiter (Edge Case):**
Drei Wochen später: Lehrling Thomas fängt an. Stefan geht in kurzum → Team → "Mitarbeiter einladen". Thomas bekommt den Link, installiert die PWA, spricht seine erste Nachricht innerhalb von 5 Minuten. Kein IT-Aufwand, kein Schulungstag.

**Climax:**
Dienstagmorgen, 6:45. Stefan öffnet kurzum statt 30 WhatsApp-Nachrichten zu lesen. Dashboard: Neubau Urfahr — 4 Nachrichten gestern, alles planmäßig. Sanierung Schule — Problem: "FI löst ständig aus, brauche Messgerät". Stefan antwortet per Sprachnachricht: "Messgerät ist im Lager, rotes Regal." Keine Telefonrunde, kein Chaos. In der "Keine Zuordnung"-Inbox: eine Nachricht von Thomas über einen neuen Kunden-Anruf — Stefan erstellt das Projekt in 30 Sekunden.

**Resolution:**
Freitag: Stefan ist krank. Sein erster Tag ohne Erreichbarkeit seit Jahren. Er schaut abends kurz aufs Dashboard — alles lief. Die Monteure haben dokumentiert, Andrea hat die Daten, niemand hat ihn angerufen. "Endlich bin ich nicht mehr der einzige der alles weiß."

---

### Journey 4: Andrea — "Montag morgens ohne Chaos" (Secondary User)

**Opening Scene:**
Montag, 8:30. Andrea kommt ins Büro. Früher: Stapel unleserlicher Stundenzettel, 15 WhatsApp-Nachrichten die sie nicht zuordnen kann, und drei Post-its von Stefan. Heute öffnet sie kurzum.

**Rising Action:**
Dashboard: Alle Projekte mit den Aktivitäten von Donnerstag und Freitag (Andrea arbeitet Mo-Do). Für jedes Projekt: wer hat was gemacht, wann, strukturiert zusammengefasst. Sie klickt auf "Sanierung Schule Graz" — Timeline der letzten zwei Tage: Markus hat die Unterverteilung fertig, Thomas hat Leerrohre verlegt, Material wurde nachbestellt.

**Climax:**
Andrea erstellt die Wochenabrechnung. Alle Arbeitszeiten sind da — kein Hinterher-Telefonieren. Sie kopiert die Zusammenfassungen für die Kundenrechnung. Was früher 3 Stunden gedauert hat, ist in 90 Minuten erledigt.

**Resolution:**
"Ich muss keinem mehr hinterherrennen. Die Infos sind einfach da — vollständig und lesbar." Andrea hat Donnerstagnachmittag erstmals seit Monaten keine Überstunden.

---

### Journey Requirements Summary

| Journey | Revealed Capabilities |
|---------|----------------------|
| **Markus Happy Path** | Voice recording (1-tap), AI transcription & summary, auto project assignment, confirmation UI |
| **Markus Offline** | Offline recording & local storage, background sync, "No Assignment" inbox, AI-prefilled project creation |
| **Stefan Setup + Daily** | Company registration, team invite (SMS/link), project CRUD, dashboard with activity feed, push notifications, voice reply, new employee onboarding |
| **Andrea Office** | Project timeline view, structured summaries, activity history, data export for billing |
| **Cross-cutting** | Role-based views (Monteur/Meister/Büro), PWA installable, mobile-first responsive, GDPR-compliant data handling |

## Domain-Specific Requirements

### Compliance & Regulatory (GDPR/DSGVO)

**Voice Data as Personal Data:**
- Voice recordings constitute biometric/personal data under GDPR — requires explicit legal basis for processing
- Processing basis: legitimate interest of the employer (internal communication efficiency) + employee consent during onboarding
- Transparency obligation: users must be informed that AI processes their voice data (clear in-app notice, not buried in T&C)

**Data Retention Policy:**
- Original audio files: retained for **90 days**, then auto-deleted
- Transcriptions + AI summaries: retained for duration of employment/project (permanent within company account)
- On employee departure: audio deleted immediately, transcriptions anonymized or deleted per company policy
- Right to deletion: users can request deletion of their audio data at any time

**Data Processing:**
- All AI processing via Mistral AI on EU servers — no third-country data transfers
- No training of AI models with customer data (strict Auftragsverarbeitung/DPA)
- Audit trail: every data access logged (who accessed what, when)
- Data export: company can export all their data (GDPR portability)

### Technical Constraints

**Offline-First Architecture:**
- Construction sites have unreliable connectivity — offline recording is mandatory, not optional
- Local storage of audio until sync, with guaranteed zero-loss delivery
- Graceful degradation: core recording function must work without any network

**Voice UI Constraints:**
- Field conditions: dust, moisture, gloves, ladders — minimal touch interaction required
- Maximum 2 taps from app open to recording
- Large touch targets, high contrast, works in direct sunlight
- Audio recording must work reliably in noisy environments (construction noise)

**Peak Load Patterns:**
- Usage peaks: 6:00-7:00 (shift start), 12:00-13:00 (lunch), 16:00-18:00 (shift end)
- Unlike typical SaaS: minimal usage outside work hours
- AI processing queue must handle burst patterns at shift boundaries

### Domain Risk Factors

| Risk | Impact | Mitigation |
|------|--------|------------|
| **Employee pushback** ("Boss is spying on me") | Adoption failure | Transparent communication, no location tracking in MVP, voice messages visible to sender |
| **Works council (Betriebsrat) objection** | Legal block in larger companies | Position as communication tool (not surveillance), provide Betriebsrat info package |

> Full risk mitigation strategy including technical, market, and resource risks: see **Project Scoping & Phased Development → Risk Mitigation Strategy**.

## Innovation & Novel Patterns

### Detected Innovation Areas

**1. "Beiläufige Dokumentation" (Incidental Documentation)**
Documentation as a byproduct of communication, not a separate task. Field workers don't "document" — they communicate, and the AI creates the documentation automatically. This is a paradigm shift for an industry where documentation is seen as punishment after a long workday.

**2. Voice-First AI for Underserved KMU Niche**
The combination of voice AI + auto-summarization + project assignment exists for large construction (Benetics AI at €20/user). The innovation is applying it to the 5-50 employee trades segment at half the price — a market segment that's 87% unserved by any dedicated solution.

**3. Invisible AI Intelligence (Zero-Friction Context Assignment)**
Users don't interact with the AI — they just speak. The AI silently assigns to projects based on context (names, addresses, keywords). No tagging, no dropdowns, no "which project?" dialogs. The intelligence is invisible, the simplicity is the feature.

### Market Context & Competitive Landscape

- **Benetics AI** (sole direct competitor): Targets large construction, US-cloud, €20/user — different segment entirely
- **Craftnote/Hero/Plancraft**: Comprehensive trade software where communication is a side feature, no AI, no voice-first
- **Threema Work**: GDPR-compliant but generic messenger, no domain intelligence
- **Gap**: No product combines GDPR compliance + voice-first AI + KMU pricing for trades

### Validation Approach

| Innovation | Validation Method | Success Signal |
|-----------|-------------------|----------------|
| Incidental documentation | Pilot: measure after-work paperwork time before/after | >80% reduction in evening documentation |
| Voice-first for KMU trades | Pilot: daily adoption rate without management pressure | >70% DAU among field workers |
| Invisible AI assignment | Pilot: AI assignment accuracy over time | >85% correct assignment after 2 weeks |

### Innovation Risk & Fallbacks

| Innovation Risk | Fallback |
|----------------|----------|
| AI assignment accuracy too low initially | Manual assignment as default, AI as suggestion — gradually increase automation as confidence improves |
| Voice quality insufficient (dialects, noise) | Audio retained 90 days for human review, user correction feedback loop |
| "Incidental documentation" insufficient for compliance needs | Structured templates as optional add-on (V2), not replacing voice-first flow |

## SaaS B2B Specific Requirements

### Project-Type Overview

kurzum.app is a multi-tenant B2B SaaS product delivered as a PWA. Each trades company (Handwerksbetrieb) is an isolated tenant. Despite the CSV suggestion to skip mobile-first, this product is fundamentally mobile-first — field workers on construction sites are the primary users.

### Multi-Tenancy Model

- **Shared database with Row-Level Security (RLS)** — pragmatic for MVP scale (<100 companies)
- Each company = one tenant, strictly isolated at the data layer
- No cross-tenant data visibility under any circumstances
- Tenant-scoped queries enforced at the application and database level
- Future: evaluate separate schemas if performance requires it at scale

### RBAC Permission Matrix

| Action | Monteur | Meister | Büro |
|--------|---------|---------|------|
| Record voice messages | ✅ | ✅ | ✅ |
| View projects | Assigned only | All | All |
| Create/edit projects | ❌ | ✅ | ✅ |
| Manage team members | ❌ | ✅ | ✅ |
| Dashboard (overview) | Own projects | All projects | All projects |
| Billing & account settings | ❌ | ✅ | ✅ |
| "No Assignment" inbox | ❌ | ✅ | ✅ |
| Confirm/correct AI assignment | Own messages | All messages | All messages |

**Design principle:** Meister and Büro are equal in permissions — in practice, the office admin (often younger, more tech-savvy) manages the system day-to-day. Monteur is the only restricted role, focused on recording and viewing.

### Subscription Tiers & Enforcement

| Plan | Price | Limit | Target |
|------|-------|-------|--------|
| **Starter** | Free | 3 users | Solo/micro, lead generation |
| **Team** | €10/user/month | 4-20 users | Core revenue segment |
| **Business** | €8/user/month | 21-50 users | Volume discount |

**Upgrade Flow (Product-Led Growth):**
- When Starter plan hits 3-user limit and tries to invite 4th user → automatic 14-day Team Trial starts
- Banner in dashboard: "Your Team Trial runs for X more days — upgrade now"
- If trial expires without upgrade → most recently added user(s) set to read-only (can view, cannot record)
- No data loss, no hard blocks — the value is felt before payment is required
- Upgrade path: self-service via in-app billing (Stripe)

### Integration Requirements

**MVP:** No external integrations
- SMS/email for team invitations (via Resend)
- Stripe for subscription billing

**Post-MVP (V2-V3):**
- API-ready architecture from day one for future integrations
- Planned: Craftnote, Hero, DATEV export
- Webhook support for custom integrations

### Compliance Requirements

- Covered in detail in Domain-Specific Requirements section (GDPR/DSGVO)
- Additional SaaS-specific: SOC 2 Type I consideration for enterprise customers (V3+)
- DPA (Auftragsverarbeitungsvertrag) template required for each customer company

### Implementation Considerations

**Authentication:**
- Company registration: email + password (Meister/owner)
- Team member onboarding: invite link via SMS/email → set password → done
- No SSO needed for MVP (KMU target doesn't use SSO)
- Session management: long-lived sessions on mobile (field workers don't want to log in daily)

**PWA-Specific (overriding CSV skip of mobile_first):**
- Service Worker for offline capability and background sync
- Web Push API for notifications
- IndexedDB for local audio storage before sync
- Install prompt for home screen placement
- Target: works on Android Chrome and iOS Safari (dominant field worker devices)

## Project Scoping & Phased Development

### MVP Strategy & Philosophy

**MVP Approach:** Problem-Solving MVP — prove that "Aktion erledigt → Sprachnachricht → Foto → fertig" transforms daily operations for trades companies. No platform play, no feature overload. Validate with 5-10 pilot companies.

**Resource Model:** Solo founder (Mario Semper) using BMAD method for AI-assisted development. Existing infrastructure (Vercel Pro, NeonDB, Upstash, Resend) reduces setup overhead. FFG Kleinprojekt funding pending.

**Core Priorities (in order):**
1. Voice → AI Pipeline (the heart of the product)
2. AI Project Assignment (the differentiator vs. "just another messenger")
3. Team Dashboard (the value proof for decision-makers)

### MVP Feature Set (Phase 1)

**Core User Journeys Supported:**
- Markus: Record voice + attach photo → AI summary → auto-assigned to project → done
- Stefan: Morning dashboard overview, project creation, team management, "No Assignment" inbox
- Andrea: Project timelines with structured summaries, activity history

**Must-Have Capabilities:**

| # | Feature | Why MVP | Key Complexity |
|---|---------|---------|----------------|
| 1 | **Voice Recording & AI Processing** | Core value — without this, no product | Mistral API integration, queuing, error handling |
| 2 | **AI Project Assignment** | Differentiator — without this, it's just GDPR-WhatsApp | NLP context matching, learning loop |
| 3 | **Team Dashboard** | Proves value to buyer (Stefan) | Real-time activity feed, project overview |
| 4 | **Basic Photo Capture** | Field documentation needs visual evidence | Camera API, file upload, cloud storage, timeline display |
| 5 | **Team Management** | Multi-user is table stakes for B2B | Invite flow, RBAC, company setup |
| 6 | **PWA (Offline-First)** | Construction sites have no reliable network | Service Worker, IndexedDB, background sync |

**"No Assignment" Inbox** is part of Feature #2 — messages without project match land here with AI-prefilled project creation.

### Post-MVP Features

**Phase 2 (V2 — after pilot validation, ~Q3-Q4 2026):**
- Automatic time tracking ("Arrived at site" → timestamp via location)
- AI auto-routing (message → right person/department)
- AI photo processing (OCR on type plates, auto-categorization)
- Billing/time export for office (CSV/PDF)
- Extended analytics & reports

**Phase 3 (V3 — DACH expansion, 2027):**
- Native mobile apps (iOS + Android)
- Integrations (Craftnote, Hero, DATEV)
- SHK trade-specific features
- DE market entry with ZVEH/ZVSHK guild partnerships
- Advanced AI insights & company-specific learning

**Phase 4 (Vision, 2028+):**
- "Intelligent communication hub" for trades companies
- Industry benchmarking platform
- Marketplace for trades integrations
- Expansion to additional trades (Maler, Dachdecker, Zimmerer)
- Multi-language (CH expansion)

### Risk Mitigation Strategy

**Technical Risks:**

| Risk | Likelihood | Impact | Mitigation |
|------|-----------|--------|------------|
| AI transcription quality (Austrian dialects, technical jargon) | Medium | High | Mistral Voxtral tested for German; user correction feedback loop; 90-day audio fallback |
| AI project assignment accuracy | Medium | High | Start with AI-as-suggestion (user confirms); accuracy improves with usage data; manual fallback always available |
| Offline sync complexity (PWA) | Medium | Medium | Battle-tested patterns (Service Worker + IndexedDB); limit offline scope to recording + local storage |
| Mistral AI availability/latency | Low | High | Queue-based processing (QStash); graceful degradation (show "processing..." state); audio always preserved locally |

**Market Risks:**

| Risk | Mitigation |
|------|------------|
| "WhatsApp works fine" inertia | Lead with GDPR compliance pressure + Zeiterfassungsgesetz; guild partnerships for credibility |
| Pilot companies don't adopt | Hands-on onboarding; measure adoption weekly; pivot quickly if <70% DAU after 4 weeks |
| Price sensitivity | Free tier as entry; €10/user positioned as "less than one hour of a worker's time" |

**Resource Risks:**

| Risk | Mitigation |
|------|------------|
| Solo founder bandwidth | BMAD method for AI-assisted development; existing tech stack reduces setup; focus on 6 features only |
| FFG funding delayed | Pre-development planning (research, PRD, architecture) doesn't count as project start; can continue planning while waiting |
| Scope creep from pilot feedback | Strict Phase 1 boundary — pilot feedback feeds Phase 2, not Phase 1 patches |

## Functional Requirements

### Voice & Media Capture

- **FR1:** Monteur/Meister/Büro can record a voice message with a maximum of 2 taps from app open
- **FR2:** User can attach a photo (camera capture or gallery) to a voice message
- **FR3:** User can take a standalone photo and post it to a project without a voice message
- **FR4:** User can record voice messages while offline, with automatic upload when connectivity returns
- **FR5:** User can capture photos while offline, with automatic upload when connectivity returns
- **FR6:** User can see a confirmation that their recording was successfully captured (even if not yet synced)
- **FR7:** User can play back their own voice messages

### AI Processing

- **FR8:** System can transcribe voice messages to text using EU-hosted AI
- **FR9:** System can generate a structured summary (max 3-5 lines) from a voice transcription
- **FR10:** System can automatically assign a voice message to the most likely project based on context (names, addresses, keywords)
- **FR11:** User can confirm or correct the AI's project assignment
- **FR12:** System can achieve >85% correct auto-assignment after 2 weeks of user corrections, measured as percentage of AI suggestions accepted without correction
- **FR13:** System can process voice messages within 30 seconds of upload
- **FR14:** System can display a "processing" state while AI analysis is in progress

### Project Management

- **FR15:** Meister/Büro can create a new project (name, address, customer)
- **FR16:** Meister/Büro can edit project details
- **FR17:** Meister/Büro can view a chronological timeline of all messages and photos for a project
- **FR18:** Monteur can view the timeline of projects they are assigned to
- **FR19:** Meister/Büro can access a "No Assignment" inbox for messages without a project match
- **FR20:** Meister/Büro can create a new project from the inbox with AI-prefilled fields (customer name, address, type of work) extracted from the unassigned message
- **FR21:** Meister/Büro can manually assign an unassigned message to an existing project
- **FR22:** Meister/Büro can assign Monteure to specific projects

### Team & Account Management

- **FR23:** Meister can register a new company account with email and password
- **FR24:** Meister/Büro can invite team members via SMS or email link
- **FR25:** Invited user can join the company by following the invite link and setting a password
- **FR26:** Meister/Büro can assign roles to team members (Monteur, Meister, Büro)
- **FR27:** Meister/Büro can remove team members from the company
- **FR28:** System can enforce Starter plan limit (3 users) and trigger automatic 14-day Team Trial when 4th user is invited
- **FR29:** System can set users to read-only when trial expires without upgrade
- **FR30:** Meister/Büro can upgrade subscription plan via self-service billing

### Dashboard & Notifications

- **FR31:** Meister/Büro can view a dashboard with all projects and their latest activity
- **FR32:** Monteur can view a dashboard filtered to their assigned projects
- **FR33:** User can receive push notifications for new messages in their relevant projects
- **FR34:** Meister/Büro can reply to messages via voice recording from the dashboard

### Data & Compliance

- **FR35:** System can automatically delete original audio files after 90 days
- **FR36:** System can log all data access events in an audit trail
- **FR37:** Meister/Büro can export company data (GDPR portability)
- **FR38:** System can display a transparency notice about AI processing to all users
- **FR39:** User can request deletion of their voice data

### Offline & Sync

- **FR40:** PWA can be installed on the home screen of Android and iOS devices
- **FR41:** App can function in offline mode for core recording and photo capture
- **FR42:** System can synchronize all locally stored data when connectivity is restored, with zero data loss
- **FR43:** App can display online/offline status to the user

## Non-Functional Requirements

### Performance

| Requirement | Target | Context |
|-------------|--------|---------|
| **NFR1:** Voice→AI summary latency | <30 seconds from upload | Field workers expect near-instant feedback |
| **NFR2:** PWA initial load | <3 seconds on 3G | Construction sites, older Android devices |
| **NFR3:** Time-to-record | <2 seconds from app open to recording | Gloves, ladders, urgency |
| **NFR4:** Photo upload processing | <5 seconds for thumbnail display | User needs visual confirmation |
| **NFR5:** Dashboard load | <2 seconds for project overview | Stefan's morning routine must be fast |
| **NFR6:** Offline→Online sync | <10 seconds after connectivity restored | Background, non-blocking |

### Security

| Requirement | Target | Context |
|-------------|--------|---------|
| **NFR7:** Data encryption in transit | TLS 1.3 for all connections | Standard, non-negotiable |
| **NFR8:** Data encryption at rest | AES-256 for audio files and database | Voice data is personal data |
| **NFR9:** Data residency | 100% EU — zero third-country transfers | GDPR-by-design core promise |
| **NFR10:** Authentication | Bcrypt/Argon2 password hashing, secure session tokens | Standard security baseline |
| **NFR11:** Tenant isolation | Row-Level Security — no cross-tenant data leakage under any circumstance | B2B trust requirement |
| **NFR12:** Audit logging | Every data access event logged with timestamp, actor, action | GDPR compliance |
| **NFR13:** Audio retention | Auto-deletion after 90 days, verifiable | Data minimization principle |

### Scalability

| Requirement | Target | Context |
|-------------|--------|---------|
| **NFR14:** MVP capacity | 10 companies, ~120 concurrent users | Pilot phase |
| **NFR15:** Year 1 capacity | 100 companies, ~1,200 concurrent users | Growth target |
| **NFR16:** Peak load handling | 3x average load at shift boundaries (6-7h, 12-13h, 16-18h) | Trade-specific usage pattern |
| **NFR17:** AI processing queue | Handle burst of 50+ voice messages within 5 minutes without degradation | Shift-end upload spikes |
| **NFR18:** Storage growth | ~50MB/company/month (audio + photos, before 90-day cleanup) | Predictable cost model |

### Accessibility (Field Conditions)

| Requirement | Target | Context |
|-------------|--------|---------|
| **NFR19:** Touch targets | Minimum 48x48px for all interactive elements | Gloves, dirty hands |
| **NFR20:** Contrast ratio | WCAG AA (4.5:1 minimum) | Direct sunlight on construction sites |
| **NFR21:** Core function without fine motor control | Voice recording via single large button | Ladders, awkward positions |
| **NFR22:** Works without stable connectivity | Core capture functions fully offline | Basements, rural areas |
| **NFR23:** Device compatibility | Android 10+ Chrome, iOS 15+ Safari | Covers 95%+ of field worker devices |

### Reliability

| Requirement | Target | Context |
|-------------|--------|---------|
| **NFR24:** Platform uptime | 99.5% during business hours (6:00-18:00 CET) | Not 24/7 — trades work daytime |
| **NFR25:** Zero data loss | No voice message or photo may be lost, even during outages | Trust-critical — "meine Nachricht ist weg" kills adoption |
| **NFR26:** Graceful degradation | AI unavailable → messages queued, user notified, no data loss | Mistral outage shouldn't block recording |
| **NFR27:** Recovery time | <30 minutes for service restoration after outage | Acceptable for SMB tool |
