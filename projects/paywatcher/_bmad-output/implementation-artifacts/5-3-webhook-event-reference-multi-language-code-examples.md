# Story 5.3: Webhook Event Reference & Multi-Language Code Examples

Status: done

<!-- Note: Validation is optional. Run validate-create-story for quality check before dev-story. -->

## Story

As a Developer,
I want eine Webhook Event Reference und Code-Beispiele in mehreren Sprachen sehen können,
So that ich Webhooks korrekt verarbeiten und PayWatcher in meiner bevorzugten Sprache integrieren kann.

## Acceptance Criteria

1. **Given** ich öffne die Webhook Event Reference (FR30, `/docs/webhooks`) **When** die Seite rendert **Then** sehe ich: Liste aller Webhook Event Types (`payment.confirmed`, `payment.expired`, `payment.failed`, `payment.test`), für jeden Event Trigger-Bedingung und Beispiel-Payload als JSON (Syntax Highlighting + Copy), HMAC-Signatur-Verifikation mit Erklärung des `x-paywatcher-signature` Headers und Code-Beispiel, Retry-Verhalten ("5 retries with exponential backoff, stops on 2xx response")

2. **Given** die Webhook Payloads angezeigt werden **When** ein Developer den JSON-Payload sieht **Then** sind alle Felder dokumentiert (event, payment_id, timestamp, data.amount, data.exact_amount, data.currency, data.chain, data.tx_hash, data.confirmations) **And** der Payload ist in Geist Mono formatiert **And** ein Copy-Button kopiert den gesamten Payload

3. **Given** ich öffne die Code Examples Seite (FR31, `/docs/examples`) **When** die Seite rendert **Then** sehe ich vollständige Integrations-Beispiele mit Tabs für mindestens drei Sprachen: curl, JavaScript (Node.js), Python **And** jedes Beispiel zeigt: Payment Intent erstellen, Payment Status abfragen, Webhook empfangen und verifizieren

4. **Given** ich zwischen Sprachen wechsle **When** ich auf einen Tab klicke (z.B. "JavaScript" → "Python") **Then** wechselt das Code-Beispiel sofort (kein Laden, kein Flackern) **And** die Tab-Auswahl bleibt über alle Abschnitte der Seite konsistent (wenn ich "Python" gewählt habe, sind alle Beispiele auf Python)

5. **Given** ein Developer ein Code-Beispiel kopiert (NFR4) **When** der Copy-Button geklickt wird **Then** wird der vollständige Code ohne Ladezeit in die Zwischenablage kopiert **And** der Code ist funktional korrekt und mit minimalem Anpassungsaufwand lauffähig (Platzhalter: `YOUR_API_KEY`)

6. **Given** die Docs-Seiten Performance gemessen wird **When** eine Docs-Seite geladen wird **Then** ist sie via SSG pre-rendered und profitiert vom Vercel CDN **And** Code-Beispiele sind sofort sichtbar und kopierbar (kein Client-Side Rendering für Syntax Highlighting)

## Tasks / Subtasks

- [x] Task 1: Webhook Event Reference MDX-Content erstellen (AC: #1, #2)
  - [x] 1.1 `content/docs/webhooks.mdx` — Placeholder-Content komplett ersetzen mit vollständiger Webhook Event Reference
  - [x] 1.2 Event Types Section — Tabelle aller Events: `payment.confirmed`, `payment.expired`, `payment.failed`, `payment.test` mit Trigger-Bedingungen
  - [x] 1.3 Payload-Format Section — Vollständige Feld-Dokumentation (event, payment_id, timestamp, data.amount, data.exact_amount, data.currency, data.chain, data.tx_hash, data.confirmations) mit JSON-Beispiel pro Event
  - [x] 1.4 HMAC-Signatur-Verifikation — `x-paywatcher-signature` Header, HMAC-SHA256 Algorithmus, Code-Beispiel für Verifikation (Node.js + Python)
  - [x] 1.5 Retry-Verhalten Section — 5 Retries, Exponential Backoff, Stopp bei 2xx
  - [x] 1.6 Best Practices Section — Idempotenz, schnelle Response, Logging

- [x] Task 2: Multi-Language CodeTabs für MDX erstellen (AC: #4, #6)
  - [x] 2.1 `src/components/docs/mdx-code-tabs.tsx` — Server Component Wrapper für Multi-Language-Tabs in MDX: Pre-rendert alle Sprach-Varianten mit Shiki serverseitig, übergibt HTML an Client-Tabs
  - [x] 2.2 Client-Side Tab-Komponente die vorgerenderte HTML-Blöcke umschaltet (kein Laden beim Tab-Wechsel)
  - [x] 2.3 Globale Tab-Synchronisation: Wenn "Python" gewählt wird, wechseln alle CodeTabs auf der Seite zu Python (React Context oder URL-State)
  - [x] 2.4 MDX-Component-Mapping: `CodeTabs` und `CodeTab` in `mdx-components.tsx` registrieren

- [x] Task 3: Code Examples MDX-Content erstellen (AC: #3, #4, #5)
  - [x] 3.1 `content/docs/examples.mdx` — Placeholder-Content komplett ersetzen mit vollständigen Multi-Language-Beispielen
  - [x] 3.2 "Create Payment Intent" Section — curl, JavaScript (Node.js fetch), Python (requests) mit Tabs
  - [x] 3.3 "Check Payment Status" Section — curl, JavaScript, Python mit Tabs
  - [x] 3.4 "Receive & Verify Webhook" Section — Node.js (Express + HMAC), Python (Flask + HMAC) mit Tabs
  - [x] 3.5 Alle Code-Beispiele mit Platzhalter `YOUR_API_KEY`, funktional korrekt, minimal anpassbar

- [x] Task 4: Content-Qualitätssicherung (AC: alle)
  - [x] 4.1 Alle JSON-Payloads gegen Story 5.2 Endpoint Reference abgleichen (Konsistenz)
  - [x] 4.2 Webhook Payload-Felder gegen bestehende Webhook-Typen in `src/types/payment.ts` prüfen
  - [x] 4.3 Build-Test: `npm run build` — SSG pre-rendering aller MDX-Seiten erfolgreich
  - [x] 4.4 Visueller Test: `/docs/webhooks` und `/docs/examples` im Browser prüfen (Syntax Highlighting, Copy-Buttons, Tabs, Navigation)
  - [x] 4.5 Tab-Synchronisation testen: Tab-Wechsel auf Examples-Seite wirkt über alle Abschnitte

## Dev Notes

### Architektur-Patterns & Constraints

- **Docs-Strategie:** Docs-Lite eingebettet in `/docs` Route (Landing Route Group, öffentlich). MDX-basiert mit @next/mdx. [Source: _bmad-output/planning-artifacts/architecture.md#Frontend Architecture]
- **Rendering:** SSG pre-rendered. `generateStaticParams()` + `dynamicParams = false`. Content profitiert von Vercel CDN. [Source: _bmad-output/planning-artifacts/architecture.md#Infrastructure & Deployment]
- **Content-Sprache:** Docs sind auf **Englisch** — Developer-Audience international. Ton: klar, direkt, technisch. [Source: Story 5.2 Dev Notes]
- **Webhook Payload Format:** snake_case — Webhook-Payloads kommen direkt vom MMS Backend, nicht durch den Frontend Proxy-Layer. Developer programmieren direkt gegen die Webhooks. [Source: Story 5.2 Dev Notes — snake_case Inkonsistenz]

### Bestehende Infrastruktur (WIEDERVERWENDEN, nicht neu bauen!)

- **MDX-Infrastruktur komplett vorhanden (Story 5.1):** @next/mdx konfiguriert, `mdx-components.tsx` mit Custom Components, Docs-Layout mit Sidebar + Prev/Next Navigation.
- **Code-Blöcke:** Jeder fenced code block wird automatisch durch `MdxPre` → `CodeBlock` (Shiki SSR, Dual-Theme, CopyButton, Sprach-Label) gerendert. KEINE zusätzliche Konfiguration nötig für einfache Code-Blöcke. [Source: src/components/docs/mdx-code-block.tsx]
- **Callout-Boxen:** `<Callout variant="info|warning|danger">` global in MDX verfügbar, kein Import nötig. [Source: src/components/docs/callout.tsx]
- **Heading Anchors:** `h1`–`h4` erhalten automatisch klickbare `#`-Links via `rehype-slug` + Custom Heading Components. [Source: src/mdx-components.tsx]
- **Docs-Navigation:** Sidebar-TOC und Prev/Next automatisch — keine Anpassung nötig. [Source: src/lib/docs.ts, src/lib/docs-content.ts]
- **Docs-Seiten-Registry:** `webhooks` und `examples` sind bereits in `src/lib/docs.ts` (docPages Array) und `src/lib/docs-content.ts` (Import-Map) registriert — KEINE Änderung nötig.
- **CodeTabs (Landing Page):** `src/components/landing/code-tabs.tsx` — Client Component mit shadcn/ui Tabs + CopyButton. Erwartet pre-rendered Shiki HTML als `html` Prop. KANN als Pattern-Referenz dienen, aber NICHT direkt in MDX verwendbar (erwartet Server-Side Pre-Rendering als Input).
- **CopyButton:** `src/components/shared/copy-button.tsx` — Inline-Checkmark-Feedback, kein Toast. [Source: src/components/shared/copy-button.tsx]

### KRITISCH: Multi-Language-Tabs Architektur-Entscheidung

**Problem:** MDX Code-Blöcke werden via `MdxPre` als Server Components gerendert (Shiki SSR). Tabs sind Client-Side interaktiv. Diese beiden müssen kombiniert werden.

**Lösung (empfohlen): Server-Pre-Render + Client-Tabs**
1. Neuer MDX Custom Component `<CodeTabs>` mit `<CodeTab lang="bash" title="curl">` Children
2. Server Component rendert ALLE Tab-Inhalte mit Shiki zur Build-Zeit (SSG)
3. Client Component zeigt nur den aktiven Tab an (CSS `display: none` oder Conditional Rendering)
4. Tab-Synchronisation via React Context: Ein `CodeTabsProvider` auf der Docs-Seite (oder im Docs-Layout) speichert die aktive Sprache. Alle `<CodeTabs>` auf der Seite reagieren auf die Auswahl.

**Bestehende CodeTabs als Referenz:**
- `src/components/landing/code-tabs.tsx` zeigt das Pattern: `tabs` Array mit `{ value, label, html, code }` — HTML ist pre-rendered Shiki Output.
- Für MDX muss das Pattern angepasst werden: Children sind `<CodeTab>` Elemente statt einem tabs-Array.

**KEINE neuen Abhängigkeiten nötig** — shadcn/ui Tabs + Shiki + CopyButton sind alle bereits installiert.

### Webhook-Payload-Referenz (für Content)

**Event Types:**

| Event | Trigger | Beschreibung |
|-------|---------|-------------|
| `payment.confirmed` | Payment erreicht `confirmationsRequired` Block-Confirmations | Zahlung erfolgreich bestätigt |
| `payment.expired` | `expiresAt` Zeitpunkt überschritten ohne Matching | Keine Zahlung eingegangen |
| `payment.failed` | Blockchain-Transaktion fehlgeschlagen | Technischer Fehler |
| `payment.test` | Manuell via Dashboard "Send Test Webhook" | Test der Webhook-Konfiguration |

**Webhook Payload Format (snake_case — direkt vom MMS Backend):**
```json
{
  "event": "payment.confirmed",
  "payment_id": "pay_7f2a3b4c-5d6e-7f8g-9h0i-1j2k3l4m5n6o",
  "timestamp": "2026-02-20T10:00:00Z",
  "data": {
    "amount": "49.00",
    "exact_amount": "49.000042",
    "currency": "USDC",
    "chain": "base",
    "tx_hash": "0x7a3f8b2c1d4e5f6a7b8c9d0e1f2a3b4c5d6e7f8a9b0c1d2e3f4a5b6c7d8e9f",
    "confirmations": 6
  }
}
```

**HMAC-Signatur-Verifikation:**
- Header: `x-paywatcher-signature`
- Algorithmus: HMAC-SHA256
- Secret: Webhook Secret (bei Tenant-Onboarding von Mario generiert, im MMS Backend konfiguriert)
- Payload: Raw Request Body (als String)
- Vergleich: `hmac_hex === x-paywatcher-signature` Header-Wert

**Retry-Logik:**
- Max 5 Retries bei Nicht-2xx-Response
- Exponential Backoff (1s, 2s, 4s, 8s, 16s Basis-Intervall)
- Stoppt sofort bei 2xx Response
- Fehlgeschlagene Deliveries im Dashboard unter Settings → Webhooks → Delivery Log einsehbar

### MMS API Endpoints (für Code-Beispiele)

**Payment Intent erstellen — POST /v1/payments:**
```
POST https://api.masem.at/v1/payments
Header: x-api-key: YOUR_API_KEY
Header: Content-Type: application/json

Body:
{
  "amount": "49.00",
  "currency": "USDC",
  "chain": "base",
  "metadata": { "order_id": "ORD-123" },
  "expires_in": 3600
}

Response:
{
  "data": {
    "id": "pay_7f2a3b4c-...",
    "amount": "49.00",
    "exactAmount": "49.000042",
    "status": "pending",
    "depositAddress": "0x...",
    "confirmations": 0,
    "confirmationsRequired": 6,
    "currency": "USDC",
    "chain": "base",
    "metadata": { "order_id": "ORD-123" },
    "expiresAt": "2026-02-20T11:00:00Z",
    "createdAt": "2026-02-20T10:00:00Z",
    "updatedAt": "2026-02-20T10:00:00Z"
  }
}
```

**Payment Status abfragen — GET /v1/payments/:id:**
```
GET https://api.masem.at/v1/payments/pay_7f2a3b4c-...
Header: x-api-key: YOUR_API_KEY

Response: { "data": { ...Payment } }
```

**WICHTIG: snake_case/camelCase Inkonsistenz:**
- Payment API Responses (`/v1/payments`) liefern **camelCase** (exactAmount, txHash, etc.)
- Webhook Payloads liefern **snake_case** (exact_amount, tx_hash, etc.)
- In den Docs beide Formate korrekt dokumentieren — Developer müssen wissen dass Webhooks snake_case verwenden

### Story 5.2 Code Review Hinweis (M4)

- **Webhook-Payload-Inkonsistenz mit Placeholder:** Placeholder-Content in `webhooks.mdx` verwendete teilweise anderes Format. Story 5.3 muss den vollständigen und korrekten Payload dokumentieren. [Source: Story 5.2 Change Log]

### Content-Sprache & Stil

- **Docs sind auf Englisch** — API-Dokumentation für Developer-Audience international
- **Ton:** Klar, direkt, technisch. Kein Marketing-Speak. Developer-to-Developer.
- **Code-Beispiele:** curl als Primary (universell) + JavaScript (Node.js) + Python. Funktional korrekt mit Platzhalter `YOUR_API_KEY`.
- **JSON-Payloads:** Alle Felder dokumentiert, mit Typen und Beschreibungen
- **HMAC-Verifikation:** Konkretes Code-Beispiel (nicht nur Erklärung) — Developer müssen Copy-Paste-fähigen Code haben

### Project Structure Notes

- **Modifizierte Dateien:** `content/docs/webhooks.mdx`, `content/docs/examples.mdx` (Content ersetzen)
- **Neue Dateien:** `src/components/docs/mdx-code-tabs.tsx` (Server+Client Wrapper für Multi-Language-Tabs in MDX)
- **Zu ändernde Dateien:** `src/mdx-components.tsx` (CodeTabs + CodeTab registrieren)
- Keine Konflikte mit bestehender Projektstruktur
- MDX-Content in `content/docs/` (Projekt-Root, nicht in `src/`)

### Learnings aus Story 5.1 & 5.2

- **Turbopack-Kompatibilität:** remark/rehype Plugins als Strings (nicht Imports). Bereits konfiguriert. [Source: Story 5.1 Debug Log]
- **MDX Dynamic Imports:** Template-Literal Dynamic Imports funktionieren NICHT mit Turbopack. Import-Map in `docs-content.ts` verwenden. Bereits gelöst. [Source: Story 5.1 Debug Log]
- **Code-Blöcke in MDX:** Einfach fenced code blocks (```bash, ```json etc.) verwenden — `MdxPre` rendert sie automatisch mit Shiki + CopyButton. [Source: Story 5.2 Dev Notes]
- **Callout in MDX:** `<Callout variant="info">` direkt ohne Import verwenden. [Source: Story 5.1]
- **Scroll-Indicator-Gradient:** Bereits in `globals.css` und `code-scroll-wrap` CSS-Klasse implementiert für Mobile Code-Blöcke. [Source: Story 5.2 Code Review Fix M3]
- **extractLanguage:** Validiert gegen Shiki `bundledLanguages`. Unbekannte Sprachen fallen auf `bash` zurück. [Source: Story 5.1 Code Review Fix M2]
- **Commit Message Pattern:** "Add [Feature] with review fixes (Story X.Y)" [Source: Git Log]

### Git Intelligence

- Letzter Commit: `54dbc7a Add Quick Start Guide & Endpoint Reference with review fixes (Story 5.2)` — Content-Story, 6 Dateien modifiziert
- Story 5.2 hat Quick Start + Endpoint Reference Content geschrieben — Story 5.3 macht das gleiche für Webhooks + Examples
- Gesamte Docs-Infrastruktur (Story 5.1) und Content-Pipeline (Story 5.2) sind stabil
- Konsistentes Commit-Pattern über alle 22+ Stories

### References

- [Source: _bmad-output/planning-artifacts/epics.md#Story 5.3] — Acceptance Criteria
- [Source: _bmad-output/planning-artifacts/epics.md#Epic 5] — FRs covered: FR30, FR31
- [Source: _bmad-output/planning-artifacts/architecture.md#Frontend Architecture] — Docs-Lite Strategie
- [Source: _bmad-output/planning-artifacts/architecture.md#Component Patterns] — Shiki, Copy-Button, Toast-Pattern
- [Source: _bmad-output/planning-artifacts/architecture.md#Implementation Patterns] — Naming, Format Patterns
- [Source: _bmad-output/implementation-artifacts/5-2-quick-start-guide-endpoint-reference.md] — Vorherige Story, MMS API Details, snake_case/camelCase Hinweis, Code Review M4
- [Source: _bmad-output/implementation-artifacts/5-1-mdx-infrastructure-docs-layout.md] — MDX-Infrastruktur, Debug Logs, Learnings
- [Source: src/mdx-components.tsx] — MDX Component-Mapping (erweiterbar für CodeTabs)
- [Source: src/components/docs/mdx-code-block.tsx] — MdxPre Wrapper (Shiki SSR)
- [Source: src/components/docs/callout.tsx] — Callout-Komponente
- [Source: src/components/landing/code-tabs.tsx] — CodeTabs Pattern-Referenz (Client Component mit pre-rendered HTML)
- [Source: src/components/shared/code-block.tsx] — CodeBlock mit Shiki SSR
- [Source: src/components/shared/copy-button.tsx] — CopyButton (Inline-Checkmark)
- [Source: src/lib/docs.ts] — Docs-Seiten-Registry (webhooks + examples bereits registriert)
- [Source: src/lib/docs-content.ts] — MDX Import-Map (webhooks + examples bereits registriert)
- [Source: src/types/payment.ts] — Payment, PaymentStatus, WebhookDelivery Types
- [Source: content/docs/webhooks.mdx] — Aktueller Placeholder-Content (zu ersetzen)
- [Source: content/docs/examples.mdx] — Aktueller Placeholder-Content (zu ersetzen)

## Dev Agent Record

### Agent Model Used

Claude Opus 4.6

### Debug Log References

Keine Debug-Probleme aufgetreten.

### Completion Notes List

- Task 1: `webhooks.mdx` komplett neu geschrieben — Event Types Tabelle, JSON-Payload pro Event (confirmed, expired, failed, test), vollständige Feld-Dokumentation (9 Felder), HMAC-Signatur-Verifikation (Node.js + Python mit timing-safe Vergleich), Retry-Verhalten-Tabelle, Best Practices Section
- Task 2: CodeTabs-Komponenten-Architektur implementiert — Server Component (`CodeTabs`) pre-rendert alle Sprach-Varianten mit Shiki bei Build-Zeit, Client Component (`MdxCodeTabsClient`) schaltet vorgerenderte HTML-Blöcke um, `CodeTabsProvider` (React Context) im Docs-Layout für globale Tab-Synchronisation über alle Abschnitte
- Task 3: `examples.mdx` komplett neu geschrieben — 3 Sections (Create Payment Intent, Check Payment Status, Receive & Verify Webhook) mit Multi-Language CodeTabs (curl/JavaScript/Python), HMAC-Verifikation in Webhook-Handler, snake_case/camelCase Hinweis
- Task 4: Content-Konsistenz mit Endpoint Reference validiert (txHash: null ergänzt), Webhook Payload-Felder gegen payment.ts geprüft, Build-Test bestanden (alle 4 Docs-Seiten SSG pre-rendered)

### Change Log

- 2026-02-20: Story 5.3 implementiert — Webhook Event Reference & Multi-Language Code Examples (4 Tasks, 20 Subtasks)
- 2026-02-20: Code Review — 6 Issues gefixed (H1: timingSafeEqual Buffer-Length-Guard, H2: Null-Checks für Signature/Secret, M1: curl-Tab für Webhook-Simulate, M2: Empty-Tabs-Guard, M3: Tab-Sync via M1 behoben, M4: fehlender `import os` in Python-Beispiel)

### File List

- `content/docs/webhooks.mdx` (modified) — Vollständige Webhook Event Reference
- `content/docs/examples.mdx` (modified) — Multi-Language Code Examples mit CodeTabs
- `src/components/docs/mdx-code-tabs.tsx` (new) — Server Component: CodeTabs + CodeTab für MDX
- `src/components/docs/mdx-code-tabs-client.tsx` (new) — Client Component: Tab-Umschaltung mit shadcn/ui Tabs
- `src/components/docs/code-tabs-provider.tsx` (new) — React Context für globale Tab-Synchronisation
- `src/app/(landing)/docs/layout.tsx` (modified) — CodeTabsProvider hinzugefügt
- `src/mdx-components.tsx` (modified) — CodeTabs + CodeTab registriert
