# kurzum.app – DO & DON'T Liste für BMAD Team

**Stand:** 2026-02-12
**Kontext:** FFG Basisprogramm Kleinprojekt, Antragsnr. 69168884
**Einreichdatum:** 10.02.2026
**FFG-Sitzung:** 19.03.2026
**Geplanter Projektstart lt. Antrag:** 01.07.2026

---

## Rechtlicher Rahmen

> „Der frühestmögliche Zeitpunkt für die Anerkennung von Kosten ist das **Einreichdatum des Antrags**."
> — [FFG Basisprogramm Kleinprojekt 2024](https://www.ffg.at/ausschreibung/basisprogramm-kleinprojekt-2024)

Das bedeutet: Alle Kosten ab 10.02.2026 sind **potenziell förderfähig**, sofern das Projekt bewilligt wird. Es gibt keine Einschränkung, was entwickelt werden darf – nur Kosten vor dem 10.02.2026 dürfen nicht abgerechnet werden.

**Risiko:** Wird das Projekt nicht bewilligt (Entscheidung 19.03.2026), gehen die bis dahin investierten Stunden auf eigene Kosten.

---

## ✅ DO – Das dürfen/sollen wir JETZT machen

### Architektur & Design
- [ ] System-Architektur für Voice Pipeline dokumentieren
- [ ] Datenmodell: Projekte, Baustellen, Sprachnachrichten, Zusammenfassungen, User
- [ ] DSGVO-Pipeline Design: Audio-Verarbeitung, Speicherdauer, Löschkonzept
- [ ] API-Design für kurzum Backend (REST/tRPC)
- [ ] Tech-Stack Entscheidungen dokumentieren

### KI-Provider: Mistral AI (EU-hosted) = Primär
- [ ] **Mistral Voxtral** für STT (Speech-to-Text) – EU-gehostet, DSGVO-konform
- [ ] **Mistral LLM** für Zusammenfassung – EU-gehostet, DSGVO-konform
- [ ] Whisper (self-hosted) als **Fallback** für STT evaluieren
- [ ] Evaluierung dokumentieren: Dialekt-Toleranz, Baustellenlärm, Fachbegriffe, Latenz, Kosten
- [ ] **Forschungsfragen aus FFG-Antrag bearbeiten:**
  1. Domänenspezifische Spracherkennung bei Baustellenlärm und österreichischen Dialekten
  2. Semantische Zusammenfassung unstrukturierter Sprachnachrichten
  3. Automatische Projekt-Zuordnung durch kontextuelles Matching
  4. Machbarkeit einer vollständig DSGVO-konformen KI-Pipeline auf EU-Servern

### Prototyp (hinter Auth!)
- [ ] Magic Link Authentication implementieren
- [ ] Voice Recording im Browser/PWA (MediaRecorder API)
- [ ] Mistral Voxtral STT-Integration
- [ ] Mistral LLM-Zusammenfassung (Prompt Engineering für Handwerker-Kontext)
- [ ] Einfaches Dashboard: Eingegangene Nachrichten + Zusammenfassungen anzeigen
- [ ] Projekt/Baustellen-Zuordnung (manuell als erster Schritt)

### Infrastruktur
- [ ] kurzum.app Backend-Projekt aufsetzen (Next.js, NeonDB, Vercel)
- [ ] Auth-geschützter Bereich (/app oder /dashboard)
- [ ] Audio-File-Handling: Upload, temporäre Speicherung, Löschung nach Verarbeitung
- [ ] EU-Region für ALLE Services sicherstellen (Frankfurt)
- [ ] QStash für Queue-basierte Verarbeitung (Mistral AI Latenz-Handling)

### Marktvalidierung & Pilot-Vorbereitung
- [ ] Pilot-Mails versenden (Welle 1–3)
- [ ] Landing Page optimieren basierend auf Feedback
- [ ] Waitlist-Anmeldungen tracken und qualifizieren
- [ ] Pilot-Onboarding-Prozess definieren

### Dokumentation (WICHTIG für FFG!)
- [ ] Stundenaufzeichnungen ab 10.02.2026 führen
- [ ] Git-Commits mit aussagekräftigen Messages
- [ ] Technische Entscheidungen dokumentieren (ADRs)
- [ ] Forschungstagebuch: Was wurde getestet, was waren die Ergebnisse
- [ ] Screenshots/Logs von STT-/LLM-Evaluierungen aufheben

---

## ❌ DON'T – Das NICHT machen

### Kosten & Abrechnung
- ❌ Kosten vor dem 10.02.2026 als Projektkosten verbuchen
- ❌ Landing Page / Waitlist als FFG-Projektkosten abrechnen (ist Pre-Project Preparation)
- ❌ Stunden ohne Dokumentation arbeiten (nicht nachweisbar = nicht förderfähig)

### KI-Provider
- ❌ **US-basierte KI-Provider als Primär-Provider nutzen** (OpenAI GPT, Anthropic Claude API für Produktions-Pipeline) – nur als Evaluierungs-Benchmark erlaubt
- ❌ Audio-Daten an US-Server senden ohne explizite EU-DPA (Data Processing Agreement)
- ❌ Mistral AI Entscheidung aus Product Brief ignorieren – Provider-Wechsel nur bei dokumentiertem Versagen
- ❌ STT/LLM-API-Keys im Frontend oder in Logs exposen

### Öffentlichkeit
- ❌ Prototyp öffentlich zugänglich machen (muss hinter Auth sein)
- ❌ App Store Listing oder öffentliche Beta vor Bewilligung
- ❌ "Produkt-Launch" kommunizieren – es ist ein Forschungs-/Entwicklungsprojekt

### Technisch
- ❌ Audio-Files dauerhaft/unkontrolliert speichern (DSGVO! Auto-Deletion nach 90 Tagen lt. PRD)
- ❌ US-basierte Services ohne EU-Datenverarbeitung nutzen
- ❌ Produktionsreife Features bauen – Fokus auf Forschung & Prototyp
- ❌ Fertige Features als "erforscht" verkaufen – der Forschungscharakter muss erkennbar bleiben

### FFG-Kommunikation
- ❌ Vor Bewilligung den Projektstart auf der FFG-Plattform melden
- ❌ Förderung als "sicher" behandeln – bis 19.03. ist es ein Risiko
- ❌ Projektplan wesentlich ändern ohne FFG zu informieren (nach Bewilligung)

---

## Prioritäten bis 19.03.2026 (FFG-Sitzung)

### Woche 1 (10.–16.02.) ✅ ERLEDIGT
1. ~~Landing Page live~~ ✅
2. ~~DNS & E-Mail Setup~~ ✅
3. ~~Pilot-Mails Welle 1~~ ✅
4. ~~E-Mail-Flow testen (Double-Opt-In)~~ ✅
5. ~~Secret Key rotieren~~ ✅

### Woche 2 (17.–23.02.)
1. Auth-System (Magic Link) implementieren
2. Mistral AI API-Zugang einrichten (Voxtral + LLM)
3. Voice Recording Prototyp (Browser)
4. Pilot-Mails Welle 2
5. Erste LinkedIn-Posts (organisch)

### Woche 3 (24.02.–02.03.)
1. Mistral Voxtral STT → LLM Zusammenfassungs-Pipeline
2. Einfaches Dashboard hinter Auth
3. Pilot-Mails Welle 3
4. Datenmodell finalisieren

### Woche 4 (03.–09.03.)
1. Ende-zu-Ende Prototyp: Sprechen → Transkription → Zusammenfassung → Dashboard
2. Whisper (self-hosted) als Fallback evaluieren (Vergleich mit Mistral Voxtral)
3. DSGVO-Dokumentation

### Woche 5 (10.–16.03.)
1. Prototyp stabilisieren
2. Pilot-Betrieben Demo zeigen (wenn Interesse)
3. Dokumentation für FFG aufbereiten

### 19.03.2026: FFG-Sitzung → Entscheidung

---

## Hinweise für BMAD-Agenten

### KI-Provider Hierarchie
1. **Mistral AI (EU)** = Primär (STT: Voxtral, LLM: Mistral Large/Medium)
2. **Whisper self-hosted** = Fallback für STT
3. **Andere Provider** = nur als Benchmark für Evaluierung, nicht für Produktion

### Commit-Messages
- ✅ "Evaluate Mistral Voxtral for Austrian dialect recognition"
- ✅ "Prototype: audio chunking for long voice messages (>60s)"
- ✅ "Compare Whisper large-v3 vs Mistral Voxtral: WER for construction noise"
- ❌ "fix button color"

### Allgemein
- **Dokumentation ist Pflicht** – die FFG prüft im Endbericht ob F&E stattgefunden hat
- **Alles hinter Auth** – der Prototyp ist nicht öffentlich
- **EU-Server only** – Frankfurt Region für NeonDB, Vercel, und alle API-Calls
- **Audio-Files** nach Verarbeitung mit klarer Retention Policy speichern (90 Tage lt. PRD, dann Auto-Delete)
- **QStash** für asynchrone Verarbeitung nutzen (Mistral API Latenz abfangen)
- **Referenz-Dokumente:** Product Brief (2026-02-08), PRD (2026-02-08), FFG-Antrag (69168884)
