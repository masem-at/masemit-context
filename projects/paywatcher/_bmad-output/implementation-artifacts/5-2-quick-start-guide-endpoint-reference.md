# Story 5.2: Quick Start Guide & Endpoint Reference

Status: done

<!-- Note: Validation is optional. Run validate-create-story for quality check before dev-story. -->

## Story

As a Developer,
I want einen Quick Start Guide und eine vollständige Endpoint Reference lesen können,
So that ich PayWatcher in unter 15 Minuten integrieren kann und alle API-Details nachschlagen kann.

## Acceptance Criteria

1. **Given** ich öffne den Quick Start Guide (FR28, `/docs/quick-start`) **When** die Seite rendert **Then** sehe ich eine Schritt-für-Schritt-Anleitung mit: (1) Get Your API Key — Verweis auf Request Access, Erklärung API Key Format (`mms_paywatcher_...`), (2) Create a Payment Intent — curl-Beispiel mit erklärendem Text (`POST /v1/payments`), (3) Customer Sends USDC — Erklärung des Deposit Address Flows (Shared Address + Unique Amount), (4) Receive Webhook — Beispiel-Payload des `payment.confirmed` Events **And** jedes Code-Beispiel hat Syntax Highlighting (Shiki) und Copy-Button **And** am Ende ein CTA: "Request Access — Free" (Link zum Request Access Formular)

2. **Given** der Quick Start Guide auf Mobile (<1024px) geladen wird **When** die Seite rendert **Then** sind Code-Blöcke horizontal scrollbar (nicht umgebrochen) **And** der Scroll-Indicator-Gradient am rechten Rand ist sichtbar

3. **Given** ich öffne die Endpoint Reference (FR29, `/docs/endpoints`) **When** die Seite rendert **Then** sehe ich für jeden API-Endpoint: HTTP Method + Path, Beschreibung, Request-Schema (Parameter, Body mit Typen), Response-Schema (Success + Error Formate), curl-Beispiel mit Copy-Button **And** die echten MMS Endpoints sind dokumentiert: `POST /v1/payments`, `GET /v1/payments/:id`, `GET /v1/payments` (Query-Params), `GET /v1/payments/:id/webhooks`, `GET /v1/paywatcher/config`, `PATCH /v1/paywatcher/config`, `POST /v1/paywatcher/config/test-webhook`, `GET /v1/paywatcher/config/api-key` **And** das `{ data }` / `{ error }` Envelope-Format ist erklärt

4. **Given** ein Developer die Endpoint Reference liest **When** er ein Code-Beispiel kopiert **Then** ist der Code funktional korrekt und sofort in einem Terminal ausführbar (mit Platzhalter für API Key)

## Tasks / Subtasks

- [x] Task 1: Quick Start Guide MDX-Content erstellen (AC: #1, #2)
  - [x] 1.1 `content/docs/quick-start.mdx` — Placeholder-Content komplett ersetzen mit vollständigem 4-Step Quick Start Guide
  - [x] 1.2 Step 1 "Get Your API Key" — Erklärung API Key Format (`mms_paywatcher_...`), Verweis auf Request Access, `x-api-key` Header-Pattern
  - [x] 1.3 Step 2 "Create a Payment Intent" — `POST /v1/payments` curl-Beispiel mit allen Parametern (amount, currency, chain, metadata, expires_in), Request/Response JSON
  - [x] 1.4 Step 3 "Customer Sends USDC" — Deposit Address Flow erklären: Shared Address + Unique Amount (exact_amount), Payment Status Lifecycle (pending → matched → confirming → confirmed)
  - [x] 1.5 Step 4 "Receive Webhook" — `payment.confirmed` Beispiel-Payload (JSON), Webhook URL Konfiguration, HMAC Signatur-Hinweis
  - [x] 1.6 "Next Steps" Section — Links zu Endpoints, Webhooks, Examples Docs-Seiten + CTA "Request Access — Free" mit Link zu `/#request-access`
  - [x] 1.7 Callout-Boxen: `<Callout variant="info">` für API Key Info, `<Callout variant="warning">` für Testnet-Hinweis oder wichtige Einschränkungen

- [x] Task 2: Endpoint Reference MDX-Content erstellen (AC: #3, #4)
  - [x] 2.1 `content/docs/endpoints.mdx` — Placeholder-Content komplett ersetzen mit vollständiger API Reference
  - [x] 2.2 "Overview" Section — Base URL (`https://api.masem.at`), Authentication (`x-api-key` Header), Envelope-Format (`{ data }` / `{ data, meta }` / `{ error }`), Error Codes Tabelle
  - [x] 2.3 "Create Payment" Endpoint — `POST /v1/payments`: Request Body (amount, currency, chain, metadata, expires_in, webhook_url), Response JSON, curl-Beispiel
  - [x] 2.4 "Get Payment" Endpoint — `GET /v1/payments/:id`: Path Param, Response JSON (alle Felder), curl-Beispiel
  - [x] 2.5 "List Payments" Endpoint — `GET /v1/payments`: Query Params Tabelle (status, from, to, page, limit, sort, order), Pagination Response mit meta, curl-Beispiel
  - [x] 2.6 "Webhook History" Endpoint — `GET /v1/payments/:id/webhooks`: Response-Felder (event, attempt_number, response_code, error, created_at), curl-Beispiel
  - [x] 2.7 "Read Config" Endpoint — `GET /v1/paywatcher/config`: Response-Felder (tenant_slug, webhook_url, confirmation_override, deposit_address, etc.), curl-Beispiel
  - [x] 2.8 "Update Config" Endpoint — `PATCH /v1/paywatcher/config`: Request Body (webhook_url, confirmation_override), Response JSON, curl-Beispiel
  - [x] 2.9 "Test Webhook" Endpoint — `POST /v1/paywatcher/config/test-webhook`: Response (success, response_code, error), curl-Beispiel
  - [x] 2.10 "View API Key" Endpoint — `GET /v1/paywatcher/config/api-key`: Response (masked_key, scopes, created_at), curl-Beispiel

- [x] Task 3: Content-Qualitätssicherung (AC: alle)
  - [x] 3.1 Alle curl-Beispiele manuell prüfen: Platzhalter `YOUR_API_KEY`, korrekte Endpoints, gültiges JSON
  - [x] 3.2 Response-Schemas gegen `src/lib/api/types.ts` (MMS-Typen) und `src/types/payment.ts` / `src/types/tenant.ts` (Frontend-Typen) abgleichen
  - [x] 3.3 Build-Test: `npm run build` — SSG pre-rendering aller MDX-Seiten erfolgreich
  - [x] 3.4 Visueller Test: `/docs/quick-start` und `/docs/endpoints` im Browser prüfen (Syntax Highlighting, Copy-Buttons, Callouts, Navigation)

## Dev Notes

### Architektur-Patterns & Constraints

- **Docs-Strategie:** Docs-Lite eingebettet in `/docs` Route (Landing Route Group, öffentlich). MDX-basiert mit @next/mdx. [Source: _bmad-output/planning-artifacts/architecture.md#Frontend Architecture]
- **Rendering:** SSG pre-rendered. `generateStaticParams()` + `dynamicParams = false`. Content profitiert von Vercel CDN. [Source: _bmad-output/planning-artifacts/architecture.md#Infrastructure & Deployment]
- **MMS API Base URL:** `https://api.masem.at` — alle Endpoints relativ dazu. [Source: src/lib/api/client.ts]
- **Authentication:** `x-api-key` Header mit Tenant API Key. [Source: src/lib/api/client.ts]

### Bestehende Infrastruktur (WIEDERVERWENDEN, nicht neu bauen!)

- **MDX-Infrastruktur komplett vorhanden (Story 5.1):** @next/mdx konfiguriert, `mdx-components.tsx` mit Custom Components, Docs-Layout mit Sidebar + Prev/Next Navigation.
- **Code-Blöcke:** Jeder fenced code block (```bash, ```json, etc.) wird automatisch durch `MdxPre` → `CodeBlock` (Shiki SSR, Dual-Theme, CopyButton, Sprach-Label) gerendert. KEINE zusätzliche Konfiguration nötig — einfach Markdown-Code-Blöcke schreiben.
- **Callout-Boxen:** `<Callout variant="info|warning|danger">` ist global in MDX verfügbar, kein Import nötig.
- **Heading Anchors:** Alle `h1`–`h4` erhalten automatisch klickbare `#`-Links via `rehype-slug` + Custom Heading Components.
- **Docs-Navigation:** Sidebar-TOC und Prev/Next sind automatisch — keine Anpassung nötig.
- **Docs-Seiten-Registry:** `src/lib/docs.ts` (docPages Array) und `src/lib/docs-content.ts` (Import-Map) — KEINE Änderung nötig für Story 5.2, da quick-start und endpoints bereits registriert sind.

### KRITISCH: Was diese Story NICHT erfordert

- **KEIN neuer Code** — nur MDX-Content in `content/docs/quick-start.mdx` und `content/docs/endpoints.mdx` ersetzen
- **KEINE neuen Components** — alle MDX-Components (CodeBlock, Callout, Headings) existieren
- **KEINE Routing-Änderungen** — `/docs/quick-start` und `/docs/endpoints` sind bereits registriert
- **KEINE Konfigurationsänderungen** — `next.config.ts`, `docs.ts`, `docs-content.ts` bleiben unverändert
- **KEINE CodeTabs in MDX** — Multi-Language-Tabs sind für Story 5.3 (Examples-Seite). Story 5.2 verwendet einfache fenced code blocks.

### MMS API Endpoints (vollständige Referenz für Content)

**Payments-Endpoints:**

| Method | Path | Beschreibung | Response-Format |
|--------|------|-------------|-----------------|
| `POST` | `/v1/payments` | Payment Intent erstellen | `{ data: Payment }` |
| `GET` | `/v1/payments/:id` | Einzelnes Payment abrufen | `{ data: Payment }` |
| `GET` | `/v1/payments` | Payments auflisten (paginiert) | `{ data: Payment[], meta: { page, limit, total } }` |
| `GET` | `/v1/payments/:id/webhooks` | Webhook History pro Payment | `{ data: WebhookDelivery[] }` |

**Config-Endpoints:**

| Method | Path | Beschreibung | Response-Format |
|--------|------|-------------|-----------------|
| `GET` | `/v1/paywatcher/config` | Tenant-Konfiguration lesen | `{ data: TenantConfig }` |
| `PATCH` | `/v1/paywatcher/config` | Konfiguration aktualisieren | `{ data: TenantConfig }` |
| `POST` | `/v1/paywatcher/config/test-webhook` | Test-Webhook senden | `{ data: TestWebhookResult }` |
| `GET` | `/v1/paywatcher/config/api-key` | Masked API Key abrufen | `{ data: ApiKeyInfo }` |

**Payment-Objekt (alle Felder, camelCase in API-Response):**
```
id                   string    UUID
amount               string    "49.00" (requested amount)
exactAmount          string    "49.000042" (unique amount with 6 decimals)
status               string    "pending"|"matched"|"confirming"|"confirmed"|"expired"|"failed"
txHash               string?   "0x..." (null bis matched)
depositAddress       string    "0x..." (shared deposit address)
confirmations        number    Aktuelle Block-Confirmations
confirmationsRequired number   Benötigte Confirmations (default 6)
currency             string    "USDC"
chain                string    "base"
metadata             object?   Frei definierbare Key-Value-Paare
expiresAt            string    ISO 8601 UTC Timestamp
createdAt            string    ISO 8601 UTC Timestamp
updatedAt            string    ISO 8601 UTC Timestamp
```

**WICHTIG: snake_case Inkonsistenz in MMS:**
- `/v1/payments` und `/v1/payments/:id` liefern **camelCase** (kein Transform nötig)
- `/v1/paywatcher/config`, `/v1/paywatcher/config/api-key`, `/v1/paywatcher/config/test-webhook`, `/v1/payments/:id/webhooks` liefern **snake_case**
- In der Endpoint Reference die **RAW MMS-Response** dokumentieren (snake_case wo MMS snake_case liefert), da Developer direkt gegen die API programmieren

**POST /v1/payments — Request Body:**
```json
{
  "amount": "49.00",
  "currency": "USDC",
  "chain": "base",
  "metadata": { "order_id": "ORD-123", "customer": "john@example.com" },
  "expires_in": 3600,
  "webhook_url": "https://your-app.com/webhook"
}
```
- `amount` (required): String, Betrag in USDC
- `currency` (required): "USDC"
- `chain` (required): "base"
- `metadata` (optional): Key-Value-Paare, max 10 Keys
- `expires_in` (optional): Sekunden bis Ablauf (default 3600 = 1h)
- `webhook_url` (optional): Override der globalen Webhook-URL für dieses Payment

**GET /v1/payments — Query Parameters:**
```
status     string   Filter by status (comma-separated: "pending,confirmed")
from       string   ISO date, only payments created after
to         string   ISO date, only payments created before
page       number   Page number (default: 1)
limit      number   Items per page (default: 25, max: 100)
sort       string   Sort field (only "created_at" supported)
order      string   Sort direction: "asc" | "desc" (default: "desc")
```

**PATCH /v1/paywatcher/config — Request Body:**
```json
{
  "webhook_url": "https://your-app.com/webhook",
  "confirmation_override": 12
}
```
- `webhook_url` (optional): HTTPS URL für Webhooks
- `confirmation_override` (optional): Ganzzahl 2–20, oder `null` für Default (6)

**GET /v1/paywatcher/config/api-key — Response (snake_case):**
```json
{
  "data": {
    "masked_key": "mms_paywatcher_abc...xyz",
    "scopes": ["payments:read", "payments:write"],
    "created_at": "2026-01-15T10:00:00Z"
  }
}
```

**POST /v1/paywatcher/config/test-webhook — Response (snake_case):**
```json
{
  "data": {
    "success": true,
    "response_code": 200,
    "error": null
  }
}
```

**GET /v1/payments/:id/webhooks — Response (snake_case):**
```json
{
  "data": [
    {
      "event": "payment.confirmed",
      "attempt_number": 1,
      "response_code": 200,
      "error": null,
      "created_at": "2026-01-15T12:00:00Z"
    }
  ]
}
```

**Error Response Format:**
```json
{
  "error": {
    "code": "UNAUTHORIZED",
    "message": "Invalid or missing API key"
  }
}
```
Error Codes: `UNAUTHORIZED` (401), `FORBIDDEN` (403), `NOT_FOUND` (404), `VALIDATION_ERROR` (400/422), `INTERNAL_ERROR` (500)

**Payment Status Lifecycle:**
```
pending → matched → confirming → confirmed
   ↓
expired (nach expires_at)
   ↓
failed (Transaktion fehlgeschlagen)
```

### Content-Sprache & Stil

- **Docs sind auf Englisch** — API-Dokumentation für Developer-Audience international
- **Ton:** Klar, direkt, technisch. Kein Marketing-Speak. Developer-to-Developer.
- **Code-Beispiele:** curl als Primary (universell), funktional korrekt mit Platzhalter `YOUR_API_KEY`
- **Struktur:** Jeder Endpoint als eigene Section mit `##` oder `###` Heading

### Project Structure Notes

- Nur 2 Dateien werden modifiziert: `content/docs/quick-start.mdx` und `content/docs/endpoints.mdx`
- Keine Konflikte mit bestehender Projektstruktur
- MDX-Content liegt in `content/docs/` (Projekt-Root, nicht in `src/`)

### Learnings aus Story 5.1

- **Turbopack-Kompatibilität:** remark/rehype Plugins als Strings (nicht Imports). Bereits konfiguriert — keine Änderung nötig. [Source: Story 5.1 Debug Log]
- **MDX Dynamic Imports:** Template-Literal Dynamic Imports funktionieren NICHT mit Turbopack. Stattdessen explizite Import-Map in `docs-content.ts`. Bereits gelöst — keine Änderung nötig. [Source: Story 5.1 Debug Log]
- **@tailwindcss/typography:** `@plugin` Syntax in Tailwind v4 (nicht `@import`). `prose dark:prose-invert` auf Article-Element. Bereits konfiguriert. [Source: Story 5.1 Debug Log]
- **Content-Alias:** `@content/*` Alias in `tsconfig.json` für `content/` Verzeichnis im Projekt-Root. [Source: Story 5.1 Debug Log]
- **Callout Light/Dark Mode:** Callout-Farben funktionieren in beiden Modi (Code Review Fix H2). [Source: Story 5.1 Change Log]
- **extractLanguage:** Validiert gegen Shiki `bundledLanguages`. Unbekannte Sprachen fallen auf `bash` zurück. [Source: Story 5.1 Code Review Fix M2]
- **Placeholder-Content-Pattern:** Story 5.1 hat bereits Draft-Content erstellt mit korrekten PayWatcher API-Patterns (x-api-key, USDC, masem.at). Story 5.2 ersetzt diesen mit vollständigem Content. [Source: Story 5.1 Code Review Fix M3]
- **Commit Message Pattern:** "Add [Feature] with review fixes (Story X.Y)" [Source: Git Log]

### Git Intelligence

- Letzter Commit: `8a5ade8 Add MDX Infrastructure & Docs Layout with review fixes (Story 5.1)` — 22 Dateien, +2678 Zeilen
- Story 5.1 hat die gesamte Docs-Infrastruktur aufgebaut: MDX-Config, Custom Components, Layout, Routing, Placeholder-Content
- Alle Epic-4-Stories (4.1–4.3) und Epic-3-Stories (3.1–3.5) sind done
- Konsistentes Commit-Pattern über alle Stories

### References

- [Source: _bmad-output/planning-artifacts/epics.md#Story 5.2] — Acceptance Criteria
- [Source: _bmad-output/planning-artifacts/epics.md#Epic 5] — FRs covered: FR28, FR29
- [Source: _bmad-output/planning-artifacts/architecture.md#Frontend Architecture] — Docs-Lite Strategie
- [Source: _bmad-output/planning-artifacts/architecture.md#Vollständige Projektverzeichnis-Struktur] — content/docs/ Pfad
- [Source: _bmad-output/planning-artifacts/architecture.md#API Communication Pattern] — mmsApi Pattern
- [Source: _bmad-output/planning-artifacts/architecture.md#Implementation Patterns] — Naming, Format Patterns
- [Source: _bmad-output/implementation-artifacts/5-1-mdx-infrastructure-docs-layout.md] — Vorherige Story, Infrastruktur-Details, Debug Logs
- [Source: src/lib/api/types.ts] — MMS API Response Types (snake_case)
- [Source: src/types/payment.ts] — Payment, PaymentStatus, PaymentFilters, WebhookDelivery
- [Source: src/types/tenant.ts] — TenantConfig, TestWebhookResult, ApiKeyInfo
- [Source: src/lib/api/client.ts] — mmsApi() Fetch-Wrapper
- [Source: src/lib/api/transform.ts] — snakeToCamel Transformation
- [Source: src/mdx-components.tsx] — MDX Component-Mapping
- [Source: src/components/docs/mdx-code-block.tsx] — MdxPre Wrapper
- [Source: src/components/docs/callout.tsx] — Callout-Komponente
- [Source: src/lib/docs.ts] — Docs-Seiten-Registry
- [Source: src/lib/docs-content.ts] — MDX Import-Map
- [Source: content/docs/quick-start.mdx] — Aktueller Placeholder-Content (zu ersetzen)
- [Source: content/docs/endpoints.mdx] — Aktueller Placeholder-Content (zu ersetzen)

## Dev Agent Record

### Agent Model Used

Claude Opus 4.6

### Debug Log References

- Keine Debug-Probleme — reine Content-Story, alle Infrastruktur von Story 5.1 wiederverwendet

### Completion Notes List

- Quick Start Guide: Vollständiger 4-Step Guide (Get API Key → Create Payment → Customer Sends USDC → Receive Webhook) mit funktionalen curl-Beispielen, JSON Request/Response, Status Lifecycle Diagramm, Callout-Boxen und CTA
- Endpoint Reference: 8 Endpoints dokumentiert mit HTTP Method + Path, Request/Response Schemas, Parameter-Tabellen, Error Codes, und kopierbaren curl-Beispielen
- snake_case/camelCase korrekt: Payment-Endpoints (camelCase), Config/Webhook-Endpoints (snake_case) — jeweils als RAW MMS-Response dokumentiert
- Schemas verifiziert gegen `src/lib/api/types.ts`, `src/types/payment.ts`, `src/types/tenant.ts`
- Alle curl-Beispiele verwenden `YOUR_API_KEY` Platzhalter und korrekte Base URL `https://api.masem.at`
- Build-Test bestanden: `npm run build` — alle 26 Seiten SSG pre-rendered ohne Fehler
- Content auf Englisch (Developer-Audience international)

### File List

- `content/docs/quick-start.mdx` — Modified: Placeholder-Content ersetzt mit vollständigem Quick Start Guide
- `content/docs/endpoints.mdx` — Modified: Placeholder-Content ersetzt mit vollständiger Endpoint Reference
- `src/components/shared/code-block.tsx` — Modified: Review-Fix M3 — `code-scroll-wrap` Klasse für Gradient-Overlay
- `src/app/globals.css` — Modified: Review-Fix M3 — Scroll-Indicator-Gradient CSS für Mobile
- `_bmad-output/implementation-artifacts/sprint-status.yaml` — Modified: Story Status updated
- `_bmad-output/implementation-artifacts/5-2-quick-start-guide-endpoint-reference.md` — Modified: Tasks/Status/Record updated

## Change Log

- 2026-02-20: Story 5.2 implementiert — Quick Start Guide und Endpoint Reference MDX-Content erstellt, Placeholder-Content in beiden Dateien vollständig ersetzt, alle Acceptance Criteria erfüllt
- 2026-02-20: Code Review (5 Fixes applied) — H1: Status-Lifecycle-Diagramm korrigiert (parallele Branches statt lineare Kette), M1: nicht-ausführbaren Header-Code-Block entfernt, M2: Webhook-Signatur-Callout mit Verweis auf Webhooks-Seite formuliert, M3: Scroll-Indicator-Gradient für Mobile hinzugefügt (CSS + Wrapper-Klasse), L1: "Next Steps" Section zur Endpoints-Seite hinzugefügt. M4 (Webhook-Payload-Inkonsistenz mit Placeholder) als Koordinations-Hinweis für Story 5.3 notiert.
