---
stepsCompleted: [1, 2, 3, 4, 5, 6]
lastStep: 6
status: 'complete'
completedAt: '2026-02-17'
supersededBy: 'v2.0 — Managed MVP (CR-PW-001)'
inputDocuments:
  - docs/_masemIT/product-brief-paywatcher-phase1-frontend-v2-2026-02-17.md
  - docs/_masemIT/company-info.md
  - mms:_bmad-output/planning-artifacts/research/market-paywatcher-research-2026-02-14.md
date: '2026-02-17'
---

# Product Brief: PayWatcher Phase 1 — Frontend

**Project:** PayWatcher (masemIT Product)
**Phase:** 1 — Landing Page, User Dashboard, Admin Dashboard
**Version:** 2.0 (merged CR-PW-001: Managed MVP)
**Date:** 2026-02-17
**Author:** Mario Semper (PO)
**Status:** Approved
**Backend:** MMS Module (api.masem.at) — ✅ ALLE 11 Endpoints live und dokumentiert
**API Docs:** https://api.masem.at/docs/paywatcher
**Referenzdokumente:** paywatcher-architecture.md, paywatcher-epics.md, market-paywatcher-research-2026-02-14.md, cr-pw-001-managed-mvp-2026-02-16.md, requirement-mms-paywatcher-frontend-endpoints-2026-02-16-v2.md, gtm-strategy-paywatcher-2026-02-17.md

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

Das Backend (MMS Module) ist fertig — alle 11 Endpoints live:

**Tenant Dashboard Endpoints:**
- GET /v1/paywatcher/config (Config lesen)
- PATCH /v1/paywatcher/config (Config updaten)
- POST /v1/paywatcher/config/test-webhook (Test Webhook senden)
- GET /v1/paywatcher/config/api-key (API Key masked anzeigen)
- GET /v1/payments (Payment-Liste mit Filtern, Pagination, Sort)
- GET /v1/payments/:id (Payment Detail)
- GET /v1/payments/:id/webhooks (Webhook Delivery History)

**Admin Endpoints:**
- POST /v1/admin/paywatcher/tenants (Tenant anlegen)
- GET /v1/admin/paywatcher/tenants (Tenant-Liste mit Aggregations)
- PATCH /v1/admin/paywatcher/tenants/:slug (Tenant updaten/aktivieren/deaktivieren)
- GET /v1/admin/paywatcher/payments (Globale Payment-Liste mit tenant_slug Filter)
- GET /v1/admin/paywatcher/health (System Health: RPC, Redis, DB, Wallets, 24h Stats)

**Auth:** API Key via `x-api-key` Header. Scopes: payments:read, payments:write, admin:read, admin:write.

**Es fehlt alles Kundenseitige:**
1. Landing Page — damit Devs PayWatcher finden und verstehen
2. User Dashboard — damit Tenants ihre Payments sehen und Settings verwalten
3. Admin Dashboard — damit ich (Mario) den Service überwachen und managen kann

---

## 2. Zielgruppe

### Primary: Web3 Developer / Technical Founder

- Baut DApp, Marketplace, oder SaaS mit Stablecoin-Zahlungen
- Will USDC-Zahlungen verifizieren, ohne einen Payment Processor zu nutzen
- Schätzt clean APIs, gute Docs, und schnelles Onboarding
- Entscheidet basierend auf Code-Beispielen, nicht Marketing-Slides
- Pain: "Coinbase Commerce nimmt 1% und diktiert meinen Checkout-Flow" oder "Ich baue gerade meinen eigenen Payment Listener auf Alchemy Webhooks und es ist ein Albtraum"

### Secondary: Crypto-Native Small Business

- Akzeptiert Stablecoin-Zahlungen (z.B. Freelancer, Agentur, kleiner E-Commerce)
- Braucht einfache Bestätigung: "Hat mein Kunde gezahlt?"
- Weniger technisch, nutzt eher Dashboard als API direkt

---

## 3. Deliverables

### 3.1 Landing Page (paywatcher.dev)

**Zweck:** Conversion — Besucher → Request Access

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
| 1 | **Hero** | Headline + Subline + CTA ("Request Access") + Code-Snippet (3 Zeilen: create → wait → webhook) |
| 2 | **Problem/Solution** | "Today: 1% fees OR build from scratch. PayWatcher: the missing middle." |
| 3 | **How It Works** | 3 Steps visuell: Create Intent → Customer Pays → Webhook Fires |
| 4 | **Code Examples** | curl + JavaScript/TypeScript Beispiele, Webhook Payload |
| 5 | **Pricing** | Free/Starter/Pro/Enterprise Tiers (Tabelle). Alle CTAs → "Request Access" (kein Self-Service Signup). |
| 6 | **Comparison** | "$100 at Coinbase Commerce vs. $0.05 at PayWatcher for a $10k payment" |
| 7 | **Trust Signals** | Non-custodial, Base Network, USDC, Open Architecture, "Built by masemIT" |
| 8 | **Footer** | Docs, GitHub, Legal, masemIT Links |

**Request Access Flow (Managed MVP — kein Self-Service Signup):**

Statt eines Signup-Flows gibt es ein Request-Access-Formular:

| Feld | Required | Typ |
|------|----------|-----|
| Name | Ja | Text |
| Email | Ja | Email |
| Company / Project Name | Nein | Text |
| Use Case | Nein | Textarea |

**Submit:** Sendet Email via Resend an contact@masem.at. Kein MMS-Call. Kein Account wird erstellt.
**Bestätigung:** "Thanks! We'll get back to you within 24 hours with your API credentials."

**Design-Prinzipien:**
- Developer Audience = Code > Copy, Klarheit > Grafiken
- Referenz-Style: Stripe, Resend, Clerk Landing Pages
- masemIT Brand Colors: Primary #093236, Accent #009BB1
- Dark Mode bevorzugt (Web3 Convention)
- Responsive, Mobile-First
- Keine generische "Web3 is the future"-Rhetorik
- Kein Wallet-Connect (das ist ein Dev-Tool, kein Consumer-Produkt)

**Analytics Events:**

| Event | Trigger | Properties |
|-------|---------|------------|
| `lp_view` | Page loaded | `referrer`, `utm_source` |
| `lp_hero_cta` | Hero CTA clicked | `cta_text` |
| `lp_pricing_view` | Pricing section scrolled into view | — |
| `lp_pricing_cta` | Pricing tier CTA clicked | `tier` |
| `lp_code_copy` | Code example copied | `language`, `section` |
| `lp_docs_click` | Docs link clicked | — |
| `lp_request_access_start` | Request Access form opened/scrolled to | `source_section` |
| `lp_request_access_submit` | Request Access form submitted | — |

---

### 3.2 User Dashboard (app.paywatcher.dev oder /dashboard Route)

**Zweck:** Self-Service Management für Tenants

**Auth:** API-Key-basierter Login (Managed MVP). Der Tenant hat seinen Key von Mario erhalten. Er gibt ihn im Dashboard ein, das Frontend validiert gegen MMS (GET /v1/paywatcher/config), und speichert den Key in der Session. Alternativ: Magic Link via NextAuth (Frontend-seitig, NICHT im MMS Backend).

**Kein /signup.** Nur /login. /signup → Redirect auf /login.

**Seiten & Funktionen:**

#### 3.2.1 Onboarding (nach erstem Login)

**"Welcome to PayWatcher"** — einmaliger Screen nach dem ersten erfolgreichen Login:
- Zeigt: API Key (masked), Deposit Address, Webhook URL (falls konfiguriert, sonst "not configured")
- CTA: "Configure Webhook" → Settings Seite
- CTA: "View API Docs" → Docs

#### 3.2.2 Overview / Home

- Payment-Statistiken: Total Payments, Confirmed, Expired, Failed (letzte 7/30/90 Tage)
- Mini-Chart: Payments over Time (einfaches Line Chart)
- Recent Payments Liste (letzte 10)
- Quick Links: API Docs, Create Payment (Test), Settings

#### 3.2.3 Payments

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

#### 3.2.4 Settings

- **API Keys:** Anzeige (masked) via GET /v1/paywatcher/config/api-key, Scopes anzeigen
- **Webhook Configuration:** URL, Test Webhook senden, Delivery Log
- **Confirmation Override:** Default Confirmations (2–20)
- **Deposit Address:** Anzeige (read-only im MVP)
- **Account:** Email, Notifications Preferences

#### 3.2.5 API Docs (Embedded oder Link)

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
- Neuen Tenant anlegen (Admin-Formular):
  - tenant_slug (required, auto-generated from name, editable)
  - tenant_name (required)
  - contact_email (required)
  - tier (Dropdown: free, starter, pro, enterprise)
  - deposit_address (pre-filled with shared address)
  - webhook_url (optional)
  - confirmation_override (optional, 2–20)
  - **Nach Erstellung:** Modal zeigt einmalig API Key + Webhook Secret im Klartext (mit Copy-Buttons). Warnung: "This is the only time these credentials will be shown."

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
| **Framework** | Next.js 16 (App Router) | Bewährt in masemIT Stack, Turbopack stabil |
| **Styling** | Tailwind CSS v4 | Standard im Stack |
| **UI Components** | shadcn/ui | Standard im Stack, konsistent mit anderen Produkten |
| **Charts** | Recharts oder tremor | Lightweight, React-native |
| **Auth** | API-Key-basierter Login oder Magic Link via NextAuth | Managed MVP |
| **API Client** | TanStack Query v5 | Für Dashboard Polling/Refresh |
| **Deployment** | Vercel Pro | Bestehendes Setup |
| **Domain** | paywatcher.dev ✅ | Gekauft, .dev = Developer-Audience perfekt, Google Registry |
| **Analytics** | masemIT Analytics (analytics.masem.at) | Eigenes Produkt, kein Third-Party |
| **Branding** | Eigenes PayWatcher-Branding | Dark Mode, Developer-Ästhetik, "by masemIT" im Footer |

### API-Integration

Das Dashboard kommuniziert mit dem bestehenden MMS Backend (api.masem.at).
**Live API Docs:** https://api.masem.at/docs/paywatcher
**Status:** ✅ Alle 11 Endpoints implementiert und dokumentiert (Stand 2026-02-18)

**Tenant Dashboard Endpoints:**

| Funktion | Pfad | Auth | Scope |
|----------|------|------|-------|
| Config lesen | GET /v1/paywatcher/config | API Key | payments:read |
| Config updaten | PATCH /v1/paywatcher/config | API Key | payments:write |
| Test Webhook | POST /v1/paywatcher/config/test-webhook | API Key | payments:write |
| API Key anzeigen (masked) | GET /v1/paywatcher/config/api-key | API Key | payments:read |
| Payment-Liste (Filter, Pagination) | GET /v1/payments | API Key | payments:read |
| Payment Detail | GET /v1/payments/:id | API Key | payments:read |
| Webhook-Historie | GET /v1/payments/:id/webhooks | API Key | payments:read |

**Admin Endpoints:**

| Funktion | Pfad | Auth | Scope |
|----------|------|------|-------|
| Tenant anlegen | POST /v1/admin/paywatcher/tenants | API Key | admin:write |
| Tenants listen (mit Aggregations) | GET /v1/admin/paywatcher/tenants | API Key | admin:read |
| Tenant updaten | PATCH /v1/admin/paywatcher/tenants/:slug | API Key | admin:write |
| Globale Payments (alle Tenants) | GET /v1/admin/paywatcher/payments | API Key | admin:read |
| System Health | GET /v1/admin/paywatcher/health | API Key | admin:read |

**Hinweise:**
- Payment-Responses nutzen camelCase (exactAmount, depositAddress, txHash, expiresAt)
- Admin Payments enthalten zusätzlich tenantSlug pro Payment
- Health Endpoint returned immer 200 — Subsystem-Status in den Feldern (Graceful Degradation)
- Landing Page: Kein API-Zugriff. Request-Access-Formular sendet Email via Resend, kein MMS-Call.

### Architektur-Entscheidungen (✅ geklärt)

1. ✅ **Ein Next.js Projekt** mit Route Groups: (landing), (dashboard), (admin)
2. ✅ **Auth-Strategie:** API-Key-basierter Login für Dashboard. Kein Signup. Alternativ: Magic Link via NextAuth (Frontend-only).
3. **Admin Auth:** Separate Admin Auth (Admin API Key oder /admin Route mit Role-Check) — zu klären
4. ✅ **Domain:** paywatcher.dev. Subdomains optional (docs.paywatcher.dev, app.paywatcher.dev oder integriert)
5. **Docs-Lösung:** Eingebettet im Dashboard, separate Docs-App (Mintlify/Fumadocs), oder einfache Markdown-Seiten — zu klären
6. ✅ **Signup-Flow:** Managed MVP: Request Access Formular, Mario onboardet manuell.

---

## 5. Abgrenzung (Nicht in Phase 1)

| Feature | Warum nicht in Phase 1 | Phase |
|---------|----------------------|-------|
| Self-Service Signup | Managed MVP — Mario onboardet manuell | Phase 2 |
| API Key Regeneration | Kein MMS-Endpoint, manuell per Admin | Phase 2 |
| Multi-Chain Support | MVP = USDC auf Base only | Phase 2 |
| HD Wallet (unique addresses per payment) | Shared Address + Unique Amount reicht für MVP | Phase 2 |
| Stripe Billing Integration | Manuelles Tier-Management im MVP, Stripe erst bei >10 Tenants | Phase 2 |
| Public Status Page | Nice-to-have, nicht kritisch für Launch | Phase 2 |
| SDK (npm Package) | Code-Beispiele + curl reichen für MVP | Phase 2 |
| Interactive API Explorer (Swagger UI) | Docs-Seite reicht | Phase 2 |
| Team/Multi-User per Tenant | Ein API Key pro Tenant im MVP | Phase 2 |
| Audit Log | webhook_attempts reicht für MVP-Observability | Phase 2 |
| Invoicing / PDF Download | Manuelle Rechnungsstellung via FreeFinance + Email | Phase 2 |
| Custom Branding per Tenant | Kein White-Label im MVP | Phase 3 |
| WebSocket Real-Time Updates im Dashboard | Polling/Refresh reicht für MVP | Phase 2 |

---

## 6. Erfolgsmetriken

### Launch-Kriterien (Phase 1 Complete)

- [ ] Landing Page live mit allen 8 Sektionen + Request Access Formular
- [ ] User Dashboard: Login + Onboarding + Payments List + Detail + Settings funktional
- [ ] Admin Dashboard: System Overview + Tenants (CRUD) + Global Payments funktional
- [ ] Mindestens 1 externer Tenant (nicht masemIT) erfolgreich onboarded
- [ ] End-to-End Flow getestet: Request Access → Mario onboarded → Tenant Login → Create Payment → USDC senden → Webhook received

### KPIs (3 Monate nach Launch)

| KPI | Ziel | Messung |
|-----|------|---------|
| Landing Page → Request Access | >3% Conversion | Analytics Events |
| Request → Onboarded | >50% | Manual Tracking |
| Active Tenants | 5 Free + 2 Paid | DB Count |
| Verified Payments | 100+ | DB Count |
| Webhook Success Rate | >99% | Admin Dashboard |
| Average Confirmation Time | <30s | Admin Dashboard |
| MRR | €87+ (1 Starter + 1 Pro) | Stripe/Manual |

---

## 7. Risiken & Mitigations

| Risiko | Impact | Mitigation |
|--------|--------|------------|
| ~~Domain paywatcher.com nicht verfügbar~~ | ~~Naming-Verwirrung~~ | ✅ Gelöst: paywatcher.dev gekauft ($10.81/Jahr) |
| Zu wenig Organic Traffic | Keine Request Access Anfragen | Content Marketing: Blog Posts über "Stablecoin verification", Dev Community Engagement |
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
| 1d | Auth-System + Login Flow | 2–3 Tage |
| 1e | Testing + Polish | 2 Tage |
| **Total** | **Phase 1 Complete** | **~13–17 Tage** |

*Annahme: Fokussierte Entwicklungszeit, kein Parallelbetrieb mit UNIQA-Arbeit.*

---

## 9. Offene Fragen für Team-Diskussion

1. ~~**Pricing finalisieren**~~ → ✅ Free/Starter $29/Pro $99/Enterprise.
2. ~~**Signup Self-Service vs. Approval**~~ → ✅ Managed MVP: Kein Self-Service. Mario legt Tenants manuell an. Self-Service → Phase 2.
3. ~~**Branding**~~ → ✅ Eigenes PayWatcher-Branding (eigene Farben, Logo, Dark Mode). "by masemIT" im Footer.
4. **Docs First:** Sollen wir API Docs VOR der Landing Page bauen, weil Devs die Docs zuerst lesen?
5. ~~**Beta Launch**~~ → ✅ Soft Open Launch via Request Access. Product Hunt nach den ersten 5-10 Tenants.
6. ~~**Auth-Strategie Dashboard**~~ → ✅ API-Key-basierter Login. Tenant gibt seinen Key ein, Frontend validiert gegen GET /v1/paywatcher/config.

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

*Dieses Product Brief (v2.0) enthält alle Änderungen aus CR-PW-001 (Managed MVP). Es dient als einziges Referenzdokument für die Frontend-Entwicklung. Der separate CR ist damit historisch und muss nicht mehr separat gelesen werden.*
