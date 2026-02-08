# MMS Project Bible

> **MANDATORY READING FOR ALL AGENTS** - This file MUST be loaded and read by every agent at session start. No exceptions.

---

## Company Reference

**Owner:** Mario "Sempre" Semper
**Company:** masemIT e.U.
**Full company details:** [Company Info](./_masemIT/company-info.md)

---

## Project: MMS (masemIT Managed Services)

**Vision:** MMS is the central service layer for all masemIT products and projects.
**Domain:** api.masem.at
**First Service:** Donations Service (Phase 1)

**Status:** In Development — Donations API live, calculation refactoring complete

**Requirements:** [MMS Donations Requirement (Phase 1)](./_masemIT/requirements/mms-donations-requirement.md)
**Architecture:** [Architecture Decision Document](./planning-artifacts/architecture.md)
**Framework:** Express 5 (migrated from Hono on 2026-02-05)

---

## Completed Epics

### DON-CALC-01: Donations Calculation Refactoring ✅

**Completed:** 2026-02-04
**Epic Doc:** [epic-donations-calculation-refactoring.md](./epic-donations-calculation-refactoring.md)

**Core Change:** Projects now send `product_price_cents` instead of `amount_cents`. MMS calculates donation amounts based on configured percentages.

**Use Cases:**
| Use Case | When | Result |
|----------|------|--------|
| Direct | `target_id` provided | 100% to specified target |
| Distributed | `target_id` omitted | Split per `target_distribution` table |

**New Tables:**
- `project_donation_settings` — per-project enabled/percentage config
- `target_distribution` — how donations without target_id are split

**Stories Completed:** 1-8 (All stories including Admin Endpoints + Percentage Override)

---

### DON-SUPPORT-01: Supporters Endpoint & Schema Hardening ✅

**Completed:** 2026-02-04
**Epic Doc:** [epic-supporters-endpoint.md](./epic-supporters-endpoint.md)

**Goal:** Spender-Wall-Support für Client-Projekte + Auth-Absicherung + Schema-Flexibilität

**Changes:**
- `donor_note` column migrated from TEXT to JSONB (flexible project data)
- `/public/summary` endpoint removed (Auth-by-Default Policy)
- New `GET /v1/donations/supporters` endpoint for wall display
- Documentation updated

**Stories Completed:** 1-5 (Schema Migration, Auth-Fix, Supporters Endpoint, Docs, Bible)

---

## Existing masemIT Products & Services

| Product | Status | URL |
|---------|--------|-----|
| TellingCube | V2 launched Jan 2026 | https://tellingcube.com |
| ChainSights | Launched Dec 2025 | https://chainsights.one |
| HoKi Help | Launched Jan 2026 | https://hoki.help |
| MailWatcher | Internal / In progress | - |
| masemIT Analytics | Internal / Build error | - |

**Planned:** tellingDash, tru3crime

---

## Tech Stack Context (masemIT Standard)

| Service | Provider |
|---------|----------|
| Hosting | Vercel Pro |
| Database | NeonDB Launch |
| Cache | Upstash |
| Queue | QStash |
| Email | Resend Pro |
| Bot Protection | Cloudflare Turnstile |
| Analytics | Vercel Analytics |
| Domain | masem.at |

---

## Key Decisions Log

| Date | Decision | Context |
|------|----------|---------|
| 2026-02-02 | Project Bible created | Central context for all agents to prevent knowledge loss between sessions |
| 2026-02-02 | Donations Service requirements finalized | Comprehensive req doc covers tech stack, data model, API spec, architecture pattern, and 4 implementation phases |
| 2026-02-02 | Project Bible mandatory loading added | All 9 agents now load project-bible.md as Step 3 during activation |
| 2026-02-02 | Architecture Document completed | Full arch doc: tech stack (Express+Drizzle+Neon+Upstash), 25+ decisions, implementation patterns, project structure, validation passed |
| 2026-02-03 | MMS Dashboard separates Repo | Dashboard wird eigenständiges Repo (`mms-dashboard`), Single-Tenant, konsumiert MMS API wie jeder andere Client |
| 2026-02-03 | API-Docs via `/docs` Endpoint | Dokumentation als Markdown über REST-Endpoint für Agent-Consumption (`GET /docs`, `GET /docs/{service}`) |
| 2026-02-03 | Doc-Automation als Phase 2 Epic | Zod → OpenAPI → Markdown Pipeline mit CI-Validierung, eigener Epic nach Donations Phase 1 |
| 2026-02-03 | Phase 2a: Manueller Docs-Endpoint zuerst | `/docs/donations` als statisches Markdown vor Project Integration, Automation später |
| 2026-02-03 | Integration Guide erstellt | Anleitung für Sempre zur Migration von hoki.help, ChainSights, tellingCube, masem.at |
| 2026-02-04 | Donations Calculation Refactoring | API Contract Change: `amount_cents` → `product_price_cents`, MMS berechnet Spendenbetrag aus Produktpreis |
| 2026-02-04 | Two Use Cases für Donations | Direct (mit target_id → 100% an Ziel) vs Distributed (ohne target_id → Split per Distribution) |
| 2026-02-04 | Neue DB Tables | `project_donation_settings` (Projekt-Config) + `target_distribution` (Verteilung) ersetzen `donation_config` |
| 2026-02-04 | Percentage Override | Optional `donation_percentage_override` für Discount-Code-Käufe, Client entscheidet wann höherer % gilt |
| 2026-02-04 | Admin Endpoints | `/v1/admin/project-settings` + `/v1/admin/target-distribution` mit `config:read/write` Scope |
| 2026-02-04 | Auth-by-Default Policy | Alle Endpoints auth-geschützt (API Key) außer `/docs`. Explizite Ausnahmen müssen dokumentiert werden. |
| 2026-02-04 | donor_note als JSONB | Flexibles Schema für projekt-spezifische Daten (first_name, is_anonymous, custom fields). Migration von TEXT. |
| 2026-02-04 | Supporters Endpoint | `GET /v1/donations/supporters` liefert Spenderliste für Wall-Display (hoki.help), Filterung client-seitig |
| 2026-02-05 | Express Migration | Hono → Express 5 wegen Body-Parsing-Issues auf Vercel Node.js Runtime (POST Requests hingen) |

---

## Session Notes

### 2026-02-05 — Express Migration Session

**Status:** Framework migration complete. Vercel deployment pending verification.

**Problem:** Hono POST requests hung indefinitely on Vercel Node.js runtime due to body stream handling issues. Multiple workarounds attempted (c.req.text(), Response wrapper, getReader(), Node.js streams) — all failed.

**Solution:** Migrate from Hono to Express 5, which has mature, reliable body parsing with `express.json()` middleware.

**Changes:**
- Replaced Hono with Express 5.2.1
- Added cors package for CORS handling
- Converted all route handlers from Hono (c) to Express (req, res, next)
- Updated middleware to Express patterns
- Added `api/index.ts` as Vercel entry point
- Added `src/dev.ts` for local development
- Fixed ESM imports with explicit `.js` extensions

**Files Changed:**
- `mms-api/package.json` — dependencies updated
- `mms-api/src/index.ts` — Express app
- `mms-api/src/middleware/auth.ts` — Express middleware
- `mms-api/src/lib/errors.ts` — Express response helpers
- `mms-api/src/lib/responses.ts` — Express response helpers
- All handler and route files in `src/services/`
- `mms-api/api/index.ts` — new Vercel entry point
- `mms-api/src/dev.ts` — new local dev server

**Commits:**
- `8309094` — feat(api): migrate from Hono to Express
- `30a53de` — fix(api): add .js extensions for ESM module resolution

**Documentation Updated:**
- `_bmad-output/planning-artifacts/architecture.md`
- `_bmad-output/project-bible.md`

---

### 2026-02-04 — Party Mode (Mary, Winston, John, Paige, Barry) Session

**Status:** Epic DON-SUPPORT-01 COMPLETE ✅

**Context:** Sempre identifizierte Security-Gap (Summary Endpoint public) und neuen Feature-Bedarf (Supporters Wall für hoki.help)

**Decisions Made:**
- Auth-by-Default Policy: Alle Endpoints außer `/docs` benötigen API Key
- donor_note wird JSONB für flexible Projekt-Daten
- Neuer Supporters Endpoint mit Pagination
- `/public/summary` Endpoint entfernt (hoki.help nutzt API Key)
- Docs-Struktur bleibt modular (ein File pro Service)

**Epic DON-SUPPORT-01 — All Stories Completed:**
| # | Story | Summary |
|---|-------|---------|
| 2 | Schema Migration | `donor_note` TEXT → JSONB mit default `{}` |
| 1 | Auth-Fix | `/public/summary` entfernt, `/summary` war schon auth-geschützt |
| 3 | Supporters Endpoint | `GET /v1/donations/supporters?source_project=X` mit Pagination |
| 4 | Docs Update | `/docs/donations` aktualisiert (JSONB, Supporters, Auth-Policy) |
| 5 | Project Bible | This update |

**Files Changed:**
- `mms-api/src/db/schema.ts` — JSONB column
- `mms-api/src/services/donations/types.ts` — Zod schemas + types
- `mms-api/src/services/donations/queries.ts` — getSupporters query
- `mms-api/src/services/donations/handlers.ts` — handleGetSupporters
- `mms-api/src/services/donations/routes.ts` — /supporters route, removed /public/summary
- `mms-api/src/services/donations/handlers.test.ts` — 10 new tests
- `mms-api/src/docs/routes.ts` — Updated documentation

**DB Migration:** `donor_note` TEXT → JSONB via Neon MCP

**Tests:** 203 passing

---

### 2026-02-04 — Barry (Solo Dev) + Mary (Analyst) Session

**Status:** DON-CALC-01 Epic complete (Stories 1-6). Admin Endpoints (Story 7) in Backlog.

**Completed:**
- Party Mode Session: Requirements Clarification mit Mary, Winston, Barry, John, Paige
- Story 1: DB Schema Changes (neue Tables, Seed Data, alte Table dropped)
- Story 2: API Contract Changes (Zod Schema, Response Type)
- Story 3: Use Case 1 - Direct Target Donation
- Story 4: Use Case 2 - Distributed Donation
- Story 5: Validation & Error Handling
- Story 6: Documentation Update (`/docs/donations`)

**Commits:**
- `47e65cd` — feat(api): refactor donations to calculate from product price
- `af93f59` — docs(api): update /docs/donations for new calculation-based API

**Project Settings (Seed Data):**
| Project | Enabled | Percentage |
|---------|---------|------------|
| hoki_help | true | 100% |
| tellingcube | true | 3% |
| chainsights | true | 5% |

**Target Distribution (Seed Data):**
| Target | Distribution |
|--------|-------------|
| HoKi NÖ | 100% |

**Next:** Completed in same session

### 2026-02-04 — Party Mode (Winston + Paige) + Barry Session

**Status:** Stories 7 + 8 complete. Epic DON-CALC-01 fully implemented.

**New Feature:** Percentage Override (Story 8)
- Optional `donation_percentage_override` field for discount code purchases
- Client sends higher percentage → MMS uses that instead of project default
- Use case: 10% discount code → donate 10% instead of default 3%

**Admin Endpoints (Story 7):**
- `GET /v1/admin/project-settings` — List all project settings
- `PUT /v1/admin/project-settings/:source_project` — Update project settings
- `GET /v1/admin/target-distribution` — List distributions with warning
- `PUT /v1/admin/target-distribution/:target_id` — Update distribution

**Files Created:**
- `mms-api/src/services/admin/types.ts`
- `mms-api/src/services/admin/queries.ts`
- `mms-api/src/services/admin/handlers.ts`
- `mms-api/src/services/admin/routes.ts`
- `mms-api/src/services/admin/handlers.test.ts`
- `mms-api/src/services/admin/index.ts`

**Tests:** 201 passing (191 + 6 override + 4 admin auth)

---

### 2026-02-02 — Winston (Architect) → Next Agent Handoff

**Status:** Architecture complete. Epics & Stories phase begins.

**Briefing fuer naechsten Agenten (John PM oder Bob SM):**
- Architecture Document liegt unter `_bmad-output/planning-artifacts/architecture.md`
- Requirements unter `_bmad-output/_masemIT/requirements/mms-donations-requirement.md`
- Beide Dokumente zusammen sind die vollstaendige Grundlage fuer Epics & Stories
- Das Requirement hat bereits 4 Implementierungsphasen definiert — diese koennen als Basis fuer Epics dienen
- Architektur definiert die Implementation Sequence (8 Schritte) und Cross-Component Dependencies
- UX-Phase entfaellt (reine API)
- Naechster Schritt: Epics & Stories → Sprint Planning → Dev

---

## Session Notes

### 2026-02-02 — Mary (Analyst) → Winston (Architect) Handoff

**Status:** Requirements complete. Architecture phase begins.

**Briefing fuer Winston:**
- Das Requirement-Dokument unter `_bmad-output/_masemIT/requirements/mms-donations-requirement.md` ist die zentrale Quelle. Es enthaelt bereits Tech Stack, Datenmodell, API Spec und Projektstruktur.
- Sempre will den vollstaendigen BMAD-Pfad (Architecture → Epics → Sprint → Dev) weil MMS das zentrale Service-Layer fuer alle masemIT-Produkte wird. Die Patterns hier werden das Template fuer jeden zukuenftigen Service.
- Das Dokument ist bewusst als Phase 1 (Donations Service) geschrieben, aber die Architektur muss skalierbar sein fuer: Users/Parties, Subscriptions, Newsletters, Consents, Auth Layer.
- Research war nicht noetig - Sempre kennt seinen Stack und hat klare Entscheidungen getroffen.
- **Offene Punkte** (Section 9 im Requirement): 5 Open Decisions die Winston beruecksichtigen sollte.
- UX-Phase entfaellt (reine API, kein Frontend).

**Naechste Schritte nach Architecture:**
PM (John) → Epics & Stories → SM (Bob) → Sprint Planning → Dev (Amelia)

---
