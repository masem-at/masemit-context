# Architecture Decision Records — kurzum.app

Technische Entscheidungen werden als ADRs in [`docs/decisions/`](decisions/) dokumentiert.
Eingeführt am 2026-02-11 (Party Mode: Mary, John, Winston, Sally, Paige) — Diskussionsergebnisse gingen zuvor verloren.

## Index

| ADR | Datum | Entscheidung | Status |
|-----|-------|-------------|--------|
| [ADR-001](decisions/ADR-001%20-%20Mistral%20AI%20Scale%20Plan%20statt%20Experiment.md) | 2026-02-13 | Mistral AI Scale Plan statt Experiment | Entschieden |
| [ADR-002](decisions/ADR-002%20-%20Kein%20Cookie%20Consent%20Banner.md) | 2026-02-11 | Kein Cookie Consent Banner | Entschieden |
| [ADR-003](decisions/ADR-003%20-%20Mistral%20SDK%20statt%20REST%20fetch.md) | 2026-02-13 | Mistral SDK statt REST fetch | Entschieden |
| [ADR-004](decisions/ADR-004%20-%20STT%20Provider%20Evaluation%20Voxtral%20vs%20Whisper.md) | 2026-02-13 | STT Provider Evaluation: Voxtral gewinnt 8/12, 2.7x schneller, EU-compliant | Entschieden |
| [ADR-005](decisions/ADR-005%20-%20Prompt%20Engineering%20for%20Handwerker%20LLM%20Summarization.md) | 2026-02-13 | Prompt Engineering: Detailed Prompt v1 gewinnt mit 100% Validität, 96.3% semantische Korrektheit | Entschieden |
| [ADR-006](decisions/ADR-006%20-%20Projekt-Zuordnungs-Evaluierung%20Assignment%20Pipeline.md) | 2026-02-15 | Projekt-Zuordnung: ASSIGNMENT_V1 erreicht 75% Genauigkeit (Sprint 1 Ziel ≥60% erfüllt) | Entschieden |
| [ADR-007](decisions/ADR-007%20-%20Magic%20Link%20Authentication%20statt%20Passwort.md) | 2026-02-15 | Magic Link Authentication via Resend statt Passwort-basierter Auth | Entschieden |
| [ADR-008](decisions/ADR-008%20-%20Synchrone%20Voice%20Pipeline%20statt%20Queue.md) | 2026-02-15 | Synchrone Voice Pipeline (Upload→STT→LLM→Assign in einem Request) statt Message Queue | Entschieden |
| [ADR-009](decisions/ADR-009%20-%20Audio%20Retention%2090%20Tage%20DSGVO.md) | 2026-02-15 | Audio-Dateien 90 Tage Aufbewahrung, dann Auto-Löschung (DSGVO Speicherbegrenzung) | Entschieden |
| [ADR-010](decisions/ADR-010%20-%20Summary-basierte%20Projekt-Zuordnung%20statt%20Rohes%20Transkript.md) | 2026-02-16 | Assignment V2 soll Summary + Transcript statt nur Transcript verwenden, Anti-Halluzinations-Regel | Vorgeschlagen |
