# Story 5.1: MDX Infrastructure & Docs Layout

Status: done

<!-- Note: Validation is optional. Run validate-create-story for quality check before dev-story. -->

## Story

As a Developer (Sempre),
I want eine MDX-basierte Docs-Infrastruktur mit Navigation und Syntax Highlighting,
So that alle Docs-Seiten konsistent gerendert werden und Code-Beispiele professionell aussehen.

## Acceptance Criteria

1. **Given** das Projekt aus Epic 1 existiert **When** @next/mdx konfiguriert wird **Then** ist das Plugin in `next.config.ts` eingebunden **And** `.mdx` Dateien in `content/docs/` werden als Seiten gerendert **And** MDX-Content wird via SSG pre-rendered (kein Client-Side Rendering)

2. **Given** die Docs-Routen erstellt werden **When** ein User `/docs` aufruft **Then** sehe ich eine Docs-Index-Seite mit Links zu allen verfügbaren Dokumenten: Quick Start, Endpoints, Webhooks, Examples **And** die Route liegt in der (landing) Route Group (öffentlich, kein Auth nötig)

3. **Given** eine einzelne Docs-Seite geladen wird (z.B. `/docs/quick-start`) **When** die Seite rendert **Then** sehe ich ein Docs-Layout mit: Seiteninhalt (max-w-4xl, zentriert) und einer Navigation zwischen den Docs-Seiten (Prev/Next oder Sidebar-TOC)

4. **Given** Code-Blöcke in MDX existieren **When** die Seite rendert **Then** werden sie mit Shiki server-side gerendert (Syntax Highlighting) **And** jeder Code-Block hat einen Copy-Button (Inline-Checkmark, kein Toast) **And** Sprach-Labels sind sichtbar (bash, javascript, typescript, json, python)

5. **Given** Custom MDX-Komponenten definiert werden **When** MDX Content sie verwendet **Then** funktionieren: CodeBlock (mit Copy), Heading Anchors (klickbare `#`-Links für Deep Linking), Callout/Note Boxen für wichtige Hinweise

6. **Given** die Docs über die Dashboard-Sidebar erreichbar sein sollen **When** ein eingeloggter Tenant auf "Docs" in der Sidebar klickt **Then** wird er zu `/docs` navigiert (gleiche Seiten wie für nicht-eingeloggte User)

## Tasks / Subtasks

- [x] Task 1: @next/mdx Paketinstallation & Next.js-Konfiguration (AC: #1)
  - [x] 1.1 `npm install @next/mdx @mdx-js/loader @mdx-js/react @types/mdx` ausführen
  - [x] 1.2 Optional: `npm install remark-gfm rehype-slug rehype-autolink-headings @tailwindcss/typography` für GFM-Support, Heading-Anchors und Prose-Styling
  - [x] 1.3 `next.config.ts` erweitern: `pageExtensions` + `createMDX()` Wrapper (Turbopack-kompatibel: Plugin-Namen als Strings)
  - [x] 1.4 `mdx-components.tsx` im Projekt-Root (oder `src/`) erstellen mit Custom-Component-Mapping

- [x] Task 2: MDX Custom Components erstellen (AC: #4, #5)
  - [x] 2.1 `src/components/docs/mdx-code-block.tsx` — Wrapper um bestehende `shared/code-block.tsx` für MDX `pre`/`code` Elemente (Shiki SSR + CopyButton + Sprach-Label)
  - [x] 2.2 `src/components/docs/callout.tsx` — Callout/Note-Box-Komponente (info, warning, danger Varianten)
  - [x] 2.3 Heading-Anchors via `rehype-slug` + `rehype-autolink-headings` Plugin ODER Custom-Heading-Komponenten in `mdx-components.tsx` (klickbare `#`-Links für Deep Linking)
  - [x] 2.4 MDX-Component-Mapping in `mdx-components.tsx` registrieren: `h1`–`h4` mit Anchors, `pre`/`code` mit CodeBlock, `a` für externe Links (target="_blank"), `Callout`, `CodeBlock`

- [x] Task 3: Docs-Routing & Layout (AC: #2, #3, #6)
  - [x] 3.1 `src/app/(landing)/docs/page.tsx` — Docs-Index-Seite mit Links zu Quick Start, Endpoints, Webhooks, Examples
  - [x] 3.2 `src/app/(landing)/docs/[slug]/page.tsx` — Dynamische Route mit `import(`@/content/docs/${slug}.mdx`)`, `generateStaticParams()` für SSG, `dynamicParams = false`
  - [x] 3.3 `src/app/(landing)/docs/layout.tsx` — Docs-Layout: Sidebar-TOC (Desktop) + Prev/Next Navigation, `max-w-4xl` zentrierter Content, `@tailwindcss/typography` Prose-Styling
  - [x] 3.4 `src/components/docs/docs-sidebar.tsx` — Docs-Navigationskomponente (Seitenlinks, Active-State basierend auf pathname)
  - [x] 3.5 `src/components/docs/docs-nav.tsx` — Prev/Next-Navigation am Seitenende

- [x] Task 4: Placeholder-MDX-Content erstellen (AC: #1, #2)
  - [x] 4.1 `content/docs/quick-start.mdx` — Platzhalter mit Titel, Intro, einem Code-Block und einer Callout-Box
  - [x] 4.2 `content/docs/endpoints.mdx` — Platzhalter mit Titel und Intro
  - [x] 4.3 `content/docs/webhooks.mdx` — Platzhalter mit Titel und Intro
  - [x] 4.4 `content/docs/examples.mdx` — Platzhalter mit Titel und Intro
  - [x] 4.5 `.gitkeep` in `content/docs/` entfernen

- [x] Task 5: Docs-Metadata & SEO (AC: #2)
  - [x] 5.1 `generateMetadata()` in `docs/[slug]/page.tsx` — Titel und Beschreibung aus MDX-Export (`export const metadata = { title, description }`)
  - [x] 5.2 Docs-Index-Seite: Static Metadata mit Titel "Documentation | PayWatcher"

- [x] Task 6: Verify & Cleanup (AC: alle)
  - [x] 6.1 Build-Test: `npm run build` erfolgreich mit MDX-Seiten (SSG pre-rendered)
  - [x] 6.2 Navigationstest: `/docs` zeigt Index, `/docs/quick-start` zeigt MDX-Content mit Syntax Highlighting
  - [x] 6.3 Dashboard-Sidebar "Docs"-Link navigiert zu `/docs`
  - [x] 6.4 Mobile-Layout: Docs-Sidebar versteckt oder als Sheet, Content responsive

## Dev Notes

### Architektur-Patterns & Constraints

- **Docs-Strategie lt. Architecture Doc:** "Docs-Lite eingebettet, /docs Route mit MDX" — Quick Start, Endpoint Reference, Webhook Payload direkt im Projekt. Separate Docs-App (Mintlify/Fumadocs) erst Phase 2. [Source: _bmad-output/planning-artifacts/architecture.md#Frontend Architecture]
- **MDX-Content Verzeichnis:** `content/docs/` (außerhalb `src/`, im Architecture Doc definiert). MDX-Dateien werden via Dynamic Import geladen, nicht als Pages im App-Directory. [Source: _bmad-output/planning-artifacts/architecture.md#Vollständige Projektverzeichnis-Struktur]
- **Route Group:** Docs liegen in `(landing)` Route Group — öffentlich, kein Auth nötig. Dashboard-Sidebar "Docs"-Link zeigt auf `/docs`. [Source: _bmad-output/planning-artifacts/architecture.md#App Structure & Routing]
- **Rendering:** SSG (Static Site Generation) — MDX wird zur Build-Zeit gerendert, profitiert von Vercel CDN. `generateStaticParams()` + `dynamicParams = false`. [Source: _bmad-output/planning-artifacts/architecture.md#Infrastructure & Deployment]

### Bestehende Infrastruktur (WIEDERVERWENDEN, nicht neu bauen!)

- **Shiki ist bereits installiert** (`shiki@3.22.0`) und konfiguriert in `src/components/shared/code-block.tsx`. Dual-Theme-Support (github-light/github-dark) mit `codeToHtml()`. Varianten: full, compact, inline. CSS für Shiki-Themes existiert in `globals.css`.
- **CopyButton existiert** in `src/components/shared/copy-button.tsx` (Client Component, Clipboard API, Inline-Checkmark-Feedback). `InlineCopyButton` ebenfalls vorhanden.
- **CodeTabs existiert** in `src/components/shared/code-block.tsx` — Tabbed Code Examples (curl/JS Tabs). Kann für Multi-Language Examples in Docs wiederverwendet werden.
- **Navigation:** Dashboard-Sidebar (`src/components/dashboard/sidebar-nav.tsx`) hat bereits "Docs"-Link auf `/docs` (`nav-config.ts`). Landing-Header (`nav-items.ts`) hat ebenfalls "Docs"-Link.
- **Theme System:** next-themes mit Dark Mode Default ist aktiv. Design Tokens (CSS Custom Properties) in `globals.css`.
- **Toast:** sonner (rechts unten). Aber Copy-Aktionen verwenden Inline-Checkmark, KEIN Toast.

### Technische Anforderungen (@next/mdx mit Next.js 16 + Turbopack)

- **Pakete:** `@next/mdx`, `@mdx-js/loader`, `@mdx-js/react`, `@types/mdx`
- **next.config.ts Anpassung:** `pageExtensions: ['js', 'jsx', 'md', 'mdx', 'ts', 'tsx']` + `createMDX()` Wrapper
- **KRITISCH: Turbopack-Kompatibilität:** remark/rehype Plugins müssen als Strings angegeben werden (nicht als importierte Funktionen), da JS-Funktionen nicht an Rust übergeben werden können. Beispiel: `remarkPlugins: ['remark-gfm']` statt `remarkPlugins: [remarkGfm]`. [Source: https://nextjs.org/docs/app/guides/mdx#using-plugins-with-turbopack]
- **mdx-components.tsx ist PFLICHT** für @next/mdx mit App Router. Datei im Projekt-Root oder `src/` erstellen.
- **Dynamic Import Pattern (empfohlen):** `const { default: Post } = await import(`@/content/docs/${slug}.mdx`)` in `app/(landing)/docs/[slug]/page.tsx` mit `generateStaticParams()`. Erlaubt Content außerhalb von `app/`.
- **Metadata via MDX-Export:** `export const metadata = { title: '...', description: '...' }` in jeder MDX-Datei. Import via Named Export: `import Post, { metadata } from '...'`.
- **KEIN Frontmatter:** @next/mdx unterstützt kein Frontmatter nativ. Stattdessen MDX-Exports verwenden (s.o.).

### Styling-Strategie für MDX-Content

- **Option A (empfohlen):** `@tailwindcss/typography` Plugin installieren und `prose` Klassen im Docs-Layout verwenden. Bietet sofort gute Typografie für Markdown-Content mit Dark-Mode-Support (`prose-invert` oder Dark-Mode-Varianten).
- **Custom Overrides:** Heading-Styles müssen PayWatcher Design Tokens respektieren (Geist Sans). Code-Blöcke verwenden bestehende Shiki-Integration (Geist Mono).
- **Docs-Layout:** `max-w-4xl` zentriert (lt. Epics), mit Sidebar-Navigation auf Desktop.

### Code-Block-Integration in MDX

- **Herausforderung:** Die bestehende `CodeBlock`-Komponente ist ein Async Server Component (nutzt `codeToHtml`). MDX-Component-Mapping für `pre`/`code` muss dies berücksichtigen.
- **Lösung:** Entweder einen Wrapper (`mdx-code-block.tsx`) erstellen, der die bestehende `CodeBlock`-Komponente wrapped, ODER `rehype-pretty-code` Plugin verwenden (Shiki-basiert, läuft zur Build-Zeit). Da Turbopack rehype-Plugins als Strings braucht, prüfen ob `rehype-pretty-code` so funktioniert.
- **Fallback-Lösung:** Custom `pre`-Component in `mdx-components.tsx` die den Code-Inhalt extrahiert und an `CodeBlock` weitergibt. Da MDX Server Components unterstützt, sollte der Async-Aufruf funktionieren.

### Docs-Navigation

- **Docs-Seiten (4 Stück, lt. Epics/Architecture):**
  1. Quick Start (`/docs/quick-start`) — FR28
  2. Endpoints (`/docs/endpoints`) — FR29
  3. Webhooks (`/docs/webhooks`) — FR30
  4. Examples (`/docs/examples`) — FR31
- **Navigation-Pattern:** Sidebar-TOC auf Desktop (links neben Content), Prev/Next am Seitenende. Auf Mobile: Sidebar als Dropdown oder Sheet.
- **Active-State:** Basierend auf `pathname` (gleiches Pattern wie Dashboard-Sidebar und Settings-Tabs).

### Project Structure Notes

- Alle neuen Dateien folgen dem bestehenden `kebab-case.tsx` Naming-Pattern
- Docs-Components in `src/components/docs/` (neues Verzeichnis, analog zu `components/landing/` und `components/dashboard/`)
- MDX-Content in `content/docs/` (bereits als Verzeichnis mit `.gitkeep` vorhanden)
- Docs-Routen in `src/app/(landing)/docs/` (öffentlich, Landing Route Group)
- Keine Konflikte mit bestehender Projektstruktur erkannt

### Learnings aus vorherigen Stories

- **React Hook Form + Zod:** `z.coerce.number()` für Number-Inputs, `valueAsNumber: true` in register(). [Source: Story 4.3 Dev Notes]
- **Touch Targets:** `h-11 lg:h-9` für Buttons, `min-h-11 min-w-11 lg:min-h-0 lg:min-w-0` für Icon-Buttons — 44px Minimum auf Mobile. [Source: Story 4.3 Review Fix M1]
- **Skeleton Loading:** Jede Komponente hat eigene Skeleton-Variante. Sidebar + Header stehen sofort. [Source: Alle Stories]
- **Commit Message Pattern:** "Add [Feature] with review fixes (Story X.Y)" [Source: Git Log]
- **InlineCopyButton:** Für Copy-Aktionen, nicht CopyButton direkt — je nach Kontext. [Source: Story 4.3]

### Git Intelligence

- Letzte Commits: Story 4.3 → 4.2 → 4.1 → 3.5 → 3.4 (alle Epic 3+4 abgeschlossen)
- Alle Commits folgen Pattern: "Add [Feature] with review fixes (Story X.Y)"
- `next.config.ts` ist aktuell leer (nur Type-Import + leeres Config-Objekt) — bereit für MDX-Erweiterung
- `content/docs/` Verzeichnis existiert mit `.gitkeep` — bereit für MDX-Dateien

### References

- [Source: _bmad-output/planning-artifacts/architecture.md#Frontend Architecture] — Docs-Strategie
- [Source: _bmad-output/planning-artifacts/architecture.md#Vollständige Projektverzeichnis-Struktur] — content/docs/ Pfad
- [Source: _bmad-output/planning-artifacts/architecture.md#App Structure & Routing] — Route-Struktur für Docs
- [Source: _bmad-output/planning-artifacts/architecture.md#Component Patterns] — Shiki, Copy-Button, Toast-Pattern
- [Source: _bmad-output/planning-artifacts/architecture.md#Implementation Patterns] — Naming, Structure, Anti-Patterns
- [Source: _bmad-output/planning-artifacts/epics.md#Story 5.1] — Acceptance Criteria
- [Source: _bmad-output/planning-artifacts/epics.md#Epic 5] — FRs covered: FR28, FR29, FR30, FR31
- [Source: https://nextjs.org/docs/app/guides/mdx] — Offizielle @next/mdx Dokumentation (Next.js 16.1.6, Stand 2026-02-20)
- [Source: src/components/shared/code-block.tsx] — Bestehende Shiki-Integration
- [Source: src/components/shared/copy-button.tsx] — Bestehende Copy-Button-Komponente
- [Source: src/components/dashboard/nav-config.ts] — Docs-Link bereits in Sidebar-Navigation
- [Source: src/app/(landing)/layout.tsx] — Landing Layout für Docs-Einbettung
- [Source: _bmad-output/implementation-artifacts/4-3-tenant-configuration-account-settings.md] — Letzte Story Dev Notes & Learnings

## Dev Agent Record

### Agent Model Used

Claude Opus 4.6 (claude-opus-4-6)

### Debug Log References

- Build-Fehler 1: `@tailwindcss/typography` als `@import` statt `@plugin` in Tailwind v4 — behoben mit `@plugin "@tailwindcss/typography"`
- Build-Fehler 2: Turbopack unterstützt keine Template-Literal Dynamic Imports für MDX — behoben mit expliziter Import-Map in `src/lib/docs-content.ts`
- Build-Fehler 3: `@/` Alias resolves zu `./src/`, aber `content/` liegt im Projekt-Root — behoben mit `@content/*` Alias in `tsconfig.json`
- TypeScript-Fehler: `ReactElement.props` Zugriff mit `unknown` Typ — behoben mit explizitem Type-Cast

### Completion Notes List

- AC#1: @next/mdx konfiguriert in `next.config.ts` mit `createMDX()`, remark-gfm + rehype-slug + rehype-autolink-headings als Turbopack-kompatible Strings. Alle 4 MDX-Seiten werden als SSG pre-rendered (bestätigt via Build-Output).
- AC#2: Docs-Index `/docs` zeigt Links zu Quick Start, Endpoints, Webhooks, Examples. Route in `(landing)` Route Group, öffentlich.
- AC#3: Docs-Layout mit Sidebar-TOC auf Desktop (sticky, 224px breit), max-w-4xl zentrierter Content, Prev/Next-Navigation am Seitenende. `@tailwindcss/typography` für Prose-Styling.
- AC#4: Code-Blöcke nutzen bestehende `CodeBlock`-Komponente (Shiki SSR, Dual-Theme) via `MdxPre`-Wrapper. CopyButton mit Inline-Checkmark. Sprach-Labels sichtbar.
- AC#5: Custom MDX-Components: `MdxPre` (CodeBlock mit Copy), Heading Anchors (`h1`-`h4` mit klickbaren `#`-Links, rehype-slug für IDs), `Callout` (info/warning/danger), externe Links mit `target="_blank"`.
- AC#6: Dashboard-Sidebar "Docs"-Link bereits konfiguriert in `nav-config.ts` → navigiert zu `/docs`.

### File List

- `next.config.ts` — MDX Plugin-Konfiguration mit createMDX()
- `tsconfig.json` — @content/* Alias + .mdx include
- `src/app/globals.css` — @tailwindcss/typography Plugin hinzugefügt
- `src/mdx-components.tsx` — MDX Component-Mapping (Headings, Pre, Links, Callout)
- `src/lib/docs.ts` — Docs-Seiten-Konfiguration und Navigation-Helpers
- `src/lib/docs-content.ts` — MDX Import-Map für Turbopack-Kompatibilität
- `src/app/(landing)/docs/page.tsx` — Docs-Index-Seite
- `src/app/(landing)/docs/layout.tsx` — Docs-Layout mit Sidebar
- `src/app/(landing)/docs/[slug]/page.tsx` — Dynamische Docs-Seite mit SSG
- `src/components/docs/mdx-code-block.tsx` — MDX Pre/Code Wrapper für Shiki CodeBlock
- `src/components/docs/callout.tsx` — Callout/Note-Box-Komponente
- `src/components/docs/docs-sidebar.tsx` — Docs-Sidebar-Navigation
- `src/components/docs/docs-nav.tsx` — Prev/Next-Navigation
- `content/docs/quick-start.mdx` — Quick Start Platzhalter-Content
- `content/docs/endpoints.mdx` — Endpoints Platzhalter-Content
- `content/docs/webhooks.mdx` — Webhooks Platzhalter-Content
- `content/docs/examples.mdx` — Examples Platzhalter-Content
- `content/docs/.gitkeep` — ENTFERNT
- `package.json` — Neue Dependencies: @next/mdx, @mdx-js/loader, @mdx-js/react, @types/mdx, remark-gfm, rehype-slug, rehype-autolink-headings, @tailwindcss/typography

## Change Log

- 2026-02-20: Story 5.1 implementiert — MDX-Infrastruktur mit @next/mdx, Docs-Layout mit Sidebar + Prev/Next Navigation, Custom Components (CodeBlock, Callout, Heading Anchors), 4 Placeholder-MDX-Seiten, SSG pre-rendered, @tailwindcss/typography für Prose-Styling
- 2026-02-20: Code Review Fixes — (H1) prose-invert → dark:prose-invert für Light-Mode-Kompatibilität, (H2) Callout-Farben für Light/Dark Mode, (M1) Mobile Docs-Navigation als Sheet hinzugefügt, (M2) extractLanguage mit Shiki bundledLanguages Validierung abgesichert, (M3) Placeholder-MDX-Content auf korrekte PayWatcher-API-Patterns aktualisiert (x-api-key, USDC, masem.at), (L1) aria-current auf aktivem Sidebar-Link, (L2) focus-visible auf Heading-Anchors
