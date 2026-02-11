# Story 4.2: SEO, Meta Tags & Social Sharing

Status: ready-for-dev

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

- [ ] Task 1: Enhance metadata in `app/layout.tsx` (AC: #1, #2)
  - [ ] 1.1 Add `metadataBase: new URL("https://kurzum.app")` to Metadata
  - [ ] 1.2 Add `alternates: { canonical: "/" }` for canonical URL
  - [ ] 1.3 Add `openGraph` object: title, description, url, siteName, locale `"de_AT"`, type `"website"`, images (auto-resolved from `opengraph-image.tsx`)
  - [ ] 1.4 Add `twitter: { card: "summary_large_image" }`
  - [ ] 1.5 Add `keywords` array with target keywords (see Dev Notes)
  - [ ] 1.6 Verify `<html lang="de">` is set (already exists — confirm only)

- [ ] Task 2: Create dynamic OG image `app/opengraph-image.tsx` (AC: #3)
  - [ ] 2.1 Create `app/opengraph-image.tsx` using Next.js `ImageResponse` API
  - [ ] 2.2 Export `size = { width: 1200, height: 630 }`, `contentType = "image/png"`, `alt` string
  - [ ] 2.3 Render brand elements: wordmark "kurzum." in Inter Bold + waveform SVG path, centered
  - [ ] 2.4 Use brand colors: Anthracite `#1C1917` background, Orange `#F97316` for logo/text
  - [ ] 2.5 Add tagline below logo: "Sprich statt tipp." in white/light color
  - [ ] 2.6 Load Inter font via `fetch` from `public/fonts/inter-var.woff2` for `ImageResponse`

- [ ] Task 3: Add JSON-LD structured data (AC: #4)
  - [ ] 3.1 Create `components/seo/json-ld.tsx` — Server Component exporting `<JsonLd />`
  - [ ] 3.2 Embed Organization schema: name, url, logo, address (masemIT e.U., Alleegasse 26, 3851 Kautzen, AT), contactPoint (contact@masem.at)
  - [ ] 3.3 Embed SoftwareApplication schema: name "kurzum", description, offers (€10/user/month), operatingSystem, applicationCategory
  - [ ] 3.4 Add `<JsonLd />` to `app/page.tsx` (NOT layout.tsx — only on landing page)

- [ ] Task 4: Create `app/robots.ts` (AC: #5)
  - [ ] 4.1 Export default function returning `MetadataRoute.Robots`
  - [ ] 4.2 Allow: `/`, `/impressum`, `/datenschutz`
  - [ ] 4.3 Disallow: `/api/*`, `/confirmed`
  - [ ] 4.4 Add sitemap URL: `https://kurzum.app/sitemap.xml`

- [ ] Task 5: Create `app/sitemap.ts` (AC: #5)
  - [ ] 5.1 Export default function returning `MetadataRoute.Sitemap`
  - [ ] 5.2 Include pages: `/` (priority 1.0), `/impressum` (priority 0.3), `/datenschutz` (priority 0.3)
  - [ ] 5.3 Set `lastModified` to current build date
  - [ ] 5.4 Set `changeFrequency` appropriately (monthly for landing, yearly for legal)

- [ ] Task 6: Verify Build & Lint (AC: #6)
  - [ ] 6.1 Run `pnpm build` — zero errors
  - [ ] 6.2 Run `pnpm lint` — zero errors
  - [ ] 6.3 Verify OG image renders at `/opengraph-image` route (build output confirms generation)

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

### Debug Log References

### Completion Notes List

### File List

### Change Log
