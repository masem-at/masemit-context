---
stepsCompleted:
  - step-01-init
  - step-02-discovery
  - step-02b-vision
  - step-02c-executive-summary
  - step-01b-continue
  - step-03-success
  - step-04-journeys
  - step-05-domain
  - step-06-innovation
  - step-07-project-type
  - step-08-scoping
  - step-09-functional
  - step-10-nonfunctional
  - step-11-polish
  - step-12-complete
classification:
  projectType: saas_b2b + blockchain_web3
  domain: fintech/crypto
  complexity: high
  projectContext: brownfield
inputDocuments:
  - _bmad-output/planning-artifacts/product-brief-new-idea-2026-02-18.md
  - _bmad-output/planning-artifacts/research/market-staking-saas-research-2026-02-18.md
  - _bmad-output/planning-artifacts/research/technical-staketrack-ai-mvp-research-2026-02-18.md
  - _bmad-output/project-context.md
  - docs/_masemIT/company-info.md
  - docs/_masemIT/gemini.md
  - docs/_masemIT/chatgpt.md
documentCounts:
  briefs: 1
  research: 2
  projectContext: 1
  projectDocs: 3
workflowType: 'prd'
date: '2026-02-18'
author: 'Sempre'
---

# Product Requirements Document - StakeTrack AI

**Author:** Sempre
**Date:** 2026-02-18

## Executive Summary

**StakeTrack AI** ist eine unabhängige Staking-Intelligence-Plattform von masemIT e.U., die die "Missing Middle" zwischen kostenlosen aber oberflächlichen Tools (DeFiLlama, StakingRewards) und unbezahlbaren Enterprise-Lösungen (Kiln, Figment ab $50K+/Jahr) besetzt. Als einzige Plattform am Markt vereint sie **Validator Risk Intelligence, Real-Yield-Transparenz und Multi-Chain-Aggregation** in einem Produkt.

Das Timing ist strategisch ideal: Die Übernahme von Rated Labs durch Figment (Oktober 2025) hat die einzige unabhängige Validator-Rating-Quelle eliminiert, während der regulatorische Tsunami 2026 (IRS 1099-DA, EU DAC8, CARF) Tool-Adoption in allen Segmenten erzwingt. 68 Millionen Staking-Wallets werden systematisch über ihre echten Erträge im Dunkeln gelassen — ein 12% APY bei 10% Token-Inflation bedeutet nur 2% Real Yield, aber kein Tool zeigt das transparent.

StakeTrack AI startet als B2C-Dashboard mit viralem Real-Yield-Hook und wächst über Prosumer- und Business-Tiers zum B2B-API-Layer. Das Produkt baut auf masemITs bewährtem Web3-Stack auf (ChainSights-Scoring-Methodik, PayWatcher-API-Architektur, MMS-Microservices) und ist der nächste Baustein in masemITs Web3-Produktfamilie.

**MVP-Scope (F1-F5):** Multi-Chain Dashboard (ETH, SOL, ATOM), Real-Yield-Kalkulator, Validator Risk Score (0-100), Portfolio-Aggregation via Wallet-Connect, Alert-System. Infrastrukturkosten: ~$0/Monat zusätzlich durch Nutzung des bestehenden Stacks.

### What Makes This Special

1. **"Das neue Rated Labs" — aber unabhängig:** Kein eigener Validator-Betrieb = keine Interessenkonflikte. Die Marktlücke nach der Figment-Übernahme ist offen und wächst.

2. **Real-Yield-Schock als viraler Hook:** Kein Wettbewerber zeigt systematisch: Nominal APY − Inflation − Validator Fees − Slashing-Risiko = Wahrer Ertrag. Der "Schock-Moment" hat virales Potenzial und ist der Türöffner für Nutzer-Akquisition.

3. **Bewährte Scoring-DNA aus ChainSights:** Die GVS-Methodik (gewichtete Multi-Faktor-Scores, Confidence Levels) ist direkt adaptierbar für den Validator Risk Score — proven methodology, neues Anwendungsfeld.

4. **Einzige Kombination am Markt:** Risk Intelligence + Real Yield + Multi-Chain-Aggregation existiert als Einzelprodukt nirgends — weder bei Enterprise-Playern noch bei Retail-Tools.

5. **Regulatorischer Rückenwind:** IRS 1099-DA, EU DAC8, CARF 2026 erzwingen Tool-Adoption in ALLEN Segmenten gleichzeitig.

## Project Classification

| Attribut | Wert |
|---|---|
| **Projekttyp** | SaaS B2B/B2C Hybrid + Blockchain/Web3 (Web-App mit Dashboard + späterer API-Layer) |
| **Domäne** | Fintech/Crypto — Staking Intelligence & Validator Risk Analytics |
| **Komplexität** | Hoch — Multi-Chain-Datenintegration, Finanzberechnungen, Scoring-Algorithmus, regulatorische Relevanz, Wallet-Auth (SIWE) |
| **Projektkontext** | Brownfield — erweitert den masemIT-Stack (Vercel Pro, NeonDB, Upstash, MMS, Resend) um ein neues Produkt |
| **Architektur** | Serverless Modular Monolith (Next.js 15 App Router + Drizzle ORM + NeonDB/TimescaleDB) |

## Success Criteria

### User Success

**Kernfrage:** *Wissen unsere Nutzer mehr als vorher?*

| Metrik | Ziel | Messmethode |
|---|---|---|
| Time-to-First-Insight | < 60 Sekunden nach Wallet-Connect | masemIT Analytics Event-Tracking |
| Real-Yield-Awareness | 80% der User sehen Real Yield in erster Session | Feature-Usage-Tracking |
| Weekly Active Users (WAU) | 30% der registrierten User kehren wöchentlich zurück | masemIT Analytics |
| Dashboard-Session-Dauer | 3-5 Minuten (fokussiert, nicht verwirrt) | masemIT Analytics |

**Emotionale Erfolgsmomente:**
- **"Schock-Moment":** User sieht zum ersten Mal seinen Real Yield vs. nominalen APY — "Mein 12% APY ist in Wahrheit 2.7%?!"
- **"Endlich Klarheit":** User schließt 6 Tabs (StakingRewards, DeFiLlama, Block-Explorer) und braucht nur noch StakeTrack AI
- **"Mein Validator ist safe":** Grüne VRS-Ampel gibt Sicherheit, rote löst informiertes Handeln aus

**Leitprinzip:** Wenn ein User StakeTrack AI öffnet und danach NICHT mehr seine Excel-Tabelle updaten muss, haben wir gewonnen.

### Business Success

**North Star Metric:** Wöchentlich aktive User die ihren Real Yield checken (WAU-RY) — verbindet User-Value (Transparenz) mit Business-Value (Retention → Revenue)

**Kanal-Einschränkungen:** Twitter/X nicht verfügbar (Account-Sperren), Reddit erfordert spezielle Content-Strategie (Lösch-Problematik). Primäre Kanäle: Farcaster (Crypto-native), Bluesky, Discord Communities, YouTube, LinkedIn.

**Phase 1 — Launch & Traktion (Monat 1-3):**

| Ziel | Target | Begründung |
|---|---|---|
| Registrierte User | 500 | Viral Real-Yield-Hook + Farcaster/Bluesky/Discord Launch |
| Pro-Subscriber ($19/Mo) | 25 | ~5% Conversion, typisch für Freemium-SaaS |
| MRR | ~€475 | 25 × €19 |
| Churn Rate | < 10%/Monat | Benchmark Early-Stage B2C SaaS |

**Phase 2 — Wachstum (Monat 4-6):**

| Ziel | Target | Begründung |
|---|---|---|
| Registrierte User | 2.000 | Organisch + Tax-Season Q1 Push |
| Pro-Subscriber | 100 | Tax-Season als Conversion-Booster |
| Business-Subscriber ($99/Mo) | 10 | Prosumer-Segment (Thomas-Persona) |
| MRR | ~€2.900 | (100 × €19) + (10 × €99) |

**Phase 3 — B2B & Skalierung (Monat 7-12):**

| Ziel | Target | Begründung |
|---|---|---|
| Registrierte User | 5.000+ | Community-Effekte, Content-Marketing |
| Enterprise/API-Kunden | 3-5 | Erste Sarah-Personas, Sales-Outreach |
| MRR | ~€8.500-€12.000 | (250×€19) + (30×€99) + (3×€500 avg) |
| **"Trägt sich selbst"** | **~€5.000 MRR** | Deckt Infrastruktur + Zeitinvestment als EPU |

**Leading Indicators:** Wallet-Connects/Woche (50→200), Free-to-Pro Conversion (5-8%), Feature-Adoption Real Yield (>80%)

**Lagging Indicators:** MRR (phasenbasiert), Net Revenue Retention (>100%), Pro-User Churn (<8%/Mo), NPS (>40)

### Technical Success

| Metrik | Ziel | Messung |
|---|---|---|
| Dashboard TTFB | < 500ms | Vercel Analytics |
| API Response (cached) | < 100ms | Custom Logging |
| API Response (uncached) | < 2s | Custom Logging |
| VRS Calculation Accuracy | > 95% Korrelation mit On-Chain-Daten | Manual Spot-Checks |
| Data Freshness | < 1 Stunde | QStash Monitoring |
| Uptime | > 99.5% | Vercel Status |
| Test Coverage (Domain Layer) | > 80% | Vitest Coverage |
| Cold Start | < 1s | Vercel Function Logs |
| Bundle Size | < 200KB First Load JS | Next.js Build Output |

### Measurable Outcomes

**Go/No-Go nach 3 Monaten:**

| Kriterium | Target | Signal |
|---|---|---|
| User-Adoption | 500 registrierte User, 150 WAU | Produkt löst echtes Problem |
| Conversion | 25 Pro-Subscriber (5%) | Zahlungsbereitschaft validiert |
| Retention | 30% Weekly Return Rate | Stickiness vorhanden |
| Real-Yield-Usage | 80% nutzen den Kalkulator | USP resoniert |
| Qualitative Signale | "Endlich sehe ich alles!" Feedback | Emotionale Resonanz |

**Entscheidungsmatrix:**
- Alle 5 Kriterien erfüllt → Vollgas Phase 2
- 3-4 Kriterien → Iterieren, Problem-Interviews, Pivot-Prüfung
- < 3 Kriterien → Ernsthafter Pivot oder Scope-Reboot

## Product Scope

### MVP — Minimum Viable Product

**Core Features (F1-F5):**

| Feature | Beschreibung | Persona-Bezug |
|---|---|---|
| **F1: Multi-Chain Dashboard** | Wallet-Connect (read-only), ETH/SOL/ATOM, alle Staking-Positionen auf einen Blick | Lisa connectet Wallets, sieht alles in < 60 Sek |
| **F2: Real-Yield-Kalkulator** | Nominal APY − Inflation − Validator Fees = Real Yield, visueller Vergleich | Lisa's Schock-Moment: "Nur 2.7%?!" |
| **F3: Validator Risk Score** | Score 0-100, Ampel (Grün/Gelb/Rot), 5 Faktoren, Confidence Level | Lisa sieht sofort ob Validators safe sind |
| **F4: Basis Alerting** | Email-Alerts bei Downtime, Commission-Erhöhung, Slashing | Lisa bekommt Push bei Validator-Problemen |
| **F5: Public Real-Yield-Rechner** | Ohne Login nutzbar, viraler Einstieg, CTA zu Wallet-Connect | Erster Touchpoint über Farcaster/Bluesky/Discord |

**MVP-Minimum Compliance:**
- DSGVO-konform: Privacy Policy, Cookie Consent, Datenminimierung, Recht auf Löschung
- Keine Speicherung von Wallet Private Keys (read-only Analytics)
- Wallet-Adressen als pseudo-anonyme Daten behandeln

### Growth & Vision (Post-MVP)

Detaillierte Post-MVP-Roadmap mit Personas und Triggern: siehe **Project Scoping & Phased Development > Post-MVP Features**.

**Langfristige Vision:**
- DAS unabhängige Staking-Intelligence-Ökosystem
- Eigener Staking-Benchmark-Index (wie S&P für Staking)
- ChainSights-Integration (Governance + Staking)
- Whitelabel-Optionen für Exchanges/Custodians

## User Journeys

### Journey 1: Lisa — "Vom Tab-Chaos zur Klarheit" (B2C Core, MVP)

**Persona:** Lisa, 34, Software-Entwicklerin in Berlin. ~€85K Staking-Portfolio über ETH, SOL, ATOM. 3 Wallets, 2 Exchanges. Frustriert vom Chaos.

**Opening Scene:** Dienstagabend, 22:15. Lisa hat 6 Browser-Tabs offen — StakingRewards, DeFiLlama, Koinly, drei Block-Explorer. Sie versucht herauszufinden, wie ihr Staking-Portfolio wirklich performt. Ihre Excel-Tabelle ist seit 3 Wochen nicht aktualisiert. Sie seufzt.

**Rising Action:** Lisa sieht einen Farcaster-Post: *"Dein 12% APY ist in Wahrheit 2%. Finde deinen echten Ertrag."* Sie klickt, landet auf der StakeTrack AI Landing Page. Der Public Real-Yield-Rechner (F5) zeigt ihr sofort: ATOM mit 18% nominal APY hat nach Inflation nur ~4% Real Yield. Sie ist schockiert. "Wie viel verliere ich wirklich?" — sie klickt "Connect Wallet".

**Climax:** 30 Sekunden nach Wallet-Connect sieht Lisa ihr komplettes Portfolio: Alle 3 Chains, alle Validators, alle Positionen — auf einen Blick. Der Real-Yield-Kalkulator zeigt: Ihr Portfolio-Gesamt-Real-Yield ist 2.7%, nicht die 7.8% die sie angenommen hat. Ihr ETH-Validator hat einen VRS von 43 (Gelb) — Commission wurde letzte Woche erhöht. "Das wusste ich gar nicht!"

**Resolution:** Lisa schließt 5 von 6 Tabs. StakeTrack AI ist ihr neuer einziger Tab. Sie richtet Alerts ein für ihre Validators und checkt das Dashboard jetzt 2x pro Woche, 3 Minuten pro Session. Nach 2 Wochen upgradet sie auf Pro ($19/Mo) weil sie eine dritte Wallet hinzufügen will. Die Excel-Tabelle hat sie gelöscht.

**Edge Cases:**
- *Wallet ohne Staking-Positionen:* Lisa connectet versehentlich eine Trading-Wallet. Dashboard zeigt: "Keine Staking-Positionen gefunden. Staket diese Wallet auf einer anderen Chain?" mit Hinweis auf unterstützte Chains.
- *Validator wird plötzlich rot:* Lisa bekommt Email-Alert: "Dein ETH-Validator 'Lido-42' hat VRS 38 (Rot) — Uptime unter 90% in den letzten 7 Tagen." Dashboard zeigt Details und alternative Validators mit hohem VRS.
- *Chain-API temporär offline:* SOL-Daten können nicht geladen werden. Dashboard zeigt letzte bekannte Daten mit Timestamp: "Solana-Daten von vor 2 Stunden. Aktualisierung läuft." Kein Crash, keine leere Seite.

---

### Journey 2: Thomas — "Vom PowerPoint-Sklaven zum professionellen Berater" (Prosumer, Phase 2)

**Persona:** Thomas, 47, ehemaliger Finanzberater in Wien. Verwaltet ~€2M Staking für sich und 3 Family-Office-Klienten. Von Kiln als "zu klein" abgelehnt.

**Opening Scene:** Thomas sitzt vor einem leeren PowerPoint-Slide. Sein Klient Dr. Huber will bis Freitag einen "Validator Risk Report" für das nächste Family-Office-Meeting. Thomas hat die letzten 2 Tage damit verbracht, Validator-Daten aus DeFiLlama und StakingRewards in eine Tabelle zu kopieren. Das Ergebnis sieht unprofessionell aus.

**Rising Action:** Ein Kollege bei einer Crypto-Konferenz erwähnt StakeTrack AI. Thomas testet den Free-Tier, sieht sofort das Multi-Chain-Dashboard. Dann entdeckt er den Business-Tier: PDF-Reports mit VRS-Scores, Real-Yield-Breakdown und Validator-Vergleich — automatisch generiert.

**Climax:** Thomas generiert seinen ersten Klienten-Report. 2 Minuten statt 2 Tage. Der PDF-Report enthält VRS-Scores, Real-Yield pro Chain, und Validator-Performance-Trends. Er sieht professioneller aus als alles, was er je in PowerPoint gebaut hat. Dr. Huber ist beeindruckt.

**Resolution:** Thomas upgradet sofort auf Business ($99/Mo). Er verwaltet alle 3 Klienten-Portfolios in StakeTrack AI, generiert monatliche Reports automatisch. Seine Klienten sind zufrieden, er gewinnt einen vierten. Die PowerPoint-Vorlage verstaubt.

**Edge Cases:**
- *Klient hat Positionen auf nicht unterstützter Chain:* Thomas' Klient Dr. Huber hat auch DOT-Positionen. Dashboard zeigt klar: "3 von 4 Chains abgedeckt. Polkadot-Support kommt in Phase 2." Kein Versprechen, transparente Kommunikation.
- *Report-Daten veraltet:* Thomas will Report generieren, aber ATOM-Daten sind 6 Stunden alt (API-Problem). Report zeigt Timestamp pro Datenpunkt. Thomas entscheidet ob er wartet oder mit Disclaimer sendet.

---

### Journey 3: Sarah — "Von der Spreadsheet-Hölle zur API-Integration" (B2B, Phase 3)

**Persona:** Sarah, 31, Risk Analyst bei einem Crypto Fund ($200M AUM) in London. Verantwortlich für Validator-Due-Diligence über 8 Chains. Seit Rated Labs → Figment hat sie keine neutrale Datenquelle mehr.

**Opening Scene:** Sarah verbringt 2 Tage pro Woche damit, Validator-Performance-Daten aus 5 verschiedenen Block-Explorern in ihre eigenen Excel-Modelle zu kopieren. Ihr Investment Committee verlangt unabhängige, auditierbare Validator-Ratings — aber ihr Fund staket NICHT bei Figment, also traut sie deren Ratings nicht (Interessenkonflikt). Nächste Woche ist Committee-Meeting.

**Rising Action:** Sarah findet StakeTrack AI über einen Blockworks-Artikel. Sie liest die Scoring-Methodik (öffentlich dokumentiert, transparent). Sie generiert einen API-Key im Sandbox-Modus und testet die VRS-API gegen ihre eigenen historischen Daten. Der VRS korreliert stark mit historischen Slashing-Events.

**Climax:** Sarah integriert die StakeTrack AI API in das Portfolio-Management-System ihres Funds. Das Investment Committee bekommt jetzt automatisch Risk-Alerts wenn ein Validator kritisch wird. Ihr erster automatischer Alert: Ein Cosmos-Validator hat innerhalb von 48 Stunden 3x die Commission erhöht — VRS fällt von 72 auf 51. Das Committee entscheidet zum Redelegieren, BEVOR ein Slashing-Event eintritt.

**Resolution:** Sarah spart 1.5 Tage pro Woche. Ihr Fund hat eine auditierbare, unabhängige Datenquelle. Bei der nächsten Compliance-Prüfung kann sie nachweisen, dass Validator-Entscheidungen auf systematischen, unabhängigen Ratings basieren. Der Fund upgradet auf Enterprise ($499/Mo).

**Edge Cases:**
- *API-Rate-Limit erreicht:* Sarahs Batch-Job feuert zu viele Requests. API gibt 429 mit `Retry-After` Header. Sarahs System wartet automatisch. Klare Dokumentation der Rate-Limits in den API-Docs.
- *VRS-Score ändert sich stark zwischen zwei Abfragen:* Sarah sieht einen Validator der von 85 auf 42 springt. API liefert `factors_changed`-Array das erklärt WARUM: "slashing_event_detected". Transparenz über Score-Änderungen.

---

### Journey 4: Dev — "API-Key, Docs, Live in 2 Stunden" (API-Consumer, Phase 3)

**Persona:** Dev, 28, Backend-Developer bei einer mittelgroßen Exchange die Staking-Services anbietet. Sucht eine API für Validator-Risk-Daten.

**Opening Scene:** Devs Exchange will ihren Nutzern Validator-Risk-Scores anzeigen, bevor sie delegieren. Selber bauen? Zu aufwändig. Devs Tech Lead sagt: "Finde eine API, Budget bis $300/Mo."

**Rising Action:** Dev findet StakeTrack AI API-Docs. Sandbox-API-Key in 2 Minuten generiert. Die REST-API ist simpel: `GET /v1/validators?chain=eth` liefert normalisierte Validator-Daten mit VRS-Score. TypeScript-SDK verfügbar. Dev baut einen Prototypen in 2 Stunden.

**Climax:** Der Prototyp zeigt VRS-Scores neben jedem Validator in der Exchange-UI. Users sehen auf einen Blick: Grün = sicher delegieren, Rot = Vorsicht. Devs Tech Lead ist überzeugt.

**Resolution:** Die Exchange integriert StakeTrack AI API in Production. Business-Tier ($99/Mo) reicht für ihr Request-Volumen. Dev ist zufrieden: Saubere Docs, schnelle Integration, keine Überraschungen.

**Edge Cases:**
- *API-Breaking-Change:* StakeTrack AI releast v2. v1 bleibt 6 Monate aktiv mit Deprecation-Warning im Response Header. Dev hat Zeit zu migrieren.
- *Webhook-Delivery fehlschlägt:* Devs Endpoint ist temporär down. QStash retried automatisch mit Exponential Backoff. Events gehen nicht verloren.

---

### Journey 5: Sempre — "Alles im Blick, alles unter Kontrolle" (Admin, MVP)

**Persona:** Sempre (Mario), 49, Produkt-Owner und Admin. Solo-Betrieb. Morgens 30 Minuten Admin-Check, dann Entwicklung.

**Opening Scene:** Montagmorgen, 7:45. Sempre öffnet StakeTrack AI, aber nicht als User — als Admin. Heute ist Tag 47 nach Launch. Fragen: Laufen alle Chain-Indexer? Gibt es neue Pro-Subscriber? Sind kritische Alerts rausgegangen?

**Rising Action:** Vercel Dashboard zeigt: Alle Functions healthy. QStash Dashboard: Ethereum-Indexer hat um 3:12 Uhr einen Timeout gehabt — automatisch retried, Daten sind vollständig. Solana und Cosmos aktuell. masemIT Analytics: 312 registrierte User, 18 Pro-Subscriber, 94 WAU.

**Climax:** masemIT Analytics zeigt: Real-Yield-Feature-Usage bei 83% — über dem 80%-Ziel. Free-to-Pro Conversion bei 5.7%. Ein neuer User hat über das Feedback-Formular geschrieben: "Endlich sehe ich was mein Staking WIRKLICH bringt. Danke!" Sempre weiß: Das Produkt funktioniert.

**Resolution:** Admin-Check in 15 Minuten erledigt via existierende Tools (Vercel + NeonDB Console + QStash + masemIT Analytics). Keine kritischen Issues. Sempre wechselt in den Dev-Modus. Die Infrastruktur läuft stabil, die Metriken stimmen.

**Edge Cases:**
- *Chain-API komplett down:* Cosmos LCD-Endpoint ist seit 4 Stunden nicht erreichbar. QStash Dashboard zeigt: 8 Retries verbraucht, Pipeline stalled. Sempre muss manuell Fallback-Endpoint in Environment Variables konfigurieren.
- *Plötzlicher User-Spike (Viral-Moment):* Farcaster-Post geht viral, 500 Wallet-Connects in 1 Stunde. Vercel skaliert automatisch, aber NeonDB Connection Pool nähert sich dem Limit. Sempre kann über Upstash Console Redis Cache-TTLs temporär erhöhen um DB-Last zu reduzieren.
- *Verdächtige API-Nutzung:* Ein einzelner API-Consumer feuert 10.000 Requests/Stunde. Rate-Limiting greift automatisch. Sempre sieht den Spike in Vercel Logs und kann reagieren.

---

### Journey Requirements Summary

| Journey | Revealed Capabilities |
|---|---|
| **Lisa (B2C)** | Wallet-Connect (Multi-Chain), Dashboard, Real-Yield-Kalkulator, VRS-Anzeige, Alert-Setup, Freemium-Gating, Empty-State-Handling |
| **Thomas (Prosumer)** | PDF-Report-Generierung, Multi-Klienten-Management, Business-Tier-Features, Chain-Coverage-Transparenz |
| **Sarah (B2B API)** | REST API, API-Key-Management, Sandbox-Modus, Rate-Limiting, Webhook-Delivery, Score-Change-Transparenz, API-Versioning |
| **Dev (Integrator)** | API-Docs, SDK/TypeScript-Client, Sandbox, Deprecation-Policy, Webhook-Retries |
| **Sempre (Admin)** | Admin-Dashboard, Pipeline-Health-Monitoring, User-Metriken, Analytics-Integration, Rate-Limit-Management, Fallback-Konfiguration |

**MVP-kritische Capabilities (aus Lisa + Sempre):**
- Wallet-Connect (read-only, ETH/SOL/ATOM)
- Dashboard mit Portfolio-Übersicht
- Real-Yield-Kalkulator
- VRS-Score-Anzeige (Ampel)
- Email-Alerting
- Graceful Degradation bei API-Ausfällen
- Admin: Pipeline-Health + Basic User-Metriken
- DSGVO: Consent, Löschung, Datenminimierung

**Post-MVP Capabilities (Thomas/Sarah/Dev):**
- PDF-Reports, Multi-Tenant, Public API, SDK, API-Versioning

## Domain-Specific Requirements

### Compliance & Regulatory

**DSGVO (MVP-Pflicht):**
- Privacy Policy mit klarer Beschreibung der Wallet-Daten-Verarbeitung
- Cookie Consent (Cloudflare Turnstile, masemIT Analytics)
- Recht auf Löschung: User kann Account + alle verknüpften Wallet-Adressen löschen
- Datenminimierung: Nur notwendige Daten speichern (Wallet-Adressen, Staking-Positionen)
- Wallet-Adressen als pseudo-anonyme personenbezogene Daten behandeln
- Kein Tracking ohne Consent, kein Profiling

**Crypto-Regulatorik (Awareness, nicht MVP-blockierend):**
- EU MiCA (Markets in Crypto-Assets Regulation): StakeTrack AI ist READ-ONLY Analytics — keine regulierte Tätigkeit, aber Disclaimer erforderlich
- EU DAC8 / CARF 2026: Steuer-Reporting-Pflichten betreffen Exchanges, nicht Analytics-Tools — aber Tax-Export-Feature (Post-MVP) muss diese Standards berücksichtigen
- IRS 1099-DA: Relevant für US-User, treibt Tool-Adoption — kein Compliance-Burden für StakeTrack AI selbst

**KYC/AML-Strategie (MVP):**
- KEIN KYC im MVP — StakeTrack AI ist Read-Only Analytics, keine Finanzdienstleistung
- Wallet-Connect ist pseudonym, keine Identitätsverifikation nötig
- Falls regulatorische Lage sich ändert: KYC als MMS-Service evaluieren (Cross-Project Service Rule)
- Email-Login (Classic Auth) sammelt nur Email — kein zusätzlicher Identitätsnachweis

### Technical Constraints

**Security:**
- SIWE-Signatur serverseitig verifizieren (Replay-Schutz mit Nonce)
- QStash Webhook Endpoints: HMAC Signature Verification Pflicht
- Rate Limiting auf allen Public API Endpoints (Upstash Rate Limit)
- Chain API Keys ausschließlich in Environment Variables
- NeonDB: SSL Required (`sslmode=require`)
- Keine User-Daten in Logs (DSGVO) — Wallet-Adressen sind pseudo-anonym
- NIEMALS Wallet Private Keys speichern oder anfordern

**Datenintegrität:**
- VRS-Scores müssen reproduzierbar sein (gleiche Inputs = gleicher Score)
- Historische Scores sind immutable — keine nachträgliche Änderung
- Daten-Freshness transparent anzeigen (Timestamp pro Datenpunkt)
- Bei Chain-API-Ausfällen: Graceful Degradation mit letztem bekanntem Stand + Warnung

**Data Retention:**
- MVP: Keine automatische Löschung, Daten bleiben erhalten
- User-initiierte Löschung (DSGVO Recht auf Löschung) muss funktionieren
- TimescaleDB Kompression nach 30 Tagen für historische Metriken
- Retention Policies können Post-MVP eingeführt werden

### Blockchain-Domain Constraints

**Multi-Chain-Heterogenität:**
- Jede Chain hat fundamental unterschiedliche Staking-Mechanismen (DPoS, NPoS, Tendermint BFT)
- Validator-Metriken sind NICHT direkt vergleichbar zwischen Chains — Normalisierung im VRS nötig
- Chain-spezifische Edge Cases: Solana's `getStakeActivation` deprecated, SIMD-123 ändert Revenue-Sharing
- Cosmos IBC-Transfers können Staking-Positionen beeinflussen
- ETH Beacon Chain hat andere Slashing-Bedingungen als SOL oder ATOM

**Marktdynamik:**
- Staking-APYs ändern sich ständig — Daten müssen mindestens stündlich aktualisiert werden
- Validator-Commissionen können jederzeit geändert werden — Alert-Trigger nötig
- Token-Inflation ist chain-spezifisch und kann sich durch Governance-Votes ändern
- Slashing-Events sind selten aber kritisch — müssen in Echtzeit erkannt werden

### Haftung & Disclaimer

**Pflicht-Disclaimer (MVP):**
- "StakeTrack AI liefert Informationen, keine Finanzberatung"
- "Vergangene Performance ist kein Indikator für zukünftige Ergebnisse"
- "Staking birgt Risiken einschließlich Slashing und Token-Wertverlust"
- "VRS-Scores basieren auf historischen Daten und sind keine Garantie"
- Disclaimer muss auf Dashboard, Landing Page und in Email-Alerts sichtbar sein

**Haftungsausschluss:**
- StakeTrack AI empfiehlt KEINE Validators und gibt KEINE Anlageempfehlungen
- Read-Only Analytics — User trifft eigene Entscheidungen
- Kein Ersatz für eigene Due Diligence

### Risk Mitigations

| Risiko | Wahrscheinlichkeit | Impact | Mitigation |
|---|---|---|---|
| Chain-API bricht ab / ändert sich | Hoch | Mittel | Adapter-Pattern isoliert Änderungen, Fallback-Endpoints |
| DSGVO-Beschwerde wegen Wallet-Daten | Niedrig | Hoch | Privacy-by-Design, Löschfunktion, Datenminimierung |
| Regulierung klassifiziert Tool als Finanzberatung | Sehr niedrig | Sehr hoch | Klare Disclaimer, Read-Only, keine Empfehlungen |
| Validator-Daten sind fehlerhaft | Mittel | Hoch | Multi-Source-Validation, Confidence Levels, Freshness-Anzeige |
| Slashing-Event wird nicht erkannt | Niedrig | Hoch | Stündliche Checks, QStash Retry, Admin-Monitoring |
| Token-Inflation-Daten veraltet | Mittel | Mittel | Tägliche Aktualisierung, Fallback auf letzte bekannte Werte |

## Innovation & Novel Patterns

### Detected Innovation Areas

**1. Marktlücken-Innovation: "Das unabhängige Rated Labs"**
Die Übernahme von Rated Labs durch Figment (Oktober 2025) hat die einzige unabhängige Validator-Rating-Quelle vom Markt genommen. StakeTrack AI besetzt diese Lücke — mit einem entscheidenden Vorteil: Kein eigener Validator-Betrieb = keine Interessenkonflikte. Unabhängigkeit ist nicht nur ein Feature, sondern das Fundament der Glaubwürdigkeit.

**2. Framing-Innovation: Real Yield als viraler Schock-Moment**
Die Rohdaten (APY, Inflation, Fees) sind öffentlich verfügbar. Die Innovation liegt im Framing: Kein Wettbewerber zeigt systematisch die Differenz zwischen nominalem APY und echtem Ertrag. Der "Schock-Moment" ("Mein 12% APY ist in Wahrheit 2.7%?!") ist kein technisches Feature — es ist eine neue Art, ein bekanntes Problem sichtbar zu machen. Das hat virales Potenzial, weil es emotionale Resonanz erzeugt.

**3. Kombinations-Innovation: Risk + Yield + Multi-Chain**
Einzeln existieren Teile des Angebots: StakingRewards zeigt APYs, DeFiLlama zeigt TVL, Block-Explorer zeigen Validator-Daten. Aber die Kombination von Validator Risk Intelligence, Real-Yield-Transparenz und Multi-Chain-Aggregation in einem Produkt existiert nirgends — weder bei Enterprise-Playern (Kiln ab $50K+/Jahr) noch bei Retail-Tools (kostenlos aber oberflächlich).

**4. Methodik-Transfer: ChainSights GVS → Validator Risk Score**
Der VRS ist keine Neuerfindung, sondern eine bewährte Scoring-Methodik (gewichtete Multi-Faktor-Scores, Confidence Levels, zeitgewichtete Durchschnitte) aus ChainSights, adaptiert für ein neues Anwendungsfeld. Innovation durch Transfer reduziert Risiko bei gleichzeitig hoher Differenzierung.

**5. Vision: AI-gestützte Staking Intelligence**
Der Name "StakeTrack AI" signalisiert die langfristige Vision: Von regelbasiertem Scoring (MVP) zu AI-gestützter Anomalie-Erkennung, prädiktiven Slashing-Warnungen und personalisierten Validator-Empfehlungen. Im MVP ist "AI" die Vision — der VRS-Algorithmus legt das Daten-Fundament für spätere ML-Modelle. Die täglichen Snapshots in TimescaleDB bauen das Training-Dataset auf, das Post-Launch den AI-Layer ermöglicht.

### Market Context & Competitive Landscape

| Aspekt | StakeTrack AI | StakingRewards | DeFiLlama | Kiln/Figment |
|---|---|---|---|---|
| **Real Yield** | Systematisch (APY − Inflation − Fees − Slashing) | Nur nominaler APY | Kein Yield-Focus | Intern, nicht öffentlich |
| **Validator Risk Score** | Unabhängig, transparent, öffentliche Methodik | Kein Scoring | Kein Scoring | Figment/Rated: Interessenkonflikt |
| **Multi-Chain-Aggregation** | ETH, SOL, ATOM (MVP) + erweiterbar | Breit aber oberflächlich | Breit, DeFi-fokussiert | Enterprise-only |
| **Preismodell** | Free / $19 / $99 | Kostenlos | Kostenlos | $50K+/Jahr |
| **Unabhängigkeit** | Kein Validator-Betrieb | Werbefinanziert | Community-driven | Eigene Validators |
| **Zielgruppe** | Retail → Prosumer → B2B | Retail | DeFi-Degens | Enterprise only |

**Timing-Vorteil:** Regulierungswelle 2026 (IRS 1099-DA, EU DAC8, CARF) erzwingt Tool-Adoption quer durch alle Segmente. Wer jetzt die Daten-Infrastruktur aufbaut, hat First-Mover-Advantage wenn Tax-Reporting Pflicht wird.

### Validation Approach

| Innovation | Validierungs-Methode | Erfolgs-Signal |
|---|---|---|
| Real-Yield-Schock-Moment | Public Rechner (F5) ohne Login, Sharing-Rate tracken | >5% Share-Rate auf Farcaster/Bluesky |
| VRS-Glaubwürdigkeit | Öffentliche Methodik-Dokumentation, Korrelation mit historischen Slashing-Events | >95% Korrelation in Backtesting |
| Kombinations-USP | Feature-Usage-Tracking: Nutzen User alle 3 Features oder nur eines? | >60% nutzen mindestens 2 von 3 Core Features |
| Unabhängigkeits-Positionierung | Qualitatives Feedback, NPS-Befragung | "Unabhängig/neutral" in Top-3 der genannten Gründe |
| AI-Vision | Dataset-Aufbau tracken, erste Anomalie-Erkennung Prototypen Post-MVP | Ausreichend Daten nach 6 Monaten für erste ML-Experimente |

### Transparenz als Strategie

Die VRS-Scoring-Methodik wird **öffentlich dokumentiert** — Gewichtungen, Faktoren, Berechnungslogik. Das ist eine bewusste strategische Entscheidung:

- **Vertrauen > Geheimhaltung:** In einem Markt wo Rated Labs gerade wegen Interessenkonflikten (Figment-Übernahme) an Glaubwürdigkeit verloren hat, ist Transparenz der stärkste Differenziator
- **Community-Feedback:** Öffentliche Methodik ermöglicht Feedback von Staking-Experten und stärkt das Produkt
- **Moat durch Execution:** Der Wettbewerbsvorteil liegt nicht im Algorithmus allein, sondern in der Kombination aus Daten-Pipeline, Multi-Chain-Coverage, UX und Markenvertrauen
- **Wissenschaftlicher Ansatz:** Versionierte Methodik-Dokumentation (v1.0, v1.1...) zeigt Evolution und Professionalität

### Innovation Risk Mitigation

| Innovations-Risiko | Wahrscheinlichkeit | Mitigation |
|---|---|---|
| Real-Yield-Framing resoniert nicht | Niedrig | A/B-Testing der Messaging, Public Rechner als validierender Testballon |
| VRS wird als "noch ein Score" wahrgenommen | Mittel | Transparente Methodik + konkrete Case Studies (historische Slashing-Events korrekt vorhergesagt) |
| Enterprise-Player kopieren das Modell | Mittel | First-Mover-Advantage, Community-Trust, Agilität als EPU vs. Corporate-Trägheit |
| AI-Vision wird nie eingelöst | Niedrig | MVP funktioniert ohne AI — "AI" ist Vision, nicht Versprechen. VRS ist regelbasiert und wertvoll ohne ML |
| Transparente Methodik wird von Validators manipuliert | Niedrig | Zeitgewichtete Scores, Multi-Faktor-Ansatz erschwert Gaming. Manipulation-Detection als Post-MVP Feature |

## SaaS B2B/B2C + Blockchain/Web3 Specific Requirements

### Project-Type Overview

StakeTrack AI ist ein **Hybrid aus SaaS B2B/B2C und Blockchain/Web3** — eine Web-App mit Dashboard-Fokus, Wallet-basierter Authentifizierung und Multi-Chain-Datenintegration. Der B2C-Layer (Lisa) ist MVP, der B2B-Layer (Thomas, Sarah, Dev) wächst Post-MVP organisch durch Tier-Erweiterung und API-Zugang.

Keine Smart Contracts, keine On-Chain-Transaktionen — StakeTrack AI ist **Read-Only Analytics**. Blockchain-Spezifika betreffen ausschließlich Daten-Ingestion (Chain APIs) und Wallet-Authentifizierung (SIWE).

### Multi-Tenancy Model

**MVP: Single-Tenant, Multi-Wallet**
- Ein User-Account verknüpft mit mehreren Wallet-Adressen (verschiedene Chains)
- Alle Wallets eines Users aggregiert in einem Dashboard
- Freemium-Gating auf User-Ebene (Free: 1 Chain / 10 Validators, Pro: 3 Chains / 100 Validators)
- Keine Trennung zwischen "eigenen" und "fremden" Portfolios

**Post-MVP: Multi-Tenant für Prosumer/Business**
- Thomas-Persona: Verwaltet separate Klienten-Portfolios unter seinem Account
- Klienten-Portfolios sind isoliert (Datentrennung)
- PDF-Reports pro Klient generierbar
- Business-Tier ($99/Mo) schaltet Multi-Portfolio-Funktion frei

### Permission & Role Model

**MVP-Rollen:**

| Rolle | Rechte | Zugang |
|---|---|---|
| `user` | Dashboard, Wallet-Connect, Real-Yield, VRS-Anzeige, Alerts (tier-limitiert) | Standard nach Registration |
| `admin` | Alles von `user` + Admin-Dashboard, Pipeline-Health, User-Metriken, Config | Sempre only |

**Post-MVP-Erweiterung:**

| Rolle | Rechte | Zugang |
|---|---|---|
| `api_consumer` | REST API Zugang, Rate-Limited, Read-Only | Business-Tier + API-Key |
| `portfolio_manager` | Multi-Portfolio-Verwaltung, Report-Generierung | Business-Tier |

**Implementierung:**
- Rolle im JWT-Token (`role` Claim) oder per DB-Lookup (cached in Upstash Redis)
- Middleware-basierter Tier-Check auf API Routes
- Admin-Zugang: Hardcoded Wallet-Adresse oder separater Auth-Flow

### Subscription & Feature-Gating

| Feature | Free | Pro ($19/Mo) | Business ($99/Mo) |
|---|---|---|---|
| Chains | 1 | 3 | 3 |
| Validators tracked | 10 | 100 | Unlimited |
| VRS | Basic (3 Faktoren) | Full (5 Faktoren) | Full + Historie |
| Real-Yield-Kalkulator | Public (ohne Login) | Personalisiert | Personalisiert + Export |
| Alerts | 1 (Proof-of-Value) | 5 | 25 |
| API Access | Nein | Nein | Read-only REST API |
| PDF-Reports | Nein | Nein | Ja |
| Multi-Portfolio | Nein | Nein | Ja |

**Gating-Implementierung:**
- Middleware auf API Routes prüft Tier aus Session/JWT
- Feature-Flags als Konfiguration, nicht hardcoded in Komponenten
- Upgrade-CTAs im UI bei Feature-Limit-Erreichen
- Stripe/Payment-Integration: Post-MVP (manuelles Onboarding für erste Pro-User evaluieren)

### Integration Map

#### Interne masemIT-Services

| Service | Zweck | Auth | Richtung |
|---|---|---|---|
| **MMS** (api.masem.at) | User/Party-Management, Consents | API-Key, Server-to-Server | StakeTrack → MMS |
| **masemIT Analytics** | User-Tracking, Feature-Usage, Session-Dauer | Client-Side Script | Browser → Analytics |
| **Resend** | Alert-Emails, Onboarding, VRS-Warnungen | API-Key, Server-to-Server | StakeTrack → Resend |

#### Infrastruktur-Services

| Service | Zweck | Verbindung |
|---|---|---|
| **Vercel** | Hosting, Serverless Functions, Preview Deployments | Git-Push → Auto-Deploy |
| **NeonDB** | PostgreSQL 16 + TimescaleDB, Hypertables, Preview Branches | `@neondatabase/serverless`, SSL Required |
| **Upstash Redis** | Cache-Layer (Validators 5min, VRS 1h, Yield 24h) | `@upstash/redis`, REST API |
| **Upstash QStash** | Cron-Jobs, Fan-out (3 Chain-Fetcher parallel), Retry-Logic | `@upstash/qstash`, Webhook → `/api/cron/*` |
| **Upstash Rate Limit** | Rate Limiting auf Public API Endpoints | `@upstash/ratelimit`, Middleware |
| **Cloudflare Turnstile** | Bot-Schutz bei Signup/Login | Client-Side Widget + Server Verification |

#### Blockchain Data Sources

| Chain | Primäre API | Fallback | Daten |
|---|---|---|---|
| **Ethereum** | Beacon Chain API (Lighthouse/Prysm) | Etherscan API, Beaconcha.in | Validators, Staking, Slashing |
| **Solana** | Solana RPC (`getAccountInfo`) | Helius, Triton | Stake Accounts, Validators |
| **Cosmos** | LCD/REST API (Cosmos Hub) | Mintscan API | Validators, Delegations, Inflation |

### Wallet Integration Architecture

| Chain | Wallet-Lösung | Auth-Methode |
|---|---|---|
| **Ethereum** | RainbowKit + wagmi + viem | SIWE (Sign-In with Ethereum) via NextAuth |
| **Solana** | `@solana/wallet-adapter-react` | Message-Signing (analog SIWE) |
| **Cosmos** | Keplr (`@keplr-wallet/types`) | ADR-036 Message-Signing |

**Unified Identity:**
- Ein User-Account kann Wallets aller 3 Chains verknüpfen
- Primäre Auth: SIWE (ETH-Wallet) oder Classic (Email via MMS)
- Zusätzliche Wallets werden als "linked wallets" gespeichert
- JWT `sub` = primäre Wallet-Adresse oder User-ID (bei Email-Login)

### Implementation Considerations

**Serverless Constraints:**
- Vercel Pro Timeout: 60 Sekunden — Batch-Verarbeitung bei großen Datenmengen via QStash Fan-out
- NeonDB Connection: `connection_limit=1` als Default für Serverless Functions
- Cold Start < 1s durch Drizzle ORM (90% kleinere Bundles als Prisma)

**Caching-Strategie (3 Layer):**
- Layer 1: RSC (revalidate-basiert: 300s Dashboard, 3600s VRS, 86400s Yield)
- Layer 2: Upstash Redis (Cache-Aside, Key-Pattern: `staketrack:{entity}:{chain}:{id}`)
- Layer 3: NeonDB (Pre-computed Aggregates, Materialized Views)
- Cache-Invalidierung bei jedem Cron-Write

**Payment (MVP-Strategie):**
- MVP: Manuelles Onboarding für erste Pro-User (Stripe-Link oder Direktkontakt)
- Evaluieren ob Payment über MMS als Cross-Project Service abgebildet werden soll
- Post-MVP: Self-Service Stripe-Integration mit Webhook → Tier-Update

## Project Scoping & Phased Development

### MVP Strategy & Philosophy

**MVP-Ansatz: Problem-Solving MVP**
"Zeige mir meinen echten Staking-Ertrag auf einen Blick" — das MVP löst ein konkretes, messbares Problem für eine klar definierte Zielgruppe (Lisa-Persona). Kein Feature-Feuerwerk, sondern ein Werkzeug das 6 Browser-Tabs durch 1 Dashboard ersetzt.

**Resource Requirements:**
- Solo-Entwicklung (Sempre) mit AI-Unterstützung
- Infrastrukturkosten: ~$0/Monat zusätzlich (bestehender masemIT-Stack)
- Kein Custom-Admin-Dashboard — existierende Tools (Vercel, NeonDB Console, QStash Dashboard) reichen für Launch

### MVP Feature Set (Phase 1)

**Core User Journeys Supported:**
- Lisa (B2C) — komplett
- Sempre (Admin) — via existierende Tooling-Infrastruktur (Vercel Dashboard, NeonDB Console, QStash Dashboard, masemIT Analytics)

**Must-Have Capabilities (F1-F5):**

| ID | Feature | Must-Have-Begründung | Details |
|---|---|---|---|
| **F1** | Multi-Chain Dashboard | Ohne Dashboard kein Produkt | Wallet-Connect (ETH/SOL/ATOM), Positionen-Übersicht, Portfolio-Aggregation |
| **F2** | Real-Yield-Kalkulator | USP und viraler Hook — ohne Real Yield kein Differenziator | Nominal APY − Inflation − Fees − Slashing = Real Yield, visueller Vergleich |
| **F3** | Validator Risk Score | Zweiter USP — Ampel-System gibt sofortigen Mehrwert | 5-Faktoren-Score (0-100), Grün/Gelb/Rot, Confidence Level |
| **F4** | Basis Alerting | Retention-Mechanismus — holt User zurück | Email-Alerts bei VRS-Änderungen, Commission-Erhöhung, Slashing. Free: 1 Alert, Pro: 5 |
| **F5** | Public Real-Yield-Rechner | Viraler Einstiegspunkt, SEO-Anker, Conversion-Funnel | Eigene `/calculator` Route, SSR-optimiert, ohne Login nutzbar, CTA → Wallet-Connect |

**Aktualisiertes Feature-Gating (MVP):**

| Feature | Free | Pro ($19/Mo) |
|---|---|---|
| Chains | 1 | 3 |
| Validators tracked | 10 | 100 |
| VRS | Basic (3 Faktoren) | Full (5 Faktoren) |
| Alerts | 1 (Proof-of-Value) | 5 |
| Real-Yield-Rechner | Public (ohne Login) | Personalisiert |

**Bewusst NICHT im MVP:**
- Custom Admin-Dashboard (existierende Tools reichen)
- PDF-Reports (Thomas-Persona ist Post-MVP)
- Public API (Sarah/Dev-Personas sind Post-MVP)
- Multi-Portfolio-Management (Post-MVP)
- Payment Self-Service (manuelles Onboarding für erste Pro-User)

### MVP Technical Scope

**Was gebaut wird:**
- Next.js 15 App Router mit 3 Hauptrouten: Dashboard, Calculator (`/calculator`), Auth
- 3 Chain Adapters (ETH, SOL, ATOM) mit Unified Model
- VRS Scoring Engine (5 Faktoren, konfigurierbare Gewichtungen)
- Real-Yield Calculator (Formel + tägliche Snapshots)
- QStash Cron Pipeline (stündlich Chain-Data, täglich VRS + Yield)
- Alerting via Resend (Email)
- Auth: SIWE (RainbowKit) + Classic (Email via MMS)
- Freemium-Gating Middleware

**Was NICHT gebaut wird (existierende Tools nutzen):**
- Admin-Dashboard → Vercel Dashboard + NeonDB Console + QStash Dashboard
- Monitoring → Vercel Analytics + QStash Monitoring
- User-Analytics → masemIT Analytics (Client-Side Script)
- Error-Tracking → Vercel Logs (MVP-ausreichend)

### Post-MVP Features

**Phase 2 — Growth (Monat 4-6):**

| Feature | Persona | Trigger |
|---|---|---|
| Tax-Export Engine (CSV/PDF) | Lisa, Thomas | Launch VOR Q1 Tax Season |
| Business-Tier ($99/Mo) | Thomas | Genug Pro-User für Upsell |
| PDF-Report-Generierung | Thomas | Business-Tier |
| 3-5 weitere Chains (DOT, ADA, AVAX) | Alle | User-Demand |
| Erweiterter VRS (mehr Faktoren, Historie) | Lisa, Thomas | Daten-Fundament steht |
| Validator-Vergleichs-Tool | Lisa | Feature-Request-Analyse |
| Custom Admin-Dashboard | Sempre | Wenn manuelle Tools nicht mehr reichen |
| Stripe Self-Service Payment | Alle | >50 Pro-User |

**Phase 3 — Expansion (Monat 7-12):**

| Feature | Persona | Trigger |
|---|---|---|
| Public REST API für VRS | Sarah, Dev | Business-Tier validiert |
| Enterprise-Tier ($299-$999/Mo) | Sarah | Erste API-Kunden |
| TypeScript SDK | Dev | API-Adoption |
| Multi-Portfolio-Management | Thomas | Business-User-Feedback |
| AI-Anomalie-Erkennung (erste ML-Modelle) | Alle | 6+ Monate Trainingsdaten |
| Slashing Early Warning System | Sarah, Lisa | ML-Modell performt |
| API-Versioning (v1/v2) | Dev | Breaking Changes nötig |

### Risk Mitigation Strategy

**Technische Risiken:**

| Risiko | Mitigation | Fallback |
|---|---|---|
| Multi-Chain-Integration komplexer als geplant | Chain Adapter Pattern isoliert Komplexität. Eine Chain nach der anderen implementieren (ETH zuerst, dann SOL, dann ATOM) | MVP mit 2 statt 3 Chains launchen |
| Solana API-Deprecation (`getStakeActivation`) | Von Anfang an `getAccountInfo` + manuelles Parsing verwenden | Helius als managed Fallback |
| TimescaleDB-Performance bei wachsenden Daten | Kompression nach 30 Tagen, wöchentliche Chunks | Materialized Views für Aggregates |
| Vercel 60s Timeout bei Chain-Fetching | QStash Fan-out: 3 parallele Fetcher, jeder < 30s | Batch-Size reduzieren |

**Marktrisiken:**

| Risiko | Mitigation | Fallback |
|---|---|---|
| Real-Yield-Hook resoniert nicht | F5 (Public Calculator) als Testballon VOR vollem Launch. Sharing-Rate als Signal | Pivot auf VRS als primären Hook |
| Zu wenig Zahlungsbereitschaft | Manuelles Onboarding für erste Pro-User, direktes Feedback | Free-Tier erweitern, B2B-API früher launchen |
| Enterprise-Player kopieren schnell | First-Mover-Advantage + Transparenz-Differenziator nutzen | Nische verteidigen: "Unabhängig" als Brand |

**Ressourcen-Risiken:**

| Risiko | Mitigation | Fallback |
|---|---|---|
| Solo-Entwicklung dauert länger als geplant | MVP strikt auf F1-F5 begrenzt, kein Feature-Creep | Launch mit F1+F2+F3 (Kern-Value), F4+F5 als Fast-Follow |
| Externe API-Kosten steigen | Free-Tier-APIs bevorzugen, Caching aggressiv nutzen | Premium-API-Kosten an User weitergeben (Tier-Pricing anpassen) |
| Burnout/Motivation | Phasen-basierte Meilensteine mit klaren Go/No-Go-Kriterien | 3-Monats-Checkpoint: Pivot oder Pause erlaubt |

### Absolute Minimum (Emergency Scope)

Falls Ressourcen knapper als geplant: **F1 + F2 + F3** reichen als launchfähiges Produkt. Das Dashboard mit Real-Yield und VRS-Scores löst bereits das Kernproblem. F4 (Alerting) und F5 (Public Calculator) können als schnelle Nachzügler kommen.

## Functional Requirements

### Wallet & Identity Management

- **FR1:** User kann eine Ethereum-Wallet verbinden und sich per Signatur authentifizieren
- **FR2:** User kann eine Solana-Wallet verbinden und sich per Signatur authentifizieren
- **FR3:** User kann eine Cosmos-Wallet verbinden und sich per Signatur authentifizieren
- **FR4:** User kann sich alternativ per Email-Login registrieren und authentifizieren
- **FR5:** User kann mehrere Wallets (verschiedene Chains) mit einem Account verknüpfen
- **FR6:** User kann verknüpfte Wallets einsehen und einzeln entfernen
- **FR7:** User kann seinen Account und alle verknüpften Daten vollständig löschen (DSGVO)

### Staking Dashboard & Portfolio

- **FR8:** User kann nach Wallet-Connect alle Staking-Positionen über alle verknüpften Wallets und Chains aggregiert auf einem Dashboard sehen
- **FR9:** User kann seine Staking-Positionen nach Chain filtern
- **FR10:** User kann pro Position den aktuellen Validator, delegierten Betrag und Status einsehen
- **FR11:** User kann den Gesamt-Portfolio-Wert und die Gesamt-Performance sehen
- **FR12:** System zeigt pro Datenpunkt den Zeitstempel der letzten Aktualisierung an
- **FR13:** System zeigt bei temporär nicht verfügbaren Chain-Daten den letzten bekannten Stand mit Hinweis an (Graceful Degradation)
- **FR14:** System zeigt bei Wallets ohne Staking-Positionen einen informativen Leer-Zustand an

### Real-Yield-Kalkulation

- **FR15:** User kann den Real Yield (Nominal APY − Inflation − Validator Fee − Slashing-Risiko) pro Staking-Position einsehen
- **FR16:** User kann den aggregierten Real Yield über sein gesamtes Portfolio sehen
- **FR17:** User kann den Real Yield zwischen verschiedenen Chains und Validators visuell vergleichen
- **FR18:** Nicht-authentifizierter Besucher kann den Public Real-Yield-Rechner auf einer eigenen Route nutzen (ohne Login)
- **FR19:** Public Real-Yield-Rechner zeigt einen Call-to-Action zur Wallet-Verbindung für personalisierte Ergebnisse
- **FR20:** System berechnet und speichert tägliche Yield-Snapshots pro Chain und Validator

### Validator Risk Score (VRS)

- **FR21:** User kann den VRS (0-100) pro Validator als Ampel-Darstellung (Grün/Gelb/Rot) einsehen
- **FR22:** User kann die Aufschlüsselung des VRS in seine 5 Faktoren (Uptime, Slashing-Historie, Commission-Stabilität, Performance, Dezentralisierung) einsehen
- **FR23:** User kann das Confidence Level eines VRS-Scores sehen (basierend auf Datenverfügbarkeit)
- **FR24:** System berechnet VRS-Scores zeitgewichtet (letzte 30 Tage stärker gewichtet als 90 Tage)
- **FR25:** System speichert historische VRS-Scores als immutable Einträge
- **FR26:** Free-User sehen einen reduzierten VRS (3 Faktoren), Pro-User den vollen VRS (5 Faktoren)

### Alert-System

- **FR27:** User kann Email-Alerts für VRS-Änderungen seiner überwachten Validators konfigurieren
- **FR28:** User kann Email-Alerts für Commission-Erhöhungen konfigurieren
- **FR29:** User kann Email-Alerts für Slashing-Events konfigurieren
- **FR30:** Free-User können maximal 1 Alert konfigurieren, Pro-User maximal 5
- **FR31:** System versendet Alerts mit kontextbezogenen Informationen (was hat sich geändert, aktueller VRS, betroffener Validator)
- **FR32:** User kann konfigurierte Alerts einsehen, bearbeiten und löschen

### Freemium & Account-Tiers

- **FR33:** System begrenzt Features basierend auf dem Tier des Users (Free vs. Pro)
- **FR34:** Free-User sind auf 1 Chain und 10 überwachte Validators begrenzt
- **FR35:** System zeigt bei Erreichen eines Feature-Limits einen Upgrade-Hinweis an
- **FR36:** Admin kann den Tier eines Users manuell ändern (MVP: kein Self-Service-Payment)

### Daten-Pipeline & System-Capabilities

- **FR37:** System aktualisiert Validator- und Staking-Daten mindestens stündlich über alle 3 Chains
- **FR38:** System aktualisiert Chain-spezifische Inflationsraten täglich
- **FR39:** System berechnet VRS-Scores und Yield-Snapshots nach jeder Datenaktualisierung
- **FR40:** System invalidiert Caches nach jeder erfolgreichen Datenaktualisierung
- **FR41:** System verifiziert die Signatur auf allen Cron-Webhook-Endpoints (HMAC)
- **FR42:** System wendet Rate-Limiting auf alle öffentlichen API-Endpoints an

## Non-Functional Requirements

### Performance

- **NFR1:** Dashboard TTFB (Time to First Byte) < 500ms für authentifizierte User
- **NFR2:** Cached API-Responses < 100ms
- **NFR3:** Uncached API-Responses < 2s (inkl. Chain-Adapter-Abfrage)
- **NFR4:** Public Real-Yield-Rechner (`/calculator`) First Load JS < 100KB (SSR-optimiert, minimaler Client-JS)
- **NFR5:** Gesamt First Load JS Bundle < 200KB
- **NFR6:** Serverless Cold Start < 1s
- **NFR7:** Wallet-Connect bis erste Daten-Anzeige < 60 Sekunden (Time-to-First-Insight)

### Security

- **NFR8:** Alle SIWE-Signaturen werden serverseitig verifiziert (Replay-Schutz mit Nonce)
- **NFR9:** Alle Datenbank-Verbindungen über SSL (`sslmode=require`)
- **NFR10:** Keine Wallet Private Keys werden gespeichert, übertragen oder angefordert
- **NFR11:** Alle Chain API Keys ausschließlich in serverseitigen Environment Variables (nie im Client-Bundle)
- **NFR12:** HMAC Signature Verification auf allen QStash Cron-Webhook-Endpoints
- **NFR13:** Rate-Limiting auf allen öffentlichen Endpoints (Upstash Rate Limit)
- **NFR14:** Bot-Schutz auf Signup/Login (Cloudflare Turnstile)
- **NFR15:** Keine personenbezogenen Daten in Server-Logs (DSGVO)
- **NFR16:** JWT-Sessions mit angemessener Expiry-Zeit und Refresh-Mechanismus

### Scalability

- **NFR17:** System unterstützt 500 registrierte User und 150 WAU ohne Performance-Degradation (Phase 1)
- **NFR18:** System skaliert auf 5.000 registrierte User ohne Architektur-Änderung (Phase 3)
- **NFR19:** NeonDB Storage bleibt unter 0.5 GiB für 12 Monate MVP-Betrieb
- **NFR20:** Caching-Layer (Upstash Redis) absorbiert mindestens 80% der Read-Requests
- **NFR21:** QStash Fan-out verarbeitet alle 3 Chain-Fetcher parallel innerhalb von 60 Sekunden (Vercel Timeout)

### Reliability

- **NFR22:** System-Uptime > 99.5%
- **NFR23:** Daten-Freshness: Validator-Daten maximal 1 Stunde alt unter Normalbetrieb
- **NFR24:** Bei Chain-API-Ausfall: Graceful Degradation mit letztem bekanntem Stand (kein Crash, keine leere Seite)
- **NFR25:** QStash Retry-Mechanismus stellt sicher, dass fehlgeschlagene Cron-Jobs automatisch wiederholt werden
- **NFR26:** VRS-Score-Berechnung ist deterministisch (gleiche Inputs = gleicher Score, reproduzierbar)
- **NFR27:** Historische VRS-Scores und Yield-Snapshots sind immutable (keine nachträgliche Änderung)

### Integration & Data Integrity

- **NFR28:** Chain Adapter sind voneinander isoliert — Ausfall eines Adapters beeinträchtigt nicht die anderen Chains
- **NFR29:** VRS-Berechnung korreliert > 95% mit historischen On-Chain-Slashing-Events (Accuracy)
- **NFR30:** Cache-Invalidierung erfolgt synchron nach jedem erfolgreichen Daten-Write
- **NFR31:** Alle externen API-Integrationen (MMS, Resend, Chain APIs) haben definierte Timeouts und Fehlerbehandlung

### Compliance

- **NFR32:** DSGVO-konform: Privacy Policy, Cookie Consent, Datenminimierung, Recht auf Löschung
- **NFR33:** Disclaimer auf Dashboard, Landing Page und in Email-Alerts sichtbar ("Keine Finanzberatung")
- **NFR34:** Wallet-Adressen werden als pseudo-anonyme personenbezogene Daten behandelt

