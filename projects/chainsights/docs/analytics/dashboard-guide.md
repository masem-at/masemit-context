# Vercel Analytics Dashboard Guide

**Last Updated:** 2025-12-23
**For:** Mario (Product Manager)
**Quick Access:** https://vercel.com → chainsights → Analytics

---

## Quick Start

### Step 1: Access Dashboard
1. Go to https://vercel.com
2. Login with your Vercel account
3. Select `chainsights` project
4. Click **"Analytics"** tab in top navigation

---

## Dashboard Overview

The Analytics dashboard has 4 main sections:

### 1. Overview (Default View)
**What You See:**
- Total page views (last 30 days)
- Unique visitors vs returning visitors
- Top pages ranked by views
- Traffic sources (direct, referral, social)
- Geographic distribution (country-level)
- Device breakdown (mobile vs desktop)

**Use For:**
- Weekly traffic trends
- Understanding where users come from
- Identifying most popular pages

---

### 2. Custom Events
**What You See:**
- All 4 custom events: `cta_click`, `social_share`, `row_expand`, `methodology_view`
- Event counts for selected date range
- Event properties (dao, score, platform, location, rank)

**Use For:**
- Tracking user engagement
- Calculating conversion funnel rates
- Understanding which DAOs generate most interest

**How to Access:**
1. Click **"Custom Events"** in left sidebar
2. See list of all tracked events
3. Click any event to see properties and details

---

### 3. Speed Insights (Core Web Vitals)
**What You See:**
- LCP (Largest Contentful Paint) - p75 score
- FCP (First Contentful Paint) - p75 score
- CLS (Cumulative Layout Shift) - p75 score
- Real user monitoring data

**Use For:**
- Monitoring site performance
- Ensuring targets are met (<2.5s LCP, <1.8s FCP, <0.1 CLS)
- Identifying performance regressions

**How to Access:**
1. Click **"Speed Insights"** in left sidebar
2. View Core Web Vitals scores
3. Check if metrics meet "Good" thresholds (green)

---

### 4. Audiences (Pro Tier Only)
**What You See:**
- Automated funnel visualization
- Cohort analysis
- Retention tracking

**Note:** This requires Vercel Pro subscription. Currently on free tier.

---

## Common Tasks

### Task 1: Check Weekly Traffic
1. Open Dashboard → Overview
2. Set date range to "Last 7 days"
3. Note total page views
4. Compare to previous week using date picker

---

### Task 2: Calculate Conversion Funnel
1. Open Dashboard → Custom Events
2. Set date range (e.g., "Last 30 days")
3. Note counts for each event:
   - `row_expand` events
   - `methodology_view` events
   - `cta_click` events
4. Open Dashboard → Top Pages
5. Note `/rankings` page views
6. Open Stripe Dashboard → Payments
7. Count successful orders in same date range
8. Calculate rates using [Conversion Funnel Guide](./conversion-funnel.md)

---

### Task 3: Find Top-Performing DAOs
1. Open Dashboard → Custom Events
2. Click `row_expand` event
3. Filter by `dao` property
4. Sort by count (descending)
5. Identify which DAOs get most clicks

---

### Task 4: Check Core Web Vitals
1. Open Dashboard → Speed Insights
2. Check p75 scores:
   - LCP should be <2.5s (green)
   - FCP should be <1.8s (green)
   - CLS should be <0.1 (green)
3. If any metric is yellow or red, investigate performance issues

---

### Task 5: Export Data for Analysis
1. Open any dashboard view
2. Click **"Export"** button in top-right
3. Choose format: CSV or JSON
4. Download file
5. Use for custom analysis in Excel/Google Sheets

---

## Date Range Selector

### How to Change Date Range
1. Click date picker in top-right corner
2. Select preset range:
   - Last 7 days
   - Last 30 days
   - Last 90 days
   - Custom range
3. All metrics update automatically

### Best Practices
- **Weekly Reviews:** Use "Last 7 days"
- **Monthly Reports:** Use "Last 30 days"
- **Trend Analysis:** Use "Last 90 days"
- **Custom Campaigns:** Use custom range to match campaign dates

---

## Understanding Metrics

### Page Views
**Definition:** Total number of page loads
**Includes:** Duplicate views from same user
**Use For:** Understanding total traffic volume

### Unique Visitors
**Definition:** Number of distinct users (based on browser session)
**Excludes:** Same user visiting multiple times
**Use For:** Understanding reach and audience size

### Returning Visitors
**Definition:** Users who visited before in the last 30 days
**Formula:** Total visitors - Unique visitors
**Use For:** Understanding user retention

### Top Pages
**Definition:** Most visited pages ranked by view count
**Shows:** Path (e.g., `/rankings`, `/quiz`)
**Use For:** Identifying popular content

---

## Troubleshooting

### Problem: No Data Showing
**Possible Causes:**
- Just deployed - wait 10-15 minutes for data to propagate
- Selected date range has no traffic
- Analytics not deployed (check `src/app/layout.tsx` has `<Analytics />`)

**Solution:**
- Wait 15 minutes after deployment
- Change date range to "Last 30 days"
- Verify Analytics component is in production build

---

### Problem: Custom Events Missing
**Possible Causes:**
- Events not triggered (no user clicked the button)
- Development mode (events don't track in `npm run dev`)
- Code not deployed to production

**Solution:**
- Test in production environment
- Trigger events manually (click CTA, expand row, etc.)
- Wait 10 minutes for events to appear

---

### Problem: Core Web Vitals Missing
**Possible Causes:**
- Not enough real user data collected yet
- Need 50+ page views for reliable metrics

**Solution:**
- Wait for more traffic (need 50+ views)
- Check back in 24-48 hours

---

## Data Retention

### Free Tier (Current)
- **Page Views:** 30 days
- **Custom Events:** 30 days
- **Core Web Vitals:** 30 days

### Pro Tier (If Upgraded)
- **Page Views:** 90 days
- **Custom Events:** 90 days
- **Core Web Vitals:** 90 days
- **Audiences:** 90 days

**Recommendation:** Export data monthly if you need longer retention.

---

## Quick Reference Card

**Dashboard URL:** https://vercel.com → chainsights → Analytics

**Key Metrics:**
- Page Views: `/rankings` should be #1 or #2 page
- Custom Events: 4 events (`cta_click`, `social_share`, `row_expand`, `methodology_view`)
- Core Web Vitals: LCP <2.5s, FCP <1.8s, CLS <0.1

**Weekly Review:**
- Check total traffic (last 7 days)
- Calculate conversion funnel rates
- Verify Core Web Vitals are green
- Document findings in [Weekly Review](./reviews/YYYY-MM-DD.md)

---

## Related Documents

- [Conversion Funnel Guide](./conversion-funnel.md) - How to calculate conversion rates
- [Weekly Review Template](./weekly-review-template.md) - Template for weekly analytics reviews
- [Analytics README](./README.md) - Complete analytics documentation

---

## Questions?

Contact: Mario (Product Manager)
Email: [Internal - see project-context.md]
