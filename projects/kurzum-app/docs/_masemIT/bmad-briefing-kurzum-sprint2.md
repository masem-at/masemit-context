# BMAD Briefing: kurzum.app – Sprint 2

**Stand:** 2026-02-15
**Projekt:** kurzum.app – KI-gestützte Voice-First Kommunikation für Handwerksbetriebe
**Scope:** Foto-Capture, Team-Einladung & RBAC, Assignment-Verbesserungen
**Vorbedingung:** Sprint 0 ✅ + Sprint 1 ✅ (Auth, Voice, STT, LLM, Projekte, KI-Zuordnung, Inbox, Dashboard)
**FFG-Sitzung:** 19.03.2026
**Referenz:** PRD (2026-02-08), ADR-004/005/006, Sprint 1 Briefing

---

## 1. Sprint-Ziel

Sprint 1 hat bewiesen: Sprechen → Zusammenfassung → Projekt-Zuordnung funktioniert (75% Accuracy, ADR-006).

Sprint 2 macht kurzum.app **multi-user-fähig und visuell**: Fotos können angehängt werden, Teams können eingeladen werden, und die Zuordnungs-KI wird durch Erkenntnisse aus ADR-006 verbessert.

**Nach Sprint 2 kann ein Pilotbetrieb kurzum.app tatsächlich nutzen:**
- Meister lädt seine Monteure ein
- Monteure sprechen Nachrichten ein UND machen Fotos
- KI ordnet zu, Meister sieht alles im Dashboard
- Rollen (Monteur/Meister/Büro) werden durchgesetzt

---

## 2. Input aus ADR-006 – Verbesserungspotenzial

ADR-006 identifiziert konkrete Schwächen die in Sprint 2 adressiert werden:

| ADR-006 Finding | Sprint 2 Maßnahme | AP |
|----------------|-------------------|-----|
| TC-03: Fachbegriff "Zählerkasten" als Projektname interpretiert | Prompt-Iteration: Explizite Unterscheidung Fachbegriff vs. Projektname | AP 2.3 |
| TC-05: Ohne Vorgänger-Nachrichten kein Kontinuitäts-Signal | Kontinuitäts-Signal implementieren (letzte 3 Nachrichten des Sprechers) | AP 2.3 |
| Confidence-Schwelle 0.7 grenzwertig (TC-03 war 0.70 = False Positive) | Schwelle auf 0.75 erhöhen | AP 2.3 |
| Single-Edge-Case gut, Multi-Projekt-Szenario ungetestet | Erweiterte Testszenarien mit 6+ Projekten | AP 2.3 |

---

## 3. Arbeitspakete

### AP 2.1: Foto-Capture (3–4 Tage)
**Owner:** Dev Agent
**PRD-Referenz:** FR2, FR3, FR4, FR5

**Zwei Modi:**

**Modus 1 – Foto an Sprachnachricht (FR2):**
- Während/nach der Aufnahme: "📷 Foto anhängen" Button
- Foto wird mit der voice_message verknüpft (voiceMessageId FK)
- In der Timeline: Zusammenfassung + Foto(s) zusammen angezeigt

**Modus 2 – Standalone-Foto (FR3):**
- Direkt ein Foto in ein Projekt posten (ohne Sprachnachricht)
- Optional: Kurze Textnotiz dazu
- Anwendungsfall: "Vorher/Nachher"-Dokumentation, Typenschilder, Schäden

**Technische Umsetzung:**
```
Capture: <input type="file" accept="image/*" capture="environment">
Upload: POST /api/photos → Vercel Blob (EU-Region)
Thumbnail: Vercel Image Optimization (/_next/image?url=...&w=400&q=75)
EXIF: Serverseitig entfernen (sharp oder exiftool) – keine GPS-Koordinaten speichern
Max: 10MB Client-seitig, Kompression via canvas.toBlob() wenn > 5MB
```

**Schema (Erweiterung):**
```typescript
export const photos = pgTable('photos', {
  id: uuid('id').primaryKey().defaultRandom(),
  userId: uuid('user_id').references(() => users.id).notNull(),
  companyId: uuid('company_id').references(() => companies.id).notNull(),
  voiceMessageId: uuid('voice_message_id').references(() => voiceMessages.id),
  projectId: uuid('project_id').references(() => projects.id),
  imageUrl: varchar('image_url', { length: 500 }).notNull(),
  caption: text('caption'),
  createdAt: timestamp('created_at').defaultNow(),
});
```

**Aufgaben:**
- [ ] Kamera-Capture Komponente (HTML5 input + Vorschau)
- [ ] Client-seitige Kompression (>5MB → canvas resize)
- [ ] Upload API → Vercel Blob
- [ ] EXIF-Stripping serverseitig
- [ ] Foto an Voice Message anhängen (UI + API)
- [ ] Standalone-Foto in Projekt posten (UI + API)
- [ ] Fotos in Projekt-Timeline anzeigen (Thumbnail + Lightbox)
- [ ] Foto-Galerie pro Projekt (optional, wenn Zeit)

**DSGVO:**
- EXIF/GPS entfernen (Privacy)
- Fotos an Company gebunden
- Bei Projekt-Löschung: Fotos mitlöschen
- Kein AI-Processing auf Fotos in Sprint 2 (OCR kommt in V2, siehe PRD Phase 2)

---

### AP 2.2: Team-Einladung & RBAC (3–4 Tage)
**Owner:** Dev Agent
**PRD-Referenz:** FR23, FR24, FR25, FR26, FR27

**Einladungs-Flow:**
```
Meister/Büro → /app/team → "Mitarbeiter einladen"
  → E-Mail + Rolle (Monteur/Meister/Büro) eingeben
  → System generiert Einladungs-Token (7 Tage gültig)
  → Resend schickt E-Mail mit Link: kurzum.app/invite/{token}
  → Neuer User klickt → Magic Link Login/Registrierung
  → Automatisch: Company zugewiesen + Rolle gesetzt
  → Erscheint in Team-Liste
```

**Schema (Erweiterung):**
```typescript
export const invitations = pgTable('invitations', {
  id: uuid('id').primaryKey().defaultRandom(),
  companyId: uuid('company_id').references(() => companies.id).notNull(),
  email: varchar('email', { length: 255 }).notNull(),
  role: varchar('role', { length: 20 }).notNull(), // monteur, meister, buero
  token: varchar('token', { length: 255 }).notNull().unique(),
  invitedBy: uuid('invited_by').references(() => users.id).notNull(),
  expiresAt: timestamp('expires_at').notNull(),
  usedAt: timestamp('used_at'),
  createdAt: timestamp('created_at').defaultNow(),
});
```

**RBAC Middleware (jetzt aktiv):**
```typescript
// lib/auth/rbac.ts
type Role = 'monteur' | 'meister' | 'buero';

const PERMISSIONS = {
  'project:create': ['meister', 'buero'],
  'project:edit': ['meister', 'buero'],
  'project:view_all': ['meister', 'buero'],
  'project:view_assigned': ['monteur'],
  'inbox:view': ['meister', 'buero'],
  'inbox:assign': ['meister', 'buero'],
  'team:invite': ['meister', 'buero'],
  'team:manage': ['meister', 'buero'],
  'voice:record': ['monteur', 'meister', 'buero'],
  'voice:confirm_assignment': ['monteur', 'meister', 'buero'],
  'photo:upload': ['monteur', 'meister', 'buero'],
} as const;

function hasPermission(role: Role, permission: keyof typeof PERMISSIONS): boolean {
  return PERMISSIONS[permission].includes(role);
}
```

**Aufgaben:**
- [ ] Einladungs-System: Token generieren, E-Mail senden, Token einlösen
- [ ] Einladungs-E-Mail Template (Resend)
- [ ] /app/invite/[token] Route: Registrierung + Auto-Join Company
- [ ] Team-Liste UI (/app/team): Name, E-Mail, Rolle, eingeladen am
- [ ] Rolle ändern (Meister/Büro kann Rollen anderer ändern)
- [ ] Mitarbeiter entfernen (Soft-Delete: deaktivieren, nicht löschen)
- [ ] RBAC Middleware implementieren (hasPermission checks)
- [ ] Route-Protection: /app/inbox, /app/team → nur Meister/Büro
- [ ] Daten-Filterung: Monteur sieht nur zugewiesene Projekte
- [ ] Monteur zu Projekten zuweisen (Meister/Büro kann das)

**NICHT in Sprint 2:**
- SMS-Einladung (nur E-Mail)
- Starter Plan Limit (3 User) – kommt in Sprint 3
- Subscription/Billing – kommt viel später

---

### AP 2.3: Assignment-Verbesserungen (2–3 Tage)
**Owner:** Dev Agent + Research Documentation
**PRD-Referenz:** FR12
**FFG-Forschungsfrage:** #3 (Iteration 2)
**Input:** ADR-006 Findings

**Maßnahmen:**

**1. Confidence-Schwelle → 0.75:**
```typescript
// app/api/voice/process/route.ts
const CONFIDENCE_THRESHOLD = 0.75; // war 0.70 in Sprint 1
```

**2. Kontinuitäts-Signal aktivieren:**
Die letzte(n) Nachrichten des Sprechers mit Projekt-Zuordnung als Kontext mitgeben. Sprint 1 Briefing hatte das vorgesehen, aber ADR-006 wurde ohne Vorgänger-Nachrichten getestet (Baseline).

```typescript
// lib/ai/assignment.ts
const recentMessages = await getRecentMessagesForUser(userId, { limit: 3 });
// → in Prompt einfügen: "Letzte Nachrichten des Monteurs: ..."
```

**3. Prompt-Iteration (ASSIGNMENT_V2):**
Explizite Anweisung hinzufügen:
```
WICHTIG: Fachbegriffe wie "Zählerkasten", "FI-Schalter", "Unterverteilung" sind 
KEINE Projektnamen. Sie beschreiben Arbeiten, nicht Baustellen. Ordne nur zu wenn 
Kundenname, Adresse oder expliziter Projektbezug im Transkript vorkommt.
```

**4. Evaluierung V2 (ADR-007):**
- [ ] Gleiche 8 Transkripte + 2 neue Edge Cases
- [ ] Mit Kontinuitäts-Signal (Vorgänger-Nachrichten simuliert)
- [ ] Vergleich V1 vs V2: Ergebnismatrix
- [ ] Ziel: TC-03 korrekt als Inbox, TC-05 korrekt als Gruber
- [ ] ADR-007 schreiben

**5. Prompt-Versionierung:**
- `ASSIGNMENT_SYSTEM_PROMPT_V2` in `lib/ai/prompts.ts`
- `llmPromptVersion: 'ASSIGNMENT_V2'` in voice_messages DB
- Vergleichbar mit V1 Ergebnissen

---

### AP 2.4: Dashboard & Timeline Erweiterung (1–2 Tage)
**Owner:** Dev Agent

**Aufgaben:**
- [ ] Fotos in Projekt-Timeline eingebettet (Thumbnail, klickbar → Lightbox)
- [ ] Timeline-Eintrag: Unterscheidung Voice + Foto vs. Standalone-Foto
- [ ] Team-Mitglieder im Dashboard anzeigen (wer hat wann zuletzt etwas gesendet)
- [ ] Monteur-Ansicht: Nur zugewiesene Projekte (RBAC-gefiltert)
- [ ] Inbox Badge-Counter aktualisieren (nur Meister/Büro sehen)

---

## 4. Verarbeitungs-Pipeline (aktualisiert für Sprint 2)

```
User nimmt Sprachnachricht auf
  + optional: Foto anhängen (1-3 Fotos)
        ↓
POST /api/voice (Audio + Foto-URLs)
        ↓
Audio → Vercel Blob (temp, 90 Tage)
Foto(s) → Vercel Blob (permanent), EXIF strip, Thumbnail
DB: voice_message + photos erstellt (status: 'processing')
        ↓
Mistral Voxtral STT → transcript
        ↓
Mistral Small LLM #1 → summary (ASSIGNMENT_V1, unverändert aus ADR-005)
        ↓
Mistral Small LLM #2 → assignment (ASSIGNMENT_V2)
  + Kontext: aktive Projekte + letzte 3 Nachrichten des Sprechers
        ↓
confidence ≥ 0.75 → projectId setzen, assignedBy: 'ai'
confidence < 0.75 → projectId: null (→ Inbox)
        ↓
DB: voice_message aktualisiert (status: 'done', promptVersion: 'V2')
        ↓
Dashboard/Timeline zeigt Ergebnis + Fotos
```

**Standalone-Foto-Flow (parallel):**
```
User macht Foto direkt in Projekt
        ↓
POST /api/photos (Image + projectId + optional caption)
        ↓
Foto → Vercel Blob, EXIF strip, Thumbnail
DB: photo erstellt
        ↓
Erscheint in Projekt-Timeline
```

---

## 5. Erfolgskriterien Sprint 2

### Funktional
- [ ] Fotos können an Sprachnachrichten angehängt werden
- [ ] Standalone-Fotos können in Projekte gepostet werden
- [ ] Fotos erscheinen in der Projekt-Timeline (Thumbnail + Lightbox)
- [ ] EXIF-Daten werden serverseitig entfernt
- [ ] Team-Einladung per E-Mail funktioniert
- [ ] Eingeladener User kann sich registrieren und wird Company zugeordnet
- [ ] RBAC: Monteur sieht nur zugewiesene Projekte
- [ ] RBAC: Inbox + Team nur für Meister/Büro zugänglich
- [ ] Rollen können geändert werden
- [ ] Mitarbeiter können entfernt werden

### Forschung (FFG)
- [ ] ASSIGNMENT_V2 Prompt mit Fachbegriff-Unterscheidung
- [ ] Kontinuitäts-Signal (letzte 3 Nachrichten) implementiert
- [ ] Confidence-Schwelle auf 0.75 erhöht
- [ ] ADR-007 dokumentiert: V1 vs V2 Vergleich
- [ ] Ziel: ≥ 80% Accuracy (Verbesserung von 75% Baseline)

### Technisch
- [ ] Tenant-Isolation für alle neuen Endpoints (companyId Filter)
- [ ] RBAC Middleware auf allen geschützten Routes
- [ ] Prompt-Version in DB geloggt
- [ ] Foto-Upload < 3s (inkl. EXIF-Strip + Thumbnail)

---

## 6. Scope-Abgrenzung

### IN Sprint 2
- Foto-Capture (an Voice Message + Standalone)
- Team-Einladung per E-Mail
- RBAC aktiv (Monteur/Meister/Büro)
- Assignment V2 (Prompt-Iteration + Kontinuität + Schwelle)

### NICHT in Sprint 2
- SMS-Einladung
- Starter Plan Limit (3 User Enforcement)
- Subscription/Billing (Stripe)
- Offline/PWA (Sprint 3)
- AI-Photo-Processing/OCR (PRD Phase 2)
- Push Notifications (Sprint 3)
- Foto-Bearbeitung/Annotation

---

## 7. FFG-Relevanz

Sprint 2 Arbeiten werden dokumentiert unter:

| Tätigkeit | FFG-AP |
|-----------|--------|
| Foto-Capture implementieren | AP2 (MVP Development) |
| Team-Einladung + RBAC | AP2 (MVP Development) |
| Assignment V2 Prompt + Evaluation | AP4 (KI-Forschung Projekt-Zuordnung) |
| ADR-007 schreiben | AP4 (KI-Forschung Projekt-Zuordnung) |
| Dashboard-Erweiterung | AP2 (MVP Development) |
| Sprint-Koordination, Briefing | AP7 (Projektmanagement) |

---

## 8. Referenzen

- **Sprint 1 Briefing:** bmad-briefing-kurzum-sprint1.md
- **ADR-004:** STT Provider Evaluation (Voxtral vs Whisper)
- **ADR-005:** LLM Summarization Prompt Engineering
- **ADR-006:** Projekt-Zuordnungs-Evaluierung (Baseline 75%)
- **PRD:** prd.md (FR2-5, FR12, FR23-27)
- **Product Brief:** product-brief-kurzum-app-2026-02-08.md
- **FFG-Antrag:** 69168884, AP2 + AP4
