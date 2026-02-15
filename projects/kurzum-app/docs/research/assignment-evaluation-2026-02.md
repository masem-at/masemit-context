# Assignment Pipeline Evaluation — Februar 2026

**Project:** kurzum.app
**FFG Antragsnr.:** 69168884
**FFG Forschungsfrage #3:** "Kann kontextuelles Matching zur automatischen Projekt-Zuordnung von Sprachnachrichten genutzt werden?"
**Date:** 2026-02-15
**Model:** Mistral Small (mistral-small-latest, 24B parameters)

## Ziel

Evaluierung der automatischen Projekt-Zuordnungs-Pipeline (Assignment Pipeline) als dritte Stufe der Voice-Verarbeitung. Nach Spracherkennung (FQ#1, STT) und Zusammenfassung (FQ#2, LLM Summarization) wird das Transkript einer Sprachnachricht automatisch einem aktiven Bauprojekt zugeordnet. Die zentrale Forschungsfrage: Kann ein LLM aus dem Transkript-Kontext (Kundennamen, Adressen, Fachbegriffe) das richtige Projekt identifizieren — auch bei STT-Fehlern, Dialekt und fehlenden expliziten Referenzen?

## Testsetup

### Modell-Konfiguration
- **Model:** `mistral-small-latest` (Mistral Small 3.2, 24B Parameter)
- **Prompt:** ASSIGNMENT_V1 (identisch mit Produktions-Prompt in `lib/ai/prompts.ts`)
- **Parameter:** `temperature: 0` (deterministisch), `responseFormat: json_object`, `maxTokens: 512`
- **EU-hosted:** Ja (Frankreich, DSGVO-konform)

### Pipeline-Architektur
Die Assignment Pipeline (`lib/ai/assignment.ts`) arbeitet als dritte Stufe im Voice-Processing:
1. **Kontext laden:** Aktive Projekte des Unternehmens (Name, Kunde, Adresse) + letzte 3 Nachrichten des Sprechers
2. **Prompt aufbauen:** Projektliste + Kontinuitäts-Signal + aktuelles Transkript als strukturierte User-Message
3. **LLM-Call:** Mistral Small mit ASSIGNMENT_SYSTEM_PROMPT
4. **Antwort:** `{ projectId, confidence, reasoning }` — Zod-validiert
5. **Routing:** `confidence >= 0.7` automatische Zuordnung, `< 0.7` Inbox (manuelle Zuordnung)

### Test-Transkripte (aus STT-Evaluation)
8 Transkripte aus der Voxtral-Evaluation (ADR-004) — reale STT-Ausgabe mit typischen Fehlern:

| # | Quelle | Variante | Hintergrund | Inhalt |
|---|--------|---------|------------|---------|
| 1 | text-01 | Hochdeutsch | Stille | Baustellenbericht |
| 2 | text-02 | Waldviertlerisch | Stille | Gleicher Bericht, Dialekt |
| 3 | text-04 | Hochdeutsch | Stille | Materialbestellung |
| 4 | text-06 | Dialekt | Stille | Problemmeldung |
| 5 | text-07 | Schnell/beilaeufig | Autofahrt | Status-Update |
| 6 | text-08 | Dialekt | Stille | Tagesbericht (lang, 2 Projekte) |
| 7 | text-10 | Dialekt | Baustelle | Notfall + starker Laerm |
| 8 | text-11 | Normal | Stille | Nachkalkulation |

### Test-Projekte (simulierte aktive Projektliste)

| # | Projektname | Kunde | Adresse |
|---|-------------|-------|---------|
| 1 | Neubau Mueller | Familie Mueller | Hauptstrasse 5 |
| 2 | Sanierung Huber | Huber | Gartenstrasse 12 |
| 3 | Zaehlerkasten Gruber | Gruber | — |
| 4 | Neubau Schrems | Familie Novak | Schrems |
| 5 | PV-Anlage Schuster | Schuster | — |

### Testbedingungen
- **Keine Vorgaenger-Nachrichten** — Baseline ohne Kontinuitaets-Signal
- **8 Standard-Tests + 2 Edge-Case-Tests** (Einzelprojekt-Szenarien)
- **Evaluierungs-Script:** `scripts/assignment-evaluation.ts` (gleiche Prompt-Logik wie Produktion)

## Ergebnis-Matrix

| TC | Transkript | Variante | Erwartetes Projekt | KI-Zuordnung | Conf. | Korrekt | Auto |
|----|-----------|----------|-------------------|-------------|-------|---------|------|
| 01 | Baustellenbericht | Hochdeutsch | Neubau Mueller | Neubau Mueller | 0.90 | **JA** | JA |
| 02 | Baustellenbericht | Waldviertlerisch | Neubau Mueller | Neubau Mueller | 0.90 | **JA** | JA |
| 03 | Materialbestellung | Hochdeutsch | Inbox (null) | Zaehlerkasten Gruber | 0.70 | **NEIN** | JA |
| 04 | Problemmeldung | Dialekt | Sanierung Huber | Sanierung Huber | 0.90 | **JA** | JA |
| 05 | Status-Update | Auto/schnell | Zaehlerkasten Gruber | Inbox (null) | 0.30 | **NEIN** | NEIN |
| 06 | Tagesbericht | Dialekt | Neubau Schrems | Neubau Schrems | 0.90 | **JA** | JA |
| 07 | Notfall | Dialekt + Laerm | Inbox (null) | Inbox (null) | 0.20 | **JA** | NEIN |
| 08 | Nachkalkulation | Normal | PV-Anlage Schuster | PV-Anlage Schuster | 0.90 | **JA** | JA |

## Quantitative Auswertung

| Metrik | Wert | Ziel |
|--------|------|------|
| **Gesamt-Genauigkeit** | **75% (6/8)** | >= 60% (Sprint 1) |
| Korrekt auto-zugeordnet | 5 | — |
| Falsch auto-zugeordnet | 1 | — |
| Korrekt Inbox | 1 | — |
| Falsch Inbox | 1 | — |
| Avg Confidence | 0.71 | — |
| Avg Latenz | 912ms | < 5s (S1-NFR5) |
| Avg Kosten/Zuordnung | $0.00007 | — |

## Confidence-Kalibrierung

| Confidence-Band | Korrekt | Total | Genauigkeit |
|----------------|---------|-------|-------------|
| >= 0.8 (Klar) | 5 | 5 | **100%** |
| 0.5 – 0.8 (Wahrscheinlich) | 0 | 1 | 0% |
| < 0.5 (Unsicher) | 1 | 2 | 50% |

**Kernaussage:** Bei hoher Confidence (>= 0.8) ist das Modell zu 100% zuverlaessig. Die Schwelle 0.7 ist grenzwertig — TC-03 wurde mit genau 0.70 falsch auto-zugeordnet. Eine hoehere Schwelle (0.75 oder 0.80) wuerde diesen Fehler eliminieren, aber das Inbox-Volumen erhoehen.

## Qualitative Analyse der Zuordnungs-Strategie

### Kundenname als staerkstes Signal
In 5 von 6 korrekten Zuordnungen war der Kundenname das primaere Signal. Das LLM erkennt zuverlaessig Namensvarianten (z.B. "Nowak" im Transkript vs. "Novak" im Projekt) und kombiniert sie mit Adresshinweisen.

**TC-01/02 (Mueller):** Klare Referenz "Mueller, Hauptstrasse 5" — auch die Dialektvariante (TC-02) wird identisch korrekt zugeordnet. Die Kombination Kunde + Adresse ergibt hohe Confidence (0.90).

**TC-04 (Huber/Gartenstrasse):** Trotz STT-Fehler "Gordenstrasse" statt "Gartenstrasse" erkennt das LLM den Kunden "Huber" und die Hausnummer 12 und ordnet korrekt zu. Das zeigt Robustheit gegenueber typischen STT-Verzerrungen.

**TC-08 (Schuster/PV-Anlage):** STT transkribiert "BV-Anlage" statt "PV-Anlage", aber der Kundenname "Schuster" plus der Kontext (Ueberspannungsschutz, Montageschienen, Dachneigung) reichen fuer korrekte Zuordnung.

### STT-Fehler als Stoerfaktor
In beiden Fehlerfaellen spielen STT-Artefakte eine zentrale Rolle:

**TC-03 (False Positive — Materialbestellung):** Das Transkript enthaelt "PE-Schiene fuer den Zaehlerkasten" — der Fachbegriff "Zaehlerkasten" erscheint zufaellig auch im Projektnamen "Zaehlerkasten Gruber". Das LLM interpretiert diesen Fachbegriff faelschlich als Projektbezug und ordnet mit Confidence 0.70 zu. **Erkenntnis:** Projektnamen sollten keine Elektro-Fachbegriffe enthalten, oder der Prompt muss expliziter zwischen Fachbegriff und Projektname unterscheiden.

**TC-05 (False Negative — Status-Update Gruber):** Voxtral transkribiert "Gruber Baustell" (abgeschnittenes Wort) und "Zollerkosten" (statt "Zaehlerkasten"). Obwohl "Gruber" im Transkript vorkommt, erkennt das LLM die Zuordnung nicht, weil die "Linzer Strasse" als unbekannte Adresse verunsichert. **Erkenntnis:** Kombination aus schneller Sprache + Autobahn-Laerm + STT-Fehlern ueberfordert das kontextuelle Matching. Kontinuitaets-Signal (Vorgaenger-Nachrichten) koennte hier helfen.

### Multi-Projekt-Szenario
**TC-06 (Tagesbericht mit 2 Projekten):** Der Sprecher erwaehnt "Neubau in Schrems, Familie Nowak" UND "Heidenreichstein zum Altbau von der Frau Winkler". Das LLM waehlt korrekt das erste/explizitere Projekt (Schrems/Novak), da Heidenreichstein/Winkler nicht in der Projektliste existiert. Bei einem echten Multi-Projekt-Szenario (beide in der Liste) wuerde die Confidence wahrscheinlich sinken.

### Graceful Degradation bei schlechter STT-Qualitaet
**TC-07 (Notfall, stark verzerrtes Transkript):** Die Voxtral-Ausgabe ist nahezu unlesbar ("herkommen zu der Hure", "Thementaste von der Teilerkosten"). Das LLM erkennt korrekt, dass kein Projektbezug herstellbar ist und gibt `null` mit Confidence 0.20 zurueck. **Erkenntnis:** Das System degradiert graceful — bei unverstaendlichen Transkripten wird die Nachricht sicher in die Inbox geleitet.

### Edge Case: Nur 1 Projekt in der Liste
Zwei zusaetzliche Tests mit nur einem Projekt in der Liste:
- TC-01 mit nur Mueller-Projekt: Korrekte Zuordnung (0.90) — kein Single-Option-Bias
- TC-03 mit nur Mueller-Projekt: Korrekt Inbox (0.30) — das LLM ordnet nicht blind dem einzigen verfuegbaren Projekt zu

## Prompt-Design

Der ASSIGNMENT_SYSTEM_PROMPT (Version V1, gespeichert in `lib/ai/prompts.ts`) folgt einem strukturierten Ansatz:

1. **Rollenkontext:** "Zuordnungs-Assistent fuer Handwerksbetriebe"
2. **Input-Beschreibung:** Transkript, Projektliste, Vorgaenger-Nachrichten
3. **Output-Schema:** JSON mit `projectId`, `confidence`, `reasoning`
4. **Zuordnungs-Strategie:** Hierarchie der Signale (Kundenname > Adresse > Fachbegriffe > Kontinuitaet)
5. **Confidence-Skala:** Klare Abstufung (>= 0.8 klar, 0.5-0.8 wahrscheinlich, < 0.5 unsicher)
6. **Sicherheitsregeln:** Nur UUIDs aus der Liste verwenden, null bei Unsicherheit

Die User-Message wird strukturiert aufgebaut (siehe `buildUserMessage()` in `lib/ai/assignment.ts`):
```
Aktive Projekte:
1. [uuid] "Projektname" | Kunde: Name | Adresse: Strasse
...

Letzte Nachrichten des Sprechers:
(keine bisherigen Nachrichten)

Aktuelles Transkript:
"..."
```

## Kostenanalyse

| Metrik | Wert |
|--------|------|
| Avg Input Tokens/Zuordnung | 856 |
| Avg Output Tokens/Zuordnung | 94 |
| Kosten pro Zuordnung | ~$0.00007 |
| 100 Zuordnungen/Tag | ~$0.007/Tag |
| Monatlich (3000 Zuordnungen) | ~$0.21/Monat |
| Combined STT + LLM + Assignment monatlich | ~$45 (STT) + $0.12 (LLM) + $0.21 (Assignment) = ~$45.33/Monat |

Die Assignment-Kosten sind gegenueber STT vernachlaessigbar.

## Latenzmessung

| Metrik | Wert |
|--------|------|
| Min Latenz | 534ms (TC-07, kurzes Transkript) |
| Max Latenz | 1352ms (TC-01) |
| Avg Latenz | 912ms |
| Gesamt-Budget (STT + LLM + Assign) | ~697ms + ~2106ms + ~912ms = ~3715ms |

Die Gesamtlatenz der synchronen Pipeline (STT + Summarization + Assignment) liegt bei ca. 3.7 Sekunden — deutlich unter dem 5-Sekunden-Budget (S1-NFR5) und dem Vercel-60s-Timeout.

## Schwellenwert-Bewertung

**Sprint 1 Ziel (AC #1): >= 60% Genauigkeit — ERFUELLT mit 75%**

| Szenario | Zuordnung korrekt | Confidence korrekt | Score |
|----------|------------------|-------------------|-------|
| 1. Hochdeutsch, klare Referenz | Ja | Ja (0.90, klar) | 2/2 |
| 2. Dialekt, gleicher Text | Ja | Ja (0.90, klar) | 2/2 |
| 3. Kein Projektbezug | Nein (False Positive) | Grenzwertig (0.70) | 0/2 |
| 4. STT-Fehler im Strassennamen | Ja | Ja (0.90, klar) | 2/2 |
| 5. Schnelle Sprache + STT-Fehler | Nein (False Negative) | Ja (0.30, unsicher) | 1/2 |
| 6. Multi-Projekt | Ja | Ja (0.90, klar) | 2/2 |
| 7. Stark verzerrtes Transkript | Ja | Ja (0.20, korrekt unsicher) | 2/2 |
| 8. STT-Fehler BV statt PV | Ja | Ja (0.90, klar) | 2/2 |

**Gesamt: 13/16 (81%) — Confidence-Kalibrierung ist gut, Fehlzuordnungen sind identifizierbar.**

## Verbesserungspotenzial (Sprint 2+)

1. **Confidence-Schwelle erhoehen (0.75 oder 0.80):** Eliminiert den TC-03 False Positive (Zaehlerkasten-Verwechslung), erhoeht aber Inbox-Volumen. Trade-off: hoehere Praezision vs. mehr manuelle Zuordnung.

2. **Kontinuitaets-Signal testen:** Die Evaluation lief ohne Vorgaenger-Nachrichten (Baseline). In der Produktion stehen die letzten 3 Nachrichten des Sprechers als Kontext zur Verfuegung. Dies sollte insbesondere TC-05 (Gruber) verbessern, da der Sprecher kurz zuvor auf der Gruber-Baustelle gearbeitet hat.

3. **Prompt-Iteration (Fachbegriff vs. Projektname):** Explizite Anweisung hinzufuegen: "Elektro-Fachbegriffe (Zaehlerkasten, FI-Schalter, Unterverteilung) sind KEINE Projektnamen — verwende sie nur als schwaches Signal." Dies koennte TC-03 direkt loesen.

4. **Mehr Testdaten:** Pilot-Feedback (geplant Q4 2026) liefert reale Zuordnungs-Szenarien fuer breitere Evaluierungsbasis.

5. **Multi-Projekt-Erweiterung:** Zweites bekanntes Projekt (z.B. "Altbau Heidenreichstein") in die Testliste aufnehmen, um echte Multi-Projekt-Konflikte zu testen.

6. **Structured Output Mode:** Wenn Zod v4 Kompatibilitaet mit `zod-to-json-schema` geloest ist, von `json_object` auf `json_schema` Mode wechseln fuer strengere JSON-Garantien.

## EU Compliance Assessment

| Kriterium | Bewertung |
|-----------|-----------|
| Anbieter | Mistral AI (Frankreich, EU) |
| Hosting | EU by default |
| DSGVO-konform | Ja (Scale Plan, kein Training mit API-Daten) |
| Datenresidenz | EU |
| FFG-konform | **Ja** |
| Training mit API-Daten | Nein (Scale Plan, siehe ADR-001) |

## Zusammenfassung

Die Assignment Pipeline erreicht im Sprint 1 eine Gesamt-Genauigkeit von **75%** und uebertrifft das Minimalziel von 60% deutlich. Die Confidence-Kalibrierung zeigt, dass das Modell bei hoher Sicherheit (>= 0.8) zu 100% korrekt zuordnet. Die beiden Fehlerfaelle sind identifizierbar und adressierbar:

- **False Positive (TC-03):** Fachbegriff-Verwechslung mit Projektname — loesbar durch Prompt-Anpassung oder hoehere Schwelle
- **False Negative (TC-05):** Kumulation von STT-Fehlern bei schneller Sprache — loesbar durch Kontinuitaets-Signal

Die Pipeline ist mit 912ms Durchschnittslatenz und $0.00007 pro Zuordnung hocheffizient. Die Inbox als Fallback fuer unsichere Zuordnungen funktioniert wie vorgesehen — die Nachricht geht nie verloren, sondern wird bei Unsicherheit zur manuellen Bearbeitung bereitgestellt.

**Empfehlung:** ASSIGNMENT_V1 als Produktions-Prompt beibehalten, Confidence-Schwelle 0.7 fuer den Prototyp akzeptabel. In Sprint 2 Kontinuitaets-Signal und Prompt-Iteration evaluieren.

## Rohdaten

- Evaluations-Script: `scripts/assignment-evaluation.ts`
- Rohergebnisse (JSON): `docs/research/assignment-evaluation-raw-results.json`
- Prompt-Quelle: `lib/ai/prompts.ts` (ASSIGNMENT_SYSTEM_PROMPT, V1)
- Pipeline-Implementierung: `lib/ai/assignment.ts` (assignToProject)
- STT-Transkripte: `docs/research/stt-evaluation-raw-results.json`
- Test-Skripte: `docs/_masemIT/audios/stt-test-skripte.md`
- ADR: `docs/decisions/ADR-006 - Projekt-Zuordnungs-Evaluierung Assignment Pipeline.md`

## Beteiligte

- Prompt-Design & Pipeline-Implementierung: Claude Opus 4.6
- Evaluierungs-Design & Analyse: Claude Opus 4.6
- Test-Transkripte: Aus STT-Evaluation (Sprecher: Mario/Sempre)
- Forschungsfrage: FFG Basisprogramm Kleinprojekt, Antragsnr. 69168884
