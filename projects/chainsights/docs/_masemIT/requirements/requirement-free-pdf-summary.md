# Requirement: Free Score Card nach Free Check

**Project:** ChainSights
**Feature:** Free Score Card Download
**Version:** 2.0
**Date:** 2026-02-08
**Author:** Mario Semper (PO)
**Status:** Ready for Implementation

---

## 1. Kontext & Problem

### Aktuelle Situation
- User durchläuft Free Check → muss Email eingeben → sieht dann Result
- **Problem:** Email-Pflicht VOR dem Result = zu hohe Friction
- **Daten (1.-8. Feb):** 5 check_page_view → 1 completed = 20% Conversion
- **Beobachtung:** 80% Drop passiert bei DAO-Auswahl — User starten, aber brechen ab

### Hypothese
Die Email-Pflicht im Step 3 (vor dem Result) schreckt User ab. Sie wollen "nur mal schauen" ohne sich zu committen.

### Lösung
1. **Free Check ohne Email** — Result sofort nach DAO + Tier Auswahl
2. **Score Card mit Email** — PDF per Email als Mehrwert nach dem Result

### Ziele
- Höhere Free Check Completion Rate (Ziel: >50% statt aktuell 20%)
- Lead Capture über Score Card (statt über Free Check selbst)
- Niedrigere Hürde für ersten "Aha-Moment"
- Natürlicher Upsell nach Value-Erlebnis

---

## 2. User Flow

```
[Neuer Free Check Flow — OHNE Email]
     │
     ▼
┌─────────────────────────────────────┐
│  ④ Your Results                     │
│                                     │
│  GVS Score: 76 (Vital)              │
│  ┌─────┬─────┬─────┬─────┐          │
│  │ HPR │ DEI │ PDI │ GPI │          │
│  │ 90  │ 55  │ 71  │ 82  │          │
│  └─────┴─────┴─────┴─────┘          │
│                                     │
│  📊 Above average for [Category]    │
│                                     │
├─────────────────────────────────────┤
│                                     │
│  📊 Get Your Governance Score Card  │
│     — Free                          │
│                                     │
│  ┌─────────────────────┬──────────┐ │
│  │ your@email.com      │  Send    │ │
│  └─────────────────────┴──────────┘ │
│                                     │
│  ☐ Send me occasional product      │
│    updates. Unsubscribe anytime.   │
│                                     │
│  By requesting, you agree to our   │
│  Privacy Policy.                   │
│                                     │
│  ✓ Official score + category ranking│
│  ✓ Benchmark vs. 46 DAOs           │
│  ✓ Ready to share with your team   │
│                                     │
├─────────────────────────────────────┤
│                                     │
│  Want deeper insights?              │
│                                     │
│  ┌──────────────┐ ┌───────────────┐ │
│  │ Deep Dive    │ │ Full Audit    │ │
│  │ €49          │ │ €149          │ │
│  └──────────────┘ └───────────────┘ │
│                                     │
└─────────────────────────────────────┘
```

### Email-Pflicht: Nur beim Score Card Download

**Änderung gegenüber aktuellem Flow:**

| Aktuell (Problem) | Neu (Lösung) |
|-------------------|--------------|
| Email VOR Result | Email NACH Result |
| Friction bei Step 3 | Friction nur bei Score Card Request |
| 80% Drop bei DAO-Auswahl | Erwartung: höhere Completion |
| Direkter PDF Download | PDF per Email (wie bestehende Reports) |

**Neuer Free Check Flow (ohne Email):**
```
1. DAO auswählen
2. Tier auswählen (Free Check)
3. Check starten
4. ✅ Result sofort anzeigen (Score + 4 Metriken)
```

**Score Card Request (mit Email):**
```
5. "Get Your Score Card" CTA
6. Email eingeben + optionaler Marketing-Consent
7. "Score Card is on its way! Check your inbox."
8. User erhält Email mit Download-Link
9. Klick → PDF Download
```

**Consent-Trennung (DSGVO):**
- Score Card Delivery = Vertragserfüllung → kein Opt-in nötig
- Marketing/Product Updates = separate Checkbox → Opt-in nötig, NICHT pre-checked

**Begründung:**
- User sehen erstmal Wert → Vertrauen aufbauen
- Email-Hürde nur für "Extra" (PDF mit Rank + Benchmark)
- Wer keine Email geben will, hat die Basis-Daten trotzdem gesehen
- Aktuell: 5 check_page_view → 1 completed (20% Conversion)
- Hypothese: Ohne Email-Pflicht im Flow → höhere Check Completion

### Exklusiver Score Card Wert (NUR im PDF)

| Screen zeigt | Score Card zeigt zusätzlich |
|--------------|----------------------------|
| Score + Grade | ✅ |
| 4 Metriken | ✅ |
| "Above average" Text | ✅ |
| — | **Kategorie-Rank** ("#5 of 28 DeFi DAOs") |
| — | **Overall Rank** ("#12 of 46") |
| — | **Benchmark-Balken** (Your Score vs. Category Avg vs. Overall Avg) |
| — | **Timestamp** ("Generated on [Date]") |
| — | **ChainSights Branding** (offizielles, teilbares Format) |

---

## 3. Score Card Inhalt

**Format:** 1-Seiter, A4, PDF
**Dateiname:** `[DAO-Name]-governance-scorecard-[Date].pdf`
**Styling:** ChainSights Brand Colors (konsistent mit Deep Dive + Full Audit Reports)

### Struktur

```
┌─────────────────────────────────────────────┐
│  ChainSights Logo                           │
│  Governance Score Card                      │
│  Generated: [Date]                          │
├─────────────────────────────────────────────┤
│                                             │
│  [DAO Name]                                 │
│  ══════════════════════════════════════     │
│                                             │
│  Overall Score: 76/100 (B - Vital)          │
│  ████████████████████░░░░░░░░░░             │
│                                             │
│  Category: [Infrastructure]                 │
│  #12 overall of 46 DAOs                     │
│  #5 in Infrastructure                       │
│                                             │
├─────────────────────────────────────────────┤
│                                             │
│  Score Breakdown                            │
│  ───────────────────────────────────────    │
│  Human Participation (35%)     90  ████████ │
│  Delegate Engagement (25%)     55  █████    │
│  Power Dynamics (20%)          71  ███████  │
│  Grassroots Participation (20%) 82 ████████ │
│                                             │
├─────────────────────────────────────────────┤
│                                             │
│  How You Compare                            │
│  ───────────────────────────────────────    │
│                                             │
│  Your Score        ████████████████ 76      │
│  Infrastructure    ████████         42.7    │
│  All DAOs          ███████████      54.0    │
│                                             │
│  ✅ You're in the top 25% of all DAOs       │
│                                             │
├─────────────────────────────────────────────┤
│                                             │
│  Key Insights                               │
│  ───────────────────────────────────────    │
│  ✅ Excels at: Human Participation          │
│  ⚠️ Needs work: Delegate Engagement         │
│                                             │
├─────────────────────────────────────────────┤
│                                             │
│  Want the full picture?                     │
│                                             │
│  Deep Dive Report — €49                     │
│  • AI-powered analysis                      │
│  • Specific recommendations                 │
│  • Historical trends                        │
│                                             │
│  → chainsights.one/check                    │
│                                             │
└─────────────────────────────────────────────┘
```

---

## 4. Technische Anforderungen

### 4.1 Architektur-Entscheidungen

| Entscheidung | Ergebnis | Begründung |
|---|---|---|
| Lead-Storage | **MMS API** (`POST /v1/parties`) | Zentrales CRM, kein lokales `leads`-Table |
| Consent-Storage | **MMS** (zentral, in Arbeit) | Einheitlich über alle masem-Projekte |
| DB für Score Card Metadaten | **Bestehende `reports`-Tabelle** mit Tier `score_card` | Gleiche Download-Token-Infrastruktur, kein neuer Code |
| PDF-Generierung | **`@react-pdf/renderer`** | Reicht für 1-Seiter, leichtgewichtig, kein Headless Browser nötig |
| Email-Service | **Resend** (bereits integriert) | `hello@chainsights.one`, bestehende Templates als Vorlage |
| Download-Mechanismus | **Bestehende Token-Infrastruktur** (`/api/download/:token`) | 30-Tage Expiry, trackbar, sicher |
| Delivery | **Link per Email** (wie bestehende Reports) | Konsistent, Email-Verification implizit |
| Rate Limiting | **V1 ohne Limit** | Bei 5 Views/Woche ist Abuse unrealistisch, beobachten + iterieren |

### 4.2 Technischer Flow

```
1. User gibt Email ein + optionaler Marketing-Consent
2. Frontend: POST /api/score-card
   ├── Body: { email, daoSlug, gvsScore, metrics, marketingConsent, utmParams }
   │
3. Backend /api/score-card:
   ├── Email Validierung
   ├── POST MMS API /v1/parties (find-or-create, source_project: chainsights)
   ├── PDF generieren (@react-pdf/renderer, server-side)
   ├── PDF hochladen (Vercel Blob oder S3)
   ├── Download-Token generieren
   ├── INSERT reports (tier: 'score_card', pdfToken, finalPdfUrl)
   ├── sendScoreCardEmail() via Resend
   └── Response: { success: true }
   │
4. User bekommt Email mit Download-Link
5. Klick → /api/download/:token → PDF
```

### 4.3 MMS API Integration

```
POST https://api.masem.at/v1/parties
Headers: x-api-key: mms_chainsights_{secret}

Body:
{
  "email": "user@example.com",
  "display_name": "user@example.com",
  "party_type": "person",
  "source_project": "chainsights"
}

Response: { party_id, contact_point_id, created }
```

- Consent wird zentral im MMS gespeichert (Consent-Modul in Arbeit)
- Bis Consent-Modul fertig: `marketing_consent` Boolean im lokalen `reports`-Eintrag als Fallback

### 4.4 Frontend

- Email-Input Feld nach `check_result_shown`
- Marketing-Consent Checkbox (optional, NICHT pre-checked)
- Privacy Policy Link
- Send Button (disabled bis Email valide)
- Loading State während Generierung + Versand
- Success State: "Your Score Card is on its way! Check your inbox."

### 4.5 Email Template

**Vorlage:** Bestehende `sendReportEmail()` in `src/lib/email/index.ts` (schlankere Version)

**Betreff:** `Your Governance Score Card: [DAO Name] — [Score]/100`

**Body-Elemente:**
- Score Card Download-Link (1 Link, kein JSON)
- Kein Billing Portal (ist kostenlos)
- Upsell-Block: "Want the full picture? → Deep Dive (€49)"
- Upsell-Link: `/check?dao=[slug]&tier=deep_dive` (Deep Dive vorausgewählt)
- Gleicher Footer wie bestehende Reports
- Branding: "ChainSights | Identity-first Web3 Analytics"

---

## 5. Custom Events (Analytics)

### Neue Events

| Event Name | Trigger | Properties |
|------------|---------|------------|
| `score_card_cta_view` | CTA wird sichtbar nach Result | `dao_name`, `gvs_score`, `category` |
| `score_card_email_entered` | User gibt Email ein | `dao_name`, `email_domain` |
| `score_card_requested` | User klickt Send | `dao_name`, `gvs_score`, `marketing_consent` |
| `score_card_email_sent` | Email erfolgreich versendet (server-side) | `dao_name`, `gvs_score` |
| `score_card_link_clicked` | User klickt Download-Link in Email | `dao_name`, `token` |
| `score_card_download_complete` | PDF erfolgreich heruntergeladen | `dao_name`, `gvs_score` |
| `score_card_error` | Fehler bei Generierung oder Versand | `dao_name`, `error_type` |

### Event Flow

```
check_result_shown
     │
     ▼
score_card_cta_view
     │
     ▼
score_card_email_entered
     │
     ▼
score_card_requested
     │
     ▼
score_card_email_sent          (server-side)
     │
     ▼
score_card_link_clicked        (token-basiert messbar)
     │
     ▼
score_card_download_complete
     │
     ▼
(Optional: cta_click auf Deep Dive / Full Audit)
```

---

## 6. Erfolgsmetriken

| KPI | Aktuell | Target | Messung |
|-----|---------|--------|---------|
| Free Check Completion | 20% | > 50% | `check_result_shown` / `check_page_view` |
| Score Card CTA View → Email | — | > 30% | `score_card_email_entered` / `score_card_cta_view` |
| Email Sent → Link Clicked | — | > 60% | `score_card_link_clicked` / `score_card_email_sent` |
| Score Card → Paid Upgrade (30d) | — | > 5% | `checkout_complete` mit matching Email |
| Email Leads / Woche | 1 | > 10 | Count `score_card_email_entered` |

---

## 7. Copy / Texte

### CTA Box (auf Result-Seite)

**Headline:** "Get Your Governance Score Card — Free"

**Bullet Points:**
- "Official score + category ranking"
- "Benchmark vs. 46 DAOs"
- "Ready to share with your team"

**Button:** "Send Score Card"

**Placeholder:** "your@email.com"

**Checkbox:** "Send me occasional product updates. Unsubscribe anytime."

**Legal:** "By requesting, you agree to our Privacy Policy."

**Alternative Share-Angles (A/B Test, nicht V1):**
- "Perfect for governance proposals & reports"
- "Show your community how you rank"

### Success State (nach Submit)

**Headline:** "Your Score Card is on its way! 📊"

**Subtext:** "Check your inbox — it should arrive within a minute."

**Upsell:** "Want AI-powered recommendations? Get your Deep Dive →"

### Score Card Email

**Subject:** `Your Governance Score Card: [DAO Name] — [Score]/100`

**Body:**
```
Hi,

Your Governance Score Card for [DAO Name] is ready.

[Download Score Card] (Button)

Link expires in 30 days.

─────────────────────────

Want the full picture?

Get your Deep Dive Report for [DAO Name]:
• AI-powered governance analysis
• Specific recommendations
• Historical trends

[Get Deep Dive — €49] (Button → /check?dao=[slug]&tier=deep_dive)

─────────────────────────

ChainSights | Identity-first Web3 Analytics
```

---

## 8. Edge Cases

| Case | Handling |
|------|----------|
| Ungueltige Email | Inline Error: "Please enter a valid email" |
| Email bereits in MMS | MMS gibt existierende `party_id` zurueck (`created: false`), Score Card trotzdem generieren |
| PDF Generierung fehlgeschlagen | Error State + Retry Button auf Frontend |
| Email Versand fehlgeschlagen | Error State + Retry Button, Error Event loggen |
| User ohne JS | Graceful Degradation (CTA nicht sichtbar) |
| Open Universe DAO (nicht in 46) | Score Card generieren, Rank = "Not ranked (Open Universe)", Benchmark = Overall Average |
| MMS API nicht erreichbar | Fallback: Score Card trotzdem generieren + senden, MMS-Sync spaeter nachholen |

---

## 9. Nicht in Scope (V1)

- [ ] Email-Sequenz nach Download (Follow-up Marketing)
- [ ] A/B Test verschiedener CTA-Texte
- [ ] Score Card mit Custom Branding fuer Enterprise
- [ ] Mehrere Score Cards ohne erneute Email-Eingabe
- [ ] Account-Erstellung nach Email

---

## 10. Offene Fragen

- [x] Email-Pflicht? → **Nur beim Score Card Download, NICHT beim Free Check**
- [x] Exklusiver PDF-Wert? → **Rank + Benchmark-Balken + Offizielles Format**
- [x] Copy? → **"Score Card" + Share-Angle**
- [x] PDF Styling? → **ChainSights Brand Colors, konsistent mit bestehenden Reports**
- [x] Rate Limiting? → **V1 ohne Limit, beobachten + iterieren**
- [x] PDF Hosting? → **Link per Email (wie bestehende Reports)**
- [x] Free Check Email-Feld entfernen? → **Separates Ticket, vorgezogen, Solo-Dev**
- [x] Lead-Storage? → **MMS API (POST /v1/parties, source_project: chainsights)**
- [x] Consent-Storage? → **MMS (zentral)**
- [x] Consent-Checkbox? → **Getrennt: Score Card Delivery (kein Opt-in) + Marketing (Opt-in)**
- [x] Email-Service? → **Resend (bereits integriert)**
- [x] DB-Architektur? → **Bestehende reports-Tabelle mit Tier score_card erweitern**
- [ ] MMS Consent-Modul: Wann fertig? (Fallback: marketing_consent lokal in reports)

---

## 11. DSGVO / GDPR Compliance

### 11.1 Cookie Consent Banner (Site-wide)

**Status:** Fehlt aktuell auf chainsights.one

**Anforderung:** Cookie Consent Banner fuer alle Besucher

**Grund:**
- Analytics Tracking (Vercel Analytics, Custom Events) = nicht-essentielle Cookies
- DSGVO erfordert Consent VOR Tracking

**Umsetzung:**
- Cookie Banner bei erstem Besuch
- Optionen: "Accept All" / "Only Essential" / "Manage Preferences"
- Tracking erst NACH Consent starten
- Consent-Entscheidung speichern (Cookie oder localStorage)

### 11.2 Email Consent (Score Card Download)

**Consent-Trennung:**

| Zweck | Typ | Opt-in noetig? |
|-------|-----|----------------|
| Score Card Delivery | Vertragserfullung | Nein (Email-Eingabe = Anforderung) |
| Product Updates / Marketing | Marketing | Ja (separate Checkbox, NICHT pre-checked) |

**Umsetzung im Formular:**

```
┌─────────────────────────────────────┐
│                                     │
│  📊 Get Your Governance Score Card │
│     — Free                          │
│                                     │
│  ┌─────────────────────────────────┐│
│  │ your@email.com                  ││
│  └─────────────────────────────────┘│
│                                     │
│  ☐ Send me occasional product     │
│    updates. Unsubscribe anytime.   │
│                                     │
│  By requesting, you agree to our   │
│  Privacy Policy.                   │
│                                     │
│  ┌─────────────────────────────────┐│
│  │       Send Score Card           ││
│  └─────────────────────────────────┘│
│                                     │
└─────────────────────────────────────┘
```

**Pflicht-Elemente:**
- Checkbox NUR fuer Marketing-Emails (NICHT pre-checked!)
- Link zu Privacy Policy
- "Unsubscribe anytime" erwaehnen
- Consent wird zentral im MMS gespeichert

### 11.3 Opt-Out / Unsubscribe

**Anforderung:** Jede Marketing-Email muss Unsubscribe-Link haben

**Umsetzung:**
- Unsubscribe-Link in Email Footer
- Link fuehrt zu `/unsubscribe?token=[unique_token]`
- MMS API: Consent widerrufen
- Bestaetigungsseite: "You've been unsubscribed"

### 11.4 Privacy Policy Update

**Anforderung:** Privacy Policy muss erwaehnen:
- Welche Daten gesammelt werden (Email, DAO, Score)
- Zweck (Score Card Delivery, ggf. Product Updates)
- Speicherdauer
- Rechte (Auskunft, Loeschung, Widerruf)
- Kontakt fuer Datenschutz-Anfragen
- Datenverarbeiter: masem IT (MMS API)

**Status:** Pruefen ob aktuelle Privacy Policy das abdeckt

### 11.5 Prioritaet

| Item | Prioritaet | Grund |
|------|-----------|-------|
| Cookie Consent Banner | P0 | Rechtlich noetig, betrifft alle Besucher |
| Email Consent Checkbox | P0 | Muss bei Score Card Launch da sein |
| Privacy Policy Update | P0 | Muss vor Go-Live aktualisiert sein |
| Unsubscribe Flow | P1 | Noetig sobald erste Marketing-Emails rausgehen |

---

## 12. Dependencies

- **Resend** — bereits integriert (`hello@chainsights.one`)
- **MMS API** — Parties/Contacts live, Consent in Arbeit
- **@react-pdf/renderer** — muss installiert werden
- **Bestehende Infrastruktur** — Download-Tokens, reports-Tabelle, Resend Templates
- **Cookie Consent Library** — muss ausgewaehlt + integriert werden
- **Privacy Policy Update** — Text muss vor Go-Live angepasst werden

---

## 13. Voraussetzungen & Reihenfolge

### Prerequisite (separates Ticket, Solo-Dev)
1. **Free Check Email-Feld entfernen** — Email-Step aus Free Check Flow nehmen, Result direkt anzeigen

### Implementation Reihenfolge
1. Cookie Consent Banner (kann unabhaengig deployed werden)
2. Privacy Policy Update
3. Score Card Feature:
   a. Backend: API Route, MMS Integration, PDF Generation, Resend Email
   b. Frontend: CTA Box, Email Form, Consent Checkbox, Success State
   c. Analytics Events
4. Unsubscribe Flow (vor ersten Marketing-Emails)
