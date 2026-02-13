# 🐛 BUG / FEATURE: Free Check muss für JEDES Snapshot-Space funktionieren

**Project:** ChainSights
**Priority:** 🔴 HIGH — Conversion-Killer
**Reporter:** Mario Semper (PO)
**Date:** 2026-02-01
**Affects:** Lead Generation, Customer Acquisition, DGI Expansion

---

## Problem

Der Free Check funktioniert aktuell **nur für DAOs die bereits in unserer internen `daos`-Tabelle** existieren (derzeit 21 DAOs). Wenn ein User ein beliebiges Snapshot-Space eingibt das nicht in der Tabelle ist, erhält er:

> ⚠️ "No governance data available yet. We don't have sufficient data for [space].eth yet. Contact us to request an analysis."

### Warum das ein Problem ist

1. **Conversion-Killer:** Visitor gibt seinen DAO ein → "No data" → Absprung. Wir verlieren den Lead komplett.
2. **Skalierungs-Blocker:** Wir planen DGI (Erweiterung auf 50 DAOs). Selbst WIR können keine Quick Checks für Kandidaten machen ohne sie erst manuell zur DB hinzuzufügen.
3. **Promise-Bruch (UX):** Die UI zeigt ein Suchfeld/"Select DAO" — der User erwartet dass er JEDES DAO eingeben kann. Die Einschränkung auf 21 vorselektierte ist nirgends kommuniziert. Das Suchfeld suggeriert Offenheit, dann blocken wir 99.8%.
4. **Wettbewerbsnachteil:** Snapshot hat 13.000+ Spaces. Wir blocken 99.8% davon ab.
5. **625x Markt-Expansion:** Aktuell 21 DAOs erreichbar (0.16% des Snapshot-Universums). Nach Fix: 13.000+ Spaces = **625-facher adressierbarer Markt**.

### Reproduktion

1. Gehe zu https://chainsights.one
2. Klicke "Free Check"
3. Gib ein: `makerdao-sky.eth` (oder jedes andere Space das nicht in den 21 ist)
4. Ergebnis: "No governance data available yet"

**Expected:** On-the-fly GVS-Berechnung und Anzeige der Ergebnisse.

---

## Kontext: Wie der Free Check aktuell funktioniert

```
User wählt DAO → Lookup in `daos`-Tabelle
                    ↓ gefunden?
              JA: Zeige gecachten GVS → Email-Gate → Quick Check Ergebnis
              NEIN: "No governance data available yet" ← 🐛 HIER
```

### Was wir bereits wissen (aus Spike & Team-Diskussion)

- **Snapshot API** liefert ALLE Daten die wir für eine GVS-Berechnung brauchen (Proposals, Votes, Voter-Adressen, VP, VP-by-Strategy, Timestamps) — bestätigt durch den ENS-Spike
- **GVS-Berechnung** (HPR, DEI, PDI, GPI) läuft bereits für die 21 getrackedten DAOs
- **Team-Decision** (Winston/Barry): Option 1 wurde gewählt — GVS-Berechnung soll im Report-Flow mitlaufen und in `gvsSnapshots` gespeichert werden
- **AI-Kosten** pro Report: ~€0.03 — es gibt kein Kostenproblem bei on-the-fly Berechnung

---

## Lösung: Open-Universe Free Check

### Ziel-Flow

```
User gibt beliebiges Snapshot-Space ein
    ↓
Space in `daos`-Tabelle?
    ├── JA → Zeige gecachte GVS-Daten (wie bisher, < 200ms)
    │
    └── NEIN → Space in `reportedDaos` mit gültigem Cache (< 24h)?
                  ├── JA → Zeige gecachte Daten aus reportedDaos
                  │
                  └── NEIN → Snapshot API: Space validieren
                                ↓ Space existiert?
                                ├── NEIN → "This Snapshot space doesn't exist. Check the ENS name."
                                │
                                └── JA → Proposals zählen
                                          ├── < 5 Proposals → "Not enough governance activity yet.
                                          │                     We need at least 5 proposals."
                                          │
                                          ├── 5-19 Proposals → On-the-fly GVS berechnen
                                          │                     → ⚠️ Low Confidence Badge
                                          │                     → Speichern in reportedDaos
                                          │                     → Email-Gate → Ergebnis
                                          │
                                          └── ≥ 20 Proposals → On-the-fly GVS berechnen
                                                                → ✅ Full Confidence
                                                                → Speichern in reportedDaos
                                                                → Email-Gate → Ergebnis
```

---

## Anforderungen

### Must-Have (MVP)

| # | Requirement | Details |
|---|-------------|---------|
| R1 | Jedes gültige Snapshot-Space soll einen Free Check erhalten können | Validierung über Snapshot GraphQL API |
| R2 | On-the-fly GVS-Berechnung für unbekannte Spaces | Gleiche Engine wie für getrackete DAOs |
| R3 | Progressiver Loading-State | Mehrstufiges Feedback: "Validating space..." → "Fetching proposals..." → "Analyzing voting patterns..." → "Calculating governance score..." Nicht nur ein Spinner — der User soll sehen dass etwas passiert. |
| R4 | Ergebnis zeigt GVS + 4 Komponenten (HPR, DEI, PDI, GPI) | Identisch zum Free Check für bekannte DAOs |
| R5 | Email-Gate bleibt bestehen | Lead-Generierung funktioniert unverändert |
| R6 | Fehlerbehandlung für ungültige/leere Spaces | Siehe Minimum-Schwelle (R8), Flow oben, und Error-Copy-Guide unten |

### Should-Have

| # | Requirement | Details |
|---|-------------|---------|
| R7 | Caching in `reportedDaos` nach erstem Check | Siehe Caching-Strategie unten |
| R8 | Minimum-Schwelle mit Confidence-Levels | Siehe Minimum-Schwelle unten |
| R9 | Autocomplete/Typeahead für Snapshot Spaces | Optional UX-Verbesserung |
| R10 | Confidence-Level anzeigen | **Upgraded von Should → Must-Have** (Team-Review). On-the-fly Checks mit wenig Daten MÜSSEN dem User kommunizieren dass es ein vorläufiger Score ist. Ohne Badge verlieren wir Glaubwürdigkeit. Badge: ⚠️ Low (5-19 Proposals) / ✅ High (≥ 20). |

### Nice-to-Have

| # | Requirement | Details |
|---|-------------|---------|
| R11 | Upsell-CTA nach on-the-fly Check | "Want this DAO tracked daily? Request inclusion in our DGI" |
| R12 | Rate-Limiting für on-the-fly Checks | Siehe Rate-Limiting unten |

---

## Design-Entscheidungen

### R8 — Minimum-Schwelle & Confidence-Levels

**Entscheidung:** Gestaffeltes Modell, nicht binär.

| Proposal-Anzahl | Verhalten | Confidence | Begründung |
|---|---|---|---|
| **< 5 Proposals** | ❌ Kein Check | — | Keine statistische Basis. 3 Proposals × 5 Votes = 15 Datenpunkte = Rauschen, kein Signal. HPR/DEI nicht sinnvoll berechenbar. |
| **5–19 Proposals** | ✅ Check mit Badge | ⚠️ **Low** | Erste Muster erkennbar, aber Trends und Gini-Koeffizient noch instabil. User muss wissen dass sich Scores stark ändern können. |
| **≥ 20 Proposals** | ✅ Full Check | ✅ **High** | Ausreichend Datenpunkte für HPR, DEI, PDI, GPI. Konsistent mit bestehendem Confidence-Level-System aus GVS-Methodologie. |

**UX-Meldungen:**
- **< 5:** "This DAO doesn't have enough governance activity yet for meaningful analysis. We need at least 5 proposals with votes."
- **5-19:** Ergebnis + gelbes Badge: "⚠️ Limited data — scores may change significantly as more proposals are created."
- **≥ 20:** Ergebnis ohne Badge (normales Verhalten)

---

### R12 — Rate-Limiting

**Entscheidung:** Differenziert nach User-Typ. Limits gelten NUR für on-the-fly Checks auf unbekannte DAOs. Gecachte DAOs (in `daos` oder `reportedDaos` mit TTL < 24h) zählen NICHT gegen das Limit.

| User-Typ | Limit | Scope | Begründung |
|---|---|---|---|
| **Anonymous** (kein Email) | 2 / IP / Stunde | IP-basiert | Genug zum Testen, schützt vor Scraping |
| **Free** (nach Email-Gate) | 5 / Email / Tag | Email-basiert | Lead ist identifiziert, mehr Vertrauen |
| **@masem.at / Admin** | Unlimited | Whitelist | Mario braucht das für DGI Prüfung |

**Wichtig:**
- Das bestehende Rate-Limit für @masem.at Test-Bypass-Emails ist ein anderer Scope — NICHT auf Free Checks anwenden.
- Bei Limit-Erreichen: "You've reached the limit for new DAO analyses. Try again in [X] or check one of our [tracked DAOs]."

---

### Caching-Strategie: 3-Stufen-Modell

**Entscheidung:** Persistent cachen, aber NICHT automatisch in den Daily Recalculation Job.

```
On-the-fly Check
    │
    ├── Stufe 1: Session-Cache (immer)
    │   Ergebnis für Browser-Session gecached.
    │   Nochmal klicken → kein neuer API-Call.
    │   TTL: Session-Ende.
    │
    ├── Stufe 2: Persistent in `reportedDaos` (immer)
    │   Jeder on-the-fly Check wird gespeichert mit Timestamp.
    │   Anderer User checkt denselben DAO innerhalb 24h → gecachtes Ergebnis.
    │   TTL: 24 Stunden, danach neu berechnen bei nächstem Request.
    │   Baut organisch eine "Discovery-Pipeline" auf.
    │
    └── Stufe 3: Daily Recalculation Job → ❌ NICHT AUTOMATISCH
        reportedDaos-Einträge haben Flag: `include_in_daily: false`
        Nur Mario kann über Admin UI (Story 1.4) ein DAO promoten:
        reportedDaos → daos-Tabelle → Daily Job
```

**Begründung gegen Auto-Promotion in Daily Job:**
- Unkontrolliertes Wachstum: Jemand checkt 50 Spam-DAOs → wir berechnen die ab morgen täglich
- API-Kosten-Explosion: Jeder DAO im Daily Job = tägliche Snapshot API-Calls
- Widerspricht dem DGI Kurations-Prinzip: Mario entscheidet manuell was in den Index kommt

**`reportedDaos` Schema-Erweiterung:**

```sql
ALTER TABLE "reportedDaos" ADD COLUMN IF NOT EXISTS
    include_in_daily BOOLEAN DEFAULT false,
    last_free_check_at TIMESTAMP,
    free_check_count INTEGER DEFAULT 0,
    confidence_level VARCHAR(10), -- 'low', 'high'
    proposal_count INTEGER,
    cached_gvs JSONB; -- { gvs, hpr, dei, pdi, gpi, calculated_at }
```

---

## Architektur-Empfehlung: Async Pattern (Team-Review)

**Entscheidung:** On-the-fly Checks für unbekannte DAOs sollen **asynchron** laufen.

**Begründung (Winston):** Der Performance-Gap zwischen gecacht (< 200ms) und on-the-fly (3-20s) ist zu gross für denselben synchronen Response-Path. Ein synchroner API-Call mit 20s Timeout ist fragil — Vercel hat ein 30s Function-Timeout, Netzwerk-Abbrüche, Browser-Timeouts.

**Pattern:**

```
Frontend                    API                      Background
   │                         │                          │
   ├── POST /api/free-check  │                          │
   │   { space: "xyz.eth" }  │                          │
   │                         ├── Space in Cache?         │
   │                         │   ├── JA → 200 { result } │
   │                         │   └── NEIN → Start Job    │
   │                         │         ├── 202 { jobId } │
   │                         │         │                 ├── Validate Space
   │   ← 202 Accepted        │         │                 ├── Fetch Proposals
   │                         │         │                 ├── Calculate GVS
   │   (Poll / SSE)          │         │                 ├── Cache in reportedDaos
   │   ├── GET /api/free-check/status?jobId=xxx          │
   │   │   ← { status: "calculating", step: "Fetching proposals..." }
   │   │   ...                │         │                 │
   │   │   ← { status: "done", result: { gvs, hpr, ... } }
   │                         │         │                 │
```

**Vorteile:**
- Kein Timeout-Risiko bei grossen DAOs
- Progressiver Loading-State wird natürlich (jeder Step meldet Status)
- Frontend kann das Ergebnis auch nach Tab-Wechsel noch abrufen
- Retry-Logik trivial (Job nochmal starten)

**Alternative (einfacher, MVP-tauglich):** Synchroner API-Call mit langem Timeout + `ReadableStream` für Progress-Updates. Weniger robust, aber schneller zu implementieren. Team-Empfehlung: **MVP synchron mit Stream, V2 async.**

### V2 Async — Trigger-Bedingungen (PO-Review)

V2 Async wird als **separates Backlog-Ticket** geführt. Auslöser ist **einer** der folgenden messbaren Trigger:

| # | Trigger | Messung | Schwelle |
|---|---------|---------|----------|
| T1 | **Volumen** | On-the-fly Checks pro Woche | > 100 / Woche |
| T2 | **Stabilität** | Erster Timeout-Fehler (>30s) in Production | 1 Vorfall |
| T3 | **Infrastruktur-Limit** | On-the-fly Check Duration p95 | > 25s (Vercel 30s Function-Limit) |

**Monitoring:** Custom Event `on_the_fly_check_duration_ms` an masemIT Analytics (`analytics.masem.at`) senden. Zusätzlich `duration_ms` in `job_logs`-Tabelle als Fallback-Quelle.

> ⚠️ Ohne Trigger-Ticket wird V2 nie passieren. Ticket muss erstellt werden bevor MVP deployed wird.

---

## Error-Copy-Guide (Team-Review)

Alle User-facing Meldungen an einem Ort, damit Copy konsistent ist:

| Szenario | Meldung | Ton |
|---|---|---|
| **Space existiert nicht** | "This Snapshot space doesn't exist. Double-check the ENS name and try again." | Neutral, hilfreich |
| **< 5 Proposals** | "This DAO is still early — not enough governance activity for a meaningful score yet. We need at least 5 closed proposals." | Wertschätzend, nicht abweisend |
| **5-19 Proposals (Low Confidence)** | Badge: "⚠️ Preliminary score — based on limited governance history. Scores may shift as more proposals are created." | Transparent, ehrlich |
| **≥ 20 Proposals (High Confidence)** | Kein Badge — normales Ergebnis | Standard |
| **Rate-Limit erreicht** | "You've reached the limit for new DAO analyses. Try again in [X] or explore one of our [tracked DAOs →]" | Freundlich, bietet Alternative |
| **Snapshot API Down** | "We're having trouble reaching Snapshot's API right now. Please try again in a few minutes." | Keine Schuldzuweisung |
| **GVS-Berechnung fehlgeschlagen** | "Something went wrong analyzing this DAO. Our team has been notified. Try again or [contact us →]" | Verantwortungsvoll |

---

## Technische Überlegungen

### Snapshot API Calls (bereits validiert)

```graphql
# Step 1: Validate space exists + get proposal count
query {
  space(id: "makerdao-sky.eth") {
    id
    name
    members
    proposals_count
    followers
    voting { type }
  }
}

# Step 2 (if proposals_count >= 5): Fetch proposals + votes
query {
  proposals(
    where: { space: "makerdao-sky.eth", state: "closed" }
    first: 100
    orderBy: "created"
    orderDirection: desc
  ) {
    id, title, choices, scores, votes, created, end
  }
}
```

### GVS Engine Refactoring

Die bestehende GVS-Engine muss einen **standalone mode** bekommen:

```typescript
// Aktuell (nur für getrackete DAOs):
calculateGVS(daoId: string): Promise<GVSResult>
//   → liest aus daos-Tabelle + gvsSnapshots

// Neu (standalone für beliebige Spaces):
calculateGVSOnTheFly(snapshotSpaceId: string): Promise<GVSResult & { confidence: 'low' | 'high' }>
//   → fetcht direkt von Snapshot API
//   → gleiche Berechnung, andere Datenquelle
//   → returned GVSResult + confidence_level
```

### Performance-Erwartung

| Szenario | Response Time | Quelle |
|---|---|---|
| Gecachtes DAO (`daos`-Tabelle) | < 200ms | PostgreSQL |
| Gecachtes DAO (`reportedDaos`, < 24h) | < 200ms | PostgreSQL |
| On-the-fly, kleines DAO (~20 Proposals) | 3-8 Sekunden | Snapshot API + Berechnung |
| On-the-fly, großes DAO (100+ Proposals) | 10-20 Sekunden | Snapshot API + Berechnung |

→ Progressiver Loading-State mit Step-Feedback ist Pflicht für on-the-fly Checks.

---

## UX-Hinweise (Team-Review)

### Loading-Erlebnis ist DAS Produkt-Erlebnis

> "Wenn jemand 10 Sekunden wartet und dann einen Score sieht, ist das **magisch**. Wenn er 10 Sekunden wartet und einen Fehler bekommt, ist er weg." — Sally (UX)

Der Loading-State für on-the-fly Checks muss **progressiv** sein, nicht nur ein generischer Spinner:

1. ✅ "Validating Snapshot space..."
2. ✅ "Fetching governance proposals..."
3. ✅ "Analyzing voting patterns..."
4. ✅ "Calculating governance score..."
5. ✅ Score-Reveal mit kurzer Animation

### Young DAOs: Soft Rejection mit Opt-In

Für DAOs mit < 5 Proposals ist die Absage eine **Chance**, keinen Lead zu verlieren:

> "This DAO is still early — not enough governance activity for a meaningful score yet. **Want us to notify you when there's enough data?**"
> [Email-Input] [Notify Me]

Das generiert trotzdem einen Lead, auch wenn wir den Check nicht durchführen können.

> ⚠️ **Scope-Entscheidung (PO-Review):** Das "Notify Me" Feature ist als **separates Nice-to-Have Ticket** ausgelagert. Es ist NICHT Teil der MVP Acceptance Criteria dieses Tickets. Begründung: Eigener DB-Schema-Scope (Email + Space + Timestamp), Notification-Trigger-Logik, separater Email-Flow. Die **wertschätzende Ablehnungs-Copy** für < 5 Proposals ist hingegen Teil des MVP (siehe AC + Error-Copy-Guide).

### Positioning-Upgrade

Wenn dieses Feature live ist, wird "Works with **any** Snapshot DAO" ein Killer-Feature. Empfehlung (Paige):
- Hero-Text auf Landing Page aktualisieren
- Pricing Page: "13,000+ DAOs supported" hinzufügen
- Free Check CTA: "Check any DAO" statt "Select your DAO"

---

## Acceptance Criteria

- [ ] User kann ein beliebiges Snapshot-Space im Free Check eingeben
- [ ] Gültige Spaces mit ≥ 20 Proposals erhalten GVS + 4 Komponenten (Full Confidence)
- [ ] Gültige Spaces mit 5-19 Proposals erhalten GVS + Low Confidence Badge
- [ ] Spaces mit < 5 Proposals erhalten klare "Not enough activity"-Meldung
- [ ] Ungültige/nicht-existierende Spaces erhalten "Space not found"-Meldung
- [ ] Loading-State wird während on-the-fly Berechnung angezeigt
- [ ] Email-Gate funktioniert unverändert
- [ ] Gecachte DAOs (bestehende 21) sind nicht langsamer als vorher
- [ ] On-the-fly Ergebnisse werden in `reportedDaos` gecached (24h TTL)
- [ ] On-the-fly DAOs werden NICHT automatisch in den Daily Recalculation Job aufgenommen
- [ ] Rate-Limits greifen differenziert (Anon: 2/h, Free: 5/Tag, Admin: unlimited)
- [ ] Admin kann via Admin UI ein DAO von reportedDaos → daos promoten
- [ ] Kein Breaking Change an bestehendem Free Check Flow

---

## Auswirkung auf andere Features

| Feature | Auswirkung |
|---------|------------|
| **DGI Expansion** | Ermöglicht Mario, neue Kandidaten direkt per Free Check zu prüfen |
| **Report Sales** | User die ihren DAO im Free Check sehen → höhere Deep Dive / Audit Conversion |
| **Lead-Gen** | Massiv größerer adressierbarer Markt (13.000+ Spaces statt 21) |
| **Pricing Page** | "Works with any Snapshot DAO" wird zum Feature statt Einschränkung |
| **Admin UI (Story 1.4)** | reportedDaos → daos Promotion-Flow muss integriert werden |
| **Daily Recalculation Job** | Keine Änderung — nur manuell promotete DAOs werden aufgenommen |

---

## Geschätzter Aufwand

_Revidierte Schätzung (PO-Review 2026-02-01) — berücksichtigt Scope-Erweiterungen aus Team-Review._

| Task | Effort | Delta vs. v1 |
|------|--------|-------------|
| Snapshot Space Validation Endpoint | 2h | — |
| GVS Engine Refactoring (standalone mode) | 6-8h | — |
| Frontend: Open Input + **Progressive** Loading State (5 Steps) + Confidence Badge | 6h | +2h (5 UI-States statt Spinner) |
| Error-Copy-Guide Implementierung (7 Szenarien) | 3h | +1h (Copy steht, nur Implementierung) |
| Stream-basierter API Response (MVP Sync+Stream) | 4h | +2h (vs. naiver sync fetch) |
| Caching Layer (`reportedDaos` Schema + 24h TTL Logic) | 3h | — |
| Rate-Limiting (differenziert nach User-Typ) | 2h | — |
| Admin UI Integration (Promote-Button reportedDaos → daos) | 2h | — |
| Testing (verschiedene Space-Größen, Cache-Expiry, Rate-Limits) | 3h | — |
| **Total (ohne Young DAO Opt-In)** | **~31-33h** | +7h |

> **Young DAO Notification Opt-In** ist als separates Ticket ausgelagert (siehe unten). Zusatz-Aufwand: ~2-3h.

---

## Priorität-Begründung

Dieses Issue blockiert:
1. **DGI Rollout** — wir können keine neuen DAOs prüfen
2. **Jede Marketing-Kampagne** — wenn wir Traffic schicken und der User sein DAO nicht findet, ist das Budget verbrannt
3. **Organic Growth** — jeder DAO-Lead der seinen Space nicht findet ist verloren

**Empfehlung:** Vor dem nächsten Marketing-Push fixen. Idealerweise zusammen mit DGI Epic 1.

---

## Team-Review Log

### 2026-02-01 — Party Mode Review (Winston, Sally, Paige, Mary)

**Teilnehmer:** Winston (Architect), Sally (UX Designer), Paige (Technical Writer), Mary (Business Analyst)

**Zentrale Ergebnisse:**

1. **Winston (Architektur):** Async Pattern empfohlen für on-the-fly Checks. Performance-Gap (200ms vs 3-20s) erfordert separaten UX-Flow. MVP: Synchron mit Stream, V2: Async mit Polling/SSE.

2. **Sally (UX):** Loading-State ist das Produkt-Erlebnis. Progressive Feedback statt generischer Spinner. Young DAOs (< 5 Proposals) verdienen eine wertschätzende Rejection mit Notification-Opt-In als Lead-Capture-Alternative.

3. **Paige (Documentation):** R10 (Confidence-Level) muss Must-Have sein — ohne Transparenz bei wenig Daten verlieren wir Glaubwürdigkeit. Error-Copy-Guide erstellt für konsistente User-Facing-Meldungen. Feature-Kommunikation: "Works with any Snapshot DAO" ist ein Positioning-Upgrade.

4. **Mary (Analyse):** 625x Markt-Expansion quantifiziert. Requirement-Doc war bereits solide. Offene Fragen zu Schwellenwerten und Caching wurden im Meeting geklärt und ins Dokument übernommen.

**Änderungen am Dokument:**
- R3 erweitert: Progressiver Loading-State mit benannten Steps
- R10 upgraded: Should-Have → Must-Have (Confidence-Badge)
- Neuer Abschnitt: Architektur-Empfehlung (Async Pattern)
- Neuer Abschnitt: Error-Copy-Guide (7 Szenarien)
- Neuer Abschnitt: UX-Hinweise (Loading-Erlebnis, Young DAO Opt-In, Positioning)
- Problem-Punkt 3 präzisiert: "Promise-Bruch"
- Problem-Punkt 5 hinzugefügt: 625x Markt-Expansion

### 2026-02-01 — PO-Review (Mario, Winston, Sally, John, Paige, Mary)

**Teilnehmer:** Mario (PO), Winston (Architect), Sally (UX), John (PM), Paige (Tech Writer), Mary (Analyst)

**3 Punkte von Mario angesprochen:**

1. **Aufwand-Schätzung revidiert:** 24-26h → **31-33h** (ohne Young DAO Opt-In). Delta: +2h Progressive Loading (5 UI-States), +1h Error-Copy-Guide Implementierung, +2h Stream-basierter API Response, +2h Buffer. Geschätzt von Winston + Sally.

2. **V2 Async als eigenes Trigger-basiertes Ticket:** "MVP synchron mit Stream, V2 async" bekommt 3 messbare Trigger-Bedingungen (Volumen >100/Woche, erster Timeout in Prod, p95 >25s). Monitoring via masemIT Analytics custom event `on_the_fly_check_duration_ms` + `job_logs` Fallback. Kein Vercel Analytics — korrigiert auf masemIT Analytics (John).

3. **Young DAO Opt-In ausgelagert:** "Notify Me" Feature ist separates Nice-to-Have Ticket (~2-3h). Die wertschätzende Ablehnungs-Copy für < 5 Proposals bleibt im MVP (AC + Error-Copy-Guide). Empfehlung von Paige, bestätigt von Sally.

**Änderungen am Dokument:**
- Aufwand-Tabelle revidiert mit Delta-Spalte
- V2 Async Trigger-Tabelle eingefügt (3 messbare Kriterien)
- Young DAO Opt-In: Scope-Hinweis im UX-Abschnitt ergänzt
- Monitoring-Quelle korrigiert: masemIT Analytics statt Vercel Analytics
