# Product Brief: PayWatcher Phase 1 — Frontend

**Project:** PayWatcher (masemIT Product)
**Phase:** 1 — Landing Page, User Dashboard, Admin Dashboard
**Version:** 1.0
**Date:** 2026-02-15
**Author:** Mario Semper (PO)
**Status:** Draft
**Backend:** MMS Module (bereits implementiert, api.masem.at)
**Referenzdokumente:** paywatcher-architecture.md, paywatcher-epics.md, market-paywatcher-research-2026-02-14.md

---

## 1. Kontext & Problem

### Was ist PayWatcher?

PayWatcher ist eine Developer-First Verification API für Stablecoin-Zahlungen auf EVM-Chains. Es ist **kein Payment Processor** — PayWatcher bewegt kein Geld, hält keine Funds, und macht keine Währungskonvertierung. Es ist ein reiner Verification Layer: "Sag mir welche Zahlung du erwartest, ich sage dir wenn sie angekommen und bestätigt ist."

**MVP-Fokus:** USDC auf Base Network.

### Marktlücke (aus Market Research)

- **Kein reines Verification-Only-Produkt existiert** — alle Wettbewerber bündeln Custody, Settlement oder Payment Processing
- **Shared Address + Unique Amount als Service** wird von keinem Anbieter unterstützt
- **Flat-Fee statt %-pro-Transaktion** gibt es nicht — alle Processors nehmen 0.5–2%
- **Developer Experience Gap** — Wahl zwischen "zu high-level" (Payment Processor diktiert UX) und "zu low-level" (rohe Transfer Events, alles selbst bauen)

### Was fehlt?

Das Backend (MMS Module) ist fertig:
- POST /v1/payments (Payment Intent erstellen)
- GET /v1/payments/:id (Status abfragen)
- Blockchain Polling (QStash Cron, ~10s)
- Webhook Delivery (payment.confirmed, payment.expired, payment.failed)
- Sweep Worker (Hot → Cold Wallet)
- Auth (API Key + Scopes: payments:read, payments:write)

**Es fehlt alles Kundenseitige:**
1. Landing Page — damit Devs PayWatcher finden und verstehen
2. User Dashboard — damit Tenants ihre Payments sehen und API Keys verwalten
3. Admin Dashboard — damit ich (Mario) den Service überwachen und managen kann

---

## 2. Zielgruppe

### Primary: Web3 Developer / Technical Founder

- Baut DApp, Marketplace, oder SaaS mit Stablecoin-Zahlungen
- Will USDC-Zahlungen verifizieren, ohne einen Payment Processor zu nutzen
- Schätzt clean APIs, gute Docs, und Self-Service Onboarding
- Entscheidet basierend auf Code-Beispielen, nicht Marketing-Slides
- Pain: "Coinbase Commerce nimmt 1% und diktiert meinen Checkout-Flow" oder "Ich baue gerade meinen eigenen Payment Listener auf Alchemy Webhooks und es ist ein Albtraum"

### Secondary: Crypto-Native Small Business

- Akzeptiert Stablecoin-Zahlungen (z.B. Freelancer, Agentur, kleiner E-Commerce)
- Braucht einfache Bestätigung: "Hat mein Kunde gezahlt?"
- Weniger technisch, nutzt eher Dashboard als API direkt

---

## 3. Deliverables

### 3.1 Landing Page (paywatcher.dev)

**Zweck:** Conversion — Besucher → API Key Signup

**Zielgruppe:** Developer, die eine Stablecoin-Verification-Lösung suchen

**Domain:** paywatcher.dev ✅ (gekauft)

**Branding:** Eigenes PayWatcher-Branding (Dark Mode, Developer-Ästhetik). "by masemIT" im Footer.

**Kern-Positionierung:**
- "Payment verification without the payment processor."
- Non-custodial, kein Money Transmitter, keine %-Fees
- Developer-first: Code-Beispiele > Marketing-Text

**Sektionen:**

| # | Sektion | Inhalt |
|---|---------|--------|
| 1 | **Hero** | Headline + Subline + CTA ("Get API Key") + Code-Snippet (3 Zeilen: create → wait → webhook) |
| 2 | **Problem/Solution** | "Today: 1% fees OR build from scratch. PayWatcher: the missing middle." |
| 3 | **How It Works** | 3 Steps visuell: Create Intent → Customer Pays → Webhook Fires |
| 4 | **Code Examples** | curl + JavaScript/TypeScript Beispiele, Webhook Payload |
| 5 | **Pricing** | Free/Starter/Pro/Enterprise Tiers (Tabelle aus Market Research) |
| 6 | **Comparison** | "$100 at Coinbase Commerce vs. $0.05 at PayWatcher for a $10k payment" |
| 7 | **Trust Signals** | Non-custodial, Base Network, USDC, Open Architecture, "Built by masemIT" |
| 8 | **Footer** | Docs, GitHub, Legal, masemIT Links |

**Design-Prinzipien:**
- Developer Audience = Code > Copy, Klarheit > Grafiken
- Referenz-Style: Stripe, Resend, Clerk Landing Pages
- masemIT Brand Colors: Primary #093236, Accent #009BB1
- Dark Mode bevorzugt (Web3 Convention)
- Responsive, Mobile-First
- Keine generische "Web3 is the future"-Rhetorik
- Kein Wallet-Connect (das ist ein Dev-Tool, kein Consumer-Produkt)

**Tagline-Kandidaten (finale Entscheidung im Design-Sprint):**
- "Watch payments. Don't touch them."
- "Payment verification without the payment processor."
- "We verify. You build."
- "Verification as a Service."
- "$0.05 to verify a $10,000 payment."

**Analytics Events:**

| Event | Trigger | Properties |
|-------|---------|------------|
| `lp_view` | Page loaded | `referrer`, `utm_source` |
| `lp_hero_cta` | Hero CTA clicked | `cta_text` |
| `lp_pricing_view` | Pricing section scrolled into view | — |
| `lp_pricing_cta` | Pricing tier CTA clicked | `tier` |
| `lp_code_copy` | Code example copied | `language`, `section` |
| `lp_docs_click` | Docs link clicked | — |
| `lp_signup_start` | Signup flow initiated | `source_section` |
| `lp_signup_complete` | Signup completed | `tier` |

---

### 3.2 User Dashboard (app.paywatcher.dev oder /dashboard Route)

**Zweck:** Self-Service Management für Tenants

**Auth:** Magic Link oder Email/Password (TBD — Erfahrung aus kurzum.app/hoki.help übernehmen)

**Seiten & Funktionen:**

#### 3.2.1 Overview / Home

- Payment-Statistiken: Total Payments, Confirmed, Expired, Failed (letzte 7/30/90 Tage)
- Mini-Chart: Payments over Time (einfaches Line Chart)
- Recent Payments Liste (letzte 10)
- Quick Links: API Docs, Create Payment (Test), Settings

#### 3.2.2 Payments

- Tabelle aller Payments mit Filtern:
  - Status (pending, matched, confirming, confirmed, expired, failed)
  - Zeitraum (from/to)
  - Amount Range
- Sortierbar nach created_at, amount, status
- Pagination
- Payment Detail View:
  - Alle Felder: id, status, amount, exactAmount, currency, chain, confirmations, confirmationsRequired, txHash, metadata, expiresAt, timestamps
  - Status-Timeline (visuell: pending → matched → confirming → confirmed)
  - Webhook Delivery History (attempts, response codes)
  - Link zum Block Explorer (Base → Basescan) wenn txHash vorhanden

#### 3.2.3 Settings

- **API Keys:** Anzeige (masked), Regenerate, Scopes anzeigen
- **Webhook Configuration:** URL, Test Webhook senden, Delivery Log
- **Confirmation Override:** Default Confirmations (2–20)
- **Deposit Address:** Anzeige (read-only im MVP)
- **Account:** Email, Notifications Preferences

#### 3.2.4 API Docs (Embedded oder Link)

- Quick Start Guide
- Endpoint Reference (aus bestehendem Backend)
- Webhook Event Reference
- Code Examples (curl, JS, Python)
- Option: Eingebettet oder Link zu separater Docs-Seite (z.B. docs.paywatcher.com via Mintlify/Fumadocs)

**Dashboard Analytics Events:**

| Event | Trigger | Properties |
|-------|---------|------------|
| `dash_view` | Dashboard page loaded | `page` |
| `dash_payment_detail` | Payment detail viewed | `payment_id`, `status` |
| `dash_filter` | Filter applied | `filter_type`, `filter_value` |
| `dash_api_key_regen` | API Key regenerated | — |
| `dash_webhook_test` | Test webhook sent | `result` |
| `dash_docs_view` | Docs page viewed | `section` |

---

### 3.3 Admin Dashboard (admin.paywatcher.dev oder /admin Route)

**Zweck:** Operations & Monitoring für mich (Mario)

**Auth:** Separate Admin Auth (hardcoded Admin API Key oder separate Admin Login)

**Seiten & Funktionen:**

#### 3.3.1 System Overview

- Gesamtstatistiken: Total Payments (all tenants), Confirmation Rate, Average Confirmation Time
- System Health: Last Poll Timestamp, RPC Error Rate, Last Block Polled
- Active Tenants Count
- Unmatched Transfers (USDC an Deposit Address ohne matching Payment)
- Hot Wallet Balance (USDC + ETH für Gas)
- Cold Wallet Balance
- Sweep History (letzte Sweeps mit txHash, Amount, Timestamp)

#### 3.3.2 Tenants

- Liste aller Tenants mit Status (enabled/disabled)
- Pro Tenant: Payment Count, Total Volume, Webhook Success Rate
- Tenant Enable/Disable Toggle
- Tenant Detail: Config anzeigen (webhook_url, confirmation_override, deposit_address)
- Neuen Tenant anlegen (mit API Key Generation → einmalig anzeigen)

#### 3.3.3 Payments (Global)

- Wie User Dashboard Payments, aber über alle Tenants
- Zusätzlich: Tenant-Filter
- Zusätzlich: Unmatched Transfers Ansicht
- Zusätzlich: Failed Payments mit Reason (Reorg, etc.)

#### 3.3.4 Webhooks (Global)

- Webhook Delivery Health: Success Rate über alle Tenants
- Failed Deliveries Liste (alle Tenants)
- Retry Queue Status

#### 3.3.5 System / Config

- Environment Variables Status (nicht Werte, nur ob gesetzt: ✅/❌)
- QStash Cron Status (Poll, Expire, Sweep — letzte Ausführung + nächste geplante)
- Redis Status (Connected, Last Block Key)
- RPC Status (Alchemy — Response Time, Error Rate)
- Manual Actions: Force Poll, Force Expire Check, Force Sweep (mit Confirmation Dialog)

---

## 4. Technische Rahmenbedingungen

### Tech Stack (Empfehlung, finale Entscheidung im Architecture Doc)

| Komponente | Empfehlung | Begründung |
|-----------|-----------|------------|
| **Framework** | Next.js 15 (App Router) | Bewährt in masemIT Stack (ChainSights, tellingCube, hoki.help) |
| **Styling** | Tailwind CSS | Standard im Stack |
| **UI Components** | shadcn/ui | Standard im Stack, konsistent mit anderen Produkten |
| **Charts** | Recharts oder tremor | Lightweight, React-native |
| **Auth** | NextAuth.js oder Custom Magic Link | Erfahrung aus kurzum.app übertragen |
| **API Client** | Fetch + SWR oder TanStack Query | Für Dashboard Polling/Refresh |
| **Deployment** | Vercel Pro | Bestehendes Setup |
| **Domain** | paywatcher.dev ✅ | Gekauft, .dev = Developer-Audience perfekt, Google Registry |
| **Analytics** | masemIT Analytics (analytics.masem.at) | Eigenes Produkt, kein Third-Party |
| **Branding** | Eigenes PayWatcher-Branding | Dark Mode, Developer-Ästhetik, "by masemIT" im Footer |

### API-Integration

Das Dashboard kommuniziert mit dem bestehenden MMS Backend (api.masem.at):

- **User Dashboard:** Nutzt reguläre API Keys mit payments:read Scope
- **Admin Dashboard:** Nutzt Admin API Key mit erweiterten Scopes oder separater Admin Auth
- **Landing Page:** Kein API-Zugriff (statisch), Signup-Flow: Free Tier Self-Service → API Key sofort, Paid Tiers nach Review/Onboarding

### Offene Architektur-Fragen (zu klären im Architecture Doc)

1. **Monorepo oder separate Repos?** Landing Page + Dashboard in einem Next.js Projekt oder getrennt?
2. **Auth-Strategie:** Magic Link, Email/Password, oder API Key-basiert für Dashboard Login?
3. **Admin Auth:** Separate App oder /admin Route mit Role-Check?
4. ~~**Domain-Strategie**~~ → ✅ **paywatcher.dev** (gekauft). Subdomains: docs.paywatcher.dev, app.paywatcher.dev (oder integriert)
5. **Docs-Lösung:** Eingebettet im Dashboard, separate Docs-App (Mintlify/Fumadocs), oder einfache Markdown-Seiten?
6. **Signup-Flow:** ✅ Free Tier Self-Service, Paid Tiers nach Review/Onboarding

---

## 5. Abgrenzung (Nicht in Phase 1)

| Feature | Warum nicht in Phase 1 | Phase |
|---------|----------------------|-------|
| Multi-Chain Support | MVP = USDC auf Base only | Phase 2 |
| HD Wallet (unique addresses per payment) | Shared Address + Unique Amount reicht für MVP | Phase 2 |
| Stripe Billing Integration | Manuelles Tier-Management im MVP, Stripe erst bei >10 Tenants | Phase 2 |
| Public Status Page | Nice-to-have, nicht kritisch für Launch | Phase 2 |
| SDK (npm Package) | Code-Beispiele + curl reichen für MVP | Phase 2 |
| Interactive API Explorer (Swagger UI) | Docs-Seite reicht | Phase 2 |
| Team/Multi-User per Tenant | Ein API Key pro Tenant im MVP | Phase 2 |
| Audit Log | webhook_attempts reicht für MVP-Observability | Phase 2 |
| Custom Branding per Tenant | Kein White-Label im MVP | Phase 3 |
| WebSocket Real-Time Updates im Dashboard | Polling/Refresh reicht für MVP | Phase 2 |

---

## 6. Erfolgsmetriken

### Launch-Kriterien (Phase 1 Complete)

- [ ] Landing Page live mit allen 8 Sektionen
- [ ] User Dashboard: Payments List + Detail + Settings funktional
- [ ] Admin Dashboard: System Overview + Tenants + Global Payments funktional
- [ ] Mindestens 1 externer Tenant (nicht masemIT) erfolgreich onboarded
- [ ] End-to-End Flow getestet: Signup → API Key → Create Payment → USDC senden → Webhook received

### KPIs (3 Monate nach Launch)

| KPI | Ziel | Messung |
|-----|------|---------|
| Landing Page → Signup | >5% Conversion | Analytics Events |
| Signups (Tenants) | 10 Free + 3 Paid | DB Count |
| Verified Payments | 100+ | DB Count |
| Webhook Success Rate | >99% | Admin Dashboard |
| Average Confirmation Time | <30s | Admin Dashboard |
| MRR | €87+ (1 Starter + 1 Pro) | Stripe/Manual |

---

## 7. Risiken & Mitigations

| Risiko | Impact | Mitigation |
|--------|--------|------------|
| ~~Domain paywatcher.com nicht verfügbar~~ | ~~Naming-Verwirrung~~ | ✅ Gelöst: paywatcher.dev gekauft ($10.81/Jahr) |
| Zu wenig Organic Traffic | Keine Signups | Content Marketing: Blog Posts über "Stablecoin verification", Dev Community Engagement |
| Security Incident (Private Key Leak) | Funds at Risk | Phase 1: Nur kleine Beträge, Sweep Threshold niedrig, ENV-only Keys, Vault in Phase 2 |
| Base Network Issues | Service Down | Monitoring im Admin Dashboard, Kommunikation über Status Page (Phase 2) |
| Regulatorische Unsicherheit | Unklar ob Verification = reguliert | Explizit non-custodial positionieren, Legal Review bei Skalierung |

---

## 8. Timeline (Grob-Schätzung)

| Phase | Deliverable | Geschätzter Aufwand |
|-------|------------|-------------------|
| 1a | Landing Page (Design + Build) | 2–3 Tage |
| 1b | User Dashboard (alle Seiten) | 4–5 Tage |
| 1c | Admin Dashboard (alle Seiten) | 3–4 Tage |
| 1d | Auth-System + Signup Flow | 2–3 Tage |
| 1e | Testing + Polish | 2 Tage |
| **Total** | **Phase 1 Complete** | **~13–17 Tage** |

*Annahme: Fokussierte Entwicklungszeit, kein Parallelbetrieb mit UNIQA-Arbeit.*

---

## 9. Offene Fragen für Team-Diskussion

1. ~~**Pricing finalisieren**~~ → ✅ Free/Starter $29/Pro $99/Enterprise. Launch mit diesen Preisen, bei Bedarf anpassen. Killer-Argument bleibt Flat-Fee vs. %-Modell.
2. ~~**Signup Self-Service vs. Approval**~~ → ✅ Hybrid: Free Tier Self-Service (sofort loslegen), Paid Tiers nach kurzem Review/Onboarding.
3. ~~**Branding**~~ → ✅ Eigenes PayWatcher-Branding (eigene Farben, Logo, Dark Mode). "by masemIT" im Footer.
4. ~~**Docs First**~~ → ✅ Landing Page first mit Docs-Lite eingebettet (Quick Start, Endpoint Reference, Webhook Payload). Vollständige Docs auf docs.paywatcher.dev als separater Task vor dem User Dashboard. Reihenfolge: Landing Page → Docs → User Dashboard → Admin Dashboard.
5. ~~**Beta Launch**~~ → ✅ Soft Open Launch: Kein Waitlist-Gate, Free Tier Self-Service, gezielte Promotion in 2-3 Web3 Communities (ChainSights-Netzwerk). Product Hunt nach den ersten 5-10 Tenants.

---

## 10. Anhang: Wettbewerbs-Vergleich (Kurzversion)

| | PayWatcher | Coinbase Commerce | NOWPayments | Alchemy Webhooks |
|--|-----------|-------------------|-------------|-----------------|
| **Typ** | Verification API | Payment Processor | Payment Gateway | Blockchain Infra |
| **Custodial** | Nein | Ja | Ja | Nein |
| **Fee** | Flat/per-verification | 1% | 0.5% | CU-based |
| **$10k Payment** | ~$0.05 | $100 | $50 | ~$0.01 + DIY |
| **Dev Effort** | API Call + Webhook | Checkout Integration | Gateway Integration | Alles selbst bauen |
| **Base Support** | Ja (First) | Ja | Limited | Ja |
| **Shared Address** | Ja | Nein | Nein | N/A |

---

*Dieses Product Brief dient als Grundlage für die Architecture Decision und die Epic/Story-Breakdown der Frontend-Entwicklung. Nächster Schritt: Architecture Doc für Phase 1 Frontend erstellen.*
