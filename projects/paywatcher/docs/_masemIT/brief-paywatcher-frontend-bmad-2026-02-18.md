# PayWatcher Frontend — BMAD Team Brief

**Date:** 2026-02-18
**From:** Mario Semper (PO)
**To:** BMAD Team
**Projekt:** PayWatcher Frontend (paywatcher.dev)
**Status:** Backend 100% complete — Frontend kann starten

---

## 1. Was ist PayWatcher?

Verification-API für Stablecoin-Zahlungen. Kein Payment Processor — wir bewegen kein Geld, halten keine Funds. Wir sagen nur: "Deine USDC sind angekommen." Developer-Audience, Dark Mode, API-first.

**Positionierung:** "Payment verification without the payment processor."
**Killer-Argument:** $10.000 USDC-Zahlung: Coinbase Commerce → $100 Fee. PayWatcher → $0.05 Flat.

---

## 2. Backend: Fertig

**Alle 11 Endpoints live und dokumentiert.**

### Live API Docs

- **PayWatcher API:** https://api.masem.at/docs/paywatcher
- **Parties API:** https://api.masem.at/docs/parties

### Tenant Dashboard Endpoints (payments:read / payments:write)

| Endpoint | Funktion |
|----------|----------|
| GET /v1/paywatcher/config | Tenant-Config lesen |
| PATCH /v1/paywatcher/config | Webhook-URL, Confirmations updaten |
| POST /v1/paywatcher/config/test-webhook | Test-Webhook senden |
| GET /v1/paywatcher/config/api-key | API Key masked anzeigen |
| GET /v1/payments | Payment-Liste (Filter, Pagination, Sort) |
| GET /v1/payments/:id | Payment Detail |
| GET /v1/payments/:id/webhooks | Webhook Delivery History |

### Admin Endpoints (admin:read / admin:write)

| Endpoint | Funktion |
|----------|----------|
| POST /v1/admin/paywatcher/tenants | Tenant anlegen (gibt einmalig API Key + Webhook Secret zurück) |
| GET /v1/admin/paywatcher/tenants | Alle Tenants mit Aggregations |
| PATCH /v1/admin/paywatcher/tenants/:slug | Tenant updaten/aktivieren/deaktivieren |
| GET /v1/admin/paywatcher/payments | Globale Payments (alle Tenants, mit tenant_slug Filter) |
| GET /v1/admin/paywatcher/health | System Health (RPC, Redis, DB, Wallets, 24h Stats) |

### Tenant Assignment Endpoints (NEU — gerade implementiert)

| Endpoint | Funktion |
|----------|----------|
| POST /v1/parties/:id/tenants | Party einem Tenant zuordnen |
| GET /v1/parties/:id/tenants | Alle Tenant-Zuordnungen einer Party |
| POST /v1/auth/login | Login-Lookup: Email + Module → Tenant-Zuordnung |
| GET /v1/tenants/:module/:slug/members | Alle Parties eines Tenants |
| DELETE /v1/parties/:id/tenants/:assignment_id | Zuordnung entfernen |

### Wichtige API-Details

- Auth: `x-api-key` Header für alle Endpoints
- Payment-Responses nutzen **camelCase** (exactAmount, depositAddress, txHash, expiresAt)
- Admin Payments enthalten zusätzlich `tenantSlug` pro Payment
- Health Endpoint returned **immer 200** — Subsystem-Status in den Feldern (Graceful Degradation)
- Test-Webhook returned **immer 200** — `success` Feld zeigt ob Target 2xx geantwortet hat

---

## 3. Auth-Flow (WICHTIG!)

Das Frontend hat sein **eigenes Auth-System** (NextAuth + Magic Link via Resend). MMS kennt keine User-Sessions.

```
┌─────────────────────────────────────────────────┐
│ 1. User gibt Email ein auf paywatcher.dev/login │
│ 2. NextAuth schickt Magic Link per Resend       │
│ 3. User klickt Link → NextAuth Session aktiv    │
└──────────────┬──────────────────────────────────┘
               │
               ▼
┌─────────────────────────────────────────────────┐
│ 4. Frontend ruft POST /v1/auth/login auf:       │
│    { "email": "user@acme.com",                  │
│      "module": "paywatcher" }                   │
│                                                 │
│ 5. MMS gibt zurück:                             │
│    { party_id, display_name,                    │
│      tenants: [{ tenant_slug, role }] }         │
└──────────────┬──────────────────────────────────┘
               │
               ▼
┌─────────────────────────────────────────────────┐
│ 6. Frontend speichert in Session:               │
│    - party_id, tenant_slug, role                │
│                                                 │
│ 7. Alle MMS-Calls gehen serverseitig            │
│    mit dem Tenant API Key                       │
└─────────────────────────────────────────────────┘
```

### Offene Architektur-Entscheidung für Winston

**Wie kommt der Tenant API Key ins Frontend?**

Der Key darf NICHT im Browser landen. Optionen:

- **Option A:** Verschlüsselt in einer Frontend-DB (NextAuth braucht ohnehin eine DB für Sessions). Mapping: tenant_slug → encrypted API Key. Mario trägt beim Onboarding ein.
- **Option B:** ENV Variable — skaliert nicht bei mehreren Tenants.
- **Option C:** Login-Endpoint gibt Key zurück — Sicherheitsbedenken.

**Empfehlung: Option A.** Aber Winston entscheidet — er kennt den Stack am besten.

### Admin-Zugang

Mario loggt sich genauso per Magic Link ein. Der Admin API Key liegt als ENV Variable im Frontend-Server. Wenn die Email zu einem Admin-User gehört, schaltet das Frontend die Admin-Seiten frei.

---

## 4. Managed MVP — Kein Self-Service!

**Kein Signup.** Statt "Sign Up" gibt es "Request Access":

- Landing Page: CTA → "Request Access" Formular (Name, Email, Company, Use Case)
- Formular sendet Email via Resend an contact@masem.at
- Mario onboardet manuell: Tenant in MMS anlegen → Party anlegen → Zuordnung erstellen → Credentials sicher übermitteln
- Tenant kann sich danach per Magic Link einloggen

### Onboarding-Flow (Mario, 3 API-Calls pro Tenant)

```
1. POST /v1/admin/paywatcher/tenants → Tenant + API Key
2. POST /v1/parties → Party mit contact_email (Find-or-Create)
3. POST /v1/parties/:party_id/tenants → Zuordnung: paywatcher + tenant_slug
4. API Key im Frontend hinterlegen (Option A: encrypted in DB)
5. Tenant kann sich ab jetzt per Magic Link einloggen
```

---

## 5. Deliverables

### 5.1 Landing Page (paywatcher.dev)

- 8 Sektionen: Hero, Problem/Solution, How It Works, Code Examples, Pricing, Comparison, Trust Signals, Footer
- Request Access Formular (statt Signup)
- Dark Mode, Developer-Ästhetik
- Kein API-Zugriff (rein statisch + Resend für Formular)

### 5.2 User Dashboard

- Login (Magic Link, kein Signup)
- Onboarding-Screen (nach erstem Login)
- Overview (Stats, Mini-Chart, Recent Payments)
- Payments (Tabelle mit Filtern, Detail-View mit Status-Timeline + Webhook History)
- Settings (API Key masked, Webhook Config, Confirmation Override)

### 5.3 Admin Dashboard

- System Overview (Health-Daten vom Health-Endpoint)
- Tenants CRUD (Formular mit einmaligem Credential-Modal)
- Global Payments (alle Tenants, mit Tenant-Filter)
- Webhooks (Delivery Health über alle Tenants)

---

## 6. Tech Stack

| Komponente | Technologie |
|-----------|------------|
| Framework | Next.js 15 (App Router) |
| Styling | Tailwind CSS |
| UI Components | shadcn/ui |
| Charts | Recharts oder tremor |
| Auth | NextAuth.js + Magic Link (Resend) |
| API Client | Fetch + SWR oder TanStack Query |
| Deployment | Vercel Pro |
| Domain | paywatcher.dev |
| Analytics | masemIT Analytics (analytics.masem.at) |
| Branding | Eigenes PayWatcher-Branding, "by masemIT" im Footer |

---

## 7. Startpaket / Referenzdokumente

| Dokument | Inhalt |
|----------|--------|
| product-brief-paywatcher-phase1-frontend-v2-2026-02-17.md | Das Gesamtbild — alle Details zu Landing Page, Dashboard, Admin |
| MMS_PayWatcher_API.md (oder api.masem.at/docs/paywatcher) | Alle 11 Endpoints mit Request/Response Beispielen |
| api.masem.at/docs/parties | Parties API — User/Party Management |
| requirement-mms-tenant-assignments-2026-02-18.md | Auth-Flow, Tenant Assignments, Login-Endpoint |
| paywatcher-brand-assets.tar.gz | Logo SVGs, TSX Components, Design Tokens |
| paywatcher-developer-handbook-2026-02-15.md | Web3-Kontext (Teil 0: Blockchain-Grundlagen) |
| gtm-strategy-paywatcher-2026-02-17.md | Go-to-Market Strategie (für Content auf Landing Page) |

---

## 8. Empfohlene Reihenfolge

| Phase | Was | Abhängigkeit |
|-------|-----|-------------|
| 1 | **Landing Page** | Keine — kann sofort starten |
| 2 | **Dashboard UI** (Payments, Settings, Overview) | Gegen API Docs bauen, Endpoints alle live |
| 3 | **Auth** (NextAuth + Magic Link + /v1/auth/login) | Tenant Assignment Endpoints live (✅ done) |
| 4 | **Admin Dashboard** | Alle Admin Endpoints live (✅ done) |

**Landing Page hat keine API-Abhängigkeit und kann sofort gebaut werden.**

---

## 9. Was sich seit der letzten Version geändert hat

Falls ihr das alte Product Brief (v1) gesehen habt — hier die wichtigsten Änderungen (CR-PW-001):

1. **Kein Self-Service Signup** — Managed MVP, "Request Access" statt "Sign Up"
2. **Auth via NextAuth Magic Link** — nicht via MMS Backend
3. **API Key Pfad:** `/v1/paywatcher/config/api-key` (nicht `/v1/paywatcher/api-key`)
4. **Tenant Assignments** — neue Tabelle und Endpoints für Party ↔ Tenant Verknüpfung
5. **Login-Endpoint:** POST /v1/auth/login für Email → Tenant Lookup
6. **Angepasste KPIs:** Request Access >3%, 5 Free + 2 Paid Tenants, €87+ MRR

---

## 10. Fragen?

Alles was unklar ist: Fragen an Sempre.

Let's build this. 🚀
