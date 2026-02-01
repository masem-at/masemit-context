# Dashboard & Pricing UX Spec

**Created:** 2026-01-19
**Author:** Sally (UX Designer)
**Status:** Approved in Party Mode
**Related:** `docs/architecture/auth-dashboard-architecture.md`, `docs/future/big-bang-launch-plan.md`

---

## Overview

UX design specification for the user dashboard, pricing page two-path flow, generation dialog, and email templates to support the Big Bang Launch features.

**Design Principles:**
- Desktop-first (B2B users)
- Clean, focused, guides users to value quickly
- Minimal friction in purchase → generation flow
- Clear tier differentiation
- Consistent with existing tellingCube design system

---

## 1. Dashboard (`/dashboard`)

### Desktop Layout

```
┌─────────────────────────────────────────────────────────────────────────┐
│  tellingCube                              Mario ▾   [Generate New]      │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │  Welcome back, Mario!                                            │   │
│  │                                                                   │   │
│  │  Lifetime Member                       3 scenarios generated      │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│                                                                         │
│  ┌─ ACTIVE GENERATION ──────────────────────────────────────────────┐  │
│  │  Generating: MidCap x Industrials                                 │  │
│  │  ████████████░░░░░░░░ 62%  •  ~1:30 remaining                    │  │
│  │                                                      [Cancel]     │  │
│  └──────────────────────────────────────────────────────────────────┘  │
│                                                                         │
│  YOUR SCENARIOS                                            [View All]   │
│  ┌────────────┐  ┌────────────┐  ┌────────────┐                        │
│  │ (factory)  │  │ (bank)     │  │ (cart)     │                        │
│  │ TechCorp   │  │ FinanceHub │  │ RetailMax  │                        │
│  │ MidCap     │  │ LargeCap   │  │ Startup    │                        │
│  │ Jan 2026   │  │ Dec 2025   │  │ Dec 2025   │                        │
│  │            │  │            │  │            │                        │
│  │ [View] [DL]│  │ [View] [DL]│  │ [View] [DL]│                        │
│  └────────────┘  └────────────┘  └────────────┘                        │
│                                                                         │
│  ┌─ QUICK STATS ────────────────────────────────────────────────────┐  │
│  │  Total Generated: 3       Member Since: Jan 2026                 │  │
│  │  Data Downloaded: 2       Your Tier: Lifetime (€99)              │  │
│  └──────────────────────────────────────────────────────────────────┘  │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

### Key Components

#### Welcome Banner
- Personalized greeting with user's name
- Member badge showing tier (e.g., "Lifetime Member")
- Quick stat: total scenarios generated

#### Active Generation Banner
- **Only visible when a generation is in progress**
- Shows: scenario type, progress bar, estimated time remaining
- Cancel button available
- Prominent placement - can't miss it

#### Scenario Cards
- Grid layout: 3 columns on desktop, 2 on tablet, 1 on mobile
- Each card shows:
  - Sector icon (factory/bank/cart)
  - Company name (generated)
  - Size badge (Startup/MidCap/LargeCap)
  - Generation date
  - Action buttons: View, Download
- Sorted by creation date (newest first)

#### Quick Stats Footer
- Total scenarios generated
- Total downloads
- Member since date
- Current tier with price paid

### Mobile Layout

```
┌─────────────────────────┐
│ tellingCube    [=] Menu │
├─────────────────────────┤
│                         │
│ Welcome back, Mario!    │
│ Lifetime Member         │
│                         │
│ ┌─ GENERATING ────────┐ │
│ │ MidCap x Industrials│ │
│ │ ████████░░░ 62%     │ │
│ │ ~1:30 remaining     │ │
│ │          [Cancel]   │ │
│ └─────────────────────┘ │
│                         │
│ YOUR SCENARIOS          │
│ ┌─────────────────────┐ │
│ │ TechCorp - MidCap   │ │
│ │ Jan 2026            │ │
│ │ [View]    [Download]│ │
│ └─────────────────────┘ │
│ ┌─────────────────────┐ │
│ │ FinanceHub - Large  │ │
│ │ Dec 2025            │ │
│ │ [View]    [Download]│ │
│ └─────────────────────┘ │
│                         │
│ [+ Generate New]        │
│                         │
└─────────────────────────┘
```

---

## 2. Pricing Page - Two-Path Selection

### Step 1: Choose Your Path

```
┌─────────────────────────────────────────────────────────────────────────┐
│                                                                         │
│              How will you use tellingCube?                              │
│                                                                         │
│   ┌──────────────────────────┐    ┌──────────────────────────────┐     │
│   │                          │    │                              │     │
│   │      INDIVIDUAL          │    │      TEAM / AGENCY           │     │
│   │                          │    │                              │     │
│   │   One-time purchase      │    │   Monthly subscription       │     │
│   │   Pay once, use forever  │    │   For ongoing data needs     │     │
│   │                          │    │                              │     │
│   │   * No subscriptions     │    │   * Multiple seats           │     │
│   │   * Founding member      │    │   * Cancel anytime           │     │
│   │   * Lifetime access      │    │   * Team management          │     │
│   │                          │    │                              │     │
│   │      [Choose ->]         │    │      [Choose ->]             │     │
│   │                          │    │                              │     │
│   └──────────────────────────┘    └──────────────────────────────┘     │
│                                                                         │
│         3% of every purchase supports hoki.help                         │
│            Children's hospice care in Austria                           │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

### Step 2a: Individual Pricing (after clicking Individual)

```
┌─────────────────────────────────────────────────────────────────────────┐
│  <- Back                     FOUNDING MEMBER ACCESS                     │
│                                                                         │
│   ┌─────────────┐   ┌─────────────┐   ┌─────────────┐                  │
│   │   SINGLE    │   │  LIFETIME   │   │ LIFETIME+   │                  │
│   │             │   │   * BEST    │   │             │                  │
│   │    €9       │   │    €99      │   │   €299      │                  │
│   │  one-time   │   │  one-time   │   │  one-time   │                  │
│   │             │   │             │   │             │                  │
│   │ 1 scenario  │   │ Unlimited   │   │ Unlimited   │                  │
│   │ 1 year data │   │ 1-3 years   │   │ 1-5 years   │                  │
│   │             │   │             │   │ + HR Domain │                  │
│   │             │   │             │   │             │                  │
│   │  [Buy Now]  │   │  [Buy Now]  │   │  [Buy Now]  │                  │
│   └─────────────┘   └─────────────┘   └─────────────┘                  │
│                                                                         │
│   Have a coupon? [Enter code]                                           │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

### Step 2b: Team/Agency Pricing (after clicking Team/Agency)

```
┌─────────────────────────────────────────────────────────────────────────┐
│  <- Back                      PRO SUBSCRIPTIONS                         │
│                                                                         │
│          ┌─────────────────┐     ┌─────────────────┐                   │
│          │      PRO        │     │      TEAM       │                   │
│          │                 │     │                 │                   │
│          │    €19/mo       │     │    €49/mo       │                   │
│          │                 │     │                 │                   │
│          │  1 seat         │     │  5 seats        │                   │
│          │  Unlimited      │     │  Unlimited      │                   │
│          │  1-5 years      │     │  1-5 years      │                   │
│          │  + HR Domain    │     │  + HR Domain    │                   │
│          │                 │     │  Team billing   │                   │
│          │                 │     │                 │                   │
│          │  [Subscribe]    │     │  [Subscribe]    │                   │
│          └─────────────────┘     └─────────────────┘                   │
│                                                                         │
│   Need more seats? Contact us for enterprise pricing                    │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

### Design Notes

- **Two-path selection reduces cognitive load** - user self-selects, sees only relevant options
- **"BEST" badge on Lifetime** - visual recommendation for most common choice
- **Back button** - easy to change mind
- **Coupon field** - supports HOKIHEART, BRIAN10, PHTC2026
- **Charity mention** - footer on all pricing views

---

## 3. Generation Dialog (Paid Users)

### Trigger
- "Generate New" button in dashboard header
- After successful purchase (land on dashboard, dialog auto-opens)

### Dialog Layout

```
┌─────────────────────────────────────────────────────────────────────────┐
│                                                                         │
│                    Generate Your Scenario                               │
│                                                                         │
│   ┌─────────────────────────────────────────────────────────────────┐  │
│   │                                                                   │  │
│   │   COMPANY SIZE                                                    │  │
│   │                                                                   │  │
│   │   ( ) Startup        10-50 employees, €0-5M revenue              │  │
│   │   (*) MidCap         200-500 employees, €50-500M revenue         │  │
│   │   ( ) LargeCap       2,000+ employees, €1B+ revenue              │  │
│   │                                                                   │  │
│   │   INDUSTRY SECTOR                                                 │  │
│   │                                                                   │  │
│   │   ( ) Consumer Staples    Retail, food & beverage, household     │  │
│   │   (*) Industrials         Manufacturing, logistics, engineering  │  │
│   │   ( ) Financials          Banking, insurance, investment         │  │
│   │                                                                   │  │
│   │   DATA DURATION                        (Your tier: up to 3 years) │  │
│   │                                                                   │  │
│   │   ( ) 1 Year                                                      │  │
│   │   (*) 3 Years                                                     │  │
│   │   ( ) 5 Years  [lock] Requires Lifetime+ or Pro                  │  │
│   │                                                                   │  │
│   └─────────────────────────────────────────────────────────────────┘  │
│                                                                         │
│              ┌─────────────────────────────────────┐                   │
│              │     Generate My Scenario            │                   │
│              └─────────────────────────────────────┘                   │
│                                                                         │
│         Takes 2-3 minutes. You'll get an email when ready!             │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

### Design Notes

- **Radio buttons, not dropdowns** - fewer clicks, clearer choice
- **Helper text** - explains each option briefly
- **Tier indicator** - shows what user's tier allows
- **Locked options** - greyed out with lock icon, shows upgrade path
- **Time expectation** - sets realistic expectations upfront
- **Email notification** - reassures user they won't miss it

---

## 4. Email Templates

### 4a. Magic Link Email

**Subject:** Access your tellingCube dashboard

```
┌─────────────────────────────────────────────────────────────────────────┐
│                                                                         │
│   tellingCube                                                           │
│                                                                         │
│   ─────────────────────────────────────────────────────────────────    │
│                                                                         │
│   Hi Mario,                                                             │
│                                                                         │
│   Click below to access your tellingCube dashboard:                     │
│                                                                         │
│        ┌────────────────────────────────┐                              │
│        │   Access My Dashboard          │                              │
│        └────────────────────────────────┘                              │
│                                                                         │
│   This link expires in 1 hour.                                          │
│                                                                         │
│   If you didn't request this, you can safely ignore this email.        │
│                                                                         │
│   ─────────────────────────────────────────────────────────────────    │
│                                                                         │
│   3% of your purchase supports children's hospice care                  │
│   via hoki.help                                                         │
│                                                                         │
│   Questions? Reply to this email.                                       │
│                                                                         │
│   - The tellingCube Team                                                │
│     masemIT e.U. - Austria                                              │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

### 4b. Generation Complete Email

**Subject:** Your tellingCube scenario is ready!

```
┌─────────────────────────────────────────────────────────────────────────┐
│                                                                         │
│   tellingCube                                                           │
│                                                                         │
│   ─────────────────────────────────────────────────────────────────    │
│                                                                         │
│   Your scenario is ready!                                               │
│                                                                         │
│   ┌─────────────────────────────────────────────────────────────────┐  │
│   │  MidCap x Industrials                                            │  │
│   │  3 years of data - 12,847 events generated                       │  │
│   └─────────────────────────────────────────────────────────────────┘  │
│                                                                         │
│        ┌────────────────────────────────┐                              │
│        │   View My Scenario             │                              │
│        └────────────────────────────────┘                              │
│                                                                         │
│   Or go to your dashboard to download CSV/JSON exports.                │
│                                                                         │
│   ─────────────────────────────────────────────────────────────────    │
│                                                                         │
│   Thanks for supporting hoki.help                                       │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

### 4c. Purchase Confirmation Email

**Subject:** Welcome to tellingCube! (Lifetime Member)

```
┌─────────────────────────────────────────────────────────────────────────┐
│                                                                         │
│   tellingCube                                                           │
│                                                                         │
│   ─────────────────────────────────────────────────────────────────    │
│                                                                         │
│   Welcome to tellingCube, Mario!                                        │
│                                                                         │
│   You're now a Lifetime Member. Here's what you get:                    │
│                                                                         │
│   * Unlimited scenario generations                                      │
│   * Up to 3 years of business data                                      │
│   * Finance, Sales, and future views                                    │
│   * Lifetime access - no subscriptions ever                             │
│                                                                         │
│        ┌────────────────────────────────┐                              │
│        │   Access My Dashboard          │                              │
│        └────────────────────────────────┘                              │
│                                                                         │
│   This link expires in 1 hour. You can always request a new one        │
│   at tellingcube.com/login                                              │
│                                                                         │
│   ─────────────────────────────────────────────────────────────────    │
│                                                                         │
│   3% of your €99 purchase (€2.97) goes to hoki.help -                   │
│   children's hospice care in Austria. Thank you!                        │
│                                                                         │
│   Questions? Reply to this email.                                       │
│                                                                         │
│   - Mario @ tellingCube                                                 │
│     masemIT e.U. - Austria                                              │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## 5. Upgrade Prompts

### In Dashboard (for Single tier users)

```
┌─────────────────────────────────────────────────────────────────────────┐
│  You've used your 1 scenario. Upgrade to Lifetime for unlimited!        │
│                                                     [Upgrade - €99]     │
└─────────────────────────────────────────────────────────────────────────┘
```

### In Generation Dialog (locked features)

```
5 Years  [lock] Requires Lifetime+ or Pro   [Upgrade]
```

### In Example View (free users)

```
┌─────────────────────────────────────────────────────────────────────────┐
│  Want your own custom data?                                             │
│  Generate unlimited scenarios starting at €9                            │
│                                                     [See Pricing]       │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## 6. Color & Style Guidelines

### Tier Badges

| Tier | Badge Color | Label |
|------|-------------|-------|
| Free | Gray | "Free" |
| Single | Blue | "Single" |
| Lifetime | Emerald | "Lifetime Member" |
| Lifetime+ | Purple | "Lifetime+ Member" |
| Pro | Teal | "Pro" |
| Team | Indigo | "Team" |

### Buttons

- **Primary action:** Blue gradient (existing style)
- **Secondary action:** Gray outline
- **Upgrade CTA:** Emerald gradient
- **Cancel/Back:** Text only, gray

### Status Colors

| Status | Color | Example |
|--------|-------|---------|
| Generating | Yellow/Amber | Progress bar |
| Complete | Green | Success badge |
| Failed | Red | Error message |
| Queued | Gray | Waiting indicator |

---

## 7. Responsive Breakpoints

| Breakpoint | Width | Layout Changes |
|------------|-------|----------------|
| Desktop | 1024px+ | 3-column scenario grid, full sidebar |
| Tablet | 768-1023px | 2-column grid, collapsed stats |
| Mobile | <768px | 1-column, hamburger menu, stacked cards |

---

## 8. Accessibility Notes

- All interactive elements keyboard navigable
- Radio buttons properly labeled
- Progress bar has aria-valuenow
- Color contrast meets WCAG AA
- Email templates work without images

---

*UX spec created by Sally (UX Designer) during Party Mode session 2026-01-19*
