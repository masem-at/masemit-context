# Ticket: VQS Pipeline — 23 DAOs ohne Vote Quality Scores

**Project:** ChainSights
**Priority:** Low — By Design, nicht Bug
**Type:** ~~Investigation + Fix~~ → Dokumentation + optionale Verbesserung
**Owner:** Winston (Architecture)
**Reviewer:** Mary (Validation)
**Created:** 2026-02-12
**Updated:** 2026-02-12 (Root Cause gefunden — kein Bug)
**Status:** Investigation abgeschlossen

---

## Problem

Von 47 DAOs in der `daos`-Tabelle haben nur 24 VQS-Daten in `voter_quality_signals`. 23 DAOs werden übersprungen.

## Root Cause (verifiziert 2026-02-12)

**Kein Bug — Designentscheidung.** Zwei Gründe, beide in `src/lib/gvs/gpi.ts`:

### Grund 1: Keine Proposals innerhalb 6-Monate-Lookback (~18 DAOs)

`gpi.ts:156-162` filtert Proposals nach 6-Monate-Lookback:

```
const cutoffTimestamp = Date.now() / 1000 - (lookbackDays * 24 * 60 * 60)
const recentProposals = allProposals.filter(p => p.created >= cutoffTimestamp)
if (recentProposals.length === 0) {
  return createNeutralGPIResult(...)  // ← VQS wird nie berechnet
}
```

Diese DAOs haben ihr letztes Snapshot-Proposal **vor mehr als 6 Monaten** — verifiziert via Snapshot GraphQL API:

| DAO | Letztes Proposal | Alter | Grund |
|---|---|---|---|
| Optimism | 2023-01-12 | 3 Jahre | Nutzt Governor on-chain |
| Curve | 2022-04-29 | 4 Jahre | Nutzt on-chain Voting |
| dYdX | 2024-02-28 | 2 Jahre | Migrated zu eigener Chain |
| ApeCoin | 2025-06-13 | 8 Monate | Inaktiv auf Snapshot |
| Safe | 2025-07-22 | 6.7 Monate | Knapp außerhalb Fenster |
| Decentraland | — | — | Kein Snapshot-Voting |
| Connext | — | — | Kein Snapshot-Voting |
| Radicle | — | — | Kein Snapshot-Voting |

Viele dieser DAOs nutzen **on-chain Governance** (Tally, Governor) statt Snapshot — deshalb haben sie auf Snapshot keine recenten Proposals, obwohl sie aktiv governed werden.

### Grund 2: Zu wenig Votes pro Voter (~5 DAOs)

`checkbox-detection.ts:108` erfordert `MIN_VOTES_FOR_ANALYSIS = 3` Votes pro Voter:

```
if (votes.length < MIN_VOTES_FOR_ANALYSIS) continue
```

DAOs wie Bancor (2 Proposals in 6 Monaten, max 6 Votes), Wormhole (2 Proposals), TallyDAO (1 Proposal) haben zu wenige Proposals. Kein Voter kann 3+ Votes sammeln → `checkboxMap` bleibt leer → kein VQS-Save.

### Warum GVS trotzdem existiert

`calculateGVS()` in `aggregate.ts` berechnet 4 Komponenten parallel (HPR, DEI, PDI, GPI). Wenn GPI keine recenten Proposals findet, returned es einen neutralen Score — aber die anderen 3 Komponenten laufen normal. GVS wird gespeichert, VQS nicht.

Der VQS-Save (`saveCheckboxSignals`) passiert erst **innerhalb** von `calculateGPI()` (Zeile 302) — nach dem Proposal/Vote-Fetch und der Checkbox-Detection. Wenn der Early Return greift, wird diese Zeile nie erreicht.

## Vollständige Liste betroffener DAOs

| DAO | snapshot_space | Grund | Letztes Snapshot-Proposal |
|---|---|---|---|
| Safe | safe.eth | > 6 Monate | 2025-07-22 |
| ApeCoin | apecoin.eth | > 6 Monate | 2025-06-13 |
| Optimism | opcollective.eth | On-chain (Governor) | 2023-01-12 |
| Wormhole | wormholegovernance.eth | Zu wenig Votes | 2 Proposals |
| TallyDAO | 0xtally.eth | Zu wenig Votes | 1 Proposal |
| PoolTogether | pooltogether.eth | > 6 Monate | Prüfen |
| Bancor | bancornetwork.eth | Zu wenig Votes | 2 Proposals in 6 Mo |
| BanklessDAO | banklessvault.eth | > 6 Monate | Prüfen |
| Radicle | gov.radicle.eth | Kein Snapshot-Voting | — |
| Hop Protocol | hop.eth | > 6 Monate | Prüfen |
| KlimaDAO | klimadao.eth | > 6 Monate | Prüfen |
| NounsDAO | nouns.eth | On-chain (Governor) | Prüfen |
| Radiant Capital | radiantcapital.eth | > 6 Monate | Prüfen |
| Curve Finance | curve.eth | On-chain Voting | 2022-04-29 |
| Decentraland | decentraland.eth | Kein Snapshot-Voting | — |
| Connext | connext.eth | Kein Snapshot-Voting | — |
| Stargate / LayerZero | stgdao.eth | > 6 Monate | Prüfen |
| Treasure DAO | treasuredao.eth | > 6 Monate | Prüfen |
| Yearn Finance | ybaby.eth | > 6 Monate | Prüfen |
| dYdX | dydxgov.eth | Eigene Chain | 2024-02-28 |
| Euler Finance | eulerdao.eth | > 6 Monate | Prüfen |
| Octan | octantapp.eth | > 6 Monate | Prüfen |
| AladdinDAO | aladdindao.eth | Zu wenig Daten | 2 GVS Snapshots |

## Bewertung

**Das Verhalten ist korrekt.** Ohne genügend Voting-Daten (< 6 Monate, < 3 Votes/Voter) kann kein aussagekräftiges Vote Quality Signal berechnet werden. Inflated oder extrapolierte Werte wären schlechter als keine Daten.

## Empfehlung

### Kurzfristig (Story 9-5 Methodology Page)

Transparenz herstellen — auf der Methodology Page dokumentieren:
- VQS benötigt mindestens 3 Votes pro Delegate innerhalb der letzten 6 Monate
- DAOs ohne recente Snapshot-Proposals zeigen keinen VQS-Abschnitt
- On-chain Governance DAOs (Tally/Governor) werden aktuell nicht unterstützt

### Mittelfristig (optionales Epic)

Falls Abdeckung erhöht werden soll:
1. **Lookback auf 12 Monate erweitern** — bringt ~3-5 DAOs zurück (Safe, ApeCoin), niedrige Effort
2. **MIN_VOTES_FOR_ANALYSIS auf 2 senken** — bringt Bancor/Wormhole rein, minimaler Impact auf Qualität
3. **Tally/Governor-Integration** — für Optimism, Curve, NounsDAO, dYdX — hoher Effort, eigenes Epic

### Nicht empfohlen

- Fake/neutrale VQS-Scores erzeugen für DAOs ohne Daten — das wäre irreführend
- 6-Monate-Filter komplett entfernen — alte Votes sind nicht repräsentativ für aktuelle Governance-Qualität

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

## Post-Fix Benchmark (2026-02-12, 24 DAOs)

| Metrik | Wert |
|---|---|
| Qualifying Voters | 8.601 |
| Median VQS | 3.69 |
| Avg VQS | 4.13 |
| Max VQS | 10.00 |
| StdDev | 1.86 |

## Abhängigkeit

- Story 9-5 (Methodology Page): Transparenzhinweis zur VQS-Abdeckung
- Kein Blocker für andere Stories
