# tellingCube: Before & After Big Bang Launch

**Created:** 2026-01-21
**Purpose:** Visual comparison of product state before and after the Big Bang Launch

---

## Executive Summary

| Aspect | Before | After |
|--------|--------|-------|
| **Free Experience** | 3×3 matrix selector (confusing) | 9 explorable curated scenarios |
| **Authentication** | None (anonymous purchases) | Magic link login + user accounts |
| **User Dashboard** | None | Full history, downloads, stats |
| **Pricing Tiers** | 1 tier (€99 lifetime) | 5 tiers (€9 to €49/mo) |
| **Generation** | Synchronous (wait on screen) | Async with progress + email |
| **Data Duration** | 1 year only | 1, 3, or 5 years |
| **HR Analytics** | Not available | Available (Lifetime+/Pro/Team) |
| **Charity** | Not integrated | 3% to hoki.help |

---

## Detailed Comparison

### 1. Free User Experience

| Before | After |
|--------|-------|
| Landing page shows abstract 3×3 matrix | Landing page shows 9 real company scenarios |
| Users must imagine what data looks like | Users can explore actual Finance/Sales views |
| Single CTA: "Buy Now €99" | Multiple CTAs: "Explore Examples" → "Generate Your Own €9" |
| High barrier to understand value | Low barrier - see real examples first |

**Impact:** Lower cognitive load, higher conversion potential

---

### 2. Authentication & Accounts

| Before | After |
|--------|-------|
| No user accounts | Full user system with Prisma |
| Anonymous Stripe purchases | Users linked to purchases via stripe_customer_id |
| No way to re-download | Permanent access to all generations |
| No login required | Magic link login (passwordless) |
| No session management | JWT sessions (7-day, httpOnly cookies) |

**New Database Tables:**
- `users` (id, email, tier, entitlements, stripe IDs...)
- `magic_links` (token, expiry, used_at...)
- `user_scenarios` (junction table)

---

### 3. Pricing Model

#### Before: Single Tier
```
€99 Lifetime Access
├── Unlimited generations
├── 1 year data only
├── No HR domain
└── No account/dashboard
```

#### After: Hybrid 5-Tier Model

**One-Time (Founding Member):**
```
€9 Single          €99 Lifetime       €299 Lifetime+
├── 1 generation   ├── Unlimited      ├── Unlimited
├── 1 year         ├── 1-3 years      ├── 1-5 years
├── No HR          ├── No HR          ├── + HR Domain
└── Dashboard      └── Dashboard      └── Dashboard
```

**Subscriptions (Pro Plans):**
```
€19/mo Pro         €49/mo Team
├── Unlimited      ├── Unlimited
├── 1-5 years      ├── 1-5 years
├── + HR Domain    ├── + HR Domain
├── 1 seat         ├── 5 seats
└── Cancel anytime └── Team billing
```

**Impact:**
- €9 entry point removes friction (vs €99)
- Subscription option for ongoing revenue
- Clear upgrade path within product

---

### 4. Generation Process

| Before | After |
|--------|-------|
| Synchronous (2-3 min wait) | Asynchronous via QStash |
| Must stay on page | Submit and leave |
| No progress indication | Real-time progress bar (0-100%) |
| No notification | Email when complete |
| No cancel option | Can cancel in-progress jobs |
| Immediate download only | Download anytime from dashboard |

**New Statuses:** `queued` → `generating` → `completed` / `failed`

---

### 5. Data Capabilities

| Capability | Before | After |
|------------|--------|-------|
| **Time Horizons** | 1 year | 1, 3, or 5 years |
| **Finance Domain** | Yes | Yes |
| **Sales Domain** | Yes | Yes |
| **HR Domain** | No | Yes (Lifetime+/Pro/Team) |
| **Event Types** | ~15 types | ~15 + HR events |

**HR Domain Events (New):**
- Hiring, Termination, Promotion
- Salary adjustments, Transfers
- Leave, Training, Performance reviews

---

### 6. User Dashboard

| Before | After |
|--------|-------|
| No dashboard | Full dashboard at /dashboard |
| No history | All generated scenarios listed |
| No stats | Quick stats (total generated, member since, tier) |
| No re-download | View + Download any past scenario |
| No account status | Tier badge, entitlements visible |
| No active job tracking | Progress banner for in-flight generations |

**Dashboard Components:**
- Welcome banner with name + tier badge
- Scenario cards (responsive grid)
- Active generation banner (when applicable)
- Quick stats footer

---

### 7. Curated Examples

| Before | After |
|--------|-------|
| 5 examples in matrix UI | 9 tiles on landing page |
| Confusing selector UX | Clear, clickable tiles |
| Missing 4 combinations | Full 3×3 coverage |

**Complete 9-Scenario Grid:**

| | Consumer Staples | Industrials | Financials |
|---|---|---|---|
| **Startup** | Alpine Bakery GmbH | GreenTech Solutions | NEW #7 |
| **MidCap** | FreshFood Distribution | TechParts Manufacturing AG | NEW #8 |
| **LargeCap** | NEW #6 | NEW #9 | EuroCredit Union |

---

### 8. Charity Integration

| Before | After |
|--------|-------|
| No charity component | 3% of ALL revenue to hoki.help |
| No social impact messaging | Live donation counter on landing |
| — | Footer badge on every page |
| — | Charity amount in purchase email |
| — | HOKIHEART coupon (5% donation) |

**hoki.help:** Austrian hospice organization for children, youth, and young adults. Website built pro-bono by masemIT.

---

### 9. Marketing & Promotions

| Before | After |
|--------|-------|
| No coupon system | 3 active coupons |
| No influencer collab | Brian Julius promo (55K followers) |
| No launch urgency | "Founding Member" + "Limited time pricing" |
| Generic messaging | "Ivory-billed woodpecker" positioning maintained |

**Coupon Codes:**
| Code | Discount | Purpose |
|------|----------|---------|
| `HOKIHEART` | 10% + 5% charity | Charity-focused buyers |
| `BRIAN10` | 10% | Brian Julius audience |
| `PHTC2026` | 10% | Product Hunt launch day |

---

### 10. Early Supporter Treatment

| Before | After |
|--------|-------|
| 3 customers with €99 purchase | Same 3 customers |
| Basic lifetime access | Upgraded to **Lifetime+** for free |
| 1 year data only | 5 years data |
| No HR domain | Full HR domain access |

**Policy:** All pre-Big-Bang customers receive full Lifetime+ benefits forever, no action required.

---

## Technical Scope Summary

| Metric | Count |
|--------|-------|
| New Epics | 4 (Epics 4-7) |
| User Stories | 31 |
| Functional Requirements | 44 |
| New Database Tables | 3 |
| New API Endpoints | ~15 |
| New Stripe Products | 5 |
| Email Templates | 3 |

---

## Visual Summary

```
BEFORE                              AFTER
┌─────────────────────┐            ┌─────────────────────┐
│   Landing Page      │            │   Landing Page      │
│  ┌───┬───┬───┐     │            │  ┌───┬───┬───┐     │
│  │ ? │ ? │ ? │     │            │  │ 🏢│ 🏭│ 🏦│     │
│  ├───┼───┼───┤     │            │  ├───┼───┼───┤     │
│  │ ? │ ? │ ? │     │            │  │ 🏢│ 🏭│ 🏦│     │
│  ├───┼───┼───┤     │            │  ├───┼───┼───┤     │
│  │ ? │ ? │ ? │     │            │  │ 🏢│ 🏭│ 🏦│     │
│  └───┴───┴───┘     │            │  └───┴───┴───┘     │
│  "Select options"   │            │  "Explore Examples" │
│                     │            │                     │
│  [Buy Now €99]      │            │  €XX donated ❤️     │
└─────────────────────┘            │                     │
                                   │  [Try €9] [See All] │
         ↓                         └─────────────────────┘
                                            ↓
┌─────────────────────┐            ┌─────────────────────┐
│   Generation        │            │   Dashboard         │
│                     │            │  ┌─────────────────┐│
│   ⏳ Please wait... │            │  │ Welcome, Mario! ││
│   (2-3 minutes)     │            │  │ Lifetime+ ⭐    ││
│                     │            │  └─────────────────┘│
│   [Stay on page]    │            │  ┌───┐ ┌───┐ ┌───┐ │
│                     │            │  │ 📊│ │ 📊│ │ 📊│ │
└─────────────────────┘            │  └───┘ └───┘ └───┘ │
                                   │  Your Scenarios     │
         ↓                         └─────────────────────┘
                                            ↓
┌─────────────────────┐            ┌─────────────────────┐
│   Download          │            │   Async Generation  │
│                     │            │  ┌─────────────────┐│
│   [Download CSV]    │            │  │ ████████░░ 80%  ││
│                     │            │  │ ~30s remaining  ││
│   (one chance!)     │            │  └─────────────────┘│
│                     │            │  📧 Email when done │
└─────────────────────┘            └─────────────────────┘
```

---

*Document prepared by Mary (Business Analyst) for Big Bang Launch planning.*
