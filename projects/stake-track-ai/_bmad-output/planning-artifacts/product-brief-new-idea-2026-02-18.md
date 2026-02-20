---
stepsCompleted: [1, 2, 3, 4, 5, 6]
status: complete
inputDocuments:
  - _bmad-output/planning-artifacts/research/market-staking-saas-research-2026-02-18.md
  - docs/_masemIT/chatgpt.md
  - docs/_masemIT/gemini.md
  - docs/_masemIT/company-info.md
date: 2026-02-18
author: Sempre
---

# Product Brief: StakeTrack AI

## Executive Summary

**StakeTrack AI** ist eine Staking-Intelligence-Plattform von masemIT e.U., die als einzige am Markt **Validator Risk Intelligence, Real-Yield-Transparenz und Tax-Compliance** in einem Produkt vereint. Sie schließt die "Missing Middle" zwischen kostenlosen aber oberflächlichen Tools (DeFiLlama, StakingRewards) und unbezahlbaren Enterprise-Lösungen (Kiln, Figment ab $50K+/Jahr).

Aufgebaut auf masemITs bewährtem Web3-Stack (ChainSights-Scoring-Methodik, PayWatcher-API-Architektur, MMS-Microservices, masemIT Analytics), liefert StakeTrack AI das, was 68 Millionen Staking-Wallets und tausende institutionelle Staker brauchen: **Alles auf einen Blick, aktuell, transparent und actionable.**

Das Timing ist ideal: Die Übernahme von Rated Labs (einziger unabhängiger Validator-Rating-Service) durch Figment im Oktober 2025 hat eine klare Marktlücke geschaffen, während der regulatorische Tsunami 2026 (IRS 1099-DA, EU DAC8, CARF) Tool-Adoption in allen Segmenten erzwingt.

---

## Core Vision

### Problem Statement

Crypto-Staking ist eine $40-Milliarden-Industrie, in der Teilnehmer systematisch blind operieren:

- **Retail-Staker** werden durch nominale APY-Anzeigen getäuscht - ein 12% APY bei 10% Token-Inflation bedeutet nur 2% Real Yield, aber kein Tool zeigt das transparent
- **Institutionelle Staker** haben nach der Übernahme von Rated Labs durch Figment (Okt 2025) keinen unabhängigen Validator-Risk-Rating-Service mehr
- **Alle Staker** stehen ab 2026 vor einem regulatorischen Tsunami (IRS 1099-DA, EU DAC8, CARF) ohne integrierte Tax-Compliance-Tools
- **Prosumer und kleine Funds** ($100K-$5M Portfolio) sind in der "Missing Middle" gefangen - DeFiLlama ist zu basic, Kiln/Figment zu teuer

### Problem Impact

- Millionenverluste durch uninformierte Validator-Auswahl und Slashing-Events
- Systematische Überbewertung von Staking-Erträgen durch fehlende Real-Yield-Transparenz
- Steuer-Non-Compliance-Risiken durch fragmentierte Daten über multiple Chains und Plattformen
- 70% der ETH-haltenden Institutionen staken bereits, aber ohne adäquate unabhängige Risk-Tools
- 74% der Family Offices investieren in Crypto, aber verfügen nicht über Staking-spezifische Analytics

### Why Existing Solutions Fall Short

| Wettbewerber | Was sie bieten | Was fehlt |
|---|---|---|
| **DeFiLlama / StakingRewards** | Kostenlose Yield-/TVL-Daten | Kein Risk Score, kein Real Yield, kein Tax-Export |
| **Kiln / Figment / P2P.org** | Enterprise-Grade Staking Infrastructure | $50K+/Jahr, primär Execution statt unabhängige Analytics |
| **Koinly / CoinTracking** | Starke Tax-Engine | Staking nur ein Feature unter vielen, keine Risk Intelligence |
| **Rated Labs** | War einziger unabhängiger Validator-Rating-Service | Von Figment absorbiert (Okt 2025) → nicht mehr unabhängig |
| **Zapper / DeBank** | Multi-Chain Portfolio Tracking | Minimal Staking-spezifisch, kein Scoring |

**Kernerkenntnis:** Kein einziges Produkt vereint Risk Intelligence + Real Yield + Tax Compliance. Die Marktkonsolidierung (Rated→Figment, TRES→Fireblocks) hat die Lücke sogar VERGRÖSSERT.

### Proposed Solution

**StakeTrack AI** - das Staking-Command-Center für alle, die es ernst meinen:

- **Validator Risk Score (VRS):** Chain-übergreifender, unabhängiger Score (0-100) basierend auf Uptime, Slashing-Historie, Commission-Verhalten, Stake-Konzentration und Client-Diversität - inspiriert von ChainSights' bewährter GVS-Scoring-Methodik (4 gewichtete Faktoren, Confidence Levels)
- **Real-Yield-Kalkulator:** Nominal APY - Inflation - Validator Fees - Slashing-Risiko = Wahrer Ertrag - der "Schock-Moment" der viral gehen kann
- **Multi-Chain Dashboard:** Alle Staking-Positionen, alle Chains, ein Blick - aktuell und actionable
- **Tax-Compliance Engine:** Automatisierter Export für IRS 1099-DA, EU/MiCA-konform, CARF-ready ab 2026
- **API-first Architecture:** B2B-API für Funds, Plattformen und Custodians die Intelligence einbetten wollen
- **AI-Powered Insights:** KI-basierte Validator-Empfehlungen und Anomaly Detection für Slashing-Frühwarnung

### Key Differentiators

1. **"Das neue Rated Labs" - aber unabhängig:** Kein eigener Validator-Betrieb = keine Interessenkonflikte. Neutrale, unabhängige Intelligence - genau was der Markt nach der Figment-Übernahme braucht.

2. **Bewährte Scoring-DNA aus ChainSights:** Die GVS-Methodik (gewichtete Multi-Faktor-Scores, Confidence Levels, Identity-first Analytics) ist direkt adaptierbar für Validator-Bewertung. Proven methodology, neues Anwendungsfeld.

3. **masemIT Full-Stack-Synergien:**
   - **MMS** → User-Management, Consents, zentrale Service-Schicht
   - **masemIT Analytics** → Eigenes Product-Analytics-Backend, User-Behavior-Tracking
   - **PayWatcher-Patterns** → API-first Design, Webhook-Architektur, Tenant-System
   - **ChainSights-Engine** → Scoring-Methodik, On-Chain-Datenverarbeitung
   - → Reduziert Time-to-Market signifikant, kein Rad muss neu erfunden werden

4. **Einzige Kombination am Markt:** Risk Intelligence + Real Yield + Tax in einem Produkt existiert nirgends - weder bei Enterprise-Playern noch bei Retail-Tools.

5. **Perfektes Timing - Regulatorischer Rückenwind:** IRS 1099-DA (2025), EU DAC8, CARF (2026) erzwingen Tool-Adoption in ALLEN Segmenten gleichzeitig. Wer jetzt baut, surft die Welle.

6. **Web3 Track Record:** masemIT hat mit ChainSights (46 DAOs, Governance Scoring), PayWatcher (USDC Payment Verification) und MMS (Microservices) bewiesen, dass es Web3-Produkte erfolgreich launchen kann.

---

## Target Users

### Primäre Nutzer

**Persona 1: "Lisa, die Multi-Chain-Stakerin" (B2C Core - LAUNCH-PRIORITÄT)**

- **Kontext:** 34, Software-Entwicklerin in Berlin, staket seit 2023 aktiv. Portfolio: ~€85K über ETH, SOL, ATOM, DOT und ADA verteilt. Nutzt 3 verschiedene Wallets und 2 Exchanges.
- **Motivation:** Maximale Rendite bei minimalem Aufwand, transparente Steuererklärung
- **Aktueller Schmerz:** Hat 6 Tabs offen (StakingRewards, DeFiLlama, Koinly, 3 Block-Explorer), weiß nicht wie ihr Real Yield aussieht, und die letzte Steuererklärung hat sie 2 Wochenenden und €400 Steuerberater-Gebühren gekostet
- **Workaround:** Excel-Tabelle die sie monatlich manuell pflegt, Koinly für Tax - aber Staking-Rewards sind oft falsch importiert
- **Erfolgsmoment:** "Ich öffne EIN Dashboard und sehe sofort: Mein Real Yield über alle Chains ist 3.2%, nicht die 7% die mir überall angezeigt werden. Und der Tax-Export geht mit einem Klick."
- **Emotional:** Frustriert vom Chaos, erleichtert wenn Klarheit kommt

**Persona 2: "Thomas, der Prosumer-Fund-Manager" (Missing Middle)**

- **Kontext:** 47, ehemaliger Finanzberater, verwaltet ein kleines Crypto-Portfolio für sich und 3 Family-Office-Klienten. Gesamt ~€2M in Staking. Registriert in Österreich.
- **Motivation:** Professionelle Reports für seine Klienten, Risk-Management, Compliance
- **Aktueller Schmerz:** Kiln hat ihn abblitzen lassen ("zu klein"), DeFiLlama ist nicht präsentabel für Klienten, Rated Labs ist jetzt bei Figment und Enterprise-only. Erstellt Reports manuell in PowerPoint.
- **Workaround:** Kombination aus DeFiLlama + StakingRewards + manuellen Spreadsheets + Koinly
- **Erfolgsmoment:** "Ich generiere einen professionellen Validator-Risk-Report als PDF und schicke ihn meinem Klienten - in 2 Minuten statt 2 Tagen."
- **Emotional:** Braucht Glaubwürdigkeit gegenüber Klienten, will professionell auftreten

**Persona 3: "Sarah, die institutionelle Risk-Analystin" (B2B API)**

- **Kontext:** 31, Risk Analyst bei einem mittelgroßen Crypto Fund ($200M AUM) in London. Verantwortlich für Validator-Due-Diligence über 8 Chains.
- **Motivation:** Unabhängige, auditierbare Validator-Ratings für Investment-Committee-Entscheidungen
- **Aktueller Schmerz:** Seit Rated Labs zu Figment gehört, hat sie keine neutrale Datenquelle mehr. Ihr Fund staket NICHT bei Figment - also vertraut sie deren Ratings nicht (Interessenkonflikt). Baut sich eigene Spreadsheets aus 5 verschiedenen Block-Explorern.
- **Workaround:** Manuelle Datensammlung, eigene Excel-Modelle, 2 Tage pro Woche für Validator-Monitoring
- **Erfolgsmoment:** "Ich integriere die VRS-API in unser Portfolio-System und unser Investment Committee bekommt automatisch Risk-Alerts wenn ein Validator kritisch wird."
- **Emotional:** Unter Druck, braucht verlässliche Daten die einem Audit standhalten

### Sekundäre Nutzer

**Persona 4: "Dev, der Plattform-Integrator" (B2B API-Consumer)**

- **Kontext:** 28, Backend-Developer bei einer mittelgroßen Exchange die Staking-Services anbietet. Sucht eine API für Validator-Risk-Daten.
- **Motivation:** Risk-Scores und Yield-Daten in die eigene Plattform einbetten, ohne sie selbst berechnen zu müssen
- **Erfolgsmoment:** "API-Key generiert, Docs gelesen, erste Integration in 2 Stunden live."

**Persona 5: "Sempre/Admin" (System-Administrator)**

- **Kontext:** Produkt-Owner und Admin von StakeTrack AI. Anfangs Sempre selbst, später delegierbar.
- **Motivation:** System-Health überwachen, Tenant-Management, Data-Pipeline-Monitoring, Scoring-Algorithmus-Tuning
- **Erfolgsmoment:** "Ich sehe auf einen Blick: Alle Chain-Indexer laufen, 247 aktive Pro-User, 3 Enterprise-API-Kunden, Revenue diesen Monat €4.2K, keine kritischen Alerts."
- **Bezug zum masemIT-Stack:** Nutzt MMS für User-Management, masemIT Analytics für Product-Metrics, ähnliches Admin-Pattern wie PayWatcher-Tenant-Dashboard

### User Journey

**Phase 1 - Discovery:**

| Schritt | Lisa (B2C) | Thomas (Prosumer) | Sarah (B2B) |
|---|---|---|---|
| **Trigger** | Sieht Twitter-Post "Dein 12% APY ist in Wahrheit 2%" | Klient fragt nach Validator-Risk-Report | Rated Labs wird zu Figment → braucht Alternative |
| **Kanal** | Twitter/X, YouTube, Reddit | LinkedIn, Crypto-Konferenz, Peer-Empfehlung | Fachpublikation (The Block), direkte Ansprache |
| **Erster Kontakt** | Landing Page mit Real-Yield-Schock-Kalkulator | Case Study "Professionelle Reports in 2 Min" | API-Docs und Scoring-Methodik-Whitepaper |

**Phase 2 - Onboarding:**

| Schritt | Lisa (B2C) | Thomas (Prosumer) | Sarah (B2B) |
|---|---|---|---|
| **Erste Aktion** | Wallet connecten (Read-only) | Free-Tier testen, Demo-Report | API-Sandbox, Scoring-Methodik prüfen |
| **Time-to-Value** | 30 Sekunden (sofort Portfolio sehen) | 5 Minuten (erster Report) | 2 Stunden (API-Integration) |
| **Aha-Moment** | "Mein Real Yield ist nur 2.7%?!" | "Dieser PDF-Report sieht professioneller aus als alles was ich bisher hatte" | "Der VRS korreliert mit historischen Slashing-Events - das ist verlässlich" |

**Phase 3 - Core Usage:**

| Nutzer | Tägliche Nutzung | Wöchentliche Nutzung | Monatliche Nutzung |
|---|---|---|---|
| **Lisa** | Dashboard-Check (2 Min) | Validator-Performance-Review | Tax-Export-Vorbereitung |
| **Thomas** | Dashboard für alle Klienten | Risk-Reports generieren | Klienten-Quartalsreports |
| **Sarah** | API-Alerts monitoren | Validator-Watchlist reviewen | Investment Committee Reporting |

**Phase 4 - Upgrade-Trigger:**

| Nutzer | Free → Pro | Pro → Business | Business → Enterprise |
|---|---|---|---|
| **Lisa** | 3+ Wallets oder Tax-Export | - | - |
| **Thomas** | Sofort (braucht Reports) | Multi-Klienten-Management | - |
| **Sarah** | - | - | API-Volume + Custom Reports |

---

## Success Metrics

### User-Erfolgsmetriken

**"Wissen unsere Nutzer mehr als vorher?"**

| Metrik | Ziel | Messmethode |
|---|---|---|
| Time-to-First-Insight | < 60 Sekunden nach Wallet-Connect | masemIT Analytics Event-Tracking |
| Real-Yield-Awareness | 80% der User sehen ihren Real Yield in der ersten Session | Feature-Usage-Tracking |
| Weekly Active Users (WAU) | 30% der registrierten User kehren wöchentlich zurück | masemIT Analytics |
| Tax-Export-Completion | 50% der Pro-User nutzen Tax-Export mindestens 1x/Jahr | Feature-Usage-Tracking |
| Dashboard-Session-Dauer | Ø 3-5 Minuten (fokussiert, nicht verwirrt) | masemIT Analytics |

**Leitprinzip:** Wenn ein User StakeTrack AI öffnet und danach NICHT mehr seine Excel-Tabelle updaten muss, haben wir gewonnen.

### Business Objectives

**Phase 1 - Launch & Traktion (Monat 1-3):**

| Ziel | Target | Warum realistisch |
|---|---|---|
| Registrierte User | 500 | Viral Real-Yield-Hook + Crypto-Twitter Launch |
| Pro-Subscriber ($19/Mo) | 25 | ~5% Conversion von Free, typisch für Freemium-SaaS |
| MRR | ~€475 | 25 × €19 |
| Unterstützte Chains | 3-4 (ETH, SOL, ATOM) | MVP-Scope, erweiterbar |
| Churn Rate | < 10%/Monat | Benchmark für Early-Stage B2C SaaS |

**Phase 2 - Wachstum & Prosumer (Monat 4-6):**

| Ziel | Target | Warum realistisch |
|---|---|---|
| Registrierte User | 2.000 | Organisches Wachstum + Tax-Season Q1 Push |
| Pro-Subscriber | 100 | Tax-Season als Conversion-Booster |
| Business-Subscriber ($99/Mo) | 10 | Thomas-Personas, Prosumer-Segment |
| MRR | ~€2.900 | (100 × €19) + (10 × €99) |
| Unterstützte Chains | 6-8 | DOT, ADA, AVAX dazu |
| Tax-Export-Nutzung | 40% der Pro-User | Tax-Season-Trigger |

**Phase 3 - B2B & Skalierung (Monat 7-12):**

| Ziel | Target | Warum realistisch |
|---|---|---|
| Registrierte User | 5.000+ | Community-Effekte, Content-Marketing |
| Pro-Subscriber | 250 | Skalierung aus Phase 2 |
| Business-Subscriber | 30 | Wachsendes Prosumer-Segment |
| Enterprise/API-Kunden | 3-5 | Erste Sarah-Personas, Sales-Outreach |
| MRR | ~€8.500-€12.000 | (250×€19) + (30×€99) + (3×€500 avg) |
| **"Trägt sich selbst"-Schwelle** | **~€5.000 MRR** | Deckt Infrastruktur + Zeitinvestment als EPU |

**12-Monats-Traum-Szenario:** €10K+ MRR, 5.000 User, 3 Enterprise-API-Kunden, als "das neue Rated Labs" in Crypto-Twitter bekannt.

### Key Performance Indicators

**North Star Metric:** **Wöchentlich aktive User die ihren Real Yield checken (WAU-RY)**
→ Diese eine Metrik verbindet User-Value (Transparenz) mit Business-Value (Retention → Revenue)

**Leading Indicators (Frühindikatoren):**

| KPI | Target | Frequenz | Warum wichtig |
|---|---|---|---|
| Wallet-Connects/Woche | 50 → 200 (wachsend) | Wöchentlich | Top-of-Funnel-Gesundheit |
| Free-to-Pro Conversion | 5-8% | Monatlich | Premium-Value-Überzeugung |
| Time-to-First-Insight | < 60 Sek | Kontinuierlich | Onboarding-Qualität |
| Feature-Adoption: Real Yield | > 80% der Sessions | Wöchentlich | USP-Resonanz |

**Lagging Indicators (Ergebnisindikatoren):**

| KPI | Target | Frequenz | Warum wichtig |
|---|---|---|---|
| MRR | Siehe Phasen-Ziele | Monatlich | Business-Gesundheit |
| Net Revenue Retention | > 100% | Quartalsweise | Expansion > Churn |
| Pro-User Churn | < 8%/Monat | Monatlich | Product-Market-Fit-Signal |
| NPS Score | > 40 | Quartalsweise | User-Zufriedenheit |

**Vanity-Metriken die wir NICHT verfolgen:**
- Total Registrierungen (ohne Aktivität = wertlos)
- Social-Media-Follower (wenn sie nicht konvertieren)
- Seitenaufrufe (ohne Wallet-Connect = Tourist)

**Strategische Alignment-Prüfung:**

| Product Vision | → User Metrik | → Business Metrik |
|---|---|---|
| "Alles auf einen Blick" | WAU + Session-Dauer | Retention + NRR |
| Real-Yield-Transparenz | Real-Yield-Feature-Usage | Free→Pro Conversion |
| Tax-Compliance | Tax-Export-Completion | Q1 Revenue Spike |
| Validator Risk Score | VRS-Nutzung + Alerts | Business/Enterprise Upgrade |
| API-first (B2B) | API-Calls/Tag | Enterprise MRR |

---

## MVP Scope

### Core Features (Must-Have für Launch)

**F1: Wallet-Connect & Multi-Chain Portfolio View**
- Read-only Wallet-Connect (keine Custody!)
- Unterstützung: ETH, SOL, ATOM (3 Chains)
- Übersicht aller Staking-Positionen auf einen Blick
- Aktuelle Balances, gestakte Beträge, Validator-Zuordnung
- _Persona:_ Lisa connectet ihre Wallets und sieht alles in < 60 Sek
- _Tech:_ RPC/API-Anbindung pro Chain, NeonDB für User-Daten

**F2: Real-Yield-Kalkulator (DER virale USP)**
- Berechnung: Nominal APY - Token-Inflation - Validator Fees = Real Yield
- Anzeige pro Chain, pro Validator, und Portfolio-Gesamt
- Visueller Vergleich: "Was dir versprochen wird" vs. "Was du wirklich bekommst"
- _Persona:_ Lisa's Schock-Moment - "Mein Real Yield ist nur 2.7%?!"
- _Tech:_ Inflationsdaten von Chain-APIs, Fee-Daten von Validators

**F3: Basis Validator Risk Score (VRS)**
- Score 0-100 basierend auf: Uptime, Slashing-Historie, Commission-Stabilität
- Einfache Ampel-Anzeige: Grün/Gelb/Rot pro Validator
- Confidence Level pro Score (High/Medium/Low - wie bei ChainSights GVS)
- _Persona:_ Lisa sieht sofort ob ihre Validators "safe" sind
- _Tech:_ Adaptierte ChainSights-Scoring-Methodik, Chain-spezifische Datenquellen

**F4: Basis Alerting**
- Email-Alerts bei: Validator Downtime, Commission-Erhöhung, Slashing-Event
- Konfigurierbar pro Wallet/Validator
- _Persona:_ Lisa bekommt Push wenn ihr Validator Probleme hat
- _Tech:_ Resend (bereits im masemIT-Stack), QStash für Queue

**F5: Landing Page mit Public Real-Yield-Rechner**
- Ohne Login nutzbar - viraler Einstieg
- "Gib deinen Staking-Betrag und Chain ein → Sieh deinen Real Yield"
- CTA: "Connecte deine Wallet für das volle Bild"
- _Persona:_ Erster Touchpoint über Twitter/Reddit-Link
- _Tech:_ Static Page, minimaler API-Call

### Out of Scope für MVP

| Feature | Warum nicht jetzt | Wann stattdessen |
|---|---|---|
| Tax-Export (CSV/PDF) | Wichtig, aber nicht für den Aha-Moment | Phase 2 (vor Q1 Tax-Season) |
| Business/Enterprise Tier | Braucht erst B2C-Traktion als Beweis | Phase 2-3 |
| API für B2B | Revenue-Treiber, aber Sales-Cycle zu lang | Phase 3 |
| PDF-Reports | Thomas-Persona, aber nicht Launch-kritisch | Phase 2 |
| Mehr als 3 Chains | Jede Chain = signifikanter Integrationsaufwand | Schrittweise ab Phase 2 |
| KI-Validator-Empfehlungen | Erst sinnvoll mit genug historischen Daten | Phase 3 |
| Slashing Early Warning | Komplexes ML-Problem | Phase 3 |
| Mobile App | Web-first, responsive reicht für MVP | Evaluierung nach 12 Monaten |
| Governance/DAO-Integration | ChainSights-Synergie, Scope-Creep-Gefahr | Langfrist-Vision |
| Eigene Indexer/Nodes | Drittanbieter-APIs reichen für MVP | Bei Skalierung evaluieren |

### MVP Success Criteria

**Go/No-Go nach 3 Monaten:**

| Kriterium | Target | Signal |
|---|---|---|
| User-Adoption | 500 registrierte User, 150 WAU | Produkt löst echtes Problem |
| Conversion | 25 Pro-Subscriber (5%) | Zahlungsbereitschaft validiert |
| Retention | 30% Weekly Return Rate | Stickiness vorhanden |
| Real-Yield-Usage | 80% nutzen den Kalkulator | USP resoniert |
| Qualitative Signale | "Endlich sehe ich alles!" Feedback | Emotionale Resonanz |

**Entscheidungsmatrix:**
- Alle 5 Kriterien erfüllt → Vollgas Phase 2 (Tax, mehr Chains, Business-Tier)
- 3-4 Kriterien erfüllt → Iterieren, Problem-Interviews, Pivot-Prüfung
- < 3 Kriterien → Ernsthafter Pivot oder Scope-Reboot

### Future Vision (Post-MVP Roadmap)

**Phase 2 (Monat 4-6): Monetarisierung & Expansion**
- Tax-Export Engine (CSV/PDF) - Launch VOR Q1 Tax Season
- 3-5 weitere Chains (DOT, ADA, AVAX, ...)
- Business-Tier ($99/Mo) mit PDF-Reports
- Erweiterter VRS mit mehr Faktoren
- Validator-Vergleichs-Tool

**Phase 3 (Monat 7-12): B2B & Intelligence**
- Public API für Validator Risk Score (das "neue Rated Labs")
- Enterprise-Tier ($299-$999/Mo)
- AI-powered Validator-Empfehlungen
- Slashing Early Warning System
- Whitelabel-Optionen für Exchanges/Custodians

**Langfrist-Vision (Jahr 2-3):**
- DAS unabhängige Staking-Intelligence-Ökosystem
- Mögliche ChainSights-Integration (Governance + Staking in einem)
- Eigener Staking-Benchmark-Index (wie S&P für Staking)
- Potenzial für Community/DAO-Governance über Scoring-Methodik

---

## Technischer Kontext (masemIT Stack)

### Vorhandene Infrastruktur

| Komponente | Service | Relevanz für StakeTrack AI |
|---|---|---|
| Hosting | Vercel Pro | Frontend + API Routes |
| Database | NeonDB Launch | User-Daten, Staking-Snapshots, VRS-History |
| Cache | Upstash (free) | Chain-Daten-Caching, Rate-Limiting |
| Queue | QStash (free) | Alert-Processing, Data-Pipeline-Jobs |
| Email | Resend Pro | Validator-Alerts, Onboarding-Mails |
| Bot Protection | Cloudflare Turnstile | Signup-Schutz |
| Analytics | masemIT Analytics | Product-Metrics, User-Behavior |
| User-Management | MMS | Auth, Consents, Parties |
| Scoring-Engine | ChainSights GVS (Pattern) | Adaptierbar für VRS |
| API-Patterns | PayWatcher | Tenant-System, Webhooks, API-Key-Management |

### Neue Infrastruktur (MVP-spezifisch)

| Bedarf | Optionen | Geschätzte Kosten |
|---|---|---|
| Chain RPC/APIs | Alchemy, QuickNode, Chain-native RPCs | Free-Tier ausreichend für MVP |
| Preis-/Inflationsdaten | CoinGecko API, Coin Metrics | Free-Tier für MVP |
| Validator-Daten | Chain-spezifische APIs, Solana Compass | Kostenlos/Open-Source |

---

*Product Brief abgeschlossen am 2026-02-18*
*Autor: Sempre (Mario Semper), masemIT e.U.*
*Facilitiert von: Mary, Business Analyst Agent*
