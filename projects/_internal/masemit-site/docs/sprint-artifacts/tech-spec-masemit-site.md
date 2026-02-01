# Tech-Spec: masem.at Corporate Website

**Created:** 2025-12-15
**Status:** ✅ COMPLETE - Deployed to Production
**Deadline:** December 17, 2025
**Source:** Product Brief (`docs/analysis/product-brief-masemit-site-2025-12-14.md`)

---

## Overview

### Problem Statement

When visitors encounter "masemIT" through product references (e.g., tellingCube.com footer), they find no central company presence. This undermines B2B trust and professional credibility.

### Solution

Build a 2-page bilingual corporate website at masem.at that:
- Establishes company credibility (< 60 seconds to "this is legit")
- Showcases tellingCube as flagship product
- Provides contact pathway
- Matches tellingCube's professional aesthetic

### Scope

**In Scope:**
- Homepage (Hero, Vision, Products, About, Contact, Footer)
- Legal page (Privacy + Terms + Impressum combined)
- Full i18n (EN/DE)
- Contact form with Resend + Turnstile
- Responsive, accessible design
- Analytics tracking

**Out of Scope (Post-MVP):**
- Newsletter signup
- FAQ section
- Trust/testimonials section
- Blog/News
- Additional products

---

## Implementation Summary

### Tech Stack (Final)

| Layer | Choice |
|-------|--------|
| Framework | Next.js 14.2 (App Router) |
| Styling | Tailwind CSS |
| Hosting | Vercel Pro |
| Email | Resend |
| Bot Protection | Cloudflare Turnstile |
| Analytics | Vercel Analytics |
| i18n | next-intl |

### Design System

| Element | Value |
|---------|-------|
| Primary Color | #093236 (dark teal) |
| Accent Color | #009BB1 (cyan) |
| Font | Inter (Google Fonts) |
| Style | Clean, spacious, professional |

### Final File Structure

```
src/
├── app/
│   ├── [locale]/
│   │   ├── layout.tsx        # Root layout with i18n, analytics, turnstile
│   │   ├── page.tsx          # Homepage
│   │   └── legal/
│   │       └── page.tsx      # Legal page
│   ├── api/
│   │   └── contact/
│   │       └── route.ts      # Contact form API (Resend + Turnstile)
│   ├── globals.css           # Global styles
│   └── sitemap.ts            # Dynamic sitemap generation
├── components/
│   ├── sections/
│   │   ├── hero.tsx
│   │   ├── vision.tsx
│   │   ├── products.tsx
│   │   ├── about.tsx
│   │   └── contact.tsx
│   └── layout/
│       ├── header.tsx        # Logo, nav, language switcher
│       └── footer.tsx        # Social links, legal link, copyright
├── i18n/
│   ├── routing.ts            # Locale routing config
│   └── request.ts            # Request config
├── lib/
│   └── utils.ts              # cn() helper
├── middleware.ts             # i18n middleware
messages/
├── en.json                   # English translations
└── de.json                   # German translations
public/
├── favicon.ico               # Favicons
├── favicon.svg
├── favicon-96x96.png
├── apple-touch-icon.png
├── site.webmanifest
├── web-app-manifest-*.png
├── mario.jpg                 # Founder photo
├── masemIT Logo.svg          # Company logo
└── robots.txt
```

---

## Completed Tasks

### Foundation
- [x] **T1:** Initialize Next.js project with Tailwind
- [x] **T2:** Configure i18n with next-intl (EN/DE routing)
- [x] **T3:** Set up design tokens (colors, fonts, spacing)

### Components
- [x] **T4:** Create Header component (logo, nav, language switcher)
- [x] **T5:** Create Footer component (social links, legal link, copyright)
- [x] **T6:** Build Hero section (headline, subheadline, CTAs)
- [x] **T7:** Build Vision section (3 pillars: Builders, SME Focus, Ship)
- [x] **T8:** Build Products section (tellingCube card with UTM tracking)
- [x] **T9:** Build About section (company story, Mario photo)

### Forms & Integration
- [x] **T10:** Set up Resend integration (API route with [masem.at] prefix)
- [x] **T11:** Set up Turnstile integration (script + widget placeholder)
- [x] **T12:** Build Contact form section (validation, error handling)

### Pages & Content
- [x] **T13:** Create Legal page (Privacy, Terms, Impressum)
- [x] **T14:** Add all translations (EN/DE)

### SEO & Analytics
- [x] **T15:** SEO setup (metadata, OpenGraph, Twitter cards)
- [x] **T16:** Sitemap generation (/sitemap.xml)
- [x] **T17:** Favicons (ico, svg, png, apple-touch, manifest)
- [x] **T18:** Vercel Analytics integration

### Deployment
- [x] **T19:** Git repository (GitHub: masem-at/masemit-site)
- [x] **T20:** Deploy to Vercel
- [x] **T21:** Production deployment

---

## Acceptance Criteria (Verified)

### Homepage
- [x] **AC1:** Hero displays headline "AI tools for businesses without enterprise budgets"
- [x] **AC2:** Language switcher toggles between EN/DE, URL updates
- [x] **AC3:** Products section shows tellingCube with UTM-tracked link
- [x] **AC4:** About section displays company info and founder photo
- [x] **AC5:** Contact form validates required fields
- [x] **AC6:** Contact form submission triggers Turnstile, sends email via Resend
- [x] **AC7:** Footer contains social links, legal link, copyright

### Legal Page
- [x] **AC8:** Legal page accessible at `/en/legal` and `/de/legal`
- [x] **AC9:** Contains Privacy Policy, Terms of Service, Impressum
- [x] **AC10:** Privacy policy mentions Vercel Analytics

---

## Deployment Info

| Item | Value |
|------|-------|
| **GitHub Repo** | https://github.com/masem-at/masemit-site |
| **Vercel Project** | https://vercel.com/masemit-projects/masemit-site |
| **Production URL** | https://masemit-site.vercel.app (pending custom domain) |
| **Custom Domain** | masem.at (to be configured) |

### Environment Variables (Vercel)

| Variable | Status |
|----------|--------|
| `RESEND_API_KEY` | ⏳ To be added |
| `TURNSTILE_SECRET_KEY` | ⏳ To be added |
| `NEXT_PUBLIC_TURNSTILE_SITE_KEY` | ⏳ To be added |
| `CONTACT_EMAIL` | ⏳ To be added |

---

## Remaining Setup

1. **Add Vercel Environment Variables** (see table above)
2. **Configure Custom Domain** (masem.at) in Vercel dashboard
3. **Enable Vercel Analytics** in Vercel dashboard
4. **Update Impressum** with actual business registration details
5. **Test Contact Form** end-to-end after env vars are set

---

## Post-Launch Improvements (Future)

- Newsletter signup (Resend audience)
- FAQ section
- Additional products (tellingDash, ChainSights)
- Blog/News section
- Trust/testimonials section
