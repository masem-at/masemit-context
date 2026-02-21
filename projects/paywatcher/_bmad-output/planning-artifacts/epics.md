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
scopeDecisions:
  - "FR26 + FR27 (Invoicing) verschoben von Phase 1a zu Phase 2 (mit Stripe Billing)"
  - "FR8 (Self-Service Registrierung) verschoben zu Phase 2 — Managed MVP: Request Access Formular statt Signup"
  - "FR11 (Automatischer API Key bei Signup) verschoben zu Phase 2 — Tenant wird von Mario manuell angelegt"
  - "FR13 (API Key regenerieren) entfällt — kein MMS-Endpoint vorhanden"
  - "FR39–FR43 (Admin Dashboard) nachträglich ergänzt — im Product Brief v2 als Phase 1 definiert, im PRD als Phase 1b, FRs und Epic fehlten in der ursprünglichen Epic-Aufschlüsselung"
---

# paywatcher - Epic Breakdown

## Übersicht

Dieses Dokument enthält die vollständige Epic- und Story-Aufschlüsselung für PayWatcher. Es zerlegt die Anforderungen aus PRD, UX-Design und Architektur in implementierbare Stories.

**Scope-Entscheidungen:**
- FR26 (Rechnungen einsehen) und FR27 (PDF Download) wurden von Phase 1a zu Phase 2 verschoben. Begründung: Kein MMS Invoice-Endpoint existiert, keine zahlenden Kunden bei Launch, manuelle Rechnungsstellung via FreeFinance + Email für 1-3 Paid Tenants ausreichend.
- FR8 (Self-Service Registrierung) verschoben zu Phase 2 — Managed MVP: Request Access Formular statt Signup-Flow.
- FR11 (Automatischer API Key bei Signup) verschoben zu Phase 2 — Tenant wird von Mario manuell angelegt, Key wird persönlich übermittelt.
- FR13 (API Key regenerieren) entfällt komplett — kein MMS-Endpoint vorhanden. Regeneration erfolgt manuell durch Mario.

## Anforderungsinventar

### Funktionale Anforderungen

**Phase 1a (33 FRs + 3 neue FRs):**

**Landing Page & Marketing:**
- FR1: Besucher kann die PayWatcher Value Proposition (Verification-Only, Non-Custodial, Flat-Fee) auf der Landing Page verstehen
- FR2: Besucher kann interaktive Code-Beispiele (curl, JavaScript) auf der Landing Page sehen und kopieren
- FR3: Besucher kann die Pricing-Tiers (Free/Starter/Pro/Enterprise) auf der Landing Page vergleichen
- FR4: Besucher kann einen Kostenvergleich zwischen PayWatcher und Wettbewerbern einsehen
- FR5: Besucher kann den 3-Step "How It Works" Flow visuell nachvollziehen
- FR6: Besucher kann Trust Signals (Non-Custodial, Base Network, USDC, "Built by masemIT") einsehen
- FR7: Besucher kann von der Landing Page zum Request Access Formular navigieren
- FR36: Besucher kann ein Request Access Formular ausfüllen um Zugang zu beantragen (Email via Resend an contact@masem.at + automatische Lead-Erfassung via POST /v1/parties mit party_status "lead")

**Onboarding & Authentication:**
- ~~FR8: Besucher kann sich für ein PayWatcher-Konto registrieren (Self-Service, Free Tier)~~ → Phase 2
- FR9: Tenant kann sich in sein Dashboard einloggen
- FR10: Tenant kann sich aus seinem Dashboard ausloggen
- ~~FR11: Tenant erhält bei Registrierung automatisch einen API Key~~ → Phase 2
- FR37: Tenant kann sich per Magic Link (Email via Resend) im Dashboard einloggen, Tenant-Zuordnung via POST /v1/auth/login (Parties API), API Key aus Frontend-DB (Neon Postgres)

**API Key Management:**
- FR12: Tenant kann seinen API Key im Dashboard einsehen (masked)
- ~~FR13: Tenant kann seinen API Key regenerieren (alter Key wird sofort invalidiert)~~ → entfällt (kein MMS-Endpoint)
- FR14: Tenant kann die Scopes seines API Keys einsehen

**Payment Monitoring:**
- FR15: Tenant kann eine Liste seiner Payments im Dashboard einsehen
- FR16: Tenant kann Payments nach Status filtern (pending, matched, confirming, confirmed, expired, failed)
- FR17: Tenant kann Payments nach Zeitraum filtern
- FR18: Tenant kann die Payment-Liste sortieren (nach Created At aufsteigend/absteigend)
- FR19: Tenant kann durch paginierte Payment-Listen navigieren
- FR38: Tenant kann die Webhook Delivery History pro Payment einsehen

**Webhook Management:**
- FR20: Tenant kann seine Webhook-URL im Dashboard konfigurieren
- FR21: Tenant kann einen Test-Webhook senden um die Konfiguration zu prüfen
- FR22: Tenant kann den Webhook Delivery Log einsehen

**Tenant Settings:**
- FR23: Tenant kann die Default-Confirmation-Anzahl anpassen (2-20)
- FR24: Tenant kann seine Deposit Address einsehen (read-only)
- FR25: Tenant kann seine Account-Einstellungen (Email) verwalten

**Documentation:**
- FR28: Developer kann einen Quick Start Guide auf der Landing Page/Docs einsehen
- FR29: Developer kann eine Endpoint Reference (alle API Endpoints) einsehen
- FR30: Developer kann eine Webhook Event Reference (Payloads, Event Types) einsehen
- FR31: Developer kann Code-Beispiele in mehreren Sprachen (curl, JS, Python) einsehen

**Analytics & Tracking:**
- FR32: System trackt Landing Page Events (Page View, CTA Clicks, Pricing Views, Code Copies, Request Access Start/Submit)
- FR33: System sendet Analytics-Events an masemIT Analytics (analytics.masem.at)

**SEO & Discoverability:**
- FR34: Landing Page ist SEO-optimiert (Meta Tags, Open Graph, strukturierte Daten, Sitemap)
- FR35: Landing Page ist für Social Sharing optimiert (OG Images, Twitter Cards)

**Admin Dashboard:**
- FR39: Admin kann System Health einsehen (RPC, Redis, DB, Wallets, 24h Stats) via GET /v1/admin/paywatcher/health
- FR40: Admin kann eine Liste aller Tenants mit Aggregationen einsehen via GET /v1/admin/paywatcher/tenants
- FR41: Admin kann einen neuen Tenant anlegen (mit einmaliger API Key + Webhook Secret Anzeige) via POST /v1/admin/paywatcher/tenants
- FR42: Admin kann einen Tenant aktivieren/deaktivieren via PATCH /v1/admin/paywatcher/tenants/:slug
- FR43: Admin kann globale Payments über alle Tenants einsehen (mit tenant_slug Filter) via GET /v1/admin/paywatcher/payments

**Phase 2 (verschoben):**
- ~~FR8: Self-Service Registrierung~~
- ~~FR11: Automatischer API Key bei Signup~~
- ~~FR13: API Key regenerieren~~
- ~~FR26: Tenant kann seine Rechnungen/Invoices im Dashboard einsehen~~
- ~~FR27: Tenant kann Rechnungen als PDF herunterladen~~

### Nichtfunktionale Anforderungen

**Performance:**
- NFR1: Landing Page: First Contentful Paint unter 1.5 Sekunden (SSG + Vercel CDN)
- NFR2: Dashboard: API-getriebene Seiten laden Daten innerhalb von 2 Sekunden
- NFR3: ~~Signup-Flow: Registrierung bis API Key Anzeige in unter 5 Sekunden~~ → entfällt (kein Signup)
- NFR4: Code-Beispiele auf der Landing Page sind sofort kopierbar ohne Ladezeit

**Security:**
- NFR5: Alle Verbindungen laufen über HTTPS (TLS 1.2+)
- NFR6: API Keys werden gehasht gespeichert und nur bei Erstellung einmalig im Klartext angezeigt
- NFR7: Auth-Tokens (JWT Session) haben eine begrenzte Lebensdauer und werden sicher übertragen
- NFR8: Dashboard-Routen sind nur für authentifizierte Tenants zugänglich (Middleware-Check)
- NFR9: CSRF-Schutz auf allen mutierenden Endpoints
- NFR10: GDPR-konform: Privacy Policy, Datenlöschung auf Anfrage möglich

**Integration:**
- NFR11: Frontend kommuniziert ausschließlich über REST API mit MMS Backend (api.masem.at)
- NFR12: Frontend hat keinerlei direkte Datenbank-Verbindung
- NFR13: Analytics-Events werden asynchron an masemIT Analytics gesendet (kein Blocking der UI)
- NFR14: Block Explorer Links (Basescan) öffnen sich in neuem Tab

**Deployment & Operations:**
- NFR15: Deployment über Vercel Pro mit automatischem CI/CD (Git Push → Deploy)
- NFR16: Umgebungsvariablen (API URLs, Analytics Keys) sind konfigurierbar pro Environment (Dev/Staging/Prod)
- NFR17: Keine Secrets im Frontend-Bundle — alle sensitiven Operationen laufen serverseitig

### Zusätzliche Anforderungen

**Aus Architektur:**
- Starter Template: `npx create-next-app@latest paywatcher --typescript --tailwind --eslint --app --turbopack` (Next.js 16)
- Auth: Magic Link Login via Resend (Auth.js v5 Email Provider) → POST /v1/auth/login (Parties API Tenant-Resolution) → Tenant API Key aus Neon Postgres DB. JWT Session mit tenant_slug, tenant_name, contact_email und verschlüsseltem API Key
- Frontend DB: Neon Postgres (Drizzle ORM) — tenant_keys Tabelle (tenant_slug → encrypted API Key), Auth.js Sessions/Accounts
- API Proxy: Server-Side Route Handlers (/api/* → api.masem.at), API Key aus DB (via Session tenant_slug) für MMS-Calls
- Data Transformation: snake_case → camelCase Transformation im Proxy-Layer (lib/api/transform.ts)
- Client Data Fetching: TanStack Query → Route Handlers (nie direkt MMS)
- Theme: next-themes, Dark Mode Default, CSS Custom Properties für Design Tokens
- Syntax Highlighting: Shiki (Server-Side Rendering für Landing Page)
- Form Handling: React Hook Form + Zod (shadcn/ui Standard-Integration)
- MDX: @next/mdx für Docs-Lite eingebettet im Projekt
- Toast: sonner (shadcn/ui Standard)
- Route Groups: (landing), (dashboard) in einem Next.js Projekt

**Aus UX-Spezifikation:**
- Responsive: 2 Modi — Mobile (<1024px), Desktop (>=1024px). Kein separater Tablet-Breakpoint.
- Accessibility: WCAG 2.1 Level AA, Farbe + Text-Label immer zusammen, Touch Targets min. 44x44px
- Payment Status: 6-State Color Mapping (pending=muted, matched=accent, confirming=warning, confirmed=success, expired=neutral, failed=error)
- Loading: Skeleton für Dashboard-Seiten, kein Full-Page-Spinner, Sidebar + Header stehen sofort
- Toasts: sonner, Position rechts unten, Copy-Aktionen nur Inline-Checkmark (kein Toast)
- Buttons: Max 1 Primary pro Screen, Danger mit Confirmation Dialog, Copy als Icon-Only
- Monospace: Alles Kopierbare (txHashes, Adressen, API Keys, Amounts, JSON) in Geist Mono
- Zustandsabhängiger First Screen: Onboarding → Overview automatisch nach erstem confirmed Payment
- Sidebar: ~200px, text-basiert, collapsible, 4 Items (Overview, Payments, Settings, Docs)
- Farbpalette: Neutral + Teal/Grün-Akzent, Brand = Success = Confirmed ("Pavlov für Developer")
- Typografie: Geist Sans + Geist Mono via next/font/local

### FR Coverage Map

| FR | Epic | Beschreibung |
|----|------|-------------|
| FR1 | Epic 1 | Value Proposition auf Landing Page |
| FR2 | Epic 1 | Interaktive Code-Beispiele |
| FR3 | Epic 1 | Pricing-Tiers vergleichen |
| FR4 | Epic 1 | Kostenvergleich mit Wettbewerbern |
| FR5 | Epic 1 | 3-Step "How It Works" Flow |
| FR6 | Epic 1 | Trust Signals |
| FR7 | Epic 1 | CTA zum Request Access Formular |
| FR36 | Epic 1 | Request Access Formular |
| ~~FR8~~ | ~~Epic 2~~ | ~~Registrierung (Self-Service)~~ → Phase 2 |
| FR9 | Epic 2 | Login (Magic Link via Resend) |
| FR10 | Epic 2 | Logout |
| ~~FR11~~ | ~~Epic 2~~ | ~~Automatischer API Key bei Signup~~ → Phase 2 |
| FR37 | Epic 2 | Magic Link Dashboard Login + Parties Tenant-Resolution |
| FR12 | Epic 2 | API Key anzeigen (masked) |
| ~~FR13~~ | — | ~~API Key regenerieren~~ → entfällt (kein MMS-Endpoint) |
| FR14 | Epic 2 | API Key Scopes einsehen |
| FR15 | Epic 3 | Payments-Liste |
| FR16 | Epic 3 | Status-Filter |
| FR17 | Epic 3 | Zeitraum-Filter |
| FR18 | Epic 3 | Sortierung (nach Created At) |
| FR19 | Epic 3 | Pagination |
| FR38 | Epic 3 | Webhook History pro Payment |
| FR20 | Epic 4 | Webhook-URL konfigurieren |
| FR21 | Epic 4 | Test-Webhook senden |
| FR22 | Epic 4 | Webhook Delivery Log |
| FR23 | Epic 4 | Confirmation Override |
| FR24 | Epic 4 | Deposit Address anzeigen |
| FR25 | Epic 4 | Account-Einstellungen |
| FR28 | Epic 5 | Quick Start Guide |
| FR29 | Epic 5 | Endpoint Reference |
| FR30 | Epic 5 | Webhook Event Reference |
| FR31 | Epic 5 | Code-Beispiele (curl, JS, Python) |
| FR32 | Epic 1 | Landing Page Event Tracking |
| FR33 | Epic 1 | Analytics an masemIT senden |
| FR34 | Epic 1 | SEO-Optimierung |
| FR35 | Epic 1 | Social Sharing |
| FR39 | Epic 6 | System Health einsehen (Admin) |
| FR40 | Epic 6 | Tenant-Liste mit Aggregationen (Admin) |
| FR41 | Epic 6 | Tenant anlegen (Admin) |
| FR42 | Epic 6 | Tenant aktivieren/deaktivieren (Admin) |
| FR43 | Epic 6 | Globale Payments aller Tenants (Admin) |

## Epic-Liste

### Epic 1: Project Foundation & Landing Page
Ein Besucher kann PayWatcher entdecken, die Value Proposition verstehen, Code-Beispiele sehen, Pricing vergleichen, und über ein Request Access Formular Zugang beantragen. Enthält Project Scaffold (Greenfield) und Theme System als Basis.
**FRs covered:** FR1, FR2, FR3, FR4, FR5, FR6, FR7, FR32, FR33, FR34, FR35, FR36

### Epic 2: Authentication, Frontend DB & API Key Management
Ein Tenant kann sich per Magic Link einloggen (Resend), wird via Parties API dem Tenant zugeordnet (POST /v1/auth/login), und sein API Key wird aus der Frontend-DB (Neon Postgres) geladen. Dashboard Shell, Key/Scopes einsehen — Managed MVP ohne Self-Service Signup.
**FRs covered:** FR9, FR10, FR12, FR14, FR37

### Epic 3: Payment Monitoring Dashboard
Ein Tenant kann seine Payments im Dashboard einsehen, nach Status/Zeitraum filtern, sortieren, durch paginierte Listen navigieren, und die Webhook History pro Payment einsehen. Neue User sehen den Onboarding-Screen.
**FRs covered:** FR15, FR16, FR17, FR18, FR19, FR38

### Epic 4: Webhook Configuration & Tenant Settings
Ein Tenant kann seine Webhook-URL konfigurieren, testen, den Delivery Log einsehen, Confirmations anpassen, und seine Account-Einstellungen verwalten.
**FRs covered:** FR20, FR21, FR22, FR23, FR24, FR25

### Epic 5: Developer Documentation (Docs-Lite)
Ein Developer kann Quick Start Guide, Endpoint Reference, Webhook Events und Code-Beispiele lesen — alles was nötig ist um PayWatcher erfolgreich zu integrieren.
**FRs covered:** FR28, FR29, FR30, FR31

### Epic 6: Admin Dashboard
Mario (Operator) kann den PayWatcher Service überwachen, Tenants managen, globale Payments einsehen und System Health prüfen — alles was für den operativen Betrieb nötig ist.
**FRs covered:** FR39, FR40, FR41, FR42, FR43

## Epic 1: Project Foundation & Landing Page

Ein Besucher kann PayWatcher entdecken, die Value Proposition verstehen, Code-Beispiele sehen, Pricing vergleichen, und über ein Request Access Formular Zugang beantragen.

**Status:** DONE (Stories 1.1–1.6 implementiert), Story 1.7 ist BACKLOG.

### Story 1.1: Project Scaffold & Vercel Deployment ✅ DONE

As a Developer (Sempre),
I want ein initialisiertes Next.js 16 Projekt mit Tailwind v4, TypeScript, App Router und Vercel-Deployment,
So that ich eine lauffähige Basis habe auf der alle weiteren Features aufbauen.

**Acceptance Criteria:**

**Given** kein Projekt existiert
**When** `npx create-next-app@latest paywatcher --typescript --tailwind --eslint --app --turbopack` ausgeführt wird
**Then** ein Next.js 16 Projekt mit App Router, Tailwind v4, TypeScript strict und ESLint ist erstellt
**And** die Projektstruktur folgt dem Architecture Doc (src/app, src/components, src/lib, src/types, src/hooks, src/styles)

**Given** das Projekt ist initialisiert
**When** shadcn/ui via `npx shadcn@latest init` eingerichtet wird
**Then** die components.json ist konfiguriert und erste Basis-Komponenten (Button, Badge, Card) sind installiert

**Given** das Projekt ist auf GitHub gepusht
**When** Vercel-Deployment konfiguriert wird
**Then** die App ist unter einer Preview-URL erreichbar und automatisches CI/CD (Git Push → Deploy) funktioniert

**Given** ein `.env.example` existiert
**When** ein Entwickler die Environment Variables prüft
**Then** sind folgende Keys dokumentiert: MMS_API_URL, MMS_API_KEY, NEXTAUTH_SECRET, NEXTAUTH_URL, RESEND_API_KEY, ANALYTICS_URL, ANALYTICS_KEY

### Story 1.2: Theme System & Design Tokens ✅ DONE

As a Developer (Sempre),
I want das PayWatcher Design System mit Dark/Light Mode, Farbpalette und Typografie eingerichtet,
So that alle nachfolgenden Components konsistente visuelle Tokens verwenden.

**Acceptance Criteria:**

**Given** das Projekt aus Story 1.1 existiert
**When** die globals.css mit Tailwind v4 @theme Directive und CSS Custom Properties konfiguriert wird
**Then** sind alle Design Tokens definiert: Background, Surface, Text Primary/Secondary, Accent (Teal ~#00D4AA), Success, Warning, Error, Neutral

**Given** next-themes installiert ist
**When** der ThemeProvider im Root Layout eingebunden wird
**Then** ist Dark Mode als Default aktiv und ein Theme Toggle wechselt zwischen Dark/Light Mode
**And** die Präferenz wird persistent gespeichert (localStorage/Cookie)

**Given** Geist Fonts via next/font/local konfiguriert sind
**When** die Seite geladen wird
**Then** wird Geist Sans für UI-Text und Geist Mono für Code/kopierbare Inhalte verwendet
**And** die Type Scale aus der UX-Spec ist implementiert (display, h1, h2, h3, body, body-sm, caption, code, code-sm, stat)

**Given** Payment Status Farben definiert sind
**When** ein Badge für einen der 6 Status gerendert wird
**Then** zeigt er die korrekte Farbe: pending=muted, matched=accent, confirming=warning, confirmed=success, expired=neutral, failed=error

### Story 1.3: Landing Page Layout, Hero Section & Problem/Solution ✅ DONE

As a Besucher von paywatcher.dev,
I want eine Landing Page mit Hero Section die mir sofort zeigt was PayWatcher ist und was es löst,
So that ich in unter 30 Sekunden verstehe ob PayWatcher für mich relevant ist.

**Acceptance Criteria:**

**Given** ich navigiere zu paywatcher.dev
**When** die Seite lädt
**Then** sehe ich einen Landing Header mit Navigation und "Request Access"-CTA
**And** die Seite ist via SSG pre-rendered (kein Client-Side Rendering)

**Given** die Hero Section geladen ist
**When** ich den Hero-Bereich sehe
**Then** sehe ich die Headline, eine Subline für beide Zielgruppen (Stripe-Devs + Web3-Native), ein prominentes Code-Snippet (curl Payment Intent), und einen CTA-Button ("Request Access — Free")
**And** der Code-Block hat Syntax Highlighting (Shiki) und einen Copy-Button (FR2 teilweise)

**Given** ich scrolle weiter
**When** die Problem/Solution Section sichtbar wird
**Then** sehe ich die Darstellung des Problems (1% Fees ODER alles selbst bauen) und PayWatcher's Lösung als "missing middle" (FR1)

**Given** ich auf "Request Access" klicke (FR7)
**When** der CTA ausgelöst wird
**Then** scrolle ich zum Request-Access-Formular oder ein Modal öffnet sich

**Given** die Landing Page auf einem Mobilgerät (<1024px) geladen wird
**When** die Seite rendert
**Then** ist das Layout responsive: Code-Block mit horizontal scroll, Header mit Hamburger-Menü, volle Breite

**Given** ein Landing Footer existiert
**When** ich zum Seitenende scrolle
**Then** sehe ich Footer-Links und "by masemIT"

### Story 1.4: How It Works & Interactive Code Examples ✅ DONE

As a Besucher,
I want den 3-Step "How It Works" Flow und interaktive Code-Beispiele sehen und kopieren können,
So that ich verstehe wie einfach die PayWatcher-Integration ist.

**Acceptance Criteria:**

**Given** ich scrolle zur "How It Works" Section (FR5)
**When** die Section sichtbar ist
**Then** sehe ich eine visuelle 3-Step-Darstellung: (1) Create Payment Intent → (2) Customer Pays → (3) Webhook Fires
**And** die Animation respektiert prefers-reduced-motion

**Given** ich scrolle zur Code Examples Section (FR2)
**When** die Section sichtbar ist
**Then** sehe ich Code-Beispiele mit Tabs für mindestens curl und JavaScript
**And** jedes Code-Beispiel hat Syntax Highlighting (Shiki, server-side rendered) und einen Copy-Button
**And** der Copy-Button zeigt Inline-Checkmark-Feedback (kein Toast)

**Given** ich klicke auf den Copy-Button eines Code-Beispiels
**When** der Code in die Zwischenablage kopiert wird
**Then** wechselt das Clipboard-Icon für 1.5s zu einem Checkmark und kehrt dann zurück

**Given** die Seite auf Mobile geladen wird
**When** ein Code-Block breiter als der Viewport ist
**Then** wird horizontal gescrollt (kein Umbruch), mit dezentem Scroll-Indicator

### Story 1.5: Pricing, Comparison & Trust Signals ✅ DONE

As a Besucher,
I want Pricing-Tiers vergleichen, den Kostenunterschied zu Wettbewerbern sehen, und Vertrauenssignale erkennen,
So that ich eine informierte Entscheidung treffen kann ob PayWatcher für mich passt.

**Acceptance Criteria:**

**Given** ich scrolle zur Pricing Section (FR3)
**When** die Pricing-Karten sichtbar sind
**Then** sehe ich 4 Tiers: Free, Starter ($29), Pro ($99), Enterprise (Custom)
**And** jeder Tier zeigt Features, Limits und einen CTA
**And** auf Desktop sind die Karten nebeneinander, auf Mobile vertikal gestackt

**Given** ich scrolle zur Comparison Section (FR4)
**When** der Vergleich sichtbar ist
**Then** sehe ich einen prominenten Kostenvergleich ("$100 at Coinbase Commerce vs. $0.05 at PayWatcher for a $10k payment")
**And** der Vergleich ist als visuelle Unterbrechung gestaltet (Callout-Stil)

**Given** ich scrolle zur Trust Signals Section (FR6)
**When** die Section sichtbar ist
**Then** sehe ich Trust Signals: Non-Custodial, Base Network, USDC, "Built by masemIT"
**And** keine Stock-Fotos oder generische Trust-Badges

**Given** jede Section einen CTA hat
**When** ich an einem beliebigen Punkt auf "Request Access" oder "Start Free" klicke
**Then** scrolle ich zum Request-Access-Formular oder ein Modal öffnet sich

### Story 1.6: SEO, Social Sharing & Analytics Integration ✅ DONE

As a Betreiber (Sempre),
I want dass die Landing Page SEO-optimiert ist, Social Sharing unterstützt, und Analytics-Events trackt,
So that PayWatcher organisch gefunden wird und ich Conversion-Daten habe.

**Acceptance Criteria:**

**Given** die Landing Page existiert (FR34)
**When** ein Suchmaschinen-Crawler die Seite besucht
**Then** findet er korrekte Meta Tags (title, description), Open Graph Tags (og:title, og:description, og:image), und strukturierte Daten
**And** eine dynamische Sitemap unter /sitemap.xml existiert
**And** eine robots.txt unter /robots.txt existiert

**Given** jemand die Landing Page URL in Social Media teilt (FR35)
**When** die Plattform die OG-Tags liest
**Then** wird ein PayWatcher-OG-Image, Titel und Beschreibung angezeigt
**And** Twitter Cards sind konfiguriert

**Given** ein Besucher die Landing Page navigiert (FR32, FR33)
**When** er interagiert (Page View, CTA Click, Pricing Section View, Code Copy, Request Access Start/Submit)
**Then** werden Analytics-Events asynchron an masemIT Analytics (analytics.masem.at) gesendet
**And** die Events blockieren nicht die UI (NFR13)
**And** Events werden nur in Production gesendet (nicht in Dev/Preview)

**Given** die Landing Page Performance gemessen wird (NFR1)
**When** ein Lighthouse-Test läuft
**Then** ist der First Contentful Paint unter 1.5 Sekunden (SSG + Vercel CDN)

### Story 1.7: Request Access Formular

As a Besucher,
I want ein Request-Access-Formular ausfüllen können,
So that ich Zugang zu PayWatcher beantragen kann.

**Acceptance Criteria:**

**Given** ich bin auf der Landing Page
**When** ich auf "Request Access" klicke
**Then** sehe ich ein Formular mit: Name (required), Email (required), Company/Project Name (optional), Use Case (optional)

**Given** ich fülle das Formular aus
**When** ich Submit klicke
**Then** wird eine Email via Resend an contact@masem.at gesendet mit allen Formulardaten

**Given** das Formular erfolgreich abgesendet wurde
**Then** sehe ich: "Thanks! We'll get back to you within 24 hours with your API credentials."

**Given** die Landing Page CTAs
**When** ich "Request Access" klicke in Hero/Pricing/Trust Signals
**Then** scrolle ich zum Request-Access-Formular oder ein Modal öffnet sich

- Kein MMS-Call. Kein Account wird erstellt. Nur Email an Mario.
- React Hook Form + Zod für Validierung
- Analytics Event: `lp_request_access_start`, `lp_request_access_submit`

## Epic 2: Authentication, Frontend DB & API Key Management

Ein Tenant kann sich per Magic Link (Resend) einloggen, wird via Parties API dem Tenant zugeordnet (POST /v1/auth/login), und sein API Key wird aus der Frontend-DB (Neon Postgres) geladen. Dashboard Shell, Key/Scopes einsehen — Managed MVP ohne Self-Service Signup.

**Status:** BACKLOG (komplett neu geschrieben für Magic Link + Parties + DB)

### Story 2.1: Dashboard Shell & Route Protection

As a authentifizierter Tenant,
I want ein Dashboard-Layout mit Sidebar-Navigation und geschützten Routen,
So that ich nach dem Login eine konsistente Oberfläche für alle Dashboard-Features habe.

**Acceptance Criteria:**

**Given** ein authentifizierter User das Dashboard aufruft
**When** das Layout rendert
**Then** sehe ich eine linke Sidebar (~200px, text-basiert) mit den Items: Overview, Payments, Settings, Docs
**And** einen Content-Bereich rechts der Sidebar
**And** einen Header mit Theme Toggle und User-Info (Tenant Name)

**Given** die Sidebar auf Desktop (>=1024px) sichtbar ist
**When** ich den Collapse-Chevron klicke
**Then** kollabiert die Sidebar auf ~60px (Icon-Only)

**Given** das Dashboard auf Mobile (<1024px) geladen wird
**When** die Seite rendert
**Then** ist die Sidebar versteckt und ein Hamburger-Icon oben links öffnet sie als Sheet/Overlay

**Given** ein nicht-authentifizierter User eine Dashboard-Route aufruft
**When** die Middleware den Auth-Check durchführt (NFR8)
**Then** wird der User zur Login-Seite redirected

**Given** die Neon Postgres DB eingerichtet wird
**When** Drizzle ORM konfiguriert wird
**Then** existiert `lib/db/index.ts` (Neon Serverless Connection), `lib/db/schema.ts` (tenant_keys Tabelle: tenant_slug, encrypted_api_key, created_at, updated_at), und `drizzle.config.ts`
**And** Auth.js v5 nutzt den Drizzle Adapter für Sessions/Accounts
**And** `lib/api/tenant-keys.ts` bietet `getTenantApiKey(tenantSlug)` und `setTenantApiKey(tenantSlug, apiKey)` Funktionen

**Given** das API Proxy Pattern implementiert ist
**When** ein Route Handler aufgerufen wird
**Then** kommuniziert er via mmsApi() mit dem MMS Backend (api.masem.at) unter Verwendung des API Keys aus der DB (via tenant_slug aus Session)
**And** Responses werden von snake_case zu camelCase transformiert (wo nötig — Payment-Responses sind bereits camelCase)
**And** das Envelope-Format { data } / { error } wird eingehalten

- Kein /signup Route — nur /login existiert
- /signup redirected auf /login
- DATABASE_URL als Environment Variable (Neon Postgres Connection String)

### Story 2.2: Magic Link Login & Tenant-Resolution

As a Tenant,
I want mich per Magic Link (Email) im Dashboard einloggen können,
So that ich ohne API Key oder Passwort einfach per Email-Klick auf mein Dashboard zugreifen kann.

**Acceptance Criteria:**

**Given** ich bin auf der Login-Seite (/login)
**When** die Seite lädt
**Then** sehe ich ein Eingabefeld für meine Email-Adresse mit Placeholder "your@email.com" und einen "Send Magic Link" Button

**Given** ich gebe meine Email ein und klicke "Send Magic Link"
**When** Auth.js den Email Provider triggert
**Then** wird ein Magic Link via Resend an meine Email gesendet
**And** ich sehe: "Check your email — we sent you a sign-in link."

**Given** ich klicke den Magic Link in meiner Email
**When** Auth.js den Link verifiziert
**Then** ruft der signIn Callback POST /v1/auth/login mit `{ email, module: "paywatcher" }` auf
**And** die Response enthält: `{ party_id, display_name, tenants: [{ tenant_slug, role }] }`

**Given** der User genau einem Tenant zugeordnet ist (tenants.length === 1)
**Then** wird der tenant_slug automatisch in die Session geschrieben
**And** der API Key wird aus der Neon Postgres DB geladen (tenant_keys Tabelle, verschlüsselt)
**And** ich werde zum Dashboard redirected

**Given** der User mehreren Tenants zugeordnet ist (tenants.length > 1)
**Then** sehe ich eine Tenant-Auswahl-Seite (/auth/resolve) mit den verfügbaren Tenants
**And** nach Auswahl wird der tenant_slug in die Session geschrieben und der API Key aus der DB geladen

**Given** die Email keinem Tenant zugeordnet ist (tenants leer oder 404)
**Then** sehe ich: "No PayWatcher account found for this email. Request access first."
**And** ein Link zum Request Access Formular

**Given** ich nach dem Login eine Dashboard-Route aufrufe
**Then** enthält die Session tenant_slug, tenant_name, contact_email und den verschlüsselten API Key für MMS-Calls

- Magic Link via Resend (Auth.js v5 Email Provider)
- Tenant-Resolution via POST /v1/auth/login (Parties API)
- API Key aus Neon Postgres DB (tenant_keys Tabelle) — nie im Browser
- /signup redirected auf /login
- Logout löscht die Session und redirected zur Landing Page

### Story 2.3: Welcome-Screen & Onboarding

As a Tenant nach dem ersten Login,
I want einen Welcome-Screen sehen der mir die wichtigsten Infos zeigt,
So that ich sofort weiß wie ich starten kann.

**Acceptance Criteria:**

**Given** ich logge mich zum ersten Mal ein
**When** das Dashboard lädt
**Then** sehe ich den Welcome-Screen mit: API Key (masked, aus GET /v1/paywatcher/config/api-key), Deposit Address (aus Config), Webhook URL (aus Config, oder "not configured")

**Given** der Welcome-Screen angezeigt wird
**Then** sehe ich CTAs: "Configure Webhook" → Settings, "View API Docs" → Docs

**Given** ich habe den Welcome-Screen gesehen
**When** ich das Dashboard später besuche
**Then** sehe ich die reguläre Overview-Seite (localStorage-Flag "onboarding_complete")

- Kein auto-provisioning. Der Tenant existiert bereits (von Mario angelegt).

### Story 2.4: API Key Anzeige & Scopes

As a Tenant,
I want meinen API Key (masked) und dessen Scopes im Dashboard einsehen können,
So that ich weiß welchen Key ich verwende und welche Berechtigungen er hat.

**Acceptance Criteria:**

**Given** ich bin auf der Settings/API Keys Seite
**When** die Seite lädt
**Then** sehe ich meinen API Key masked (z.B. "mms_paywat****...****") via GET /v1/paywatcher/config/api-key

**Given** der API Key angezeigt wird
**Then** sehe ich die Scopes als Badges (z.B. payments:read, payments:write)

**Given** der API Key angezeigt wird
**Then** sehe ich das Erstellungsdatum (created_at)

**Given** der Copy-Button geklickt wird
**Then** wird der masked Key kopiert (Inline-Checkmark, kein Toast)

- KEIN Regenerate-Button — kein MMS-Endpoint dafür. Regeneration erfolgt manuell durch Mario.
- Proxy Route: GET /api/api-keys → GET /v1/paywatcher/config/api-key (Response: { masked_key, scopes, created_at })

## Epic 3: Payment Monitoring Dashboard

Ein Tenant kann seine Payments im Dashboard einsehen, nach Status/Zeitraum filtern, sortieren, durch paginierte Listen navigieren, und die Webhook History pro Payment einsehen. Neue User sehen den Onboarding-Screen.

**Status:** BACKLOG

### Story 3.1: Payments Data Layer & PaymentStatusBadge

As a Developer (Sempre),
I want Payment-Typen, API-Proxy-Routen, TanStack Query Hooks und die PaymentStatusBadge-Komponente,
So that alle nachfolgenden Payment-Features eine stabile Datengrundlage und die zentrale Status-Darstellung haben.

**Acceptance Criteria:**

**Given** die Dashboard Shell aus Epic 2 existiert
**When** die Payment-Typen definiert werden
**Then** existiert `types/payment.ts` mit: `PaymentStatus` (Union Type: `'pending' | 'matched' | 'confirming' | 'confirmed' | 'expired' | 'failed'`), `Payment` Interface (id, amount, exactAmount, status, txHash, depositAddress, confirmations, confirmationsRequired, currency, chain, metadata, expiresAt, createdAt, updatedAt), `PaymentFilters` Interface
**And** `PAYMENT_STATUSES` Konstante mit Farb- und Label-Mapping gemäß UX-Spec

**Given** der API Proxy Pattern aus Story 2.1 existiert (mmsApi, API Key aus Session)
**When** die Payments-Proxy-Routen erstellt werden
**Then** existiert `app/api/payments/route.ts` (GET → `api.masem.at/v1/payments` mit Query-Params Weiterleitung)
**And** existiert `app/api/payments/[id]/route.ts` (GET → `api.masem.at/v1/payments/:id`)
**And** das Envelope-Format `{ data }` / `{ data, meta }` wird eingehalten
**And** Payment-Responses sind bereits camelCase vom MMS Backend — keine Transformation nötig

**Given** die Proxy-Routen existieren
**When** TanStack Query Hooks erstellt werden
**Then** existiert `usePayments(filters?: PaymentFilters)` mit Query Key `['payments', filters]`
**And** existiert `usePayment(id: string)` mit Query Key `['payments', id]`
**And** `staleTime` ist 30 Sekunden für Payments

**Given** das STATUS_CONFIG Mapping definiert ist
**When** eine PaymentStatusBadge gerendert wird
**Then** zeigt sie den korrekten farbigen Badge: pending=muted, matched=accent, confirming=warning, confirmed=success/accent, expired=neutral/grau, failed=error/rot
**And** der Badge zeigt immer Farbe + Text-Label zusammen (nie nur Farbe — Accessibility)
**And** der Badge hat zwei Größen: `sm` (Tabelle) und `md` (Detail Header)

**Given** ein Amount-Wert aus der API kommt (String, 6 Dezimalen)
**When** der Betrag formatiert werden muss
**Then** existiert ein `formatAmount()` Helper der "49.000042 USDC" korrekt formatiert
**And** Amounts werden in Geist Mono dargestellt

### Story 3.2: Overview Page & Zustandsabhängiger First Screen

As a Tenant,
I want beim Dashboard-Einstieg entweder einen Onboarding-Screen (wenn ich neu bin) oder einen 5-Sekunden Health Check (wenn ich Payments habe) sehen,
So that ich als neuer User sofort loslegen kann und als wiederkehrender User in Sekunden weiß ob alles läuft.

**Acceptance Criteria:**

**Given** ein neu ongeboardeter Tenant das Dashboard öffnet (0 Payments)
**When** die Overview-Seite lädt
**Then** sehe ich den OnboardingScreen mit:
- API Key (masked, mit Copy-Button, aus GET /v1/paywatcher/config/api-key)
- curl-Beispiel zum Erstellen eines Payment Intent (mit vorausgefülltem API Key), Copy-Button
- Deposit Address + erwarteter Betrag (Platzhalter bis Payment erstellt)
- "Waiting for first payment..." mit Spinner
- Links zu "Full Docs" und "Webhook Settings"

**Given** der Onboarding-Screen aktiv ist
**When** der Tenant über seine API ein Payment erstellt und es den Status `confirmed` erreicht
**Then** wechselt der Spinner zu einem grünen Haken ("First payment verified!")
**And** nach kurzer Verzögerung wechselt der Screen automatisch zur Overview-Ansicht
**And** kein "Dismiss"- oder "Skip"-Button existiert — der Wechsel ist systemgesteuert

**Given** ein Tenant mit bestehenden Payments das Dashboard öffnet
**When** die Overview-Seite lädt
**Then** sehe ich StatCards: Confirmed (success), Expired (neutral), Failed (error) — Zahlen der letzten 7 Tage
**And** eine "Recent Payments"-Liste mit den letzten ~5 Payments (PaymentStatusBadge + Amount + ID + Timestamp)
**And** eine HealthBar am unteren Rand (Webhook Health %, API Calls, Avg Confirmation Time)

**Given** die Overview-Daten laden
**When** die API-Response noch aussteht
**Then** sehe ich Skeleton-Placeholders: Card-Shapes für StatCards, Zeilen-Shapes für Recent Payments
**And** Sidebar + Header stehen sofort (kein Full-Page-Spinner)

**Given** der Onboarding-Screen auf Mobile (<1024px) angezeigt wird
**When** der Screen rendert
**Then** ist das Layout einspaltig, Code-Block mit horizontal Scroll, alle Touch Targets min. 44x44px

### Story 3.3: Payments List Page & DataTable

As a Tenant,
I want eine vollständige Liste meiner Payments in einer Tabelle sehen können,
So that ich alle Payments im Überblick habe und schnell den Status jeder Zahlung erfasse.

**Acceptance Criteria:**

**Given** ich klicke auf "Payments" in der Sidebar (FR15)
**When** die Payments-Seite lädt
**Then** sehe ich eine Tabelle mit den Spalten: Payment ID (monospace, truncated), Amount (monospace, USDC), Status (PaymentStatusBadge), txHash (monospace, truncated, Copy-Button), Created (Timestamp)
**And** die Tabelle nutzt volle Breite (minus Sidebar) — kein max-width

**Given** die Payments-Tabelle auf Desktop (>=1024px) gerendert wird
**When** ein Payment in der Tabelle angezeigt wird
**Then** sind Payment ID und txHash als monospace (Geist Mono) dargestellt und truncated (z.B. `pay_7f2a...` / `0x7a3f...8b2c`)
**And** jeder txHash hat einen Copy-Button (Inline-Checkmark, kein Toast)
**And** txHash ist als Link zu Basescan klickbar (öffnet in neuem Tab, NFR14)

**Given** die Payments-Tabelle auf Mobile (<1024px) gerendert wird
**When** die Seite rendert
**Then** wechselt die Darstellung zu einem Card-Layout: Pro Payment eine Card mit Status-Badge + Amount + Payment ID + Timestamp
**And** kein horizontaler Scroll der Tabelle (Cards statt Tabelle)

**Given** die Payments-Daten laden
**When** die API-Response noch aussteht
**Then** sehe ich Skeleton-Zeilen (Desktop) bzw. Skeleton-Cards (Mobile) als Loading-State

**Given** keine Payments im gewählten Zeitraum existieren
**When** die leere Liste angezeigt wird
**Then** sehe ich einen dezenten Hinweis: "No payments in this period." (kein großes Illustration-Empty-State)

**Given** Payments mit Status `pending`, `matched` oder `confirming` in der Liste sind
**When** die Seite aktiv ist
**Then** werden die Daten via TanStack Query alle 10 Sekunden automatisch refreshed (`refetchInterval: 10_000`)
**And** der Refresh blockiert nicht die UI und zeigt keinen Loading-State

### Story 3.4: Payment Filters, Sorting & Pagination

As a Tenant,
I want Payments nach Status und Zeitraum filtern, sortieren und durch Seiten blättern können,
So that ich bei vielen Payments schnell die relevanten finde.

**Acceptance Criteria:**

**Given** ich bin auf der Payments-Seite (FR16)
**When** ich den Status-Filter verwende
**Then** kann ich nach einem oder mehreren der 6 Status filtern: pending, matched, confirming, confirmed, expired, failed
**And** die Tabelle wird sofort aktualisiert
**And** der aktive Filter ist visuell erkennbar

**Given** ich bin auf der Payments-Seite (FR17)
**When** ich den Zeitraum-Filter verwende
**Then** kann ich aus vordefinierten Zeiträumen wählen: 7d, 30d, 90d, All
**And** die Tabelle wird sofort aktualisiert
**And** der Default-Zeitraum ist "30d"

**Given** ich bin auf der Payments-Seite (FR18)
**When** ich auf die Spaltenüberschrift "Created" klicke
**Then** wird die Liste nach Created At sortiert
**And** ein Klick togglet zwischen aufsteigend und absteigend
**And** ein Sortier-Indikator (Pfeil) zeigt die aktive Sortierung und Richtung
**And** HINWEIS: Die MMS API unterstützt NUR Sortierung nach `created_at` — keine Sortierung nach Amount oder Status möglich

**Given** mehr Payments existieren als auf einer Seite passen (FR19)
**When** die Pagination angezeigt wird
**Then** sehe ich Prev/Next-Buttons und die aktuelle Seitenzahl
**And** die Navigation funktioniert via Ghost-Buttons

**Given** ich Filter, Sortierung oder Pagination ändere
**When** die URL aktualisiert wird
**Then** werden die Filter-Parameter als Query-Params in der URL gespeichert (z.B. `?status=confirmed&period=7d&order=desc&page=2`)
**And** die URL ist teilbar/bookmarkbar — beim Öffnen werden die Filter wiederhergestellt

**Given** die gesamte Payments-Seite mit Filtern und Pagination geladen wird (NFR2)
**When** die Daten vom MMS Backend geholt werden
**Then** dauert der Datenabruf maximal 2 Sekunden

**Given** die Filter-Leiste auf Mobile (<1024px) angezeigt wird
**When** die Seite rendert
**Then** sind die Filter als kompakte Dropdowns/Selects dargestellt und Touch-freundlich (min. 44x44px)

### Story 3.5: Payment Detail & Webhook History

As a Tenant,
I want ein einzelnes Payment im Detail sehen können inklusive Webhook Delivery History,
So that ich den Status und die Webhook-Zustellung debuggen kann.

**Acceptance Criteria:**

**Given** ich klicke auf ein Payment in der Liste
**When** die Detail-Seite lädt
**Then** sehe ich alle Payment-Felder: id, status, amount, exactAmount, currency, chain, depositAddress, confirmations, confirmationsRequired, txHash, metadata, expiresAt, createdAt, updatedAt

**Given** ein txHash vorhanden ist
**Then** ist er als Link zu Basescan klickbar (neuer Tab)

**Given** die Payment-Detail-Seite geladen ist
**Then** sehe ich eine Webhook Delivery History via GET /v1/payments/:id/webhooks mit: Event Name, Attempt Number, Response Code (farbig: 2xx=grün, 4xx/5xx=rot), Error Message (falls vorhanden), Timestamp

**Given** keine Webhooks für dieses Payment existieren
**Then** sehe ich: "No webhooks sent yet."

- Proxy Route: GET /api/payments/[id]/webhooks → GET /v1/payments/:id/webhooks
- HINWEIS: Webhook History Response nutzt snake_case (event, attempt_number, response_code, error, created_at) — Transformation zu camelCase im Proxy-Layer nötig

## Epic 4: Webhook Configuration & Tenant Settings

Ein Tenant kann seine Webhook-URL konfigurieren, testen, den Delivery Log einsehen, Confirmations anpassen, und seine Account-Einstellungen verwalten.

**Status:** BACKLOG

### Story 4.1: Settings Layout & Webhook URL Configuration

As a Tenant,
I want meine Webhook-URL im Dashboard konfigurieren können,
So that PayWatcher mir automatisch Benachrichtigungen bei Payment-Statusänderungen sendet.

**Acceptance Criteria:**

**Given** ich klicke auf "Settings" in der Sidebar
**When** die Settings-Seite lädt
**Then** sehe ich eine Settings-Navigation mit den Bereichen: Webhooks, API Keys und Account
**And** der Content-Bereich ist max-w-3xl und zentriert
**And** ich werde standardmäßig zum Webhooks-Bereich geleitet

**Given** die Tenant-/Webhook-Typen definiert werden
**When** die Types erstellt werden
**Then** existiert `types/tenant.ts` mit: `TenantConfig` (tenant_slug, tenant_name, contact_email, tier, webhook_url, confirmation_override, deposit_address, enabled, created_at, updated_at)
**And** API-Proxy-Routen existieren: `app/api/config/route.ts` (GET, PATCH → PATCH /v1/paywatcher/config)
**And** TanStack Query Hooks existieren: `useConfig()` mit `staleTime: 5min`
**And** HINWEIS: Kein separater webhook-config Proxy-Endpoint — webhook_url ist Teil der Config (PATCH /v1/paywatcher/config)

**Given** keine Webhook-URL konfiguriert ist
**When** die Webhook-Settings-Seite angezeigt wird
**Then** sehe ich ein gelbes Info-Banner: "No webhook URL configured. Payments are processed normally, but you won't receive notifications."
**And** ein leeres URL-Input-Feld mit Placeholder "https://your-app.com/webhook"
**And** der "Send Test Webhook"-Button ist disabled

**Given** ich gebe eine Webhook-URL ein (FR20)
**When** ich auf "Save" klicke
**Then** wird die URL via React Hook Form + Zod validiert (muss HTTPS sein, gültige URL)
**And** bei Validierungsfehler erscheint Inline-Error unter dem Feld: "URL must use HTTPS" (rot, caption-size)
**And** der Fehler verschwindet bei erneutem Tippen

**Given** ich eine gültige URL eingebe und "Save" klicke
**When** die URL gespeichert wird
**Then** wird die URL via Proxy an PATCH /v1/paywatcher/config mit `{ webhook_url: "..." }` gesendet
**And** ein Toast erscheint: "Webhook URL saved" (rechts unten, 3s auto-dismiss)
**And** das Info-Banner verschwindet
**And** der "Send Test Webhook"-Button wird aktiv

**Given** bereits eine Webhook-URL konfiguriert ist
**When** die Settings-Seite lädt
**Then** sehe ich die bestehende URL im Input-Feld (aus GET /v1/paywatcher/config Response)
**And** ich kann die URL ändern und erneut speichern

### Story 4.2: Test Webhook & Delivery Log

As a Tenant,
I want einen Test-Webhook an meine URL senden und den Delivery Log einsehen können,
So that ich sicher sein kann dass meine Webhook-Integration funktioniert und Probleme debuggen kann.

**Acceptance Criteria:**

**Given** eine Webhook-URL ist konfiguriert (FR21)
**When** ich auf "Send Test Webhook" klicke
**Then** wird ein Test-Webhook via Proxy an POST /v1/paywatcher/config/test-webhook gesendet
**And** der Button zeigt einen Loading-State während der Request läuft

**Given** der Test-Webhook erfolgreich zugestellt wird (success: true in Response)
**When** die Response angezeigt wird
**Then** sehe ich: "Test webhook delivered successfully", Response Status (z.B. "200 OK"), und die Webhook URL
**And** ein Toast: "Test webhook sent — 200 OK" (grün, 3s auto-dismiss)

**Given** der Test-Webhook fehlschlägt (success: false in Response)
**When** die Response angezeigt wird
**Then** sehe ich: "Webhook delivery failed", Response Status (z.B. "500 Internal Server Error") oder Error Message (z.B. "Connection refused"), und einen hilfreichen Hint: "Check that your endpoint is reachable and returns 2xx."
**And** ein Toast: "Test webhook failed" (rot, bleibt bis dismissed)

**Given** keine Webhook-URL konfiguriert ist
**When** die Settings-Seite angezeigt wird
**Then** ist der "Send Test Webhook"-Button disabled
**And** kein Test kann gesendet werden

**Given** ein Delivery Log auf der Settings-Seite angezeigt werden soll (FR22)
**When** der Delivery Log geladen wird
**Then** werden die letzten N Payments abgerufen und deren Webhook-Histories via GET /v1/payments/:id/webhooks aggregiert
**And** jede Zeile zeigt: Event Name (z.B. `payment.confirmed`), Status Code (farbig: 2xx=grün, 4xx/5xx=rot, null=grau), Timestamp, und Attempt-Nummer
**And** HINWEIS: Es gibt keinen globalen Webhook-Delivery-Log-Endpoint — der Log wird pro-Payment via GET /v1/payments/:id/webhooks zusammengestellt

**Given** der Delivery Log leer ist
**When** die Liste angezeigt wird
**Then** sehe ich: "No webhooks sent yet."

**Given** die Delivery Log Daten laden
**When** die API-Response noch aussteht
**Then** sehe ich Skeleton-Zeilen als Loading-State

### Story 4.3: Tenant Configuration & Account Settings

As a Tenant,
I want meine Confirmation-Anzahl anpassen, meine Deposit Address sehen, und meine Account-Einstellungen verwalten,
So that ich PayWatcher an meine Bedürfnisse anpassen und meine Kontodaten einsehen kann.

**Acceptance Criteria:**

**Given** ich bin im Account-Bereich der Settings (FR23)
**When** die Confirmation-Override-Einstellung angezeigt wird
**Then** sehe ich die aktuelle Default-Confirmation-Anzahl (Standard: 6, aus GET /v1/paywatcher/config → confirmation_override Feld)
**And** ich kann den Wert über ein Zahlenfeld oder Slider zwischen 2 und 20 anpassen
**And** eine kurze Erklärung ist sichtbar: "Number of block confirmations required before a payment is marked as confirmed."

**Given** ich ändere die Confirmation-Anzahl
**When** ich auf "Save" klicke
**Then** wird der Wert via Proxy an PATCH /v1/paywatcher/config mit `{ confirmation_override: value }` gesendet
**And** Zod validiert: Ganzzahl, min 2, max 20 (oder null für Default)
**And** ein Toast erscheint: "Settings updated" (3s auto-dismiss)

**Given** ich bin im Account-Bereich der Settings (FR24)
**When** die Deposit Address angezeigt wird
**Then** sehe ich meine Deposit Address in Geist Mono (read-only, nicht editierbar, aus GET /v1/paywatcher/config → deposit_address Feld)
**And** ein Copy-Button neben der Adresse kopiert sie in die Zwischenablage (Inline-Checkmark, kein Toast)
**And** die Adresse ist als Link zu Basescan klickbar (öffnet in neuem Tab, NFR14)

**Given** ich bin im Account-Bereich der Settings (FR25)
**When** meine Account-Einstellungen angezeigt werden
**Then** sehe ich meine Email-Adresse (contact_email aus Session/Config, read-only)
**And** ein Logout-Button ist verfügbar (konsistent mit Story 2.2)

**Given** die Settings-Daten laden
**When** die API-Response noch aussteht
**Then** sehe ich Skeleton-Placeholders für die Felder
**And** Sidebar + Header stehen sofort

**Given** die Settings auf Mobile (<1024px) angezeigt werden
**When** die Seite rendert
**Then** ist das Layout einspaltig, Formulare nutzen volle Breite, alle Touch Targets min. 44x44px

- KEIN separater Tenant-Proxy-Endpoint nötig — alle Daten kommen aus GET/PATCH /v1/paywatcher/config

## Epic 5: Developer Documentation (Docs-Lite)

Ein Developer kann Quick Start Guide, Endpoint Reference, Webhook Events und Code-Beispiele lesen — alles was nötig ist um PayWatcher erfolgreich zu integrieren.

**Status:** BACKLOG

### Story 5.1: MDX Infrastructure & Docs Layout

As a Developer (Sempre),
I want eine MDX-basierte Docs-Infrastruktur mit Navigation und Syntax Highlighting,
So that alle Docs-Seiten konsistent gerendert werden und Code-Beispiele professionell aussehen.

**Acceptance Criteria:**

**Given** das Projekt aus Epic 1 existiert
**When** @next/mdx konfiguriert wird
**Then** ist das Plugin in `next.config.ts` eingebunden
**And** `.mdx` Dateien in `content/docs/` werden als Seiten gerendert
**And** MDX-Content wird via SSG pre-rendered (kein Client-Side Rendering)

**Given** die Docs-Routen erstellt werden
**When** ein User `/docs` aufruft
**Then** sehe ich eine Docs-Index-Seite mit Links zu allen verfügbaren Dokumenten: Quick Start, Endpoints, Webhooks, Examples
**And** die Route liegt in der (landing) Route Group (öffentlich, kein Auth nötig)

**Given** eine einzelne Docs-Seite geladen wird (z.B. `/docs/quick-start`)
**When** die Seite rendert
**Then** sehe ich ein Docs-Layout mit: Seiteninhalt (max-w-4xl, zentriert) und einer Navigation zwischen den Docs-Seiten (Prev/Next oder Sidebar-TOC)

**Given** Code-Blöcke in MDX existieren
**When** die Seite rendert
**Then** werden sie mit Shiki server-side gerendert (Syntax Highlighting)
**And** jeder Code-Block hat einen Copy-Button (Inline-Checkmark, kein Toast)
**And** Sprach-Labels sind sichtbar (bash, javascript, typescript, json, python)

**Given** Custom MDX-Komponenten definiert werden
**When** MDX Content sie verwendet
**Then** funktionieren: CodeBlock (mit Copy), Heading Anchors (klickbare `#`-Links für Deep Linking), Callout/Note Boxen für wichtige Hinweise

**Given** die Docs über die Dashboard-Sidebar erreichbar sein sollen
**When** ein eingeloggter Tenant auf "Docs" in der Sidebar klickt
**Then** wird er zu `/docs` navigiert (gleiche Seiten wie für nicht-eingeloggte User)

### Story 5.2: Quick Start Guide & Endpoint Reference

As a Developer,
I want einen Quick Start Guide und eine vollständige Endpoint Reference lesen können,
So that ich PayWatcher in unter 15 Minuten integrieren kann und alle API-Details nachschlagen kann.

**Acceptance Criteria:**

**Given** ich öffne den Quick Start Guide (FR28, `/docs/quick-start`)
**When** die Seite rendert
**Then** sehe ich eine Schritt-für-Schritt-Anleitung mit:
1. **Get Your API Key** — Verweis auf Request Access, Erklärung des API Key Formats (`mms_paywatcher_...`)
2. **Create a Payment Intent** — curl-Beispiel mit erklärendem Text (`POST /v1/payments`)
3. **Customer Sends USDC** — Erklärung des Deposit Address Flows (Shared Address + Unique Amount)
4. **Receive Webhook** — Beispiel-Payload des `payment.confirmed` Events
**And** jedes Code-Beispiel hat Syntax Highlighting (Shiki) und Copy-Button
**And** am Ende ein CTA: "Request Access — Free" (Link zum Request Access Formular)

**Given** der Quick Start Guide auf Mobile (<1024px) geladen wird
**When** die Seite rendert
**Then** sind Code-Blöcke horizontal scrollbar (nicht umgebrochen)
**And** der Scroll-Indicator-Gradient am rechten Rand ist sichtbar

**Given** ich öffne die Endpoint Reference (FR29, `/docs/endpoints`)
**When** die Seite rendert
**Then** sehe ich für jeden API-Endpoint:
- HTTP Method + Path (z.B. `POST /v1/payments`)
- Beschreibung des Endpoints
- Request-Schema (Parameter, Body mit Typen)
- Response-Schema (Success + Error Formate)
- curl-Beispiel mit Copy-Button
**And** die echten MMS Endpoints sind dokumentiert:
  - `POST /v1/payments` (Create Payment Intent)
  - `GET /v1/payments/:id` (Get Payment Status)
  - `GET /v1/payments` (List Payments — Query-Params: status, from, to, amount_min, amount_max, page, limit, sort, order)
  - `GET /v1/payments/:id/webhooks` (Webhook Delivery History)
  - `GET /v1/paywatcher/config` (Read Config)
  - `PATCH /v1/paywatcher/config` (Update Config — webhook_url, confirmation_override)
  - `POST /v1/paywatcher/config/test-webhook` (Test Webhook)
  - `GET /v1/paywatcher/config/api-key` (View Masked API Key)
**And** das `{ data }` / `{ error }` Envelope-Format ist erklärt

**Given** ein Developer die Endpoint Reference liest
**When** er ein Code-Beispiel kopiert
**Then** ist der Code funktional korrekt und sofort in einem Terminal ausführbar (mit Platzhalter für API Key)

### Story 5.3: Webhook Event Reference & Multi-Language Code Examples

As a Developer,
I want eine Webhook Event Reference und Code-Beispiele in mehreren Sprachen sehen können,
So that ich Webhooks korrekt verarbeiten und PayWatcher in meiner bevorzugten Sprache integrieren kann.

**Acceptance Criteria:**

**Given** ich öffne die Webhook Event Reference (FR30, `/docs/webhooks`)
**When** die Seite rendert
**Then** sehe ich:
- Liste aller Webhook Event Types (z.B. `payment.confirmed`, `payment.expired`, `payment.failed`, `payment.test`)
- Für jeden Event: Trigger-Bedingung, Beispiel-Payload als JSON (mit Syntax Highlighting + Copy)
- HMAC-Signatur-Verifikation: Erklärung des `x-paywatcher-signature` Headers mit Code-Beispiel
- Retry-Verhalten: "5 retries with exponential backoff, stops on 2xx response"

**Given** die Webhook Payloads angezeigt werden
**When** ein Developer den JSON-Payload sieht
**Then** sind alle Felder dokumentiert (event, payment_id, timestamp, data.amount, data.exact_amount, data.currency, data.chain, data.tx_hash, data.confirmations)
**And** der Payload ist in Geist Mono formatiert
**And** ein Copy-Button kopiert den gesamten Payload

**Given** ich öffne die Code Examples Seite (FR31, `/docs/examples`)
**When** die Seite rendert
**Then** sehe ich vollständige Integration-Beispiele mit Tabs für mindestens drei Sprachen: curl, JavaScript (Node.js), Python
**And** jedes Beispiel zeigt: Payment Intent erstellen, Payment Status abfragen, Webhook empfangen und verifizieren

**Given** ich zwischen Sprachen wechsle
**When** ich auf einen Tab klicke (z.B. "JavaScript" → "Python")
**Then** wechselt das Code-Beispiel sofort (kein Laden, kein Flackern)
**And** die Tab-Auswahl bleibt über alle Abschnitte der Seite konsistent (wenn ich "Python" gewählt habe, sind alle Beispiele auf Python)

**Given** ein Developer ein Code-Beispiel kopiert (NFR4)
**When** der Copy-Button geklickt wird
**Then** wird der vollständige Code ohne Ladezeit in die Zwischenablage kopiert
**And** der Code ist funktional korrekt und mit minimalem Anpassungsaufwand lauffähig (Platzhalter: `YOUR_API_KEY`)

**Given** die Docs-Seiten Performance gemessen wird
**When** eine Docs-Seite geladen wird
**Then** ist sie via SSG pre-rendered und profitiert vom Vercel CDN
**And** Code-Beispiele sind sofort sichtbar und kopierbar (kein Client-Side Rendering für Syntax Highlighting)

## Epic 6: Admin Dashboard

Mario (Operator) kann den PayWatcher Service überwachen, Tenants managen, globale Payments einsehen und System Health prüfen — alles was für den operativen Betrieb nötig ist.

**Status:** BACKLOG

**Referenzen:**
- Product Brief v2, Abschnitt 3.3 (Admin Dashboard)
- Architecture Doc: `app/(admin)/` Route Group, `isAdmin` Check via Email, `MMS_ADMIN_API_KEY` ENV Variable
- User Journey 4 (Mario — Admin/Operator)
- MMS Admin Endpoints: POST/GET/PATCH /v1/admin/paywatcher/tenants, GET /v1/admin/paywatcher/payments, GET /v1/admin/paywatcher/health

### Story 6.1: Admin Shell, Auth & Route Protection

As a Admin (Mario),
I want ein Admin-Dashboard-Layout mit eigenem Auth-Check und geschützten Routen,
So that nur ich als Admin auf die Admin-Seiten zugreifen kann.

**Acceptance Criteria:**

**Given** ich bin per Magic Link eingeloggt und meine Email ist als Admin konfiguriert (z.B. mario@masem.at)
**When** ich `/admin` aufrufe
**Then** sehe ich das Admin-Dashboard-Layout mit einer Sidebar mit den Items: Overview, Tenants, Payments
**And** der Header zeigt "Admin" als Kennzeichnung

**Given** ein eingeloggter User, dessen Email NICHT als Admin konfiguriert ist
**When** er `/admin` aufruft
**Then** wird er auf das User Dashboard redirected (oder sieht eine 403-Seite)

**Given** ein nicht-authentifizierter User
**When** er `/admin` aufruft
**Then** wird er auf `/login` redirected

**Given** das Admin-Layout implementiert wird
**When** die Route Group erstellt wird
**Then** existiert `app/(admin)/layout.tsx` mit `isAdmin` Check (Email-basiert, konfigurierbar via ENV `ADMIN_EMAILS`)
**And** die Admin-Sidebar ist separat vom User-Dashboard-Sidebar

**Given** Admin API Proxy Routes erstellt werden
**When** ein Admin Route Handler aufgerufen wird
**Then** nutzt er `process.env.MMS_ADMIN_API_KEY` für MMS-Calls (nicht die tenant_keys DB)
**And** existieren: `app/api/admin/health/route.ts`, `app/api/admin/tenants/route.ts`, `app/api/admin/tenants/[slug]/route.ts`, `app/api/admin/payments/route.ts`
**And** jeder Route Handler prüft `isAdmin` bevor er den Request weiterleitet

- Admin ist Desktop-Only — kein Mobile-Layout nötig
- `isAdmin` Check via Email-Whitelist in ENV Variable (z.B. `ADMIN_EMAILS=mario@masem.at`)
- Admin API Key aus `MMS_ADMIN_API_KEY` ENV Variable

### Story 6.2: System Health & Overview

As a Admin (Mario),
I want auf einen Blick sehen ob alle Systeme laufen und die wichtigsten Kennzahlen der letzten 24h kennen,
So that ich in unter 2 Minuten weiß ob alles in Ordnung ist oder ob ich eingreifen muss.

**Acceptance Criteria:**

**Given** ich bin als Admin eingeloggt und öffne `/admin` (FR39)
**When** die Overview-Seite lädt
**Then** sehe ich System Health Status Cards für: RPC Status (connected/error + letzte Response Time), Redis Status (connected/error), Database Status (connected/error), Last Poll Timestamp (wann wurde zuletzt gepollt)
**And** jede Card zeigt einen farbigen Indikator: grün (healthy), gelb (degraded), rot (error)
**And** die Daten kommen von GET /v1/admin/paywatcher/health via Proxy

**Given** die Health-Daten geladen sind
**When** die 24h-Statistiken angezeigt werden
**Then** sehe ich: Total Payments (24h), Confirmed Count, Expired Count, Failed Count, Webhook Success Rate (%)
**And** die Zahlen kommen aus dem Health Endpoint (24h Stats Felder)

**Given** die Health-Daten Wallet-Informationen enthalten
**When** die Wallet-Section angezeigt wird
**Then** sehe ich: Hot Wallet Balance (USDC + ETH), Cold Wallet Balance (falls im Health Endpoint vorhanden)
**And** Balances werden in Geist Mono dargestellt

**Given** die Admin Overview-Seite aktiv ist
**When** die Seite geöffnet bleibt
**Then** werden die Health-Daten alle 30 Sekunden automatisch refreshed (TanStack Query `refetchInterval: 30_000`)

**Given** die Health-Daten laden
**When** die API-Response noch aussteht
**Then** sehe ich Skeleton-Placeholders für die Cards

- Proxy Route: GET /api/admin/health → GET /v1/admin/paywatcher/health (mit MMS_ADMIN_API_KEY)
- Health Endpoint returned immer HTTP 200 — Subsystem-Status in den Response-Feldern (Graceful Degradation)

### Story 6.3: Tenant Management

As a Admin (Mario),
I want alle Tenants sehen, neue Tenants anlegen und bestehende Tenants aktivieren/deaktivieren können,
So that ich den Managed MVP Onboarding-Flow vollständig über das Dashboard abwickeln kann.

**Acceptance Criteria:**

**Given** ich bin als Admin eingeloggt und öffne `/admin/tenants` (FR40)
**When** die Tenants-Seite lädt
**Then** sehe ich eine Tabelle aller Tenants mit: tenant_slug, tenant_name, Status (enabled/disabled als Badge), Tier (free/starter/pro/enterprise), Payment Count, Total Volume, Webhook Success Rate
**And** die Daten kommen von GET /v1/admin/paywatcher/tenants (inkl. Aggregationen)

**Given** ich klicke auf "New Tenant" (FR41)
**When** das Erstell-Formular angezeigt wird
**Then** sehe ich Felder für: tenant_name (required), tenant_slug (required, auto-generated aus Name, editierbar), contact_email (required), tier (Dropdown: free, starter, pro, enterprise), webhook_url (optional), confirmation_override (optional, 2-20)
**And** React Hook Form + Zod Validierung

**Given** ich fülle das Formular aus und klicke "Create Tenant"
**When** der Tenant erfolgreich erstellt wird (POST /v1/admin/paywatcher/tenants)
**Then** zeigt ein Modal einmalig: API Key im Klartext (mit Copy-Button), Webhook Secret im Klartext (mit Copy-Button)
**And** eine Warnung: "This is the only time these credentials will be shown."
**And** nach Schließen des Modals ist der neue Tenant in der Liste sichtbar

**Given** ich klicke auf den Enable/Disable Toggle eines Tenants (FR42)
**When** ich den Status ändere
**Then** wird der Status via PATCH /v1/admin/paywatcher/tenants/:slug mit `{ enabled: true/false }` aktualisiert
**And** bei Deaktivierung erscheint ein Confirmation Dialog: "Disable tenant [name]? They will lose API access."
**And** der Status-Badge in der Tabelle aktualisiert sich sofort

**Given** ich klicke auf einen Tenant in der Tabelle
**When** die Detail-Ansicht angezeigt wird
**Then** sehe ich: Config-Details (webhook_url, confirmation_override, deposit_address, tier), Created/Updated Timestamps
**And** einen Link zu den Payments dieses Tenants (→ `/admin/payments?tenant=slug`)

- Proxy Routes: GET/POST /api/admin/tenants, PATCH /api/admin/tenants/[slug]
- Nach Tenant-Erstellung: Mario muss den Onboarding-Flow manuell fortführen (Parties API + tenant_keys DB) — das ist ein separater Prozess außerhalb dieses Epics (CLI Script oder manuell)

### Story 6.4: Global Payments

As a Admin (Mario),
I want alle Payments über alle Tenants sehen und nach Tenant filtern können,
So that ich das Gesamtvolumen überblicke und bei Problemen schnell den betroffenen Tenant identifizieren kann.

**Acceptance Criteria:**

**Given** ich bin als Admin eingeloggt und öffne `/admin/payments` (FR43)
**When** die Payments-Seite lädt
**Then** sehe ich eine Tabelle aller Payments über alle Tenants mit: Payment ID (monospace), Tenant (tenant_slug), Amount (monospace, USDC), Status (PaymentStatusBadge), txHash (monospace, truncated, Copy-Button), Created (Timestamp)
**And** die Daten kommen von GET /v1/admin/paywatcher/payments via Proxy

**Given** ich den Tenant-Filter verwende
**When** ich einen Tenant aus dem Dropdown wähle
**Then** werden nur Payments dieses Tenants angezeigt (Query-Param `tenant_slug`)
**And** der Filter ist als URL Query-Param gespeichert (`?tenant=slug`)

**Given** die Admin Payments-Tabelle angezeigt wird
**When** ich die gleichen Filter wie im User Dashboard nutze
**Then** kann ich nach Status filtern (6 States), nach Zeitraum filtern (7d/30d/90d/All), und nach Created At sortieren
**And** Pagination funktioniert identisch zum User Dashboard

**Given** ich auf ein Payment in der Tabelle klicke
**When** die Detail-Ansicht angezeigt wird
**Then** sehe ich die gleiche Payment-Detail-View wie im User Dashboard (Story 3.5) plus den Tenant-Slug als zusätzliches Feld

**Given** die Payments-Daten laden
**When** die API-Response noch aussteht
**Then** sehe ich Skeleton-Zeilen als Loading-State

- Proxy Route: GET /api/admin/payments → GET /v1/admin/paywatcher/payments (mit MMS_ADMIN_API_KEY)
- Wiederverwendung von PaymentStatusBadge, DataTable und Filter-Komponenten aus Epic 3
- Admin Payments Response enthält zusätzlich tenantSlug pro Payment
