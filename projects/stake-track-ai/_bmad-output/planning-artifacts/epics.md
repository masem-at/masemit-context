---
stepsCompleted:
  - step-01-validate-prerequisites
  - step-02-design-epics
  - step-03-create-stories
  - step-04-final-validation
inputDocuments:
  - _bmad-output/planning-artifacts/prd.md
  - _bmad-output/planning-artifacts/architecture.md
  - _bmad-output/planning-artifacts/ux-design-specification.md
---

# StakeTrack AI - Epic Breakdown

## Overview

This document provides the complete epic and story breakdown for StakeTrack AI, decomposing the requirements from the PRD, UX Design if it exists, and Architecture requirements into implementable stories.

## Requirements Inventory

### Functional Requirements

- **FR1:** User kann eine Ethereum-Wallet verbinden und sich per Signatur authentifizieren
- **FR2:** User kann eine Solana-Wallet verbinden und sich per Signatur authentifizieren
- **FR3:** User kann eine Cosmos-Wallet verbinden und sich per Signatur authentifizieren
- **FR4:** User kann sich alternativ per Email-Login registrieren und authentifizieren
- **FR5:** User kann mehrere Wallets (verschiedene Chains) mit einem Account verknüpfen
- **FR6:** User kann verknüpfte Wallets einsehen und einzeln entfernen
- **FR7:** User kann seinen Account und alle verknüpften Daten vollständig löschen (DSGVO)
- **FR8:** User kann nach Wallet-Connect alle Staking-Positionen über alle verknüpften Wallets und Chains aggregiert auf einem Dashboard sehen
- **FR9:** User kann seine Staking-Positionen nach Chain filtern
- **FR10:** User kann pro Position den aktuellen Validator, delegierten Betrag und Status einsehen
- **FR11:** User kann den Gesamt-Portfolio-Wert und die Gesamt-Performance sehen
- **FR12:** System zeigt pro Datenpunkt den Zeitstempel der letzten Aktualisierung an
- **FR13:** System zeigt bei temporär nicht verfügbaren Chain-Daten den letzten bekannten Stand mit Hinweis an (Graceful Degradation)
- **FR14:** System zeigt bei Wallets ohne Staking-Positionen einen informativen Leer-Zustand an
- **FR15:** User kann den Real Yield (Nominal APY - Inflation - Validator Fee - Slashing-Risiko) pro Staking-Position einsehen
- **FR16:** User kann den aggregierten Real Yield über sein gesamtes Portfolio sehen
- **FR17:** User kann den Real Yield zwischen verschiedenen Chains und Validators visuell vergleichen
- **FR18:** Nicht-authentifizierter Besucher kann den Public Real-Yield-Rechner auf einer eigenen Route nutzen (ohne Login)
- **FR19:** Public Real-Yield-Rechner zeigt einen Call-to-Action zur Wallet-Verbindung für personalisierte Ergebnisse
- **FR20:** System berechnet und speichert tägliche Yield-Snapshots pro Chain und Validator
- **FR21:** User kann den VRS (0-100) pro Validator als Ampel-Darstellung (Grün/Gelb/Rot) einsehen
- **FR22:** User kann die Aufschlüsselung des VRS in seine 5 Faktoren (Uptime, Slashing-Historie, Commission-Stabilität, Performance, Dezentralisierung) einsehen
- **FR23:** User kann das Confidence Level eines VRS-Scores sehen (basierend auf Datenverfügbarkeit)
- **FR24:** System berechnet VRS-Scores zeitgewichtet (letzte 30 Tage stärker gewichtet als 90 Tage)
- **FR25:** System speichert historische VRS-Scores als immutable Einträge
- **FR26:** Free-User sehen einen reduzierten VRS (3 Faktoren), Pro-User den vollen VRS (5 Faktoren)
- **FR27:** User kann Email-Alerts für VRS-Änderungen seiner überwachten Validators konfigurieren
- **FR28:** User kann Email-Alerts für Commission-Erhöhungen konfigurieren
- **FR29:** User kann Email-Alerts für Slashing-Events konfigurieren
- **FR30:** Free-User können maximal 1 Alert konfigurieren, Pro-User maximal 5
- **FR31:** System versendet Alerts mit kontextbezogenen Informationen (was hat sich geändert, aktueller VRS, betroffener Validator)
- **FR32:** User kann konfigurierte Alerts einsehen, bearbeiten und löschen
- **FR33:** System begrenzt Features basierend auf dem Tier des Users (Free vs. Pro)
- **FR34:** Free-User sind auf 1 Chain und 10 überwachte Validators begrenzt
- **FR35:** System zeigt bei Erreichen eines Feature-Limits einen Upgrade-Hinweis an
- **FR36:** Admin kann den Tier eines Users manuell ändern (MVP: kein Self-Service-Payment)
- **FR37:** System aktualisiert Validator- und Staking-Daten mindestens stündlich über alle 3 Chains
- **FR38:** System aktualisiert Chain-spezifische Inflationsraten täglich
- **FR39:** System berechnet VRS-Scores und Yield-Snapshots nach jeder Datenaktualisierung
- **FR40:** System invalidiert Caches nach jeder erfolgreichen Datenaktualisierung
- **FR41:** System verifiziert die Signatur auf allen Cron-Webhook-Endpoints (HMAC)
- **FR42:** System wendet Rate-Limiting auf alle öffentlichen API-Endpoints an

### NonFunctional Requirements

- **NFR1:** Dashboard TTFB (Time to First Byte) < 500ms für authentifizierte User
- **NFR2:** Cached API-Responses < 100ms
- **NFR3:** Uncached API-Responses < 2s (inkl. Chain-Adapter-Abfrage)
- **NFR4:** Public Real-Yield-Rechner (/calculator) First Load JS < 100KB (SSR-optimiert, minimaler Client-JS)
- **NFR5:** Gesamt First Load JS Bundle < 200KB
- **NFR6:** Serverless Cold Start < 1s
- **NFR7:** Wallet-Connect bis erste Daten-Anzeige < 60 Sekunden (Time-to-First-Insight)
- **NFR8:** Alle SIWE-Signaturen werden serverseitig verifiziert (Replay-Schutz mit Nonce)
- **NFR9:** Alle Datenbank-Verbindungen über SSL (sslmode=require)
- **NFR10:** Keine Wallet Private Keys werden gespeichert, übertragen oder angefordert
- **NFR11:** Alle Chain API Keys ausschließlich in serverseitigen Environment Variables (nie im Client-Bundle)
- **NFR12:** HMAC Signature Verification auf allen QStash Cron-Webhook-Endpoints
- **NFR13:** Rate-Limiting auf allen öffentlichen Endpoints (Upstash Rate Limit)
- **NFR14:** Bot-Schutz auf Signup/Login (Cloudflare Turnstile)
- **NFR15:** Keine personenbezogenen Daten in Server-Logs (DSGVO)
- **NFR16:** JWT-Sessions mit angemessener Expiry-Zeit und Refresh-Mechanismus
- **NFR17:** System unterstützt 500 registrierte User und 150 WAU ohne Performance-Degradation (Phase 1)
- **NFR18:** System skaliert auf 5.000 registrierte User ohne Architektur-Änderung (Phase 3)
- **NFR19:** NeonDB Storage bleibt unter 0.5 GiB für 12 Monate MVP-Betrieb
- **NFR20:** Caching-Layer (Upstash Redis) absorbiert mindestens 80% der Read-Requests
- **NFR21:** QStash Fan-out verarbeitet alle 3 Chain-Fetcher parallel innerhalb von 60 Sekunden (Vercel Timeout)
- **NFR22:** System-Uptime > 99.5%
- **NFR23:** Daten-Freshness: Validator-Daten maximal 1 Stunde alt unter Normalbetrieb
- **NFR24:** Bei Chain-API-Ausfall: Graceful Degradation mit letztem bekanntem Stand (kein Crash, keine leere Seite)
- **NFR25:** QStash Retry-Mechanismus stellt sicher, dass fehlgeschlagene Cron-Jobs automatisch wiederholt werden
- **NFR26:** VRS-Score-Berechnung ist deterministisch (gleiche Inputs = gleicher Score, reproduzierbar)
- **NFR27:** Historische VRS-Scores und Yield-Snapshots sind immutable (keine nachträgliche Änderung)
- **NFR28:** Chain Adapter sind voneinander isoliert — Ausfall eines Adapters beeinträchtigt nicht die anderen Chains
- **NFR29:** VRS-Berechnung korreliert > 95% mit historischen On-Chain-Slashing-Events (Accuracy)
- **NFR30:** Cache-Invalidierung erfolgt synchron nach jedem erfolgreichen Daten-Write
- **NFR31:** Alle externen API-Integrationen (MMS, Resend, Chain APIs) haben definierte Timeouts und Fehlerbehandlung
- **NFR32:** DSGVO-konform: Privacy Policy, Cookie Consent, Datenminimierung, Recht auf Löschung
- **NFR33:** Disclaimer auf Dashboard, Landing Page und in Email-Alerts sichtbar ("Keine Finanzberatung")
- **NFR34:** Wallet-Adressen werden als pseudo-anonyme personenbezogene Daten behandelt

### Additional Requirements

**Aus der Architektur:**
- Starter Template: `create-next-app@latest` mit TypeScript, Tailwind, App Router, src-dir, npm — DDD-Ordnerstruktur manuell anlegen (betrifft Epic 1, Story 1)
- Biome als Linter/Formatter von Tag 1 (native create-next-app-Unterstützung)
- Drizzle ORM + `@neondatabase/serverless` als ORM/DB-Layer
- NeonDB PostgreSQL 16 + TimescaleDB für Hypertables (Zeitreihen)
- Zod v4.3 für Runtime-Validierung an allen System-Grenzen (Chain-API-Responses, API-Input, Env-Vars, Forms)
- Environment Validation: Zod Env Schema in `src/lib/env.ts` (Fail-fast bei fehlenden Vars)
- TanStack Query v5.90 für Client-Side Data Layer (kommt als RainbowKit-Dependency)
- 3-Layer-Caching: RSC revalidate + Upstash Redis + NeonDB Materialized Views
- QStash Fan-out für parallele Chain-Fetcher (3 Chains parallel, je < 30s)
- NextAuth + SIWE + RainbowKit für Auth-Layer
- REST API via Next.js API Routes (kein tRPC)
- DDD-Layer-Boundaries: domain darf NUR domain/models importieren, KEIN React, KEIN Next.js, KEIN infra
- Admin-Zugang: Hardcoded Wallet-Adresse in Env-Var (`ADMIN_WALLET_ADDRESS`)
- Migration-Workflow: drizzle-kit generate lokal, commit, migrate in Build — NIEMALS push in Production
- Tests co-located neben Source-Dateien, Domain-Layer > 80% Coverage
- Named Exports (kein default export außer Next.js-Konventionen)
- Naming: snake_case (DB), camelCase (TS/JSON), kebab-case (Dateien), PascalCase (Komponenten)
- API Response Format: `{ data, meta? }` (Success), `{ error, code, details? }` (Error)
- Logging: `[component] message` Format, KEINE Wallet-Adressen in Logs (nur gekürzt)

**Aus dem UX-Design:**
- Dark Mode als Default (Crypto-native), Light Mode als Option
- Responsive: Mobile gleichwertig zu Desktop (kein "Mobile-Kompromiss")
- Mobile: Bottom-Navigation (4 Tabs), Bottom Sheets für Details, Thumb-Zone-optimiert
- Desktop: Collapsed Sidebar (64px default, 240px expanded)
- Tablet: Top-Navigation-Bar, kein Sidebar
- Tremor + Tailwind CSS als Design-System-Basis, Dark-Mode-Customization
- Custom-Komponenten: VRS-Ampel (VrsScoreBadge), Real-Yield-Schock-Balken (RealYieldComparison), Bottom Sheet, Wallet-Connect-Button, Freshness-Indicator, Confidence-Badge
- Farbsystem: Brand Teal #06B6D4, bg-base #0A0A0F, bg-surface #12121A, Status-Farben für VRS/Yield
- System-Font-Stack (kein Custom-Font-Loading), Monospace für Wallet-Adressen/Zahlen
- WCAG 2.1 AA: Kontraste validiert, Keyboard-Navigation, Screen-Reader-Support (ARIA)
- Touch-Targets Minimum 44x44px
- Optimistic UI bei Chain-Filter-Wechsel, Progressive Disclosure (Ampel → Detail)
- Skeleton-Loading statt Spinner, progressiver Load bei Wallet-Connect
- Calculator: Live-Berechnung (debounced 300ms), kein Submit-Button, Share-Funktion
- Inline-Upgrade-Nudges (sachlich, keine Dark Patterns)
- prefers-reduced-motion, prefers-color-scheme, prefers-contrast respektieren
- axe-core Integration in Playwright E2E Tests

### FR Coverage Map

| FR | Epic | Beschreibung |
|---|---|---|
| FR1 | Epic 1 | ETH-Wallet-Connect + SIWE-Auth |
| FR2 | Epic 1 | SOL-Wallet-Connect + Message-Signing |
| FR3 | Epic 1 | ATOM-Wallet-Connect + ADR-036 |
| FR4 | Epic 1 | Email-Login via MMS |
| FR5 | Epic 1 | Multi-Wallet-Linking |
| FR6 | Epic 1 | Wallet-Verwaltung (einsehen/entfernen) |
| FR7 | Epic 1 | Account-Löschung (DSGVO) |
| FR8 | Epic 2 | Dashboard: Aggregierte Staking-Positionen |
| FR9 | Epic 2 | Dashboard: Chain-Filter |
| FR10 | Epic 2 | Dashboard: Position-Details (Validator, Betrag, Status) |
| FR11 | Epic 2 | Dashboard: Portfolio-Gesamt-Wert und Performance |
| FR12 | Epic 2 | Dashboard: Freshness-Timestamps |
| FR13 | Epic 2 | Dashboard: Graceful Degradation |
| FR14 | Epic 2 | Dashboard: Empty States |
| FR15 | Epic 3 | Real Yield pro Position |
| FR16 | Epic 3 | Aggregierter Portfolio Real Yield |
| FR17 | Epic 3 | Real Yield visueller Vergleich |
| FR18 | Epic 3 | Public Calculator ohne Login |
| FR19 | Epic 3 | Calculator CTA → Wallet-Connect |
| FR20 | Epic 3 | Tägliche Yield-Snapshots |
| FR21 | Epic 4 | VRS-Ampel-Anzeige (0-100) |
| FR22 | Epic 4 | VRS 5-Faktoren-Aufschlüsselung |
| FR23 | Epic 4 | VRS Confidence Level |
| FR24 | Epic 4 | Zeitgewichtete VRS-Berechnung |
| FR25 | Epic 4 | Immutable VRS-Historie |
| FR26 | Epic 4 | VRS Tier-Gating (3 vs. 5 Faktoren) |
| FR27 | Epic 5 | Alerts: VRS-Änderungen |
| FR28 | Epic 5 | Alerts: Commission-Erhöhungen |
| FR29 | Epic 5 | Alerts: Slashing-Events |
| FR30 | Epic 5 | Alerts: Tier-Limits (1 vs. 5) |
| FR31 | Epic 5 | Alerts: Kontextbezogene Informationen |
| FR32 | Epic 5 | Alerts: CRUD-Verwaltung |
| FR33 | Epic 6 | Feature-Gating nach Tier |
| FR34 | Epic 6 | Free-Tier-Limits (1 Chain, 10 Validators) |
| FR35 | Epic 6 | Upgrade-Hinweise bei Limit |
| FR36 | Epic 6 | Admin: Tier manuell ändern |
| FR37 | Epic 2 | Stündliche Daten-Aktualisierung (3 Chains) |
| FR38 | Epic 2 | Tägliche Inflationsraten-Updates |
| FR39 | Epic 2 | VRS/Yield-Berechnung nach Daten-Update |
| FR40 | Epic 2 | Cache-Invalidierung nach Updates |
| FR41 | Epic 2 | HMAC auf Cron-Endpoints |
| FR42 | Epic 2 | Rate-Limiting auf Public Endpoints |

## Epic List

### Epic 1: Projekt-Foundation & User-Authentifizierung
User können sich per Wallet (ETH/SOL/ATOM) oder Email registrieren, einloggen, ihre Wallets verwalten und ihren Account löschen. Enthält Sprint 0 Setup (create-next-app, DDD-Struktur, Biome, Drizzle, NeonDB/TimescaleDB, Zod Env) als enabling Story, dann vollständiges Auth-System (SIWE + Email via MMS, NextAuth, JWT) und Wallet-/Account-Management inkl. DSGVO.
**FRs covered:** FR1, FR2, FR3, FR4, FR5, FR6, FR7

### Epic 2: Multi-Chain Staking Dashboard
User sehen nach Wallet-Connect alle Staking-Positionen über alle Chains aggregiert auf einem Dashboard — mit Chain-Filter, Freshness-Timestamps und Graceful Degradation. Enthält Chain Adapters (ETH/SOL/ATOM), QStash Cron-Pipeline (stündlich), Upstash Redis Cache-Layer, Dashboard UI mit Portfolio-Aggregation, API-Security (HMAC auf Cron-Endpoints, Rate Limiting).
**FRs covered:** FR8, FR9, FR10, FR11, FR12, FR13, FR14, FR37, FR38, FR39, FR40, FR41, FR42

### Epic 3: Real-Yield Intelligence
User verstehen ihre echten Staking-Erträge (Real Yield). Der Public Calculator auf /calculator ist ohne Login nutzbar als viraler Einstiegspunkt mit Share-Funktion und Conversion-CTA. Enthält Yield Calculator Domain-Logik, Public Calculator Page (SSR-optimiert), tägliche Yield-Snapshots, Real-Yield-Schock-Balken-Komponente, Portfolio-Real-Yield-Aggregation.
**FRs covered:** FR15, FR16, FR17, FR18, FR19, FR20

### Epic 4: Validator Risk Scoring (VRS)
User können das Risiko ihrer Validators auf einen Blick einschätzen — VRS-Ampel (Grün/Gelb/Rot), 5-Faktoren-Aufschlüsselung, Confidence Level, historische Scores. Enthält VRS Engine (5 Faktoren, konfigurierbare Gewichtungen, zeitgewichtet), VRS-Ampel-Komponente, VRS-Detail-Ansicht, Confidence-Badge, VRS-Cron-Berechnung.
**FRs covered:** FR21, FR22, FR23, FR24, FR25, FR26

### Epic 5: Alert & Benachrichtigungs-System
User werden proaktiv per Email über wichtige Validator-Änderungen informiert — VRS-Drops, Commission-Erhöhungen, Slashing-Events. Alerts sind konfigurierbar und verwaltbar. Enthält Alert Engine (Evaluator, Conditions), Email-Templates via Resend, Alert-Management UI, Alert-Cron-Processing, Deep-Links zum Dashboard.
**FRs covered:** FR27, FR28, FR29, FR30, FR31, FR32

### Epic 6: Freemium-Gating & Subscription-Management
User erleben ein faires Freemium-Modell — Free-Tier zeigt genug Value für Conversion, Pro-Tier schaltet erweiterte Features frei. Upgrade-Hinweise sind sachlich, nicht drängend. Enthält Tier-System (Free vs. Pro), Feature-Gating Middleware, Chain-/Validator-/Alert-Limits, VRS-Granularität (3 vs. 5 Faktoren), Upgrade-CTAs, Admin-Tier-Verwaltung.
**FRs covered:** FR33, FR34, FR35, FR36

## Epic 1: Projekt-Foundation & User-Authentifizierung

User können sich per Wallet (ETH/SOL/ATOM) oder Email registrieren, einloggen, ihre Wallets verwalten und ihren Account löschen.

### Story 1.1: Projekt-Initialisierung & Infrastruktur-Setup

As a Entwickler,
I want ein vollständig konfiguriertes Next.js 15 Projekt mit DDD-Struktur, Datenbank-Anbindung und Design-System-Grundgerüst,
So that alle nachfolgenden Stories auf einer konsistenten, produktionsreifen Basis aufbauen können.

**Acceptance Criteria:**

**Given** ein leeres Repository
**When** das Projekt initialisiert wird
**Then** ist ein Next.js 15 App Router Projekt mit TypeScript, Tailwind CSS, `src/`-Verzeichnis und npm erstellt
**And** Biome ist als Linter/Formatter konfiguriert (biome.json)
**And** die DDD-Ordnerstruktur existiert (`src/domain/`, `src/domain/models/`, `src/infrastructure/`, `src/infrastructure/db/`, `src/infrastructure/cache/`, `src/infrastructure/queue/`, `src/infrastructure/email/`, `src/infrastructure/mms/`, `src/components/`, `src/lib/`, `src/providers/`)
**And** Drizzle ORM ist mit `@neondatabase/serverless` konfiguriert (drizzle.config.ts, `src/infrastructure/db/index.ts`)
**And** die Zod Environment Validation existiert (`src/lib/env.ts`) mit Server/Client-Trennung und Fail-fast bei fehlenden Vars
**And** eine `.env.example` mit allen benötigten Environment Variables existiert
**And** Tailwind ist mit dem StakeTrack AI Dark-Mode-Farbsystem konfiguriert (Custom Tokens: bg-base, bg-surface, bg-surface-raised, border-subtle, text-primary, text-secondary, brand-primary, status-positive/warning/negative)
**And** Tremor (`@tremor/react`) ist installiert und Dark-Mode-kompatibel konfiguriert
**And** ein Root Layout (`src/app/layout.tsx`) existiert mit System-Font-Stack, Dark Mode als Default und Theme-Provider
**And** eine responsive App-Shell existiert: Collapsed Sidebar (Desktop, 64px), Bottom-Navigation (Mobile, 4 Tabs), Top-Bar (Tablet)
**And** Vitest ist konfiguriert (`vitest.config.ts`) mit Coverage-Setup
**And** Playwright ist konfiguriert (`playwright.config.ts`) für E2E Tests
**And** `tsconfig.json` hat strict mode und `@/` Path-Alias
**And** der Dev-Server startet fehlerfrei (`npm run dev`)

### Story 1.2: ETH-Wallet-Authentifizierung (SIWE)

As a User mit einer Ethereum-Wallet,
I want mich per Wallet-Signatur authentifizieren können,
So that ich sicher und ohne Passwort auf mein StakeTrack AI Dashboard zugreifen kann.

**Acceptance Criteria:**

**Given** ein nicht-authentifizierter User auf der Signin-Page
**When** er auf "Connect Wallet" klickt
**Then** öffnet sich das RainbowKit-Modal mit verfügbaren ETH-Wallets (MetaMask, WalletConnect, etc.)
**And** das Modal ist im Dark Theme gestaltet (konsistent mit StakeTrack AI Design)

**Given** ein User hat eine ETH-Wallet im RainbowKit-Modal ausgewählt
**When** die SIWE-Signatur-Anfrage erscheint
**Then** enthält die Nachricht eine Server-generierte Nonce (Replay-Schutz, NFR8)
**And** die Nachricht erklärt klar "Read-Only-Zugang zu StakeTrack AI"

**Given** ein User signiert die SIWE-Nachricht erfolgreich
**When** die Signatur serverseitig verifiziert wird
**Then** wird ein User-Eintrag in der `users`-Tabelle erstellt (falls neu) oder aktualisiert (falls bestehend)
**And** eine JWT-Session wird erstellt (NextAuth) mit angemessener Expiry-Zeit (NFR16)
**And** der User wird zum Dashboard weitergeleitet
**And** die Wallet-Adresse wird NICHT vollständig in Server-Logs geschrieben (nur gekürzt: 0x1234...5678, NFR15)

**Given** ein User lehnt die Signatur ab
**When** die Ablehnung erkannt wird
**Then** zeigt die UI einen sachlichen Hinweis: "Signatur nötig für Read-Only-Zugang"
**And** der User kann es erneut versuchen

**Given** ein bereits authentifizierter User
**When** er die App öffnet
**Then** wird die bestehende JWT-Session genutzt (kein erneuter Wallet-Connect nötig)

**Given** die Wallet-Verbindung schlägt fehl (Timeout, Netzwerkfehler)
**When** der Fehler erkannt wird
**Then** zeigt die UI "Verbindung fehlgeschlagen. Nochmal versuchen?" mit Retry-Button

**DB-Schema (in dieser Story erstellt):**
- `users`-Tabelle: id, wallet_address, email (nullable), tier (default: 'free'), created_at, updated_at

### Story 1.3: Email-Authentifizierung via MMS

As a User ohne Crypto-Wallet,
I want mich per Email registrieren und einloggen können,
So that ich StakeTrack AI auch ohne Wallet-Besitz nutzen kann.

**Acceptance Criteria:**

**Given** ein nicht-authentifizierter User auf der Signin-Page
**When** er den Email-Login-Tab wählt
**Then** sieht er ein Email-Eingabefeld mit Cloudflare Turnstile Bot-Schutz (NFR14)

**Given** ein User gibt eine gültige Email-Adresse ein und besteht den Turnstile-Check
**When** er auf "Login / Registrieren" klickt
**Then** wird über den MMS-Client (`src/infrastructure/mms/mms-client.ts`) ein Magic-Link oder Verification-Code an die Email gesendet
**And** die UI zeigt "Prüfe deine Email für den Login-Link"

**Given** ein User klickt den Magic-Link in seiner Email
**When** der Link serverseitig validiert wird
**Then** wird ein User-Eintrag erstellt (falls neu, mit email statt wallet_address) oder die Session aktualisiert
**And** eine JWT-Session wird erstellt (NextAuth)
**And** der User wird zum Dashboard weitergeleitet

**Given** ein User gibt eine ungültige Email ein
**When** die Validierung fehlschlägt (Zod)
**Then** zeigt die UI eine spezifische Inline-Fehlermeldung unter dem Feld

**Given** der Turnstile-Check schlägt fehl
**When** der Bot-Verdacht erkannt wird
**Then** wird kein MMS-Request gesendet
**And** der User wird aufgefordert, den Check erneut zu versuchen

### Story 1.4: Multi-Chain Wallet-Linking (SOL + ATOM)

As a authentifizierter User,
I want meine Solana- und Cosmos-Wallets zusätzlich verknüpfen können,
So that ich mein komplettes Multi-Chain-Staking-Portfolio an einem Ort sehen kann.

**Acceptance Criteria:**

**Given** ein authentifizierter User (via ETH-Wallet oder Email)
**When** er auf "Wallet hinzufügen" klickt
**Then** sieht er einen Chain-Selector (SOL / ATOM)

**Given** ein User wählt "SOL"
**When** der Solana Wallet Adapter sich öffnet (Phantom, Solflare, etc.)
**Then** wird eine Message-Signatur (analog SIWE) angefordert
**And** bei erfolgreicher Signatur wird die SOL-Wallet in der `linked_wallets`-Tabelle gespeichert
**And** das Dashboard aktualisiert sich und zeigt die neue Chain

**Given** ein User wählt "ATOM"
**When** Keplr Connect sich öffnet
**Then** wird eine ADR-036 Message-Signatur angefordert
**And** bei erfolgreicher Signatur wird die ATOM-Wallet in der `linked_wallets`-Tabelle gespeichert

**Given** ein User versucht eine bereits verknüpfte Wallet erneut zu verbinden
**When** die Duplikat-Prüfung greift
**Then** zeigt die UI einen Toast: "Diese Wallet ist bereits verbunden"

**Given** das Wallet-Linking fehlschlägt (Signatur abgelehnt, Timeout)
**When** der Fehler erkannt wird
**Then** zeigt die UI "Verbindung fehlgeschlagen" mit Retry-Option

**Given** ein User hat 2+ Chains verknüpft
**When** das Dashboard geladen wird
**Then** erscheint der Chain-Filter als Tabs ("Alle / ETH / SOL / ATOM") statt eines einfachen Banners

**DB-Schema (in dieser Story erstellt):**
- `linked_wallets`-Tabelle: id, user_id (FK → users), chain (enum: eth/sol/atom), wallet_address, linked_at

### Story 1.5: Wallet-Verwaltung & Einstellungen

As a authentifizierter User,
I want meine verknüpften Wallets einsehen und einzeln entfernen können,
So that ich die Kontrolle über meine Daten behalte.

**Acceptance Criteria:**

**Given** ein authentifizierter User auf der Settings-Page (`/settings`)
**When** die Seite geladen wird
**Then** sieht er eine Liste aller verknüpften Wallets mit: Chain-Badge (ETH/SOL/ATOM), gekürzter Adresse (0x1234...5678), Verknüpfungsdatum
**And** die primäre Auth-Wallet ist als "Primär" markiert

**Given** ein User klickt "Entfernen" bei einer verknüpften Wallet
**When** es sich um eine sekundäre (nicht-primäre) Wallet handelt
**Then** erscheint ein Bestätigungs-Dialog: "Wallet [gekürzte Adresse] auf [Chain] trennen?"
**And** bei Bestätigung wird die Wallet aus `linked_wallets` entfernt
**And** ein Success-Toast erscheint: "Wallet getrennt"
**And** die Liste aktualisiert sich

**Given** ein User versucht seine einzige/primäre Auth-Wallet zu entfernen
**When** keine alternative Auth-Methode existiert (kein Email-Login)
**Then** zeigt die UI einen Hinweis: "Deine primäre Wallet kann nicht entfernt werden, solange keine alternative Login-Methode existiert"

**Given** ein User hat sowohl Wallet als auch Email verknüpft
**When** er die primäre Wallet entfernen will
**Then** wird die Email zum primären Auth-Mechanismus
**And** die Wallet wird entfernt

### Story 1.6: Account-Löschung (DSGVO)

As a authentifizierter User,
I want meinen Account und alle verknüpften Daten vollständig löschen können,
So that mein Recht auf Löschung (DSGVO Art. 17) gewährleistet ist.

**Acceptance Criteria:**

**Given** ein authentifizierter User auf der Settings-Page
**When** er auf "Account löschen" klickt
**Then** erscheint ein Bestätigungs-Modal (Destructive Confirmation) mit: Titel "Account unwiderruflich löschen?", Erklärung was gelöscht wird (Account, alle Wallets, alle Daten), Eingabefeld wo User "LÖSCHEN" tippen muss
**And** der Confirm-Button ist im `status-negative` Destructive-Stil

**Given** ein User bestätigt die Löschung korrekt
**When** die Löschung ausgeführt wird
**Then** werden alle Daten des Users gelöscht: `users`-Eintrag, alle `linked_wallets`, alle zukünftigen verknüpften Daten (Alerts, Positionen — Cascade Delete)
**And** die JWT-Session wird invalidiert
**And** der User wird zur Landing Page weitergeleitet
**And** ein sachlicher Hinweis erscheint: "Dein Account wurde gelöscht"

**Given** ein User tippt nicht das korrekte Bestätigungswort
**When** er auf "Löschen" klickt
**Then** wird die Löschung NICHT ausgeführt
**And** das Eingabefeld zeigt einen Inline-Error

**Given** die Löschung schlägt technisch fehl
**When** ein DB-Fehler auftritt
**Then** zeigt die UI "Löschung fehlgeschlagen. Bitte kontaktiere den Support."
**And** der Account bleibt intakt

## Epic 2: Multi-Chain Staking Dashboard

User sehen nach Wallet-Connect alle Staking-Positionen über alle Chains aggregiert auf einem Dashboard — mit Chain-Filter, Freshness-Timestamps und Graceful Degradation.

### Story 2.1: Chain Adapter Infrastructure & Ethereum Adapter

As a System-Betreiber,
I want dass Ethereum-Validator- und Staking-Daten automatisch stündlich abgerufen und normalisiert gespeichert werden,
So that das Dashboard aktuelle ETH-Staking-Positionen anzeigen kann.

**Acceptance Criteria:**

**Given** die Chain Adapter Infrastructure wird aufgebaut
**When** die Implementierung abgeschlossen ist
**Then** existiert ein `ChainAdapter` Interface in `src/domain/chain-adapters/types.ts` mit Methoden: `fetchValidators()`, `fetchStakingPositions(walletAddress)`, `fetchInflationRate()`
**And** existieren Unified Domain Models in `src/domain/models/`: `UnifiedValidator`, `StakingPosition`, `Chain` enum
**And** alle Models haben korrespondierende Zod Schemas für Runtime-Validierung

**Given** der Ethereum Adapter (`src/domain/chain-adapters/ethereum/`)
**When** `fetchValidators()` aufgerufen wird
**Then** werden Validator-Daten von der Beacon Chain API abgerufen
**And** die Raw API Response wird per Zod Schema validiert
**And** die Daten werden via `eth-mapper.ts` auf das `UnifiedValidator` Model gemappt
**And** die Validators werden in der `validators`-Tabelle gespeichert (upsert)

**Given** der Ethereum Adapter
**When** `fetchStakingPositions(walletAddress)` aufgerufen wird
**Then** werden die Staking-Positionen für die gegebene ETH-Adresse abgerufen
**And** die Positionen werden in der `staking_positions`-Tabelle gespeichert

**Given** der Ethereum Adapter
**When** `fetchInflationRate()` aufgerufen wird
**Then** wird die aktuelle ETH-Inflationsrate abgerufen und in der `chain_metrics`-Tabelle gespeichert (FR38)

**Given** ein QStash Cron-Job für `/api/cron/fetch-validators`
**When** der Cron stündlich feuert (FR37)
**Then** wird der ETH-Adapter aufgerufen
**And** die HMAC-Signatur des QStash-Webhooks wird verifiziert (FR41)
**And** bei Erfolg wird der Cache invalidiert (FR40)
**And** bei Fehler loggt das System `[eth-adapter] Fetch failed: {error}` und QStash retried automatisch

**Given** die Beacon Chain API ist nicht erreichbar
**When** ein Timeout oder Fehler auftritt
**Then** werden die anderen Chain Adapters NICHT beeinträchtigt (NFR28, Isolation)
**And** der Fehler wird geloggt ohne Wallet-Adressen

**DB-Schema (in dieser Story erstellt):**
- `validators`: id, chain, address, name, commission, uptime, status, is_active, raw_data (jsonb), fetched_at, created_at, updated_at
- `staking_positions`: id, user_id (FK), chain, validator_id (FK), wallet_address, delegated_amount, status, fetched_at, created_at, updated_at
- `chain_metrics`: id, chain, inflation_rate, total_staked, avg_apy, fetched_at, created_at

**Tests:** eth-adapter.test.ts, eth-mapper.test.ts mit MSW für API-Mocking

### Story 2.2: Solana Chain Adapter

As a User mit SOL-Staking-Positionen,
I want dass meine Solana-Staking-Daten automatisch abgerufen werden,
So that mein SOL-Portfolio im Dashboard erscheint.

**Acceptance Criteria:**

**Given** der Solana Adapter (`src/domain/chain-adapters/solana/`)
**When** `fetchValidators()` aufgerufen wird
**Then** werden Solana-Validator-Daten via RPC (`getVoteAccounts`) abgerufen
**And** die Daten werden per Zod Schema validiert und auf `UnifiedValidator` gemappt

**Given** der Solana Adapter
**When** `fetchStakingPositions(walletAddress)` aufgerufen wird
**Then** werden Stake Accounts via `getAccountInfo` + manuelles Parsing abgerufen (NICHT `getStakeActivation` — deprecated!)
**And** die Positionen werden normalisiert in `staking_positions` gespeichert

**Given** der Solana Adapter
**When** `fetchInflationRate()` aufgerufen wird
**Then** wird die aktuelle Solana-Inflationsrate abgerufen und in `chain_metrics` gespeichert

**Given** der QStash Cron-Job `/api/cron/fetch-validators`
**When** der Cron feuert
**Then** wird der SOL-Adapter parallel zum ETH-Adapter aufgerufen (QStash Fan-out, NFR21)
**And** beide Adapter laufen unabhängig — ein SOL-Fehler blockiert nicht ETH

**Given** die Solana RPC API ist nicht erreichbar
**When** ein Fehler auftritt
**Then** wird Helius als Fallback-RPC verwendet (falls konfiguriert)
**And** der Fehler wird geloggt

**Tests:** sol-adapter.test.ts, sol-mapper.test.ts

### Story 2.3: Cosmos Chain Adapter

As a User mit ATOM-Staking-Positionen,
I want dass meine Cosmos-Staking-Daten automatisch abgerufen werden,
So that mein ATOM-Portfolio im Dashboard erscheint.

**Acceptance Criteria:**

**Given** der Cosmos Adapter (`src/domain/chain-adapters/cosmos/`)
**When** `fetchValidators()` aufgerufen wird
**Then** werden Cosmos Hub Validator-Daten via LCD/REST API abgerufen
**And** die Daten werden per Zod Schema validiert und auf `UnifiedValidator` gemappt

**Given** der Cosmos Adapter
**When** `fetchStakingPositions(walletAddress)` aufgerufen wird
**Then** werden Delegations via `/cosmos/staking/v1beta1/delegations/{address}` abgerufen
**And** die Positionen werden normalisiert gespeichert

**Given** der Cosmos Adapter
**When** `fetchInflationRate()` aufgerufen wird
**Then** wird die Cosmos-Inflationsrate via `/cosmos/mint/v1beta1/inflation` abgerufen

**Given** der QStash Cron-Job
**When** alle 3 Adapter parallel laufen (Fan-out)
**Then** sind ETH, SOL und ATOM jeweils unabhängig (NFR28)
**And** die Gesamtverarbeitung bleibt unter 60 Sekunden (NFR21)

**Given** die Cosmos LCD API ist nicht erreichbar
**When** ein Fehler auftritt
**Then** wird Mintscan API als Fallback verwendet (falls konfiguriert)

**Tests:** atom-adapter.test.ts, atom-mapper.test.ts

### Story 2.4: Upstash Redis Cache-Layer & API-Security

As a User,
I want dass das Dashboard blitzschnell lädt und die API sicher ist,
So that ich eine performante und vertrauenswürdige Erfahrung habe.

**Acceptance Criteria:**

**Given** der Redis Cache-Layer (`src/infrastructure/cache/`)
**When** konfiguriert ist
**Then** existiert ein `redis-client.ts` mit Upstash Redis Instance
**And** existiert `cache-keys.ts` mit Key-Pattern: `staketrack:{entity}:{chain}:{id}`
**And** existiert `cache-utils.ts` mit `get`, `set`, `invalidate` Helpers
**And** Validator-Daten werden mit TTL 5 Minuten gecacht
**And** VRS-Scores werden mit TTL 1 Stunde gecacht
**And** Yield-Daten werden mit TTL 24 Stunden gecacht

**Given** ein Cron-Job aktualisiert Daten erfolgreich
**When** die Cache-Invalidierung läuft (FR40)
**Then** werden die betroffenen Cache-Keys synchron invalidiert
**And** der nächste Request liefert frische Daten

**Given** Rate Limiting (`src/lib/auth/middleware.ts`)
**When** auf öffentliche API-Endpoints zugegriffen wird
**Then** greift Upstash Rate Limit (FR42)
**And** bei Überschreitung wird 429 mit `Retry-After` Header zurückgegeben

**Given** ein Request an `/api/cron/*` Endpoints
**When** die HMAC-Signatur fehlt oder ungültig ist
**Then** wird der Request mit 401 abgelehnt (FR41)
**And** nur QStash-signierte Requests werden verarbeitet

**Given** der Cache 80%+ der Read-Requests absorbiert (NFR20)
**When** ein Dashboard-Request eintrifft
**Then** wird zuerst Redis geprüft → bei Hit: Response <100ms (NFR2)
**And** bei Miss: DB-Query → Redis Set → Response <2s (NFR3)

### Story 2.5: Staking Dashboard UI & Portfolio-Aggregation

As a authentifizierter User,
I want alle meine Staking-Positionen über alle Chains auf einem Dashboard sehen,
So that ich sofort einen Überblick über mein gesamtes Staking-Portfolio habe.

**Acceptance Criteria:**

**Given** ein authentifizierter User öffnet `/dashboard`
**When** die Seite geladen wird (RSC, <500ms TTFB, NFR1)
**Then** sieht er eine KPI-Row mit 3 Cards: Gesamt gestaked (€-Betrag), Anzahl Validators, Anzahl Chains
**And** die KPI-Zahlen nutzen `kpi-large`/`kpi-medium` Font-Size mit `tabular-nums`

**Given** ein User hat Positionen auf mehreren Chains
**When** das Dashboard geladen wird
**Then** zeigt der Chain-Filter horizontale Tabs: "Alle / ETH / SOL / ATOM" (Tremor TabGroup, FR9)
**And** "Alle" ist der Default-Tab
**And** der aktive Tab bleibt zwischen Sessions erhalten (localStorage)

**Given** ein User klickt auf einen Chain-Tab
**When** der Filter gewechselt wird
**Then** aktualisiert sich die Validator-Liste sofort (Optimistic UI)
**And** die Daten laden im Hintergrund nach (TanStack Query)

**Given** die Validator-Liste wird angezeigt
**When** der User sie betrachtet
**Then** sieht er pro Validator: Name, Chain-Badge, delegierter Betrag, Status (FR10)
**And** auf Desktop: Full-width Tremor Table mit allen Spalten
**And** auf Mobile: Kompakte Cards mit Name, Chain-Badge und Betrag (kein Horizontal-Scroll)

**Given** ein User sieht die Portfolio-Aggregation (FR8, FR11)
**When** die Daten geladen sind
**Then** zeigt das Dashboard den Gesamt-Portfolio-Wert und die Gesamt-Performance
**And** die Aggregation berücksichtigt alle verknüpften Wallets und Chains

**Given** der User nutzt Pull-to-Refresh auf Mobile
**When** er nach unten zieht
**Then** werden die Dashboard-Daten manuell aktualisiert

**DB-Zugriff:** Portfolio Aggregator (`src/domain/portfolio/portfolio-aggregator.ts`) aggregiert Daten aus `staking_positions` + `validators`
**API:** `GET /api/portfolio`, `GET /api/validators?chainId={chain}`

### Story 2.6: Daten-Freshness, Graceful Degradation & Empty States

As a User,
I want immer wissen wie aktuell meine Daten sind und auch bei API-Ausfällen eine nutzbare Erfahrung haben,
So that ich dem Dashboard vertrauen kann.

**Acceptance Criteria:**

**Given** das Dashboard zeigt Staking-Daten
**When** die Daten angezeigt werden
**Then** zeigt jeder Datenpunkt einen Freshness-Indicator (FR12): Grüner Dot + "Vor X Min" (<30 Min), Gelber Dot + "Vor X Min" (30 Min – 2h), Roter Dot + "Vor X h" (>2h)
**And** der Freshness-Indicator hat `aria-label="Daten aktualisiert vor X Minuten"`

**Given** eine Chain-API ist temporär nicht erreichbar
**When** das Dashboard geladen wird (FR13)
**Then** werden die letzten bekannten Daten für diese Chain angezeigt
**And** ein Stale-Data-Banner erscheint: "[Chain]-Daten von vor [Zeitraum]. Aktualisierung läuft." (status-info Farbe, nicht Alarm-Rot)
**And** die anderen Chains werden NICHT beeinträchtigt (NFR28)
**And** es gibt keinen Crash und keine leere Seite (NFR24)

**Given** ein User hat eine Wallet verbunden aber keine Staking-Positionen (FR14)
**When** das Dashboard geladen wird
**Then** zeigt ein freundlicher Empty State: "Keine Staking-Positionen gefunden."
**And** ein Aktions-Hinweis: "Hast du auf einer anderen Chain gestaked? → SOL hinzufügen · ATOM hinzufügen"
**And** kein Schuldzuweisungs-Text, minimales Line-Icon, monochrom

**Given** ein User hat keine Wallets verknüpft
**When** das Dashboard geladen wird
**Then** zeigt ein Empty State: "Verbinde deine Wallet und sieh dein Staking-Portfolio."
**And** ein Primary Button: "Connect Wallet"

**Given** der Chain-Filter zeigt keine Ergebnisse nach Filterung
**When** der User eine Chain ohne Positionen wählt
**Then** zeigt ein Empty State: "Keine Validators für [Chain] gefunden."
**And** ein Link: "Alle Chains anzeigen →"

**Komponenten:** FreshnessIndicator (`inline` + `banner` Variante), StaleDataBanner, EmptyState

## Epic 3: Real-Yield Intelligence

User verstehen ihre echten Staking-Erträge (Real Yield). Der Public Calculator auf /calculator ist ohne Login nutzbar als viraler Einstiegspunkt mit Share-Funktion und Conversion-CTA.

### Story 3.1: Real-Yield Calculator Domain-Logik & Tägliche Snapshots

As a System-Betreiber,
I want dass Real-Yield-Werte korrekt berechnet und täglich als Snapshots gespeichert werden,
So that User und der Public Calculator jederzeit auf aktuelle Real-Yield-Daten zugreifen können.

**Acceptance Criteria:**

**Given** der Yield Calculator (`src/domain/yield/yield-calculator.ts`)
**When** `calculateRealYield()` aufgerufen wird
**Then** berechnet er: Real Yield = Nominal APY − Inflation Rate − Validator Fee − Slashing-Risiko
**And** jede Komponente der Formel ist einzeln im Ergebnis enthalten (für Breakdown-Anzeige)
**And** die Berechnung ist deterministisch (gleiche Inputs = gleiches Ergebnis)
**And** das Ergebnis enthält: `nominalApy`, `inflationRate`, `validatorFee`, `slashingRisk`, `realYield`, `chain`, `validatorId`

**Given** eine `yield-config.ts` mit konfigurierbaren Parametern
**When** sich Chain-spezifische Faktoren ändern (z.B. SIMD-123 auf Solana)
**Then** kann die Formel über Konfiguration angepasst werden ohne Code-Änderung

**Given** der QStash Cron-Job `/api/cron/calculate-yield`
**When** der Cron täglich feuert (FR20)
**Then** werden Yield-Snapshots für alle aktiven Validators über alle Chains berechnet
**And** die Snapshots werden in der `yield_snapshots`-Tabelle gespeichert (TimescaleDB Hypertable)
**And** historische Snapshots sind immutable (NFR27)
**And** der Cache wird nach erfolgreicher Berechnung invalidiert

**Given** Chain-Daten fehlen (z.B. Inflationsrate nicht verfügbar)
**When** die Berechnung versucht wird
**Then** wird der letzte bekannte Wert als Fallback verwendet
**And** das Ergebnis enthält ein `confidence`-Feld das den Fallback kennzeichnet

**DB-Schema (in dieser Story erstellt):**
- `yield_snapshots` (TimescaleDB Hypertable): id, chain, validator_id (FK), nominal_apy, inflation_rate, validator_fee, slashing_risk, real_yield, snapshot_date, created_at

**Tests:** yield-calculator.test.ts (P0 Coverage — Domain-Logik, Grenzfälle, negative Real Yields, fehlende Daten)

### Story 3.2: Dashboard Real-Yield Integration

As a authentifizierter User,
I want den Real Yield pro Staking-Position und als Portfolio-Gesamt auf meinem Dashboard sehen,
So that ich sofort verstehe, was mein Staking wirklich einbringt.

**Acceptance Criteria:**

**Given** ein authentifizierter User öffnet das Dashboard
**When** die Daten geladen sind
**Then** zeigt die KPI-Row einen zusätzlichen/aktualisierten KPI-Card: "Portfolio Real Yield" als prominenteste Zahl (FR16)
**And** positiver Real Yield wird in `status-positive` (Grün) mit "+" Vorzeichen angezeigt
**And** negativer Real Yield wird in `status-negative` (Rot) mit "−" Vorzeichen angezeigt
**And** die Zahl hat 1 Dezimalstelle (z.B. +2.7%, −1.3%)

**Given** die Validator-Tabelle im Dashboard
**When** die Daten angezeigt werden (FR15)
**Then** enthält jede Validator-Row eine "Real Yield"-Spalte
**And** der Wert ist farbcodiert (Grün/Rot) mit Vorzeichen
**And** auf Mobile: Real Yield ist in der kompakten Card sichtbar (nicht versteckt)

**Given** ein User will Real Yields vergleichen (FR17)
**When** er das Dashboard mit mehreren Chains/Validators betrachtet
**Then** zeigt ein RealYieldComparison-Element (`dashboard`-Variante, kompakt) den Vergleich: Nominal APY vs. Real Yield als zwei Balken
**And** der Breakdown (Inflation, Fee, Slashing) ist per Klick/Tap erweiterbar (Progressive Disclosure)
**And** auf Mobile: Breakdown öffnet als Bottom Sheet

**Given** ein User wechselt den Chain-Filter
**When** "Alle" ausgewählt ist
**Then** zeigt der Portfolio Real Yield den gewichteten Durchschnitt über alle Chains
**When** eine einzelne Chain ausgewählt ist
**Then** zeigt er den Real Yield nur für diese Chain

**API:** `GET /api/yield?chainId={chain}` liefert Real-Yield-Daten pro Position

### Story 3.3: Public Real-Yield-Rechner (/calculator)

As a nicht-authentifizierter Besucher,
I want meinen Real Yield berechnen können ohne mich einloggen zu müssen,
So that ich sofort verstehe was mein Staking wirklich einbringt und motiviert werde StakeTrack AI zu nutzen.

**Acceptance Criteria:**

**Given** ein Besucher öffnet `/calculator` (FR18)
**When** die Seite geladen wird
**Then** ist die Seite SSR-optimiert mit First Load JS < 100KB (NFR4)
**And** er sieht: Headline "Was verdienst du WIRKLICH?", Chain-Selector (ETH/SOL/ATOM), Betrags-Input, optionaler Validator-Dropdown
**And** ein dezenter Disclaimer ist sichtbar: "Keine Finanzberatung" (NFR33)
**And** kein Login oder Registrierung ist erforderlich

**Given** ein Besucher wählt eine Chain und gibt einen Betrag ein
**When** die Eingabe erfolgt
**Then** berechnet das System live (debounced 300ms) — kein Submit-Button nötig
**And** das Ergebnis erscheint in < 1 Sekunde

**Given** das Ergebnis wird angezeigt
**When** der Real-Yield-Schock-Balken (`calculator`-Variante) erscheint
**Then** zeigt er zwei horizontale Balken: Nominal APY (grauer Balken) oben, Real Yield (Grün/Rot) darunter
**And** die Balken erscheinen mit kurzer Enthüllungs-Animation (Nominal zuerst, Real danach — der Schock-Effekt)
**And** die Animation respektiert `prefers-reduced-motion` (dann statisch)
**And** darunter: Formel-Breakdown (− Inflation X%, − Validator Fee Y%, − Slashing Z%)
**And** bei negativem Real Yield erscheint ein Hinweis in `status-negative`: "Du verlierst effektiv Wert"

**Given** ein Besucher sieht das Ergebnis
**When** er auf "Teile deinen echten Ertrag" klickt
**Then** wird ein teilbarer Link generiert (mit Chain + Betrag als Query-Parameter)
**And** ein Toast bestätigt: "Link kopiert!"

**Given** ein Besucher sieht das Ergebnis (FR19)
**When** er den CTA-Bereich unterhalb des Ergebnisses sieht
**Then** zeigt ein sachlicher CTA: "Verbinde deine Wallet für dein persönliches Portfolio" mit Secondary Button "Connect Wallet"
**And** der CTA ist nicht drängend — kein Modal, kein FOMO-Text

**Given** ein Besucher gibt einen Betrag von 0 oder negativ ein
**When** die Validierung greift (Zod)
**Then** wird keine Berechnung ausgeführt
**And** ein Inline-Error erscheint: "Betrag muss größer als 0 sein"

**Given** ein bereits authentifizierter User öffnet `/calculator`
**When** die Seite geladen wird
**Then** wird er zum Dashboard mit Calculator-Tab weitergeleitet (personalisierte Erfahrung statt generischer Rechner)

**Given** die Chain-API-Daten nicht verfügbar sind
**When** der Calculator geladen wird
**Then** werden letzte bekannte Durchschnittswerte verwendet
**And** ein Info-Banner zeigt: "Daten von [Timestamp]"

**API:** `POST /api/yield/calculate` (Public, Rate-Limited) — akzeptiert chain, amount, validatorId (optional)

**Komponenten:** YieldCalculatorForm, RealYieldComparison (`calculator`-Variante mit Animation), YieldResultCard

**SEO:** Meta-Tags, Open Graph Image, strukturierte Daten für Suchmaschinen

## Epic 4: Validator Risk Scoring (VRS)

User können die Vertrauenswürdigkeit von Validators über einen 0-100 Score mit Ampel-Darstellung bewerten — mit Faktoren-Aufschlüsselung, Confidence Level, historischem Trend und Tier-Gating für Free/Pro.

### Story 4.1: VRS Engine — Scoring-Logik & Konfiguration

As a System-Betreiber,
I want dass VRS-Scores korrekt, zeitgewichtet und deterministisch berechnet werden,
So that User verlässliche Validator-Bewertungen erhalten.

**Acceptance Criteria:**

**Given** die VRS Engine (`src/domain/vrs/vrs-engine.ts`)
**When** `calculateVrsScore(validatorId, chain)` aufgerufen wird
**Then** berechnet sie einen Score von 0-100 basierend auf 5 Faktoren: Uptime (Gewicht 30%), Slashing-Historie (25%), Commission-Stabilität (20%), Performance (15%), Dezentralisierung (10%)
**And** jeder Faktor-Score ist einzeln im Ergebnis enthalten (0-100 pro Faktor)
**And** die Gewichte sind in `vrs-config.ts` konfigurierbar ohne Code-Änderung
**And** die Berechnung ist deterministisch (gleiche Inputs = gleiches Ergebnis)

**Given** die Zeitgewichtung (FR24)
**When** historische Daten in die Berechnung einfließen
**Then** werden die letzten 30 Tage mit 70% gewichtet und die 31-90 Tage mit 30%
**And** Daten älter als 90 Tage fließen NICHT in die aktuelle Berechnung ein
**And** die Zeitfenster sind konfigurierbar

**Given** das Confidence Level (FR23)
**When** ein VRS-Score berechnet wird
**Then** enthält das Ergebnis ein `confidence`-Feld: "high" (alle 5 Datenquellen verfügbar, ≥60 Tage Historie), "medium" (3-4 Datenquellen oder 30-59 Tage Historie), "low" (<3 Datenquellen oder <30 Tage Historie)
**And** das Confidence Level wird mit dem Score gespeichert

**Given** der QStash Cron-Job `/api/cron/calculate-vrs`
**When** der Cron nach jedem Daten-Fetch feuert (FR39)
**Then** werden VRS-Scores für alle aktiven Validators berechnet
**And** die Scores werden als immutable Einträge in `vrs_scores` gespeichert (FR25, NFR27)
**And** der Cache wird invalidiert

**Given** nicht ausreichend Daten für einen Validator
**When** weniger als 7 Tage Historie vorhanden sind
**Then** wird KEIN VRS-Score berechnet
**And** der Validator erhält den Status `vrs_pending`

**DB-Schema (in dieser Story erstellt):**
- `vrs_scores` (TimescaleDB Hypertable): id, chain, validator_id (FK), total_score, uptime_score, slashing_score, commission_score, performance_score, decentralization_score, confidence (enum: high/medium/low), calculated_at, created_at
- `vrs_config`: id, factor_name, weight, time_window_recent_days, time_window_recent_weight, updated_at

**Tests:** vrs-engine.test.ts (P0 Coverage — Scoring-Logik, Zeitgewichtung, Confidence Levels, Edge Cases, Determinismus)

### Story 4.2: VRS-Ampel & Dashboard-Integration

As a authentifizierter User,
I want den VRS-Score jedes Validators als Ampel-Darstellung (Grün/Gelb/Rot) auf meinem Dashboard sehen,
So that ich sofort erkenne welchen Validators ich vertrauen kann.

**Acceptance Criteria:**

**Given** das Dashboard zeigt die Validator-Tabelle
**When** VRS-Scores verfügbar sind (FR21)
**Then** zeigt jede Validator-Row ein VrsScoreBadge: Grün (80-100): "Niedrig" + Score, Gelb (50-79): "Mittel" + Score, Rot (0-49): "Hoch" + Score
**And** das Badge hat `aria-label="Validator Risk Score: [Score] von 100, Risiko [Stufe]"`
**And** auf Mobile: Das Badge ist kompakt (nur Farbe + Score, kein Label-Text)

**Given** ein VrsScoreBadge wird angezeigt
**When** der Score ein "low" Confidence hat
**Then** zeigt ein ConfidenceBadge daneben: "Begrenzte Daten" (subtil, nicht alarmierend)
**And** bei "medium" Confidence: kein Badge (Standard)
**And** bei "high" Confidence: kein Badge (Standard)

**Given** ein Validator hat den Status `vrs_pending`
**When** das Dashboard geladen wird
**Then** zeigt statt des Badges: "Score wird berechnet..." als grauer Text
**And** kein Ampel-Farbcode

**Given** die Validator-Tabelle sortierbar ist
**When** der User nach VRS-Score sortiert
**Then** werden Validators absteigend nach Score sortiert (höchster Score = geringstes Risiko zuerst)
**And** Validators ohne Score erscheinen am Ende

**Komponenten:** VrsScoreBadge (mit `size: compact | full`), ConfidenceBadge

### Story 4.3: VRS-Detail-Ansicht & Faktoren-Aufschlüsselung

As a authentifizierter User,
I want die Aufschlüsselung des VRS in seine einzelnen Faktoren sehen,
So that ich verstehe WARUM ein Validator einen bestimmten Score hat.

**Acceptance Criteria:**

**Given** ein User klickt auf ein VrsScoreBadge im Dashboard
**When** die Detail-Ansicht öffnet (FR22)
**Then** zeigt ein Slide-Panel (Desktop) oder Bottom Sheet (Mobile): Validator-Name und Gesamt-Score prominent, 5 Faktoren als horizontale Balken: Uptime (0-100), Slashing-Historie (0-100), Commission-Stabilität (0-100), Performance (0-100), Dezentralisierung (0-100)
**And** jeder Balken ist farbcodiert (Grün/Gelb/Rot nach Score-Schwelle)
**And** der Gesamt-Score zeigt die Gewichtung: "30% Uptime, 25% Slashing, ..."

**Given** das Confidence Level wird angezeigt (FR23)
**When** der User die Detail-Ansicht sieht
**Then** zeigt ein Info-Bereich: Confidence Level (High/Medium/Low), Datenbasis: "Basierend auf [X] Tagen Historie", bei Low: "Mehr Daten werden in den nächsten Tagen verfügbar"

**Given** ein Free-User sieht die Detail-Ansicht (FR26)
**When** nur 3 Faktoren im Free-Tier verfügbar sind
**Then** zeigt die Ansicht die 3 Free-Faktoren (Uptime, Slashing, Commission) vollständig
**And** die 2 Pro-Faktoren (Performance, Dezentralisierung) sind als geblurrte/gesperrte Zeilen sichtbar
**And** ein Upgrade-Hinweis erscheint: "Schalte alle 5 Faktoren frei → Pro" (sachlich, kein FOMO)

**Given** der User nutzt Keyboard-Navigation
**When** er Tab drückt
**Then** kann er alle Faktoren-Balken fokussieren
**And** jeder Balken hat `aria-label="[Faktor]: [Score] von 100"`

**API:** `GET /api/vrs/{validatorId}` liefert Gesamt-Score + Faktoren

### Story 4.4: VRS-Historie & Trend-Anzeige

As a authentifizierter User,
I want den historischen Verlauf des VRS-Scores eines Validators sehen,
So that ich Trends und Veränderungen erkennen kann.

**Acceptance Criteria:**

**Given** die VRS-Detail-Ansicht eines Validators
**When** der User den "Verlauf"-Tab öffnet (FR25)
**Then** zeigt ein Tremor AreaChart den VRS-Score der letzten 30 Tage
**And** die X-Achse zeigt Daten (Format: "TT.MM"), die Y-Achse den Score (0-100)
**And** die Ampel-Zonen sind als horizontale Hintergrundbänder sichtbar (Grün/Gelb/Rot)
**And** Hover/Touch zeigt Tooltip: Datum, Score, Delta zum Vortag

**Given** ein signifikanter Score-Rückgang
**When** der Score um >10 Punkte innerhalb von 7 Tagen fällt
**Then** markiert ein roter Punkt die kritische Stelle im Chart
**And** ein Hinweis-Text erklärt: "Score gefallen um [X] Punkte seit [Datum]"

**Given** ein Free-User betrachtet die Historie
**When** die Daten angezeigt werden
**Then** zeigt der Chart nur die letzten 7 Tage (statt 30)
**And** ein Upgrade-Hinweis: "30-Tage-Historie mit Pro freischalten"

**Given** weniger als 7 Tage Historie vorhanden
**When** der Chart angezeigt wird
**Then** zeigt er die verfügbaren Datenpunkte
**And** ein Info-Hinweis: "Historische Daten werden ab [Datum des ersten Scores] gesammelt"

**API:** `GET /api/vrs/{validatorId}/history?days=30` liefert historische Score-Daten

### Story 4.5: VRS Tier-Gating

As a Plattform-Betreiber,
I want dass Free-User einen reduzierten VRS sehen und Pro-User den vollen Score,
So that der VRS ein klarer Upgrade-Treiber ist.

**Acceptance Criteria:**

**Given** ein Free-User (FR26)
**When** VRS-Scores angezeigt werden
**Then** wird der Gesamt-Score auf Basis von nur 3 Faktoren berechnet (Uptime, Slashing, Commission)
**And** der Score ist weiterhin auf 0-100 skaliert (Gewichte werden auf 3 Faktoren normalisiert)
**And** ein Label "Basic VRS" erscheint neben dem Score

**Given** ein Pro-User
**When** VRS-Scores angezeigt werden
**Then** wird der volle 5-Faktor-Score berechnet und angezeigt
**And** ein Label "Full VRS" erscheint neben dem Score

**Given** die VRS-Engine berechnet Scores
**When** ein Score-Request eintrifft
**Then** prüft die API den User-Tier via `userTier` aus der Session
**And** bei Free: `calculateVrsScore(validatorId, chain, { factors: 3 })`
**And** bei Pro: `calculateVrsScore(validatorId, chain, { factors: 5 })`
**And** der Tier wird NICHT im Score-Record gespeichert — die Filterung geschieht bei der Auslieferung

**Given** ein Free-User sieht das VrsScoreBadge
**When** er darauf klickt
**Then** zeigt die Detail-Ansicht 3 offene + 2 gesperrte Faktoren (wie in Story 4.3)
**And** der Upgrade-CTA ist prominent aber sachlich

**Given** ein User upgradet von Free auf Pro
**When** der Tier wechselt
**Then** zeigt das Dashboard sofort den vollen 5-Faktor-Score (Cache-Invalidierung)
**And** kein Page-Reload nötig (TanStack Query refetch)

## Epic 5: Alert & Benachrichtigungs-System

User können Email-Alerts für VRS-Änderungen, Commission-Erhöhungen und Slashing-Events konfigurieren — mit Tier-Limits, zuverlässiger Zustellung und DSGVO-konformem Unsubscribe.

### Story 5.1: Alert Engine & Email-Infrastruktur

As a System-Betreiber,
I want dass Alerts zuverlässig erkannt und als Emails versendet werden,
So that User rechtzeitig über wichtige Validator-Änderungen informiert werden.

**Acceptance Criteria:**

**Given** die Alert Engine (`src/domain/alerts/alert-engine.ts`)
**When** nach einem VRS-/Daten-Update der Check läuft
**Then** prüft die Engine alle aktiven Alerts gegen die aktuellen Daten
**And** unterstützt 3 Alert-Typen: VRS-Änderung (>konfigurierbarer Schwellwert, z.B. 10 Punkte Delta, FR27), Commission-Erhöhung (FR28), Slashing-Event (FR29)
**And** für jeden ausgelösten Alert wird ein Eintrag in `alert_history` erstellt

**Given** ein Alert wird ausgelöst
**When** die Email versendet werden soll
**Then** wird Resend als Email-Provider verwendet (`src/infrastructure/email/resend-client.ts`)
**And** bei Resend-Fehler: Retry via QStash (at-least-once Delivery)
**And** Logging ohne PII (keine Email-Adressen oder Wallet-Adressen in Logs)

**Given** derselbe Alert-Condition
**When** innerhalb von 24 Stunden erneut erkannt wird
**Then** wird KEIN erneuter Alert gesendet (Deduplication via `alert_history.triggered_at`)
**And** nach 24 Stunden wird der Alert bei erneutem Auftreten wieder gesendet

**Given** der QStash Cron `/api/cron/check-alerts`
**When** nach VRS-Berechnung und Daten-Updates gefeuert wird
**Then** läuft der Alert-Check als letzter Schritt in der Pipeline (fetch → VRS-calc → alert-check)
**And** die HMAC-Signatur wird verifiziert
**And** bei Fehler wird geloggt und QStash retried automatisch

**DB-Schema (in dieser Story erstellt):**
- `alerts`: id, user_id (FK), alert_type (enum: vrs_change/commission_increase/slashing_event), chain, validator_id (FK, nullable — null = alle Validators), threshold (jsonb), is_active (boolean), created_at, updated_at
- `alert_history`: id, alert_id (FK), triggered_at, payload (jsonb), email_sent_at, created_at

**Tests:** alert-engine.test.ts (P0 — Trigger-Logik, Deduplication, Schwellwert-Prüfung)

### Story 5.2: Email-Templates & Benachrichtigungs-Inhalt

As a User der einen Alert erhält,
I want eine klare, informative Email erhalten,
So that ich sofort verstehe was passiert ist und was ich tun kann.

**Acceptance Criteria:**

**Given** ein VRS-Alert wird gesendet
**When** die Email generiert wird
**Then** enthält sie: Betreff "[StakeTrack] VRS-Score geändert: [Validator] auf [Chain]", Validator-Name + Chain-Badge, VRS-Score Vorher → Nachher (mit Ampel-Farbe), betroffene Faktoren (welcher Faktor hat sich verschlechtert), Deep-Link zum Dashboard: "Im Dashboard ansehen →"

**Given** ein Commission-Alert wird gesendet
**When** die Email generiert wird
**Then** enthält sie: Betreff "[StakeTrack] Commission erhöht: [Validator] auf [Chain]", Alte Commission → Neue Commission (z.B. 5% → 8%), Kontext: "Das reduziert deinen effektiven Yield", Deep-Link zum Dashboard

**Given** ein Slashing-Alert wird gesendet
**When** die Email generiert wird
**Then** enthält sie: Betreff "[StakeTrack] Slashing-Event: [Validator] auf [Chain]", Art des Events, betroffener Betrag (falls verfügbar), Deep-Link zum Dashboard
**And** die Email nutzt `status-negative` Farbgebung für Dringlichkeit

**Given** jede Alert-Email
**When** sie gesendet wird
**Then** enthält sie: StakeTrack Logo im Header, Brand-konforme Gestaltung, einen Unsubscribe-Link (One-Click, DSGVO Art. 7 Abs. 3), einen Plain-Text Fallback, Disclaimer "Keine Finanzberatung" im Footer

**Given** ein User klickt "Unsubscribe" in der Email
**When** der Link aufgerufen wird
**Then** wird der spezifische Alert deaktiviert (`is_active = false`)
**And** eine Bestätigungs-Seite zeigt: "Alert deaktiviert. Du kannst ihn jederzeit in den Einstellungen wieder aktivieren."

**Komponenten:** `src/infrastructure/email/templates/` — vrs-alert.tsx, commission-alert.tsx, slashing-alert.tsx (React Email oder HTML)

### Story 5.3: Alert-Verwaltung UI

As a authentifizierter User,
I want meine Alerts konfigurieren, aktivieren und deaktivieren können,
So that ich nur die Benachrichtigungen erhalte die mir wichtig sind.

**Acceptance Criteria:**

**Given** ein authentifizierter User öffnet `/settings/alerts`
**When** die Seite geladen wird
**Then** sieht er eine Liste aller konfigurierten Alerts mit: Alert-Typ Icon + Label, Chain + Validator-Name, Schwellwert, Status-Toggle (aktiv/inaktiv)
**And** bei keinen Alerts: Empty State "Noch keine Alerts konfiguriert" mit CTA "Alert erstellen"

**Given** ein User klickt "Alert erstellen"
**When** das Formular erscheint (Desktop: Inline-Form, Mobile: Bottom Sheet)
**Then** kann er wählen: Alert-Typ (VRS-Änderung / Commission-Erhöhung / Slashing-Event), Chain (ETH/SOL/ATOM oder "Alle"), Validator (Dropdown mit Suchfunktion oder "Alle überwachten"), Schwellwert (nur bei VRS: Score-Delta, z.B. 10 Punkte)
**And** bei Bestätigung wird der Alert in `alerts` gespeichert und sofort aktiv

**Given** ein User hat keinen Email verknüpft (Wallet-Only Auth)
**When** er den ersten Alert erstellen will
**Then** wird er aufgefordert eine Email-Adresse einzugeben
**And** die Email wird verifiziert (Bestätigungs-Email mit Code/Link)
**And** erst nach Verifizierung wird der Alert aktiviert

**Given** ein User toggled einen Alert auf "inaktiv"
**When** der Toggle geklickt wird
**Then** wird `is_active = false` gesetzt (Optimistic UI)
**And** keine Emails werden mehr für diesen Alert gesendet
**And** der Alert bleibt in der Liste (kann reaktiviert werden)

**Given** ein User klickt "Alert löschen"
**When** der Bestätigungs-Dialog erscheint
**Then** zeigt er: "Alert für [Typ] auf [Validator] löschen?"
**And** bei Bestätigung: Alert wird aus `alerts` gelöscht (Hard Delete)
**And** zugehörige `alert_history`-Einträge bleiben erhalten (Audit-Trail)

**Given** ein Free-User hat bereits 1 Alert konfiguriert (FR30)
**When** er einen weiteren erstellen will
**Then** zeigt die UI: "Du hast dein Alert-Limit erreicht (1/1)"
**And** ein Upgrade-CTA: "Mit Pro bis zu 5 Alerts konfigurieren"
**And** der "Erstellen"-Button ist disabled

**Given** ein Pro-User
**When** er Alerts verwaltet (FR30)
**Then** kann er bis zu 5 Alerts konfigurieren
**And** der Zähler zeigt: "[X]/5 Alerts"

### Story 5.4: Alert-Cron & Tier-Enforcement

As a System-Betreiber,
I want dass Alert-Limits pro Tier serverseitig durchgesetzt werden und der Alert-Check effizient läuft,
So that das System skalierbar bleibt und Tier-Grenzen nicht umgangen werden können.

**Acceptance Criteria:**

**Given** die Alert-Pipeline
**When** der Cron-Job läuft
**Then** ist die Reihenfolge: fetch-validators → calculate-vrs → calculate-yield → check-alerts
**And** check-alerts läuft nur wenn die vorherigen Steps erfolgreich waren

**Given** viele aktive Alerts
**When** der Alert-Check läuft
**Then** werden alle Alerts desselben Typs in einer Batch-Query geprüft (nicht N+1)
**And** ausgelöste Alerts werden als QStash Batch-Messages an Resend versendet
**And** Rate-Limiting gegenüber Resend wird respektiert (max 10 Emails/Sekunde)

**Given** Tier-Enforcement
**When** ein API-Request zum Erstellen eines Alerts eintrifft
**Then** prüft die API serverseitig: Anzahl aktive Alerts des Users vs. Tier-Limit
**And** bei Free + bereits 1 aktiv: 403 mit `{ error: "Alert limit reached", code: "TIER_LIMIT" }`
**And** bei Pro + bereits 5 aktiv: 403 mit gleicher Struktur
**And** die Prüfung erfolgt NICHT nur in der UI (Defense in Depth)

**Given** Alert-History Retention
**When** Einträge älter als 90 Tage sind
**Then** werden sie durch einen monatlichen Cleanup-Job entfernt
**And** der Cleanup läuft als separater QStash Cron (nicht im Alert-Check)

**Given** ein Massenauslösungs-Szenario (>100 Alerts gleichzeitig)
**When** z.B. ein großes Slashing-Event auftritt
**Then** werden Emails in Batches versendet (QStash Batch)
**And** die Gesamtverarbeitung bleibt unter 5 Minuten
**And** bei partiellen Fehlern: erfolgreiche Emails werden nicht wiederholt

## Epic 6: Freemium-Gating & Subscription-Management

User erleben ein faires Freemium-Modell mit klaren Upgrade-Pfaden — Free-User bekommen genug Wert um den Nutzen zu erkennen, Pro-User ($19/mo) erhalten vollen Zugang ohne künstliche Einschränkungen.

### Story 6.1: Tier-System & Feature-Gate Middleware

As a System-Betreiber,
I want ein zentrales Tier-System das Free/Pro-Limits serverseitig durchsetzt,
So that Feature-Gating konsistent und nicht umgehbar ist.

**Acceptance Criteria:**

**Given** die Tier-Konfiguration (`src/domain/subscription/tier-config.ts`)
**When** die Tier-Definitionen geladen werden
**Then** existieren zwei Tiers: Free (1 Chain, 10 Validators, 3-Faktor VRS, 1 Alert, 7-Tage-Historie), Pro (3 Chains, unbegrenzt Validators, 5-Faktor VRS, 5 Alerts, 30-Tage-Historie)
**And** die Limits sind als typisiertes Config-Objekt definiert (nicht Magic Numbers im Code)

**Given** ein neuer User registriert sich
**When** der Account erstellt wird
**Then** wird automatisch ein `user_subscriptions`-Eintrag mit `tier: free` angelegt
**And** der Tier wird in die JWT-Session-Claims aufgenommen (schneller Client-Check)

**Given** die Feature-Gate Middleware (`src/lib/auth/feature-gate.ts`)
**When** ein gated API-Endpoint aufgerufen wird
**Then** prüft `checkFeatureAccess(userId, feature)` den Tier des Users
**And** bei Zugang: Request wird durchgelassen
**And** bei Limit erreicht: 403 mit `{ error: "Feature requires Pro", code: "TIER_LIMIT", upgrade_url: "/settings/subscription" }`

**Given** die Feature-Gate Middleware
**When** ein Chain-bezogener Endpoint aufgerufen wird (FR33)
**Then** prüft sie: Free-User dürfen nur 1 Chain verknüpfen
**And** bei Versuch eine 2. Chain hinzuzufügen: 403 mit Tier-Limit-Response

**Given** serverseitiges Enforcement
**When** jeder gated Endpoint aufgerufen wird
**Then** wird der Tier IMMER serverseitig geprüft (Defense in Depth)
**And** Client-Side Checks (JWT-Claims) dienen nur der UX, nicht der Sicherheit

**DB-Schema (in dieser Story erstellt):**
- `user_subscriptions`: id, user_id (FK → users, unique), tier (enum: free/pro), started_at, expires_at (nullable — null = kein Ablauf), stripe_subscription_id (nullable), stripe_customer_id (nullable), created_at, updated_at

**Tests:** feature-gate.test.ts (P0 — alle Tier-Kombinationen, Edge Cases: expired Pro, missing subscription)

### Story 6.2: Feature-Gating UI & Upgrade-CTAs

As a Free-User,
I want klar sehen welche Features eingeschränkt sind und wie ich upgraden kann,
So that ich den Wert von Pro verstehe ohne mich bedrängt zu fühlen.

**Acceptance Criteria:**

**Given** ein Free-User nutzt die App
**When** er ein Tier-limitiertes Feature erreicht (FR34, FR35)
**Then** zeigen sachliche Upgrade-Hinweise den Wert von Pro:
- Chain-Limit: "Mit Pro alle 3 Chains überwachen" (beim Versuch 2. Chain hinzuzufügen)
- VRS: "Basic VRS (3 Faktoren)" Label + "Alle 5 Faktoren mit Pro" in Detail-Ansicht
- Alerts: "[1/1] Alerts" + "Bis zu 5 Alerts mit Pro"
- Historie: "7-Tage-Historie" + "30 Tage mit Pro"
**And** Upgrade-CTAs sind Secondary Buttons, kein Modal, kein Pop-up
**And** kein FOMO-Text, kein Countdown-Timer, keine aggressive Wortwahl

**Given** ein React Hook `useFeatureGate(feature)`
**When** er in einer Komponente aufgerufen wird
**Then** liefert er: `{ allowed: boolean, limit: number, current: number, upgradeHint: string }`
**And** der Hook liest den Tier aus den JWT-Session-Claims (kein API-Call)

**Given** ein Pro-User
**When** er die App nutzt
**Then** erscheint ein dezentes Pro-Badge im Navigation-Header (`brand-teal` Farbe)
**And** keine Upgrade-CTAs werden angezeigt
**And** alle Features sind ohne Einschränkung nutzbar

**Given** gesperrte Features in der UI
**When** ein Free-User ein gesperrtes Element sieht (z.B. geblurrte VRS-Faktoren, gesperrte Historie)
**Then** ist das Element visuell erkennbar eingeschränkt (Blur, Lock-Icon, grauer Overlay)
**And** ein Klick darauf zeigt den Upgrade-Hinweis inline (nicht als Modal)
**And** der "Upgrade"-Button führt zu `/settings/subscription`

**Komponenten:** FeatureGate (Wrapper-Komponente), UpgradeHint, ProBadge, useFeatureGate Hook

### Story 6.3: Subscription-Seite & Upgrade-Information

As a User,
I want meinen aktuellen Plan sehen und verstehen was Pro bietet,
So that ich eine informierte Upgrade-Entscheidung treffen kann.

**Acceptance Criteria:**

**Given** ein Free-User öffnet `/settings/subscription`
**When** die Seite geladen wird
**Then** sieht er: Aktueller Plan "Free" mit Feature-Liste, Feature-Vergleichstabelle (Free vs Pro) mit allen Limits, Kontakt-Information für Upgrade ("Schreibe an upgrade@staketrack.ai für Pro-Zugang — $19/Monat")

**Given** ein Pro-User öffnet `/settings/subscription`
**When** die Seite geladen wird
**Then** sieht er: Aktueller Plan "Pro" mit Feature-Liste, Pro-Badge im Header, Hinweis "Dein Pro-Zugang wird von einem Admin verwaltet"

**Given** ein Free-User klickt auf den Upgrade-Kontakt-Link
**When** der Link aufgerufen wird
**Then** öffnet sich der Default-Mail-Client mit vorausgefülltem Betreff "StakeTrack AI Pro Upgrade"

**Given** ein Admin hat den Tier eines Users auf Pro gesetzt (via Story 6.4)
**When** der User die App öffnet
**Then** wird der neue Tier sofort wirksam (JWT-Session-Cache invalidiert)
**And** alle Pro-Features sind freigeschaltet
**And** das Pro-Badge erscheint im Navigation-Header

**Hinweis:** Stripe Self-Service Payment ist Post-MVP (Phase 2, Trigger: >50 Pro-User). Im MVP erfolgt das Onboarding manuell via Admin (Story 6.4).

### Story 6.4: Admin Tier-Management

As a Admin (Sempre),
I want User-Tiers manuell verwalten können,
So that ich bei Support-Fällen oder Beta-Testern Tiers anpassen kann.

**Acceptance Criteria:**

**Given** ein Admin (identifiziert via hardcoded Wallet-Adresse)
**When** er `/admin/subscriptions` öffnet
**Then** sieht er eine Tabelle aller User mit: User-ID (gekürzt), Auth-Methode (Wallet/Email), aktueller Tier, Registrierungsdatum

**Given** der Admin die User-Liste sieht
**When** er nach einem User sucht
**Then** kann er nach Wallet-Adresse oder Email filtern
**And** die Suche ist serverseitig (kein Client-Filter bei vielen Usern)

**Given** der Admin einen User-Tier ändern will
**When** er auf "Tier ändern" klickt
**Then** kann er zwischen Free und Pro wechseln
**And** manuell gesetzte Tiers haben `stripe_subscription_id = null` und `expires_at = null` (kein Auto-Expire)
**And** ein Bestätigungs-Dialog zeigt: "Tier von [User] auf [Neuer Tier] setzen?"

**Given** eine manuelle Tier-Änderung
**When** sie durchgeführt wird
**Then** wird ein Audit-Log-Eintrag erstellt: Admin-ID, User-ID, alter Tier, neuer Tier, Zeitstempel
**And** der User sieht den neuen Tier sofort (Cache-Invalidierung)

**Given** die Admin-Endpoints (`/api/admin/subscriptions`)
**When** ein Request eintrifft
**Then** wird Auth + Admin-Check durchgeführt (Wallet-Adresse muss mit hardcoded Admin-Adresse übereinstimmen)
**And** bei Nicht-Admin: 403
**And** Rate-Limiting greift auch für Admin-Endpoints

**DB-Schema (in dieser Story ergänzt):**
- `admin_audit_log`: id, admin_user_id (FK), action, target_user_id (FK), old_value (jsonb), new_value (jsonb), created_at
