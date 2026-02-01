# tellingCube v2.0 Release Notes

**Release Name:** Big Bang Launch
**Version:** 2.0.0
**Release Date:** January 23, 2026

---

## Overview

tellingCube v2.0 marks a major milestone: the transformation from demonstration tool to full commercial product. This release introduces user accounts, flexible pricing tiers, background scenario generation, and a complete HR data simulation domain.

---

## Highlights

- **Passwordless Authentication** — Sign in with magic links sent to your email
- **Personal Dashboard** — Manage all your scenarios in one place
- **Background Generation** — Queue scenarios and get notified when ready
- **Flexible Pricing** — One-time purchases or monthly subscriptions
- **HR Data Domain** — Complete workforce simulation with IBCS-compliant charts
- **Charity Integration** — 3% of every purchase supports children's hospice care

---

## New Features

### Authentication & User Dashboard

| Feature | Description |
|---------|-------------|
| Magic Link Login | Passwordless authentication via email—no passwords to remember |
| Personal Dashboard | Centralized view of all generated scenarios with quick stats |
| Scenario Management | View details and download data directly from your dashboard |
| Smart Navigation | Automatic redirect to dashboard when logged in; access controls on every page |

### Background Generation

| Feature | Description |
|---------|-------------|
| Async Processing | Generate scenarios in the background while you continue working |
| Progress Tracking | Real-time progress bar with estimated completion time |
| Email Notifications | Receive an email when your scenario is ready to view |
| Cancellation | Cancel in-progress generations at any time |

### Pricing & Plans

**Individual Plans (One-Time Purchase)**

| Plan | Price | Scenarios | Data Duration | HR Domain |
|------|-------|-----------|---------------|-----------|
| Single | €9 | 1 | 1 year | No |
| Lifetime | €99 | Unlimited | 1–3 years | No |
| Lifetime+ | €299 | Unlimited | 1–5 years | Yes |

**Team Plans (Monthly Subscription)**

| Plan | Price | Seats | Data Duration | HR Domain |
|------|-------|-------|---------------|-----------|
| Pro | €19/mo | 1 | 1–5 years | Yes |
| Team | €49/mo | 5 | 1–5 years | Yes |

**Promotional Codes:** BRIAN10, PHTC2026, HOKIHEART (10% discount; HOKIHEART also increases charity contribution to 5%)

### HR Data Domain

- **Employee Lifecycle Simulation** — Realistic events including hires, terminations, promotions, transfers, and salary adjustments
- **Dedicated HR View** — New tab with workforce analytics and metrics
- **IBCS-Compliant Charts:**
  - Headcount by Department
  - Turnover Trend Analysis
  - Compensation Distribution
  - Workforce Demographics (Age & Tenure)
- **Data Export** — Download HR data in JSON or CSV format
- **Privacy Indicator** — Clear "Synthetic Data" badge on all views

### Launch Polish

- **9 Curated Examples** — Complete coverage of all company sizes (Startup, MidCap, LargeCap) and sectors (Consumer Staples, Industrials, Financials)
- **Live Donation Counter** — Real-time display of charitable contributions
- **Charity Integration** — 3% of every purchase supports [hoki.help](https://hoki.help) children's hospice care in Austria
- **Founding Member Recognition** — Early supporters automatically upgraded to Lifetime+
- **Streamlined Onboarding** — Generation dialog opens automatically after purchase

---

## API Reference

### New Endpoints

**Authentication**

| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/api/auth/request-link` | Request a magic link email |
| GET | `/api/auth/verify` | Verify magic link token and create session |
| GET | `/api/auth/me` | Retrieve current user session |
| POST | `/api/auth/logout` | End current session |

**Dashboard**

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/api/dashboard` | Retrieve dashboard summary and stats |
| GET | `/api/dashboard/scenarios` | List user scenarios (paginated) |

**Generation**

| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/api/generate` | Queue a new scenario generation |
| GET | `/api/scenarios/:id/status` | Check generation progress |
| POST | `/api/scenarios/:id/cancel` | Cancel an in-progress generation |

**HR Data**

| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/api/views/hr` | Retrieve HR analytics data |
| GET | `/api/export/:id/hr.json` | Export HR data as JSON |
| GET | `/api/export/:id/hr.csv` | Export HR data as CSV |

**Miscellaneous**

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/api/charity/total` | Retrieve total charitable donations |

---

## Database Schema Changes

| Change | Description |
|--------|-------------|
| New `users` table | Stores user accounts with entitlements (tier, billing model, limits) |
| New `magic_links` table | Tracks passwordless authentication tokens |
| New `user_scenarios` table | Links users to their generated scenarios |
| Extended `scenarios` table | Added status tracking fields (queued, generating, completed, failed) |

---

## Configuration

### Required Environment Variables

| Variable | Description |
|----------|-------------|
| `JWT_SECRET` | Secret key for signing session tokens (minimum 32 characters) |
| `STRIPE_PRICE_SINGLE` | Stripe Price ID for Single tier (€9) |
| `STRIPE_PRICE_LIFETIME` | Stripe Price ID for Lifetime tier (€99) |
| `STRIPE_PRICE_LIFETIME_PLUS` | Stripe Price ID for Lifetime+ tier (€299) |
| `STRIPE_PRICE_PRO_MONTHLY` | Stripe Price ID for Pro subscription (€19/mo) |
| `STRIPE_PRICE_TEAM_MONTHLY` | Stripe Price ID for Team subscription (€49/mo) |
| `STRIPE_COUPON_BRIAN10` | Stripe Coupon ID for BRIAN10 code |
| `STRIPE_COUPON_PHTC2026` | Stripe Coupon ID for PHTC2026 code |
| `STRIPE_COUPON_HOKIHEART` | Stripe Coupon ID for HOKIHEART code |

---

## Bug Fixes

| Issue | Resolution |
|-------|------------|
| Login page autofill contrast | Fixed dark-on-dark text when browser autofills email field |
| Pricing page Euro symbol | Corrected Unicode escape sequence displaying as literal text |
| API docs navigation | Fixed broken header links and dark mode styling issues |

---

## Migration Guide

### Existing Users

Early supporters who purchased before the Big Bang Launch have been automatically migrated to **Lifetime+** tier with full access to all features.

To manually run the migration script:

```bash
pnpm migrate:early-supporters
```

### Deployment Checklist

1. Set `JWT_SECRET` environment variable in production
2. Configure Stripe webhook endpoint for payment events
3. Set up QStash for background generation processing
4. Verify all `STRIPE_PRICE_*` and `STRIPE_COUPON_*` variables are configured

---

## Release Statistics

| Metric | Value |
|--------|-------|
| Epics Delivered | 4 (Epics 4–7) |
| Stories Completed | 32 |
| Total Project Stories | 60+ across 7 epics |
| Curated Examples | 9 (realistic employee counts: 12–75) |

---

## Additional Resources

- [API Documentation](/api-docs) — Complete endpoint reference
- [Pricing Page](/pricing) — View all plans and purchase options
- [Examples](/examples) — Explore curated scenario examples

---

*tellingCube — Realistic Business Data That Reconciles*
