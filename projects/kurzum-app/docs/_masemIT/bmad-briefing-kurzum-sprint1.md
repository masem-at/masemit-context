# BMAD Briefing: kurzum.app – Sprint 1

**Stand:** 2026-02-13
**Projekt:** kurzum.app – KI-gestützte Voice-First Kommunikation für Handwerksbetriebe
**Scope:** Projekt-Zuordnung mit KI, "Keine Zuordnung"-Inbox, Foto-Capture, Team-Grundlagen
**Vorbedingung:** Sprint 0 abgeschlossen ✅ (Auth, Voice Recording, STT, LLM, Dashboard)
**FFG-Sitzung:** 19.03.2026
**Referenz:** Product Brief (2026-02-08), PRD (2026-02-08), ADR-004 (STT), ADR-005 (LLM)

---

## 1. Sprint-Ziel

Sprint 0 hat bewiesen: Sprechen → Transkription → Zusammenfassung funktioniert.

Sprint 1 macht daraus ein **brauchbares Produkt**: Nachrichten werden automatisch dem richtigen Projekt zugeordnet (Forschungsfrage #3), nicht zugeordnete landen in einer Inbox, und Fotos können angehängt werden. Damit wird kurzum.app vom Prototyp zur Demo-fähigen Anwendung für Pilotbetriebe.

**Nach Sprint 1 kann man einem Elektro-Meister Folgendes zeigen:**
"Dein Monteur spricht rein, optional macht er ein Foto dazu. Die KI ordnet es dem richtigen Projekt zu. Du siehst im Dashboard was auf welcher Baustelle passiert ist. Wenn die KI unsicher ist, landet es in deiner Inbox und du ordnest es mit einem Klick zu."

---

## 2. FFG-Forschungsfrage #3

> **"Automatische Projekt-Zuordnung durch kontextuelles Matching"**
> Zuordnungsgenauigkeit von ≥75% nach 2 Wochen Nutzung (FFG-Antrag)
> Erweitert: >85% correct auto-assignment after 2 weeks of user corrections (PRD FR12)

Das ist das Kern-Differenzierungsmerkmal gegenüber "DSGVO-WhatsApp". Ohne Projekt-Zuordnung ist kurzum.app ein besserer Diktierapp. MIT Projekt-Zuordnung wird Kommunikation zu Dokumentation.

---

## 3. Arbeitspakete

### AP 1.1: Projekte/Baustellen – Datenmodell & CRUD (2–3 Tage)
**Owner:** Dev Agent
**PRD-Referenz:** FR15, FR16, FR17, FR22

**Schema (Drizzle, Erweiterung):**
```typescript
// companies (Tenant)
export const companies = pgTable('companies', {
  id: uuid('id').primaryKey().defaultRandom(),
  name: varchar('name', { length: 255 }).notNull(), // "Elektro Maier GmbH"
  createdAt: timestamp('created_at').defaultNow(),
  plan: varchar('plan', { length: 20 }).default('starter'), // starter, team, business
  trialEndsAt: timestamp('trial_ends_at'), // 14-Tage Team Trial
});

// users erweitern
export const users = pgTable('users', {
  // ... bestehende Felder aus Sprint 0
  companyId: uuid('company_id').references(() => companies.id),
  role: varchar('role', { length: 20 }).default('monteur'), // monteur, meister, buero
});

// projects (Baustellen)
export const projects = pgTable('projects', {
  id: uuid('id').primaryKey().defaultRandom(),
  companyId: uuid('company_id').references(() => companies.id).notNull(),
  name: varchar('name', { length: 255 }).notNull(), // "Neubau Urfahr, Fam. Gruber"
  customerName: varchar('customer_name', { length: 255 }), // "Familie Gruber"
  address: varchar('address', { length: 500 }), // "Hauptstraße 5, 4020 Linz"
  description: text('description'), // "Komplettsanierung Elektro EG+OG"
  status: varchar('status', { length: 20 }).default('active'), // active, paused, completed
  createdAt: timestamp('created_at').defaultNow(),
  updatedAt: timestamp('updated_at').defaultNow(),
});

// project_members (Monteur-Zuordnung zu Projekten)
export const projectMembers = pgTable('project_members', {
  projectId: uuid('project_id').references(() => projects.id).notNull(),
  userId: uuid('user_id').references(() => users.id).notNull(),
  assignedAt: timestamp('assigned_at').defaultNow(),
}, (table) => ({
  pk: primaryKey(table.projectId, table.userId),
}));

// voice_messages erweitern
export const voiceMessages = pgTable('voice_messages', {
  // ... bestehende Felder aus Sprint 0
  projectId: uuid('project_id').references(() => projects.id), // null = nicht zugeordnet
  aiSuggestedProjectId: uuid('ai_suggested_project_id').references(() => projects.id),
  aiConfidence: real('ai_confidence'), // 0.0 - 1.0
  assignedBy: varchar('assigned_by', { length: 20 }), // 'ai', 'user', 'meister'
  assignedAt: timestamp('assigned_at'),
});

// photos
export const photos = pgTable('photos', {
  id: uuid('id').primaryKey().defaultRandom(),
  userId: uuid('user_id').references(() => users.id).notNull(),
  companyId: uuid('company_id').references(() => companies.id).notNull(),
  voiceMessageId: uuid('voice_message_id').references(() => voiceMessages.id), // optional
  projectId: uuid('project_id').references(() => projects.id), // optional
  imageUrl: varchar('image_url', { length: 500 }).notNull(),
  thumbnailUrl: varchar('thumbnail_url', { length: 500 }),
  caption: text('caption'), // KI-generiert oder manuell
  createdAt: timestamp('created_at').defaultNow(),
});
```

**Aufgaben:**
- [ ] Schema-Migration (Drizzle)
- [ ] Projekte CRUD API: `POST/GET/PUT /api/projects`
- [ ] Projekt-Erstellung UI (Name, Kunde, Adresse, Beschreibung)
- [ ] Projekt-Liste im Dashboard
- [ ] Projekt-Detail-Seite mit chronologischer Timeline aller Nachrichten (FR17)
- [ ] Monteur-Zuordnung zu Projekten (FR22)
- [ ] Row-Level Security: Company-Tenant-Isolation (NFR11)

**RBAC beachten (aus PRD):**

| Aktion | Monteur | Meister | Büro |
|--------|---------|---------|------|
| Projekte sehen | Nur zugewiesene | Alle | Alle |
| Projekte erstellen/bearbeiten | ❌ | ✅ | ✅ |
| "Keine Zuordnung"-Inbox | ❌ | ✅ | ✅ |

---

### AP 1.2: KI-Projekt-Zuordnung (4–5 Tage)
**Owner:** Dev Agent + Research Documentation
**PRD-Referenz:** FR10, FR11, FR12
**FFG-Forschungsfrage:** #3

**Ansatz: Context-Matching via LLM**

Die KI bekommt das Transkript + alle aktiven Projekte des Betriebs und wählt das wahrscheinlichste Projekt. Kein NLP-Modell trainieren – LLM-basiertes Matching reicht für MVP.

**Ablauf:**
```
Sprachnachricht transkribiert
        ↓
Mistral LLM bekommt:
  - Transkript
  - Liste aller aktiven Projekte (Name, Kunde, Adresse)
  - Letzte 3 Nachrichten des Monteurs (Kontext)
        ↓
LLM antwortet: { projectId: "...", confidence: 0.87, reasoning: "..." }
        ↓
confidence ≥ 0.7 → Auto-Zuordnung (User kann korrigieren)
confidence < 0.7 → "Keine Zuordnung"-Inbox
```

**Prompt-Design:**
```
Du bist ein Projektmanagement-Assistent für einen Elektrotechnik-Betrieb.

Ordne folgende Sprachnachricht dem wahrscheinlichsten Projekt zu.

## Aktive Projekte des Betriebs:
{{#each projects}}
- ID: {{id}} | {{name}} | Kunde: {{customerName}} | Adresse: {{address}}
{{/each}}

## Letzte Nachrichten des Monteurs (Kontext):
{{#each recentMessages}}
- {{createdAt}}: {{summary.status}} (Projekt: {{projectName}})
{{/each}}

## Neue Sprachnachricht:
"{{transcript}}"

Antworte NUR mit validem JSON:
{
  "projectId": "<UUID des Projekts oder null wenn kein Match>",
  "confidence": <0.0-1.0>,
  "reasoning": "<1 Satz warum>"
}
```

**Aufgaben:**
- [ ] Assignment-Prompt entwickeln und testen
- [ ] Integration in die bestehende Voice-Processing-Pipeline (nach Zusammenfassung)
- [ ] Confidence-Threshold konfigurierbar (Start: 0.7)
- [ ] Bestätigung/Korrektur-UI: Monteur sieht "Zugeordnet zu: Neubau Urfahr ✓/✗"
- [ ] Korrektur-Feedback speichern (für spätere Verbesserung)
- [ ] Metriken tracken: Wie oft stimmt die KI? Wie oft wird korrigiert?
- [ ] Edge Cases:
  - Kein Projekt vorhanden → immer Inbox
  - Nur 1 Projekt → trotzdem Confidence prüfen
  - Monteur erwähnt 2 Projekte in einer Nachricht → höchste Confidence gewinnt

**Evaluierung (FFG-Dokumentation):**
- [ ] ADR-006 erstellen: Projekt-Zuordnungs-Algorithmus
- [ ] Test mit den 8 Beispiel-Transkripten aus STT-Evaluierung
- [ ] Testprojekte anlegen die zu den Texten passen
- [ ] Ergebnismatrix: Transkript → Erwartetes Projekt → KI-Zuordnung → Korrekt?

---

### AP 1.3: "Keine Zuordnung"-Inbox (2–3 Tage)
**Owner:** Dev Agent
**PRD-Referenz:** FR19, FR20, FR21

**Aufgaben:**
- [ ] Inbox-Seite (`/app/inbox`) für Meister/Büro
- [ ] Liste aller Nachrichten ohne Projekt-Zuordnung (projectId = null)
- [ ] Pro Nachricht:
  - Zusammenfassung anzeigen
  - "Projekt erstellen" Button → KI-vorbefülltes Formular (FR20)
  - "Zuordnen" Button → Dropdown mit bestehenden Projekten (FR21)
  - Transkript aufklappbar
- [ ] KI-Vorbefüllung bei "Neues Projekt":
  - Aus Zusammenfassung extrahieren: Kundenname, Adresse, Art der Arbeit
  - Felder vorausfüllen, Meister bestätigt/korrigiert
- [ ] Badge/Counter im Dashboard-Nav: "3 nicht zugeordnet"
- [ ] Nach Zuordnung: Nachricht verschwindet aus Inbox, erscheint in Projekt-Timeline

**UI:**
```
[Inbox /app/inbox]
┌─────────────────────────────────────┐
│  📥 Nicht zugeordnet (3)            │
│                                     │
│  ┌─────────────────────────────┐    │
│  │ 📝 "Unterverteilung fertig, │    │
│  │    24 Automaten eingebaut"  │    │
│  │ 🕐 vor 2 Stunden           │    │
│  │ 💡 KI vermutet: Neubau?    │    │
│  │                             │    │
│  │ [Neues Projekt] [Zuordnen▾] │    │
│  └─────────────────────────────┘    │
│                                     │
│  ┌─────────────────────────────┐    │
│  │ 📝 "Keller fertig, Leerr..." │    │
│  │ 🕐 vor 5 Stunden           │    │
│  │                             │    │
│  │ [Neues Projekt] [Zuordnen▾] │    │
│  └─────────────────────────────┘    │
└─────────────────────────────────────┘
```

---

### AP 1.4: Foto-Capture (3–4 Tage)
**Owner:** Dev Agent
**PRD-Referenz:** FR2, FR3, FR4, FR5

**Aufgaben:**
- [ ] Kamera-Capture via `<input type="file" accept="image/*" capture="environment">`
- [ ] Zwei Modi:
  1. **An Sprachnachricht anhängen** (FR2): Foto wird mit Voice Message verknüpft
  2. **Standalone-Foto** (FR3): Foto direkt in ein Projekt posten
- [ ] Image-Upload an Vercel Blob (EU-Region)
- [ ] Thumbnail-Generierung (serverseitig oder via Vercel Image Optimization)
- [ ] Fotos in Projekt-Timeline anzeigen (mit Zeitstempel, Ersteller)
- [ ] Foto-Vorschau vor Upload
- [ ] Max. Dateigröße: 10MB (Client-seitige Kompression wenn nötig)
- [ ] EXIF-Daten entfernen (Privacy – keine GPS-Koordinaten speichern)

**UI-Integration:**
```
[Aufnahme-Seite erweitert]
┌─────────────────────────────────┐
│  🎙️ kurzum                      │
│                                 │
│  🔴 [  Aufnahme  ]              │
│  ▓▓▓▓░░░░░░ 0:12               │
│                                 │
│  [▶ Anhören] [📷 Foto] [Senden] │
│                                 │
│  📎 1 Foto angehängt            │
│  [🖼️ Vorschau]                  │
└─────────────────────────────────┘
```

**Projekt-Timeline mit Fotos:**
```
[Projekt: Neubau Urfahr]
┌─────────────────────────────────┐
│ 📝 Markus, 14:30                │
│ "Unterverteilung OG fertig"     │
│ Material: 3x B16, 1x FI 40er   │
│ [🖼️ Foto 1] [🖼️ Foto 2]        │
│                                 │
│ 📷 Thomas, 11:15                │
│ [🖼️ Zählerkasten vorher]        │
│                                 │
│ 📝 Markus, 08:20                │
│ "Angefangen EG, Dosen setzen"   │
└─────────────────────────────────┘
```

---

### AP 1.5: Team-Grundlagen (2–3 Tage)
**Owner:** Dev Agent
**PRD-Referenz:** FR23, FR24, FR25, FR26, FR27

**Scope für Sprint 1 (Minimum Viable Team):**
- [ ] Company-Erstellung bei Erstregistrierung (automatisch)
- [ ] Einladungslink generieren (per E-Mail via Resend)
- [ ] Eingeladener User kann sich registrieren und wird automatisch zur Company hinzugefügt
- [ ] Rollen zuweisen: Monteur / Meister / Büro
- [ ] Team-Liste im Dashboard (wer ist im Betrieb)
- [ ] Meister/Büro können Rollen ändern
- [ ] Meister/Büro können Mitarbeiter entfernen (FR27)

**NICHT in Sprint 1:**
- SMS-Einladung (nur E-Mail)
- Starter Plan Limit Enforcement (FR28, kommt in Sprint 2)
- Subscription/Billing (kommt viel später)

**Einladungs-Flow:**
```
Meister → "Team" → "Einladen"
  → E-Mail eingeben + Rolle wählen
  → Resend schickt Einladungslink
  → Neuer User klickt Link → Login/Registrierung
  → Automatisch in Company + Rolle zugewiesen
```

---

### AP 1.6: Dashboard-Erweiterung (2 Tage)
**Owner:** Dev Agent
**PRD-Referenz:** FR31, FR32

**Aufgaben:**
- [ ] Dashboard umbauen: Projekt-basierte Ansicht statt Nachrichten-Liste
  - Pro Projekt: letzte Aktivität, Anzahl Nachrichten heute, Status
  - Klick → Projekt-Timeline
- [ ] Meister/Büro: alle Projekte sehen
- [ ] Monteur: nur zugewiesene Projekte (FR32)
- [ ] Inbox-Badge: "3 nicht zugeordnet" (nur Meister/Büro)
- [ ] Aktivitäts-Feed: "Was ist heute/diese Woche passiert?"
- [ ] Fotos in Timeline eingebettet

---

## 4. Verarbeitungs-Pipeline (aktualisiert)

```
User nimmt Sprachnachricht auf (optional + Foto)
        ↓
POST /api/voice (Audio + optional Foto)
        ↓
Audio → Vercel Blob (temp)
Foto → Vercel Blob (permanent, Thumbnail generieren)
DB: voice_message erstellt (status: 'processing')
        ↓
Mistral Voxtral STT → transcript
        ↓
Mistral LLM #1 → summary (JSON: status, material, nextSteps, problems, project)
        ↓
Mistral LLM #2 → project assignment (projectId, confidence, reasoning)
        ↓
confidence ≥ 0.7 → projectId setzen, assignedBy: 'ai'
confidence < 0.7 → projectId: null (→ Inbox)
        ↓
DB: voice_message aktualisiert (status: 'done')
        ↓
Dashboard/Timeline zeigt Ergebnis
```

**Hinweis:** LLM #1 (Summary) und LLM #2 (Assignment) können in einem einzigen API-Call kombiniert werden, wenn der Prompt es hergibt. Evaluieren ob ein kombinierter Prompt die gleiche Qualität liefert – dann spart das Latenz und Kosten.

---

## 5. Nicht-funktionale Anforderungen (Sprint 1 spezifisch)

### Row-Level Security (Tenant Isolation)
- Jede DB-Query muss `companyId` filtern
- Kein User darf Projekte/Nachrichten anderer Companies sehen
- Middleware: `companyId` aus Session extrahieren, in alle Queries injizieren

### RBAC Middleware
- Route-basiert: `/app/inbox` → nur Meister/Büro
- API-basiert: `POST /api/projects` → nur Meister/Büro
- Daten-basiert: Monteur sieht nur zugewiesene Projekte

### DSGVO (Fotos)
- EXIF-Daten serverseitig entfernen (keine GPS-Koordinaten)
- Fotos an Company/Projekt gebunden
- Bei Company-Löschung: alle Fotos löschen

---

## 6. Erfolgskriterien Sprint 1

- [ ] Projekte können erstellt und bearbeitet werden
- [ ] KI ordnet Sprachnachrichten automatisch Projekten zu
- [ ] Zuordnungs-Confidence wird angezeigt
- [ ] User kann Zuordnung bestätigen oder korrigieren
- [ ] Nicht zugeordnete Nachrichten landen in der Inbox
- [ ] Aus Inbox: neues Projekt (KI-vorbefüllt) oder bestehendem Projekt zuordnen
- [ ] Fotos können aufgenommen und an Nachrichten/Projekte angehängt werden
- [ ] Projekt-Timeline zeigt Nachrichten + Fotos chronologisch
- [ ] Team-Einladung funktioniert (E-Mail)
- [ ] RBAC: Monteur sieht nur seine Projekte, Meister/Büro sieht alles
- [ ] ADR-006 dokumentiert: Projekt-Zuordnungs-Evaluierung

---

## 7. Referenzen

- **Sprint 0 Briefing:** bmad-briefing-kurzum-sprint0.md
- **Product Brief:** product-brief-kurzum-app-2026-02-08.md (Abschnitt: Core Features #2, #3, #4)
- **PRD:** prd.md (FR10-FR22, FR31-FR32, NFR11, RBAC Matrix)
- **ADR-004:** STT Provider Evaluation
- **ADR-005:** LLM Summarization Prompt
- **User Journeys:** PRD Journey 2 (Offline → Inbox), Journey 3 (Stefan Setup), Journey 4 (Andrea)
