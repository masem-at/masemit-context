# ADR-005: Prompt Engineering for Handwerker LLM Summarization

**Datum:** 2026-02-13
**Status:** Entschieden
**FFG Forschungsfrage #2:** "Wie können unstrukturierte Sprachnachrichten semantisch in strukturierte Zusammenfassungen transformiert werden?"

## Kontext

kurzum.app transformiert Sprachnachrichten von Elektro-Handwerkern in strukturierte JSON-Zusammenfassungen (VoiceSummary). Der System-Prompt muss:
- Handwerker-Domäne verstehen (Elektro-Fachbegriffe, Baustellen-Kontext)
- Österreichische Dialekte korrekt interpretieren
- Konsistent valides JSON produzieren (kein Retry-Mechanismus)
- Deutsche Sprache in den Ausgabewerten verwenden

## Entscheidung

**Detailed Prompt (v1)** als Produktions-Prompt. Gespeichert in `lib/ai/prompts.ts` als `VOICE_SUMMARY_SYSTEM_PROMPT`.

## Evaluierung

### Testsetup
- Model: Mistral Small (mistral-small-latest, 24B, EU-hosted)
- 8 Transkripte aus der STT-Evaluation (Hochdeutsch, Dialekt, Akzent, Lärm)
- 3 Prompt-Varianten: Minimal, Detailed, Few-Shot
- 24 API-Calls total

### Vergleichsmatrix

| Kriterium | Minimal | Detailed | Few-Shot |
|-----------|---------|----------|----------|
| JSON-Validität | 100% | **100%** | 88% |
| Deutsche Sprache | 0% | **100%** | 100% |
| Semantische Korrektheit | Niedrig | **96.3%** | ~90% |
| Materialerkennung | Gut | **Gut** | Teilweise |
| Problemerkennung | 4/4 | **4/4** | 3/3 |
| Avg Latenz | 1844ms | **2106ms** | 1610ms |
| Avg Kosten/Summary | $0.014¢ | **$0.004¢** | $0.037¢ |

### Ausschlussgründe

- **Minimal**: Produziert englische Status-Werte ("in progress") statt deutsche Beschreibungen. Semantisch arm.
- **Few-Shot**: 1 Validierungsfehler bei langem Dialekt-Transkript (88% Zuverlässigkeit). Nicht produktionstauglich.

## Begründung

| Kriterium | Gewicht | Detailed gewinnt weil |
|-----------|---------|----------------------|
| Zuverlässigkeit | Kritisch | 100% JSON-Validität (kein Retry nötig) |
| Semantische Qualität | Hoch | Deutsche Sätze, korrekte Fachbegriffe |
| Dialekt-Handling | Hoch | Alle österreichischen Ausdrücke korrekt |
| Kosten | Niedrig | $0.00004/Summary ist vernachlässigbar |
| Latenz | Mittel | 2.1s akzeptabel im 60s-Budget |

## Konsequenzen

- Detailed Prompt (v1) in `lib/ai/prompts.ts` deployed
- `temperature: 0` für deterministische Ausgabe
- `json_object` Response-Modus (statt `json_schema` wegen Zod v4 Kompatibilität)
- Zod v4 Server-seitige Validierung als Sicherheitsnetz
- **Zukünftig:** Prompt-Iteration basierend auf Pilot-Feedback (Q4 2026)

## Rohdaten

- Evaluations-Script: `scripts/llm-evaluation.ts`
- Rohergebnisse: `docs/research/llm-evaluation-raw-results.json`
- Evaluations-Bericht: `docs/research/llm-summarization-evaluation-2026-02.md`
- Winning Prompt: `lib/ai/prompts.ts`

## Beteiligte

- Prompt-Design & Evaluation: Claude Opus 4.6
- Test-Transkripte: Aus STT-Evaluation (Sprecher: Mario/Sempre)
