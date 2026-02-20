---
title: 'Brand Assets Integration'
slug: 'brand-assets-integration'
created: '2026-02-16'
status: 'completed'
stepsCompleted: [1, 2, 3, 4]
tech_stack: ['next.js 16', 'react 19', 'tailwind v4', 'shadcn/ui', 'next-themes', 'geist fonts', 'lucide-react']
files_to_modify: ['src/components/brand/eye-logo.tsx', 'src/components/brand/radar-icon.tsx', 'src/components/brand/tokens.ts', 'src/components/brand/index.ts', 'public/brand/radar-favicon.svg', 'public/brand/eye-logo-dark.svg', 'public/brand/eye-logo-light.svg', 'src/app/globals.css', 'src/components/landing/landing-header.tsx', 'src/components/dashboard/sidebar-nav.tsx', 'src/components/dashboard/mobile-sidebar.tsx', 'src/components/dashboard/dashboard-header.tsx', 'src/app/layout.tsx']
code_patterns: ['kebab-case files', 'React FC with Props interface', 'CSS custom properties (OKLch + HEX coexist)', 'barrel exports', 'Server Components default - use client only when needed', 'next-themes useTheme for dark/light']
test_patterns: ['npm run build', 'npm run type-check', 'npm run lint', 'build output: Landing Page must stay Static']
---

# Tech-Spec: Brand Assets Integration

**Created:** 2026-02-16

## Overview

### Problem Statement

PayWatcher nutzt ueberall Plain-Text "PayWatcher" und ein generisches Favicon. Professionelle Brand Assets (EyeLogo, RadarIcon, Farb-Tokens) liegen als Package bereit (`paywatcher-brand-assets.tar.gz`), sind aber nicht ins Projekt eingebunden.

### Solution

Brand-Komponenten (`EyeLogo`, `RadarIcon`, `RadarLoader`) und SVGs ins Projekt kopieren, `--pw-*` CSS Variablen als zusaetzliche Token-Schicht einfuehren (neben bestehendem OKLch-System), Logos in Landing Header, Dashboard Sidebar und Favicon einbauen. Font-Tokens ignorieren (Geist Sans/Mono hat Vorrang per Architecture Decision).

### Scope

**In Scope:**
- Dateien kopieren: `tsx/` → `src/components/brand/`, `svg/` → `public/brand/`
- `--pw-*` CSS Variablen in `globals.css` als zusaetzliche Schicht (HEX-Werte, neben OKLch)
- `EyeLogo` in Landing Header, Dashboard Sidebar, Mobile Sidebar einbauen
- `RadarIcon` als Favicon (SVG) in `app/layout.tsx` Metadata
- Font-Referenzen in `tokens.ts` auf Geist anpassen oder entfernen
- Barrel Export pruefen/anpassen

**Out of Scope:**
- Bestehende OKLch-Tokens in `globals.css` aendern oder harmonisieren
- `RadarLoader` irgendwo einbauen (kommt in Story 3.2 Onboarding)
- Status-Farben Migration (Brand `statusColors` vs. bestehende Status-Tokens)
- Landing Page Redesign mit Brand-Farben
- OG-Image Update

## Context for Development

### Codebase Patterns

- Dateinamen: `kebab-case.tsx` — IMMER
- React Components: FC mit Props Interface
- CSS: Tailwind v4 mit `@theme` Directive und CSS Custom Properties in `globals.css`
- Bestehende Farb-Tokens nutzen OKLch Farbraum — Brand-Tokens kommen als HEX daneben
- Geist Sans + Geist Mono via `next/font/local` (Architecture Decision, nicht aendern)
- Barrel Exports: `index.ts` fuer Component-Verzeichnisse
- Server Components als Default — `'use client'` nur bei Hooks/State/Browser APIs
- `LandingHeader` ist Server Component (kein `'use client'`) — EyeLogo muss als reine Props-Komponente funktionieren (kein useTheme), Variante wird statisch oder per Wrapper gesetzt
- `SidebarNav`, `MobileSidebar`, `DashboardHeader` sind Client Components — koennen useTheme nutzen
- Sidebar: collapsed (60px) vs expanded (200px) — Logo muss in beiden Zustaenden funktionieren
- `DashboardHeader` zeigt "PayWatcher" Text nur auf Mobile (`lg:hidden`) — Logo ebenfalls nur Mobile

### Files to Reference

| File | Purpose |
| ---- | ------- |
| `paywatcher-brand-assets/tsx/eye-logo.tsx` | EyeLogo Komponente (dark/light, skalierbar, showText/showSubtitle) |
| `paywatcher-brand-assets/tsx/radar-icon.tsx` | RadarIcon (static/animated) + RadarLoader |
| `paywatcher-brand-assets/tsx/tokens.ts` | Farb-Tokens, Tailwind map, CSS Vars, Status-Mapping |
| `paywatcher-brand-assets/tsx/index.ts` | Barrel Export |
| `paywatcher-brand-assets/svg/radar-favicon.svg` | Static SVG Favicon |
| `paywatcher-brand-assets/svg/eye-logo-dark.svg` | Eye Logo SVG (dark bg) |
| `paywatcher-brand-assets/svg/eye-logo-light.svg` | Eye Logo SVG (light bg) |
| `src/components/landing/landing-header.tsx` | Landing Header — aktuell Plain-Text "PayWatcher" |
| `src/components/dashboard/sidebar-nav.tsx` | Dashboard Sidebar — aktuell kein Logo |
| `src/components/dashboard/mobile-sidebar.tsx` | Mobile Sidebar — aktuell Plain-Text "PayWatcher" |
| `src/app/layout.tsx` | Root Layout — Favicon Metadata |
| `src/app/globals.css` | Design Tokens — OKLch System, hier kommen --pw-* dazu |

### Technical Decisions

- **Farb-Tokens:** `--pw-*` CSS Variablen als ZUSAETZLICHE Schicht in `globals.css` (HEX-Werte). Bestehende OKLch-Tokens NICHT anfassen.
- **Fonts:** Brand-Kit Font-Tokens (JetBrains Mono/Inter) werden IGNORIERT. `fonts` Export aus `tokens.ts` entfernen oder als Kommentar markieren. Geist Sans + Geist Mono bleibt per Architecture Decision.
- **EyeLogo in LandingHeader (Server Component):** `variant` wird NICHT dynamisch per useTheme gesetzt — stattdessen `variant="dark"` als Default (Dark Mode ist Default). Fuer echtes Theme-Switching: einen kleinen Client-Wrapper `<ThemedEyeLogo />` erstellen oder CSS-basiert loesen (Logo SVG-Farben per CSS vars steuern). EINFACHSTE Loesung: `variant="dark"` hardcoden, da Landing Page primär Dark-Mode ist.
- **EyeLogo in Sidebar/Mobile (Client Components):** Hier kann `useTheme()` direkt verwendet werden fuer dynamische Variante.
- **Favicon:** SVG Favicon (`radar-favicon.svg`) wird nach `public/brand/` kopiert. In `layout.tsx` Metadata `icons.icon` auf `/brand/radar-favicon.svg` setzen. Bestehendes `favicon.ico` BEHALTEN als Fallback.
- **Component Location:** `src/components/brand/` — neues Verzeichnis, konsistent mit `components/landing/`, `components/dashboard/`, `components/shared/`.
- **Sidebar Collapsed State:** Bei `collapsed=true` (60px): `EyeLogo` mit `showText={false}` (nur Icon). Bei expanded: voller Logo mit Text, `showSubtitle={false}` (Platz sparen).

## Implementation Plan

### Tasks

- [x] Task 1: Brand-Komponenten ins Projekt kopieren
  - File: `src/components/brand/eye-logo.tsx`
  - Action: Kopiere `paywatcher-brand-assets/tsx/eye-logo.tsx` nach `src/components/brand/eye-logo.tsx`
  - Notes: Datei 1:1 uebernehmen, keine Aenderungen noetig

- [x] Task 2: RadarIcon Komponente kopieren
  - File: `src/components/brand/radar-icon.tsx`
  - Action: Kopiere `paywatcher-brand-assets/tsx/radar-icon.tsx` nach `src/components/brand/radar-icon.tsx`
  - Notes: Datei 1:1 uebernehmen, RadarLoader bleibt enthalten (wird erst in Story 3.2 verwendet)

- [x] Task 3: Tokens anpassen und kopieren
  - File: `src/components/brand/tokens.ts`
  - Action: Kopiere `paywatcher-brand-assets/tsx/tokens.ts` nach `src/components/brand/tokens.ts`. Danach `fonts` Export entfernen oder auskommentieren (Geist hat Vorrang per Architecture Decision).
  - Notes: `colors`, `twColors`, `cssVars`, `statusColors` bleiben erhalten. `fonts` wird entfernt.

- [x] Task 4: Barrel Export anpassen
  - File: `src/components/brand/index.ts`
  - Action: Kopiere `paywatcher-brand-assets/tsx/index.ts` nach `src/components/brand/index.ts`. `fonts` aus dem Export entfernen (da in Task 3 entfernt).
  - Notes: Finaler Export: `EyeLogo`, `RadarIcon`, `RadarLoader`, `colors`, `twColors`, `cssVars`, `statusColors`

- [x] Task 5: SVG Assets nach public kopieren
  - File: `public/brand/radar-favicon.svg`, `public/brand/eye-logo-dark.svg`, `public/brand/eye-logo-light.svg`
  - Action: Erstelle Verzeichnis `public/brand/` und kopiere alle 3 SVG-Dateien aus `paywatcher-brand-assets/svg/`
  - Notes: Werden fuer Favicon und ggf. statische Referenzen gebraucht

- [x] Task 6: CSS Brand-Tokens in globals.css einfuegen
  - File: `src/app/globals.css`
  - Action: `--pw-*` CSS Custom Properties als zusaetzlichen Block in `:root` einfuegen (nach den bestehenden OKLch-Tokens). Werte: `--pw-accent: #009BB1`, `--pw-verified: #00E5A0`, `--pw-primary: #093236`, `--pw-bg: #0a1a1c`, `--pw-surface: #0d1f21`, `--pw-border: #1a3a3e`, `--pw-text: #e0f0f2`, `--pw-muted: #5a8a8e`, `--pw-disabled: #3a5a5e`, `--pw-error: #ff5555`, `--pw-warning: #f59e0b`
  - Notes: NICHT in bestehende OKLch-Tokens eingreifen. Neuer kommentierter Block `/* PayWatcher Brand Tokens (HEX) */` am Ende von `:root`.

- [x] Task 7: EyeLogo in LandingHeader einbauen
  - File: `src/components/landing/landing-header.tsx`
  - Action: Plain-Text "PayWatcher" im Logo-Link durch `<EyeLogo size={32} variant="dark" showSubtitle={false} />` ersetzen. Import: `import { EyeLogo } from "@/components/brand"`
  - Notes: Server Component — `variant="dark"` hardcoden (Dark Mode ist Default). KEIN `useTheme()`. Landing Page bleibt Static.

- [x] Task 8: EyeLogo in SidebarNav einbauen
  - File: `src/components/dashboard/sidebar-nav.tsx`
  - Action: Logo-Bereich oberhalb der Navigation hinzufuegen. Bei `collapsed=true`: `<EyeLogo size={28} variant={theme === "dark" ? "dark" : "light"} showText={false} />` (nur Icon). Bei `collapsed=false`: `<EyeLogo size={28} variant={theme === "dark" ? "dark" : "light"} showSubtitle={false} />` (Icon + Text). Import `useTheme` von `next-themes` und `EyeLogo` von `@/components/brand`.
  - Notes: Client Component — `useTheme()` verfuegbar. Logo als Link zu `/dashboard`. Muss in 60px (collapsed) und 200px (expanded) funktionieren.

- [x] Task 9: EyeLogo in MobileSidebar einbauen
  - File: `src/components/dashboard/mobile-sidebar.tsx`
  - Action: Plain-Text "PayWatcher" im `SheetTitle` durch `<EyeLogo size={28} variant={theme === "dark" ? "dark" : "light"} showSubtitle={false} />` ersetzen. Import `useTheme` und `EyeLogo`.
  - Notes: Client Component — `useTheme()` verfuegbar.

- [x] Task 10: EyeLogo in DashboardHeader einbauen (Mobile only)
  - File: `src/components/dashboard/dashboard-header.tsx`
  - Action: Plain-Text "PayWatcher" (nur sichtbar auf Mobile via `lg:hidden`) durch `<EyeLogo size={24} variant={theme === "dark" ? "dark" : "light"} showText={false} />` ersetzen. Import `useTheme` und `EyeLogo`.
  - Notes: Client Component. Logo nur auf Mobile sichtbar (`lg:hidden` beibehalten). Nur Icon, kein Text (Platz sparen).

- [x] Task 11: Favicon in layout.tsx Metadata setzen
  - File: `src/app/layout.tsx`
  - Action: In der `metadata` Export `icons` Property hinzufuegen: `icons: { icon: "/brand/radar-favicon.svg" }`. Bestehendes `favicon.ico` in `app/` BEHALTEN als Fallback.
  - Notes: SVG Favicon wird von modernen Browsern bevorzugt, `favicon.ico` greift als Fallback fuer aeltere Browser.

### Acceptance Criteria

- [x] AC 1: Given das Projekt gebaut wird (`npm run build`), when der Build durchlaeuft, then gibt es keine Build-Fehler und die Landing Page ist weiterhin `○ (Static)` im Build Output.

- [x] AC 2: Given ein User die Landing Page oeffnet, when der Header rendert, then ist das EyeLogo (Icon + Wordmark) sichtbar anstelle von Plain-Text "PayWatcher".

- [x] AC 3: Given ein User das Dashboard oeffnet, when die Sidebar expanded ist (200px), then zeigt sie das EyeLogo mit Icon und Wordmark (ohne Subtitle). When die Sidebar collapsed ist (60px), then zeigt sie nur das EyeLogo Icon ohne Text.

- [x] AC 4: Given ein User das Dashboard auf einem Mobile-Geraet oeffnet, when die Mobile Sidebar geoeffnet wird, then ist das EyeLogo im Sheet-Header sichtbar anstelle von Plain-Text "PayWatcher".

- [x] AC 5: Given ein User das Dashboard auf einem Mobile-Geraet oeffnet, when der DashboardHeader rendert, then zeigt er das EyeLogo Icon (ohne Text) anstelle von Plain-Text "PayWatcher".

- [x] AC 6: Given ein User eine beliebige Seite oeffnet, when der Browser-Tab angezeigt wird, then ist das Radar-Icon als Favicon sichtbar.

- [x] AC 7: Given das Projekt mit `npm run type-check` geprueft wird, when der Check laeuft, then gibt es keine TypeScript-Fehler.

- [x] AC 8: Given das Projekt mit `npm run lint` geprueft wird, when der Linter laeuft, then gibt es keine Lint-Fehler.

- [x] AC 9: Given ein User zwischen Dark und Light Mode wechselt im Dashboard, when die Sidebar oder Mobile-Sidebar sichtbar ist, then passt sich das EyeLogo dynamisch an (variant="dark" bzw. variant="light").

## Additional Context

### Dependencies

- Keine neuen npm Packages noetig
- Brand Assets aus `paywatcher-brand-assets.tar.gz` (bereits entpackt)
- `next-themes` bereits installiert (fuer EyeLogo dark/light Variante)

### Testing Strategy

- `npm run build` — keine Build-Fehler
- `npm run type-check` — keine Type-Fehler
- `npm run lint` — keine Lint-Fehler
- Visuell: Logo erscheint in Landing Header, Dashboard Sidebar, Mobile Sidebar
- Visuell: Favicon ist Radar-Icon im Browser-Tab
- Landing Page bleibt `○ (Static)` im Build Output

### Notes

## Review Notes
- Adversarial review completed
- Findings: 13 total, 5 fixed, 8 skipped (noise/out-of-scope)
- Resolution approach: auto-fix
- F1 fixed: ThemedEyeLogo Client-Wrapper fuer LandingHeader (theme-aware statt hardcoded dark)
- F4 fixed: cssVars Dead Code aus tokens.ts entfernt
- F5 fixed: resolvedTheme Fallback auf "dark" (Default-Theme) statt "light"
- F6 fixed: useId() fuer unique SVG Gradient IDs in RadarIcon
- F11 fixed: role="img" auf SVG-Elementen fuer bessere Accessibility

- `RadarLoader` wird NICHT eingebaut — kommt erst in Story 3.2 (Onboarding "Waiting for first payment...")
- `statusColors` aus `tokens.ts` werden NICHT verwendet — bestehende Status-Farben in globals.css haben Vorrang
- Spaetere Harmonisierung (Brand-HEX → OKLch) ist moeglich, aber explizit out of scope
