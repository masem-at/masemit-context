# BMAD Briefing: kurzum.app Prototype – Sprint 0

**Stand:** 2026-02-12
**Projekt:** kurzum.app – KI-gestützte Voice-First Kommunikation für Handwerksbetriebe
**Scope:** Auth-geschützter Prototyp (Voice Recording → Transkription → Zusammenfassung)
**Timeline:** KW 8–12 (17.02. – 16.03.2026)
**FFG-Sitzung:** 19.03.2026 – Prototyp soll bis dahin grundlegend funktionieren
**Referenz:** Product Brief (2026-02-08), PRD (2026-02-08), FFG-Antrag 69168884

---

## 1. Kontext & Zielsetzung

### Was ist kurzum.app?
Monteure im Handwerk sprechen 15–30 Sekunden ins Handy statt zu tippen. Die KI transkribiert, fasst zusammen, ordnet dem richtigen Projekt zu und dokumentiert. DSGVO-konform, ab €10/User/Monat.

**Kernprinzip:** "Aktion erledigt → Sprachaufnahme → fertig." Kommunikation IST die Dokumentation.

### Was soll der Prototyp können?
Ein authentifizierter Benutzer kann:
1. Sich per Magic Link einloggen
2. Eine Sprachnachricht aufnehmen (Browser/PWA)
3. Die KI transkribiert die Nachricht (Mistral Voxtral STT)
4. Die KI fasst die Nachricht strukturiert zusammen (Mistral LLM)
5. Die Zusammenfassung wird im Dashboard angezeigt

### Was soll der Prototyp NICHT können (erst später)?
- Automatische Projekt-Zuordnung mit KI (kommt in Sprint 1)
- Team-Management / Multi-User / Rollen (kommt in Sprint 2)
- Offline-Fähigkeit / PWA install (kommt in Sprint 3)
- Foto-Capture (kommt in Sprint 1-2)
- Abrechnung / Stripe (kommt viel später)

---

## 2. Tech Stack

### Bestehend bei masemIT (bewährt)

| Bereich | Technologie | Begründung |
|---------|-------------|------------|
| **Frontend** | Next.js 14+ (App Router) | Standard bei masemIT, SSR/SSG |
| **Styling** | Tailwind CSS | Standard bei masemIT |
| **Database** | NeonDB PostgreSQL + Drizzle ORM | Standard bei masemIT, Frankfurt Region |
| **Deployment** | Vercel Pro | Standard bei masemIT |
| **Cache** | Upstash Redis | Standard bei masemIT, Frankfurt Region |
| **Queue** | QStash | Standard bei masemIT, asynchrone Verarbeitung |
| **E-Mail** | Resend | Standard bei masemIT (Magic Link) |
| **Bot Protection** | Cloudflare Turnstile | Bestehendes shared Widget |
| **Domain** | kurzum.app (Cloudflare DNS, Porkbun Registrar) | Bereits konfiguriert ✅ |

### KI-Stack (festgelegt im Product Brief)

| Bereich | Technologie | Status | Begründung |
|---------|-------------|--------|------------|
| **STT (Speech-to-Text)** | **Mistral Voxtral** | Primär | EU-gehostet, DSGVO-konform |
| **STT Fallback** | Whisper (self-hosted) | Fallback | Open-source, kann EU-hosted werden |
| **LLM (Zusammenfassung)** | **Mistral Large/Medium** | Primär | EU-gehostet, DSGVO-konform |
| **Audio Storage** | Vercel Blob / Cloudflare R2 | Temp | EU-only, Auto-Delete nach 90 Tagen |

**WICHTIG:** Mistral AI ist keine Option unter vielen – es ist die im Product Brief und FFG-Antrag festgelegte Entscheidung. DSGVO-by-Design erfordert EU-hosted AI. Andere Provider (OpenAI, Anthropic) dürfen nur als Evaluierungs-Benchmark verwendet werden, NICHT als Produktions-Provider.

---

## 3. Arbeitspakete

### AP 0.1: Projekt-Setup (2–3 Tage)
**Owner:** Dev Agent

**Aufgaben:**
- [ ] Next.js Projekt initialisieren (`kurzum-app` Repository)
- [ ] NeonDB Datenbank anlegen (Frankfurt Region, separates Projekt)
- [ ] Drizzle ORM konfigurieren
- [ ] Vercel Projekt verknüpfen (kurzum.app Domain ist bereits konfiguriert)
- [ ] Environment Variables setzen (DB, Resend, Turnstile, Mistral API Key)
- [ ] Basis-Layout: Header, Footer, Landing Page Route (`/`), App Route (`/app`)
- [ ] Tailwind + gemeinsame Komponenten (Button, Card, Input)
- [ ] QStash-Integration vorbereiten (für asynchrone KI-Verarbeitung)

**Ergebnis:** Leeres aber deploybares Next.js Projekt unter kurzum.app

**Schema (Drizzle):**
```typescript
// users
export const users = pgTable('users', {
  id: uuid('id').primaryKey().defaultRandom(),
  email: varchar('email', { length: 255 }).notNull().unique(),
  name: varchar('name', { length: 255 }),
  role: varchar('role', { length: 50 }).default('user'), // user, admin
  createdAt: timestamp('created_at').defaultNow(),
  lastLoginAt: timestamp('last_login_at'),
});

// magic_links
export const magicLinks = pgTable('magic_links', {
  id: uuid('id').primaryKey().defaultRandom(),
  userId: uuid('user_id').references(() => users.id),
  token: varchar('token', { length: 255 }).notNull().unique(),
  expiresAt: timestamp('expires_at').notNull(),
  usedAt: timestamp('used_at'),
  createdAt: timestamp('created_at').defaultNow(),
});

// voice_messages
export const voiceMessages = pgTable('voice_messages', {
  id: uuid('id').primaryKey().defaultRandom(),
  userId: uuid('user_id').references(() => users.id).notNull(),
  audioUrl: varchar('audio_url', { length: 500 }), // temp storage URL
  audioDurationSeconds: integer('audio_duration_seconds'),
  transcript: text('transcript'),
  summary: jsonb('summary'), // { status, material, nextSteps, problems, project }
  sttProvider: varchar('stt_provider', { length: 50 }), // voxtral, whisper
  sttDurationMs: integer('stt_duration_ms'), // processing time (for research docs)
  llmProvider: varchar('llm_provider', { length: 50 }), // mistral-large, mistral-medium
  llmDurationMs: integer('llm_duration_ms'), // processing time (for research docs)
  status: varchar('status', { length: 20 }).default('pending'), // pending, processing, done, error
  createdAt: timestamp('created_at').defaultNow(),
  processedAt: timestamp('processed_at'),
  audioDeletedAt: timestamp('audio_deleted_at'), // DSGVO: wann gelöscht
});
```

---

### AP 0.2: Magic Link Authentication (2–3 Tage)
**Owner:** Dev Agent

**Aufgaben:**
- [ ] Login-Seite (`/login`) mit E-Mail-Eingabe + Turnstile
- [ ] Magic Link generieren (Token, 15 Min. Gültigkeit)
- [ ] E-Mail via Resend senden (Template: "Dein kurzum Login-Link")
- [ ] Verification-Route (`/api/auth/verify?token=...`)
- [ ] Session-Management (httpOnly Cookie, JWT oder Session-Token)
- [ ] Middleware: `/app/*` Routes nur für authentifizierte User
- [ ] Logout-Funktion

**Sicherheit:**
- Token: UUID v4, einmalig verwendbar
- Rate Limiting: Max. 5 Magic Links pro E-Mail pro Stunde (Upstash Redis)
- Session Expiry: 7 Tage (konfigurierbar)

**UI:**
```
[Login-Seite]
┌─────────────────────────────┐
│  🎙️ kurzum                  │
│                             │
│  Sprich statt tipp.         │
│                             │
│  E-Mail: [____________]     │
│  [🔒 Turnstile]             │
│  [   Login-Link senden   ]  │
│                             │
│  Prüfe dein Postfach!       │
└─────────────────────────────┘
```

---

### AP 0.3: Voice Recording (3–4 Tage)
**Owner:** Dev Agent

**Aufgaben:**
- [ ] Aufnahme-Komponente mit MediaRecorder API
- [ ] Start/Stop Button: großer, auffälliger Mikrofon-Button (≥48x48px, NFR19)
- [ ] Audio-Visualisierung während der Aufnahme (Waveform oder Pegel)
- [ ] Audio-Vorschau vor dem Absenden (Play/Pause)
- [ ] Audio-Upload an API-Endpoint (`POST /api/voice`)
- [ ] Fortschrittsanzeige beim Upload
- [ ] Max. Aufnahmedauer: 5 Minuten (konfigurierbar)
- [ ] Audio-Format: WebM/Opus (Browser-Standard) oder WAV als Fallback
- [ ] Fehlerbehandlung: Kein Mikrofon, Permission denied, Upload-Fehler
- [ ] "Processing"-State anzeigen während KI arbeitet (FR14)

**DSGVO-Anforderungen (aus PRD):**
- Audio-File wird nach 90 Tagen automatisch gelöscht (FR35)
- `audioDeletedAt` Timestamp in DB setzen
- Alle Datenzugriffe in Audit Trail loggen (FR36)
- Transparency Notice über KI-Verarbeitung anzeigen (FR38)

**UI (Mobile-First!):**
```
[Aufnahme-Seite /app]
┌─────────────────────────────┐
│  🎙️ kurzum    [👤 Profil]   │
│                             │
│  ┌───────────────────────┐  │
│  │                       │  │
│  │   🔴 [  Aufnahme  ]   │  │
│  │   ▓▓▓▓░░░░░░ 0:12    │  │
│  │                       │  │
│  │   [▶ Anhören] [Senden]│  │
│  └───────────────────────┘  │
│                             │
│  --- Letzte Nachrichten --- │
│  📝 Müller Hauptstr. 5      │
│     OG fertig, Material...  │
│     vor 2 Stunden           │
│                             │
│  📝 Huber Gartenweg 12      │
│     Zählerkasten EG...      │
│     vor 5 Stunden           │
└─────────────────────────────┘
```

---

### AP 0.4: Mistral Voxtral STT-Integration (4–5 Tage)
**Owner:** Dev Agent + Research Documentation

**Forschungsfrage #1 (aus FFG-Antrag):**
> Wie lässt sich domänenspezifische Spracherkennung bei Baustellenlärm und österreichischen Dialekten optimieren?

**Aufgaben:**
- [ ] Mistral AI Account einrichten, API-Key generieren
- [ ] Mistral Voxtral STT-API integrieren
- [ ] Test-Audio vorbereiten:
  - Klare Sprache, Hochdeutsch
  - Österreichischer Dialekt (Waldviertlerisch, Wienerisch)
  - Baustellenlärm (Hintergrundgeräusche simulieren)
  - Elektro-Fachbegriffe (FI-Schalter, Sicherungsautomat B16, Zählerkasten, etc.)
- [ ] Evaluierungsmetriken dokumentieren:
  - Word Error Rate (WER) pro Szenario
  - Verarbeitungszeit (Latenz)
  - Kosten pro Minute Audio
  - EU-Datenresidenz bestätigen
- [ ] Whisper large-v3 als Benchmark testen (gleiche Test-Audio)
- [ ] Ergebnis: Vergleichsmatrix Voxtral vs. Whisper (dokumentiert für FFG)

**Asynchrone Verarbeitung (QStash):**
```
User Upload → API speichert Audio → QStash Job → Mistral Voxtral STT
                                                      ↓
                                                 Transcript in DB
                                                      ↓
                                              QStash Job → Mistral LLM
                                                      ↓
                                               Summary in DB → Push
```

**API-Endpoint:**
```typescript
// POST /api/voice (Upload + Queue)
// Input: Audio-File (multipart/form-data)
// Output: { id: string, status: 'processing' }

// GET /api/voice/:id (Poll for result)
// Output: { id, status, transcript?, summary? }
```

**Dokumentation (FFG-relevant!):**
- Jeder Test muss im Forschungstagebuch dokumentiert werden
- Screenshots/Logs der Ergebnisse aufheben
- Commit-Messages: "Evaluate Mistral Voxtral for Austrian dialect: WER 12.3%"

---

### AP 0.5: Mistral LLM-Zusammenfassung (3–4 Tage)
**Owner:** Dev Agent

**Forschungsfrage #2 (aus FFG-Antrag):**
> Wie können unstrukturierte Sprachnachrichten semantisch in strukturierte Zusammenfassungen transformiert werden?

**Aufgaben:**
- [ ] Mistral Large/Medium API integrieren
- [ ] Prompt Engineering für Handwerker-Kontext:
  ```
  Du bist ein Assistent für Handwerksbetriebe. Fasse folgende 
  Sprachnachricht eines Monteurs strukturiert zusammen.
  
  Transkript: "{transcript}"
  
  Erstelle folgende Struktur als JSON:
  - status: Was wurde erledigt?
  - material: Welches Material wird benötigt? (Array)
  - nextSteps: Was steht als nächstes an?
  - problems: Gibt es Probleme oder Hindernisse? (null wenn keine)
  - project: Welche Baustelle/welcher Kunde wird erwähnt?
  
  Antworte NUR mit validem JSON, kein Fließtext.
  ```
- [ ] Prompt-Varianten testen und Ergebnisse dokumentieren
- [ ] Evaluierung: Semantische Korrektheit ≥85% (Ziel aus FFG-Antrag)
- [ ] Ziel: Zusammenfassung von 2 Min Sprache auf ≤5 strukturierte Sätze
- [ ] Integration: Transcript → QStash → Summarize → Store → Notify

**Output-Format (JSON):**
```json
{
  "status": "FI-Schalter im OG getauscht, funktioniert",
  "material": ["3x Sicherungsautomat B16", "1x FI 40/0,03"],
  "nextSteps": "Morgen Zählerkasten im EG",
  "problems": null,
  "project": "Müller, Hauptstraße 5"
}
```

---

### AP 0.6: Dashboard (2–3 Tage)
**Owner:** Dev Agent

**Aufgaben:**
- [ ] Dashboard-Seite (`/app`) mit Liste der Sprachnachrichten
- [ ] Karten-Ansicht pro Nachricht:
  - Zeitstempel
  - Erkanntes Projekt (oder "Nicht zugeordnet")
  - Status / Material / Nächste Schritte (farbcodiert)
  - Probleme (rot hervorgehoben, wenn vorhanden)
  - Link zum Original-Transkript (aufklappbar)
  - Processing-State ("KI verarbeitet...") für aktive Jobs
- [ ] Filterung: Alle / Heute / Diese Woche
- [ ] Sortierung: Neueste zuerst
- [ ] Mobile-First Design (Monteure nutzen Handy!)
- [ ] Touch-Targets ≥48x48px (NFR19)
- [ ] WCAG AA Kontrast (NFR20) – Baustelle = Sonnenlicht
- [ ] Loading States und Error Handling

---

## 4. Nicht-funktionale Anforderungen (aus PRD)

### DSGVO (Pflicht!)
- [ ] Alle Datenverarbeitung in EU (Frankfurt) – kein US-Hosting
- [ ] Audio-Files nach 90 Tagen automatisch löschen (NFR13)
- [ ] Consent beim ersten Login ("Ich stimme der Verarbeitung meiner Sprachnachrichten zu")
- [ ] Audit Trail für alle Datenzugriffe (NFR12)
- [ ] Datenschutzerklärung bereits vorhanden unter kurzum.app/datenschutz ✅

### Security
- [ ] HTTPS only (Vercel Standard)
- [ ] httpOnly Cookies für Sessions
- [ ] Rate Limiting auf alle API-Endpoints (Upstash Redis)
- [ ] Input Validation (Audio-Dateigröße, Format)
- [ ] Keine API-Keys im Frontend
- [ ] Tenant Isolation vorbereiten (Row-Level Security, NFR11)

### Performance (aus PRD)
- [ ] Voice→AI Summary: <30 Sekunden (NFR1)
- [ ] PWA Load: <3 Sekunden auf 3G (NFR2)
- [ ] Time-to-Record: <2 Sekunden von App-Open (NFR3)
- [ ] Dashboard Load: <2 Sekunden (NFR5)

---

## 5. Branching & Deployment

- **Repository:** `masem-at/kurzum-app` (GitHub)
- **Branch Strategy:** `main` → Vercel Production, `develop` → Vercel Preview
- **URL:** kurzum.app (Landing Page öffentlich, `/app/*` hinter Auth)
- **Environment Variables (Vercel):**
  - `DATABASE_URL` (NeonDB Frankfurt)
  - `RESEND_API_KEY`
  - `TURNSTILE_SITE_KEY` + `TURNSTILE_SECRET_KEY`
  - `MISTRAL_API_KEY`
  - `QSTASH_TOKEN` + `QSTASH_CURRENT_SIGNING_KEY` + `QSTASH_NEXT_SIGNING_KEY`
  - `UPSTASH_REDIS_REST_URL` + `UPSTASH_REDIS_REST_TOKEN`
  - `SESSION_SECRET`

---

## 6. Risiken & Mitigationen

| Risiko | Wahrscheinlichkeit | Mitigation |
|--------|-------------------|------------|
| Mistral Voxtral STT-Qualität bei Dialekt unzureichend | Mittel | Whisper self-hosted als Fallback, dokumentierte Evaluierung |
| Audio-Upload zu groß/langsam | Niedrig | Client-seitige Kompression (Opus), Chunked Upload |
| Mistral LLM halluziniert Inhalte | Mittel | Transcript immer als Referenz behalten, Prompt-Tuning |
| Mistral AI Latenz/Verfügbarkeit | Niedrig-Mittel | QStash-Queue, graceful degradation (FR26), "Processing" State |
| FFG-Ablehnung am 19.03. | Mittel | Kosten bis dahin überschaubar, Projekt trotzdem fortsetzen |

---

## 7. Erfolgskriterien (bis 19.03.)

- [ ] Authentifizierter Login funktioniert
- [ ] Sprachnachricht kann aufgenommen und abgespielt werden
- [ ] Mistral Voxtral STT-Integration funktioniert
- [ ] STT-Evaluierung dokumentiert (Voxtral vs. Whisper Benchmark)
- [ ] Mind. 1 Sprachnachricht end-to-end verarbeitet (Sprache → Zusammenfassung)
- [ ] Dashboard zeigt Zusammenfassungen an
- [ ] Alles in EU gehostet, Audio-Retention-Policy implementiert
- [ ] Stundenaufzeichnungen ab 10.02.2026 geführt
- [ ] Forschungstagebuch mit Test-Ergebnissen gepflegt

---

## 8. Referenzen

- **Product Brief:** product-brief-kurzum-app-2026-02-08.md
- **PRD:** prd.md (2026-02-08)
- **FFG-Antrag:** Eingereicht 10.02.2026, Antragsnr. 69168884
- **Landing Page:** https://kurzum.app
- **DO/DON'T Liste:** kurzum-bmad-do-dont-liste.md
- **Pilot-Mails:** pilot-mails-komplett.md
- **Existing Stack Referenz:** ChainSights (gleicher Tech-Stack, gleiche Patterns)
- **Personas:** Markus (Monteur), Stefan (Meister), Andrea (Büro) – siehe Product Brief
