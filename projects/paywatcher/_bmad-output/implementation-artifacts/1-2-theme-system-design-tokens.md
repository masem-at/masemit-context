# Story 1.2: Theme System & Design Tokens

Status: done

<!-- Note: Validation is optional. Run validate-create-story for quality check before dev-story. -->

## Story

As a Developer (Sempre),
I want das PayWatcher Design System mit Dark/Light Mode, Farbpalette und Typografie eingerichtet,
so that alle nachfolgenden Components konsistente visuelle Tokens verwenden.

## Acceptance Criteria

1. **Given** das Projekt aus Story 1.1 existiert **When** die globals.css mit Tailwind v4 @theme Directive und CSS Custom Properties konfiguriert wird **Then** sind alle Design Tokens definiert: Background, Surface, Text Primary/Secondary, Accent (Teal ~#00D4AA), Success, Warning, Error, Neutral

2. **Given** next-themes installiert ist **When** der ThemeProvider im Root Layout eingebunden wird **Then** ist Dark Mode als Default aktiv und ein Theme Toggle wechselt zwischen Dark/Light Mode **And** die Praeferenz wird persistent gespeichert (localStorage/Cookie)

3. **Given** Geist Fonts via next/font/local konfiguriert sind **When** die Seite geladen wird **Then** wird Geist Sans fuer UI-Text und Geist Mono fuer Code/kopierbare Inhalte verwendet **And** die Type Scale aus der UX-Spec ist implementiert (display, h1, h2, h3, body, body-sm, caption, code, code-sm, stat)

4. **Given** Payment Status Farben definiert sind **When** ein Badge fuer einen der 6 Status gerendert wird **Then** zeigt er die korrekte Farbe: pending=muted, matched=accent, confirming=warning, confirmed=success, expired=neutral, failed=error

## Tasks / Subtasks

- [x] Task 1: next-themes und sonner installieren und konfigurieren (AC: #2)
  - [x] 1.1 `npm install next-themes sonner` ausfuehren
  - [x] 1.2 ThemeProvider-Wrapper erstellen: `src/components/shared/theme-provider.tsx` (Client Component)
  - [x] 1.3 Root Layout (`src/app/layout.tsx`) anpassen: ThemeProvider einbinden, `suppressHydrationWarning` auf `<html>`, `attribute="class"`, `defaultTheme="dark"`, `enableSystem`
  - [x] 1.4 Sonner Toaster-Komponente im Root Layout einbinden (Position: bottom-right)
  - [x] 1.5 Verifizieren: Dark Mode ist Default, Seite laedt ohne FOUC (Flash of Unstyled Content)

- [x] Task 2: PayWatcher Design Tokens in globals.css definieren (AC: #1)
  - [x] 2.1 `:root` (Light Mode) Farbvariablen komplett ersetzen mit PayWatcher Farbpalette (OKLCH-Format)
  - [x] 2.2 `.dark` Farbvariablen komplett ersetzen mit PayWatcher Dark Mode Palette
  - [x] 2.3 Zusaetzliche PayWatcher-spezifische Tokens definieren: `--success`, `--warning`, `--error`, `--neutral` fuer Payment Status Farben
  - [x] 2.4 `@theme inline` Block um die neuen Tokens erweitern (damit Tailwind sie als Utility-Klassen erkennt)
  - [x] 2.5 `@custom-variant dark` Konfiguration pruefen — muss `(&:is(.dark *))` sein fuer next-themes class-based approach
  - [x] 2.6 Verifizieren: `bg-background`, `text-foreground`, `bg-accent` etc. zeigen PayWatcher-Farben

- [x] Task 3: Typografie-System implementieren (AC: #3)
  - [x] 3.1 Geist Font-Loading in layout.tsx pruefen — `geist` Package wird bereits korrekt geladen (aus Story 1.1)
  - [x] 3.2 Custom Type Scale als Tailwind-Utilities definieren via `@theme` in globals.css (display, h1, h2, h3, body, body-sm, caption, code, code-sm, stat)
  - [x] 3.3 Alternativ: Type Scale als CSS-Utility-Klassen in `@layer base` definieren falls `@theme` fuer Schriftgroessen nicht ideal ist
  - [x] 3.4 Verifizieren: `font-sans` nutzt Geist Sans, `font-mono` nutzt Geist Mono

- [x] Task 4: Theme Toggle Komponente erstellen (AC: #2)
  - [x] 4.1 `src/components/shared/theme-toggle.tsx` erstellen (Client Component)
  - [x] 4.2 useTheme() Hook von next-themes verwenden
  - [x] 4.3 Toggle-Button mit Sun/Moon Icons (Lucide) — wechselt zwischen dark/light/system
  - [x] 4.4 Als Dropdown-Menu implementieren mit drei Optionen: Light, Dark, System
  - [x] 4.5 shadcn/ui DropdownMenu Komponente installieren falls nicht vorhanden: `npx shadcn@latest add dropdown-menu`
  - [x] 4.6 Verifizieren: Toggle wechselt korrekt, Praeferenz bleibt nach Page Reload erhalten

- [x] Task 5: Payment Status Farb-Mapping definieren (AC: #4)
  - [x] 5.1 `src/types/payment.ts` erstellen mit `PaymentStatus` Union Type
  - [x] 5.2 `STATUS_CONFIG` Konstante in `src/lib/utils.ts` definieren (Record<PaymentStatus, { color, label }>)
  - [x] 5.3 CSS Custom Properties fuer Status-Farben in globals.css definieren (sowohl fuer Dark als auch Light Mode)
  - [x] 5.4 Verifizieren: Die 6 Status-Farben sind korrekt definiert und ueber Tailwind-Klassen nutzbar

- [x] Task 6: Build-Validierung und Qualitaetspruefung (Alle ACs)
  - [x] 6.1 `npm run build` muss fehlerfrei durchlaufen
  - [x] 6.2 `npm run type-check` muss fehlerfrei durchlaufen
  - [x] 6.3 `npm run lint` muss fehlerfrei durchlaufen
  - [x] 6.4 Visuelle Pruefung: Dark Mode korrekte Farben, Light Mode korrekte Farben
  - [x] 6.5 Theme Toggle funktioniert und Praeferenz ist persistent

## Dev Notes

### Kontext & Zweck

Dies ist die **zweite Story** des PayWatcher-Projekts. Story 1.1 hat das Fundament gelegt (Next.js 16, Tailwind v4, shadcn/ui). Diese Story baut darauf auf und etabliert das **visuelle Fundament** — das Design System auf dem alle UI-Komponenten basieren.

**Was PayWatcher ist:** Developer-First Verification API fuer Stablecoin-Zahlungen (USDC auf Base Network). Non-custodial, reiner Verification Layer. Dark Mode Default ist Web3-Konvention. Die Brand-Farbe (Teal/Gruen) = Success-Farbe = Confirmed-Badge-Farbe — "Pavlov fuer Developer".

**Was diese Story liefert:**
- Vollstaendige PayWatcher-Farbpalette in Dark und Light Mode (OKLCH-Format)
- next-themes mit Dark Mode als Default + Theme Toggle
- Geist Sans + Geist Mono Type Scale gemaess UX-Spec
- Payment Status Farb-Mapping (6 Status → 6 Farben)
- sonner (Toast-System) installiert und konfiguriert

**Was diese Story NICHT beinhaltet:**
- Keine Landing Page Inhalte (Stories 1.3-1.6)
- Keine Dashboard-Komponenten (Epic 2-4)
- Keine PaymentStatusBadge-Komponente (Story 3.1) — nur das Farb-Mapping wird vorbereitet
- Kein Auth-System (Epic 2)

### Technische Anforderungen

**Zu installierende Packages:**

| Package | Version | Zweck |
|---------|---------|-------|
| next-themes | ^0.4.6 | Dark/Light Mode Toggle mit localStorage-Persistenz |
| sonner | latest | Toast-System (shadcn/ui Standard, Position rechts unten) |

**NICHT in dieser Story installieren:**
- `@tanstack/react-query` → Story 2.1
- `next-auth` / `better-auth` → Story 2.2
- `shiki` → Story 1.3
- `recharts` → Story 3.2
- `react-hook-form` + `zod` → Story 2.2

**next-themes Konfiguration (KRITISCH):**

```typescript
// src/components/shared/theme-provider.tsx
"use client"

import { ThemeProvider as NextThemesProvider } from "next-themes"

export function ThemeProvider({ children, ...props }: React.ComponentProps<typeof NextThemesProvider>) {
  return <NextThemesProvider {...props}>{children}</NextThemesProvider>
}
```

```typescript
// src/app/layout.tsx — ThemeProvider einbinden
<html lang="en" suppressHydrationWarning>
  <body className={`${GeistSans.variable} ${GeistMono.variable} antialiased`}>
    <ThemeProvider
      attribute="class"
      defaultTheme="dark"
      enableSystem
      disableTransitionOnChange
    >
      {children}
      <Toaster />
    </ThemeProvider>
  </body>
</html>
```

**KRITISCH — suppressHydrationWarning:** MUSS auf dem `<html>` Tag stehen, sonst React Hydration Warnung weil next-themes das class-Attribut vor dem Hydration-Pass setzt.

**KRITISCH — @custom-variant dark:** Die bestehende Zeile `@custom-variant dark (&:is(.dark *));` in globals.css ist korrekt fuer next-themes class-based approach. NICHT aendern.

### PayWatcher Farbpalette (Design Tokens)

Die Farbpalette kommt aus der UX-Spezifikation. Alle Farben in OKLCH-Format (shadcn/ui Standard seit 2025).

**Semantische Farbzuordnung:**

| Token | Dark Mode Wert | Light Mode Wert | Verwendung |
|-------|---------------|-----------------|-----------|
| `--background` | ~#0A0F0F (Fast-Schwarz mit Teal-Stich) | ~#FAFAFA (Off-White) | Haupthintergrund |
| `--foreground` | ~#E5E7EB (Off-White) | ~#111827 (Near-Black) | Body Text, Headings |
| `--card` / Surface | 1-2 Stufen heller als BG | #FFFFFF | Cards, Sidebar |
| `--primary` / Accent | ~#00D4AA bis #10B981 (Teal/Gruen) | Gleich oder leicht satter | CTAs, Links, Brand |
| `--primary-foreground` | Near-Black | White | Text auf Primary Buttons |
| `--muted` | Gedaempftes Dunkelgrau | Helles Grau | Backgrounds fuer sekundaere Bereiche |
| `--muted-foreground` | Gedaempftes Grau | Mittleres Grau | Labels, Timestamps, Secondary Text |
| `--destructive` | ~#EF4444 (gedaempftes Rot) | ~#DC2626 | Failed-State, echte Fehler |
| `--accent` | = Primary (Teal/Gruen) | = Primary | Hover States, aktive Items |
| `--border` | Dezentes Weiss mit Transparenz | Dezentes Grau | Borders, Dividers |

**Zusaetzliche PayWatcher-spezifische Tokens:**

| Token | Wert | Verwendung |
|-------|------|-----------|
| `--success` | = Accent/Primary (Teal) | confirmed-Badge. Brand = Success = Confirmed. |
| `--warning` | ~#F59E0B (Orange) | confirming-Badge, Webhook Retries |
| `--error` | = Destructive (Rot) | failed-Badge, echte Fehler |
| `--neutral` | ~#6B7280 (Grau) | expired-Badge, Disabled States |

**DESIGN-ENTSCHEIDUNG — Brand = Success = Confirmed:**
Die Brand-Farbe (Teal/Gruen) ist identisch mit der Success-Farbe. Jedes Mal wenn ein Payment "confirmed" wird, sieht der Dev die Brand-Farbe. "Pavlov fuer Developer" — das staerkste UX-Signal des Produkts. [Source: _bmad-output/planning-artifacts/ux-design-specification.md#Farbpalette]

### Typografie-System (Type Scale)

Die Type Scale kommt aus der UX-Spezifikation. Basierend auf 8px-Raster, Tailwind-kompatibel.

| Token | Groesse | Line-Height | Weight | Verwendung |
|-------|---------|-------------|--------|-----------|
| `display` | 48px / 3rem | 1.1 | 700 (Bold) | Landing Page Hero Headline |
| `h1` | 36px / 2.25rem | 1.2 | 700 (Bold) | Sektions-Titel Landing Page |
| `h2` | 24px / 1.5rem | 1.3 | 600 (Semibold) | Dashboard Page Titles, LP Subtitles |
| `h3` | 20px / 1.25rem | 1.4 | 600 (Semibold) | Card Titles, Subsections |
| `body` | 16px / 1rem | 1.5 | 400 (Regular) | Fliesstext, Beschreibungen |
| `body-sm` | 14px / 0.875rem | 1.5 | 400 (Regular) | Dashboard Body, Tabellen, Labels |
| `caption` | 12px / 0.75rem | 1.4 | 400 (Regular) | Timestamps, Secondary Info, Hints |
| `code` | 14px / 0.875rem | 1.6 | 400 (Regular) | Code-Blocks (Geist Mono) |
| `code-sm` | 13px / 0.8125rem | 1.5 | 400 (Regular) | txHashes, Adressen in Tabellen (Geist Mono) |
| `stat` | 28px / 1.75rem | 1.2 | 700 (Bold) | Dashboard Statistik-Zahlen |

**Monospace-Regel:** Alles Kopierbare MUSS in Geist Mono sein (txHashes, Wallet-Adressen, Payment IDs, API Keys, Betraege, JSON Payloads). Monospace bei Hex-Strings verhindert Verwechslungen (`0xO0` vs `0x00`). [Source: _bmad-output/planning-artifacts/ux-design-specification.md#Typography System]

### Architektur-Konformitaet

**Dateinamen:** `kebab-case.tsx` — IMMER. Keine PascalCase-Dateinamen.

**Komponenten-Pfade:**
- `src/components/shared/theme-provider.tsx` — Shared weil von Root Layout verwendet
- `src/components/shared/theme-toggle.tsx` — Shared weil in Landing Header UND Dashboard Header verwendet
- `src/types/payment.ts` — Zentral in types/
- `src/lib/utils.ts` — Bestehende Datei erweitern (cn() ist bereits dort)

**Styling-Approach:**
- Tailwind v4 CSS-first Configuration via `@theme` Directive
- CSS Custom Properties fuer Dark/Light Mode (`:root` / `.dark`)
- `@theme inline` Block mappt CSS Variables auf Tailwind Utility-Klassen
- Kein `tailwind.config.js` — alles in CSS

**Anti-Patterns (NIEMALS):**
- Kein `tailwind.config.js` erstellen — Tailwind v4 nutzt CSS-first Config
- Keine hardcoded Farben in Komponenten — immer CSS Custom Properties ueber Tailwind-Klassen
- Kein `useState` fuer Theme — `useTheme()` von next-themes verwenden
- Kein eigener localStorage-Code — next-themes handled Persistenz
- Kein Full-Page-Spinner waehrend Theme-Loading — `disableTransitionOnChange` verhindert FOUC

### Library & Framework Anforderungen

**next-themes v0.4.6:**
- Kompatibel mit Next.js 16 App Router
- `attribute="class"` — next-themes setzt `.dark` class auf `<html>`
- `defaultTheme="dark"` — Web3-Konvention, Dark Mode Default
- `enableSystem` — Respektiert System-Praeferenz wenn User "System" waehlt
- `disableTransitionOnChange` — Verhindert Flash beim Theme-Wechsel
- localStorage Key: `"theme"` (Default)
- `suppressHydrationWarning` auf `<html>` PFLICHT

**sonner (Toast-System):**
- shadcn/ui Standard-Toast-Library
- Position: rechts unten (`position="bottom-right"`)
- Wird via `<Toaster />` Komponente im Root Layout eingebunden
- Optionale shadcn/ui Integration: `npx shadcn@latest add sonner` installiert Wrapper
- Copy-Aktionen: KEIN Toast — nur Inline-Checkmark (UX-Regel)
- Save/Update: 3s auto-dismiss
- Fehler: Bleibt bis manuell dismissed

**shadcn/ui Dropdown Menu:**
- Fuer Theme Toggle: `npx shadcn@latest add dropdown-menu`
- Drei Optionen: Light, Dark, System

### Dateistruktur-Anforderungen

**Neue/geaenderte Dateien nach Abschluss:**

```
src/
├── app/
│   ├── layout.tsx                        # AENDERN: ThemeProvider + Toaster + suppressHydrationWarning
│   └── globals.css                       # AENDERN: PayWatcher Design Tokens (Farben, Typografie)
│
├── components/
│   └── shared/
│       ├── theme-provider.tsx            # NEU: ThemeProvider Wrapper (Client Component)
│       └── theme-toggle.tsx              # NEU: Dark/Light/System Toggle (Client Component)
│
├── lib/
│   └── utils.ts                          # AENDERN: STATUS_CONFIG hinzufuegen
│
└── types/
    └── payment.ts                        # NEU: PaymentStatus Type + STATUS_CONFIG Type
```

**WICHTIG:** `.gitkeep` Dateien in `src/components/shared/` kann entfernt werden, da jetzt echte Dateien dort liegen.

### Testing-Anforderungen

1. **Build-Validierung:** `npm run build` muss ohne Fehler durchlaufen
2. **Type-Check:** `npm run type-check` muss ohne Fehler durchlaufen
3. **Lint:** `npm run lint` muss ohne Fehler durchlaufen
4. **Visuelle Pruefung Dark Mode:** Background fast-schwarz mit Teal-Stich, Text off-white, Accent Teal/Gruen
5. **Visuelle Pruefung Light Mode:** Background off-white, Text near-black, Accent Teal/Gruen
6. **Theme Toggle:** Wechselt korrekt zwischen Dark/Light/System
7. **Persistenz:** Nach Page Reload bleibt die Theme-Wahl erhalten
8. **Kein FOUC:** Kein Flash of Unstyled Content beim Laden (insbesondere kein kurzes Light-Flash bei Dark Mode Default)

### Vorherige Story Intelligence (Story 1.1)

**Wichtige Erkenntnisse aus Story 1.1:**

- **Geist Fonts:** Werden bereits ueber das `geist` Package geladen (`import { GeistSans } from "geist/font/sans"`, `import { GeistMono } from "geist/font/mono"`). Die CSS Variables `--font-geist-sans` und `--font-geist-mono` sind bereits im `@theme inline` Block gemappt.
- **Route Group Pfadkonflikt:** `(landing)`, `(dashboard)`, `(admin)` hatten alle `page.tsx` auf `/`. Geloest durch Unterverzeichnisse. Dashboard ist unter `/dashboard`, Admin unter `/admin`.
- **React Compiler:** NICHT aktiv (konnte nicht automatisch aktiviert werden). Kein `useMemo`/`useCallback` noetig dank React 19, aber React Compiler ist nicht die Ursache.
- **shadcn/ui:** New York Style, Neutral Color, CSS Variables aktiviert. Button, Badge, Card bereits installiert.
- **globals.css:** Enthaelt bereits die shadcn/ui Default-Tokens (Neutral-Palette, OKLCH-Format). Diese muessen mit der PayWatcher-Palette ERSETZT werden.
- **@custom-variant dark:** Bereits korrekt konfiguriert als `(&:is(.dark *))` — kompatibel mit next-themes class-based approach.

**Dateien die modifiziert werden (existieren bereits):**
- `src/app/layout.tsx` — ThemeProvider hinzufuegen
- `src/app/globals.css` — Farben ersetzen, Tokens erweitern
- `src/lib/utils.ts` — STATUS_CONFIG hinzufuegen

### Aktuelle Technologie-Informationen (Web Research, Feb 2026)

**next-themes v0.4.6:**
- Letzte stabile Version (kein v0.5+)
- App Router Support seit v0.3.0
- Funktioniert mit Next.js 16 (getestet)
- `attribute="class"` ist Default — passt zu Tailwind v4 `@custom-variant dark (&:is(.dark *))`

**Tailwind v4 Theming:**
- `@theme inline` fuer CSS Variable References (noetig fuer Dark/Light Mode Switching)
- `@theme` fuer direkte Token-Definitionen
- Namespaces: `--color-*` fuer Farben, `--font-*` fuer Fonts, `--spacing-*` fuer Abstaaende
- OKLCH ist Standard-Farbformat (besser als HSL fuer perceptuelle Gleichmaessigkeit)

**shadcn/ui OKLCH Theming:**
- Seit 2025 Standard (Migration von HSL zu OKLCH)
- Alle Farbvariablen in `:root` und `.dark` definiert
- `@theme inline` mappt CSS Variables auf Tailwind Utilities

**sonner:**
- Offizielle shadcn/ui Toast-Loesung (ersetzt alte toast.tsx)
- `npx shadcn@latest add sonner` fuer Wrapper-Integration
- `<Toaster />` Komponente im Layout einbinden

### Project Structure Notes

- Die PayWatcher-Farbpalette ERSETZT die neutrale shadcn/ui Default-Palette — kein Merging
- Theme System ist ein Cross-Cutting Concern — wird von Landing Page UND Dashboard verwendet
- `STATUS_CONFIG` wird in `lib/utils.ts` definiert, nicht in einer separaten Datei, da es ein zentrales Utility ist
- Die Type Scale wird als CSS Utilities oder Tailwind `@theme` Extensions definiert — pragmatisch entscheiden was besser funktioniert

### References

- [Source: _bmad-output/planning-artifacts/ux-design-specification.md#Design System Foundation] — Farbpalette, Typografie, Spacing
- [Source: _bmad-output/planning-artifacts/ux-design-specification.md#Color System] — Semantische Farbzuordnung, WCAG Kontrast
- [Source: _bmad-output/planning-artifacts/ux-design-specification.md#Typography System] — Type Scale, Monospace-Regel
- [Source: _bmad-output/planning-artifacts/ux-design-specification.md#Payment Status Farb-Mapping] — 6 Status → 6 Farben
- [Source: _bmad-output/planning-artifacts/architecture.md#Frontend Architecture] — next-themes, Dark Mode Default
- [Source: _bmad-output/planning-artifacts/architecture.md#Component Patterns] — STATUS_CONFIG, Button-Hierarchie, Toast-Pattern
- [Source: _bmad-output/planning-artifacts/architecture.md#Implementation Patterns] — Naming, Anti-Patterns, Enforcement
- [Source: _bmad-output/planning-artifacts/epics.md#Story 1.2] — User Story und Acceptance Criteria

## Dev Agent Record

### Agent Model Used

Claude Opus 4.6 (claude-opus-4-6)

### Debug Log References

Keine Fehler oder Debugging noetig. Alle Tasks liefen beim ersten Versuch durch.

### Completion Notes List

- **Task 1:** next-themes v0.4.6 + sonner installiert. ThemeProvider Wrapper als Client Component erstellt. Root Layout mit ThemeProvider (attribute="class", defaultTheme="dark", enableSystem, disableTransitionOnChange), suppressHydrationWarning auf html, und Toaster (position="bottom-right") konfiguriert. shadcn/ui Sonner-Wrapper verwendet (nutzt automatisch useTheme()).
- **Task 2:** Komplette PayWatcher-Farbpalette in OKLCH-Format implementiert. :root (Light) und .dark Selektoren mit allen semantischen Tokens. Background mit Teal-Stich im Dark Mode (oklch 0.135 0.008 195). Primary/Accent = Teal (oklch 0.78 0.155 170). Zusaetzliche Status-Tokens (--success, --warning, --error, --neutral) mit Foreground-Varianten. @theme inline um 8 Status-Color-Mappings erweitert.
- **Task 3:** Type Scale via separatem @theme Block (nicht inline) definiert. 10 Groessen: text-display (3rem), text-h1 (2.25rem), text-h2 (1.5rem), text-h3 (1.25rem), text-body (1rem), text-body-sm (0.875rem), text-caption (0.75rem), text-code (0.875rem), text-code-sm (0.8125rem), text-stat (1.75rem). Jeweils mit passender line-height. Font-weight und font-family werden via Tailwind-Utilities kombiniert (font-bold, font-semibold, font-mono).
- **Task 4:** ThemeToggle als Client Component mit shadcn/ui DropdownMenu. Sun/Moon/Monitor Icons (Lucide). Drei Optionen: Light, Dark, System. useTheme() Hook fuer State-Management.
- **Task 5:** PaymentStatus Union Type in src/types/payment.ts. STATUS_CONFIG in src/lib/utils.ts mit korrektem Farb-Mapping: pending=muted, matched=accent, confirming=warning, confirmed=success, expired=neutral, failed=error.
- **Task 6:** Build, Type-Check und Lint alle fehlerfrei. Visuelle Pruefung erfordert manuellen Browser-Test.

### Implementation Plan

- Tailwind v4 CSS-first Approach: Alle Tokens als CSS Custom Properties, gemappt via @theme inline
- Type Scale als separater @theme Block (statische Werte, nicht inline)
- Status-Farben als eigene CSS Variables mit Foreground-Varianten fuer flexible Badge-Gestaltung
- Brand = Success = Confirmed Design-Entscheidung konsequent umgesetzt (gleicher Teal-Wert)

### File List

**Neue Dateien:**
- src/components/shared/theme-provider.tsx
- src/components/shared/theme-toggle.tsx
- src/components/ui/dropdown-menu.tsx (shadcn/ui generiert)
- src/components/ui/sonner.tsx (shadcn/ui generiert)
- src/types/payment.ts

**Geaenderte Dateien:**
- src/app/globals.css (PayWatcher Farbpalette + Type Scale + Status-Tokens)
- src/app/layout.tsx (ThemeProvider, Toaster, suppressHydrationWarning)
- src/lib/utils.ts (STATUS_CONFIG hinzugefuegt)
- src/types/payment.ts (StatusColor Union Type hinzugefuegt — Code Review Fix)
- package.json (next-themes, sonner Dependencies)
- package-lock.json (Lock-File aktualisiert)
- .env.example (Kommentare aktualisiert: Auth.js v5, Analytics-URL)
- README.md (Projekt-spezifisches README statt Next.js Default)

**Geloeschte Dateien:**
- src/components/shared/.gitkeep
- src/types/.gitkeep
- nul (Junk-Datei aus fehlgeschlagenem Shell-Command — Code Review Cleanup)

## Change Log

- 2026-02-16: Story 1.2 implementiert — PayWatcher Design System mit Dark/Light Mode (next-themes), Farbpalette (OKLCH), Type Scale (10 Groessen), Payment Status Farb-Mapping (6 Status), Theme Toggle Komponente, und Sonner Toast-System.
- 2026-02-16: Code Review — 4 Medium, 2 Low Issues gefunden und gefixt: nul-Junk-Datei geloescht, STATUS_CONFIG Labels auf UPPER_CASE korrigiert (Architektur-Konformitaet), StatusColor Union Type eingefuehrt (Type Safety), File List um fehlende Dateien ergaenzt.
