# ADR-010: Summary-basierte Projekt-Zuordnung statt Rohes Transkript

**Datum:** 2026-02-16
**Status:** Vorgeschlagen
**Vorgaenger:** ADR-006 (Assignment V1 Evaluation)
**Betrifft:** Story 14.1 (Assignment V2 Prompt & Continuity Signal)

## Kontext

Bei manuellen Tests mit realen Sprachnachrichten (Frauenstimme, oberoesterreichische Mundart, text-07) wurde festgestellt, dass die Assignment-Pipeline eine klar erkennbare "Gruber"-Nachricht **nicht** dem aktiven Projekt "Zaehlerkasten Gruber" zuordnet — obwohl die Summary den Kunden korrekt erkennt.

### Beobachtete Daten (2026-02-16, Produktion)

| Schicht | Inhalt |
|---------|--------|
| **Rohes Transkript (Voxtral STT)** | "Bin fertig bei Gruber Baustell, alles erledigt, **Zollerkosten** montiert, Abnahme gemacht. Fahre jetzt weiter zum naechsten Kunden in die Linzer Strasse..." |
| **Summary (Mistral Small)** | "Arbeiten beim **Kunden Gruber** abgeschlossen. **Zaehlerkasten** montiert und Abnahme durchgefuehrt." |
| **Assignment-Ergebnis** | `{ projectId: null, confidence: 0.3, reasoning: "...Projekt ist bereits abgeschlossen" }` |

### Aktive Projekte in DB

| Projekt | Kunde | Adresse | Status |
|---------|-------|---------|--------|
| Zaehlerkasten Gruber | Gruber | Alleegasse 48 | **active** |
| Elektroinstallation Mueller | Familie Mueller | Hauptstrasse 5 | active |

### Zwei Probleme identifiziert

**Problem 1 — STT-Fehler werden an Assignment weitergereicht:**
Die aktuelle Pipeline fuettert das **rohe Transkript** in die Assignment-LLM-Stufe:
```
Audio -> STT -> Transcript -> Summarize -> Summary
                    |
                    v
              [Assignment] <-- benutzt "Zollerkosten", "Gruber Baustell"
```
Die Summary korrigiert STT-Fehler bereits (Zollerkosten -> Zaehlerkasten, Gruber Baustell -> Kunden Gruber), aber die Assignment-Stufe profitiert nicht davon.

**Problem 2 — LLM halluziniert Projekt-Status:**
Das LLM interpretiert den Nachrichteninhalt ("alles erledigt, Abnahme gemacht") als Signal, dass das Projekt abgeschlossen sei. Es behauptet "das letzte Projekt mit diesem Kunden ist bereits abgeschlossen" — obwohl der Projektstatus in der DB `active` ist. Das Prompt sagt dem LLM nicht, dass es den Projekt-Status **nicht** aus dem Nachrichteninhalt ableiten soll.

## Entscheidung

### 1. Assignment auf Summary + Transcript basieren

Die Assignment-Stufe erhaelt **beide** Texte: die bereinigte Summary als primaeres Signal und das rohe Transkript als zusaetzlichen Kontext. Die Summary enthaelt bereits korrigierte Namen, Fachbegriffe und strukturierte Informationen.

Neue Pipeline:
```
Audio -> STT -> Transcript -> Summarize -> Summary
                                              |
                                    [Assignment] <-- bekommt Summary + Transcript
```

Umsetzung in `lib/ai/assignment.ts`:
- `assignToProject()` erhaelt neuen Parameter: `summary: VoiceSummary | null`
- `buildUserMessage()` zeigt Summary-Felder (status, project, material) vor dem Transkript
- Voice-Route uebergibt Summary-Objekt an `assignToProject()`

### 2. Prompt-Ergaenzung: Kein Projektstatus aus Nachrichteninhalt

Neuer Abschnitt im ASSIGNMENT_SYSTEM_PROMPT_V2:
```
WICHTIG: Leite den Projektstatus NICHT aus dem Nachrichteninhalt ab.
"Alles erledigt", "Abnahme gemacht" oder "Bin fertig" bedeuten, dass HEUTE
die Arbeit abgeschlossen ist — NICHT dass das Projekt beendet ist.
Die Projektliste enthaelt nur AKTIVE Projekte.
```

### 3. Signal-Hierarchie aktualisieren

Neuer Prompt fuegt die Summary als staerkstes Signal ein:
```
Zuordnungs-Strategie (in Prioritaetsreihenfolge):
1. Summary-Felder "project" und Kundenname — staerkstes Signal (STT-Fehler bereits korrigiert)
2. Kundenname, Adresse, Projektname im Transkript — direkte Referenzen
3. Kontinuitaets-Signal — letzte Nachrichten des Sprechers
4. Elektro-Fachbegriffe — nur in Kombination mit anderen Signalen
```

## Begruendung

| Kriterium | Transcript-only (V1) | Summary + Transcript (V2) |
|-----------|---------------------|---------------------------|
| STT-Fehler "Zollerkosten" | Kein Match mit "Zaehlerkasten" | Summary sagt "Zaehlerkasten" -> Match |
| STT-Fehler "Gruber Baustell" | Partieller Match, unsicher | Summary sagt "Kunden Gruber" -> klar |
| LLM-Halluzination (Status) | "Projekt abgeschlossen" (falsch) | Prompt verbietet Status-Ableitung |
| Zusaetzliche Latenz | 0ms | 0ms (Summary existiert bereits) |
| Zusaetzliche Kosten | $0 | ~50 Tokens mehr im Prompt (~$0.00002) |

Die Summary wird ohnehin **vor** dem Assignment berechnet und ist in der DB gespeichert. Es entsteht kein zusaetzlicher LLM-Call, nur ein groesserer User-Message-Block.

## Erwartete Verbesserung

| Test-Case (ADR-006) | V1 Ergebnis | V2 Erwartung |
|---------------------|-------------|--------------|
| TC-03 (Materialbestellung) | False Positive (0.70) | Inbox (Summary sagt "Material", kein Projekt) |
| TC-05 (Gruber Status-Update) | False Negative (0.30) | Gruber (Summary sagt "Kunden Gruber") |
| Neuer Test: Frauenstimme/Mundart | False Negative (0.30) | Gruber (Summary korrekt) |

Erwartete Genauigkeit: 75% -> >=87.5% (min. 7/8 korrekt, TC-03 + TC-05 gefixt)

## Konsequenzen

### Story 14.1 Erweiterungen
1. `assignToProject()` bekommt optionalen `summary`-Parameter
2. `buildUserMessage()` zeigt Summary-Felder im User-Message
3. ASSIGNMENT_SYSTEM_PROMPT_V2 enthaelt Anti-Halluzinations-Regel und Summary-Signal-Hierarchie
4. Voice-Route uebergibt Summary an Assignment-Stufe

### Story 14.2 Erweiterungen
- V2-Evaluation muss den Frauenstimmen-Testfall (text-07-ooe-weiblich) einschliessen
- ADR-011 dokumentiert V1-vs-V2 Vergleich inklusive Summary-Effekt
- TC-09 (neuer Edge Case): Identisches Transkript, unterschiedliche Summary-Qualitaet

## Rohdaten

- **Produktions-DB-Abfrage:** voice_messages.id = `7557310d-ecfd-419a-9169-34a0851c6d80` (Gruber False Negative)
- **Aktives Projekt:** projects.id = `16a34665-28cb-4ad1-bc13-e21800a3ccad` (Zaehlerkasten Gruber, status=active)
- **Test-Audio:** `docs/_masemIT/audios/text-04-ooe.mp3` (Frauenstimme, Mundart, text-07 Inhalt)
- **Pipeline-Code:** `lib/ai/assignment.ts:108` (`assignToProject()` nimmt nur `transcript`)
- **Prompt:** `lib/ai/prompts.ts:32` (ASSIGNMENT_SYSTEM_PROMPT, keine Summary-Referenz)

## Beteiligte

- Analyse: Winston (Architect Agent), Claude Opus 4.6
- Testdaten: Mario/Sempre (manuelle Produktion-Tests mit Frauenstimme/Mundart-Aufnahmen)
- Vorgaenger-Analyse: ADR-006 (Party Mode: Mary, Winston, Paige)
