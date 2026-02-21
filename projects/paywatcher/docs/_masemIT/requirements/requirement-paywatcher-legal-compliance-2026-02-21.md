# Requirement: Legal & Compliance Pages + Consent Banner

**Project:** PayWatcher (masemIT Product)
**Type:** Change Request (CR-PW-002)
**Date:** 2026-02-21
**Author:** Mario Semper (PO)
**Status:** Approved
**Priority:** High — blockiert Go-Live
**Betrifft:** Landing Page, Dashboard, Admin — alle Bereiche

---

## 1. Kontext

PayWatcher ist feature-complete (alle 27 Stories done, Epic 1–6). Vor dem öffentlichen Go-Live fehlen die gesetzlich vorgeschriebenen Legal Pages und ein Cookie/Tracking Consent Banner. Ohne diese ist ein Launch in Österreich/EU nicht DSGVO-konform.

**Gesetzliche Grundlage:**
- §5 ECG (Impressumspflicht für österreichische Websites)
- DSGVO Art. 13/14 (Informationspflichten bei Datenerhebung)
- DSGVO Art. 6 (Rechtsgrundlage für Verarbeitung)
- DSGVO Art. 7 (Einwilligung für nicht-essenzielle Cookies/Tracking)
- ePrivacy-Richtlinie (Cookie-Consent)

---

## 2. Deliverables

### 2.1 Impressum (Pflicht — §5 ECG)

**Route:** `/legal/imprint` (oder `/imprint`)

**Inhalt:**

```
masemIT e.U.
Mario Semper
Alleegasse 26
3851 Kautzen
Austria

Firmenbuchnummer: FN 661236g
UID-Nr: ATU82330407
E-Mail: contact@masem.at
Website: https://masem.at
```

**Zusätzlich:**
- Unternehmensgegenstand: IT-Dienstleistungen und Softwareentwicklung
- Zuständige Behörde: Bezirkshauptmannschaft Waidhofen an der Thaya
- Mitglied der WKO Niederösterreich
- Anwendbare Rechtsvorschriften: Gewerbeordnung (www.ris.bka.gv.at)
- Verbraucherstreitbeilegung: Link auf https://ec.europa.eu/consumers/odr

**Design:** Einfache Text-Seite, konsistent mit PayWatcher Dark Mode Theme. Keine besondere Gestaltung nötig.

---

### 2.2 Datenschutzerklärung / Privacy Policy (Pflicht — DSGVO)

**Route:** `/legal/privacy` (oder `/privacy`)

**Sprache:** Englisch (Primary Target Audience = internationale Devs). Optionaler Hinweis: "Eine deutsche Fassung ist auf Anfrage erhältlich."

**Abschnitte:**

| # | Abschnitt | Inhalt |
|---|-----------|--------|
| 1 | **Verantwortlicher** | masemIT e.U., Mario Semper, Kontaktdaten (wie Impressum) |
| 2 | **Welche Daten wir verarbeiten** | Siehe Tabelle unten |
| 3 | **Rechtsgrundlagen** | Art. 6(1)(a) Einwilligung (Analytics), Art. 6(1)(b) Vertragserfüllung (Login, API), Art. 6(1)(f) berechtigte Interessen (Security Logs) |
| 4 | **Cookies & Tracking** | Essenzielle Cookies (Auth Session), Analytics (masemIT Analytics — nur mit Consent) |
| 5 | **Drittanbieter / Sub-Processors** | Vercel (Hosting, USA — EU SCCs), Neon (DB, EU-Region Frankfurt), Resend (Email, USA — EU SCCs), QStash/Upstash (Queue/Cache) |
| 6 | **Datenweitergabe** | Keine Weitergabe an Dritte außer den genannten Sub-Processors. Kein Verkauf von Daten. |
| 7 | **Speicherdauer** | Auth-Tokens: Session-Dauer. Request Access Daten: bis Onboarding abgeschlossen oder 6 Monate. Payment-Daten: gemäß gesetzlicher Aufbewahrungsfristen (7 Jahre AT). Analytics: 26 Monate. |
| 8 | **Deine Rechte** | Auskunft, Berichtigung, Löschung, Einschränkung, Datenübertragbarkeit, Widerspruch, Beschwerde bei der Datenschutzbehörde (dsb.gv.at) |
| 9 | **Internationale Transfers** | USA-Transfers über EU Standard Contractual Clauses (Vercel, Resend) |
| 10 | **Änderungen** | Letzte Aktualisierung + Hinweis dass Änderungen auf dieser Seite veröffentlicht werden |

**Verarbeitete Daten (Detailtabelle für den Texter/Dev):**

| Datentyp | Wann | Rechtsgrundlage | Speicherdauer |
|----------|------|-----------------|---------------|
| E-Mail-Adresse (Request Access) | Formular auf Landing Page | Art. 6(1)(b) vorvertragliche Maßnahme | Bis Onboarding oder 6 Monate |
| Name, Company, Use Case (Request Access) | Formular auf Landing Page | Art. 6(1)(b) vorvertragliche Maßnahme | Bis Onboarding oder 6 Monate |
| E-Mail-Adresse (Login) | Magic Link Login | Art. 6(1)(b) Vertragserfüllung | Dauer der Geschäftsbeziehung |
| Auth Session Token (Cookie) | Nach Login | Art. 6(1)(b) Vertragserfüllung | Session-Dauer (max. 30 Tage) |
| Tenant API Key (verschlüsselt) | Nach Onboarding | Art. 6(1)(b) Vertragserfüllung | Dauer der Geschäftsbeziehung |
| Payment-Daten (Amount, Status, txHash) | API-Nutzung | Art. 6(1)(b) Vertragserfüllung | 7 Jahre (AT Aufbewahrungspflicht) |
| Webhook URLs | Settings-Konfiguration | Art. 6(1)(b) Vertragserfüllung | Dauer der Geschäftsbeziehung |
| IP-Adresse, Browser-Info | Automatisch bei Seitenaufruf | Art. 6(1)(f) berechtigtes Interesse (Security) | 30 Tage (Server Logs) |
| Analytics-Events (anonymisiert) | Mit Consent | Art. 6(1)(a) Einwilligung | 26 Monate |

---

### 2.3 Terms of Service / Nutzungsbedingungen (Empfohlen)

**Route:** `/legal/terms` (oder `/terms`)

**Sprache:** Englisch

**Abschnitte:**

| # | Abschnitt | Kerninhalt |
|---|-----------|------------|
| 1 | **Service Description** | PayWatcher ist ein Verification Layer für Stablecoin-Zahlungen. Kein Payment Processor, kein Custodian, kein Money Transmitter. Wir bewegen kein Geld. |
| 2 | **Eligibility** | Mindestalter 18, geschäftsfähig. B2B-Service. |
| 3 | **Account & API Keys** | Managed Onboarding. API Key ist vertraulich, User ist verantwortlich für sichere Aufbewahrung. Missbrauch = Deaktivierung. |
| 4 | **Acceptable Use** | Keine illegalen Aktivitäten, keine Geldwäsche, kein Terrorismusfinanzierung, keine Sanctions-Evasion, kein Betrug. masemIT kann Accounts ohne Vorwarnung sperren bei Verdacht. |
| 5 | **Service Level** | "As-is" im MVP. Kein SLA garantiert. Best-effort Availability. Geplante Wartungen werden angekündigt. |
| 6 | **Fees & Billing** | Pricing gemäß aktueller Pricing Page. Flat-Fee pro Verification. Keine versteckten Kosten. Änderungen mit 30 Tagen Vorlauf. |
| 7 | **Limitation of Liability** | PayWatcher verifiziert Zahlungen, garantiert aber nicht die Korrektheit der Blockchain-Daten. Keine Haftung für: verpasste Zahlungen, Reorgs, Smart Contract Bugs, Verlust von Funds. Haftung begrenzt auf die in den letzten 12 Monaten gezahlten Fees. |
| 8 | **Intellectual Property** | API, Docs, Dashboard = masemIT IP. User behält Rechte an eigenen Daten. |
| 9 | **Termination** | Beide Seiten können jederzeit kündigen. Bei Kündigung: API Key wird deaktiviert, Daten gemäß Aufbewahrungsfristen gelöscht. |
| 10 | **Governing Law** | Österreichisches Recht, Gerichtsstand Waidhofen an der Thaya (für B2B). Für Konsumenten: gesetzlicher Gerichtsstand. |
| 11 | **Changes** | Änderungen mit 30 Tagen Vorlauf per Email. Weiternutzung = Zustimmung. |
| 12 | **Contact** | contact@masem.at |

**Wichtige Abgrenzung (muss prominent platziert sein):**

> PayWatcher is a payment **verification** service, not a payment **processor**. We do not hold, transfer, or have custody of any funds. We monitor public blockchain data and notify you when a payment matching your criteria has been confirmed. You remain solely responsible for your funds, wallets, and private keys.

---

### 2.4 Consent Banner (Pflicht — ePrivacy + DSGVO)

**Implementierung:** Cookie/Consent Banner Component + MMS Consents API Integration

**Backend:** MMS Consents API (api.masem.at) — bereits live.
**API Docs:** https://api.masem.at/docs/consents

#### Verhalten

| Szenario | Aktion |
|----------|--------|
| Erster Besuch | Banner anzeigen (Bottom oder Bottom-Left) |
| "Accept All" | Consent lokal + in MMS gespeichert, Analytics aktiviert, Banner verschwindet |
| "Essential Only" / "Reject" | Consent lokal + in MMS gespeichert (granted: false), kein Analytics, Banner verschwindet |
| "Settings" / "Customize" | Optionaler Detail-View mit Kategorien (Essential, Analytics) |
| Wiederkehrender Besuch | Banner nicht zeigen wenn Consent lokal gespeichert. Consent-Änderung über Link im Footer ("Cookie Settings") |
| Consent-Änderung | Neuer MMS Consent Record (immutable audit trail), localStorage aktualisiert |

#### Cookie-Kategorien

| Kategorie | Cookies/Tracking | Required? | Default | MMS consent_type |
|-----------|-----------------|-----------|---------|------------------|
| **Essential** | Auth Session (NextAuth), CSRF Token, Consent-Preference, device_id | Ja — immer aktiv | Immer an | — (kein MMS-Record nötig) |
| **Analytics** | masemIT Analytics (analytics.masem.at) | Nein — nur mit Consent | Aus | `cookie_analytics` |

#### Technische Umsetzung

**Dual-Storage: localStorage (sofort) + MMS Consents API (audit trail)**

**1. localStorage (sofortige UI-Reaktion):**
- Key: `pw_consent` → `{ essential: true, analytics: false, timestamp: "2026-02-21T..." }`
- Key: `pw_device_id` → UUID (einmalig generiert, persistent)
- Consent-Status wird sofort aus localStorage gelesen — kein API-Call für UI-Entscheidung

**2. MMS Consents API (DSGVO-konformer Audit Trail):**
- Bei jeder Consent-Entscheidung (Grant oder Revoke): POST /v1/consents
- Immutable Records — jede Änderung erzeugt einen neuen Record (DSGVO-Anforderung)
- Fire-and-forget: UI wartet nicht auf API-Response

**Consent-Flow (Detail):**

```
1. User besucht paywatcher.dev zum ersten Mal
2. Consent Banner erscheint
3. User klickt "Accept All" oder "Essential Only"
4. Frontend:
   a) Generiert device_id (UUID) falls noch nicht vorhanden → localStorage pw_device_id
   b) Speichert Consent in localStorage → pw_consent
   c) Aktiviert/deaktiviert Analytics-Script sofort basierend auf Entscheidung
   d) Sendet POST an Route Handler /api/consent (fire-and-forget):
      {
        "device_id": "abc-123-...",
        "consent_type": "cookie_analytics",
        "granted": true/false,
        "consent_text_version": "v1.0"
      }
5. Route Handler /api/consent:
   a) Liest IP + User-Agent aus Request Headers
   b) Prüft ob User eingeloggt ist (auth() Session)
      - Eingeloggt: party_id aus Session verwenden
      - Nicht eingeloggt: nur device_id
   c) Ruft POST /v1/consents auf MMS auf:
      {
        "party_id": "uuid" (optional, nur wenn eingeloggt),
        "device_id": "abc-123-...",
        "consent_type": "cookie_analytics",
        "scope": "project",
        "project": "paywatcher",
        "granted": true/false,
        "ip_address": "...",
        "user_agent": "...",
        "consent_text_version": "v1.0"
      }
```

**Consent-Änderung (Cookie Settings):**
- Gleicher Flow — neuer POST /v1/consents Record in MMS (alter bleibt bestehen = Audit Trail)
- localStorage wird überschrieben

**Login-Verknüpfung:**
- Wenn ein User sich einloggt und bereits eine device_id hat, kann ein zusätzlicher Consent-Record mit party_id + device_id geschrieben werden
- Damit sind anonyme (pre-login) und authentifizierte (post-login) Consents verknüpfbar

#### Route Handler

```
app/api/consent/route.ts    # POST → MMS POST /v1/consents (mit PayWatcher API Key, Scope: consents:write)
```

**Auth:** PayWatcher braucht einen API Key mit Scope `consents:write` für den MMS-Call. Dieser Key kann der bestehende PayWatcher-Admin-Key sein oder ein separater Key. → Als ENV Variable: `MMS_CONSENT_API_KEY` (oder den bestehenden Admin-Key verwenden falls Scopes passen).

#### Analytics-Script Conditional Loading

```tsx
// In Root Layout oder Consent Provider:
// Analytics-Script wird NUR gerendert wenn Consent gegeben
{isAnalyticsConsented && (
  <Script
    src="https://analytics.masem.at/tracker.js"
    strategy="afterInteractive"
    data-project="paywatcher"
  />
)}
```

**Wichtig:** Das Script-Tag komplett nicht rendern wenn kein Consent — nicht nur Events unterdrücken. Der Tracker selbst sendet bereits beim Laden einen PageView.

#### Design

- Konsistent mit PayWatcher Dark Mode Theme
- Nicht-aufdringlich aber klar sichtbar
- Keine Dark Patterns (z.B. "Accept" Button größer als "Reject")
- Beide Optionen gleich prominent

**Hinweis:** Wenn das Projekt bereits eine ConsentBanner-Komponente aus anderen masemIT-Projekten (ChainSights, hoki.help) hat, kann diese als Basis verwendet und angepasst werden.

---

### 2.5 Footer-Links & Navigation

**Bestehender Footer muss ergänzt werden um:**

| Link | Route | Bereich |
|------|-------|---------|
| Imprint | `/legal/imprint` | Legal |
| Privacy Policy | `/legal/privacy` | Legal |
| Terms of Service | `/legal/terms` | Legal |
| Cookie Settings | Re-opens Consent Banner | Legal |

**Placement:** Neue "Legal" Spalte im Footer, neben bestehenden Links (Docs, GitHub, masemIT).

**Auch im Dashboard-Footer** (falls vorhanden) oder als Links in der Sidebar unter "Legal".

---

## 3. Architektur-Hinweise

### Routing

```
app/(landing)/
├── legal/
│   ├── imprint/page.tsx      # Impressum
│   ├── privacy/page.tsx      # Datenschutzerklärung
│   └── terms/page.tsx        # Terms of Service

app/api/
├── consent/route.ts          # POST → MMS POST /v1/consents (fire-and-forget, consents:write)
```

Alle Legal Pages sind **public** (kein Auth-Check), **SSG** (statischer Content), und SEO-indexierbar.
Der Consent Route Handler ist **public** (kein Auth-Check nötig — device_id reicht), aber rate-limited.

### Shared Components

```
components/shared/
├── consent-banner.tsx        # Cookie Consent Banner UI
└── consent-provider.tsx      # React Context: Consent-Status (localStorage + conditional Script rendering)
```

### Environment Variables (neu)

| Variable | Zweck |
|----------|-------|
| `MMS_CONSENT_API_KEY` | API Key mit Scope `consents:write` für MMS Consent-Records. Alternativ: bestehenden Admin-Key verwenden falls Scopes passen. |

### Analytics-Integration

Das bestehende Analytics-Script in `app/layout.tsx` muss **conditional** werden:

```tsx
// VORHER (immer geladen):
<Script src="https://analytics.masem.at/tracker.js" strategy="afterInteractive" data-project="paywatcher" />

// NACHHER (nur mit Consent):
// ConsentProvider liest localStorage pw_consent
// Rendert Script-Tag nur wenn analytics: true
<ConsentProvider>
  {/* Script-Tag wird nur bei analytics consent gerendert */}
</ConsentProvider>
```

Die bestehende `lib/analytics.ts` (Custom Events via `masemit.track()`) muss ebenfalls den Consent-Check machen:

```typescript
export function trackEvent(name: string, properties?: Record<string, string>) {
  if (typeof window === 'undefined') return
  if (!isAnalyticsConsented()) return        // ← NEU: Consent-Check
  if (!window.masemit?.isActive()) return     // ← Owner-Exclusion (besteht bereits)
  window.masemit.track(name, properties)
}
```

---

## 4. Content-Erstellung

Die **Texte** für Impressum, Privacy Policy und Terms of Service sollen vom BMAD-Team (Paige oder John) als Content erstellt werden. Die oben definierten Abschnitte und Tabellen dienen als Struktur-Vorgabe.

**Stilrichtlinien:**
- Englisch, klar und verständlich (kein Legalese wo vermeidbar)
- Developer-Audience: direkt, transparent, keine Marketing-Sprache
- Impressum: zusätzlich auf Deutsch (gesetzlich sicherer für AT)
- Ton: professionell aber zugänglich — wie Stripe's oder Resend's Legal Pages

---

## 5. Akzeptanzkriterien

- [ ] Impressum-Seite erreichbar unter `/legal/imprint` mit allen §5-ECG-Pflichtangaben
- [ ] Privacy Policy erreichbar unter `/legal/privacy` mit allen DSGVO-Pflichtinformationen
- [ ] Terms of Service erreichbar unter `/legal/terms` mit Service-Abgrenzung (Verification ≠ Custody)
- [ ] Consent Banner erscheint bei erstem Besuch auf allen Seiten (Landing, Dashboard, Admin)
- [ ] "Accept All" aktiviert Analytics, "Essential Only" deaktiviert Analytics
- [ ] Analytics-Script (`tracker.js`) wird NUR als Script-Tag gerendert wenn Analytics-Consent gegeben
- [ ] Custom Events (`masemit.track()`) prüfen Consent bevor sie senden
- [ ] Consent-Entscheidung wird in localStorage gespeichert (sofortige UI-Reaktion)
- [ ] Consent-Entscheidung wird via POST /api/consent → MMS POST /v1/consents persistiert (Audit Trail)
- [ ] MMS Consent-Record enthält: device_id, consent_type, granted, ip_address, user_agent, consent_text_version
- [ ] Eingeloggte User: MMS Consent-Record enthält zusätzlich party_id
- [ ] Consent-Änderung erzeugt neuen MMS-Record (immutable, kein Update)
- [ ] Footer enthält Links zu allen drei Legal Pages + "Cookie Settings"
- [ ] "Cookie Settings" im Footer öffnet Consent Banner erneut
- [ ] Alle Legal Pages sind Dark-Mode-kompatibel
- [ ] Alle Legal Pages sind SSG (kein API-Call, kein Auth)
- [ ] Consent Banner hat keine Dark Patterns (beide Optionen gleich prominent)
- [ ] Route Handler /api/consent ist rate-limited

---

## 6. Empfohlene Epic-Struktur

| Story | Titel | Aufwand |
|-------|-------|---------|
| 7-1 | Legal Pages Layout + Impressum | Klein |
| 7-2 | Privacy Policy (Content + Page) | Mittel |
| 7-3 | Terms of Service (Content + Page) | Mittel |
| 7-4 | Consent Banner + MMS Consents API + Analytics Conditional Loading | Mittel-Groß |
| 7-5 | Footer-Links + Navigation Update | Klein |

**Geschätzter Gesamtaufwand:** 2–3 Tage

**Story 7-4 Detail (größte Story):**
- ConsentProvider (React Context + localStorage)
- ConsentBanner UI Component
- Route Handler `/api/consent` → MMS POST /v1/consents
- Analytics Script conditional rendering (Layout ändern)
- `lib/analytics.ts` Consent-Check für Custom Events
- device_id Generation + Persistenz

---

## 7. Referenzdokumente

| Dokument | Relevanz |
|----------|----------|
| product-brief-paywatcher-phase1-frontend-v2-2026-02-17.md | Tech Stack, Analytics Events, Domain |
| architecture.md | Route-Struktur, Analytics Pattern, Component Boundaries |
| company-info.md | Firmendaten für Impressum |
| https://api.masem.at/docs/consents | MMS Consents API — Endpoints, Enums, Consent-Immutability |
| https://analytics.masem.at/docs/tracker-integration.md | Analytics Tracker — Script-Integration, Custom Events, Owner Exclusion |
| DSGVO Art. 13/14 | Pflichtinformationen Privacy Policy |
| §5 ECG | Impressumspflicht |

---

*Dieses Requirement (CR-PW-002) ergänzt die bestehende Phase 1 um die fehlenden Legal & Compliance Pages. Es blockiert den öffentlichen Go-Live von paywatcher.dev.*
