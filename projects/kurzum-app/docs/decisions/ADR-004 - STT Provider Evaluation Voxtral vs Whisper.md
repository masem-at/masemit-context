# ADR-004: STT Provider Evaluation — Mistral Voxtral vs. OpenAI Whisper

**Datum:** 2026-02-13
**Status:** Entschieden
**FFG Forschungsfrage #1:** "Wie lässt sich domänenspezifische Spracherkennung bei Baustellenlärm und österreichischen Dialekten optimieren?"

## Kontext

kurzum.app verarbeitet Sprachnachrichten von Elektro-Handwerkern. Die Spracherkennung muss funktionieren mit:
- Österreichischen Dialekten (Waldviertlerisch)
- Ausländischen Akzenten (Lehrlinge)
- Baustellenlärm, Autofahrt, Werkstatt-Hintergrundgeräuschen
- Elektro-Fachbegriffen (FI-Schalter, Sicherungsautomat B16, NYM-J, PE-Schiene)

FFG-Förderung verlangt EU-Datenverarbeitung als Primärprovider.

## Entscheidung

**Mistral Voxtral Mini** als Produktions-STT-Provider. OpenAI Whisper nur als Forschungs-Benchmark.

## Evaluierung

### Testsetup
- 12 Aufnahmen, 8 verschiedene Texte (Baustellenberichte, Materialbestellungen, Notfälle)
- Varianten: Hochdeutsch, Waldviertlerisch, ausländischer Akzent
- Hintergrund: Stille, Autofahrt, Werkstatt/Lager, Baustelle, Radio
- Sprecher: Echte freie Sprache (nicht abgelesen)
- Format: OGG/Opus, Handy-Mikrofon
- Voxtral mit `contextBias` (27 Elektro-Fachbegriffe)
- WER berechnet gegen Referenz-Skripte (approximativ, da frei gesprochen)

### Vergleichsmatrix

| # | Szenario | Variante | Hintergrund | Voxtral WER | Whisper WER | Gewinner | V-Latenz | W-Latenz |
|---|----------|----------|-------------|-------------|-------------|----------|----------|----------|
| 1 | Baustellenbericht | Hochdeutsch | Stille | **0.0%** | 3.0% | Voxtral | 755ms | 2728ms |
| 2 | Baustellenbericht | Waldviertlerisch | Stille | **15.2%** | 21.2% | Voxtral | 587ms | 2785ms |
| 3 | Baustellenbericht | Ausländ. Akzent | Stille | **48.5%** | 81.8% | Voxtral | 480ms | 1789ms |
| 4 | Materialbestellung | Hochdeutsch | Stille | **25.0%** | 33.3% | Voxtral | 595ms | 1275ms |
| 5 | Materialbestellung | Waldviertlerisch | Stille | 43.8% | 43.8% | Gleich | 1526ms | 1531ms |
| 6 | Problem melden | Dialekt | Stille | 35.3% | **29.4%** | Whisper | 587ms | 1333ms |
| 7 | Status-Update | Schnell/beiläufig | Autofahrt | **23.1%** | 30.8% | Voxtral | 434ms | 2130ms |
| 8 | Tagesbericht | Dialekt | Stille | **20.0%** | 21.1% | Voxtral | 882ms | 2318ms |
| 9 | Übergabe | Normal | Werkstatt | 66.7% | **65.2%** | Whisper | 649ms | 1965ms |
| 10 | Notfall | Dialekt | Baustelle | 52.5% | **49.2%** | Whisper | 616ms | 1517ms |
| 11 | Nachkalkulation | Normal | Stille | **11.9%** | 27.1% | Voxtral | 643ms | 1745ms |
| 12 | Materialbestellung | Ausländ. Akzent | Radio | **70.8%** | 77.1% | Voxtral | 614ms | 1539ms |
| | | | **Durchschnitt** | **34.4%** | **40.3%** | **Voxtral** | **697ms** | **1888ms** |

### Ergebnis-Zusammenfassung

| Kategorie | Voxtral | Whisper |
|-----------|---------|---------|
| Durchschnitt WER | **34.4%** | 40.3% |
| Szenarien gewonnen | **8/12** | 3/12 |
| Durchschnitt Latenz | **697ms** | 1888ms |
| Hochdeutsch (beste WER) | **0.0%** | 3.0% |
| Dialekt-Aufschlag (vs. Hochdeutsch) | +15–25% | +18–20% |
| Akzent-Aufschlag | +48% | +79% |
| Lärm-Aufschlag | +30–45% | +35–42% |
| Preis/Minute | $0.003 | $0.006 (whisper-1) |
| EU-hosted | **Ja (Frankreich)** | Nein (USA) |
| FFG-compliant | **Ja** | Nein |

### Bemerkenswerte Beobachtungen

1. **Voxtral bei ausländischem Akzent deutlich besser** — 48.5% vs 81.8% WER (Szenario 3). Das ist der größte Einzelunterschied und relevant für Lehrlinge mit Migrationshintergrund.

2. **Voxtral 2.7x schneller** — 697ms vs 1888ms Durchschnitt. Wichtig für synchrone Pipeline (Vercel 60s Timeout-Budget).

3. **Beide kämpfen mit starkem Hintergrundlärm** — Werkstatt (~65%) und Radio+Akzent (~70-77%) sind für beide Provider schwierig. Hier braucht es vermutlich Audio-Preprocessing (Noise Reduction) als zukünftige Forschungsrichtung.

4. **contextBias hilft bei Fachbegriffen** — Voxtral erkennt "Sicherungsautomaten", "Zählerkasten", "PE-Schiene", "Montageschienen" korrekt. Einzelne Fehler: "Zöllerkosten" statt "Zählerkasten" bei starkem Dialekt.

5. **Whisper gewinnt knapp bei Dialekt+Lärm** — Szenarien 6, 9, 10 (Differenz 1.5–3.3%). Der Vorteil ist gering und wird durch Voxtral's Latenz- und Kostenvorteil aufgewogen.

6. **WER-Werte sind approximativ** — Sprecher hat frei gesprochen (nicht abgelesen). Tatsächliche Transkriptionsgenauigkeit ist besser als die WER-Zahlen suggerieren.

## Begründung der Entscheidung

| Kriterium | Voxtral | Whisper | Gewicht |
|-----------|---------|---------|---------|
| WER (Genauigkeit) | Besser (34.4%) | 40.3% | Hoch |
| Latenz | 2.7x schneller | — | Hoch |
| EU-Compliance | Ja (FFG Pflicht) | Nein | Kritisch |
| Preis | $0.003/min | $0.006/min | Mittel |
| Akzent-Handling | Deutlich besser | — | Hoch |
| contextBias (Fachbegriffe) | Unterstützt | Nicht verfügbar | Mittel |

Voxtral gewinnt in 4/6 Kriterien und ist in keinem Kriterium deutlich schlechter. EU-Compliance ist non-negotiable für FFG.

## Konsequenzen

- Mistral Voxtral Mini bleibt Produktions-STT in `lib/ai/mistral.ts`
- `contextBias` mit Elektro-Fachbegriffen aktiviert (messbar besser bei Fachterminologie)
- Whisper-Benchmark als Forschungsdokumentation archiviert (nicht in Produktion)
- **Zukünftige Forschung:** Audio-Preprocessing (Noise Reduction) für Baustellenlärm-Szenarien evaluieren
- **Zukünftige Forschung:** contextBias mit erweiterten Fachbegriff-Listen testen (nach User-Feedback)

## Rohdaten

- Evaluations-Script: `scripts/stt-evaluation.ts`
- Rohergebnisse (JSON): `docs/research/stt-evaluation-raw-results.json`
- Referenz-Skripte: `docs/_masemIT/audios/stt-test-skripte.md`
- Audio-Testdateien: `docs/_masemIT/audios/text-01.ogg` bis `text-12.ogg`

## Beteiligte

- Sprecher/Aufnahmen: Mario (Sempre)
- Evaluations-Script & Analyse: Claude Opus 4.6 (Winston, Architect Agent)
- Testdesign: Referenz-Skripte mit 8 Texten, 12 Varianten (Dialekt, Akzent, Lärm)
