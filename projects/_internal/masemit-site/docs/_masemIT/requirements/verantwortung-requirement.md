# Requirement: Verantwortungsseite masem.at + MMS Donations API

**Autor:** Claude (Co-Founder masemIT)  
**Datum:** 2026-02-02  
**Betrifft:** masem.at, MMS (api.masem.at)  
**Status:** Draft

---

## 1. Kontext & Motivation

masemIT spendet auf zwei Wegen:

- **Direkte Spenden** über hoki.help (100% der Spenden fließen an HoKi NÖ)
- **Gewinnbeteiligung** – ein konfigurierbarer Prozentsatz des Nettoumsatzes von tellingCube und ChainSights wird an ehrenamtliche Projekte gespendet

Aktuell werden Spenden in jedem Projekt in eigenen `donations`-Tabellen geführt. Diese Logik soll in MMS (api.masem.at) zentralisiert werden. Auf masem.at soll eine Verantwortungsseite die Gesamtspenden transparent darstellen.

### Grundprinzip

> Kein Marketing. Kein Greenwashing. Keine CTAs.  
> Wer es finden will, findet es. Wer draufklickt, klickt drauf.

---

## 2. Verantwortungsseite auf masem.at

### 2.1 Navigation & Platzierung

- Link **"Verantwortung"** im Header (gleichwertig mit anderen Nav-Items)
- Zusätzlich im Footer
- **Kein** CTA-Styling, kein Badge, kein Highlight – normaler Navigationslink
- Kein Spendenaufruf auf der Seite selbst

### 2.2 Seiteninhalt

#### Einleitung
Kurzer, ehrlicher Text im Ton von:
> *"Teile unseres Gewinns fließen an ehrenamtliche Projekte. Nicht als Marketing-Maßnahme, sondern weil es uns wichtig ist."*

Optional: 2-3 Sätze zur persönlichen Motivation (HoKi NÖ, ehrenamtliche Begleitung von Familien mit schwer erkrankten Kindern). Authentisch, nicht poliert.

#### Spendenübersicht (live aus MMS API)
Pro Spendenziel (aktuell nur HoKi NÖ, erweiterbar):

| Datenfeld | Beschreibung |
|-----------|-------------|
| Name des Spendenziels | z.B. "HoKi NÖ – Kinderhospizteam" |
| Kurzbeschreibung | Was die Organisation macht |
| Externer Link | z.B. hospiz-noe.at |
| Gesamtbetrag gespendet | Summe aller Spenden (alle Quellen) |
| Letztes Update | Zeitpunkt der letzten Aktualisierung |

**Kein Breakdown** nach Quelle (hoki.help vs. tellingCube vs. ChainSights) – das wäre zu promotional. Nur der Gesamtbetrag pro Spendenziel.

#### Hinweis auf hoki.help
Dezenter Hinweis:
> *"Auch Sie können direkt spenden: [hoki.help](https://hoki.help)"*

### 2.3 Design-Richtlinien

- Ruhig, clean, content-focused
- masemIT Brand Colors (Primary #093236, Accent #009BB1 nur sparsam)
- Keine Spendenbalken, Fortschrittsanzeigen oder Gamification
- Responsive, aber kein Schnickschnack

---

## 3. MMS Donations API (api.masem.at)

### 3.1 Datenmodell (Konzept)

```
donation_targets
├── id (uuid)
├── name (string) – z.B. "HoKi NÖ"
├── description (text)
├── external_url (string)
├── is_active (boolean)
├── created_at / updated_at

donations
├── id (uuid)
├── target_id (fk → donation_targets)
├── source_project (enum: hoki_help, tellingcube, chainsights, manual)
├── amount_cents (integer)
├── currency (string, default: EUR)
├── donation_type (enum: direct, revenue_share)
├── reference (string, nullable) – z.B. Stripe Payment ID
├── donated_at (timestamp)
├── created_at / updated_at

donation_config
├── id (uuid)
├── source_project (enum)
├── target_id (fk → donation_targets)
├── percentage (decimal) – Prozentsatz vom Nettoumsatz
├── is_active (boolean)
├── valid_from / valid_until
```

### 3.2 API Endpoints für masem.at (Public, read-only)

Diese Endpoints benötigt die Verantwortungsseite:

```
GET /v1/donations/public/summary
```
Liefert aggregierte Spendendaten pro Spendenziel.

**Response:**
```json
{
  "data": [
    {
      "target": {
        "id": "uuid",
        "name": "HoKi NÖ – Kinderhospizteam",
        "description": "Begleitung von Familien mit schwer erkrankten Kindern und Jugendlichen",
        "external_url": "https://www.hospiz-noe.at/projekte/hospizteam-fuer-jugendliche/"
      },
      "total_donated_cents": 11000,
      "currency": "EUR",
      "last_donation_at": "2026-01-28T14:30:00Z",
      "donation_count": 15
    }
  ],
  "updated_at": "2026-02-02T10:00:00Z"
}
```

### 3.3 API Endpoints für interne Projekte (Authenticated)

Diese Endpoints brauchen hoki.help, tellingCube und ChainSights:

```
POST   /v1/donations                  – Spende erfassen
GET    /v1/donations                  – Spenden auflisten (mit Filtern)
GET    /v1/donations/:id              – Einzelne Spende
GET    /v1/donations/summary          – Detailliertes Summary (inkl. Aufschlüsselung nach source)

GET    /v1/donation-targets           – Alle Spendenziele
POST   /v1/donation-targets           – Spendenziel anlegen
PATCH  /v1/donation-targets/:id       – Spendenziel aktualisieren

GET    /v1/donation-config            – Konfiguration pro Projekt
PUT    /v1/donation-config/:project   – Prozentsatz etc. setzen
```

**POST /v1/donations – Request Body:**
```json
{
  "target_id": "uuid",
  "source_project": "hoki_help",
  "amount_cents": 2500,
  "currency": "EUR",
  "donation_type": "direct",
  "reference": "stripe_pi_xxx",
  "donated_at": "2026-02-01T12:00:00Z"
}
```

### 3.4 Auth & Sicherheit

- Public endpoint (`/public/summary`): Kein Auth, rate-limited, cached (z.B. 15min via Upstash)
- Interne Endpoints: API-Key pro Projekt (einfach zum Start, später ggf. JWT über MMS Auth-Layer)

---

## 4. Migrationspfad

1. **MMS Donations Service** aufsetzen (DB-Schema, Endpoints)
2. **Bestehende Daten** aus hoki.help, tellingCube, ChainSights migrieren
3. **Projekte umstellen** – eigene donations-Tabellen durch MMS API Calls ersetzen
4. **masem.at Verantwortungsseite** bauen, public endpoint anbinden
5. **Alte donations-Tabellen** in den Projekten deprecaten/entfernen

---

## 5. Offene Fragen

- [ ] Soll die Verantwortungsseite auch die Spendenhistorie (Timeline) zeigen, oder nur Gesamtbeträge?
- [ ] Brauchen wir eine Admin-UI in MMS für Donation Targets, oder reicht API + DB?
- [ ] Soll der konfigurierbare Prozentsatz auf der Verantwortungsseite sichtbar sein? (z.B. "3% unseres Nettoumsatzes")
- [ ] Caching-Strategie: Reicht 15min für den public endpoint?
