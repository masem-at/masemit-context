# 🎨 UX REFACTOR: Free Check — Von Modal-Dschungel zu Flow-Page

**Project:** ChainSights
**Priority:** 🟡 MEDIUM — UX Debt + Conversion Impact
**Reporter:** Mario Semper (PO)
**Date:** 2026-02-01
**Depends on:** Free Check Open-Universe (parallel, nicht blockierend)
**Affects:** Conversion Rate, Mobile UX, Report-Kaufprozess

---

## Problem

Der aktuelle Free Check öffnet je nach Flow (Free / Paid / Admin) **4-5 aufeinanderfolgende Modals:**

1. DAO-Auswahl (Modal oder Dropdown)
2. Report-Typ-Auswahl (Modal)
3. Email-Eingabe (Modal)
4. Stripe Payment (Modal, nur bei Paid)
5. Ergebnis / Bestätigung (Modal)

### Warum das ein Problem ist

1. **Modal-Fatigue:** Jedes Popup ist ein Moment wo der User denkt "was will die Seite jetzt schon wieder?" — Conversion-Killer
2. **Mobile Horror:** Modals auf Mobile sind unzuverlässig (Scroll-Lock, Keyboard-Overlap, Back-Button-Verhalten)
3. **Kein Kontext:** User sieht nie den Gesamtprozess — "wie viele Steps kommen noch?"
4. **Conditional Chaos:** Free vs. Paid vs. Admin haben unterschiedliche Modal-Ketten → fragile Code-Logik
5. **Inkompatibel mit Open-Universe:** Der geplante on-the-fly Loading-State (3-20 Sekunden) funktioniert nicht gut in einem Modal

---

## Lösung: Eigene Flow-Page `/check`

Alle Modals werden durch **eine dedizierte Seite** ersetzt, die den gesamten Prozess als vertikalen Flow mit Progressive Disclosure zeigt.

### Design-Prinzip

- **Alles auf einer Seite** — User sieht den Gesamtprozess
- **Progressive Disclosure** — Steps erscheinen erst wenn der vorherige abgeschlossen ist
- **Conditional Rendering** — Stripe-Element nur bei Paid sichtbar, kein Step-Skipping
- **Mobile-First** — Vertikaler Flow funktioniert nativ auf allen Geräten

### Wireframe

```
chainsights.one/check

┌──────────────────────────────────────────────────┐
│                                                  │
│  Check Your DAO's Governance Health              │
│  "Wallets lie. We don't."                        │
│                                                  │
├──────────────────────────────────────────────────┤
│                                                  │
│  ① Select Your DAO                               │
│  ┌────────────────────────────────────────────┐  │
│  │ [Search any Snapshot space...]        🔍  │  │
│  └────────────────────────────────────────────┘  │
│  Autocomplete mit bekannten DAOs                 │
│  Freie Eingabe für beliebige Snapshot Spaces     │
│                                                  │
│  [Continue →]                                    │
│                                                  │
├──── ↓ nach DAO-Auswahl ─────────────────────────┤
│                                                  │
│  ② Choose Your Report                            │
│                                                  │
│  ┌────────────┐ ┌────────────┐ ┌────────────┐   │
│  │   FREE     │ │ DEEP DIVE  │ │ GOV AUDIT  │   │
│  │            │ │            │ │            │   │
│  │  Quick     │ │  Detailed  │ │  Full      │   │
│  │  Check     │ │  Analysis  │ │  Audit     │   │
│  │            │ │            │ │            │   │
│  │  ✓ GVS    │ │  ✓ GVS    │ │  ✓ GVS    │   │
│  │  ✓ 4 KPIs │ │  ✓ 4 KPIs │ │  ✓ 4 KPIs │   │
│  │            │ │  ✓ Trends  │ │  ✓ Trends  │   │
│  │            │ │  ✓ Risks   │ │  ✓ Risks   │   │
│  │            │ │            │ │  ✓ Roadmap │   │
│  │            │ │            │ │  ✓ AI Rec. │   │
│  │            │ │            │ │            │   │
│  │   Free     │ │    €49     │ │    €149    │   │
│  │  ─────     │ │  ──────    │ │  ───────   │   │
│  │  🟢 grün   │ │  🟡 gelb   │ │  🔴 rot    │   │
│  └────────────┘ └────────────┘ └────────────┘   │
│                                                  │
│  ▸ Compare report types (expandable)             │
│                                                  │
├──── ↓ nach Report-Auswahl ──────────────────────┤
│                                                  │
│  ③ Your Details                                  │
│                                                  │
│  Email: [_______________________________]        │
│  ☐ Send me weekly governance updates             │
│                                                  │
│  ┌─── nur bei Deep Dive / Gov Audit: ────────┐  │
│  │                                            │  │
│  │  💳 Payment                                │  │
│  │  ┌──────────────────────────────────────┐  │  │
│  │  │  [Stripe Payment Element]            │  │  │
│  │  └──────────────────────────────────────┘  │  │
│  │                                            │  │
│  └────────────────────────────────────────────┘  │
│                                                  │
│  [Get Your Report →]                             │
│                                                  │
├──── ↓ nach Submit ──────────────────────────────┤
│                                                  │
│  ④ Your Results                                  │
│                                                  │
│  ┌────────────────────────────────────────────┐  │
│  │                                            │  │
│  │  [DAO Name]          GVS: 7.2 / 10        │  │
│  │                      Status: Stable        │  │
│  │                                            │  │
│  │  HPR  ████████░░  8.0                      │  │
│  │  DEI  ██████░░░░  6.0                      │  │
│  │  PDI  ███████░░░  7.0                      │  │
│  │  GPI  ████████░░  7.8                      │  │
│  │                                            │  │
│  │  [⚠️ Low Confidence — wenn < 20 Proposals] │  │
│  │                                            │  │
│  └────────────────────────────────────────────┘  │
│                                                  │
│  Bei FREE:                                       │
│  "Want deeper insights? Upgrade to Deep Dive"    │
│  [Upgrade → €49]                                 │
│                                                  │
│  Bei PAID:                                       │
│  "Your report is being generated.                │
│   You'll receive it at [email] within            │
│   15 minutes."                                   │
│  [Download Preview PDF]                          │
│                                                  │
└──────────────────────────────────────────────────┘
```

---

## Verhaltens-Matrix

| Szenario | Step ① | Step ② | Step ③ | Step ④ |
|----------|--------|--------|--------|--------|
| **Free Check (bekannter DAO)** | DAO auswählen | "Free" wählen | Email + Checkbox | GVS sofort (< 200ms) |
| **Free Check (unbekannter DAO)** | DAO eingeben | "Free" wählen | Email + Checkbox | Loading-State → GVS (3-20s) |
| **Deep Dive** | DAO auswählen | "Deep Dive" wählen | Email + Stripe (€49) | GVS + "Report kommt per Email" |
| **Gov Audit** | DAO auswählen | "Gov Audit" wählen | Email + Stripe (€149) | GVS + "Report kommt per Email" |
| **Admin (@masem.at)** | DAO auswählen | Alle sichtbar, kein Preis | Email (pre-filled) | GVS sofort, kein Stripe |
| **DAO < 5 Proposals** | DAO eingeben | — (Step ② nicht erreichbar) | — | "Not enough data" + Notify Me |

---

## Progressive Disclosure: Details

### Step-Transitions

Jeder Step hat drei Zustände:

```
LOCKED    → Ausgegraut, nicht interagierbar
ACTIVE    → Aktueller Step, volle Interaktion
COMPLETED → Zusammengefasst, editierbar (Klick zum Ändern)
```

**Beispiel nach Step ② abgeschlossen:**

```
✅ ① Lido (lido-snapshot.eth)           [Change]
✅ ② Deep Dive — €49                    [Change]
→  ③ Your Details                       ← AKTIV
🔒 ④ Your Results
```

### Scroll-Verhalten

- Nach Step-Completion: Smooth-Scroll zum nächsten Step
- Completed Steps bleiben sichtbar (User kann zurückscrollen)
- Mobile: Step-Header sticky am Top bei langem Content

### Loading-State in Step ④ (Open-Universe)

Für unbekannte DAOs (on-the-fly Berechnung):

```
④ Your Results

   ✅ Validating Snapshot space...
   ✅ Fetching governance proposals...
   ⏳ Analyzing voting patterns...
   ○  Calculating governance score...

   [Fortschrittsbalken ~60%]
   
   "First-time analysis — this may take up to 30 seconds"
```

Passt perfekt in den Flow — bei einem Modal wäre dieser State extrem awkward.

---

## URL-Strategie

| URL | Verhalten |
|-----|-----------|
| `/check` | Leere Seite, Start bei Step ① |
| `/check?dao=lido-snapshot.eth` | Step ① pre-filled, direkt zu Step ② |
| `/check?dao=lido-snapshot.eth&type=free` | Step ① + ② pre-filled, direkt zu Step ③ |

**Vorteile:**
- Deeplinks aus Marketing-Posts: "Check Lido's score → chainsights.one/check?dao=lido-snapshot.eth"
- Tracking: UTM-Parameter funktionieren normal
- Sharing: User kann Link teilen

---

## Farbcodierung Report-Typen

| Typ | Farbe | Hex (Vorschlag) | Begründung |
|-----|-------|-----------------|------------|
| Free / Quick Check | 🟢 Grün | #22C55E | Kostenlos, niedrige Schwelle |
| Deep Dive | 🟡 Gelb/Orange | #F59E0B | Mittleres Investment |
| Gov Audit | 🔴 Rot/Premium | #EF4444 | Premium, höchster Wert |

Angelehnt an Marios Skizze. Die Farben kommunizieren intuitiv die Eskalation.

---

## Migration: Was passiert mit den bestehenden Modals?

| Bestehendes Element | Aktion |
|---------------------|--------|
| DAO-Auswahl Modal | ❌ Entfernen → Step ① auf `/check` |
| Report-Typ Modal | ❌ Entfernen → Step ② auf `/check` |
| Email-Gate Modal | ❌ Entfernen → Step ③ auf `/check` |
| Stripe Checkout Modal | ❌ Entfernen → Stripe Element inline in Step ③ |
| Ergebnis-Modal | ❌ Entfernen → Step ④ auf `/check` |
| CTA-Buttons auf Landing/Rankings | 🔄 Ändern → Link zu `/check` (ggf. mit Query-Params) |
| Rankings "Quick Check" Button | 🔄 Ändern → Link zu `/check?dao=[space]&type=free` |

**Kein paralleler Betrieb:** Sobald `/check` live ist, werden alle Modals entfernt. Kein "manche User sehen Modals, andere die Page."

---

## Acceptance Criteria

- [ ] Neue Route `/check` existiert und ist von Landing Page + Rankings erreichbar
- [ ] Step ① erlaubt freie Eingabe UND Autocomplete für bekannte DAOs
- [ ] Step ② zeigt drei Report-Typen mit Farbcodierung und Feature-Vergleich
- [ ] Step ③ zeigt Email + Checkbox für alle; Stripe nur bei Paid-Reports
- [ ] Step ④ zeigt GVS + 4 Komponenten (Free) oder Bestätigung (Paid)
- [ ] Progressive Disclosure: Steps erscheinen sequentiell
- [ ] Completed Steps sind zusammengefasst + editierbar ("Change")
- [ ] URL-Parameter funktionieren (dao, type)
- [ ] Loading-State für on-the-fly DAOs funktioniert in Step ④
- [ ] Confidence-Badge bei < 20 Proposals sichtbar
- [ ] < 5 Proposals: "Not enough data" + Notify Me Option
- [ ] Mobile: Responsive, kein Modal-Overflow, Keyboard-safe
- [ ] Alle bestehenden Modals entfernt
- [ ] CTA-Buttons auf Landing + Rankings verlinken auf `/check`
- [ ] Admin-Flow (@masem.at): Stripe übersprungen, Email pre-filled
- [ ] Kein Regression: Bestehende Free Check Funktionalität bleibt erhalten

---

## Abhängigkeiten

| Ticket | Beziehung | Blockiert? |
|--------|-----------|------------|
| Free Check Open-Universe | Backend (GVS on-the-fly) | ❌ Nicht blockiert — Page kann mit bestehenden 21 DAOs launchen, Open-Universe wird nachgerüstet |
| Stripe Integration | Bestehendes Stripe Payment Element | ❌ Nicht blockiert — Element wird nur von Modal nach Inline verschoben |
| Admin UI (Story 1.4) | Admin-Bypass-Logik | ❌ Nicht blockiert — bestehende @masem.at Logik wird übernommen |

**Empfehlung:** Kann parallel zum Open-Universe Backend entwickelt werden. Frontend baut `/check` Page mit bestehender DAO-Liste, Backend liefert Open-Universe nach → Frontend erweitert Step ① um freie Eingabe.

---

## Geschätzter Aufwand

| Task | Effort |
|------|--------|
| Page Route + Layout (`/check`) | 2h |
| Step ① — DAO Input mit Autocomplete | 3h |
| Step ② — Report-Typ Cards mit Farbcodierung | 2h |
| Step ③ — Email + conditional Stripe Element | 3h |
| Step ④ — Ergebnis-Anzeige + Loading-State | 3h |
| Progressive Disclosure Logic (Step-States) | 2h |
| URL-Parameter Handling (Deep-Links) | 1h |
| Migration: Alle Modal-Referenzen entfernen | 2h |
| Migration: CTA-Buttons auf `/check` umleiten | 1h |
| Mobile Responsiveness + Testing | 2h |
| E2E Testing (Free/Deep/Audit/Admin Flows) | 3h |
| **Total** | **~24h** |

---

## Auswirkung auf andere Features

| Feature | Auswirkung |
|---------|------------|
| **Open-Universe Free Check** | Loading-State integriert sich natürlich in Step ④ |
| **Marketing / Promotion** | Deep-Links ermöglichen gezielte CTAs pro DAO |
| **Conversion Rate** | Erwartung: +20-30% durch transparenten Prozess |
| **Mobile Users** | Erwartung: signifikante Verbesserung der Completion Rate |
| **Rankings Page** | "Quick Check" Button → `/check?dao=[space]&type=free` |
| **Landing Page** | Hero CTA → `/check` |
