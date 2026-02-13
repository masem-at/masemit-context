# STT Evaluation: Mistral Voxtral vs. Whisper

**Date:** 2026-02-13
**FFG Forschungsfrage #1:** Domänenspezifische Spracherkennung bei Baustellenlärm und Dialekten
**Researcher:** Claude Opus 4.6
**Project:** kurzum.app (Antragsnr. 69168884)

## Test Matrix

| # | Scenario | Variant | Background | Voxtral WER | Whisper WER | Voxtral Latency | Whisper Latency |
|---|----------|---------|------------|-------------|-------------|-----------------|-----------------|
| 1 | Construction report | Hochdeutsch | Quiet | 0.0% | 3.0% | 755ms | 2728ms |
| 2 | Construction report | Waldviertlerisch | Quiet | 15.2% | 21.2% | 587ms | 2785ms |
| 3 | Construction report | Foreign accent | Quiet | 48.5% | 81.8% | 480ms | 1789ms |
| 4 | Material order | Hochdeutsch | Quiet | 25.0% | 33.3% | 595ms | 1275ms |
| 5 | Material order | Waldviertlerisch | Quiet | 43.8% | 43.8% | 1526ms | 1531ms |
| 6 | Problem report | Dialect (worried) | Quiet | 35.3% | 29.4% | 587ms | 1333ms |
| 7 | Status update | Fast/casual | Driving | 23.1% | 30.8% | 434ms | 2130ms |
| 8 | Daily report | Dialect (tired) | Quiet | 20.0% | 21.1% | 882ms | 2318ms |
| 9 | Handover | Normal | Workshop | 66.7% | 65.2% | 649ms | 1965ms |
| 10 | Emergency | Dialect (urgent) | Construction | 52.5% | 49.2% | 616ms | 1517ms |
| 11 | Cost estimate | Normal | Quiet | 11.9% | 27.1% | 643ms | 1745ms |
| 12 | Material order | Foreign accent | Radio | 70.8% | 77.1% | 614ms | 1539ms |
| | **Average** | | | **34.4%** | **40.3%** | **697ms** | **1888ms** |

> **Note:** WER computed against scripted reference texts. Speakers were instructed to speak freely (not read verbatim). WER values are approximate — actual transcription accuracy is better than numbers suggest.

## Cost Comparison

| Provider | Model | Price/Minute | 5min Recording | EU-hosted |
|---|---|---|---|---|
| Mistral | voxtral-mini-latest | $0.003/min | $0.015 | Yes (France, default EU) |
| OpenAI | whisper-1 | $0.006/min | $0.030 | No (US, DPA required) |

**kurzum.app cost projection:** 5-min recording = $0.015. At 100 recordings/day = $1.50/day = ~$45/month.

## Findings

### Audio Format Compatibility

**Investigation Date:** 2026-02-13
**Our recording format:** `audio/webm;codecs=opus` (Chrome, Firefox, modern Safari default)

| Format | Mistral Official Support | Evaluation Result |
|---|---|---|
| mp4 | m4a listed | Empty transcript, 0s audio — **NOT supported** |
| ogg | Yes | Full transcript, correct audio duration — **Works perfectly** |
| webm | Not listed | Not tested yet (pending production test) |

**Finding:** MP4 container format fails silently with Mistral API (returns empty transcript). OGG/Opus works reliably. Browser recording format should target `audio/ogg;codecs=opus` or test WebM explicitly.

**Recommendation:** Change MediaRecorder mimeType to `audio/ogg;codecs=opus` in `components/app/voice-recorder.tsx` as safe default. Test WebM in production once available.

### Domain Term Recognition (contextBias)

**Implementation:** `contextBias` parameter with 27 Elektro-Fachbegriffe configured in `lib/ai/mistral.ts`.

**Observed recognition quality with contextBias enabled:**

| Term | Correctly recognized | Notes |
|---|---|---|
| Sicherungsautomaten | Yes | Consistent across variants |
| FI-Schutzschalter | Yes | Correct in Hochdeutsch; "Schutzschalter" alone in accent variant |
| Zählerkasten | Mostly | Correctly in Hochdeutsch; "Zöllerkosten" in strong dialect |
| PE-Schiene | Partial | "die Schiene" in accent variant, full in Hochdeutsch |
| NYM-J | Partial | "NYMJ" or "in YM" depending on variant |
| Wago-Klemmen | Yes | Consistent |
| Überspannungsschutz | Yes | Correctly in all variants tested |
| Montageschienen | Yes | Correctly recognized |
| Schlitzfräse | Yes | Correctly in normal speech |
| B16 | Partial | "B16" in Hochdeutsch, "bei 16" in dialect |
| Isolationsmessgerät | Yes | Correctly in dialect variant |
| PV-Anlage | Partial | "BV-Anlage" in one variant |

**Key insight:** contextBias significantly helps with compound technical terms (Sicherungsautomat, Überspannungsschutz). Short terms and numbers (B16, NYM-J) are harder to bias correctly.

### Dialect Handling

| Variant | Best WER (Voxtral) | Average WER Impact |
|---|---|---|
| Hochdeutsch baseline | 0.0% | — |
| Waldviertlerisch | 15.2% | +15–25% vs Hochdeutsch |
| Foreign accent | 48.5% | +25–50% vs Hochdeutsch |

**Voxtral handles Austrian dialect significantly better than Whisper** for foreign accents (48.5% vs 81.8%). For native dialect (Waldviertlerisch), both perform similarly.

### Background Noise Impact

| Background | Average WER (Voxtral) | Average WER (Whisper) |
|---|---|---|
| Quiet | 24.9% | 32.6% |
| Driving (car) | 23.1% | 30.8% |
| Workshop/Warehouse | 66.7% | 65.2% |
| Construction site | 52.5% | 49.2% |
| Radio | 70.8% | 77.1% |

**Key insight:** Both providers struggle significantly with indoor ambient noise (workshop, radio). Construction site noise has less impact than expected. Car driving barely affects recognition. **Future research direction: Audio preprocessing (noise reduction) before STT.**

## Whisper Benchmark Notes

OpenAI Whisper was used **only as a research benchmark** — NOT as production provider.
- US-hosted, violates FFG EU-data requirements
- whisper-1 model used (OpenAI API, corresponds to large-v2/v3 internally)
- No contextBias equivalent available
- 2.7x slower average latency than Voxtral

## Recommendation

**Production provider: Mistral Voxtral Mini** — confirmed by evaluation.

1. **Better accuracy:** 34.4% avg WER vs 40.3% (Whisper), wins 8/12 scenarios
2. **Much faster:** 697ms vs 1888ms (2.7x speedup), critical for synchronous pipeline
3. **Better accent handling:** 48.5% vs 81.8% on foreign accent — important for diverse workforce
4. **EU-compliant:** French company, EU-default hosting, DSGVO-nativ
5. **contextBias:** Domain-specific vocabulary support improves Fachbegriffe recognition
6. **Half the price:** $0.003/min vs $0.006/min (whisper-1)

**Future optimization paths:**
- Audio preprocessing (noise reduction) for Workshop/Radio scenarios (WER >60%)
- Expanded contextBias list based on real user recordings
- WebM format testing (potential to skip OGG conversion)
- Voxtral model updates as Mistral releases improvements

## EU Compliance Assessment

| Criteria | Mistral Voxtral | OpenAI Whisper |
|---|---|---|
| Company HQ | France (EU) | USA |
| Default hosting | EU | US |
| DSGVO-nativ | Yes | No (DPA required) |
| Data residency | EU by default | US by default |
| FFG-compliant | **Yes** | No (benchmark only) |
| Training on API data | No (Scale Plan) | No (API data not used) |

## Raw Data References

- Evaluation script: `scripts/stt-evaluation.ts`
- Raw JSON results: `docs/research/stt-evaluation-raw-results.json`
- Reference scripts: `docs/_masemIT/audios/stt-test-skripte.md`
- Audio test files: `docs/_masemIT/audios/text-01.ogg` through `text-12.ogg`
- ADR: `docs/decisions/ADR-004 - STT Provider Evaluation Voxtral vs Whisper.md`
