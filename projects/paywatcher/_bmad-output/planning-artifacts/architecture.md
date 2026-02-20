---
stepsCompleted: [1, 2, 3, 4, 5, 6, 7, 8]
lastStep: 8
status: 'complete'
completedAt: '2026-02-15'
inputDocuments:
  - _bmad-output/planning-artifacts/prd.md
  - _bmad-output/planning-artifacts/product-brief-paywatcher-2026-02-15.md
  - _bmad-output/planning-artifacts/ux-design-specification.md
  - docs/_masemIT/company-info.md
  - docs/_masemIT/product-brief-paywatcher-phase1-frontend-2026-02-15.md
  - mms:_bmad-output/planning-artifacts/paywatcher-architecture.md
  - mms:_bmad-output/planning-artifacts/paywatcher-epics.md
  - mms:_bmad-output/planning-artifacts/architecture.md
  - mms:_bmad-output/planning-artifacts/epics.md
  - mms:_bmad-output/planning-artifacts/implementation-readiness-report-2026-02-02.md
  - mms:_bmad-output/planning-artifacts/research/market-paywatcher-research-2026-02-14.md
workflowType: 'architecture'
project_name: 'paywatcher'
user_name: 'Sempre'
date: '2026-02-15'
---

# Architecture Decision Document

_This document builds collaboratively through step-by-step discovery. Sections are appended as we work through each architectural decision together._

## Project Context Analysis

### Requirements Overview

**Functional Requirements (35 FRs, aktualisiert für Managed MVP v2):**

Aus dem PRD extrahiert, in 9 Kategorien:

- **Landing Page & Marketing (FR1–FR7, FR36):** 8 Sektionen (Hero, Problem/Solution, How It Works, Code Examples, Pricing, Comparison, Trust Signals, Footer), interaktive Code-Beispiele, Pricing-Tiers, Kostenvergleich, Request Access Formular (FR36)
- **Auth & Onboarding (FR9, FR10, FR37):** Magic Link Login (Resend) + Parties API Tenant-Resolution (Managed MVP), Logout. Kein Self-Service Signup (~~FR8~~). Kein automatischer API Key (~~FR11~~). Frontend-DB (Neon Postgres) für Tenant API Key Storage.
- **API Key Management (FR12, FR14):** Key anzeigen (masked via GET /v1/paywatcher/config/api-key), Scopes einsehen. Kein Regenerieren (~~FR13~~ — kein MMS-Endpoint).
- **Payment Monitoring (FR15–FR19, FR38):** Payments-Liste mit Status-Filter (6 States), Zeitraum-Filter, Sortierung (nur created_at), Pagination, Webhook History pro Payment (FR38)
- **Webhook Management (FR20–FR22):** Webhook-URL konfigurieren, Test-Webhook senden, Delivery Log
- **Tenant Settings (FR23–FR25):** Confirmation Override (2–20), Deposit Address (read-only), Account-Settings
- **Documentation (FR28–FR31):** Quick Start Guide, Endpoint Reference, Webhook Event Reference, Code-Beispiele (curl, JS, Python)
- **Analytics & SEO (FR32–FR35):** Landing Page Event Tracking (masemIT Analytics), SEO-Optimierung (Meta Tags, OG, Sitemap)

**Entfallene FRs (Managed MVP):** FR8 (Self-Service Signup), FR11 (Auto API Key), FR13 (Key Regenerate), FR26/FR27 (Invoicing) — alle verschoben auf Phase 2 oder entfallen mangels MMS-Endpoint.

**Non-Functional Requirements (17 NFRs):**

- **Performance (NFR1, NFR2, NFR4):** Landing Page FCP < 1.5s (SSG + CDN), Dashboard API < 2s. ~~NFR3 (Signup < 5s)~~ entfällt (kein Self-Service Signup).
- **Security (NFR5–NFR10):** HTTPS, API Keys gehasht, Auth-Tokens mit begrenzter Lebensdauer, Dashboard-Routen protected, CSRF-Schutz, GDPR-konform
- **Integration (NFR11–NFR14):** Ausschließlich REST API mit MMS Backend, keine direkte DB-Verbindung, asynchrone Analytics-Events, Block Explorer Links
- **Deployment (NFR15–NFR17):** Vercel Pro mit CI/CD, konfigurierbare Environment Variables, keine Secrets im Frontend-Bundle

**Scale & Complexity:**

- Primary domain: Frontend Web App (Next.js 15 App Router)
- Complexity level: Medium — drei Bereiche mit unterschiedlichen Rendering-Strategien, aber schmale API-Fläche (11 MMS-Endpoints, davon 7 Tenant + 4 Admin)
- Estimated architectural components: Landing Page (SSG), User Dashboard Shell (Layout, Sidebar, Auth), 4–5 Dashboard Pages, Auth System, API Client Layer, Theme System, Analytics Integration

### Technical Constraints & Dependencies

- **MMS Backend (api.masem.at):** Einzige Datenquelle. REST API mit API Key Auth (x-api-key Header). Scopes: payments:read, payments:write, admin:read, admin:write. 11 Endpoints live (7 Tenant, 5 Admin). Response-Format: `{ data: {...} }` / `{ data: [...], meta: {...} }`. API Docs: https://api.masem.at/docs/paywatcher
- **Vercel Pro:** Bestehendes Deployment-Setup. SSG für Landing Page, Serverless Functions für Auth-Flows. Automatisches CI/CD via Git Push.
- **masemIT Analytics (analytics.masem.at):** Eigenes Analytics-System, kein Third-Party. Asynchrone Event-Sends (kein UI-Blocking).
- **Domain:** paywatcher.dev (gekauft). Routing-Strategie (Subdomains vs. Route Groups) ist offene Architekturentscheidung.
- **Existing masemIT Tech Stack:** Next.js 15, Tailwind CSS, shadcn/ui — bewährt in ChainSights, tellingCube, hoki.help. Konsistenz gewünscht.
- **UX-Spezifikation:** Sehr detailliert (Farben, Typografie, Spacing, Komponenten, Flows). Definiert Design Tokens, Komponentenhierarchie, Status-Farb-Mapping, Responsive-Strategie. Architektur muss diese Spec 1:1 unterstützen.
- **Backend API-Vertrag:** Payment-States (6-stufige State Machine), Webhook-Payload-Struktur, BigInt-Amounts (6 Dezimalen), snake_case in API-Responses — alles im MMS Architecture Doc definiert.

### Cross-Cutting Concerns Identified

1. **Auth & Session Management:** Dashboard-Login via Magic Link (Managed MVP). User gibt Email ein, erhält Magic Link via Resend, klickt Link → NextAuth Session aktiv. Post-Login: Frontend ruft POST /v1/auth/login (email, module:"paywatcher") auf → erhält party_id, display_name, tenants[{tenant_slug, role}]. Bei Multi-Tenant: User wählt Tenant. Frontend-DB (Neon Postgres) speichert verschlüsseltes Mapping tenant_slug → API Key. Route Handlers holen den API Key aus der DB für MMS-Calls.
2. **API Client Pattern:** Alle Dashboard-Daten kommen vom MMS Backend. Braucht einheitliches Fetch-Pattern mit Auth-Header-Injection, Error-Handling, und Refresh-Logik (SWR / TanStack Query). Server-Side vs. Client-Side Fetching muss klar getrennt sein.
3. **Dark/Light Mode Theming:** Dark Mode Default (Web3 Convention). CSS Custom Properties für Theme-Tokens. shadcn/ui Dark-Mode-ready. Toggle muss persistent sein (localStorage oder Cookie).
4. **Analytics Event Tracking:** Einheitliches Event-Dispatch-Pattern für Landing Page und Dashboard. Asynchron, kein UI-Blocking. masemIT Analytics SDK/Integration.
5. **Error Handling & User Feedback:** Toast-System (rechts unten), Inline-Validation, Skeleton Loading. Muss konsistent über alle Dashboard-Seiten sein. Spezielle Fehler-Visualisierung für Payment-States (proportionale Schwere).
6. **Responsive Strategy:** Mobile-First für Landing Page (alle Breakpoints), Desktop-First für Dashboard (funktioniert auf Mobile, aber nicht optimiert). Admin Dashboard Desktop-Only. Sidebar als Sheet auf Mobile.
7. **Route Protection & Middleware:** Public Routes (Landing Page), Protected Routes (Dashboard), Admin Routes (Phase 1b). Next.js Middleware für Auth-Check.

## Starter Template Evaluation

### Primary Technology Domain

Frontend Web App (Next.js App Router) basierend auf Projekt-Requirements und bestehendem masemIT Tech Stack.

### Technical Preferences (aus Projektkontext)

Alle Preferences sind durch den bestehenden masemIT Stack und die Planungsdokumente vordefiniert:

- **Framework:** Next.js (App Router) — bewährt in ChainSights, tellingCube, hoki.help
- **Styling:** Tailwind CSS — Standard im Stack
- **UI Components:** shadcn/ui — konsistent mit anderen masemIT Produkten
- **Deployment:** Vercel Pro — bestehendes Setup
- **Language:** TypeScript (strict)
- **Fonts:** Geist Sans + Geist Mono (via next/font)

### Starter Options Considered

| Option | Assessment |
|--------|-----------|
| **`create-next-app@latest`** | Offizielle Next.js CLI. Liefert Next.js 16 + Tailwind v4 + TypeScript + App Router + Turbopack out of the box. **Empfohlen.** |
| T3 Stack (`create-t3-app`) | Bringt tRPC, Prisma, NextAuth mit. Overkill — PayWatcher nutzt nur minimale DB (Neon Postgres für Tenant Keys), braucht kein tRPC. |
| Custom Boilerplate | Kein bestehendes masemIT Template. Unnötiger Aufwand bei verfügbarer offizieller CLI. |

### Selected Starter: `create-next-app@latest`

**Rationale:**
- Offizieller Next.js Starter, bestens maintained durch Vercel
- Liefert Next.js 16 mit Turbopack stabil (Build + Dev)
- Tailwind v4 wird automatisch eingerichtet (CSS-first Config)
- Geist Fonts bereits integriert (zero config)
- TypeScript strict mode voreingestellt
- App Router als Default
- Minimaler Overhead — kein unnötiger Ballast

**Note:** PRD und Product Brief spezifizieren "Next.js 15", aber seit der Planungsphase ist Next.js 16 als stabile Version erschienen. Da PayWatcher ein Greenfield-Projekt ist, wird Next.js 16 verwendet (Turbopack stabil, React Compiler stabil, bessere Performance).

**Initialization Command:**

```bash
npx create-next-app@latest paywatcher --typescript --tailwind --eslint --app --turbopack
```

### Additional Dependencies (Post-Init)

| Package | Zweck | Begründung |
|---------|-------|-------------|
| shadcn/ui | UI Components | `npx shadcn@latest init` — Buttons, Tables, Badges, Cards, Dialogs, Forms |
| @tanstack/react-query | Data Fetching + Caching | Dashboard-Daten: Polling, Pagination, Mutations, DevTools |
| next-auth@beta (Auth.js v5) | Auth System | Magic Link Login (Resend Provider), Session Management (JWT), Route Protection |
| @auth/drizzle-adapter | Auth DB Adapter | NextAuth Account/Verification Token Storage (Magic Link benötigt DB für Token) |
| drizzle-orm + drizzle-kit | ORM | Lightweight ORM für tenant_keys Tabelle + NextAuth Tables. Type-safe, zero-overhead. |
| @neondatabase/serverless | DB Driver | Neon Postgres Serverless Driver (HTTP-basiert, kein Connection Pooling nötig auf Vercel) |
| shiki | Syntax Highlighting | Code-Beispiele auf Landing Page + Onboarding |
| lucide-react | Icons | Standard mit shadcn/ui, konsistentes Icon-Set |
| recharts | Charts | Dashboard-Statistiken (Payment-Trends, Overview) |

### Architectural Decisions Provided by Starter

**Language & Runtime:**
- TypeScript mit strict mode (Next.js 16 Default)
- React 19 mit stabilem React Compiler (auto-memoization)
- Node.js Runtime auf Vercel

**Styling Solution:**
- Tailwind CSS v4 mit CSS-first Configuration (@theme Directive statt tailwind.config.js)
- Automatische Content Detection (keine content-Pfade nötig)
- CSS Custom Properties für Theme-Tokens (Dark/Light Mode)

**Build Tooling:**
- Turbopack (stabil in Next.js 16) für Dev + Build
- 5x schnellere Full Builds, 100x schnellere Incremental Builds
- Vercel-native Build Pipeline

**Code Organization:**
- App Router mit `app/` Directory
- Route Groups für Bereichstrennung: `(landing)`, `(dashboard)`, `(admin)`
- Server Components als Default, Client Components explizit mit `'use client'`
- `next/font/local` für Geist Sans + Geist Mono

**Development Experience:**
- `npm run dev` mit Turbopack Hot Reloading
- TypeScript Type Checking im Build
- ESLint für Code Quality

**Note:** Projektinitialisierung mit diesem Command sollte die erste Implementation Story sein.

## Core Architectural Decisions

### Decision Priority Analysis

**Critical Decisions (Block Implementation):**
- Auth-Architektur: Magic Link Login (Resend) + Auth.js v5 JWT Strategy + Parties API + Frontend-DB für API Key Storage (Managed MVP)
- App-Struktur: Route Groups in einem Next.js Projekt
- API-Kommunikation: Server-Side Proxy via Route Handlers
- Data Transformation: camelCase im Frontend, Transformation im Proxy-Layer

**Important Decisions (Shape Architecture):**
- Theme Toggle: next-themes
- Client State: React Context (Theme + Auth Session)
- Docs-Strategie: Docs-Lite eingebettet mit MDX

**Deferred Decisions (Phase 2+):**
- Separate Docs-App (Mintlify/Fumadocs)
- Subdomain-Routing (app.paywatcher.dev)
- WebSocket Real-Time Updates
- Stripe Billing Integration

### Auth & Data Architecture (Managed MVP v3 — Magic Link + Parties)

| Decision | Choice | Rationale |
|----------|--------|-----------|
| User Identity | **MMS Parties API** | Users sind "Parties" in MMS. Party ↔ Tenant Zuordnung via POST /v1/parties/:id/tenants. Login-Lookup via POST /v1/auth/login (email + module). |
| Login Method | **NextAuth Magic Link (Resend)** | User gibt Email ein, erhält Magic Link per Resend, klickt → Session aktiv. Kein API Key im Browser, kein Passwort. Bewährtes Pattern für Developer-Audience. |
| Session Storage | **Auth.js v5 JWT Strategy (Cookie)** | JWT enthält party_id, display_name, tenant_slug, role. Kein API Key im JWT — Key kommt aus Frontend-DB. |
| API Key Storage | **Frontend-DB (Neon Postgres, verschlüsselt)** | Tabelle `tenant_keys`: tenant_slug → encrypted API Key. Mario trägt Key beim Onboarding ein. Route Handlers holen Key aus DB für MMS-Calls. |
| Frontend-eigene DB | **Minimal — nur für Auth-Mapping** | Neon Postgres für tenant_keys Tabelle + NextAuth Session/Account Tables (optional, JWT reicht für Sessions). Keine Business-Daten — alle kommen weiterhin aus MMS. |
| Multi-Tenant | **Tenant-Auswahl beim Login** | POST /v1/auth/login gibt tenants[] zurück. Bei length > 1: User wählt Tenant. Tenant-Switch im Dashboard-Header möglich. |

**Login-Flow (Managed MVP — Magic Link + Parties):**
```
1. User gibt Email ein auf paywatcher.dev/login
2. NextAuth schickt Magic Link per Resend
3. User klickt Link → NextAuth Session aktiv (party_id noch unbekannt)
4. Frontend ruft POST /v1/auth/login auf:
   { "email": "user@acme.com", "module": "paywatcher" }
5. MMS gibt zurück:
   { party_id, display_name, tenants: [{ tenant_slug, role }] }
6. Bei tenants.length === 0 → "No account found. Request access?"
7. Bei tenants.length === 1 → Auto-Select, weiter zu Dashboard
8. Bei tenants.length > 1 → Tenant-Auswahl-UI
9. Frontend speichert in JWT Session:
   party_id, display_name, tenant_slug, role
10. Route Handlers: API Key aus tenant_keys DB-Tabelle (verschlüsselt)
11. Alle MMS-Calls serverseitig mit dem Tenant API Key
```

**Admin-Zugang:**
- Mario loggt sich genauso per Magic Link ein
- Admin API Key liegt als ENV Variable (MMS_ADMIN_API_KEY) im Frontend-Server
- Wenn die Email zu einem Admin-User gehört (z.B. mario@masem.at), schaltet das Frontend Admin-Seiten frei
- Admin MMS-Calls nutzen den Admin API Key aus ENV, nicht die tenant_keys DB

**Onboarding-Flow (Mario, pro Tenant):**
```
1. POST /v1/admin/paywatcher/tenants → Tenant + API Key + Webhook Secret
2. POST /v1/parties → Party mit contact_email (Find-or-Create)
3. POST /v1/parties/:party_id/tenants → Zuordnung: paywatcher + tenant_slug
4. API Key verschlüsselt in Frontend-DB hinterlegen (tenant_keys Tabelle)
5. Tenant kann sich ab jetzt per Magic Link einloggen
```

**Request Access (Landing Page):**
- Besucher füllt Request-Access-Formular aus (Name, Email, Company, Use Case)
- Email wird via Resend an contact@masem.at gesendet
- Kein MMS-Call. Kein Account. Mario onboardet manuell.

**MMS-Endpoints für Auth (alle live):**

| Endpoint | Funktion |
|----------|----------|
| POST /v1/auth/login | Login-Lookup: Email + Module → Party + Tenants |
| POST /v1/parties/:id/tenants | Party einem Tenant zuordnen |
| GET /v1/parties/:id/tenants | Alle Tenant-Zuordnungen einer Party |
| GET /v1/tenants/:module/:slug/members | Alle Parties eines Tenants |
| DELETE /v1/parties/:id/tenants/:id | Zuordnung entfernen |

### App Structure & Routing

| Decision | Choice | Rationale |
|----------|--------|-----------|
| App-Struktur | **Ein Next.js Projekt mit Route Groups** | Ein Repo, ein Deployment, geteilte Components (Theme, UI). Subdomains später per Middleware möglich ohne Umbau. |
| Landing Page | `app/(landing)/` | SSG, public, SEO-optimiert |
| User Dashboard | `app/(dashboard)/` | CSR, Auth-Check im Layout, Sidebar-Navigation |
| Admin Dashboard | `app/(admin)/` | CSR, Admin-Auth-Check, Phase 1b |
| API Proxy | `app/api/` | Route Handlers als Proxy zu MMS Backend |

**Route-Struktur:**
```
app/
├── (landing)/
│   ├── page.tsx              # Hero, Problem/Solution, How It Works, Pricing, etc.
│   ├── login/                # Magic Link Login (Email eingeben → Link klicken)
│   ├── signup/               # Redirect → /login
│   └── docs/                 # Docs-Lite (MDX)
├── (dashboard)/
│   ├── layout.tsx            # Sidebar, Auth-Check, Tenant-Resolution
│   ├── page.tsx              # Overview (Onboarding oder Stats)
│   ├── payments/             # Payments List + Detail
│   └── settings/             # API Keys, Webhook, Config
├── (admin)/
│   ├── layout.tsx            # Admin Auth-Check (isAdmin via Email)
│   ├── page.tsx              # System Overview (Health Endpoint)
│   ├── tenants/              # Tenant CRUD + Credential-Modal
│   ├── payments/             # Global Payments (alle Tenants)
│   └── webhooks/             # Delivery Health über alle Tenants
└── api/
    ├── auth/                 # Auth.js Endpoints (Magic Link Provider)
    ├── auth/resolve/         # POST: /v1/auth/login Call → Party + Tenants Resolution
    ├── payments/             # Proxy zu MMS /v1/payments + /v1/payments/:id/webhooks
    ├── config/               # Proxy zu MMS /v1/paywatcher/config (GET, PATCH)
    ├── config/api-key/       # Proxy zu MMS /v1/paywatcher/config/api-key (GET)
    ├── config/test-webhook/  # Proxy zu MMS /v1/paywatcher/config/test-webhook (POST)
    ├── admin/                # Admin Proxy Routes (MMS_ADMIN_API_KEY)
    └── request-access/       # Email via Resend an contact@masem.at
```

### API Communication Pattern

| Decision | Choice | Rationale |
|----------|--------|-----------|
| Communication Pattern | **Server-Side Proxy (Route Handlers)** | MMS API Key ist Secret und gehört nicht in den Browser. Alle MMS-Calls laufen über Next.js Route Handlers. |
| API Key Storage | **Frontend-DB (Neon Postgres, verschlüsselt)** | tenant_slug → encrypted API Key. Mario trägt Key beim Onboarding ein. Route Handlers holen Key aus DB, entschlüsseln, und injizieren als x-api-key Header. |
| Admin API Key | **ENV Variable (MMS_ADMIN_API_KEY)** | Admin-Calls nutzen separaten Key aus Environment — nicht die tenant_keys DB. |
| Client Data Fetching | **TanStack Query → Route Handlers** | Browser ruft /api/payments, Route Handler holt API Key aus DB (via tenant_slug aus JWT Session) und ruft api.masem.at/v1/payments. |

**Data Flow (Tenant):**
```
Browser (TanStack Query)
    → /api/payments (Next.js Route Handler)
        → auth() → tenant_slug aus JWT Session
        → getApiKeyForTenant(tenant_slug) → Key aus DB (entschlüsselt)
        → api.masem.at/v1/payments (MMS, mit Tenant API Key als x-api-key)
            ← Response (snake_case)
        ← Transformed Response (camelCase)
    ← React Query Cache
```

**Data Flow (Admin):**
```
Browser (TanStack Query)
    → /api/admin/tenants (Next.js Route Handler)
        → auth() → isAdmin Check (Email-basiert)
        → process.env.MMS_ADMIN_API_KEY
        → api.masem.at/v1/admin/paywatcher/tenants (MMS, mit Admin API Key)
            ← Response
        ← Transformed Response
    ← React Query Cache
```

### Data Transformation

| Decision | Choice | Rationale |
|----------|--------|-----------|
| Naming Convention | **camelCase im Frontend** | TypeScript/React Convention, bessere IDE-Unterstützung. Transformation einmal im API-Proxy-Layer. |
| Transformation Layer | **Route Handlers (lib/api/transform.ts)** | Eine Funktion pro Entity-Type, ein Ort. Alles downstream ist sauberes TypeScript. |
| Amount Handling | **String-basiert** | MMS liefert Amounts als Strings ("49.000042"). Frontend zeigt sie als Strings an — keine BigInt-Konvertierung im Frontend nötig (das ist Backend-Logik). |

### Frontend Architecture

| Decision | Choice | Rationale |
|----------|--------|-----------|
| Theme Toggle | **next-themes** | Bewährt, 1.5kB, App Router Support, Dark Mode Default. |
| Client State | **React Context** | Nur für Theme + Auth Session. Kein komplexer Client-State — TanStack Query verwaltet allen Server-State. |
| Docs-Strategie | **Docs-Lite eingebettet, /docs Route mit MDX** | Quick Start, Endpoint Reference, Webhook Payload direkt im Projekt. Separate Docs-App (Mintlify/Fumadocs) erst Phase 2. |
| Syntax Highlighting | **Shiki** | Server-Side Rendering möglich, bessere Performance als Prism, mehr Sprachen. |

### Infrastructure & Deployment

| Decision | Choice | Rationale |
|----------|--------|-----------|
| Hosting | **Vercel Pro** | Bestehendes Setup, SSG + Serverless Functions, automatisches CI/CD. |
| Domain | **paywatcher.dev** | Gekauft, .dev = Developer-Audience. Route Groups, keine Subdomains im MVP. |
| Environment | **Vercel Environment Variables** | MMS_API_KEY, NEXTAUTH_SECRET, RESEND_API_KEY, ANALYTICS_KEY. Konfigurierbar per Environment (Dev/Preview/Prod). |
| Email | **Resend** | Bereits im masemIT Stack (Pro Plan). Request Access Notification Emails an contact@masem.at. |

### Decision Impact Analysis

**Implementation Sequence:**
1. Projekt-Scaffold (`create-next-app`) + Vercel Deployment
2. Auth-System (Auth.js v5 Email Provider + Magic Link + Neon Postgres DB + Parties API)
3. API Proxy Layer (Route Handlers + Transformation)
4. Dashboard Layout (Sidebar, Route Protection)
5. Landing Page (SSG, 8 Sektionen)
6. Dashboard Pages (Overview, Payments, Settings)
7. Admin Dashboard (Phase 1b)

**Cross-Component Dependencies:**
- Auth-System muss vor Dashboard existieren (Route Protection)
- API Proxy Layer muss vor Dashboard Pages existieren (Daten)
- Theme System muss früh stehen (alle Komponenten nutzen es)
- Landing Page ist unabhängig vom Dashboard (kann parallel entwickelt werden)

## Implementation Patterns & Consistency Rules

_Alle Patterns sind darauf ausgelegt, dass mehrere AI-Agents konsistenten, kompatiblen Code produzieren. Die UX-Spezifikation definiert visuelle Patterns — dieses Dokument definiert die technische Umsetzung._

### Critical Conflict Points Identified

12 Bereiche wo AI-Agents unterschiedliche Entscheidungen treffen könnten — alle unten aufgelöst.

### Naming Patterns

**File Naming (Next.js + React Convention):**
- React Components: `kebab-case.tsx` → `payment-status-badge.tsx`, `sidebar-nav.tsx`
- Route Handlers: `app/api/{resource}/route.ts` → `app/api/payments/route.ts`
- Hooks: `use-{name}.ts` → `use-payments.ts`, `use-auth.ts`
- Utilities: `kebab-case.ts` → `transform.ts`, `format-amount.ts`
- Types: `types.ts` (zentral) oder `{feature}.types.ts` (feature-spezifisch)
- Tests: `{name}.test.ts` co-located neben Source

**Component Naming (React Convention):**
- Components: `PascalCase` → `PaymentStatusBadge`, `SidebarNav`, `CodeBlock`
- Props: `PascalCase + Props` → `PaymentStatusBadgeProps`
- Hooks: `camelCase` mit `use` Prefix → `usePayments()`, `useAuth()`

**TypeScript Naming:**
- Types/Interfaces: `PascalCase` → `Payment`, `PaymentStatus`, `AuthSession`
- Enums: `PascalCase` → `PaymentStatus` (als union type, nicht als enum)
- Constants: `UPPER_SNAKE_CASE` → `PAYMENT_STATUSES`, `DEFAULT_CONFIRMATIONS`
- Functions: `camelCase` → `formatAmount()`, `getPaymentStatusColor()`
- Zod Schemas: `camelCase` + `Schema` → `createPaymentSchema`, `signupSchema`

**API Proxy Route Naming:**
- Endpoints: `kebab-case`, plural → `/api/payments`, `/api/webhook-config`
- Route params: `[id]` → `/api/payments/[id]/route.ts`
- Query params: `camelCase` in Frontend, transformiert zu `snake_case` für MMS

**Anti-Patterns:**
- NEVER `UserCard.tsx` (PascalCase File) — immer `user-card.tsx`
- NEVER `enum PaymentStatus {}` — immer Union Type: `type PaymentStatus = 'pending' | 'matched' | ...`
- NEVER `interface IPayment {}` — kein `I`-Prefix, einfach `Payment`

### Structure Patterns

**Projekt-Organisation (Feature + Shared):**

```
src/
├── app/                          # Next.js App Router
│   ├── (landing)/                # Public Landing Page
│   ├── (dashboard)/              # Auth-protected Dashboard
│   ├── (admin)/                  # Admin-protected (Phase 1b)
│   ├── api/                      # Route Handlers (Proxy)
│   └── layout.tsx                # Root Layout (Theme, Fonts)
├── components/
│   ├── ui/                       # shadcn/ui Components (generiert)
│   ├── landing/                  # Landing Page Components
│   ├── dashboard/                # Dashboard Components
│   └── shared/                   # Shared across areas (CodeBlock, etc.)
├── hooks/                        # Custom React Hooks
├── lib/
│   ├── api/                      # MMS API Client + Transformation
│   │   ├── client.ts             # Fetch-Wrapper mit Auth
│   │   ├── transform.ts          # snake_case → camelCase
│   │   └── types.ts              # MMS API Response Types (snake_case)
│   ├── auth.ts                   # Auth.js Konfiguration
│   ├── analytics.ts              # masemIT Analytics Helper
│   └── utils.ts                  # Shared Utilities
├── types/                        # Zentrale TypeScript Types (camelCase)
│   ├── payment.ts                # Payment, PaymentStatus, etc.
│   ├── tenant.ts                 # Tenant, WebhookConfig, etc.
│   └── auth.ts                   # AuthSession, User, etc.
└── styles/                       # Globale Styles + Theme Tokens
    └── globals.css               # Tailwind v4 @theme + Custom Properties
```

**Regeln:**
- Components in `components/` — nie in `app/` neben Pages
- Shared Components nur in `components/shared/` — nie in `components/landing/` oder `components/dashboard/` wenn in beiden Bereichen verwendet
- Types zentral in `types/` — nie in Component-Files definiert (außer Props direkt im File)
- `lib/` für Utilities und Konfiguration — keine React Components in `lib/`
- Tests co-located: `payment-status-badge.test.tsx` neben `payment-status-badge.tsx`
- Kein `src/utils/` oder `src/helpers/` — alles in `lib/`

### Format Patterns

**Route Handler Response Format:**

Route Handlers geben das MMS-Envelope-Format weiter, aber mit camelCase:

Success (single):
```json
{ "data": { "id": "uuid", "exactAmount": "49.000042" } }
```

Success (list):
```json
{
  "data": [{ "id": "uuid", "exactAmount": "49.000042" }],
  "meta": { "page": 1, "limit": 50, "total": 127 }
}
```

Error:
```json
{
  "error": { "code": "UNAUTHORIZED", "message": "Invalid or expired session" }
}
```

**Error Codes (Frontend-spezifisch):**
- `UNAUTHORIZED` (401) — Session expired, redirect to login
- `FORBIDDEN` (403) — Keine Berechtigung
- `NOT_FOUND` (404) — Resource nicht gefunden
- `VALIDATION_ERROR` (400) — Input-Validierung fehlgeschlagen
- `MMS_ERROR` (502) — MMS Backend nicht erreichbar
- `INTERNAL_ERROR` (500) — Unerwarteter Server-Fehler

**Date Formatting:**
- API: ISO 8601 Strings in UTC (`"2026-02-15T10:00:00Z"`) — durchgereicht von MMS
- UI-Anzeige: `Intl.DateTimeFormat` oder `date-fns` — kein Moment.js
- Timestamps in Dashboard: `HH:mm:ss UTC` (wie in UX-Spec Timelines)
- Relative Zeiten: Nicht verwenden — absolute Timestamps sind klarer für Developer-Audience

**Amount Formatting:**
- API liefert Strings: `"49.000042"`
- UI zeigt mit USDC-Suffix: `49.000042 USDC`
- Monospace (Geist Mono) für alle Amounts — immer
- Keine Rundung, keine Locale-Formatierung (Dezimalpunkt, nicht Komma)

### Component Patterns

**Payment Status → Farbe/Stil Mapping (aus UX-Spec, kodifiziert):**

```typescript
// types/payment.ts
type PaymentStatus = 'pending' | 'matched' | 'confirming' | 'confirmed' | 'expired' | 'failed'

// lib/utils.ts
const STATUS_CONFIG: Record<PaymentStatus, { color: string; label: string }> = {
  pending:    { color: 'muted',   label: 'PENDING' },
  matched:    { color: 'accent',  label: 'MATCHED' },
  confirming: { color: 'warning', label: 'CONFIRMING' },
  confirmed:  { color: 'success', label: 'CONFIRMED' },
  expired:    { color: 'neutral', label: 'EXPIRED' },
  failed:     { color: 'error',   label: 'FAILED' },
}
```

**Button-Hierarchie (aus UX-Spec, durchgesetzt):**
- Max 1 Primary Button pro Screen — Verletzung = UX-Bug
- Destruktive Aktionen: Danger-Variant + Confirmation Dialog
- Copy-Buttons: Icon-only, Inline-Checkmark-Feedback (kein Toast)

**Loading States:**
- Dashboard-Seiten: Skeleton (shadcn/ui `<Skeleton />`)
- Sidebar + Page-Header stehen sofort — nur Content-Bereich zeigt Skeleton
- Kein Full-Page-Spinner — nirgendwo, niemals
- Einzige Ausnahme: Onboarding "Waiting for first payment..." Spinner

**Toast Pattern (aus UX-Spec):**
- Library: sonner (shadcn/ui Standard)
- Position: rechts unten
- Copy-Aktionen: KEIN Toast — nur Inline-Checkmark am Button
- Save/Update: 3s auto-dismiss
- Fehler: Bleibt bis manuell dismissed
- Netzwerk-Fehler: "Could not reach server" — bleibt

### API Client Pattern

**MMS API Client (lib/api/client.ts):**

```typescript
// Alle MMS Calls gehen durch diese Funktion
async function mmsApi<T>(path: string, options?: RequestInit): Promise<T> {
  const res = await fetch(`${process.env.MMS_API_URL}${path}`, {
    ...options,
    headers: {
      'Content-Type': 'application/json',
      ...options?.headers,
      // x-api-key wird vom Route Handler per options.headers übergeben
    },
  })
  if (!res.ok) { /* Error handling */ }
  return res.json()
}
```

**Regeln:**
- `mmsApi()` ist die EINZIGE Funktion die MMS aufruft — nie raw `fetch` zu api.masem.at
- API Key kommt aus der JWT Session (im Route Handler extrahiert, als x-api-key Header übergeben)
- Transformation (snake → camel) passiert im Route Handler NACH dem mmsApi-Call
- TanStack Query im Browser ruft IMMER `/api/...` Route Handlers auf — nie direkt MMS

**TanStack Query Patterns:**

```typescript
// hooks/use-payments.ts
export function usePayments(filters?: PaymentFilters) {
  return useQuery({
    queryKey: ['payments', filters],
    queryFn: () => fetchApi('/api/payments', { params: filters }),
  })
}
```

- Query Keys: Array-basiert, hierarchisch: `['payments']`, `['payments', id]`, `['payments', filters]`
- Mutations invalidieren relevante Queries: `queryClient.invalidateQueries({ queryKey: ['payments'] })`
- Polling für aktive Payments: `refetchInterval: 10_000` (10s, konsistent mit Backend-Polling)
- Stale Time: 30s für Payments, 5min für Settings

### Auth Pattern

**Auth.js v5 Integration (Magic Link + Parties API):**

```typescript
// lib/auth.ts
export const { handlers, signIn, signOut, auth } = NextAuth({
  providers: [
    // Magic Link via Resend (Email Provider)
    Resend({
      from: process.env.RESEND_FROM_EMAIL,
    }),
  ],
  callbacks: {
    async signIn({ user }) {
      // Post-Login: Resolve Party + Tenants via MMS
      // POST /v1/auth/login { email: user.email, module: "paywatcher" }
      // Store party_id, tenants[] in user object
      return true
    },
    jwt({ token, user }) {
      // party_id, display_name, tenant_slug, role ins JWT
      // KEIN API Key im JWT — Key kommt aus DB
    },
    session({ session, token }) {
      // party_id, display_name, tenant_slug, role in Session
    },
  },
  session: { strategy: 'jwt' },
})
```

**API Key Lookup (lib/api/tenant-keys.ts):**

```typescript
// Server-Side only — nie im Browser importieren
async function getApiKeyForTenant(tenantSlug: string): Promise<string> {
  // 1. Lookup in tenant_keys DB (Neon Postgres)
  // 2. Decrypt API Key
  // 3. Return plaintext Key für MMS-Call
}
```

**Route Protection:**
- Dashboard: `auth()` Check in `(dashboard)/layout.tsx` — redirect zu Login wenn keine Session
- Admin: `auth()` Check + isAdmin Check (Email-basiert) in `(admin)/layout.tsx`
- Route Handlers (Tenant): `auth()` Check + `getApiKeyForTenant(session.tenant_slug)` für MMS-Calls
- Route Handlers (Admin): `auth()` Check + isAdmin + `process.env.MMS_ADMIN_API_KEY` für MMS-Calls
- Landing Page: Kein Auth-Check — vollständig public

**Tenant-Auswahl (Multi-Tenant):**
- POST /v1/auth/login gibt `tenants[]` zurück
- `tenants.length === 0`: Redirect zu "No account — Request Access?"
- `tenants.length === 1`: Auto-Select, tenant_slug in JWT
- `tenants.length > 1`: Tenant-Auswahl-UI, User wählt, tenant_slug in JWT
- Tenant-Switch: Im Dashboard-Header, ändert tenant_slug in Session

### Analytics Pattern

**masemIT Analytics Integration:**

```typescript
// lib/analytics.ts
export function trackEvent(name: string, properties?: Record<string, string>) {
  // Async, non-blocking
  // Nur in Production (nicht in Dev/Preview)
}
```

- Events: `snake_case` Naming (wie im PRD definiert: `lp_view`, `lp_hero_cta`, `dash_view`)
- Async: `navigator.sendBeacon` oder `fetch` mit `keepalive: true`
- Kein UI-Blocking — Fire-and-Forget
- Nur in Production: `process.env.NODE_ENV === 'production'`

### Enforcement Guidelines

**All AI Agents MUST:**
1. Dateien in `kebab-case.tsx` benennen — keine PascalCase-Dateien
2. Types in `types/` definieren — nicht inline in Components (außer Props)
3. MMS-Calls ausschließlich über `mmsApi()` in `lib/api/client.ts` — nie raw fetch
4. Route Handler Responses im Envelope-Format (`{ data }` / `{ error }`) — nie raw Objects
5. snake_case → camelCase Transformation im Route Handler — nie im Client-Code
6. Status-Badges immer mit Farbe + Text-Label — nie nur Farbe
7. Skeleton Loading für Dashboard-Seiten — nie Full-Page-Spinner
8. Copy-Feedback als Inline-Checkmark — nie als Toast
9. Max 1 Primary Button pro Screen — Verletzung ist ein Bug
10. Amounts immer in Geist Mono — nie in Sans

**Anti-Patterns (NEVER do this):**
- Direct fetch zu `api.masem.at` aus dem Browser
- API Key in Client-Side Code oder `.env.local` ohne `_` Prefix
- `useState` für Server-State der in TanStack Query gehört
- Eigene Loading-Spinner statt Skeleton
- Toast für Copy-Aktionen
- `moment.js` für Date-Formatting
- `enum` statt Union Types
- Inline-Styles statt Tailwind Utilities
- `any` Type — immer explizite Types

## Project Structure & Boundaries

### Requirements → Architektur Mapping

**FR-Kategorie: Landing Page & Marketing (FR1–FR7):**
→ `app/(landing)/page.tsx` + `components/landing/`
- Hero Section, Problem/Solution, How It Works, Code Examples, Pricing, Comparison, Trust Signals, Footer
- SSG, public, SEO-optimiert
- Shared: `components/shared/code-block.tsx` (Shiki)

**FR-Kategorie: Auth & Onboarding (FR9, FR10, FR36, FR37):**
→ `app/(landing)/login/`, `lib/auth.ts`, `lib/db/`, `app/api/auth/`, `app/api/request-access/`
- Magic Link Login (Auth.js v5 Email Provider via Resend), Auth.js JWT
- POST /v1/auth/login → Parties API Tenant-Resolution (Email → tenant_slug + role)
- Tenant API Key aus Neon Postgres DB (`lib/db/` + `lib/api/tenant-keys.ts`)
- Multi-Tenant: Tenant-Auswahl wenn User mehrere Tenants hat
- Request Access Formular → Email via Resend an contact@masem.at (FR36)
- Kein Self-Service Signup (~~FR8~~), kein Auto-API-Key (~~FR11~~)
- Cross-Dependencies: `lib/api/client.ts` (mmsApi), `lib/db/` (Neon Postgres), Resend (Magic Link)

**FR-Kategorie: API Key Management (FR12, FR14):**
→ `app/(dashboard)/settings/api-keys/`, `components/dashboard/api-key-card.tsx`
- Anzeigen (masked via GET /v1/paywatcher/config/api-key), Scopes einsehen
- Kein Regenerieren (~~FR13~~ — kein MMS-Endpoint)
- Proxy: `app/api/config/api-key/route.ts`

**FR-Kategorie: Payment Monitoring (FR15–FR19, FR38):**
→ `app/(dashboard)/payments/`, `components/dashboard/payment-*.tsx`, `hooks/use-payments.ts`
- Payments-Liste, Status-Filter, Zeitraum-Filter, Sortierung (nur created_at), Pagination
- Webhook History pro Payment (FR38) via GET /v1/payments/:id/webhooks
- Proxy: `app/api/payments/route.ts`, `app/api/payments/[id]/route.ts`, `app/api/payments/[id]/webhooks/route.ts`
- Shared: Payment Status Badge, Amount Display

**FR-Kategorie: Webhook Management (FR20–FR22):**
→ `app/(dashboard)/settings/webhooks/`, `hooks/use-webhook-config.ts`
- URL konfigurieren (PATCH /v1/paywatcher/config), Test-Webhook senden (POST /v1/paywatcher/config/test-webhook), Delivery Log
- Proxy: `app/api/config/route.ts`, `app/api/config/test-webhook/route.ts`

**FR-Kategorie: Tenant Settings (FR23–FR25):**
→ `app/(dashboard)/settings/`, `components/dashboard/settings-*.tsx`
- Confirmation Override (PATCH /v1/paywatcher/config), Deposit Address (read-only), Account Settings
- Proxy: `app/api/config/route.ts`

**~~FR-Kategorie: Invoicing (FR26–FR27):~~ Entfällt (Managed MVP Phase 2)**

**FR-Kategorie: Documentation (FR28–FR31):**
→ `app/(landing)/docs/`, `content/docs/`
- Quick Start, Endpoint Reference, Webhook Events, Code Examples
- MDX-basiert (Docs-Lite, eingebettet)

**FR-Kategorie: Analytics & SEO (FR32–FR35):**
→ `lib/analytics.ts`, `app/(landing)/layout.tsx` (Meta Tags), `app/sitemap.ts`, `app/robots.ts`
- masemIT Analytics Events, OG Images, Sitemap

**Cross-Cutting Concerns:**
- **Auth System:** `lib/auth.ts` + `middleware.ts` + `app/api/auth/[...nextauth]/route.ts` (Magic Link + Parties API Tenant-Resolution) + `lib/db/` (Neon Postgres für Tenant API Keys)
- **API Client:** `lib/api/client.ts` + `lib/api/transform.ts` + `lib/api/types.ts`
- **Theme System:** `styles/globals.css` + `components/shared/theme-provider.tsx`
- **Error Handling:** `lib/api/errors.ts` + `components/shared/error-boundary.tsx`
- **Loading States:** `components/ui/skeleton.tsx` (shadcn/ui)

### Vollständige Projektverzeichnis-Struktur

```
paywatcher/
├── .env.example                    # Vorlage für Environment Variables
├── .env.local                      # Lokale Secrets (gitignored)
├── .eslintrc.json                  # ESLint Konfiguration
├── .gitignore
├── .github/
│   └── workflows/
│       └── ci.yml                  # GitHub Actions (Lint, Type-Check)
├── next.config.ts                  # Next.js 16 Konfiguration
├── package.json
├── tsconfig.json
├── drizzle.config.ts              # Drizzle ORM Konfiguration (Neon Postgres)
├── README.md
├── public/
│   ├── og-image.png                # Open Graph Default Image
│   ├── favicon.ico
│   └── assets/
│       └── logo.svg                # PayWatcher Logo
│
├── content/                        # MDX Docs Content
│   └── docs/
│       ├── quick-start.mdx         # FR28: Quick Start Guide
│       ├── endpoints.mdx           # FR29: Endpoint Reference
│       ├── webhooks.mdx            # FR30: Webhook Event Reference
│       └── examples.mdx            # FR31: Code Examples (curl, JS, Python)
│
└── src/
    ├── app/
    │   ├── layout.tsx              # Root Layout (Theme, Fonts, Analytics)
    │   ├── not-found.tsx           # Custom 404
    │   ├── sitemap.ts              # FR34: Dynamic Sitemap
    │   ├── robots.ts               # FR34: Robots.txt
    │   │
    │   ├── (landing)/              # PUBLIC — SSG, SEO-optimiert
    │   │   ├── layout.tsx          # Landing Layout (Header, Footer)
    │   │   ├── page.tsx            # FR1-FR7, FR36: Landing Page (8 Sektionen + Request Access)
    │   │   ├── login/
    │   │   │   └── page.tsx        # FR9: Login (Magic Link via Resend)
    │   │   └── docs/
    │   │       ├── page.tsx        # Docs Index
    │   │       └── [slug]/
    │   │           └── page.tsx    # FR28-FR31: MDX Doc Pages
    │   │
    │   ├── (dashboard)/            # AUTH-PROTECTED — CSR, Desktop-First
    │   │   ├── layout.tsx          # Dashboard Layout (Sidebar, Auth-Check)
    │   │   ├── page.tsx            # Overview (Onboarding oder Stats)
    │   │   ├── payments/
    │   │   │   ├── page.tsx        # FR15-FR19: Payments List
    │   │   │   └── [id]/
    │   │   │       └── page.tsx    # Payment Detail + Webhook History (FR38)
    │   │   └── settings/
    │   │       ├── page.tsx        # Settings Overview / Redirect
    │   │       ├── api-keys/
    │   │       │   └── page.tsx    # FR12-FR14: API Key Management
    │   │       ├── webhooks/
    │   │       │   └── page.tsx    # FR20-FR22: Webhook Config + Log
    │   │       └── account/
    │   │           └── page.tsx    # FR23-FR25: Confirmations, Address, Email
    │   │
    │   ├── (admin)/                # ADMIN-PROTECTED — Phase 1b, Desktop-Only
    │   │   ├── layout.tsx          # Admin Layout (Auth + Admin-Check)
    │   │   ├── page.tsx            # System Overview
    │   │   ├── tenants/
    │   │   │   ├── page.tsx        # Tenant List
    │   │   │   └── [id]/
    │   │   │       └── page.tsx    # Tenant Detail
    │   │   ├── payments/
    │   │   │   └── page.tsx        # Global Payments
    │   │   └── system/
    │   │       └── page.tsx        # System Config
    │   │
    │   └── api/                    # ROUTE HANDLERS — Server-Side Proxy
    │       ├── auth/
    │       │   └── [...nextauth]/
    │       │       └── route.ts    # Auth.js v5 (Magic Link + Parties Tenant-Resolution)
    │       ├── payments/
    │       │   ├── route.ts        # GET (list) → MMS /v1/payments
    │       │   └── [id]/
    │       │       ├── route.ts    # GET (single) → MMS /v1/payments/:id
    │       │       └── webhooks/
    │       │           └── route.ts # GET (webhook history) → MMS /v1/payments/:id/webhooks
    │       ├── config/
    │       │   ├── route.ts        # GET, PATCH → MMS /v1/paywatcher/config
    │       │   ├── api-key/
    │       │   │   └── route.ts    # GET (masked key) → MMS /v1/paywatcher/config/api-key
    │       │   └── test-webhook/
    │       │       └── route.ts    # POST → MMS /v1/paywatcher/config/test-webhook
    │       └── request-access/
    │           └── route.ts        # POST → Email via Resend an contact@masem.at
    │
    ├── components/
    │   ├── ui/                     # shadcn/ui (generiert via CLI)
    │   │   ├── button.tsx
    │   │   ├── badge.tsx
    │   │   ├── card.tsx
    │   │   ├── dialog.tsx
    │   │   ├── input.tsx
    │   │   ├── skeleton.tsx
    │   │   ├── table.tsx
    │   │   ├── tabs.tsx
    │   │   ├── toast.tsx           # sonner Integration
    │   │   └── ...                 # Weitere nach Bedarf
    │   │
    │   ├── landing/                # Landing Page Components
    │   │   ├── hero-section.tsx           # FR1: Hero + CTA
    │   │   ├── problem-solution.tsx       # FR1: Problem/Solution
    │   │   ├── how-it-works.tsx           # FR5: 3-Step Flow
    │   │   ├── code-examples.tsx          # FR2: Interactive Code Tabs
    │   │   ├── pricing-table.tsx          # FR3: Pricing Tiers
    │   │   ├── comparison-table.tsx       # FR4: Kostenvergleich
    │   │   ├── trust-signals.tsx          # FR6: Trust Badges
    │   │   ├── landing-header.tsx         # Navigation + CTA
    │   │   └── landing-footer.tsx         # Footer Links
    │   │
    │   ├── dashboard/              # Dashboard Components
    │   │   ├── sidebar-nav.tsx            # Sidebar Navigation
    │   │   ├── dashboard-header.tsx       # Top Bar (User, Theme Toggle)
    │   │   ├── payment-list.tsx           # FR15: Payments Table
    │   │   ├── payment-filters.tsx        # FR16-FR18: Status, Zeitraum, Sort
    │   │   ├── payment-status-badge.tsx   # Status Badge (Farbe + Label)
    │   │   ├── payment-detail.tsx         # Payment Detail + Webhook History (FR38)
    │   │   ├── payment-timeline.tsx       # State Timeline
    │   │   ├── webhook-history-table.tsx  # FR38: Webhook History pro Payment
    │   │   ├── api-key-card.tsx           # FR12, FR14: Key Display (masked) + Scopes
    │   │   ├── webhook-config-form.tsx    # FR20: URL Config (PATCH)
    │   │   ├── webhook-test-button.tsx    # FR21: Test Webhook (POST)
    │   │   ├── settings-form.tsx          # FR23-FR25: Confirmation Override, Address
    │   │   ├── onboarding-wizard.tsx      # First-Time Setup Flow
    │   │   └── overview-stats.tsx         # Dashboard Overview Cards
    │   │
    │   └── shared/                 # Shared across areas
    │       ├── code-block.tsx             # Shiki Syntax Highlighting
    │       ├── copy-button.tsx            # Copy-to-Clipboard + Checkmark
    │       ├── amount-display.tsx         # USDC Amount (Geist Mono)
    │       ├── theme-provider.tsx         # next-themes Provider
    │       ├── theme-toggle.tsx           # Dark/Light Switch
    │       ├── error-boundary.tsx         # React Error Boundary
    │       ├── pagination.tsx             # Reusable Pagination Controls
    │       └── empty-state.tsx            # Empty List State
    │
    ├── hooks/                      # Custom React Hooks
    │   ├── use-payments.ts         # TanStack Query: Payments List
    │   ├── use-payment.ts          # TanStack Query: Single Payment + Webhook History
    │   ├── use-config.ts           # TanStack Query: Tenant Config (GET, PATCH)
    │   ├── use-api-key.ts          # TanStack Query: Masked API Key
    │   └── use-copy.ts             # Clipboard Copy + Checkmark State
    │
    ├── lib/
    │   ├── api/
    │   │   ├── client.ts           # mmsApi() — einziger MMS Fetch Wrapper
    │   │   ├── tenant-keys.ts      # getApiKeyForTenant() — DB Lookup + Decrypt (Server-Side only)
    │   │   ├── transform.ts        # snake_case → camelCase per Entity
    │   │   ├── types.ts            # MMS API Response Types (snake_case)
    │   │   └── errors.ts           # Error Handling + Error Code Mapping
    │   ├── db/
    │   │   ├── index.ts            # Drizzle Client (Neon Serverless)
    │   │   └── schema.ts           # DB Schema: tenant_keys + NextAuth Tables
    │   ├── auth.ts                 # Auth.js v5 Konfiguration (Magic Link + Parties Resolution)
    │   ├── analytics.ts            # masemIT Analytics (async, non-blocking)
    │   └── utils.ts                # Shared Utilities (cn, formatAmount, STATUS_CONFIG)
    │
    ├── types/                      # Zentrale TypeScript Types (camelCase)
    │   ├── payment.ts              # Payment, PaymentStatus, PaymentFilters, WebhookDelivery
    │   ├── tenant.ts               # TenantConfig, ApiKeyInfo
    │   └── auth.ts                 # AuthSession, User, JwtPayload
    │
    ├── middleware.ts               # Next.js Middleware (Auth-Check, Route Protection)
    │
    └── styles/
        └── globals.css             # Tailwind v4 @theme + Design Tokens + Custom Properties
```

### Architectural Boundaries

**API Boundaries:**

| Boundary | Richtung | Mechanismus |
|----------|----------|-------------|
| Browser → Route Handlers | `/api/*` | TanStack Query → fetch. Auth via Auth.js Session Cookie |
| Route Handlers → MMS | `api.masem.at/v1/*` | `mmsApi()` mit Tenant API Key aus JWT Session |
| Browser → Analytics | `analytics.masem.at` | `sendBeacon` / `fetch(keepalive)`, async |
| Route Handler → Resend | Resend API | Request Access Notification Email an contact@masem.at |

**Regel:** Browser hat NIEMALS direkten Kontakt zu `api.masem.at`. Alles geht über `/api/` Route Handlers.

**Component Boundaries:**

| Boundary | Beschreibung |
|----------|-------------|
| `(landing)` → `(dashboard)` | Keine Shared Components außer über `components/shared/`. Landing Page importiert NIE aus `components/dashboard/` und umgekehrt. |
| `components/ui/` | Nur shadcn/ui generierte Dateien. Keine Custom-Logik in `ui/`. |
| `hooks/` → `lib/api/` | Hooks rufen nur `/api/` Route Handlers auf. Nie `mmsApi()` direkt (das ist Server-Side only). |
| `lib/api/` | Server-Side only. Wird nur in Route Handlers (`app/api/`) importiert. Nie in Client Components. |

**Data Boundaries:**

| Schicht | Datenformat | Verantwortung |
|---------|-------------|---------------|
| MMS API Response | `snake_case` | MMS Backend |
| Route Handler (transform) | `snake_case` → `camelCase` | `lib/api/transform.ts` |
| Route Handler Response | `camelCase` + Envelope `{ data }` | Route Handler |
| TanStack Query Cache | `camelCase` TypeScript Types | `types/*.ts` |
| React Components | `camelCase` Props | Component Props |

### Integration Points

**Internal Communication:**

```
┌──────────────────────────────────────────────────────────────┐
│  Browser                                                      │
│  ┌──────────────┐    ┌──────────────┐    ┌──────────────┐   │
│  │ Landing Page  │    │  Dashboard   │    │  Admin        │   │
│  │ (SSG/Static)  │    │ (CSR/Auth)   │    │ (CSR/Admin)   │   │
│  └──────┬───────┘    └──────┬───────┘    └──────┬───────┘   │
│         │                   │                    │            │
│         │            ┌──────┴───────┐            │            │
│         │            │ TanStack     │            │            │
│         │            │ Query Cache  │            │            │
│         │            └──────┬───────┘            │            │
└─────────┼───────────────────┼────────────────────┼───────────┘
          │                   │                    │
    ┌─────┴───────────────────┴────────────────────┴──────┐
    │  Next.js Server (Vercel Serverless)                  │
    │  ┌───────────────┐  ┌───────────────┐               │
    │  │ middleware.ts  │  │ Route Handlers│               │
    │  │ (Auth Check)   │  │ /api/*        │               │
    │  └───────────────┘  └───────┬───────┘               │
    │                             │                        │
    │              ┌──────────────┼──────────────┐         │
    │              │              │              │         │
    │    ┌─────────┴───┐  ┌──────┴──────┐  ┌───┴──────┐  │
    │    │ lib/auth.ts  │  │ lib/api/    │  │ lib/db/  │  │
    │    │ Magic Link   │  │ client.ts   │  │ Neon PG  │  │
    │    │ + Parties    │  │ mmsApi()    │  │ tenant   │  │
    │    │ Resolution   │  │             │  │ _keys    │  │
    │    └──────┬──────┘  └──────┬──────┘  └───┬──────┘  │
    └───────────┼────────────────┼──────────────┼─────────┘
                │                │              │
     ┌──────────┴──┐  ┌─────────┴───────┐  ┌──┴────────┐
     │  Resend      │  │  MMS Backend     │  │  Neon     │
     │  Magic Link  │  │  api.masem.at    │  │  Postgres │
     │  Emails      │  │  ┌─────────────┐ │  │  (DB)     │
     └─────────────┘  │  │Payments     │ │  └───────────┘
                       │  │PayWatcher   │ │
                       │  │Parties/Auth │ │
                       │  └─────────────┘ │
                       └──────────────────┘
```

**External Integrations:**

| Service | Zweck | Ort im Code |
|---------|-------|-------------|
| MMS Backend (api.masem.at) | Alle Business-Daten + Parties/Auth | `lib/api/client.ts` via Route Handlers |
| Neon Postgres | Frontend-DB (Tenant API Keys, Auth.js Sessions) | `lib/db/index.ts`, `lib/db/schema.ts` |
| Resend | Magic Link Emails + Request Access Notifications | Auth.js Email Provider + `app/api/request-access/route.ts` |
| masemIT Analytics | Event Tracking | `lib/analytics.ts` |
| Basescan | txHash Block Explorer Links | `components/dashboard/payment-detail.tsx` (Links) |
| Vercel | Hosting + CI/CD | `next.config.ts`, GitHub Integration |

### File Organization Patterns

**Configuration Files:**

| Datei | Zweck |
|-------|-------|
| `.env.example` | Template: `MMS_API_URL`, `MMS_ADMIN_API_KEY` (Admin-Calls), `DATABASE_URL` (Neon Postgres), `AUTH_SECRET`, `AUTH_URL`, `RESEND_API_KEY`, `RESEND_FROM_EMAIL`, `NEXT_PUBLIC_ANALYTICS_PROJECT` |
| `next.config.ts` | Images (domains), Redirects, MDX Plugin |
| `tsconfig.json` | Path Aliases: `@/` → `src/`, strict mode |
| `components.json` | shadcn/ui Konfiguration |
| `middleware.ts` | Route Protection Regeln |

**Test Organization:**

```
src/
├── components/
│   ├── dashboard/
│   │   ├── payment-status-badge.tsx
│   │   └── payment-status-badge.test.tsx    # Co-located Unit Test
│   └── shared/
│       ├── amount-display.tsx
│       └── amount-display.test.tsx
├── hooks/
│   ├── use-payments.ts
│   └── use-payments.test.ts
├── lib/
│   ├── api/
│   │   ├── transform.ts
│   │   └── transform.test.ts               # Transform Logic Tests
│   └── utils.ts
│       └── utils.test.ts
└── __tests__/                               # Integration + E2E
    ├── api/                                 # Route Handler Integration Tests
    │   └── payments.test.ts
    └── e2e/                                 # Playwright E2E (Phase 2)
        └── login-flow.spec.ts
```

- Unit Tests: Co-located (`*.test.ts` neben Source)
- Integration Tests: `src/__tests__/api/` für Route Handler Tests
- E2E Tests: `src/__tests__/e2e/` für Playwright (Phase 2)
- Test Library: Vitest + React Testing Library

### Development Workflow Integration

**Development Server:**
```bash
npm run dev          # Next.js 16 + Turbopack (localhost:3000)
```

**Build Process:**
```bash
npm run build        # Next.js Production Build (Turbopack)
npm run lint         # ESLint
npm run type-check   # tsc --noEmit
npm run test         # Vitest
```

**Deployment:**
- Push to `main` → Vercel Production Deploy (paywatcher.dev)
- Push to Feature Branch → Vercel Preview Deploy
- Environment Variables in Vercel Dashboard pro Environment (Dev/Preview/Production)

**Monorepo-Status:** Kein Monorepo. Ein einzelnes Next.js Projekt. `content/docs/` für MDX-Content im gleichen Repo.

## Architecture Validation Results

### Coherence Validation ✅

**Decision Compatibility:**

| Entscheidung A | Entscheidung B | Kompatibel? |
|----------------|----------------|-------------|
| Next.js 16 (React 19) | Auth.js v5 (App Router) | ✅ |
| Next.js 16 (React 19) | TanStack Query v5 | ✅ |
| Next.js 16 (React 19) | Recharts | ✅ |
| Tailwind v4 (CSS-first) | shadcn/ui | ✅ |
| Tailwind v4 (CSS-first) | next-themes | ✅ |
| Server-Side Proxy | TanStack Query | ✅ |
| JWT Strategy (Cookie) + Neon DB | Route Handler Auth | ✅ |
| MDX Content | Next.js 16 App Router | ✅ |
| Shiki (SSG Highlighting) | MDX | ✅ |
| Vercel Pro | Alle oben | ✅ |

Keine Konflikte. Alle Technologie-Entscheidungen sind untereinander kompatibel.

**Pattern Consistency:**
- File Naming (kebab-case) konsistent über alle Bereiche ✅
- Component Naming (PascalCase) konsistent ✅
- Type Naming (PascalCase, Union Types statt Enum) ✅
- API Response Format (Envelope) konsistent ✅
- snake→camel Transformation an genau einer Stelle ✅
- TanStack Query Key Convention hierarchisch ✅
- Error Code Schema über alle Route Handlers ✅

**Structure Alignment:**
- Route Groups unterstützen SSG/CSR Split ✅
- `lib/api/` Server-Only Boundary respektiert ✅
- Component Boundaries (landing/dashboard/shared) klar ✅
- Test Co-Location Pattern konsistent ✅
- MDX Content außerhalb src/ (content/) korrekt ✅

### Requirements Coverage Validation ✅

**Functional Requirements: 30/38 aktiv im Managed MVP (100% abgedeckt)**

| FR | Requirement | Architektur-Support | Status |
|----|------------|---------------------|--------|
| FR1 | Value Proposition | `components/landing/hero-section.tsx` | ✅ |
| FR2 | Code Examples | `code-examples.tsx` + `shared/code-block.tsx` | ✅ |
| FR3 | Pricing Tiers | `pricing-table.tsx` | ✅ |
| FR4 | Kostenvergleich | `comparison-table.tsx` | ✅ |
| FR5 | How It Works | `how-it-works.tsx` | ✅ |
| FR6 | Trust Signals | `trust-signals.tsx` | ✅ |
| FR7 | Request Access CTA | `landing-header.tsx` CTA → Request Access Formular | ✅ |
| ~~FR8~~ | ~~Signup~~ | — | Entfällt (Managed MVP) |
| FR9 | Login (Magic Link) | `login/page.tsx` + `api/auth/` (Email Provider + Parties Resolution) | ✅ |
| FR10 | Logout | Auth.js `signOut()` | ✅ |
| ~~FR11~~ | ~~Auto API Key~~ | — | Entfällt (Managed MVP) |
| FR12 | Key Display | `api-key-card.tsx` + `api/config/api-key/` | ✅ |
| ~~FR13~~ | ~~Key Regenerate~~ | — | Entfällt (kein MMS-Endpoint) |
| FR14 | Key Scopes | `api-key-card.tsx` (aus Config Response) | ✅ |
| FR15 | Payments List | `payment-list.tsx` + `use-payments.ts` | ✅ |
| FR16 | Status Filter | `payment-filters.tsx` | ✅ |
| FR17 | Zeitraum Filter | `payment-filters.tsx` | ✅ |
| FR18 | Sortierung | `payment-filters.tsx` (nur created_at) | ✅ |
| FR19 | Pagination | `shared/pagination.tsx` + `use-payments.ts` | ✅ |
| FR20 | Webhook URL | `webhook-config-form.tsx` (PATCH config) | ✅ |
| FR21 | Test Webhook | `webhook-test-button.tsx` (POST test-webhook) | ✅ |
| FR22 | Delivery Log | `webhook-history-table.tsx` (pro Payment) | ✅ |
| FR23 | Confirmations | `settings-form.tsx` (PATCH config) | ✅ |
| FR24 | Deposit Address | `settings-form.tsx` (read-only aus Config) | ✅ |
| FR25 | Account Settings | `settings/account/page.tsx` | ✅ |
| ~~FR26~~ | ~~Invoices List~~ | — | Entfällt (Managed MVP Phase 2) |
| ~~FR27~~ | ~~PDF Download~~ | — | Entfällt (Managed MVP Phase 2) |
| FR28 | Quick Start | `content/docs/quick-start.mdx` | ✅ |
| FR29 | Endpoint Ref | `content/docs/endpoints.mdx` | ✅ |
| FR30 | Webhook Ref | `content/docs/webhooks.mdx` | ✅ |
| FR31 | Code Examples | `content/docs/examples.mdx` | ✅ |
| FR32 | Analytics Events | `lib/analytics.ts` | ✅ |
| FR33 | Analytics Send | `lib/analytics.ts` (sendBeacon) | ✅ |
| FR34 | SEO | `sitemap.ts`, `robots.ts`, Layout Meta | ✅ |
| FR35 | Social Sharing | OG Image, Layout Meta | ✅ |
| FR36 | Request Access Form | `api/request-access/route.ts` + Landing Page Form | ✅ NEU |
| FR37 | Magic Link Login | `login/page.tsx` + Email Provider + `lib/db/` (Tenant Keys) + POST /v1/auth/login | ✅ NEU |
| FR38 | Webhook History | `api/payments/[id]/webhooks/route.ts` + `webhook-history-table.tsx` | ✅ NEU |

**Non-Functional Requirements: 16/17 aktiv (~~NFR3~~ entfällt)**

| NFR | Requirement | Architektur-Support | Status |
|-----|------------|---------------------|--------|
| NFR1 | FCP < 1.5s | SSG Landing Page + Vercel CDN | ✅ |
| NFR2 | Dashboard < 2s | TanStack Query + Route Handlers | ✅ |
| ~~NFR3~~ | ~~Signup < 5s~~ | — | Entfällt (kein Self-Service Signup) |
| NFR4 | Code Copy instant | Shiki SSG pre-rendered | ✅ |
| NFR5 | HTTPS | Vercel enforced TLS | ✅ |
| NFR6 | Keys gehasht | MMS Backend, Frontend zeigt masked | ✅ |
| NFR7 | Token Lifetime | Auth.js JWT Expiry | ✅ |
| NFR8 | Protected Routes | `middleware.ts` + Layout Auth | ✅ |
| NFR9 | CSRF | Auth.js v5 built-in | ✅ |
| NFR10 | GDPR | Neon Postgres (EU-Region), MMS Frankfurt, minimale PII | ✅ |
| NFR11 | REST Only | `lib/api/client.ts` | ✅ |
| NFR12 | Minimale DB-Nutzung | Neon Postgres nur für Tenant API Keys + Auth Sessions (keine Business-Daten) | ✅ |
| NFR13 | Async Analytics | sendBeacon / fetch keepalive | ✅ |
| NFR14 | Block Explorer | payment-detail.tsx Basescan-Links | ✅ |
| NFR15 | Vercel CI/CD | Git Push → Auto Deploy | ✅ |
| NFR16 | Env Vars | `.env.example` + Vercel Dashboard | ✅ |
| NFR17 | No Secrets | Server-Side Proxy Pattern (Tenant API Key verschlüsselt in DB + JWT, nie im Browser) | ✅ |

### Implementation Readiness Validation ✅

**Decision Completeness:**
- Alle kritischen Entscheidungen dokumentiert mit Versionen ✅
- Code-Beispiele für alle Major Patterns ✅
- 10 Enforcement Rules + Anti-Pattern Liste ✅
- Starter Command dokumentiert ✅

**Structure Completeness:**
- Vollständiger Dateibaum (~80 Dateien/Verzeichnisse) ✅
- Jede Datei mit FR-Referenz annotiert ✅
- 6 External Integration Points + Architektur-Diagramm ✅
- 4 Component Boundary Rules + 5-Schichten Data Boundaries ✅

**Pattern Completeness:**
- 6 Naming-Kategorien + Anti-Patterns ✅
- Component Patterns (Status, Buttons, Loading, Toasts) ✅
- API Client + Auth + Analytics Patterns mit Code ✅
- Format Patterns (Dates, Amounts, Responses, Errors) ✅

### Gap Analysis

**Kritische Gaps: Keine.**

**Wichtige Gaps (2, nicht blockierend):**

1. **MDX-Integration Details:** `content/docs/` definiert, aber Plugin-Wahl offen. Empfehlung: **@next/mdx** (offizielles Next.js Plugin, zero-config mit App Router). In erster Docs-Story festnageln.

2. **Neon Postgres Region:** EU-Region (Frankfurt) empfohlen für GDPR-Konformität. Genaue Neon-Projekt-Konfiguration (Connection Pooling, Branching) in Epic 2 Setup-Story festnageln.

**Kleinere Gaps (2, Phase 2):**

1. **Error Monitoring:** Kein Sentry o.ä. — Vercel Logs reichen für MVP.
2. **Accessibility Testing:** shadcn/ui WCAG 2.1 AA als Baseline reicht für MVP.

### Architecture Completeness Checklist

**✅ Requirements Analysis**
- [x] Projektkontext analysiert (38 FRs, davon 30 aktiv im Managed MVP; 17 NFRs, davon 16 aktiv)
- [x] Scale & Complexity bewertet (Medium)
- [x] Technische Constraints identifiziert (MMS Backend, Vercel, Analytics)
- [x] 7 Cross-Cutting Concerns gemappt

**✅ Architectural Decisions**
- [x] Kritische Entscheidungen dokumentiert (Auth, Routing, Proxy, Transform)
- [x] Tech Stack vollständig spezifiziert (mit Versionen)
- [x] Integration Patterns definiert (mmsApi, Route Handlers)
- [x] Performance Considerations adressiert (SSG, CDN, TanStack Query)

**✅ Implementation Patterns**
- [x] Naming Conventions etabliert (6 Kategorien + Anti-Patterns)
- [x] Structure Patterns definiert (Feature + Shared)
- [x] Communication Patterns spezifiziert (API Flow, Auth Flow)
- [x] Process Patterns dokumentiert (Error Handling, Loading, Toasts)

**✅ Project Structure**
- [x] Vollständiger Dateibaum definiert
- [x] Component Boundaries etabliert
- [x] Integration Points gemappt
- [x] Requirements → Structure Mapping komplett

### Architecture Readiness Assessment

**Overall Status:** READY FOR IMPLEMENTATION

**Confidence Level:** HIGH

**Stärken:**
- 100% Coverage aller aktiven FRs (30/38) und NFRs (16/17)
- Klare Boundaries (Server-Side only, Component Isolation, Data Transformation)
- Explizite Anti-Patterns verhindern AI-Agent-Inkonsistenzen
- Code-Beispiele für alle kritischen Patterns
- Architektur respektiert "minimale DB" Prinzip (Neon Postgres nur für Tenant API Keys + Auth Sessions, keine Business-Daten)

**Offene Ergänzungen (nicht blockierend):**
- Form Handling Pattern (React Hook Form + Zod) in Epics/Stories spezifizieren
- MDX Plugin-Wahl (@next/mdx) in erster Docs-Story festnageln
- Error Monitoring (Sentry) in Phase 2 evaluieren

### Implementation Handoff

**AI Agent Guidelines:**
- Alle architektonischen Entscheidungen exakt wie dokumentiert befolgen
- Implementation Patterns konsistent über alle Komponenten anwenden
- Projektstruktur und Boundaries respektieren
- Dieses Dokument für alle Architektur-Fragen referenzieren

**Erste Implementation-Priorität:**
```bash
npx create-next-app@latest paywatcher --typescript --tailwind --eslint --app --turbopack
```
