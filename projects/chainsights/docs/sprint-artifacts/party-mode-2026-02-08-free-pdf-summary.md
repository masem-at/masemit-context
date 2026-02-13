# Party Mode Session — Free Score Card Requirement

**Datum:** 2026-02-08
**Status:** ABGESCHLOSSEN — Requirement v2.0 finalisiert
**Agents:** Mary (Analyst), John (PM), Sally (UX), Paige (Tech Writer), Winston (Architect)
**Requirement:** `docs/_masemIT/requirements/requirement-free-pdf-summary.md`

---

## Session-Verlauf

### Runde 1 — Initiale Review (vor MMS-Klaerung)

**Diskutierte Punkte:**
1. Email-Pflicht ja/nein → Starke Argumente gegen Pflicht im Free Check (John, Sally)
2. Exklusiver PDF-Wert → Rank + Benchmark als PDF-Exclusive (Sally, Paige)
3. Copy & Naming → "Governance Score Card" statt "PDF Summary" (Paige)
4. Baseline-Traffic fehlt → Funnel-Daten noetig fuer KPI-Validierung (Mary)
5. Technische Richtung → @react-pdf/renderer, pre-generated PDFs (Winston)

**Pause:** Sempre klaert etwas mit MMS

### Runde 2 — Nach MMS-Klaerung

**Sempre brachte mit:**
- Aktualisiertes Requirement (v1 → v2) mit allen Punkten aus Runde 1 adressiert
- MMS Parties API ist live: `https://api.masem.at/docs/parties`
- Baseline-Daten: 5 check_page_view → 1 completed = 20% Conversion, 1 Lead/Woche
- DSGVO-Abschnitt neu hinzugefuegt

**Team-Analyse des MMS API:**
- `POST /v1/parties` — Find-or-create by Email, `source_project: chainsights` bereits als Enum
- `party_status: lead` — perfekt fuer Lead Capture
- Contact-Types: email, wallet — passt zum DAO-Kontext
- Consent-Modul in Arbeit

**Neue Entscheidungen:**
- Kein lokales `leads`-Table → MMS API als Lead-Storage
- Consent zentral im MMS (Fallback: lokaler Boolean bis MMS Consent fertig)
- Consent-Checkbox aufteilen: Score Card Delivery (kein Opt-in) vs. Marketing (Opt-in)
- reports-Tabelle mit Tier `score_card` erweitern (Winston: Option A)

### Runde 3 — Architektur + Codebase-Analyse

**Codebase-Findings:**
- Resend bereits voll integriert (`hello@chainsights.one`)
- Bestehende `sendReportEmail()` als Vorlage fuer Score Card Email
- Download-Token-Infrastruktur (`/api/download/:token`) wiederverwendbar
- HTML-Templates inline (kein React Email)
- QStash fuer async Jobs verfuegbar

**Finale Entscheidungen (Sempre):**

| Punkt | Entscheidung |
|---|---|
| Email-Service | Resend (bereits integriert) |
| Lead-Storage | MMS API |
| Consent | MMS (zentral) |
| Consent-Checkbox | Getrennt: Delivery + Marketing |
| PDF Styling | Brand Colors, konsistent mit bestehenden Reports |
| Rate Limiting | V1 ohne Limit |
| Delivery | Link per Email (wie bestehende Reports) |
| DB-Architektur | reports-Tabelle mit Tier `score_card` |
| Free Check Email entfernen | Separates Ticket, vorgezogen, Solo-Dev |
| Email-Copy | Paige bereitet vor (Subject, Body, Upsell-CTA) |
| Upsell-Link | `/check?dao=[slug]&tier=deep_dive` (John) |

---

## Finales Requirement-Dokument

**Version:** 2.0
**Status:** Ready for Implementation
**Datei:** `docs/_masemIT/requirements/requirement-free-pdf-summary.md`

Alle 12 von 13 offenen Fragen geloest. Einzige verbleibende Frage:
- [ ] MMS Consent-Modul: Wann fertig? (Fallback vorhanden)

---

## Naechste Schritte

1. **SOFORT (Solo-Dev):** Free Check Email-Feld entfernen (separates Ticket)
2. **Danach:** Score Card Feature gemaess Requirement v2.0 implementieren
3. **Reihenfolge:** Cookie Consent → Privacy Policy → Score Card → Unsubscribe Flow
