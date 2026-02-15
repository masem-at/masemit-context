# ADR-006: Projekt-Zuordnungs-Evaluierung — Assignment Pipeline

**Datum:** 2026-02-15
**Status:** Entschieden
**FFG Forschungsfrage #3:** "Kann kontextuelles Matching zur automatischen Projekt-Zuordnung von Sprachnachrichten genutzt werden?"

## Kontext

kurzum.app ordnet eingehende Sprachnachrichten automatisch aktiven Bauprojekten zu. Die Pipeline:
1. Erhält ein Voxtral-Transkript (mit typischen STT-Fehlern, siehe ADR-004)
2. Lädt aktive Projekte (Name, Kunde, Adresse) + letzte 3 Nachrichten des Sprechers
3. Sendet an Mistral Small mit dem ASSIGNMENT_SYSTEM_PROMPT
4. Erhält: `{ projectId, confidence, reasoning }`
5. Routing: `confidence >= 0.7` → Auto-Zuordnung, `< 0.7` → Inbox (manuelle Zuordnung)

Die Evaluierung misst die Baseline-Genauigkeit des Systems vor dem Pilot-Einsatz.

## Entscheidung

**ASSIGNMENT_V1 Prompt** als Produktions-Prompt. Gespeichert in `lib/ai/prompts.ts` als `ASSIGNMENT_SYSTEM_PROMPT`. Confidence-Schwelle `0.7` für Auto-Zuordnung beibehalten.

## Evaluierung

### Testsetup

- **Model:** Mistral Small (mistral-small-latest, 24B, EU-hosted)
- **Prompt:** ASSIGNMENT_V1 (identisch mit Produktion)
- **Parameter:** `temperature: 0`, `responseFormat: json_object`, `maxTokens: 512`
- **8 Transkripte** aus der STT-Evaluation (ADR-004), reale Voxtral-Ausgabe
- **5 simulierte Test-Projekte** die den Testszenarien entsprechen
- **Keine Vorgänger-Nachrichten** (Baseline ohne Kontinuitäts-Signal)
- **8 Standard-Tests + 2 Edge-Case-Tests** = 10 API-Calls total

### Test-Projekte

| # | Projektname | Kunde | Adresse |
|---|-------------|-------|---------|
| 1 | Neubau Müller | Familie Müller | Hauptstraße 5 |
| 2 | Sanierung Huber | Huber | Gartenstraße 12 |
| 3 | Zählerkasten Gruber | Gruber | — |
| 4 | Neubau Schrems | Familie Novak | Schrems |
| 5 | PV-Anlage Schuster | Schuster | — |

### Ergebnis-Matrix

| TC | Transkript | Variante | Erwartetes Projekt | KI-Zuordnung | Conf. | Korrekt | Auto |
|----|-----------|----------|-------------------|-------------|-------|---------|------|
| 01 | Baustellenbericht | Hochdeutsch | Neubau Müller | Neubau Müller | 0.90 | **JA** | JA |
| 02 | Baustellenbericht | Waldviertlerisch | Neubau Müller | Neubau Müller | 0.90 | **JA** | JA |
| 03 | Materialbestellung | Hochdeutsch | Inbox (null) | Zählerkasten Gruber | 0.70 | **NEIN** | JA |
| 04 | Problem melden | Dialekt | Sanierung Huber | Sanierung Huber | 0.90 | **JA** | JA |
| 05 | Status-Update | Auto/schnell | Zählerkasten Gruber | Inbox (null) | 0.30 | **NEIN** | NEIN |
| 06 | Tagesbericht | Dialekt | Neubau Schrems | Neubau Schrems | 0.90 | **JA** | JA |
| 07 | Notfall | Dialekt + Lärm | Inbox (null) | Inbox (null) | 0.20 | **JA** | NEIN |
| 08 | Nachkalkulation | Normal | PV-Anlage Schuster | PV-Anlage Schuster | 0.90 | **JA** | JA |

### Gesamt-Genauigkeit

| Metrik | Wert | Ziel |
|--------|------|------|
| **Gesamt-Genauigkeit** | **75% (6/8)** | ≥ 60% (Sprint 1) |
| Korrekt auto-zugeordnet | 5 | — |
| Falsch auto-zugeordnet | 1 | — |
| Korrekt Inbox | 1 | — |
| Falsch Inbox | 1 | — |
| Avg Confidence | 0.71 | — |
| Avg Latenz | 912ms | < 5s (S1-NFR5) |
| Avg Kosten/Zuordnung | $0.00007 | — |

### Confidence-Kalibrierung

| Confidence-Band | Korrekt | Total | Genauigkeit |
|----------------|---------|-------|-------------|
| ≥ 0.8 (Klar) | 5 | 5 | **100%** |
| 0.5 – 0.8 (Wahrscheinlich) | 0 | 1 | 0% |
| < 0.5 (Unsicher) | 1 | 2 | 50% |

**Interpretation:** Die Confidence-Kalibrierung zeigt, dass das Modell bei hoher Confidence (≥ 0.8) zuverlässig ist (100%). Die Schwelle 0.7 ist jedoch problematisch — TC-03 wurde mit genau 0.70 falsch auto-zugeordnet. Eine höhere Schwelle (0.75 oder 0.80) würde diesen Fehler vermeiden.

### Edge-Case-Analyse

**TC-03 — Materialbestellung → Falsche Zuordnung (False Positive):**
Das Transkript enthält "PE-Schiene für den Zählerkasten" — das Wort "Zählerkasten" erscheint auch im Projektnamen "Zählerkasten Gruber". Das LLM interpretiert diesen Fachbegriff fälschlich als Projektbezug. *Erkenntnis: Projektname sollte KEINE Fachbegriffe enthalten, oder der Prompt muss expliziter zwischen Fachbegriff und Projektname unterscheiden.*

**TC-05 — Status-Update Gruber → Nicht erkannt (False Negative):**
Voxtral transkribiert "Gruber Baustell" (abgeschnittenes Wort) und "Zollerkosten" (statt "Zählerkasten"). Trotz "Gruber" im Transkript erkennt das LLM die Zuordnung nicht und nennt "Linzer Straße" als unbekannte Adresse. *Erkenntnis: STT-Fehler in Kombination mit fehlender Adresse im Projekt führen zu Unsicherheit. Kontinuitäts-Signal (Vorgänger-Nachrichten) könnte helfen.*

**TC-06 — Tagesbericht mit 2 Projekten:**
Transkript erwähnt "Neubau in Schrems, Familie Nowak" UND "Heidenreichstein zum Altbau von der Frau Winkler". Das LLM wählt korrekt das erste/explizitere Projekt (Schrems/Novak). Heidenreichstein/Winkler existiert nicht als Projekt in der Liste, was die Entscheidung vereinfacht. *Erkenntnis: Bei echtem Multi-Projekt-Szenario (beide in der Liste) würde die Confidence wahrscheinlich sinken.*

**TC-07 — Stark verzerrtes Transkript:**
Voxtral-Ausgabe ist nahezu unlesbar ("herkommen zu der Hure", "Thementaste von der Teilerkosten"). Das LLM erkennt korrekt, dass kein Projektbezug herstellbar ist und gibt `null` mit Confidence 0.20 zurück. *Erkenntnis: Das System degradiert graceful bei schlechter STT-Qualität.*

**Edge Case: Nur 1 Projekt in der Liste:**
- TC-01 mit nur Müller-Projekt: Korrekte Zuordnung (0.90) — kein Bias durch Single-Option
- TC-03 mit nur Müller-Projekt: Korrekt Inbox (0.30) — das LLM ordnet nicht blind dem einzigen Projekt zu

### Prompt-Version

```
Version: ASSIGNMENT_V1
Datei: lib/ai/prompts.ts
Konstante: ASSIGNMENT_SYSTEM_PROMPT
Confidence-Schwelle: 0.7 (Routing in app/api/voice/process/route.ts)
```

## Begründung

| Kriterium | Bewertung |
|-----------|-----------|
| Sprint 1 Ziel (≥ 60%) | **75% erreicht — ERFÜLLT** |
| FFG Endziel (≥ 75%) | 75% — auf Kante, Verbesserung nötig |
| Latenz | 912ms avg — weit unter 5s Budget |
| Kosten | $0.00007/Zuordnung — vernachlässigbar |
| Confidence-Kalibrierung | Gut bei hoher Confidence (100%), Schwelle 0.7 grenzwertig |
| Graceful Degradation | Stark verzerrte Transkripte → korrekt Inbox |
| Multi-Projekt | Erster/explizitester Bezug wird gewählt |

## Konsequenzen

### Sofort (Sprint 1)
- ASSIGNMENT_V1 Prompt bleibt als Produktions-Prompt
- Confidence-Schwelle 0.7 beibehalten (1 Fehlzuordnung akzeptabel für Prototyp)
- Fehlzuordnungen landen im Inbox und können manuell korrigiert werden

### Verbesserungspotenzial (Sprint 2+)
1. **Confidence-Schwelle auf 0.75 oder 0.80 erhöhen** — eliminiert TC-03 False Positive, erhöht Inbox-Volumen
2. **Kontinuitäts-Signal testen** — Vorgänger-Nachrichten als Kontext sollte TC-05 (Gruber) verbessern
3. **Prompt-Iteration** — Explizite Anweisung: "Fachbegriffe (Zählerkasten, FI-Schalter) sind KEINE Projektnamen"
4. **Mehr Testdaten** — Pilot-Feedback (Q4 2026) für breitere Evaluierungsbasis
5. **Altbau Heidenreichstein als 6. Projekt** — Echtes Multi-Projekt-Szenario testen

## Rohdaten

- **Evaluations-Script:** `scripts/assignment-evaluation.ts`
- **Rohergebnisse:** `docs/research/assignment-evaluation-raw-results.json`
- **Prompt-Quelle:** `lib/ai/prompts.ts` (ASSIGNMENT_SYSTEM_PROMPT)
- **Pipeline-Implementierung:** `lib/ai/assignment.ts` (assignToProject)
- **STT-Transkripte:** `docs/research/stt-evaluation-raw-results.json`
- **Test-Skripte:** `docs/_masemIT/audios/stt-test-skripte.md`

## Beteiligte

- Prompt-Design & Pipeline: Claude Opus 4.6
- Evaluierungs-Design & Analyse: Claude Opus 4.6 (Party Mode: Mary, Winston, Paige)
- Test-Transkripte: Aus STT-Evaluation (Sprecher: Mario/Sempre)
- Forschungsfrage: FFG Basisprogramm Kleinprojekt, Antragsnr. 69168884
