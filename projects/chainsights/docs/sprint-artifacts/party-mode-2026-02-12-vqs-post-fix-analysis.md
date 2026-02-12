# Party Mode Session — VQS Post-Fix Analysis & Missing DAOs Investigation

**Datum:** 2026-02-12
**Status:** ABGESCHLOSSEN
**Agents:** Mary (Analyst), Winston (Architect), Paige (Tech Writer)
**Initiiert von:** Sempre
**Artefakte:**
- `docs/_masemIT/bugs-tickets-issues/ticket-vqs-ranked-choice-fix.md`
- `docs/_masemIT/bugs-tickets-issues/ticket-vqs-missing-daos.md`

---

## Kontext

Ranked-Choice/Weighted-Voting Fix (aus ticket-vqs-ranked-choice-fix.md) wurde implementiert und deployed. Daily Job wurde getriggert. Team analysierte die Post-Fix-Ergebnisse und untersuchte warum nur 24 von 47 DAOs VQS-Daten haben.

---

## Teil 1: Post-Fix Benchmark (Mary)

### Globale VQS-Statistiken (24 DAOs, 8.601 Voters)

| Metrik | Pre-Fix | Post-Fix | Delta |
|---|---|---|---|
| Qualifying Voters | 4.303 | 8.601 | +100% (neue Berechnung) |
| Median VQS | 3.70 | 3.69 | -0.01 (stabil) |
| Mean VQS | 4.30 | 4.13 | -0.17 |
| Max VQS | 10.00 | 10.00 | unchanged |
| StdDev | — | 1.86 | — |

### Null-Signal-Verteilung (Post-Fix)

| Signal | Null Count | % of Total | Ursache |
|---|---|---|---|
| Independence | 1.364 | 15.9% | Ranked-Choice/Weighted Voting |
| Originality | 419 | 4.9% | Keine Pattern-Correlation möglich |

### Betroffene DAOs (5 mit Ranked-Choice/Weighted Voting)

| DAO | Avg VQS Post-Fix | Voting Type | Null Signals |
|---|---|---|---|
| Frax | 9.78 | Weighted/Gauge | Independence + Originality |
| Aavegotchi | 7.65 | Ranked-Choice | Independence + Originality |
| CVX | 6.99 | Mixed | Teilweise Independence |
| Balancer | 6.19 | Weighted | Independence + Originality |
| StakeDAO | 5.44 | Mixed | Teilweise Independence |

**Anmerkung:** VQS bleibt hoch bei diesen DAOs weil das Gewicht auf Deliberation + Focus umverteilt wird. Das ist by design — diese Voter zeigen tatsächlich gutes Deliberation-Verhalten.

---

## Teil 2: Missing DAOs Investigation (Winston)

### Ergebnis: Kein Bug — Designentscheidung

Von 47 DAOs haben nur 24 VQS-Daten. Root Cause in `src/lib/gvs/gpi.ts`:

### Grund 1: Keine Proposals innerhalb 6-Monate-Lookback (~18 DAOs)

`gpi.ts:156-162` filtert nach 6-Monate-Lookback. Viele DAOs nutzen **On-Chain Governance** (Tally, Governor) statt Snapshot — deshalb keine recenten Snapshot-Proposals.

Beispiele (via Snapshot GraphQL API verifiziert):
- Optimism: letztes Proposal 2023-01-12 (3 Jahre alt, nutzt Governor)
- Curve: 2022-04-29 (4 Jahre, on-chain Voting)
- dYdX: 2024-02-28 (eigene Chain)
- Safe: 2025-07-22 (6.7 Monate, knapp außerhalb)
- ApeCoin: 2025-06-13 (8 Monate, inaktiv auf Snapshot)

### Grund 2: Zu wenig Votes pro Voter (~5 DAOs)

`checkbox-detection.ts:108` erfordert `MIN_VOTES_FOR_ANALYSIS = 3`. DAOs wie Bancor (2 Proposals), Wormhole (2 Proposals), TallyDAO (1 Proposal) haben zu wenig Daten.

### Vollständige Liste: siehe `ticket-vqs-missing-daos.md`

---

## Teil 3: Entscheidungen

### Entscheidung 1: Ranked-Choice Fix — DEPLOYED ✅
- Option A implementiert: null statt inflated 10.0 für Independence/Originality
- Gewichts-Redistribution auf verfügbare Signals
- Commits: `24b1c88`, `3d17b84`

### Entscheidung 2: Missing DAOs — DOKUMENTIEREN, kein Fix
- Kein Bug, by design
- Priorität: Low
- Kurzfristig: Methodology Page dokumentiert die 51% Abdeckung (24/47 DAOs)

### Entscheidung 3: Consensus Penalty — DEFERRED
- DAOs wie Uniswap (Median 3.3) und ENS (Median 2.4) scoren niedrig weil Governance uncontroversial ist
- Entscheidung verschoben bis Post-Fix-Daten vorliegen (jetzt verfügbar)
- Optionen: (A) Kontextualisieren, (B) Gewichtsanpassung, (C) Nur dokumentieren

### Entscheidung 4: Story 9-5 Methodology Page v1.3 — PAIGE
Paige übernimmt folgende Inhalte:

1. **VQS-Sektion**: 4 Signals erklären (Deliberation, Independence, Focus, Originality)
2. **N/A-Erklärung**: Warum Ranked-Choice DAOs null für Independence/Originality zeigen
3. **Gewichts-Redistribution**: Wie sich Gewicht auf verfügbare Signals verteilt
4. **51% Abdeckung**: 24/47 DAOs — transparent warum (6-Monate-Lookback, On-Chain Governance)
5. **DAO-Archetypes**: Consensus ("Rubber Stamp"), Mixed, Pluralistic
6. **Global Benchmarks**: Median 3.69, Mean 4.13, Top-Quartil ≥5.0
7. **Consensus Penalty**: Als bekanntes Phänomen dokumentieren, kein Scoring-Change

---

## Action Items

| Prio | Item | Owner | Status |
|---|---|---|---|
| 1 | Story 9-5 Methodology Page v1.3 | Paige | Ready to start |
| 2 | Consensus Penalty Entscheidung treffen | Team | Deferred — Daten verfügbar |
| 3 | Lookback auf 12 Monate erweitern (optional) | Dev | Backlog |
| 3 | Tally/Governor Integration (optional, eigenes Epic) | Winston | Backlog |

---

## Codepfad-Referenz

```
daily-recalculation.ts          → enqueueAllGVSCalculations()
  ↓ QStash
gvs-calculate/route.ts          → calculateGVS(spaceId)
  ↓
aggregate.ts                     → calculateGPI(spaceId) [parallel mit HPR, DEI, PDI]
  ↓
gpi.ts:153                       → fetchProposals(spaceId, 100, 'closed', {lite: true})
gpi.ts:157                       → filter by 6-month lookback
gpi.ts:160-162                   → if (recentProposals.length === 0) return neutral ← EXIT, no VQS
  ↓ (if proposals exist)
gpi.ts:166-169                   → fetchVotesForProposal() for each proposal
gpi.ts:207                       → calculateCheckboxSignals(voteRecords)
gpi.ts:210                       → if (checkboxMap.size > 0) { ... Phase 2 VQS ... }
gpi.ts:301-303                   → if (checkboxMap.size > 0) saveCheckboxSignals() ← VQS SAVE
```

---

## Teil 4: Session 2 — Methodology Page v1.3 Implementierung

**Agents:** Paige (Tech Writer), Winston (Architect), Mary (Analyst)
**Auslöser:** Sempre aktivierte Paige für Story 9-5

### Durchgeführte Änderungen

**Datei:** `src/app/rankings/methodology/page.tsx`

| Änderung | Detail |
|---|---|
| Version | `1.2` → `1.3` |
| `dateModified` (JSON-LD) | `2025-12-23` → `2026-02-12` |
| Neue Sektion | VQS: 4 Signals, N/A-Handling, Coverage, Benchmarks |
| Update Frequency | "every 7 days" → "daily" (korrigiert nach QStash-Refactor) |
| Change Log | v1.3 Eintrag hinzugefügt, v1.2 separiert |
| Icons | `Eye` + `Info` importiert |

### VQS-Sektion Inhalte (Paige)

1. **Einleitung**: VQS als Delegate-Level Score (0-10), Abgrenzung zu GVS
2. **4 Signals**: Deliberation, Independence, Focus, Originality — jeweils mit verständlicher Erklärung
3. **Composite-Berechnung**: 25% pro Signal, Redistribution bei N/A
4. **Ranked-Choice Box** (gelb): Warum Independence + Originality bei manchen DAOs N/A sind
5. **Coverage**: 24/47 DAOs, Gründe (Snapshot-only, 6-Monate-Lookback)
6. **Benchmarks**: Median 3.7, Mean 4.1, Top-Quartil ≥5.0, 8.600+ Voters

### Nicht umgesetzt (bewusst weggelassen)

- **DAO-Archetypes** (Consensus/Mixed/Pluralistic): Verschoben — hängt an Consensus Penalty Entscheidung
- **Consensus Penalty Erklärung**: Noch deferred, keine Dokumentation ohne Entscheidung

### Validierung

- TypeScript Build: OK (keine Fehler in methodology page)
- Methodology Tests: 5/5 bestanden
- VQS Tests: 61/61 bestanden
- Commit: `b5c5764`

### Offene Punkte

| Item | Status |
|---|---|
| Consensus Penalty Entscheidung | Deferred — Post-Fix-Daten verfügbar, Team muss entscheiden |
| DAO-Archetypes auf Methodology Page | Wartet auf Consensus Penalty Entscheidung |
| Vercel Deploy verifizieren | Sempre prüft nach Build |

---

*Session 1 nachträglich protokolliert — Lektion gelernt: IMMER vor Party-Exit speichern.*
*Session 2 protokolliert VOR Party-Exit. ✓*
