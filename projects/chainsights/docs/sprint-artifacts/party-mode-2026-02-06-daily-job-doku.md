# Party Mode Notes: Daily Job Issues & Dokumentations-Probleme

**Datum:** 2026-02-06
**Teilnehmer:** Mary (Analyst), Winston (Architect), John (PM), Paige (Tech Writer)
**Initiiert von:** Mario
**Status:** Pausiert (Mario in anderem SPM)

---

## Kontext

Mario rief Party Mode auf wegen zwei wiederkehrender Probleme:
1. Daily GVS Job Failures (50% Failure Rate)
2. Dokumentation hinkt immer hinter Implementation nach

---

## Problem 1: Daily Job Issues

### Aktueller Stand
- QStash-Refactor deployed
- Staggering: 2 DAOs alle 30 Sekunden (vorher: 3 DAOs/10s)
- 46 DAOs = ~12 Minuten Spread
- DGI-Cron auf 00:45 UTC verschoben

### Root Cause
- Snapshot API Rate Limiting
- Vercel 5-Minuten Timeout (vor QStash-Refactor)

### Winston's Empfehlungen
1. **Option A:** Manueller Test JETZT triggern, Logs beobachten
2. **Option B:** Falls weiterhin Failures → Staggering auf 1 DAO/45s erhöhen (~35 Min Spread)
3. **Option C:** Heavy DAOs (Aave, Arbitrum, Lido, Uniswap, Optimism) in separate Queue

### Status
- Job sollte mittlerweile durchgelaufen sein (Stand: Ende Party Mode)
- **TODO:** Logs prüfen, Erfolgsrate validieren

---

## Problem 2: Dokumentations-Drift

### Symptom
- `epics-phase-3.5.md` stand auf "Complete" obwohl Vanity URLs nicht funktionierten
- Story 21-5 musste nachträglich hinzugefügt werden
- Mario: "warum ist dann der Status 'Complete'??? 😠"

### Root Cause Analyse (Team)

| Agent | Root Cause | Vorgeschlagene Lösung |
|-------|-----------|----------------------|
| Mary | Fehlende Validierung vor "Complete" | Smoke-Test-Pflicht |
| Winston | Kein Gate zwischen Code und Doku | CI/CD Gate für Doku |
| John | Kein Definition of Done | DoD mit 3 Checkboxen |
| Paige | Doku als Nachgedanke | sprint-status.yaml als Single Source |

### Mario's Feedback
- Prio 2 (Daily Job ist Prio 1)
- Sieht Doku-Pflege nicht als PO/BO-Hauptaufgabe
- Fragt: Kann Murat (TEA) mehr eingebunden werden?

### John's Vorschlag: Murat als Gate
- Murat validiert sowieso nach Story-Abschluss
- **"Doku-Check"** in TEA Review-Workflow einbauen
- Story nicht `done` bis Murat bestätigt: "Epic-Status aktuell"
- Natürlicher Gate ohne extra PO-Aufwand

---

## Action Items

| Prio | Item | Owner | Status |
|------|------|-------|--------|
| 1 | Daily Job Logs prüfen nach manuellem Trigger | Mario | Pending |
| 1 | Falls <80% Erfolg: Staggering auf 1 DAO/45s erhöhen | Dev | Pending |
| 2 | TEA Review-Workflow um Doku-Check erweitern | Murat/Team | Diskussion offen |
| 2 | Definition of Done für Stories definieren | John/Team | Diskussion offen |

---

## Nächste Schritte

1. Mario prüft Job-Ergebnisse
2. Bei Erfolg: Nichts tun, beobachten
3. Bei Failure: Winston's Option B implementieren
4. Später: Party Mode fortsetzen für Doku-Thema mit Murat

---

*Party Mode pausiert auf Mario's Anfrage*
