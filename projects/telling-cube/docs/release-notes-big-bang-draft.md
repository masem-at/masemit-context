# tellingCube - Big Bang Launch Release Notes

**Version:** 2.0 (Big Bang)
**Release Date:** January 23, 2026
**Status:** DRAFT - Pending Tech Writer Polish

---

## Overview

The Big Bang Launch transforms tellingCube from a demo tool into a fully-fledged commercial product with user accounts, flexible pricing, background generation, and complete HR data simulation.

---

## What's New

### User Accounts & Dashboard (Epic 4)

- **Magic Link Authentication** - Passwordless login via email. No passwords to remember, just click and you're in.
- **Personal Dashboard** - View all your generated scenarios in one place with quick stats (total generated, member since, tier).
- **Scenario Management** - View and download your scenarios directly from the dashboard.
- **Smart Navigation** - Logged-in users are automatically redirected to their dashboard. Access Dashboard/Logout from any page.

### Background Generation (Epic 5)

- **Async Processing** - Generate scenarios in the background while you do other things.
- **Real-time Progress** - Live progress bar with estimated time remaining.
- **Email Notifications** - Get notified when your scenario is ready.
- **Cancel Anytime** - Changed your mind? Cancel in-progress generations.

### Flexible Pricing (Epic 6)

- **Two-Path Pricing** - Choose between Individual (one-time) or Team/Agency (subscription).
- **Individual Options:**
  - Single (€9) - 1 scenario, 1 year data
  - Lifetime (€99) - Unlimited scenarios, 1-3 years data - **BEST VALUE**
  - Lifetime+ (€299) - Unlimited scenarios, 1-5 years data + HR Domain
- **Team Options:**
  - Pro (€19/mo) - Full access for individuals with ongoing needs
  - Team (€49/mo) - 5 seats, full access, team billing
- **Coupon Support** - Use BRIAN10, PHTC2026, or HOKIHEART at checkout.
- **Tier-based Restrictions** - Generation dialog shows what's available at your tier with upgrade prompts.

### Launch Polish (Epic 7)

- **9 Curated Examples** - Complete 3x3 matrix covering all company sizes and industry sectors.
- **Live Donation Counter** - See how much has been donated to children's hospice care.
- **Charity Integration** - 3% of every purchase supports hoki.help (5% with HOKIHEART coupon).
- **Founding Member Status** - Early supporters automatically upgraded to Lifetime+.
- **Seamless Onboarding** - Generation dialog auto-opens after purchase.

### HR Data Domain (Epic 3)

- **Complete HR Simulation** - Realistic employee lifecycle events: hires, terminations, promotions, salary changes.
- **HR View** - Dedicated tab with workforce analytics.
- **4 IBCS Charts:**
  - Headcount by Department
  - Turnover Trend Analysis
  - Compensation Distribution
  - Workforce Demographics (Age & Tenure)
- **HR Export** - Download HR data as JSON or CSV.
- **Privacy Badge** - Clear "Synthetic Data" disclaimer on all views.

---

## Technical Changes

### New API Endpoints

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/api/auth/request-link` | POST | Request magic link email |
| `/api/auth/verify` | GET | Verify magic link token |
| `/api/auth/me` | GET | Get current user session |
| `/api/auth/logout` | POST | Clear session |
| `/api/dashboard` | GET | Dashboard summary data |
| `/api/dashboard/scenarios` | GET | Paginated user scenarios |
| `/api/generate` | POST | Queue scenario generation |
| `/api/scenarios/:id/status` | GET | Generation progress |
| `/api/scenarios/:id/cancel` | POST | Cancel generation |
| `/api/views/hr` | POST | HR analytics data |
| `/api/export/:id/hr.json` | GET | HR data export (JSON) |
| `/api/export/:id/hr.csv` | GET | HR data export (CSV) |
| `/api/charity/total` | GET | Total charity donations |

### Database Changes

- New `users` table with entitlements model
- New `magic_links` table for passwordless auth
- New `user_scenarios` junction table
- Extended `scenarios` table with status tracking

### Environment Variables

New required variables:
- `JWT_SECRET` - For session token signing (min 32 chars)
- `STRIPE_PRICE_*` - 5 price IDs for all tiers
- `STRIPE_COUPON_*` - 3 coupon IDs

---

## Bug Fixes (January 23, 2026)

- **Login Page** - Fixed autofill contrast issue (dark background with dark text)
- **Pricing Page** - Fixed Euro symbol displaying as `\u20AC` instead of `€`
- **API Docs** - Fixed header navigation links and dark mode styling

---

## Migration Notes

### For Existing Users

- Early supporters (pre-Big Bang purchases) automatically migrated to Lifetime+ tier
- Run `pnpm migrate:early-supporters` if manual migration needed

### For Developers

- Ensure `JWT_SECRET` is set in production environment
- Stripe webhook endpoint must be configured for payment processing
- QStash must be configured for background generation

---

## Stats

- **4 Epics** delivered (Epics 4-7)
- **32 Stories** completed
- **60+ Total Stories** across all 7 epics
- **9 Curated Examples** with realistic employee counts (12-75 employees)

---

*Draft created by Mary (Business Analyst) - Ready for Tech Writer polish*
