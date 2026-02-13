# ADR-003: Mistral SDK statt REST fetch

**Datum:** 2026-02-13
**Status:** Entschieden

**Kontext:** Integration der Mistral Voxtral STT API für Spracherkennung (Story 6.2).
             Zwei Optionen: offizielles `@mistralai/mistralai` SDK oder direkter REST fetch.

**Entscheidung:** Offizielles Mistral SDK (`@mistralai/mistralai` v1.14.0) verwenden.

**Begründung:**
- TypeScript-Typen out-of-the-box — kein manuelles Typing der API-Response
- SDK-maintained API-Kompatibilität — Breaking Changes werden vom SDK-Team abgefangen
- Keine manuelle Serialisierung für Datei-Upload (multipart/form-data) nötig
- `contextBias`-Parameter direkt im SDK typisiert

**Alternativen betrachtet:**
- Direkter REST fetch → simpler, keine Dependency, aber: manuelle FormData-Konstruktion für Audio-Upload, kein Typing, API-Änderungen erfordern manuelles Update
- Vercel AI SDK → unterstützt Mistral für Chat/Completion, aber NICHT für Audio Transcription

**Konsequenzen:**
- Neue Dependency: `@mistralai/mistralai` (server-side only)
- SDK wird nur in `lib/ai/mistral.ts` importiert — isolierter Blast Radius
- Bei Mistral API-Änderungen: SDK-Update statt Code-Anpassung

**Kontext:** Story 6.2 — Mistral Voxtral STT Integration & Evaluation
