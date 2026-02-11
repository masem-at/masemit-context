# Story 4.2: SEO, Meta Tags & Social Sharing

Status: done

## Story

As a **founder (Mario)**,
I want kurzum.app to be discoverable via Google search and look professional when shared on LinkedIn,
so that organic traffic and LinkedIn Ads campaigns drive qualified visitors to the page.

## Acceptance Criteria

1. **Given** the landing page content exists (Epic 2) **When** metadata is configured in `app/layout.tsx` **Then** the page title is set, meta description targets primary keywords, `<html lang="de">` is verified, and canonical URL points to `https://kurzum.app`

2. **Given** metadata is configured **When** OG tags are implemented **Then** `og:title`, `og:description`, `og:image`, `og:url`, `og:type` are set **And** `twitter:card` is set to `"summary_large_image"` **And** `og:locale` is set to `"de_AT"`

3. **Given** OG tags exist **When** `app/opengraph-image.tsx` is implemented **Then** it generates a 1200x630 OG image dynamically using Next.js `ImageResponse` **And** the image includes the kurzum logo (wordmark + waveform) centered **And** the image uses brand colors (Orange on Anthracite) **And** the image renders correctly in LinkedIn post preview

4. **Given** SEO metadata is configured **When** structured data is added **Then** JSON-LD for Organization (masemIT e.U.) and Product (kurzum.app) is embedded

5. **Given** search engine indexing is needed **When** `app/robots.ts` is implemented **Then** it allows `/`, `/impressum`, `/datenschutz` **And** disallows `/api/*`, `/confirmed` **When** `app/sitemap.ts` is implemented **Then** it lists all public pages with `lastModified` dates

6. **Given** the page is deployed **When** a Lighthouse audit is run **Then** the SEO score is ≥90

## Tasks / Subtasks

- [x] Task 1: Enhance metadata in `app/layout.tsx` (AC: #1, #2)
  - [x] 1.1 Add `metadataBase: new URL("https://kurzum.app")` to Metadata
  - [x] 1.2 Add `alternates: { canonical: "/" }` for canonical URL
  - [x] 1.3 Add `openGraph` object: title, description, url, siteName, locale `"de_AT"`, type `"website"`, images (auto-resolved from `opengraph-image.tsx`)
  - [x] 1.4 Add `twitter: { card: "summary_large_image" }`
  - [x] 1.5 Add `keywords` array with target keywords (see Dev Notes)
  - [x] 1.6 Verify `<html lang="de">` is set (already exists — confirmed)

- [x] Task 2: Create dynamic OG image `app/opengraph-image.tsx` (AC: #3)
  - [x] 2.1 Create `app/opengraph-image.tsx` using Next.js `ImageResponse` API
  - [x] 2.2 Export `size = { width: 1200, height: 630 }`, `contentType = "image/png"`, `alt` string
  - [x] 2.3 Render brand elements: wordmark "kurzum." + waveform SVG path, centered
  - [x] 2.4 Use brand colors: Anthracite `#1C1917` background, Orange `#F97316` for logo/text
  - [x] 2.5 Add tagline below logo: "Sprich statt tipp." in Stone 50 color
  - [x] 2.6 Font: Uses Satori default font (woff2 not supported by Satori, runtime=nodejs with force-dynamic)

- [x] Task 3: Add JSON-LD structured data (AC: #4)
  - [x] 3.1 Create `components/seo/json-ld.tsx` — Server Component exporting `<JsonLd />`
  - [x] 3.2 Embed Organization schema: name, url, logo, address (masemIT e.U., Alleegasse 26, 3851 Kautzen, AT), contactPoint (contact@masem.at)
  - [x] 3.3 Embed SoftwareApplication schema: name "kurzum", description, offers (€10/user/month), operatingSystem, applicationCategory
  - [x] 3.4 Add `<JsonLd />` to `app/page.tsx` (NOT layout.tsx — only on landing page)

- [x] Task 4: Create `app/robots.ts` (AC: #5)
  - [x] 4.1 Export default function returning `MetadataRoute.Robots`
  - [x] 4.2 Allow: `/`, `/impressum`, `/datenschutz`
  - [x] 4.3 Disallow: `/api/*`, `/confirmed`
  - [x] 4.4 Add sitemap URL: `https://kurzum.app/sitemap.xml`

- [x] Task 5: Create `app/sitemap.ts` (AC: #5)
  - [x] 5.1 Export default function returning `MetadataRoute.Sitemap`
  - [x] 5.2 Include pages: `/` (priority 1.0), `/impressum` (priority 0.3), `/datenschutz` (priority 0.3)
  - [x] 5.3 Set `lastModified` to current build date
  - [x] 5.4 Set `changeFrequency` appropriately (monthly for landing, yearly for legal)

- [x] Task 6: Verify Build & Lint (AC: #6)
  - [x] 6.1 Run `pnpm build` — zero errors
  - [x] 6.2 Run `pnpm lint` — zero errors
  - [x] 6.3 Verify OG image renders at `/opengraph-image` route (build output confirms ƒ dynamic generation)

## Dev Notes

### Critical: Existing Metadata — EXTEND, DO NOT REPLACE

`app/layout.tsx` already has:
```ts
export const metadata: Metadata = {
  title: "kurzum. — Sprich statt tipp. Die KI-App für Handwerksbetriebe.",
  description: "kurzum macht Kommunikation im Handwerk intelligent. Voice-First, DSGVO-konform, ab €10/User. Jetzt für den Pilotbetrieb anmelden.",
  icons: { /* favicon config — keep as-is */ },
};
```
**EXTEND this object** — add `metadataBase`, `alternates`, `openGraph`, `twitter`, `keywords`. Do NOT change existing `title`, `description`, or `icons`.

### Critical: Existing Files — USE, DO NOT RECREATE

| File | Exports | How to Use |
|------|---------|------------|
| `app/layout.tsx` | Root layout with Metadata | EXTEND metadata — add OG/Twitter/canonical fields |
| `app/page.tsx` | Home page | ADD `<JsonLd />` import here |
| `components/landing/logo.tsx` | `Logo` component | REFERENCE for brand specs (colors, sizing) |
| `components/landing/waveform-icon.tsx` | `WaveformIcon` | REFERENCE SVG path for OG image |
| `app/impressum/page.tsx` | Impressum page | EXISTS — include in sitemap |
| `app/datenschutz/page.tsx` | Datenschutz page | EXISTS — include in sitemap |
| `app/confirmed/page.tsx` | Confirmation page | EXISTS — EXCLUDE from sitemap, disallow in robots |

### OG Image Technical Details

Next.js `ImageResponse` (from `next/og`) generates images at build/request time. Key constraints:
- **No JSX component imports** — must inline all visual elements as JSX within the `ImageResponse`
- **Font loading**: Use `fetch(new URL("../public/fonts/inter-var.woff2", import.meta.url))` then pass to `fonts` option
- **SVG in ImageResponse**: Inline SVG elements directly (no `<img>` tags for SVGs)
- **Waveform SVG**: Extract the SVG path data from `components/landing/waveform-icon.tsx` and inline it
- **File convention**: `opengraph-image.tsx` in `app/` is auto-discovered by Next.js — no manual og:image URL needed

### Target Keywords

From requirement §6.3:
- "Handwerker App"
- "Baustellen Kommunikation"
- "DSGVO Messenger Handwerk"
- "Monteur App Österreich"

### JSON-LD Structured Data

Use `<script type="application/ld+json">` in a Server Component. Two schemas:

**Organization:**
```json
{
  "@context": "https://schema.org",
  "@type": "Organization",
  "name": "masemIT e.U.",
  "url": "https://kurzum.app",
  "logo": "https://kurzum.app/opengraph-image",
  "address": {
    "@type": "PostalAddress",
    "streetAddress": "Alleegasse 26",
    "addressLocality": "Kautzen",
    "postalCode": "3851",
    "addressCountry": "AT"
  },
  "contactPoint": {
    "@type": "ContactPoint",
    "email": "contact@masem.at",
    "contactType": "customer service"
  }
}
```

**SoftwareApplication:**
```json
{
  "@context": "https://schema.org",
  "@type": "SoftwareApplication",
  "name": "kurzum",
  "description": "KI-gestützte Kommunikations-App für Handwerksbetriebe. Voice-First, DSGVO-konform.",
  "url": "https://kurzum.app",
  "applicationCategory": "BusinessApplication",
  "operatingSystem": "Web",
  "offers": {
    "@type": "Offer",
    "price": "10",
    "priceCurrency": "EUR",
    "description": "Pro Nutzer/Monat"
  }
}
```

### robots.ts Pattern

```ts
import type { MetadataRoute } from "next";

export default function robots(): MetadataRoute.Robots {
  return {
    rules: [
      {
        userAgent: "*",
        allow: ["/", "/impressum", "/datenschutz"],
        disallow: ["/api/", "/confirmed"],
      },
    ],
    sitemap: "https://kurzum.app/sitemap.xml",
  };
}
```

### sitemap.ts Pattern

```ts
import type { MetadataRoute } from "next";

export default function sitemap(): MetadataRoute.Sitemap {
  return [
    { url: "https://kurzum.app", lastModified: new Date(), changeFrequency: "monthly", priority: 1.0 },
    { url: "https://kurzum.app/impressum", lastModified: new Date(), changeFrequency: "yearly", priority: 0.3 },
    { url: "https://kurzum.app/datenschutz", lastModified: new Date(), changeFrequency: "yearly", priority: 0.3 },
  ];
}
```

### Brand Colors for OG Image

| Token | Hex | Usage in OG Image |
|-------|-----|-------------------|
| `--color-primary` | `#1C1917` (Anthracite/Stone 900) | Background |
| `--color-accent` | `#F97316` (Orange 500) | Logo wordmark + waveform |
| `--color-bg-light` | `#FAFAF9` (Stone 50) | Tagline text |

### Learnings from Story 4.1

- **Metadata export pattern**: `export const metadata: Metadata = { title: "kurzum. — ..." }`
- **Server Components only** — no `"use client"` needed for any SEO file
- **ESLint**: Internal links must use `<Link>` from `next/link` (not relevant here, but awareness)
- **Build verification**: Always run `pnpm build && pnpm lint` as final step

### Project Structure Notes

- All new files follow existing `app/` directory convention
- `app/robots.ts`, `app/sitemap.ts`, `app/opengraph-image.tsx` — all at root of `app/`
- `components/seo/json-ld.tsx` — new `seo/` subdirectory under `components/`
- No changes to `tsconfig.json`, `package.json`, or any configuration files needed
- `next/og` is included with Next.js — no additional package install required

### References

- [Source: _bmad-output/planning-artifacts/epics.md — Story 4.2, lines 621-663]
- [Source: _bmad-output/planning-artifacts/architecture.md — app/ structure, line 609-611; requirements mapping, line 820]
- [Source: docs/_masemIT/requirements/requirement-kurzum-landing-page-2026-02-10.md — §6.3 SEO]
- [Source: _bmad-output/planning-artifacts/ux-design-specification.md — Logo specs, color tokens, typography]

## Dev Agent Record

### Agent Model Used

Claude Opus 4.6

### Debug Log References

- OG Image build error: `fetch` with `import.meta.url` fails in Turbopack prerender (`not implemented... yet...`)
- OG Image build error: Satori rejects woff2 format (`Unsupported OpenType signature wOF2`)
- OG Image build error: Satori rejects OTF format (`Unsupported OpenType signature`)
- Resolution: Fetch Inter Bold as woff from Google Fonts API (old User-Agent trick for woff format), use `revalidate = 86400` for daily cache with graceful fallback to Satori default font

### Completion Notes List

- Task 1: Extended `app/layout.tsx` Metadata with `metadataBase`, `alternates.canonical`, `openGraph` (title, description, url, siteName, locale de_AT, type website), `twitter` (summary_large_image), `keywords` (4 target keywords). Existing title/description/icons preserved.
- Task 2: Created `app/opengraph-image.tsx` with 1200x630 dynamic OG image. Waveform SVG inlined from waveform-icon.tsx. Anthracite background, Orange logo/waveform, Stone 50 tagline. Uses dynamic runtime due to Satori woff2 incompatibility.
- Task 3: Created `components/seo/json-ld.tsx` with Organization (masemIT e.U.) + SoftwareApplication (kurzum) schemas. Embedded via `<JsonLd />` in `app/page.tsx`.
- Task 4: Created `app/robots.ts` — allows public pages, disallows /api/ and /confirmed, includes sitemap URL.
- Task 5: Created `app/sitemap.ts` — 3 public pages with priorities and change frequencies.
- Task 6: Build + lint pass with zero errors. OG image confirmed as dynamic route in build output.

### File List

- `app/layout.tsx` (modified — extended Metadata)
- `app/page.tsx` (modified — added JsonLd import)
- `app/opengraph-image.tsx` (new — dynamic OG image generation)
- `app/robots.ts` (new — robots.txt generation)
- `app/sitemap.ts` (new — sitemap.xml generation)
- `components/seo/json-ld.tsx` (new — JSON-LD structured data)
- `_bmad-output/implementation-artifacts/sprint-status.yaml` (modified — story status)

### Change Log

- 2026-02-11: Implemented Story 4.2 — SEO, Meta Tags & Social Sharing. All 6 tasks completed. Build + lint pass.
- 2026-02-11: Code review fixes — M1: Replaced force-dynamic with revalidate=86400 for OG image caching. M2: Added Inter Bold font loading via Google Fonts API (woff format). M3: Removed redundant openGraph.title/description (auto-inherited from root metadata).

## Senior Developer Review (AI)

**Review Outcome:** Approve (with fixes applied)
**Review Date:** 2026-02-11
**Issues Found:** 0 High, 3 Medium, 2 Low

### Action Items

- [x] M1: OG Image force-dynamic → revalidate=86400 for caching [app/opengraph-image.tsx]
- [x] M2: Load Inter Bold via Google Fonts API instead of Satori default font [app/opengraph-image.tsx]
- [x] M3: Remove redundant openGraph.title/description — Next.js auto-inherits [app/layout.tsx]
- L1: JsonLd placement outside main — informational only, no change needed
- L2: /confirmed double-covered in robots.ts + page metadata — informational only
