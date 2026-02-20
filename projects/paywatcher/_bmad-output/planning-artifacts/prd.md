---
stepsCompleted: [step-01-init, step-02-discovery, step-03-success, step-04-journeys, step-05-domain, step-06-innovation, step-07-project-type, step-08-scoping, step-09-functional, step-10-nonfunctional, step-11-polish, step-12-complete]
status: 'complete'
completedAt: '2026-02-18'
inputDocuments:
  - _bmad-output/planning-artifacts/product-brief-paywatcher-2026-02-15.md
  - docs/_masemIT/product-brief-paywatcher-phase1-frontend-v2-2026-02-17.md
  - mms:_bmad-output/planning-artifacts/research/market-paywatcher-research-2026-02-14.md
  - docs/_masemIT/company-info.md
documentCounts:
  briefs: 2
  research: 1
  brainstorming: 0
  projectDocs: 1
classification:
  projectType: web_app
  domain: fintech
  complexity: medium
  projectContext: greenfield
workflowType: 'prd'
---

# Product Requirements Document — PayWatcher Phase 1 Frontend

**Author:** Sempre
**Date:** 2026-02-18
**Project:** PayWatcher (masemIT Product)
**Domain:** paywatcher.dev
**Backend:** MMS Module (api.masem.at) — bereits implementiert
**Version:** Aktualisiert für Product Brief v2 (Managed MVP)

## Executive Summary

PayWatcher ist eine Developer-First Verification API für Stablecoin-Zahlungen auf EVM-Chains. Kein Payment Processor — PayWatcher bewegt kein Geld, hält keine Funds, macht keine Währungskonvertierung. Es ist ein reiner Verification Layer: "Sag mir welche Zahlung du erwartest, ich sage dir wenn sie angekommen und bestätigt ist."

**MVP-Fokus:** USDC auf Base Network.

**Differentiator:** Kein reines Verification-Only-Produkt existiert am Markt. Alle Wettbewerber bündeln Custody, Settlement oder Payment Processing. PayWatcher füllt die Lücke zwischen "zu high-level" (Payment Processors diktieren UX, nehmen 0.5-2%) und "zu low-level" (rohe Transfer Events, alles selbst bauen).

**Dieses PRD:** Definiert Phase 1 Frontend — Landing Page, User Dashboard, Auth/Login. Das Backend existiert bereits. Admin Dashboard ist Teil von Phase 1.

**MVP-Ansatz: Managed MVP.** Kein Self-Service Signup. Interessenten stellen eine Zugriffsanfrage über ein Request-Access-Formular auf der Landing Page. Mario (Operator) onboardet Tenants manuell über das Admin Dashboard und sendet API-Credentials per Email. Self-Service Signup ist für Phase 2 geplant.

**Zielgruppe:**
- **Primary:** Web3 Developer / Technical Founder — baut DApp/SaaS mit Stablecoin-Zahlungen, entscheidet anhand von Code-Beispielen
- **Secondary:** Crypto-Native Small Business — Freelancer/Agentur, weniger technisch, nutzt eher Dashboard als API

## Success Criteria

### User Success

- Developer versteht PayWatcher innerhalb von 30 Sekunden auf der Landing Page
- Erste Payment-Verification integriert innerhalb einer Stunde (mit Docs + Code Examples)
- Tenant kann API Keys einsehen und Settings verwalten ohne Support-Anfrage

### Business Success

| Metrik | Ziel (3 Monate nach Launch) |
|--------|----------------------------|
| Landing Page → Request Access Conversion | >3% |
| Request Access → Onboarded | >50% |
| Active Tenants | 5 Free + 2 Paid |
| Verifizierte Payments | 100+ |
| Webhook Success Rate | >99% |
| Average Confirmation Time | <30s |
| MRR | €87+ (1 Starter + 1 Pro) |

### Technical Success

- Landing Page läuft flüssig, keine spürbaren Performance-Probleme
- Dashboard reagiert responsiv bei API-Calls
- Stabile Integration mit MMS Backend (api.masem.at)

### Measurable Outcomes

- End-to-End Flow funktioniert: Request Access → Mario onboarded → Tenant Login → Create Payment → USDC senden → Webhook received
- Mindestens 1 externer Tenant (nicht masemIT) erfolgreich onboarded
- Landing Page live mit allen 8 Sektionen + Request Access Formular
- User Dashboard: API Key Anzeige + Payments-Übersicht + Webhook Delivery History + Settings funktional

## Project Scoping & Phased Development

### MVP Strategy

**Approach:** Problem-Solving MVP — den bestehenden Backend-Service für Kunden zugänglich und sichtbar machen. Managed MVP: Mario onboardet Tenants manuell, kein Self-Service Signup.

**Resource:** Solo-Developer (Mario/Sempre), bewährter masemIT Tech Stack. Backend existiert, Frontend wird neu gebaut.

**Kernprinzip:** Alles was ein Tenant braucht um PayWatcher zu finden, Zugang anzufragen, und den Service zu nutzen. Alles was nur Mario intern braucht, kann warten.

### Phase 1a — MVP

**Supported Journeys:**
- Journey 1 (Alex — Developer Success Path): Vollständig
- Journey 3 (Sandra — Freelancerin): Vollständig
- Journey 5 (Priya — Evaluator): Vollständig
- Journey 2 (Alex — Edge Case/Recovery): Teilweise (Payments-Übersicht ja, Detail-View als Stretch)

**Must-Have Capabilities:**

| Bereich | Feature | Begründung |
|---------|---------|-------------|
| **Landing Page** | Alle 8 Sektionen (Hero, Problem/Solution, How It Works, Code Examples, Pricing, Comparison, Trust Signals, Footer) | Conversion — ohne Landing Page kein Traffic, keine Request Access Anfragen |
| **Landing Page** | Request Access Formular (Name, Email, Company, Use Case) — Email via Resend an contact@masem.at | Managed MVP — kein Self-Service Signup, Mario onboardet manuell |
| **Landing Page** | SEO-Optimierung (Meta Tags, OG, Sitemap) | Organic Discovery für Developer-Audience |
| **Docs-Lite** | Quick Start + Endpoint Reference + Webhook Payload (eingebettet in Landing Page) | Devs brauchen Docs um die API zu integrieren |
| **Auth** | Magic Link Login (Resend) + Parties API Tenant-Resolution (POST /v1/auth/login) + Neon Postgres (Tenant API Keys) | Gate zum Dashboard, kein Signup nötig |
| **User Dashboard** | API Key Anzeige (masked) | Tenant muss seinen Key einsehen können |
| **User Dashboard** | Payments-Übersicht (Liste mit Status-Filter) | Tenants müssen ihre Payments sehen können |
| **User Dashboard** | Payment Detail mit Webhook Delivery History (GET /v1/payments/:id/webhooks) | Tenants brauchen Einblick in Webhook-Zustellungen pro Payment |
| **User Dashboard** | Webhook-Config (URL setzen, testen) | Ohne Webhook-Config funktioniert der Service nicht |
| **Analytics** | Landing Page Events (masemIT Analytics) | Conversion-Tracking ab Tag 1 |

### Phase 1b — Direkt nach Launch

| Feature | Begründung |
|---------|-------------|
| **Admin Dashboard** | System Overview, Tenant Management (CRUD), Global Payments, Webhook Health — sobald erste Tenants aktiv sind |
| **Payment Detail View** | Timeline, Block Explorer Link — verbessert Debugging für Tenants |
| **Dashboard Analytics Events** | Usage-Tracking im Dashboard |

### Phase 2 — Growth

- Self-Service Signup (automatische Tenant-Erstellung bei Registration)
- Rechnungen/Invoice Download im Dashboard
- API Key Regenerieren (Endpoint im MMS Backend)
- Vollständige Docs auf docs.paywatcher.dev (Fumadocs/Mintlify)
- Dashboard Monitoring-Features (Charts, Payment-Trends)
- Multi-Chain Support
- SDK/npm Package
- Stripe Billing Integration
- Public Status Page

### Phase 3 — Vision

- WebSocket Real-Time Updates
- HD Wallet (unique addresses per payment)
- Team/Multi-User per Tenant
- Custom Branding (White-Label)
- Interactive API Explorer
- Audit Log

### Risk Mitigation Strategy

**Technical Risks:**
- Auth-System: Magic Link Login via Resend + Parties API Tenant-Resolution. Frontend-DB (Neon Postgres) für verschlüsselte Tenant API Keys. Bewährte Bausteine (Auth.js v5, Drizzle ORM)
- API-Integration mit MMS Backend ist straightforward (REST, gut dokumentiert)

**Market Risks:**
- Zu wenig Organic Traffic → Content Marketing + Web3 Community Engagement ab Launch
- Request Access als Validierung: Wenn niemand Request Access stellt, stimmt das Product-Market-Fit nicht

**Resource Risks:**
- Solo-Developer → Admin Dashboard bewusst in Phase 1b verschoben, DB-Zugriff als Brücke
- Absolutes Minimum: Landing Page + Request Access + Magic Link Login + Payments-Übersicht

## User Journeys

### Journey 1: Alex — Web3 Developer (Primary, Success Path)

**Wer:** Alex, 29, Full-Stack Dev bei einem kleinen Web3 Startup in Berlin. Baut einen NFT-Marketplace, auf dem Künstler ihre Werke für USDC verkaufen können.

**Opening Scene:** Alex hat gerade 3 Wochen damit verbracht, einen eigenen Payment Listener auf Alchemy Webhooks zu bauen. Transfer Events parsen, Amount Matching, Confirmation Tracking, Timeout-Logik, Idempotency — alles selbst. Es funktioniert... meistens. Letzte Woche hat ein Reorg eine Zahlung verschluckt und ein Künstler hat sich beschwert. Alex googelt "stablecoin payment verification API" und landet auf paywatcher.dev.

**Rising Action:** Die Landing Page zeigt ein 3-Zeilen Code-Snippet: Create Intent → Customer Pays → Webhook Fires. Alex denkt: "Das ist genau die Abstraktion die mir fehlt." Er scrollt zu den Code Examples, kopiert das curl-Beispiel, sieht die Pricing-Tabelle — $0.05 pro Verification statt 1% bei Coinbase Commerce. Bei seinem Volumen wäre das Free Tier erstmal genug. Er klickt "Request Access", füllt das Formular aus (Name, Email, Projekt-URL, Use Case).

**Climax:** Innerhalb von 24 Stunden bekommt Alex eine Email von Mario mit seinen API-Credentials. Er loggt sich per Magic Link im Dashboard ein, sieht seinen Key (masked) und die Deposit Address. Alex ersetzt seinen 800-Zeilen Payment Listener durch 3 API Calls und einen Webhook Endpoint. Er erstellt einen Test-Payment-Intent, sendet 5.37 USDC von seiner Wallet, und 15 Sekunden später feuert der Webhook. `payment.confirmed`. Es funktioniert einfach.

**Resolution:** Alex löscht seinen selbstgebauten Payment Listener. Sein NFT-Marketplace hat jetzt zuverlässige Payment Verification ohne Reorg-Risiko. Er upgraded auf Starter als das Volumen wächst.

**Requirements:** Landing Page mit Code-Snippets, Request Access Formular, API Credentials per Email erhalten, Dashboard Login per Magic Link, Webhook Delivery History, Docs mit curl/JS Examples

---

### Journey 2: Alex — Web3 Developer (Edge Case: Fehler & Recovery)

**Opening Scene:** Alex hat PayWatcher seit 2 Wochen im Einsatz. Ein Kunde meldet sich: "Ich hab bezahlt aber die Bestellung ist noch offen."

**Rising Action:** Alex loggt sich ins User Dashboard ein, geht auf Payments, filtert nach "expired". Er findet den Payment — Status "expired", kein txHash. Der Kunde hat zu spät bezahlt, nach dem 15-Minuten-Timeout. Alex sieht in der Payment Detail View die Timeline: created → expired. Kein Match.

**Climax:** Alex erstellt einen neuen Payment Intent für den Kunden mit gleichem Betrag. Diesmal zahlt der Kunde sofort. Im Dashboard sieht Alex den Status live wechseln: pending → matched → confirming → confirmed. Webhook gefeuert.

**Resolution:** Alex baut in seinen Flow eine "Payment expired — retry" UX ein. Er passt in den Settings die Confirmation Override auf einen höheren Wert an, damit Kunden mehr Zeit haben. Problem gelöst, ohne Support-Ticket bei PayWatcher.

**Requirements:** Payment-Filter nach Status, Payment Detail mit Timeline, Retry-Fähigkeit (neuer Intent), Settings für Confirmation Override

---

### Journey 3: Sandra — Freelance Designerin (Secondary User)

**Wer:** Sandra, 34, UX-Designerin aus Wien. Nimmt seit einem Jahr Crypto-Zahlungen an, weil internationale Kunden damit schneller und günstiger zahlen als per Banküberweisung. Bisher nutzt sie Coinbase Commerce, zahlt aber 1% pro Transaktion.

**Opening Scene:** Sandra bekommt einen Auftrag über €8.000 von einem US-Startup. Bei Coinbase Commerce wären das €80 Gebühren — nur für die Bestätigung dass die USDC angekommen sind. Eine Freundin (Web3 Dev) empfiehlt ihr PayWatcher.

**Rising Action:** Sandra ist nicht super technisch, aber sie versteht APIs grundsätzlich. Sie geht auf paywatcher.dev, sieht "Payment verification without the payment processor" und "$0.05 to verify a $10,000 payment". Das überzeugt sie. Sie füllt das Request Access Formular aus. Am nächsten Tag bekommt sie eine Email von Mario mit ihrem API Key und einer kurzen Anleitung.

**Climax:** Sandra loggt sich per Magic Link im Dashboard ein. Ihr Freund hilft ihr, ein simples Invoicing-Script aufzusetzen. Sandra schickt ihrem Kunden eine Rechnung mit Deposit-Adresse und exaktem USDC-Betrag. Der Kunde zahlt, Sandra sieht im Dashboard den Webhook als zugestellt, markiert die Rechnung als bezahlt. Kosten: $0.05 statt €80.

**Resolution:** Sandra nutzt das Dashboard hauptsächlich um ihre Payments und Webhook-Status zu prüfen. Sie schaut gelegentlich in die Payments-Übersicht. Der Rest läuft automatisch über die API.

**Requirements:** Einfaches Request Access Formular, API Credentials per Email erhalten, Dashboard Login per Magic Link, Payments-Übersicht, Webhook Delivery History, klare Pricing-Kommunikation auf Landing Page

---

### Journey 4: Mario — Admin/Operator (Phase 1b)

**Wer:** Mario (Sempre), Gründer von masemIT. Einziger Operator des PayWatcher Service.

**Opening Scene:** Montagmorgen, Kaffee in der Hand. Mario öffnet das Admin Dashboard für den täglichen Health Check.

**Rising Action:** System Overview zeigt: alle Systeme grün. Last Poll vor 8 Sekunden, keine RPC Errors, Hot Wallet Balance ausreichend. 3 aktive Tenants, 47 Payments in den letzten 7 Tagen, 100% Webhook Success Rate. Ein kurzer Blick auf die Tenants-Liste — alle enabled, keine Auffälligkeiten.

**Climax:** Heute ist alles ruhig. Aber letzte Woche gab es eine Situation: Ein Tenant hatte seine Webhook-URL falsch konfiguriert, alle Deliveries gingen auf 404. Mario hat das im Webhooks-Tab gesehen (Failed Deliveries für einen spezifischen Tenant), den Tenant kontaktiert, Problem gelöst. Ohne Admin Dashboard hätte er das erst gemerkt wenn sich der Tenant beschwert hätte.

**Resolution:** Marios Idealzustand: 2 Minuten pro Tag im Admin Dashboard. Alles grün = weiterarbeiten. Wenn etwas rot ist, kann er sofort sehen was und wo.

**Requirements:** System Overview mit Health-Indikatoren, Tenant-Management, Global Webhook Health, Failed Deliveries mit Tenant-Zuordnung, Hot/Cold Wallet Balances, Sweep History

---

### Journey 5: Priya — Evaluator/Researcher

**Wer:** Priya, 31, CTO eines DeFi-Startups. Evaluiert Payment-Lösungen für ihr Produkt. Hat bereits Coinbase Commerce und NOWPayments verglichen.

**Opening Scene:** Priya findet PayWatcher über einen Blog-Post in einer Web3-Community. Sie klickt auf paywatcher.dev und ist skeptisch — noch ein Payment-Tool?

**Rising Action:** Priya scannt die Landing Page systematisch. Hero-Section: "Payment verification without the payment processor" — das ist anders. How It Works: nur 3 Steps, kein Checkout-Widget, kein Custody. Code Examples: clean, minimale API Surface. Pricing: Flat-Fee, nicht prozentual. Comparison-Tabelle: "$100 at Coinbase Commerce vs. $0.05 at PayWatcher for a $10k payment" — das ist der Killer für ihr Volumen.

**Climax:** Priya klickt auf "Request Access", füllt das Formular in 30 Sekunden aus. Am nächsten Tag hat sie ihren API Key per Email, loggt sich im Dashboard ein, und testet die API in 10 Minuten mit curl. Das funktioniert tatsächlich so einfach wie versprochen. Non-custodial bedeutet auch: keine regulatorischen Bedenken für ihr Startup.

**Resolution:** Priya präsentiert PayWatcher ihrem Team als Empfehlung. Argument: 95% günstiger als Coinbase Commerce, keine Custody-Risiken, saubere API. Sie starten mit Starter Tier.

**Requirements:** Klare Differenzierung auf Landing Page, Comparison-Tabelle, Request Access Formular, API Credentials per Email erhalten, Dashboard Login per Magic Link, Code Examples die in Minuten funktionieren, Trust Signals (non-custodial, no Money Transmitter)

---

### Journey Requirements Summary

| Capability | Journeys |
|-----------|----------|
| Landing Page mit Code-Snippets & Pricing | Alex, Sandra, Priya |
| Request Access Formular + Managed Onboarding | Alex, Sandra, Priya |
| User Dashboard: Payments-Übersicht & Filter | Alex, Sandra |
| User Dashboard: Payment Detail mit Timeline | Alex |
| User Dashboard: Webhook Delivery History pro Payment | Alex, Sandra |
| User Dashboard: API Key Anzeige (masked) | Alex, Sandra |
| User Dashboard: Settings (Webhook, Confirmations) | Alex |
| Admin Dashboard: System Health Overview | Mario (Phase 1b) |
| Admin Dashboard: Tenant Management | Mario (Phase 1b) |
| Admin Dashboard: Global Webhook Health | Mario (Phase 1b) |
| Admin Dashboard: Wallet Balances & Sweeps | Mario (Phase 1b) |
| Docs/Code Examples (curl, JS) | Alex, Priya |
| Comparison/Trust Signals auf Landing Page | Sandra, Priya |

## Domain-Specific Requirements

### Compliance & Regulatory

- **Kein Money Transmitter:** PayWatcher bewegt kein Geld, hält keine Funds, macht keine Währungskonvertierung. Explizit non-custodial positionieren
- **Keine KYC/AML-Pflicht im MVP:** Da kein Custody/Processing, entfällt die regulatorische Pflicht. Legal Review bei Skalierung eingeplant
- **DSGVO/GDPR:** Tenants sind EU-basiert möglich (masemIT ist in Österreich). Personenbezogene Daten: Email bei Request Access, API Keys. Standard-GDPR-Compliance (Privacy Policy, Data Processing)

### Technical Constraints

- **Webhook Security:** Webhook Payloads sollten signiert sein (HMAC), damit Tenants die Authentizität prüfen können
- **Non-Custodial Architektur:** Frontend hat keinen Zugriff auf Private Keys. Wallet-Management ist rein Backend-seitig (MMS)

### Integration Requirements

- **MMS Backend (api.masem.at):** Einzige Datenquelle. Frontend ist ein reiner API-Consumer
- **Basescan:** Block Explorer Links für txHash-Verifizierung
- **masemIT Analytics (analytics.masem.at):** Eigenes Analytics-System für Landing Page + Dashboard Events
- **Resend:** Email-Versand für Request Access Formular (an contact@masem.at)

### Risk Mitigations

- **Private Key Exposure:** Frontend hat keinerlei Zugang zu Wallet-Credentials — alles serverseitig im MMS Backend
- **API Key Leak durch Tenant:** Rate Limiting und Scope-Beschränkung begrenzen den Schaden
- **Webhook Replay Attacks:** Signierte Webhooks + Idempotency Keys

## Innovation & Novel Patterns

### Detected Innovation Areas

- **Verification-Only API:** Erste reine Verification API für Stablecoin-Payments — kein Custody, kein Processing, kein Settlement. Vergleichbar mit einer Bank-Zahlungsbestätigung, nicht mit einem Payment Processor
- **Shared Address + Unique Amount Pattern:** Aus dem Traditional Banking bekannt, als Crypto-Service bisher nicht existent. Eliminiert die Notwendigkeit für HD Wallets im MVP
- **Flat-Fee Pricing:** Strukturell anders als der Markt (0.5-2% pro Transaktion). $0.05 für eine $10.000 Zahlung vs. $100 bei Coinbase Commerce

### Market Context & Competitive Landscape

- Stablecoin Payment Infrastructure Markt ~$1.7B (2025), 13-19% CAGR
- Alle 7 analysierten Payment Processors sind custodial + %-basiert
- Alle 6 analysierten Infrastructure Provider liefern nur Raw Data (DIY)
- Kein Produkt besetzt die Mitte zwischen diesen beiden Extremen

### Validation Approach

- **Request Access als Validierung:** Anzahl der Request Access Anfragen zeigt Product-Market-Fit
- **Conversion-Metriken:** Landing Page → Request Access als früher Indikator für Product-Market-Fit
- **Erstes externes Tenant-Onboarding:** Proof of Concept dass der Value Proposition real ist

### Innovation Risk Mitigation

- **Falls Shared Address Pattern nicht skaliert:** HD Wallet Support in Phase 2 als Fallback geplant
- **Falls Flat-Fee nicht attraktiv genug:** Pricing kann angepasst werden, das Modell selbst ist der Differentiator
- **Falls "Verification Only" zu nischig:** Multi-Chain + mehr Features in Phase 2 erweitern die Zielgruppe

## Web App Specific Requirements

### Architecture Overview

Next.js 15 App Router Hybrid-Architektur: Statische Landing Page (SSG) + dynamische Dashboards (Client-Side). Drei Hauptbereiche in einer Applikation: Public Landing Page, authentifiziertes User Dashboard, authentifiziertes Admin Dashboard.

### Rendering-Strategie

- **Landing Page:** Static Site Generation (SSG) — maximale Performance, SEO-optimiert
- **User Dashboard:** Client-Side Rendering mit Server Components wo sinnvoll — dynamische Daten via API
- **Admin Dashboard:** Client-Side Rendering — nur interner Zugriff, SEO irrelevant

### Browser Support

- Modern Browsers (Chrome, Firefox, Safari, Edge — jeweils letzte 2 Major Versions)
- Developer-Audience = keine Legacy-Browser-Unterstützung nötig

### SEO

- Landing Page: Kritisch — Meta Tags, Open Graph, strukturierte Daten, Sitemap, schnelle Ladezeiten
- Dashboards: Nicht relevant — hinter Auth, kein Crawling

### Responsive Design

- Mobile-First Ansatz (Web3 Convention)
- Landing Page: Vollständig responsive, Mobile-optimiert
- Dashboards: Desktop-First für Daten-Tabellen, aber auf Tablet/Mobile nutzbar
- Breakpoints: Mobile (< 768px), Tablet (768-1024px), Desktop (> 1024px)

### Accessibility

- Standard WCAG 2.1 Level AA Compliance
- shadcn/ui liefert gute Accessibility-Defaults out of the box
- Keyboard Navigation, Screen Reader Support, ausreichende Kontraste

### Tech Stack

| Komponente | Technologie | Begründung |
|-----------|------------|-------------|
| Framework | Next.js 15 (App Router) | Bewährt im masemIT Stack |
| Styling | Tailwind CSS | Standard im Stack |
| UI Components | shadcn/ui | Konsistent mit anderen masemIT Produkten |
| Charts | Recharts oder tremor | Lightweight, für Dashboard-Statistiken |
| Auth | Magic Link Login (Auth.js v5 + Resend) + Parties API Tenant-Resolution + Neon Postgres (Tenant API Keys) | Managed MVP — kein Self-Service Signup, user-freundlicher Login per Email |
| API Client | Fetch + SWR oder TanStack Query | Dashboard Polling/Refresh |
| Deployment | Vercel Pro | Bestehendes Setup |
| Analytics | masemIT Analytics (analytics.masem.at) | Eigenes Produkt |
| Email | Resend | Magic Link Emails + Request Access Formular → Email an contact@masem.at |
| Frontend DB | Neon Postgres + Drizzle ORM | Tenant API Keys (verschlüsselt) + Auth.js Sessions |

### Implementation Considerations

- **Monorepo vs. Separate Repos:** Zu klären im Architecture Doc. Empfehlung: Ein Next.js Projekt mit Route Groups (`/(landing)`, `/(dashboard)`, `/(admin)`)
- **API Integration Pattern:** Alle Business-Daten kommen vom MMS Backend (api.masem.at). Frontend-DB (Neon Postgres) nur für Tenant API Keys und Auth Sessions
- **Environment-Trennung:** Landing Page ist public, Dashboards sind protected. Middleware für Auth-Check
- **Dark Mode:** Primär Dark Mode (Web3 Convention), Light Mode optional/später
- **Real-Time:** MVP nutzt Polling/SWR Refresh. WebSocket erst Phase 2
- **Request Access Formular:** Kein MMS-Call. Email via Resend an contact@masem.at. Kein Account wird erstellt.

## Functional Requirements

### Landing Page & Marketing

- **FR1:** Besucher kann die PayWatcher Value Proposition (Verification-Only, Non-Custodial, Flat-Fee) auf der Landing Page verstehen
- **FR2:** Besucher kann interaktive Code-Beispiele (curl, JavaScript) auf der Landing Page sehen und kopieren
- **FR3:** Besucher kann die Pricing-Tiers (Free/Starter/Pro/Enterprise) auf der Landing Page vergleichen
- **FR4:** Besucher kann einen Kostenvergleich zwischen PayWatcher und Wettbewerbern einsehen
- **FR5:** Besucher kann den 3-Step "How It Works" Flow visuell nachvollziehen
- **FR6:** Besucher kann Trust Signals (Non-Custodial, Base Network, USDC, Open Architecture, "Built by masemIT") einsehen
- **FR7:** Besucher kann von der Landing Page direkt zum Request Access Formular navigieren

### Request Access & Onboarding

- **FR36:** Besucher kann ein Request-Access-Formular ausfüllen (Name, Email, Company/Project Name, Use Case) um Zugang zu beantragen. Formular sendet Email via Resend an contact@masem.at. Kein MMS-Call, kein Account wird erstellt.
- **FR37:** Tenant kann sich per Magic Link (Email via Resend) im Dashboard einloggen. Tenant-Zuordnung via POST /v1/auth/login (Parties API). API Key aus Frontend-DB (Neon Postgres). Kein Signup, nur Login.

### Authentication

- **FR9:** Tenant kann sich per Magic Link in sein Dashboard einloggen (Email via Resend → POST /v1/auth/login → Tenant-Resolution)
- **FR10:** Tenant kann sich aus seinem Dashboard ausloggen

### API Key Management

- **FR12:** Tenant kann seinen API Key im Dashboard einsehen (masked)
- **FR14:** Tenant kann die Scopes seines API Keys einsehen

### Payment Monitoring

- **FR15:** Tenant kann eine Liste seiner Payments im Dashboard einsehen
- **FR16:** Tenant kann Payments nach Status filtern (pending, matched, confirming, confirmed, expired, failed)
- **FR17:** Tenant kann Payments nach Zeitraum filtern
- **FR18:** Tenant kann die Payment-Liste sortieren (nach Datum, Betrag, Status)
- **FR19:** Tenant kann durch paginierte Payment-Listen navigieren
- **FR38:** Tenant kann die Webhook Delivery History pro Payment einsehen (GET /v1/payments/:id/webhooks)

### Webhook Management

- **FR20:** Tenant kann seine Webhook-URL im Dashboard konfigurieren
- **FR21:** Tenant kann einen Test-Webhook senden um die Konfiguration zu prüfen
- **FR22:** Tenant kann den Webhook Delivery Log einsehen

### Tenant Settings

- **FR23:** Tenant kann die Default-Confirmation-Anzahl anpassen (2-20)
- **FR24:** Tenant kann seine Deposit Address einsehen (read-only)
- **FR25:** Tenant kann seine Account-Einstellungen (Email) verwalten

### Documentation

- **FR28:** Developer kann einen Quick Start Guide auf der Landing Page/Docs einsehen
- **FR29:** Developer kann eine Endpoint Reference (alle API Endpoints) einsehen
- **FR30:** Developer kann eine Webhook Event Reference (Payloads, Event Types) einsehen
- **FR31:** Developer kann Code-Beispiele in mehreren Sprachen (curl, JS, Python) einsehen

### Analytics & Tracking

- **FR32:** System trackt Landing Page Events (Page View, CTA Clicks, Pricing Views, Code Copies, Request Access Start/Submit)
- **FR33:** System sendet Analytics-Events an masemIT Analytics (analytics.masem.at)

### SEO & Discoverability

- **FR34:** Landing Page ist SEO-optimiert (Meta Tags, Open Graph, strukturierte Daten, Sitemap)
- **FR35:** Landing Page ist für Social Sharing optimiert (OG Images, Twitter Cards)

## Non-Functional Requirements

### Performance

- **NFR1:** Landing Page: First Contentful Paint unter 1.5 Sekunden (SSG + Vercel CDN)
- **NFR2:** Dashboard: API-getriebene Seiten laden Daten innerhalb von 2 Sekunden
- **NFR4:** Code-Beispiele auf der Landing Page sind sofort kopierbar ohne Ladezeit

### Security

- **NFR5:** Alle Verbindungen laufen über HTTPS (TLS 1.2+)
- **NFR6:** API Keys werden gehasht gespeichert und nur bei Erstellung einmalig im Klartext angezeigt
- **NFR7:** Auth-Tokens/API Keys in der Session haben eine begrenzte Lebensdauer und werden sicher übertragen
- **NFR8:** Dashboard-Routen sind nur für authentifizierte Tenants zugänglich (Middleware-Check)
- **NFR9:** CSRF-Schutz auf allen mutierenden Endpoints
- **NFR10:** GDPR-konform: Privacy Policy, Datenlöschung auf Anfrage möglich

### Integration

- **NFR11:** Frontend kommuniziert ausschließlich über REST API mit MMS Backend (api.masem.at)
- **NFR12:** Frontend nutzt minimale DB (Neon Postgres) nur für Tenant API Keys und Auth Sessions — keine Business-Daten
- **NFR13:** Analytics-Events werden asynchron an masemIT Analytics gesendet (kein Blocking der UI)
- **NFR14:** Block Explorer Links (Basescan) öffnen sich in neuem Tab

### Deployment & Operations

- **NFR15:** Deployment über Vercel Pro mit automatischem CI/CD (Git Push → Deploy)
- **NFR16:** Umgebungsvariablen (API URLs, Analytics Keys) sind konfigurierbar pro Environment (Dev/Staging/Prod)
- **NFR17:** Keine Secrets im Frontend-Bundle — alle sensitiven Operationen laufen serverseitig
