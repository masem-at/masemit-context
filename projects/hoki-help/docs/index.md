# hoki.help - Project Documentation

> **Generated:** 2026-01-28 | **Scan Level:** Exhaustive | **Project Type:** Web (Next.js)

## 1. Project Overview

**hoki.help** is a donation website for HoKi NÖ (Children's Hospice Lower Austria), operated voluntarily by masemIT e.U. The platform enables both online donations via Stripe and traditional bank transfers via downloadable payment slips (Erlagschein).

### Key Characteristics
- **Purpose:** Collect donations for HoKi NÖ (Hospizteams für Kinder, Jugendliche und junge Erwachsene in Niederösterreich)
- **Domain:** https://hoki.help
- **Operator:** masemIT e.U. (volunteer-operated, no profit)
- **100% pass-through:** All donations go directly to Landesverband Hospiz NÖ
- **Languages:** German (primary), English

### Tech Stack
| Layer | Technology |
|-------|------------|
| Framework | Next.js 14 (App Router) |
| Language | TypeScript |
| Runtime | React 18 |
| Styling | Tailwind CSS |
| Payment | Stripe Checkout |
| Database | Neon (Serverless PostgreSQL) |
| Hosting | Vercel |
| PDF Generation | pdf-lib |
| QR Codes | qrcode, qrcode.react |

---

## 2. Architecture

### 2.1 Directory Structure

```
src/
├── app/                    # Next.js App Router
│   ├── api/                # API Routes
│   │   ├── barometer/      # Donation statistics API
│   │   ├── checkout/       # Stripe checkout session creation
│   │   ├── erlagschein/    # PDF generation for payment slips
│   │   ├── og/             # Dynamic OpenGraph image generation
│   │   └── webhook/        # Stripe webhook handler
│   ├── danke/              # Thank you page after donation
│   ├── datenschutz/        # Privacy policy (GDPR compliant)
│   ├── erlagschein/        # Payment slip download page
│   ├── impressum/          # Legal notice (Austrian ECG)
│   ├── globals.css         # Global styles
│   ├── layout.tsx          # Root layout with providers
│   └── page.tsx            # Main landing page
├── components/             # React components
│   ├── DonationBarometer.tsx    # Progress visualization with badges
│   ├── DonationWidget.tsx       # Amount selection & checkout trigger
│   ├── FAQ.tsx                  # Accordion FAQ with Schema.org
│   ├── Footer.tsx               # Footer with partner logos
│   ├── Header.tsx               # Navigation header
│   ├── LanguageProvider.tsx     # i18n context provider
│   ├── LanguageSwitcher.tsx     # DE/EN toggle
│   ├── LegalPageWrapper.tsx     # Wrapper for legal pages
│   ├── SocialShare.tsx          # Share buttons
│   └── ThankYouContent.tsx      # Post-donation content
├── hooks/
│   └── useTranslation.ts   # Translation hook wrapper
├── i18n/                   # Internationalization
│   ├── config.ts           # Language detection & persistence
│   ├── de.json             # German translations
│   ├── en.json             # English translations
│   └── index.ts            # Translation exports
├── lib/                    # Utilities
│   ├── badges.ts           # Badge tier logic
│   ├── badge-schema.sql    # Badge tier database schema
│   ├── db.ts               # Neon database connection
│   ├── kv.ts               # Donation statistics logic
│   └── stripe.ts           # Stripe client initialization
└── types/                  # TypeScript types
    └── global.d.ts         # Global type declarations
```

### 2.2 Data Flow

```
┌─────────────────────────────────────────────────────────────────────┐
│                         DONATION FLOWS                               │
└─────────────────────────────────────────────────────────────────────┘

ONLINE DONATION (Stripe):
┌──────────┐    ┌─────────────┐    ┌─────────┐    ┌──────────┐
│ Frontend │───>│ /api/       │───>│ Stripe  │───>│ /danke   │
│ Widget   │    │ checkout    │    │ Checkout│    │ Page     │
└──────────┘    └─────────────┘    └─────────┘    └──────────┘
                                        │
                                        v
                               ┌─────────────────┐    ┌─────────┐
                               │ /api/webhook    │───>│ Neon DB │
                               │ (Stripe events) │    │ Stats   │
                               └─────────────────┘    └─────────┘

ERLAGSCHEIN (Bank Transfer):
┌──────────────┐    ┌─────────────────┐    ┌─────────┐
│ /erlagschein │───>│ /api/erlagschein│───>│ PDF     │
│ Page         │    │ (pdf-lib)       │    │ Download│
└──────────────┘    └─────────────────┘    └─────────┘
                           │
                           v
                    ┌─────────────┐
                    │ Neon DB     │
                    │ (tracking)  │
                    └─────────────┘
```

---

## 3. Components

### 3.1 DonationWidget (`src/components/DonationWidget.tsx`)
Core donation form with:
- Preset amounts: €10, €25, €50, €100
- Custom amount input
- One-time vs. monthly toggle
- Anonymous donation option
- Stripe Checkout integration
- Event tracking via `window.masemit.track()`

### 3.2 DonationBarometer (`src/components/DonationBarometer.tsx`)
Progress visualization showing:
- Current donation total
- Badge progression (6 stages: Egg → Caterpillar → Cocoon → Small Butterfly → Butterfly → Radiant Butterfly)
- Responsive layout (vertical on mobile, horizontal on desktop)
- Animated progress bars

### 3.3 LanguageProvider (`src/components/LanguageProvider.tsx`)
Context provider for i18n:
- Auto-detects language from localStorage or browser
- Persists preference to localStorage
- Prevents hydration mismatch

### 3.4 SocialShare (`src/components/SocialShare.tsx`)
Share buttons for:
- WhatsApp, Facebook, Twitter/X, LinkedIn
- Copy link functionality
- Platform-specific share text

---

## 4. API Routes

### 4.1 POST /api/checkout
Creates Stripe Checkout session.

**Request Body:**
```typescript
{
  amount: number;      // EUR amount
  isMonthly: boolean;  // Recurring subscription
  isAnonymous: boolean;
  lang: 'de' | 'en';
}
```

**Response:**
```typescript
{ url: string }  // Stripe Checkout URL
```

**Features:**
- Supports one-time payments and subscriptions
- Collects billing address for non-anonymous donations
- Custom field for birthdate (tax deductibility)
- Product metadata: `product: 'hoki'`

### 4.2 POST /api/webhook
Handles Stripe webhook events.

**Supported Events:**
- `checkout.session.completed` - Records one-time donations
- `invoice.paid` - Records recurring subscription payments

**Security:**
- Validates `stripe-signature` header
- Only processes events with `metadata.product === 'hoki'`

### 4.3 GET /api/barometer
Returns donation statistics and badge progress.

**Response:**
```typescript
{
  total: number;           // Total cents
  count: number;           // Donation count
  visible: boolean;        // Show barometer (>€500)
  showAmount: boolean;     // Show actual amount
  lastUpdated: string | null;
  badgeProgress: {
    currentTier: BadgeTier | null;
    nextTier: BadgeTier | null;
    allTiers: BadgeTier[];
    progressPercent: number;
    overallProgressPercent: number;
  } | null;
}
```

### 4.4 GET/POST /api/erlagschein
Generates PDF payment slips.

**GET:** Returns empty payment slip (blank form)

**POST Request Body:**
```typescript
{
  vorname?: string;
  nachname?: string;
  strasse?: string;
  plz?: string;
  ort?: string;
  geburtsdatum?: string;
  betrag?: string;
}
```

**Features:**
- EPC/Girocode QR code for banking apps
- hoki.help branding with logo and butterfly
- Tax deductibility note
- Tracks download statistics

### 4.5 GET /api/og
Generates dynamic OpenGraph images for social sharing.

**Query Parameters:**
- `amount` - Donation amount (optional)
- `anonymous` - Hide amount if true
- `lang` - de/en

---

## 5. Database Schema

### 5.1 Tables

**donation_stats** - Aggregated Stripe donation statistics
```sql
CREATE TABLE donation_stats (
  id TEXT PRIMARY KEY DEFAULT 'current',
  total_cents INTEGER DEFAULT 0,
  count INTEGER DEFAULT 0,
  updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);
```

**badge_tiers** - Badge progression tiers
```sql
CREATE TABLE badge_tiers (
  id SERIAL PRIMARY KEY,
  sort_order INTEGER NOT NULL UNIQUE,
  min_cents INTEGER NOT NULL,
  max_cents INTEGER NOT NULL,
  badge_path TEXT NOT NULL,
  label_de TEXT NOT NULL,
  label_en TEXT NOT NULL,
  description_de TEXT,
  description_en TEXT,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);
```

**Badge Tiers (default):**
| Order | Range | Badge | Label (DE) |
|-------|-------|-------|------------|
| 1 | €0-500 | stage-1-egg.svg | Ei |
| 2 | €500-1000 | stage-2-caterpillar.svg | Raupe |
| 3 | €1000-3000 | stage-3-cocoon.svg | Kokon |
| 4 | €3000-5000 | stage-4-small-butterfly.svg | Kleiner Falter |
| 5 | €5000-100000 | stage-5-butterfly.svg | Schmetterling |
| 6 | €100000+ | stage-6-radiant-butterfly.svg | Strahlender Falter |

**erlagschein_donations** - Tracks payment slip amounts
```sql
CREATE TABLE erlagschein_donations (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  betrag_cents INTEGER NOT NULL,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  status TEXT DEFAULT 'pending',  -- pending/confirmed/expired
  expires_at TIMESTAMP WITH TIME ZONE
);
```

**erlagschein_stats** - Download statistics
```sql
CREATE TABLE erlagschein_stats (
  id TEXT PRIMARY KEY DEFAULT 'current',
  empty_downloads INTEGER DEFAULT 0,
  filled_downloads INTEGER DEFAULT 0,
  updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);
```

---

## 6. Configuration

### 6.1 Environment Variables

```env
# Stripe
STRIPE_SECRET_KEY=sk_...
STRIPE_PUBLISHABLE_KEY=pk_...
STRIPE_WEBHOOK_SECRET=whsec_...

# Neon Database
DATABASE_URL=postgresql://...

# App
NEXT_PUBLIC_BASE_URL=https://hoki.help

# Barometer
BAROMETER_SHOW_AMOUNT_THRESHOLD=0  # Show amount after this threshold (cents)
```

### 6.2 Tailwind Theme

Custom colors defined in `tailwind.config.js`:
```javascript
colors: {
  teal: { DEFAULT: '#3D9A8B', light: '#7BC5AE' },
  gold: { DEFAULT: '#F5A623', light: '#FFE066' },
  cream: '#FFFAF5',
  coral: '#E85D75',
  'soft-blue': '#6BA3D6',
  'soft-green': '#7BC5AE',
}
fontFamily: {
  sans: ['Inter', 'sans-serif'],
  heading: ['Montserrat', 'sans-serif'],
}
```

---

## 7. Internationalization

### 7.1 Structure
```
src/i18n/
├── config.ts    # Language detection (localStorage > browser > default)
├── de.json      # German translations
├── en.json      # English translations
└── index.ts     # Export combined translations
```

### 7.2 Usage
```typescript
// In components
const { t, locale, setLocale } = useTranslation();
return <h1>{t.hero.headline}</h1>;
```

### 7.3 Translation Keys
- `meta.*` - SEO metadata
- `header.*` - Navigation
- `hero.*` - Hero section
- `trust.*` - Trust badges
- `about.*` - About section
- `faq.*` - FAQ items
- `donation.*` - Donation widget
- `barometer.*` - Progress display
- `thankyou.*` - Thank you page
- `share.*` - Social sharing
- `footer.*` - Footer
- `legal.*` - Legal pages
- `og.*` - OpenGraph images
- `erlagschein.*` - Payment slip page

---

## 8. Legal Compliance

### 8.1 GDPR Features
- No tracking cookies (uses localStorage for language only)
- Anonymous donation option
- Privacy-first analytics (self-hosted tracker.js)
- Erlagschein data not stored on server (PDF-only)
- 7-year retention for tax documentation

### 8.2 Austrian ECG Compliance
- Full Impressum with company details
- UID-Nummer: ATU82330407
- Firmenbuchnummer: FN 661236g

### 8.3 Tax Deductibility
- Donations to Landesverband Hospiz NÖ are tax-deductible in Austria
- Birthdate collected for automatic tax reporting
- IBAN: AT97 3225 0082 0070 7760

---

## 9. Event Tracking

Custom tracking via `window.masemit.track()`:
- `amount_select` - Amount button clicked
- `donate_start` - Checkout initiated
- `donate_complete` - Donation successful
- `share_click` - Social share button clicked

---

## 10. Development

### 10.1 Setup
```bash
npm install
cp .env.example .env.local
# Configure environment variables
npm run dev
```

### 10.2 Database Setup
Run the SQL from `src/lib/db.ts` and `src/lib/badge-schema.sql` in Neon Console.

### 10.3 Stripe Testing
- Use test keys (sk_test_..., pk_test_...)
- Test webhook: `stripe listen --forward-to localhost:3000/api/webhook`

---

## 11. Key Files Reference

| File | Purpose | Lines |
|------|---------|-------|
| `src/app/page.tsx` | Main landing page | ~150 |
| `src/components/DonationWidget.tsx` | Donation form | ~225 |
| `src/components/DonationBarometer.tsx` | Badge progress | ~270 |
| `src/app/api/checkout/route.ts` | Stripe session | ~100 |
| `src/app/api/webhook/route.ts` | Stripe webhook | ~95 |
| `src/app/api/erlagschein/route.ts` | PDF generation | ~630 |
| `src/lib/kv.ts` | Stats logic | ~65 |
| `src/lib/badges.ts` | Badge logic | ~115 |
| `src/i18n/de.json` | German translations | ~180 |

---

## 12. External Dependencies

### Production
- `@neondatabase/serverless` - Neon PostgreSQL client
- `@vercel/og` - OpenGraph image generation
- `pdf-lib` - PDF generation
- `qrcode` / `qrcode.react` - QR code generation
- `stripe` - Payment processing

### Development
- `typescript` - Type checking
- `tailwindcss` - Styling
- `eslint` - Linting

---

*Documentation generated by BMAD Document Project workflow*
