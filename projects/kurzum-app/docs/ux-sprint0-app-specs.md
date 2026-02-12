# UX Specification: Sprint 0 — App Prototype

**Author:** Sally (UX Designer), BMAD Party Mode
**Date:** 2026-02-12
**Status:** Draft — pending team approval
**Base Document:** `_bmad-output/planning-artifacts/ux-design-specification.md` (2026-02-10)
**Architecture:** `docs/architecture-sprint0-supplement.md`
**Personas:** Markus (Monteur, 34), Stefan (Meister, 48)

---

## 1. Design Principles for Sprint 0

Carried forward from the base UX spec — these are non-negotiable:

1. **Relief over innovation** — every screen feels like taking a weight off
2. **Familiar beats innovative** — WhatsApp, Camera App mental models
3. **Built for dirty hands** — 48px touch targets, high contrast, voice-first
4. **Subtraction is the luxury** — fewer elements than expected
5. **Errors are adjustments, not failures** — calm pragmatism

**Sprint 0 addition:**
6. **Prototype ≠ ugly** — the prototype should feel intentional and polished in the core flow, even if sparse. First impressions with pilot testers matter.

---

## 2. Login Flow

### User Story
> Stefan erhält eine E-Mail von Mario (Pilot-Einladung). Er klickt den Link, landet auf kurzum.app/login, tippt seine E-Mail ein, bekommt einen Magic Link, klickt ihn — und ist drin. Kein Passwort, kein "Konto erstellen", kein OAuth-Dialog.

### States

#### State 1: Login Form (`/login`)

```
┌─────────────────────────────────┐
│                                 │
│         kurzum.               │
│                                 │
│     Sprich statt tipp.          │
│                                 │
│  ┌───────────────────────────┐  │
│  │  E-Mail-Adresse           │  │
│  │  stefan@elektro-huber.at  │  │
│  └───────────────────────────┘  │
│                                 │
│  [  🔒 Turnstile  ]            │
│                                 │
│  ┌───────────────────────────┐  │
│  │    Login-Link senden      │  │
│  └───────────────────────────┘  │
│                                 │
│  Kein Passwort nötig.           │
│  Wir senden dir einen           │
│  Einmal-Link per E-Mail.        │
│                                 │
└─────────────────────────────────┘
```

**Details:**
- Logo: `kurzum.` mit orangem Punkt (gleich wie Landing Page)
- Tagline: "Sprich statt tipp." — Wiedererkennung von der Landing Page
- E-Mail-Feld: großes Input, autofocus, type="email", autocomplete="email"
- Button: Voll-Breite, Brand-Accent (Orange), disabled bis E-Mail valide
- Hint-Text unter dem Button: Erklärt das Konzept für Erstbesucher
- **Kein "Registrieren"-Link** — Magic Link erstellt den Account automatisch
- Turnstile: Invisible mode bevorzugt, managed widget als Fallback

#### State 2: Check Your Email (`/login/check-email`)

```
┌─────────────────────────────────┐
│                                 │
│         kurzum.               │
│                                 │
│           ✉️                     │
│                                 │
│     Prüfe dein Postfach!        │
│                                 │
│  Wir haben einen Login-Link     │
│  an stefan@elektro-huber.at     │
│  gesendet.                      │
│                                 │
│  Der Link ist 15 Minuten        │
│  gültig.                        │
│                                 │
│  ┌───────────────────────────┐  │
│  │   Neue E-Mail anfordern   │  │
│  └───────────────────────────┘  │
│                                 │
│  Keine E-Mail? Schau im         │
│  Spam-Ordner nach.              │
│                                 │
└─────────────────────────────────┘
```

**Details:**
- E-Mail-Adresse anzeigen (damit klar ist wohin gesendet wurde)
- "Neue E-Mail anfordern" Button: Ghost/Outline-Style, rate-limited (disabled für 60s nach Klick)
- Spam-Hinweis: Dezent, aber sichtbar
- Kein Auto-Redirect — User navigiert aktiv zum E-Mail-Client

#### State 3: Token Verification (kein eigener Screen)

- User klickt Magic Link in E-Mail → `/api/auth/verify?token=xxx`
- **Erfolg:** Session-Cookie gesetzt, Redirect → `/app`
- **Abgelaufen/Ungültig:** Redirect → `/login?error=expired`

#### State 4: Error States auf Login-Seite

| Error | Anzeige | Verhalten |
|-------|---------|-----------|
| Token abgelaufen | Banner: "Dieser Link ist abgelaufen. Fordere einen neuen an." | E-Mail-Feld vorausgefüllt wenn möglich |
| Token ungültig | Banner: "Dieser Link ist ungültig." | Zurück zum Login-Formular |
| Rate Limit | Banner: "Zu viele Anfragen. Bitte warte ein paar Minuten." | Button disabled |
| Server-Fehler | Banner: "Etwas ist schiefgelaufen. Versuch es nochmal." | Retry möglich |

**Banner-Design:** Muted, nicht alarmierend. Warme Farbe (Amber/Orange-Variante), kein aggressives Rot. Tonalität: "Kein Problem, versuch's nochmal" — nicht "FEHLER!"

#### State 5: Already Logged In

- User besucht `/login` mit gültiger Session → automatischer Redirect zu `/app`
- Kein "Du bist bereits eingeloggt"-Screen nötig

---

## 3. App Shell & Navigation

### Layout (`/app` Route Group)

```
┌─────────────────────────────────┐
│  kurzum.          Stefan  [↪]  │  ← Header (sticky)
├─────────────────────────────────┤
│                                 │
│         [Page Content]          │
│                                 │
│                                 │
│                                 │
├─────────────────────────────────┤
│  [🎙️ Aufnahme]   [📋 Übersicht] │  ← Bottom Nav (mobile, sticky)
└─────────────────────────────────┘
```

**Header:**
- Logo links: `kurzum.` (Link zu /app)
- Rechts: Vorname des Users + Logout-Icon (↪ oder "Abmelden")
- Kein Hamburger-Menü, kein Dropdown — es gibt nur 2 Seiten
- Höhe: kompakt (~56px), mehr Platz für Content

**Bottom Navigation (Mobile):**
- 2 Tabs: 🎙️ Aufnahme | 📋 Übersicht
- Active Tab: Orange Unterstrich + gefülltes Icon
- Inactive: Muted Farbe + Outline Icon
- Sticky am unteren Bildschirmrand
- Touch-Target: volle Breite pro Tab, min. 48px Höhe

**Desktop (≥768px):**
- Bottom Nav verschwindet
- Navigation in Header integriert (2 Links neben Logo)
- Content zentriert, max-width ~720px

**Warum nur 2 Tabs:**
Sprint 0 hat genau 2 Funktionen: Aufnehmen und Ansehen. Mehr gibt es nicht. Mehr zeigen wir nicht. "Where's the rest?" — das IST das Design.

---

## 4. Voice Recording Page (`/app/record` oder `/app`)

### User Story
> Markus steht auf der Baustelle. Er hat gerade den FI-Schalter im OG getauscht. Er öffnet kurzum, drückt den großen Button, spricht 20 Sekunden, drückt nochmal — fertig. Zurück zur Arbeit.

### 6 States des Recording-Flows

#### State A: Idle (Startscreen)

```
┌─────────────────────────────────┐
│  kurzum.          Stefan  [↪]  │
├─────────────────────────────────┤
│                                 │
│                                 │
│                                 │
│        Was gibt's Neues?        │
│                                 │
│          ┌─────────┐            │
│          │         │            │
│          │   🎙️    │            │  ← 80x80px, Orange
│          │         │            │
│          └─────────┘            │
│       Zum Aufnehmen tippen      │
│                                 │
│                                 │
├─────────────────────────────────┤
│  [🎙️ Aufnahme]   [📋 Übersicht] │
└─────────────────────────────────┘
```

**Details:**
- Mikrofon-Button: **80x80px**, rund, Brand-Accent (Orange), zentriert
- Schatten/Elevation für 3D-Gefühl — soll sich "drückbar" anfühlen
- Text darunter: "Zum Aufnehmen tippen" — verschwindet nach erster Nutzung
- Restlicher Screen: bewusst leer. Reduktion. Einladung.
- **First Visit:** Zusätzlich eine kurze Zeile: "Sprich einfach los — die KI kümmert sich um den Rest."

#### State B: Permission Prompt (nur beim ersten Mal)

```
┌─────────────────────────────────┐
│                                 │
│  ┌───────────────────────────┐  │
│  │  kurzum.app möchte auf   │  │
│  │  dein Mikrofon zugreifen  │  │
│  │                           │  │
│  │  [Blockieren] [Erlauben]  │  │
│  └───────────────────────────┘  │
│                                 │
│  Damit du Sprachnachrichten     │
│  aufnehmen kannst, braucht      │
│  kurzum Zugriff auf dein        │
│  Mikrofon.                      │
│                                 │
└─────────────────────────────────┘
```

**Details:**
- Browser-native Permission Dialog — kann nicht gestyled werden
- **Darunter** (im App-UI): Erklärender Text warum Mikrofon nötig
- Falls abgelehnt: → State F (Permission Denied Error)

#### State C: Recording (aktiv)

```
┌─────────────────────────────────┐
│  kurzum.          Stefan  [↪]  │
├─────────────────────────────────┤
│                                 │
│                                 │
│          🔴  0:12               │  ← Pulsierende rote Anzeige + Timer
│                                 │
│    ▓▓▓▓▓▓▒▒░░░░░░░░░░░░        │  ← Live Audio-Pegel (animiert)
│                                 │
│          ┌─────────┐            │
│          │         │            │
│          │   ⏹️    │            │  ← 80x80px, jetzt Rot
│          │         │            │
│          └─────────┘            │
│       Zum Stoppen tippen        │
│                                 │
│                                 │
│  Max. 5 Min                     │  ← Dezent, Muted
├─────────────────────────────────┤
│  [🎙️ Aufnahme]   [📋 Übersicht] │
└─────────────────────────────────┘
```

**Details:**
- Button wechselt zu Stopp-Symbol (⏹), Farbe wird Rot
- Pulsierende rote Anzeige (CSS animation) links neben Timer
- Audio-Pegel: Einfacher Balken der auf Lautstärke reagiert (AnalyserNode)
- Timer: Zählt hoch (0:00, 0:01, ...), mono-spaced Font
- Max. 5 Minuten: Automatischer Stopp bei 5:00
- **Bottom Nav bleibt sichtbar** — User kann abbrechen indem er woanders hintippt
- Hintergrund: leicht getönter Hintergrund (sehr subtil) um den "Recording-Modus" zu signalisieren

#### State D: Preview (Aufnahme fertig, noch nicht gesendet)

```
┌─────────────────────────────────┐
│  kurzum.          Stefan  [↪]  │
├─────────────────────────────────┤
│                                 │
│      Aufnahme fertig (0:23)     │
│                                 │
│  ┌───────────────────────────┐  │
│  │  ▶  ▓▓▓▓▓▓▓░░░░░  0:23  │  │  ← Waveform + Play/Pause
│  └───────────────────────────┘  │
│                                 │
│                                 │
│  ┌──────────┐  ┌──────────────┐ │
│  │ Verwerfen │  │   Senden  ▸ │ │
│  └──────────┘  └──────────────┘ │
│                                 │
│  (nochmal aufnehmen)            │  ← Text-Link, kein Button
│                                 │
├─────────────────────────────────┤
│  [🎙️ Aufnahme]   [📋 Übersicht] │
└─────────────────────────────────┘
```

**Details:**
- Waveform-Darstellung der Aufnahme (statisch, aus AudioBuffer generiert)
- Play/Pause Button zum Anhören vor dem Senden
- **"Senden"**: Primärer Button (Orange, voll), prominent
- **"Verwerfen"**: Sekundärer Button (Outline/Ghost), weniger prominent
- **"Nochmal aufnehmen"**: Text-Link, tertiäre Aktion
- Keine Textfelder, keine Tags, keine Projekt-Zuordnung — die KI macht das

**Design-Entscheidung: Preview überspringbar?**
Für Sprint 0: Preview immer anzeigen. Gibt dem User Kontrolle und Vertrauen. In späteren Sprints: Option "Direkt senden" als Einstellung für erfahrene User.

#### State E: Processing (Upload + KI-Verarbeitung)

```
┌─────────────────────────────────┐
│  kurzum.          Stefan  [↪]  │
├─────────────────────────────────┤
│                                 │
│                                 │
│       KI verarbeitet...         │
│                                 │
│    ┌─────────────────────┐      │
│    │ ✅ Hochgeladen       │      │
│    │ ⏳ Wird transkribiert│      │  ← Schritt-Anzeige
│    │ ○  Zusammenfassung   │      │
│    └─────────────────────┘      │
│                                 │
│    ░░░░░░░░░░░░░░░░░░░░░        │  ← Progress bar (indeterminate)
│                                 │
│    Das dauert wenige Sekunden.  │
│                                 │
│                                 │
├─────────────────────────────────┤
│  [🎙️ Aufnahme]   [📋 Übersicht] │
└─────────────────────────────────┘
```

**Details:**
- **3-Schritte-Anzeige**: Hochladen → Transkription → Zusammenfassung
  - ✅ = erledigt (grün)
  - ⏳ = aktiv (animiert/orange)
  - ○ = ausstehend (muted)
- Progress bar: Indeterminate (da wir die genaue Dauer nicht kennen)
- Beruhigender Text: "Das dauert wenige Sekunden." — setzt Erwartung
- **User kann wegnavigieren** (zu Übersicht) — die Nachricht erscheint dort wenn fertig
- Erwartete Dauer: 6-15 Sekunden gesamt

**Warum Schritte statt nur Spinner?**
Transparenz baut Vertrauen. "Die KI macht gerade etwas Konkretes" fühlt sich besser an als ein leerer Spinner. Und es beantwortet die Frage "Dauert das noch lang?" ohne dass der User fragen muss.

#### State F: Error States

| Fehler | Screen | Aktion |
|--------|--------|--------|
| **Kein Mikrofon** | "Kein Mikrofon gefunden. Bitte prüfe deine Geräte-Einstellungen." | Link zu Browser-Einstellungen (wenn möglich) |
| **Permission denied** | "Mikrofon-Zugriff nicht erlaubt. Aktiviere ihn in den Browser-Einstellungen." | Anleitung-Link (je nach Browser) |
| **Upload fehlgeschlagen** | "Upload fehlgeschlagen. Deine Aufnahme ist gespeichert." + [Nochmal versuchen] | Aufnahme bleibt lokal, Retry-Button |
| **STT fehlgeschlagen** | "Die Spracherkennung hat nicht funktioniert. Deine Aufnahme wurde trotzdem gespeichert." | Transkript bleibt leer, User kann in Übersicht manuell ergänzen (Sprint 1+) |
| **LLM fehlgeschlagen** | "Die Zusammenfassung konnte nicht erstellt werden." + Transkript wird angezeigt | Transkript als Fallback anzeigen |

**Design-Prinzip für Fehler:**
- Warme Farben (Amber), nicht aggressives Rot
- Immer einen Ausweg bieten (Retry, Alternative, Kontakt)
- Nie technische Details ("500 Internal Server Error")
- Tonalität: "Kein Problem, hier ist was du tun kannst"

---

## 5. Dashboard / Übersicht (`/app`)

### User Story
> Stefan öffnet morgens kurzum. Er sieht sofort: 4 neue Nachrichten seit gestern. Markus war auf der Müller-Baustelle, FI-Schalter getauscht, braucht Material. Huber Gartenweg: Zählerkasten fertig. Alles auf einen Blick, ohne einen einzigen Anruf.

### Dashboard-Layout

```
┌─────────────────────────────────┐
│  kurzum.          Stefan  [↪]  │
├─────────────────────────────────┤
│                                 │
│  Heute (2)                      │  ← Zeitgruppe
│                                 │
│  ┌───────────────────────────┐  │
│  │ 🎙️ Markus · vor 2 Std    │  │
│  │                           │  │
│  │ FI-Schalter im OG         │  │  ← Summary-Headline (status)
│  │ getauscht ✅                │  │
│  │                           │  │
│  │ 📦 3x Sicherungsautomat   │  │  ← Material (wenn vorhanden)
│  │    B16, 1x FI 40/0,03    │  │
│  │                           │  │
│  │ 📍 Müller, Hauptstr. 5    │  │  ← Projekt
│  └───────────────────────────┘  │
│                                 │
│  ┌───────────────────────────┐  │
│  │ 🎙️ Markus · vor 5 Std    │  │
│  │                           │  │
│  │ Zählerkasten EG            │  │
│  │ fertig ✅                   │  │
│  │                           │  │
│  │ 📍 Huber, Gartenweg 12    │  │
│  └───────────────────────────┘  │
│                                 │
│  Gestern (1)                    │
│                                 │
│  ┌───────────────────────────┐  │
│  │ 🎙️ Markus · gestern 16:30│  │
│  │                           │  │
│  │ ⏳ KI verarbeitet...       │  │  ← Falls noch Processing
│  └───────────────────────────┘  │
│                                 │
├─────────────────────────────────┤
│  [🎙️ Aufnahme]   [📋 Übersicht] │
└─────────────────────────────────┘
```

### Summary Card — Anatomie

```
┌─────────────────────────────────────┐
│ 🎙️ {Username} · {relative Zeit}     │  ← Header
│                                     │
│ {summary.status} ✅                  │  ← Was wurde erledigt
│                                     │
│ 📦 {summary.material}               │  ← Material (nur wenn vorhanden)
│                                     │
│ ⚠️ {summary.problems}               │  ← Probleme (nur wenn vorhanden, Orange)
│                                     │
│ → {summary.nextSteps}               │  ← Nächste Schritte (nur wenn vorhanden)
│                                     │
│ 📍 {summary.project}                │  ← Erkanntes Projekt
└─────────────────────────────────────┘
```

**Regeln:**
- **Nur gefüllte Felder anzeigen.** Wenn kein Material → kein Material-Block. Kein "Material: —" oder "Material: Keins".
- **Probleme visuell hervorheben:** Orange/Amber Hintergrund-Tint wenn `problems` vorhanden. Stefan sieht sofort: "Da stimmt was nicht."
- **Relative Zeitangaben:** "vor 2 Std", "gestern 16:30", "Mo 09:15". Nie ISO-Timestamps.
- **Projekt fallback:** Wenn kein Projekt erkannt → "Nicht zugeordnet" in muted Farbe

### Card Expansion (Tap)

Tap auf eine Card zeigt zusätzlich:
- **Vollständiges Transkript** (aufklappbar, collapsed by default)
- **Audio-Player** (Waveform + Play/Pause, wenn Audio noch verfügbar)
- **Metadaten:** Dauer der Aufnahme, STT-Provider, Verarbeitungszeit (für FFG-Dokumentation, klein/muted)

```
┌─────────────────────────────────────┐
│ 🎙️ Markus · vor 2 Std               │
│                                     │
│ FI-Schalter im OG getauscht ✅       │
│ 📦 3x Sicherungsautomat B16, ...    │
│ 📍 Müller, Hauptstraße 5            │
│                                     │
│ ─── Transkript ─────────────────    │
│ "Jo, also ich bin jetzt fertig      │
│  beim Müller in der Hauptstraße.    │
│  FI-Schalter im OG hab ich         │
│  getauscht, der hat eh schon        │
│  wieder funktioniert..."            │
│                                     │
│ ▶ ▓▓▓▓▓▓░░░░░ 0:23                 │  ← Audio Player
│                                     │
│ Aufnahme: 23s · Voxtral · 4.2s     │  ← Metadaten (muted, klein)
└─────────────────────────────────────┘
```

### Filter & Sortierung

- **Default:** Neueste zuerst, gruppiert nach Tag (Heute / Gestern / Datum)
- **Sprint 0:** Keine Filter-UI. Nur chronologisch.
- **Sprint 1+:** Filter: Alle / Heute / Diese Woche + Projekt-Filter

### Processing State in Dashboard

Nachrichten die noch verarbeitet werden:
```
┌─────────────────────────────────────┐
│ 🎙️ Markus · gerade eben             │
│                                     │
│ ⏳ KI verarbeitet...                 │  ← Animated Spinner
│ ░░░░░░░░░░░░░░░░                    │  ← Mini Progress
└─────────────────────────────────────┘
```

In Sprint 0 (synchrone Verarbeitung) wird das selten vorkommen — die Nachricht kommt bereits mit Ergebnis zurück. Aber der State muss für Edge Cases existieren (z.B. Tab-Wechsel während Processing).

---

## 6. Empty State (Erster Login)

```
┌─────────────────────────────────┐
│  kurzum.          Stefan  [↪]  │
├─────────────────────────────────┤
│                                 │
│                                 │
│           🎙️                    │
│                                 │
│    Noch keine Nachrichten.      │
│                                 │
│    Sprich deine erste           │
│    Nachricht und schau was      │
│    die KI daraus macht.         │
│                                 │
│  ┌───────────────────────────┐  │
│  │   Erste Aufnahme starten  │  │  ← CTA Button (Orange)
│  └───────────────────────────┘  │
│                                 │
│                                 │
├─────────────────────────────────┤
│  [🎙️ Aufnahme]   [📋 Übersicht] │
└─────────────────────────────────┘
```

**Details:**
- Freundlich, einladend, nicht "leer"
- CTA führt direkt zur Aufnahme-Seite
- **Kein Onboarding-Wizard**, kein Tutorial, kein "Schritt 1 von 5"
- Nach der ersten Nachricht: Empty State verschwindet, Dashboard zeigt die Zusammenfassung
- Das IS das Onboarding: "Probier's aus und sieh was passiert"

---

## 7. Responsive Breakpoints

| Breakpoint | Layout-Anpassung |
|------------|-----------------|
| **< 640px (Mobile, primary)** | Bottom Nav, volle Breite Cards, großer Record-Button (80px) |
| **640–768px (Tablet)** | Bottom Nav, etwas mehr Padding, Cards mit max-width |
| **≥ 768px (Desktop)** | Header-Navigation statt Bottom Nav, Content zentriert max-width 720px, Record-Button kleiner (64px) |

---

## 8. Color Usage im App-Bereich

Bestehende Design Tokens aus der Landing Page übernehmen:

| Element | Farbe | Token |
|---------|-------|-------|
| Record-Button (idle) | Orange | `--color-accent` / brand-accent |
| Record-Button (recording) | Rot | `--color-destructive` |
| Success-Checkmarks | Grün | `--color-brand-success` |
| Probleme-Highlight | Amber/Orange | `--color-warning` (neu, weiche Orange-Variante) |
| Processing/Loading | Orange | `--color-accent` |
| Muted Text | Stone 500 | `--color-muted-foreground` |
| Card Background | White | `--color-card` |
| Page Background | Stone 50 | `--color-background` |
| Error Banners | Amber (warm) | Nicht: aggressives Rot |

---

## 9. Accessibility (Field Conditions)

| Anforderung | Implementation |
|-------------|---------------|
| Touch Targets | ≥ 48x48px überall, Record-Button 80x80px |
| Kontrast | WCAG AA (4.5:1) auf allen Text-Elementen |
| Sonnenlicht | Hoher Kontrast zwischen Card-Background und Text |
| Handschuhe | Große Buttons, keine präzisen Gesten (kein Drag, kein Long-Press) |
| Screen Reader | ARIA labels auf allen interaktiven Elementen, Recording-State als live region |
| Reduced Motion | `prefers-reduced-motion`: Pulsing/Animationen deaktivieren |

**Tap-to-Record statt Hold-to-Record:**
Entscheidung für Sprint 0: **Tap** zum Starten, **Tap** zum Stoppen. Hold-to-Record (WhatsApp-Style) ist mit Handschuhen schwierig und fehleranfällig. Kann als Alternative in Sprint 1+ evaluiert werden.

---

## 10. Magic Link E-Mail Template

```
┌─────────────────────────────────────┐
│                                     │
│         kurzum.                   │
│                                     │
│  Hallo Stefan,                      │
│                                     │
│  hier ist dein Login-Link für       │
│  kurzum.app:                        │
│                                     │
│  ┌───────────────────────────────┐  │
│  │       Jetzt einloggen         │  │  ← Orange Button
│  └───────────────────────────────┘  │
│                                     │
│  Der Link ist 15 Minuten gültig     │
│  und kann nur einmal verwendet      │
│  werden.                            │
│                                     │
│  Falls du keinen Login angefordert  │
│  hast, ignoriere diese E-Mail.      │
│                                     │
│  ─────────────────────────────────  │
│  © 2026 masemIT e.U. — kurzum.app   │
│                                     │
└─────────────────────────────────────┘
```

**Details:**
- Gleiche visuelle Sprache wie Confirmation-Email (bestehend in `lib/email.ts`)
- Inline CSS, kein @react-email
- `escapeHtml()` für alle dynamischen Inhalte
- Subject: "kurzum. — Dein Login-Link"
- From: "Mario von kurzum <notification@kurzum.app>"

---

## 11. Offene Fragen an das Team

1. **KI-Transparenz-Hinweis (FR38):** Wo genau zeigen wir "Diese Zusammenfassung wurde von KI erstellt"? Vorschlag: Dezenter Hinweis in der erweiterten Card-Ansicht, nicht auf jeder Card. EU AI Act verlangt Transparenz, aber nicht prominente Labels auf jeder Interaktion.

2. **Audio-Consent (DSGVO):** Beim ersten Login "Ich stimme der Verarbeitung meiner Sprachnachrichten zu" — Modal-Dialog oder Inline auf der Login-Page? Vorschlag: Einmaliger Inline-Banner beim ersten App-Zugriff, mit "Verstanden"-Button. Kein Blocker-Modal.
