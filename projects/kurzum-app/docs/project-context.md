---
project_name: 'kurzum-app'
domain: 'kurzum.app'
owner: 'Mario Semper (masemIT e.U.)'
date_created: '2026-02-08'
status: 'pre-development (UX + Architecture complete, Epics/Stories next)'
---

# Project Context: kurzum.app

## What is kurzum.app?

KI-gestützte, DSGVO-konforme Kommunikations-App für Handwerksbetriebe mit Außendienst im DACH-Raum. Voice-First: Monteure sprechen statt tippen, KI fasst zusammen und ordnet zu.

**Einzeiler:** "WhatsApp-Einfachheit + DSGVO-Konformität + KI-Intelligenz — für Handwerker."

## The Problem

- 62% der Handwerksbetriebe nutzen WhatsApp geschäftlich — DSGVO-illegal
- €15-20 Mrd./Jahr Verlust durch Kommunikationsineffizienz in der DACH-Baubranche
- ~€5.000/Monat pro Kleinbetrieb durch Nacharbeit, Informationsverlust, Dokumentationslücken
- 87% der ~708.000 DACH-Handwerksbetriebe nutzen KEINE dedizierte Kommunikationslösung

## Target Customer

**Primär:** Elektrotechniker + SHK-Installateure (Sanitär, Heizung, Klima) in Österreich
- Betriebsgröße: 5-50 Mitarbeiter
- Hoher Außendienstanteil, technisch affin, starke Innungsstruktur
- Entscheider: Meister/Inhaber, entscheidet allein in 1-4 Wochen

**Expansion:** Weitere Gewerke (Maler, Dachdecker, Zimmerer) → DE → CH

## Core Value Proposition

| Was | Wie |
|-----|-----|
| **Voice-First** | Sprachnachricht → KI-Zusammenfassung → automatische Projekt-Zuordnung |
| **DSGVO-by-Design** | EU-Server (Mistral AI), kein Telefonbuchzugriff, Privacy-by-Design |
| **Beiläufige Dokumentation** | Kommunikation IST die Dokumentation — automatisch archiviert |
| **Beiläufige Zeiterfassung** | "Bin auf der Baustelle" → Zeitstempel automatisch |
| **Made in Austria** | DACH-Vertrauen, regionale Identität |

## Market Sizing

| Metrik | Wert |
|--------|------|
| TAM | ~€425M/Jahr (DACH, alle Gewerke) |
| SAM | ~€99M/Jahr (Elektro + SHK, 5-50 MA) |
| SOM Year 1 | ~€120k ARR (AT-Pilotmarkt) |
| SOM Year 3 | ~€1.8M ARR (DACH Elektro + SHK) |

## Pricing

| Plan | Preis | Zielgruppe |
|------|-------|-----------|
| Starter | Kostenlos (bis 3 Nutzer) | Solo/Micro, Lead-Gen |
| Team | €10/Nutzer/Monat | 5-20 MA (Core Revenue) |
| Business | €8/Nutzer/Monat | 20-50 MA (Volumenrabatt) |

## Key Competitor

**Benetics AI** (CH/US) — Voice-First KI für Baubranche, €20/Nutzer, 250+ Firmen.
Aber: Fokus Großbau, nicht KMU-Handwerk. kurzum.app = halber Preis + DSGVO-by-Design + KMU-Nische.

## Technology Direction

| Layer | Technologie |
|-------|------------|
| Frontend/Hosting | Vercel (bestehendes Pro-Konto) |
| Database | NeonDB (bestehendes Konto) |
| Cache/Queue | Upstash + QStash (bestehend) |
| LLM / KI | Mistral AI (EU-hosted, DSGVO-konform) |
| Speech-to-Text | Mistral Voxtral / Whisper (self-hosted als Fallback) |
| Email | Resend (bestehend) |
| Auth | TBD |

## Go-to-Market

```
Q1-Q2 2026: FFG-Antrag + MVP + 5-10 Pilotbetriebe (Elektro, AT)
Q3-Q4 2026: SHK dazu, Innungs-Partnerschaft, €3-5k MRR
2027:       DE-Expansion (ZVEH, ZVSHK), Messen, €10-15k MRR
2028:       Weitere Gewerke, CH, €50k+ MRR
```

## Funding Status

- **FFG Kleinprojekt:** Antrag eingereicht am 10.02.2026 (Basisprogramm Kleinprojekt, €98.800, Antragsnr. 69168884). Bewilligung erwartet Mai/Juni 2026, Projektstart 01.07.2026.
- **FFG Markt.Einstieg:** Nach Kleinprojekt (2027) → bis €80.000
- **KMU.DIGITAL:** Für Kunden als Verkaufsargument (nicht für eigene Entwicklung)
- **KRITISCH:** Kein Projektstart vor FFG-Bewilligung!

## Scope Decisions

### UX-Design Scope (2026-02-10)

**Entscheidung:** Gesamtes UX auf High-Level designen, Recruiting-Landingpage im Detail.

**Begründung:** FFG-Förderantrag (Basisprogramm Kleinprojekt, €98.800, Antragsnr. 69168884) eingereicht am 10.02.2026. Vor Bewilligung darf nur geplant, nicht entwickelt werden. Die Landingpage zur Pilotbetrieb-Rekrutierung ist als Vorbereitungsmaßnahme erlaubt und darf entwickelt und ausgeliefert werden.

**Scope:**
- High-Level: Design-System & UX-Patterns für die gesamte kurzum.app (Farben, Typografie, Komponenten-Sprache, Navigation-Konzept)
- Detail: Vollständiges UX-Design nur für die Recruiting-Landingpage (kurzum.app)

### Architektur Scope (2026-02-10) — ABGESCHLOSSEN

**Entscheidung:** Gleicher Ansatz wie UX — Gesamtarchitektur high-level, Landingpage-Architektur im Detail.

**Scope:**
- High-Level: Gesamtarchitektur der kurzum.app planen (Tech-Stack, Datenmodell, API-Struktur, Offline-Sync-Konzept)
- Detail: Nur Landingpage-Architektur ausimplementierbar (Next.js, Formular-Backend, NeonDB, Resend)

## Aktueller Entwicklungsfokus: Recruiting Landing Page

**Was wird JETZT gebaut:** Eine Single-Page Landing Page unter kurzum.app zur Rekrutierung von 5–10 Pilotbetrieben für die Pilotphase (Okt 2026 – Apr 2027).

**Warum nur die Landing Page:** Laut FFG-Richtlinien sind Vorbereitungsmaßnahmen (Research, Marktvalidierung, Rekrutierung) vor Projektstart erlaubt und gelten NICHT als Projektbeginn. Die Landing Page ist eine solche Vorbereitungsmaßnahme. Die eigentliche App-Entwicklung darf erst nach FFG-Bewilligung (erwartet Mai/Juni 2026) starten.

**Landing Page Umfang:**
- Hero + Problem + Beispiel + Lösung + USP + Pilotprogramm + Anmeldeformular + Footer
- Wartelisten-Formular: Name, E-Mail, Firma, Gewerk, Betriebsgröße, optional Telefon + Nachricht
- Double-Opt-In via Resend E-Mail
- MMS API (api.masem.at) für Party/Contact/Consent-Management
- NeonDB für kurzum-spezifische Business-Daten (Gewerk, Betriebsgröße etc.)
- masemIT Analytics (analytics.masem.at) für Tracking
- Cloudflare Turnstile für Bot-Schutz
- Mobile-First, DSGVO-konform, Lighthouse ≥90

**KPIs:** 30–50 Wartelisten-Anmeldungen in 4 Monaten, ≥70% qualifizierte Leads (Elektro/SHK, 5–50 MA), ≥5% Conversion Rate

**Detailliertes Requirement:** `docs/_masemIT/requirements/requirement-kurzum-landing-page-2026-02-10.md`

**Abgeschlossene Planungsartefakte:**
- UX-Design: `_bmad-output/planning-artifacts/ux-design-specification.md`
- Architektur: `_bmad-output/planning-artifacts/architecture.md`
- PRD: `_bmad-output/planning-artifacts/prd.md`

## Key Risk

**Inertia** — "WhatsApp funktioniert doch." Gegenmittel: Regulatorischer Druck (DSGVO, Zeiterfassungsgesetz) + überwältigend einfaches Produkt + Innungs-Vertrieb.

## Research Documents

- `_bmad-output/planning-artifacts/research/domain-interne-kommunikation-kmu-handwerk-research-2026-02-08.md`
- `_bmad-output/planning-artifacts/research/market-ki-kommunikation-kmu-handwerk-research-2026-02-08.md`
- `_bmad-output/planning-artifacts/research/foerderungs-leitfaden-kurzum-app-2026.md`

## Owner Context

Mario Semper (Sempre), Einzelunternehmer, masemIT e.U., Kautzen/AT. Solo-Founder mit bestehendem Tech-Stack und mehreren laufenden Produkten (TellingCube, ChainSights, HoKi, MailWatcher). Intermediate skill level. Die Domäne Handwerk/Bau ist neu für ihn.
